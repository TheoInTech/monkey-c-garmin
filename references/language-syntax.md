# Monkey C Language Syntax Deep Dive

Complete reference for the Monkey C programming language syntax, operators, control flow, and data structures.

## Operators

### Arithmetic

| Operator | Description | Example |
|---|---|---|
| `+` | Addition / String concatenation | `3 + 4`, `"a" + "b"` |
| `-` | Subtraction | `10 - 3` |
| `*` | Multiplication | `5 * 6` |
| `/` | Division | `10 / 3` (integer: 3) |
| `%` | Modulo | `10 % 3` (result: 1) |

### Comparison

| Operator | Description |
|---|---|
| `==` | Equal |
| `!=` | Not equal |
| `<` | Less than |
| `>` | Greater than |
| `<=` | Less than or equal |
| `>=` | Greater than or equal |

### Logical

| Operator | Description | Notes |
|---|---|---|
| `&&` | Logical AND | Short-circuit: if left is false, right not evaluated |
| `\|\|` | Logical OR | Short-circuit: if left is true, right not evaluated |
| `!` | Logical NOT | `!true` → `false` |

### Bitwise

| Operator | Description | Example |
|---|---|---|
| `&` | Bitwise AND | `$FF & $0F` → `$0F` |
| `\|` | Bitwise OR | `$F0 \| $0F` → `$FF` |
| `~` | Bitwise NOT | `~$FF` |
| `<<` | Shift left | `1 << 4` → `16` |
| `>>` | Shift right | `16 >> 4` → `1` |

### Assignment

| Operator | Description |
|---|---|
| `=` | Assign |
| `+=` | Add and assign |
| `-=` | Subtract and assign |
| `*=` | Multiply and assign |
| `/=` | Divide and assign |
| `%=` | Modulo and assign |
| `&=` | Bitwise AND and assign |
| `\|=` | Bitwise OR and assign |
| `<<=` | Shift left and assign |
| `>>=` | Shift right and assign |

### Ternary

```mc
var result = (condition) ? valueIfTrue : valueIfFalse;
```

## Control Flow

### If / Else If / Else

```mc
if (temperature > 100) {
    status = "hot";
} else if (temperature > 50) {
    status = "warm";
} else {
    status = "cold";
}
```

### Switch / Case

```mc
switch (mode) {
    case MODE_RUNNING:
        handleRunning();
        break;
    case MODE_WALKING:
    case MODE_HIKING:
        ' Fall-through: both cases handled here
        handleWalking();
        break;
    default:
        handleDefault();
        break;
}
```

### For Loop

```mc
for (var i = 0; i < 10; i++) {
    ' i = 0, 1, 2, ..., 9
}

' Counting down
for (var i = 9; i >= 0; i--) {
    ' i = 9, 8, 7, ..., 0
}
```

### While Loop

```mc
var count = 0;
while (count < 10) {
    count++;
}
```

### Do-While Loop

```mc
var input;
do {
    input = getNextInput();
} while (input != null);
```

### For-Each-In Loop

```mc
' Iterate over Array
var items = [1, 2, 3, 4, 5];
for (var i = 0; i < items.size(); i++) {
    var item = items[i];
}

' Iterate over Dictionary keys
var dict = { "a" => 1, "b" => 2, "c" => 3 };
var keys = dict.keys();
for (var i = 0; i < keys.size(); i++) {
    var key = keys[i];
    var value = dict[key];
}
```

### Loop Control

```mc
for (var i = 0; i < 100; i++) {
    if (i == 50) {
        break;     ' Exit loop entirely
    }
    if (i % 2 == 0) {
        continue;  ' Skip to next iteration
    }
}
```

## Arrays

### Creation

```mc
var empty = [];
var numbers = [1, 2, 3, 4, 5];
var mixed = [1, "two", 3.0, true, null];
var typed = new Array<Number>[10];  ' Pre-allocated typed array
var nested = [[1, 2], [3, 4]];
```

### Common Methods

```mc
var arr = [1, 2, 3];

arr.add(4);                    ' Append: [1, 2, 3, 4]
arr.addAll([5, 6]);            ' Append all: [1, 2, 3, 4, 5, 6]
var size = arr.size();         ' Length: 6
var first = arr[0];            ' Access by index: 1
arr[0] = 10;                   ' Set by index
var idx = arr.indexOf(3);      ' Find index: 2 (or -1 if not found)
arr.remove(3);                 ' Remove first occurrence of value 3
arr.removeAll();               ' Clear array
var slice = arr.slice(1, 3);   ' Subarray from index 1 to 3 (exclusive)
var reversed = arr.reverse();  ' Reverse a copy

' IMPORTANT: Arrays are reference types
var a = [1, 2, 3];
var b = a;        ' b is a REFERENCE to a, not a copy
b[0] = 99;        ' a[0] is now also 99
var c = a.slice(0, a.size());  ' c is an independent copy
```

## Dictionaries

### Creation

```mc
var empty = {};
var simple = { "name" => "Garmin", "year" => 2025 };
var symbolKeys = { :name => "Garmin", :year => 2025 };
var typed = {} as Dictionary<String, Number>;
```

### Common Methods

```mc
var dict = { "a" => 1, "b" => 2, "c" => 3 };

dict.put("d", 4);             ' Add or update: {"a"=>1, "b"=>2, "c"=>3, "d"=>4}
dict["e"] = 5;                ' Also works for set
var val = dict.get("a");      ' Get value: 1 (or null if key not found)
var val2 = dict["a"];         ' Also works for get
dict.remove("a");             ' Remove key
var has = dict.hasKey("b");   ' Check existence: true
var keys = dict.keys();       ' Array of keys: ["b", "c", "d", "e"]
var vals = dict.values();     ' Array of values: [2, 3, 4, 5]
var size = dict.size();       ' Number of entries: 4
var isEmpty = dict.isEmpty(); ' Check if empty: false

' IMPORTANT: Dictionaries are reference types (same as arrays)
```

## Strings

Strings are **immutable** — all operations return new strings.

```mc
var str = "Hello World";

var len = str.length();                  ' 11
var sub = str.substring(0, 5);           ' "Hello"
var upper = str.toUpper();               ' "HELLO WORLD"
var lower = str.toLower();               ' "hello world"
var found = str.find("World");           ' 6 (index) or null
var chars = str.toCharArray();           ' Array of Char
var num = "42".toNumber();               ' 42 (Number)
var flt = "3.14".toFloat();              ' 3.14 (Float)
var trimmed = "  hello  ".toString();    ' String representation

' Concatenation
var full = "Hello" + " " + "World";

' Formatted strings (Lang.format)
var formatted = Lang.format("$1$ of $2$", [current, total]);
var padded = Lang.format("$1$:$2$", [hours.format("%02d"), mins.format("%02d")]);

' Number formatting
var n = 42;
n.format("%d");     ' "42"
n.format("%04d");   ' "0042"
n.format("%x");     ' hex: "2a"

var f = 3.14159;
f.format("%.2f");   ' "3.14"
f.format("%06.2f"); ' "003.14"
```

## Symbols

Symbols are lightweight identifiers prefixed with `:`. They are interned (only one instance per name) and used extensively for dictionary keys, method references, and resource IDs.

```mc
var sym = :mySymbol;

' Common uses
var dict = { :name => "Garmin", :version => 9 };
var callback = method(:onResponse);
var resource = Rez.Drawables.MyIcon;

' Duck-type checking
if (obj has :myMethod) {
    obj.myMethod();
}
```

## Method References

```mc
' Reference to an instance method
var callback = method(:onResponse);

' Pass as callback
Communications.makeWebRequest(url, params, options, method(:onResponse));

' Reference to a method for timer
timer.start(method(:onTimer), 1000, true);
```

## Enums

```mc
' Basic enum (values start at 0, auto-increment)
enum {
    STATE_IDLE,       ' 0
    STATE_RUNNING,    ' 1
    STATE_PAUSED,     ' 2
    STATE_STOPPED     ' 3
}

' Enum with explicit values
enum {
    COLOR_RED = $FF0000,
    COLOR_GREEN = $00FF00,
    COLOR_BLUE = $0000FF
}

' Enums must be declared at module or class level, NOT inside functions
```

## Classes

### Basic Class

```mc
class MyClass {
    hidden var _name as String;
    hidden var _count as Number = 0;

    function initialize(name as String) {
        _name = name;
    }

    function getName() as String {
        return _name;
    }

    function increment() as Void {
        _count++;
    }
}

var obj = new MyClass("example");
obj.increment();
```

### Inheritance

```mc
class Animal {
    hidden var _name as String;

    function initialize(name as String) {
        _name = name;
    }

    function speak() as String {
        return "";
    }
}

class Dog extends Animal {
    function initialize(name as String) {
        Animal.initialize(name);  ' MUST call parent
    }

    function speak() as String {
        return "Woof!";
    }
}
```

### Access Modifiers

```mc
class MyClass {
    var publicField;                ' Public (default)
    hidden var _hiddenField;       ' Hidden (like protected/private)
    static var sharedField = 0;    ' Shared across all instances

    function publicMethod() {}      ' Public (default)
    hidden function _hiddenMethod() {}  ' Hidden

    ' "hidden" is equivalent to protected — accessible in subclasses
    ' There is no true "private" in Monkey C
}
```

### Static Members

```mc
class Counter {
    hidden static var _count as Number = 0;

    static function getCount() as Number {
        return _count;
    }

    static function increment() as Void {
        _count++;
    }
}

Counter.increment();
var c = Counter.getCount();
```

## Interfaces

```mc
interface Drawable {
    function draw(dc as Dc) as Void;
}

interface Updatable {
    function update(elapsed as Number) as Void;
}

class Sprite implements Drawable, Updatable {
    function draw(dc as Dc) as Void {
        ' Implementation required
    }

    function update(elapsed as Number) as Void {
        ' Implementation required
    }
}
```

## Modules

```mc
module MathUtils {
    function clamp(value as Number, min as Number, max as Number) as Number {
        if (value < min) { return min; }
        if (value > max) { return max; }
        return value;
    }

    function lerp(a as Float, b as Float, t as Float) as Float {
        return a + (b - a) * t;
    }

    const PI = 3.14159265f;
}

' Usage
var clamped = MathUtils.clamp(150, 0, 100);
```

## Scope and Visibility

```mc
' Module scope — accessible with module prefix
module MyModule {
    var moduleVar = 0;          ' Accessible as MyModule.moduleVar

    class MyClass {
        var classVar = 0;       ' Instance variable

        function myMethod() {
            var localVar = 0;   ' Function-local, destroyed on return

            ' Access chain:
            ' localVar → classVar → moduleVar → imported module scope
        }
    }
}
```

## Exception Handling

```mc
try {
    var result = riskyOperation();
} catch (ex instanceof Lang.InvalidValueException) {
    ' Handle specific exception type
    System.println("Invalid value: " + ex.getErrorMessage());
} catch (ex instanceof Lang.Exception) {
    ' Handle any Lang.Exception
    System.println("Error: " + ex.getErrorMessage());
} finally {
    ' Always executes (cleanup)
    cleanup();
}

' Throwing exceptions
throw new Lang.InvalidValueException("Value out of range");

' Custom exception
class MyException extends Lang.Exception {
    function initialize(msg as String) {
        Exception.initialize();
        self.setErrorMessage(msg);
    }
}
```

**IMPORTANT**: Many Monkey C runtime errors are **fatal and uncatchable**:
- Symbol not found errors
- Stack overflow
- Out of memory
- Null pointer dereference (in some contexts)
- Type check failures in strict mode

Always use defensive coding: check for `null`, verify types with `instanceof` or `has`, and validate data before use.

## Annotations

```mc
' Test annotation — excluded from release builds
(:test)
function testAddition(logger as Logger) as Boolean {
    var result = add(2, 3);
    logger.debug("Result: " + result);
    return (result == 5);
}

' Debug-only code
(:debug)
function debugLog(msg as String) as Void {
    System.println(msg);
}

' Release-only code
(:release)
function getApiUrl() as String {
    return "https://api.production.com";
}

' Background-safe annotation (required for background service code)
(:background)
class MyService extends System.ServiceDelegate {
    ' ...
}

' Disable type checking for a method
(:typecheck(false))
function dynamicOperation(obj) {
    return obj.someMethod();
}

' Deprecated
(:deprecated)
function oldMethod() as Void {
    ' Use newMethod() instead
}
```

## Test Assertions

```mc
using Toybox.Test;

(:test)
function testExample(logger as Logger) as Boolean {
    Test.assert(1 == 1);
    Test.assertEqual(add(2, 3), 5);
    Test.assertNotEqual("a", "b");
    Test.assertMessage(value > 0, "Value must be positive");
    Test.assertEqualMessage(result, expected, "Unexpected result");
    return true;  ' Test passed
}
```

## Conditional Compilation

```mc
' Device-specific code using has operator
if (Toybox.WatchUi has :GlanceView) {
    ' CIQ 3.1.3+ only
}

if (Toybox.Background has :registerForTemporalEvent) {
    ' Background service support available
}

' Compile-time exclusion via annotations
(:exclForDevice("fenix5"))
function fenix5SpecificFunction() {
    ' Only included for non-fenix5 builds
}
```

## Weak References

```mc
using Toybox.Lang;

var strongRef = new MyObject();
var weakRef = strongRef.weak();

' Later: check if object still exists
var obj = weakRef.get();
if (obj != null) {
    ' Object still alive, use it
    obj.doSomething();
} else {
    ' Object was garbage collected
}
```
