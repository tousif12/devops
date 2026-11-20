# Bash Scripting Guide: From Basics to Pro

This guide explains every script in the `bash_scripts_DPC` folder. Each section has:

- **What it does**: a simple description
- **The code**: the script itself
- **Line by line**: how it works
- **Pro tips**: things experienced engineers know, plus common mistakes

The scripts are ordered the way you should learn them. Each one builds on the one before.

---

## Table of Contents

1. [Before You Start](#before-you-start)
2. [1_firstscript.sh: Your First Script](#1_firstscriptsh-your-first-script)
3. [2_websetup.sh: Automating a Website Setup](#2_websetupsh-automating-a-website-setup)
4. [3_vars_websetup.sh: Using Variables](#3_vars_websetupsh-using-variables)
5. [testvars.sh: Environment Variables and export](#testvarssh-environment-variables-and-export)
6. [4_args.sh: Command Line Arguments](#4_argssh-command-line-arguments)
7. [5_args_websetup.sh: Arguments in a Real Script](#5_args_websetupsh-arguments-in-a-real-script)
8. [6_command_subs.sh: Command Substitution](#6_command_subssh-command-substitution)
9. [7_userInput.sh: Reading User Input](#7_userinputsh-reading-user-input)
10. [8if1.sh: The if Statement](#8if1sh-the-if-statement)
11. [9_if1.sh: if / else](#9_if1sh-if--else)
12. [10_ifelif.sh: if / elif / else](#10_ifelifsh-if--elif--else)
13. [11_monit.sh: Monitoring with Exit Codes](#11_monitsh-monitoring-with-exit-codes)
14. [12_monit.sh: Monitoring with File Tests](#12_monitsh-monitoring-with-file-tests)
15. [13_for.sh: The for Loop](#13_forsh-the-for-loop)
16. [14_for.sh: Creating Users with a Loop](#14_forsh-creating-users-with-a-loop)
17. [15_while.sh: The while Loop](#15_whilesh-the-while-loop)
18. [16_while.sh: Infinite Loops](#16_whilesh-infinite-loops)
19. [dismantle.sh: Cleanup Script](#dismantlesh-cleanup-script)
20. [remote_websetup/multios_websetup.sh: Multi OS Setup](#remote_websetupmultios_websetupsh-multi-os-setup)
21. [remote_websetup/webdeploy.sh: Deploying to Many Servers](#remote_websetupwebdeploysh-deploying-to-many-servers)
22. [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)
23. [Going Pro: Habits That Matter](#going-pro-habits-that-matter)

---

## Before You Start

### How to run a script

```bash
chmod +x 1_firstscript.sh     # give it permission to run (only once)
./1_firstscript.sh            # run it
```

Or run it without changing permissions:

```bash
bash 1_firstscript.sh
```

### The shebang line

Every script starts with:

```bash
#!/bin/bash
```

This is called the **shebang**. It tells Linux "use the bash program to run this file". Without it, the system may use a different shell (like `sh`) and some features may break.

### Comments

Anything after `#` is a comment. Bash ignores it. Comments are for humans.

```bash
# This is a comment
echo "Hello"   # comments can also go at the end of a line
```

### Where these scripts run

Most scripts here use `yum` and `httpd`, which means they are written for **CentOS / RHEL / Amazon Linux**. On Ubuntu, the equivalents are `apt` and `apache2`. The multi OS script near the end handles both.

---

## 1_firstscript.sh: Your First Script

### What it does

Prints basic health information about your server: how long it has been running, how much memory is used, and how much disk space is left.

### The code

```bash
#!/bin/bash

### This script prints system info ###

echo "Welcome to bash script."
echo

#checking systemt uptime
echo "#####################################"
echo "The uptime of the system is: "
uptime
echo

# Memory Utilization
echo "#####################################"
echo "Memory Utilization"
free -m
echo

# Disk Utilization
echo "#####################################"
echo "Disk Utilization"
df -h
```

### Line by line

| Command | Meaning |
|---|---|
| `echo "text"` | Prints text to the screen |
| `echo` (alone) | Prints an empty line, used for spacing |
| `uptime` | Shows how long the system has been running and the load average |
| `free -m` | Shows RAM usage in megabytes (`-m`) |
| `df -h` | Shows disk usage in human readable form (`-h` means GB, MB instead of raw bytes) |

### Key idea

A bash script is just **a list of commands you would normally type**, saved in a file. Nothing magic. If you can type it in the terminal, you can put it in a script.

### Pro tips

- Use `free -h` for human readable memory (shows `G` and `M` automatically).
- `uptime` output example: `10:15:01 up 3 days, load average: 0.10, 0.05, 0.01`. The three numbers are load over the last 1, 5, and 15 minutes. If the number is higher than your CPU count, the server is overloaded. Check CPU count with `nproc`.
- `df -h /` shows only the root partition, which is usually what you care about.

---

## 2_websetup.sh: Automating a Website Setup

### What it does

Turns a fresh Linux server into a working website. It installs a web server (Apache / httpd), downloads a free website template, copies it into the web folder, restarts the server, and cleans up.

This is your first **real automation script**. Doing this by hand takes 10 minutes. The script does it in seconds, every time, without mistakes.

### The code

```bash
#!/bin/bash

# Installing Dependencies
echo "########################################"
echo "Installing packages."
echo "########################################"
sudo yum install wget unzip httpd -y > /dev/null
echo

# Start & Enable Service
echo "########################################"
echo "Start & Enable HTTPD Service"
echo "########################################"
sudo systemctl start httpd
sudo systemctl enable httpd
echo

# Creating Temp Directory
echo "########################################"
echo "Starting Artifact Deployment"
echo "########################################"
mkdir -p /tmp/webfiles
cd /tmp/webfiles
echo

wget https://www.tooplate.com/zip-templates/2098_health.zip > /dev/null
unzip 2098_health.zip > /dev/null
sudo cp -r 2098_health/* /var/www/html/
echo

# Bounce Service
echo "########################################"
echo "Restarting HTTPD service"
echo "########################################"
systemctl restart httpd
echo

# Clean Up
echo "########################################"
echo "Removing Temporary Files"
echo "########################################"
rm -rf /tmp/webfiles
echo

sudo systemctl status httpd
ls /var/www/html/
```

### Step by step

1. **Install packages**: `sudo yum install wget unzip httpd -y`
   - `wget` downloads files from the internet
   - `unzip` extracts zip files
   - `httpd` is the Apache web server
   - `-y` answers "yes" automatically so the script does not stop and wait
2. **Start and enable the service**
   - `systemctl start httpd` starts it now
   - `systemctl enable httpd` makes it start automatically after every reboot
3. **Make a temp folder**: `mkdir -p /tmp/webfiles` (`-p` means "do not complain if it already exists")
4. **Download and unzip** the website template
5. **Copy files** into `/var/www/html/`, the folder Apache serves to visitors
6. **Restart** Apache so it picks up the new files
7. **Clean up** the temp folder
8. **Verify**: show service status and list the web files

### What is `> /dev/null`?

`/dev/null` is a "black hole" file. Anything sent there disappears. `> /dev/null` hides the normal output of a command so your screen stays clean.

| Syntax | What it hides |
|---|---|
| `> /dev/null` | Normal output (stdout) only |
| `2> /dev/null` | Error output (stderr) only |
| `&> /dev/null` | Both normal output and errors |

Note: `wget` prints its progress to **stderr**, so `wget ... > /dev/null` still shows progress. Use `wget -q` (quiet) instead.

### Pro tips

- **Problem with this script**: every value (URL, folder name, package names) is typed directly in the code. If you want a different template, you must edit many lines. The next script fixes this with variables.
- `systemctl restart httpd` has no `sudo`. It works only if you run the whole script as root. Be consistent: either use `sudo` on every privileged command, or run the script with `sudo ./2_websetup.sh` and remove all the `sudo` inside.
- `systemctl enable --now httpd` does start and enable in one command.
- `systemctl status` can open a pager and wait for you to press `q`. Use `systemctl status httpd --no-pager` in scripts.

---

## 3_vars_websetup.sh: Using Variables

### What it does

Same website setup as before, but now all the changeable values are stored in **variables** at the top. To deploy a different template, you only change one or two lines.

### The code (key part)

```bash
#!/bin/bash

# Variable Declaration
PACKAGE="httpd wget unzip"
SVC="httpd"
URL='https://www.tooplate.com/zip-templates/2098_health.zip'
ART_NAME='2098_health'
TEMPDIR="/tmp/webfiles"

sudo yum install $PACKAGE -y > /dev/null
sudo systemctl start $SVC
sudo systemctl enable $SVC

mkdir -p $TEMPDIR
cd $TEMPDIR

wget $URL > /dev/null
unzip $ART_NAME.zip > /dev/null
sudo cp -r $ART_NAME/* /var/www/html/

systemctl restart $SVC
rm -rf $TEMPDIR

sudo systemctl status $SVC
ls /var/www/html/
```

### How variables work

**Create** a variable (no spaces around `=`):

```bash
NAME="Tousif"      # correct
NAME = "Tousif"    # WRONG: bash thinks NAME is a command
```

**Use** a variable by putting `$` in front:

```bash
echo $NAME
echo "Hello $NAME"
echo "File is ${NAME}_backup.txt"   # use {} when text follows directly
```

### Single quotes vs double quotes

```bash
CITY="Lahore"
echo "I live in $CITY"   # Output: I live in Lahore
echo 'I live in $CITY'   # Output: I live in $CITY
```

- **Double quotes** `" "`: variables are expanded (replaced by their value)
- **Single quotes** `' '`: everything is taken literally, nothing is expanded

The script uses single quotes for the URL because it is a fixed value with nothing to expand. That is a safe habit.

### Pro tips

- Always wrap variables in double quotes when using them: `cd "$TEMPDIR"`. If a value contains spaces, unquoted variables break into multiple words and cause bugs.
- Exception: `$PACKAGE` here is intentionally unquoted because we **want** it split into three package names. A cleaner pro way is an array:
  ```bash
  PACKAGES=(httpd wget unzip)
  sudo yum install "${PACKAGES[@]}" -y
  ```
- **Dangerous pattern**: `rm -rf $TEMPDIR`. If `TEMPDIR` is empty by mistake, this becomes `rm -rf` on nothing, but similar code like `rm -rf $DIR/*` with an empty `DIR` becomes `rm -rf /*`. Protect yourself:
  ```bash
  rm -rf "${TEMPDIR:?TEMPDIR is not set}"
  ```
  This stops the script with an error if the variable is empty.
- Convention: UPPERCASE for constants and environment variables, lowercase for local working variables.

---

## testvars.sh: Environment Variables and export

### What it does

Prints a sentence using a variable called `SEASON`. But the variable is **not defined inside the script**. This script teaches you how variables pass (or do not pass) from your terminal into a script.

### The code

```bash
#!/bin/bash

echo "The $SEASON season is more than expected, this time."
```

### Try this

```bash
SEASON="Monsoon"
./testvars.sh
# Output: The  season is more than expected, this time.   (empty!)

export SEASON="Monsoon"
./testvars.sh
# Output: The Monsoon season is more than expected, this time.
```

### Why?

When you run a script, it starts in a **new child process**. A normal variable only lives in your current shell. `export` marks the variable as an **environment variable**, which gets copied to every child process.

| Type | Visible to scripts you run? | How to create |
|---|---|---|
| Shell variable | No | `SEASON="Monsoon"` |
| Environment variable | Yes | `export SEASON="Monsoon"` |

### Parent shell and child shell

When you open a terminal, you get a bash program running. That's the **parent shell**. When you run a script like `./testvars.sh`, bash doesn't run it inside your terminal's shell. It starts a **new, separate bash process** to run the script, and that new process is the **child shell**. When the script finishes, the child shell closes and you're back in the parent.

```
Terminal (parent shell, PID 1000)
   │
   └── ./testvars.sh  →  new bash (child shell, PID 1050)
                          runs the script, then exits
```

#### A simple analogy

Think of the parent shell as a manager and the child shell as a new employee.

- **Shell variable** (`SEASON="Monsoon"`): a note on the manager's private desk. The new employee can't see it.
- **Environment variable** (`export SEASON="Monsoon"`): the manager writes the note into the official handover file that every new employee gets on day one.

A child gets a copy of every exported variable when it starts. It never gets the parent's private, unexported variables.

#### See it yourself

Run these in your terminal:

```bash
echo $$                  # PID of your current shell, e.g. 1000
bash                     # start a child shell by hand
echo $$                  # different PID, e.g. 1050 (you are now in the child)
exit                     # leave the child, back to the parent
```

Now with variables:

```bash
SEASON="Monsoon"
bash -c 'echo "Child sees: $SEASON"'     # Child sees:          (empty)

export SEASON
bash -c 'echo "Child sees: $SEASON"'     # Child sees: Monsoon
```

`bash -c '...'` starts a child shell and runs that one command in it.

#### Copying only goes one way

The child gets a copy, not the original. Changes in the child never flow back to the parent:

```bash
export SEASON="Monsoon"
bash -c 'SEASON="Winter"; echo "Child: $SEASON"'   # Child: Winter
echo "Parent: $SEASON"                              # Parent: Monsoon
```

This is also why a script that runs `cd /tmp` doesn't change your terminal's folder after it finishes. The `cd` happened in the child.

#### Running a script inside the parent

If you want a script to run in your current shell, so it can see all your variables and its changes stay after it ends, use `source`:

```bash
SEASON="Monsoon"         # not exported
source testvars.sh       # or the short form:  . testvars.sh
# Output: The Monsoon season is more than expected, this time.
```

No child shell is created, so even unexported variables are visible. This is why you run `source ~/.bashrc` after editing it: you want the new settings in your current terminal, not in a child that closes straight away.

#### Quick summary

| Action | Creates a child? | Sees unexported variables? | Changes stay after? |
|---|---|---|---|
| `./script.sh` or `bash script.sh` | Yes | No | No |
| `source script.sh` or `. script.sh` | No | Yes | Yes |
| `export VAR` then `./script.sh` | Yes | Yes (gets a copy) | No |

### Making it permanent

Variables disappear when you close the terminal. To keep them:

- **For your user only**: add `export SEASON="Monsoon"` to `~/.bashrc`, then run `source ~/.bashrc`
- **For all users**: add it to `/etc/profile` (needs root)

### Pro tips

- One-time variable for a single command: `SEASON="Winter" ./testvars.sh`
- See all environment variables: `env` or `printenv`
- Built in variables you can use anywhere: `$USER`, `$HOME`, `$HOSTNAME`, `$PWD`, `$PATH`, `$RANDOM`
- Give a default value if the variable is empty: `echo "${SEASON:-Summer}"`

---

## 4_args.sh: Command Line Arguments

### What it does

Shows how a script receives values you type after its name. These values are called **arguments**.

### The code

```bash
#!/bin/bash

echo "Value of 0 is "
echo $0

echo "Value of 1"
echo $1

echo "Value of 2"
echo $2

echo "Value of 3"
echo $3
```

### Try this

```bash
./4_args.sh Linux AWS Docker
```

Output:

```
Value of 0 is
./4_args.sh
Value of 1
Linux
Value of 2
AWS
Value of 3
Docker
```

### Special argument variables

| Variable | Meaning |
|---|---|
| `$0` | The script name itself |
| `$1` to `$9` | First to ninth argument |
| `${10}` | Tenth argument and beyond (braces needed) |
| `$#` | How many arguments were given |
| `$@` | All arguments, each kept separate (best for loops) |
| `$*` | All arguments joined as one string |
| `$?` | Exit code of the last command (0 means success) |
| `$$` | Process ID of the current script |

### Pro tips

Always check that the user gave the right number of arguments:

```bash
if [ $# -ne 2 ]; then
  echo "Usage: $0 <url> <artifact_name>"
  exit 1
fi
```

Loop over all arguments safely:

```bash
for arg in "$@"; do
  echo "Got: $arg"
done
```

---

## 5_args_websetup.sh: Arguments in a Real Script

### What it does

The website setup script again, but now the URL and folder name come from **arguments** instead of being written inside the script. One script can now deploy **any** template without editing.

### The code (key change)

```bash
#URL='https://www.tooplate.com/zip-templates/2098_health.zip'
#ART_NAME='2098_health'
...
wget $1 > /dev/null
unzip $2.zip > /dev/null
sudo cp -r $2/* /var/www/html/
```

### How to run

```bash
./5_args_websetup.sh https://www.tooplate.com/zip-templates/2098_health.zip 2098_health
```

- `$1` = the download URL
- `$2` = the name of the zip / folder inside it

### Why this matters

You went from:

1. **Hardcoded** (script 2): works for one template only
2. **Variables** (script 3): easy to edit, but still need to edit
3. **Arguments** (script 5): no editing at all, reusable by anyone

This is how real DevOps tools are built.

### Pro tips

Give arguments clear names at the top. It makes the script much easier to read:

```bash
URL="$1"
ART_NAME="$2"

if [ -z "$URL" ] || [ -z "$ART_NAME" ]; then
  echo "Usage: $0 <url> <artifact_name>"
  exit 1
fi
```

`-z` means "is this string empty?". If the user forgets an argument, the script stops with a helpful message instead of breaking halfway.

---

## 6_command_subs.sh: Command Substitution

### What it does

Shows a short system report: free RAM, current load, and free disk space. It **captures the output of commands into variables** so it can print them in nice sentences.

### The code

```bash
#!/bin/bash

echo "Welcome $USER on $HOSTNAME."
echo "#######################################################"

FREERAM=$(free -m | grep Mem | awk '{print $4}')
LOAD=`uptime | awk '{print $9}'`
ROOTFREE=$(df -h | grep '/dev/sda1' | awk '{print $4}')

echo "#######################################################"
echo "Available free RAM is $FREERAM MB"
echo "#######################################################"
echo "Current Load Average $LOAD"
echo "#######################################################"
echo "Free ROOT partiotion size is $ROOTFREE"
```

### Command substitution

Run a command and store its output in a variable:

```bash
TODAY=$(date +%F)      # modern style, recommended
TODAY=`date +%F`       # old style with backticks, same result
```

Use `$( )`. It is easier to read and can be nested: `$(dirname $(which bash))`.

### The pipe `|`

The pipe sends the output of one command into the next one, like an assembly line.

Let's break down `free -m | grep Mem | awk '{print $4}'`:

**Step 1**: `free -m`

```
              total   used   free  shared  buff/cache  available
Mem:           3900    800   2100      10        1000       2900
Swap:             0      0      0
```

**Step 2**: `grep Mem` keeps only lines containing "Mem"

```
Mem:           3900    800   2100      10        1000       2900
```

**Step 3**: `awk '{print $4}'` prints the 4th column (columns are split by spaces)

```
2100
```

### awk in 30 seconds

`awk` splits each line into columns: `$1` is the first, `$2` the second, `$NF` the last.

```bash
echo "a b c d" | awk '{print $2}'      # b
echo "a b c d" | awk '{print $NF}'     # d
echo "a:b:c"   | awk -F: '{print $3}'  # c   (-F sets the separator)
```

Note: inside awk, `$4` means column 4. It is **not** the same as bash argument `$4`. The single quotes stop bash from touching it.

### Pro tips (important fixes)

- **`/dev/sda1` may not exist** on your machine. Cloud servers often use `/dev/xvda1` or `/dev/nvme0n1p1`. A safer way to get free space on root:
  ```bash
  ROOTFREE=$(df -h / | awk 'NR==2 {print $4}')
  ```
  `NR==2` means "the second line" (skipping the header).
- **`awk '{print $9}'` on uptime is fragile.** The column number changes depending on whether uptime shows minutes, hours, or days, and how many users are logged in. A reliable way:
  ```bash
  LOAD=$(cat /proc/loadavg | awk '{print $1}')
  ```
- For "free" memory, the **available** column (`$7`) is usually more meaningful than `free` (`$4`), because Linux uses spare RAM for cache and gives it back when needed.

---

## 7_userInput.sh: Reading User Input

### What it does

Asks the user questions and uses their answers. It also shows how to read a password without showing it on screen.

### The code

```bash
#!/bin/bash

echo "Enter your skills:"
read SKILL

echo "Your $SKILL skill is in high Demand in the IT Industry."

read -p 'Username: ' USR
read -sp 'Password: ' pass

echo

echo "Login Successfull: Welcome USER $USR,"
```

### The `read` command

| Usage | What it does |
|---|---|
| `read VAR` | Waits for input, saves it in `VAR` |
| `read -p 'Prompt: ' VAR` | Shows a prompt on the same line |
| `read -s VAR` | Silent: typing is hidden (for passwords) |
| `read -sp 'Password: ' VAR` | Silent with prompt |
| `read -t 10 VAR` | Timeout: give up after 10 seconds |
| `read -r VAR` | Raw: treats backslashes as normal text (recommended) |

The `echo` after the password read is needed because `-s` hides the Enter key too, so the next output would appear on the same line.

### Pro tips

- Use `read -r` by default. It avoids weird behaviour with `\` in input.
- Never store real passwords in plain variables in production scripts. Use a secrets manager or environment variables.
- Scripts that need input cannot run unattended (for example in cron or CI pipelines). For automation, prefer **arguments** or **environment variables** over `read`.
- Give a default value:
  ```bash
  read -rp "Port [80]: " PORT
  PORT=${PORT:-80}
  ```

---

## 8if1.sh: The if Statement

### What it does

Asks for a number. If it is greater than 100, it prints a message and the date. Otherwise it skips that part.

### The code

```bash
#!/bin/bash

read -p "Enter a number: " NUM
echo

if [ $NUM -gt 100 ]
then
   echo "We have entered in IF block."
   sleep 3
   echo "Your Number is greater than 100"
   echo
   date
fi

echo "Script execution completed successfully."
```

### Structure of if

```bash
if [ condition ]
then
   # runs only when condition is true
fi
```

`fi` is `if` backwards. It closes the block. You can also write `if [ condition ]; then` on one line.

### The spaces matter

```bash
if [ $NUM -gt 100 ]    # correct
if [$NUM -gt 100]      # WRONG: [ is actually a command and needs spaces
```

### Number comparison operators

| Operator | Meaning | Remember it as |
|---|---|---|
| `-eq` | equal | **eq**ual |
| `-ne` | not equal | **n**ot **e**qual |
| `-gt` | greater than | **g**reater **t**han |
| `-ge` | greater or equal | **g**reater or **e**qual |
| `-lt` | less than | **l**ess **t**han |
| `-le` | less or equal | **l**ess or **e**qual |

### Pro tips

- If the user presses Enter without typing, `$NUM` is empty and the test becomes `[ -gt 100 ]`, which gives an error. Always quote: `[ "$NUM" -gt 100 ]`.
- Validate that the input is a number:
  ```bash
  if ! [[ "$NUM" =~ ^[0-9]+$ ]]; then
    echo "Please enter a valid number"
    exit 1
  fi
  ```
- For math comparisons, bash also has `(( ))` which reads more naturally:
  ```bash
  if (( NUM > 100 )); then echo "big"; fi
  ```

---

## 9_if1.sh: if / else

### What it does

Same as before, but now there is an **else** block, so the script always says something, whichever way the condition goes.

### The code

```bash
if [ $NUM -gt 100 ]
then
   echo "We have entered in IF block."
   sleep 3
   echo "Your Number is greater than 100"
   echo
   date
else
  echo "You have entered number less than 100."
fi
```

### Structure

```bash
if [ condition ]
then
   # when true
else
   # when false
fi
```

### Pro tips

- Small logic bug: if you enter exactly `100`, the script says "less than 100". The correct message would be "100 or less". Always test the **edge cases** (the boundary values) of your conditions.
- `sleep 3` pauses for 3 seconds. Useful for waiting on services to start, but avoid it in scripts that need to be fast.

---

## 10_ifelif.sh: if / elif / else

### What it does

Counts how many active network interfaces (not counting loopback) the server has and reports: one, many, or none.

### The code

```bash
#!/bin/bash

value=$(ip addr show | grep -v LOOPBACK | grep -ic mtu)

if [ $value -eq 1 ]
then
  echo "1 Active Network Interface found."
elif [ $value -gt 1 ]
then
  echo "Found Multiple active Interface."
else
  echo "No Active interface found."
fi
```

### Breaking down the command

`ip addr show | grep -v LOOPBACK | grep -ic mtu`

| Part | Meaning |
|---|---|
| `ip addr show` | Lists all network interfaces |
| `grep -v LOOPBACK` | `-v` = invert: remove lines with LOOPBACK (the `lo` interface) |
| `grep -ic mtu` | `-i` = ignore case, `-c` = count matching lines. Each interface has one line with "mtu" |

Result: the number of non-loopback interfaces.

### Structure

```bash
if [ condition1 ]; then
   ...
elif [ condition2 ]; then
   ...
elif [ condition3 ]; then
   ...
else
   ...
fi
```

Bash checks from top to bottom and runs **only the first** block that is true.

### Useful grep flags

| Flag | Meaning |
|---|---|
| `-i` | Ignore uppercase / lowercase |
| `-v` | Show lines that do NOT match |
| `-c` | Count matching lines |
| `-n` | Show line numbers |
| `-r` | Search in all files inside a folder |
| `-q` | Quiet: no output, just success or failure (great inside `if`) |
| `-w` | Match whole words only |

### Pro tips

- This counts interfaces that **exist**, not only ones that are "UP". To count only active ones: `ip -o link show up | grep -vc LOOPBACK`.
- When there are many options for one value, `case` is cleaner than a long `elif` chain:
  ```bash
  case "$value" in
    0) echo "None" ;;
    1) echo "One" ;;
    *) echo "Many" ;;
  esac
  ```

---

## 11_monit.sh: Monitoring with Exit Codes

### What it does

A tiny monitoring tool. It checks if the Apache web server is running. If not, it tries to start it and reports whether that worked. You can run this every few minutes using cron to keep your website alive automatically.

### The code

```bash
#!/bin/bash

echo "#####################################################"
date
ls /var/run/httpd/httpd.pid &> /dev/null

if [ $? -eq 0 ]
then
   echo "Httpd process is running."
else
   echo "Httpd process is NOT Running."
   echo "Starting the process"
   systemctl start httpd
   if [ $? -eq 0 ]
   then
      echo "Process started successfully."
   else
      echo "Process Starting Failed, contact the admin."
   fi
fi
echo "#####################################################"
echo
```

### Exit codes: the most important concept in bash

Every command returns a number when it finishes:

- **0** = success
- **anything else (1 to 255)** = failure

`$?` holds the exit code of the **last** command.

```bash
ls /etc        ; echo $?   # 0  (folder exists)
ls /nothing    ; echo $?   # 2  (error)
```

### How the script uses it

1. When Apache runs, it creates a file `/var/run/httpd/httpd.pid` containing its process ID.
2. `ls` on that file succeeds (exit 0) if the file exists, fails if it does not.
3. `&> /dev/null` hides any output or error from `ls`.
4. `$?` tells us whether `ls` worked, which tells us whether Apache is running.

There is also a **nested if**: an `if` inside another `if`, checking whether `systemctl start` succeeded.

### Running it automatically with cron

```bash
crontab -e
```

Add this line to run every minute and save the output to a log:

```
* * * * * /opt/scripts/11_monit.sh >> /var/log/monit_httpd.log 2>&1
```

Cron format: `minute hour day-of-month month day-of-week command`

| Example | Meaning |
|---|---|
| `* * * * *` | Every minute |
| `*/5 * * * *` | Every 5 minutes |
| `0 2 * * *` | Every day at 2:00 AM |
| `0 9 * * 1-5` | 9:00 AM, Monday to Friday |

### Pro tips

- `$?` changes after **every** command, even `echo`. Read it immediately or save it: `rc=$?`.
- You do not need `$?` at all. `if` can test a command directly:
  ```bash
  if ls /var/run/httpd/httpd.pid &> /dev/null; then
  ```
- A PID file can be left behind after a crash, so it is not 100% reliable. Better checks:
  ```bash
  systemctl is-active --quiet httpd     # best for services
  pgrep httpd > /dev/null               # checks the actual process
  curl -s -o /dev/null -w "%{http_code}" http://localhost   # checks the website really responds
  ```
- Use `exit 0` and `exit 1` at the end of your own scripts so other tools know if they succeeded.

---

## 12_monit.sh: Monitoring with File Tests

### What it does

Exactly the same job as `11_monit.sh`, but cleaner. Instead of running `ls` and checking `$?`, it uses a built in **file test**: `-f`.

### The code (key change)

```bash
if [ -f /var/run/httpd/httpd.pid ]
then
   echo "Httpd process is running."
else
   ...
fi
```

`[ -f file ]` is true if the file exists and is a regular file. Simple, fast, no extra command needed.

### File test operators

| Test | True when |
|---|---|
| `-f file` | File exists and is a regular file |
| `-d dir` | Directory exists |
| `-e path` | Anything exists at that path |
| `-s file` | File exists and is not empty |
| `-r file` | File is readable |
| `-w file` | File is writable |
| `-x file` | File is executable |
| `-L file` | It is a symbolic link |

### String test operators

| Test | True when |
|---|---|
| `-z "$s"` | String is empty |
| `-n "$s"` | String is not empty |
| `"$a" = "$b"` | Strings are equal |
| `"$a" != "$b"` | Strings are different |

### Combining conditions

```bash
if [ -f file ] && [ -r file ]; then ...      # AND
if [ -f a ] || [ -f b ]; then ...            # OR
if [ ! -d /backup ]; then mkdir /backup; fi  # NOT
```

### Pro tips

- Prefer `[[ ]]` over `[ ]` in bash scripts. It is safer with empty variables and supports `&&`, `||`, and pattern matching inside:
  ```bash
  if [[ -f $FILE && $USER == "root" ]]; then ...
  ```
- One-line shortcuts:
  ```bash
  [ -d /backup ] || mkdir /backup
  [ -f config.txt ] && echo "Config found"
  ```

---

## 13_for.sh: The for Loop

### What it does

Goes through a list of programming languages one by one and prints each, with the date and a 1 second pause between them.

### The code

```bash
#!/bin/bash

for VAR1 in java .net python ruby php
do
  echo "Looping....."
  sleep 1
  echo "###################################################"
  echo "Value of VAR1 is $VAR1."
  echo "###################################################"
  date
  echo
done
```

### Structure

```bash
for VARIABLE in item1 item2 item3
do
   # runs once for each item; $VARIABLE holds the current item
done
```

The block runs 5 times. First `VAR1=java`, then `.net`, then `python`, and so on.

### Other ways to loop

```bash
# Numbers 1 to 5
for i in {1..5}; do echo $i; done

# Numbers with step (0, 10, 20 ... 100)
for i in {0..100..10}; do echo $i; done

# C style
for (( i=1; i<=5; i++ )); do echo $i; done

# Every .log file in a folder
for f in /var/log/*.log; do echo "Found $f"; done

# Every line of command output
for u in $(cut -d: -f1 /etc/passwd); do echo $u; done
```

### Pro tips

- `break` exits the loop early, `continue` skips to the next item.
- When looping over files, always quote: `"$f"`. File names with spaces will break otherwise.

---

## 14_for.sh: Creating Users with a Loop

### What it does

Creates three Linux users (`alpha`, `beta`, `gamma`) and shows their details. A real admin task that would be boring to do by hand for 50 users.

### The code

```bash
#!/bin/bash

MYUSERS="alpha beta gamma"

for usr in $MYUSERS
do
   echo "Adding user $usr."
   useradd $usr
   id $usr
   echo "#####################################"
done
```

### Line by line

- `MYUSERS="alpha beta gamma"`: a string with names separated by spaces
- `for usr in $MYUSERS`: bash splits the string at spaces, giving three items
- `useradd $usr`: creates the user (needs root, so run with `sudo`)
- `id $usr`: shows the user's ID and groups, confirming it worked

### Pro tips

- Running it a second time gives "user already exists" errors. Make it **idempotent** (safe to run many times):
  ```bash
  if id "$usr" &> /dev/null; then
     echo "$usr already exists, skipping."
  else
     sudo useradd "$usr"
  fi
  ```
  Idempotency is a core DevOps idea. Tools like Ansible and Terraform are built around it.
- Read users from a file instead of hardcoding:
  ```bash
  while read -r usr; do sudo useradd "$usr"; done < users.txt
  ```
- Delete them again: `sudo userdel -r alpha` (`-r` removes the home folder too).

---

## 15_while.sh: The while Loop

### What it does

Counts from 0 to 4 using a counter. The loop keeps running **while** the condition is true.

### The code

```bash
#!/bin/bash

counter=0

while [ $counter -lt 5 ]
do
  echo "Looping...."
  echo "Value of counter is $counter."
  counter=$(( $counter + 1 ))
  sleep 1
done

echo "Out of the loop"
```

### for vs while

- **for**: use when you know the list of items in advance
- **while**: use when you loop until something changes (a counter, a file appears, a service starts)

### Math in bash

```bash
counter=$(( counter + 1 ))   # $ inside (( )) is optional
(( counter++ ))              # shorter
echo $(( 10 / 3 ))           # 3   (bash only does whole numbers)
echo $(( 10 % 3 ))           # 1   (remainder)
echo $(( 2 ** 8 ))           # 256 (power)
```

For decimals, use `bc`: `echo "10 / 3" | bc -l`

### Pro tips

A very common real use: wait for something to be ready, with a time limit.

```bash
tries=0
until systemctl is-active --quiet httpd || [ $tries -ge 10 ]; do
  echo "Waiting for httpd..."
  sleep 2
  (( tries++ ))
done
```

`until` is the opposite of `while`: it loops **until** the condition becomes true.

---

## 16_while.sh: Infinite Loops

### What it does

Starts at 2 and doubles the number every second, forever. It never stops on its own.

### The code

```bash
#!/bin/bash

counter=2

while true
do
  echo "Looping...."
  echo "Value of counter is $counter."
  counter=$(( $counter * 2 ))
  sleep 1
done

echo "Out of the loop"
```

### Key points

- `true` is a command that always succeeds, so `while true` never ends.
- The last line `echo "Out of the loop"` **never runs**.
- Press **Ctrl + C** to stop it.

### Interesting thing to watch

After about 62 loops, the number becomes negative and then 0. Bash uses 64 bit integers, and when the number gets too big it **overflows**. A good lesson: computers have limits.

### Pro tips

- Infinite loops are useful for simple watchers and workers. Always add a way out:
  ```bash
  while true; do
    if [ -f /tmp/stop ]; then
      echo "Stop file found, exiting."
      break
    fi
    sleep 5
  done
  ```
- Clean up when someone presses Ctrl + C using `trap`:
  ```bash
  trap 'echo "Stopped by user"; exit 0' INT
  ```
- For long running jobs in production, use a systemd service instead of an infinite bash loop.

---

## dismantle.sh: Cleanup Script

### What it does

Undoes the website setup. Stops Apache, deletes the website files, and uninstalls the packages. Useful for resetting a test machine to start over.

### The code

```bash
#!/bin/bash
sudo systemctl stop httpd
sudo rm -rf /var/www/html/*
sudo yum remove httpd wget unzip -y
```

### Pro tips

- `rm -rf` with `sudo` is the most dangerous command in Linux. There is no recycle bin. Double check paths before running.
- Add a confirmation step for destructive scripts:
  ```bash
  read -rp "This will delete the website. Continue? (y/n) " ans
  [ "$ans" = "y" ] || exit 1
  ```
- Professional habit: for every **setup** script, write a matching **teardown** script. It keeps test environments clean and repeatable.

---

## remote_websetup/multios_websetup.sh: Multi OS Setup

### What it does

The website setup script, but smart enough to work on **both CentOS (Red Hat family) and Ubuntu (Debian family)**. It detects the OS first, then uses the right package manager and service name.

### Detection logic

```bash
yum --help &> /dev/null

if [ $? -eq 0 ]
then
   # CentOS: yum exists
   PACKAGE="httpd wget unzip"
   SVC="httpd"
   sudo yum install $PACKAGE -y > /dev/null
   ...
else
   # Ubuntu: no yum
   PACKAGE="apache2 wget unzip"
   SVC="apache2"
   sudo apt update
   sudo apt install $PACKAGE -y > /dev/null
   ...
fi
```

If `yum` works, it is a Red Hat style OS. If not, the script assumes Ubuntu.

### Differences between the two families

| | CentOS / RHEL / Amazon Linux | Ubuntu / Debian |
|---|---|---|
| Package manager | `yum` / `dnf` | `apt` |
| Web server package | `httpd` | `apache2` |
| Service name | `httpd` | `apache2` |
| Web root | `/var/www/html` | `/var/www/html` |
| Refresh package list | not required | `apt update` required first |

### Pro tips (improvements)

- **Bug**: the Ubuntu branch still prints "Running Setup on CentOS". Change it to "Running Setup on Ubuntu".
- **Duplicate code**: almost everything in both branches is the same. Only the variables and install command differ. A pro version sets variables in the `if`, then runs the common steps once:
  ```bash
  if command -v yum &> /dev/null; then
     PACKAGE="httpd wget unzip"; SVC="httpd"
     sudo yum install -y $PACKAGE
  else
     PACKAGE="apache2 wget unzip"; SVC="apache2"
     sudo apt update && sudo apt install -y $PACKAGE
  fi

  # common steps below, written only once
  sudo systemctl enable --now "$SVC"
  ...
  ```
- `command -v yum` is the standard way to check if a program exists.
- An even more accurate way to detect the OS is the file `/etc/os-release`:
  ```bash
  source /etc/os-release
  echo "$ID"    # centos, ubuntu, amzn, rhel, debian ...
  ```
- Rule of thumb: **DRY** (Don't Repeat Yourself). If you copy and paste a block, it probably belongs in a variable or a function.

---

## remote_websetup/webdeploy.sh: Deploying to Many Servers

### What it does

Deploys the website to **many servers at once** from one control machine. It reads a list of server names, copies the setup script to each one, runs it there, and deletes it afterwards. This is a mini version of what tools like Ansible do.

### Files involved

- `remhosts`: the list of target servers, one per line
  ```
  web01
  web02
  web03
  ```
- `multios_websetup.sh`: the script that gets copied and run on each server
- `testfile.txt`: a simple file for testing that `scp` copying works

### The code

```bash
#!/bin/bash

USR='devops'

for host in `cat remhosts`
do
   echo
   echo "#########################################################"
   echo "Connecting to $host"
   echo "Pushing Script to $host"
   scp multios_websetup.sh $USR@$host:/tmp/
   echo "Executing Script on $host"
   ssh $USR@$host sudo /tmp/multios_websetup.sh
   ssh $USR@$host sudo rm -rf /tmp/multios_websetup.sh
   echo "#########################################################"
   echo
done
```

### Step by step

1. `for host in $(cat remhosts)`: loop over each server name in the file
2. `scp file user@host:/tmp/`: securely **copy** the script to the remote server
3. `ssh user@host command`: **run** a command on the remote server
4. Delete the script from the remote server after it runs

### Prerequisites to make it work

1. **Hostnames resolve**: `web01`, `web02`, `web03` must point to IPs. Add them to `/etc/hosts` on the control machine:
   ```
   192.168.10.11  web01
   192.168.10.12  web02
   192.168.10.13  web03
   ```
2. **A `devops` user exists** on every server.
3. **Passwordless SSH** with keys, so the script does not stop and ask for passwords:
   ```bash
   ssh-keygen                     # create a key once (press Enter for defaults)
   ssh-copy-id devops@web01       # copy your key to each server
   ssh-copy-id devops@web02
   ssh-copy-id devops@web03
   ```
4. **Passwordless sudo** for `devops` on every server. Add a file with `sudo visudo -f /etc/sudoers.d/devops`:
   ```
   devops ALL=(ALL) NOPASSWD: ALL
   ```
5. **The script is executable** before copying: `chmod +x multios_websetup.sh`

### Pro tips

- Read the host file line by line (safer than `cat` in a `for` loop):
  ```bash
  while read -r host; do
     ...
  done < remhosts
  ```
  Note: `ssh` inside a `while read` loop eats the rest of the input. Use `ssh -n` or `ssh ... < /dev/null` to avoid that.
- Skip the `scp` step entirely by piping the script through ssh:
  ```bash
  ssh devops@web01 'sudo bash -s' < multios_websetup.sh
  ```
- Stop waiting forever on dead servers: `ssh -o ConnectTimeout=5 ...`
- Skip the "are you sure you want to connect" prompt on first connect (lab use only): `-o StrictHostKeyChecking=no`
- Record success and failure per host:
  ```bash
  if ssh "$USR@$host" sudo /tmp/multios_websetup.sh; then
     echo "$host OK" >> deploy.log
  else
     echo "$host FAILED" >> deploy.log
  fi
  ```
- When you have more than a handful of servers, this is the point where you move to **Ansible**. The ideas you learned here (inventory file, push, execute, idempotency) are exactly how Ansible works.

---

## Quick Reference Cheat Sheet

### Variables

```bash
NAME="value"            # create (no spaces around =)
echo "$NAME"            # use (always quote)
echo "${NAME}_suffix"   # braces when text follows
export NAME             # make visible to child scripts
readonly NAME           # cannot be changed later
unset NAME              # delete
"${NAME:-default}"      # use default if empty
"${NAME:?error msg}"    # stop script if empty
```

### Special variables

```bash
$0  $1..$9  $#  $@  $?  $$  $USER  $HOME  $HOSTNAME  $PWD  $RANDOM
```

### Conditions

```bash
if [[ cond ]]; then ...; elif [[ cond ]]; then ...; else ...; fi

# numbers:  -eq -ne -gt -ge -lt -le     or (( a > b ))
# strings:  = != -z -n
# files:    -f -d -e -s -r -w -x
# logic:    && || !
```

### Loops

```bash
for x in a b c; do ...; done
for i in {1..10}; do ...; done
for (( i=0; i<10; i++ )); do ...; done
while [[ cond ]]; do ...; done
until [[ cond ]]; do ...; done
while read -r line; do ...; done < file.txt
break / continue
```

### Redirection

```bash
cmd > file        # write output to file (overwrite)
cmd >> file       # append output to file
cmd 2> file       # write errors to file
cmd &> file       # write output and errors to file
cmd > /dev/null   # discard output
cmd1 | cmd2       # send output of cmd1 into cmd2
cmd < file        # use file as input
```

### Command substitution and math

```bash
VAR=$(command)
SUM=$(( a + b ))
```

### Text tools you will use daily

| Tool | Use |
|---|---|
| `grep` | Find lines matching a pattern |
| `awk` | Pick columns, simple calculations |
| `sed` | Find and replace: `sed 's/old/new/g' file` |
| `cut` | Cut columns: `cut -d: -f1 /etc/passwd` |
| `sort` / `uniq` | Sort lines, remove duplicates: `sort file \| uniq -c` |
| `wc -l` | Count lines |
| `head` / `tail` | First / last lines; `tail -f` follows a log live |
| `xargs` | Turn input into arguments for another command |

---

## Going Pro: Habits That Matter

### 1. Start every serious script with strict mode

```bash
#!/bin/bash
set -euo pipefail
```

| Option | Effect |
|---|---|
| `-e` | Stop the script immediately if any command fails |
| `-u` | Stop if you use a variable that was never set (catches typos) |
| `-o pipefail` | A pipeline fails if any part of it fails, not just the last |

Without this, a failed `wget` in the web setup script would be ignored and the script would continue copying files that do not exist.

### 2. Use functions to organise code

```bash
log() {
  echo "[$(date '+%F %T')] $*"
}

install_packages() {
  log "Installing $*"
  sudo yum install -y "$@" > /dev/null
}

install_packages httpd wget unzip
```

Functions take arguments just like scripts (`$1`, `$2`, `$@`) and make long scripts easy to read and reuse.

### 3. Quote your variables

`"$VAR"` instead of `$VAR`. This one habit prevents most bash bugs.

### 4. Check your script with ShellCheck

```bash
sudo yum install ShellCheck     # or: sudo apt install shellcheck
shellcheck myscript.sh
```

ShellCheck finds bugs, missing quotes, and bad practices automatically. Pros run it on every script.

### 5. Debug with `-x`

```bash
bash -x myscript.sh
```

Prints every command before it runs, with variables already replaced. The fastest way to see what is going wrong. You can also turn it on and off inside a script with `set -x` and `set +x`.

### 6. Exit with meaningful codes

```bash
exit 0   # success
exit 1   # general failure
```

Other tools (cron, CI/CD pipelines, Jenkins, GitHub Actions) decide success or failure based on this number.

### 7. Make scripts idempotent

Running a script twice should be safe and give the same result. Check before you create, install, or delete.

### 8. Log what you do

Send output to a log file with a timestamp so you can investigate later:

```bash
exec > >(tee -a /var/log/myscript.log) 2>&1
```

This line at the top sends everything to both the screen and the log file.

### 9. Never hardcode secrets

No passwords or API keys inside scripts, especially if they go into Git. Use environment variables or a secrets manager.

### 10. A professional script template

```bash
#!/bin/bash
#
# Script : websetup.sh
# Purpose: Deploy a website template on CentOS or Ubuntu
# Usage  : ./websetup.sh <url> <artifact_name>
#
set -euo pipefail

URL="${1:?Usage: $0 <url> <artifact_name>}"
ART_NAME="${2:?Usage: $0 <url> <artifact_name>}"
TEMPDIR="/tmp/webfiles"

log() { echo "[$(date '+%F %T')] $*"; }

cleanup() { rm -rf "${TEMPDIR:?}"; }
trap cleanup EXIT     # always clean up, even if the script fails

if command -v yum &> /dev/null; then
  SVC="httpd"
  sudo yum install -y httpd wget unzip > /dev/null
else
  SVC="apache2"
  sudo apt-get update -qq
  sudo apt-get install -y apache2 wget unzip > /dev/null
fi

sudo systemctl enable --now "$SVC"

log "Downloading $URL"
mkdir -p "$TEMPDIR"
cd "$TEMPDIR"
wget -q "$URL"
unzip -q "$ART_NAME.zip"
sudo cp -r "$ART_NAME"/* /var/www/html/

sudo systemctl restart "$SVC"

if systemctl is-active --quiet "$SVC"; then
  log "Deployment successful"
else
  log "Deployment failed"
  exit 1
fi
```

This combines everything from the folder: variables, arguments, command substitution, if / else, exit codes, OS detection, plus the pro habits. If you can read and explain every line of this template, you have mastered the material in this folder.

### Suggested practice exercises

1. Extend `1_firstscript.sh` to print a warning when disk usage of `/` is above 80%.
2. Rewrite `11_monit.sh` using `systemctl is-active` and run it from cron every 5 minutes.
3. Change `14_for.sh` to read user names from a file and skip users that already exist.
4. Combine `multios_websetup.sh` into a single non duplicated version using functions.
5. Update `webdeploy.sh` to log OK / FAILED per host and print a summary at the end.
