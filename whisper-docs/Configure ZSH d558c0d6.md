The [Z shell (zsh)](https://en.wikipedia.org/wiki/Z_shell) has a built-in command, called `zle`, which, when used with the `reset-prompt` argument forces the prompt to be re-expanded, then redisplays the edit buffer.

If you want to try it, use the following steps:

Install Z shell with this command:

```shell
sudo apt-get install zsh -y
```

When you run `zsh` for the first time, choose `0` when you are asked.

Next, we'll install `oh-my-zsh` to add a clean theme:

```shell
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

Edit the `~/.zshrc` file and edit the following lines:

```shell
nano ~/.zshrc
```

```shell
# If you come from bash you might have to change your $PATH.
# export PATH=$HOME/bin:/usr/local/bin:$PATH

# Path to your oh-my-zsh installation.
export ZSH="$HOME/.oh-my-zsh"

# Set name of the theme to load --- if set to "random", it will
# load a random theme each time oh-my-zsh is loaded, in which case,
# to know which specific one was loaded, run: echo $RANDOM_THEME
# See https://github.com/ohmyzsh/ohmyzsh/wiki/Themes
ZSH_THEME="xiong-chiamiov-plus"

# Set list of themes to pick from when loading at random
# Setting this variable when ZSH_THEME=random will cause zsh to load
# a theme from this variable instead of looking in $ZSH/themes/
# If set to an empty array, this variable will have no effect.
# ZSH_THEME_RANDOM_CANDIDATES=( "robbyrussell" "agnoster" )
```

Save the file and close.

In your terminal, when you are still using zsh, run `source ~/.zshrc`, or simply `zsh`, to reset your prompt.&#x20;

Next, we'll make ZSH our default shell.

Open /etc/passwd:

```shell
sudo nano /etc/passwd
```

Find the line with your username:

```shell
username:x:1634231:100:Your Name:/home/username:/bin/bash
```

and replace bash with zsh:

```shell
username:x:1634231:100:Your Name:/home/username:/bin/zsh
```

Log out and log in back for the changes to take effect. Now, you have a ZSH shell by default that displays colors and a live clock.
