# Pwn_review writeup (150)

---

Info:

> Name: pwn_review
> 
> Category: Pwn
> 
> Access: nc ctf-league.osusec.org 31304
> 
> Desc: None

Attachments:

- pwn_review

---

In this challenge, we're given a binary, if we run it locally we get the following output:

$ ./pwn_review

> This is a review challenge, you know the drill
> Return to the win function and get the flag

Alright, well we have our mission, so let's check it out in ghidra:

![](C:\Users\rudyp\Pictures\WriteupSCs\pt_01.png)

Alright, sweet, we can check out the symbol tree and we see the described **win** function that we're trying to get to:

![](C:\Users\rudyp\Pictures\WriteupSCs\pt_02.png)

![](C:\Users\rudyp\Pictures\WriteupSCs\pt_03.png)

Alright, so based on all of this information, our objective is to buffer overflow our **part2** function and force the program to jump to the **win** function.

However, that's only the first part of our challenge, the second would be in the actual win function. After we get there the program wants us to give it some shellcode to run. So we're going to have to make that too.

I decided to start by making my buffer overflow.

---

## Part 1: Buffer Overflow

So the buffer we're given is only **32** bytes long, but we're allowed to write up to 100 bytes of data to that, if we send a bunch of A's we'd definitely overwrite everything, but we're not trying to break everything, we need to hijack the return.

First we need to find the offset of the return, so let's check out the assembly:

![](C:\Users\rudyp\Pictures\WriteupSCs\pt_04.png)

![](C:\Users\rudyp\Pictures\WriteupSCs\pt_05.png)

Ay, thanks Binja and GEF! Alright, so we can see that the program fills the userBuffer with standard input, then makes a reference to our string buffer with **RDI**. 

The arguments that we pass to fgets come from our string reference (**RDI**), the total size we can write to that buffer (**RSI**), and our stream pointer (**RDX**) .

So we can overload **RDI** with anything over **32 bytes** of input. If we send 32 A's, here's what the stack looks like.

![](C:\Users\rudyp\Pictures\WriteupSCs\pt_06.png)

So, if we use python to send 32 A's over into the program, we can see that we have access to the address **0x18**, the start of the base pointer **RBP**, is at **0x20** (32), so therefore, 8 bytes after that should be where we can screw up the return. So let's try and send 48 A's to the program:

![](C:\Users\rudyp\Pictures\WriteupSCs\pt_07.png)

And viola!! We overloaded the stack pointer, cool!

So now we can theoretically just get the location of the **win** function, jump to that, and then start working on shellcode! Let's grab the location of **win**:

![](C:\Users\rudyp\Pictures\WriteupSCs\pt_08.png)

Alright cool! Let's make a python script for that:

![](C:\Users\rudyp\Pictures\WriteupSCs\pt_09.png)

After executing that we get the **win** function!!!

![](C:\Users\rudyp\Pictures\WriteupSCs\pt_10.png)

Alright! Time to write some shellcode!

---

## Part 2: Shellcode

![](C:\Users\rudyp\Pictures\WriteupSCs\pt_11.png)

Okay, so all we need to do is send some shellcode. There's only one problem, I didn't know how to do it. So after a LOT of research I learned that there were a lot of approaches we can take with this, but the quickest is by writing an assembly program that executes the code that we want to run.

I'll save you the boring research, but essentially we want to run **execve("/bin/sh", 0, 0)**, and we'll be in! Technically we can write this all using pwntools, but I didn't want to rely on pwntools, so I made the assembly based off of the assembly template that we have on the OSUSEC shellcode github. Except for one change, I wanted to make my shellcode using intel flavor instead of AT&T assembly. Which is what I did:

![](C:\Users\rudyp\Pictures\WriteupSCs\pt_12.png)

The cliffsnotes is I just wanted to run **execve("//bin/sh", 0, 0)**, so I just wrote assembly to do that.

Next I needed the opcodes of this program, to which I used these commands:

![](C:\Users\rudyp\Pictures\WriteupSCs\pt_13.png)

And with that I got my shellcode! Let's write that into our exploit script now!

![](C:\Users\rudyp\Pictures\WriteupSCs\pt_14.png)

AYYYYYYYYYYYYYYYYYY

![](C:\Users\rudyp\Pictures\WriteupSCs\pt_15.png)

Now let's change the process to connect to the server and test out our exploit

![](C:\Users\rudyp\Pictures\WriteupSCs\pt_16.png)

AND IT WORKS!!!

![](C:\Users\rudyp\Pictures\WriteupSCs\pt_17.png)
