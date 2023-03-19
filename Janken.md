
Tags: **Miscellaneous**, **Easy**

### Challenge
As you approach an ancient tomb, you're met with a wise guru who guards its entrance. In order to proceed, he challenges you to a game of Janken, a variation of rock paper scissors with a unique twist. But there's a catch: you must win 100 rounds in a row to pass. Fail to do so, and you'll be denied entry.

### Downloadable Files
[link]

### Solution

When you look through the binary on `ghidra`, you'll realize the game does a string comparison with `strstr`, which takes two strings, **s1** and **s2** as arguments and finds the first occurrence of the substring **s2** in the string **s1**. If we pass all the required viable strings as a combined payload, i.e. `rockscissorspaper`,  then the program will produce a win for us everytime.


test script for local binary:
```python
from pwn import *
import sys

# context.log_level = 'debug'

conn = process('./janken')

conn.recvuntil(b">> ")
conn.sendline(b"1")

while (True):
	try: 
		data = conn.recvuntil(b">> ")
	except:
		break
		conn.sendline(b"rockscissorspaper")

print(conn.recvall().decode())
conn.close()
```

final script for remote instance:
```python
# Usage: python script.py <IP>:<PORT>

from pwn import *
import sys

try:
	host,port = sys.argv[1].split(":")

except:
	print("provide host stuff bro!")
	exit(0)

conn = remote(host, port)

conn.recvuntil(b">> ")
conn.sendline(b"1")

while (True):
	try:
		data = conn.recvuntil(b">> ")
	except:
		break
	conn.sendline(b"rockscissorspaper")

print(conn.recvall().decode())
conn.close()
```

### Screenshots

