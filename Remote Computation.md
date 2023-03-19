
Tags: **Miscellaneous**, **Easy**

### Challenge
The alien species use remote machines for all their computation needs. Pandora managed to hack into one, but broke its functionality in the process. Incoming computation requests need to be calculated and answered rapidly, in order to not alarm the aliens and ultimately pivot to other parts of their network. Not all requests are valid though, and appropriate error messages need to be sent depending on the type of error. Can you buy us some time by correctly responding to the next 500 requests?


### Downloadable Files
`none`

### Solution

python script:
```python

# Usage: python script.py <IP>:<PORT>

from pwn import *
import sys

# context.log_level = 'debug'

try:
        host,port = sys.argv[1].split(":")

except:
        print("provide host stuff bro!")
        exit(0)

print(host,port)


conn = remote(host, port)

conn.recvuntil(b"> ")

conn.sendline(b"1")

while (True):
        try:
                data = conn.recvuntil(b"> ")
        except:
                break

        eq = data.split(b"]: ")[1].split(b" = ?")[0]

        try:
                ans = round(eval(eq),2)
                if ans < -1337 or ans > 1337:
                        conn.sendline(b"MEM_ERR")
                else:
                        conn.sendline(str(ans).encode())
        except ZeroDivisionError:
                conn.sendline(b"DIV0_ERR")
        except SyntaxError:
                conn.sendline(b"SYNTAX_ERR")

print(conn.recvall())

conn.close()

```

### Screenshots

![[Pasted image 20230319164143.png]]

![[Pasted image 20230319164009.png]]

![[Pasted image 20230319165409.png]]
