## RHEL Evnvironment Automation and PostgreSQL Setup
Netsmart demo

## Project Overview
This project demonstrates how to configure and automate a Red Hat Enterprise Linux(RHEL) evvironment to ensure reliability, scalability, and security. The setup includes configuring GRUB for system recovery, installing and optimizing PostgreSQL for database management, and automating routing administrative tasks like system updates, user management, and backups

The purpose of this project is to create a foundation for high performing and resilient IT systems, particularly in environments requiring regulatory compliance and high availability, such as healthcare IT

## 1.Setting up RHEL Environment
-sudo dnf update -y
(updates the system's software packages)
(sudo = superuser or root to gain admin privileges)

Configure GRUB Bootloader 
Customize GRUB for dual boot setup, kernal turning, or add recovery options

view kernels installed on the system
-sudo rpm -q kernel

edit file -- open the GRUB configuration file
-sudo nano /etc/default/grub

modification to set debugging
-GRUB_CMDLINE_LINUX="console=ttyS0,115200n8 debug"

add a recovery kernel option or modify existing entries
-sudo grub2-editenv /boot/grub2/grubenv create
-sudo grub2-mkconfig -o /boot/grub2/grub.cfg

Rebuild gurb configuration
-sudo grub2-mkconfig -o /boot/grub2/grub.cfg


Install and configure PostgreSQL
-sudo dnf install postgresql-server postgresql-contrib -y

Initialize the postsql database
-sudo postgresql-setup --initdb

Start and enable PostgreSQL to run on boot
-sudo systemctl start postgresql
-sudo systemctl enable postgresql

Create a simple database and table
-sudo -u postgres psql
postgres=#CREATE DATABASE Employees;
postgres=#CREATE USER admin WITH PASSWORD '12478761247876'
postgres=#ALTER ROLE admin SET client_encoding TO 'UTF_8';
postgres=#ALTER ROLE admin SET default_transaction_isolation TO 'read commited';
postgres=#ALTER ROLE admin SET timezone TO 'UTC';
GRANT ALL PRIVILEGES ON DATABASE Employees TO admin;

Creating table
stay in -u postgres psql
postgres=#CREATE TABLE Information(
postgres(#id SERIAL PRIMARY KEY,
postgres(#name VARCHAR(100),
postgres(#position VARCHAR(100),
postgres(#salary VARCHAR(100),
postgres(# );

Insert data into table 
stay in -u postgres psql 
INSERT INTO Information(name, position, salary) VALUES('Eli', 'James', '50,000');
INSERT INTO Information(name, position, salary) VALUES('Elon', 'Tusk', '100,000');

Optimize PostgreSQL adjust parameters for performance by editing the postgresql.conf file
sudo nano /var/lib/pgsql/data/postgresql.conf
shared_buffers 512MB
effective_cache_size = 1GB
work_mem = 32MB

Modify the pg_hba.conf file to allow replication
sudo nano /var/lib/pgsql/data/pg_hba.conf
ADD
host replication all 192.168.x.x/32 md5

List all databases 
\l
connects to the specific database
\c +name of db+

YOU CAN CONFIGURE POSTGRESQL REPLICATION(MASTER SLAVE SETUPS)

AUTOMATING SCRIPTS
-Create the script
nano system_update.sh
Write the script
#!/bin/bash
#Automate system updates
#Update package list
sudo dnf upgrade -y
#Upgrade installed packages
sudo dnf upgrade -y
#Clean up unused packages and cache
sudo dnf autoremove -y
sudo dnf clean all
echo "System update completed successfully :)"
-Make the script executable
chmod +x system_update.sh
-Run the script
./system_update.sh

Automate user Management 
Create script
nano user_management.sh
Write the script
#Automate user creation and add to sudo group
echo "Enter new username: "
read username
#Create user
sudo useradd -m -s /bin/bash $username
#Set password
sudo passwd $username
#Add user to sudo (wheel) group
sudo usermod -aG wheel $username
echo "User $username created and added to the sudo group."
Make the script executable
chmod +x user_management.sh
Run the script
./user_management.sh

AUTOMATE BACKUPS USING rsync
nano backup_script.sh
Write the script
#!/bin/bash
#Automate system backup using rsync
DATE=$(date + '%Y-%m-%d')
BACKUP_DIR=" /backup/pg_backup_$DATE"
#Create a backup directory
mkdir -p $BACKUP_DIR
#Backup inportant directories (e.g., /etc, /var, /home)
rsync -av /etc /var /home $BACKUP_DIR/
echo "Backup completed successfully at $BACKUP_DIR"
Make the script executable
chmod +x backup_script.sh
Run the script
./backup_script.sh

System monitoring with htop or dstat
install htop or dstat
sudo dnf install htop dstat


CONFIGURE SSH FOR AUTOMATION AND SECURITY

Install anmd configure ssh
sudo dnf install openssh-server -y


+++ Attempted CUSTOMIZE INITRD bit there was something wrong with the drivers 
install required packages for initrd modification
sudo dnf install dracut+++++++sudo systemctl









  
