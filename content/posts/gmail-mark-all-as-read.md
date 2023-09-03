+++
title = 'Gmail Mark All as Read'
date = 2023-09-02T16:33:28-07:00
draft = false
+++

## Too many un-read Gmail messages

My gmail pertutually contains thousand of unread messages. Most of them are low signal but I don't have a quick (enough) to mark all as read. My old method was to 

* Create a earch filter like https://mail.google.com/mail/u/0/#search/label%3Aunread
* Select-all threads, and then *...* and then "Mark all as read"
* Sometimes, you can expand the selection to all  if there are paginations 

## Keyboard Shortcuts

This is too slow, so I looked up [Gmail keyboard shortcuts](https://support.google.com/mail/answer/6594?hl=en&co=GENIE.Platform%3DDesktop)

* Select all conversations	"* + a"
* Mark all as read "I"
* Deselect all conversations	* + n

This is 5 keystrokes, or I need to use the mouse. Still too slow.

## Mapping these into one single keystroke

On a mac keyboard, F6 is not mapped to anything useful. Let's use that to produce the 5 keystroke, with the help of [Karabiner-Elements](https://karabiner-elements.pqrs.org/)

Download and install is easy. Once launched, you do need to open "System References" to approve a few times (like 3).

Open/Create this file 
```
.config/karabiner/karabiner.json
```

Find the appropiate section for the following 

```
      "complex_modifications": {
        "parameters": {
          "basic.simultaneous_threshold_milliseconds": 50,
          "basic.to_delayed_action_delay_milliseconds": 500,
          "basic.to_if_alone_timeout_milliseconds": 1000,
          "basic.to_if_held_down_threshold_milliseconds": 500,
          "mouse_motion_to_scroll.speed": 100
        },
        "rules": [
          {
            "description": "gmail mark all aset read",
            "manipulators": [
              {
                "from": {
                  "key_code": "f6"
                },
                "to": [
                  {
                    "key_code": "8",
                    "modifiers": [
                      "left_shift"
                    ]
                  },
                  {
                    "key_code": "a"
                  },
                  {
                    "key_code": "i",
                    "modifiers": [
                      "left_shift"
                    ]
                  },
                  {
                    "key_code": "8",
                    "modifiers": [
                      "left_shift"
                    ]
                  },
                  {
                    "key_code": "n"
                  }
                ],
                "type": "basic"
              }
            ]
          }
        ]
```

For a US keyboard, "Shift + 8 " is bascally a *. 

The above maps F6 into "* + a + I + * + n" 

- Select All
- Mark All as Read
- Unselect All 

Now it's finally one keystroke.