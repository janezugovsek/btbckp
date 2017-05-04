# btbckp
is a CLI backup tool for BTRFS writning in bash program language.

### Current version v1.0-rc2
### Use the program at your own risk

## Features
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
 - current suported protocols: local, davfs, sshfs

## Known limitations and bugs
 - Works only with BTRFS
 - Can not change passphrase for backups

## Required libraries and software
### basic
 - btrfs v3.12+ (btrfs-tools)
 - bash 4.3.11+
 - pv 1.2.0+ (pipe viewer)
 - dd 8.21+ (coreutils)
 - openssl 1.0+
 - gzip 1.6+
 - nano 2.2.6+ (GNU nano)
 - lockfile-progs 0.1.12

Debian like distributions:

	sudo apt-get install bash btrfs-tools pv coreutils openssl gzip nano

### for ssh protocol
 - sshfs

Debian like distributions:

	sudo apt-get install sshfs

### for caldav protocol
 - davfs

Debian like distributions:

	sudo apt-get install davfs2

### for cifs protocol
 - cifs

Debian like distributions:

	sudo apt-get install cifs-utils

# How to use it
## Install on system
### Install

	git clone https://github.com/janezugovsek/btbckp.git
	cd btbckp
	sudo ./btbckp install

### UnInstall

	sudo btbckp uninstall

## Set configuration
### Set server(s) configuration
Create new server configuration with name 'myhomeserver' and communication protocol 'ssh'.
You can use other suported protocols (local - local storage, davfs - caldav storage).

	sudo btbckp server add myhomeserver -p sshfs

Then you can change parameters for access to your SSH server (hostname, user, port etc.).
If you want to change or remove existing configuration for server 'myhomeserver' use:

	sudo btbckp server edit myhomeserver 
	sudo btbckp server rm myhomeserver

### Set backup(s) configuration
Create new backup configuration with name 'rootfs'

	sudo btbckp backup add rootfs

If you want to change or remove existing configuration for backup 'rootfs' use:

	sudo btbckp backup edit rootfs
	sudo btbckp backup rm rootfs

## "Work" functions
### Create backup
Create full or incremental 'rootfs' backup and add comment (optional).

	sudo btbckp full -b rootfs -c "manual backup"
	sudo btbckp inc -b rootfs -c "manual backup"


### Send backup to server
Send new 'rootfs' backups to server 'myhomeserver'.

	sudo btbckp send -b rootfs -s myhomeserver


### Check integrity on server
Check size ('checksize') is fastest way to check integrity. If connection with server was droped, files have different sizes. For check complete backups integrity with 'checksum', program must download all backups!
If checksum or checksize is wrong, program delete bad backup (backup chunk). Missing backups will be updated in next sending (btbckp send ...).
So, check 'rootfs' backup on 'myhomeserver' server

	sudo btbckp checksize -b rootfs -s myhomeserver
	sudo btbckp checksum -b rootfs -s myhomeserver

If you have access to the server, you can separate run checksum and manually delete bad backups.

	sha1sum -c sha1sum.log


### Clean local backups
Remove 'rootfs' backup, older than "10 days" (don't forget minus -)!. Cleanlocal remove one full backup and its incremental parts. For deleting "10 days" old backups, the last incremental backup must be older than "10 days". You can not delete current backup chain. 

	sudo btbckp cleanlocal -b rootfs -t "-10 days"

### Clean server backups
Remove 'rootfs' backups on 'myhomeserver' server configuration older than "10 days" (don't forget minus -)!. Servers must have older backups as local. (if not, they will be sent in the next 'send').

	sudo btbckp cleanlocal -b rootfs -s myhomeserver -t "-10 days"

## Other usefull function
### Realtime log show
Some function such as inc, full, send, cleanserver, cleanlocal (see help) have optional parameters '-l' (save log) and '-e' (save errors).
If you use cron job you can call:

	sudo btbckp full -b rootfs -c "manual backup" -e errorlog -l statuslog
	sudo btbckp inc -b rootfs -c "manual backup" -e errorlog -l statuslog
	sudo btbckp send -b rootfs -s myhomeserver -e errorlog -l statuslog

For live show log use:

	sudo btbckp showlog statuslog
	sudo btbckp showlog errorlog


### Save backup configuration and program
You can save configuration and program (as tgz or crypted tgz.aes256cbc).

	sudo btbckp save myfilename

## Restore
Restore last 10 days backup 'rootfs' from server 'myhomeserver'

	sudo btbckp restore -b rootfs -s myhomeserver -t "-10 days"

# How to test it
## Create backup test enviroment
Create new BTRFS filesystem with size 1GB

	sudo dd if=/dev/zero of=/home/$USER/backupfs.btrfs bs=1M count=1024
	sudo mkfs.btrfs /home/$USER/backupfs.btrfs

Mount and create test enviroment with subvolumes

	mkdir /home/$USER/backup_test
	sudo mount /home/$USER/backupfs.btrfs /home/$USER/backup_test
	sudo btrfs subvolume create /home/$USER/backup_test/rootfs
	sudo btrfs subvolume create /home/$USER/backup_test/home
	sudo btrfs subvolume create /home/$USER/backup_test/var
	sudo btrfs subvolume create /home/$USER/backup_test/other

## Create server configuration
Create backup local server storage
	
	mkdir /home/$USER/backups
	sudo btbckp server add localstorage -p local

Change default configuration

	DIR=/home/$USER/backups

NOTE: instead $USER use real username

## Create backup configuration

	sudo btbckp backup add localbackup

Change default configuration

	BLOCKDEV=/home/$USER/backupfs.btrfs
	SUBVOL=rootfs
	DEST=localstorage 

NOTE: instead $USER use real username

## Change data on mounted disk
You can change data in subvolume rootfs (/home/$USER/backup_test/rootfs).
For example create 3 random files, sizes: 20MB, 30MB and 40MB.

	sudo dd if=/dev/urandom of=/home/$USER/backup_test/rootfs/randomfile0 bs=1M count=20
	sudo dd if=/dev/urandom of=/home/$USER/backup_test/rootfs/randomfile1 bs=1M count=30
	sudo dd if=/dev/urandom of=/home/$USER/backup_test/rootfs/randomfile2 bs=1M count=40

## Create backup

	sudo btbckp full -b localbackup
	sudo btbckp inc -b localbackup

## Send backup to the server
	
	sudo btbckp send -b localbackup -s localstorage

## Restore backup

Umount btrfs dirs

	sudo umount -l /home/$USER/backup_test
	sudo umount -l /var/btbckp/*

Create empty btrfs storage and mount
	
	sudo dd if=/dev/zero of=/home/$USER/backupfs.btrfs bs=1M count=1024
	sudo mkfs.btrfs /home/$USER/backupfs.btrfs
	sudo mount /home/$USER/backupfs.btrfs /home/$USER/backup_test

Restore backup from local server
	
	sudo btbckp restore -b localbackup -s localstorage

You can look, in director '/home/$USER/backup_test/rootfs' and here are restored backups. Backup files are in directory '/home/$USER/backups'

## Remove test enviroment

	sudo umount -l /home/$USER/backup_test
	sudo umount -l /var/btbckp/*
	sudo rm -rf /home/$USER/backup_test/
	sudo rm -rf /home/$USER/backupfs.btrfs
	sudo rm -rf /home/$USER/backups

## uninstall program
	
	sudo btbckp uninstall

