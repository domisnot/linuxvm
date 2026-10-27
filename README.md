GitHub Codespaces Linux Desktop Setup & Troubleshooting Guide
This guide covers common issues, fix commands, and browser installation steps when running a Linux desktop environment (noVNC + XFCE) inside GitHub Codespaces.

1. Fixing White Screen / Missing Desktop Panels
If the noVNC window displays a blank white screen, missing taskbar, or no wallpaper, XFCE desktop components need to be manually installed or initialized.

Install XFCE Desktop
Bash
sudo apt update
sudo apt install -y xfce4 xfce4-goodies
Start Desktop Components Manually
Run these commands in your VS Code terminal to load the taskbar, desktop background, and window controls onto Display :1:

Bash
DISPLAY=:1 xfce4-panel &
DISPLAY=:1 xfdesktop &
DISPLAY=:1 xfwm4 &
2. Browser Setup (Bypassing Ubuntu Snap Errors)
Standard Ubuntu commands like sudo apt install firefox or sudo apt install chromium-browser fail in Docker containers because they rely on snapd, which is incompatible with containerized environments.

Install Google Chrome (Native .deb)
Bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo apt update
sudo apt install -y ./google-chrome-stable_current_amd64.deb
Launch Google Chrome
Chromium-based browsers require the --no-sandbox flag to run inside Docker/Codespaces:

Bash
DISPLAY=:1 google-chrome-stable --no-sandbox &
Fix Desktop Menu Shortcuts for Chrome
To make Chrome launch directly from the XFCE Applications Menu without crashing:

Bash
sudo sed -i 's|Exec=/usr/bin/google-chrome-stable %U|Exec=/usr/bin/google-chrome-stable --no-sandbox %U|g' /usr/share/applications/google-chrome.desktop
sudo update-alternatives --set x-www-browser /usr/bin/google-chrome-stable
3. Fixing Git Push & LFS Errors
If running git push fails with the error 'git-lfs' was not found on your path:

Option A: Install Git LFS (Recommended)
Bash
sudo apt update && sudo apt install -y git-lfs
git lfs install
git push origin main
Option B: Remove the Pre-Push Hook
If Git LFS is not actively needed for large files:

Bash
rm -f .git/hooks/pre-push
git push origin main
4. Display & Scaling Fixes (noVNC)
If menu options are cut off or screen resolution looks misaligned:

Open the noVNC browser tab (Port 6080).

Click the Gear Icon (Settings) on the dark panel on the left edge.

Under Scaling Mode, change the option to Remote Resizing or Local Scaling.

Refresh the page (Ctrl + R).

5. Automated Setup via devcontainer.json
To ensure packages install automatically whenever a new Codespace is created, add a postCreateCommand block to .devcontainer/devcontainer.json:

JSON
{
  "name": "Linux Desktop",
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "features": {
    "ghcr.io/devcontainers/features/desktop-lite:1": {
      "desktopUI": "xfce"
    }
  },
  "forwardPorts": [6080],
  "portsAttributes": {
    "6080": {
      "label": "Desktop Lite (noVNC)"
    }
  },
  "postCreateCommand": "sudo apt update && sudo apt install -y xfce4 xfce4-goodies git-lfs"
}
