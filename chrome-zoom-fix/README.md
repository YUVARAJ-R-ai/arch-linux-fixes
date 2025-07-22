The problem boils down to this: by default, Chrome (and other Chromium-based browsers) might not be running in its native Wayland mode. When it runs in the older X11 mode (via a compatibility layer called XWayland), it doesn't get direct access to the high-resolution touchpad events, including pinch-to-zoom.

Here is a step-by-step guide to fix this, starting with the most likely solution.

### 1. The Primary Fix: Force Chrome to Run in Wayland Mode

You need to tell Chrome to use its Wayland backend. You can do this by launching it with specific command-line flags.

**The Flags:**
*   `--ozone-platform-hint=auto` (This is the modern, preferred flag. It tells Chrome to auto-detect the environment and use Wayland if available).
*   `--enable-features=UseOzonePlatform` (This helps ensure the Ozone platform is used).

**How to Apply the Flags (Choose ONE Method):**

**Method A: Editing the `.desktop` File (Recommended)**

This is the best method because it will apply the fix every time you launch Chrome from your application launcher (like Rofi, Wofi, or the GNOME/KDE menu).

1.  **Copy the default `.desktop` file to your local directory.** This prevents your changes from being overwritten by a system update.
    ```bash
    cp /usr/share/applications/google-chrome.desktop ~/.local/share/applications/
    ```
    *If you are using Chromium, the file will be `chromium.desktop`.*

2.  **Edit the new file** with your favorite text editor (e.g., `nano`, `vim`, `vscode`).
    ```bash
    nano ~/.local/share/applications/google-chrome.desktop
    ```

3.  **Find the `Exec=` lines.** You will see a few of them, like `Exec=/usr/bin/google-chrome-stable %U`, `Exec=/usr/bin/google-chrome-stable`, etc.

4.  **Append the flags to EACH `Exec=` line.** Make sure to leave a space after the original command.

    **Before:**
    ```ini
    Exec=/usr/bin/google-chrome-stable %U
    ```
    **After:**
    ```ini
    Exec=/usr/bin/google-chrome-stable --ozone-platform-hint=auto --enable-features=UseOzonePlatform %U
    ```
    Modify all `Exec=` lines in the file this way.

5.  Save the file and exit the editor.

6.  **Important:** Close Chrome completely. Make sure no background processes are running (`killall chrome` or check with `ps aux | grep chrome`). Then, relaunch it from your application launcher.

Your pinch-to-zoom should now work smoothly.

**Method B: Using a Shell Alias (For Terminal Users)**

If you primarily launch Chrome from the terminal, you can create an alias in your shell's configuration file (`~/.bashrc`, `~/.zshrc`, etc.).

1.  Open your shell's config file: `nano ~/.zshrc`
2.  Add this line at the end:
    ```bash
    alias chrome="google-chrome-stable --ozone-platform-hint=auto --enable-features=UseOzonePlatform"
    ```
3.  Save the file and reload your shell configuration: `source ~/.zshrc`
4.  Now, typing `chrome` in the terminal will launch it with the correct flags.

**Method C: The Arch Linux `chrome-flags.conf` file**

The Arch Linux package for `google-chrome` provides a convenient way to set flags.

1.  Create or edit the file `~/.config/chrome-flags.conf`.
    ```bash
    nano ~/.config/chrome-flags.conf
    ```
2.  Add the flags to this file, one per line.
    ```
    --ozone-platform-hint=auto
    --enable-features=UseOzonePlatform
    ```
3.  Save the file. This method should be picked up automatically by the launch script.

---

### 2. Verify Your Input System (`libinput`)

Hyprland uses `libinput` to handle all input devices. Your touchpad gestures are processed by it.

1.  **Ensure `libinput` is installed.** It's a dependency of Hyprland, so it should be, but it's good to check.
    ```bash
    pacman -Qs libinput
    ```

2.  **Debug touchpad events.** This is a great way to see if the system is even recognizing your pinch gesture.
    ```bash
    sudo libinput debug-events
    ```
    Now, perform a pinch gesture on your touchpad. You should see output in the terminal that includes `GESTURE_PINCH_BEGIN`, `GESTURE_PINCH_UPDATE`, and `GESTURE_PINCH_END`.

    *   If you **see these events**, the problem is definitely with Chrome's configuration (go back to Step 1).
    *   If you **do not see these events**, the problem is deeper, possibly with your kernel drivers or touchpad configuration.

---

### 3. Check Your Hyprland Configuration

By default, Hyprland passes gestures it doesn't recognize (like pinch) to the active application. However, it's possible you have a custom gesture that is overriding this.

1.  Open your Hyprland config file: `~/.config/hypr/hyprland.conf`.
2.  Look for the `gestures { ... }` section.
3.  Make sure you **do not** have a `bind` for `pinch`. For example, if you have a line like `bind = , pinch, workspace, e+1`, it means Hyprland is "catching" the pinch gesture to switch workspaces instead of passing it to Chrome. You would need to remove or comment out that line.

A standard Hyprland gesture section is for things like swiping workspaces, not for application-specific zooming.

---

### Summary and Troubleshooting

1.  **Start with Step 1 (Chrome Flags).** This solves the issue for 99% of users on Hyprland/Wayland.
2.  **Restart Chrome Completely.** A simple close/re-open isn't always enough. Use `killall chrome` to be sure.
3.  **Verify with `libinput debug-events`** to confirm your hardware and drivers are working correctly.
4.  **Check `hyprland.conf`** for any conflicting gesture binds.
5.  **Bonus Tip:** To confirm Wayland is working, open `chrome://gpu` in Chrome's address bar. Look for "Ozone platform" and it should say **Wayland**. If it says X11, the flags have not been applied correctly.