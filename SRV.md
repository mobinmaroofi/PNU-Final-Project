**Default Gateway:** 10.5.0.1  
**IPv4 Address:** 10.5.0.4  
**Subnet Mask:** 255.255.255.0 (/24)

به‌ صورت پیش‌ فرض، Packet Tracer برای آدرس IP `10.5.0.4` یک Subnet Mask با پیشوند `/8` در نظر می‌ گیرد، زیرا این آدرس در محدوده آدرس‌ های Class A قرار دارد. با این حال، در این سناریو از پیشوند `/24` استفاده می‌ کنیم؛ بنابراین Subnet Mask را به `/24` تغییر می‌ دهیم.

---
### DNS
برای ایجاد مکانیزم ترجمه یک Domain Name به یک آدرس IPv4، باید یک رکورد از نوع A Record ایجاد کنیم.
در قسمت Name، مقدار `google.com` را وارد کرده و سپس در قسمت Address، آدرس IP `172.253.62.100` را وارد می‌ کنیم.
دوباره `youtube.com` را وارد کرده و آدرس `152.250.31.93` را در Address قرار می‌ دهیم.

برای ترجمه یک hostname به hostname دیگر از نوع رکورد CNAME استفاده میکنیم.
در قسمت Name، مقدار `www.yahoo.com` را وارد کرده در قسمت Host name `yahoo.com` را وارد می‌ کنیم.

بررسی وضعیت ارتباط با DNS Server از PC1 :
```
PING 10.5.0.4
```
سپس `google.com` را ping میکنیم :
```
PING google.com
```
در پروسه Ping کردن میبینیم که با شکست مواجه میشویم ولی فرایند DNS Resolution به درستی انجام شده. 

---
### DNS Server
```
ip domain name finalpnumain.com
ip name-server 10.5.0.4
```