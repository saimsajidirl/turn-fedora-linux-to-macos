# Complete WhiteSur macOS Theme Setup Guide for Fedora / GNOME

This guide provides a step-by-step walkthrough for turning your Fedora GNOME desktop into a full macOS WhiteSur aesthetic. It includes setting up Extension Manager, installing essential GNOME extensions, applying light mode, fixing LibAdwaita/GTK4 compatibility issues, and applying traffic light controls across all app types.

---

## 1. Prerequisites & Essential Packages

Before configuring visual themes, install the required build packages, theme engine dependencies, and system integration tools.

Open your terminal and execute:

```bash
# Update package repositories and install core dependencies
sudo dnf install -y sassc optipng inkscape gtk3-devel gnome-tweaks extension-manager

# Install system-level User Themes extension and Flatpak theme converters
sudo dnf install -y gnome-shell-extension-user-theme ostree libappstream-glib
```

---

## 2. Downloading & Installing Theme Assets

Download the WhiteSur GTK theme and icon repositories into your home folder.

```bash
# Clone the WhiteSur GTK theme repository
cd ~
git clone https://github.com/vinceliuice/WhiteSur-gtk-theme.git --depth=1

# Clone the WhiteSur Icon theme repository
git clone https://github.com/vinceliuice/WhiteSur-icon-theme.git --depth=1

# Install WhiteSur GTK Theme (Light & Dark variants with solid backgrounds for better performance)
cd ~/WhiteSur-gtk-theme
./install.sh -N mojave --opacity solid -c light

# Install WhiteSur Icon Theme
cd ~/WhiteSur-icon-theme
./install.sh -a
```

---

## 3. Configuring GNOME Extensions via Extension Manager

To achieve the macOS layout (dock at the bottom, top menu bar, and user themes enabled), configure extensions:

1. Open **Extension Manager** from your Application Grid.
2. Go to the **Installed** tab and ensure **User Themes** is toggled **ON**.
3. Go to the **Browse** tab in Extension Manager and search for and install:
   * **Dash to Dock** (Creates a macOS-style dock at the bottom)
   * **Blur my Shell** (Adds dynamic background blurring behind the top bar)
   * **Compiz alike magic lamp effect** (Adds the macOS "Genie" minimize animation)

### Configuring Dash to Dock for macOS Look:
1. Open **Extension Manager**, find **Dash to Dock**, and click the settings icon gear.
2. In **Position and Size**:
   * Set **Position on screen** to **Bottom**.
   * Turn **OFF** *Intellihide* if you want the dock visible constantly, or leave it **ON** for autohide.
3. In **Appearance**:
   * Turn **OFF** *Use built-in theme*.
   * Enable *Shrink to dash* to round the dock corners like macOS.

---

## 4. Setting Window Control Layout (Traffic Lights Position)

Move standard GNOME window controls to the top-left corner:

```bash
# Place close, minimize, and maximize buttons on the left side
gsettings set org.gnome.desktop.wm.preferences button-layout 'close,minimize,maximize:'
```

---

## 5. Applying Global Theme & Light Mode

Apply the installed WhiteSur theme across all desktop environments and set system preferences to Light mode:

```bash
# Set GTK and Icon themes
gsettings set org.gnome.desktop.interface gtk-theme 'WhiteSur-Light'
gsettings set org.gnome.desktop.interface icon-theme 'WhiteSur-light'

# Set GNOME Shell theme
gsettings set org.gnome.shell.extensions.user-theme name 'WhiteSur-Light'

# Enable GNOME User Theme extension via shell settings
gnome-extensions enable user-theme@gnome-shell-extensions.gcampax.github.com

# Set system-wide preference to light mode
gsettings set org.gnome.desktop.interface color-scheme 'prefer-light'
```

---

## 6. Fixing GTK4 / LibAdwaita Apps (Nautilus Traffic Lights Fix)

Modern GTK4 / LibAdwaita applications (like Files/Nautilus, Settings, and Text Editor) ignore `~/.themes` settings by default and hit strict `color-mix()` CSS syntax parsing errors on modern GTK releases (GTK 4.14+).

To fix the broken controls and render colored traffic lights on GTK4 apps:

```bash
# 1. Clear any broken auto-generated GTK4 configuration files
rm -rf ~/.config/gtk-4.0/*

# 2. Ensure GTK4 configuration directory exists
mkdir -p ~/.config/gtk-4.0

# 3. Create direct links to pre-compiled WhiteSur stylesheets
echo '@import url("file://'${HOME}'/.themes/WhiteSur-Light/gtk-4.0/gtk.css");' > ~/.config/gtk-4.0/gtk.css
echo '@import url("file://'${HOME}'/.themes/WhiteSur-Light/gtk-4.0/gtk-dark.css");' > ~/.config/gtk-4.0/gtk-dark.css
```

---

## 7. Configuring Flatpak Applications

Flatpak applications run in isolated sandboxes and need explicit permission to access user theme directories and GTK configs:

```bash
# Allow Flatpaks to read system GTK4 settings and themes
flatpak override --user --filesystem=~/.config/gtk-4.0:ro
flatpak override --user --filesystem=~/.themes:ro
flatpak override --user --filesystem=~/.local/share/icons:ro
```

---

## 8. App-Specific Titlebar Configurations

### Google Chrome / Chromium / Brave / Vivaldi
1. Open the browser.
2. Right-click any empty space in the **top tab bar area** (next to the `+` button).
3. Check **Use system title bar and borders**.

### Mozilla Firefox
Ensure Firefox is completely closed before patching:

```bash
# Terminate Firefox process
killall firefox 2>/dev/null

# Apply WhiteSur Firefox theme patch
cd ~/WhiteSur-gtk-theme
./tweaks.sh -f -c light
```

### Electron Apps (VS Code, Discord, Spotify, Slack)
1. Open application Settings (`Ctrl + ,`).
2. Search for **Title Bar Style**.
3. Change setting from `custom` to **`native`**.
4. Restart the app.

---

## 9. Hardware Acceleration & Performance Optimization

To prevent animation micro-stutters or UI lagging caused by visual theme effects, force GPU rendering for GTK apps:

```bash
# Force NGL renderer for GTK4
echo "export GSK_RENDERER=ngl" >> ~/.bashrc
source ~/.bashrc

# Clear GTK render caches
rm -rf ~/.cache/fontconfig ~/.cache/mesa_shader_cache

# Restart Nautilus to load all clean theme configs
nautilus -q
nautilus &
```