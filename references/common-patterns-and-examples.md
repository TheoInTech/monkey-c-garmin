# Common Patterns and Examples

Complete working templates for Garmin Connect IQ app types and common development patterns.

## Minimal Watch Face

```mc
' source/MyWatchFaceApp.mc
using Toybox.Application;

class MyWatchFaceApp extends Application.AppBase {
    function initialize() {
        AppBase.initialize();
    }

    function getInitialView() as [Views] {
        return [new MyWatchFaceView()];
    }
}
```

```mc
' source/MyWatchFaceView.mc
using Toybox.WatchUi;
using Toybox.Graphics;
using Toybox.System;
using Toybox.Time;
using Toybox.Time.Gregorian;

class MyWatchFaceView extends WatchUi.WatchFace {

    function initialize() {
        WatchFace.initialize();
    }

    function onLayout(dc as Dc) as Void {
        ' Optional: load a layout from XML
        ' setLayout(Rez.Layouts.WatchFace(dc));
    }

    function onUpdate(dc as Dc) as Void {
        ' Clear screen
        dc.setColor(Graphics.COLOR_WHITE, Graphics.COLOR_BLACK);
        dc.clear();

        var width = dc.getWidth();
        var height = dc.getHeight();
        var centerX = width / 2;
        var centerY = height / 2;

        ' Get current time
        var clockTime = System.getClockTime();
        var hours = clockTime.hour;
        var minutes = clockTime.min;
        var seconds = clockTime.sec;

        ' Format time string (12-hour or 24-hour)
        var deviceSettings = System.getDeviceSettings();
        if (!deviceSettings.is24Hour) {
            if (hours > 12) {
                hours = hours - 12;
            } else if (hours == 0) {
                hours = 12;
            }
        }

        var timeStr = Lang.format("$1$:$2$", [hours.format("%02d"), minutes.format("%02d")]);

        ' Draw time
        dc.setColor(Graphics.COLOR_WHITE, Graphics.COLOR_TRANSPARENT);
        dc.drawText(centerX, centerY - 30, Graphics.FONT_NUMBER_HOT, timeStr,
            Graphics.TEXT_JUSTIFY_CENTER | Graphics.TEXT_JUSTIFY_VCENTER);

        ' Draw date
        var today = Gregorian.info(Time.now(), Time.FORMAT_MEDIUM);
        var dateStr = Lang.format("$1$ $2$ $3$", [today.day_of_week, today.month, today.day]);
        dc.drawText(centerX, centerY + 40, Graphics.FONT_SMALL, dateStr,
            Graphics.TEXT_JUSTIFY_CENTER | Graphics.TEXT_JUSTIFY_VCENTER);

        ' Draw battery
        var stats = System.getSystemStats();
        var batteryStr = Lang.format("$1$%", [stats.battery.format("%d")]);
        dc.drawText(centerX, height - 30, Graphics.FONT_XTINY, batteryStr,
            Graphics.TEXT_JUSTIFY_CENTER | Graphics.TEXT_JUSTIFY_VCENTER);
    }

    ' Called when watch enters low-power (sleep) mode
    function onEnterSleep() as Void {
        ' Reduce update frequency, hide seconds
        WatchUi.requestUpdate();
    }

    ' Called when watch exits low-power mode
    function onExitSleep() as Void {
        ' Resume full updates
        WatchUi.requestUpdate();
    }

    ' Optional: partial screen update for second hand on MIP displays
    ' Called every second even in low-power mode
    function onPartialUpdate(dc as Dc) as Void {
        ' Only update the seconds region to save power
        ' Set a clip region to limit drawing area
        dc.setClip(120, 120, 40, 20);
        var seconds = System.getClockTime().sec;
        dc.setColor(Graphics.COLOR_WHITE, Graphics.COLOR_BLACK);
        dc.drawText(140, 130, Graphics.FONT_TINY, seconds.format("%02d"),
            Graphics.TEXT_JUSTIFY_CENTER | Graphics.TEXT_JUSTIFY_VCENTER);
        dc.clearClip();
    }
}
```

## Minimal Data Field

```mc
' source/MyDataFieldApp.mc
using Toybox.Application;

class MyDataFieldApp extends Application.AppBase {
    function initialize() {
        AppBase.initialize();
    }

    function getInitialView() as [Views] {
        return [new MyDataFieldView()];
    }
}
```

```mc
' source/MyDataFieldView.mc
using Toybox.WatchUi;
using Toybox.Graphics;
using Toybox.Activity;

class MyDataFieldView extends WatchUi.DataField {

    hidden var _currentHeartRate as Number = 0;
    hidden var _avgHeartRate as Number = 0;

    function initialize() {
        DataField.initialize();
    }

    ' Called every second during an activity — compute your values here
    function compute(info as Activity.Info) as Void {
        if (info.currentHeartRate != null) {
            _currentHeartRate = info.currentHeartRate as Number;
        }
        if (info.averageHeartRate != null) {
            _avgHeartRate = info.averageHeartRate as Number;
        }
    }

    function onUpdate(dc as Dc) as Void {
        ' Background
        dc.setColor(Graphics.COLOR_WHITE, getBackgroundColor());
        dc.clear();

        ' Determine text color based on background
        var textColor = (getBackgroundColor() == Graphics.COLOR_BLACK) ?
            Graphics.COLOR_WHITE : Graphics.COLOR_BLACK;
        dc.setColor(textColor, Graphics.COLOR_TRANSPARENT);

        var centerX = dc.getWidth() / 2;
        var centerY = dc.getHeight() / 2;

        ' Current heart rate (large)
        dc.drawText(centerX, centerY - 15, Graphics.FONT_NUMBER_MEDIUM,
            _currentHeartRate.format("%d"),
            Graphics.TEXT_JUSTIFY_CENTER | Graphics.TEXT_JUSTIFY_VCENTER);

        ' Average heart rate (small, below)
        dc.drawText(centerX, centerY + 25, Graphics.FONT_TINY,
            Lang.format("Avg: $1$", [_avgHeartRate.format("%d")]),
            Graphics.TEXT_JUSTIFY_CENTER | Graphics.TEXT_JUSTIFY_VCENTER);

        ' Label
        dc.drawText(centerX, 10, Graphics.FONT_XTINY, "Heart Rate",
            Graphics.TEXT_JUSTIFY_CENTER);
    }
}
```

## Minimal Widget with Glance

```mc
' source/MyWidgetApp.mc
using Toybox.Application;
using Toybox.WatchUi;

class MyWidgetApp extends Application.AppBase {
    function initialize() {
        AppBase.initialize();
    }

    function getInitialView() as [Views] or [Views, InputDelegates] {
        return [new MyWidgetView(), new MyWidgetDelegate()];
    }

    ' REQUIRED for CIQ 4.0+ devices
    function getGlanceView() as [GlanceView] or Null {
        return [new MyGlanceView()];
    }
}
```

```mc
' source/MyGlanceView.mc
using Toybox.WatchUi;
using Toybox.Graphics;

class MyGlanceView extends WatchUi.GlanceView {
    function initialize() {
        GlanceView.initialize();
    }

    function onUpdate(dc as Dc) as Void {
        dc.setColor(Graphics.COLOR_WHITE, Graphics.COLOR_BLACK);
        dc.clear();

        ' Glance is 64px tall, full device width
        ' Keep it simple — text and basic shapes only
        dc.drawText(0, dc.getHeight() / 2, Graphics.FONT_GLANCE,
            "My Widget: Ready",
            Graphics.TEXT_JUSTIFY_LEFT | Graphics.TEXT_JUSTIFY_VCENTER);
    }
}
```

```mc
' source/MyWidgetView.mc
using Toybox.WatchUi;
using Toybox.Graphics;

class MyWidgetView extends WatchUi.View {
    function initialize() {
        View.initialize();
    }

    function onUpdate(dc as Dc) as Void {
        View.onUpdate(dc);
        dc.setColor(Graphics.COLOR_WHITE, Graphics.COLOR_TRANSPARENT);
        dc.drawText(dc.getWidth() / 2, dc.getHeight() / 2,
            Graphics.FONT_MEDIUM, "Widget Content",
            Graphics.TEXT_JUSTIFY_CENTER | Graphics.TEXT_JUSTIFY_VCENTER);
    }
}
```

```mc
' source/MyWidgetDelegate.mc
using Toybox.WatchUi;

class MyWidgetDelegate extends WatchUi.BehaviorDelegate {
    function initialize() {
        BehaviorDelegate.initialize();
    }

    function onSelect() as Boolean {
        ' Handle tap / SELECT button
        return true;
    }

    function onBack() as Boolean {
        ' Exit widget
        WatchUi.popView(WatchUi.SLIDE_RIGHT);
        return true;
    }
}
```

## Minimal Device App

```mc
' source/MyDeviceApp.mc
using Toybox.Application;
using Toybox.WatchUi;

class MyDeviceApp extends Application.AppBase {
    function initialize() {
        AppBase.initialize();
    }

    function getInitialView() as [Views] or [Views, InputDelegates] {
        return [new MyMainView(), new MyMainDelegate()];
    }

    function getGlanceView() as [GlanceView] or Null {
        return [new MyGlanceView()];
    }

    function onStop(state as Dictionary?) as Void {
        ' Cleanup on exit
    }
}
```

## HTTP Request Pattern

```mc
using Toybox.Communications;
using Toybox.System;

class ApiHelper {

    function makeRequest() as Void {
        var url = "https://api.example.com/data";
        var params = {
            "apiKey" => "your-key",
            "units" => "metric"
        };
        var options = {
            :method => Communications.HTTP_REQUEST_METHOD_GET,
            :headers => {
                "Content-Type" => Communications.REQUEST_CONTENT_TYPE_JSON
            },
            :responseType => Communications.HTTP_RESPONSE_CONTENT_TYPE_JSON
        };

        Communications.makeWebRequest(url, params, options, method(:onResponse));
    }

    function onResponse(responseCode as Number, data as Dictionary or String or Null) as Void {
        if (responseCode == 200) {
            ' Success — process data
            if (data instanceof Dictionary) {
                var value = data["key"];
                ' Update app state and refresh UI
                WatchUi.requestUpdate();
            }
        } else {
            ' Error handling
            System.println("HTTP Error: " + responseCode);
        }
    }

    ' POST request example
    function postData(payload as Dictionary) as Void {
        var url = "https://api.example.com/submit";
        var options = {
            :method => Communications.HTTP_REQUEST_METHOD_POST,
            :headers => {
                "Content-Type" => Communications.REQUEST_CONTENT_TYPE_JSON
            },
            :responseType => Communications.HTTP_RESPONSE_CONTENT_TYPE_JSON
        };

        Communications.makeWebRequest(url, payload, options, method(:onResponse));
    }
}
```

## Sensor Reading Pattern

```mc
using Toybox.Sensor;
using Toybox.SensorHistory;
using Toybox.System;

class SensorHelper {

    function enableHeartRate() as Void {
        ' Enable heart rate sensor
        Sensor.setEnabledSensors([Sensor.SENSOR_HEARTRATE]);
        Sensor.enableSensorEvents(method(:onSensor));
    }

    function onSensor(sensorInfo as Sensor.Info) as Void {
        ' Always check for null — sensors may not have data
        if (sensorInfo.heartRate != null) {
            var hr = sensorInfo.heartRate as Number;
            System.println("HR: " + hr);
        }
        if (sensorInfo.temperature != null) {
            var temp = sensorInfo.temperature as Float;
            System.println("Temp: " + temp);
        }
        if (sensorInfo.altitude != null) {
            var alt = sensorInfo.altitude as Float;
            System.println("Alt: " + alt);
        }
    }

    ' Read historical sensor data
    function getHeartRateHistory() as Void {
        if (SensorHistory has :getHeartRateHistory) {
            var iterator = SensorHistory.getHeartRateHistory({
                :period => 3600,   ' Last hour in seconds
                :order => SensorHistory.ORDER_NEWEST_FIRST
            });

            var sample = iterator.next();
            while (sample != null) {
                if (sample.data != null) {
                    System.println("HR: " + sample.data + " at " + sample.when.value());
                }
                sample = iterator.next();
            }
        }
    }

    function disableSensors() as Void {
        Sensor.setEnabledSensors([]);
        Sensor.enableSensorEvents(null);
    }
}
```

## Timer Pattern

```mc
using Toybox.Timer;
using Toybox.WatchUi;

class MyTimerView extends WatchUi.View {
    hidden var _timer as Timer.Timer?;
    hidden var _count as Number = 0;

    function initialize() {
        View.initialize();
    }

    function onShow() as Void {
        ' Start a repeating timer (1000ms = 1 second)
        _timer = new Timer.Timer();
        _timer.start(method(:onTimer), 1000, true);  ' true = repeat
    }

    function onTimer() as Void {
        _count++;
        WatchUi.requestUpdate();  ' Trigger redraw
    }

    function onUpdate(dc as Dc) as Void {
        View.onUpdate(dc);
        dc.setColor(Graphics.COLOR_WHITE, Graphics.COLOR_TRANSPARENT);
        dc.drawText(dc.getWidth() / 2, dc.getHeight() / 2,
            Graphics.FONT_NUMBER_MEDIUM, _count.format("%d"),
            Graphics.TEXT_JUSTIFY_CENTER | Graphics.TEXT_JUSTIFY_VCENTER);
    }

    function onHide() as Void {
        ' ALWAYS stop timer when view is hidden
        if (_timer != null) {
            _timer.stop();
            _timer = null;
        }
    }
}
```

## Menu and Sub-View Navigation

```xml
<!-- resources/menus/menu.xml -->
<menu2 id="MainMenu" title="@Strings.MenuTitle">
    <menu-item id="item1" label="@Strings.Option1" />
    <menu-item id="item2" label="@Strings.Option2" />
    <toggle-menu-item id="toggle1" label="@Strings.Toggle" />
</menu2>
```

```mc
' Open a menu
function onMenu() as Boolean {
    WatchUi.pushView(
        new Rez.Menus.MainMenu(),
        new MyMenuDelegate(),
        WatchUi.SLIDE_UP
    );
    return true;
}

class MyMenuDelegate extends WatchUi.Menu2InputDelegate {
    function initialize() {
        Menu2InputDelegate.initialize();
    }

    function onSelect(item as MenuItem) as Void {
        var id = item.getId();
        if (id == :item1) {
            ' Navigate to a sub-view
            WatchUi.pushView(new DetailView(), new DetailDelegate(), WatchUi.SLIDE_LEFT);
        } else if (id == :item2) {
            ' Do something else
        }
    }

    function onBack() as Void {
        WatchUi.popView(WatchUi.SLIDE_DOWN);
    }
}
```

### View Stack Navigation

```mc
' Push a new view onto the stack
WatchUi.pushView(new NextView(), new NextDelegate(), WatchUi.SLIDE_LEFT);

' Pop current view (go back)
WatchUi.popView(WatchUi.SLIDE_RIGHT);

' Replace current view (no back navigation)
WatchUi.switchToView(new OtherView(), new OtherDelegate(), WatchUi.SLIDE_LEFT);

' Transition types
' WatchUi.SLIDE_LEFT, SLIDE_RIGHT, SLIDE_UP, SLIDE_DOWN, SLIDE_BLINK, SLIDE_IMMEDIATE
```

## Settings / Properties Pattern

```xml
<!-- resources/properties.xml -->
<properties>
    <property id="updateInterval" type="number">5</property>
    <property id="showSeconds" type="boolean">true</property>
    <property id="units" type="string">metric</property>
    <property id="colorTheme" type="number">0</property>
</properties>
```

```xml
<!-- resources/settings/settings.xml -->
<settings>
    <setting propertyKey="@Properties.updateInterval" title="@Strings.UpdateInterval">
        <settingConfig type="numeric" min="1" max="60" />
    </setting>
    <setting propertyKey="@Properties.showSeconds" title="@Strings.ShowSeconds">
        <settingConfig type="boolean" />
    </setting>
    <setting propertyKey="@Properties.units" title="@Strings.Units">
        <settingConfig type="list">
            <listEntry value="metric">@Strings.Metric</listEntry>
            <listEntry value="imperial">@Strings.Imperial</listEntry>
        </settingConfig>
    </setting>
</settings>
```

```mc
' Reading properties in code
using Toybox.Application.Properties;

function loadSettings() as Void {
    var interval = Properties.getValue("updateInterval") as Number;
    var showSeconds = Properties.getValue("showSeconds") as Boolean;
    var units = Properties.getValue("units") as String;
}

' Called when settings change in Garmin Connect Mobile
function onSettingsChanged() as Void {
    loadSettings();
    WatchUi.requestUpdate();
}
```

## Background Service Pattern

```mc
' source/MyApp.mc
using Toybox.Application;
using Toybox.Background;
using Toybox.Time;

class MyApp extends Application.AppBase {
    function initialize() {
        AppBase.initialize();
    }

    function getInitialView() as [Views] or [Views, InputDelegates] {
        ' Register background temporal event (every 15 minutes)
        if (Background has :registerForTemporalEvent) {
            Background.registerForTemporalEvent(new Time.Duration(900));
        }
        return [new MyView(), new MyDelegate()];
    }

    function getServiceDelegate() as [ServiceDelegate] {
        return [new MyBackgroundService()];
    }

    ' Called when background service returns data
    function onBackgroundData(data) as Void {
        ' data is whatever was passed to Background.exit()
        if (data instanceof Dictionary) {
            ' Store for later use
            Application.Storage.setValue("bgData", data);
        }
        WatchUi.requestUpdate();
    }
}
```

```mc
' source/MyBackgroundService.mc — MUST be annotated (:background)
using Toybox.System;
using Toybox.Background;
using Toybox.Communications;

(:background)
class MyBackgroundService extends System.ServiceDelegate {
    function initialize() {
        ServiceDelegate.initialize();
    }

    function onTemporalEvent() as Void {
        ' Runs every 15 minutes (or whatever Duration was registered)
        ' Maximum 30 seconds execution time
        ' Limited API access — no UI, no sensors
        makeApiCall();
    }

    function makeApiCall() as Void {
        var url = "https://api.example.com/status";
        Communications.makeWebRequest(url, null, {
            :method => Communications.HTTP_REQUEST_METHOD_GET,
            :responseType => Communications.HTTP_RESPONSE_CONTENT_TYPE_JSON
        }, method(:onReceive));
    }

    function onReceive(responseCode as Number, data as Dictionary or Null) as Void {
        if (responseCode == 200 && data != null) {
            ' Return data to foreground app
            Background.exit(data);
        } else {
            Background.exit(null);
        }
    }
}
```

## Custom Drawable Pattern

```mc
using Toybox.WatchUi;
using Toybox.Graphics;

class ProgressBar extends WatchUi.Drawable {
    hidden var _percent as Float = 0.0;
    hidden var _barColor as Number = Graphics.COLOR_GREEN;

    function initialize(params as Dictionary) {
        Drawable.initialize(params);
        if (params.hasKey(:percent)) {
            _percent = params[:percent] as Float;
        }
        if (params.hasKey(:color)) {
            _barColor = params[:color] as Number;
        }
    }

    function draw(dc as Dc) as Void {
        var barWidth = (dc.getWidth() * _percent / 100.0).toNumber();
        var barHeight = 10;
        var y = locY;

        ' Background track
        dc.setColor(Graphics.COLOR_DK_GRAY, Graphics.COLOR_TRANSPARENT);
        dc.fillRectangle(locX, y, dc.getWidth(), barHeight);

        ' Filled portion
        dc.setColor(_barColor, Graphics.COLOR_TRANSPARENT);
        dc.fillRectangle(locX, y, barWidth, barHeight);
    }

    function setPercent(percent as Float) as Void {
        _percent = percent;
    }
}
```

## GPS / Position Pattern

```mc
using Toybox.Position;
using Toybox.System;

class LocationHelper {

    function startTracking() as Void {
        Position.enableLocationEvents(
            Position.LOCATION_CONTINUOUS,
            method(:onPosition)
        );
    }

    function onPosition(info as Position.Info) as Void {
        if (info.accuracy == Position.QUALITY_GOOD || info.accuracy == Position.QUALITY_USABLE) {
            var location = info.position.toDegrees();
            var lat = location[0];
            var lon = location[1];
            var altitude = info.altitude;
            var speed = info.speed;  ' m/s
            var heading = info.heading; ' radians

            System.println(Lang.format("Lat: $1$, Lon: $2$", [lat, lon]));
        }
    }

    function stopTracking() as Void {
        Position.enableLocationEvents(Position.LOCATION_DISABLE, null);
    }
}
```

## Confirmation Dialog Pattern

```mc
function showConfirmation() as Void {
    var dialog = new WatchUi.Confirmation("Are you sure?");
    WatchUi.pushView(dialog, new ConfirmDelegate(), WatchUi.SLIDE_IMMEDIATE);
}

class ConfirmDelegate extends WatchUi.ConfirmationDelegate {
    function initialize() {
        ConfirmationDelegate.initialize();
    }

    function onResponse(response as WatchUi.Confirm) as Boolean {
        if (response == WatchUi.CONFIRM_YES) {
            ' User confirmed
        } else {
            ' User declined
        }
        return true;
    }
}
```

## Activity Recording Pattern

```mc
using Toybox.ActivityRecording;
using Toybox.Activity;

class RecordingHelper {
    hidden var _session as ActivityRecording.Session?;

    function startRecording() as Void {
        if (ActivityRecording has :createSession) {
            _session = ActivityRecording.createSession({
                :name => "My Activity",
                :sport => Activity.SPORT_GENERIC,
                :subSport => Activity.SUB_SPORT_GENERIC
            });
            _session.start();
        }
    }

    function addLap() as Void {
        if (_session != null) {
            _session.addLap();
        }
    }

    function stopRecording() as Void {
        if (_session != null && _session.isRecording()) {
            _session.stop();
        }
    }

    function saveRecording() as Void {
        if (_session != null) {
            _session.save();
            _session = null;
        }
    }

    function discardRecording() as Void {
        if (_session != null) {
            _session.discard();
            _session = null;
        }
    }
}
```
