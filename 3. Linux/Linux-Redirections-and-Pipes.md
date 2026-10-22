# Linux Redirections and Pipes — Easy Notes

Small examples, our questions and answers, and a practical system report.
Examples use the Bash shell on Linux.

## 1. What is redirection?

**Redirection changes where a command sends its output or gets its input.**
Normally, a command displays its output on your terminal:

```bash
echo "Hello"
```

Screen output:

```text
Hello
```

You can send that output into a file instead.

## 2. `>` — Save output, replacing old contents

```bash
echo "Hello" > notes.txt
```

This saves `Hello` inside `notes.txt`. Nothing is printed on the terminal.

- If the file does not exist, it is created.
- If it exists, its previous contents are replaced.

Read the file:

```bash
cat notes.txt
```

Output:

```text
Hello
```

## 3. `>>` — Add output to the end

```bash
echo "Learning Linux" >> notes.txt
```

Now the file contains:

```text
Hello
Learning Linux
```

**Remember: `>` replaces; `>>` adds.** Both create the file if it does not exist.

## 4. Why do we use numbers such as `2`?

Linux commands have three standard streams. Think of them as three channels:

| Number | Name | Easy meaning | Usual source or destination |
|---|---|---|---|
| `0` | Standard input (stdin) | What the command reads | Keyboard |
| `1` | Standard output (stdout) | Normal results | Terminal |
| `2` | Standard error (stderr) | Error messages | Terminal |

`>` is shorthand for `1>`: save normal output.

`2>` saves error output. `2>>` appends error output.

The number must touch the operator: write `2>>`, not `2 >>`.

## 5. `2>>` — Append only errors

```bash
ls /missing-folder 2>> errors.txt
```

Break it into separate parts:

| Part | Meaning |
|---|---|
| `ls` | List files in a folder |
| `/missing-folder` | An example absolute folder path; assume it does not exist |
| `2>>` | Append error messages |
| `errors.txt` | The file receiving the errors |

**The `2` is not part of the folder name.** There is no folder called “missing folder 2” in this command.

Because the folder does not exist, `ls` produces an error similar to:

```text
ls: cannot access '/missing-folder': No such file or directory
```

That error goes into `errors.txt` instead of appearing on the terminal.

Read the saved error:

```bash
cat errors.txt
```

### Our question: What if the folder exists?

For example, `/tmp` normally exists:

```bash
ls /tmp 2>> errors.txt
```

If listing succeeds:

- Folder contents appear on the terminal.
- No error is appended to `errors.txt`.
- Existing contents of `errors.txt` stay unchanged.
- If `errors.txt` does not exist, the shell still creates an empty file.

An existing folder can still produce an error, such as `Permission denied`. That error would be appended.

**`2>>` catches errors, not successful results.**

## 6. `&>>` — Append normal output AND errors

In Bash:

```bash
ls /etc/passwd /missing-folder &>> results.txt
```

Assuming `/etc/passwd` exists and `/missing-folder` does not, both the successful listing and the error go into `results.txt`.

Example contents (order can vary):

```text
ls: cannot access '/missing-folder': No such file or directory
/etc/passwd
```

Nothing from these two streams appears on the terminal if the redirection succeeds.

Related forms:

| Operator | What goes into the file? | Existing contents |
|---|---|---|
| `>` or `1>` | Normal output | Replaced |
| `>>` or `1>>` | Normal output | Kept; new output added |
| `2>` | Errors | Replaced |
| `2>>` | Errors | Kept; new errors added |
| `&>` | Both normal output and errors | Replaced |
| `&>>` | Both normal output and errors | Kept; new output added |

## 7. `<` — Read input from a file

```bash
wc -l < notes.txt
```

- `< notes.txt` supplies the file contents as input.
- `wc -l` counts newline characters (normally one per text line).
- For our two-line `notes.txt`, the result is `2`.

**`<` brings input from a file; `>` sends output to a file.**

## 8. The document's `sed` example

Suppose `devopstools` contains:

```text
tech for deployment
tech for monitoring
```

Run:

```bash
sed 's/tech/tools/g' devopstools > updated-tools.txt
```

This means:

1. Read `devopstools`.
2. Replace every occurrence of `tech` with `tools` on each line.
3. Save the changed output into `updated-tools.txt`.

New file contents:

```text
tools for deployment
tools for monitoring
```

**The original `devopstools` file stays unchanged.**

Do not use the same input and output filename with `>`:

```bash
# Do not run this: the shell empties the file before sed reads it.
sed 's/tech/tools/g' devopstools > devopstools
```

## 9. What is a pipe: `|`?

**A pipe gives one command's normal output to another command as input.**

Read this pattern as “do the first job, then let the second command process its result”:

```bash
first-command | second-command
```

The pipe passes data. It does not automatically save a file.

### Example A: Find Sara in a list

Suppose `names.txt` contains:

```text
Ali
Sara
Ahmed
```

Run:

```bash
cat names.txt | grep "Sara"
```

What happens:

1. `cat names.txt` reads all three names.
2. `|` gives all those lines to `grep`.
3. `grep "Sara"` shows only lines containing `Sara`.

Final screen output:

```text
Sara
```

**Analogy:** Hand a list to a friend and say, “Find Sara in this list.”

Here, `cat` provides the list, the pipe hands it over, and `grep` searches it.

For this particular task, you can also use `grep "Sara" names.txt` directly. We used `cat` to make the pipe easy to see.

### Example B: Count directory entries

```bash
ls -1 | wc -l
```

The option `-1` is the digit one: list one name per line.

Suppose `ls -1` produces:

```text
notes.txt
app.py
images
```

The pipe sends those three lines to `wc -l`, which counts them.

Final output:

```text
3
```

Read it as: **“List the names, then give that list to the line counter.”**

This simple example counts listed entries, including directories. By default, `ls` excludes hidden entries. Unusual filenames containing newlines can make this approach unsuitable for precise scripting.

### Pipe versus redirection

| Command | What happens? |
|---|---|
| `cat names.txt > copy.txt` | Save the names into a file |
| `cat names.txt \| grep "Sara"` | Give the names to another command for filtering |
| `cat names.txt \| grep "Sara" > matches.txt` | Filter the names, then save the matching lines |

A plain pipe passes normal output (stream `1`). Error messages (stream `2`) still go to the terminal unless redirected separately.

## 10. Your screenshot: Build a system report

Your screenshot demonstrates writing several commands' results into `/tmp/sysinfo.txt`.

![Terminal example: date, uptime, memory, and disk information redirected into a system report](redirection-output-example.png)

*The screenshot shows the commands being entered. The complete report output below is an illustrative sample, not output visible in the screenshot.*

### Step 1: Start a fresh report

```bash
echo "########################################" > /tmp/sysinfo.txt
```

The first `>` creates the file or clears an earlier report.

### Step 2: Append the date and time

```bash
date >> /tmp/sysinfo.txt
```

### Step 3: Append uptime

```bash
echo "########################################" >> /tmp/sysinfo.txt
uptime >> /tmp/sysinfo.txt
```

`uptime` shows how long the machine has been running, logged-in user count, and load averages.

### Step 4: Append memory information

```bash
echo "########################################" >> /tmp/sysinfo.txt
free -m >> /tmp/sysinfo.txt
```

`free -m` shows memory and swap figures in mebibytes (MiB).

### Step 5: Append disk information

```bash
echo "########################################" >> /tmp/sysinfo.txt
df -h >> /tmp/sysinfo.txt
echo "########################################" >> /tmp/sysinfo.txt
```

`df -h` shows filesystem space in readable units such as G and M.

### Step 6: Display the completed report

```bash
cat /tmp/sysinfo.txt
```

Illustrative sample output; your date, devices, and numbers will differ:

```text
########################################
Fri Oct 23 09:00:00 PKT 2026
########################################
 09:00:00 up 2 hours, 1 user, load average: 0.05, 0.08, 0.10
########################################
              total        used        free      shared  buff/cache   available
Mem:           2048         700         800          10         548        1200
Swap:          1024           0        1024
########################################
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        20G  6.0G   14G  30% /
########################################
```

**Why `>>` after the first line?** Each command adds its result to the same report. Using `>` repeatedly would replace the previous results each time.

`/tmp` is temporary storage; use a file in your home directory if you want to keep a report long term.

## 11. Quick revision

| Symbol | Remember it as |
|---|---|
| `>` | Save normal output; replace old contents |
| `>>` | Add normal output to the end |
| `2>` | Save errors; replace old contents |
| `2>>` | Add errors to the end |
| `&>` | Save normal output and errors; replace old contents (Bash) |
| `&>>` | Add normal output and errors to the end (Bash) |
| `<` | Get input from a file |
| `\|` | Give normal output to another command |

**One sentence to remember:** Redirection connects commands to files; a pipe connects one command's output to another command's input.



## 12. Practical Examples: Redirection and Pipes

### Example 1: Save system information into one file

```bash
echo "===== DATE =====" > /tmp/sysinfo.txt
date >> /tmp/sysinfo.txt

echo "===== UPTIME =====" >> /tmp/sysinfo.txt
uptime >> /tmp/sysinfo.txt

echo "===== MEMORY =====" >> /tmp/sysinfo.txt
free -m >> /tmp/sysinfo.txt

echo "===== DISK SPACE =====" >> /tmp/sysinfo.txt
df -h >> /tmp/sysinfo.txt
```

**What happens?**

- The first `>` starts a fresh report, replacing any existing contents.
- Each `>>` adds more information to the same file.
- `echo` adds headings to make the report readable.

Display the completed report:

```bash
cat /tmp/sysinfo.txt
```

**Use case:** Collect a server’s date, uptime, memory usage, and disk space into one report.

### Example 2: Search the last 20 log lines

```bash
tail -20 /var/log/messages | grep -i vagrant
```

| Part | Easy meaning |
|---|---|
| `tail -20 /var/log/messages` | Read the last 20 lines of the log |
| `\|` | Give those lines to the next command |
| `grep -i vagrant` | Show only lines containing `vagrant`, ignoring letter case |

**Sample output from your screenshot:**

```text
Dec 6 08:06:48 localhost systemd: Removed slice User Slice of vagrant.
Dec 6 08:07:00 localhost systemd: Created slice User Slice of vagrant.
Dec 6 08:07:00 localhost systemd: Started Session 7 of user vagrant.
```

**Read it as:** “Get the last 20 log lines, then show only those mentioning vagrant.”

It searches **only those last 20 lines**, so you may get fewer than 20 matches—or none.

### Bonus: Monitor matching logs live

```bash
tail -f /var/log/messages | grep --line-buffered -i vagrant
```

- `tail -f` keeps watching the log for new lines.
- The pipe sends those lines to `grep`.
- `grep` displays only lines mentioning `vagrant`.
- `--line-buffered` makes `grep` print matching lines promptly.

Press **Ctrl+C** to stop.

**Log location:** `/var/log/messages` is common on CentOS/RHEL systems. On Ubuntu, a common equivalent is `/var/log/syslog`, when present.



## 13. Find Command — Search for Files

**Basic structure:**

```bash
find WHERE -type f -name "FILENAME"
```

- `WHERE`: Where to search.
- `-type f`: Search for files only.
- `-name`: Match the filename; case-sensitive.

`find` searches inside the chosen directory **and its subdirectories**.

### Example 1: Search inside a specific directory

Find `notes.txt` anywhere inside `/home`:

```bash
find /home -type f -name "notes.txt"
```

### Example 2: Search inside the current directory

`.` means your current directory:

```bash
find . -type f -name "notes.txt"
```

### Example 3: Search the whole Linux system

`/` means start searching from the root directory:

```bash
sudo find / -type f -name "notes.txt"
```

This can take time. Some protected or virtual locations may still produce errors.

### Example 4: Find a file by its exact name

```bash
find /tmp -type f -name "report.pdf"
```

Matches `report.pdf`, but not `report-old.pdf` or `REPORT.PDF`.

To ignore letter case:

```bash
find /tmp -type f -iname "report.pdf"
```

This also matches names such as `REPORT.PDF`.

### Example 5: Find files whose names start with a word

Find files starting with `report`:

```bash
find . -type f -name "report*"
```

Possible matches:

```text
./report.pdf
./report-old.txt
./reports/report2026.csv
```

`*` means **zero or more characters**.

### Example 6: Find only files with a `.pdf` extension

```bash
find . -type f -name "*.pdf"
```

Possible matches:

```text
./notes.pdf
./documents/linux.pdf
./report.pdf
```

To include uppercase extensions such as `.PDF`:

```bash
find . -type f -iname "*.pdf"
```

This matches filenames ending in `.pdf`, ignoring case. It does not inspect whether the contents are actually PDF data.

### Bonus: Find PDF files starting with `report`

```bash
find . -type f -iname "report*.pdf"
```

Possible matches:

```text
./report.pdf
./report-final.pdf
./reports/Report2026.PDF
```

### Quick Revision

| Requirement | Command |
|---|---|
| Search a directory and its subdirectories | `find /home -type f -name "notes.txt"` |
| Search the current directory and its subdirectories | `find . -type f -name "notes.txt"` |
| Search the whole system | `sudo find / -type f -name "notes.txt"` |
| Match an exact filename | `find . -type f -name "report.pdf"` |
| Match names starting with `report` | `find . -type f -name "report*"` |
| Match `.pdf` filenames | `find . -type f -name "*.pdf"` |
| Ignore letter case | `find . -type f -iname "*.pdf"` |

**Tip:** Keep patterns such as `"*.pdf"` inside quotes so `find` receives the pattern correctly.

