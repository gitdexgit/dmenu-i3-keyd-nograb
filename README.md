# dmenu-i3-keyd (Arch Linux)

Custom `dmenu` build optimized for i3-wm and keyd. Features non-blocking input, toggle-undo, and clipboard integration.

## 1. dmenu.c Modifications

### A. Global Variables & Undo Helper
Add near the top of `dmenu.c`:
```c
static char utext[BUFSIZ] = "";
static size_t ucursor = 0;

static void
saveundo(void)
{
	strcpy(utext, text);
	ucursor = cursor;
}
```

### B. Keyboard Grab (Non-blocking)
In `main()`, comment out `grabkeyboard()` to allow dmenu to float without locking the X session:
```c
// grabkeyboard();
readstdin();
setup();
```

### C. Keypress Logic (`keypress` function)
Add these cases inside the `if (ev->state & ControlMask)` block:

```c
/* 1. Toggle Undo */
case XK_z:
case XK_Z:
    {
        char tmpt[BUFSIZ]; size_t tmpc;
        strcpy(tmpt, text); tmpc = cursor;
        strcpy(text, utext); cursor = ucursor;
        strcpy(utext, tmpt); ucursor = tmpc;
        match();
    }
    break;

/* 2. Copy Input to Clipboard */
case XK_c:
case XK_C:
    {
        FILE *f = popen("xclip -selection clipboard", "w");
        if (f) { fprintf(f, "%s", text); pclose(f); }
    }
    break;

/* 3. Modern Paste */
case XK_v:
case XK_V:
case XK_y:
case XK_Y:
    XConvertSelection(dpy, (ev->state & ShiftMask) ? clip : XA_PRIMARY,
                      utf8, utf8, win, CurrentTime);
    return;
```

**Note:** Call `saveundo();` at the start of `insert()`, `XK_k`, `XK_u`, and `XK_w` cases to ensure state is captured before destruction.

---

## 2. i3 Configuration
Add to `~/.config/i3/config` to handle the non-grabbing dmenu window:

```bash
# Float dmenu and snap to top-left
for_window [class="dmenu"] floating enable, border pixel 2, move absolute position 0 0
```

---

## 3. keyd Integration
One-liner for `copyq` menu support without external scripts.

**`keyd` config:**
```python
c = command(sudo -u dex -H env DISPLAY=:0 XAUTHORITY=/home/dex/.Xauthority DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus copyq menu &)
```

---

## 4. Requirements
- **Arch Linux**
- **i3-wm**
- **keyd**
- **xclip** (Required for `Ctrl+c` functionality)
- **copyq** (Required for the `c` shortcut)

## 5. Build
```bash
make
sudo make install
```
