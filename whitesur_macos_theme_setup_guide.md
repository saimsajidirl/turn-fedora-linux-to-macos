# WhiteSur macOS Theme Setup Guide for Fedora / GNOME

This guide provides the exact sequence of commands that successfully themed Fedora GNOME with the macOS WhiteSur aesthetic, enabled light mode, fixed LibAdwaita/GTK4 compatibility issues, and restored colored "traffic light" window controls across applications.

---

## 1. Install System Dependencies & Extensions

Install the necessary dependencies, flatpak theme converters, and the GNOME Shell User Theme extension:

```bash
# Install user-theme extension package
sudo dnf install gnome-shell-extension-user-theme

# Install flatpak theme conversion dependencies
sudo dnf install -y ostree libappstream-glib
```

---

## 2. Configure macOS Window Controls (Traffic Lights)

Move window control buttons to the top-left corner (macOS standard):

```bash
gsettings set org.gnome.desktop.wm.preferences button-layout 'close,minimize,maximize:'
```

---

## 3. Set Global Light Theme Preferences

Apply WhiteSur Light across GTK, Icons, Shell, and system preferences:

```bash
# Set GTK and Icon themes to WhiteSur Light
gsettings set org.gnome.desktop.interface gtk-theme 'WhiteSur-Light'
gsettings set org.gnome.desktop.interface icon-theme 'WhiteSur-light'

# Set Shell theme to WhiteSur Light
gsettings set org.gnome.shell.extensions.user-theme name 'WhiteSur-Light'

# Enable GNOME User Theme extension
gnome-extensions enable user-theme@gnome-shell-extensions.gcampax.github.com

# Set system preference to light mode
gsettings set org.gnome.desktop.interface color-scheme 'prefer-light'
```

---

## 4. Fix GTK4 / LibAdwaita Theme Loading (Traffic Light Fix)

Modern GTK4 / LibAdwaita apps (like Files/Nautilus) require direct CSS linking to avoid `color-mix()` syntax parsing errors in newer GTK releases.

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

## 5. Enable Themes for Flatpak Applications

Allow Flatpak applications in sandboxed environments to read local theme and GTK4 configuration files:

```bash
flatpak override --user --filesystem=~/.config/gtk-4.0:ro
flatpak override --user --filesystem=~/.themes:ro
flatpak override --user --filesystem=~/.local/share/icons:ro
```

---

## 6. Restart Desktop Services and Apply Changes

Flush theme/font caches and restart Nautilus to verify:

```bash
# Clear render caches
rm -rf ~/.cache/fontconfig ~/.cache/mesa_shader_cache

# Restart Nautilus
nautilus -q
nautilus &
```

---

## 7. App-Specific Adjustments

* **Web Browsers (Chrome / Brave):** Right-click the top bar and enable **"Use system title bar and borders"**.
* **Firefox:** Ensure Firefox is closed, then run:
  ```bash
  cd ~/WhiteSur-gtk-theme
  ./tweaks.sh -f -c light
  ```
* **Electron Apps (VS Code, Discord, Spotify):** Go to Settings (`Ctrl + ,`), search for **Title Bar Style**, and set it to **`native`**.
