## Lab 01 - Populate a fresh server with users and groups

### Screenshots

#### Output of `id alice` and `id carol` — both users' group memberships visible

![Screenshot output of id alice and id carol — both users' group memberships visible](screenshots/lab1_1.png)

#### `grep -E "alice|bob|carol|dave|svcapp" /etc/passwd` — all 5 rows visible

![Screenshot grep -E "alice|bob|carol|dave|svcapp" /etc/passwd — all 5 rows visible](screenshots/lab1_2.png)

### In report

```bash
ldnbao145@linux-lab:~$ sudo useradd -r -s /usr/sbin/nologin svcapp
ldnbao145@linux-lab:~$ id svcapp
uid=998(svcapp) gid=998(svcapp) groups=998(svcapp)
```

## Lab 02 - The silent group-wipe trap

### Screenshots

#### `id alice` showing 4 groups, then `id alice` after the `-G` trap (1 group left)

![Screenshot id alice showing 4 groups, then id alice after the -G trap (1 group left)](screenshots/lab2_1.png)

#### The `/etc/shadow` entry for `alice` showing the `!` lock prefix, then after unlocking

![Screenshot the /etc/shadow entry for alice showing the ! lock prefix, then after unlocking](screenshots/lab2_2.png)

### In report

## Lab 03 - Read, write, execute — controlling file access

### Screenshots

#### `ls -la` showing `secret.txt` at `600` and `reports/` at `750` with `alice:dev` ownership

![Screenshot ls -la showing secret.txt at 600 and reports/ at 750 with alice:dev ownership](screenshots/lab3_1.png)

#### `umask` set to `027` and the resulting permissions on a newly created file and directory

![Screenshot umask set to 027 and the resulting permissions on a newly created file and directory](screenshots/lab3_2.png)

### In report

## Lab 04 - Special permission bits: shared directories and elevated execution

### Screenshots

#### `ls -ld /opt/teamshared` showing the `s` (SGID) and `t` (sticky) characters in the permission string

![Screenshot ls -ld /opt/teamshared showing the s (SGID) and t (sticky) characters in the permission string](screenshots/lab4_1.png)

#### Bob's failed `rm` attempt on `alice`'s file (Operation not permitted)

![Screenshot Bob's failed rm attempt on alice's file (Operation not permitted)](screenshots/lab4_2.png)

#### Output of the `SUID` binary search — at least 3 results visible

![Screenshot output of the `SUID` binary search — at least 3 results visible](screenshots/lab4_3.png)

What identity does it report?

```bash
alice@linux-lab:~$ bash /tmp/whotest.sh
Running as: alice
```

### In report

## Lab 05 - Fine-grained access control with ACLs

### Screenshots

#### Full `getfacl /opt/project` output showing user, group, mask, and default ACL entries

![Screenshot full getfacl /opt/project output showing user, group, mask, and default ACL entries](screenshots/lab5_1.png)

#### `getfacl` before and after `chmod g-x` — show the `#effective` comment change in the mask trap

![Screenshot getfacl before and after chmod g-x — show the #effective comment change in the mask trap](screenshots/lab5_2.png)

### In report

## Lab 06 - Grant surgical sudo access to a CI deploy user

### Screenshots

#### The allowed command succeeding and the denied command being rejected — both in the same terminal session

![Screenshot the allowed command succeeding and the denied command being rejected — both in the same terminal session](screenshots/lab6_1.png)

#### The `auth.log` entries showing an allowed `COMMAND` and a `DENIED` attempt with timestamps

![Screenshot the `auth.log` entries showing an allowed `COMMAND` and a `DENIED` attempt with timestamps](screenshots/lab6_2.png)

### In report

## Lab 07 - Incident response — compromised account containment

### Screenshots

#### `/etc/shadow` entry for `bob` showing the `!` lock prefix

![Screenshot `/etc/shadow` entry for `bob` showing the `!` lock prefix](screenshots/lab7_1.png)

#### `id bob` before containment (`sudo` + `docker`) and after (only `docker`)

![Screenshot `id bob` before containment (`sudo` + `docker`) and after (only `docker`)](screenshots/lab7_2.png)

#### Output of `last bob` showing login history

### In report

## Lab 08 - Survive SSH disconnections with tmux

### Screenshots

#### `tmux ls` showing the session alive after detaching, with the step counter number visible

![Screenshot `tmux ls` showing the session alive after detaching, with the step counter number visible](screenshots/lab8_1.png)

#### The 2-pane view: the counter loop running in one pane and `top` in the other

![Screenshot the 2-pane view: the counter loop running in one pane and `top` in the other](screenshots/lab8_2.png)

#### The window list `(Ctrl+B W)` showing both windows, with the renamed `monitor` window visible

![Screenshot the window list `(Ctrl+B W)` showing both windows, with the renamed `monitor` window visible](screenshots/lab8_3.png)

### In report

## Lab 09 - Diagnose and tame a runaway process

### Screenshots

#### `top` showing the python3 process near 100% CPU before renicing (NI = 0)

![Screenshot `top` showing the python3 process near 100% CPU before renicing (NI = 0)](screenshots/lab9_1.png)

#### `top` after `renice -n 19` showing the NI column changed to 19

![Screenshot `top` after `renice -n 19` showing the NI column changed to 19](screenshots/lab9_2.png)

#### The `jobs` listing and the bg/fg/kill cycle for the `sleep 999` job

![Screenshot the `jobs` listing and the bg/fg/kill cycle for the `sleep 999` job](screenshots/lab9_3.png)

### In report

## Lab 10 - Harden a multi-role development server from scratch

### Screenshots

#### `getfacl /opt/appdata` and `getfacl /var/log/applog/` showing complete ACL setup

![Screenshot `getfacl /opt/appdata` and `getfacl /var/log/applog/` showing complete ACL setup](screenshots/lab10_1.png)

#### Access verification: `alice` writing successfully, `carol` denied, `eve` denied — all in one terminal session

![Screenshot access verification: `alice` writing successfully, `carol` denied, `eve` denied — all in one terminal session](screenshots/lab10_2.png)

#### `sudo` tests — each role's allowed command succeeding and a disallowed command being rejected

![Screenshot `sudo` tests — each role's allowed command succeeding and a disallowed command being rejected](screenshots/lab10_3.png)

#### Security audit results: `UID=0` check, `world-writable` check, `SUID` check — all clean

![Screenshot security audit results: `UID=0` check, `world-writable` check, `SUID` check — all clean](screenshots/lab10_4.png)

### In report
