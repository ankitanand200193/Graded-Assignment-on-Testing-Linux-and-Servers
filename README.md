# Graded-Assignment-on-Testing-Linux-and-Servers

# **Task 1: System Monitoring Setup**

## **1️⃣ Install Monitoring Tools**
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y htop nmon
```

## **2️⃣ Monitor System Usage**
- `top` → View live CPU, memory, and process usage.
- `htop` → User-friendly system monitor.
- `nmon` → Performance statistics.

## **3️⃣ Check Disk Usage**
```bash
df -h        # Show disk space usage
du -sh /home/*  # Show folder sizes
```

## **4️⃣ Monitor Resource-Intensive Processes**
```bash
ps -eo pid,user,%cpu,%mem,cmd --sort=-%cpu | head -n 10
```

## **5️⃣ Automate Monitoring with Cron Jobs**
### **Create Log Directory:**
```bash
mkdir -p ~/sys_monitor_logs
```
### **Schedule Tasks Every 10 Minutes:**
```bash
crontab -e
```
#### **Add These Lines:**
```bash
*/10 * * * * top -b -n 1 >> ~/sys_monitor_logs/top.log 2>> ~/sys_monitor_logs/cron_error.log
*/10 * * * * script -q -c "htop" ~/sys_monitor_logs/htop.log 2>> ~/sys_monitor_logs/cron_error.log
*/10 * * * * nmon -f -s 10 -c 6 -m ~/sys_monitor_logs/ 2>> ~/sys_monitor_logs/cron_error.log
*/10 * * * * df -h > ~/sys_monitor_logs/disk_usage.log 2>> ~/sys_monitor_logs/cron_error.log
*/10 * * * * ps -eo pid,user,%cpu,%mem,cmd --sort=-%cpu | awk 'NR==1 || $3>0.1' > ~/sys_monitor_logs/process_monitor.log 2>> ~/sys_monitor_logs/cron_error.log
```

## **6️⃣ Verify Logs**
```bash
ls -lh ~/sys_monitor_logs/         # List log files
tail -f ~/sys_monitor_logs/top.log # View live logs
cat ~/sys_monitor_logs/cron_error.log  # Check cron errors
```

## **Troubleshooting**
**Log Files Empty?**
- Use `script -q -c "htop"` instead of `htop -b`.

**Cron Jobs Not Running?**
```bash
grep CRON /var/log/syslog   # Check cron logs
crontab -l                  # Verify cron jobs
```

## **Terminal Result**

![Alt text](Task_1_terminal_Output.png)


**System monitoring is now automated & logging!**




# **Task 2: User Management and Access Control**

### **Objective:**

Set up user accounts and configure secure access controls for new developers.

### **Scenario:**

Two new developers, Sarah and Mike, require system access.

- Each developer needs an isolated working directory to maintain security and confidentiality.
- Security policies must ensure proper password management and access restrictions.

---

## **Steps to Implement:**

### **Step 1: Create User Accounts**

Use the `useradd` command to create user accounts for Sarah and Mike.

```bash
sudo useradd -m -s /home Sarah
sudo useradd -m -s /home Mike
```

- `-m` creates the home directory.
- `-s /bin/bash` sets Bash as their default shell.

### **Step 2: Set Secure Passwords**

Set passwords for both users:

```bash
sudo passwd Sarah
sudo passwd Mike
```

### **Step 3: Create Dedicated Workspaces**

Create separate directories for each user **without using \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*****`-p`**:

```bash
sudo mkdir /home/Sarah
sudo mkdir /home/Sarah/workspace

sudo mkdir /home/Mike
sudo mkdir /home/Mike/workspace
```

### **Step 4: Set Ownership and Permissions**

Ensure only the respective users can access their workspace directories:

```bash
sudo chown Sarah:Sarah /home/Sarah/workspace
sudo chown Mike:Mike /home/Mike/workspace

sudo chmod 700 /home/Sarah/workspace
sudo chmod 700 /home/Mike/workspace
```

#### **Explanation:**

- `sudo chown Sarah:Sarah /home/Sarah/workspace`

  - `chown` changes ownership.
  - `Sarah:Sarah` sets both the **owner** and **group** to Sarah.
  - `/home/Sarah/workspace` is the target directory.
  - This ensures only Sarah has control over her workspace.

- `sudo chmod 700 /home/Sarah/workspace`

  - `chmod` sets file/directory permissions.
  - `700` means **only the owner (Sarah) has read, write, and execute permissions**.
  - **No one else (group or others) can access it**.
  - This ensures privacy and security.

The same logic applies to Mike’s workspace.

### **Step 5: Enforce Password Policy**

Install and configure `libpam-pwquality`:

```bash
sudo apt install libpam-pwquality -y
```

Edit the password policy file:

```bash
sudo nano /etc/security/pwquality.conf
```

Modify or add the following lines:

```
minlen = 12
dcredit = -1
ucredit = -1
lcredit = -1
ocredit = -1
```

- **`minlen = 12`** → Minimum password length of 12 characters.
- **`dcredit = -1`** → Requires at least one digit (0-9).
- **`ucredit = -1`** → Requires at least one uppercase letter (A-Z).
- **`lcredit = -1`** → Requires at least one lowercase letter (a-z).
- **`ocredit = -1`** → Requires at least one special character (e.g., @, #, \$).

These settings enforce strong password security.

### **Step 6: Set Password Expiry Policy**

Modify password expiration settings:

```bash
sudo chage -M 30 -m 7 -W 7 Sarah
sudo chage -M 30 -m 7 -W 7 Mike
```

#### **Explanation:**

- `sudo chage -M 30 -m 7 -W 7 Sarah`
  - `-M 30` → Password **expires every 30 days**.
  - `-m 7` → Minimum **7 days before a password change is allowed**.
  - `-W 7` → Warns the user **7 days before password expiration**.

This ensures users regularly update their passwords and get notified in advance.

### **Step 7: Verify Settings**

Check user details:

```bash
sudo ls -ld /home/Mike/workspace
sudo ls -ld /home/Sarah/workspace

sudo chage -l Sarah
sudo chage -l Mike
```

#### **Explanation:**

- `sudo ls -ld /home/Sarah/workspace`
  - `ls` → Lists directory contents.
  - `-l` → Displays detailed (long-format) information, including permissions, owner, group, size, and timestamp.
  - `-d` → Shows information **about the directory itself** rather than its contents.
  - `/home/Sarah/workspace` → The target directory.

**Purpose:**
- It verifies **ownership and permissions** of `/home/Sarah/workspace` without listing its files.
- Helps confirm that only Sarah has access (if `chmod 700` is correctly applied).

- `sudo chage -l Sarah`
  - `chage` is used to **display or modify** password aging information.
  - `-l` lists current settings for **password expiration, minimum days, warning period, and inactivity**.
  - This allows the administrator to check Sarah’s password expiration details.

- `sudo chage -l Mike`
  - Performs the same check for Mike.
  - Ensures that Mike’s password policies are correctly applied.

- `drwx------ 2 Mike Mike 4096 Mar 16 03:34 /home/Mike/workspace`

  - `drwx------` → Directory (`d`), **owner has full permissions (rwx)**, no access for group or others.
  - `2` → Hard link count (typically refers to directories linking to `.` and `..`).
  - `root root` → Owner and group are **both root**.
  - `4096` → Directory size in bytes.
  - `Mar 16 03:34` → Last modification date/time.
  - `/home/Mike/workspace` → The directory path.

## **Terminal Results**

! [Alt text ](Task_2_User_Management.png)

 
# Task 3 : Automated Backup Configuration for Apache and Nginx Servers

## **Objective**

Automate the backup process for:

- **Sarah’s Apache Server**
- **Mike’s Nginx Server**

## **Scenario**

- Sarah manages an **Apache Web Server**.
- Mike manages an **Nginx Web Server**.
- Both require **scheduled backups** of configurations and document roots to ensure **data integrity and disaster recovery**.

## **Backup Requirements**

Each backup should:

- Include configuration files and document root.
- Be compressed and stored in `/backups/`.
- Be scheduled for **every Tuesday at 12:00 AM**.
- Include **integrity verification**.

---

# **Prerequisites**

### **1. Create Backup Directory**

Ensure the `/backups/` directory exists and is accessible:

```bash
sudo mkdir -p /backups
sudo chmod 777 /backups
```

### **2. Install ********`tar`******** Utility**

Ensure the `tar` command is installed:

```bash
sudo apt update && sudo apt install tar -y  # Debian/Ubuntu
sudo yum install tar -y  # RHEL/CentOS
```

### **3. Verify Apache and Nginx Installation**

For Apache:

```bash
sudo systemctl status httpd
```

For Nginx:

```bash
sudo systemctl status nginx
```

If not installed:

```bash
sudo apt install apache2 -y
sudo yum install httpd -y
sudo apt install nginx -y
sudo yum install nginx -y
```

---

# **Step 1: Create Backup Scripts**

## **1.1 Apache Backup Script (Sarah)**

Create the script:

```bash
sudo nano /usr/local/bin/apache_backup.sh
```

**Content:**

```bash
#!/bin/bash
DATE=$(date +'%Y-%m-%d')
BACKUP_DIR="/backups"
APACHE_CONF="/etc/apache2/apache2.conf"
APACHE_HTML="/var/www/html/"
BACKUP_FILE="$BACKUP_DIR/apache_backup_$DATE.tar.gz"
LOG_FILE="$BACKUP_DIR/apache_backup.log"

tar -czf "$BACKUP_FILE" -C / etc/apache2/apache2.conf

if tar -tzf "$BACKUP_FILE" &>/dev/null; then
    echo "$DATE - Apache backup successful: $BACKUP_FILE" >> "$LOG_FILE"
else
    echo "$DATE - Apache backup failed!" >> "$LOG_FILE"
fi
```

Make it executable:

```bash
sudo chmod +x /usr/local/bin/apache_backup.sh
```

## **1.2 Nginx Backup Script (Mike)**

Create the script:

```bash
sudo nano /usr/local/bin/nginx_backup.sh
```

**Content:**

```bash
#!/bin/bash
DATE=$(date +'%Y-%m-%d')
BACKUP_DIR="/backups"
NGINX_CONF="/etc/nginx/"
NGINX_HTML="/usr/share/nginx/html/"
BACKUP_FILE="$BACKUP_DIR/nginx_backup_$DATE.tar.gz"
LOG_FILE="$BACKUP_DIR/nginx_backup.log"

tar -czf "$BACKUP_FILE" "$NGINX_CONF" "$NGINX_HTML"

if tar -tzf "$BACKUP_FILE" &>/dev/null; then
    echo "$DATE - Nginx backup successful: $BACKUP_FILE" >> "$LOG_FILE"
else
    echo "$DATE - Nginx backup failed!" >> "$LOG_FILE"
fi
```

Make it executable:

```bash
sudo chmod +x /usr/local/bin/nginx_backup.sh
```

---

# **Step 2: Schedule the Cron Jobs**

Edit the cron job file:

```bash
crontab -e
```

For **Sarah (Apache backup)**:

```
0 0 * * 2 /usr/local/bin/apache_backup.sh
```

For **Mike (Nginx backup)**:

```
0 0 * * 2 /usr/local/bin/nginx_backup.sh
```

- `0 0 * * 2` → Runs **every Tuesday at 12:00 AM**.

---

# **Step 3: Verify the Backups**

### **3.1 Test Scripts Manually**

For Apache:

```bash
sudo  /usr/local/bin/apache_backup.sh
```

For Nginx:

```bash
sudo /usr/local/bin/nginx_backup.sh
```

### **3.2 Check Backup Files**

```bash
ls -lh /backups/
```

Expected Output:

```
-rw-r--r-- 1 root root  5.0M Mar 16 00:00 apache_backup_YYYY-MM-DD.tar.gz
-rw-r--r-- 1 root root  4.2M Mar 16 00:00 nginx_backup_YYYY-MM-DD.tar.gz
```

### **3.3 Verify Backup Logs**

```bash
cat /backups/apache_backup.log
cat /backups/nginx_backup.log
```

Expected Output:

```
2025-03-16 - Apache backup successful: /backups/apache_backup_YYYY-MM-DD.tar.gz
2025-03-16 - Nginx backup successful: /backups/nginx_backup_YYYY-MM-DD.tar.gz
```

### **3.4 Verify Backup Integrity**

```bash
tar -tzf /backups/apache_backup_YYYY-MM-DD.tar.gz
tar -tzf /backups/nginx_backup_YYYY-MM-DD.tar.gz
```

No errors mean the backup is valid.

## **Terminal Results**


# **Color Coding in Terminal**

If you see `.tar.gz` files appearing **red** in your terminal output, this is normal.

### **Why Are They Red?**

The color coding depends on your terminal’s **LS\_COLORS** setting:

- **Red** → Compressed files (`.tar.gz`, `.zip`, `.rar`).
- **Blue** → Directories.
- **Green** → Executable files.

✅ **No issues here! Your backup files look fine.** 🚀

---

# **Final Summary**

✔ **Backup scripts created** for Apache and Nginx.\
✔ **Cron jobs scheduled** for every Tuesday at 12:00 AM.\
✔ **Backup files stored** in `/backups/` with timestamps.\
✔ **Verification logs generated** for tracking success/failure.








