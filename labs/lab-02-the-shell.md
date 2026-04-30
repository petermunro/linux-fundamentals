# Lab 02: The Shell

**Objective:** Work with the shell more effectively — using wildcards, brace expansion, `find`, environment variables, and aliases. By the end, you will understand how Bash interprets what you type and how to use its features to work more efficiently.

---

## Part 1: Wildcards

Wildcards allow you to match multiple files in a single command. The shell expands them before passing arguments to the command.

### Exercise 1.1 — Set up a working directory

1. Create a directory and move into it:

   ```bash
   mkdir ~/shell-lab && cd ~/shell-lab
   ```

2. Create a set of files to work with:

   ```bash
   touch app.log app.conf app.py server.log server.conf server.py test.log
   ```

3. List everything:

   ```bash
   ls
   ```

### Exercise 1.2 — Match by extension

List only the `.log` files using the `*` wildcard:

```bash
ls *.log
```

Now list only the `.conf` files:

<details>
<summary>Reveal Solution</summary>

```bash
ls *.conf
```

</details>

### Exercise 1.3 — Match by prefix

List all files whose name starts with `app`:

<details>
<summary>Reveal Solution</summary>

```bash
ls app.*
```

</details>

### Exercise 1.4 — The ? wildcard

The `?` wildcard matches exactly **one** character. Predict which files this will match, then run it:

```bash
ls ???.log
```

<details>
<summary>Reveal Answer</summary>

It matches `app.log` — exactly three characters before `.log`. `server.log` has six characters before the dot, so it does not match.

</details>

---

## Part 2: Brace Expansion

Brace expansion is not a wildcard — the shell generates combinations from a pattern *before* checking whether files exist. It is extremely useful for creating or referencing multiple things at once.

### Exercise 2.1 — Create multiple directories at once

Instead of running `mkdir` three times, use brace expansion:

```bash
mkdir -p project/{src,tests,docs}
ls project/
```

### Exercise 2.2 — Create a range of files

Create ten numbered log files:

```bash
touch log_{01..10}.txt
ls log_*.txt
```

### Exercise 2.3 — Use brace expansion with cp

Make a backup of `app.py` without retyping the filename:

```bash
cp app.py{,.bak}
ls app.py*
```

Notice the pattern: `{,.bak}` expands to two arguments — the original name and the name with `.bak` appended.

<details>
<summary>What does the shell actually run here?</summary>

```bash
cp app.py app.py.bak
```

Brace expansion happens in the shell before `cp` ever sees the arguments. You can verify what any expansion produces using `echo`:

```bash
echo app.py{,.bak}
```

</details>

---

## Part 3: Finding Files with find

`find` searches the file system recursively and is far more powerful than `ls`. Its basic form is:

```
find <where to search> <what to match>
```

### Exercise 3.1 — Find files by name

Find all `.log` files in your current directory and below:

```bash
find . -name "*.log"
```

### Exercise 3.2 — Find files by type

Find only directories (not files) under your current directory:

```bash
find . -type d
```

The `-type f` flag matches regular files. The `-type l` flag matches symlinks.

### Exercise 3.3 — Find recently modified files

Find all files modified in the last 1 minute (useful for confirming that something just changed):

```bash
find . -mmin -1
```

Try touching a file and running it again:

```bash
touch fresh.txt
find . -mmin -1
```

### Exercise 3.4 — Find and act on results

Find all `.bak` files and delete them using `-delete`:

```bash
find . -name "*.bak" -delete
ls app.py*
```

> **Note:** Always run `find` without `-delete` first to confirm the matches look right before deleting.

---

## Part 4: Shell History and Shortcuts

The shell keeps a history of every command you run, and provides shortcuts to reuse it efficiently.

### Exercise 4.1 — View your history

```bash
history
```

Each line is numbered. You can re-run any command by number:

```bash
!<number>
```

Replace `<number>` with any line number from your history output.

### Exercise 4.2 — Search history interactively

Press `Ctrl+R` and start typing part of a command you ran earlier (e.g., `find`). The shell will show the most recent match. Press `Ctrl+R` again to cycle through older matches. Press `Enter` to run it, or `Esc` to cancel.

### Exercise 4.3 — Repeat the last command

Run any command, then repeat it immediately:

```bash
!!
```

### Exercise 4.4 — Useful keyboard shortcuts

Try each of these:

| Shortcut | Effect |
|----------|--------|
| `Ctrl+A` | Jump to the start of the line |
| `Ctrl+E` | Jump to the end of the line |
| `Ctrl+W` | Delete the word to the left of the cursor |
| `Ctrl+L` | Clear the screen (same as `clear`) |
| `Alt+.`  | Paste the last argument from the previous command |

---

## Part 5: Environment Variables

Environment variables are key-value pairs that the shell and programs use to configure their behaviour. They are always written in uppercase by convention.

### Exercise 5.1 — View the current environment

Print all environment variables:

```bash
env
```

There will be many. Find your home directory variable:

```bash
echo $HOME
```

### Exercise 5.2 — View the PATH

`PATH` is the most important environment variable — it tells the shell where to look for commands:

```bash
echo $PATH
```

Each directory is separated by `:`. When you type `ls`, the shell searches each of these directories in order until it finds an executable named `ls`.

Find out exactly which `ls` the shell is using:

```bash
which ls
```

### Exercise 5.3 — Set a variable in the current shell

```bash
MY_NAME="Ada Lovelace"
echo $MY_NAME
```

### Exercise 5.4 — Export a variable to child processes

A variable set without `export` exists only in the current shell. Child processes cannot see it. Try this:

1. Set a variable without exporting:

   ```bash
   SECRET="hidden"
   bash -c 'echo $SECRET'
   ```

   The child shell prints nothing.

2. Now export it:

   ```bash
   export SECRET="hidden"
   bash -c 'echo $SECRET'
   ```

   The child shell can now see it.

### Exercise 5.5 — Add a directory to PATH

This is a very common real-world task. Add a custom `bin` directory to your PATH:

```bash
mkdir ~/bin
export PATH="$HOME/bin:$PATH"
echo $PATH
```

Confirm your directory now appears at the front.

> **Note:** This change only lasts for the current session. To make it permanent, you would add the `export` line to `~/.bashrc` or `~/.bash_profile`.

---

## Part 6: Aliases

Aliases let you create shorthand names for commands you use frequently.

### Exercise 6.1 — Create a simple alias

```bash
alias ll='ls -lh'
ll
```

### Exercise 6.2 — Create a safer rm alias

Many people alias `rm` to prompt for confirmation:

```bash
alias rm='rm -i'
```

Now try removing a file — the shell will ask you to confirm:

```bash
touch throwaway.txt
rm throwaway.txt
```

### Exercise 6.3 — List and remove aliases

View all active aliases:

```bash
alias
```

Remove an alias:

```bash
unalias rm
```

> **Note:** Like exported variables, aliases defined in the terminal last only for the current session. To make them permanent, add them to `~/.bashrc`.

---

## Part 7: Clean Up

Remove the lab directory:

```bash
cd ~
rm -r ~/shell-lab
```
