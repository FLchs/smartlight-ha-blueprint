# Home Assistant smart light blueprint

![GitHub License](https://img.shields.io/github/license/FLchs/smartlight-ha-blueprint%20)

<p align="center">
    <img src="./.github/assets/logo.small.png" alt="blueprint logo">
</p>

Make your lights turn on, off, change their brightness and color temperature, automatically.

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fraw.githubusercontent.com%2FFLchs%2Fsmartlight-ha-blueprint%2Frefs%2Fheads%2Fmaster%2Fsmartlight-ha-blueprint.yaml)

## Why ?

There are many blueprints that already do the same thing, partly or with other (often unnecessary) features.

[Blackshome's blueprint]([https://community.home-assistant.io/t/sensor-light-motion-sensor-door-sensor-sun-elevation-lux-value-scenes-time-light-control-device-tracker-night-lights/481048) is awesome, but many features are unnecessary for many use cases, and if something goes wrong it is really hard to debug due to the many branches.
My blueprint is not as smart, it only does one thing but tries to do it as well as possible.

## So what do this blueprint does ?

It turns lights on or off and changes their brightness according to a motion and light sensor, and changes their color temperature according to the sun’s position. That’s all.

## What it doesn't do ?

It doesn’t handle any edge cases. If you need a reversed light sensor, a manual conditional switch, RGB party lights, or anything fancy: use another blueprint.

## How to use it ?

Add the blueprint to Home Assistant : [Using automation blueprints](https://www.home-assistant.io/docs/automation/using_blueprints/) (really read it if you never used a blueprint)

Then the automation configuration is (or at least should be) self-explanatory.

## How does it work ?

This blueprint tries to follow [KISS principle](https://en.wikipedia.org/wiki/KISS_principle), anyone with basic knowledge of Home Assistant automation should be able to understand and troubleshoot it.

### Logic

Here is how values are computed :

#### Brightness

The brightness of the lights is calculated using the current lux from the sensor.
It stays at maximum brightness when the lux is below the "low" threshold, gradually decreases as lux rises between the "low" and "high" values, and turns off once it reaches the minimum brightness at the "high" threshold.

```
max_b - (lux - low) * (max_b - min_b) / (high - low))
```

It is a simple decreasing [linear interpolation](https://en.wikipedia.org/wiki/Linear_interpolation).

#### Color temperature

- Warm at low sun elevation (sunrise/sunset)
- Cool at high elevation (noon)
- Smooth nonlinear curve

```
min_ct + (max_ct - min_ct) * (clamped / 90) ** 0.8
```

The equation scales the color temperature between min_ct and max_ct based on the sun’s elevation, using a nonlinear exponent to make it warmer at low angles and cooler at high angles.

### Flowchart

![Flow chart describing the blueprint process](/flowchart.drawio.png)
