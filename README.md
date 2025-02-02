# bENV - Basher ENVironments
![](logo.jpg)

[![basher install](https://www.basher.it/assets/logo/basher_install.svg)](https://www.basher.it/package/)

bENV is like Python virtual environments but for Bash scripts. It wraps [Basher](https://github.com/basherpm/basher), a popular package manager for shell scripts. Inspired by [Desk](https://github.com/jamesob/desk), it simplifies and extends Basher without reinventing the wheel.

# Installation

Run `basher install benvsh/benv` to install bENV.

# Usage

`benv <benv_id> [<package_url1:package_url2:...>]`

Parameters:
- <benv_id> Identifier for the virtual environment. For the id, use folder names under the benv root folder.
- <package_url[n]> URL to the package repository, same structure as basher's install.

# Example

To create and activate an environment `my_env` with specified packages:

```bash
benv my_env --ssh mygit.local/myuser/somepackage:LuRsT/hr:--ssh mygit.local/myuser/otherpackage:sstephenson/bats
```

This will:

1. Create the `my_env` folder under `.basher/cellar/benv`.
2. Activate `my_env` environment: install all packages specified and launch a Bash subshell. By default, Basher will only use packages installed in that environment.

To exit the bENV environment, simply run `exit` or press `Ctrl-D`. If you’ve activated multiple environments nested within each other, exiting the last will return you to the parent environment and so on.

# Motivation

Like many, you're likely juggling multiple projects, each requiring a different set of shell scripts.  
If you used Basher alone, all packages and their environments (functions, variables, completions) would be loaded, even if you only needed a small subset for your task.

bENV allows you to create and manage multiple environments, each tailored to a specific set of scripts, while only loading what’s necessary for the task at hand.

# Features

- **Multiple bENV environments**  
  Effortlessly activate and switch between environments, each with its own set of packages. This allows you to focus on the task at hand without loading unnecessary scripts.

- **Transparent Basher usage**  
  Basher commands are wrapped by bENV, making package management seamless. Refer to the section "Transparent Basher usage" for details.

- **Package sharing between environments**  
  Packages can be shared between sibling environments by symlinking to a central location. This makes it easier to update packages across all environments. You can also opt to install packages locally within a specific environment. See "Flexible. Customizable" for more.

- **Hierarchical environment grouping**  
  bENV supports nested sub-environments, enabling you to organize environments in a parent-child structure while maintaining shared packages between sibling environments. Check "Enter Sub-environments" for more details.

# Efficient package management

```
.basher/cellar/benv/.benv0/packages
```

By default, all bENV environments will symlink their packages to this folder, allowing shared access across sibling environments.

Similar to `pnpm`'s approach over `npm`, this reduces redundant package installations and disk space usage by creating symlinks instead of duplicating packages across environments. 

Additionally, activating a bENV environment only loads the functions and environment variables needed for the current task, improving efficiency and reducing unnecessary overhead compared to using Basher alone.

# Transparent Basher usage

bENV wraps Basher’s commands to provide transparent package management. For instance, when you run:

```bash
basher install <some_package>
```

bENV will automatically symlink the package to the central package repository shared across all bENV environments, located under `BENV0_ROOT`. If the package already exists, it will not be cloned again, but reused.

# Flexible. Customizable

Take this for example. BENV_ROOT path is configurable. Just like you could customize BASHER's environment variables, you can do the same for bENV.

```bash
# create a script in one of your Basher packages, let's call it my_user/benv2
# .basher/cellar/packages/my_user/benv2/benv2.sh
export BENV_ROOT="$HOME/.basher/cellar/pkg_envs"
export BENV0_ROOT="$BENV_ROOT/root_env"

# both paths don't even have to be under basher at all. So if you want you can also do:
# export BENV_ROOT="some/other/path"
# export BENV0_ROOT="and/another/path"
benv $@
```

Next use benv2.sh everywhere you want to use bENV and this pattern will apply to all your created environments.

## Enter, Sub-environments

Now try changing benv2.sh to:

```bash
# .basher/cellar/packages/my_user/benv2/benv2.sh
export BENV_ROOT="$BASHER_PREFIX/benvs"
export BENV0_ROOT="$BENV_ROOT/.benv0"
benv $@
```

Fun fact, since activating a bENV environment internally resets the BASHER_PREFIX var to that env's path, by using the above you can achieve sub-environments. That is because while parent bENV is active, BASHER_PREFIX would point to it's path.
Each new bENV will be created under the parent bENV's path.

Like:

```txt
.basher/cellar/benvs
├── my_env
│   └── my_sub_env
│       └── my_sub_sub_env
└── my_other_env
```

Then in a new terminal, run:

```bash
benv2.sh my_env
basher install <some_package>:<some_other_package>

# and while still in my_env, activate a sub-environment and install some packages in it:
benv2.sh my_sub_env --ssh mygit.local/myuser/some_package:LuRsT/hr:--ssh mygit.local/myuser/some_other_package:sstephenson/bats
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
│   │           └── sstephenson
│   │               └── bats <symlink2>
│   │           └── LuRsT
│   │               └── hr <symlink2>
│   │       └── .benv0
│   |           ├── packages <symlink2_target>
│   |           │   ├── myuser
│   |           │   │   └── some_package 
│   |           │   │   └── other_package
│   |           │   └── sstephenson
│   |           │       └── bats 
│   |           │   └── LuRsT
│   |           │       └── hr
└── .benv0 
  └── packages <symlink1_target>
      ├── some_package
      └── some_other_package
```

Currently, to reenter the bENV sub-environment, do:

```bash
benv2.sh my_env
#followed by
benv2.sh my_sub_env
```

Note: While working with bENV enviroments structured like this(hierarchically), it is important to always keep using benv2.sh instead of benv. 
This ensures the sub-environments will correctly reference the packages at correct locations.

TODO: add support for benv my_env/my_sub_env

# Support My Work

[Buy me a coffee!](https://ko-fi.com/s/5d943125ff)

I’ve put 30 hours into this script, refining and simplifying it. If you find it useful, consider buying me a coffee to show your appreciation.  

Thanks for your support!