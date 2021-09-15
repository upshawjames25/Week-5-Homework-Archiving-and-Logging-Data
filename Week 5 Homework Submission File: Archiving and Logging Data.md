## Week 5 Homework Submission File: Archiving and Logging Data

### Step 1: Create, Extract, Compress, and Manage tar Backup Archives

1. Command to **extract** the `TarDocs.tar` archive to the current directory:   
  `tar -xvf TarDocs.tar`  

2. Command to **create** the `Javaless_Doc.tar` archive from the `TarDocs/` directory, while excluding the `TarDocs/Documents/Java` directory:  
  `tar -cvf Javaless_Docs.tar --exclude-tag-under=Java ~/Projects/TarDocs`  
  
3. Command to ensure `Java/` is not in the new `Javaless_Docs.tar` archive:
  `tar -tf Javaless_Docs.tar`  
  `tar -tf Javaless_Docs.tar | grep -rw Java *`

**Bonus**
- Command to create an incremental archive called `logs_backup_tar.gz` with only changed files to `snapshot.file` for the `/var/log` directory:  
  `sudo tar --listed-incremental=snapshot.file -cvzf logs_backup.tar.gz /var/log`

#### Critical Analysis Question

- Why wouldn't you use the options -x and -c at the same time with tar?  
  `-c Create a new archive. and -x Extract files from an archive. You cannot create a tar file and also extract at the same time.`

---

### Step 2: Create, Manage, and Automate Cron Jobs  

1. Cron job for backing up the /var/log/auth.log file:  
  `0 6 * * 3 tar -czf /auth_backup.tgz /var/log/auth.log`

---

### Step 3: Write Basic Bash Scripts  

1. Brace expansion command to create the four subdirectories:  
  `sudo mkdir -p ~/backups/{freemem,diskuse,openlist,freedisk}`  

2. Paste your system.sh script edits below:

**sudo nano system.sh**

![bash_script](/Images-1/bash_script.PNG)  

  **`# Free memory output to a free_mem.txt file`**  
  *`free -mh > awk 'NR==2{printf "Memory Usage: %s/%sMB (%.2f%%)\n", $3,$2,$3*100/$2 }' > ~/backups/freemem/free_mem.txt`*  
  
  **`# Disk usage output to a disk_usage.txt file`**  
  *`df -h | awk '$NF=="/"{printf "Disk Usage: %d/%dGB (%s)\n", $3,$2,$5}' > ~/backups/diskuse/disk_usage.txt`*  

  **`# List open files to a open_list.txt file`**  
  *`lsof > ~/backups/openlist/open_list.txt`*  
  
  **`# Free disk space to a free_disk.txt file`**  
  *`df -h >> ~/backups/freedisk/free_disk.txt`*  

3. Command to make the system.sh script executable:  
  `chmod +x system.sh`

**Optional**  
- Commands to test the script and confirm its execution:  
  `sudo ./system.sh`  
  ![system.sh](/Images-1/system_sh.png)  
    
  `cat ~/backups/diskuse/disk_usage.txt`  
  ![disk_usage.txt](/Images-1/disk_usage_txt.png)  

**Bonus**  
- Command to copy system to system-wide cron directory:  
  `sudo cp system.sh /etc/cron.weekly`  

---

### Step 4. Manage Log File Sizes  

1. Run sudo nano /etc/logrotate.conf to edit the logrotate configuration file.  
    
    Configure a log rotation scheme that backs up authentication messages to the /var/log/auth.log.  

    - Add your config file edits below:  
    ```
    /var/log/auth.log {  
      Weekly  
      rotate 7  
      Notifempty  
      Delaycompress  
      Missingok  
      endscript  
    }  
    ```  
    
### Bonus: Check for Policy and File Violations  

1. Command to verify auditd is active:  
  `sudo systemctl status auditd`  
  ![File_Violation](/Images-1/systemctl_status_auditd.png)  

2. Command to set number of retained logs and maximum log file size:  
    - Add the edits made to the configuration file below:  
      **sudo nano /etc/audit/auditd.conf**  
      
      `max_log_file = 35`  
      `num_logs = 7`  

3. Command using auditd to set rules for /etc/shadow, /etc/passwd and /var/log/auth.log:  

    - Add the edits made to the rules file below:  
      **sudo nano /etc/audit/rules.d/audit.rules**  
      ## For the permissions to monitor and set the keyname  
      `-w /etc/shadow -p wra -k hashpass_audit`  
      `-w /etc/passwd -p wra -k userpass_audit`  
      `-w /var/log/auth.log -p wra -k authlog_audit`  

4. Command to restart auditd:  
      `sudo systemctl restart auditd`

5. Command to list all auditd rules:  
      `sudo auditctl -l`  
      ![audit_-l](/Images-1/auditctl_-l.png)  

6. Command to produce an audit report:  
     `sudo aureport -au`  
     ![aureport](/Images-1/aureport_-au.png)  

7. Create a user with sudo useradd attacker and produce an audit report that lists account modifications:  
     **sudo useradd attacker**  
     `sudo aureport -m`  
     [aureport2](/Images-1/aureport_-m.png)  

8. Command to use auditd to watch /var/log/cron:  
     `sudo auditctl -w /var/log/cron`

9. Command to verify auditd rules:  
     `sudo auditctl -l`  
     ![auditctl](/Images-1/auditctl_-l-2.png)  

### Bonus (Research Activity): Perform Various Log Filtering Techniques  

1. Command to return journalctl messages with priorities from emergency to error:  
     `sudo journalctl -b -p emerg..err`  
     ![journalctl](/Images-1/journalctl_-b_-p_emerg__err.png)
     
2. Command to check the disk usage of the system journal unit since the most recent boot:  
     `sudo journalctl -b -u systemd-journald | less`  
     ![journalctl2](/Images-1/journalctl_-b_-u_systemd-journald_less.png)  

3. Command to remove all archived journal files except the most recent two:  
     `sudo journalctl --vacuum-files=2`  
     ![journalctl3](/Images-1/journalctl_-vaccum-files=2.png)  

4. Command to filter all log messages with priority levels between zero and two, and save output to /home/sysadmin/Priority_High.txt:
     **sudo su**  
     **cd ..**  
     `journalctl -p 0..2 > /home/student/Priority_High.txt`  
     
5. Command to automate the last command in a daily cron job. Add the edits made to the crontab file below:  
     **sudo crontab -e**  
     `@daily journalctl -p 0..2 > /home/student/Priority_High.txt`  

© 2020 Trilogy Education Services, a 2U, Inc. brand. All Rights Reserved.

---
  
## :sunglasses: `Ketan Vithal Patel` :sunglasses:  


### `Thursday April 22, 2021 -- UofT Cybersecurity - Boot Camp`
#### :rose::rose:`Jai Shri Swaminarayan`:rose::rose:
```
હરે કૃષ્ણ હરે કૃષ્ણ, કૃષ્ણ કૃષ્ણ હરે હરે |  Hare Krishna Hare Krishna, Krishna Krishna Hare Hare |
હરે રામ હરે રામ, રામ રામ હરે હરે ||   Hare Ram Hare Ram, Ram Ram Hare Hare ||
```
---  
