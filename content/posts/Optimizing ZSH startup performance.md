---
timestamp: 2025-05-09T13:07:07+03:00
modified: 2025-05-10T07:37:07+03:00
draft: "false"
title: "Speed Matters: How I Optimized My ZSH Startup to Under 70ms"
creation_date: 2025-05-09T20:07:07+03:00
tags: ["zsh", "performance", "shell", "macos", "software", "software-development", "dotfiles", "terminal"]
categories: ["tutorials", "productivity", "software-development"]
---
# Speed Matters: How I Optimized My ZSH Startup to Under 70ms

Speed isn’t just about shaving milliseconds for fun (although... that _is_ kind of fun). It’s about protecting **flow**.

When you’re in the zone, every interruption counts - even a half-second delay opening a terminal. These delays break momentum. They add friction. And over time, they affect your behavior subconsciously: you stop opening new shells, hesitate to try quick commands, avoid the very tools that should be helping you.

That’s the thing - tools should feel like extensions of your body. You shouldn’t _think_ about them. They should just be there, ready, seamless. When your tools are fast, they disappear into the background and let you focus entirely on the work.

That’s why I care. Speed isn’t just a metric - it’s a multiplier on creativity, productivity, and joy.

This is the story of how I fixed one of my most important tools in my toolkit when it started slowing me down.

> You can find my existing [dotfiles](https://github.com/TheSantacloud/dotfiles) with my [zshrc](https://github.com/TheSantacloud/dotfiles/blob/main/zsh/.zshrc) config on my [GitHub page](https://github.com/TheSantacloud).

## Start with why

I got up this morning, started working on some project ([Lance](https://github.com/TheSantacloud/lance), fyi), put on some music to get the flow going, and opened up my terminal - and then I noticed that my shell takes FOREVER to load. Which is weird because I fixed it like a year ago. It **really** bugged me. I tried ignoring it, but it has a compounding effect every time I opened a new shell (which is ALL the time, in my workflow).

This wasn't the "right" thing to do at the moment, I thought... but I knew that I had a hard stop in 1 hour anyway, so down the rabbit-hole I went.
## What I'm using

- [Macbook Pro M1](https://www.youtube.com/watch?v=dQw4w9WgXcQ&pp=ygUJcmljayByb2xs) 2020, 8-core 3.2GHz CPU
* macOS Sequoia 15.4.1
- [Ghostty](https://ghostty.org/)
- [ZSH](https://en.wikipedia.org/wiki/Z_shell) obviously
- My IP address is 127.0.0.1

## Initial state

### Why are you not using `zsh-defer`, `oh-my-zsh`, `zplug`, `zdeeznuts`

I did use [oh-my-zsh](https://ohmyz.sh/) for a long time (years, actually) - but it became too tedious and slow. I had a lot of custom plugins that modified my prompt in zsh so I was somewhat familiar with zsh, so I just resorted to creating my own `~/.zshrc` file, because I figured - how hard can it be? it wasn't. It also gave me power that I didn't have before - in not using this "declarative" style configuration for it with oh-my-zsh (by just having a list that contains all the plugins I want from a magical and unknown location), I gained the ability to control my shell **and its load order** myself using ✨code✨.

As for the other things - I haven't tried most of them. But I find it hard to believe that they are something that I HAVE TO HAVE. They just sound simple enough... defer is just doing something later. zplug? just use a [git submodule](https://git-scm.com/book/en/v2/Git-Tools-Submodules). Less dependencies == Less things to break (I actually briefly wrote about it in my first blogpost about [isolating unknowns](https://santacloud.dev/posts/creating-my-blog---a-developers-tale-of-over-engineering-using-obsidian-hugo-and-github-pages/#isolating-unknowns))

I should really write sometime about the dependency hellscape jungle in the modern software age that we live in right now, but that's not the point right now.

### So- my baseline was 452ms startup time

Honestly, at that point I didn't measure how much my startup time takes. It just felt slow and tedious. For this blogpost I reverted and measured it and got that it takes ~**452ms** to load using:

```bash
time zsh -i -c exit
```

Feel free to [skip this rant](#initial-zsh-config), this next part is me trying to put into words why I think it's absurd that it takes this long.

[ChatGPT](https://www.pngall.com/clown-face-png/download/149439/) says it's OK, but this just feels insane to me. I create new shells *constantly*, for everything. This was a very noticeable thing for me, and it really bugged me. With my current setup, having 8 cores (4 performance, 4 energy-efficient) with a rough average of 2.5GHz (1GHz == 1B cycles per second), and a conservative rough average of 2.5 [IPC](https://en.wikipedia.org/wiki/Instructions_per_cycle) between them, and removing around 10% for general purpose computer stuff (and also to account for i/o access etc) this brings us to:

$$
((8 \times 2.5 \times 2.5 \times 10^9) - 10\%) \times 0.452  \approx 20,340,000,000
$$

That's ~**20.34B** instructions for EVERY TIME I run the shell (and that's being VERY conservative). This is crazy town. So I tried measuring vanilla ZSH using the same approach (just emptied out my `zshrc` file) - it took ~**38ms**. So basically, launching ZSH, which (at the very minimum):
- Spawns the process
- Reads from disk (I guess we call it drive now) and loads system wide config
- Some terminal emulator overhead

takes ~**38ms**, or ~**1.71B** instructions (according to previous calculation).

This means that my very custom setup takes ~**414ms** / ~**18.63B** instructions / **91.59%** of the time for every shell spawn.

To put this into perspective, if this were a game, and the amount of resources spent for the general benefit to load my pretty colors and auto-complete functionality, we would be playing at 2.41 FPS. I know that this is not the case, and it's a tough comparison (because it's meant to only load once, not 60 times per second), but I mean... why did we get so complex? it's just input listeners and hashmap retrieval for completions, some aliases, and a splash of color. How is it so bloated for what it's doing?!

I know I got too deep into the numbers, and yeah - I know most of it isn't really just instructions; it's mostly I/O latency, and the numbers are not accurate by any means necessary. What mattered was that **I felt slowed down, it frustrated me, and I couldn't justify the benefit I got for the time and frustration spent**.

$$ Benefit < F(Time, Frustration) $$
### Initial ZSH Config

You can see it in my [GitHub](https://github.com/TheSantacloud/dotfiles/blob/32793e3d5ca136f71a5c363bc8d5298478e9d54e/zsh/.zshrc) as well, but I'll put it here so that we can refer to specific things as we go:

```bash
# use this for profiling in case the shell becomes slow
export PROFILING_MODE=0
if [ $PROFILING_MODE -ne 0 ]; then
    zmodload zsh/zprof
fi

# general settings
export ZSH=$(readlink -f $HOME/.config/zsh)
export HISTFILE=$ZSH/.zsh_history
export HISTSIZE=10000
export SAVEHIST=10000
setopt HIST_IGNORE_ALL_DUPS
setopt HIST_FIND_NO_DUPS
export PATH="$PATH:$HOME/.local/bin/"

# theme
source $ZSH/themes/dracula/dracula.zsh-theme

# plugins
source $ZSH/plugins/fast-syntax-highlighting/fast-syntax-highlighting.plugin.zsh
source $ZSH/plugins/zsh-autosuggestions/zsh-autosuggestions.plugin.zsh
zstyle ':completion:*:*:git:*' script $ZSH/plugins/git-completions/git-completion.bash
fpath=($ZSH/plugins/zsh-completions/src $ZSH/plugins/git-completions $fpath)
autoload -Uz compinit && compinit

# golang
export PATH=$PATH:$(go env GOPATH)/bin

# python (uv)
source "$HOME/.local/bin/env"

# platformio
export PATH=$PATH:${HOME}/.platformio/packages/toolchain-xtensa/bin

# docker
source <(docker completion zsh)

# pnpm
export PNPM_HOME="${HOME}/Library/pnpm"
case ":$PATH:" in
  *":$PNPM_HOME:"*) ;;
  *) export PATH="$PNPM_HOME:$PATH" ;;
esac

# tmux
[ -f ~/.fzf.zsh ] && source ~/.fzf.zsh

# aliases
source $ZSH/aliases/customized.plugin.zsh
source $ZSH/aliases/kubectl.plugin.zsh
source $ZSH/aliases/git.plugin.zsh
export KUBE_EDITOR=nvim
bindkey -s '^f' "tmux-sessionizer\n"

# pyenv
export PYENV_ROOT="$HOME/.pyenv"
command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"

if [ $PROFILING_MODE -ne 0 ]; then
    zprof
fi

# google sdk
if [ -f "${HOME}/Downloads/google-cloud-sdk/path.zsh.inc" ]; then . "${HOME}/Downloads/google-cloud-sdk/path.zsh.inc"; fi
if [ -f "${HOME}/Downloads/google-cloud-sdk/completion.zsh.inc" ]; then . "${HOME}/Downloads/google-cloud-sdk/completion.zsh.inc"; fi

desc
```

## Profiling and optimizing

What do you know - thank you past-self for giving me this profiling thing to work with (literally the first line of code), I had absolutely no recollection of doing this.
### Figuring out the primary culprits

I switched the `PROFILING_MODE` flag to 1 and fired a new shell (I won't show the output here, because it's very big and the formatting just doesn't work here. Just try it for yourself to see) -

`compaudit` and `compinit` take the most time, and they're constructed from different things in their description below. This is helpful, but not for everything in my configurations, so I resorted to using:

```bash
zsh_start_time=$(python3 -c 'import time; print(int(time.time() * 1000))')
...<content of zshrc>
zsh_end_time=$(python3 -c 'import time; print(int(time.time() * 1000))')
echo "Shell init time: $((zsh_end_time - zsh_start_time)) ms"
```

This wouldn't give me the initial shell startup (that we already know takes 38ms), but I have nothing I can do about it, so this is plenty good enough, and I just subtracted this time (21ms) in the table below.

I proceeded to comment out everything, and start with a basic binary search to find the major culprits. For this test I created a new shell to avoid cache as much as possible (since the issue I wanted to solve was **new** shell spawns). This here is the breakdown I did after the fact (for the purposes of this blogpost. In reality I just binary searched, and squashed the biggest problems in order until I was happy)

| Component                              | Duration (in milliseconds) | Comment             |
| -------------------------------------- | -------------------------- | ------------------- |
| general settings                       | 3ms                        |                     |
| dracula theme                          | 25ms                       |                     |
| compinit (with nothing else)           | 242ms / 25ms               | Baseline, coldstart |
| fast-syntax-highlighting (w/ compinit) | 225ms / 13ms               | Coldstart           |
| zsh-autosuggestions (w/ compinit)      | 195ms / 26ms               | Coldstart           |
| git-completions (w/ compinit)          | 242ms / 28ms               |                     |
| golang PATH                            | 9ms                        | This is a funny one |
| python uv                              | 66ms                       |                     |
| platformio PATH                        | 1ms                        |                     |
| docker (w/ compinit)                   | 218ms / 40ms               | Coldstart           |
| pnpm                                   | 1ms                        |                     |
| fzf                                    | 3ms                        |                     |
| aliases - customized                   | 1ms                        |                     |
| aliases - kubectl                      | 3ms                        |                     |
| aliases - git                          | 30ms                       |                     |
| bindkey tmux-sessionizer               | 1ms                        |                     |
| pyenv                                  | 172ms                      |                     |
| google-sdk                             | 1ms                        |                     |

### Removing everything I don't need

Pretty straight forward, and easiest

1. python UV - I forgot you existed (using mostly [poetry](https://python-poetry.org/)). Outta here, you and your 66ms.
2. pnpm - don't use it
3. git aliases - I just never use them, and I mostly use [vim-fugitive](https://github.com/tpope/vim-fugitive) anyways (also, 30ms? come on man)

## pyenv - you 172ms bastard

This one bugged me. I use Python pretty frequently, and [pyenv](https://github.com/pyenv/pyenv) is a good tool to manage python versions, and integrates well with the rest of the python virtualenv programs like `poetry`.

But this program man... Just try to type in `poetry` in your terminal to get the help menu.

```bash
$ time poetry
<help content>
poetry  0.22s user 0.05s system 97% cpu 0.276 total
```

Yep. you're reading this right. 276ms to print 74 lines/4323 characters to screen.

A while ago I removed the base python binary that comes preinstalled on Mac because it kept on confusing me, and breaking things. So I use pyenv exclusively for python.

OK. let's take a look at it (this is how they tell you how to install in [their official readme](https://github.com/pyenv/pyenv/tree/f216b4bfb1598347137ecb3c4a8f893baf9ea37f?tab=readme-ov-file#zsh), expand ZSH):
```bash
# pyenv
export PYENV_ROOT="$HOME/.pyenv"
command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
```

But let's break it down. What am I actually trying to achieve? because I rarely use `pyenv` as it is (poetry does). But I do want **python** to be available. And without it - there's just no python in my PATH.

Oh. Simple. Just put python in my PATH.

```bash
$ which python
/Users/santacloud/.pyenv/shims/python
```

Cool cool cool, so let's do:

```bash
ln -sf "$PYENV_ROOT/shims/python"       ~/.local/bin/python
ln -sf "$PYENV_ROOT/shims/pip"       ~/.local/bin/pip
ln -sf "$PYENV_ROOT/shims/python3"       ~/.local/bin/python3
ln -sf "$PYENV_ROOT/shims/pip3"       ~/.local/bin/pip3
```

And as for pyenv - let's just lazy load it. When I do need it - make the first time a bit slower.

```bash
export PYENV_ROOT="$HOME/.pyenv"
pyenv() {
  unset -f pyenv
  eval "$(command pyenv init -)"
  pyenv "$@"
}
```

This basically creates a function called `pyenv`, and since it's not in my PATH yet - that's the one that would be used when I type in `pyenv` in my shell. The first thing it does is unsets itself (because we're about to put the binary in the PATH), evaluates their init thing (whatever the hell they do there) and starts it.

### Docker you nasty b****

Remember this guy? with the cold-start taking over 200ms?
```bash
source <(docker completion zsh)
```

This one *really* bugged me. So I tried compiling it, but it didn't really take - it's just aliases, so it had a very minor effect. So I tried lazy loading it (and went down a rabbit hole of trying to parallelize, or maybe even load it in the background while the shell has already spawned). Nothing worked.

And then I just removed it. And to my surprise - I had docker completions.

What? How? Who? What?

I obviously have this somewhere, and the only place it can be is where I load completions. So I added this script to figure it out:

```bash
for dir in $fpath; do
  if [[ -f "$dir/_docker" ]]; then
    echo "Found _docker in: $dir"
  fi
done
```

And what do you know, docker exists there already (in `$ZSH/plugins/zsh-completions/src`).

Onwards and upwards!

### Colors

There's really no justification to take 25ms to color the prompt and put some extra information there.

So I tried looking at what [dracula](https://github.com/dracula/zsh/tree/master) does, and I played around with it to see how long it takes. And then I figured - this shell is almost perfect for me... what do I **actually** need? just to see some git information, and basic colors for the prompt.

So I created `$ZSH/themes/minimal-falcon.zsh-theme`.

I've been meaning to do that for a while now, but I never got to it - I use [falcon](https://github.com/fenetikm/falcon) for [neovim](https://neovim.io/) and I really like it, but they don't have an official ZSH theme. Which is good, because it probably would have been with bloat as well. Let's just take it's colors and create:

ChatGPT was actually very helpful here - I just took a screenshot of my colors, and copied over the falcon colors. Told it that I want to color my zsh prompt, and it suggested to give me fzf colors as well - why not! This is after some minor tweaking.

```bash
# zsh/themes/minimal-falcon.zsh-theme
autoload -Uz colors && colors

# LS colors and such (8-bit color)
export CLICOLOR=1
export LSCOLORS=dxfxfxdxfxdedeacacad

# fzf
export FZF_DEFAULT_OPTS="$FZF_DEFAULT_OPTS \
--color='dark\
,fg:#b4b4b9\
,bg:#000000\
,hl:#ffc552\
,fg+:#f8f8ff\
,bg+:#36363a\
,hl+:#ffc552\
,query:#f8f8ff\
,gutter:#020221\
,prompt:#ffc552\
,header:#ffd392\
,info:#bfdaff\
,pointer:#ffe8c8\
,marker:#ff3600\
,spinner:#bfdaff\
,border:#36363a'\
"

# prompt
function git_prompt() {
  git rev-parse --is-inside-work-tree &>/dev/null || return
  ref=$(git symbolic-ref --short HEAD 2>/dev/null || git describe --tags --always)
  echo "%F{173}($ref)%f"
}
setopt PROMPT_SUBST
PROMPT='%F{245}%1~%f$(git_prompt) %F{180}❯%f '
```

Simple, elegant, everything I need, and **FAST**.

### Compiling

Here I wanted to address those coldstarts that I've been experiencing on and off with everything relating `compinit`. I didn't really know what it is, so I asked ChatGPT what is it, and what are the common ways to make it faster.

Apparently - `compinit` is `zsh`'s completion system. So this is why it's needed by autosuggestion, git-completions, zsh-completions, etc.

And it also told me 2 more interesting things:
- The entire zshrc file can be compiled as well, and the dump loaded with `compinit`
- You can compile zsh scripts with `zcompile` - it creates a `{filename}.zwc` file that is the compiled version of it (which is obviously faster)

Just don't forget to add these to my `.gitignore` for my dotfiles:
```
zsh/.zcompdump
*.zwc
```

#### Compiling zshrc as a whole

```bash
autoload -Uz compinit
ZSH_COMPDUMP="${ZSH}/.zcompdump"
compinit -C -d "$ZSH_COMPDUMP"
```

> **Note:** `-C` to bypass the check for rebuilding the dumpfile and the call to `compaudit`. This can shave off 15ms more. I feel comfortable disabling `compaudit` (security check) because it checks the security within `fpath` because all of my sources are trusted by me, I never update them, and the fact that my computer is just for me. However, if these don't apply to you - I would advise to rethink your decision and understand thoroughly what `compaudit` does.

#### zsource

I created a minor helper function:
```bash
zsource() {
  local file=$1
  local zwc="${file}.zwc"
  if [[ -f "$file" && (! -f "$zwc" || "$file" -nt "$file") ]]; then
    zcompile "$file"
  fi
  source "$file"
}
```

This just checks if there's a zwc file exists, and if it doesn't or the main file has been modified - it uses `zcompile` on the file. And then it sources the file (zsh automatically checks if a zwc file is present, and uses it if it is).

And then every time I just replaced all my `source` functions with `zsource`, like so:

```bash
zsource $ZSH/plugins/fast-syntax-highlighting/fast-syntax-highlighting.plugin.zsh
zsource $ZSH/plugins/zsh-autosuggestions/zsh-autosuggestions.plugin.zsh
...
zsource $ZSH/themes/minimal-falcon.zsh-theme
```

### Consolidating

Some final cleanup, now that the file has been optimized is to consolidate some things - like putting all the PATH into one (this actually isn't necessary, it's not like that it takes too long to concat to PATH)

```bash
export PATH="$PATH:$HOME/.local/bin/:${HOME}/go/bin:${HOME}/.platformio/packages/toolchain-xtensa/bin"
```

However - do you see this nice golang optimization? `$(go env GOPATH)/bin` turned to `${HOME}/go/bin`. Apparently, `go env GOPATH` took 9ms. So this just very easily shaved it off.

### Are we done?

Yep. 30ms for the content, 38ms for a vanilla shell experience. **68ms**. This is good enough for now. We're done.

Throughout this process I constantly kept asking myself - am I done? am I exaggerating? am I too tunnel-visioned? And often time I was. But my goal was to get it under 100ms, and since I got the big things down first, and I could see a clear path (from my newfound knowledge) to remove other things to shave it off even more - I did those as well.

#### Final time measurements

Overall: **30ms** (full load, not considering vanilla zsh time). I can work with that.
Like in the previous table, the sum of the individual parts is greater than the actual zshrc runtime. But to illustrate the vast difference:

| Component                    | Duration (in milliseconds) |
| ---------------------------- | -------------------------- |
| general settings             | 3ms                        |
| falcon theme                 | 4ms                        |
| compinit (with nothing else) | 15ms                       |
| all plugins + compinit       | 23ms                       |
| golang PATH                  | 9ms                        |
| python uv                    | removed                    |
| docker (w/ compinit)         | removed (already exists)   |
| pnpm                         | removed                    |
| fzf+tmux                     | 2ms                        |
| aliases - all                | 2ms                        |
| pyenv                        | 0ms                        |
| google-sdk                   | removed                    |

#### Final product

```bash
# use this for profiling in case the shell becomes slow
export PROFILING_MODE=0
if [ $PROFILING_MODE -ne 0 ]; then
    zmodload zsh/zprof
    zsh_start_time=$(python3 -c 'import time; print(int(time.time() * 1000))')
fi

# compile zsh file, and source them - first run is slower
zsource() {
  local file=$1
  local zwc="${file}.zwc"
  if [[ -f "$file" && (! -f "$zwc" || "$file" -nt "$file") ]]; then
    zcompile "$file"
  fi
  source "$file"
}


# general settings
export ZSH=$(readlink -f $HOME/.config/zsh)
export HISTFILE=$ZSH/.zsh_history
export HISTSIZE=10000
export SAVEHIST=10000
setopt HIST_IGNORE_ALL_DUPS
setopt HIST_FIND_NO_DUPS
export KUBE_EDITOR=nvim

# NOTE : using ${HOME}/go/bin instead of $(go env GOPATH)/bin for optimization
export PATH="$PATH:$HOME/.local/bin/:${HOME}/go/bin:${HOME}/.platformio/packages/toolchain-xtensa/bin"

# theme
zsource $ZSH/themes/minimal-falcon.zsh-theme

# plugins
zsource $ZSH/plugins/fast-syntax-highlighting/fast-syntax-highlighting.plugin.zsh
zsource $ZSH/plugins/zsh-autosuggestions/zsh-autosuggestions.plugin.zsh
zstyle ':completion:*:*:git:*' script $ZSH/plugins/git-completions/git-completion.bash
fpath=($ZSH/plugins/zsh-completions/src $ZSH/plugins/git-completions $fpath ~/.zfunc)

autoload -Uz compinit
ZSH_COMPDUMP="${ZSH}/.zcompdump"
compinit -C -d "$ZSH_COMPDUMP"

# aliases
zsource $ZSH/aliases/customized.plugin.zsh
zsource $ZSH/aliases/kubectl.plugin.zsh

# pyenv
export PYENV_ROOT="$HOME/.pyenv"
pyenv() {
  unset -f pyenv
  eval "$(command pyenv init -)"
  pyenv "$@"
}

# utilities
[ -f ~/.fzf.zsh ] && zsource ~/.fzf.zsh
bindkey -s '^f' "tmux-sessionizer\n"

# profiling
if [ $PROFILING_MODE -ne 0 ]; then
    zsh_end_time=$(python3 -c 'import time; print(int(time.time() * 1000))')
    zprof
    echo "Shell init time: $((zsh_end_time - zsh_start_time - 21)) ms"
fi
```

## Summary - was it a waste of time? (and my thoughts on tools)

Have you ever been to a really good craftsman’s shop? Their tools are often well-worn but finely tuned - oiled, efficient, familiar. They might be missing a few bolts, but nothing that disrupts their flow. And chances are, the craftsman has disassembled and reassembled every one of them, if only to understand what makes them tick.

That’s the thing: tools don’t have to be perfect. They just have to fit _you_. That’s what makes them perfect. There’s no one-size-fits-all - not for tools, and definitely not for workflows.

Tools are extensions of our mind and body. Without them, we would've still been animals (we love you, opposable thumbs). If your tools slow you down, they’re no longer tools - they’re obstacles. And obstacles are worth removing.

So yes, we should **respect our tools**.

But we should also remember why they exist in the first place: **to help us get the *actual* work done**. The best way to understand a tool is to _use_ it - not to obsess over how someone else might configure it or chase every theoretical optimization. Make it work. Use it. Maintain it when needed.

It’s easy to fall into the trap of endlessly tweaking your setup, chasing that last millisecond. But tools are only valuable if they help you _do_ the work - not just prepare to do it.

By understanding our tools, shaping them to fit us, and maintaining them from time to time, they’ll repay us - in professionalism, in product quality, and in the joy of the craft.
