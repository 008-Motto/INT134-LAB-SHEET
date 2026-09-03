# INT134 System Deployment
## Class 01 Package Management and Service Manager

### Lab Instructions

1. It is best to learn from experiment, read command help, and figure out the solution yourself.

2. Do not search for the answer on internet or use AI.

3. Do not ask or copy answers from your friend.

---

### Setup

1. Connect to `int134` vm with ssh.

2. Create a directory `~/int134/lab01/` under your home directory and change working directory to this directory.

3. Copy `/home/public/labs/lab01/answers.yaml` to `~/int134/lab01/answers.yaml`.

4. Modify `answers.yaml`: set your student ID and your full name.

---

### Submission

1. Answer in `answers.yaml` file. Each answer goes inside a `|` block, and the
   indentation matters — the answer lines sit two spaces further in than their key:

    ```yaml
    answers:
      T6: |
        your answer here
    ```

2. Test your work using `int134 check lab01`. Run it as often as you like.

    ```bash
    $ int134 check lab01

      PASS  CTRL+S is freed for history search
      FAIL  The test user exists
            hint: Step 10.
      ...

      7 of 24 checks passed
    ```

    You will see how many checks passed, not a score.

3. A step with a question to answer is marked **→ `T<n>`**, which is the key to fill
   in `answers.yaml`. Steps without a mark still count — most of them are checked on
   your machine instead.

&nbsp;

#### How to read this sheet

`<...>` hides the part you have to work out. Sometimes that is the command:

```bash
$ <...>
Listing... Done
```

and sometimes it is what the command prints:

```bash
$ id
uid=<...> gid=<...>
```

Everything not hidden is there to help you. The sample output shows you *where to
look* and *what shape the answer takes*, so you can tell a working system from a
broken one.

&nbsp;


---

### Part 1 - Basic Setup

`~/.bashrc` runs every time you start an interactive shell. Editing it does **not**
change the shell you are already in — open a new session, or run `source ~/.bashrc`,
before you check your work.

| Setting | What it does |
| :--- | :--- |
| `stty -ixon` | Frees `CTRL+S`, which the terminal otherwise uses to freeze output |
| `HISTSIZE` | How many commands are kept in memory for the current shell |
| `HISTFILESIZE` | How many lines are kept in `~/.bash_history` between sessions |

1. Append `stty -ixon` to your `.bashrc`, so that `CTRL+S` can be used for forward
   history search.

    ```bash
    # disable terminal resume key, to use CTRL+S for forward search
    stty -ixon
    ```

    ```bash
    $ tail -2 ~/.bashrc
    # disable terminal resume key, to use CTRL+S for forward search
    stty -ixon
    ```

2. In the same file, set `HISTSIZE` to `10000` and `HISTFILESIZE` to `20000`.

    *Both settings are already in `.bashrc` — change them rather than adding new lines.
    If a setting appears twice, the last one wins.*

    ```bash
    $ grep -E '^HISTSIZE=|^HISTFILESIZE=' ~/.bashrc
    HISTSIZE=10000
    HISTFILESIZE=20000
    ```


&nbsp;


---

### Part 2 - Execute Command as root

`sudo` runs a single command as another user — root unless you say otherwise. The
commands below all answer *"who am I?"*, and under `sudo` they do **not** all give the
same answer. That difference is the point of this part.

| Command | What it reports |
| :--- | :--- |
| `id` | uid, gid and groups of the user running this process |
| `whoami` | The **effective** user of this process |
| `who am i` | The user who **logged in** to this terminal |
| `sudo -u <user> <cmd>` | Run one command as `<user>`, using **your** password |
| `su <user>` | Start a shell as `<user>`, using **their** password. Leave it with `exit` |

3. Run `id`, and note your own uid, gid and groups.

    ```bash
    $ id
    uid=1000(sysadmin) gid=1000(sysadmin) groups=1000(sysadmin),27(sudo),...
    ```

4. Run `id` again, this time with `sudo`. What is the uid value? **→ `T4`**

    ```bash
    $ sudo id
    uid=<...> gid=<...> groups=<...>
    ```

    *Compare with step 3, and record the uid value only.*

5. Run `who am i` with `sudo`. What is the output? **→ `T5`**

    ```bash
    $ sudo who am i
    <...>  pts/0  2026-08-05 13:20 (10.4.86.1)
    ```

6. Run `whoami` with `sudo`. What is the output? **→ `T6`**

    ```bash
    $ sudo whoami
    <...>
    ```

    *Steps 5 and 6 look like the same question. They are not — if you gave the same
    answer to both, read the table above again.*

7. Create a `tempdir` directory inside `~/int134/lab01/`, using `sudo`. Who owns it? **→ `T7`**

    ```bash
    $ sudo mkdir tempdir
    $ ls -ld tempdir
    drwxr-xr-x 2 <...> <...> 4096 Aug  5 13:22 tempdir
    ```

8. Create a file called `hello` inside `tempdir`.

    ```bash
    $ touch tempdir/hello
    touch: cannot touch 'tempdir/hello': Permission denied
    ```

    *Your first attempt is meant to fail. Read the error and work out what gets past
    it: you own your home directory, but you do not own `tempdir`.*

9. Take ownership of `tempdir` with `sudo chown -R sysadmin: tempdir`.

    ```bash
    $ ls -ld tempdir
    drwxr-xr-x 2 sysadmin sysadmin 4096 Aug  5 13:22 tempdir
    ```

10. Create a `test` user with `sudo useradd -m -s /bin/bash test`.

11. Give the `test` user the password `mflv[`.

    ```bash
    $ sudo passwd test
    New password:
    Retype new password:
    passwd: password updated successfully
    ```

    *Nothing appears as you type. That is deliberate, not a broken keyboard. You will
    need this password in step 16.*

12. Find the `test` user's `uid` and `gid`.

    ```bash
    $ id test
    uid=1001(test) gid=1001(test) groups=1001(test)
    ```

    *Your numbers may differ. Notice that a new user also gets a group of its own name.*

13. Add `test` to the `sysadmin` group with `sudo usermod -aG sysadmin test`.

14. Show that `test` is now a member of `sysadmin`.

    ```bash
    $ <...>
    test sysadmin
    ```

15. Let members of the `sysadmin` group write to `tempdir`. **→ `T15`**

    ```bash
    $ <...>
    $ ls -ld tempdir
    drwxrwxr-x 2 sysadmin sysadmin 4096 Aug  5 13:25 tempdir
    ```

    *The middle three letters are the group's permissions — compare with step 9.*

16. Become the `test` user with `su test`, then create a file called `testfile` inside
    `tempdir`. Note who owns it.

    ```bash
    $ su test
    Password:
    test@int134:/home/sysadmin/int134/lab01$ touch tempdir/testfile
    test@int134:/home/sysadmin/int134/lab01$ ls -l tempdir/testfile
    -rw-rw-r-- 1 test test 0 Aug  5 13:28 tempdir/testfile
    ```

    *`su` asks for `test`'s password — the one you set in step 11, not your own. Used
    without `-`, it leaves you in the same directory.*

    *This only works because of steps 13 and 15. Without them, `test` could neither
    reach the directory nor write in it.*

17. Return to your own account, then use `sudo -u` to create a file called `testfile2`
    inside `tempdir`. Note who owns it. **→ `T17`**

    ```bash
    $ exit
    exit
    $ <...>
    $ ls -l tempdir/testfile2
    -rw-rw-r-- 1 test test 0 Aug  5 13:30 tempdir/testfile2
    ```

    *Same owner as step 16, reached without becoming that user — and using your own
    password rather than theirs.*

&nbsp;


---

### Part 3 - Package Management

`apt` keeps a local copy of what the archive offers, and that copy is separate from
what is installed on your machine. **Refreshing the list and upgrading packages are
two different operations** — assuming one does the other is the most common mistake in
this part.

| Command | What it does |
| :--- | :--- |
| `apt update` | Refresh the **meta-information**: what exists, and at what version. Installs nothing |
| `apt list --installed` | Packages present on this machine |
| `apt list --upgradable` | Installed packages that have a newer version available |
| `apt list <name> -a` | Every version of a package the archive offers |
| `apt show <name>` | Details of one package: sizes, dependencies, description |
| `apt install <name>` | Install the newest available version |
| `apt install <name>=<version>` | Install one specific version |

18. Refresh the package meta-information on your server. **→ `T18`**

    ```bash
    $ <...>
    Hit:1 http://th.archive.ubuntu.com/ubuntu noble InRelease
    Get:2 http://th.archive.ubuntu.com/ubuntu noble-updates InRelease [126 kB]
    ...
    Reading package lists... Done
    ```

19. List the packages installed on this machine.

    ```bash
    $ <...>
    Listing...
    adduser/noble,now 3.137ubuntu1 all [installed,automatic]
    amd64-microcode/noble-updates,now 3.20251202.1ubuntu0.24.04.1 amd64 [installed,automatic]
    ...
    ```

20. List the installed packages that have a newer version available. **→ `T20`**

    ```bash
    $ <...>
    Listing...
    apport/noble-updates 2.28.2-0ubuntu0.1 all [upgradable from: 2.28.1-0ubuntu3.8]
    base-files/noble-updates 13ubuntu10.4 amd64 [upgradable from: 13ubuntu10.3]
    ...
    ```

21. List every version of `openssh` and of `unzip` that the archive offers.

    *Hint: one option makes `apt list` show all versions rather than just the newest.*

    ```bash
    $ apt list unzip <...>
    Listing...
    unzip/noble-updates,noble-security 6.0-28ubuntu4.1 amd64
    unzip/noble 6.0-28ubuntu4 amd64
    ```

    *Two versions are on offer. That matters for the next few steps.*

22. What is the download size of `unzip` version `6.0-28ubuntu4`? **→ `T22`**

    *Hint: one `apt` subcommand shows a single package's details.*

    ```bash
    $ apt <...> unzip=6.0-28ubuntu4
    Package: unzip
    Version: 6.0-28ubuntu4
    Priority: optional
    Section: utils
    Origin: Ubuntu
    Installed-Size: 384 kB
    Download-Size: <...>
    APT-Sources: http://th.archive.ubuntu.com/ubuntu noble/main amd64 Packages
    Description: De-archiver for .zip files
    ```

    *Ask about the version this step names, not just `unzip`. Step 21 showed you they
    are two different package files, and they do not weigh the same.*

23. Install `unzip`. Which version did you get?

    ```bash
    $ dpkg -l unzip | tail -1
    ii  unzip  6.0-28ubuntu4.1  amd64  De-archiver for .zip files
    ```

24. Now install version `6.0-28ubuntu4` specifically, with
    `sudo apt install unzip=6.0-28ubuntu4`.

    *This is an **older** version than step 23 left you with. apt will say so and ask
    you to confirm — read the message before you answer.*

25. Check whether `unzip` is upgradable.

    ```bash
    $ <...>
    Listing...
    unzip/noble-updates,noble-security 6.0-28ubuntu4.1 amd64 [upgradable from: 6.0-28ubuntu4]
    ```

26. Upgrade `unzip`.

    *Hint: install it again, without naming a version.*

27. Confirm that the latest version is now installed, the same way you checked in
    step 23.

&nbsp;


---

### Part 4 - Service Manager

`systemd` is the first process to start on the system, and every other process
descends from it. A service it manages has **two independent states**:

* **enabled / disabled** — whether it starts at the next boot
* **active / inactive** — whether it is running right now

Disabling a service does not stop it, and starting one does not bring it back after a
reboot. Steps 33 and 34 exist to show you exactly that, so read the output rather than
assuming.

| Command | What it does |
| :--- | :--- |
| `systemctl status <svc>` | Full report: both states, plus recent log lines |
| `systemctl is-enabled <svc>` | Just the boot state: `enabled` / `disabled` |
| `systemctl is-active <svc>` | Just the running state: `active` / `inactive` |
| `systemctl start` / `stop` / `restart` | Change what is running **now** |
| `systemctl enable` / `disable` | Change what happens **at boot** |

In `systemctl status` output, the two states appear here:

```bash
● cron.service - Regular background program processing daemon
     Loaded: loaded (/usr/lib/systemd/system/cron.service; enabled; preset: enabled)
                                                           ^^^^^^^ boot state
     Active: active (running) since Wed 2026-08-05 12:07:04 +07; 10h ago
             ^^^^^^^^^^^^^^^^ running state
```

28. Which program runs first on your system, as pid 1? **→ `T28`**

    *Hint: `ps 1` reports on the process with that pid.*

    ```bash
    $ ps 1
        PID TTY      STAT   TIME COMMAND
          1 ?        Ss     0:23 <...>
    ```

29. View the status of the system as a whole.

    *Hint: the same command as below, with no service name after it.*

    ```bash
    $ systemctl <...>
    ● int134
        State: running
        Units: 341 loaded (incl. loaded aliases)
         Jobs: 0 queued
       Failed: 0 units
        Since: Wed 2026-08-05 12:07:03 +07; 10h ago
    ...
    ```

    *Press `q` to leave the pager.*

30. View the status of the `ssh` service. Is it enabled?

    ```bash
    $ systemctl status ssh
    ● ssh.service - OpenBSD Secure Shell server
         Loaded: loaded (/usr/lib/systemd/system/ssh.service; enabled; preset: enabled)
         Active: active (running) since Wed 2026-08-05 12:07:04 +07; 10h ago
    ...
    ```

    *Worth a thought: you are connected over this service right now.*

31. Is `cron` enabled? Is it running?

    ```bash
    $ systemctl status cron
    ● cron.service - Regular background program processing daemon
         Loaded: loaded (/usr/lib/systemd/system/cron.service; enabled; preset: enabled)
         Active: active (running) since Wed 2026-08-05 12:07:04 +07; 10h ago
    ...
    ```

    *This is your starting point. Note both states — the next steps change them one at
    a time.*

32. Restart `cron`, then confirm it restarted by looking at the `Active` field.

    ```bash
    $ sudo systemctl restart cron
    $ systemctl status cron
         Active: active (running) since Wed 2026-08-05 22:41:12 +07; 3s ago
    ```

    *The state is unchanged. The `since` time is not.*

33. Disable `cron`, and confirm that it is disabled. What is cron's status now? **→ `T33`**

    ```bash
    $ sudo systemctl disable cron
    Synchronizing state of cron.service with SysV service script...
    Removed "/etc/systemd/system/multi-user.target.wants/cron.service".
    $ systemctl status cron
         Loaded: loaded (/usr/lib/systemd/system/cron.service; disabled; preset: enabled)
         Active: <...>
    ```

    *Record the **Active** line, not the Loaded one. Look before you answer — this is
    where most people guess, and guess wrongly.*

34. Reboot the system. What is cron's status once it comes back? **→ `T34`**

    ```bash
    $ sudo reboot
    Connection to int134 closed by remote host.
    ```

    Wait about a minute, connect again with ssh, then:

    ```bash
    $ systemctl status cron
         Loaded: loaded (/usr/lib/systemd/system/cron.service; disabled; preset: enabled)
         Active: <...>
    ```

    *Compare with your answer to step 33. If they differ, you have just seen what
    `disable` really does — and what it does not do.*

35. Enable `cron`. What is its status? **→ `T35`**

    ```bash
    $ sudo systemctl enable cron
    Synchronizing state of cron.service with SysV service script...
    Created symlink /etc/systemd/system/multi-user.target.wants/cron.service → /usr/lib/systemd/system/cron.service.
    $ systemctl is-enabled cron
    enabled
    $ systemctl is-active cron
    <...>
    ```

    *Record the **Active** state, as in steps 33 and 34. You have now seen all
    three combinations — and only one command left to run.*

36. Start `cron`, and confirm that it is running.

    ```bash
    $ <...>
    $ systemctl is-active cron
    active
    ```

    *It took two commands to get back to where step 31 started. That is the whole
    lesson of Part 4.*

&nbsp;


---

### What is checked

`int134 check lab01` looks at two things: the state of your machine, and your answers.

**On your machine.** These need the work actually done, not described:

| Part | Checked |
| :--- | :--- |
| 1 | `stty -ixon` active in `.bashrc`; `HISTSIZE` and `HISTFILESIZE` set |
| 2 | `tempdir` exists and belongs to you; the group can write to it; `hello` is in it; `test` exists and is in `sysadmin`; `testfile` and `testfile2` are owned by `test` |
| 3 | `unzip` installed, and at the latest available version |
| 4 | `cron` enabled, and running |

**From your answer file.** `T4`, `T5`, `T6`, `T7`, `T15`, `T17`, `T18`, `T20`, `T22`,
`T28`, `T33`, `T34`, `T35`.

Step 18 is asked as a written answer only: the system refreshes package lists on its
own schedule, so your machine cannot show who did it.
