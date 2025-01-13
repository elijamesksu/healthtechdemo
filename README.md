## Project Overview  

***RHEL Evnvironment Automation and PostgreSQL Setup***  

This project demonstrates how to configure and automate a Red Hat Enterprise Linux(RHEL) evvironment to ensure reliability, scalability, and security. The setup includes configuring GRUB for system recovery, installing and optimizing PostgreSQL for database management, and automating routing administrative tasks like system updates, user management, and backups

The purpose of this project is to create a foundation for high performing and resilient IT systems, particularly in environments requiring regulatory compliance and high availability, such as healthcare IT

## 1.Setting up RHEL Environment  
***Update the system's software packages***  
```bash
#sudo = superuser or root to gain admin privileges)

> sudo dnf update -y  
```

## 2.Configure and Customize GRUB Bootloader  
***Customize GRUB for dual boot setup, kernel tuning, or recovery options***   

```bash
View kernels installed on the system

> sudo rpm -q kernel
```

***Edit the GRUB Configuration***

```bash
#Opens the GRUB configuration file for editing and controls the bootloader behavior,  
#Boot entries, timeouts, and kernel parameters.  
#Modifying allows the addition of kernel parameters and to customize boot behavior for performance optimization or troubleshooting

> sudo nano /etc/default/grub
```

***Modification to set debugging***  

```bash
#Adds parameters to the kernel command line for enhanced logging and debugging.  
#Enabling remote monitoring and debugging.
#Provides detailed logs during system boot and runtime.
#This could help Netsmart by ensuring high system availability and allow sysadmins to quickly identify and resolve issues that might effect patient amnagement or compliance systems

> GRUB_CMDLINE_LINUX="console=ttyS0,115200n8 debug"
```
![GRUB menu with changes](assets/grub%20menu.png)


***Add a recovery kernel option or modify existing entries***  

```bash
#Modifies the GRUB environment file which stores non default bootloader settings like enabling a recover kernel. Mkconfig generates a new GRUB config file. Adding a recovery kernel ensures that you have a fallback option in case of a failed boot and allows admins to boot into a minimal emvironment for troubleshooting without risking further damage

> sudo grub2-editenv /boot/grub2/grubenv create  
> sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

***Rebuild GRUB configuration***  

```bash
#Rebuilds the GRUB config file. This guarantees your system's bootloader reflects the latest configurations and avoids mismatches  

> sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```
![Grub2 mkconfig/editenv commands](assets/grub%20configs.png)


## 3.PostgreSQL

***Installs the postsql database***

```bash
#Installs the PostgreSQL server package for db management.  
#PostgreSQL-contrib includes additional utilities and extensions.
#-y means auto config  

> sudo dnf install postgresql-server postgresql-contrib -y
```
![PostgreSQL Install](assets/postgresql%20download%20.png)

  
***Initialize the PostgreSQL database***  

```bash
#Initializes a new db cluster and configures the necessary files.
#Initialization is the first step in making PostgreSQL ready to use  

> sudo postgresql-setup --initdb
```

***Start and enable PostgreSQL to run on boot***  

```bash
#Start launches the PostgreSQL service immediately.  
#Enable configures the service to start automatically on boot.  
#Netsmart probably relies on persistent databases for patient records and operations and ensuring
#PostgreSQL starts on boot reduces downtime  

> sudo systemctl start postgresql  
> sudo systemctl enable postgresql
```

***Create a database and table***  

```bash
#Switches to postgres user and launches the psql terminal  

> sudo -u postgres psql
```

Creating separate users with specific privileges ensure secure db access. Using role attributes like encoding, iso level, and timezone ensures consistency and reliability  

```sql
#Creating the database named Employees
> `postgres=#CREATE DATABASE Employees;`  

Creating a user named admin with password `12478761247876`.
> `postgres=#CREATE USER admin WITH PASSWORD '12478761247876';`  

#Configuring the client encoding for admin.
> `postgres=#ALTER ROLE admin SET client_encoding TO 'UTF_8';`  

#Setting the default transaction isolation for admin to `read committed`.
> `postgres=#ALTER ROLE admin SET default_transaction_isolation TO 'read committed';`  

#Configuring the timezone for admin to UTC.
> `postgres=#ALTER ROLE admin SET timezone TO 'UTC';`  


#Granting the admin user full control over the Employees database.
> `GRANT ALL PRIVILEGES ON DATABASE Employees TO admin;`
```

***Creating table***  

stay in -u postgres psql  

```sql
#Creating a table named `Information` with the following columns  
#`id`: A serial column that automatically increments and serves as the primary key.  
#`name`: A `VARCHAR` column that holds the name of the employee (up to 100 characters).    
#`position`: A `VARCHAR` column that holds the position of the employee (up to 100 characters).   
#`salary`: A `VARCHAR` column that holds the salary of the employee (up to 100 characters).  
`CREATE TABLE Information(`  
  `id SERIAL PRIMARY KEY,`  
  `name VARCHAR(100),`  
  `position VARCHAR(100),`  
  `salary VARCHAR(100)`  
`);`  
``` 

***Insert data into table***  

stay in -u postgres sql

This demonstrates inserting records into the database which is important for managing and storing any type of organizational data. For Netsmart it could be patient data etc.  

```sql
INSERT INTO Information(name, position, salary) VALUES('Eli', 'James', '50,000');
INSERT INTO Information(name, position, salary) VALUES('Elon', 'Tusk', '100,000');
```

***Optimize PostgreSQL***  

```bash
#Adjust parameters for performance by editing the postgresql.conf file

> sudo nano /var/lib/pgsql/data/postgresql.conf
```
![PostgreSQL Parameter Edit](assets/postgresql%20parameter%20edit.png)



```bash
# This setting demonstrates the amount of memory PostgreSQL uses for caching data.
> shared_buffers = 512MB

# This setting helps PostgreSQL determine how much memory it can use for caching queries and index scans.
> effective_cache_size = 1GB

# This setting controls the amount of memory used for sorting and hashing operations during a JOIN or ORDER BY operation.
# It enables these operations to be performed without hitting the disk for temporary storage.
> work_mem = 32MB
```
![PostgreSQL Parameter Update](assets/postgresql%20parameter%20updated.png)


***Modify the pg_hba.conf file to allow replication***

```bash
#The pg_hba.conf file controls which hosts and users can connect to the PostgreSQL server and  how they are authenticated.  
#To allow replication from specific hosts(ex. a standby server) this should be modified

> sudo nano /var/lib/pgsql/data/pg_hba.conf
```
![PostgreSQL Replication](assets/postgresql%20replication.png)


**What the modification does**
```bash
#Host allows TCP/IP connections to the server  
#Replication specifies the connection type  

> host replication all 192.168.x.x/32 md5
```

## 4.Automating Scripts

**System update script**

```bash
#Create the system_update script

> nano system_update.sh
```

```bash
Write the script

#!/bin/bash
#Automate system updates

#Update package list
> sudo dnf upgrade -y

#Upgrade installed packages
> sudo dnf upgrade -y

#Clean up unused packages and cache
> sudo dnf autoremove -y  
> sudo dnf clean all

> echo "System update completed successfully :)"
```
![BASH Script](assets/BASH%20SCript.png)


```bash
#Make the script executable

> chmod +x system_update.sh
```

```bash
#Run the script

> ./system_update.sh
```
![Running BASH Script](assets/running%20bash%20script.png)


**User management script**

```bash
Create script
> nano user_management.sh
```

```bash
Write the script
#!/bin/bash
#Automate user creation and add to sudo group

> echo "Enter new username: "
> read username

#Create user
> sudo useradd -m -s /bin/bash $username

#Set password
> sudo passwd $username

#Add user to sudo (wheel) group
> sudo usermod -aG wheel $username

> echo "User $username created and added to the sudo group."
```
![Automate User Management Script](assets/automate%20user%20management%20script.png)


```bash
#Make the script executable

> chmod +x user_management.sh
```

```bash
#Run the script

> ./user_management.sh
```
![Running BASH Script](assets/running%20bash%20script.png)


**Automate backups script with rsync**  

```bash
##Create script

> nano backup_script.sh
```

```bash
#Write the script

#!/bin/bash
#Automate system backup using rsync

> DATE=$(date + '%Y-%m-%d')
> BACKUP_DIR=" /backup/pg_backup_$DATE"

#Create a backup directory
> mkdir -p $BACKUP_DIR

#Backup inportant directories (e.g., /etc, /var, /home)
> rsync -av /etc /var /home $BACKUP_DIR/

> echo "Backup completed successfully at $BACKUP_DIR"
```
![Automate Backups Rsync Script](assets/automate%20backups%20rsync%20script.png)


```bash
#Make the script executable

> chmod +x backup_script.sh
```

```bash
#Run the script

> ./backup_script.sh
```

## 5.Firewall and Encryption

***Setting up basic firewall***
```bash
#Start firewalld

> sudo systemctl start firewalld
```
**Allow PostgeSQL port 5432 through the firewall**
```bash
#By default PostgreSQL listens on port 5432

> sudo firewall-cmd --permanent --add-port=5432/tcp
> sudo firewall-cmd --reload
```

**Verify firewall rules**
```bash
Verify the rule was added successfully

> sudo firewall-cmd --list-all
```
***Encrypting PostgreSQL Backups with GPG***
```bash
#Install GPG

> sudo dnf install gnupg
```

**Generate a GPG key**
```bash
> gpg --gen-key
```

**Create key then list keys**
```bash
> gpg --list-keys
```

**Modify script for encryption**
```bash
#Assuming PostgreSQL data directory is being backed up

> nano/home/postgres/script.sh
```

**Backup script**
```bash
#!/bin/bash
# Automate system backup using rsync

DATE=$(date +'%Y-%m-%d')
BACKUP_DIR="/backup/pg_backup_$DATE"

# Create a backup directory
mkdir -p $BACKUP_DIR

# Backup PostgreSQL databases
pg_dumpall -U postgres > $BACKUP_DIR/pg_backup.sql

# Encrypt the backup using GPG
gpg --output $BACKUP_DIR/pg_backup.sql.gpg --encrypt --recipient 'elijames@piclist.ai' $BACKUP_DIR/pg_backup.sql

# Clean up the unencrypted backup file
rm $BACKUP_DIR/pg_backup.sql

echo "Backup completed and encrypted successfully at $BACKUP_DIR/pg_backup.sql.gpg"
```

**Make script executable**
```bash
> chmod +x /home/postgres/script.sh
```

**Run the script**
```bash
> /home/postgres/script.sh
```

***Automating the backup process***
```bash
#Edit crontab

> crontab -e
```

**Add a cron job to run the backup daily**
```bash
#Add a cron job to run the backup daily at a specific time

> 0 2 * * * /home/postgres/script.sh
```

**Command Review**
```bash
#Psql command that lists all the databases in the PostgreSQL instance

> \l

#Psql command that is used to connect to a simple dtabase
> \c <name_of_database>

# Create a new database (must have appropriate privileges)
> CREATE DATABASE <database_name>;

# Drop a database (permanently deletes the database)
> DROP DATABASE <database_name>;

# List all roles and their attributes
> \du

# Create a new user/role
> CREATE USER <username> WITH PASSWORD '<password>';

# Grant a role privileges on a database
> GRANT ALL PRIVILEGES ON DATABASE <database_name> TO <username>;

# Revoke privileges from a user/role
> REVOKE ALL PRIVILEGES ON DATABASE <database_name> FROM <username>;

# Alter a role's attributes
> ALTER ROLE <username> SET <parameter> TO '<value>';

# List all schemas in the current database
> \dn

# List all tables in the current schema
> \dt

# Show the structure of a specific table
> \d <table_name>

# Create a new table
CREATE TABLE <table_name>(
    column1 datatype,
    column2 datatype
);

# Drop a table
> DROP TABLE <table_name>;

# Execute a SQL query
> SELECT * FROM <table_name>;

# Count rows in a table
> SELECT COUNT(*) FROM <table_name>;

# Insert data into a table
> INSERT INTO <table_name>(column1, column2) VALUES ('value1', 'value2');

# Update data in a table
> UPDATE <table_name> SET column1 = 'new_value' WHERE column2 = 'some_value';

# Delete data from a table
> DELETE FROM <table_name> WHERE column1 = 'value';

# Create an index on a table
> CREATE INDEX <index_name> ON <table_name>(<column_name>);

# Drop an index
> DROP INDEX <index_name>;

# Analyze query execution plan
> EXPLAIN ANALYZE <query>;

# Vacuum (clean up and optimize the database)
> VACUUM;

# Vacuum and analyze (update optimizer statistics)
> VACUUM ANALYZE;

# Show current database connection info
> \conninfo

# Display the current query buffer
> \p

# Cancel the current query
> \reset

# Check current settings
> SHOW ALL;

# View logs and troubleshoot issues (from command line, not `psql`)
> tail -f /var/log/postgresql/postgresql.log

# Display command history
> \s

# Show help for SQL commands
> \h <SQL_command>

# Quit the `psql` session
> \q

```

***How can these be relevant for Netsmart***  

Database Management - Managing multiple databases for different clients or systems  

User and Role Management - Ensure secure access to sensitive healthcare data  

Schema and Table Management - Organizes data efficiently for complex systems like patient records  

Diagnostics and Debugging - Quickly identifies and resolves performance or system issues critical for maintaining compliance and uptime  

File Management - Enables batch updates and backups, ensuring smooth database operations  

![PostgreSQL](assets/Screenshot%202025-01-12%20145259.png)

![Screenshot 2025-01-11](assets/Screenshot%202025-01-11%20105932.png)
Dracut attempt 


