# Rclone Setup & Sync Guide for Arch Linux

A complete walkthrough for setting up rclone on Arch Linux to sync local directories (including mounted drives) with Google Drive. This guide covers initial configuration, manual syncing, automation with cron, and solutions to common errors.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Part 1: Initial Rclone Configuration](#part-1-initial-rclone-configuration)
- [Part 2: Manual Syncing and Verification](#part-2-manual-syncing-and-verification)
- [Part 3: Debugging Common Issues](#part-3-debugging-common-issues)
- [Part 4: Automating Sync with Cron](#part-4-automating-sync-with-cron)
- [Troubleshooting](#troubleshooting)

## Prerequisites

- Arch Linux system
- Google account with Google Drive access
- Basic familiarity with terminal commands
- Admin/sudo access for package installation

## Part 1: Initial Rclone Configuration

### 1.1 Install Rclone

Install rclone from the official Arch repositories:

```bash
sudo pacman -S rclone
```

### 1.2 Configure Google Drive Remote

Launch the interactive configuration tool to create a "remote" (rclone's term for a configured cloud storage connection):

```bash
rclone config
```

Follow these prompts:

1. **New remote**: Enter `n`
2. **name>**: Enter a short, simple name (we'll use `gdrive`)
3. **Storage>**: Find Google Drive in the list and enter its corresponding number
4. **client_id>**: Press Enter (leave blank)
5. **client_secret>**: Press Enter (leave blank)
6. **scope>**: Choose `1` for full read/write access
7. **root_folder_id>**: Press Enter (gives access to entire Drive)
8. **service_account_file>**: Press Enter (leave blank)
9. **Edit advanced config?**: Enter `n` (No)
10. **Use auto config?**: Enter `y` (Yes) - This opens a browser window for authentication
11. **Configure as team drive?**: Enter `n` (No), unless using Google Workspace
12. **Confirm**: Enter `y` (Yes, this is OK)
13. **Exit**: Enter `q` to quit config

✅ Your `gdrive` remote is now configured and ready to use.

## Part 2: Manual Syncing and Verification

### 2.1 Understanding the Sync Command

Basic syntax: `rclone sync <source> <destination>`

- **Source**: Local directory to back up
- **Destination**: Remote location on Google Drive

> ⚠️ **Warning**: `rclone sync` makes the destination an exact mirror of the source. Files in the destination that don't exist in the source will be **deleted**. Always test with `--dry-run` first.

### 2.2 Finding the Correct Source Path

**For home directories**: Use `~/` notation
```bash
~/Documents
~/Pictures
```

**For mounted drives**: Use full system path
```bash
# Find mount points
df -h

# Example paths
/mnt/data1/folder
/media/username/drive/folder
```

**Paths with spaces**: Must be quoted
```bash
"/mnt/data1/college/Sem 3"
```

### 2.3 Running Manual Sync

**Step 1: Dry Run (Test)**
```bash
rclone sync --dry-run --progress "/mnt/data1/college/Sem 3" "gdrive:College/Semester 3"
```

**Step 2: Actual Sync**
```bash
rclone sync --progress "/mnt/data1/college/Sem 3" "gdrive:College/Semester 3"
```

The `--progress` flag provides live status updates.

## Part 3: Debugging Common Issues

### 3.1 Error: "directory not found" (Remote)

**Problem**: Running `rclone ls remote` fails

**Solution**: Specify the remote name with colon

```bash
# ❌ Incorrect
rclone ls remote

# ✅ Correct
rclone ls gdrive:
```

### 3.2 Error: "directory not found" (Local)

**Problem**: Source path doesn't exist or is misspelled

**Solutions**:
- Verify exact spelling and capitalization
- For mounted drives, use `df -h` to find correct mount point
- Use full absolute paths for mounted drives

### 3.3 Rclone Appears to "Hang" or "Freeze"

**Problem**: Rclone seems stuck at startup

**Explanation**: Rclone builds a complete file list before transferring. This can take time with large directories.

**Solutions**: Use these flags for better visibility and performance

```bash
rclone sync --progress --fast-list -vv "/mnt/data1/college/Sem 3" "gdrive:College/Semester 3"
```

- `--progress`: Shows live status
- `-vv`: Verbose logging
- `--fast-list`: Speeds up Google Drive file listing

## Part 4: Automating Sync with Cron

### 4.1 Install and Enable Cron Daemon

Arch Linux doesn't include cron by default:

```bash
# Install cronie
sudo pacman -S cronie

# Enable and start the service
sudo systemctl enable --now cronie.service
```

### 4.2 Setting Up Automated Sync

**Open crontab for editing**:
```bash
crontab -e
```

**Add sync schedule** (runs every hour):
```bash
0 * * * * /usr/bin/rclone sync "/mnt/data1/college/Sem 3" "gdrive:College/Semester 3" --log-file=/home/yuvaraj/rclone_sync.log
```

**Cron schedule breakdown**:
- `0 * * * *`: Run at minute 0 of every hour
- `/usr/bin/rclone`: Full path to rclone (find with `which rclone`)
- `--log-file=...`: Essential for debugging - logs all output

### 4.3 Crontab Editor Issues

**Problem**: VS Code doesn't save crontab properly

**Solution 1 (Recommended)**: Use nano
```bash
EDITOR=nano crontab -e
```
Save with `Ctrl+X`, `Y`, then `Enter`

**Solution 2**: Configure VS Code properly
```bash
export EDITOR="code --wait"
crontab -e
```
Must save and close the file tab in VS Code.

**Verify installation**:
```bash
crontab -l
```

You should see: `crontab: installing new crontab`

## Troubleshooting

### Check Cron Logs
```bash
# View system cron logs
sudo journalctl -u cronie

# View your rclone sync log
cat /home/yuvaraj/rclone_sync.log
```

### Test Rclone Commands
```bash
# List files in Google Drive
rclone ls gdrive:

# Check rclone version
rclone version

# Test connectivity
rclone lsd gdrive:
```

### Common Cron Issues
- **Paths**: Always use absolute paths in cron jobs
- **Environment**: Cron has minimal environment variables
- **Permissions**: Ensure cron can access source directories
- **Logs**: Always include `--log-file` for debugging

## Useful Commands

```bash
# Manual sync with full logging
rclone sync --progress --fast-list -vv "source" "dest" --log-file=sync.log

# Copy instead of sync (doesn't delete)
rclone copy --progress "source" "dest"

# Show what would be synced without doing it
rclone sync --dry-run --progress "source" "dest"

# Check differences between local and remote
rclone check "source" "dest"
```

---

## Contributing

Found an issue or have improvements? Please open an issue or submit a pull request.

## License

This guide is provided under the MIT License - feel free to use and modify as needed.