```
enable
conf t
hostname R1,ASW
do write
```

```
enable secret m0b1n
username mobin secret @m0b1n
```

```
line console 0
login local
exec-timeout 30
logging synchronous
```

---
اینترفیس‌ های **G0/0/0** و **G0/1/0** به اینترنت متصل هستند و برای دریافت آدرس IP از **DHCP** استفاده خواهند کرد.
```
interface range g0/0/0,g0/1/0
ip address dhcp
no shutdown
```
اینترفیس **G0/0**، متصل به **CSW1**
```
interface g0/0
ip address 10.0.0.33 255.255.255.252
no shutdown
```
اینترفیس **G0/1**، متصل به **CSW2**
```
interface g0/1
ip address 10.0.0.37 255.255.255.252
no shutdown
```
اینترفیس loopback را ایجاد کرده
```
interface loopback0
ip address 10.0.0.76 255.255.255.255
exit
do write
```

بررسی وضعیت اینترفیس های R1
```
do show ip interface brief
```
---
---
### OSPF
بررسی Router ID فعلی :
```
do show ip ospf
```
میبینیم که IP روتر **10.0.0.76** است که همان IP اینترفیس **Loopback** است
```
router ospf 1
router-id 10.0.0.76  
passive-interface loopback0
```
فعال کردن **OSPF** روی **loopback0**
```
interface loopback0
ip ospf 1 area 0
```
فعال کردن **OSPF** روی دو اینترفیس **LAN** روتر **R1** که به **CSW1** و **CSW2** متصل هستند
پیکربندی نوع شبکه به صورت **Point-to-Point**
```
interface range g0/0-1
ip ospf 1 area 0
ip ospf network point-to-point
```

---
روتر R1 به دو لینک اینترنت متصل است.
برای اینکه لینک دوم تنها در صورت در دسترس نبودن لینک اصلی مورد استفاده قرار گیرد، مقدار Administrative Distance مسیر دوم را از مقدار پیش‌ فرض بیشتر در نظر می‌ گیریم.

در Cisco IOS، مقدار پیش‌ فرض Administrative Distance برای Static Route برابر با 1 است؛
پس، برای مسیر دوم مقدار 2 را تنظیم می‌ کنیم تا این مسیر تنها به عنوان مسیر پشتیبان (Floating Static Route) مورد استفاده قرار گیرد.
```
ip route 0.0.0.0 0.0.0.0 203.0.113.1
ip route 0.0.0.0 0.0.0.0 203.0.113.5 2
```
برای انتشار مسیر پیش‌ فرض (Default Route) در OSPF، روتر R1 را به‌ عنوان یک Autonomous System Boundary Router (ASBR) پیکربندی می‌ کنیم.
```
router ospf 1
default-information originate
exit
do write
```

---
---
### DHCP
ابتدا DHCP Pool ها را روی روتر R1 پیکربندی می‌ کنیم تا این روتر به‌ عنوان DHCP Server برای کلاینت های Office A و Office B عمل کند.

```
ip dhcp pool A-Mgmt
network 10.0.0.0 255.255.255.240
default-router 10.0.0.1
domain-name finalpnumain.com
dns-server 10.5.0.4
option 43  ip 10.0.0.7
```

```
ip dhcp pool A-PCs
network 10.1.0.0 255.255.255.0
default-router 10.1.0.1
dns-server 10.5.0.4
domain-name finalpnumain.com
```

```
ip dhcp pool A-Phones
network 10.2.0.0 255.255.255.0
default-router 10.2.0.1
dns-server 10.5.0.4
domain-name finalpnumain.com
```

```
ip dhcp pool B-Mgmt
network 10.0.0.16 255.255.255.240
default-router 10.0.0.17
dns-server 10.5.0.4
domain-name finalpnumain.com
```

```
ip dhcp pool B-PCs
network 10.3.0.0 255.255.255.0
default-router 10.3.0.1
dns-server 10.5.0.4
domain-name finalpnumain.com
```

```
ip dhcp pool B-Phones
network 10.4.0.0 255.255.255.0
default-router 10.4.0.1
dns-server 10.5.0.4
domain-name finalpnumain.com
```

```
ip dhcp pool Wi-Fi
network 10.6.0.0 255.255.255.0
default-router 10.6.0.1
dns-server 10.5.0.4
domain-name finalpnumain.com
```
همچنین، برای جلوگیری از تخصیص آدرس‌ های رزرو شده، ۱۰ آدرس قابل استفاده اول هر Pool را از محدوده تخصیص DHCP مستثنی (Exclude) می‌ کنیم.
```
ip dhcp excluded-address 10.0.0.1 10.0.0.10
ip dhcp excluded-address 10.1.0.1 10.1.0.10
ip dhcp excluded-address 10.2.0.1 10.2.0.10

ip dhcp excluded-address 10.0.0.17 10.0.0.26

ip dhcp excluded-address 10.3.0.1 10.3.0.10
ip dhcp excluded-address 10.4.0.1 10.4.0.10
ip dhcp excluded-address 10.6.0.1 10.6.0.10
```

---
---
### NTP
برای همگام سازی زمان تمامی تجهیزات، سرور NTP معرفی می‌شود.
```
ntp master 5
ntp server 216.239.35.0
```

```
ntp authentication-key 1 md5 finalpnumain
ntp trusted-key 1
```
---
### SNMP
برای مدیریت و مانیتورینگ تجهیزات از طریق نرم‌افزار های مدیریتی، SNMP پیکربندی می‌شود.
```
snmp-server community FINALPNUMAIN RO
```

---
### Syslog
به منظور ثبت و نگهداری رخدادهای شبکه، پیام‌ های Syslog به سرور ارسال می‌شوند.
```
logging 10.5.0.4
logging trap debugging
logging buffered 8192
```
---
