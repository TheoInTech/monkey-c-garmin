# Sensors, Communications, and Background Services

Complete reference for hardware sensors, networking, wireless protocols, and background execution in Connect IQ.

## Sensor API (Toybox.Sensor)

**Permission required**: `Sensor` in manifest.xml

### Enabling Sensors

```mc
using Toybox.Sensor;

' Enable specific sensors
Sensor.setEnabledSensors([
    Sensor.SENSOR_HEARTRATE,
    Sensor.SENSOR_TEMPERATURE,
    Sensor.SENSOR_ONBOARD_PULSE_OXIMETRY
]);

' Register for sensor events
Sensor.enableSensorEvents(method(:onSensor));

function onSensor(info as Sensor.Info) as Void {
    ' ALWAYS check for null — sensors may not have data
    if (info.heartRate != null) {
        var hr = info.heartRate as Number;     ' beats per minute
    }
    if (info.temperature != null) {
        var temp = info.temperature as Float;  ' degrees Celsius
    }
    if (info.altitude != null) {
        var alt = info.altitude as Float;      ' meters above sea level
    }
    if (info.pressure != null) {
        var pressure = info.pressure as Float; ' atmospheric pressure in Pascals
    }
    if (info.accel != null) {
        var accel = info.accel as Array;       ' [x, y, z] in milli-g
    }
    if (info.mag != null) {
        var mag = info.mag as Array;           ' [x, y, z] magnetometer
    }
    if (info.heading != null) {
        var heading = info.heading as Float;   ' compass heading in radians
    }
    if (info.cadence != null) {
        var cadence = info.cadence as Number;  ' steps or revolutions per minute
    }
    if (info.speed != null) {
        var speed = info.speed as Float;       ' meters per second
    }
    if (info.power != null) {
        var power = info.power as Number;      ' watts (cycling power meter)
    }
    if (info.oxygenSaturation != null) {
        var spo2 = info.oxygenSaturation as Number; ' SpO2 percentage
    }
}

' Disable all sensors when done
function cleanup() as Void {
    Sensor.setEnabledSensors([]);
    Sensor.enableSensorEvents(null);
}
```

### Sensor Constants

```mc
Sensor.SENSOR_HEARTRATE;
Sensor.SENSOR_TEMPERATURE;
Sensor.SENSOR_BIKECADENCE;
Sensor.SENSOR_BIKESPEED;
Sensor.SENSOR_BIKEPOWER;
Sensor.SENSOR_FOOTPOD;
Sensor.SENSOR_ONBOARD_PULSE_OXIMETRY;
```

### Important Notes

- **Watch faces cannot access sensors** — use `ActivityMonitor` instead
- Sensor data arrives asynchronously via callbacks
- Always disable sensors when not needed (battery drain)
- Not all devices support all sensors — check with `has` operator

## Sensor History (Toybox.SensorHistory)

**Permission required**: `SensorHistory`

Access recorded sensor data over time.

```mc
using Toybox.SensorHistory;

' Check availability before use
if (SensorHistory has :getHeartRateHistory) {
    var iter = SensorHistory.getHeartRateHistory({
        :period => 3600,    ' Last hour (seconds)
        :order => SensorHistory.ORDER_NEWEST_FIRST
    });

    var sample = iter.next();
    while (sample != null) {
        if (sample.data != null) {
            var hr = sample.data as Number;
            var when = sample.when;  ' Moment
        }
        sample = iter.next();
    }
}

' Available history methods (check with has):
SensorHistory.getHeartRateHistory({});
SensorHistory.getTemperatureHistory({});
SensorHistory.getPressureHistory({});
SensorHistory.getElevationHistory({});
SensorHistory.getOxygenSaturationHistory({});
SensorHistory.getStressHistory({});
SensorHistory.getBodyBatteryHistory({});

' Options
' :period => Number (seconds to look back)
' :order => ORDER_NEWEST_FIRST or ORDER_OLDEST_FIRST
```

## GPS / Positioning (Toybox.Position)

**Permission required**: `Positioning`

```mc
using Toybox.Position;

' Start continuous GPS tracking
Position.enableLocationEvents(Position.LOCATION_CONTINUOUS, method(:onPosition));

' Single location fix
Position.enableLocationEvents(Position.LOCATION_ONE_SHOT, method(:onPosition));

function onPosition(info as Position.Info) as Void {
    ' Check GPS quality
    if (info.accuracy == Position.QUALITY_NOT_AVAILABLE) {
        ' No GPS signal
        return;
    }

    if (info.accuracy == Position.QUALITY_LAST_KNOWN) {
        ' Cached position, may be stale
    }

    if (info.accuracy >= Position.QUALITY_USABLE) {
        ' Good enough to use
        var location = info.position.toDegrees();
        var lat = location[0] as Double;
        var lon = location[1] as Double;

        var altitude = info.altitude as Float;  ' meters
        var speed = info.speed as Float;        ' m/s
        var heading = info.heading as Float;    ' radians
    }
}

' Stop GPS
Position.enableLocationEvents(Position.LOCATION_DISABLE, null);

' Quality constants (ascending quality)
Position.QUALITY_NOT_AVAILABLE;  ' 0
Position.QUALITY_LAST_KNOWN;     ' 1
Position.QUALITY_POOR;           ' 2
Position.QUALITY_USABLE;         ' 3
Position.QUALITY_GOOD;           ' 4

' Location conversions
var degreesArray = location.toDegrees();     ' [lat, lon] in decimal degrees
var radiansArray = location.toRadians();     ' [lat, lon] in radians
```

## Activity Monitoring (Toybox.ActivityMonitor)

For daily step/activity tracking data. Works in watch faces.

```mc
using Toybox.ActivityMonitor;

var info = ActivityMonitor.getInfo();
info.steps;              ' Number — today's steps
info.stepGoal;           ' Number — daily step goal
info.calories;           ' Number — today's calories burned
info.distance;           ' Float — today's distance in centimeters
info.activeMinutesDay;   ' ActiveMinutes object
info.floorsClimbed;      ' Number
info.floorsDescended;    ' Number
info.floorsClimbedGoal;  ' Number
info.moveBarLevel;       ' Number — inactivity alert level (0-5)
info.isSleepMode;        ' Boolean

' Heart rate from activity monitor (works in watch faces)
var hrIter = ActivityMonitor.getHeartRateHistory(1, true);
var hrSample = hrIter.next();
if (hrSample != null && hrSample.heartRate != ActivityMonitor.INVALID_HR_SAMPLE) {
    var currentHr = hrSample.heartRate;
}
```

## HTTP Communications (Toybox.Communications)

**Permission required**: `Communications`

### GET Request

```mc
using Toybox.Communications;

function fetchData() as Void {
    var url = "https://api.example.com/data";
    var params = {
        "apiKey" => "your-key",
        "units" => "metric"
    };
    var options = {
        :method => Communications.HTTP_REQUEST_METHOD_GET,
        :headers => {
            "Accept" => "application/json"
        },
        :responseType => Communications.HTTP_RESPONSE_CONTENT_TYPE_JSON
    };

    Communications.makeWebRequest(url, params, options, method(:onReceive));
}

function onReceive(responseCode as Number, data as Dictionary or String or Null) as Void {
    if (responseCode == 200) {
        if (data instanceof Dictionary) {
            var value = data["key"];
            WatchUi.requestUpdate();
        }
    } else {
        handleError(responseCode);
    }
}
```

### POST Request

```mc
function postData() as Void {
    var url = "https://api.example.com/submit";
    var payload = {
        "name" => "Test",
        "value" => 42
    };
    var options = {
        :method => Communications.HTTP_REQUEST_METHOD_POST,
        :headers => {
            "Content-Type" => Communications.REQUEST_CONTENT_TYPE_JSON,
            "Authorization" => "Bearer " + token
        },
        :responseType => Communications.HTTP_RESPONSE_CONTENT_TYPE_JSON
    };

    Communications.makeWebRequest(url, payload, options, method(:onReceive));
}
```

### Image Download

```mc
function downloadImage() as Void {
    var url = "https://example.com/image.png";
    var options = {
        :maxWidth => 100,
        :maxHeight => 100,
        :dithering => Communications.IMAGE_DITHERING_FLOYD_STEINBERG
    };

    Communications.makeImageRequest(url, null, options, method(:onImage));
}

function onImage(responseCode as Number, data as WatchUi.BitmapResource or Null) as Void {
    if (responseCode == 200 && data != null) {
        ' data is a BitmapResource — draw with dc.drawBitmap()
        _bitmap = data;
        WatchUi.requestUpdate();
    }
}
```

### Common Response Codes

```mc
' HTTP standard codes
200  ' OK
400  ' Bad Request
401  ' Unauthorized
403  ' Forbidden
404  ' Not Found
500  ' Server Error

' Connect IQ specific codes
Communications.BLE_HOST_TIMEOUT;            ' -104 — Phone not connected
Communications.BLE_SERVER_TIMEOUT;          ' -103 — Server timeout
Communications.BLE_CONNECTION_UNAVAILABLE;  ' -101 — No connection
Communications.NETWORK_REQUEST_TIMED_OUT;   ' -102 — Request timeout
Communications.INVALID_HTTP_BODY_IN_NETWORK_RESPONSE; ' -400 — Bad response body
Communications.NETWORK_RESPONSE_TOO_LARGE;  ' -403 — Response exceeds size limit
Communications.NETWORK_RESPONSE_OUT_OF_MEMORY; ' -404 — Not enough memory
```

### Important Constraints

- **HTTPS required** for all external connections (HTTP only allowed for same-phone)
- Responses are limited in size (varies by device, typically 16KB-64KB)
- Only one web request can be active at a time
- Requests are routed through the connected phone
- Phone must be connected and have internet access

## Phone App Messaging

Direct communication between watch app and companion phone app.

```mc
' Send data to phone app
Communications.transmit("Hello from watch", null, new MyCommListener());

' Register to receive from phone
Communications.registerForPhoneAppMessages(method(:onPhoneMessage));

function onPhoneMessage(msg as Communications.PhoneAppMessage) as Void {
    var data = msg.data;
    ' Process received data
}

class MyCommListener extends Communications.ConnectionListener {
    function initialize() {
        ConnectionListener.initialize();
    }

    function onComplete() as Void {
        ' Message sent successfully
    }

    function onError() as Void {
        ' Message failed
    }
}
```

## Bluetooth Low Energy (Toybox.BluetoothLowEnergy)

**Permission required**: `BluetoothLowEnergy`

```mc
using Toybox.BluetoothLowEnergy as Ble;

' Define the BLE profile to connect to
class MyBleDelegate extends Ble.BleDelegate {
    function initialize() {
        BleDelegate.initialize();
    }

    function onScanResults(scanResults as Ble.Iterator) as Void {
        var result = scanResults.next();
        while (result != null) {
            ' result.getDeviceName()
            ' result.getRssi()
            ' result.getServiceUuids()

            ' Connect to matching device
            Ble.pairDevice(result);
            result = scanResults.next();
        }
    }

    function onConnectedStateChanged(device as Ble.Device, state as Ble.ConnectionState) as Void {
        if (state == Ble.CONNECTION_STATE_CONNECTED) {
            ' Read characteristics, subscribe to notifications
            var service = device.getService(serviceUuid);
            if (service != null) {
                var char = service.getCharacteristic(characteristicUuid);
                if (char != null) {
                    var cccd = char.getDescriptor(Ble.cccd());
                    cccd.requestWrite([0x01, 0x00]b);  ' Enable notifications
                }
            }
        }
    }

    function onCharacteristicChanged(char as Ble.Characteristic, value as Lang.ByteArray) as Void {
        ' Handle incoming data from BLE device
        var data = value;
    }

    function onCharacteristicRead(char as Ble.Characteristic, status as Ble.Status, value as Lang.ByteArray) as Void {
        if (status == Ble.STATUS_SUCCESS) {
            var data = value;
        }
    }
}

' Register profile and start scanning
Ble.registerProfile(profileDefinition);
Ble.setScanState(Ble.SCAN_STATE_SCANNING);
```

## ANT / ANT+ (Toybox.Ant, Toybox.AntPlus)

**Permission required**: `Ant`

### ANT+ Predefined Profiles

```mc
using Toybox.AntPlus;

' Heart Rate Monitor
var hrMonitor = new AntPlus.HeartRateMonitor(null);

' Set data listener
class MyHrListener extends AntPlus.HeartRateMonitor.HeartRateDataListener {
    function onHeartRateData(data as AntPlus.HeartRateMonitor.HeartRateData) as Void {
        var heartRate = data.computedHeartRate;
    }
}

' Bike Speed/Cadence
var bikeSensor = new AntPlus.BikeCadence(null);
var bikeSpeed = new AntPlus.BikeSpeed(null);
var bikePower = new AntPlus.BikePower(null);
```

### Raw ANT Channel

```mc
using Toybox.Ant;

' Open a raw ANT channel for custom protocols
var channel = new Ant.GenericChannel(
    method(:onMessage),
    new Ant.ChannelAssignment(Ant.CHANNEL_TYPE_RX_NOT_TX, Ant.NETWORK_PLUS)
);

channel.setDeviceConfig(new Ant.DeviceConfig({
    :deviceNumber => 0,
    :deviceType => 120,
    :transmissionType => 0,
    :messagePeriod => 8070,
    :radioFrequency => 57,
    :searchTimeoutLowPriority => 10
}));

channel.open();

function onMessage(msg as Ant.Message) as Void {
    var payload = msg.getPayload();
    ' Process ANT message
}
```

## Background Services (Toybox.Background)

**Permission required**: `Background`

### Temporal Events

```mc
using Toybox.Background;
using Toybox.Time;

' Register for periodic execution
' MINIMUM interval: 5 minutes (300 seconds)
Background.registerForTemporalEvent(new Time.Duration(300));

' Register for a specific time
var targetTime = Time.now().add(new Time.Duration(600));  ' 10 min from now
Background.registerForTemporalEvent(targetTime);

' Cancel temporal event
Background.deleteTemporalEvent();

' Get last temporal event time
var lastEvent = Background.getLastTemporalEventTime();
```

### Service Delegate

```mc
(:background)
class MyServiceDelegate extends System.ServiceDelegate {
    function initialize() {
        ServiceDelegate.initialize();
    }

    ' Called when temporal event fires
    function onTemporalEvent() as Void {
        ' Maximum 30 seconds execution time
        ' Very limited API access:
        '   - NO UI (WatchUi)
        '   - NO sensors (Sensor)
        '   - YES Communications (HTTP requests)
        '   - YES Storage/Properties
        '   - YES Time, Math, Lang

        makeBackgroundRequest();
    }

    function makeBackgroundRequest() as Void {
        Communications.makeWebRequest(
            "https://api.example.com/check",
            null,
            {
                :method => Communications.HTTP_REQUEST_METHOD_GET,
                :responseType => Communications.HTTP_RESPONSE_CONTENT_TYPE_JSON
            },
            method(:onReceive)
        );
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

### Receiving Background Data in Foreground

```mc
class MyApp extends Application.AppBase {
    function onBackgroundData(data) as Void {
        ' Called when background service returns via Background.exit()
        if (data != null) {
            Storage.setValue("latestData", data);
            WatchUi.requestUpdate();
        }
    }

    function getServiceDelegate() as [ServiceDelegate] {
        return [new MyServiceDelegate()];
    }
}
```

### Other Background Events

```mc
' Register for phone app messages in background
Background.registerForPhoneAppMessageEvent();

' Register for step milestone events (every 1000 steps)
' Available on some devices

' Sleep/wake events
' Registered automatically, handled by onSleepEvent / onWakeEvent in ServiceDelegate
```

### Background Service Constraints

- **5-minute minimum** between temporal events
- **30-second maximum** execution time per invocation
- **Limited memory** allocation — keep data structures small
- **No UI access** — cannot show views or request updates directly
- **No sensor access** — cannot read heart rate, GPS, etc.
- **Can make HTTP requests** — primary use case is data sync
- **Must call `Background.exit()`** — or system terminates the service
- **(:background) annotation required** — on all classes used in background context
- System may skip temporal events if device is busy or low on battery

## Complications (Toybox.Complications)

**Permission required**: `Complications` — CIQ 4.0+

Allow your app to provide data that watch faces can display.

```mc
using Toybox.Complications;

' In AppBase.getInitialView() or onStart():
if (Complications has :registerComplicationProvider) {
    Complications.registerComplicationProvider(
        :myComplication,
        method(:onComplicationUpdate)
    );
}

function onComplicationUpdate(complicationId as Complications.Id) as Complications.Data {
    ' Return current data for this complication
    return new Complications.Data(
        Complications.COMPLICATION_TYPE_SHORT_TEXT,
        {
            :shortLabel => "HR",
            :value => currentHeartRate.toString(),
            :unit => "bpm"
        }
    );
}

' Complication types
Complications.COMPLICATION_TYPE_SHORT_TEXT;
Complications.COMPLICATION_TYPE_LONG_TEXT;
Complications.COMPLICATION_TYPE_RANGED_VALUE;
Complications.COMPLICATION_TYPE_ICON;
```
