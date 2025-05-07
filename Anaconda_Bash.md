# Understanding Bash Scripts and Variables

This notebook explains basic concepts about Bash scripts (`.sh` files), environment variables, and important commands such as `export` and `source ~/.bashrc`.


## What is Bash?

Bash (Bourne Again Shell) is a command-line interpreter that lets you interact with your operating system through commands. It is widely used in Linux and macOS.

## What is a Bash Script (`.sh`)?

A Bash script is a text file containing a sequence of commands that Bash executes. These files usually end with `.sh`. You run these scripts to automate tasks.

Example of a simple bash script:

```bash
#!/bin/bash

echo "Hello, World!"
```

## Why Do We Set Variables in `.bashrc`?

Environment variables store information used by the operating system and other software programs. Setting these variables in `.bashrc` makes sure they are available every time you open a new terminal session.

Common reasons to set variables in `.bashrc`:

- **Software paths**: Easily access frequently used programs or scripts.
- **Configuration options**: Customize the behavior of your shell and software.

Example:

```bash
export MY_VARIABLE="my_value"
```

The `export` command makes a variable available to all programs running in the current shell session or scripts launched from it.

Example:

```bash
export PATH="/my/custom/path:$PATH"
```

This adds `/my/custom/path` to your current system path.

## What Does `source ~/.bashrc` Do?

The command `source ~/.bashrc` reloads your `.bashrc` file in the current terminal session. This is useful after making changes to `.bashrc`, so you don't need to restart the terminal to apply changes.

Example:

```bash
source ~/.bashrc
```

## Summary

- **Bash scripts** (`.sh`) automate command-line tasks.
- Environment variables set in `.bashrc` configure your shell environment.
- `export` makes variables accessible to all programs and scripts running from the shell.
- `source ~/.bashrc` applies changes made to `.bashrc` immediately.
"""
