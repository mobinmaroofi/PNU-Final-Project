**CSW1,2:**

نام هر سوئیچ مطابق توپولوژی پروژه :
```
enable
conf t
hostname CSW1
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
### فعال سازی قابلیت مسیریابی لایه 3
قابلیت مسیریابی IP را فعال می کنیم
**CSW1 ,CSW2**

```
ip routing
```
بررسی فعال بودن قابلیت مسیریابی:
```
do show ip route
```
---
برای برقراری یک EtherChannel لایه ۳ بین سوئیچ‌ های CSW1 و CSW2، از پروتکل اختصاصی سیسکو (PAgP) استفاده میکنیم. با توجه به اینکه هر دو سوئیچ باید به صورت فعال فرآیند تشکیل EtherChannel را آغاز کنند، از حالت **desirable** روی هر دو سوئیچ استفاده میکنیم.
**CSW1 :**
```
do show cdp neighbors
```

اینترفیس‌ های **G1/0/2** و **G1/0/3** روی سوئیچ **CSW2** به سوئیچ **CSW1** متصل هستند. این اینترفیس‌ ها را با استفاده از دستور `no switchport` به پورت‌ های Routed تبدیل میکنیم.

**CSW1 :**
```
interface range g1/0/2-3
no switchport
channel-group 1 mode desirable
```
از آنجا که این EtherChannel از نوع لایه ۳ (Layer 3) است، آدرس IP باید روی اینترفیس `Port-channel1` پیکربندی شود
```
interface port-channel1
ip address 10.0.0.41 255.255.255.252
```
**CSW2 :**
```
interface range g1/0/2-3
no switchport
channel-group 1 mode desirable
```

```
interface port-channel1
ip address 10.0.0.42 255.255.255.252
```
بررسی وضعیت EtherChannel :
```
do show etherchannel summary
```
---
روی هر یک از اینترفیس‌ های فیزیکی، ابتدا با استفاده از دستور `no switchport` آن‌ ها را بهRouted Port تبدیل کرده و سپس آدرس IP را برای هر اینترفیس پیکربندی میکنیم.
**CSW1**
```
interface g1/0/1
no switchport
ip address 10.0.0.34 255.255.255.252

interface g1/1/1
no switchport
ip address 10.0.0.45 255.255.255.252

interface g1/1/2
no switchport
ip address 10.0.0.49 255.255.255.252

interface g1/1/3
no switchport
ip address 10.0.0.53 255.255.255.252

interface g1/1/4
no switchport
ip address 10.0.0.57 255.255.255.252
```
در ادامه، اینترفیس **Loopback0** را پیکربندی کرده و اینترفیس‌ های بلا استفاده `G1/0/4` تا `G1/0/24` را غیرفعال (Shutdown) میکنیم.
```
interface loopback0
ip address 10.0.0.77 255.255.255.255

interface range g1/0/4-24
shutdown
```

**CSW2**
```
interface g1/0/1
no switchport
ip address 10.0.0.38 255.255.255.252

interface g1/1/1
no switchport
ip address 10.0.0.61 255.255.255.252

interface g1/1/2
no switchport
ip address 10.0.0.65 255.255.255.252

interface g1/1/3
no switchport
ip address 10.0.0.69 255.255.255.252

interface g1/1/4
no switchport
ip address 10.0.0.73 255.255.255.252
```

```
interface loopback0
ip address 10.0.0.78 255.255.255.255

interface range g1/0/4-24
shutdown
```
---
---
### OSPF
بررسی وضعیت اینترفیس‌ های دارای آدرس IP
**CSW1**
```
do show ip interface brief | exclude unassigned
```

```
router ospf 1
router-id 10.0.0.77
passive-interface loopback0
```

```
network 10.0.0.41 0.0.0.0 area 0
network 10.0.0.34 0.0.0.0 area 0
network 10.0.0.45 0.0.0.0 area 0
network 10.0.0.49 0.0.0.0 area 0
network 10.0.0.53 0.0.0.0 area 0
network 10.0.0.57 0.0.0.0 area 0
network 10.0.0.77 0.0.0.0 area 0
```

```
interface range g1/0/1,g1/1/1-4
ip ospf network point-to-point
```
بررسی وضعیت اینترفیس‌ های دارای آدرس IP
**CSW2**
```
do show ip interface brief | exclude unassigned
```

```
router ospf 1
router-id 10.0.0.78
passive-interface loopback0
```

```
network 10.0.0.42 0.0.0.0 area 0
network 10.0.0.38 0.0.0.0 area 0
network 10.0.0.61 0.0.0.0 area 0
network 10.0.0.65 0.0.0.0 area 0
network 10.0.0.69 0.0.0.0 area 0
network 10.0.0.73 0.0.0.0 area 0
network 10.0.0.78 0.0.0.0 area 0
```

```
interface range g1/0/1,g1/1/1-4
ip ospf network point-to-point
```
---
### DNS Server
**CSW1 ,CSW2**
```
ip domain name finalpnumain.com
ip name-server 10.5.0.4
```

---
### NTP
برای همگام سازی زمان تمامی تجهیزات، سرور NTP معرفی می‌شود.
**CSW1 ,CSW2**
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
**CSW1 ,CSW2**
```
snmp-server community FINALPNUMAIN RO
```
---
### Syslog
به منظور ثبت و نگهداری رخدادهای شبکه، پیام‌ های Syslog به سرور ارسال می‌شوند.
**CSW1 ,CSW2**
```
logging 10.5.0.4
logging trap debugging
logging buffered 8192
```
---
