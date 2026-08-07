**DSW-A1 , A2 , B1 , B2:**

نام هر سوئیچ مطابق توپولوژی پروژه :
```
enable
conf t
hostname DSW,CSW
do write
```
تنظیمات امنیتی اولیه بر روی تمامی سوئیچ‌ ها :
```
enable algorithm-type scrypt secret finalpnumain
username mobin algorithm-type scrypt secret @finalpnumain
```
تنظیمات خط کنسول:
```
line console 0
login local
exec-timeout 30
logging synchronous
```

---
### کانفیگ EtherChannel
**DSW-A1 , A2 :**

در دفتر **A** ارتباط بین **DSW-A1** و **DSW-A2** از طریق دو لینک گیگابیتی برقرار می‌شود.
ابتدا بررسی وضعیت همسایه ها :
```
do show cdp neighbors
```


دو اینترفیس G1/0/4 و G1/0/5 را عضو **Port-Channel1** کرده :
```
interface range g1/0/4-5
channel-group 1 mode desirable
```
بررسی وضعیت:
```
do show etherchannel summary
```
---
**DSW-B1 , B2 :**

در دفتر B برای تشکیل EtherChannel از پروتکل **LACP** استفاده می‌شود، بنابراین حالت **active** تنظیم می‌شود.

```
interface range g1/0/4-5
channel-group 1 mode active
```
بررسی وضعیت:
```
do show etherchannel summary
```


---
### پیکربندی لینک‌ های Trunk

**دفتر A :** 
ابتدا بررسی وضعیت همسایه ها :
```
do show cdp neighbors
```

اینترفیس‌ های **G1/0/1 تا G1/0/3** به سوئیچ‌ های Access متصل بوده و برای اینکه این لینک‌ ها بتوانند ترافیک چند VLAN را منتقل کنند آن ها را در حالت **Trunk** تنظیم می کنیم 
و قابلیت **DTP** را روی این پورت‌ ها غیرفعال میکنیم :

**DSW-A1 , DSW-A2 :**
```
interface range g1/0/1-3
switchport mode trunk
switchport nonegotiate
switchport trunk native vlan 1000
switchport trunk allowed vlan 10,20,40,99
```
همان تنظیمات را روی **Port-Channel** اعمال میکنیم :
```
interface port-channel1
switchport mode trunk
switchport nonegotiate
switchport trunk native vlan 1000
switchport trunk allowed vlan 10,20,40,99
exit
```
---
**دفتر B :**
در این دفتر تنها تفاوت، استفاده از **VLAN 30** به جای **VLAN 40** است.
**DSW-B1 , DSW-B2 :** 
```
interface range g1/0/1-3
switchport mode trunk
switchport nonegotiate
switchport trunk native vlan 1000
switchport trunk allowed vlan 10,20,30,99
```
همان تنظیمات را روی **Port-Channel** اعمال میکنیم :
```
interface port-channel1
switchport mode trunk
switchport nonegotiate
switchport trunk native vlan 1000
switchport trunk allowed vlan 10,20,30,99
exit
```

---
### پیکربندی VTP
در هر دفتر، یکی از سوئیچ‌ های Distribution به عنوان **VTP Server** و سایر سوئیچ‌ ها به عنوان **VTP Client** عمل می‌کنند.

نام دامنه **finalpnumain** و نسخه **VTP Version 2** تنظیم می‌شود.

بررسی وضعیت فعلی VTP :
```
do show vtp status
```

کافی است یک نام دامنه (Domain name) برای VTP تعریف کرده و نسخه آن را روی VTP Version 2 تنظیم کنیم.
**DSW-A1:**
فعال کردن VTP در دفتر A :
```
vtp domain finalpnumain
vtp version 2
```

فعال کردن VTP در دفتر B :
**DSW-B1:**
```
vtp domain finalpnumain
vtp version 2
```

---
### ایجاد VLANها
**دفتر A**
ایجاد VLANهای مربوط به کاربران، تلفنهای IP، شبکه بی‌سیم - Wi-Fi، مدیریت و Native
**DSW-A1**
```
vlan 10
name PCs

vlan 20
name Phones

vlan 40
name Wi-Fi

vlan 99
name Management

exit
```
بررسی وضعیت  VLAN های ایجاد شده:
```
do show vlan brief
```

**دفتر B**
در دفتر B به جای VLAN 40 از  VLAN 30 برای سرور ها استفاده می‌شود.
**DSW-B1**
```
vlan 10
name PCs

vlan 20
name Phones

vlan 30
name Servers

vlan 99
name Management

exit
```
بررسی وضعیت  VLAN های ایجاد شده:
```
do show vlan brief
```
---
### خاموش کردن پورت‌ های بلا استفاده
برای افزایش امنیت، تمامی پورت‌ هایی که در حال حاضر مورد استفاده قرار نمی‌گیرند را خاموش میکنیم

**DSW-A1 , A2 , B1 , B2:**
بررسی وضعیت اینترفیس‌ ها : 
```
do show interface status 
```

پورت‌ های بلا استفاده را انتخاب و غیرفعال کرده
```
interface range g1/0/6-24,g1/1/3-4
shutdown
exit
do write
```

----
---
### فعال سازی قابلیت مسیریابی لایه 3
سوئیچ‌ های Distribution از نوع **لایه 3** هستند، قابلیت مسیریابی IP را فعال می کنیم .
**DSW-A1 ,A2 ,B1 ,B2**

```
ip routing
```
بررسی فعال بودن قابلیت مسیریابی:
```
do show ip route
```
---
---
### پیکربندی لینک‌ های لایه 3
پیکربندی آدرس های IP روی اینترفیس‌ های ارتباطی بین سوئیچ‌ های Distribution و Core، به همراه ایجاد یک اینترفیس Loopback

قابلیت سوئیچینگ لایه دوم روی اینترفیس غیرفعال شود
**DSW-A1**
```
interface g1/1/1
no switchport
ip address 10.0.0.46 255.255.255.252

interface g1/1/2
no switchport
ip address 10.0.0.62 255.255.255.252

interface loopback0
ip address 10.0.0.79 255.255.255.255
```

**DSW-A2**
```
interface g1/1/1
no switchport
ip address 10.0.0.50 255.255.255.252

interface g1/1/2
no switchport
ip address 10.0.0.66 255.255.255.252

interface loopback0
ip address 10.0.0.80 255.255.255.255
```

**DSW-B1**
```
interface g1/1/1
no switchport
ip address 10.0.0.54 255.255.255.252

interface g1/1/2
no switchport
ip address 10.0.0.70 255.255.255.252

interface loopback0
ip address 10.0.0.81 255.255.255.255
```

**DSW-B2**
```
interface g1/1/1
no switchport
ip address 10.0.0.58 255.255.255.252

interface g1/1/2
no switchport
ip address 10.0.0.74 255.255.255.252

interface loopback0
ip address 10.0.0.82 255.255.255.255
```

بررسی وضعیت اینترفیس ها
```
do show ip interface brief
```
---
---
### HSRP 
با افزایش اولویت DSW-A1 به اندازه **5 واحد** بالاتر از مقدار پیشفرض (**100**) ، آن را به عنوان Active Router تنظیم می کنیم.

سوئیچ‌ های Distribution وظیفه فراهم کردن Default Gateway برای هر subnet را بر عهده دارند و با استفاده از HSRP افزونگی لازم را ایجاد می‌کنند.

سوئیچ DSW-A1 برای گروه‌ های **1** و **2** به عنوان **Active** و برای گروه‌ های **3** و **4** به عنوان **Standby** عمل خواهد کرد.

- **Group 1** - **Management (VLAN 99)**
- **Group 2** - **PCs (VLAN 10)**
- **Group 3** - **Phones (VLAN 20)**
- **Group 4** - **Wi-Fi (VLAN 40)**

**DSW-A1**
```
interface vlan 99
ip address 10.0.0.2 255.255.255.240
standby version 2
standby 1 ip 10.0.0.1
standby 1 priority 105
standby 1 preempt

interface vlan 10
ip address 10.1.0.2 255.255.255.0
standby version 2
standby 2 ip 10.1.0.1
standby 2 priority 105
standby 2 preempt

interface vlan 20
ip address 10.2.0.2 255.255.255.0
standby version 2
standby 3 ip 10.2.0.1

interface vlan 40
ip address 10.6.0.2 255.255.255.0
standby version 2
standby 4 ip 10.6.0.1
```

سوئیچ DSW-A2 برای گروه‌ های **4** و **3** به عنوان **Active** و برای گروه‌ های **1** و **2** به عنوان **Standby** عمل خواهد کرد.
**DSW-A2**
```
interface vlan 99
ip address 10.0.0.3 255.255.255.240
standby version 2
standby 1 ip 10.0.0.1

interface vlan 10
ip address 10.1.0.3 255.255.255.0
standby version 2
standby 2 ip 10.1.0.1

interface vlan 20
ip address 10.2.0.3 255.255.255.0
standby version 2
standby 3 ip 10.2.0.1
standby 3 priority 105
standby 3 preempt

interface vlan 40
ip address 10.6.0.3 255.255.255.0
standby version 2
standby 4 ip 10.6.0.1
standby 4 priority 105
standby 4 preempt
```

برای اطمینان از وضعیت عملکرد**HSRP**
```
do show standby brief
```


---

سوئیچ **DSW-B1** برای گروه‌ های **1** و **2** به عنوان **Primary** و برای گروه‌ های **3** و **4** به عنوان **Standby** عمل خواهد کرد.
- **Group 1** - **Management (VLAN 99)**
- **Group 2** - **PCs (VLAN 10)** 
- **Group 3** - **Phones (VLAN 20)** 
- **Group 4** - **Servers (VLAN 30)**

**DSW-B1**
```
interface vlan 99
ip address 10.0.0.18 255.255.255.240
standby version 2
standby 1 ip 10.0.0.17
standby 1 priority 105
standby 1 preempt

interface vlan 10
ip address 10.3.0.2 255.255.255.0
standby version 2
standby 2 ip 10.3.0.1
standby 2 priority 105
standby 2 preempt

interface vlan 20
ip address 10.4.0.2 255.255.255.0
standby version 2
standby 3 ip 10.4.0.1

interface vlan 30
ip address 10.5.0.2 255.255.255.0
standby version 2
standby 4 ip 10.5.0.1
```

سوئیچ DSW-B2 برای گروه‌ های **4** و **3** به عنوان **Active** و برای گروه‌ های **1** و **2** به عنوان **Standby** عمل خواهد کرد.

**DSW-B2**
```
interface vlan 99
ip address 10.0.0.19 255.255.255.240
standby version 2
standby 1 ip 10.0.0.17

interface vlan 10
ip address 10.3.0.3 255.255.255.0
standby version 2
standby 2 ip 10.3.0.1


interface vlan 20
ip address 10.4.0.3 255.255.255.0
standby version 2
standby 3 ip 10.4.0.1
standby 3 priority 105
standby 3 preempt

interface vlan 30
ip address 10.5.0.3 255.255.255.0
standby version 2
standby 4 ip 10.5.0.1
standby 4 priority 105
standby 4 preempt
```

برای اطمینان از وضعیت عملکرد**HSRP**
```
do show standby brief
```

---
### Rapid PVST+
به صورت پیشفرض، سوئیچ ها در Packet Tracer از **+PVST معمولی** استفاده می‌کنند، نه نسخه سریعتر آن یعنی **+Rapid PVST**

**DSW-A1 , A2 , B1 , B2:**
```
spanning-tree mode rapid-pvst
```

این کار باعث می‌شود ترافیک در هر دو لینک بین سوئیچ‌ های Distribution و Core توزیع شود و از هر دو مسیر استفاده گردد.

اولویت (Priority) در STP بر مضرب‌ های 4096 تنظیم می‌شود؛ بنابراین مقدار بعد از پیشفرض (0) برابر با 4096 خواهد بود.

- سوئیچ **DSW-A1** به عنوان **Root Bridge** برای **VLAN 10 (PCs)** و **VLAN 99 (Management)** تنظیم می کنیم.
- سوئیچ **DSW-A2** به عنوان **Root Bridge** برای **VLAN 20 (Phones)** و **VLAN 40 (Wi-Fi)** تنظیم می کنیم.

**DSW-A1:**
```
spanning-tree vlan 10,99 priority 0
spanning-tree vlan 20,40 priority 4096
```
**DSW-A2:**
```
spanning-tree vlan 20,40 priority 0
spanning-tree vlan 10,99 priority 4096
```

- سوئیچ **DSW-B1** به عنوان **Root Bridge** برای **VLAN 10 (PCs)** و **VLAN 99 (Management)** تنظیم می کنیم.
- سوئیچ **DSW-B2** به عنوان **Root Bridge** برای **VLAN 20 (Phones)** و **VLAN 30 (Servers)** تنظیم می کنیم.

**DSW-B1:**
```
spanning-tree vlan 10,99 priority 0
spanning-tree vlan 20,30 priority 4096
```
**DSW-B2:**
```
spanning-tree vlan 20,30 priority 0
spanning-tree vlan 10,99 priority 4096
```

بررسی وضعیت Spanning Tree :
```
do show spanning-tree
```

---
---
### OSPF
ابتدا فرآیند OSPF ایجاد می شود.
**DSW-A1:**
```
router ospf 1
router-id 10.0.0.79
```
شبکه‌ های متصل به این سوئیچ در Area 0 معرفی می شوند.
```

network 10.0.0.46 0.0.0.0 area 0
network 10.0.0.62 0.0.0.0 area 0
network 10.0.0.79 0.0.0.0 area 0
network 10.1.0.2 0.0.0.0 area 0
network 10.2.0.2 0.0.0.0 area 0
network 10.0.0.2 0.0.0.0 area 0
network 10.6.0.2 0.0.0.0 area 0
```
برای جلوگیری از ارسال بسته‌ های Hello روی VLANها، اینترفیس‌ های مربوطه در حالت Passive قرار گیرند.
```
passive-interface loopback0
passive-interface vlan 10
passive-interface vlan 20
passive-interface vlan 40
```
نوع شبکه OSPF به **Point-to-Point** تغییر داده شود.
```
interface range g1/1/1-2
ip ospf network point-to-point
```

**DSW-A2:**
```
router ospf 1
router-id 10.0.0.80
passive-interface loopback0

passive-interface vlan 10
passive-interface vlan 20
passive-interface vlan 40

network 10.0.0.50 0.0.0.0 area 0
network 10.0.0.66 0.0.0.0 area 0
network 10.0.0.80 0.0.0.0 area 0
network 10.1.0.3 0.0.0.0 area 0
network 10.2.0.3 0.0.0.0 area 0
network 10.0.0.3 0.0.0.0 area 0
network 10.6.0.3 0.0.0.0 area 0

interface range g1/1/1-2
ip ospf network point-to-point
```
بررسی وضعیت همسایه‌ های OSPF
```
do show ip ospf neighbor
```

**DSW-B1:**
```
router ospf 1
router-id 10.0.0.81
passive-interface loopback0

passive-interface vlan 10
passive-interface vlan 20
passive-interface vlan 30

network 10.0.0.54 0.0.0.0 area 0
network 10.0.0.70 0.0.0.0 area 0
network 10.0.0.81 0.0.0.0 area 0
network 10.3.0.2 0.0.0.0 area 0
network 10.4.0.2 0.0.0.0 area 0
network 10.5.0.2 0.0.0.0 area 0
network 10.0.0.18 0.0.0.0 area 0

interface range g1/1/1-2
ip ospf network point-to-point
```

**DSW-B2:**
```
router ospf 1
router-id 10.0.0.82

passive-interface loopback0
passive-interface vlan 10
passive-interface vlan 20
passive-interface vlan 30

network 10.0.0.58 0.0.0.0 area 0
network 10.0.0.74 0.0.0.0 area 0
network 10.0.0.82 0.0.0.0 area 0
network 10.3.0.3 0.0.0.0 area 0
network 10.4.0.3 0.0.0.0 area 0
network 10.5.0.3 0.0.0.0 area 0
network 10.0.0.19 0.0.0.0 area 0

interface range g1/1/1-2
ip ospf network point-to-point
```

---
---
### DHCP

**DSW-A1 , A2:**
```
interface vlan 10
ip helper-address 10.0.0.76

interface vlan 20
ip helper-address 10.0.0.76

interface vlan 40
ip helper-address 10.0.0.76

interface vlan 99
ip helper-address 10.0.0.76
```

**DSW-B1 , B2:**
```
interface vlan 10
ip helper-address 10.0.0.76

interface vlan 20
ip helper-address 10.0.0.76

interface vlan 30
ip helper-address 10.0.0.76

interface vlan 99
ip helper-address 10.0.0.76
```

---
### DNS Server
**DSW-A1 , A2 , B1 , B2:**
```
ip domain name finalpnumain.com
ip name-server 10.5.0.4
```
---
---
### NTP
برای همگام سازی زمان تمامی تجهیزات، سرور NTP معرفی می‌شود.
**DSW-A1 , A2 , B1 , B2:**
```
ntp authentication-key 1 md5 finalpnumain
ntp trusted-key 1
ntp server 10.0.0.76 key 1
```

بررسی وضعیت NTP:
```
do show ntp status
```
---
### SNMP
برای مدیریت و مانیتورینگ تجهیزات از طریق نرم‌افزار های مدیریتی، SNMP پیکربندی می‌شود.

**DSW-A1 , A2 , B1 , B2:**
```
snmp-server community FINALPNUMAIN RO
```
---
### Syslog
به منظور ثبت و نگهداری رخدادهای شبکه، پیام‌ های Syslog به سرور ارسال می‌شوند.
**DSW-A1 , A2 , B1 , B2:**
```
logging 10.5.0.4
logging trap debugging
logging buffered 8192
```
---
