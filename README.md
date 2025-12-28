[![Build Status](https://fyhn.semaphoreci.com/badges/xDrip-Pebble-E/branches/master.svg?key=3e1c06da-a9ac-4071-ba4a-7bf6e74aac95)](https://fyhn.semaphoreci.com/projects/xDrip-Pebble-E)

# xDrip-Pebble-E

My Pebble xDrip watchface:

* Simple layout with time of day, blood glucose, big glucose graph, and not much else
* Only tested on the 2025 Pebble 2 Duo, but should be easy to adapt to other Pebbles (PRs welcome!)
* For offline use, just CGM->phone->watch, no internet connectivity.
* Based on the "Pebble Classic Trend Watchface" bundled in xDrip (see below)

![Photo of Pebble 2 Duo with this watchface](https://github.com/user-attachments/assets/ff1a2a58-7295-430d-85ad-e902b1bafa0f)

## Installation

* Install xDrip
    * At least [2025.12.07](https://github.com/NightscoutFoundation/xDrip/releases/tag/2025.12.07) to make it run
    * Or [my fork](https://github.com/mortenfyhn/xDrip) for full compatibility (big graph, and graph tweaks)
* Enable Pebble integration in xDrip
* Select "Pebble Classic Trend Watchface", say no to the install prompt
* Enable:
    * Display Trend
    * Display Low Line
    * Display High Line
    * Display Slope Arrows
* [Build and install the watchface](https://developer.rebble.io/sdk/)
    * Turn on "Use LAN developer connection" in Pebble app settings
    * Turn on "Dev Connection" in the Pebble app settings for your watch
    * `pebble build`
    * `pebble install --phone IP`

## Emulator

Testing layout changes is much faster with the emulator:

```sh
pebble build
pebble install --emulator diorite
```

Seems platform `flint` (new Pebble 2 Duo) isn't supported by the emulator yet but `diorite` (older Pebble 2) is pretty much the same.

## Repo landscape

There are several xDrip Pebble watchface repos that build on each other:

| Repo | Description | Last Updated |
|------|-------------|--------------|
| [nightscout/cgm-pebble](https://github.com/nightscout/cgm-pebble) | Original watchface, fetches CGM data from Nightscout web servers | November 2015 |
| [ktind/cgm-pebble-offline](https://github.com/ktind/cgm-pebble-offline) | Offline version of nightscout/cgm-pebble | December 2014 |
| [jstevensog/xDrip-pebble](https://github.com/jstevensog/xDrip-pebble) | Offline xDrip watchface, based on nightscout/cgm-pebble v6.0 and ktind/cgm-pebble-offline | September 2015 |
| [jstevensog/xDrip-Pebble-E](https://github.com/jstevensog/xDrip-Pebble-E) | Experimental variant (hence "-E") with glucose graph support, sent as PNG from xDrip | October 2017 (branch `new`: December 2018) |
| [jamorham/xDrip-Pebble-E](https://github.com/jamorham/xDrip-Pebble-E) | Fork of jstevensog/xDrip-Pebble-E with improved hardware support (Pebble 2, Round) and stability fixes | November 2017 |
| [mortenfyhn/xDrip-Pebble-E](https://github.com/mortenfyhn/xDrip-Pebble-E) | This fork, modified for Pebble 2 Duo (flint platform) compatibility with Rebble firmware | - |

## xDrip's bundled watchfaces

| Watchface name                   | Description                      | Bundled binary (in xDrip)        | Source                    |
|----------------------------------|----------------------------------|----------------------------------|---------------------------|
| Standard Watchface (old)         | Basic glucose display, no graph  | `xdrip_pebble.bin`               | jstevensog/xDrip-pebble   |
| Color Trend Watchface            | Color display with graph         | `xdrip_pebble_classic_trend.bin` | jamorham/xDrip-Pebble-E   |
| Pebble Classic Trend Watchface   | B&W display with graph           | `xdrip_pebble_classic_trend.bin` | jamorham/xDrip-Pebble-E   |
| Pebble Trend Clay Version (test) | Graph display with Clay† support | `xdrip_pebble2.bin`              | jstevensog/xDrip-Pebble-E |

† [Clay](https://developer.rebble.io/blog/2016/06/24/introducing-clay/)

---

Please correct me if I've made any mistakes.
