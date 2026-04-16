# UX Guidelines and Device Targets

Design principles, screen considerations, and device compatibility for Garmin Connect IQ apps.

## Screen Shapes

| Shape | Constant | Devices |
|---|---|---|
| Round | `System.SCREEN_SHAPE_ROUND` | fenix, Forerunner, Venu, Enduro, epix |
| Semi-Round | `System.SCREEN_SHAPE_SEMI_ROUND` | Vivoactive 3 |
| Rectangle | `System.SCREEN_SHAPE_RECTANGLE` | Venu Sq, Edge cycling computers |
| Semi-Octagon | `System.SCREEN_SHAPE_SEMI_OCTAGON` | Instinct series (partial) |

```mc
' Detect screen shape at runtime
var shape = System.getDeviceSettings().screenShape;

if (shape == System.SCREEN_SHAPE_ROUND) {
    ' Most common — design for this first
} else if (shape == System.SCREEN_SHAPE_RECTANGLE) {
    ' More horizontal space, less wasted corners
}
```

## Display Technologies

### Memory-In-Pixel (MIP)

- Used by: fenix, Forerunner (non-AMOLED), Instinct, Enduro, Tactix
- Always-on, no burn-in risk
- Limited color palette (8-64 colors depending on device)
- Colors automatically mapped to nearest available
- Excellent outdoor visibility
- Low power consumption
- `onPartialUpdate()` supported for second-level updates
- No anti-aliasing support
- No alpha/transparency

### AMOLED

- Used by: Venu, epix, Forerunner 265/965, fenix 8 AMOLED
- Full 16-bit color (65,536 colors)
- Alpha transparency supported (0xAARRGGBB)
- Anti-aliasing available: `dc.setAntiAlias(true)`
- AnimationLayer for smooth animations
- **Burn-in risk**: Implement `onEnterSleep()` to:
  - Reduce bright pixels
  - Shift content position slightly
  - Use dim colors or minimal content
  - Avoid static bright elements
- Higher power consumption when screen is on
- Screen turns off to save battery (raise-to-wake)

### LCD

- Used by: Edge cycling computers, some older devices
- Full color support
- Backlight required for visibility in dark
- Standard rendering capabilities

## Common Resolutions

| Resolution | Devices |
|---|---|
| 240x240 | Forerunner 245/255, Vivoactive 4, Venu Sq |
| 260x260 | Forerunner 265, Venu 2, Venu 3 |
| 280x280 | Forerunner 955/965 |
| 390x390 | fenix 7/8, epix |
| 416x416 | Venu 3 (larger) |
| 454x454 | fenix 7X/8X, epix 2 Pro |
| 176x176 | Instinct, Forerunner 55 |
| 208x208 | Instinct 2, Forerunner 158 |

```mc
' Always use relative positioning, not hardcoded pixels
var width = dc.getWidth();
var height = dc.getHeight();
var centerX = width / 2;
var centerY = height / 2;
```

## Touch vs Button-Only Devices

### Touch Screen Devices

- Venu series, Vivoactive, Forerunner 265/965, epix, fenix 8 (some)
- Support tap, swipe, hold, drag gestures
- Use `BehaviorDelegate` for cross-device compatibility
- Can use `InputDelegate` for direct touch events

### Button-Only Devices

- Most fenix (non-touch), Forerunner 245/255/955, Instinct, Enduro
- 5-button layout: UP, DOWN, SELECT (START), BACK (LAP), LIGHT
- All navigation through button presses
- Use `BehaviorDelegate` — maps buttons to logical actions automatically

```mc
' BehaviorDelegate works on BOTH touch and button devices
class MyDelegate extends WatchUi.BehaviorDelegate {
    function onSelect() as Boolean { }     ' START button or screen tap
    function onBack() as Boolean { }       ' BACK button or swipe right
    function onNextPage() as Boolean { }   ' DOWN button or swipe up
    function onPreviousPage() as Boolean { }' UP button or swipe down
    function onMenu() as Boolean { }       ' UP long-press
}

' Detect touch support
var isTouchScreen = System.getDeviceSettings().isTouchScreen;
```

## Design Principles

### Glanceable Information

- Users look at their watch for 2-3 seconds
- **Most important information first**: Time > primary metric > secondary info
- Use large, readable fonts for key data
- Limit to 3-5 pieces of information per screen
- Use color to highlight status (green = good, red = alert)

### Readability

```mc
' Font size guidelines by importance
' Primary data (time, main metric): FONT_NUMBER_HOT or FONT_NUMBER_MEDIUM
' Secondary data (date, sub-metric):  FONT_MEDIUM or FONT_SMALL
' Labels and tertiary info:            FONT_SMALL or FONT_TINY
' Fine print / status:                 FONT_XTINY

' Ensure contrast
' Light text on dark background (preferred for AMOLED — saves battery)
' Dark text on light background (preferred for MIP — better outdoor visibility)
```

### Round Screen Layout

```
         12 o'clock
           ╭───╮
          ╱     ╲        Usable area is smaller than the full circle
         │  TOP  │       Corners are clipped on round screens
         │       │
         │ CENTER│       Center the most important content
         │       │
         │ BOTTOM│       Keep margins from edges (15-20px)
          ╲     ╱
           ╰───╯
         6 o'clock
```

- Don't place content in corners (clipped on round displays)
- Center important elements
- Use arc-based layouts for information around the edge
- Keep 15-20px margin from screen edge

### Color Guidelines

**MIP Displays** (limited palette):
```mc
' Safe colors that render well on MIP
Graphics.COLOR_WHITE;
Graphics.COLOR_BLACK;
Graphics.COLOR_RED;
Graphics.COLOR_GREEN;
Graphics.COLOR_BLUE;
Graphics.COLOR_YELLOW;
Graphics.COLOR_ORANGE;
Graphics.COLOR_DK_GRAY;
Graphics.COLOR_LT_GRAY;

' Avoid subtle color differences — they may map to the same MIP color
' Test on MIP simulator to verify color rendering
```

**AMOLED Displays**:
```mc
' Full color available — but prefer dark themes
' Dark background = less battery drain
' Pure black (0x000000) pixels are completely off on AMOLED

' Use vibrant accent colors on dark backgrounds
' Avoid large areas of bright white
' Use dimmer colors for always-on / sleep mode
```

## Connect IQ Version Compatibility

| CIQ Version | Key Features Added |
|---|---|
| 1.x | Basic app types, limited API |
| 2.0 | Enhanced graphics, BLE |
| 2.4 | Application.Storage, Application.Properties |
| 3.0 | Enhanced type checking, improved graphics |
| 3.1 | GlanceView for widgets |
| 3.2 | AnimationLayer (AMOLED) |
| 4.0 | Complications, mandatory glances, enhanced drawables |
| 5.0 | Weather API, improved AMOLED support |
| 6.0 | Latest features, newest devices |

```mc
' Set minimum SDK version in manifest.xml
' minSdkVersion="3.1.0" — supports CIQ 3.1.0 and above

' Always check feature availability at runtime
if (Toybox.WatchUi has :GlanceView) { }       ' CIQ 3.1+
if (Toybox.Weather has :getCurrentConditions) { } ' CIQ 5.0+
if (Toybox.Complications has :registerComplicationProvider) { } ' CIQ 4.0+
```

## Device-Specific Resources

Override resources per screen shape or specific device.

```
resources/                      ' Default (fallback)
resources-round/                ' All round devices
resources-round-240x240/        ' Round, 240x240 specifically
resources-rectangle/            ' All rectangular devices
resources-fenix7/               ' fenix 7 specifically
```

The build system uses the most specific match:
1. Device-specific (`resources-fenix7/`)
2. Resolution-specific (`resources-round-240x240/`)
3. Shape-specific (`resources-round/`)
4. Default (`resources/`)

## Watch Face UX

- **Update frequency**: Once per minute (save battery)
- **Always visible**: Must look good at all times
- **Sleep mode**: Reduce complexity (fewer bright pixels for AMOLED, stop second hand for MIP)
- **Complications**: Support reading data from other apps (CIQ 4.0+)
- **Customization**: Let users configure colors, data fields, layout via Properties

## Data Field UX

- **Minimal content**: Usually displays 1-2 values
- **Activity context**: Adapts to light/dark activity backgrounds
- **Fast updates**: `compute()` called every second — keep it fast
- **Clear labels**: Include field label so user knows what they're seeing

## Widget UX

- **Fast loading**: Users scroll quickly through widget carousel
- **Glance first**: Show key info in the 64px glance strip
- **Full view on demand**: Detailed info when user taps the glance
- **Auto-refresh**: Use timers to update data periodically

## Device App UX

- **Clear navigation**: Use menus and consistent back behavior
- **Confirmation for actions**: Confirm before destructive operations
- **Progress feedback**: Show progress bars for long operations
- **Respect exit**: BACK from main view should exit cleanly
- **Save state**: Persist user progress across sessions

## Performance Guidelines

- **Memory**: Monitor with `System.getSystemStats().usedMemory`
- **Draw time**: Keep `onUpdate()` under 100ms
- **Sensor polling**: Only enable sensors you need; disable when done
- **HTTP requests**: Cache responses; respect rate limits
- **Timers**: Use reasonable intervals (1s minimum for display, 5min+ for background)
- **Bitmaps**: Pre-size to display dimensions; avoid runtime scaling
- **Strings**: Minimize concatenation in loops (creates garbage)
- **Containers**: Pre-allocate arrays when size is known
