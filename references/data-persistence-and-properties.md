# Data Persistence and Properties

Complete reference for storing data and managing user settings in Connect IQ applications.

## Overview: Storage vs Properties

| Feature | Application.Storage | Application.Properties |
|---|---|---|
| Purpose | App-managed data | User-configurable settings |
| Defined in | Code only | properties.xml |
| Editable by user | No | Yes, via Garmin Connect Mobile |
| Key type | String | String (from property id) |
| Background access | Yes | Yes |
| Persistence | Across app restarts | Across app restarts |
| Size limit | Device-dependent (8KB-32KB typical) | Part of same storage pool |

**Rule of thumb**: Use **Storage** for app state data (cached API responses, last sync time, user progress). Use **Properties** for user settings (units, thresholds, color choices, API keys).

## Application.Storage

Key-value store for arbitrary app data. Available since CIQ 2.4.0.

```mc
using Toybox.Application.Storage;

' --- Write ---
Storage.setValue("lastSyncTime", Time.now().value());
Storage.setValue("cachedData", { "temp" => 72, "humidity" => 45 });
Storage.setValue("favorites", [1, 2, 3, 4, 5]);
Storage.setValue("isFirstRun", false);

' --- Read ---
var lastSync = Storage.getValue("lastSyncTime");
if (lastSync != null) {
    var syncMoment = new Time.Moment(lastSync as Number);
}

var data = Storage.getValue("cachedData");
if (data != null && data instanceof Dictionary) {
    var temp = data["temp"];
}

' --- Delete ---
Storage.deleteValue("cachedData");

' --- Clear all ---
Storage.clearValues();
```

### Supported Value Types

```mc
' Primitives
Storage.setValue("number", 42);              ' Number
Storage.setValue("float", 3.14f);            ' Float
Storage.setValue("long", 123456789l);        ' Long
Storage.setValue("double", 3.14159265d);     ' Double
Storage.setValue("boolean", true);           ' Boolean
Storage.setValue("string", "hello");         ' String
Storage.setValue("char", 'A');               ' Char

' Containers (nested supported)
Storage.setValue("array", [1, 2, 3]);        ' Array
Storage.setValue("dict", { "a" => 1 });      ' Dictionary
Storage.setValue("nested", {
    "items" => [1, 2, 3],
    "meta" => { "count" => 3 }
});

' NOT supported:
' - Custom objects (class instances)
' - BitmapResource, FontResource
' - Method references
' - Symbols (stored as their integer representation)
```

### Storage in Background Services

```mc
(:background)
class MyService extends System.ServiceDelegate {
    function onTemporalEvent() as Void {
        ' Can read/write Storage from background
        var lastData = Storage.getValue("apiCache");

        ' After fetching new data:
        Storage.setValue("apiCache", newData);
        Storage.setValue("lastFetchTime", Time.now().value());

        Background.exit(newData);
    }
}
```

### Storage Size Limits

- Total storage varies by device (typically 8KB-32KB)
- Shared between Storage and Properties
- Writing oversized data throws an exception
- Monitor usage and keep stored data minimal
- Clear stale data proactively

## Application.Properties

User-configurable settings defined in XML. Available since CIQ 2.4.0.

### Defining Properties

```xml
<!-- resources/properties.xml -->
<properties>
    <!-- Number property with default value -->
    <property id="updateInterval" type="number">5</property>

    <!-- Boolean property -->
    <property id="showSeconds" type="boolean">true</property>

    <!-- String property -->
    <property id="units" type="string">metric</property>

    <!-- Float property -->
    <property id="threshold" type="float">0.75</property>
</properties>
```

### Reading Properties

```mc
using Toybox.Application.Properties;

function loadSettings() as Void {
    ' IMPORTANT: Property must be defined in properties.xml
    ' Accessing undefined property throws Lang.InvalidKeyException

    var interval = Properties.getValue("updateInterval") as Number;
    var showSec = Properties.getValue("showSeconds") as Boolean;
    var units = Properties.getValue("units") as String;
    var threshold = Properties.getValue("threshold") as Float;
}
```

### Writing Properties (from code)

```mc
' Update a property value programmatically
Properties.setValue("updateInterval", 10);
Properties.setValue("showSeconds", false);

' This also updates the value shown in Garmin Connect Mobile settings
```

### Settings UI (Garmin Connect Mobile)

Define the settings interface shown in the companion phone app.

```xml
<!-- resources/settings/settings.xml -->
<settings>
    <!-- Numeric input with range -->
    <setting propertyKey="@Properties.updateInterval"
             title="@Strings.UpdateInterval">
        <settingConfig type="numeric" min="1" max="60" />
    </setting>

    <!-- Boolean toggle -->
    <setting propertyKey="@Properties.showSeconds"
             title="@Strings.ShowSeconds">
        <settingConfig type="boolean" />
    </setting>

    <!-- List selection (dropdown) -->
    <setting propertyKey="@Properties.units"
             title="@Strings.Units">
        <settingConfig type="list">
            <listEntry value="metric">@Strings.Metric</listEntry>
            <listEntry value="imperial">@Strings.Imperial</listEntry>
            <listEntry value="nautical">@Strings.Nautical</listEntry>
        </settingConfig>
    </setting>

    <!-- Text input -->
    <setting propertyKey="@Properties.apiKey"
             title="@Strings.ApiKey">
        <settingConfig type="alphaNumeric" />
    </setting>

    <!-- Password input (masked) -->
    <setting propertyKey="@Properties.password"
             title="@Strings.Password">
        <settingConfig type="password" />
    </setting>

    <!-- Date picker -->
    <setting propertyKey="@Properties.targetDate"
             title="@Strings.TargetDate">
        <settingConfig type="date" />
    </setting>
</settings>
```

### Setting Config Types

| Type | Description | Attributes |
|---|---|---|
| `numeric` | Number input | `min`, `max` |
| `boolean` | Toggle switch | — |
| `list` | Dropdown list | `<listEntry>` children |
| `alphaNumeric` | Text input | — |
| `password` | Masked text input | — |
| `date` | Date picker | — |
| `phone` | Phone number input | — |
| `email` | Email input | — |
| `url` | URL input | — |

### Responding to Settings Changes

```mc
class MyApp extends Application.AppBase {
    ' Called when user changes settings in Garmin Connect Mobile
    function onSettingsChanged() as Void {
        ' Reload all settings
        loadSettings();

        ' Refresh the display
        WatchUi.requestUpdate();
    }
}
```

### Settings with String Resources

```xml
<!-- resources/strings/strings.xml -->
<strings>
    <string id="UpdateInterval">Update Interval (minutes)</string>
    <string id="ShowSeconds">Show Seconds</string>
    <string id="Units">Distance Units</string>
    <string id="Metric">Metric (km)</string>
    <string id="Imperial">Imperial (mi)</string>
    <string id="Nautical">Nautical (nm)</string>
    <string id="ApiKey">API Key</string>
</strings>
```

## Legacy Object Store (Pre-CIQ 2.4.0)

**Deprecated** — use Application.Storage and Application.Properties instead.

```mc
' Legacy API (still works but not recommended)
var app = Application.getApp();
app.setProperty("key", value);
var data = app.getProperty("key");
```

### Migration from Legacy

```mc
' Old code:
var app = Application.getApp();
var value = app.getProperty("myKey");
app.setProperty("myKey", newValue);

' New code:
' For app data:
var value = Storage.getValue("myKey");
Storage.setValue("myKey", newValue);

' For user settings:
var value = Properties.getValue("myKey");
Properties.setValue("myKey", newValue);
```

## Best Practices

### Data Serialization

```mc
' Store complex data as Dictionary
var userData = {
    "name" => "John",
    "scores" => [95, 87, 92],
    "preferences" => {
        "theme" => "dark",
        "notifications" => true
    },
    "lastUpdated" => Time.now().value()
};
Storage.setValue("userData", userData);

' Read and validate
var stored = Storage.getValue("userData");
if (stored != null && stored instanceof Dictionary) {
    var name = stored["name"];
    if (name != null && name instanceof String) {
        ' Safe to use
    }
}
```

### Caching Pattern

```mc
function getData() as Dictionary? {
    ' Check cache first
    var cached = Storage.getValue("apiCache");
    var cacheTime = Storage.getValue("apiCacheTime");

    if (cached != null && cacheTime != null) {
        var age = Time.now().value() - (cacheTime as Number);
        if (age < 300) {  ' Cache valid for 5 minutes
            return cached as Dictionary;
        }
    }

    ' Cache expired or missing — fetch fresh data
    fetchFromApi();
    return null;
}

function onApiResponse(data as Dictionary) as Void {
    Storage.setValue("apiCache", data);
    Storage.setValue("apiCacheTime", Time.now().value());
}
```

### Defensive Property Reading

```mc
' Always handle potential errors when reading properties
function getSafeSetting(key as String, defaultValue) {
    try {
        var value = Properties.getValue(key);
        if (value != null) {
            return value;
        }
    } catch (ex instanceof Lang.Exception) {
        ' Property might not exist or be wrong type
    }
    return defaultValue;
}

' Usage
var interval = getSafeSetting("updateInterval", 5) as Number;
var units = getSafeSetting("units", "metric") as String;
```
