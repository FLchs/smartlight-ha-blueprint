# Home Assistant smart light blueprint

![GitHub License](https://img.shields.io/github/license/FLchs/smartlight-ha-blueprint%20) ![GitHub Actions Workflow Status](https://img.shields.io/github/actions/workflow/status/FLchs/smartlight-ha-blueprint/ci.yaml)

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

- Brighter when ambient light is low
- Dimmer when ambient light is high
- Adjusts brightness smoothly between defined limits

```
max_b - (lux - low) * (max_b - min_b) / (high - low))
```

This formula linearly decreases brightness from *max_b* to *min_b* as the measured light (*lux*) increases between the *low* and *high* thresholds.
If the *light level* is below *low*, brightness is set at *max_b*;
if it’s above *high*, it’s clamped to *min_b*.

ex:

- lux = 45
- low = 15
- high = 200
- min_b = 20%
- max_b = 100%

```bash
100−(45−15)×(100−20)/(200−15)
=100−30×80/185
=100−12.97 
= 87%
```

→ The lights are set to 87% brightness, since the room is moderately bright.

#### Color temperature

- Warm at low sun elevation (sunrise/sunset)
- Cool at high elevation (noon)
- Smooth nonlinear curve

```bash
home-assistant_ct + (max_ct - min_ct) * (sun_elevation / 90) ** 0.8
```

This formula gradually increases the color temperature from min_ct to max_ct as the sun rises.
The exponent 0.8 softens the curve, making the transition slower near the horizon and faster as the sun climbs.

Example:

- min_ct = 2400 K
- max_ct = 4200 K
- sun_elevation = 11°

```bash
2400+(4200−2400)×(11/90)0.8=2740 K
2400+(4200−2400)×(11/90)
0.8
= 2740 K
```

→ The light is warm (≈ 2700 K) since the sun is low.

### Flowchart

![Flow chart describing the blueprint process](/flowchart.drawio.png)

## Roadmap

- [x] Release a working blueprint
- [ ] Improve logic for color temperature
- [ ] Decide if adding an overide sensor is fancy or not

## Contributing

Issues and pull-request welcome.
