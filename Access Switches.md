```
enable
conf t
hostname ASW-A1
do write
```

```
enable secret finalpnumain
username mobin secret @finalpnumain
```

```
line console 0
login local
exec-timeout 30
logging synchronous
```
---
---

### EtherChannel
**ASW-A1 ,A2 ,A3:**
ارتباط این سوئیچ ها با سوئیچ‌ های Distribution از طریق اینترفیس‌ های `G0/1` و `G0/2` برقرار می‌شود.
```
interface range g0/1-2
switchport mode trunk
switchport nonegotiate
switchport trunk native vlan 1000
switchport trunk allowed vlan 10,20,40,99
```

**ASW-B1 ,B2 ,B3:**
```
interface range g0/1-2
switchport mode trunk
switchport nonegotiate
switchport trunk native vlan 1000
switchport trunk allowed vlan 10,20,30,99
```

---
### VTP
سوئیچ‌ های Access را به عنوان **VTP Client** پیکربندی می کنیم
**ASW-A1 ,A2 ,A3:**
```
vtp mode client
```

پیام‌ های **VTP** مربوط به **Branch A** به **Branch B** نمیرسند.
این پیام ها فقط از طریق پورت‌ های **Trunk** ارسال می‌شوند و ما این لینک‌ ها را بین سوئیچ‌ های Distribution و Core به صورت **Trunk** پیکربندی نکرده ایم؛ در حال حاضر همه آن‌ ها در حالت **Access** هستند. بعداً این لینک‌ ها را به لینک‌ های **Layer-3** تبدیل خواهیم کرد. بنابراین، باید **VTP** را در **Office B** نیز فعال کنیم.

**ASW-B1 , B2 , B3 :**
```
vtp mode client
```
---
بررسی موفقیت آمیز بودن همگام سازی پایگاه داده **VLAN** ها :
**ASW-A3 , B3 :** 
```
do show vlan brief
```
--- 
### Access ports
سوئیچ‌ های ASW-A1,B1 هر دو به Lightweight AP (LWAP) متصل هستند.
از آنجا که LWAPها از قابلیت Flex Connect استفاده نمی‌کنند، تمام ترافیک وایرلس کاربران از طریق Management VLAN (VLAN 99) به سمت WLC1 تونل خواهد شد.
پورت‌ های متصل به LWAPها میتوانند به‌ صورت Access Port پیکربندی شوند؛ زیرا تنها نیاز است Management VLAN را پشتیبانی کنند.
همچنین به همین دلیل، در Office B نیازی به ایجاد Wi-Fi VLAN وجود ندارد؛ زیرا ترافیک کاربران بی‌سیم این شعبه ابتدا به WLC1 در Office A تونل شده و پس از رسیدن به WLC، به Wi-Fi VLAN (VLAN 40) اختصاص داده می‌شود.
**ASW-A1 , B1 :**
```
interface f0/1
switchport mode access
switchport nonegotiate
switchport access vlan 99
exit
```
 در نتیجه اجرای دستور **`switchport mode access`** قابلیت **DTP** غیرفعال می‌شود؛ پس پیام‌ های **DTP** را ارسال یا دریافت نخواهد کرد.
 بنابراین با قرار دادن پورت در حالت **Access**، ارسال و دریافت پیام‌ های DTP به‌ طور خودکار متوقف می‌شود.
**ASW-A2, A3, B2 :**
```
interface f0/1
switchport mode access
switchport nonegotiate
switchport access vlan 10
switchport voice vlan 20
exit
```
در مرحله آخر، اتصال ASW-B3 به SRV1 را در VLAN 30 (Servers VLAN) تنظیم می‌ کنیم.
**ASW-B3 :** 
```
interface f0/1
switchport mode access
switchport nonegotiate
switchport access vlan 30
exit
```

---
### اتصال Trunk بین ASW-A1 و WLC1
اتصال بین ASW-A1 و WLC1 را پیکربندی میکنیم. این اتصال باید از Wi-Fi VLAN و Management VLAN را پشتیبانی کند.
اتصال WLC1 از طریق اینترفیس `F0/2` به سوئیچ برقرار می‌شود
**ASW-A1 :**
```
interface f0/2
switchport mode trunk
switchport trunk allowed vlan 40,99
switchport trunk native vlan 99
switchport nonegotiate
exit
```
---
### غیرفعال کردن پورت‌ های بلا استفاده
**ASW-A1 :**
```
do show interface status 
```

**ASW-A1 :**
```
interface range f0/3-24
shutdown
exit
do write
```
**ASW-A2,A3,B1,B2,B3 :**
```
interface range f0/2-24
shutdown
exit
do write
```
---
---
این سوئیچ‌ های Access از نوع لایه 2 هستند، اما برای اینکه بتوانیم از راه دور و از طریق SSH آن‌ ها را مدیریت و پیکربندی کنیم، باید برای هر یک یک آدرس IP و یک Default Gateway تنظیم میکنیم.

سوئیچ‌ های ASW-A1، ASW-A2 و ASW-A3 دارای آدرس IP در زیرشبکه `10.0.0.0/28` هستند؛ بنابراین، اولین آدرس قابل استفاده این زیرشبکه، یعنی `10.0.0.1`، را به‌ عنوان Default Gateway آن‌ ها پیکربندی میکنیم.

**ASW-A1 :**
```
ip default-gateway 10.0.0.1
interface vlan 99
ip address 10.0.0.4 255.255.255.240
exit
do write
```

**ASW-A2 :**
```
ip default-gateway 10.0.0.1
interface vlan 99
ip address 10.0.0.5 255.255.255.240
exit
do write
```

**ASW-A3 :**
```
ip default-gateway 10.0.0.1
interface vlan 99
ip address 10.0.0.6 255.255.255.240
exit
do write
```
زیرشبکه `10.0.0.16/28` را برای بخش مدیریت (Management) در نظر می‌ گیریم و آدرس‌ های `10.0.0.20`، `10.0.0.21` و `10.0.0.22` را به‌ ترتیب به سوئیچ‌ های ASW-B1، ASW-B2 و ASW-B3 اختصاص می‌ دهیم.

از آنجا که این سوئیچ‌ ها در زیرشبکه `10.0.0.16/28` قرار دارند، اولین آدرس قابل استفاده این زیرشبکه، یعنی `10.0.0.17`، را به‌ عنوان Default Gateway آن‌ ها پیکربندی می‌ کنیم.
**ASW-B1**
```
ip default-gateway 10.0.0.17
interface vlan 99
ip address 10.0.0.20 255.255.255.240
exit
do write
```

**ASW-B2**
```
ip default-gateway 10.0.0.17
interface vlan 99
ip address 10.0.0.21 255.255.255.240
exit
do write
```

**ASW-B3**
```
ip default-gateway 10.0.0.17
interface vlan 99
ip address 10.0.0.22 255.255.255.240
exit
do write
```

---
---
### Rapid PVST+

**ASW-A1 ,A2,A3,B1,B2,B3 :**
```
spanning-tree mode rapid-pvst
```
قابلیت‌ های **PortFast** و **BPDU Guard** را روی تمام پورت‌ هایی که به تجهیزات انتهایی متصل هستند، از جمله **WLC1**، فعال میکنیم.

بنابراین، **PortFast** باعث می‌شود پورت‌ های متصل به تجهیزات انتهایی، بدون اینکه ۳۰ ثانیه منتظر بمانند تا مراحل **Listening** و **Learning** را طی کنند، مستقیماً وارد حالت **STP Forwarding** شوند.
**ASW-A1 :**
اینترفیس **F0/1** به **LWAP1** متصل است.
```
interface f0/1
spanning-tree portfast
spanning-tree bpduguard enable
```
اینترفیس **F0/2** به **WLC1** متصل است.

قابلیت **PortFast** روی **F0/2** پیکربندی شده است، اما تنها زمانی مؤثر خواهد بود که اینترفیس در حالت **غیر Trunk** باشد.
```
interface f0/2
spanning-tree portfast trunk
spanning-tree bpduguard enable
```
**ASW-A2, A3, B1, B2, B3**
```
interface f0/1
spanning-tree portfast
spanning-tree bpduguard enable
```
---
---
### DNS Server
**ASW-A1 ,A2 ,A3 , B1, B2, B3:**
```
ip domain name finalpnumain.com
ip name-server 10.5.0.4
```
---
---
### NTP
برای همگام سازی زمان تمامی تجهیزات، سرور NTP معرفی می‌شود.
**ASW-A1 ,A2 ,A3 , B1, B2, B3:**
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
**ASW-A1 ,A2 ,A3 , B1, B2, B3:**
برای مدیریت و مانیتورینگ تجهیزات از طریق نرم‌افزار های مدیریتی، SNMP پیکربندی می‌شود.
```
snmp-server community FINALPNUMAIN RO
```

---
### Syslog
به منظور ثبت و نگهداری رخدادهای شبکه، پیام‌ های Syslog به سرور ارسال می‌شوند.
**ASW-A1 ,A2 ,A3 , B1, B2, B3:**
```
logging 10.5.0.4
logging trap debugging
logging buffered 8192
```
---