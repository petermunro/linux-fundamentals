# Lab 05: I/O Redirection and Pipelines

**Objective:** Put standard streams, redirection operators, pipelines, process substitution, and `xargs` into practice. By the end, you will be able to wire Linux's tools together with precision — routing output to files, separating errors from results, and building multi-stage pipelines that process data without ever touching a temporary file.

---

## Setup

Create a working directory and the files you will use throughout this lab.

1. Create the directory:

   ```bash
   mkdir ~/redirect-lab && cd ~/redirect-lab
   ```

2. Create a web server access log:

   ```bash
   cat > access.log << 'EOF'
   192.168.1.10 GET /index.html 200 1823
   10.0.0.5 GET /about.html 200 512
   192.168.1.10 POST /login 302 0
   10.0.0.8 GET /missing.html 404 128
   192.168.1.15 GET /index.html 200 1823
   10.0.0.5 GET /index.html 200 1823
   192.168.1.10 GET /dashboard 200 4521
   10.0.0.8 POST /login 401 64
   192.168.1.20 GET /index.html 200 1823
   10.0.0.5 DELETE /user/42 403 64
   192.168.1.10 GET /index.html 200 1823
   10.0.0.8 GET /index.html 200 1823
   192.168.1.15 GET /about.html 200 512
   10.0.0.5 GET /dashboard 200 4521
   192.168.1.20 POST /login 302 0
   172.16.0.3 GET /api/users 200 2048
   172.16.0.3 POST /api/users 201 256
   172.16.0.3 PUT /api/users/7 200 256
   172.16.0.3 DELETE /api/users/7 204 0
   192.168.1.10 GET /about.html 200 512
   EOF
   ```

3. Create a controlled directory structure for the stderr exercises — one directory readable, one that is not:

   ```bash
   mkdir -p searchdir/visible searchdir/private
   echo "db_host=localhost" > searchdir/visible/app.conf
   chmod 000 searchdir/private
   ```

---

## Part 1: Redirecting stdout

### Exercise 1.1 — Capture output to a file

1. Run `ls /etc` normally and observe the output appearing on your terminal.
2. Run the same command again, this time redirecting stdout to a file called `etc-files.txt`. Nothing should appear on the terminal.
3. Verify the output went into the file by displaying the first 5 lines and counting its total lines with `wc -l`.

<details>
<summary>Reveal Solution</summary>

```bash
ls /etc
ls /etc > etc-files.txt
head -5 etc-files.txt
wc -l etc-files.txt
```

</details>

### Exercise 1.2 — Overwrite vs append

1. Create a file called `build.log` containing the line `Build started`.
2. Append two more lines to it — `Running tests` and `Tests passed` — without overwriting the first line.
3. Display the file to confirm all three lines are there.
4. Now use `>` to write a completely new message to `build.log`, then display it again to confirm the original content is gone.

<details>
<summary>Reveal Solution</summary>

```bash
echo "Build started" > build.log
echo "Running tests" >> build.log
echo "Tests passed" >> build.log
cat build.log
echo "Completely new content" > build.log
cat build.log
```

</details>

### Exercise 1.3 — Redirect stdin

1. Count the lines in `access.log` two ways: first by passing it as a filename argument to `wc -l`, then by redirecting it as stdin with `<`. Notice that the output format differs — the filename version includes the filename; the stdin version does not.
2. Combine `<` and `>` in a single command: sort `access.log` and save the result to `access-sorted.txt`. Verify with `head`.

<details>
<summary>Reveal Solution</summary>

```bash
wc -l access.log
wc -l < access.log
sort < access.log > access-sorted.txt
head -5 access-sorted.txt
```

</details>

---

## Part 2: stderr and `/dev/null`

### Exercise 2.1 — Observe the stdout/stderr split

Run `find` to search `searchdir` for any `*.conf` files:

```bash
find searchdir -name "*.conf"
```

You will see a file path (stdout) and a permission-denied error (stderr). Both appear on the terminal because by default both streams go there — but they are separate. The next exercises show how to route them independently.

### Exercise 2.2 — Redirect stderr

1. Run the same `find` command, this time sending stderr to `/dev/null` so only the result appears.
2. Run it again, redirecting stderr to a file called `find-errors.txt`. Verify the error was captured by displaying that file.

<details>
<summary>Reveal Solution</summary>

```bash
find searchdir -name "*.conf" 2> /dev/null
find searchdir -name "*.conf" 2> find-errors.txt
cat find-errors.txt
```

</details>

### Exercise 2.3 — Capture both streams

1. Run the `find` command redirecting stdout to `all-output.txt` and merging stderr into it with `2>&1`. Then display the file — it should contain both the file path and the error.
2. Now try the **wrong** order: put `2>&1` before `> wrong-order.txt`. Watch your terminal — the error message will appear there instead of in the file. Display `wrong-order.txt` and compare its contents with `all-output.txt`.

<details>
<summary>Reveal Solution</summary>

```bash
find searchdir -name "*.conf" > all-output.txt 2>&1
cat all-output.txt

find searchdir -name "*.conf" 2>&1 > wrong-order.txt
cat wrong-order.txt
```

`wrong-order.txt` contains only the file path. The error appeared on the terminal because `2>&1` was processed first, when stdout was still the terminal — the redirect to `wrong-order.txt` had not happened yet.

</details>

### Exercise 2.4 — The `&>` shorthand

1. Use `&>` to redirect both streams to `all-output2.txt` in a single step. Display the file.
2. Append a separator line `---` to the same file using `&>>`. Verify it was appended.

<details>
<summary>Reveal Solution</summary>

```bash
find searchdir -name "*.conf" &> all-output2.txt
cat all-output2.txt
echo "---" &>> all-output2.txt
cat all-output2.txt
```

</details>

### Exercise 2.5 — `/dev/null`

1. Read from `/dev/null` to confirm it returns nothing.
2. Run the `find` command with stdout silenced — stderr should still appear on the terminal.
3. Run it again with both streams silenced. Nothing should appear at all.

<details>
<summary>Reveal Solution</summary>

```bash
cat /dev/null
find searchdir -name "*.conf" > /dev/null
find searchdir -name "*.conf" > /dev/null 2>&1
```

</details>

---

## Part 3: Pipelines

### Exercise 3.1 — Build a pipeline step by step

1. Count how many entries are in `/etc` by piping `ls /etc` into `wc -l`.
2. Build a longer pipeline: list `/etc`, sort the output alphabetically, then take just the first 10 entries.

<details>
<summary>Reveal Solution</summary>

```bash
ls /etc | wc -l
ls /etc | sort | head -10
```

</details>

### Exercise 3.2 — Multi-stage pipeline

Build a pipeline to answer: **which IP addresses made error requests, and how many times?**

Build it one step at a time, verifying each stage before adding the next:

1. Filter the log to lines containing error responses (status codes starting with `4`).
2. Extract just the IP address from each matching line.
3. Add sorting and counting to rank the IPs.

<details>
<summary>Reveal Solution</summary>

```bash
grep " 4" access.log
grep " 4" access.log | cut -d' ' -f1
grep " 4" access.log | cut -d' ' -f1 | sort | uniq -c | sort -rn
```

Expected output for step 3:
```
      2 10.0.0.8
      1 10.0.0.5
```

</details>

### Exercise 3.3 — Save pipeline output

Find the most-visited successful (200-status) URLs and redirect the ranked result to a file called `popular-pages.txt`. Display the file to verify.

<details>
<summary>Reveal Solution</summary>

```bash
grep " 200 " access.log | cut -d' ' -f3 | sort | uniq -c | sort -rn > popular-pages.txt
cat popular-pages.txt
```

Expected output:
```
      5 /index.html
      2 /dashboard
      2 /about.html
      1 /api/users
      1 /api/users/7
```

</details>

### Exercise 3.4 — Challenge: top IP by successful requests

Using `grep`, `cut`, `sort`, `uniq`, and `>`, find which IP address made the most successful (200-status) requests and save the full ranked list to `success-counts.txt`.

<details>
<summary>Reveal Solution</summary>

```bash
grep " 200 " access.log | cut -d' ' -f1 | sort | uniq -c | sort -rn > success-counts.txt
cat success-counts.txt
```

Expected output:
```
      5 192.168.1.10
      3 10.0.0.5
      2 172.16.0.3
      2 192.168.1.15
      1 192.168.1.20
      1 10.0.0.8
```

</details>

---

## Part 4: Process Substitution

### Exercise 4.1 — Compare without temporary files

> **`diff` in brief:** `diff file1 file2` compares two files line by line. Lines prefixed with `<` are only in the first file; lines prefixed with `>` are only in the second.

Create two unsorted user lists:

```bash
cat > server1-users.txt << 'EOF'
alice
charlie
bob
diana
EOF

cat > server2-users.txt << 'EOF'
bob
alice
eve
charlie
EOF
```

1. The traditional approach — compare the sorted versions using temporary files, then clean them up:

   ```bash
   sort server1-users.txt > /tmp/sorted1.txt
   sort server2-users.txt > /tmp/sorted2.txt
   diff /tmp/sorted1.txt /tmp/sorted2.txt
   rm /tmp/sorted1.txt /tmp/sorted2.txt
   ```

2. Now do the same comparison in a single command using `<()` process substitution — no temporary files.

<details>
<summary>Reveal Solution for step 2</summary>

```bash
diff <(sort server1-users.txt) <(sort server2-users.txt)
```

Expected output:
```
4c4
< diana
---
> eve
```

`diana` is on server1 but not server2; `eve` is on server2 but not server1.

</details>

### Exercise 4.2 — `tee`: observe and save simultaneously

> **`tee` in brief:** `tee` reads from stdin and writes to **both** stdout and a named file — like a T-junction in a pipeline. This lets you observe what is flowing through while also saving it.

Filter the log to 404 errors. Using `tee`, save the matching lines to `errors-404.txt` while simultaneously counting them with `wc -l` — in a single pipeline, without running `grep` twice.

<details>
<summary>Reveal Solution</summary>

```bash
grep " 404 " access.log | tee errors-404.txt | wc -l
```

The count appears on the terminal. Check the file was also written:

```bash
cat errors-404.txt
```

</details>

### Exercise 4.3 — Fan out with `tee` and `>()`

Process `access.log` in two directions at once using `tee` with `>()`:

- Save all error lines (status starting with `4`) to `error-lines.txt`
- Save all unique IP addresses (one per line, sorted) to `unique-ips.txt`

Both files should be written in a single pipeline pass over the log.

<details>
<summary>Reveal Solution</summary>

```bash
cat access.log | tee >(grep " 4" > error-lines.txt) >(cut -d' ' -f1 | sort | uniq > unique-ips.txt) > /dev/null
```

Check both output files:

```bash
cat error-lines.txt
cat unique-ips.txt
```

Expected contents of `unique-ips.txt`:
```
10.0.0.5
10.0.0.8
172.16.0.3
192.168.1.10
192.168.1.15
192.168.1.20
```

</details>

---

## Part 5: `xargs`

### Exercise 5.1 — Why `xargs`?

1. Create two files: `remove-me-1.txt` and `remove-me-2.txt`.
2. Try to delete them by piping their names to `rm`. Observe that `rm` ignores stdin.
3. Use `xargs` to pass the names as arguments to `rm`. Verify the files are gone.

<details>
<summary>Reveal Solution</summary>

```bash
touch remove-me-1.txt remove-me-2.txt
echo "remove-me-1.txt remove-me-2.txt" | rm        # does nothing (or errors)
touch remove-me-1.txt remove-me-2.txt
echo "remove-me-1.txt remove-me-2.txt" | xargs rm
ls remove-me-*.txt 2>/dev/null || echo "files removed"
```

</details>

### Exercise 5.2 — `xargs` with `find`

Create a small directory of config files to work with:

```bash
mkdir configs
echo "host=db1" > configs/database.conf
echo "host=cache1" > configs/cache.conf
echo "port=8080" > configs/app.conf
```

1. Use `find` and `xargs` to count the lines in all `.conf` files at once.
2. Use `find`, `xargs`, and `grep` to list only the filenames that contain the word `host`.

<details>
<summary>Reveal Solution</summary>

```bash
find configs -name "*.conf" | xargs wc -l
find configs -name "*.conf" | xargs grep -l "host"
```

</details>

### Exercise 5.3 — Controlling batch size with `-n`

Use `xargs -n 2` to process the string `"a b c d e f"` two items at a time, passing each pair to `echo`. Then try `-n 1` to process one item at a time.

<details>
<summary>Reveal Solution</summary>

```bash
echo "a b c d e f" | xargs -n 2 echo
echo "a b c d e f" | xargs -n 1 echo
```

Expected output for `-n 2`:
```
a b
c d
e f
```

</details>

### Exercise 5.4 — Placing arguments with `-I`

Use `xargs -I {}` to create a `.bak` copy of each `.conf` file in the `configs` directory. The placeholder `{}` should appear twice in the command — once as source, once as destination.

<details>
<summary>Reveal Solution</summary>

```bash
find configs -name "*.conf" | xargs -I {} cp {} {}.bak
ls configs/
```

Expected output:
```
app.conf  app.conf.bak  cache.conf  cache.conf.bak  database.conf  database.conf.bak
```

</details>

### Exercise 5.5 — Safe filenames with `-0`

1. Create three files with spaces in their names:

   ```bash
   touch "report 2024.txt" "report 2025.txt" "meeting notes.txt"
   ```

2. Try to count the lines in them using `find | xargs wc -l`. Observe that `xargs` splits on spaces and treats each word as a separate filename, causing errors.

3. Fix the command using `find`'s `-print0` flag and `xargs`'s `-0` flag to use null bytes as the delimiter instead of whitespace.

<details>
<summary>Reveal Solution</summary>

```bash
# Broken:
find . -maxdepth 1 -name "*.txt" | xargs wc -l

# Fixed:
find . -maxdepth 1 -name "*.txt" -print0 | xargs -0 wc -l
```

`-print0` makes `find` separate results with a null byte (`\0`) instead of a newline. `-0` tells `xargs` to split on null bytes. Since filenames can never contain a null byte, this is unambiguous regardless of what other characters appear in the name.

</details>

### Exercise 5.6 — Parallel execution with `-P`

> **`seq` in brief:** `seq 1 4` prints the integers 1, 2, 3, 4 — one per line. It is useful for generating test data or driving loops.

> **`sh -c 'command'`** runs a shell command given as a string, allowing a multi-part command to be passed as a single unit.

Compare sequential and parallel execution by timing both runs:

1. Run four jobs one at a time, each sleeping half a second. Note the elapsed time.
2. Run the same four jobs with up to four running simultaneously. Note the elapsed time and whether the output order changes.

<details>
<summary>Reveal Solution</summary>

```bash
# Sequential (~2 seconds)
time seq 1 4 | xargs -P 1 -I {} sh -c 'sleep 0.5; echo "done: {}"'

# Parallel (~0.5 seconds)
time seq 1 4 | xargs -P 4 -I {} sh -c 'sleep 0.5; echo "done: {}"'
```

With `-P 4` all four jobs start at once. The total time drops from ~2 seconds to ~0.5 seconds. The output order may differ between runs because whichever process finishes first prints first.

</details>

---

## Part 6: Clean Up

Restore the permissions on `searchdir/private` before deleting (you cannot remove a directory you cannot read):

```bash
chmod 755 searchdir/private
cd ~
rm -r ~/redirect-lab
```
