
Tags: **Forensics**, **Medium**

### Challenge
Pandora received an email with a link claiming to have information about the location of the relic and attached ancient city maps, but something seems off about it. Could it be rivals trying to send her off on a distraction? Or worse, could they be trying to hack her systems to get what she knows? Investigate the given attachment and figure out what's going on and get the flag. The link is to http://relicmaps.htb:/relicmaps.one. The document is still live (relicmaps.htb should resolve to your docker instance).


### Downloadable Files
[zip](files/relicmaps)

### Solution

#### downloading the attachment
Another great challenge!? We start by hitting that provided endpoint to get us the attachment file(present in the zip attachment as `relicmaps.one`). You could use something like `wget`, `curl`, `burpsuite` etc. Just make sure to set the `Hosts` header to `relicmaps.htb` explicitly.
```
wget --header "HOST: relicmaps.htb" http://<IP>:<PORT>/relicmaps.one
```
![1](https://user-images.githubusercontent.com/74212182/227627433-965dce6b-5511-4233-be8b-a9e926d4a96f.png)

#### analyzing relicmaps.one
When you run `file` on that attachment, you get the response that it's just data, **not exactly RAW**, so there might be something readable in it!? 
Whenever I see this, the second thing I try to do is run `strings` to see if there's any ascii info, and to my luck, there was valuable stuff. 
```
strings -d relicmaps.one
```
![2](https://user-images.githubusercontent.com/74212182/227627488-0f3af2a8-7a89-4b96-803f-50976f6cdfe8.png)

There's a `HTML` webpage inside, running a `vbscript`, which if you look at it, you see the `AutoOpen()` section, which, intuitively, should tell you that the subsequent commands in it will be run when you click on some action defined in that script. I don't know what that action is, but I can see that the maps  will be downloaded, and a some other `windows.bat` file. Now, if you look at the challenge description, the maps file was supposed to be downloaded, but the `windows.bat`? Not really, something  smells **malicious**!😏

### obtaining the windows.bat
Since the full path is provided there, and it seems to be using our hostname (`relicmaps.htb`), we make an attempt to get that file, through our method earlier
```
wget --header "HOST: relicmaps.htb" http://<IP>:<PORT>/get/DdAbds/window.bat
```
![3](https://user-images.githubusercontent.com/74212182/227627495-8f65076f-2086-4d68-a413-3f12ef745dc4.png)

### analyzing windows.bat
A DOS batch file, and contains `ascii` text, apparently with very long lines, and uses CRLF. So, let's have a look at its contents, using `cat`
From the start, you could see it's obfuscated, and just at the top, we see how the gibberish is made using `set` (which also obfuscates itself😑).
![4](https://user-images.githubusercontent.com/74212182/227627499-a543421c-8f0a-4e80-8321-37b3384e3cdf.png)

Now, analyzing the format of all the lines;
- obfuscation is done on the left, and the actual readable command is on the right
- we are invoking set on each line, to assign the `commands` on the right, to the gibberish on the left
- after the `set` statements, we are having some long encoded statements, which possibly are using the previously set gibberish
- there's a comment, a very long line, which is visible and starts with `::` (.bat syntax)

Hint: With syntax highlighting, the file is more understandable, and therefore I would recommend `visual studio` or `sublime-text`

#### deobfuscation
The program is clearly doing something, and to figure that out, we need to edit out those `set` statements on the file, map the gibberish on the longer statements, and produce some readable code. I decided to split the long statements and short `set` statements in to two files. Short `set` statements were placed individually on their own file, called `short.bat`, and then I made a python script that contained the long encoded commands, found at the bottom of the batch file.
![5](https://user-images.githubusercontent.com/74212182/227627517-18320957-3dd4-49b5-a5e5-ca42f6c735a8.png)

Running that script revealed the actual batch statements, and after a few edits and reformatting, I was able to get the deobfuscated version of the windows batch file.
![6](https://user-images.githubusercontent.com/74212182/227627523-c2a355f7-fd4e-4ac8-afb6-543001e9df73.png)
![7](https://user-images.githubusercontent.com/74212182/227627526-9887e4de-19c7-464f-b0ed-22fdc5f07bd9.png)

#### analyzing windows.bat again
The .bat file is readable now, and we can see what is happening:
- the program is loading the long  `comment` at the start and removing the first three commenting characters `":: "` (via the substring command) to create a new string
- it's then initializing an `AES CBC` object, with padding set as `PKCS7`, key and IV provided explicitly as base64, and performing a decryption on the string
- a stream object is created from our decrypted string 
- then a Gzip stream compression object is created, that performs a decompression process on our stream
- the stream is then loaded into an assembly object as an array, that produces a PE from that file and finally gets invoked

#### producing the PE
There are many ways to do that; and one most obvious route is to edit the .bat file, such that the end instruction is to create the PE file on our current directory. But this `.bat`, who writes windows batch script nowadays? Well you could, but I think cyberchef here was most convenient for me, and I was able to obtain the flag by looking at the file in little endian.
![8](https://user-images.githubusercontent.com/74212182/227627530-1118058e-2e84-40ed-b099-f09c57358fe2.png)
