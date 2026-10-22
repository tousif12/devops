# Linux Text Commands — Small Examples for DevOps

A beginner-friendly guide to `grep`, `head`, `tail`, `less`, `more`, `cut`, `awk`, `sed`, and Vim text replacement.

Run terminal commands on Linux. Vim commands go **inside Vim** after pressing `Esc`.

## 1. Quick overview

| Tool | What it does | Simple example | How to exit |
|---|---|---|---|
| `grep` | Finds matching lines | `grep 'ERROR' app.log` | Finishes automatically for ordinary files |
| `head` | Shows the beginning | `head -n 5 app.log` | Finishes automatically |
| `tail` | Shows the end | `tail -n 5 app.log` | Finishes automatically |
| `tail -f` | Watches newly appended lines | `tail -f app.log` | `Ctrl+C` |
| `less` | Opens a scrollable viewer | `less app.log` | `q` |
| `more` | Views text a screen at a time | `more app.log` | `q` |
| `cut` | Extracts fields using a separator | `cut -d ':' -f 1 users.txt` | Finishes automatically |
| `awk` | Extracts fields, filters rows, calculates | `awk -F ':' '{print $1}' users.txt` | Finishes automatically |
| `sed` | Transforms text, including replacements | `sed 's/old/new/g' file.txt` | Finishes automatically |
| Vim | Edits files interactively | `vim file.txt` | `Esc`, then `:q`, then Enter |

**If a terminal command is still running:** press `Ctrl+C` to stop it. A normal command such as `grep` can wait for keyboard input if you omit the filename and provide no pipe; `Ctrl+D` ends that input, while `Ctrl+C` cancels.

## 2. Create a small practice file

Copy this whole block into your Linux terminal. It creates `app.log` in your current directory.

```bash
cat > app.log <<'LOG'
INFO Application started
INFO User ali logged in
ERROR Database connection failed
WARN Disk space low
ERROR Request timed out
INFO Application stopped
LOG
```

Use a practice folder so you do not overwrite an existing `app.log`.

## 3. grep — find matching lines

Think: **“Show me the lines containing this text.”** Searching does not change the file.

### Find ERROR

```bash
grep 'ERROR' app.log
```

Output:

```text
ERROR Database connection failed
ERROR Request timed out
```

### Ignore uppercase/lowercase: -i

```bash
grep -i 'error' app.log
```

Finds `error`, `ERROR`, or `Error`.

### Show line numbers: -n

```bash
grep -n 'ERROR' app.log
```

Output:

```text
3:ERROR Database connection failed
5:ERROR Request timed out
```

### Exclude matching lines: -v

```bash
grep -v 'INFO' app.log
```

Shows the ERROR and WARN lines; hides the INFO lines.

### Count matching lines: -c

```bash
grep -c 'ERROR' app.log
```

Output: `2`. This counts **lines**, not individual occurrences within a line.

### Match a whole word: -w

```bash
grep -w 'ali' app.log
```

Matches the word `ali`, rather than a longer word containing those letters.

### Search for either of two patterns: -E

```bash
grep -E 'ERROR|WARN' app.log
```

The `|` inside the quotes means **OR** here.

### Search literal text: -F

```bash
grep -F '192.168.1.9' app.log
```

Use `-F` when you want ordinary text matching. Without it, grep treats patterns as regular expressions; for example, `.` means any character.

### Show two lines after a match: -A

```bash
grep -A 2 'Database' app.log
```

Shows the matching line plus the next two lines. Use `-B 2` for two lines before, or `-C 2` for two lines on each side.

### Search inside a directory: -r

```bash
grep -rn 'ERROR' ./logs/
```

Searches files under `./logs/`, including subdirectories, and shows line numbers. The directory must exist.

### Show only matching filenames: -l

```bash
grep -l 'ERROR' *.log
```

Useful when you want to know **which log files** contain errors.

## 4. head — first lines

```bash
head app.log
```

Shows the first **10 lines**, or the whole file if it is shorter.

```bash
head -n 3 app.log
```

Shows only the first three lines. Useful for checking a file's format or header.

**Exit:** automatic; this does not open an editor.

## 5. tail — last lines

```bash
tail app.log
```

Shows the last **10 lines**.

```bash
tail -n 3 app.log
```

Shows the last three lines. Useful for seeing the most recent log entries.

### Professional tip: watch logs appear live

```bash
tail -f app.log
```

It displays the last 10 lines, then keeps running and shows new lines as they are appended.

**Try it:** leave that terminal open. In a second terminal, in the same directory, run:

```bash
echo 'ERROR New live test message' >> app.log
```

The new message appears in the first terminal.

**Stop watching:** press `Ctrl+C`. This stops your viewer; it does not stop the application writing the log.

### Real server examples

On systems that have this log file, such as many RHEL/CentOS setups:

```bash
sudo tail -f /var/log/messages
```

**Path correction:** `/var/log/messages`, not `/var/logs/messages`.

On Ubuntu systems configured to write a syslog file:

```bash
sudo tail -f /var/log/syslog
```

These files depend on logging configuration and may not exist. For the systemd journal, use:

```bash
sudo journalctl -f
```

Stop with `Ctrl+C`.

### Extra tip: survive log rotation with -F

```bash
tail -F app.log
```

`-F` follows the **filename** and retries if the file disappears. Useful when a logging service renames the old log and creates a new file with the same name.

### Watch only new ERROR lines

```bash
tail -F app.log | grep --line-buffered 'ERROR'
```

`tail` watches; `grep` filters. `--line-buffered` helps matching lines appear promptly. Stop the pipeline with `Ctrl+C`.

## 6. less — browse a file comfortably

```bash
less app.log
```

It opens a viewer, not an editor. Especially useful for large logs.

| Key | Action |
|---|---|
| Down / Up arrow | Move one line |
| `Space` | Move forward one screen |
| `b` | Move back one screen |
| `g` | Go to the beginning |
| `G` (`Shift+g`) | Go to the end |
| `/ERROR`, then Enter | Search forward for ERROR |
| `n` | Next match in the search direction |
| `N` | Match in the opposite direction |
| `q` | Quit and return to the terminal |

**Useful tip:** `Shift+f` starts watching appended data inside `less`. Press `Ctrl+C` to return to browsing, then `q` to quit.

## 7. more — view one screen at a time

```bash
more app.log
```

| Key | Action |
|---|---|
| `Space` | Next screen |
| `Enter` | Next line |
| `/ERROR`, then Enter | Search forward |
| `q` | Quit |

It may exit automatically at the end, depending on its implementation and options. Short files may print and finish immediately. For convenient forward/backward navigation, use `less`.

## 8. cut — get usernames from colon-separated text

A **delimiter** is the character separating fields. Here, it is `:`.

Create a practice file:

```bash
cat > users.txt <<'USERS'
ali:x:1000:1000:Ali:/home/ali:/bin/bash
sara:x:1001:1001:Sara:/home/sara:/bin/bash
ahmed:x:1002:1002:Ahmed:/home/ahmed:/bin/sh
USERS
```

Each line has these fields:

| Field | Meaning | Example |
|---|---|---|
| 1 | Username | `ali` |
| 2 | Password placeholder | `x` |
| 3 | User ID (UID) | `1000` |
| 4 | Group ID (GID) | `1000` |
| 5 | Comment / display name | `Ali` |
| 6 | Home directory | `/home/ali` |
| 7 | Login shell | `/bin/bash` |

### Extract only usernames

```bash
cut -d ':' -f 1 users.txt
```

Output:

```text
ali
sara
ahmed
```

- `-d ':'`: split using the colon delimiter.
- `-f 1`: print field number 1.

### Extract usernames from the real local account file

```bash
cut -d ':' -f 1 /etc/passwd
```

This includes **system/service accounts** as well as human accounts. It lists entries in that file, not necessarily every account available through network identity services. Reading the file does not modify it.

### Extract username and shell

```bash
cut -d ':' -f 1,7 users.txt
```

Output:

```text
ali:/bin/bash
sara:/bin/bash
ahmed:/bin/sh
```

### Extract a range of fields

```bash
cut -d ':' -f 1-3 users.txt
```

Prints fields 1, 2, and 3.

**Remember:** `cut` is convenient for a known single-character delimiter. For columns separated by varying spaces, use `awk`. A basic comma split is not a full CSV parser if fields contain quoted commas.

## 9. awk — choose columns and filter rows

Think: **“For each line, split it into fields, then do something.”**

By default, `awk` separates fields on whitespace. `$1` means the first field, `$2` the second, and `$0` the whole line.

### Show the first word of each log line

```bash
awk '{print $1}' app.log
```

For the original six-line practice log, output is:

```text
INFO
INFO
ERROR
WARN
ERROR
INFO
```

### Extract colon-separated usernames

```bash
awk -F ':' '{print $1}' users.txt
```

`-F ':'` sets the field separator to a colon. This produces the same usernames as the earlier `cut` example.

### Print username and home directory

```bash
awk -F ':' '{print $1, $6}' users.txt
```

Output:

```text
ali /home/ali
sara /home/sara
ahmed /home/ahmed
```

The comma between `$1` and `$6` prints them with the default output separator: a space.

### Filter users with the Bash shell

```bash
awk -F ':' '$7 == "/bin/bash" {print $1}' users.txt
```

Output:

```text
ali
sara
```

Meaning: **if field 7 equals `/bin/bash`, print field 1**.

### Filter ERROR rows

```bash
awk '$1 == "ERROR" {print $0}' app.log
```

Prints whole lines whose first field is exactly `ERROR`.

### Add line numbers

```bash
awk '{print NR, $0}' app.log
```

`NR` is the record number, normally the line number for these text files.

### Tiny calculation

```bash
printf 'api 100\ndb 200\n' | awk '{total += $2} END {print total}'
```

Output: `300`. It adds the second column, then prints the total after reading all rows.

**When to use which:** `grep` for matching lines; `cut` for simple delimiter fields; `awk` for fields plus conditions or calculations.

## 10. Pipe — connect commands

A pipe `|` sends one command's output to the next command's input.

### Show the last two matching ERROR lines

```bash
grep 'ERROR' app.log | tail -n 2
```

First find ERROR lines, then keep the last two matches.

### Browse search results

```bash
grep -n 'ERROR' app.log | less
```

Press `q` to leave the viewer.

### Show the first two usernames

```bash
cut -d ':' -f 1 users.txt | head -n 2
```

Output:

```text
ali
sara
```

## 11. Vim — replace text professionally

Open a **practice copy**:

```bash
cp app.log vim-practice.log
vim vim-practice.log
```

Press `Esc` before typing each colon command, and press **Enter** to execute it.

### Replace the first match on the current line

```vim
:s/ERROR/FAILURE/
```

### Replace every match on the current line

```vim
:s/ERROR/FAILURE/g
```

### Replace the first match on each line throughout the file

```vim
:%s/ERROR/FAILURE/
```

### Replace every match throughout the file

```vim
:%s/ERROR/FAILURE/g
```

| Part | Meaning |
|---|---|
| `:` | Enter command-line mode |
| `%` | Apply to every line in the file |
| `s` | Substitute / replace |
| `ERROR` | Text/pattern to find |
| `FAILURE` | Replacement text |
| `g` | Replace all matches **within each selected line** |

**Important:** `%` selects the whole file. `g` handles repeated matches on a line. They do different jobs.

For example, with a line containing `ERROR ERROR`, the command without `g` changes only its first match under Vim's usual defaults. These examples assume the default `nogdefault` setting; customized Vim settings can change this behavior.

### Best tip: confirm each replacement

```vim
:%s/ERROR/FAILURE/gc
```

- `y`: replace this match.
- `n`: skip it.
- `a`: replace this and all remaining matches.
- `q`: stop replacing.

### Ignore uppercase/lowercase

```vim
:%s/error/FAILURE/gi
```

### Replace only a whole word

```vim
:%s/\<old\>/new/g
```

Replaces the word `old`, without replacing the `old` inside `folder`.

### Replace paths using another separator

```vim
:%s#/old/path#/new/path#g
```

Using `#` makes paths easier to read than escaping each slash.

### Undo, save, and quit

| What you want | Keys / command |
|---|---|
| Undo the last change | `Esc`, then lowercase `u` |
| Save | `Esc`, then `:w`, Enter |
| Save and quit | `Esc`, then `:wq`, Enter |
| Quit if there are no unsaved changes | `Esc`, then `:q`, Enter |
| Quit and discard unsaved changes | `Esc`, then `:q!`, Enter |

Lowercase `u` is the usual undo key. Uppercase `U` has a different purpose: undoing changes on the current line.

## 12. sed — preview replacements, then save

`sed` transforms text from a file or pipeline. It does **not** open an editor.

Create a small practice file:

```bash
printf 'oldtext oldtext\nkeep this\n' > replace-demo.txt
```

### Preview the replacement — original file stays unchanged

```bash
sed 's/oldtext/newtext/g' replace-demo.txt
```

Output:

```text
newtext newtext
keep this
```

This prints the transformed **whole file** to your terminal. It does not save the changes.

### Confirm the original is unchanged

```bash
cat replace-demo.txt
```

It still contains `oldtext oldtext`.

### Save changes in the file on Linux

```bash
sed -i 's/oldtext/newtext/g' replace-demo.txt
```

`-i` means edit the file in place. This modifies the file on disk.

### Professional tip: save with a backup

After checking your preview, use this **instead of** the previous save command:

```bash
sed -i.bak 's/oldtext/newtext/g' replace-demo.txt
```

- `replace-demo.txt`: updated file.
- `replace-demo.txt.bak`: original file before this command.

Run the backup command first if you want to preserve the original `oldtext` version. Repeated runs can overwrite an existing `.bak` backup.

### First match per line versus all matches

```bash
sed 's/oldtext/newtext/' replace-demo.txt
```

Replaces the first match on **each line** in the printed output.

```bash
sed 's/oldtext/newtext/g' replace-demo.txt
```

Replaces every match on each line in the printed output.

Unlike Vim, `sed` processes every input line by default; it does not need `%`.

### Replace a path cleanly

```bash
sed 's#/old/path#/new/path#g' config.txt
```

Preview only. Replace `config.txt` with your actual file.

### Save into a separate file

```bash
sed 's/oldtext/newtext/g' replace-demo.txt > updated-demo.txt
```

Keeps the input file unchanged and writes the result into a different file.

**Avoid this:** `sed 's/old/new/g' file.txt > file.txt`. The shell truncates the input file before `sed` reads it. Use `-i` or a different output filename.

**Pattern reminder:** Vim and `sed` substitutions use regular expressions. Search text such as `.` or `*` has special meaning; these examples use simple patterns. In replacement text, `&` represents the matched text; use `\&` for a literal ampersand. The `sed -i` examples here target Linux/GNU sed; macOS uses different syntax for editing without a backup.

## 13. Vim versus sed — same idea, different place

| Task | Inside Vim | Linux terminal |
|---|---|---|
| Replace all matches across a file | `:%s/old/new/g` | `sed 's/old/new/g' file.txt` prints a preview |
| Review each match | `:%s/old/new/gc` | Preview the output before using `-i` |
| Save changes | `:w` | `sed -i 's/old/new/g' file.txt` |
| Preserve a backup while editing | Save a separate copy first | `sed -i.bak 's/old/new/g' file.txt` |

## 14. Pocket cheat sheet

```bash
# Find lines containing ERROR
grep 'ERROR' app.log

# Ignore case and show line numbers
grep -in 'error' app.log

# First 5 lines
head -n 5 app.log

# Last 5 lines
tail -n 5 app.log

# Watch live logs; Ctrl+C stops
tail -F app.log

# Browse a file; q quits
less app.log

# View one screen at a time; q quits
more app.log

# Extract local account usernames
cut -d ':' -f 1 /etc/passwd

# Extract usernames with awk
awk -F ':' '{print $1}' /etc/passwd

# Preview replacement
sed 's/oldtext/newtext/g' replace-demo.txt

# Save replacement and keep a backup
sed -i.bak 's/oldtext/newtext/g' replace-demo.txt
```

Inside Vim, press `Esc`, type the following, and press Enter:

```vim
:%s/oldtext/newtext/gc
```

Then `:wq` to save and quit, or `:q!` to discard unsaved changes.
