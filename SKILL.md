---
name: monkey-c-garmin
description: >
  Garmin Connect IQ and Monkey C wearable development. Use when writing or editing
  Monkey C (.mc) files, working with Connect IQ projects (.jungle, manifest.xml),
  building Garmin watch faces, data fields, widgets, device apps, or audio content
  providers, using the Toybox API, or targeting Garmin wearable devices.
---

# Monkey C / Garmin Connect IQ Development

## When to Use This Skill

- Writing or editing `.mc` (Monkey C) source files
- Creating Garmin Connect IQ apps: watch faces, data fields, widgets, device apps, audio content providers
- Working with `manifest.xml`, `.jungle` build files, or Connect IQ resource XML
- Using the Toybox API (`Toybox.*` modules)
- Debugging or optimizing for Garmin wearable devices
- Setting up a Connect IQ development environment

## Syntax Quick Reference

Monkey C looks like Java/JavaScript but has critical differences. Pay close attention.

### Comments

```mc
' This is a single-line comment (apostrophe, NOT //)

#rem
This is a block comment.
Block comments can be nested.
#end
```

### Literals and Keywords

```mc
var decimal = 123;
var hex = $FF;              ' Dollar sign prefix, NOT 0xFF
var float = 3.14f;
var long = 12345678l;
var double = 3.14d;
var char = 'A';             ' Single character
var str = "Hello World";
var bool = true;            ' true / false
var nothing = null;
var arr = [1, 2, 3];
var dict = { "a" => 1, "b" => 2 };
```

### Imports

```mc
using Toybox.WatchUi;       ' NOT import — use "using" keyword
using Toybox.Graphics;
using Toybox.System;
using Toybox.Lang;
using Toybox.Application;

' After importing, use short names:
' WatchUi.View, Graphics.Dc, System.println()
```

### String Operations

```mc
var greeting = "Hello" + " " + "World";   ' Concatenation with +
var formatted = Lang.format("$1$ is $2$", ["age", 25]);  ' Positional formatting
var len = str.length();
var sub = str.substring(0, 5);
var upper = str.toUpper();
var found = str.find("World");  ' Returns index or null
```

### Enums

```mc
enum {
    VALUE_A,         ' 0
    VALUE_B,         ' 1
    VALUE_C = 10,    ' 10
    VALUE_D          ' 11
}

' Typed enum
enum MyEnum {
    OPTION_ONE = "one",
    OPTION_TWO = "two"
}
```

### Type Definitions

```mc
typedef NumberOrString as Number or String;
typedef Callback as Method(value as Number) as Void;
```

### Type Checks

```mc
if (value instanceof Number) {
    ' Compiler narrows type in this branch
    var result = value + 1;
}

if (obj has :methodName) {
    ' Duck-type check: obj has the symbol :methodName
    obj.methodName();
}
```

## Naming Conventions

These are enforced by the Garmin community and expected by the compiler/tools:

| Identifier | Convention | Example |
|---|---|---|
| Classes | PascalCase | `MyWatchView`, `DataFieldApp` |
| Functions/Methods | camelCase | `onUpdate`, `getInitialView`, `initialize` |
| Variables/Fields | camelCase | `heartRate`, `currentTime` |
| Constants | ALL_CAPS | `MAX_HEART_RATE`, `DEFAULT_COLOR` |
| Enums | ALL_CAPS | `MODE_RUNNING`, `STATE_IDLE` |
| Modules | PascalCase | `Toybox.WatchUi` |
| Symbols | colon prefix | `:mySymbol`, `:onSelect` |
| Files/source | PascalCase | `MyWatchView.mc`, `DataFieldApp.mc` |

## Type System

```mc
' Duck-typed by default — no type annotations required
function add(a, b) {
    return a + b;  ' Works with Number, Float, String, etc.
}

' Strict typing with annotations
function add(a as Number, b as Number) as Number {
    return a + b;
}

' Nullable types
var sensor as Number? = null;  ' Can be Number or null

' Union types
var value as Number or String = 42;

' Typed containers
var numbers as Array<Number> = [1, 2, 3];
var lookup as Dictionary<String, Number> = { "a" => 1 };

' Type assertion
var hr = Activity.getActivityInfo().currentHeartRate as Number;
```

**Type checking modes** (set in project settings):
- **Gradual** (default): Warnings for obvious issues
- **Informative**: More warnings, helps migration to strict
- **Strict**: All types required, null safety enforced

## Memory Management

- **Reference counting** — not garbage collected
- **NEVER create circular references** — they leak permanently
- Typical app memory: 16KB–128KB depending on device
- Use `Toybox.Lang.WeakReference` when back-references are needed
- Set references to `null` when done; call `.clear()` on containers
- Monitor usage: `System.getSystemStats().usedMemory`

```mc
' BAD: Circular reference — will leak
class Parent {
    var child;
    function setChild(c) { child = c; }
}
class Child {
    var parent;  ' Creates circular ref
    function setParent(p) { parent = p; }
}

' GOOD: Use WeakReference for back-references
class Child {
    var parentRef as WeakReference;
    function setParent(p) { parentRef = p.weak(); }
    function getParent() { return parentRef.get(); }
}
```

## App Types

| Type | Base Class | Entry Method | Glance | Runs When |
|---|---|---|---|---|
| Watch Face | `WatchUi.WatchFace` | `getInitialView()` | No | Always on watch face |
| Data Field | `Application.AppBase` | `getInitialView()` | No | During activities |
| Widget | `Application.AppBase` | `getInitialView()` | Required CIQ 4+ | User-launched |
| Device App | `Application.AppBase` | `getInitialView()` | Required CIQ 4+ | User-launched |
| Audio Content Provider | `Application.AppBase` | `getInitialView()` | Optional | Media playback |

## Application Lifecycle

```mc
using Toybox.Application;
using Toybox.WatchUi;

class MyApp extends Application.AppBase {

    function initialize() {
        AppBase.initialize();  ' MUST call parent
    }

    function onStart(state as Dictionary?) as Void {
        ' Called when app starts
    }

    function getInitialView() as [Views] or [Views, InputDelegates] {
        return [new MyView(), new MyDelegate()];
    }

    ' Required for widgets/device apps on CIQ 4+ devices
    function getGlanceView() as [GlanceView] or [GlanceView, GlanceViewDelegate] or Null {
        return [new MyGlanceView()];
    }

    function onStop(state as Dictionary?) as Void {
        ' Called when app exits
    }

    ' Called when background service returns data
    function onBackgroundData(data) as Void {
        ' Process background result
    }
}
```

## View Lifecycle

```mc
using Toybox.WatchUi;
using Toybox.Graphics;

class MyView extends WatchUi.View {

    function initialize() {
        View.initialize();  ' MUST call parent
    }

    ' Called once when view is created — set up layout
    function onLayout(dc as Dc) as Void {
        setLayout(Rez.Layouts.MainLayout(dc));
    }

    ' Called when view becomes visible
    function onShow() as Void { }

    ' Called to draw the screen — this is your main render method
    function onUpdate(dc as Dc) as Void {
        View.onUpdate(dc);  ' MUST call first — clears screen, draws layout

        ' Custom drawing after layout
        dc.setColor(Graphics.COLOR_WHITE, Graphics.COLOR_TRANSPARENT);
        dc.drawText(
            dc.getWidth() / 2,
            dc.getHeight() / 2,
            Graphics.FONT_MEDIUM,
            "Hello Garmin!",
            Graphics.TEXT_JUSTIFY_CENTER | Graphics.TEXT_JUSTIFY_VCENTER
        );
    }

    ' Called when view is hidden
    function onHide() as Void { }
}
```

### Input Handling

```mc
class MyDelegate extends WatchUi.BehaviorDelegate {

    function initialize() {
        BehaviorDelegate.initialize();
    }

    function onSelect() as Boolean {
        ' SELECT/START button or tap on touchscreen
        return true;  ' true = handled
    }

    function onBack() as Boolean {
        ' BACK button
        WatchUi.popView(WatchUi.SLIDE_RIGHT);
        return true;
    }

    function onMenu() as Boolean {
        ' MENU/UP long press
        WatchUi.pushView(new Rez.Menus.MainMenu(), new MyMenuDelegate(), WatchUi.SLIDE_UP);
        return true;
    }

    function onNextPage() as Boolean { return false; }  ' DOWN button
    function onPreviousPage() as Boolean { return false; }  ' UP button
}
```

## Graphics Essentials

- **Coordinate system**: Origin (0,0) at top-left; X increases right, Y increases down
- **Colors**: `0xRRGGBB` format (alpha `0xAARRGGBB` on AMOLED only)
- **Always call** `View.onUpdate(dc)` first in `onUpdate()` — it clears the screen

```mc
function onUpdate(dc as Dc) as Void {
    View.onUpdate(dc);

    var width = dc.getWidth();
    var height = dc.getHeight();
    var centerX = width / 2;
    var centerY = height / 2;

    ' Set foreground and background colors
    dc.setColor(Graphics.COLOR_WHITE, Graphics.COLOR_BLACK);

    ' Drawing methods
    dc.drawText(centerX, centerY, Graphics.FONT_MEDIUM, "Text", Graphics.TEXT_JUSTIFY_CENTER);
    dc.drawLine(0, 0, width, height);
    dc.drawRectangle(10, 10, 100, 50);
    dc.fillRectangle(10, 10, 100, 50);
    dc.drawCircle(centerX, centerY, 50);
    dc.fillCircle(centerX, centerY, 50);
    dc.drawArc(centerX, centerY, 80, Graphics.ARC_CLOCKWISE, 0, 90);

    ' Polygon (array of [x,y] points)
    dc.fillPolygon([[10,10], [50,10], [30,50]]);

    ' Bitmaps
    var bmp = WatchUi.loadResource(Rez.Drawables.MyBitmap);
    dc.drawBitmap(centerX - bmp.getWidth() / 2, centerY - bmp.getHeight() / 2, bmp);

    ' Text measurement
    var dims = dc.getTextDimensions("Hello", Graphics.FONT_MEDIUM);
    ' dims[0] = width, dims[1] = height

    ' Anti-aliasing (AMOLED devices only)
    dc.setAntiAlias(true);
}
```

**Built-in fonts**: `FONT_XTINY`, `FONT_TINY`, `FONT_SMALL`, `FONT_MEDIUM`, `FONT_LARGE`, `FONT_NUMBER_MILD`, `FONT_NUMBER_MEDIUM`, `FONT_NUMBER_HOT`, `FONT_NUMBER_THAI_HOT`, `FONT_SYSTEM_XTINY` through `FONT_SYSTEM_LARGE`

**Color constants**: `COLOR_BLACK`, `COLOR_DK_GRAY`, `COLOR_LT_GRAY`, `COLOR_WHITE`, `COLOR_RED`, `COLOR_DK_RED`, `COLOR_ORANGE`, `COLOR_YELLOW`, `COLOR_GREEN`, `COLOR_DK_GREEN`, `COLOR_BLUE`, `COLOR_DK_BLUE`, `COLOR_PURPLE`, `COLOR_PINK`, `COLOR_TRANSPARENT`

## Project Structure

```
my-project/
  manifest.xml            ' App metadata, permissions, devices
  monkey.jungle           ' Build configuration
  source/
    MyApp.mc              ' AppBase subclass (entry point)
    MyView.mc             ' View + InputDelegate
    MyDelegate.mc         ' Input handler
    MyGlanceView.mc       ' Glance view (CIQ 3.1.3+)
  resources/
    strings/strings.xml   ' Localized strings
    drawables/            ' Bitmaps, shapes, animations
    layouts/layout.xml    ' UI layout definitions
    properties.xml        ' App settings/properties
    menus/menu.xml        ' Menu definitions
```

### manifest.xml Key Elements

```xml
<iq:manifest xmlns:iq="http://www.garmin.com/xml/connectiq" version="3">
    <iq:application entry="MyApp" id="com.example.myapp"
                    launcherIcon="@Drawables.LauncherIcon"
                    minSdkVersion="3.1.0" name="@Strings.AppName"
                    type="watch-app">
        <iq:permissions>
            <iq:uses-permission id="Sensor" />
            <iq:uses-permission id="Communications" />
            <iq:uses-permission id="Positioning" />
        </iq:permissions>
        <iq:languages>
            <iq:language>eng</iq:language>
        </iq:languages>
        <iq:devices>
            <iq:product id="fenix7" />
            <iq:product id="venu2" />
        </iq:devices>
    </iq:application>
</iq:manifest>
```

**App types in manifest**: `watch-app`, `watchface`, `datafield`, `widget`, `audio-content-provider`

## Data Persistence

```mc
using Toybox.Application.Storage;
using Toybox.Application.Properties;

' --- Application.Storage: app-managed key-value store ---
Storage.setValue("lastSync", Time.now().value());
var lastSync = Storage.getValue("lastSync");
Storage.deleteValue("lastSync");

' --- Application.Properties: user-configurable settings ---
' Values MUST be defined in resources/properties.xml first
var threshold = Properties.getValue("batteryThreshold") as Number;
Properties.setValue("batteryThreshold", 20);
```

**Supported types**: `Number`, `Float`, `Long`, `Double`, `Boolean`, `String`, `Array`, `Dictionary`

## Background Services

```mc
' In AppBase — register temporal event
function getServiceDelegate() as [ServiceDelegate] {
    return [new MyServiceDelegate()];
}

' Service delegate — runs in background
(:background)
class MyServiceDelegate extends System.ServiceDelegate {
    function initialize() {
        ServiceDelegate.initialize();
    }

    function onTemporalEvent() as Void {
        ' Do background work (HTTP requests, etc.)
        ' Maximum 30 seconds execution time
        Background.exit(resultData);  ' Return data to foreground
    }
}

' Register timer — MINIMUM 5 minutes between events
Background.registerForTemporalEvent(new Time.Duration(300));
```

## Glances (CIQ 3.1.3+)

Required for widgets and device apps on CIQ 4.0+ devices. A 64px-tall strip showing a preview.

```mc
class MyGlanceView extends WatchUi.GlanceView {
    function initialize() {
        GlanceView.initialize();
    }

    function onUpdate(dc as Dc) as Void {
        dc.setColor(Graphics.COLOR_WHITE, Graphics.COLOR_BLACK);
        dc.clear();
        dc.drawText(0, dc.getHeight() / 2, Graphics.FONT_GLANCE, "My App", Graphics.TEXT_JUSTIFY_LEFT | Graphics.TEXT_JUSTIFY_VCENTER);
    }
}
```

## Common Pitfalls

1. **Forgetting parent `initialize()`**: Always call `View.initialize()`, `AppBase.initialize()`, `BehaviorDelegate.initialize()` etc. in constructors
2. **Not calling `View.onUpdate(dc)`**: Must be called first in `onUpdate()` — clears the screen and draws the layout
3. **Circular references**: Will permanently leak memory. Use `WeakReference` for back-references
4. **Using `//` for comments**: Monkey C uses apostrophe `'` for single-line comments, `#rem/#end` for blocks
5. **Using `0x` hex prefix**: Monkey C uses `$FF` dollar-sign prefix
6. **Using `import`**: Monkey C uses `using Toybox.Module;`
7. **Not calling `WatchUi.requestUpdate()`**: Views only redraw when explicitly requested (except watch faces which update on system timer)
8. **Null from sensors**: Sensor data often returns `null` — always check before using
9. **Exceeding memory**: Wearables have very limited RAM. Monitor with `System.getSystemStats()`
10. **Blocking main thread**: No long-running operations. Use background services or `Communications.makeWebRequest()` (async)

## Reference Files

For deeper information on specific topics, load these reference files:

- [Language Syntax Deep Dive](references/language-syntax.md) — operators, control flow, containers, strings, symbols, scope, inheritance
- [App Types and Lifecycle](references/app-types-and-lifecycle.md) — detailed lifecycle for each app type, view stack management
- [UI, Graphics, and Layouts](references/ui-graphics-layouts.md) — full Dc API, layout XML, input handling, animations
- [Toybox API Reference](references/toybox-api-reference.md) — all 37 Toybox modules with key classes and methods
- [Project Structure and Build](references/project-structure-and-build.md) — manifest.xml schema, jungle files, resources, barrels
- [Sensors, Comms, and Background](references/sensors-comms-background.md) — sensors, GPS, BLE, ANT+, HTTP, background services
- [Data Persistence and Properties](references/data-persistence-and-properties.md) — Storage, Properties, settings UI
- [UX Guidelines and Device Targets](references/ux-guidelines-and-device-targets.md) — screen types, UX patterns, device compatibility
- [Common Patterns and Examples](references/common-patterns-and-examples.md) — complete working templates for each app type
