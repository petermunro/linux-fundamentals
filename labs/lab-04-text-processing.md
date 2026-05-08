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

1. Display the contents of `access.log` using `cat`.
2. Display it again, this time with line numbers. Use `cat`'s `-n` flag.
3. Count the lines using `wc -l`. You should see `20`.
4. `wc` also accepts multiple filenames — count both `access.log` and `app.conf` in a single command.

<details>
<summary>Reveal Solution</summary>

```bash
cat access.log
cat -n access.log
wc -l access.log
wc -l access.log app.conf
```

</details>

### Exercise 1.2 — Count words, lines, and bytes

1. Run `wc` on `app.conf` without any flags. The three numbers are **lines**, **words**, and **bytes**.
2. Run `wc` three more times with the `-l`, `-w`, and `-c` flags to get each count separately.

<details>
<summary>Reveal Solution</summary>

```bash
wc app.conf
wc -l app.conf
wc -w app.conf
wc -c app.conf
```

</details>

### Exercise 1.3 — View the start and end of a file

1. Print the first 5 lines of `access.log` using `head`.
2. Print the last 3 lines using `tail`.

<details>
<summary>Reveal Solution</summary>

```bash
head -5 access.log
tail -3 access.log
```

</details>

### Exercise 1.4 — Page through a file with `less`

Open `access.log` with `less`:

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

1. Use `grep` to find all log entries with a `404` status code.
2. Find all `GET` requests.

<details>
<summary>Reveal Solution</summary>

```bash
grep "404" access.log
grep "GET" access.log
```

</details>

### Exercise 2.2 — Invert the match and count

1. Use `grep` to show every request that was *not* a GET.
2. Count how many non-GET requests there were using the `-c` flag.
3. Count the successful requests. Use spaces around `200` (i.e. `" 200 "`) to prevent false matches against IP addresses or response sizes.

<details>
<summary>Reveal Solution</summary>

```bash
grep -v "GET" access.log
grep -vc "GET" access.log
grep -c " 200 " access.log
```

</details>

### Exercise 2.3 — Line numbers and case-insensitive search

1. Find all POST requests, showing the line number of each match.
2. Search `app.conf` for the word "host", case-insensitively, so that `db_host`, `cache_host`, and any capitalised variants all match.

<details>
<summary>Reveal Solution</summary>

```bash
grep -n "POST" access.log
grep -i "host" app.conf
```

</details>

### Exercise 2.4 — Search across multiple files

First, create a second config file:

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

1. Search for the word `log` across all `.conf` files. `grep` will prefix each match with the filename.
2. Show only the filenames (not the matching lines) that contain the word `host`.

<details>
<summary>Reveal Solution</summary>

```bash
grep "log" *.conf
grep -l "host" *.conf
```

</details>

### Exercise 2.5 — Anchors

1. Find all comment lines in `app.conf` — lines that start with `#`. Use the `^` anchor.
2. Find all lines that end with `false`. Use the `$` anchor.
3. Find only the `db_` settings by anchoring the pattern to the start of the line.

<details>
<summary>Reveal Solution</summary>

```bash
grep "^#" app.conf
grep "false$" app.conf
grep "^db_" app.conf
```

Expected output for step 3:
```
db_host=localhost
db_port=5432
db_name=myapp_production
db_user=appuser
db_password=changeme
```

</details>

### Exercise 2.6 — Match error responses with a regular expression

Use `grep -E` (extended regex) to match all 4xx and 5xx error responses in the access log. The status code is the fourth space-separated field; use surrounding spaces in your pattern to avoid matching other numbers in the line.

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

`[45]` matches either `4` or `5`; `[0-9]{2}` matches exactly two digits.

</details>

---

## Part 3: Extracting and Counting

### Exercise 3.1 — Extract a column with `cut`

The access log is space-separated. Use `cut` to:

1. Extract just the IP addresses (field 1).
2. Extract the HTTP method (field 2).
3. Extract both the method and the path together (fields 2 and 3).

<details>
<summary>Reveal Solution</summary>

```bash
cut -d' ' -f1 access.log
cut -d' ' -f2 access.log
cut -d' ' -f2,3 access.log
```

</details>

### Exercise 3.2 — Extract from a colon-delimited file

`/etc/passwd` uses `:` as its delimiter. Its fields are: username, password placeholder, UID, GID, comment, home directory, and login shell.

1. List all usernames.
2. List usernames and their login shells together.

<details>
<summary>Reveal Solution</summary>

```bash
cut -d: -f1 /etc/passwd
cut -d: -f1,7 /etc/passwd
```

</details>

### Exercise 3.3 — Translate and delete characters with `tr`

`tr` reads from stdin only — it cannot accept a filename argument, so it is always used in a pipeline.

1. Extract the HTTP methods from the log and convert them all to lowercase.
2. Extract the URL paths from the log and strip all `/` characters from them using `tr -d`.
3. `tr -s` squeezes runs of a repeated character down to one. Use it to collapse the extra spaces in this string: `echo "too    many   spaces"`.

<details>
<summary>Reveal Solution</summary>

```bash
cut -d' ' -f2 access.log | tr 'A-Z' 'a-z'
cut -d' ' -f3 access.log | tr -d '/'
echo "too    many   spaces" | tr -s ' '
```

Expected output for step 2 (first few lines):
```
index.html
about.html
login
missing.html
index.html
```

</details>

### Exercise 3.4 — Sort output

1. Extract the URL paths from the log and sort them alphabetically.
2. Extract the status codes and sort them numerically, so that `200` sorts before `404` rather than after it.

<details>
<summary>Reveal Solution</summary>

```bash
cut -d' ' -f3 access.log | sort
cut -d' ' -f4 access.log | sort -n
```

</details>

### Exercise 3.5 — Remove duplicates with `uniq`

`uniq` only removes adjacent duplicates, so you must sort the input first.

1. Get the unique paths visited in the log.
2. Get the unique status codes, sorted numerically.

<details>
<summary>Reveal Solution</summary>

```bash
cut -d' ' -f3 access.log | sort | uniq
cut -d' ' -f4 access.log | sort -n | uniq
```

</details>

### Exercise 3.6 — Count occurrences with `uniq -c`

`uniq -c` prefixes each unique line with a count of how many times it appeared.

Count how often each IP address appears in the log, then sort the result largest-first to find the most active client.

<details>
<summary>Reveal Solution</summary>

```bash
cut -d' ' -f1 access.log | sort | uniq -c | sort -rn
```

Expected output:
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

### Exercise 3.7 — Challenge: count HTTP methods

Use `cut`, `sort`, and `uniq -c` to count how many times each HTTP method (GET, POST, PUT, DELETE) appears in the log, ranked most-frequent first.

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

1. Use `sed` to replace `GET` with `[GET]` throughout the log output. Confirm the file itself is unchanged by checking the first three lines with `head`.
2. Note that `sed` only replaces the first match per line by default. Try adding the `g` flag to your substitution to replace every occurrence on each line.

<details>
<summary>Reveal Solution</summary>

```bash
sed 's/GET/[GET]/' access.log
sed 's/GET/[GET]/g' access.log
head -3 access.log    # file unchanged
```

</details>

### Exercise 5.2 — Use a different delimiter

The `/` in `s/find/replace/` is a convention, not a requirement. When your pattern or replacement contains slashes (as file paths do), switching the delimiter avoids messy escaping.

Replace `/index.html` with `/home`. Choose a delimiter other than `/`.

<details>
<summary>Reveal Solution</summary>

```bash
sed 's|/index.html|/home|' access.log
```

Any character can be used as the delimiter — `|`, `,`, `#`, etc.

</details>

### Exercise 5.3 — Delete lines by pattern

1. Print the log with all lines containing `40` (error responses) removed.
2. Remove the comment lines from `app.conf` (lines starting with `#`).
3. Chain a second `sed` command to also remove the blank lines.

<details>
<summary>Reveal Solution</summary>

```bash
sed '/40/d' access.log
sed '/^#/d' app.conf
sed '/^#/d' app.conf | sed '/^$/d'
```

Expected output for step 3:
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

By default `sed` applies its command to every line. An address restricts it.

1. Apply a substitution (replace `GET` with `---`) only to lines 1 through 5.
2. Apply a substitution only to lines matching the pattern `^db_host` — update that host to `db.internal`, leaving `cache_host` untouched.

<details>
<summary>Reveal Solution</summary>

```bash
sed '1,5s/GET/---/' access.log
sed '/^db_host/s/localhost/db.internal/' app.conf
```

In step 2, only the `db_host` line changes. `cache_host=localhost` is untouched because it does not match the address `/^db_host/`.

</details>

### Exercise 5.5 — Edit a file in place

Make a backup copy of `app.conf`, then use `sed -i` to change `log_level` from `info` to `warn` directly in the file. Verify the change took effect, and verify the backup is unchanged.

> **macOS note:** Use `sed -i '' 's/...' file` (empty string after `-i`) instead of just `-i`.

<details>
<summary>Reveal Solution</summary>

```bash
cp app.conf app.conf.orig
sed -i 's/log_level=info/log_level=warn/' app.conf
grep "log_level" app.conf
grep "log_level" app.conf.orig
```

</details>

---

## Part 6: Field Processing with `awk`

### Exercise 6.1 — Print specific fields

`awk` splits each line into fields automatically (whitespace-separated by default). Fields are `$1`, `$2`, etc.; `$0` is the whole line; `$NF` is the last field; `NR` is the current line number.

1. Print just the IP address (field 1) from each line.
2. Print the IP address and status code side by side.
3. Print the line number, method, and path for each request.

<details>
<summary>Reveal Solution</summary>

```bash
awk '{print $1}' access.log
awk '{print $1, $4}' access.log
awk '{print NR, $2, $3}' access.log
```

</details>

### Exercise 6.2 — Use a custom field separator

1. Use `awk -F:` to print usernames and home directories from `/etc/passwd` (fields 1 and 6).
2. Use `=` as the separator to print just the setting name (the part before `=`) from each line of `app.conf`. Note that comment and blank lines will appear — you will filter them in the next exercise.

<details>
<summary>Reveal Solution</summary>

```bash
awk -F: '{print $1, $6}' /etc/passwd
awk -F= '{print $1}' app.conf
```

</details>

### Exercise 6.3 — Filter with a pattern

The `awk` pattern is evaluated for each line; the action only runs when it matches.

1. Print only the lines where the status code (field 4) is exactly `404`.
2. Print the IP address and URL for every `POST` request.
3. Print all requests where the response size (field 5) exceeds 1000 bytes, showing the status, size, and path.

<details>
<summary>Reveal Solution</summary>

```bash
awk '$4 == 404' access.log
awk '$2 == "POST" {print $1, $3}' access.log
awk '$5 > 1000 {print $4, $5, $3}' access.log
```

Expected output for step 3:
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

1. Print all lines from IPs in the `192.168.*` range using an `awk` regex pattern.
2. Print the method, path, and status for every successful API request — where the path contains `/api/` and the status is below 400.

<details>
<summary>Reveal Solution</summary>

```bash
awk '/^192\.168/' access.log
awk '/\/api\// && $4 < 400 {print $2, $3, $4}' access.log
```

Expected output for step 2:
```
GET /api/users 200
POST /api/users 201
PUT /api/users/7 200
DELETE /api/users/7 204
```

</details>

### Exercise 6.5 — BEGIN and END blocks

1. Print a header line reading `IP Status` before the data, then print the IP and status for each request.
2. Calculate and print the total bytes transferred across all requests.

<details>
<summary>Reveal Solution</summary>

```bash
awk 'BEGIN {print "IP", "Status"} {print $1, $4}' access.log
awk '{total += $5} END {print "Total bytes:", total}' access.log
```

Expected output for step 2:
```
Total bytes: 24332
```

</details>

### Exercise 6.6 — Challenge: total bytes per IP address

Use `awk` to calculate the total response bytes sent to each IP address, then print the results sorted largest-first.

You will need an associative array to accumulate the bytes for each IP, and a loop in the `END` block to print the results.

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

`bytes[$1]` is an associative array keyed on the IP address. `+= $5` adds each response size to the running total. The `END` block then iterates over all keys with a `for` loop.

</details>

---

## Part 7: Clean Up

```bash
cd ~
rm -r ~/text-lab
```
