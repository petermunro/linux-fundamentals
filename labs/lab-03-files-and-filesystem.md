# Lab 03: Files, Directories, and the File System Hierarchy

**Objective:** Explore the Linux file system hierarchy, understand what lives where, and get hands-on experience with file metadata, hard links, symbolic links, and different file types.

---

## Part 1: Exploring the File System Hierarchy

The Linux file system is a single tree rooted at `/`. Everything — disks, devices, network mounts — appears somewhere in that tree. Knowing what lives where saves a great deal of searching.

### Exercise 1.1 — Explore /etc

`/etc` contains system-wide configuration files. It is read by applications and system services at startup.

1. List the contents:

   ```bash
   ls /etc
   ```

2. Look at the hostname file — it contains the machine's name:

   ```bash
   cat /etc/hostname
   ```

3. Look at the OS release information:

   ```bash
   cat /etc/os-release
   ```

### Exercise 1.2 — Explore /var

`/var` holds data that changes at runtime: logs, mail spools, caches, and databases.

1. List the top-level contents:

   ```bash
   ls /var
   ```

2. List the log directory:

   ```bash
   ls /var/log
   ```

3. Look at the last 20 lines of the system log (it may be called `syslog` or `messages` depending on your distribution):

   ```bash
   tail -20 /var/log/syslog 2>/dev/null || tail -20 /var/log/messages 2>/dev/null
   ```

### Exercise 1.3 — Explore /proc

`/proc` is a **virtual file system** — it contains no real files on disk. The kernel generates its contents on the fly to expose system state. Every number you see in `/proc` is a process ID.

1. See how long the system has been running:

   ```bash
   cat /proc/uptime
   ```

   The first number is seconds. Divide by 3600 mentally to get hours.

2. Count the number of CPU cores:

   ```bash
   grep -c processor /proc/cpuinfo
   ```

3. Check available memory:

   ```bash
   cat /proc/meminfo | head -5
   ```

4. Find your current shell's process ID, then look at its entry in `/proc`:

   ```bash
   echo $$
   ls /proc/$$
   ```

   You are looking at live kernel data about your own shell process.

### Exercise 1.4 — Explore /dev

`/dev` contains device files — the kernel's representation of hardware and virtual devices.

1. List a few:

   ```bash
   ls /dev | head -20
   ```

2. `/dev/null` discards anything written to it and returns nothing when read. It is the standard way to suppress output:

   ```bash
   echo "this goes nowhere" > /dev/null
   cat /dev/null
   ```

3. `/dev/random` and `/dev/urandom` are random byte generators:

   ```bash
   head -c 16 /dev/urandom | xxd
   ```

   > `xxd` formats binary data as hexadecimal. It is useful for inspecting raw bytes.

---

## Part 2: File Metadata with stat

`ls -l` shows you some file metadata, but `stat` shows everything the kernel records about a file.

### Exercise 2.1 — Inspect a file with stat

Create a test file and inspect it:

```bash
mkdir ~/fs-lab && cd ~/fs-lab
echo "hello" > sample.txt
stat sample.txt
```

You will see output similar to:

```
  File: sample.txt
  Size: 6               Blocks: 8          IO Block: 4096   regular file
Device: 10302h/66306d   Inode: 1234567     Links: 1
Access: (0644/-rw-r--r--)  Uid: ( 1000/   alice)   Gid: ( 1000/   alice)
Access: 2025-01-15 10:00:00
Modify: 2025-01-15 10:00:01
Change: 2025-01-15 10:00:01
```

Note the **Inode** number — this is the file's unique identifier in the file system. The name is just a label pointing to the inode.

### Exercise 2.2 — Touch a file and see timestamps change

`touch` updates a file's access and modification timestamps without changing its content.

1. Run `stat` on the file and note the timestamps.

2. Touch the file:

   ```bash
   touch sample.txt
   ```

3. Run `stat` again. Which timestamps changed?

<details>
<summary>Reveal Answer</summary>

The **Access** and **Change** timestamps update. The **Modify** timestamp (when the content last changed) stays the same — you did not write any data.

</details>

---

## Part 3: Hard Links and Symbolic Links

Linux has two kinds of links. Understanding the difference is important.

### Exercise 3.1 — Create a hard link

A hard link is a second directory entry pointing to the same inode (the same underlying data). The file has no "original" — both names are equal.

1. Create a hard link:

   ```bash
   ln sample.txt sample-hard.txt
   ```

2. Compare their inode numbers:

   ```bash
   stat sample.txt sample-hard.txt | grep Inode
   ```

   They are identical — both names point to the same data.

3. Verify the link count has increased:

   ```bash
   stat sample.txt | grep Links
   ```

   It should now say `Links: 2`.

4. Edit the original and read the link — confirm they are in sync:

   ```bash
   echo "updated content" > sample.txt
   cat sample-hard.txt
   ```

5. Delete the original and confirm the data is still accessible via the link:

   ```bash
   rm sample.txt
   cat sample-hard.txt
   ls
   ```

   The data survives until the link count reaches zero.

### Exercise 3.2 — Create a symbolic link

A symbolic link (symlink) is different: it is a separate file that contains a *path* to another file. If the target is deleted, the symlink breaks.

1. Create a symlink:

   ```bash
   ln -s sample-hard.txt link-to-sample.txt
   ```

2. List with `ls -l` — symlinks are shown with `->`:

   ```bash
   ls -l
   ```

3. Check the inode numbers:

   ```bash
   stat sample-hard.txt link-to-sample.txt | grep Inode
   ```

   They are different — the symlink is its own file with its own inode.

4. Delete the target and try to read the symlink:

   ```bash
   rm sample-hard.txt
   cat link-to-sample.txt
   ```

   The symlink is now **broken** (dangling). List it again:

   ```bash
   ls -l link-to-sample.txt
   ```

   Many terminals highlight broken symlinks in red.

### Exercise 3.3 — Use a symlink to a directory

Symlinks are commonly used to create stable names for things that might change location — for example, pointing `/var/www/current` at a versioned deployment directory.

1. Create two directories representing deployment versions:

   ```bash
   mkdir release-1.0 release-1.1
   echo "v1.0" > release-1.0/index.html
   echo "v1.1" > release-1.1/index.html
   ```

2. Create a symlink `current` pointing to `release-1.0`:

   ```bash
   ln -s release-1.0 current
   cat current/index.html
   ```

3. "Deploy" version 1.1 by updating the symlink atomically:

   ```bash
   ln -sfn release-1.1 current
   cat current/index.html
   ```

   The `-f` flag overwrites the existing symlink; `-n` prevents following it as a directory. This pattern is used extensively in real deployment tooling.

---

## Part 4: File Types

Linux treats almost everything as a file. The first character in `ls -l` output tells you the file type.

| Character | Type |
|-----------|------|
| `-` | Regular file |
| `d` | Directory |
| `l` | Symbolic link |
| `c` | Character device |
| `b` | Block device |
| `p` | Named pipe (FIFO) |
| `s` | Socket |

### Exercise 4.1 — Identify file types in /dev

```bash
ls -l /dev | head -30
```

Find at least one example each of a character device (`c`) and a block device (`b`). Block devices represent storage (disks); character devices handle data streams (terminals, random number generators).

### Exercise 4.2 — Create a named pipe

A named pipe (FIFO) allows two processes to communicate. Data written to one end is read by the other — in order, with no intermediate file on disk.

1. Create a named pipe:

   ```bash
   mkfifo ~/fs-lab/mypipe
   ls -l ~/fs-lab/mypipe
   ```

   Notice the `p` at the start of the permissions.

2. Open a second terminal. In your original terminal, write to the pipe:

   ```bash
   echo "through the pipe" > ~/fs-lab/mypipe
   ```

   This will appear to hang — it is waiting for a reader.

3. In the second terminal, read from the pipe:

   ```bash
   cat ~/fs-lab/mypipe
   ```

   The message appears, and the first terminal unblocks.

---

## Part 5: Clean Up

```bash
cd ~
rm -r ~/fs-lab
```
