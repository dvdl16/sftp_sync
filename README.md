# SFTP Sync
Sync a local directory on Linux with a remote sftp server

## Setup
Install rsync
```bash
sudo -v ; curl https://rclone.org/install.sh | sudo bash
```

Config rclone
```bash
rclone config
```

```bash
server@home-server:~/Scripts/sftp_sync$ rclone config
2023/01/27 16:45:26 NOTICE: Config file "/home/server/.config/rclone/rclone.conf" not found - using defaults
No remotes found, make a new one?
n) New remote
s) Set configuration password
q) Quit config
n/s/q> n

Enter name for new remote.
name> kitabu

Option Storage.
Type of storage to configure.
Choose a number from below, or type in your own value.
 1 / 1Fichier
   \ (fichier)
.
.
.
50 / seafile
   \ (seafile)
Storage> sftp

Option host.
SSH host to connect to.
E.g. "example.com".
Enter a value.
host> my-site.pikapod.net

Option user.
SSH username.
Enter a string value. Press Enter for the default (server).
user> p11111

Option port.
SSH port number.
Enter a signed integer. Press Enter for the default (22).
port>

Option pass.
SSH password, leave blank to use ssh-agent.
Choose an alternative below. Press Enter for the default (n).
y) Yes, type in my own password
g) Generate random password
n) No, leave this optional password blank (default)
y/g/n> y
Enter the password:
password:
Confirm the password:
password:

Option key_pem.
Raw PEM-encoded private key.
If specified, will override key_file parameter.
Enter a value. Press Enter to leave empty.
key_pem>

Option key_file.
Path to PEM-encoded private key file.
Leave blank or set key-use-agent to use ssh-agent.
Leading `~` will be expanded in the file name as will environment variables such as `${RCLONE_CONFIG_DIR}`.
Enter a value. Press Enter to leave empty.
key_file>

Option key_file_pass.
The passphrase to decrypt the PEM-encoded private key file.
Only PEM encrypted key files (old OpenSSH format) are supported. Encrypted keys
in the new OpenSSH format can't be used.
Choose an alternative below. Press Enter for the default (n).
y) Yes, type in my own password
g) Generate random password
n) No, leave this optional password blank (default)
y/g/n>

Option pubkey_file.
Optional path to public key file.
Set this if you have a signed certificate you want to use for authentication.
Leading `~` will be expanded in the file name as will environment variables such as `${RCLONE_CONFIG_DIR}`.
Enter a value. Press Enter to leave empty.
pubkey_file>

Option key_use_agent.
When set forces the usage of the ssh-agent.
When key-file is also set, the ".pub" file of the specified key-file is read and only the associated key is
requested from the ssh-agent. This allows to avoid `Too many authentication failures for *username*` errors
when the ssh-agent contains many keys.
Enter a boolean value (true or false). Press Enter for the default (false).
key_use_agent>

Option use_insecure_cipher.
Enable the use of insecure ciphers and key exchange methods.
This enables the use of the following insecure ciphers and key exchange methods:
- aes128-cbc
- aes192-cbc
- aes256-cbc
- 3des-cbc
- diffie-hellman-group-exchange-sha256
- diffie-hellman-group-exchange-sha1
Those algorithms are insecure and may allow plaintext data to be recovered by an attacker.
This must be false if you use either ciphers or key_exchange advanced options.
Choose a number from below, or type in your own boolean value (true or false).
Press Enter for the default (false).
 1 / Use default Cipher list.
   \ (false)
 2 / Enables the use of the aes128-cbc cipher and diffie-hellman-group-exchange-sha256, diffie-hellman-group-exchange-sha1 key exchange.
   \ (true)
use_insecure_cipher>

Option disable_hashcheck.
Disable the execution of SSH commands to determine if remote file hashing is available.
Leave blank or set to false to enable hashing (recommended), set to true to disable hashing.
Enter a boolean value (true or false). Press Enter for the default (false).
disable_hashcheck>

Edit advanced config?
y) Yes
n) No (default)
y/n>

Configuration complete.
Options:
- type: sftp
- host: devious-wildebeest.pikapod.net
- user: p12478
- pass: *** ENCRYPTED ***
Keep this "kitabu" remote?
y) Yes this is OK (default)
e) Edit this remote
d) Delete this remote
y/e/d> y

Current remotes:

Name                 Type
====                 ====
kitabu               sftp

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> q
```

## Example sync command

```bash
rclone sync -i /home/server/Downloads/temp-sync kitabu:books
```

## Scheduled sync

Based on [this link](https://medium.com/swlh/using-rclone-on-linux-to-automate-backups-to-google-drive-d599b49c42e8)

Automating the rclone command with flock and cron

The rclone copy command in the previous section can be implemented as a cron job, scheduled to run at desired intervals. Before we go into the scheduling, we need to deal with the situation where a backup may still be executing while the next scheduled instance is due to commence. A simple approach to managing this scenario would be via the concept of Advisory file locking with flock.

According to the man page for flock, we can use a one-liner flock command to implement our rclone backup.

```bash
flock -n /tmp/sftp_sync.lock /usr/bin/rclone sync "/home/server/Dropbox/Dirk ebooks" "kitabu:books"
```

The above flock command will create the file sftp_sync_sync.lock (if it does not already exist) and acquire an exclusive (write) lock on the file for the duration of the rclone backup. The -n option tells flock that in the case where the lock cannot be acquired (i.e. previous backup is still running), exit immediately with return code 1 (i.e. do not wait for lock to be released).

Thus add the following `crontab` entry:
```bash
crontab -e
```

```bash
*/60 * * * * flock -n /tmp/sftp_sync.lock /usr/bin/rclone sync "/home/server/Dropbox/Dirk ebooks" "kitabu:books"
```
for sync every hour.


## Optional
Install Dropbox and sync a Dropbox folder with the SFTP server using rsync

Install Dropbox
```bash
cd ~ && wget -O - "https://www.dropbox.com/download?plat=lnx.x86_64" | tar xzf -
```

Thanks to [this link](https://askubuntu.com/questions/377383/have-to-run-dropbox-with-dropbox-dist-dropboxd-why-not-just-dropbox):
In `~/.bash_profile`:
```bash
export PATH=~/.local/bin:$PATH
```

Then run:
```bash
mkdir -p ~/.local/bin
wget -O ~/.local/bin/dropbox.py "https://www.dropbox.com/download?dl=packages/dropbox.py"
chmod ug+x ~/.local/bin/dropbox.py
ln -s -T ~/.local/bin/dropbox.py ~/.local/bin/dropbox
source ~/.bash_profile
```

`dropbox help` should now return:
```bash
status       get current status of the dropboxd
throttle     set bandwidth limits for Dropbox
help         provide help
.
.
.
sharelink    get a shared link for a file in your dropbox
proxy        set proxy settings for Dropbox
```

Enable Dropbox autostart with:
```bash
dropbox autostart y
```

Exclude certain directories from syncing:
```bash
dropbox exclude add [DIRECTORY] [DIRECTORY]
```
