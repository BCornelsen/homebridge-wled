# homebridge-better-http-rgb

#### Homebridge plugin to control a HTTP-based RGB device.

Supports RGB HTTP(S) devices on the HomeBridge Platform and provides a readable
callback for getting and setting the following characteristics to Homekit:

* Characteristic.On
* Characteristic.Brightness
* Characteristic.Hue
* Characteristic.Saturation


# Installation

1. Install homebridge using: `npm install -g homebridge`
2. Install homebridge-http using:
```
sudo npm install -g git+https://github.com/QuickSander/homebridge-better-http-rgb.git
```
3. Update your configuration file.  See below for examples.


# Configuration

## Structure

The following is an overview of the structure of your HTTP-RGB accessory.

All `powerOn`, `powerOff` and `status` can either be a `string` or an `object`.
If a `string` is provided it is filled in as the `url` and the `body` will be
blank. Most devices will be ok with the `string` option.

The purpose of `powerOff`/`powerOn` for an RGB light is not to physically power
or de-power the device (as then how would it respond to further commands?), but
to set the LED color to black (for `powerOff`), and restore the color (for
`powerOn`).  Your backend device should already be doing this.  This is just
a convenience function so that your HomeBridge knows this device can turn off
or on.

Additionally, both `brightness` and `color` share the same structure (with the
exception that the `color` structure allows for a `.brightness` variable), they
can either be a `string` or an `object`.  If it is a `string`, it is filled in
as the `status` and the other fields are left blank. When this is the case, you
can only read the settings, you may not change them.

This accessory supports push notification from the physical device via 
'homebridge-http-notification-server'. This allows the device to modify the
switch's status by pushing the new status instead of Homebridge pulling it.
This can be realized by supplying the `notificationID`.
To get more details about the push configuration have a look at this 
[README](https://github.com/Supereg/homebridge-http-notification-server).

`service` is one of `['Light', 'Switch']`.


    {
        "accessory": "HTTP-RGB",
        "name": string,
        "service": string,

        "http_method": string-optional,
        "username": string-optional,
        "password": string-optional,
        "sendImmediately": string-optional,

        "switch": {
            "status": string-or-object-optional,
            "notificationID": string-optional,
            "notificationPassword": string-optional,
            "powerOn": string-or-object,
            "powerOff": {
                url: string,
                body: string
            }
        },

        "lock": {
            "status": url-optional,
            "secure": string-or-object,
            "unsecure": {
                url: string,
                body: string
            }
        },

        "brightness": string-or-object,
        "color": {
            "status": url-status,
            "url": url-optional,
            "brightness": boolean,
            "http_method": string-optional
        }
    }


## Examples

### Full RGB Device

    "accessories": [
        {
            "accessory": "HTTP-RGB",
            "name": "RGB Led Strip",
            "service": "Light",

            "switch": {
                "status": "http://localhost/api/v1/status",
                "powerOn": "http://localhost/api/v1/on",
                "powerOff": "http://localhost/api/v1/off"
            },

            "brightness": {
                "status": "http://localhost/api/v1/brightness",
                "url": "http://localhost/api/v1/brightness/%s"
            },

            "color": {
                "status": "http://localhost/api/v1/set",
                "url": "http://localhost/api/v1/set/%s",
                "brightness": true
            }
        }
    ]

### Single Color Light that only turns "off" and "on"

    "accessories": [
        {
            "accessory": "HTTP-RGB",
            "name": "Single Color Light",
            "service": "Light",

            "switch": {
                "status": "http://localhost/api/v1/status",
                "powerOn": "http://localhost/api/v1/on",
                "powerOff": "http://localhost/api/v1/off"
            }
        }
    ]

### Single Color Light with Brightness

    "accessories": [
        {
            "accessory": "HTTP-RGB",
            "name": "Single Color Light",
            "service": "Light",

            "switch": {
                "status": "http://localhost/api/v1/status",
                "powerOn": "http://localhost/api/v1/on",
                "powerOff": "http://localhost/api/v1/off"
            },

            "brightness": {
                "status": "http://localhost/api/v1/brightness",
                "url": "http://localhost/api/v1/brightness/%s"
            }
        }
    ]

### RGB Light without Brightness

    "accessories": [
        {
            "accessory": "HTTP-RGB",
            "name": "Single Color Light",
            "service": "Light",

            "switch": {
                "status": "http://localhost/api/v1/status",
                "powerOn": "http://localhost/api/v1/on",
                "powerOff": "http://localhost/api/v1/off"
            },

            "color": {
                "status": "http://localhost/api/v1/set",
                "url": "http://localhost/api/v1/set/%s"
            }
        }
    ]

This normally will not occur, however, you may not want your application to
display a "brightness" slider to the user.  In this case, you will want to
remove the brightness component from the config.

### Regular expression on-body matching

    "accessories": [
        {
            "accessory": "HTTP-RGB",
            "name": "JSON body matching",
            "service": "Light",

            "switch": {
                "status": {
                    "url": "http://localhost/api/v1/status",
                    "bodyRegEx": "\"switch\":\s*\"on\""
                }
                "powerOn": "http://localhost/api/v1/on",
                "powerOff": "http://localhost/api/v1/off"
            }
            }
        }
    ]

# Interfacing
All `.status` urls expect a 200 HTTP status code.

`switch.status` Can be configured to parse the body to determine the switch
status. When the specified `switch.status.bodyRegEx` matches the body the
switch is considered to be in the on status. If this parameter is left out
`switch.status` expects `0` for Off, and `1` for On.

All other `.status` urls expect a body of a single
string with no HTML markup.

* `brightness.status` expects a number from 0 to 100.
* `color.status` expects a 6-digit hexidemial number.

# Why 'better'?

See original statement from [jnovack/homebridge-better-http-rgb](https://github.com/jnovack/homebridge-better-http-rgb#why-better).

## Uninstall

To uninstall homebridge-better-http-rgb, simply run:
```
sudo npm uninstall -g homebridge-better-http-rgb
```
