## Lab 1 — Map an unknown directory

Commands run:
```bash
mkdir -p ~/lab01/{src,test,docs}/{img,raw}
touch ~/lab01/{src,test,docs}/{img,raw}/empty.file
tree ~/lab01
find ~/lab01 -type d | wc -l
find ~/lab01 -type f
```

- `mkdir -p ~/lab01/{src,test,docs}/{img,raw}`: creates the directory tree under `~/lab01` using brace expansion. The `-p` flag makes parent directories as needed and ignores existing directories (no error if they exist).

- `touch ~/lab01/{src,test,docs}/{img,raw}/empty.file`: creates an empty file named `empty.file` in each leaf directory produced by the brace expansion; `touch` updates timestamps or creates files if they don't exist.

### Screenshots

#### Output of tree ~/lab01 showing the full structure
![Screenshot output of tree ~/lab01 showing the full structure](screenshots/lab1_1.png)

- `tree ~/lab01`: prints a recursive, tree-like listing of the directory contents. No flags were used; many `tree` implementations support options for depth or hidden files.

#### Directory count and full file-path listing
![Screenshot directory count and full file-path listing](screenshots/lab1_2.png)

- `find ~/lab01 -type d | wc -l`: `find` lists filesystem entries matching the predicate `-type d` (directories); the output is piped to `wc -l` which counts lines, giving the number of directories. Note this count includes the root directory itself.

- `find ~/lab01 -type f`: `find` with `-type f` lists all regular files under `~/lab01`, printing full paths.

### In report
Brace expansion is a mechanism in Bash that allows you to generate arbitrary strings. When the shell encounters a comma-separated list or a sequence inside curly braces {...}, it expands them from left to right, creating a combination of all elements.

```bash
mkdir -p ~/lab01/{src,test,docs}/{img,raw}
```

Bash processes the expansion mathematically like a Cartesian Product (multiplication) between the sets:

$$\{\text{src}, \text{test}, \text{docs}\} \times \{\text{img}, \text{raw}\}$$

1. **Step 1**: The shell looks at the first brace {src,test,docs} and prepares to generate 3 base paths: ~/lab01/src, ~/lab01/test, and ~/lab01/docs.

2. **Step 2**: It then moves left-to-right to the second brace {img,raw} and multiplies it across all 3 base paths.This results in `3 x 2 = 6` distinct directory paths generated instantly before the mkdir command even executes:

- ~/lab01/src/img
- ~/lab01/src/raw
- ~/lab01/test/img
- ~/lab01/test/raw
- ~/lab01/docs/img
- ~/lab01/docs/raw

#### The Role of the -p Flag
Without the -p (parents) flag, mkdir would crash because the parent directories (src, test, docs) do not exist yet when it tries to create the img and raw subfolders. The -p flag tells Linux to automatically create any missing parent directories along the path, allowing all 6 leaf folders to be built flawlessly in a single line.

## Lab 2 — Stdout vs stderr — split the streams

Commands run:
```bash
ls /etc /nonexistent > good.txt 2> bad.txt
ls /etc /nonexistent > all.txt 2>&1
ls /etc /nonexistent 2>/dev/null
uptime | tee uptime.txt
cat good.txt
cat bad.txt
cat uptime.txt
```

### Explanation

- `ls /etc /nonexistent > good.txt 2> bad.txt`: runs `ls` on two paths. `> good.txt` redirects stdout into `good.txt`. `2> bad.txt` redirects stderr (file descriptor 2) into `bad.txt`. Errors from the nonexistent path go to `bad.txt` while normal output goes to `good.txt`.
- `ls /etc /nonexistent > all.txt 2>&1`: redirects stdout to `all.txt`, then `2>&1` redirects stderr to the same place as stdout (the `&1` means file descriptor 1). The order matters: doing `2>&1` after `>all.txt` ensures both streams go into `all.txt`.
- `ls /etc /nonexistent 2>/dev/null`: redirects stderr to `/dev/null` (discarding error messages) while leaving stdout on the terminal.
- `uptime | tee uptime.txt`: pipes the output of `uptime` into `tee`, which writes the output to both the terminal and the file `uptime.txt` (without losing the interactive display).
- `cat good.txt` / `cat bad.txt` / `cat uptime.txt`: print the contents of each file to the terminal for inspection.

### Screenshots

#### cat good.txt — show they contain different data
![Screenshot cat good.txt — show they contain different data](screenshots/lab2_1.png)

#### cat bad.txt — show they contain different data
![Screenshot cat bad.txt — show they contain different data](screenshots/lab2_2.png)

#### The tee command running, output appearing on screen and in the file
![Screenshot the tee command running, output appearing on screen and in the file](screenshots/lab2_3.png)

### In report
In Linux, standard data streams are managed using file descriptors (numeric IDs):

`0`: Standard Input (`stdin`)

`1`: Standard Output (`stdout`) — The default target for normal command output.

`2`: Standard Error (`stderr`) — The default target for error messages.

The syntax `2>&1` means: **"Redirect Standard Error (`2`) to the same destination where Standard Output (`1`) is currently pointing."** The ampersand (`&`) is crucial because it tells the shell that `1` is a file descriptor, not a literal file named "1".

#### Why Order Matters
Redirection operators are processed by the shell from left to right before the command even executes. The shell duplicates the file descriptors sequentially, creating a massive difference between the two expressions.

Case A: `cmd > f 2>&1` (Combines both streams into file f)
1. Initially: Both 1 and 2 point to the screen (Terminal).

2. `> f` **handles stream 1**: Stream 1 (`stdout`) is redirected to point directly to file f.

3. 2>&1 **handles stream 2**: Stream 2 (`stderr`) looks at where stream 1 is pointing at that exact moment (which is file f) and copies that destination.

Result: Both `stdout` and `stderr` point to file f. All normal outputs and error messages are successfully logged together inside the file.

Case B: `cmd 2>&1 > f` (Separates the streams — Errors stay on screen!)
1. **Initially**: Both 1 and 2 point to the screen (Terminal).

2. `2>&`1 **handles stream 2**: Stream `2` (`stderr`) looks at where stream 1 is pointing at that exact moment (which is the screen) and copies that destination. (Essentially doing nothing since it already pointed to the screen).

3. `> f` **handles stream 1**: Stream `1` (`stdout`) is redirected away from the screen to point to file f.

Result: Stream 1 (`stdout`) goes into the file f, but stream 2 (`stderr`) is left pointing to the screen. Any error messages will pop up on your terminal and will not be saved in the file.

## Lab 3 — Find every TODO in a codebase

Repo: `https://github.com/redis/redis.git`

It has roughly `1838` files.

Commands run:
```bash
grep -rnI -B 1 -E --exclude-dir={node_modules,.git} 'TODO|FIXME|XXX' redis/
grep -nli --exclude-dir={node_modules,.git} 'TODO' redis/ | wc -l
```

### Screenshots

#### At least 10 lines of matches showing filename, line number, and context
![Screenshot at least 10 lines of matches showing filename, line number, and context](screenshots/lab3_1.png)

- `grep -rnI -B 1 -E --exclude-dir={node_modules,.git} 'TODO|FIXME|XXX' redis/`:
  - `grep` searches files for patterns.
  - `-r` recursively searches directories.
  - `-n` shows matching line numbers.
  - `-I` ignores binary files.
  - `-B 1` includes 1 line of context before each match.
  - `-E` enables extended regular expressions so the alternation `|` works.
  - `--exclude-dir={node_modules,.git}` skips those directories to avoid noise.
  - `'TODO|FIXME|XXX'` is the pattern to match.

#### The file count from grep -l … | wc -l
![Screenshot the file count from grep -l … | wc -l](screenshots/lab3_2.png)

- `grep -nli --exclude-dir={node_modules,.git} 'TODO' redis/ | wc -l`:
  - `-l` lists filenames only for matches.
  - `-i` makes the search case-insensitive.
  - Piping into `wc -l` counts the number of files that contain `TODO`.

The search surfaced comments in source files and third-party code, plus a few TODO-style markers in metadata like `.gitignore`. I counted 49 distinct files containing at least one TODO in the repository subset I searched. 

### In report
- **File**: src/config.c

- **Target Function**: void configSetCommand(client *c)

- **The Code Snippet**:
```C
void configSetCommand(client *c) {
    const char *errstr = NULL;
    const char *invalid_arg_name = NULL;
    const char *err_arg_name = NULL;
    standardConfig **set_configs; /* TODO: make this a dict for better performance */
    const char **config_names;
    sds *new_values;
    // ...
```

**What to Fix:**
I would refactor the set_configs pointer array into an associative dictionary (specifically using Redis’s native dict structure defined in dict.h), rather than keeping it as a dynamically allocated array (standardConfig ) that requires sequential lookups.

**Why Fix It:**
1. **Performance Optimization** ($O(N)$ to $O(1)$): Currently, when a user executes a massive CONFIG SET command with multiple parameters, Redis has to look through or build an array sequentially to keep track of configurations being modified. This results in a time complexity of $O(N)$ for lookups, where $N$ is the number of configs. Changing this to a hash dictionary drops lookup time down to an average of $O(1)$.

2. **Scalability**: Redis has hundreds of configuration parameters, and modules can add their own custom configs at runtime. As the number of runtime configurations grows, dictionary lookups ensure that execution times remain flat and deterministic.

3. **Code Cleanliness**: Utilizing a dict allows developers to leverage existing Redis dictionary helper API functions (like dictFind or dictFetchValue), which eliminates boilerplate array-looping code and reduces potential indexing bugs during multi-argument parsing.

## Lab 4 — Top 10 IPs in access.log

Commands run:
```bash
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10
awk '$9==404 {print $7}' access.log | sort | uniq -c | sort -rn | head -5
```

### Screenshots

#### Output of the top-10-IPs pipeline
![Screenshot output of the top-10-IPs pipeline](screenshots/lab4_1.png)

- `awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10`:
  - `awk '{print $1}'` prints the first whitespace-separated field (typically the client IP) from each log line.
  - `sort` sorts IPs so `uniq -c` can count consecutive identical lines.
  - `uniq -c` collapses duplicates and prefixes counts.
  - `sort -rn` sorts numerically in reverse (largest counts first).
  - `head -10` shows the top 10 IPs by request count.

#### Output of the top-5 404 URLs pipeline
![Screenshot output of the top-5 404 URLs pipeline](screenshots/lab4_2.png)

- `awk '$9==404 {print $7}' access.log | sort | uniq -c | sort -rn | head -5`:
  - `awk '$9==404 {print $7}'` filters log lines where the 9th field (status code) equals `404`, printing the requested path (7th field).
  - The same `sort | uniq -c | sort -rn | head -5` pipeline produces the top 5 most frequent 404 URLs.

### In report
The top-IP list is heavily skewed, with `203.0.113.42` far ahead of the rest, which looks more like automated traffic than normal browsing. The 404 list is even more telling: paths like `/wp-admin`, `/wp-login.php`, `/.env`, `/admin`, and `/phpmyadmin` are classic probing targets. That usually points to bots or scanners rather than real users.

## Lab 5 — Build a request-rate report by hour

Commands run:
```bash
awk '{ split($4, a, ":"); hours[a[2]]++ } END { for (i in hours) print i, hours[i] }' access.log | sort -n
awk '{ total++; status[$9]++ } END { for (i in status) printf "%s %d %.1f%%\n", i, status[i], (status[i] / total) * 100 }' access.log | sort -n
awk '$9 ~ /^5/ { split($4, a, ":"); count[a[2]]++ } END { for (i in count) print i, count[i] }' access.log | sort -k2 -n
```

- `awk '$9 ~ /^5/ { split($4, a, ":"); count[a[2]]++ } END { for (i in count) print i, count[i] }' access.log | sort -k2 -n`:
  - Filters only 5xx status codes (`$9 ~ /^5/`), buckets them by hour (same `split` trick), then prints hour/count.
  - `sort -k2 -n` sorts primarily by the count field numerically.

### Screenshots

#### The 24-line hour-by-hour count
![Screenshot the 24-line hour-by-hour count](screenshots/lab5_1.png)

- `awk '{ split($4, a, ":"); hours[a[2]]++ } END { for (i in hours) print i, hours[i] }' access.log | sort -n`:
  - `split($4, a, ":")` splits the 4th field (typically the timestamp like `[22/May/2026:19:00:01`) on `:` into array `a`.
  - `a[2]` is the hour component; `hours[a[2]]++` counts occurrences per hour.
  - The `END` block prints hour and count; `sort -n` orders hours numerically.

#### Status-code breakdown with percentages
![Screenshot status-code breakdown with percentages](screenshots/lab5_2.png)

- `awk '{ total++; status[$9]++ } END { for (i in status) printf "%s %d %.1f%%\n", i, status[i], (status[i] / total) * 100 }' access.log | sort -n`:
  - Counts total requests and groups by status code (`$9`).
  - The `printf` formats each status code with its count and percentage of the total.
  - `sort -n` orders by status code number.

### In report
The single hour with the highest 5xx error count is 10:00

```bash
ldnbao145@linux-lab:~$ awk '$9 ~ /^5/ { split($4, a, ":"); count[a[2]]++ } END { for (i in count) print i, count[i] }' access.log | sort -k2 -n
21 49
14 52
04 54
15 54
01 55
09 58
17 59
12 62
16 62
23 62
02 63
07 63
08 63
18 63
05 64
13 64
03 66
20 66
06 68
22 69
11 72
00 73
19 73
10 75
```

Based on the aggregated data from access.log, here is the comparative analysis of the overall traffic peak versus the server-side error peaks ($5\text{xx}$ status codes):

- Peak Traffic Hour: 19:00 (with a maximum of 4370 total requests).

- Worst $5\text{xx}$ Error Hour: 10:00 (with a peak of 75 error responses).

**Are they the same?** No, they are not the same. The overall traffic peaks in the evening at 19:00, whereas the server failures spike earlier in the morning at 10:00.

**What does this mean for the system?**

Because the worst error hour does not align with the peak traffic hour, we can rule out simple traffic saturation (high volume overloading the server) as the primary root cause. Instead, this distinct pattern points toward two likely engineering scenarios:

1. **Application/Infrastructure Layer Issues**: The spike at 10:00 AM likely indicates an issue independent of user load. This could be a failed daily deployment, a third-party API dependency outage, or heavy automated internal tasks (like daily reports or scheduled database synchronization) that run at 10:00 AM and lock database rows.

2. **High Baseline Errors**: It is also worth noting that the $5\text{xx}$ errors remain relatively flat and high across almost every hour of the day (ranging from 49 to 75 errors hourly). This steady distribution implies a persistent code bug, unhandled edge cases in specific API endpoints, or regular network flakiness rather than a system collapsing under heavy concurrent user traffic.

## Lab 6 — Bulk-replace strings in config files

Commands run:
```bash
cd ~/lab06/configs
find . -type f -name '*.conf' -print0 | xargs -0 sed -i.bak 's/old-server.local/10.5.0.2/g'
grep -r '10.5.0.2' .
```

### Screenshots

#### Listing of the 5 files and cat of one of them before the replace
![Screenshot listing of the 5 files and cat of one of them before the replace](screenshots/lab6_1.png)

#### The find/xargs/sed command and a grep -r proving all 5 files changed
![Screenshot the find/xargs/sed command and a grep -r proving all 5 files changed](screenshots/lab6_2.png)

- `find . -type f -name '*.conf' -print0 | xargs -0 sed -i.bak 's/old-server.local/10.5.0.2/g'`:
  - `find . -type f -name '*.conf' -print0` finds regular files ending in `.conf` and prints the paths separated by NUL characters instead of newlines, which is safe for filenames containing spaces or line breaks.
  - `xargs -0` tells `xargs` to read NUL-separated input, so it matches the output from `find -print0` exactly and forwards the filenames to `sed` without splitting them on spaces.
  - `sed -i.bak 's/old-server.local/10.5.0.2/g'` edits files in-place, creating a `.bak` backup for each file before modification. The `s///g` substitution replaces all occurrences on each line.

- `grep -r '10.5.0.2' .`: recursively searches to confirm the replacement occurred.

### In report
**Why Use xargs Instead of a Shell for Loop?**

- **Performance via Parallelism**: A standard shell `for` loop executes commands sequentially (one after another). `xargs` can spawn multiple processes simultaneously using the `-P` (`--max-procs`) flag, significantly speeding up bulk operations on multi-core systems.

- **Efficiency (The Argument List Limit)**: A `for` loop forks a new process for every single item in the list. If you have 10,000 files, it runs `sed` 10,000 times. By default, `xargs` bundles as many arguments as possible into a single command invocation (up to the system's `ARG_MAX` limit), vastly reducing the overhead of process creation.

**When Does xargs Break?**

1. **Spaces and Special Characters in Filenames (The Default Delimiter Issue)**

By default, `xargs` uses whitespace (`spaces`, `tabs`, and `newlines`) as delimiters to separate arguments. If a file is named `my database.conf`, `xargs` will split this single filename into two separate arguments: `my` and `database.conf`. The underlying command (like `sed`) will then fail because neither file exists.
- **The Fix**: Use `find -print0` combined with `xargs -0`. This changes the delimiter to a null character (`\0`), which safely handles any spaces, quotes, or special characters in filenames.

2. **Quotes in Filenames (Single/Double Quotes)**

`xargs` has a strict default behavior where it tries to parse single (`'`) and double (`"`) quotes as grouping characters, rather than literal parts of a filename.
- **The Problem**: If an input stream contains a file named `report_'2026'.txt`, `xargs` will strip the quotes or throw an error like `xargs: unmatched single quote`.
- **The Fix**: Using the `-0` (null delimiter) flag also completely fixes this issue.

3. **Empty Input Streams (Unnecessary Execution)**

By default, if the preceding command in the pipeline returns absolutely nothing, `xargs` will still blindly execute the target command once without any arguments.
- **The Problem**: If `find` finds `0` files, `xargs rm` might run as a bare `rm` command, which can cause an error or unexpected behavior.
- **The Fix**: Use the `-r` (or `--no-run-if-empty`) flag. This tells `xargs` to exit immediately without running the command if the input is empty.

4. **Exceeding System Constraints (ARG_MAX)**

Although `xargs` is designed to mitigate the `Argument list too long` error by bundling arguments safely, it can still break if a single individual argument itself is larger than the system's buffer size, or if combined environment variables consume too much space.
- **The Fix**: Use the `-s` flag to manually set the maximum number of bytes per command line invocation if running into strict system limits.

5. **Resource Exhaustion via Aggressive Parallelism (-P)**

As analyzed before, setting the `-P` flag to an excessively high number ($X \gg \text{CPU Cores}$) will not break `xargs`'s internal logic, but it can break the system environment.
- **The Problem**: It can lead to severe context-switching overhead, trigger the Linux Out-Of-Memory (OOM) Killer due to RAM exhaustion, or bottleneck the Disk I/O queue, effectively freezing the process.

## Lab 7 — Disk-hog hunt & permission audit

Commands run:
```bash
find ~ -type f -printf '%s %p\n' | sort -rn | head -10 | numfmt --field=1 --to=iec
sudo find /tmp -type f -perm -o+w
```

### Screenshots

#### Top-10 largest files in your home
![Screenshot of top-10 largest files in your home](screenshots/lab7_1.png)

- `find ~ -type f -printf '%s %p\n' | sort -rn | head -10 | numfmt --field=1 --to=iec`:
  - `find ~ -type f -printf '%s %p\n'` prints file size in bytes (`%s`) followed by path (`%p`).
  - `sort -rn` sorts numerically in reverse, largest first.
  - `head -10` shows the top 10 largest files.
  - `numfmt --field=1 --to=iec` reformats the first whitespace-separated field (the size) into human-readable IEC units (KiB, MiB, etc.).

#### World-writable files under /tmp
![Screenshot of world-writable files under /tmp](screenshots/lab7_2.png)

- `sudo find /tmp -type f -perm -o+w`:
  - `sudo` runs the command with elevated privileges so it can inspect files owned by other users.
  - `find /tmp -type f -perm -o+w` lists regular files under `/tmp` that have the `others` write bit set (`-o+w`), i.e., world-writable files; these can be a security concern.

### In report
The `/tmp` audit returned no world-writable regular files in the screenshot, which is a good sign because it means there was nothing obvious to inspect further. A world-writable file would be dangerous because any local user could tamper with it, inject content, or replace it with something malicious.

**Why a World-Writable File is Potentially Dangerous**

In Linux file permissions, a world-writable file (indicated by a `w` in the final triplet of the permission string, or an octal mode like `777` or `666`) means that any user on the system, regardless of their privileges or intentions, can modify, overwrite, or delete the contents of that file.

This completely breaks the principle of **least privilege** and isolation. Security controls in Linux rely on the assumption that unprivileged users cannot touch critical system or application configurations. When a file becomes world-writable, it creates a massive vector for **privilege escalation**, where a low-privileged local user or a compromised service account can hijack the entire system.

---

**Realistic Attack Scenario: The Malicious Cron Job**

Imagine a scenario where a system administrator creates a maintenance script to clear logs and schedules it to run automatically every hour as the root superuser via `cron`.

```bash
# Location of the script
/opt/scripts/cleanup.sh
```

If the administrator accidentally misconfigures the permissions of this file to be world-writable (e.g., `chmod 777 /opt/scripts/cleanup.sh`), a low-privileged attacker who has initial access to the machine can execute the following exploit:

1. Inspection: The attacker scans the system for world-writable files and discovers `/opt/scripts/cleanup.sh`.

2. Injection: Instead of destroying the file, the attacker appends a malicious payload to the end of the script:

```bash
echo "echo 'attacker ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers" >> /opt/scripts/cleanup.sh
```

3. Execution: The attacker waits for the hourly `cron` job to run. Because `cron` executes this script with root privileges, the injected command runs as root.

4. Takeover: The payload successfully appends the attacker's username to the `/etc/sudoers` file. The attacker can now run `sudo su` and instantly gain permanent, unrestricted root control over the entire Linux server.

To safeguard against this, administrators must run audits to identify these files. You can find more details about why world-writable permissions are high-severity vulnerabilities and how to properly find and mitigate them by checking the official documentation on [CWE-732: Incorrect Permission Assignment for Critical Resource](https://cwe.mitre.org/data/definitions/732.html).

## Lab 8 — Hard links, soft links, and the inode

Commands run:
```bash
echo 'Day la noi dung goc' > original.txt
ln original.txt hard.txt
ln -s original.txt soft.txt
ls -li
rm -f original.txt
cat hard.txt
cat soft.txt
```

### Screenshots

#### ls -li showing inode numbers for all 3 files before deletion
![Screenshot ls -li showing inode numbers for all 3 files before deletion](screenshots/lab8_1.png)

- `echo 'Đây là nội dung gốc' > original.txt`: writes the given string into `original.txt`, creating the file.
- `ln original.txt hard.txt`: creates a hard link named `hard.txt` that points to the same inode as `original.txt`.
- `ln -s original.txt soft.txt`: creates a symbolic (soft) link `soft.txt` that points to the pathname `original.txt`.
- `ls -li`: lists files with inode numbers (`-i`) and long format (`-l`) so you can compare inodes and link counts.

#### cat hard.txt and cat soft.txt after deleting the original
![Screenshot cat hard.txt and cat soft.txt after deleting the original](screenshots/lab8_2.png)

- `rm -f original.txt`: force-removes the original filename; `-f` suppresses errors if the file is missing.
- `cat hard.txt`: shows that the hard link still contains the file contents because it points to the same inode.
- `cat soft.txt`: shows that the symlink breaks after removing the original name (it points to a pathname that no longer exists), demonstrating the difference between hard and symbolic links.

### In report
The inode numbers show that `original.txt` and `hard.txt` point to the same underlying file, while `soft.txt` is only a path reference. After deleting the original name, the hard link still works because the inode still has another directory entry pointing to it. The symlink breaks because it points to a pathname that no longer exists.

**Before Deleting original.txt**
```ascii
[ Soft Link ]                            [ Directory Entries ]         [ Inode System ]          [ Data Blocks ]
+-------------------+                    +-------------------+        +-----------------+        +------------------------+
|   soft.txt        | --(references ---> |   original.txt    | -----> | Inode 655407    | -----> |  "Đây là nội dung gốc" |
+-------------------+    to path)        +-------------------+        | (Links: 2)      |        +------------------------+
                                                                      +-----------------+
[ Hard Link ]                                                             ^
+-------------------+                                                     |
|   hard.txt        | ----------------------------------------------------+
+-------------------+
```

**After Deleting original.txt**

```ascii
[ Soft Link ]                                                          [ Inode System ]          [ Data Blocks ]
+-------------------+                                                 +-----------------+        +------------------------+
|   soft.txt        | --(references to path) ---> [ X ]               | Inode 655407    | -----> |  "Đây là nội dung gốc" |
+-------------------+                                                 | (Links: 1)      |        +------------------------+
                                                                      +-----------------+
[ Hard Link ]                                                             ^
+-------------------+                                                     |
|   hard.txt        | ----------------------------------------------------+
+-------------------+
```

## Lab 9 — Make your shell yours

Commands run:
```bash
source ~/.bashrc
mkcd test123
pwd
history | tail -10
```

### Explanation

- `source ~/.bashrc`: reads and executes commands from `~/.bashrc` in the current shell, applying aliases, functions, and environment variables without starting a new shell.

### My .bashrc Snippets:
```bash
export HISTTIMEFORMAT="%F %T "

alias rm='rm -i'
alias mv='mv -i'
alias cp='cp -i'
alias ..='cd ..'
alias meminfo='free -m -l -t'
alias cpuinfo='lscpu'

# mkcd: make directory and cd into it
mkcd() { mkdir -p "$1" && cd "$1"; }

# extract: universal archive extractor
extract() {
  case "$1" in
    *.tar.gz) tar xzf "$1" ;;
    *.tar.bz2) tar xjf "$1" ;;
    *.zip) unzip "$1" ;;
    *.gz) gunzip "$1" ;;
  esac
}
```

The aliases reduce repetitive typing and also make destructive commands safer by prompting before overwrite or delete.

### Screenshots

#### mkcd test123
![Screenshot of mkcd test123](screenshots/lab9_1.png)

```bash
mkcd() { mkdir -p "$1" && cd "$1"; }
```
- `mkcd test123`: a shell function defined in the `.bashrc` that typically runs `mkdir -p test123 && cd test123` — it creates the directory (and parents) and changes into it.

- `pwd`: prints the current working directory to confirm `mkcd` changed directories.

`mkcd` saved one step every time I needed to create a new folder and and immediately cd's into it. The screenshot confirms it changed into `test123` immediately. 

#### history | tail-10
![Screenshot of history | tail-10](screenshots/lab9_2.png)

- `history | tail -10`: shows the last 10 entries from shell command history; `tail -10` limits to the final 10 lines.

```bash
export HISTTIMEFORMAT="%F %T "
```

In Linux, the `HISTTIMEFORMAT` environment variable uses the standard time format specifiers from the `date` command (POSIX strftime) to record the timestamp of when each command was executed.

Specifically, `%F` and `%T` represent the following:

- `%F`: Displays the Year-Month-Day in the full format `YYYY-MM-DD` (e.g., `2026-05-29`). This is equivalent to writing `%Y-%m-%d`.

- `%T`: Displays the Hour:Minute:Second in a 24-hour format `HH:MM:SS` (e.g., `09:51:32`). This is equivalent to writing `%H:%M:%S`.

 `HISTTIMEFORMAT` makes history much more useful because I can see when each command actually ran, not just what it was.

## Lab 10 — The 3 AM pager — find the bad deploy

Commands run:
```bash
awk '$6 ~ /POST/ && $7 ~ /^\/api\/v1\/payment/ && $9 ~ /^5/ { count[$1]++; if (!first[$1]) first[$1]=$4; last[$1]=$4 } END { for (i in count) print i, count[i], first[i], last[i] }' access.log | sort -nr | head -5
awk '$6 ~ /POST/ && $7 ~ /^\/api\/v1\/payment/ && $9 ~ /^5/ { split($4, a, ":"); hours[a[2]]++ } END { for (i in hours) print i, hours[i] }' access.log | sort -n
awk '$6 ~ /POST/ && $7 ~ /^\/api\/v1\/payment/ && $9 ~ /^5/ { count[$1]++ } END { for (i in count) print count[i], i }' access.log | sort -nr | head -5
```

### Screenshots

#### My one-liner running in the terminal with the top-5 IP table visible
![Screenshot your one-liner running in the terminal with the top-5 IP table visible](screenshots/lab10_1.png)

- `awk '$6 ~ /POST/ && $7 ~ /^\/api\/v1\/payment/ && $9 ~ /^5/ { count[$1]++ } END { for (i in count) print count[i], i }' access.log | sort -nr | head -5`:
  - This shorter form tallies failures per source IP (`count[$1]++`) and prints the aggregate as `count IP` in the `END` block.
  - Piping to `sort -nr | head -5` shows the top 5 IPs by failure count, with the highest counts first.

#### The hour-bucket showing when failures clustered
![Screenshot the hour-bucket showing when failures clustered](screenshots/lab10_2.png)

- `awk '$6 ~ /POST/ && $7 ~ /^\/api\/v1\/payment/ && $9 ~ /^5/ { split($4, a, ":"); hours[a[2]]++ } END { for (i in hours) print i, hours[i] }' access.log | sort -n`:
  - Similar filter, but `split($4, a, ":")` extracts the hour from the timestamp and `hours[a[2]]++` buckets failures by hour.
  - The `END` block prints hour/count pairs; `sort -n` orders them numerically by hour.

The failures cluster in hour 23.

### In report

#### The one-liner pasted as code

```bash
awk '$6 ~ /POST/ && $7 ~ /^\/api\/v1\/payment/ && $9 ~ /^5/ { count[$1]++; if (!first[$1]) first[$1]=$4; last[$1]=$4 } END { for (i in count) print i, count[i], first[i], last[i] }' access.log | sort -nr | head -5
```

- `awk '$6 ~ /POST/ && $7 ~ /^\/api\/v1\/payment/ && $9 ~ /^5/ { count[$1]++; if (!first[$1]) first[$1]=$4; last[$1]=$4 } END { for (i in count) print i, count[i], first[i], last[i] }' access.log | sort -nr | head -5`:
  - The `awk` script filters log lines where the 6th field contains `POST`, the 7th field matches the request path `^/api/v1/payment`, and the 9th field (status) starts with `5` (server errors).
  - For each matching line it increments `count[$1]` (per-IP counter), records the first seen timestamp in `first[$1]` and updates `last[$1]` to the latest timestamp.
  - In the `END` block it prints `ip count first last` for each IP. The pipeline `sort -nr | head -5` shows the top 5 IPs by count (descending).

#### The result after run command
```text
198.51.100.17 4 22/May/2026:04:06:59 22/May/2026:15:43:02
96.162.171.44 1 22/May/2026:07:05:07 22/May/2026:07:05:07
93.235.78.112 1 22/May/2026:23:36:59 22/May/2026:23:36:59
8.8.8.8 1 22/May/2026:10:59:23 22/May/2026:10:59:23
76.63.200.44 1 22/May/2026:17:39:04 22/May/2026:17:39:04
```

The top offenders are headed by `198.51.100.17`, which shows four failing requests `POST /api/v1/payment` requests spanning from `22/May/2026:04:06:59` to `22/May/2026:15:43:02` and a much wider time span than the rest. The other IPs only appear once, so this looks like one repeated offender plus a few isolated failures rather than a single noisy burst.
Because all of these are 5xx responses on the payment endpoint, the first thing I would check is the payment service itself and any upstream dependency it calls before blaming the client.
