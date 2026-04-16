# App Types and Lifecycle

Detailed reference for each Connect IQ application type, their lifecycle, view management, and input handling.

## Watch Face

The most common app type. Displays time and information on the main watch screen.

### Manifest Type

```xml
<iq:application type="watchface" ... >
```

### Base Class

```mc
class MyWatchFace extends WatchUi.WatchFace {
    function initialize() {
        WatchFace.initialize();
    }

    ' Called every minute (or every second if onPartialUpdate is implemented)
    function onUpdate(dc as Dc) as Void {
        dc.setColor(Graphics.COLOR_WHITE, Graphics.COLOR_BLACK);
        dc.clear();
        ' Draw watch face content
    }

    ' Low-power mode — reduce visual complexity, hide seconds
    function onEnterSleep() as Void {
        WatchUi.requestUpdate();
    }

    function onExitSleep() as Void {
        WatchUi.requestUpdate();
    }

    ' Partial update — for second hand on MIP displays
    ' Called every second, even in sleep mode on some devices
    ' Use clipping to only redraw the seconds area (saves battery)
    function onPartialUpdate(dc as Dc) as Void {
        var seconds = System.getClockTime().sec;
        dc.setClip(100, 100, 60, 30);
        dc.setColor(Graphics.COLOR_WHITE, Graphics.COLOR_BLACK);
        dc.fillRectangle(100, 100, 60, 30);
        dc.drawText(130, 115, Graphics.FONT_TINY, seconds.format("%02d"),
            Graphics.TEXT_JUSTIFY_CENTER | Graphics.TEXT_JUSTIFY_VCENTER);
        dc.clearClip();
    }
}
```

### Watch Face Constraints

- **No direct input handling** — no BehaviorDelegate for watch faces
- **Cannot push views** — watch faces are always the only view
- **No sensors** — use `ActivityMonitor` for HR, steps, etc.
- **Battery critical** — minimize processing, avoid frequent redraws
- **Always-on display** (AMOLED): Implement `onEnterSleep()` to show minimal content with fewer bright pixels
- **MIP displays**: Use `onPartialUpdate()` for second-level updates with clipping
- System calls `onUpdate()` once per minute unless `onPartialUpdate()` is implemented

### Watch Face Data Sources

```mc
' Time
var clock = System.getClockTime();
clock.hour; clock.min; clock.sec;

' Date
var today = Gregorian.info(Time.now(), Time.FORMAT_MEDIUM);
today.day; today.month; today.day_of_week;

' Battery
var stats = System.getSystemStats();
stats.battery; stats.charging;

' Steps / Activity
var actInfo = ActivityMonitor.getInfo();
actInfo.steps; actInfo.stepGoal; actInfo.calories;

' Heart Rate (from activity monitor, works in watch faces)
var hrIter = ActivityMonitor.getHeartRateHistory(1, true);
var sample = hrIter.next();
if (sample != null && sample.heartRate != ActivityMonitor.INVALID_HR_SAMPLE) {
    var hr = sample.heartRate;
}

' Device settings
var settings = System.getDeviceSettings();
settings.phoneConnected; settings.alarmCount; settings.notificationCount;

' Weather (CIQ 5.0+)
if (Toybox.Weather has :getCurrentConditions) {
    var weather = Weather.getCurrentConditions();
}
```

## Data Field

Custom metrics displayed during an activity.

### Manifest Type

```xml
<iq:application type="datafield" ... >
```

### Base Class

```mc
class MyDataField extends WatchUi.DataField {
    hidden var _value as Number = 0;

    function initialize() {
        DataField.initialize();
    }

    ' Called every second during an activity
    function compute(info as Activity.Info) as Numeric or Duration or String or Null {
        ' info contains all current activity data
        if (info.currentHeartRate != null) {
            _value = info.currentHeartRate as Number;
        }

        ' Return value is displayed in simple data field layouts
        return _value;
    }

    function onUpdate(dc as Dc) as Void {
        ' For custom layouts — draw your own content
        var bgColor = getBackgroundColor();
        var fgColor = (bgColor == Graphics.COLOR_BLACK) ?
            Graphics.COLOR_WHITE : Graphics.COLOR_BLACK;

        dc.setColor(fgColor, bgColor);
        dc.clear();

        dc.drawText(dc.getWidth() / 2, dc.getHeight() / 2,
            Graphics.FONT_NUMBER_MEDIUM, _value.format("%d"),
            Graphics.TEXT_JUSTIFY_CENTER | Graphics.TEXT_JUSTIFY_VCENTER);
    }

    ' Label shown above the data field
    function getFieldLabel() as String {
        return "My Field";
    }
}
```

### Activity.Info Fields Available in compute()

```mc
info.currentHeartRate;      ' Number? — bpm
info.averageHeartRate;      ' Number?
info.maxHeartRate;          ' Number?
info.currentSpeed;          ' Float? — m/s
info.averageSpeed;          ' Float? — m/s
info.maxSpeed;              ' Float? — m/s
info.currentCadence;        ' Number? — rpm or spm
info.averageCadence;        ' Number?
info.maxCadence;            ' Number?
info.currentPower;          ' Number? — watts
info.averagePower;          ' Number?
info.maxPower;              ' Number?
info.elapsedDistance;        ' Float? — meters
info.elapsedTime;           ' Number? — milliseconds
info.timerTime;             ' Number? — active timer milliseconds
info.calories;              ' Number?
info.currentLocation;       ' Position.Location?
info.totalAscent;           ' Float? — meters
info.totalDescent;          ' Float? — meters
info.currentOxygenSaturation; ' Number? — SpO2 %
info.altitude;              ' Float? — meters
info.timerState;            ' TIMER_STATE_OFF, ON, PAUSED, STOPPED
info.startTime;             ' Moment?
info.startLocation;         ' Position.Location?
info.ambientPressure;       ' Float? — Pascals
info.meanSeaLevelPressure;  ' Float? — Pascals
```

### FIT Contributions

Add custom fields to the recorded FIT file.

```mc
using Toybox.FitContributor;

class MyDataField extends WatchUi.DataField {
    hidden var _fitField as FitContributor.Field?;

    function initialize() {
        DataField.initialize();

        _fitField = createField("customMetric", 0, FitContributor.DATA_TYPE_FLOAT, {
            :mesgType => FitContributor.MESG_TYPE_RECORD,
            :units => "units"
        });
    }

    function compute(info as Activity.Info) as Void {
        var value = calculateMyMetric(info);
        if (_fitField != null) {
            _fitField.setData(value);
        }
    }
}
```

## Widget

Lightweight apps displayed in the widget loop/carousel.

### Manifest Type

```xml
<iq:application type="widget" ... >
```

### Widget Lifecycle

```mc
class MyWidgetApp extends Application.AppBase {
    function initialize() {
        AppBase.initialize();
    }

    ' REQUIRED for CIQ 4.0+ — glance shown in widget carousel
    function getGlanceView() as [GlanceView] or [GlanceView, GlanceViewDelegate] or Null {
        return [new MyGlanceView()];
    }

    ' Full widget view — launched when user taps glance
    function getInitialView() as [Views] or [Views, InputDelegates] {
        return [new MyWidgetView(), new MyWidgetDelegate()];
    }
}
```

### Glance View (CIQ 3.1.3+)

```mc
class MyGlanceView extends WatchUi.GlanceView {
    function initialize() {
        GlanceView.initialize();
    }

    function onUpdate(dc as Dc) as Void {
        dc.setColor(Graphics.COLOR_WHITE, Graphics.COLOR_BLACK);
        dc.clear();

        ' Glance constraints:
        ' - 64 pixels tall, full device width
        ' - Text and basic shapes only
        ' - No images/bitmaps
        ' - No touch input (tap launches full widget)
        ' - Keep simple and fast-rendering

        dc.drawText(5, dc.getHeight() / 2, Graphics.FONT_GLANCE,
            "Status: OK",
            Graphics.TEXT_JUSTIFY_LEFT | Graphics.TEXT_JUSTIFY_VCENTER);
    }
}
```

### Widget Behavior

- **CIQ 3 devices**: Widget runs in either full-screen "widget" mode or "glance" mode
- **CIQ 4+ devices**: Always shows glance first; tap launches full widget
- **getGlanceView() is REQUIRED** for CIQ 4+ — without it, only launcher icon and name appear
- Full widget view has full input handling
- Widget should be fast to load and display

## Device App

Full-featured application with complete UI and input.

### Manifest Type

```xml
<iq:application type="watch-app" ... >
```

### Device App Lifecycle

```mc
class MyDeviceApp extends Application.AppBase {
    function initialize() {
        AppBase.initialize();
    }

    function onStart(state as Dictionary?) as Void {
        ' App is starting
        ' state may contain data from a previous run
    }

    function getInitialView() as [Views] or [Views, InputDelegates] {
        return [new MainView(), new MainDelegate()];
    }

    function getGlanceView() as [GlanceView] or Null {
        ' Optional but recommended for CIQ 4+
        return [new AppGlanceView()];
    }

    function onStop(state as Dictionary?) as Void {
        ' App is stopping — save state
    }

    function onSettingsChanged() as Void {
        ' Settings changed in Garmin Connect Mobile
        WatchUi.requestUpdate();
    }
}
```

### Multi-View Navigation

```mc
' Push a new view onto the stack (back button returns to previous)
WatchUi.pushView(new DetailView(), new DetailDelegate(), WatchUi.SLIDE_LEFT);

' Pop current view (go back)
WatchUi.popView(WatchUi.SLIDE_RIGHT);

' Replace current view (no back navigation to previous)
WatchUi.switchToView(new OtherView(), new OtherDelegate(), WatchUi.SLIDE_LEFT);

' Transition animations
WatchUi.SLIDE_LEFT;      ' New view slides in from right
WatchUi.SLIDE_RIGHT;     ' New view slides in from left
WatchUi.SLIDE_UP;        ' New view slides in from bottom
WatchUi.SLIDE_DOWN;      ' New view slides in from top
WatchUi.SLIDE_BLINK;     ' Quick flash transition
WatchUi.SLIDE_IMMEDIATE; ' No animation
```

### Menu Navigation Pattern

```mc
class MainDelegate extends WatchUi.BehaviorDelegate {
    function initialize() {
        BehaviorDelegate.initialize();
    }

    function onMenu() as Boolean {
        ' Show main menu
        WatchUi.pushView(
            new Rez.Menus.MainMenu(),
            new MainMenuDelegate(),
            WatchUi.SLIDE_UP
        );
        return true;
    }

    function onSelect() as Boolean {
        ' Navigate to detail view
        WatchUi.pushView(
            new DetailView(),
            new DetailDelegate(),
            WatchUi.SLIDE_LEFT
        );
        return true;
    }

    function onBack() as Boolean {
        ' Exit app
        return false;  ' Let system handle (exits app)
    }
}
```

## Audio Content Provider

For music/podcast/audio streaming apps.

### Manifest Type

```xml
<iq:application type="audio-content-provider-app" ... >
```

### Key Classes

```mc
using Toybox.Media;

' Manage playlists and audio content
' Sync audio files from phone to device
' Handle playback controls

class MyAudioProvider extends Application.AppBase {
    function initialize() {
        AppBase.initialize();
    }

    function getInitialView() as [Views] or [Views, InputDelegates] {
        return [new AudioBrowseView(), new AudioBrowseDelegate()];
    }

    ' Called to sync content
    function getPlaybackConfiguration() as Media.PlaybackConfiguration {
        return new Media.PlaybackConfiguration({
            :playbackNotificationThreshold => 25  ' Notify at 25% remaining
        });
    }
}
```

## View Lifecycle Summary

All view types share this lifecycle:

```
initialize()    → Constructor, allocate resources
    ↓
onLayout(dc)    → Called once, set up layout from XML or code
    ↓
onShow()        → View becomes visible
    ↓
onUpdate(dc)    → Draw frame (called on requestUpdate() or system timer)
    ↓               ↑ (loops on each requestUpdate)
onHide()        → View is hidden (another view pushed, or app exits)
```

### When onUpdate is Called

| Scenario | Trigger |
|---|---|
| Watch face | Every minute (or every second with onPartialUpdate) |
| Data field | Every second during activity (via compute → onUpdate) |
| Widget/App | Only on `WatchUi.requestUpdate()` call |
| Any view | When view first becomes visible |
| Any view | When returning from a pushed view |

### View Stack

```
[MainView]              ← Initial state
[MainView, DetailView]  ← After pushView
[MainView]              ← After popView
[SettingsView]          ← After switchToView (MainView gone)
```

- `pushView()` — adds to stack (back returns to previous)
- `popView()` — removes top view (returns to previous)
- `switchToView()` — replaces top view (previous is gone)
