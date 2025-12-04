# <img src='.github/xremap.png' style='height: 32px; margin-top: 8px; margin-bottom: -4px;' alt='Xremap'> Experimental fork.

[![GitHub Actions](https://github.com/hpccc53/xremap/actions/workflows/build.yml/badge.svg)](https://github.com/hpccc53/xremap/actions/workflows/build.yml)

`xremap` is a key remapper for Linux.

This is a fork for experimenting with new features for xremap. Full compatibility is retained with official xremap.

This experimental fork is compatible with xremap in the following way:

| Version | Xremap version |
| ------- | -------------- |
| 50.0.1  | 0.14.5         |

## Changelog

Changes made on top of xremap:

### Remove repeat events from free_hold before decision [PR 14](https://github.com/hpccc53/xremap/pull/14)

There should be no purpose for emitting repeat events. Because the multipurpose key does not emit press before decision,
so it's logical inconsistent to emit repeat events. When trying it manually the repeat events are
also just ignored.

Note: I believe it's a regression introduced here: [feat: no timeout hold option ](https://github.com/xremap/xremap/pull/705/commits/643f4bf801013240526d174755f70f7811895439). Where the `held_down` check should also have been used in the repeat-function.

### Add all mouse buttons to output device [PR 6](https://github.com/hpccc53/xremap/pull/6)

Make it possible to emit all mouse buttons from a config file. Before it was only possible to click some of the mouse buttons.

### Throttle output events [PR 5](https://github.com/hpccc53/xremap/pull/5), [13](https://github.com/hpccc53/xremap/pull/13)

Delay (if needed) between:

- press and release of the same key. But not the other way around.
- press of ordinary key and press/release of modifier key.
- press/release of modifier key and press of ordinary key.

Config file:

```yml
throttle_ms: 10 # Defaults to 0
```

Notes

This is useful because some applications and desktops don't handle key events correctly when they are emitted fast. By adding these delays there is time to register combos.

There's a similar configuration `keypress_delay_ms`, but it's only added when emitting key combos from `keymap`, but there are other places, where it's useful.
