---
name: device-hand
description: >-
  Drive a live Telephort device through MCP with the hand inside an attended
  app. Use when an agent taps, swipes, types, scrolls, or opens an app on a
  virtual device session. Forbids shell input, launching an attended app by
  package, and an accessibility click. Play Store and Chrome, used as the
  workplace, are passages. Does not create a device and does not place the finger.
compatibility: Any agent with the Telephort MCP session tools.
---

# Device hand

Speak the user's language. This skill is how to touch a running device. It does not create, start, or configure the device, and it does not sign into Play Store or install an app.

## Attended app

An attended app is any app other than Play Store (`com.android.vending`) and Chrome (`com.android.chrome`). A page that an attended app itself opened is still the attended app, including a page it opened in Chrome.

Enter it by tapping its icon or its card in Recents. Either is fine at any time. Do not open it with `action_launch_app` or a shell `am start`.

Inside it, only the hand changes the screen:

- `action_tap` names the control. Do not send x, y, or a point taken from a screenshot.
- `action_scroll` names a direction. Do not draw the swipe.
- `action_input_text` sends the whole string in one call. Do not send one key at a time and do not choose the gaps between keys.

You do not place the finger, shape the swipe, or space the keys. The product does that: a tap lands inside the named control in a different place each time, a swipe is uneven and a different length each time, and the gaps between keys vary, with a longer gap after a word.

## Limits

This skill does not make the device look like a physical phone. A virtual device and an open debugging channel stay visible to the device. Do not try to hide them. A passage driven by a command is still a command.

## Sight

See the screen with the accessibility tree or a UI dump, whichever shows the control.

- Tree: `ui_get_tree`, `ui_find_nodes`, `ui_eval_xpath`. Name the tap with `accessibility_xpath`.
- Dump: `ui_get_uiautomator_xml`, `ui_eval_uiautomator_xpath`. Name the tap with `uiautomator_xpath`.

A dump does not change the screen. You do not have to look again or wait before the next gesture.

## Instrument

An instrument changes the screen without the hand. Inside an attended app, do not use one:

- `action_launch_app`
- `adb_shell`, or a shell stream, that runs `input`, `am start`, an activity name, or a setting that changes the screen
- a click performed through accessibility instead of `action_tap`

Shell that only reads, including a UI dump, is not an instrument.

## Passage

Play Store and Chrome, when you are working in that app, are passages. An instrument is allowed there, including opening them by package. The hand is not required.

A page an attended app opened is not a passage, even when its package is Chrome.
