# Linux File Permissions — Easy Guide

Based on the File Permissions section of **Linux Quickstart V5**, plus all our simplified explanations and follow-up examples.

**One example throughout:** Ali owns `notes.txt`. Its group is `developers`. Sara belongs to that group.

## 1. What are file permissions?

Permissions answer: **Who can read, edit or run this file?**

| Who? | Our example |
|---|---|
| Owner | Ali |
| Group | Developers team, including Sara |
| Others | Everyone who is neither the owner nor a member of the file's group |

Permissions always follow this order: **owner → group → others**.

## 2. The four symbols

| Symbol | Name | Easy meaning for a file |
|---|---|---|
| `r` | Read | Open and see the contents |
| `w` | Write | Edit the contents |
| `x` | Execute | Run it as a program, when applicable |
| `-` | Missing permission | That action is not allowed by this permission bit |

For an ordinary text file, start by focusing on **read and write**.

Example permissions, separated to make them easy to read:

```text
rw-   r--   ---
```

| Owner: Ali | Group: developers | Others |
|---|---|---|
| `rw-` | `r--` | `---` |
| Read and edit | Read only | No permissions |

**Sara can read Ali's notes, but she cannot edit them.**

## 3. See permissions with `ls -l`

```bash
ls -l notes.txt
```

Example output:

```text
-rw-r----- 1 ali developers 120 Oct 25 10:00 notes.txt
```

Break the permission part into four pieces:

| File type | Owner permissions | Group permissions | Others permissions |
|---|---|---|---|
| `-` | `rw-` | `r--` | `---` |

The **first character is the file type**, not one of the nine permission characters:

- `-` = regular file.
- `d` = directory.
- `l` = symbolic link. Access through a link normally depends on the target and directory permissions rather than the link's displayed mode.

The remaining output:

| Part | Meaning |
|---|---|
| `1` | Hard-link count; not a permission |
| `ali` | User owner |
| `developers` | Group owner |
| `120` | Size in bytes |
| `Oct 25 10:00` | Modification time |
| `notes.txt` | Filename |

To view a directory's own permissions:

```bash
ls -ld projects
```

Without `-d`, `ls -l projects` normally lists entries inside it instead.

## 4. Change permissions with letters: `chmod`

Suppose Ali wants Sara's team to edit `notes.txt`.

```bash
chmod g+w notes.txt
```

Read it as:

> For the **group (`g`)**, **add (`+`)** **write permission (`w`)**.

Before:

```text
rw-   r--   ---
```

After:

```text
rw-   rw-   ---
```

Now Ali and the developers can read and edit. Others still have no access through these bits.

### Who do the letters select?

| Letter | Applies to |
|---|---|
| `u` | User owner |
| `g` | Group |
| `o` | Others |
| `a` | All three |

**`u` means the file's owner. It does not mean whichever user happens to type the command.**

### What do the operators do?

| Operator | Meaning |
|---|---|
| `+` | Add the specified permission; keep the others |
| `-` | Remove the specified permission; keep the others |
| `=` | Set that category to exactly the specified permissions |

### Small examples

| Command | Read it as |
|---|---|
| `chmod g+w notes.txt` | Group: add write |
| `chmod g-w notes.txt` | Group: remove write |
| `chmod g=r notes.txt` | Group: set to read only |
| `chmod u+x backup.sh` | Owner: add execute |
| `chmod o-r notes.txt` | Others: remove read |
| `chmod ugo+r notes.txt` | Owner, group and others: add read |
| `chmod a+r notes.txt` | Same: add read for everyone |
| `chmod o-wx projects` | Others: remove write and directory traversal |

**Difference between `+` and `=`:**

If the group's permissions are `rw-`:

- `chmod g+r notes.txt` keeps them as `rw-` because read was already present.
- `chmod g=r notes.txt` changes them to `r--` because the group is set to read only.

## 5. Numeric permissions: just addition

Each permission has a value:

| Permission | Value |
|---|---:|
| Read: `r` | **4** |
| Write: `w` | **2** |
| Execute: `x` | **1** |
| No permission | **0** |

**Add the permissions you want.**

| You want | Calculation | Number |
|---|---|---:|
| Read | 4 | **4** |
| Read and write | 4 + 2 | **6** |
| Read and execute | 4 + 1 | **5** |
| Read, write and execute | 4 + 2 + 1 | **7** |
| Nothing | 0 | **0** |

### All numbers from 0 to 7

| Number | Letters | Meaning |
|---:|---|---|
| 0 | `---` | None |
| 1 | `--x` | Execute only |
| 2 | `-w-` | Write only |
| 3 | `-wx` | Write and execute |
| 4 | `r--` | Read only |
| 5 | `r-x` | Read and execute |
| 6 | `rw-` | Read and write |
| 7 | `rwx` | Read, write and execute |

You will often see **0, 4, 5, 6 and 7**.

## 6. Why three digits, like `640`?

One digit for each category, in the same familiar order:

| Owner | Group | Others |
|---:|---:|---:|
| **6** | **4** | **0** |
| Read + write | Read | Nothing |

```bash
chmod 640 notes.txt
```

Meaning:

> Ali can read/edit. Developers can read. Everyone else gets no access through these bits.

The resulting permissions are:

```text
-rw-r-----
```

The first `-` indicates a regular file. The other nine characters correspond to `640`.

**Numeric mode sets all three ordinary permission categories at once.**

## 7. More numeric examples

### Text files

| Command | Owner | Group | Others | Letters |
|---|---|---|---|---|
| `chmod 600 notes.txt` | Read/edit | Nothing | Nothing | `rw-------` |
| `chmod 640 notes.txt` | Read/edit | Read | Nothing | `rw-r-----` |
| `chmod 644 notes.txt` | Read/edit | Read | Read | `rw-r--r--` |
| `chmod 660 notes.txt` | Read/edit | Read/edit | Nothing | `rw-rw----` |

### Scripts or programs

```bash
chmod 750 backup.sh
```

| Owner: 7 | Group: 5 | Others: 0 |
|---|---|---|
| Read, edit and run | Read and run | Nothing |

Other examples from the document:

| Command | Letters | Meaning |
|---|---|---|
| `chmod 700 backup.sh` | `rwx------` | Only owner gets permissions |
| `chmod 755 backup.sh` | `rwxr-xr-x` | Owner can read/edit/run; group and others can read/run |

The `x` bit does not turn arbitrary text into a program. The contents must also be suitable for execution. For a shell script, you might run it directly with `./backup.sh` once its contents and permissions are appropriate.

## 8. Follow the document's command sequence

The document uses a file named `newtools.txt`:

| Command | What happens |
|---|---|
| `chmod u+x newtools.txt` | Add owner execute permission |
| `chmod o-r newtools.txt` | Remove others' read permission |
| `chmod 700 newtools.txt` | Set owner to all; group and others to nothing |
| `chmod 755 newtools.txt` | Set owner to all; group and others to read/execute |

These are successive changes. The final ordinary permissions are `rwxr-xr-x`, because the last command sets them to `755`.

The document also demonstrates `chmod 640 myfile`: owner read/write, group read, others nothing.

## 9. Permissions on folders

On a directory, the same letters describe different actions:

| Permission | Easy meaning |
|---|---|
| `r` | List the names inside |
| `w` | Add, delete or rename entries, normally with `x` too |
| `x` | Enter/traverse the directory and access items by name |

Think of a folder as a room:

- `r`: see the list of names of things in the room.
- `x`: pass through the doorway to reach a named item.
- `w` with `x`: add, remove or rename items in the room.

### Example: a private folder

```bash
chmod 700 private-folder
```

| Owner | Group | Others |
|---|---|---|
| Can list, enter and manage entries | No permissions | No permissions |

### Example: a folder others can browse

```bash
chmod 755 projects
```

Owner can manage its entries. Group and others can list and enter, but cannot create/delete entries through these mode bits.

**Directory permissions and file permissions are checked separately.** Being able to enter the folder does not automatically mean being able to read every file inside it.

**Deleting a file is mainly controlled by the containing directory's permissions**, not the file's write bit. Extra rules, such as a directory's sticky bit, can further restrict deletion.

## 10. Ownership versus permissions

These answer different questions:

| Concept | Question |
|---|---|
| Ownership | Who owns the file? Which group is associated with it? |
| Permissions | What may the owner, group and others do? |

### Change the user owner with `chown`

```bash
sudo chown sara notes.txt
```

Sara becomes the owner. The command does not itself rewrite the file's contents.

### Change the group with `chgrp`

```bash
sudo chgrp developers notes.txt
```

Sets the file's group to developers.

### Change both together

```bash
sudo chown sara:developers notes.txt
```

Owner becomes Sara; group becomes developers.

```bash
ls -l notes.txt
```

Example:

```text
-rw-r----- 1 sara developers 120 Oct 25 10:00 notes.txt
```

With this setting, Sara can read/edit; other developers can read; others get no access through these bits.

### Who may change these settings?

- The owner or an administrator can normally change permissions with `chmod`.
- Changing the user owner normally requires administrative privilege.
- An owner can normally change a file's group to a group they belong to; an administrator has broader control.
- `sudo` runs a command with elevated privilege when your account is authorized.

The document shows an ordinary `chown` failing with `Operation not permitted`, then succeeding with `sudo`. That is why its ownership example uses administrative privilege.

### Remember the three commands

| Command | What it changes |
|---|---|
| `chmod` | Permissions |
| `chown` | User ownership, optionally group too |
| `chgrp` | Group ownership |

## 11. `-R`: also apply inside the folder

`-R` means **recursive**: include contents and subfolders.

```bash
chmod -R g+w projects
```

Adds group write permission to `projects` and its contents. It does not add directory traversal permission if it is missing.

```bash
sudo chown -R ali:developers projects
```

Sets owner/group on `projects` and its contents.

```bash
sudo chgrp -R developers projects
```

Sets the group on the folder and its contents.

Use the intended folder path: `-R` can affect many items. Files and directories often need different permissions, so applying one numeric mode recursively is not always suitable.

## 12. `-v`: show what changed

```bash
chmod -v 640 notes.txt
```

`-v` means **verbose**: print information about the operation.

Example message, depending on the previous mode:

```text
mode of 'notes.txt' changed from 0644 (rw-r--r--) to 0640 (rw-r-----)
```

## 13. `--reference`: copy FROM which file, apply TO which file?

```bash
chmod --reference=notes.txt copy.txt
```

| Role | File |
|---|---|
| **Copy permissions FROM** | **`notes.txt`** |
| **Apply permissions TO** | **`copy.txt`** |

Read the command as:

> Make `copy.txt` use the same permission mode as `notes.txt`.

### Before

| File | Permissions |
|---|---|
| `notes.txt` | `640` |
| `copy.txt` | `600` |

Run:

```bash
chmod --reference=notes.txt copy.txt
```

### After

| File | Permissions |
|---|---|
| `notes.txt` | `640` — unchanged |
| `copy.txt` | `640` — updated |

**Only the permission mode is copied. File contents and ownership stay unchanged.** This does not create `copy.txt`; the target must already exist. It does not copy access-control lists (ACLs).

Memory aid: **the file after `--reference=` is the example to copy from; the final file is the one to change.**

## 14. Small practice: see the changes yourself

Run these in your Linux VM as your normal user. Use a new practice directory so the examples do not overwrite existing notes.

### Step 1: Make a practice directory

```bash
mkdir permissions-practice
cd permissions-practice
```

### Step 2: Create a file

```bash
echo "My practice notes" > notes.txt
```

### Step 3: Give only the owner read/write

```bash
chmod 600 notes.txt
ls -l notes.txt
```

Permissions start with:

```text
-rw-------
```

### Step 4: Allow the group to read

```bash
chmod g+r notes.txt
ls -l notes.txt
```

Now:

```text
-rw-r-----
```

This is `640`.

### Step 5: Allow the group to edit too

```bash
chmod g+w notes.txt
ls -l notes.txt
```

Now:

```text
-rw-rw----
```

This is `660`.

### Step 6: Set the group back to read only

```bash
chmod g=r notes.txt
ls -l notes.txt
```

Back to `640`:

```text
-rw-r-----
```

### Step 7: Try copying permissions

```bash
touch copy.txt
chmod 600 copy.txt
chmod --reference=notes.txt copy.txt
ls -l notes.txt copy.txt
```

Both files now have `640` permissions. `notes.txt` still contains the text; `copy.txt` is still empty. **Copying permissions did not copy the text.**

These commands change the files you own. The owner/group names shown by `ls` will be your actual account and group, not necessarily Ali/developers.

## 15. Common questions

### Does Linux add owner, group and others permissions together?

For ordinary permission checks, it chooses the applicable category:

1. If you are the owner, use the owner bits.
2. Otherwise, if your active groups match the file's group, use the group bits.
3. Otherwise, use others bits.

For example, an owner cannot use more generous “others” bits to escape restrictive owner bits. ACLs and elevated privileges can add other rules.

### I joined the group but still get Permission denied. Why?

Start a fresh login session so the membership becomes active. Then check:

```bash
id
ls -ld projects
ls -l projects/notes.txt
```

The file and the directories leading to it must allow the required access. Additional controls may also apply.

### Does a group automatically get access to all files?

No. Membership matters when the relevant file/folder grants that group permission.

### Is `777` a good general fix?

`777` gives owner, group and others read/write/execute permissions. Use the permissions the task needs rather than granting everyone all three. Ordinary text files usually do not need execute permission.

### Can root access a file with no permissions?

Root can generally bypass ordinary file permissions, although other controls can still matter. “No permissions” in these examples means no access granted by those ordinary mode bits.

## 16. Quick revision sheet

| Remember | Meaning |
|---|---|
| Order | Owner → group → others |
| `r = 4` | Read |
| `w = 2` | Write |
| `x = 1` | Execute/traverse |
| `6 = 4 + 2` | Read + write |
| `5 = 4 + 1` | Read + execute |
| `7 = 4 + 2 + 1` | All three |
| `chmod 640 notes.txt` | Owner read/write; group read; others nothing |
| `chmod g+w notes.txt` | Add group write |
| `chmod g-w notes.txt` | Remove group write |
| `chmod g=r notes.txt` | Set group to read only |
| `chown` | Change user owner, optionally group |
| `chgrp` | Change group owner |
| `-R` | Include contents recursively |
| `-v` | Show operation details |
| `--reference=notes.txt copy.txt` | Copy permissions FROM notes.txt TO copy.txt |

**Most useful sentence:** `640` means **6 for owner, 4 for group, 0 for others**. And **6 is read + write because 4 + 2 = 6**.
