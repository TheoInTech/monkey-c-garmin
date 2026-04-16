# Toybox API Reference

All Toybox modules organized by functional area. Each entry includes the module purpose, key classes, and most-used methods.

API Docs: https://developer.garmin.com/connect-iq/api-docs/

## Core Modules

### Toybox.Lang

Foundation types and utilities. Automatically available without `using`.

**Key Types**: `Number`, `Float`, `Long`, `Double`, `Char`, `String`, `Boolean`, `Array`, `Dictionary`, `Symbol`, `Method`, `WeakReference`, `Exception`

```mc
' String formatting
Lang.format("$1$ bpm at $2$", [heartRate, timeStr]);

' Type conversion
var n = "42".toNumber();         ' String → Number
var f = "3.14".toFloat();        ' String → Float
var s = 42.toString();           ' Number → String

' Number formatting
42.format("%04d");               ' "0042"
3.14f.format("%.1f");            ' "3.1"

' Array/Dictionary creation
var arr = new Array<Number>[10];
var dict = new Dictionary<String, Number>;
```

### Toybox.Math

Mathematical functions.

```mc
using Toybox.Math;

Math.sqrt(16.0);     ' 4.0
Math.pow(2, 10);     ' 1024
Math.sin(angle);     ' Radians
Math.cos(angle);
Math.atan2(y, x);
Math.abs(-5);        ' 5
Math.ceil(3.2);      ' 4
Math.floor(3.8);     ' 3
Math.round(3.5);     ' 4
Math.rand();         ' Random number
Math.srand(seed);    ' Seed random
Math.PI;             ' 3.14159...
Math.toRadians(180); ' Convert degrees to radians
Math.toDegrees(Math.PI); ' Convert radians to degrees
Math.ln(x);          ' Natural log
Math.log(x, base);   ' Log with base
```

### Toybox.System

System information and utilities.

```mc
using Toybox.System;

' Clock
var clock = System.getClockTime();
clock.hour;    ' 0-23
clock.min;     ' 0-59
clock.sec;     ' 0-59

' Device info
var settings = System.getDeviceSettings();
settings.screenWidth;
settings.screenHeight;
settings.screenShape;        ' SCREEN_SHAPE_ROUND, SCREEN_SHAPE_RECTANGLE, etc.
settings.is24Hour;
settings.phoneConnected;
settings.connectionAvailable;
settings.isTouchScreen;

' System stats
var stats = System.getSystemStats();
stats.battery;        ' Battery percentage (Float)
stats.charging;       ' Boolean
stats.usedMemory;     ' Bytes used
stats.totalMemory;    ' Total bytes available
stats.freeMemory;     ' Bytes free

' Debug output (simulator only)
System.println("Debug message");

' Exit to another app
System.exitTo(intent);
System.exit();  ' Exit current app
```

### Toybox.Time

Time and date manipulation.

```mc
using Toybox.Time;
using Toybox.Time.Gregorian;

' Current time
var now = Time.now();                    ' Moment object
var seconds = now.value();               ' Seconds since epoch

' Duration
var fiveMin = new Time.Duration(300);    ' 300 seconds
var later = now.add(fiveMin);

' Gregorian date info
var info = Gregorian.info(now, Time.FORMAT_MEDIUM);
info.year;        ' 2025
info.month;       ' "Jan", "Feb", etc. (FORMAT_MEDIUM) or 1-12 (FORMAT_SHORT)
info.day;         ' 1-31
info.day_of_week; ' "Mon", "Tue", etc. (FORMAT_MEDIUM) or 1-7 (FORMAT_SHORT)
info.hour;        ' 0-23
info.min;         ' 0-59
info.sec;         ' 0-59

' Create a specific moment
var moment = Gregorian.moment({
    :year => 2025,
    :month => 6,
    :day => 15,
    :hour => 12,
    :minute => 0,
    :second => 0
});

' Compare moments
if (moment1.greaterThan(moment2)) { }
if (moment1.lessThan(moment2)) { }
```

### Toybox.Timer

Periodic and one-shot timers.

```mc
using Toybox.Timer;

var timer = new Timer.Timer();

' Repeating timer (1000ms interval)
timer.start(method(:onTimer), 1000, true);

' One-shot timer
timer.start(method(:onTimer), 5000, false);

' Stop timer
timer.stop();

function onTimer() as Void {
    ' Timer callback
    WatchUi.requestUpdate();
}
```

### Toybox.StringUtil

String utilities for encoding/decoding.

```mc
using Toybox.StringUtil;

' UTF-8 encoding/decoding
var bytes = StringUtil.convertEncodedString("Hello", {
    :fromRepresentation => StringUtil.REPRESENTATION_STRING_PLAIN_TEXT,
    :toRepresentation => StringUtil.REPRESENTATION_BYTE_ARRAY
});

' Character array conversion
var charArray = StringUtil.utf8ArrayToString(byteArray);
```

### Toybox.Test

Unit testing framework.

```mc
using Toybox.Test;

(:test)
function testMyFunction(logger as Test.Logger) as Boolean {
    Test.assert(condition);
    Test.assertEqual(actual, expected);
    Test.assertNotEqual(a, b);
    Test.assertMessage(condition, "Failure message");
    logger.debug("Test info");
    return true;
}
```

## UI Modules

### Toybox.WatchUi

Core UI framework — views, input, navigation, drawables.

**Key Classes**: `View`, `WatchFace`, `DataField`, `GlanceView`, `Drawable`, `BehaviorDelegate`, `InputDelegate`, `Menu2`, `MenuItem`, `Confirmation`, `ProgressBar`, `Picker`, `TextPicker`

```mc
using Toybox.WatchUi;

' View management
WatchUi.pushView(view, delegate, transition);
WatchUi.popView(transition);
WatchUi.switchToView(view, delegate, transition);
WatchUi.requestUpdate();

' Transitions
WatchUi.SLIDE_LEFT;
WatchUi.SLIDE_RIGHT;
WatchUi.SLIDE_UP;
WatchUi.SLIDE_DOWN;
WatchUi.SLIDE_BLINK;
WatchUi.SLIDE_IMMEDIATE;

' Load resources
var bitmap = WatchUi.loadResource(Rez.Drawables.MyBitmap);
var string = WatchUi.loadResource(Rez.Strings.MyString);

' Confirmation dialog
var confirm = new WatchUi.Confirmation("Sure?");
WatchUi.pushView(confirm, new MyConfirmDelegate(), WatchUi.SLIDE_IMMEDIATE);

' Text picker
var picker = new WatchUi.TextPicker("Initial text");
WatchUi.pushView(picker, new MyPickerDelegate(), WatchUi.SLIDE_UP);
```

### Toybox.Graphics

Drawing context and graphics operations.

**Key Classes**: `Dc`, `BufferedBitmap`, `FontReference`, `BitmapReference`

```mc
using Toybox.Graphics;

' In onUpdate(dc):
dc.setColor(foreground, background);
dc.clear();

' Drawing
dc.drawText(x, y, font, text, justification);
dc.drawLine(x1, y1, x2, y2);
dc.drawRectangle(x, y, width, height);
dc.fillRectangle(x, y, width, height);
dc.drawRoundedRectangle(x, y, width, height, radius);
dc.fillRoundedRectangle(x, y, width, height, radius);
dc.drawCircle(x, y, radius);
dc.fillCircle(x, y, radius);
dc.drawArc(x, y, radius, direction, startAngle, endAngle);
dc.fillPolygon(pointsArray);
dc.drawBitmap(x, y, bitmap);
dc.drawScaledBitmap(x, y, width, height, bitmap);

' Text measurement
var dims = dc.getTextDimensions(text, font);  ' [width, height]
var w = dc.getTextWidthInPixels(text, font);

' Clipping
dc.setClip(x, y, width, height);
dc.clearClip();

' Pen width
dc.setPenWidth(2);

' Anti-aliasing (AMOLED only)
dc.setAntiAlias(true);

' Buffered bitmap (offscreen drawing)
var bmpRef = Graphics.createBufferedBitmap({
    :width => 100, :height => 100
});
var bmp = bmpRef.get();
var bmpDc = bmp.getDc();
' Draw to bmpDc, then dc.drawBitmap(x, y, bmp);

' Text justification
Graphics.TEXT_JUSTIFY_LEFT;
Graphics.TEXT_JUSTIFY_CENTER;
Graphics.TEXT_JUSTIFY_RIGHT;
Graphics.TEXT_JUSTIFY_VCENTER;
```

### Toybox.Attention

Alerts, vibrations, tones.

```mc
using Toybox.Attention;

' Vibration
if (Attention has :vibrate) {
    var vibeData = [
        new Attention.VibeProfile(50, 1000),  ' 50% intensity, 1000ms
        new Attention.VibeProfile(0, 500),    ' off, 500ms
        new Attention.VibeProfile(50, 1000)   ' 50% intensity, 1000ms
    ];
    Attention.vibrate(vibeData);
}

' Tone
if (Attention has :playTone) {
    Attention.playTone(Attention.TONE_START);
    ' TONE_START, TONE_STOP, TONE_LAP, TONE_KEY, TONE_ALERT_HI, TONE_ALERT_LO, etc.
}

' Backlight
if (Attention has :backlight) {
    Attention.backlight(true);
}
```

## Application Modules

### Toybox.Application

App lifecycle and entry points.

**Key Classes**: `AppBase`, `Properties`, `Storage`

```mc
using Toybox.Application;
using Toybox.Application.Storage;
using Toybox.Application.Properties;

' Storage (app data)
Storage.setValue("key", value);
var data = Storage.getValue("key");
Storage.deleteValue("key");

' Properties (user settings from properties.xml)
var setting = Properties.getValue("propertyKey");
Properties.setValue("propertyKey", newValue);

' Get app reference
var app = Application.getApp();
```

### Toybox.Background

Background service execution.

```mc
using Toybox.Background;

' Register for periodic execution (minimum 5 minutes)
Background.registerForTemporalEvent(new Time.Duration(300));

' Unregister
Background.deleteTemporalEvent();

' Exit background service and return data
Background.exit(data);

' Register for other events
Background.registerForPhoneAppMessageEvent();
Background.registerForTemporalEvent(moment);  ' Specific time
```

### Toybox.Authentication

OAuth authentication flow.

```mc
using Toybox.Authentication;

Authentication.makeOAuthRequest(
    "https://auth.provider.com/authorize",
    {
        "client_id" => "YOUR_CLIENT_ID",
        "response_type" => "code",
        "scope" => "read",
        "redirect_uri" => "https://localhost"
    },
    "https://auth.provider.com/token",
    Authentication.OAUTH_RESULT_TYPE_URL,
    {
        "client_id" => "YOUR_CLIENT_ID",
        "client_secret" => "YOUR_SECRET"
    },
    method(:onOAuthComplete)
);

function onOAuthComplete(code as Number, data as Dictionary) as Void {
    if (code == Authentication.OAUTH_SUCCESS) {
        var token = data["access_token"];
    }
}
```

## Sensor Modules

### Toybox.Sensor

Real-time sensor data.

```mc
using Toybox.Sensor;

' Enable sensors
Sensor.setEnabledSensors([Sensor.SENSOR_HEARTRATE, Sensor.SENSOR_TEMPERATURE]);
Sensor.enableSensorEvents(method(:onSensor));

function onSensor(info as Sensor.Info) as Void {
    info.heartRate;       ' Number? — beats per minute
    info.temperature;     ' Float? — degrees Celsius
    info.altitude;        ' Float? — meters
    info.pressure;        ' Float? — pascals
    info.accel;           ' Array? — [x, y, z] accelerometer data
    info.mag;             ' Array? — [x, y, z] magnetometer
    info.heading;         ' Float? — compass heading in radians
    info.cadence;         ' Number? — steps/revolutions per minute
    info.speed;           ' Float? — m/s
    info.power;           ' Number? — watts (cycling)
}

' Disable
Sensor.setEnabledSensors([]);
Sensor.enableSensorEvents(null);
```

### Toybox.SensorHistory

Historical sensor data.

```mc
using Toybox.SensorHistory;

if (SensorHistory has :getHeartRateHistory) {
    var iter = SensorHistory.getHeartRateHistory({
        :period => 3600,  ' seconds
        :order => SensorHistory.ORDER_NEWEST_FIRST
    });
    var sample = iter.next();
    while (sample != null) {
        ' sample.data — value (Number or Float)
        ' sample.when — Moment timestamp
        sample = iter.next();
    }
}

' Available history methods (check with has):
' getHeartRateHistory, getTemperatureHistory, getPressureHistory,
' getElevationHistory, getOxygenSaturationHistory, getStressHistory,
' getBodyBatteryHistory
```

### Toybox.Position

GPS positioning.

```mc
using Toybox.Position;

' Enable location tracking
Position.enableLocationEvents(Position.LOCATION_CONTINUOUS, method(:onPosition));

function onPosition(info as Position.Info) as Void {
    info.accuracy;     ' QUALITY_NOT_AVAILABLE, QUALITY_LAST_KNOWN, QUALITY_POOR, QUALITY_USABLE, QUALITY_GOOD
    info.position;     ' Location object
    info.altitude;     ' Float — meters
    info.speed;        ' Float — m/s
    info.heading;      ' Float — radians

    var coords = info.position.toDegrees();  ' [lat, lon]
    var lat = coords[0];
    var lon = coords[1];
}

' Disable
Position.enableLocationEvents(Position.LOCATION_DISABLE, null);

' Location modes
Position.LOCATION_CONTINUOUS;   ' Continuous updates
Position.LOCATION_ONE_SHOT;     ' Single fix
Position.LOCATION_DISABLE;      ' Turn off
```

### Toybox.Activity

Current activity data (during recording).

```mc
using Toybox.Activity;

var info = Activity.getActivityInfo();
if (info != null) {
    info.currentHeartRate;    ' Number?
    info.averageHeartRate;    ' Number?
    info.maxHeartRate;        ' Number?
    info.currentSpeed;        ' Float? — m/s
    info.averageSpeed;        ' Float?
    info.maxSpeed;            ' Float?
    info.currentCadence;      ' Number?
    info.elapsedDistance;      ' Float? — meters
    info.elapsedTime;         ' Number? — milliseconds
    info.currentPower;        ' Number? — watts
    info.calories;            ' Number?
    info.currentLocation;     ' Position.Location?
    info.totalAscent;         ' Float? — meters
    info.totalDescent;        ' Float?
    info.timerState;          ' TIMER_STATE_OFF, ON, PAUSED, STOPPED
}
```

### Toybox.ActivityMonitor

Daily activity tracking.

```mc
using Toybox.ActivityMonitor;

var info = ActivityMonitor.getInfo();
info.steps;           ' Number — today's steps
info.stepGoal;        ' Number — daily step goal
info.calories;        ' Number — today's calories
info.distance;        ' Float — today's distance in cm
info.activeMinutesDay; ' ActiveMinutes
info.floorsClimbed;   ' Number
info.floorsDescended; ' Number
info.floorsClimbedGoal; ' Number
info.moveBarLevel;    ' Number — inactivity level (0-5)
```

### Toybox.ActivityRecording

Record activities to FIT files.

```mc
using Toybox.ActivityRecording;

var session = ActivityRecording.createSession({
    :name => "My Activity",
    :sport => Activity.SPORT_RUNNING,
    :subSport => Activity.SUB_SPORT_GENERIC
});

session.start();       ' Begin recording
session.addLap();      ' Mark a lap
session.stop();        ' Pause recording
session.save();        ' Save to device
session.discard();     ' Discard recording
session.isRecording(); ' Boolean
```

### Toybox.FitContributor

Add custom data fields to FIT files.

```mc
using Toybox.FitContributor;

var field = session.createField("myField", 0, FitContributor.DATA_TYPE_FLOAT, {
    :mesgType => FitContributor.MESG_TYPE_RECORD,
    :units => "bpm"
});

field.setData(72.5);  ' Set value each compute cycle
```

## Communication Modules

### Toybox.Communications

HTTP requests and phone communication.

```mc
using Toybox.Communications;

' GET request
Communications.makeWebRequest(
    "https://api.example.com/data",
    { "param" => "value" },              ' Query parameters
    {
        :method => Communications.HTTP_REQUEST_METHOD_GET,
        :headers => { "Authorization" => "Bearer token" },
        :responseType => Communications.HTTP_RESPONSE_CONTENT_TYPE_JSON
    },
    method(:onResponse)
);

' POST request
Communications.makeWebRequest(
    "https://api.example.com/submit",
    { "key" => "value" },                ' POST body
    {
        :method => Communications.HTTP_REQUEST_METHOD_POST,
        :headers => {
            "Content-Type" => Communications.REQUEST_CONTENT_TYPE_JSON
        },
        :responseType => Communications.HTTP_RESPONSE_CONTENT_TYPE_JSON
    },
    method(:onResponse)
);

function onResponse(responseCode as Number, data as Dictionary or String or Null) as Void {
    if (responseCode == 200) {
        ' Success
    } else if (responseCode == Communications.BLE_HOST_TIMEOUT) {
        ' Phone not connected
    } else {
        ' Error
    }
}

' Image download
Communications.makeImageRequest(url, params, options, method(:onImageResponse));

function onImageResponse(responseCode as Number, data as WatchUi.BitmapResource or Null) as Void {
    if (responseCode == 200 && data != null) {
        ' data is a BitmapResource, draw with dc.drawBitmap()
    }
}

' Phone app messaging
Communications.transmit(data, null, new CommListener());
Communications.registerForPhoneAppMessages(method(:onPhone));
```

### Toybox.BluetoothLowEnergy

BLE peripheral communication.

```mc
using Toybox.BluetoothLowEnergy as Ble;

' Register BLE profiles
Ble.registerProfile({
    :uuid => Ble.longToUuid($180D00001000800000805F9B34FB),  ' Heart Rate Service
    :characteristics => [{
        :uuid => Ble.longToUuid($2A3700001000800000805F9B34FB),
        :descriptors => [Ble.cccd()]
    }]
});

' Scan for devices
var scanResult = Ble.setScanState(Ble.SCAN_STATE_SCANNING);

' BLE delegate handles events
class MyBleDelegate extends Ble.BleDelegate {
    function onScanResults(scanResults as Ble.Iterator) as Void { }
    function onConnectedStateChanged(device as Ble.Device, state as Ble.ConnectionState) as Void { }
    function onCharacteristicChanged(characteristic as Ble.Characteristic, value as ByteArray) as Void { }
}
```

### Toybox.Ant / Toybox.AntPlus

ANT and ANT+ wireless protocol for sport sensors.

```mc
using Toybox.AntPlus;

' Heart rate sensor via ANT+
var hrSensor = new AntPlus.HeartRateMonitor(null);

class MyAntListener extends AntPlus.HeartRateMonitor.HeartRateDataListener {
    function onHeartRateData(data as AntPlus.HeartRateMonitor.HeartRateData) as Void {
        var hr = data.computedHeartRate;
    }
}
```

## Data Modules

### Toybox.PersistedContent

Access to stored activities and routes.

```mc
using Toybox.PersistedContent;

var routes = PersistedContent.getRoutes();
if (routes != null) {
    var iter = routes.iterator();
    var route = iter.next();
    while (route != null) {
        var name = route.getName();
        route = iter.next();
    }
}
```

### Toybox.PersistedLocations

Saved locations on device.

```mc
using Toybox.PersistedLocations;

var locations = PersistedLocations.getLocations();
' Iterate and use saved location data
```

### Toybox.UserProfile

User profile data (age, weight, heart rate zones, etc.).

```mc
using Toybox.UserProfile;

var profile = UserProfile.getProfile();
profile.birthYear;
profile.gender;          ' GENDER_MALE, GENDER_FEMALE
profile.weight;          ' grams
profile.height;          ' cm
profile.restingHeartRate;
profile.activityClass;   ' 0-100

var hrZones = UserProfile.getHeartRateZones(UserProfile.HR_ZONE_SPORT_GENERIC);
' Returns array of zone boundaries
```

### Toybox.Weather

Weather data (if available on device).

```mc
using Toybox.Weather;

if (Weather has :getCurrentConditions) {
    var conditions = Weather.getCurrentConditions();
    if (conditions != null) {
        conditions.temperature;        ' Celsius
        conditions.feelsLikeTemperature;
        conditions.condition;          ' CONDITION_CLEAR, CONDITION_CLOUDY, etc.
        conditions.windSpeed;          ' m/s
        conditions.windBearing;        ' degrees
        conditions.relativeHumidity;   ' percentage
        conditions.precipitationChance;
        conditions.observationLocationName;
    }

    var forecast = Weather.getDailyForecast();
    var hourly = Weather.getHourlyForecast();
}
```

### Toybox.Complications

Provide data to watch faces (CIQ 4.0+).

```mc
using Toybox.Complications;

' Register as a complication provider
Complications.registerComplicationProvider(
    :myComplication,
    method(:onComplicationUpdate)
);

function onComplicationUpdate(complicationId as Complications.Id) as Complications.Data {
    return new Complications.Data(
        Complications.COMPLICATION_TYPE_SHORT_TEXT,
        { :shortLabel => "HR", :value => heartRate.toString() }
    );
}
```

## Security Module

### Toybox.Cryptography

Hashing and encryption.

```mc
using Toybox.Cryptography;

' SHA-256 hash
var hash = new Cryptography.Hash({ :algorithm => Cryptography.HASH_SHA256 });
hash.update("data to hash");
var digest = hash.digest();  ' ByteArray

' HMAC
var hmac = new Cryptography.HashBasedMessageAuthenticationCode({
    :algorithm => Cryptography.HASH_SHA256,
    :key => keyByteArray
});
hmac.update("data");
var mac = hmac.digest();
```

## Media Module

### Toybox.Media

Audio content and playback management.

```mc
using Toybox.Media;

' Used in Audio Content Provider apps
' Manage playlists, sync audio content, control playback
```

## Notifications Module

### Toybox.Notifications (SDK 8.1.0+)

Display notifications on device.

```mc
using Toybox.Notifications;

' Register for notification events
Notifications.registerForNotificationMessages(method(:onNotification));

' Show a notification
Notifications.showNotification(
    "Title",
    "Subtitle",
    {
        :icon => Rez.Drawables.NotifIcon,
        :data => {},
        :actions => [
            { :label => Rez.Strings.Reply, :data => REPLY_ACTION },
            { :label => Rez.Strings.Dismiss, :data => DISMISS_ACTION }
        ]
    }
);
```
