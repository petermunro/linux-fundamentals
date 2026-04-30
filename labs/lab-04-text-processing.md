# Lab 04: Text Processing and Standard Streams

**Objective:** Master redirection, pipelines, and Linux's text-processing tools. By the end, you will be able to combine commands into pipelines that filter, transform, and summarise data — one of the most productive skills in Linux.

---

## Setup

Create a working directory and a sample data file you will use throughout this lab:

```bash
mkdir ~/text-lab && cd ~/text-lab
```

Create a file of fictional web server access log entries:

```bash
cat > access.log << 'EOF'
192.168.1.10 GET /index.html 200
10.0.0.5 GET /about.html 200
192.168.1.10 POST /login 302
10.0.0.8 GET /missing.html 404
192.168.1.15 GET /index.html 200
10.0.0.5 GET /index.html 200
192.168.1.10 GET /dashboard 200
10.0.0.8 POST /login 401
192.168.1.20 GET /index.html 200
10.0.0.5 DELETE /user/42 403
192.168.1.10 GET /index.html 200
10.0.0.8 GET /index.html 200
192.168.1.15 GET /about.html 200
10.0.0.5 GET /dashboard 200
192.168.1.20 POST /login 302
EOF
```

---

## Part 1: Redirection

The shell connects programs using three standard streams:

- **stdin** (0) — input
- **stdout** (1) — normal output
- **stderr** (2) — error output

### Exercise 1.1 — Redirect stdout to a file

Save the output of `ls` to a file:

```bash
ls /etc > etc-listing.txt
cat etc-listing.txt | head -5
```

### Exercise 1.2 — Append to a file

`>` overwrites; `>>` appends:

```bash
echo "First line" > notes.txt
echo "Second line" >> notes.txt
cat notes.txt
```

Now try overwriting and confirm the first line is gone:

```bash
echo "Replacement" > notes.txt
cat notes.txt
```

### Exercise 1.3 — Redirect stderr

Commands write errors to stderr, not stdout. Try listing a directory that does not exist:

```bash
ls /nonexistent
```

Redirect stderr to a file:

```bash
ls /nonexistent 2> errors.txt
cat errors.txt
```

### Exercise 1.4 — Discard output

To silence a command completely — both stdout and stderr — redirect to `/dev/null`:

```bash
ls /nonexistent > /dev/null 2>&1
```

The `2>&1` means "redirect stderr to wherever stdout is currently going" (which is `/dev/null`). You will see this pattern everywhere in shell scripts and cron jobs.

### Exercise 1.5 — Redirect stdin

Use `<` to feed a file into a command as stdin. Count the lines in `access.log` by redirecting it into `wc`:

```bash
wc -l < access.log
```

---

## Part 2: Pipelines

A pipe (`|`) connects the stdout of one command to the stdin of the next. The shell sets this up — neither command knows or cares where its data comes from.

### Exercise 2.1 — Your first pipeline

List `/etc` and page through the output:

```bash
ls /etc | less
```

Press `q` to quit.

### Exercise 2.2 — Count files in a directory

Count how many entries are in `/etc` using `wc -l`:

```bash
ls /etc | wc -l
```

### Exercise 2.3 — Chain three commands

Find all unique IP addresses in the log file. The fields are space-separated; the IP is the first field.

1. Extract the first field with `cut`:

   ```bash
   cut -d' ' -f1 access.log
   ```

2. Sort the results:

   ```bash
   cut -d' ' -f1 access.log | sort
   ```

3. Remove duplicates with `uniq`:

   ```bash
   cut -d' ' -f1 access.log | sort | uniq
   ```

<details>
<summary>Reveal Solution</summary>

```bash
cut -d' ' -f1 access.log | sort | uniq
```

Expected output:
```
10.0.0.5
10.0.0.8
192.168.1.10
192.168.1.15
192.168.1.20
```

Note: `uniq` only removes **adjacent** duplicates, which is why `sort` must come first.

</details>

---

## Part 3: Searching with grep

`grep` filters lines by pattern. Its name comes from the `ed` editor command `g/re/p` — globally search for a regular expression and print matching lines.

### Exercise 3.1 — Basic search

Find all lines in the log that contain `404`:

```bash
grep 404 access.log
```

### Exercise 3.2 — Case-insensitive search

```bash
grep -i "get" access.log
```

### Exercise 3.3 — Count matches

Count how many requests returned a 200 status:

```bash
grep -c " 200$" access.log
```

The `$` anchors the pattern to the end of the line, preventing false matches on IP addresses.

### Exercise 3.4 — Invert the match

Show all lines that are NOT a GET request:

```bash
grep -v "GET" access.log
```

### Exercise 3.5 — Search recursively in files

Create a small directory with files to search:

```bash
mkdir config-files
echo "db_host=localhost" > config-files/database.conf
echo "db_port=5432" >> config-files/database.conf
echo "cache_host=localhost" > config-files/cache.conf
echo "debug=true" > config-files/app.conf
```

Find all files containing `localhost`:

```bash
grep -r "localhost" config-files/
```

Add `-l` to show only filenames (not matching lines):

```bash
grep -rl "localhost" config-files/
```

---

## Part 4: Sorting and Counting

### Exercise 4.1 — Sort text alphabetically

Sort the IP addresses from the log:

```bash
cut -d' ' -f1 access.log | sort
```

### Exercise 4.2 — Count how often each IP appears

`uniq -c` counts consecutive identical lines. Combined with `sort`, this gives you a frequency count:

```bash
cut -d' ' -f1 access.log | sort | uniq -c
```

### Exercise 4.3 — Sort numerically by count

The counts from `uniq -c` are the first field. Sort numerically in descending order to find the most frequent IP:

```bash
cut -d' ' -f1 access.log | sort | uniq -c | sort -rn
```

<details>
<summary>Reveal Solution and Expected Output</summary>

```bash
cut -d' ' -f1 access.log | sort | uniq -c | sort -rn
```

```
      5 192.168.1.10
      4 10.0.0.5
      3 10.0.0.8
      2 192.168.1.15
      1 192.168.1.20
```

`-r` reverses the sort (largest first); `-n` sorts numerically rather than lexicographically (so `10` sorts after `9`, not before `2`).

</details>

### Exercise 4.4 — Count unique HTTP status codes in the log

Use `cut`, `sort`, and `uniq -c` to produce a count of each HTTP status code that appears in the log. The status code is the fourth field.

<details>
<summary>Reveal Solution</summary>

```bash
cut -d' ' -f4 access.log | sort | uniq -c | sort -rn
```

Expected output:
```
      9 200
      2 302
      2 404
      1 401
      1 403
```

</details>

---

## Part 5: Transforming Text

### Exercise 5.1 — Extract specific fields with cut

`cut` slices columns out of text. `-d` sets the delimiter; `-f` selects the field(s).

Extract just the HTTP method (field 2) and URL (field 3):

```bash
cut -d' ' -f2,3 access.log
```

### Exercise 5.2 — Substitute text with sed

`sed` is a stream editor. Its most common use is substitution: `s/find/replace/`.

Replace all occurrences of `GET` with `[GET]` in the log:

```bash
sed 's/GET/[GET]/' access.log
```

This only replaces the first match per line. To replace all occurrences on each line, add the `g` flag:

```bash
sed 's/GET/[GET]/g' access.log
```

### Exercise 5.3 — Delete lines matching a pattern with sed

Delete all lines containing `404` from the output (the file is not modified):

```bash
sed '/404/d' access.log
```

### Exercise 5.4 — Field processing with awk

`awk` processes text field by field. Fields are accessed as `$1`, `$2`, etc. The whole line is `$0`.

Print the IP address and status code for each request:

```bash
awk '{print $1, $4}' access.log
```

### Exercise 5.5 — awk with a condition

Print only lines where the status code (field 4) is 200:

```bash
awk '$4 == 200 {print $1, $3}' access.log
```

### Exercise 5.6 — awk for aggregation

Sum a column: count the total number of requests per IP address.

```bash
awk '{count[$1]++} END {for (ip in count) print count[ip], ip}' access.log | sort -rn
```

<details>
<summary>Explanation</summary>

- `count[$1]++` — increments a counter for each unique value of field 1 (the IP address)
- `END { ... }` — runs after all lines have been processed
- The `for` loop prints each IP and its count

</details>

---

## Part 6: Building Pipelines

Now put the tools together on more realistic tasks.

### Exercise 6.1 — Find the top 3 most-visited URLs

<details>
<summary>Reveal Solution</summary>

```bash
cut -d' ' -f3 access.log | sort | uniq -c | sort -rn | head -3
```

Expected output:
```
      5 /index.html
      3 /login
      2 /about.html
```

</details>

### Exercise 6.2 — List all failed requests (status >= 400) with their IPs

<details>
<summary>Reveal Solution</summary>

```bash
awk '$4 >= 400 {print $4, $1, $3}' access.log | sort
```

Or using `grep`:

```bash
grep -E " (4[0-9]{2})$" access.log
```

</details>

### Exercise 6.3 — Count lines in all .conf files combined

Use `xargs` to pass a list of filenames to `wc`. `xargs` reads stdin and builds a command from it.

```bash
find config-files/ -name "*.conf" | xargs wc -l
```

### Exercise 6.4 — Search for a pattern across many files using xargs

Find all config files containing the word `host` and print only the filenames:

```bash
find config-files/ -name "*.conf" | xargs grep -l "host"
```

<details>
<summary>Why xargs instead of grep -r?</summary>

For small directory trees, `grep -r` is simpler. `xargs` becomes valuable when the list of files comes from a complex `find` command with multiple conditions — you cannot always pass that directly to `grep -r`.

</details>

---

## Part 7: Clean Up

```bash
cd ~
rm -r ~/text-lab
```
