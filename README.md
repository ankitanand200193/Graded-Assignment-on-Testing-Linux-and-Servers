# Graded-Assignment-on-Testing-Linux-and-Servers

# **Task 1:System Monitoring Setup**

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

**System monitoring is now automated & logging!**

