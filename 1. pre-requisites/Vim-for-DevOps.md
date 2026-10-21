# Vim for DevOps — Easy Practical Guide

A beginner-friendly guide for editing Linux configuration files, scripts, YAML, Dockerfiles, and Git commit messages.

> **Learn this first:** Open a file → press `i` → edit → press `Esc` → type `:wq` → press `Enter`.
>
> **Stuck inside Vim?** Press `Esc`, type `:q!`, and press `Enter` to exit without saving your unsaved changes.

## Contents

1. [Why DevOps engineers use Vim](#1-why-devops-engineers-use-vim)
2. [Check and open Vim](#2-check-and-open-vim)
3. [Understand Vim modes](#3-understand-vim-modes)
4. [Your first file: step by step](#4-your-first-file-step-by-step)
5. [Save and exit](#5-save-and-exit)
6. [Move around a file](#6-move-around-a-file)
7. [Insert and change text](#7-insert-and-change-text)
8. [Delete, copy, and paste](#8-delete-copy-and-paste)
9. [Undo and redo](#9-undo-and-redo)
10. [Search and replace](#10-search-and-replace)
11. [Select text and indent](#11-select-text-and-indent)
12. [YAML and pasted configuration](#12-yaml-and-pasted-configuration)
13. [Work with multiple files](#13-work-with-multiple-files)
14. [Edit a real configuration safely](#14-edit-a-real-configuration-safely)
15. [Common DevOps examples](#15-common-devops-examples)
16. [Permissions, swap files, and common problems](#16-permissions-swap-files-and-common-problems)
17. [Simple Vim settings](#17-simple-vim-settings)
18. [Practice lab](#18-practice-lab)
19. [Quick cheat sheet](#19-quick-cheat-sheet)

## 1. Why DevOps engineers use Vim

Vim is a terminal text editor. You can edit files directly on a Linux server, including through SSH, without a graphical desktop.

| DevOps task | Example file | What you might change |
| --- | --- | --- |
| Web server configuration | `/etc/nginx/nginx.conf` | Logging or server settings |
| Shell automation | `backup.sh` | Backup commands |
| Container image definition | `Dockerfile` | Base image and build steps |
| Container services | `compose.yaml` | Ports and environment settings |
| Kubernetes configuration | `deployment.yaml` | Replicas and container settings |
| Infrastructure as code | `main.tf` | Terraform resource definitions |
| Git commit message | `.git/COMMIT_EDITMSG` | Description of a change |
| Local name resolution | `/etc/hosts` | Hostname-to-IP mappings |

**Simple example:** SSH into Ubuntu → open a configuration in Vim → change it → save → validate the configuration → reload the service if appropriate.

Vim only edits text. Saving a file does not automatically apply a service configuration or Kubernetes manifest.

## 2. Check and open Vim

Run these commands in your **Linux terminal**, outside Vim.

Check whether Vim is available:

```bash
vim --version
```

If it is missing, install it using your distribution's package manager.

Ubuntu / Debian:

```bash
sudo apt update
sudo apt install vim
```

CentOS Stream / RHEL / Rocky Linux / AlmaLinux with DNF:

```bash
sudo dnf install vim-enhanced
```

Older systems that use YUM:

```bash
sudo yum install vim-enhanced
```

Open an existing file or start a new one:

```bash
vim notes.txt
```

If `notes.txt` does not exist, Vim starts a new buffer. The file is created when you save it. Its parent directory must already exist.

Open directly at line 25:

```bash
vim +25 deployment.yaml
```

Open in read-only mode for inspection:

```bash
vim -R deployment.yaml
```

Read-only mode helps prevent accidental writes; it is not a security boundary.

**Vim versus vi:** `vi` is often available on minimal servers, but it may be a smaller editor or a link to Vim. Basic editing commands are similar; this guide targets Vim.

## 3. Understand Vim modes

Think of Vim as having a **control mode** and a **typing mode**.

| Mode | Purpose | How to enter |
| --- | --- | --- |
| Normal | Move, delete, copy, undo, and run editing actions | Press `Esc` |
| Insert | Type text into the file | From Normal mode, press `i` |
| Visual | Select characters or lines | From Normal mode, press `v` or `V` |
| Command-line | Save, quit, replace text, or change settings | From Normal mode, press `:` |

Vim normally starts in **Normal mode**. This is why typing immediately may execute commands instead of adding text.

### How to read commands in this guide

- `Esc`: press the Escape key.
- `Ctrl+r`: hold Ctrl and press r.
- `dd`: press d twice, one after the other.
- `:w`: press `Esc`, type `:w`, then press `Enter`.
- `/error`: press `Esc`, type `/error`, then press `Enter`.
- Uppercase matters: `o` and `O` perform different actions.
- Unless stated otherwise, editing shortcuts start in **Normal mode**.

> If you lose track of the mode, press `Esc` before your next command.

## 4. Your first file: step by step

### Step 1 — Open a practice file

In the terminal:

```bash
vim devops-notes.txt
```

### Step 2 — Enter typing mode

Press:

```text
i
```

### Step 3 — Type these lines

```text
I am learning Vim.
I will use Vim to edit Linux configuration files.
```

### Step 4 — Stop typing

Press `Esc` to return to Normal mode.

### Step 5 — Save and exit

Type this, then press `Enter`:

```vim
:wq
```

### Step 6 — Check the saved content

Back in the terminal:

```bash
cat devops-notes.txt
```

**Remember:** `i` means start typing; `Esc` means return to controls; `:wq` means write and quit.

## 5. Save and exit

Press `Esc` before any command below. Press `Enter` after commands beginning with `:`.

| Command | Meaning | Example use |
| --- | --- | --- |
| `:w` | Save the current file and stay in Vim | Save progress |
| `:q` | Quit the current window; refuses if unsaved changes would be lost | Exit after inspection |
| `:wq` | Save and quit the current window | Finish editing |
| `:x` | Save if modified, then quit | Finish editing |
| `:q!` | Quit the current window without saving its unsaved changes | Abandon a mistaken edit |
| `:w copy.txt` | Write the buffer to another path | Save a copy; usually keeps the original buffer name |
| `:saveas copy.txt` | Save under another name and switch to that file | Rename your working destination |
| `:qa` | Quit all windows; refuses if changes would be lost | Close a multi-file session |

`!` means force the operation where Vim allows it. It **does not grant Linux file permissions**.

If a file has several open windows, quitting one window may leave Vim running. `:q!` does not undo changes already written to disk.

## 6. Move around a file

Arrow keys work in typical Vim setups. These shortcuts are useful over SSH and in large files.

| Key | Action | DevOps use |
| --- | --- | --- |
| `h` / `j` / `k` / `l` | Left / down / up / right | Navigate without leaving the keyboard |
| `w` | Next word start | Move through configuration values |
| `b` | Previous word start | Return to a setting name |
| `0` | Start of line | Go to the first column |
| `^` | First non-blank character | Skip indentation |
| `$` | End of line | Append a value |
| `gg` | First line | Inspect the file header |
| `G` | Last line | Jump to the bottom |
| `25G` | Line 25 | Find a line reported in an error |
| `:25` | Line 25, after Enter | Another way to jump to a line |
| `Ctrl+d` | Down about half a screen | Browse a large file |
| `Ctrl+u` | Up about half a screen | Browse backward |
| `%` | Jump to a matching bracket when on a bracket | Inspect paired braces or parentheses |

Show line numbers:

```vim
:set number
```

Hide them:

```vim
:set nonumber
```

Counts repeat many actions: `5j` moves down five lines; `3w` moves forward three words.

## 7. Insert and change text

| Key | Action | Example |
| --- | --- | --- |
| `i` | Insert before the cursor | Edit a value |
| `a` | Insert after the cursor | Extend a word |
| `I` | Insert at the first non-blank character | Add a comment marker at line start |
| `A` | Insert at the end of the line | Append a flag |
| `o` | Open a new line below and enter Insert mode | Add a configuration entry |
| `O` | Open a new line above and enter Insert mode | Add a comment |
| `r` then a character | Replace one character | Change a single digit |
| `ciw` | Change the word under the cursor | Replace a setting name |
| `cw` | Change from the cursor through the relevant word text | Replace text from your current position |
| `cc` | Change the current line | Rewrite one setting |
| `C` | Change from the cursor to the end of the line | Rewrite the rest of a command |

Example: change `replicas: 2` to `replicas: 3`.

1. Search for `/replicas` and press `Enter`.
2. Press `$` to reach the final digit, assuming it is the last character.
3. Press `r`, then `3`.
4. Type `:w` and press `Enter`.

**Difference:** `r` replaces one character and stays in Normal mode. Commands such as `ciw`, `cc`, and `C` enter Insert mode; press `Esc` when finished.

## 8. Delete, copy, and paste

In Vim, copying is called **yanking**. Deleting usually stores the removed text in a register so it can be pasted.

| Command | Action |
| --- | --- |
| `x` | Delete the character under the cursor |
| `dd` | Delete the current line |
| `3dd` | Delete three lines starting at the current line |
| `diw` | Delete the word under the cursor |
| `dw` | Delete forward to the next word boundary; whitespace may also be removed |
| `D` | Delete from the cursor to the end of the line |
| `yy` | Copy the current line |
| `3yy` | Copy three lines starting at the current line |
| `p` | Paste after the cursor, or below for whole lines |
| `P` | Paste before the cursor, or above for whole lines |
| `.` | Repeat the last change |

### Duplicate a configuration line

1. Move to the line.
2. Press `yy`.
3. Press `p`.
4. Edit the new line.

### Move a line

1. Press `dd` on the line.
2. Move to the destination.
3. Press `p` to place it below that line.

### Vim copy versus your computer's clipboard

`yy` copies inside Vim. It does not necessarily copy to the Windows clipboard, especially over SSH.

Some Vim builds support `"+y` and `"+p` for the system clipboard, but remote clipboard access depends on the environment. Use your terminal's copy/paste shortcuts when appropriate; many Linux terminals use `Ctrl+Shift+C` and `Ctrl+Shift+V`.

## 9. Undo and redo

| Key | Action |
| --- | --- |
| `u` | Undo the last change |
| `Ctrl+r` | Redo an undone change |

**Example:** You accidentally deleted a line with `dd`. Press `u` to restore it.

Undo modifies the current buffer. Save with `:w` if the restored content should be written to disk.

> `Ctrl+z` is not undo in terminal Vim. It commonly suspends Vim and returns you to the shell. Run `fg` in that shell to resume it.

## 10. Search and replace

### Search

| Command | Action |
| --- | --- |
| `/timeout` | Search forward for `timeout` |
| `?timeout` | Search backward for `timeout` |
| `n` | Repeat the search in its original direction |
| `N` | Repeat in the opposite direction |
| `*` | Search forward for the word under the cursor |
| `:noh` | Clear current search highlighting |

Searches use regular expressions by default. A dot matches any character unless escaped.

To search for the literal IP address `192.168.1.10`, use very nomagic mode:

```vim
/\V192.168.1.10
```

Here `\V` makes most pattern characters literal; backslash still has special meaning.

### Replace one occurrence on the current line

```vim
:s/staging/production/
```

### Replace all occurrences on the current line

```vim
:s/staging/production/g
```

### Replace throughout the file and confirm each match

```vim
:%s/staging/production/gc
```

| Part | Meaning |
| --- | --- |
| `%` | All lines in the file |
| `s` | Substitute |
| `staging` | Pattern to find |
| `production` | Replacement text |
| `g` | Every occurrence on each line |
| `c` | Ask before each replacement |

During confirmation: `y` replaces, `n` skips, and `q` stops.

To replace all matches without confirmation:

```vim
:%s/staging/production/g
```

Limit replacements to lines 10–20:

```vim
:10,20s/staging/production/gc
```

For paths containing `/`, choose another delimiter:

```vim
:%s#/var/www/old#/var/www/new#gc
```

**DevOps habit:** Use confirmation when changing hostnames, environments, ports, or paths. Search patterns and replacement text have special syntax, so review broad replacements.

## 11. Select text and indent

| Command | Action |
| --- | --- |
| `v` | Start character selection |
| `V` | Start whole-line selection |
| Arrow keys or `j` / `k` | Extend selection |
| `y` | Copy the selection |
| `d` | Delete the selection |
| `>` | Indent selected lines |
| `<` | Reduce indentation of selected lines |
| `Esc` | Leave Visual mode |
| `>>` | Indent the current line in Normal mode |
| `<<` | Reduce the current line's indentation in Normal mode |

Example: select three lines with `Vjj`, then press `>` to indent them.

Indentation width depends on `shiftwidth`. For YAML, set it to two spaces as shown below.

## 12. YAML and pasted configuration

Kubernetes manifests, Ansible playbooks, and Compose files commonly use YAML. **YAML indentation uses spaces, not tabs.** Two spaces per level is a common convention; the correct structure matters more than the chosen width.

### Use two-space indentation for the current buffer

Enter these commands inside Vim:

```vim
:setlocal expandtab
:setlocal tabstop=2
:setlocal shiftwidth=2
:setlocal softtabstop=2
```

| Setting | Meaning |
| --- | --- |
| `expandtab` | Insert spaces when you press Tab |
| `tabstop=2` | Display a tab character as stops two columns apart |
| `shiftwidth=2` | Use two spaces for indentation actions |
| `softtabstop=2` | Make typing Tab and using Backspace work in two-column steps |

These settings do not automatically remove existing tabs or validate YAML.

Display tabs and line endings to help inspect whitespace:

```vim
:set list
```

Turn that display off:

```vim
:set nolist
```

### Paste without unwanted indentation

Modern Vim and terminals often handle pasted blocks correctly. If pasted lines gain extra indentation:

1. Press `Esc`.
2. Enter `:set paste`.
3. Press `i`, then use your terminal's paste command.
4. Press `Esc`.
5. Enter `:set nopaste`.

Turn paste mode off afterward so normal indentation and mappings work again.

**Important:** Do not globally change every file to spaces without considering its format. Makefile recipe lines normally require real tabs.

## 13. Work with multiple files

### Open two files

From the terminal:

```bash
vim app.conf backup.sh
```

Inside Vim:

| Command | Action |
| --- | --- |
| `:next` | Next file in the argument list |
| `:previous` | Previous file in the argument list |
| `:e another.conf` | Open another file in the current window |
| `:ls` | List buffers |
| `:bnext` | Next buffer |
| `:bprevious` | Previous buffer |

A **buffer** holds file text in memory. A **window** displays a buffer. Save with `:w` before switching if Vim warns about unsaved changes. Avoid adding `!` unless you intend to discard changes.

### Display files side by side

```vim
:vsplit backup.sh
```

### Display one file above another

```vim
:split backup.sh
```

Press `Ctrl+w`, release the keys, then press `w` to switch windows. Use `:q` to close the current window.

## 14. Edit a real configuration safely

Example: editing Nginx on a server where it is already installed and managed by systemd. Commands below run in the **terminal**, except the Vim editing steps.

### Step 1 — Inspect and back up the file

```bash
sudo cp -a /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak
```

This uses one backup name. Choose a different name if that backup already exists and you need to keep it.

### Step 2 — Open with the required privileges

```bash
sudoedit /etc/nginx/nginx.conf
```

`sudoedit` uses the configured editor. To request Vim for this invocation:

```bash
SUDO_EDITOR=vim sudoedit /etc/nginx/nginx.conf
```

Where permitted, another familiar approach is:

```bash
sudo vim /etc/nginx/nginx.conf
```

`sudoedit` edits a temporary copy as your normal user and installs it with elevated permissions. `sudo vim` runs the whole editor as root. Follow your team's access policy.

### Step 3 — Change only the intended setting

In Vim: search for the setting, press `i` to edit, press `Esc`, then use `:wq`.

### Step 4 — Review the difference

```bash
sudo diff -u /etc/nginx/nginx.conf.bak /etc/nginx/nginx.conf
```

A `diff` exit status of 1 means differences were found; it does not itself mean the configuration is invalid.

### Step 5 — Validate

```bash
sudo nginx -t
```

If validation fails, fix the file before proceeding.

### Step 6 — Reload after successful validation

```bash
sudo systemctl reload nginx
```

### Step 7 — Check the result

```bash
sudo systemctl status nginx --no-pager
```

Also check the affected endpoint and relevant logs. A running service alone does not prove the intended behavior.

**Remember:** Back up → edit → review → validate → reload → verify.

## 15. Common DevOps examples

### A. Edit and check a Bash script

Open:

```bash
vim backup.sh
```

Insert this small practice script:

```bash
#!/usr/bin/env bash
set -euo pipefail

printf 'Backup job started\n'
```

Save and exit. Check syntax without executing it:

```bash
bash -n backup.sh
```

Run it when ready:

```bash
bash backup.sh
```

Alternatively, make it executable and run directly:

```bash
chmod +x backup.sh
./backup.sh
```

`bash -n` checks syntax; it does not prove that commands or business logic work.

### B. Edit Kubernetes replica count

```bash
vim deployment.yaml
```

Example snippet, **not a complete manifest**:

```yaml
spec:
  replicas: 2
```

Search `/replicas`, edit the number, and save. On an authorized cluster, review the change:

```bash
kubectl diff -f deployment.yaml
```

If the reviewed change is intended, applying it updates the cluster:

```bash
kubectl apply -f deployment.yaml
```

These commands require a complete manifest, a configured Kubernetes context, and appropriate access. Saving in Vim alone does not change the cluster.

### C. Edit a Dockerfile

```bash
vim Dockerfile
```

Use `/FROM` to locate the base image or `/RUN` to find build steps. Save after editing. Building and testing the image is a separate step; a saved Dockerfile has not been validated by Vim.

### D. Edit a Terraform file

```bash
vim main.tf
```

After saving, check formatting:

```bash
terraform fmt -check
```

In an initialized working directory, validate the configuration:

```bash
terraform validate
```

Formatting and validation do not apply infrastructure changes. Review a plan before applying changes through your usual workflow.

### E. Write a Git commit message

If `git commit` opens Vim:

1. Press `i`.
2. Write a short message explaining the change.
3. Press `Esc`.
4. Enter `:wq`.

To make Vim your Git editor:

```bash
git config --global core.editor vim
```

To abort the commit from Vim, use:

```vim
:cq
```

This exits with an error status so Git stops. `:q!` is not a reliable way to abort a commit because existing message content may still be accepted.

### F. Inspect a log

For a small log or saved log sample:

```bash
vim -R app.log
```

Use `/ERROR`, `n`, and `N` to navigate matches. For a large live log, `less` or `tail -f` is usually a better fit:

```bash
tail -f app.log
```

Press `Ctrl+c` to stop following the log.

## 16. Permissions, swap files, and common problems

### “I cannot type”

You are probably in Normal mode. Press `i`.

### “Letters are deleting or moving things”

You may be typing in Normal mode. Press `Esc`, use `u` to undo accidental changes, then press `i` to type.

### “No write since last change”

Vim is protecting unsaved changes.

- Save and exit with `:wq`.
- Discard unsaved changes with `:q!` only if that is your intention.

### “Permission denied” or “Can't open file for writing”

You may lack permission, the parent directory may be missing, or the filesystem may be read-only or full.

If you have unsaved work, first save it to a private writable location, for example:

```vim
:w ~/nginx-working-copy.conf
```

For sensitive files, ensure that location has suitable access permissions. Then resolve permissions or reopen the original using an authorized method such as `sudoedit`. `:w!` cannot bypass Linux permissions.

### “A swap file already exists” — E325

A swap file may mean another editor is using the same file, or a previous Vim session crashed.

1. Read the warning and check the listed process/session details.
2. If another session is active, quit this attempt or inspect read-only rather than editing concurrently.
3. If the previous session crashed, recover from the terminal:

```bash
vim -r filename
```

Review the recovered text and save it. Remove an old swap file only after confirming it is stale and recovery is complete. Do not blindly choose “Delete it.”

### “I accidentally pressed Ctrl+s and the terminal froze”

In terminals where software flow control is enabled, `Ctrl+s` pauses output. Press `Ctrl+q` to resume. Save in Vim with `:w`.

### “Windows line endings appear as ^M”

Check the file format:

```vim
:set fileformat?
```

If the file should use Unix line endings:

```vim
:setlocal fileformat=unix
:w
```

This changes line endings on save. Use it when Unix format is appropriate for the file.

### “My changes disappeared after SSH disconnected”

Vim may have recoverable swap data. Reopen the file and read any recovery warning, or use `vim -r filename`. Recovery is not a substitute for saving regularly.

## 17. Simple Vim settings

Temporary settings for the current session:

```vim
:set number
:syntax on
:set hlsearch
:set incsearch
```

| Setting | Purpose |
| --- | --- |
| `number` | Show line numbers |
| `syntax on` | Enable syntax highlighting when available |
| `hlsearch` | Highlight search matches |
| `incsearch` | Show matches while entering a search |

To make settings persistent for your Linux user, edit:

```bash
vim ~/.vimrc
```

Merge the following with existing settings instead of replacing a configuration you already use:

```vim
" Basic readability
set number
syntax on
set hlsearch
set incsearch

" Recognize file types and enable indentation support
filetype plugin indent on

" Use two spaces for YAML buffers only
augroup DevOpsYaml
  autocmd!
  autocmd FileType yaml setlocal expandtab tabstop=2 shiftwidth=2 softtabstop=2
augroup END
```

In a Vim configuration file, a line beginning with `"` is a comment. Commands in `.vimrc` do not need the leading `:`.

Save and reopen Vim to load the configuration. System or root sessions may use different settings from your regular user.

## 18. Practice lab

Practice in your home directory before editing real service files.

### Step 1 — Create a practice folder

```bash
mkdir -p ~/vim-practice
cd ~/vim-practice
vim app.conf
```

### Step 2 — Press i and type

```ini
APP_ENV=development
APP_PORT=8080
LOG_LEVEL=debug
REPLICAS=2
```

### Step 3 — Save without exiting

Press `Esc`, enter `:w`, and press `Enter`.

### Step 4 — Change the environment

Enter:

```vim
/APP_ENV
```

Press `Enter`, then `$`, then `ciw`. Type `staging`, then press `Esc`.

The line should now be:

```ini
APP_ENV=staging
```

### Step 5 — Copy and undo

1. Search `/LOG_LEVEL` and press `Enter`.
2. Press `yy`, then `p` to duplicate the line.
3. Press `u` to undo the duplicate.

### Step 6 — Replace with confirmation

```vim
:%s/debug/info/gc
```

Press `y` to accept the match.

### Step 7 — Save, exit, and inspect

```vim
:wq
```

In the terminal:

```bash
cat app.conf
```

Expected content:

```ini
APP_ENV=staging
APP_PORT=8080
LOG_LEVEL=info
REPLICAS=2
```

### Step 8 — Try Vim's built-in tutorial

From the terminal, if installed:

```bash
vimtutor
```

For built-in help inside Vim:

```vim
:help
```

For help on a specific command:

```vim
:help :substitute
```

Help opens in a window. Enter `:q` to close that window and return to your file.

## 19. Quick cheat sheet

Start in Normal mode by pressing `Esc`. Press `Enter` after `:` and search commands.

| Need | Command |
| --- | --- |
| Open a file from terminal | `vim filename` |
| Start typing | `i` |
| Return to Normal mode | `Esc` |
| Save | `:w` |
| Save and quit | `:wq` |
| Quit without saving unsaved changes | `:q!` |
| Undo / redo | `u` / `Ctrl+r` |
| Delete a line | `dd` |
| Copy a line | `yy` |
| Paste below | `p` |
| Add a line below | `o` |
| Change the current word | `ciw` |
| Go to first / last line | `gg` / `G` |
| Go to line 50 | `50G` |
| Search | `/text` |
| Next / previous search match | `n` / `N` |
| Replace throughout file, confirming matches | `:%s/old/new/gc` |
| Show line numbers | `:set number` |
| Select whole lines | `V` then move |
| Indent / unindent current line | `>>` / `<<` |
| Clear search highlights | `:noh` |

**First commands to memorize:** `i`, `Esc`, `:w`, `:wq`, `:q!`, `u`, `dd`, `yy`, `p`, and `/text`.
