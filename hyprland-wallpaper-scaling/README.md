# Hyprland Wallpaper Scaling Fix: swww + HyDE Theming Scripts

A comprehensive troubleshooting guide for resolving wallpaper scaling issues in Hyprland when using swww with HyDE theming scripts.

## 🐛 Problem Description

### Symptoms
- ✅ **Initial boot**: Wallpaper displays correctly, filling the entire screen
- ❌ **After script change**: Newly selected wallpaper doesn't fill screen (appears centered with borders or scaled down)
- ❌ **Subsequent changes**: All wallpaper changes via script result in incorrect scaling
- 🔄 **Temporary fix**: Manually restarting `swww-daemon` and using `swww img /path/to/image.png` works for one instance
- 📱 **Image quality**: Issue persists even with correct resolution images that normally scale properly

## 🔍 Root Cause

The issue stems from a **Hyprland configuration reload** (`hyprctl reload config-only -q`) being triggered mid-process during wallpaper changes.

### Problem Sequence
1. User initiates wallpaper change (keybinding)
2. Master script (`wallpaper.sh`) executes
3. Color generation script (`color.set.sh`) runs
4. **🚨 Critical point**: `color.set.sh` triggers `hyprctl reload config-only -q` on exit
5. Hyprland reload puts `swww-daemon` into incorrect state for screen dimensions/scaling
6. Backend script (`wallpaper.swww.sh`) calls `swww img`
7. **Result**: Wallpaper fails to scale correctly

## 🛠️ Solution

### Quick Fix
Comment out or remove the Hyprland reload command from your theming script:

```bash
# In ~/.local/lib/hyde/color.set.sh (or your equivalent theming script)
[[ -n $HYPRLAND_INSTANCE_SIGNATURE ]] && {
    hyprctl keyword misc:disable_autoreload 1 -q
    # trap "hyprctl reload config-only -q" EXIT  # <-- COMMENT THIS LINE OUT
}
```

### Step-by-Step Instructions

1. **Locate the theming script**
   ```bash
   # Common location for HyDE users
   nano ~/.local/lib/hyde/color.set.sh
   ```

2. **Find the problematic line**
   Look for lines containing:
   - `trap "hyprctl reload config-only -q" EXIT`
   - `hyprctl reload`

3. **Comment out the reload command**
   ```bash
   # Before
   trap "hyprctl reload config-only -q" EXIT
   
   # After
   # trap "hyprctl reload config-only -q" EXIT
   ```

4. **Save and test**
   Save the file and test wallpaper changes

## 🧪 Debugging Process

If you encounter similar issues, follow this systematic approach:

### 1. Initial Verification
- ✅ Confirm swww version is up-to-date
- ✅ Test direct `swww img /path/to/image.png` in clean environment
- ✅ Verify image resolution and format

### 2. Script Isolation
```bash
# Temporarily disable script components in your main wallpaper script
# wallpaper.sh example:
swwwallcache.sh    # Comment out to test
# color.set.sh     # Comment out to test  
wallpaper.swww.sh  # Keep this enabled
```

### 3. Identify the Culprit
- Disable scripts one by one
- Test wallpaper changes after each modification
- Note which script causes the issue

### 4. Examine Script Content
```bash
# Look for suspicious commands
grep -n "hyprctl" ~/.local/lib/hyde/color.set.sh
grep -n "reload" ~/.local/lib/hyde/color.set.sh
grep -n "trap" ~/.local/lib/hyde/color.set.sh
```

## ⚖️ Trade-offs

### ✅ Pros
- Wallpaper scaling works correctly for all changes
- No more manual daemon restarts needed
- Consistent behavior across all wallpaper operations

### ⚠️ Cons
- Dynamic theme changes may not reflect immediately in all components
- Manual intervention might be needed for theme updates

### 🔧 Mitigation Strategies

**Option 1: Manual Reload**
```bash
# Add keybinding for manual reload when needed
hyprctl reload
```

**Option 2: Delayed Reload**
Move the reload command to the end of your main wallpaper script:
```bash
# At the end of wallpaper.sh
sleep 1  # Give swww time to complete
hyprctl reload config-only -q
```

**Option 3: Conditional Reload**
Only reload if theme actually changed:
```bash
if [[ "$OLD_THEME" != "$NEW_THEME" ]]; then
    hyprctl reload config-only -q
fi
```

## 🎯 General Troubleshooting Tips

1. **Test in isolation**: Run scripts individually to identify conflicts
2. **Check timing**: System-wide operations like `hyprctl reload` can interfere with running processes
3. **Monitor logs**: Check terminal output for error messages
4. **Environment stability**: Avoid system reloads during multi-step operations
5. **Version compatibility**: Ensure all tools (swww, Hyprland, HyDE) are compatible

## 📋 Environment Details

This guide was tested with:
- **Hyprland**: Latest stable
- **swww**: Latest version
- **HyDE Framework**: Recent version
- **Scripts involved**: `wallpaper.sh`, `color.set.sh`, `wallpaper.swww.sh`

## 🤝 Contributing

Found this helpful? Have a similar issue with different scripts? Feel free to:
- Open an issue with your specific case
- Submit a PR with additional solutions
- Share your configuration details

## 📄 License

This troubleshooting guide is provided as-is for the community. Feel free to adapt and share.

---

**💡 Pro Tip**: Always backup your configuration files before making changes, and test modifications in a safe environment first.