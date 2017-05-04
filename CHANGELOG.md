## v1.0-rc2
### New features
 - More help for sshfs (how create identity file)
 - Multiple runs program (wait for PID)
 - Add -y (yes) parameter for checksum and checksize
 - Add connection timeout for sshfs protocol
 - Add protocol cifs (Windows, SAMBA)
 - Add connection timeout

### Bugfix
 - umount after connection down
 - saving data into unmounted storage

## v1.0-rc1
### New features
 - incremental and full backup (snapshot based)
 - compress backup data (gzip)
 - encrypt backup data (aes-256-cbc)
 - check backup integrity with size and checksum (sha1)
 - check backup integrity on (linux) server (sha1sum -c)
 - asynchronous send data to server
 - partition backup data on small chunks (chunk size can be set)
 - clean old local backups (remove snapshots)
 - clean old server backups (remove old backups on servers)
 - backups and servers configuration are in config files
 - add, edit and remove configurations
 - self install/uninstall
 - save program with backup configuration in file (tgz or. tgz.aes256cbc)
 - realtime log show
 - suported protocols: local, davfs, sshfs

### Bugfix
 - First version - no bug fix

