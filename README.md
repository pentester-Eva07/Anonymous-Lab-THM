# TryHackMe – Anonymous Playground

**Difficulty:** Hard  
**Operating System:** Linux  
**Focus:** Web Enumeration, Binary Analysis and Privilege Escalation

## 1. Web Enumeration and Credential Discovery

I started by examining the target's web application and its accessible resources.

During enumeration, I inspected `robots.txt` and discovered a hidden directory containing an operative list. I continued investigating the application and encountered a cookie-based restriction. By examining and modifying the relevant cookie information, I was able to continue accessing the application.

I also discovered encoded information. I used Python to analyse and decode it, which revealed credentials for the next stage.

**Screenshot 1:** Hidden directory and Python decoding results. Redact any credentials.

## 2. Initial Access and Linux Enumeration

Using the recovered credentials, I established an SSH connection to the target machine.

After gaining access, I started enumerating the Linux system, including its files, permissions and available binaries. During this process, I discovered an interesting SUID binary that required further analysis.

**Screenshot 2:** Successful SSH access and SUID binary discovery.

## 3. Binary Analysis and User Privilege Escalation

I used `strings` to inspect the binary and Radare2 to analyse its functions.

I examined the `main` function and the `call_bash` function. During the analysis, I identified the use of `gets()`, which does not properly check input length and can introduce a buffer overflow vulnerability.

I investigated the vulnerability and successfully obtained access as the user `spooky`.

**Tools:** `strings`, Radare2

**Screenshot 3:** Relevant binary analysis and successful access as `spooky`.

## 4. Root Privilege Escalation

After gaining access as `spooky`, I inspected `/etc/crontab` and discovered a scheduled task running every minute with root privileges:

```bash
*/1 * * * * root cd /home/spooky && tar -zcf /var/backups/spooky.tgz *
```

The wildcard used in the `tar` command created an opportunity for argument injection.

I then discovered a `.webscript` file. It compiled a C source file into an executable, assigned SUID permissions and removed temporary files.

I investigated how the scheduled task could trigger this script. After preparing the required files and allowing the cronjob to run, a root-owned SUID executable was created:

`/home/spooky/.cache/.cachefile`

Executing this binary gave me root privileges. I confirmed the result using the `id` command.

**Screenshot 4:** Cronjob, generated SUID executable and terminal output confirming root access.

## 5. Conclusion

I successfully completed the Anonymous Playground machine and obtained root-level access.

During this assessment, I practised:

- Web enumeration and cookie inspection
- Python scripting and decoding
- SSH access and Linux enumeration
- SUID permissions and binary analysis
- Buffer overflow vulnerability analysis
- Cronjob enumeration
- Tar wildcard argument injection

The lab helped me understand how vulnerabilities in applications, binaries and Linux configurations can be combined to escalate privileges.

**Final result:** Root access obtained.
