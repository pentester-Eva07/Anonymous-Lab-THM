# TryHackMe – Anonymous Playground

**Difficulty:** Hard  
**Operating System:** Linux  
**Focus:** Web Enumeration, Binary Analysis and Privilege Escalation

## 1. Web Enumeration and Credential Discovery

I started by examining the target's web application and its accessible resources.

During enumeration, I inspected `robots.txt` and discovered a hidden directory containing an operative list.
<img width="1920" height="816" alt="Screenshot (439)" src="https://github.com/user-attachments/assets/6322f34a-18ac-438a-a925-469a7b49e8e8" />

I continued investigating the application and encountered a cookie-based restriction.
<img width="1920" height="804" alt="Screenshot (440)" src="https://github.com/user-attachments/assets/6799e42b-b370-4264-a419-40f190ba7c9f" />

By examining and modifying the relevant cookie information, I was able to continue accessing the application.
<img width="1920" height="824" alt="Screenshot (441)" src="https://github.com/user-attachments/assets/0f6fe2c9-642e-4848-afc8-c86e7de26bdc" />
<img width="1920" height="809" alt="Screenshot (442)" src="https://github.com/user-attachments/assets/8646bc0c-4b0a-4830-84c7-f5a518da039d" />


I also discovered encoded information. I used Python to analyse and decode it, which revealed credentials for the next stage.
<img width="1920" height="822" alt="Screenshot (445)" src="https://github.com/user-attachments/assets/314ed45f-ed67-4b98-9902-c72997aae8d3" />
<img width="1920" height="815" alt="Screenshot (446)" src="https://github.com/user-attachments/assets/025ca6e3-5a8b-4321-8c2c-b520cd42bc82" />
<img width="1920" height="818" alt="Screenshot (447)" src="https://github.com/user-attachments/assets/56aec289-033f-403c-bd53-7fa623443719" />
<img width="1920" height="806" alt="Screenshot (448)" src="https://github.com/user-attachments/assets/74b74731-c692-41ae-bc6d-38e3050a6c43" />
<img width="1920" height="820" alt="Screenshot (449)" src="https://github.com/user-attachments/assets/72469fc4-5ffc-456e-a01c-6e1d1c75c2be" />
<img width="1920" height="806" alt="Screenshot (450)" src="https://github.com/user-attachments/assets/a4a7829e-f69d-400c-a4b0-d58c31b03e32" />








## 2. Initial Access and Linux Enumeration

Using the recovered credentials, I established an SSH connection to the target machine.
<img width="1920" height="802" alt="Screenshot (452)" src="https://github.com/user-attachments/assets/7e0db683-f6c5-4eee-8a2e-57d131bc3c1e" />

After gaining access, I started enumerating the Linux system, including its files, permissions and available binaries.
<img width="1920" height="824" alt="Screenshot (453)" src="https://github.com/user-attachments/assets/57956157-545c-4523-9b8c-e3f0b7b017a9" />

During this process, I discovered an interesting SUID binary that required further analysis.


## 3. Binary Analysis and User Privilege Escalation

I used `strings` to inspect the binary and Radare2 to analyse its functions.
<img width="1920" height="817" alt="Screenshot (454)" src="https://github.com/user-attachments/assets/ae7bf4e5-4a19-4ddd-b26c-2d0ce99b3dae" />
<img width="1920" height="795" alt="Screenshot (455)" src="https://github.com/user-attachments/assets/b81855b6-6b25-4c03-88fe-15e221c48835" />


I examined the `main` function and the `call_bash` function.
<img width="1920" height="815" alt="Screenshot (456)" src="https://github.com/user-attachments/assets/326f74a1-a3a0-403e-b223-ecb0ee058d2f" />
<img width="1920" height="806" alt="Screenshot (457)" src="https://github.com/user-attachments/assets/5686b5a8-cc08-4909-864f-dceedf3f8994" />
<img width="1920" height="811" alt="Screenshot (458)" src="https://github.com/user-attachments/assets/7970b2fb-a1a8-48e5-90e0-42a724a4c0dd" />
<img width="1920" height="815" alt="Screenshot (459)" src="https://github.com/user-attachments/assets/43db81e3-b169-4a5f-aa57-f5c823ea7340" />


During the analysis, I identified the use of `gets()`, which does not properly check input length and can introduce a buffer overflow vulnerability.

I investigated the vulnerability and successfully obtained access as the user `spooky`.
<img width="1920" height="791" alt="Screenshot (462)" src="https://github.com/user-attachments/assets/4785bffc-43e6-4375-9fd1-9ec736c0d117" />
<img width="1920" height="820" alt="Screenshot (463)" src="https://github.com/user-attachments/assets/943e8b6b-8238-4a61-aaf8-91bee42d2ddd" />


**Tools:** `strings`, Radare2



## 4. Root Privilege Escalation

After gaining access as `spooky`, I inspected `/etc/crontab` and discovered a scheduled task running every minute with root privileges:
<img width="1920" height="800" alt="Screenshot (460)" src="https://github.com/user-attachments/assets/de43575e-14bf-4451-8afc-c50dafe3d833" />


```bash
*/1 * * * * root cd /home/spooky && tar -zcf /var/backups/spooky.tgz *
```

The wildcard used in the `tar` command created an opportunity for argument injection.

I further investigated the scheduled task and identified an argument injection vulnerability involving the wildcard in the tar command.

I examined how the scheduled task could be abused in the lab environment and verified the resulting privilege escalation.

After completing the process, I obtained root-level access and confirmed it using the id command.
<img width="1920" height="815" alt="Screenshot (466)" src="https://github.com/user-attachments/assets/8aaa3636-6c71-4bb2-a3b2-0f1895513320" />
<img width="1920" height="1080" alt="Screenshot (467)" src="https://github.com/user-attachments/assets/3946c939-64f2-41a3-b65d-b33d5760964c" />
<img width="1920" height="810" alt="Screenshot (468)" src="https://github.com/user-attachments/assets/eebb4d18-d84c-4477-99fd-1a90bf578be1" />






Executing this binary gave me root privileges. I confirmed the result using the `id` command.
<img width="1920" height="779" alt="Screenshot (469)" src="https://github.com/user-attachments/assets/7162f97f-0051-4e7a-9c2d-15e777484301" />





<img width="1920" height="791" alt="Screenshot (470)" src="https://github.com/user-attachments/assets/33434f5c-9656-44bd-8b0d-df110f56e1f5" />



## 5. Conclusion

I successfully completed the Anonymous Playground machine and obtained root-level access.
The assessment provided practical experience in identifying and analysing vulnerabilities across multiple layers of a Linux-based environment.

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
