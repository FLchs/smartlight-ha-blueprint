# Home Assistant smart light blueprint

Make your lights turn on, change brightness and temperature automatically.

## Why ?

There is many blueprints that already do the same thing, partly or along other (uneeded) features.

[Blackshome's blueprint]([https://community.home-assistant.io/t/sensor-light-motion-sensor-door-sensor-sun-elevation-lux-value-scenes-time-light-control-device-tracker-night-lights/481048) is awesome, but many features are unnecessary for many use cases, and if something goes wrong it is really hard to debug due to the many branches.
My blueprint is not as smart, it does only one thing but try to do it as well as possible.

## So what do this blueprint do ?

It turns on or off lights and change their brightness according to a motion and light sensor. Change their color temperature according to the sun position. And that's all.

## What it doesn't ?

Any edge cases. If you need a reversed light sensor, a manual conditional switch, RGB party lights or anything fancy : use another blueprint.

## How to use it ?

Add the blueprint to Home Assistant : [Using automation blueprints](https://www.home-assistant.io/docs/automation/using_blueprints/) (really read it if you never used a blueprint)

Then the automation configuration is self explanatory.

## How does it work ?

This blueprint is [KISS](https://en.wikipedia.org/wiki/KISS_principle).

### Here is the flowchart that explains how it works

![Flow chart describing the blueprint process](/flowchart.drawio.png)
