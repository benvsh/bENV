![](logo.jpg)

# bENV - Basher ENVironments
[![basher install](https://www.basher.it/assets/logo/basher_install.svg)](https://www.basher.it/package/)

bENV is like python's virtual environments but for Bash scripts. 
It is a wrapper around [Basher](https://github.com/basherpm/basher), a successful package manager for shell scripts.

It was built out of desire to make something similar to [Desk](https://github.com/jamesob/desk) but simpler and more natural in usage. 
Desk is a nice tool but it felt it reinvents the wheel and doesn't take full advantage of what Basher already offers.
I needed something that supports/extends how Basher works internally and builds on top of it and still use it's functionality transparently.

And so bENV was born.

# Motivation

Like most, you're probably busy working on multiple projects or tasks. Each requiring a different set of shell scripts.

If you were to only use Basher, all packages and their environment(functions, variables, completions) would have to be loaded. Even if your current task only requires a small subset of them.

With bENV and Basher, you can now easily create and manage multiple environments. 

# Features

- multiple bENV environments. Each with its own set of packages.
Effortlessly activate/switch between them depending on the task you're working on.
This allows you to focus on what you're working on and load only the packages and functions you need. 

- transparent Basher usage.
All Basher commands are internally wrapped to also do the magic of bENV. Please see "Transparent Basher usage".

- share packages between multiple bENV environments 
By symlinking them to a central location, common to all of them. This makes it easier to update your packages across all environments.
Still, you can easily opt-out and have packages installed locally to that environment. Please look at "Flexible. Everything is customizable" section below on how to achieve this.

- group environments in a hierarchy
This allows you to create sub-environments and still share packages between them. Please see "Enter, Sub-environments".

# Installation

Run `basher install benvsh/benv` to install bENV.

# Usage

`benv <benv_id> [<package_url1:package_url2:...>]`

Parameters:
- <benv_id> Identifier for the virtual environment. For the id, use folder names under the benv root folder.
- <package_url[n]> URL to the package repository, same structure as basher's install.

# Example

`benv my_env --ssh mygit.local/myuser/somepackage:LuRsT/hr:--ssh mygit.local/myuser/otherpackage:sstephenson/bats`

This does 2 things:
- creates the my_env folder 
under .basher/cellar/benv

- activates the bENV environment `my_env` 
by starting a Bash subshell. And enables Basher to work only with packages installed in this environment. 

To exit this bENV environment, just exit the subshell with `exit` or Ctrl-D.
A nice thing, if you've activated multiple environments, one inside another, exiting the last will follow the breadcrumb and activate the parent. And so on.

# Efficient package management

Any bENV activation will first create ensure a bENV packages folder exist under:

.basher/cellar/benv/.benv0/packages

Thereafter, all bENV enviroments under .basher/cellar/benv/ will symlink their packages to this folder.

## How does this help?

Well, it works similarly with pnpm versus npm. Take a look at this comment:

https://www.reddit.com/r/node/comments/144xqd8/comment/jni5lex/
>>>  pnpm all the way for me.
The speed is a huge bonus.
The other thing that's a big plus for me is that I don't end up with gigabytes of node_modules strewn around my disk from various different projects. pnpm installs everything into one central place (which is easy to configure your backup to ignore) and then creates symlinks to it. 

When the number of common Basher packages grows and they're used between multiple bENV enviromnents, this is a nice win.

But the biggest win is that only the functions and enviroment variables needed for what you're working on currently are loaded. 
If you were to only use Basher, all packages and their environment(functions + variables) would be loaded. Even if at your current task you'd not need most of them.

# Transparent Basher usage

Basher's commands are internally wrapped. 
For example, `basher install <some_package>` will also do the magic of bENV. 

It automatically symlinks this package inside the central package repo shared by all bENV environments, the packages under BENV0_ROOT.
If it already exists there, it will not be cloned again but instead reused.

# Flexible. Customizable

Take this for example. BENV_ROOT path is configurable. Just like you could customize BASHER's environment variables.

```bash
export BENV_ROOT="$HOME/.basher/cellar/pkg_envs"
export BENV0_ROOT="$BENV_ROOT/root_env"
# and yet, both paths don't have to be under basher at all if you want
# export BENV_ROOT="some/other/path"
# export BENV0_ROOT="and/another/path"
benv my_env --ssh mygit.local/myuser/somepackage:LuRsT/hr:--ssh mygit.local/myuser/otherpackage:sstephenson/bats
```

Put this in a script and you can use this pattern to all your created environments.

## Enter, Sub-environments

One caveat of this is that you can create nested sub-environments with this approach.
By default, all bENV environments created go in the same BENV_ROOT folder thus creating a linear horizontal structure.

But what if you wanted a hierarchical structure instead?
Like:

```txt
.basher/cellar/benvs
├── my_env
│   └── my_sub_env
│       └── my_sub_sub_env
└── my_other_env
```

This can be easily achieved by using the base pattern offered by bENV.
Just set the BENV_ROOT0 folder to be related to BASHER_PREFIX folder which always points to the current bENV environment path.
Like:

```bash
# create a script in one of your Basher packages, let's call it benv2
# .basher/cellar/packages/my_user/benv2/benv2.sh
export BENV_ROOT="$BASHER_PREFIX/benvs"
export BENV0_ROOT="$BENV_ROOT/.benv0"
benv $@

# then in a new terminal, run:
benv2 my_env
basher install <some_package>:<some_other_package>

# and while still in my_env, do this:
benv2 my_sub_env --ssh mygit.local/myuser/somepackage:LuRsT/hr:--ssh mygit.local/myuser/otherpackage:sstephenson/bats
```

This will create:

```txt
.basher/cellar/benvs 
├── my_env
│   └── packages 
│       ├── some_package <symlink1>
│       ├── some_other_package <symlink1> 
│   ├── benvs
│   │   └── my_sub_env
│   │       └── packages 
│   │           ├── myuser 
│   │           │   ├── some_package <symlink2>
│   │           │   └── some_other_package <symlink2>
│   │   └── .benv0
│   |       ├── packages <symlink2_target>
│   |       │   ├── myuser
│   |       │   │   └── some_package 
│   |       │   │   └── other_package
│   |       │   └── sstephenson
│   |       │       └── bats 
│   |       │   └── LuRsT
│   |       │       └── hr
└── .benv0 
  └── packages <symlink1_target>
      ├── some_package
      └── some_other_package
```

Currently, to reenter the bENV sub-environment, do:

```bash
benv2 my_env
#followed by
benv2 my_sub_env
```

Note: While working with bENV enviroments structured like this(hierarchically), it is important to always keep using benv2 instead of benv. 
This ensures the sub-environments will correctly reference the packages at correct locations.

TODO: add support for benv my_env/my_sub_env

# Support My Work

[Buy me a coffee!](https://ko-fi.com/s/5d943125ff)

I’ve put 30 hours into this script, refining and simplifying it. If you find it useful, consider buying me a coffee to show your appreciation.  

Thanks for your support!