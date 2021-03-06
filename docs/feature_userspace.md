# Userspace: sharing code between keymaps

If you use more than one keyboard with a similar keymap, you might see the benefit in being able to share code between them. Create your own folder in `users/` named the same as your keymap (ideally your github username, `<name>`) with the following structure:

* `/users/<name>/` (added to the path automatically)
  * `readme.md`
  * `rules.mk` (included automatically)
  * `<name>.h` (optional)
  * `<name>.c` (optional)

`<name>.c` will need to be added to the SRC in `rules.mk` like this:

    SRC += <name>.c

Additional files may be added in the same way - it's recommended you have one named `<name>`.c/.h though.

All this only happens when you build a keymap named `<name>`, like this:

    make planck:<name>

For example, 

    make planck:jack

Will include the `/users/jack/` folder in the path, along with `/users/jack/rules.mk`.

## Readme

Please include authorship (your name, github username, email), and optionally [a license that's GPL compatible](https://www.gnu.org/licenses/license-list.html#GPLCompatibleLicenses).

## Example

For a brief example, checkout `/users/_example/` , or for a more detailed examples check out [`template.h`](https://github.com/qmk/qmk_firmware/blob/master/users/drashna/template.h) and [`template.c`](https://github.com/qmk/qmk_firmware/blob/master/users/drashna/template.c) in `/users/drashna/` .

### Consolidated Macros 

If you wanted to consoludate macros and other functions into your userspace for all of your keymaps, you can do that.  The issue is that you then cannot call any function defined in your userspace, or it gets complicated.  To better handle this, you can call the functions here and create new functions to use in individual keymaps. 

First, you'd want to go through all of your `keymap.c` files and replace `process_record_user` with `process_record_keymap` instead.   This way, you can still use keyboard specific codes on those boards, and use your custom "global" keycodes as well.   You'll also want to replace `SAFE_RANGE` with `NEW_SAFE_RANGE` so that you wont have any overlappind keycodes

Then add `#include <name.h>` to all of your keymap.c files.  This allows you to use these new keycodes without having to redefine them in each keymap.

Once you've done that, you'll want to set the keycode definitions that you need to the `<name>.h`  file. For instance:
```
#ifndef USERSPACE
#define USERSPACE

#include "quantum.h"

// Define all of 
enum custom_keycodes {
  KC_MAKE = SAFE_RANGE,
  NEW_SAFE_RANGE  //use "NEW_SAFE_RANGE" for keymap specific codes
};

#endif
```

Now you want to create the `<name>.c` file, and add this content to it:

```
#include "<name>.h"
#include "quantum.h"
#include "action.h"
#include "version.h"

__attribute__ ((weak))
bool process_record_keymap(uint16_t keycode, keyrecord_t *record) {
  return true;
}

bool process_record_user(uint16_t keycode, keyrecord_t *record) {
  switch (keycode) {
  case KC_MAKE:
    if (!record->event.pressed) {
      SEND_STRING("make " QMK_KEYBOARD ":" QMK_KEYMAP);
      SEND_STRING(SS_TAP(X_ENTER));
    }
    return false;
    break;
  }
  return process_record_keymap(keycode, record);
}
```

This will add a new `KC_MAKE`  keycode that can be used in any of your keymaps.  And this keycode will output `make <keyboard>:<keymap">`, making frequent compiling easier.  And this will work with any keyboard and any keymap as it will output the current boards info, so that you don't have to type this out every time. 

