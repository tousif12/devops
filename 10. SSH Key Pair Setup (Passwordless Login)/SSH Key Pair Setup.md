# SSH Key Pair Setup (Passwordless Login)

This guide shows how to log in from one machine (`scriptbox`) to another (`web01`) as user `devops` without typing a password each time.

| Machine     | Role                                  |
|-------------|---------------------------------------|
| `scriptbox` | Client (where you run `ssh`)          |
| `web01`     | Server (the machine you log in to)    |

---

## How SSH Keys Work

An SSH key pair is two files that are created together and are mathematically linked. One is locked away (private), the other is handed out (public).

### Simple analogy: a lock and a key

- **Public key = a padlock.** You can give copies of this padlock to as many servers as you want. Anyone can see it, and that is safe, because a padlock alone cannot open anything.
- **Private key = the only key that opens that padlock.** You keep it on your own machine and never give it to anyone.

When you put your padlock (public key) on web01, only the person holding the matching key (private key) can get in.

### What each key does

| Key         | File                         | Lives on                                    | Job                                                                 |
|-------------|------------------------------|---------------------------------------------|---------------------------------------------------------------------|
| Private key | `~/.ssh/id_rsa`              | **scriptbox** (the client you connect from) | Proves your identity. It signs a challenge sent by the server.     |
| Public key  | `~/.ssh/id_rsa.pub`          | scriptbox (original copy)                   | The shareable half. Gets copied to every server you want to log in to. |
| Public key  | `~/.ssh/authorized_keys`     | **web01** (the server you connect to)       | The list of public keys allowed to log in as this user.            |

### Why you need both

Neither key works alone:

- The **public key alone** cannot log in. It can only check a signature, not create one. That is why it is safe to share.
- The **private key alone** is useless if the server does not have the matching public key, because the server has nothing to check your signature against.

Only the pair together completes the login.

### What happens when you run `ssh devops@web01`

1. scriptbox says: "I want to log in as `devops`, using this key."
2. web01 looks in `/home/devops/.ssh/authorized_keys` and finds the matching public key.
3. web01 sends a random challenge (a one-time message).
4. scriptbox signs the challenge with its **private key**. The private key itself never leaves scriptbox.
5. web01 checks the signature with the **public key**. If it matches, you are logged in, with no password.

```
   scriptbox (client)                          web01 (server)
   ------------------                          --------------
   ~/.ssh/id_rsa       (private, stays here)
   ~/.ssh/id_rsa.pub   (public)  ── ssh-copy-id ──>  ~/.ssh/authorized_keys

   ssh devops@web01  ───────── "log me in" ─────────>
                     <──────── random challenge ──────
   sign with id_rsa  ───────── signature ───────────>  verify with public key
                     <──────── access granted ────────
```

### Why this is better than a password

- No password is sent over the network.
- Nobody can guess or brute-force a 4096-bit key the way they can guess a password.
- Scripts and automation tools (Ansible, cron jobs, CI pipelines) can log in without a human typing anything.
- To block access, you just delete that public key line from `authorized_keys` on the server.

> **Golden rule:** Public key can go anywhere. Private key never leaves your machine. If the private key is leaked, anyone holding it can log in to every server that has your public key. If that happens, delete the old pair, make a new one, and copy the new public key to your servers.

---

## Step 1: Generate the key pair (on scriptbox)

```bash
[root@scriptbox ~]# ssh-keygen
```

- Press **Enter** to accept the default file location (`/root/.ssh/id_rsa` or `/root/.ssh/id_ed25519`).
- Press **Enter** twice to leave the passphrase empty (or set one for extra security).

This creates two files in `~/.ssh/`:

| File                               | What it is   | Share it?                        |
|------------------------------------|--------------|----------------------------------|
| `id_rsa` / `id_ed25519`            | Private key  | **Never.** Keep it on scriptbox. |
| `id_rsa.pub` / `id_ed25519.pub`    | Public key   | Yes, this goes to web01.         |

> Note: Newer OpenSSH versions create `id_ed25519` by default instead of `id_rsa`. To force RSA, run `ssh-keygen -t rsa -b 4096`.

---

## Step 2: Copy the public key to web01

```bash
[root@scriptbox ~]# ssh-copy-id devops@web01
```

- Type `yes` if asked to confirm the host fingerprint.
- Enter the `devops` user's password **one last time**.

This appends your public key to `/home/devops/.ssh/authorized_keys` on web01.

---

## Step 3: Test the passwordless login

```bash
[root@scriptbox ~]# ssh devops@web01 uptime
 21:11:17 up 51 min,  0 users,  load average: 0.00, 0.01, 0.00
```

If you see the output without a password prompt, the setup works.

To open a full shell instead:

```bash
[root@scriptbox ~]# ssh devops@web01
```

---

## Step 4: Verify the keys exist

### On scriptbox (the client)

```bash
[root@scriptbox ~]# ls -a          # shows hidden files, including .ssh
[root@scriptbox ~]# ls .ssh/
id_rsa  id_rsa.pub  known_hosts
```

Read the keys:

```bash
[root@scriptbox ~]# cat .ssh/id_rsa       # private key (do not share)
[root@scriptbox ~]# cat .ssh/id_rsa.pub   # public key
```

### On web01 (the server)

```bash
[devops@web01 ~]$ ls -a
[devops@web01 ~]$ ls .ssh/
authorized_keys
[devops@web01 ~]$ cat .ssh/authorized_keys
```

The content of `authorized_keys` on web01 should match `id_rsa.pub` on scriptbox.

---

## Quick Reference

```bash
# On scriptbox
ssh-keygen                       # 1. create key pair
ssh-copy-id devops@web01         # 2. send public key to web01
ssh devops@web01 uptime          # 3. test login, no password

# Verify
ls -a                            # see hidden .ssh folder
ls .ssh/                         # list key files
cat .ssh/id_rsa.pub              # read public key (scriptbox)
cat .ssh/authorized_keys         # read authorized keys (web01)
```

---

## Troubleshooting

| Problem                              | Fix                                                                 |
|--------------------------------------|---------------------------------------------------------------------|
| Still asks for password              | Check permissions on web01: `chmod 700 ~/.ssh` and `chmod 600 ~/.ssh/authorized_keys` |
| `Could not resolve hostname web01`   | Add web01 to `/etc/hosts` or use its IP address                     |
| `Permission denied (publickey)`      | Make sure the key was copied to the correct user (`devops`)         |
| `ssh-copy-id: command not found`     | Copy manually: `cat ~/.ssh/id_rsa.pub \| ssh devops@web01 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"` |
| Want to see what's going wrong       | Run `ssh -v devops@web01` for verbose debug output                  |

---

## Pro Tip: Run Commands and Scripts on Many Machines at Once

Once passwordless SSH works, you can control many servers from scriptbox with a simple `for` loop. You don't need to log in to each one by hand.

> Before you start: repeat `ssh-copy-id devops@<host>` once for **every** host (web01, web02, web03), so none of them ask for a password.

### Project files on scriptbox

Keep all of these in the same folder on scriptbox (for example `/opt/scripts`):

| File                  | Purpose                                                              |
|-----------------------|----------------------------------------------------------------------|
| `remhosts`            | List of target hostnames or IPs, one per line                        |
| `multios_websetup.sh` | The setup script that runs **on each remote machine** (CentOS or Ubuntu) |
| `webdeploy.sh`        | The **main** script. Copies `multios_websetup.sh` to every host and runs it |
| `testfile.txt`        | A sample file for testing `scp`                                      |

### Step 1: Create a hosts file

Create a file called `remhosts` with one hostname (or IP) per line:

```
web01
web02
web03
```

You can use IP addresses instead, for example `192.168.10.11`. Hostnames only work if they resolve, either through DNS or entries in `/etc/hosts` on scriptbox.

### Step 2: Run a command on every host with a for loop

```bash
for host in `cat remhosts`; do ssh devops@$host uptime; done
```

How it works:

- `` `cat remhosts` `` reads the file and gives back `web01 web02 web03`.
- The loop runs once for each name and puts it in `$host`.
- `ssh devops@$host uptime` runs `uptime` on that host and prints the result back on scriptbox.

More examples:

```bash
# Check hostname of every server
for host in `cat remhosts`; do ssh devops@$host hostname; done

# Install a package on every CentOS server
for host in `cat remhosts`; do ssh devops@$host sudo yum install git -y; done
```

> Note: `for host in ...; do ssh devops@$host; done` without a command opens an **interactive shell** on each host one by one. You have to type `exit` to move to the next one. Add a command after the host (like `uptime`) to make it fully automatic.

### Step 3: Copy files to another machine with scp

`scp` (secure copy) uses the same SSH keys, so it doesn't ask for a password either.

```bash
# Syntax
scp <local-file> <user>@<host>:<remote-path>

# Example: copy testfile.txt to /tmp on web01
scp testfile.txt devops@web01:/tmp/

# Example: copy to /opt/scripts on web01
scp testfile.txt devops@web01:/opt/scripts/
```

Things to watch:

- **No space** between the host and the colon. `devops@web01:/opt/scripts` is correct. `devops@web01 :/opt/scripts` breaks the command.
- The remote folder must already exist, and `devops` must have write permission on it. `/tmp/` always works. For `/opt/scripts`, create it first:
  ```bash
  ssh devops@web01 "sudo mkdir -p /opt/scripts && sudo chown devops:devops /opt/scripts"
  ```
- To copy a whole folder, add `-r`: `scp -r myfolder devops@web01:/tmp/`

Copy a file to all hosts with a loop:

```bash
for host in `cat remhosts`; do scp testfile.txt devops@$host:/tmp/; done
```

### Step 4: The multi OS setup script (`multios_websetup.sh`)

This script runs **on the remote machine**. It works out whether the machine is CentOS or Ubuntu, then installs and starts Apache and deploys a website template:

- It runs `yum --help`. If that works (`$? -eq 0`), the machine is CentOS, so it uses `yum`, `httpd`.
- Otherwise it treats the machine as Ubuntu and uses `apt`, `apache2`.
- After that, both branches do the same steps: start and enable the service, download the zip, unzip it into `/var/www/html/`, restart the service and clean up.

```bash
#!/bin/bash

# Variable Declaration
#PACKAGE="httpd wget unzip"
#SVC="httpd"
URL='https://www.tooplate.com/zip-templates/2098_health.zip'
ART_NAME='2098_health'
TEMPDIR="/tmp/webfiles"

yum --help &> /dev/null

if [ $? -eq 0 ]
then
   # Set Variables for CentOS
   PACKAGE="httpd wget unzip"
   SVC="httpd"

   echo "Running Setup on CentOS"
   # Installing Dependencies
   echo "########################################"
   echo "Installing packages."
   echo "########################################"
   sudo yum install $PACKAGE -y > /dev/null
   echo

   # Start & Enable Service
   echo "########################################"
   echo "Start & Enable HTTPD Service"
   echo "########################################"
   sudo systemctl start $SVC
   sudo systemctl enable $SVC
   echo

   # Creating Temp Directory
   echo "########################################"
   echo "Starting Artifact Deployment"
   echo "########################################"
   mkdir -p $TEMPDIR
   cd $TEMPDIR
   echo

   wget $URL > /dev/null
   unzip $ART_NAME.zip > /dev/null
   sudo cp -r $ART_NAME/* /var/www/html/
   echo

   # Bounce Service
   echo "########################################"
   echo "Restarting HTTPD service"
   echo "########################################"
   systemctl restart $SVC
   echo

   # Clean Up
   echo "########################################"
   echo "Removing Temporary Files"
   echo "########################################"
   rm -rf $TEMPDIR
   echo

   sudo systemctl status $SVC
   ls /var/www/html/

else
    # Set Variables for Ubuntu
   PACKAGE="apache2 wget unzip"
   SVC="apache2"

   echo "Running Setup on CentOS"
   # Installing Dependencies
   echo "########################################"
   echo "Installing packages."
   echo "########################################"
   sudo apt update
   sudo apt install $PACKAGE -y > /dev/null
   echo

   # Start & Enable Service
   echo "########################################"
   echo "Start & Enable HTTPD Service"
   echo "########################################"
   sudo systemctl start $SVC
   sudo systemctl enable $SVC
   echo

   # Creating Temp Directory
   echo "########################################"
   echo "Starting Artifact Deployment"
   echo "########################################"
   mkdir -p $TEMPDIR
   cd $TEMPDIR
   echo

   wget $URL > /dev/null
   unzip $ART_NAME.zip > /dev/null
   sudo cp -r $ART_NAME/* /var/www/html/
   echo

   # Bounce Service
   echo "########################################"
   echo "Restarting HTTPD service"
   echo "########################################"
   systemctl restart $SVC
   echo

   # Clean Up
   echo "########################################"
   echo "Removing Temporary Files"
   echo "########################################"
   rm -rf $TEMPDIR
   echo

   sudo systemctl status $SVC
   ls /var/www/html/
fi 
```

> Small fix: in the Ubuntu (`else`) branch, the message says `Running Setup on CentOS`. Change it to `Running Setup on Ubuntu` so the output isn't misleading.

### Step 5: The main deploy script (`webdeploy.sh`)

This is the script you actually run on scriptbox. For each host in `remhosts`, it:

1. Copies `multios_websetup.sh` to `/tmp/` on the host (`scp`)
2. Runs it with `sudo` on the host (`ssh`)
3. Deletes the script from the host afterwards (`ssh ... rm`)

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

### Step 6: Give execute permission and run

On scriptbox, make **both** scripts executable:

```bash
chmod +x webdeploy.sh multios_websetup.sh
```

`multios_websetup.sh` also needs `+x`, because `webdeploy.sh` runs it directly as `/tmp/multios_websetup.sh`. `scp` keeps the execute permission when it copies the file.

Run the main script from the folder that contains all the files:

```bash
./webdeploy.sh
```

Then check each site in a browser at `http://<host-ip>`, or from scriptbox:

```bash
for host in `cat remhosts`; do curl -s -o /dev/null -w "$host: %{http_code}\n" http://$host; done
```

A `200` means the website is up.

### Flow diagram

```
scriptbox
  |
  |-- ./webdeploy.sh
  |      |
  |      |-- reads remhosts -> web01, web02, web03
  |      |
  |      |-- for each host:
  |            scp multios_websetup.sh  ->  host:/tmp/
  |            ssh sudo /tmp/multios_websetup.sh   (installs + deploys site)
  |            ssh sudo rm /tmp/multios_websetup.sh (cleanup)
  v
web01 (CentOS: yum/httpd)   web02 (Ubuntu: apt/apache2)   web03 (...)
```

### Common issues

| Problem | Fix |
|---|---|
| Asks for a password on some host | Run `ssh-copy-id devops@<that-host>` for it |
| `sudo: a terminal is required` or sudo asks for a password | Give `devops` passwordless sudo on each host: run `sudo visudo` and add `devops ALL=(ALL) NOPASSWD: ALL` |
| `Permission denied` when running `/tmp/multios_websetup.sh` | Run `chmod +x multios_websetup.sh` on scriptbox, then run the deploy again |
| `cat: remhosts: No such file or directory` | Run `./webdeploy.sh` from the folder where `remhosts` lives |
| `scp: /opt/scripts: Permission denied` | Create the folder and set its owner first (see Step 3), or copy to `/tmp/` |
