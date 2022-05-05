---
title: Configuration
url: Configuration
layout: page
---

Logid uses a standard libconfig-style config file stored in `/etc/logid.cfg` by default (although you launch logid with the -c option to change it).

The config style may change in the future, however backwards-compatibility will be maintained.

For a sample config, see [logid.example.cfg](https://github.com/PixlOne/logiops/blob/master/logid.example.cfg).

`:` and `=` are used for defining variables and are interchangeable. (e.g. `name: "foo";` is the same as `name = "foo";`)

# Ignoring Devices
Devices can be ignored from being detected and used in logid. To ignore a device, create a field called `ignore` as an array of the PIDs of the devices you want to ignore.

e.g. `ignore: [0x00a7]`

# Defining Devices
Devices are defined in an array field called `devices`. This array consists of objects that define a device.

e.g. `devices: ( { device object }, { device object } ... );`

## Device Object
The following are a list of fields that a device object contains.
### name
This is a required string field that defines the name of the device. To get the name of the device, launch logid with the device connected and it should print out a message with the device name. (e.g. `name: "MX Master";`)

### buttons
This is an optional array field that defines the mappings for buttons.

e.g. `buttons: ( { button object }, { button object } ... );`

#### Button Objects
##### cid
This is a required integer field that defines the Control ID of the button that is being remapped. (e.g. `cid: 0xc4;`)
##### action
This is a required object field that defines the new action of the button. (e.g. `action: { ... };` )

Refer to the actions section for more information.

### dpi
This is an optional integer/array field that defines the DPI for a mouse that supports adjustable DPI. (e.g. `dpi: 1000;`)

If your mouse has multiple sensors, you may define separate DPIs for those sensors by using an array and placing the value in the sensor's index (e.g. sensor 0: 1000 dpi, sensor 1: 800 dpi -> `dpi: [1000, 800]`)

### smartshift
This is an optional object field that defines the SmartShift settings for a mouse that supports it.

```
smartshift:
{
    on: true;
    threshold: 30;
    default_threshold: 30;
};
```

#### on
This is an optional boolean field that defines whether SmartShift should be activated. (`true` for SmartShift, `false` for free-spin).

#### threshold
This is an optional integer field between 1-255 that defines the threshold required to change the SmartShift wheel to free-spin.

#### default_threshold
This is an optional integer field between 1-255 that defines the mouse's default threshold required to change the SmartShift wheel to free-spin.

### hiresscroll
This is an optional object field that defines the HiRes mouse-scrolling settings for a device that supports it.

```
hiresscroll:
{
    hires: true;
    invert: false;
    target: false;
};
```
#### hires
This is an optional boolean field that defines whether the mouse wheel should be high resolution or not.

#### invert
This is an optional boolean field that defines whether to invert the mouse wheel.

#### target
This is an optional boolean field that defines whether mousewheel events should send as an HID++ notification or work normally (`true` for HID++ notification, `false` for normal usage). This option must be set to true to remap the scroll wheel action.

#### up
This is a gesture that defines the action that will be taken when the scroll wheel is moved up (down if inverted). Refer to the Actions/Gestures section for further info. To function, this field requires target to be enabled.

### down
This is a gesture that defines the action that will be taken when the scroll wheel is moved down (up if inverted). Refer to the Actions/Gestures section for further info. To function, this field requires target to be enabled.

### thumbwheel
This is an optional object field that defines the thumb wheel settings on a device that supports it (e.g. MX Master 3).

Some devices will not support the proxy, touch, and tap fields. Thumb wheel capabilities can be determined by running logid in debug mode and looking at the output.

#### divert
This is a boolean field that determines whether the thumb wheel should be handled by logid (`true`) or the OS (`false`).

#### invert
This is a boolean field that determines whether the thumb wheel's scrolling should be diverted.

#### left
This is an optional gesture that defines the action that will be taken when the scroll wheel is moved left (right if inverted). Refer to the Actions/Gestures section for further info. To function, this field requires divert to be enabled.

#### right
This is an optional gesture that defines the action that will be taken when the scroll wheel is moved right (left if inverted). Refer to the Actions/Gestures section for further info. To function, this field requires divert to be enabled.

#### proxy
This is an optional action that defines what will occur when the user is in proximity of the thumb wheel. (will be pressed upon entering proximity, released upon leaving). Refer to the Actions section for further info.

#### touch
This is an optional action that defines what will occur when the user touches the thumb wheel. (will be pressed upon touching the thumb wheel, released upon letting go). Refer to the Actions section for further info.

#### tap
This is an optional action that defines what will occur when the user taps the thumb wheel once. (Pressed and immediately released after tap). Refer to the Actions section for further info.

# Actions
### type
This is a required string field that defines the type of action. (e.g. `type: "None";`). The following is a list of possible actions with their additional fields.

## None
This does nothing. There are no additional fields.

## Keypress
This maps press and release events of a button to a list of keys/buttons.
### keys
This is a required string/integer array/list field that defines the keys to be pressed/released. For a list of key/button strings, refer to [linux/input-event-codes.h](https://github.com/torvalds/linux/blob/master/include/uapi/linux/input-event-codes.h). (e.g. `keys: ["KEY_A", "KEY_B"];`). Alternatively, you may use integer keycodes to define the keys.

Please note that these event codes work in a US (QWERTY) keyboard layout. If you have a locale set that does not use this keyboard layout, please map it to whatever key it would be on a QWERTY keyboard. (e.g. "KEY_Z" on a QWERTZ layout should be "KEY_Y")

## Gestures
This action disables mouse movement while the button is pressed and allows you to assign actions for each direction. The possible directions are `Up`, `Down`, `Left`, `Right`, and `None`.

Note that any button can be used as a gesture button if the device supports gestures.

Additionally, some scroll wheels can have their up/down or left/right scrolls mapped to some gestures. Gesture modes will state their scroll wheel compatibility in the documentation.

### gestures
This is a required array of gesture objects that map a direction to a gesture mode and an action.

e.g. `gestures: ( { gesture object }, { gesture object } ... );`

#### direction
This is a required string field in a gesture object that defines the direction of the gesture. (e.g. `direction: "Up"`)

#### threshold
This is an optional integer field that determines the number of pixels at which a gesture should activate. (e.g. `threshold: 50`). This value must be greater than 0. If not set or set to an invalid value, the threshold will default to 50 pixels.

#### mode
This is an optional string field that defines the mode of the gesture. This field defaults to `OnRelease` if it is omitted.

The following is a list of gesture modes:

##### NoPress
This mode does nothing. The `action` field is ignored when this mode is used. This gesture is compatible with scroll wheels.
##### OnRelease
This mode presses and releases an action when the gesture button is released. This gesture is not compatible with scroll wheels.
##### OnInterval
This mode presses and releases an action after the mouse is moved every n pixels (where n is the integer field `interval`). This gesture is compatible with scroll wheels.
##### OnThreshold
Activates once the `threshold` is met, and never again.
##### Axis
This mode maps a gesture movement to an axis. The axis is defined as a string (e.g. "REL_WHEEL") in the `axis` field and the multiplier for its movement is defined in the `axis_multiplier` field. For a list of axis strings, refer to [linux/input-event-codes.h](https://github.com/torvalds/linux/blob/master/include/uapi/linux/input-event-codes.h). This gesture is compatible with scroll wheels.

#### action
This is a mandatory field that defines the action the gesture uses. This can be any action other than `Gestures`. Refer to the entire Actions section for more details.

## ToggleSmartShift
This action toggles the SmartShift scrolling feature when pressed. There are no additional fields.

## ToggleHiresScroll
This action toggles high resolution scrolling when pressed. There are no additional fields.

## CycleDPI
This action cycles between the given DPIs.

### dpis
This is a mandatory integer array field that defines what DPIs to cycle through. (e.g. `dpis: [800, 1000, 1200];`)

### sensor
This is an optional field that will determine what sensor the DPI will switch on. This value defaults to 0. In almost all cases, this does not need to be used.

## ChangeDPI
This action increments the DPI by the value given in the `inc`.

### inc
This is an integer array field that defines what to increase the DPI by.

### sensor
See the CycleDPI `sensor` field.

## ChangeHost
This action changes the host to a specified host if the device supports changing hosts.

### host
This field may either be an integer representing the host number (one-indexed, as shown on the bottom of the mouse), "next" to go to the next host number, or "prev"/"previous" to go to the previous host number. (e.g. `host = 2;` or `host = "next";`).

# Miscellaneous

### workers
This is an optional field that determines the number of worker threads that logid's workqueue uses. (e.g. `workers = 4;`)

This defaults to 4 if no value is given.

**Warning:** Decreasing the worker thread count is likely to result in reduced performance and counter-intuitively, more threads being created (since the workqueue will run a task in a new thread if it is busy for more than 500ms).

### io_timeout
This is an optional field that determines how how many milliseconds device I/O will wait for before timing out. (e.g. `timeout = 2000;`)

This defaults to 2000 ms if left blank.
