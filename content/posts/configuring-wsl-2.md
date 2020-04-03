---
title: "Setting Up WSL 2 for Web Development"
date: 2020-04-02T23:35:31-04:00
draft: true
---

Recently, WSL 2 landed on the slow ring of Windows Preview. As a fan of WSL myself, I am eager to try it out and enjoy the improved I/O performance it brings to the table. So without further delay, I opted in and upgraded Win10 2004 and enabled WSL 2. HOWEVER (there is always a "however"), not all things are smoothed out yet at the time of writing. In this post, I am going to write about my experience of setting up WSL 2 for a Django web development workflow.

## zsh

My `.zshrc`:

```shell
# enabling some built-in features
zstyle ':completion:*' menu select
autoload -U compinit && compinit

# personal aliases
alias mooc="cd /home/xzhan/Development/MOOC"
alias proj="cd /home/xzhan/Development/Projects"
alias pk="cd /home/xzhan/Development/Packages"
alias wk="cd /home/xzhan/Development/Work"
alias da="deactivate"
alias dcu="docker-compose up"
alias dcud="docker-compose up -d"
alias dex="docker-compose exec"
alias dcd="docker-compose down"
alias dcs="docker-compose stop"
alias dcl="docker-compose logs"
alias vim="nvim"
alias ls="lsd"
alias ll="lsd -l"

# added for vscode
alias code="/mnt/c/Users/zhanx/AppData/Local/Programs/'Microsoft VS Code'/bin/code"

# added for miniconda3
export PATH="/home/xzhan/miniconda3/bin:$PATH"

# added for pipenv
export PIPENV_VENV_IN_PROJECT=True
eval "$(pipenv --completion)"

# added for rust
export PATH="/home/xzhan/.cargo/bin:$PATH"

# added for homebrew
export PATH="/home/linuxbrew/.linuxbrew/bin:$PATH"

# added for nvm
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

source /home/linuxbrew/.linuxbrew/share/zsh-history-substring-search/zsh-history-substring-search.zsh
bindkey "$terminfo[kcuu1]" history-substring-search-up
bindkey "$terminfo[kcud1]" history-substring-search-down

eval "$(starship init zsh)"
```

A few things to pay attention here:

### WSL 2 I/O performance catch

WSL 2 has a much improved I/O performance, but only on the Linux partition. As described in [this documentation](https://docs.microsoft.com/en-us/windows/wsl/wsl2-ux-changes#cross-os-file-speed-will-be-slower-in-initial-preview-builds) WSL 2 has a much slower performance when performing cross OS I/O tasks. Outcome include:

- **Unacceptable slow performance with zsh shell prompt/plugins**, like [oh-my-zsh](https://ohmyz.sh/) or [starship](https://starship.rs/). I don't know how the internal implementation of these projects to identify the direct reason, but the outcome is printing a new line takes over 30s:

  ![zsh with starship in Windows User Directory](/zsh_with_starship.gif)

- **Slow start up performance.** Sometimes the WSL prompt can take over 15s to start. One way to mitigate this is to exclude the inclusion of Windows PATH, with which I can reduce the start up time down to 1s. This will be elaborated in the next point.

### Exclude Windows PATH

As mentioned, excluding Windows PATH can have some nice performance boost. If you are not relying Windows programs anyways, you can safely exclude it by adding this to `/etc/wsl.conf`:

```toml
[Interop]
appendWindowsPath = False
```

"But what about VS Code!?" I hear you my friend. Notice that we have a special alias on line 22 of the `.zshrc` file:

```shell
alias code="/mnt/c/Users/zhanx/AppData/Local/Programs/'Microsoft VS Code'/bin/code"
```

With this alias we can now use the `code` command inside WSL 2 as we would in any local shell terminal.

### zsh history search and starship

With a ton of experiments, some of which can be seen in [this GitHub issue](https://github.com/starship/starship/issues/1038), oh-my-zsh was proven to be slower in WSL 2. I used to be a big fan as it really helps configure a nice and usable zsh out of the box. However, it includes too many things under the hood which I don't really need. Some of the features I would really like to keep are:

- Menu-like options when hitting Tab, which is built-in and can be enabled like shown at the beginning of my `.zshrc`
- A nice-looking theme. I really like the [spaceship theme](https://github.com/denysdovhan/spaceship-prompt)
- History search with up arrow key

After searching for a while, I landed on [starship](https://starship.rs/), a Rust-powered and spaceship-inspired cross-shell prompt which also works with Powershell (Yay!！ ( •̀ ω •́ )y), and this nice [zsh-history-substring-search](https://github.com/zsh-users/zsh-history-substring-search) plugin. I installed th latter via [homebrew](https://docs.brew.sh/Homebrew-on-Linux). You can configure them just like I did at the end of the `.zshrc` file.

### typeface choice & nerd font

As you would expect from a spaceship theme user, I like icons and emojis. However, not all emojis look nice in every terminal on every OS. On the other hand, nerd font icons are much are consistent and renders nicer in VS Code's built-in terminal. You can find [a preset config from starship](https://starship.rs/presets/).