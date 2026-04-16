# UI, Graphics, and Layouts

Complete reference for the Connect IQ drawing API, layout system, input handling, and UI components.

## Drawing Context (Dc)

The `Graphics.Dc` object is passed to `onUpdate()` and provides all drawing operations. The coordinate system has origin (0,0) at the top-left corner, with X increasing right and Y increasing down.

### Color System

```mc
' Colors are 24-bit RGB: 0xRRGGBB
dc.setColor(0xFF0000, Graphics.COLOR_BLACK);  ' Red foreground, black background

' AMOLED devices support 32-bit with alpha: 0xAARRGGBB
dc.setColor(0x80FF0000, Graphics.COLOR_TRANSPARENT);  ' Semi-transparent red

' Built-in color constants
Graphics.COLOR_BLACK;       ' 0x000000
Graphics.COLOR_DK_GRAY;    ' 0x555555
Graphics.COLOR_LT_GRAY;    ' 0xAAAAAA
Graphics.COLOR_WHITE;       ' 0xFFFFFF
Graphics.COLOR_RED;         ' 0xFF0000
Graphics.COLOR_DK_RED;      ' 0xAA0000
Graphics.COLOR_ORANGE;      ' 0xFF5500
Graphics.COLOR_YELLOW;      ' 0xFFAA00
Graphics.COLOR_GREEN;       ' 0x00FF00
Graphics.COLOR_DK_GREEN;    ' 0x00AA00
Graphics.COLOR_BLUE;        ' 0x0055FF
Graphics.COLOR_DK_BLUE;     ' 0x0000AA
Graphics.COLOR_PURPLE;      ' 0x550055
Graphics.COLOR_PINK;        ' 0xFF00FF
Graphics.COLOR_TRANSPARENT; ' Transparent (no fill)

' MIP displays have limited color palettes (8-64 colors)
' Colors are automatically mapped to nearest available color
```

### Complete Drawing API

```mc
function onUpdate(dc as Dc) as Void {
    ' ALWAYS call parent first to clear screen
    View.onUpdate(dc);

    ' --- Text ---
    dc.drawText(x, y, font, text, justification);
    ' justification: combine horizontal and vertical with |
    ' Graphics.TEXT_JUSTIFY_LEFT | Graphics.TEXT_JUSTIFY_VCENTER

    ' --- Lines ---
    dc.drawLine(x1, y1, x2, y2);
    dc.setPenWidth(3);  ' Set line thickness (default 1)

    ' --- Rectangles ---
    dc.drawRectangle(x, y, width, height);       ' Outline
    dc.fillRectangle(x, y, width, height);        ' Filled
    dc.drawRoundedRectangle(x, y, w, h, radius);  ' Rounded outline
    dc.fillRoundedRectangle(x, y, w, h, radius);  ' Rounded filled

    ' --- Circles ---
    dc.drawCircle(centerX, centerY, radius);      ' Outline
    dc.fillCircle(centerX, centerY, radius);      ' Filled

    ' --- Arcs ---
    dc.drawArc(centerX, centerY, radius, direction, startAngle, endAngle);
    ' direction: Graphics.ARC_CLOCKWISE or Graphics.ARC_COUNTER_CLOCKWISE
    ' Angles in degrees: 0 = 3 o'clock, 90 = 12 o'clock, 180 = 9 o'clock, 270 = 6 o'clock

    ' --- Polygons ---
    var points = [[10, 10], [50, 10], [30, 50]];
    dc.fillPolygon(points);  ' Fill only (no drawPolygon)

    ' --- Bitmaps ---
    dc.drawBitmap(x, y, bitmapResource);
    dc.drawScaledBitmap(x, y, width, height, bitmapResource);  ' CIQ 4.0+

    ' --- Clear ---
    dc.clear();  ' Fill entire screen with background color
    dc.setColor(fg, bg);  ' First param = foreground (text/lines), second = background (fill)

    ' --- Clipping ---
    dc.setClip(x, y, width, height);  ' Restrict drawing area
    dc.clearClip();                     ' Remove clipping

    ' --- Anti-aliasing (AMOLED only) ---
    dc.setAntiAlias(true);

    ' --- Screen dimensions ---
    var width = dc.getWidth();
    var height = dc.getHeight();
}
```

### Text Measurement

```mc
' Get text dimensions
var dims = dc.getTextDimensions("Hello World", Graphics.FONT_MEDIUM);
var textWidth = dims[0];
var textHeight = dims[1];

' Width only
var w = dc.getTextWidthInPixels("Hello", Graphics.FONT_MEDIUM);

' Font height
var h = dc.getFontHeight(Graphics.FONT_MEDIUM);
var ascent = dc.getFontAscent(Graphics.FONT_MEDIUM);
var descent = dc.getFontDescent(Graphics.FONT_MEDIUM);
```

### Text Justification Constants

```mc
' Horizontal
Graphics.TEXT_JUSTIFY_LEFT;
Graphics.TEXT_JUSTIFY_CENTER;
Graphics.TEXT_JUSTIFY_RIGHT;

' Vertical
Graphics.TEXT_JUSTIFY_VCENTER;

' Combine with bitwise OR
dc.drawText(x, y, font, text, Graphics.TEXT_JUSTIFY_CENTER | Graphics.TEXT_JUSTIFY_VCENTER);
```

### Built-in Fonts

```mc
' System fonts (smallest to largest)
Graphics.FONT_XTINY;
Graphics.FONT_TINY;
Graphics.FONT_SMALL;
Graphics.FONT_MEDIUM;
Graphics.FONT_LARGE;

' System fonts (alternate names)
Graphics.FONT_SYSTEM_XTINY;
Graphics.FONT_SYSTEM_TINY;
Graphics.FONT_SYSTEM_SMALL;
Graphics.FONT_SYSTEM_MEDIUM;
Graphics.FONT_SYSTEM_LARGE;

' Number fonts (optimized for displaying numbers)
Graphics.FONT_NUMBER_MILD;
Graphics.FONT_NUMBER_MEDIUM;
Graphics.FONT_NUMBER_HOT;
Graphics.FONT_NUMBER_THAI_HOT;

' Glance font
Graphics.FONT_GLANCE;  ' For use in GlanceView only

' Custom fonts (loaded from resources)
var customFont = WatchUi.loadResource(Rez.Fonts.MyCustomFont);
dc.drawText(x, y, customFont, text, justification);
```

### Buffered Bitmaps

For offscreen rendering and caching complex drawings.

```mc
' Create offscreen buffer
var bmpRef = Graphics.createBufferedBitmap({
    :width => 100,
    :height => 100,
    :palette => [Graphics.COLOR_BLACK, Graphics.COLOR_WHITE, Graphics.COLOR_RED]
        ' Optional: limit palette for MIP displays
});

' Get the bitmap and its drawing context
var bmp = bmpRef.get() as BufferedBitmap;
var bmpDc = bmp.getDc();

' Draw to offscreen buffer
bmpDc.setColor(Graphics.COLOR_WHITE, Graphics.COLOR_BLACK);
bmpDc.clear();
bmpDc.fillCircle(50, 50, 40);

' Draw buffered bitmap to screen
dc.drawBitmap(x, y, bmp);
```

### Round Screen Math

Common patterns for drawing on round watch faces.

```mc
' Draw around the edge of a circular screen
function drawTickMarks(dc as Dc) as Void {
    var centerX = dc.getWidth() / 2;
    var centerY = dc.getHeight() / 2;
    var radius = centerX - 5;  ' Slight inset from edge

    dc.setColor(Graphics.COLOR_WHITE, Graphics.COLOR_TRANSPARENT);
    dc.setPenWidth(2);

    for (var i = 0; i < 12; i++) {
        var angle = i * 30.0;  ' 360 / 12
        var radians = Math.toRadians(angle - 90);  ' -90 to start at 12 o'clock

        var outerX = centerX + (radius * Math.cos(radians)).toNumber();
        var outerY = centerY + (radius * Math.sin(radians)).toNumber();
        var innerX = centerX + ((radius - 15) * Math.cos(radians)).toNumber();
        var innerY = centerY + ((radius - 15) * Math.sin(radians)).toNumber();

        dc.drawLine(innerX, innerY, outerX, outerY);
    }
}

' Draw analog watch hands
function drawHand(dc as Dc, angle as Float, length as Number, width as Number) as Void {
    var centerX = dc.getWidth() / 2;
    var centerY = dc.getHeight() / 2;
    var radians = Math.toRadians(angle - 90);

    var endX = centerX + (length * Math.cos(radians)).toNumber();
    var endY = centerY + (length * Math.sin(radians)).toNumber();

    dc.setPenWidth(width);
    dc.drawLine(centerX, centerY, endX, endY);
}
```

## Layout XML

### Layout File Structure

```xml
<!-- resources/layouts/layout.xml -->
<layout id="MainLayout">
    <label id="TimeLabel"
        x="center" y="40"
        font="Graphics.FONT_NUMBER_HOT"
        justification="Graphics.TEXT_JUSTIFY_CENTER"
        color="Graphics.COLOR_WHITE" />

    <label id="DateLabel"
        x="center" y="120"
        font="Graphics.FONT_SMALL"
        justification="Graphics.TEXT_JUSTIFY_CENTER"
        color="Graphics.COLOR_LT_GRAY" />

    <bitmap id="Icon"
        x="center" y="160"
        filename="../drawables/icon.png" />
</layout>
```

### Positioning

```xml
' Horizontal alignment
x="center"   ' LAYOUT_HALIGN_CENTER
x="left"     ' LAYOUT_HALIGN_LEFT
x="right"    ' LAYOUT_HALIGN_RIGHT
x="50"       ' Absolute pixels

' Vertical alignment
y="center"   ' LAYOUT_VALIGN_CENTER
y="top"      ' LAYOUT_VALIGN_TOP
y="bottom"   ' LAYOUT_VALIGN_BOTTOM
y="100"      ' Absolute pixels

' Expressions (computed at layout time)
x="(dc.getWidth() - 50) / 2"
```

### Loading and Using Layouts

```mc
function onLayout(dc as Dc) as Void {
    setLayout(Rez.Layouts.MainLayout(dc));
}

function onUpdate(dc as Dc) as Void {
    ' Update layout elements before drawing
    var timeLabel = findDrawableById("TimeLabel") as Text;
    timeLabel.setText("12:00");

    var dateLabel = findDrawableById("DateLabel") as Text;
    dateLabel.setText("Monday");

    ' Draw layout (call parent)
    View.onUpdate(dc);

    ' Additional custom drawing after layout
    dc.setColor(Graphics.COLOR_RED, Graphics.COLOR_TRANSPARENT);
    dc.fillCircle(100, 200, 5);
}
```

## Drawable Classes

Built-in drawables for use in layouts and programmatic UI.

```mc
' Text drawable
class WatchUi.Text extends Drawable {
    ' setText(text) - set display text
    ' setColor(color) - set text color
    ' setFont(font) - change font
    ' setJustification(justification)
    ' setLocation(x, y)
}

' Bitmap drawable
class WatchUi.Bitmap extends Drawable {
    ' setBitmap(resource)
    ' setLocation(x, y)
}

' All Drawable subclasses have:
' locX, locY — position
' width, height — dimensions
' setLocation(x, y)
' draw(dc) — render to drawing context
```

## Input Handling

### BehaviorDelegate (Recommended)

Handles logical actions mapped to physical buttons.

```mc
class MyDelegate extends WatchUi.BehaviorDelegate {
    function initialize() {
        BehaviorDelegate.initialize();
    }

    ' Physical button/gesture → Logical behavior mapping:
    function onSelect() as Boolean { return false; }      ' START/SELECT button or tap
    function onBack() as Boolean { return false; }        ' BACK button
    function onMenu() as Boolean { return false; }        ' UP long-press (opens menu)
    function onNextPage() as Boolean { return false; }    ' DOWN button or swipe up
    function onPreviousPage() as Boolean { return false; }' UP button or swipe down
    function onNextMode() as Boolean { return false; }    ' Mode switch
    function onPreviousMode() as Boolean { return false; }

    ' Return true if handled, false to pass to next delegate
}
```

### InputDelegate (Low-Level)

Direct button and touch events.

```mc
class MyInputDelegate extends WatchUi.InputDelegate {
    function initialize() {
        InputDelegate.initialize();
    }

    function onKey(keyEvent as KeyEvent) as Boolean {
        var key = keyEvent.getKey();
        if (key == WatchUi.KEY_ENTER) { }      ' START/SELECT
        if (key == WatchUi.KEY_ESC) { }        ' BACK
        if (key == WatchUi.KEY_UP) { }         ' UP
        if (key == WatchUi.KEY_DOWN) { }       ' DOWN
        if (key == WatchUi.KEY_MENU) { }       ' MENU (UP long-press)
        if (key == WatchUi.KEY_LAP) { }        ' BACK long-press
        return false;
    }

    function onTap(clickEvent as ClickEvent) as Boolean {
        var coords = clickEvent.getCoordinates();
        var x = coords[0];
        var y = coords[1];
        var type = clickEvent.getType();  ' CLICK_TYPE_TAP
        return false;
    }

    function onSwipe(swipeEvent as SwipeEvent) as Boolean {
        var direction = swipeEvent.getDirection();
        ' WatchUi.SWIPE_UP, SWIPE_DOWN, SWIPE_LEFT, SWIPE_RIGHT
        return false;
    }

    function onHold(clickEvent as ClickEvent) as Boolean {
        ' Long press on touch screen
        return false;
    }

    function onRelease(clickEvent as ClickEvent) as Boolean {
        return false;
    }

    function onDrag(dragEvent as DragEvent) as Boolean {
        return false;
    }
}
```

### Key Constants

```mc
WatchUi.KEY_ENTER;   ' START/SELECT
WatchUi.KEY_ESC;     ' BACK/LAP
WatchUi.KEY_UP;      ' UP
WatchUi.KEY_DOWN;    ' DOWN
WatchUi.KEY_MENU;    ' MENU
WatchUi.KEY_LAP;     ' LAP (BACK long-press on some devices)
WatchUi.KEY_POWER;   ' POWER
WatchUi.KEY_CLOCK;   ' CLOCK
```

## Native UI Controls

### Menu2 (Modern Menu)

```xml
<!-- resources/menus/menu.xml -->
<menu2 id="MainMenu" title="@Strings.MenuTitle">
    <menu-item id="option1" label="@Strings.Option1" />
    <menu-item id="option2" label="@Strings.Option2" />
    <toggle-menu-item id="toggle1" label="@Strings.Toggle" enabled="true" />
    <icon-menu-item id="icon1" label="@Strings.IconOption" icon="@Drawables.MenuIcon" />
</menu2>
```

```mc
' Show menu
WatchUi.pushView(new Rez.Menus.MainMenu(), new MainMenuDelegate(), WatchUi.SLIDE_UP);

class MainMenuDelegate extends WatchUi.Menu2InputDelegate {
    function initialize() {
        Menu2InputDelegate.initialize();
    }

    function onSelect(item as MenuItem) as Void {
        var id = item.getId();
        if (id == :option1) {
            ' Handle option 1
        } else if (id == :toggle1) {
            ' Toggle item
            var toggleItem = item as ToggleMenuItem;
            var enabled = toggleItem.isEnabled();
        }
    }

    function onBack() as Void {
        WatchUi.popView(WatchUi.SLIDE_DOWN);
    }
}
```

### Programmatic Menu

```mc
var menu = new WatchUi.Menu2({ :title => "Options" });
menu.addItem(new WatchUi.MenuItem("Item 1", "Subtitle", :item1, {}));
menu.addItem(new WatchUi.MenuItem("Item 2", null, :item2, {}));
menu.addItem(new WatchUi.ToggleMenuItem("Toggle", null, :toggle, true, {}));
WatchUi.pushView(menu, new MyMenuDelegate(), WatchUi.SLIDE_UP);
```

### Confirmation Dialog

```mc
var dialog = new WatchUi.Confirmation("Delete all data?");
WatchUi.pushView(dialog, new DeleteConfirmDelegate(), WatchUi.SLIDE_IMMEDIATE);

class DeleteConfirmDelegate extends WatchUi.ConfirmationDelegate {
    function initialize() {
        ConfirmationDelegate.initialize();
    }

    function onResponse(response as Confirm) as Boolean {
        if (response == WatchUi.CONFIRM_YES) {
            ' Confirmed
        }
        return true;
    }
}
```

### Progress Bar

```mc
' Show a progress bar during long operations
WatchUi.pushView(
    new WatchUi.ProgressBar("Loading...", null),
    null,
    WatchUi.SLIDE_DOWN
);

' Update progress (0.0 to 1.0)
' progressBar.setProgress(0.5);

' Dismiss when done
WatchUi.popView(WatchUi.SLIDE_UP);
```

### Number Picker

```mc
var picker = new WatchUi.NumberPicker(
    WatchUi.NUMBER_PICKER_NORMAL,
    new [3]  ' 3 digits
);
WatchUi.pushView(picker, new NumberPickerDelegate(), WatchUi.SLIDE_UP);
```

## AnimationLayer (AMOLED Devices)

For smooth animations on AMOLED displays.

```mc
' AnimationLayer provides a separate drawing layer for animations
' Only available on AMOLED devices (CIQ 5.0+)

if (WatchUi has :AnimationLayer) {
    ' Device supports animation layers
    var layer = new WatchUi.AnimationLayer(
        method(:onAnimationUpdate),
        { :period => 33 }  ' ~30fps
    );
}
```

## Multi-Shape Screen Support

Pattern for supporting both round and rectangular screens.

```mc
function onUpdate(dc as Dc) as Void {
    View.onUpdate(dc);

    var settings = System.getDeviceSettings();
    var shape = settings.screenShape;

    if (shape == System.SCREEN_SHAPE_ROUND) {
        drawRoundLayout(dc);
    } else if (shape == System.SCREEN_SHAPE_RECTANGLE) {
        drawRectLayout(dc);
    } else if (shape == System.SCREEN_SHAPE_SEMI_ROUND) {
        drawSemiRoundLayout(dc);
    } else if (shape == System.SCREEN_SHAPE_SEMI_OCTAGON) {
        drawSemiOctagonLayout(dc);
    }
}
```

## Device-Specific Resources

Use resource qualifiers to provide different layouts/drawables per device shape.

```
resources/
  layouts/layout.xml            ' Default layout
resources-round/
  layouts/layout.xml            ' Round screen layout
resources-rectangle/
  layouts/layout.xml            ' Rectangle screen layout
resources-round-240x240/
  layouts/layout.xml            ' Specific resolution
```

The build system automatically selects the most specific matching resource directory.
