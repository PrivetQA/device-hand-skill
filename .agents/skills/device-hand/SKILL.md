---
name: device-hand
description: >-
  Drive a live Telephort device through MCP with the hand inside an attended
  app. Use when an agent taps, swipes, types, scrolls, or opens an app on a
  device session. Type from the on-screen keyboard. A physical model uses
  the measured Gboard layout. A virtual model measures the keys on its own
  screen and does not reuse those points. Forbids launching an attended app by package
  and an accessibility click. Play Store and Chrome, used as the workplace,
  are passages. Does not create a device.
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
- Type every character the user asked for from the on-screen keyboard, on a virtual model and on a physical one. Do not use `action_input_text`, `input text`, an accessibility `setText`, or the clipboard.

You do not place the finger on a named control or shape the swipe. The product does that: a tap lands inside the named control in a different place each time, a second tap in the same series lands somewhere else inside it, a swipe is uneven and a different length each time, and a short gap is left before the next gesture. Typing is the exception: each character is a tap on its key.

## Limits

This skill does not make the device look like a physical phone. A virtual device and an open debugging channel stay visible to the device. Do not try to hide them. A passage driven by a command is still a command.

## Sight

See the screen with the accessibility tree. Use a UI dump only when that tree does not show the control. A dump starts a process named `uiautomator`.

- Tree: `ui_get_tree`, `ui_find_nodes`, `ui_eval_xpath`. Name the tap with `accessibility_xpath`.
- Dump, only when the tree has no match: `ui_get_uiautomator_xml`, `ui_eval_uiautomator_xpath`. Name the tap with `uiautomator_xpath`.

A dump does not change an attended page. It does dismiss the on-screen keyboard. Do not dump, and do not read the accessibility tree, while that keyboard is open. See whether it is open with `dumpsys input_method`: `mInputShown=true`, and `mServedView` is the field you are typing into. That read does not close it.

## Instrument

An instrument changes the screen without the hand. Inside an attended app, do not use one:

- `action_launch_app`
- `adb_shell`, or a shell stream, that runs `input text`, `input swipe`, `input keyevent`, `am start`, an activity name, or a setting that changes the screen
- a click performed through accessibility instead of `action_tap`

The one shell write allowed in an attended app is `input tap` on a key in the Typing layout. A swipe from a key opened another app. Do not swipe a key.

Shell that only reads, including `dumpsys input_method`, is not an instrument. A UI dump is a read that still closes the keyboard.

## Passage

Play Store and Chrome, when you are working in that app, are passages. An instrument is allowed there, including opening them by package. The hand is not required.

A page an attended app opened is not a passage, even when its package is Chrome.

## Typing

On a physical model and on a virtual one, each character is a tap on the key that is on the screen. The keyboard stays up, the same way for both: `dumpsys input_method` must show `mInputShown=true` and the served view must be the field. If the keyboard is closed, stop. Do not tap the field, the page, or Back to open it. Do not press Back, tap outside the keyboard, tap the field again, or dump the UI while it is open. One `input tap` per character, a few pixels off the center of that key, 110–180 ms apart. Do not swipe a key. Do not tap Continue, the eye, or the checkmark unless the user asked to submit. Do not tap the navigation bar under the keyboard.

A physical model uses the Gboard layout below. A virtual model does not. Its keyboard can be a different size and a different arrangement. If its letter keys are nodes, `action_tap` names the key. If they are not, take one screenshot and count the keys that are actually there, then tap those points. Do not copy the physical coordinates onto a virtual screen. If a tap does not produce the character, count again from a new screenshot. Do not insert the whole string.

The physical points were measured on Gboard (`com.google.android.inputmethod.latin`) in portrait on a 540×1200 screen. Scale them for another physical `wm size`: `x = round(px * width / 540)`, `y = round(py * height / 1200)`. Those letter keys are not in the accessibility tree, so a named tap cannot hit them. On that screen the keys are about 47×60; stay at least 8 px inside, and scale the inset with the width. Nothing below y 1112: that is the navigation bar.

Letters, top to bottom. The y values are the key centers.

| y | keys, left to right, x |
| --- | --- |
| 852 | q 29, w 82, e 135, r 188, t 241, y 294, u 347, i 401, o 455, p 509 |
| 928 | a 56, s 109, d 162, f 215, g 268, h 321, j 374, k 428, l 482 |
| 1004 | z 109, x 162, c 215, v 268, b 321, n 374, m 428 |

Shift is the blue key at 42, 1004. Backspace is the blue key at 496, 1004. Tap shift once, then the letter, for one capital.

Digits are the top row after one tap on ?123 at 40, 1080. That row uses the same centers as q–p at y 852: 1 is q, 8 is i, 7 is u, 0 is p. Do not long-press a letter to reach its digit. Space is 295, 1084. Comma is 109, 1084. Period is 429, 1084. The checkmark is 496, 1084; leave it.

Letter taps, one tap on ?123, then digit taps on that top row landed in the field. The rest of the grid is the same measurement. If a tap does not produce the character, take one screenshot and remeasure. Do not switch to inserting the whole string.

How to count keys on a screenshot. A white key is RGB at least 248 and wider than 18 px. Ten keys in a row are q–p or 1–0. Nine are a–l. Seven are z–m. The x center is the middle of that blob. The y center is the middle of the white run on the side of the key: the glyph is dark and splits a column through the middle. ?123 is the light-blue key at the lower left, above the navigation icons.
