# Flow Blue - Quick Start Guide

## First Boot

After rebasing to Flow Blue, you'll boot into a greetd login screen.

### Step 1: Install Nix + Home Manager

Run this single command:
```bash
ujust setup-nix-complete
```

This installs:
- Nix package manager (via nix-toolbox)
- Home Manager for dotfile management

**Or do it step-by-step:**
```bash
ujust install-nix
ujust setup-home-manager
```

### Step 2: Configure Home Manager

Edit your Home Manager configuration:
```bash
home-manager edit
```

**Example minimal config for River WM:**
```nix
{ config, pkgs, ... }:

{
  # Let Home Manager manage itself
  programs.home-manager.enable = true;
  
  # Home Manager needs this
  home.username = "yourusername";
  home.homeDirectory = "/home/yourusername";
  home.stateVersion = "24.05";
  
  # Install River and essential tools
  home.packages = with pkgs; [
    # Compositor
    river
    
    # Essential Wayland tools
    waybar      # Status bar
    fuzzel      # App launcher
    foot        # Terminal
    
    # Utilities
    grim        # Screenshots
    slurp       # Region selector
    wl-clipboard # Clipboard
    
    # Apps
    firefox
    # Add more packages here
  ];
  
  # River configuration
  home.file.".config/river/init" = {
    text = ''
      #!/bin/sh
      
      # Super+Return: Launch terminal
      riverctl map normal Super Return spawn ${pkgs.foot}/bin/foot
      
      # Super+D: Launch app launcher
      riverctl map normal Super D spawn ${pkgs.fuzzel}/bin/fuzzel
      
      # Super+Q: Close window
      riverctl map normal Super Q close
      
      # Super+Shift+E: Exit River
      riverctl map normal Super+Shift E exit
      
      # Add more River config here...
    '';
    executable = true;
  };
  
  # Session file for greetd
  home.file.".local/share/wayland-sessions/river.desktop".text = ''
    [Desktop Entry]
    Name=River
    Comment=Dynamic Wayland compositor
    Exec=${pkgs.river}/bin/river
    Type=Application
  '';
}
```

### Step 3: Apply Configuration

```bash
home-manager switch
```

This installs all packages and sets up your dotfiles.

### Step 4: Update greetd to Launch River

Edit `/etc/greetd/config.toml` (requires sudo):
```toml
[initial_session]
command = "/home/yourusername/.nix-profile/bin/river"
user = "yourusername"
```

Or keep tuigreet and select River from the session menu.

### Step 5: Reboot

```bash
systemctl reboot
```

You'll boot directly into River!

## Daily Workflow

### Update Packages
```bash
# Update Nix channels
nix-channel --update

# Update Home Manager config
home-manager switch
```

### Add New Packages
Edit `~/.config/home-manager/home.nix` and add to `home.packages`:
```nix
home.packages = with pkgs; [
  river
  waybar
  # Add new packages here:
  neovim
  htop
];
```

Apply changes:
```bash
home-manager switch
```

### Switch Compositors

Want to try Hyprland instead? Just change your config:
```nix
home.packages = with pkgs; [
  hyprland  # Changed from river
  waybar
  # ...
];
```

Update session file and apply:
```bash
home-manager switch
```

## Useful Commands

```bash
# Check Nix status
ujust nix-info

# Enter Nix environment (if needed)
nix-toolbox

# Search for packages
nix search nixpkgs firefox

# Install package temporarily
nix-shell -p neovim

# Rollback Home Manager
home-manager generations
home-manager switch --rollback
```

## Troubleshooting

### River won't start
```bash
# Check seatd is running
systemctl status seatd

# Check logs
journalctl -b -u greetd
```

### Audio not working
```bash
# Check PipeWire
systemctl --user status pipewire

# Test audio
pactl list sinks
```

### Need to rebuild system
```bash
# Update Flow Blue base image
rpm-ostree upgrade

# Reboot
systemctl reboot
```

## File Locations

- **Home Manager config:** `~/.config/home-manager/home.nix`
- **River config:** `~/.config/river/init`
- **Nix packages:** `~/.nix-profile/`
- **System config:** `/etc/greetd/config.toml`

## Philosophy

**Flow Blue provides:**
- Minimal, stable system base
- Display manager (greetd)
- Essential system services

**You provide (via Nix/Home Manager):**
- Compositor (River, Hyprland, etc.)
- Applications
- Configuration
- Everything user-facing

This separation gives you:
- 🔒 Stable, atomic system updates
- 🚀 Fast, flexible userspace updates
- 📝 Declarative, reproducible config
- 🔄 Easy rollbacks at both levels

Enjoy your minimal Wayland system! 🌊
