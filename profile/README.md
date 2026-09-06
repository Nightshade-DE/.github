# Nightshade Desktop Environment (NSDE)

NSDE is a lightweight Wayland desktop environment. It aims to approach the desktop experience creatively while focusing on the essential features you actually enjoy using during your daily work. Let's see if that works out! ^^

`Python 3` | `Wayland` | `Morph` | `wxPython` | `Modular Architecture`

---

The Nightshade Desktop Environment evolved from [Fvwm-Nightshade](https://github.com/Fvwm-Nightshade/Fvwm-Nightshade). Unfortunately, that desktop environment is no longer viable for modern use because its configuration GUI tools relied on the long-obsolete GTK 2. Furthermore, it is still based on FVWM 2.6, even though FVWM3 has already reached version 1.1.4. On top of that, FVWM utilizes X11, which is currently being phased out by many distributions in favor of Wayland.

Because of this, the underlying foundation has been migrated from X11 to Wayland. Additionally, NSDE relies entirely on Python. A major benefit: the environment can easily be installed and uninstalled anywhere. 

It is completely decoupled from the default compositor configuration. This means that if NSDE is uninstalled, the used compositor and its previous setup remain completely untouched.

# Roadmap
Right now, the roadmap looks a bit like a jigsaw puzzle, but the project is still young – the idea was born in September 2025 and has been taking shape piece by piece ever since.

<!-- Placeholder for a screenshot or architecture diagram when available -->
<!-- ![NSDE Architecture Overview](path/to/architecture_diagram.png) -->

## Wayland Compositor in Use

NSDE will use [Morph](https://github.com/Nightshade-DE/Morph) as its main window manager.

Morph is a stacking, tiling, and scrolling hybrid compositor built with wlroots. The compositor is in a working alpha state - not all is available but you can use it... more or less :wink:

## Central Control Daemon

For a desktop to function smoothly, it requires a control unit to coordinate various actions, such as:

- Automounting USB devices and displaying them on the desktop.
- Sending, receiving, and forwarding notifications to a notification tool.
- Controlling the compositor (closing windows, switching workspaces, triggering reconfigurations).
- Providing a global clipboard history that can be accessed from anywhere.
- Menu monitoring: When a new application is installed, a trigger is sent to start the update via an integrated menu tool.
- Automatic configuration adjustments to correctly map NSDE settings into the compositor configuration.
- Internal routing between different components to enable seamless communication.

The daemon [nsd](https://github.com/Nightshade-DE/nsd) already exists to handle these core functions.

## Wayland Protocol Tool

Unlike in X11, information like window positions or the number of workspaces cannot easily be requested from a window manager in the Wayland world, as everything is sandboxed for security reasons. However, since Morph understands standard Wayland protocols and `wlr-protocols`, this information can be extracted and actions can be triggered through these protocols.

To achieve this, [wayctl](https://github.com/Nightshade-DE/wayctl) was developed. This tool not only retrieves information but also executes direct actions. Here is a feature snippet:

- **Structured Queries:** Fetch toplevel windows, workspaces, and outputs.
- **Direct Window Control:** Focus, close, fullscreen, maximize, or minimize windows.
- **Workspace Control:** Switch and assign workspaces (depending on compositor capabilities).
- **Action/Fallback Layer:** Set/send `trigger_action` and `send_key`.
- **Global Logging:** `--verbose [LEVEL]` and `--log [PATH]` with invocation headers and run summaries.
- **PID Heuristic (`-H pid`):** Stack-order-based mapping of Wayland toplevels to process IDs (PIDs).
- **Action Mapping Model:** Logical actions can be mapped both via direct protocol commands (`direct = [...]`) and labwc-native actions (`action = "..."`).

## Graphical Toolkit

For NSDE, GTK 3 GUI tools are typically created. To speed up their development, [SimpleWx](https://github.com/ThomasFunk/SimpleWx) is used – a wxPython RAD (Rapid Application Development) library. It is built on top of wxWidgets, which in turn uses GTK 3 as its backend.

This makes it possible to quickly build configuration utilities or small helpers and integrate them into NSDE. Using Qt Designer, the GUI can be designed visually and then ported to SimpleWx (the `swx-builder` reads the `.ui` file and generates a Python GUI scaffolding). After that, you only need to implement the actual functionality.

## Monitor Management

All major desktop environments come with their own monitor management: when a new monitor is connected, the configuration tool pops up, saves the user settings, and automatically reactivates them next time. Connection profiles are absolutely essential in a mobile environment – on the couch with just the laptop today, at your private desk with an external monitor tomorrow, and at the office with a dual-screen setup the next day. Plug it in – it just works. That's how it should be. While tools like [kanshi](https://gitlab.freedesktop.org/emersion/kanshi) support profiles, they have to be configured manually via text files.

The search for a flexible display tool with profile support wasn't successful. The only candidate would have been `nwg-displays`, but adapting it to cleanly support e.g. labwc was just too extensive. Therefore, [wlc-displays](https://github.com/ThomasFunk/wlc-displays) was developed based on SimpleWx.

It supports freely configurable monitor profiles by utilizing kanshi in the background, automatically writing the profiles into its config and activating them.

Work is currently underway on the nsd integration: as soon as a monitor is plugged in, nsd should automatically invoke `wlc-displays` if no matching profile exists in kanshi yet.

**Note:** Unfortunately Morph doesn't support tools like kanshi at the moment. But is in planing.

## Desktop Icons

Icons on the desktop are a controversial topic. Some love them, others find them useless or call them a mere gimmick – nice to have, but not necessary.

However, when used wisely, they offer real value – for example, with *Activities*, where you can drop current project folders or create symlinks right in front of you. Or for pinning important applications so you don't have to hunt for them in the menu. Dynamically displaying mounted drives on the desktop is another great use case.

That's why [ld-icons](https://github.com/Nightshade-DE/ld-icons) was created. It is lightweight, written entirely in Python, and can theoretically be used on any wlroots-based Wayland compositor (it already runs stably with labwc). In tandem with nsd, it also handles the visualization of dynamically mounted drives.

## Shell Components

In the Wayland ecosystem (similar to GNOME/KDE), the "shell" refers to the component that manages panels, runners, and the desktop background. To realize custom taskbars, button bars, or panels, `SimpleWxCS` is currently under development. Building on top of it, a taskbar in the style of [sfwbar](https://github.com/LBCrion/sfwbar) and an *Activities* concept – heavily inspired by KDE Activities – are being created.

## Power Management

This is on mobile devices like laptops essential. But most of the available power managers are included in their desktops environments and bring often too much dependencies with them. Or they haven't enough stearing possibilities: for example closing the lid takes the system into sleep but for a critical battery state another tool is needed and needs configuration. But the power manager GUI doesn't show or tell this mostly.

For such reasons, [nspower](https://github.com/Nightshade-DE/nspower) is currently being developed. It is a modular, desktop-agnostic power manager without dependencies on a specific desktop environment. It takes control of battery states and thresholds (warnings and critical actions), handles display idle management, and integrates upower and TLP cleanly. A planned SimpleWx GUI will bring all those settings under one hood.

## Additional Components (3rd-Party Applications)

To complement the desktop environment, the following programs (available through mainstream distribution repositories) are currently planned:

- **Runner / Application Launcher:** A custom SimpleWx application will likely be created here, though the exact implementation is still open.
- **Audio/Volume & Network:** The integration of common applets (like `network-manager-applet`/`nm-applet` or a PipeWire volume control) is firmly planned for the `SimpleWxCS` panel but has not been finalized yet.
- **Screenshots:** [Grim](https://gitlab.freedesktop.org/emersion/grim) combined with [Slurp](https://github.com/emersion/slurp).
- **Notification Daemon:** [Dunst](https://github.com/dunst-project/dunst).
- **Wallpaper:** [Waypaper](https://github.com/anufrievroman/waypaper) together with [swaybg](https://github.com/swaywm/swaybg) for static backgrounds, and [mpvpaper](https://github.com/GhostNaN/mpvpaper) to play videos as wallpapers.
- **Theme Management:** [nwg-look](https://github.com/nwg-piotr/nwg-look) or [ThemeChanger](https://github.com/ALEX11BR/ThemeChanger).
- **Icon Theme:** Currently the [Slot-Symbolic-Dark-Icons](https://github.com/L4ki/Slot-Plasma-Themes) theme combined with the [Dream-Color-Plasma](https://github.com/L4ki/Dream-Plasma-Themes) theme.
- **Configuration:** A dedicated, tree-view-based SimpleWx application is planned to allow full configuration.
- **Policy Kit:** `lxpolkit` (due to its minimal dependencies).
- **Idle Daemon:** [hypridle](https://wiki.hypr.land/Hypr-Ecosystem/hypridle/). It simply brings more configuration options than [swayidle](https://github.com/swaywm/swayidle).
- **Lock Daemon:** [hyprlock](https://wiki.hypr.land/Hypr-Ecosystem/hyprlock/). Offers a ton of customization possibilities.
- **Session Management:** Currently in the conceptual phase.

## Future Ideas

- There's the idea to implement AI into the desktop like a daily helper - showing the weather app or open an activity. More crazy is the idea of a talking AI like in Star Trek :joy: :upside_down_face:


# Looking for Contributors

If you are interested in contributing to a fast, powerful, yet lightweight alternative to the heavyweight desktop environments, feel free to reach out at: nightshade.desktop@gmail.com
