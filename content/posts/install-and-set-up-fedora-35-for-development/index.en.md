---
title: "Get Up and Running With Fedora 35"
subtitle: "A somewhat developer-oriented guide"
date: 2021-11-27T23:11:57-05:00
draft: false
tags: ["Linux", "Fedora"]
categories: ["Dev Tools"]
toc: true
featuredImage: "featured.svg"
featuredImagePreview: "featured-preview.png"
summary: ""
---

Recently, I find myself quite a bit distracted when using Windows for work, with all social media and entertainment apps readily available, so I want a more focused working environment. I have not used Linux since I decided to do a fresh install of Fedora 34 when it first came out and failed. The live USB image behaved quite strangely that time, greeting me with only a blank desktop with no cursor and no installation prompt. As it turned out, the live image worked perfectly fine, but because my PC was connected to my TV (which was off btw) it treated the TV as the primary screen, so my cursor and the installation prompt were not showing on my monitor.

Anyways, enough of my stupid miss. Let's dive into my setup and some of the pitfalls I've encountered in the process, so you will have a better idea when it comes to your Linux setup.

## Installation (Dual-Boot with Windows)

The installation of Fedora itself has been pretty straight forward. I simply downloaded Fedora Media Writer from [getfedora.org](https://getfedora.org/en/workstation/download/) and let it do its job. The tricky part comes after the installation as I am dual-booting Fedora alongside with Windows 10.

First, remember to disable Secure Boot and Fast Boot in you motherboard's BIOS, especially if you are planning to use an Nvidia card. That's basically common sense in Linux community, so bear that in mind. You can find more information about it on [rpmfusion](https://rpmfusion.org/Howto/NVIDIA?highlight=%28%5CbCategoryHowto%5Cb%29) if you are going with Fedora.

The second point is a bit more hardware-specific. Usually, people have only one large disk and install both Windows and Fedora on it. However, I want to keep the two clean and separated, so I have a **dedicated** SSD for my Linux installation. When you install Fedora on a drive with a preexisting Windows installation, Anaconda, the Fedora installer, is usually intelligent enough and will pick up Windows in most cases. Hence, when you boot into Fedora and get to the grub2 page, there will be a Windows option for you to boot into Windows.

<!-- ![Fedora GRUB with Windows](https://www.tecmint.com/wp-content/uploads/2017/11/Fedora-Windows-Dual-boot-Menu.png "Fedora Grub2 with Windows") -->

{{< image src="https://www.tecmint.com/wp-content/uploads/2017/11/Fedora-Windows-Dual-boot-Menu.png" caption="Fedora Grub2 with Windows" width="70%" >}}

However, when you install Fedora on a different drive, the story is a bit different, as in a UEFI system, every new OS installation creates a new boot entry, and previous ones will get wiped. I am not sure if this will happen on every motherboard, but it does on mine (Asus Prime X370-Pro for AMD Ryzen). And because Anaconda will not probe OS installed on other disks, it will not add Windows to GRUB.

So what should we do? We need to repair the Windows EFI boot loader. I won't go into details as there are many other tutorials on this topic, and [here](https://recoverit.wondershare.com/partition-tips/repair-efi-bootloader-in-windows.html) is the one I followed. As a side note, I fixed my boot loader with method 2 (repair commands) **followed by** method 1 (automatic boot repair).

## System Config Tips

With Fedora 35 comes the latest Gnome 41, which is quite a bit different from what Gnome 3.38 offered. More info can be found on Gnome's website and blogs like [this one](https://blogs.gnome.org/shell-dev/tag/uxd-gnome-40/). The new design is great and all, but with the new UI design comes a few quirks.

### Keyboard Shortcuts

As you have probably noticed, Gnome 40 comes with a horizontal workspace layout. Personally, I find it easier switch between Windows and Linux now as workspace/desktop navigation can now use the same set of keyboard shortcuts, and that is pretty nice. The **problem** is that with a horizontal workspace in, shortcuts for vertical workspace navigation are removed from the settings.

{{< image src="gnome-settings-workspace-shortcuts.png" caption="Horizontal Workspace Shortcuts Gone" width="60%" >}}

However, many of these default shortcuts interferes with application ones, especially dev tools like VSCode. For example, VS Code uses `Ctrl+Alt+Up/Down` to add cursor one line up/down. However, this will not work by default because this combo is occupied by the Gnome's `switch-to-workspace-up/down` command. You will have to install [dconf Editor](https://wiki.gnome.org/Apps/DconfEditor) to browse and configure all such hidden shortcuts.

{{< image src="dconf-editor-workspace-shortcuts.png" caption="dconf Editor - pay attention to `switch-to-workspace-up/down`" width="65%" >}}

I [posted about this on Gnome's Discourse](https://discourse.gnome.org/t/some-keyboard-shortcuts-are-occupied-by-default-but-the-options-are-not-shown-in-settings-app/8199) but got no reply at the time of writing. There's also [a similar issue](https://github.com/pop-os/pop/issues/409#issuecomment-493598873) brought up and discussed in PopOS's GitHub repo.

For me personally, not disabling such deprecated shortcuts by default **and also** removing them from the Settings app is a very irresponsible move. I hope Gnome devs will come to realize how frustrating this is.

### Emoji Input

Continuing the complaint of shortcuts, `Ctrl+.` is configured as the default hotkey for toggling emoji input and if you are a VS Code user, you know how annoying this is. There are two ways you can change it:

- Install `ibus-setup` by `sudo dnf install ibus-setup`, launch it from the terminal and configure it in the `Emoji` tab.
- Install `dconf Editor` and configure `/desktop/ibus/panel/emoji/hotkey`

I set it to be the same as Windows, which is `Super+;`(`['<Super>semicolon']` in dconf Editor).

I do want to elaborate on how to use this feature as a) I only became aware of its existence today and b) it's a bit confusing and it took me quite a while to figure out.

To use it, you enter emoji typing mode by the hotkey and you shall see an underlined `e` in you textbox. Go ahead and type to search whatever emoji you want to input, hit `Space` once to select the default option, hit `Space` again to open the pop-up option menu, and finally use arrow keys and `Enter` key to select the emoji you want.

{{< image src="input-emoji.gif" caption="Input Emoji" width="75%" >}}

Obviously, there's a small problem here: The pop-up menu is not next to the input cursor. This bug seems to be [reported and tracked here](https://gitlab.gnome.org/GNOME/gtk/-/issues/4503), and closed two weeks ago at the time of writing. Hope the fix gets released soon! :tada: There are also some emojis that are not included in the current emoji typeface and get rendered as tofus. Honestly, I have no idea what they are. :man_shrugging: Leave a comment if you know how to fix it!


### SSH

As per [this reddit post](https://www.reddit.com/r/Fedora/comments/jhxbdh/no_ssh_public_key_auth_after_upgrade_to_fedora_33/), the new default crypto policy since Fedora 33 has changed and for online services that hasn't updated their standard, you need to create a whitelist for them in `~/.ssh/config`:

```
Host git.imooc.com
  PubkeyAcceptedKeyTypes=ssh-rsa
```

### Nvidia GPU

You can install Nvidia driver from the [RPM Fusion](https://rpmfusion.org/Howto/NVIDIA) repo.

{{< admonition tip "RPM Fusion is your best friend" >}}
You should always enable [RPM Fusion](https://rpmfusion.org/) after installing Fedora. 
{{< /admonition >}}

If you are running Fedora on a laptop, Optimus should work right out of the box. However, if you don't care much about battery life but want to make the most use of your dedicated Nvidia GPU all the time (Gnome's UI will also be noticeably smoother), you can follow [this guide](https://docs.fedoraproject.org/en-US/quick-docs/how-to-set-nvidia-as-primary-gpu-on-optimus-based-laptops/) from Fedora's doc and set it as your primary GPU.

## Apps
Apps on Linux has always been an interesting area to me, and I am happy to say that it has improved quite a LOT over the years. Here I will not post a lengthy list of apps that I use (that would probably be a post of its own), but instead I will talk about some issues and workarounds for a few app categories.

### Electron Apps (Mailspring, Etcher, etc.)

{{< admonition tip "It's Nvidia AGAIN! (sort of)" >}}
Just remember to add the goddamned **`--disable-gpu-sandbox`**
{{< /admonition >}}

This one is particularly annoying as it affects a dozen apps that I use frequently. :imp: IIRC, it seems to be plaguing all apps that are not using the latest version of Election, both on Wayland and Xorg. Because I have been running solely on a discrete GTX1080 and switched to Xorg from Day 1, I can't really say for Wayland. But if you are running Xorg with Nvidia driver, add  `--disable-gpu-sandbox` and hopefully, it will solve your `The display compositor is frequently crashing. Goodbye.` and mysterious `core dumped` errors too.

{{< admonition tip "Some more discussion on this issue" >}}
- [My Post on Mailspring Discourse](https://community.getmailspring.com/t/unable-to-launch-the-app-on-fedora-35/3424)
- [Etcher Github Issue](https://github.com/balena-io/etcher/issues/3639)
{{< /admonition >}}

To apply this fix to launcher/desktop files, simply locate the corresponding `.desktop` file and add it to the `Exec` line (Mailspring in this example). Usually, such files either live in `/usr/share/applications/` or `~/.local/share/applications/`:

```shell
> sudo nvim /usr/share/applications/Mailspring.desktop

# in vim or other editors
Exec=mailspring --disable-gpu-sandbox %U # add --disable-gpu-sandbox here as such
```

### `.deb`-only Apps (e.g. Lunacy)

Follow [this tutorial](https://fedingo.com/how-to-convert-deb-to-rpm-files-in-linux/) to convert a deb installer to rpm format. The `alien` package can be installed by  `sudo dnf install alien`. In the case of Lunacy, simply download the deb file and run `sudo alien -r -c Lunacy.deb`. Works beautifully for me! :raised_hands: You could turn to the snap package of course, but the slow startup time is really a bad experience. I wish they will add flatpak support in the future.

### Avalonia Apps (e.g. Lunacy)

Your Avalonia apps might not scale correctly and the UI font might be tiny if you are using a HiDPI display and running X11 like me. In this case, you can add `export AVALONIA_SCREEN_SCALE_FACTORS="DP-0=2.0"` to your `~/.profile` file.  More details can be found [here in Avalonia's GitHub Wiki](https://github.com/AvaloniaUI/Avalonia/wiki/Configuring-X11-per-monitor-DPI).

### Windows Apps with CrossOver (WeChat)

I use WeChat on a daily basis and Tencent, just like many other Chinese tech giants, don't really give a sh*t about Linux desktop. Even recently after they paid more attention, the attitude is indifferent at best. :shrug: 

Anyhow, there has long been a way to run Windows apps on Linux and that's [Wine](https://www.winehq.org/). I came across this commercialized version of it called CrossOver last year, and it has been working pretty well for me.    

{{< admonition warning "Disclaimer" false >}}
CrossOver is from the the biggest contributor of Wine, CodeWeavers. CodeWeavers (together with the community I believe) maintains a list of popular Windows-only apps and pre-configures them for a one-click installation so you, as a user, don't have to worry about configuring your own Wine and "bottles", which can be time-consuming, intimidating and frustrating :cry:.

The author, which is myself, is **not** affiliated with CodeWeavers, and **nor** is he getting paid by promoting CrossOver. I am just a normal customer who finds CrossOver worthy of some good words as a useful product.
{{< /admonition >}}

The gist of the story is that WeChat runs quite well on CrossOver right out of the box. However, before I got that far some extra steps were required for the successful installation of CrossOver.
sud

### Firefox Video Playback

You might find Firefox super slow when playing videos, especially HD videos on YouTube in full screen. Enable RPM Fusion and install ffmpeg. It will probably solve your issue. If you are running an Optimus device, you might also want to try turning on `layers.acceleration.force-enabled`. 

## Summary

Fedora has always been my go-to when it comes to Linux distro. Latest packages that really suits my development use case, a vanilla Gnome experience that also serves as a great foundation for personalized tweaking and customizing, fast and stable day-to-day performance, etc. There are a lot of good things about it. However, there can be hiccups every now and then, partly being a Linux distro and partly being Fedora. Hope this guide can save you some time and headaches in your Fedora journey. In my next blog, I will spend some time on the application side of things and talk about the apps I use on a daily basis. See you there! :wave: 