---
title: "Setting up Fedora (Wayland) on the Framework Laptop"
date: 2022-01-28T14:36:51-05:00
draft: false
tags: ["Linux", "Fedora"]
categories: ["Dev Tools"]
toc: true
featuredImage: "featured.avif"
summary: ""
toc: true
autoCollapseToc: true
---

## Backstory

A while ago, I got introduced to the Framework laptop and ended up purchasing one as my secondary dev machine. You are welcome to check reviews online, but what got me into it was the "open-source" attitude from a hardware perspective and the utter respect to the repair right. I am all for "making the best possible use of things". The fact that Fedora runs great OOB on this machine (see [this review](https://www.youtube.com/watch?v=jl4ik7PifpY)) clears the last bit of concern. Long story short, I really like this machine!

I wrote [a blog post](/get-up-and-running-with-fedora35/) about how to get up and running with Fedora on a desktop and while the majority of that post remains applicable, there are some extra configs and settings that require some attention when it comes to laptop and Wayland.

## Wayland

The Framework laptop doesn't have a Nvidia GPU so I went for Wayland, which is also the default of Fedora running Gnome 41. (It's also said the latest Nvidia driver supports Wayland now, but I haven't tested it yet.) Wayland is super nice and smooth and all common apps and built-in gestures just work. The problems show themselves when you add "customization" to the equation.

### Custom Gestures

The default Wayland gestures all utilizes three fingers. Three-finger swiping down to enter application overview, three-finger swiping right/left to go to next/previous workspace. However, as someone who also learned my gestures on Windows machines, I would really prefer the something that's close to the following:

| gesture              | action                         |
| -------------------- | ------------------------------ |
| 3-finger swipe up    | application overview           |
| 3-finger swipe down  | hide current window/nothing    |
| 3-finger swipe left  | backward(`alt+Left`)           |
| 3-finger swipe right | forward/(`alt+Right`)          |
| 4-finger swipe left  | move to workspace to the left  |
| 4-finger swipe right | move to workspace to the right |

Unfortunately, there is currently no official way to customize touchpad gestures. However, [Harshad](https://github.com/harshadgavali) is kind and capable enough to develope a Gnome extension [Gesture Improvements](https://extensions.gnome.org/extension/4245/gesture-improvements/) that offers some level of customization. Here is my configuration:

{{< image src="gesture-customization.png" caption="Gesture Improvements Customization" width="70%" >}}

As nice as it is, this extension cannot map gestures to keyboard shortcuts, which means you still need [libinput-gestures](https://github.com/bulletmark/libinput-gestures). You can find the installation and config guide in the Github repo. However, because we are not running X server, xdotool will not work. You need another tool called [ydotool](https://github.com/ReimuNotMoe/ydotool). To use libinput-gestures with ydotool, you need to configure it to invoke ydotool commands in your `~/.config/libinput-gestures.conf`. You can follow [this guide](https://www.reddit.com/r/gnome/comments/qrhu0e/guide_to_customize_gnome_40_touchpad_gestures_on/) here on Reddit but the basic steps are:

- Install dependencies: `sudo dnf install cmake gcc-c++ scdoc`
- Install ydotool:

  ```shell
  git clone https://github.com/ReimuNotMoe/ydotool
  cd ydotool
  mkdir build
  cd build
  cmake ..
  make -j `nproc`
  ```

- Copy compiled binaries to one of the locations in `$PATH`: `cp -t /usr/local/bin ~/ydotool/build/ydotool ~/ydotool/build/ydotoold`
- Configure ydotool service:

  ```shell
  cp ~/ydotool/build/ydotool.service /usr/lib/systemd/system/ydotool.service
  sudo systemctl daemon-reload
  sudo systemctl enable ydotool.service
  sudo systemctl start ydotool.service
  ```

- Disable password requirement when executing `sudo ydotool`
  - Edit `/etc/sudoers`: `sudo visudo`
  - Add this to the end: <your_username> ALL=(ALL) NOPASSWD: /usr/local/bin/ydotool
  - Check if you messed up while editing `/etc/sudoers`: `sudo visudo -cs`. If there is a error here, revert the changes or you might break your superuser rights.
- Check if everything is working: `sudo /usr/local/bin/ydotool key alt+tab`
- Update `~/.config/libinput-gestures.conf`:

  ```shell
  gesture swipe left 3    sudo ydotool key alt+Left
  gesture swipe right 3    sudo ydotool key alt+Right
  ```

- Restart libinput-gestures: `libinput-gestures-setup restart`. You should be good to go!

### X Cursor Theme

My personal favorite cursor theme is [Qogir](https://github.com/vinceliuice/Qogir-icon-theme) but when I installed it like before, it surprisingly didn't work. Well, at least it worked on the desktop and other UI element but as soon as I hovered over native Wayland apps (not the ones running XWayland) like Firefox or Gnome Settings, a tiny little fall back cursor would show up. Fortunately, there's actually a quick fix (from [Arch Wiki](https://wiki.archlinux.org/title/Cursor_themes#GNOME)):

> By default, on Wayland, Gnome applications should be unable to display your cursor themes located in `~/.local/share/icons`. As a workaround, you can add that path to `XCURSOR_PATH` (via `export XCURSOR_PATH=${XCURSOR_PATH}:~/.local/share/icons`)

However, depending on your shell setup, it can be _**very tricky**_ to determine where this line should live. If you are like me and use **zsh** as the login shell, you should create a new profile file at `~/.zprofile` and add the line `export XCURSOR_PATH=${XCURSOR_PATH}:~/.local/share/icons`. Restart your machine and the cursor theme should be working.

I don't know why adding it to `~/.profile` is not working. Shouldn't `~/.profile` and the shell profile file both get executed on startup? If you know the reason and mechanism behind this I am all ears!

## Conclusion

I really like the Framework laptop and I am happy to report that Fedora runs super well on it. Gnome 41 with Wayland feels fast and responsive. You even get a GUI to set fingerprints and you can use finger print to authorize sudo commands! How cool is that! :heart_eyes:

What's most surprising is how smooth and fluid the touchpad gestures and corresponding animation felt. I know it may sound a bit crazy, but it's getting pretty close to what a MacOS + MacBook touchpad feels like. :scream\*cat: I can safely say it's better than my XPS 15 7590 (i7 4k model) with Windows 11. I **\*do**\_ hope touchpad gesture customization will land soon and X cursor themes can just work as before.

Hope you find this post helpful and hope to see you in the next one! :wave:
