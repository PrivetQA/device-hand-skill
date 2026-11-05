---
name: device-hand
description: >-
  Drive a live Telephort device through MCP with the hand. Use when an agent
  taps, swipes, types, scrolls, or opens an app on a device session. Every
  character, in Play and in an attended app, comes from the on-screen
  keyboard. A physical model uses the measured Gboard layout. A virtual
  model measures the keys on its own screen. A swipe, including a date
  wheel, is one finger stroke. A UI dump may close the keyboard; reopen it
  with a random tap in the field. Play Store and Chrome are passages for
  opening only. Does not create a device.
compatibility: Any agent with the Telephort MCP session tools.
---

# Device hand

Speak the user's language. This skill is how to touch a running device. It does not create, start, or configure the device, and it does not sign into Play Store or install an app.

## Attended app

An attended app is any app other than Play Store (`com.android.vending`) and Chrome (`com.android.chrome`). A page that an attended app itself opened is still the attended app, including a page it opened in Chrome.

Enter it by tapping its icon or its card in Recents. Either is fine at any time. Do not open it with `action_launch_app` or a shell `am start`.

Inside it, only the hand changes the screen:

- A button tap is `input tap` at a point you choose. Do not use `action_tap` for it. That call asks the running service to place the finger, and the service still running does not cover the whole button.
- A swipe is one finger that stays down. Do not use `action_scroll` or `input swipe`. `input swipe` is a straight injected line and can leave the app.
- Type every character from the on-screen keyboard, on a virtual model and on a physical one, in an attended app and in a passage. Email, password, a code, a name, a username, a phone number, and an SMS code are that kind of text. Do not use `action_input_text`, `input text`, an accessibility `setText`, the clipboard, or a digit key code.

You place every tap. Read the control's rectangle from the tree or, when the keyboard is closed, from a dump. The rectangle is `[left, right)` by `[top, bottom)`. Pick a new integer each time:

`x = left + random(0, right - left - 1)`
`y = top + random(0, bottom - top - 1)`

If `right` or `bottom` is past `wm size`, the bounds are in the larger frame. Halve all four edges, then pick inside that smaller rectangle. One hundred taps on the same button must be able to land on any pixel of it, including the corners, not only the center. Typing uses the same pick, inside the key.

## Swipe

Send it on the input channel as `multi-touch` for one pointer. The first message has `down: 1` at the start. The later messages keep `down: 1` and move. The last message has `down: 0` at the end. Do not lift between samples.

Do not use one clock. The gap between samples wanders from about 4 ms to about 22 ms, and a stroke is sometimes a quick flick of roughly 150–310 ms and sometimes a slower drag of roughly 280–480 ms. Place samples with minimum-jerk progress, `t³ (10 - 15t + 6t²)`, warped so the fast part is not always in the middle. The finger is stopped at the press and at the lift.

The stroke does not cross the whole path you were given. Its length changes every time: usually about a quarter to a half of that path, and at most about two thirds. On a phone that is a short flick, about 2–6 cm, not a drag across the screen. A path that is already shorter than about 48 px stays nearly whole. Add one shallow bow, at most 9% of the stroke. Keep it on the page, away from the navigation bar.

A date wheel uses this same stroke on the wheel. So does every other swipe. Do not use `input swipe` for one.

The message has only `x`, `y`, and `down`. It has no pressure and no contact size, so the phone does not receive a fingertip ellipse. A real finger on a 540×1200 screen would cover about 8–11 mm along the stroke and 6–8 mm across, roughly 65–90 by 50–65 px. Android pressure, on a 0–1 scale, would be about 0.3 at contact, about 0.5 in the middle, and about 0.2 at the lift. Do not add those fields until the host reads them.

## Back

Back is a control on the screen. Do not send `input keyevent`, including key 4. A key event has no finger and no place on the glass.

Look at the navigation strip first. On the measured 540×1200 phone that strip is below y 1112. If three buttons are there, Back is the triangle, usually the left one. Pick a point inside that button the same way as any other button, and send `input tap`. Do not tap the circle or the square. If the strip is only a pill, there is no Back button. A short finger swipe in from the left or right edge is Back. Do not tap the pill.

Do not press Back while a character is still being typed. Use it to leave a page, or to put the keyboard down, only when that is the action.

## Submit

Submitting a field is a tap on the checkmark key. That key sits at the bottom right of the keyboard: a light key with a tick on it. Do not send `input keyevent 66`. Do not send Enter by any other call.

On the measured Gboard portrait layout the tick is centered at 496, 1084. The key is about 47×60 around that center. Scale it with `wm size` on another physical screen. Pick the point inside the key, and keep it above the navigation strip. A virtual keyboard can put this key somewhere else. Find the tick on that screen. Do not copy 496, 1084.

Tap it only when the field is ready to submit. While characters are still being typed, leave it.

## Clear

Clearing a field is one tap on the backspace key for each character. That key shows an arrow box with a cross, on the right of the bottom letter row. Do not send `input keyevent 67`, do not send `input keyevent 123`, and do not delete the field in one batch.

On the measured Gboard portrait layout the backspace key is centered at 496, 1004. The key is about 47×60 around that center. Scale it with `wm size` on another physical screen. Pick a new point inside the key for every tap, and wait 110–180 ms. A virtual keyboard can put this key somewhere else. Find the arrow-box glyph on that screen. Do not copy 496, 1004.

Tap once for each character that has to go. Stop when those characters are gone. If the keyboard closes, tap a random point in the field to bring it back, then continue on the backspace key.

## Limits

This skill does not make the device look like a physical phone. A virtual device and an open debugging channel stay visible to the device. Do not try to hide them.

An app does not see a process named `uiautomator` or `monkey`. Android does not give an app the list of other processes. A dump shows up inside the app as the keyboard closing. A launch from the shell is not labeled monkey. The app receives an ordinary open intent. On Android 14 and newer the activity can be asked which package started it, and a shell start is not the launcher. Do not claim the app saw the process name.

## Sight

See the screen with the accessibility tree. Use a UI dump only when that tree does not show the control. The shell runs that dump as `uiautomator`. The app is not given that process name.

- Tree: `ui_get_tree`, `ui_find_nodes`, `ui_eval_xpath`. Name the tap with `accessibility_xpath`.
- Dump, only when the tree has no match: `ui_get_uiautomator_xml`, `ui_eval_uiautomator_xpath`. Name the tap with `uiautomator_xpath`.

A dump does not change the page. It does dismiss the on-screen keyboard. That is fine. Bring the keyboard back with one `input tap` at a random point inside the input field, the same pick as any other control. Do not use a key to show it. See whether it is open with `dumpsys input_method`: `mInputShown=true`, and `mServedView` is the field you are typing into. That read does not close it.

## Instrument

An instrument changes the screen without the hand. Inside an attended app, do not use one:

- `action_launch_app`
- `adb_shell`, or a shell stream, that runs `input text`, `input swipe`, `input keyevent`, `am start`, an activity name, or a setting that changes the screen. The ban on `input text`, `input swipe`, and `input keyevent` also holds in a passage. Back is the navigation control. Submit is the checkmark on the keyboard. Delete is the backspace key, one tap per character. Digits are the keyboard row, not key codes 7–16. Do not send key 4, key 66, key 67, or key 123.
- a click performed through accessibility instead of `action_tap`

The shell write allowed in an attended app is `input tap` at the point you picked inside a button or a key. A swipe from a key opened another app. Do not swipe a key.

Shell that only reads, including `dumpsys input_method`, is not an instrument. A UI dump is a read that still closes the keyboard.

## Passage

Play Store and Chrome, when you are working in that app, are passages. Opening them by package is allowed. Typing, deleting, submitting, and swiping there use the same hand as everywhere else.

A page an attended app opened is not a passage, even when its package is Chrome.

## Typing

On a physical model and on a virtual one, in Play and in an attended app, each character is a tap on the key that is on the screen. That includes email, password, a verification code, a name, a username, a phone number, and an SMS code. The keyboard stays up, the same way for both: `dumpsys input_method` must show `mInputShown=true` and the served view must be the field. If it is closed, open it with one random tap inside the field. Do not press Back, tap outside the field, or send a key to open it. One `input tap` per character. The table below gives the key's center only so you know which key. The rectangle is about 47×60 around that center on a 540×1200 screen; scale it with `wm size`. Pick the tap with the same formula as a button, inside that rectangle, and wait 110–180 ms. Do not aim at the center. Do not swipe a key. Do not tap Continue or the eye unless the user asked. To submit the field, tap the checkmark key. Digits are the top row after ?123, never key codes. Do not tap the navigation bar under the keyboard.

A physical model uses the Gboard layout below. A virtual model does not. Its keyboard can be a different size and a different arrangement. Take one screenshot and count the keys that are actually there, then pick a point inside each key yourself. Do not copy the physical coordinates onto a virtual screen, and do not hand the point to `action_tap`. If a tap does not produce the character, count again from a new screenshot. Do not insert the whole string.

The physical points were measured on Gboard (`com.google.android.inputmethod.latin`) in portrait on a 540×1200 screen. Scale them for another physical `wm size`: `x = round(px * width / 540)`, `y = round(py * height / 1200)`. Those letter keys are not in the accessibility tree, so a named tap cannot hit them. On that screen a key is about 47×60 around the center in the table. Pick the tap uniformly inside that rectangle. Nothing below y 1112: that is the navigation bar.

Letters, top to bottom. The y values are the key centers.

| y | keys, left to right, x |
| --- | --- |
| 852 | q 29, w 82, e 135, r 188, t 241, y 294, u 347, i 401, o 455, p 509 |
| 928 | a 56, s 109, d 162, f 215, g 268, h 321, j 374, k 428, l 482 |
| 1004 | z 109, x 162, c 215, v 268, b 321, n 374, m 428 |

Shift is the blue key at 42, 1004. Backspace is the arrow-box key at 496, 1004. One tap removes one character. Tap shift once, then the letter, for one capital.

Digits are the top row after one tap on ?123 at 40, 1080. That row uses the same centers as q–p at y 852: 1 is q, 8 is i, 7 is u, 0 is p. Do not long-press a letter to reach its digit. Space is 295, 1084. Comma is 109, 1084. Period is 429, 1084. The checkmark is the light key at 496, 1084. That tap submits the field.

Letter taps, one tap on ?123, then digit taps on that top row landed in the field. The rest of the grid is the same measurement. If a tap does not produce the character, take one screenshot and remeasure. Do not switch to inserting the whole string.

How to count keys on a screenshot. A white key is RGB at least 248 and wider than 18 px. Ten keys in a row are q–p or 1–0. Nine are a–l. Seven are z–m. The x center is the middle of that blob. The y center is the middle of the white run on the side of the key: the glyph is dark and splits a column through the middle. ?123 is the light-blue key at the lower left, above the navigation icons.
