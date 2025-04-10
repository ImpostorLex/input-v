---
{"dg-publish":true,"permalink":"/cards/linux/display-servers-in-linux/","tags":["sunday"]}
---

[[Parent\|Parent]]
### Introduction 
---
The responsibilities of display servers on Linux is to handle inputs from users and interacting with Graphical User Interface (GUI), Xorg is the default display manager for most Linux distributions however it comes with performance and security risks that is where Wayland comes in.
## History

Xorg was first developed in 1984 at MIT as a cross-platform display server for different Unix operating system while Wayland started in 2008 with the goal of addressing issues found on Xorg.
## Pre-requisite

- **Window Managers** – Optional components that control window placement/decorations and handle user interactions. Popular ones include Mutter, KWin, Openbox, i3, and many more.
- **X-Protocol** - Defines how communication works with **X-Server** (that manages the display and input devices) and **X-Clients** (Applications that makes request to the server to display graphical elements and receives user input)
- **Wayland compositor** - a control center for your computer's graphical interface:
	- Controls how windows are displayed on your screen. It handles visual effects, such as moving or resizing windows.
	- Manage inputs from mouse, keyboards, and other input devices that interacts with those windows.
- **Wayland Client Apps** – GUI apps that are specifically designed to output to a Wayland compositor rather than Xorg.
- **Wayland Protocols** - Standard APIs that defines how clientss talks to the wayland compositor and other wayland components.

### Questions and Problems
---
## Conclusion
---

