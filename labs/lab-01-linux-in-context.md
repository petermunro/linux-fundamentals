# Lab 01: First Steps in Linux

**Objective:** Gain confidence with essential Linux commands — navigating the file system, creating and manipulating files and directories, and using the built-in help system.

---

## Part 1: Where Am I?

### Exercise 1.1 — Print your current location

When you open a terminal in Linux, you are always somewhere in the file system. Find out where you are by running:

```bash
pwd
```

You should see a path such as `/home/yourname`. This is your **home directory** — your personal working space.

### Exercise 1.2 — List the contents of your home directory

Use `ls` to see what is in your current directory:

```bash
ls
```

Now try `ls` with a flag that shows more detail, including file sizes and ownership:

```bash
ls -l
```

Try also `ls -la` to include hidden files (those whose names begin with a `.`). Notice the difference.

---

## Part 2: Creating Structure

### Exercise 2.1 — Create a working directory for this lab

Create a directory called `linux-lab` in your home directory:

```bash
mkdir linux-lab
```

Confirm it exists:

```bash
ls
```

### Exercise 2.2 — Navigate into it and create some files

1. Move into the new directory:

   ```bash
   cd linux-lab
   ```

2. Confirm you are now inside it:

   ```bash
   pwd
   ```

3. Create three empty files:

   ```bash
   touch notes.txt config.cfg readme.md
   ```

4. List the directory to confirm all three files were created:

   ```bash
   ls
   ```

### Exercise 2.3 — Create a subdirectory structure

1. Create two subdirectories, `docs` and `data`:

   ```bash
   mkdir docs data
   ```

2. List the contents again to see the full structure:

   ```bash
   ls -l
   ```

---

## Part 3: Moving and Copying Files

### Exercise 3.1 — Copy a file

Copy `notes.txt` into the `docs` directory:

```bash
cp notes.txt docs/
```

Verify it is there:

```bash
ls docs/
```

### Exercise 3.2 — Rename a file

`mv` is used both to move files and to rename them. Rename `config.cfg` to `settings.cfg`:

```bash
mv config.cfg settings.cfg
```

List the directory to confirm the rename:

```bash
ls
```

### Exercise 3.3 — Move a file into a subdirectory

Move `settings.cfg` into the `data` directory:

```bash
mv settings.cfg data/
```

Confirm it has moved:

```bash
ls
ls data/
```

### Exercise 3.4 — Write some content and view it

1. Write a line of text into `readme.md` using `echo`:

   ```bash
   echo "This is my Linux lab." > readme.md
   ```

2. View the file contents using `cat`:

   ```bash
   cat readme.md
   ```

<details>
<summary>What does > do here?</summary>

The `>` symbol redirects the output of `echo` into the file instead of printing it to the screen. You will explore redirection in depth in Module 4.

</details>

---

## Part 4: Removing Files and Directories

### Exercise 4.1 — Delete a file

Remove the `readme.md` file:

```bash
rm readme.md
```

Confirm it is gone:

```bash
ls
```

### Exercise 4.2 — Delete a directory

Try removing the `docs` directory with `rm`:

```bash
rm docs
```

You will see an error — `rm` will not remove a directory without the `-r` (recursive) flag. Try:

```bash
rm -r docs
```

Confirm the directory is gone. Notice that `data` is still there.

> **Note:** `rm -r` permanently deletes a directory and everything inside it. There is no recycle bin in Linux. Use it carefully.

---

## Part 5: Getting Help

### Exercise 5.1 — Read the manual for a command

Every standard Linux command has a manual page. Open the manual for `ls`:

```bash
man ls
```

- Use the arrow keys or `j`/`k` to scroll.
- Press `q` to quit.

Find the flag that sorts output by file size. It is a single letter.

<details>
<summary>Reveal Answer</summary>

The `-S` flag sorts by file size (largest first). Try it:

```bash
ls -lS
```

</details>

### Exercise 5.2 — Use --help for a quick summary

For a faster overview, most commands accept `--help`:

```bash
mkdir --help
```

This is useful when you just need to remind yourself of a flag without reading the full manual.

### Exercise 5.3 — Look up a command you haven't used yet

Use `man` to read the manual for `cp`. Find the flag that copies a directory and all its contents recursively.

<details>
<summary>Reveal Answer</summary>

The `-r` (or `-R`) flag copies directories recursively:

```bash
cp -r data data-backup
ls
```

</details>

---

## Part 6: Putting It Together

### Exercise 6.1 — Navigate the file system

1. Go back to your home directory:

   ```bash
   cd ~
   ```

   (The `~` is shorthand for your home directory.)

2. Navigate to `/etc`:

   ```bash
   cd /etc
   ls
   ```

   You are looking at system configuration files. Do not modify anything here.

3. Return home in one step:

   ```bash
   cd
   ```

   Running `cd` with no argument always takes you home.

### Exercise 6.2 — Tidy up

Remove the `linux-lab` directory you created at the start:

```bash
rm -r ~/linux-lab
```

Confirm your home directory is back to its original state:

```bash
ls ~
```
