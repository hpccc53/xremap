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

### Add actions to set lock key state [PR 8](https://github.com/hpccc53/xremap/pull/8)

It's usually only possible to toggle the `capslock`. These actions make et possible to
set the state of `capslock`, `numlock` and `scrolllock`.

```yml
keymap:
  - remap:
      control-j:
        - numlock: true
        - capslock: false
        - scrolllock: false
```

If the LED states are out of sync with what the desktop thinks the lock key state is, then these actions will not work.
But in that case it's possible to sync the LEDs. It takes some time for the desktop to register it and set the LED to the correct state, so the sleep is needed.

```yml
keymap:
  - remap:
      control-k:
        - capslock
        - sleep: 10
        - capslock: false
```

### Add tap preferred to multipurpose key (tap-hold key) [PR 15](https://github.com/hpccc53/xremap/pull/15)

The tap-hold key has only supported hold-preferred until now. With this change there
is also support for a tap-preferred version. And even a mixture based on how the
parameters `held_threshold_millis` and `tap_timeout_millis` are set.

The key difference between tap-preferred and hold-preferred is what happens when the tap-hold key
is interrupted by another key.

The tap-hold key starts out in the tap-preferred state, which means it will press and release the tap-action
right away if it's interrupted by another key. After `held_threshold_millis` it goes into the
hold-preferred state, where it will emit the hold-action if it's interrupted. And finally
at `tap_timeout_millis` it will press the hold-action right away.

The parameters `tap_timeout_millis` and `held_threshold_millis` denote time since the tap-hold key was pressed.

The default held_threshold_millis is set to 0. Meaning the tap-hold key is hold-preferred by default.

In more detail:

- When emitting tap-action

  - If the tap-action consists of multiple keys, they are all pressed before any of they are released.
  - Repeat from the tap-hold key is suppresed and nothing happens when it's released.

- Tap-preferred from `0ms` to `held_threshold_millis`:

  - If interrupted by another key press → tap-action
    - The tap-action is pressed then released before the interupting key is emitted.
  - If released alone → tap-action

- Hold-preferred from `held_threshold_millis` to `tap_timeout_millis`:

  - If interrupted by another key press → hold-action
    - The hold-action is pressed before the interupting key is pressed,
      and the hold-action is released when the tap-hold key is released, independent of the interupting key.
    - Repeat from the tap-hold is suppresed
  - If released alone → tap-action

- Always-hold from `tap_timeout_millis` to `∞`:
  - At `tap_timeout_millis` the hold-action is pressed and it's released when the tap-hold key is released.

Repeat is suppressed when a decision about tap/hold hasn't been made yet.

The meaning of interrupted is when another key press is emitted from the `modmap`, this means
a physical key can be pressed without interupting a tap-hold key, as long as it's 'squashed' by
a remap in the `modmap`. But what happens in `keymap` doesn't matter as remapping there is performed after
tap-hold keys are interrupted.

Repeat and release events from another key pressed before the tap-hold do not
interrupt the tap-hold key. Except if they're mapped to a press in `modmap`.

Tap-hold keys can interrupt each other. But that complicated, and is it good or bad?

#### Example: tap-preferred

This configuration is like zmk's tap-preferred and qmk's default-tap-hold. Except the
tap-action is emitted right away if interrupted before timeout, while it's buffered until timeout
in the two other tools.

```yml
modmap:
  - remap:
      A:
        tap: A
        hold: Shift_l
        hold_threshold_millis: 200
        tap_timeout_millis: 200
```

#### Example: hold-preferred

This configuration is like zmk's hold-preferred and qmk's hold-on-other-key-press.

```yml
modmap:
  - remap:
      capslock:
        tap: esc
        hold: Shift_l
        hold_threshold_millis: 0 # This can be omitted, as 0 is the default value.
        tap_timeout_millis: 200
```

[Originally work](https://github.com/xremap/xremap/pull/718)

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
