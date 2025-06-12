# pgAdmin4 Setup on Arch Linux (Hyprland) 🐘

> A comprehensive guide for installing pgAdmin4 in a Python virtual environment on Arch Linux with custom launcher integration for Hyprland window manager.

## 📋 Table of Contents

- [Introduction](#-introduction)
- [Prerequisites](#-prerequisites)
- [Installation Steps](#-installation-steps)
  - [1. Install PostgreSQL](#1-install-postgresql)
  - [2. Create Virtual Environment](#2-create-virtual-environment)
  - [3. Create Launch Script](#3-create-launch-script)
  - [4. Create Desktop Entry](#4-create-desktop-entry)
  - [5. Add Custom Icon](#5-add-custom-icon)
  - [6. Launch pgAdmin4](#6-launch-pgadmin4)
  - [7. Connect to PostgreSQL](#7-connect-to-postgresql)
- [Troubleshooting](#-troubleshooting)
- [Features](#-features)

## 🌟 Introduction

**pgAdmin4** is a powerful open-source administration and development platform for PostgreSQL. This guide provides a clean installation method using Python virtual environments to avoid dependency conflicts, with seamless integration into Hyprland's application launcher.

### Why Use This Approach?

- ✅ **Isolated Dependencies** - No conflicts with system Python packages
- ✅ **Custom Integration** - Perfect for minimal window managers like Hyprland
- ✅ **Easy Management** - Simple launcher with custom icon support
- ✅ **Clean Uninstall** - Remove virtual environment without system impact

## 🔧 Prerequisites

Ensure you have the following installed on your Arch Linux system:

| Requirement | Installation Command | Purpose |
|-------------|---------------------|---------|
| **Python 3** | Usually pre-installed | Runtime environment |
| **pip & virtualenv** | `sudo pacman -S python-pip python-virtualenv` | Package management |
| **git** | `sudo pacman -S git` | Repository cloning |
| **Terminal Emulator** | `kitty`, `alacritty`, `wezterm`, etc. | Script execution |
| **Application Launcher** | `rofi`, `wofi`, `fuzzel`, etc. | Menu integration |

## 🚀 Installation Steps

### 1. Install PostgreSQL

If PostgreSQL isn't already installed:

```bash
# Install PostgreSQL
sudo pacman -S postgresql

# Initialize database
sudo -u postgres initdb --locale en_US.UTF-8 -D /var/lib/postgres/data

# Start and enable service
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Set postgres user password
sudo -u postgres psql
```

```sql
ALTER USER postgres WITH PASSWORD 'your_strong_password';
\q
```

> ⚠️ **Important**: Replace `'your_strong_password'` with a secure password.

### 2. Create Virtual Environment

Create isolated Python environment for pgAdmin4:

```bash
# Install required packages
sudo pacman -S python-pip python-virtualenv

# Create pgAdmin4 directories
sudo mkdir -p /var/lib/pgadmin /var/log/pgadmin
sudo chown $USER:$USER /var/lib/pgadmin /var/log/pgadmin

# Create virtual environment
python3 -m venv ~/pgadmin4_venv

# Activate environment
source ~/pgadmin4_venv/bin/activate

# Install pgAdmin4
pip install pgadmin4

# Deactivate environment
deactivate
```

### 3. Create Launch Script

Create a custom launcher script:

```bash
# Create scripts directory
mkdir -p ~/scripts/

# Create launch script
nano ~/scripts/launch_pgadmin4.sh
```

Add the following content:

```bash
#!/bin/bash

# --- Configuration ---
PGADMIN_VENV_PATH="$HOME/pgadmin4_venv"
TERMINAL_EMULATOR="kitty"  # Customize to your preferred terminal
TERMINAL_ARGS="-e"         # Adjust based on your terminal

# --- Script Logic ---
if [ ! -d "$PGADMIN_VENV_PATH" ]; then
    "$TERMINAL_EMULATOR" "$TERMINAL_ARGS" bash -c \
        "echo 'Error: pgAdmin4 virtual environment not found at $PGADMIN_VENV_PATH'; \
         echo 'Please ensure the path is correct and the virtual environment is created.'; \
         echo 'Press Enter to close this window.'; \
         read"
    exit 1
fi

PGADMIN_RUN_COMMAND="source \"$PGADMIN_VENV_PATH/bin/activate\" && pgadmin4 && \
                     echo -e '\\npgAdmin4 exited. Press Enter to close this window.' && read"

"$TERMINAL_EMULATOR" "$TERMINAL_ARGS" bash -c "$PGADMIN_RUN_COMMAND"
```

Make script executable:

```bash
chmod +x ~/scripts/launch_pgadmin4.sh
```

### 4. Create Desktop Entry

Create application launcher integration:

```bash
# Create applications directory
mkdir -p ~/.local/share/applications/

# Create desktop entry
nano ~/.local/share/applications/pgadmin4-venv.desktop
```

Add the following content:

```ini
[Desktop Entry]
Name=pgAdmin4 (Virtual Env)
Comment=Launch pgAdmin4 from its Python virtual environment
Exec=/home/YOUR_USERNAME/scripts/launch_pgadmin4.sh
Icon=pgadmin4
Terminal=false
Type=Application
Categories=Development;Database;Utility;
```

> 📝 **Note**: Replace `YOUR_USERNAME` with your actual username.

Make desktop entry executable:

```bash
chmod +x ~/.local/share/applications/pgadmin4-venv.desktop
```

### 5. Add Custom Icon

Personalize with a custom icon:

```bash
# Create custom icons directory
mkdir -p ~/.local/share/icons/custom/

# Move your icon file (replace with actual path)
mv /path/to/your/pgadmin4_icon.png ~/.local/share/icons/custom/pgadmin4_custom.png
```

Update the desktop entry:

```bash
nano ~/.local/share/applications/pgadmin4-venv.desktop
```

Change the Icon line:

```ini
Icon=/home/YOUR_USERNAME/.local/share/icons/custom/pgadmin4_custom.png
```

### 6. Launch pgAdmin4

After setup completion:

1. **Refresh Application Launcher**: Close and reopen your launcher (rofi/wofi)
2. **Find Application**: Search for "pgAdmin4" or "Virtual Env"
3. **Launch**: Select the entry to start pgAdmin4
4. **Access Interface**: Browser will open to `http://127.0.0.1:5050`

> 💡 **Tip**: Keep the terminal window open while using pgAdmin4.

### 7. Connect to PostgreSQL

On first launch:

1. Set up initial user account (email and password)
2. Click **"Add New Server"**
3. Configure connection:

| Field | Value |
|-------|-------|
| **Name** | My Local PostgreSQL |
| **Host** | `localhost` or `127.0.0.1` |
| **Port** | `5432` |
| **Database** | `postgres` |
| **Username** | `postgres` |
| **Password** | Your PostgreSQL password |

4. Click **"Save"** to connect

## 🔧 Troubleshooting

<details>
<summary><strong>Virtual Environment Not Found</strong></summary>

**Issue**: Error message about missing virtual environment

**Solution**: 
- Check `PGADMIN_VENV_PATH` in your launch script
- Ensure the path points to your `pgadmin4_venv` directory
- Verify the virtual environment was created successfully
</details>

<details>
<summary><strong>Server Could Not Be Contacted</strong></summary>

**Issue**: pgAdmin4 server connection fails

**Solutions**:
- Check terminal output for error messages
- Verify port 5050 is not in use: `sudo ss -tuln | grep 5050`
- Reinstall pgAdmin4: `pip install pgadmin4` (in activated venv)
</details>

<details>
<summary><strong>Icon Not Showing</strong></summary>

**Issue**: Custom icon doesn't appear in launcher

**Solutions**:
- Verify `Icon=` path in `.desktop` file is absolute and correct
- Check icon file format (PNG/SVG recommended)
- Log out and back into your desktop session
</details>

<details>
<summary><strong>Script Not Launching</strong></summary>

**Issue**: Launcher doesn't execute the script

**Solutions**:
- Verify `Exec=` path in `.desktop` file is correct
- Ensure script is executable: `chmod +x ~/scripts/launch_pgadmin4.sh`
- Check `Terminal=false` in `.desktop` file
</details>

## ✨ Features

- 🔒 **Isolated Installation** - Virtual environment prevents conflicts
- 🎨 **Custom Integration** - Seamless Hyprland launcher integration
- 🚀 **Easy Launch** - One-click access from application menu
- 🎯 **Custom Icon** - Personalized appearance
- 🛠️ **Terminal Output** - Easy debugging with visible error messages
- 🧹 **Clean Removal** - Simple uninstall by removing virtual environment

---

<div align="center">

**Made with ❤️ for the Arch Linux & Hyprland community**

[Report Bug](../../issues) • [Request Feature](../../issues) • [Contribute](../../pulls)

</div>