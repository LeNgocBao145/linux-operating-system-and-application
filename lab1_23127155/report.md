## Lab 1 — Map an unknown directory

![Screenshot output of tree ~/lab01 showing the full structure](screenshots/lab1_1.png)

![Screenshot directory count and full file-path listing](screenshots/lab1_2.png)

## Lab 2 — Stdout vs stderr — split the streams

![Screenshot cat good.txt — show they contain different data](screenshots/lab2_1.png)

![Screenshot cat bad.txt — show they contain different data](screenshots/lab2_2.png)

![Screenshot the tee command running, output appearing on screen and in the file](screenshots/lab2_3.png)

## Lab 3 — Find every TODO in a codebase

Repo: https://github.com/redis/redis.git

It has roughly 1838 files

![Screenshot at least 10 lines of matches showing filename, line number, and context](screenshots/lab3_1.png)

![Screenshot the file count from grep -l … | wc -l](screenshots/lab3_2.png)

## Lab 4 — Top 10 IPs in access.log

![Screenshot output of the top-10-IPs pipeline](screenshots/lab4_1.png)

![Screenshot output of the top-5 404 URLs pipeline](screenshots/lab4_2.png)

## Lab 5 — Build a request-rate report by hour

![Screenshot the 24-line hour-by-hour count](screenshots/lab5_1.png)

![Screenshot status-code breakdown with percentages](screenshots/lab5_2.png)

The single hour with the highest 5xx error count is: 10

The single hour with the lowest 5xx error count is: 21

## Lab 6 — Bulk-replace strings in config files

![Screenshot listing of the 5 files and cat of one of them before the replace](screenshots/lab6_1.png)

![Screenshot the find/xargs/sed command and a grep -r proving all 5 files changed](screenshots/lab6_2.png)

## Lab 7 — Disk-hog hunt & permission audit

![Screenshot of top-10 largest files in your home](screenshots/lab7_1.png)

![Screenshot of world-writable files under /tmp](screenshots/lab7_2.png)

## Lab 8 — Hard links, soft links, and the inode

![Screenshot ls -li showing inode numbers for all 3 files before deletion](screenshots/lab8_1.png)

![Screenshot cat hard.txt and cat soft.txt after deleting the original](screenshots/lab8_2.png)

## Lab 9 — Make your shell yours

alias rm='rm -i'

alias mv='mv -i'

alias cp='cp -i'

alias ..='cd ..'

alias meminfo='free -m -l -t'

alias cpuinfo='lscpu'

![Screenshot of mkcd test123](screenshots/lab9_1.png)

![Screenshot of history | tail -10](screenshots/lab9_2.png)

## Lab 10 — The 3 AM pager — find the bad deploy

![Screenshot your one-liner running in the terminal with the top-5 IP table visible](screenshots/lab10_1.png)

![Screenshot the hour-bucket showing when failures clustered](screenshots/lab10_2.png)

198.51.100.17 4 22/May/2026:04:06:59 22/May/2026:15:43:02

96.162.171.44 1 22/May/2026:07:05:07 22/May/2026:07:05:07

93.235.78.112 1 22/May/2026:23:36:59 22/May/2026:23:36:59

8.8.8.8 1 22/May/2026:10:59:23 22/May/2026:10:59:23

76.63.200.44 1 22/May/2026:17:39:04 22/May/2026:17:39:04
