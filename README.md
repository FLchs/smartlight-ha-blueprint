# Home Assistant smart light blueprint

![GitHub License](https://img.shields.io/github/license/FLchs/smartlight-ha-blueprint%20) ![GitHub Actions Workflow Status](https://img.shields.io/github/check-runs/FLchs/smartlight-ha-blueprint/master) ![GitHub Release](https://img.shields.io/github/v/release/FLchs/smartlight-ha-blueprint?include_prereleases)

<p align="center">
    <img src="./.github/assets/logo.small.png" alt="blueprint logo">
</p>

Make your lights turn on, off, change their brightness and color temperature automatically.

<p align="center">
    <a href="https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fraw.githubusercontent.com%2FFLchs%2Fsmartlight-ha-blueprint%2Frefs%2Fheads%2Fmaster%2Fsmartlight-ha-blueprint.yaml">
        <img src="https://my.home-assistant.io/badges/blueprint_import.svg" alt="Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.">
</p>

## Why ?

There are many blueprints that already do the same thing, partly or with other (often unnecessary) features.

[Blackshome's blueprint]([https://community.home-assistant.io/t/sensor-light-motion-sensor-door-sensor-sun-elevation-lux-value-scenes-time-light-control-device-tracker-night-lights/481048) is awesome, but some features are unnecessary for the simplest use cases, and if something goes wrong it is hard to debug due to the many branches and edge cases.
My blueprint is definitely not as smart, it only does one thing but tries to do it as well as possible.

## So what does this blueprint do ?

It turns lights on or off and changes their brightness according to a motion and light sensor, and changes their color temperature according to the sun’s position. That’s all.

## What doesn't it do ?

It doesn’t handle any edge cases. If you need a reversed light sensor, a manual conditional switch, RGB party lights, or anything fancy: use another blueprint.
I recommend [Blackshome's blueprint]([https://community.home-assistant.io/t/sensor-light-motion-sensor-door-sensor-sun-elevation-lux-value-scenes-time-light-control-device-tracker-night-lights/481048), it is well tested and documented.

## How to use it ?

Add the blueprint to Home Assistant: [Using automation blueprints](https://www.home-assistant.io/docs/automation/using_blueprints/) (really read it if you never used a blueprint).

Then the automation configuration is (or at least should be) self-explanatory.

## How does it work ?

This blueprint tries to follow the [KISS principle](https://en.wikipedia.org/wiki/KISS_principle), anyone with basic knowledge of Home Assistant automation should be able to understand and troubleshoot it.

### Logic

Here is how values are computed:

#### Brightness

- Brighter when ambient light is low
- Dimmer when ambient light is high
- Adjusts brightness smoothly between defined limits

```
max_b - (lux - low) * (max_b - min_b) / (high - low)
```

This formula linearly decreases brightness from *max_b* to *min_b* as the measured light (*lux*) increases between the *low* and *high* thresholds.
If the *light level* is below *low*, brightness is set at *max_b*;
if it’s above *high*, it’s clamped to *min_b*.

Example:

- *lux* = __45__
- *low* = __15__
- *high* = __200__
- *min_b* = __20__
- *max_b* = __100%__

```bash
100−(45−15)×(100−20)/(200−15)
=100−30×80/185
=100−12.97 
=87%
```

→ The lights are set to 87% brightness since the room is quite dark (depend on sensors).

#### Color Temperature

- Warm when the sun is low (sunrise/sunset)
- Cooler as the sun rises higher
- Follows a smooth nonlinear curve based on solar elevation

min_ct + (max_ct - min_ct) * ( (elevation - e_min) / (90 - e_min) ) ** gamma

This equation maps the sun’s elevation to a color temperature between *min_ct* (warmest) and *max_ct* (coolest).  
The elevation is first clamped between *e_min* (civil twilight) and 90° (sun at zenith), then normalized to a 0–1 scale.  
The *gamma* value controls how quickly the color temperature transitions: lower gamma keeps the temperature warmer for longer; higher gamma shifts more quickly toward cooler tones.

Examples:

1. Sun is low in the sky:

- elevation = 12°  
- e_min = -6°  
- min_ct = 2000 K  
- max_ct = 4000 K  
- gamma = 0.8  

norm = (12 - (-6)) / (90 - (-6)) = 18 / 96 = 0.1875  
ct = 2000 + (4000 - 2000) *(0.1875 ** 0.8)  
= 2000 + 2000* 0.261  
≈ 2000 + 522  
≈ 2522 K  

→ The lights are set to ~2520 K, very warm because the sun is still low.

2. Sun is near zenith:

- elevation = 75°  
- e_min = -6°  
- min_ct = 2000 K  
- max_ct = 4000 K  
- gamma = 0.8  

norm = (75 - (-6)) / (90 - (-6)) = 81 / 96 = 0.84375  
ct = 2000 + (4000 - 2000) *(0.84375 ** 0.8)  
= 2000 + 2000* 0.873  
≈ 2000 + 1746  
≈ 3746 K  

→ The lights are set to ~3750 K, cooler as the sun is high.

### Flowchart

![Flow chart describing the blueprint process](/flowchart.drawio.png)

## Roadmap

- [x] Release a working blueprint
- [ ] Improve logic for color temperature
- [ ] Decide if adding an override sensor is fancy or not

## Contributing

Issues and pull requests welcome.
