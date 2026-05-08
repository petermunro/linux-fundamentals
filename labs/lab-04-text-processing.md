# Lab 04: Text Processing and Filters

**Objective:** Get hands-on with Linux's text processing toolbox. You will view, search, extract, sort, and transform structured text — moving from simple inspection through to `sed` and `awk`, which together cover the vast majority of real-world text transformation tasks.

---

## Setup

Create a working directory and two sample files you will use throughout this lab.

1. Create the directory:

   ```bash
   mkdir ~/text-lab && cd ~/text-lab
   ```

2. Create a web server access log. Each line has five space-separated fields: IP address, HTTP method, URL path, status code, and response size in bytes.

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

3. Create an application configuration file:

   ```bash
   cat > app.conf << 'EOF'
   # Application Configuration
   # Environment: production

   db_host=localhost
   db_port=5432
   db_name=myapp_production
   db_user=appuser
   db_password=changeme

   cache_host=localhost
   cache_port=6379

   debug=false
   log_level=info
   max_connections=100
   EOF
   ```

> **Note:** The `cat > file << 'EOF' ... EOF` syntax is a convenient way to write multi-line text to a file. It uses shell redirection and heredoc syntax, which we will cover properly in Module 5.

---

## Part 1: Viewing and Inspecting Files

### Exercise 1.1 — Display file contents with `cat`

Display the access log:

```bash
cat access.log
```

Now display it with line numbers using the `-n` flag:

```bash
cat -n access.log
```

Count the lines using `wc -l`:

```bash
wc -l access.log
```

You should see `20`. `wc` also accepts multiple filenames — count both files at once:

```bash
wc -l access.log app.conf
```

### Exercise 1.2 — Count words, lines, and bytes

Run `wc` on `app.conf` without any flags:

```bash
wc app.conf
```

The three numbers are **lines**, **words**, and **bytes**. Try the individual flags to extract each:

```bash
wc -l app.conf    # lines only
wc -w app.conf    # words only
wc -c app.conf    # bytes only
```

### Exercise 1.3 — View the start and end of a file

Print the first 5 lines of the access log:

```bash
head -5 access.log
```

Print the last 3 lines:

```bash
tail -3 access.log
```

### Exercise 1.4 — Page through a file with `less`

Open the access log with `less`:

```bash
less access.log
```

Practice the following navigation:
- Press `G` to jump to the end of the file
- Press `g` to jump back to the beginning
- Type `/GET` and press Enter to search forward for `GET`
- Press `n` to jump to the next match
- Press `q` to quit

`less` becomes especially valuable with large files. Try `less /etc/passwd` to see it in context.

---

## Part 2: Searching with `grep`

### Exercise 2.1 — Basic search

Find all log entries that resulted in a 404 status:

```bash
grep "404" access.log
```

Find all GET requests:

```bash
grep "GET" access.log
```

### Exercise 2.2 — Invert the match

Show every request that was *not* a GET:

```bash
grep -v "GET" access.log
```

Count how many non-GET requests there were using the `-c` flag:

```bash
grep -vc "GET" access.log
```

Now count the successful requests. Using spaces around `200` prevents false matches against IP addresses or response sizes:

```bash
grep -c " 200 " access.log
```

### Exercise 2.3 — Line numbers and case-insensitive search

Show line numbers alongside matches. Find all POST requests with their line numbers:

```bash
grep -n "POST" access.log
```

Search `app.conf` for "host", case-insensitively (matching `db_host`, `cache_host`, etc.):

```bash
grep -i "host" app.conf
```

### Exercise 2.4 — Search across multiple files

Create a second config file:

```bash
cat > nginx.conf << 'EOF'
server {
    listen 80;
    server_name example.com;
    root /var/www/html;
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
}
EOF
```

Search for `log` across all `.conf` files. `grep` will prefix each match with the filename:

```bash
grep "log" *.conf
```

Show only the filenames that contain the word `host`:

```bash
grep -l "host" *.conf
```

### Exercise 2.5 — Anchors

Use `^` to match lines that start with `#` (comment lines) in `app.conf`:

```bash
grep "^#" app.conf
```

Use `$` to find lines that end with `false`:

```bash
grep "false$" app.conf
```

Find only the `db_` settings by anchoring to the start of the line:

```bash
grep "^db_" app.conf
```

<details>
<summary>Expected output</summary>

```
db_host=localhost
db_port=5432
db_name=myapp_production
db_user=appuser
db_password=changeme
```

</details>

### Exercise 2.6 — Match error responses with a regular expression

Use `grep -E` (extended regex) to match all 4xx and 5xx error responses in the access log.

<details>
<summary>Reveal Solution</summary>

```bash
grep -E " [45][0-9]{2} " access.log
```

Expected output:
```
10.0.0.8 GET /missing.html 404 128
10.0.0.8 POST /login 401 64
10.0.0.5 DELETE /user/42 403 64
```

`[45]` matches either `4` or `5`; `[0-9]{2}` matches exactly two digits. The surrounding spaces ensure we only match the status code field, not other numbers in the line.

</details>

---

## Part 3: Extracting and Counting

### Exercise 3.1 — Extract a column with `cut`

The access log is space-separated. Extract just the IP addresses (field 1):

```bash
cut -d' ' -f1 access.log
```

Extract the HTTP method (field 2):

```bash
cut -d' ' -f2 access.log
```

Extract both the method and the path (fields 2 and 3 together):

```bash
cut -d' ' -f2,3 access.log
```

### Exercise 3.2 — Extract from a colon-delimited file

`/etc/passwd` uses `:` as its delimiter. Its fields are: username, password placeholder, UID, GID, comment, home directory, and login shell.

List all usernames (field 1):

```bash
cut -d: -f1 /etc/passwd
```

List usernames and their login shells (fields 1 and 7):

```bash
cut -d: -f1,7 /etc/passwd
```

### Exercise 3.3 — Sort output

Extract the URL paths from the log and sort them alphabetically:

```bash
cut -d' ' -f3 access.log | sort
```

Sort the status codes numerically (so `200` sorts before `404`, not after it):

```bash
cut -d' ' -f4 access.log | sort -n
```

### Exercise 3.4 — Remove duplicates with `uniq`

`uniq` removes adjacent duplicate lines, so the input must be sorted first.

Get the unique paths visited:

```bash
cut -d' ' -f3 access.log | sort | uniq
```

Get the unique status codes that appear in the log:

```bash
cut -d' ' -f4 access.log | sort -n | uniq
```

### Exercise 3.5 — Count occurrences with `uniq -c`

`uniq -c` prefixes each line with how many times it appeared consecutively.

Count how often each IP address appears:

```bash
cut -d' ' -f1 access.log | sort | uniq -c
```

Sort the result largest-first to find the most active client:

```bash
cut -d' ' -f1 access.log | sort | uniq -c | sort -rn
```

<details>
<summary>Expected output</summary>

```
      5 192.168.1.10
      4 172.16.0.3
      4 10.0.0.5
      3 10.0.0.8
      2 192.168.1.20
      2 192.168.1.15
```

This is the **frequency count pattern**: `sort | uniq -c | sort -rn`. It works for any structured text.

</details>

### Exercise 3.6 — Challenge: count HTTP methods

Use `cut`, `sort`, and `uniq -c` to count how many times each HTTP method appears in the log.

<details>
<summary>Reveal Solution</summary>

```bash
cut -d' ' -f2 access.log | sort | uniq -c | sort -rn
```

Expected output:
```
     13 GET
      4 POST
      2 DELETE
      1 PUT
```

</details>

---

## Part 4: A Brief Look at `ed`

`ed` is the original Unix text editor, written in 1969. You will not use it day-to-day, but its commands are the direct ancestors of `grep` and `sed` — understanding them makes both tools click.

### Exercise 4.1 — Explore `ed`

Open the access log in `ed`:

```bash
ed access.log
```

`ed` prints the file size in bytes and waits silently for a command. Try each of the following, pressing Enter after each:

| Command | What it does |
|---------|-------------|
| `1p` | Print line 1 |
| `$p` | Print the last line |
| `3p` | Print line 3 |
| `1,5p` | Print lines 1 to 5 |
| `g/404/p` | Print all lines matching `404` |
| `q` | Quit |

The command `g/404/p` is **g**lobally search for a regular expression and **p**rint. That `g/re/p` pattern is literally where `grep` got its name — and it is what `sed` was modelled on.

---

## Part 5: Stream Editing with `sed`

### Exercise 5.1 — Basic substitution

Replace `GET` with `[GET]` in the log output. The file is not changed — `sed` writes to stdout:

```bash
sed 's/GET/[GET]/' access.log
```

By default, only the first match per line is replaced. Add the `g` (global) flag to replace every occurrence:

```bash
sed 's/GET/[GET]/g' access.log
```

Check that the file is unchanged:

```bash
head -3 access.log
```

### Exercise 5.2 — Use a different delimiter

The `/` in `s/find/replace/` is just a convention — you can use any character. When the pattern or replacement contains slashes (common with file paths), this avoids messy escaping.

Replace `/index.html` with `/home` using `|` as the delimiter:

```bash
sed 's|/index.html|/home|' access.log
```

### Exercise 5.3 — Delete lines by pattern

Print the log with all error responses (lines containing `40`) removed:

```bash
sed '/40/d' access.log
```

Remove the comment lines from `app.conf` (lines starting with `#`):

```bash
sed '/^#/d' app.conf
```

Chain two `sed` commands to also remove the blank lines:

```bash
sed '/^#/d' app.conf | sed '/^$/d'
```

<details>
<summary>Expected output</summary>

```
db_host=localhost
db_port=5432
db_name=myapp_production
db_user=appuser
db_password=changeme
cache_host=localhost
cache_port=6379
debug=false
log_level=info
max_connections=100
```

</details>

### Exercise 5.4 — Restrict to an address

By default, `sed` applies its command to every line. An address restricts it to specific lines.

Apply a substitution only to lines 1 through 5:

```bash
sed '1,5s/GET/---/' access.log
```

Apply a substitution only to lines that match a pattern — update only the `db_` host setting, not the `cache_host`:

```bash
sed '/^db_host/s/localhost/db.internal/' app.conf
```

<details>
<summary>Expected output (relevant lines)</summary>

```
db_host=db.internal
```

Only the `db_host` line changes. `cache_host=localhost` is untouched because it does not match the address `/^db_host/`.

</details>

### Exercise 5.5 — Edit a file in place

So far, `sed` has only written to stdout. Use `-i` to modify the file directly.

First, make a backup:

```bash
cp app.conf app.conf.orig
```

Change `log_level` from `info` to `warn`:

```bash
sed -i 's/log_level=info/log_level=warn/' app.conf
```

Verify the change:

```bash
grep "log_level" app.conf
```

Verify the backup is unchanged:

```bash
grep "log_level" app.conf.orig
```

> **macOS note:** On macOS, `-i` requires an extension argument. Use `sed -i '' 's/...' file` (empty string, no space) instead of just `-i`.

---

## Part 6: Field Processing with `awk`

### Exercise 6.1 — Print specific fields

`awk` splits each line into fields automatically. Whitespace is the default separator. Fields are `$1`, `$2`, etc. `$0` is the whole line; `$NF` is the last field.

Print just the IP address (field 1):

```bash
awk '{print $1}' access.log
```

Print the IP address and the status code side by side:

```bash
awk '{print $1, $4}' access.log
```

Print the line number, method, and path using the built-in `NR` variable:

```bash
awk '{print NR, $2, $3}' access.log
```

### Exercise 6.2 — Use a custom field separator

`awk -F` sets the field separator, much like `cut -d`.

Print usernames and home directories from `/etc/passwd` (fields 1 and 6):

```bash
awk -F: '{print $1, $6}' /etc/passwd
```

Print just the setting name (the part before `=`) from `app.conf`:

```bash
awk -F= '{print $1}' app.conf
```

This includes comment and blank lines — you will filter those out in the next exercise.

### Exercise 6.3 — Filter with a pattern

The pattern in `awk` is evaluated for each line; the action only runs when it matches.

Print only the lines where the status code (field 4) is exactly `404`:

```bash
awk '$4 == 404' access.log
```

Print the IP and URL for every POST request:

```bash
awk '$2 == "POST" {print $1, $3}' access.log
```

Print all requests where the response size (field 5) is larger than 1000 bytes:

```bash
awk '$5 > 1000 {print $4, $5, $3}' access.log
```

<details>
<summary>Expected output</summary>

```
200 1823 /index.html
200 1823 /index.html
200 1823 /index.html
200 4521 /dashboard
200 1823 /index.html
200 1823 /index.html
200 1823 /index.html
200 4521 /dashboard
200 2048 /api/users
```

</details>

### Exercise 6.4 — Filter with a regex pattern

Match lines where the IP begins with `192.168`:

```bash
awk '/^192\.168/' access.log
```

Print the method, path, and status for every successful API request (path contains `/api/`, status below 400):

```bash
awk '/\/api\// && $4 < 400 {print $2, $3, $4}' access.log
```

<details>
<summary>Expected output</summary>

```
GET /api/users 200
POST /api/users 201
PUT /api/users/7 200
DELETE /api/users/7 204
```

</details>

### Exercise 6.5 — BEGIN and END blocks

Add a header line before the output using `BEGIN`:

```bash
awk 'BEGIN {print "IP", "Status"} {print $1, $4}' access.log
```

Calculate the total bytes transferred across all requests using `END`:

```bash
awk '{total += $5} END {print "Total bytes:", total}' access.log
```

<details>
<summary>Expected output</summary>

```
Total bytes: 24332
```

</details>

### Exercise 6.6 — Challenge: total bytes per IP address

Use `awk` to calculate the total response bytes sent to each IP address. Print the results sorted largest-first.

This requires an associative array (a variable indexed by a string) to accumulate bytes for each IP, then a loop in the `END` block to print the results.

<details>
<summary>Reveal Solution</summary>

```bash
awk '{bytes[$1] += $5} END {for (ip in bytes) print bytes[ip], ip}' access.log | sort -rn
```

Expected output:
```
8679 192.168.1.10
6920 10.0.0.5
2560 172.16.0.3
2335 192.168.1.15
2015 10.0.0.8
1823 192.168.1.20
```

`bytes[$1]` is an associative array keyed on the IP address. `+= $5` adds the response size to the running total for that IP. The `END` block iterates over all keys with a `for` loop and prints each total.

</details>

---

## Part 7: Clean Up

```bash
cd ~
rm -r ~/text-lab
```
