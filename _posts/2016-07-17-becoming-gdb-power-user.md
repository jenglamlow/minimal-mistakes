---
title: "\"Becoming a GDB Power User!\" Highlights"
categories:
  - Knowledge Sharing
tags:
  - C/C++
  - GDB
---

My first programming language is C, after that picking up C++. Whenever I want to debug C/C++ program, I will use Visual Studio debugger tools. 
However, when debugging for embedded project (ARM), debugging using GUI is not straightforward. Thus, GDB comes in handy
as it is lightweight, fast and offers great features.

For my past embedded debugging experience, I used GDB to perform basic debugging like <code>step</code>, <code>print</code> and <code>b</code>.
Then, I came across with this video "Becoming a GDB Power User!" by Greg Law. I changed my view about GDB, GDB has a lot of cool features 
which I did not take advantages on it. I will share the cool features highlighted in this video. The following are the video conference 
with the [slide](https://www.docdroid.net/QWXfBPJ/greglawundobecomeagdbpoweruser.pdf.html).

<iframe width="560" height="315" src="https://www.youtube.com/embed/713ay4bZUrw" frameborder="0" allowfullscreen></iframe>

## Text User Interface (TUI) Mode
GDB has a "GUI" mode to display current code execution called Text User Interface (TUI) mode. 

To activate TUI mode, open GDB by typing command <code>gdb</code> in the terminal, and <code>Ctrl x-a</code> to toggle between TUI 
window and normal window

TUI window:
[![alt text](http://i.imgur.com/4KmC7Pw.jpg "GDB TUI window")](http://i.imgur.com/4KmC7Pw.jpg)

Wow!!!! If I have know TUI back then, this saves a lot of time, I no need to type <code>l</code> everytime to see where is my current code execution.

Command in TUI window:

```shell
<up> or <down> arrow      # To navigate the code up and down
Ctrl-p / Ctrl-n           # Previous/Next GDB history command 
Ctrl-l                    # Refresh/Redraw windows 
                          # (Code didn't reflect changes properly during stepping)
Ctrl-x-2                  # Open/Cycle thru Disassembly view
Crrl-x-1                  # Exit Disassembly view
```

Disassembly view:
[![alt text](http://i.imgur.com/TQr6VTm.jpg "Disassembly view")](http://i.imgur.com/TQr6VTm.jpg)

## Python Support
I must admit that this is the greatest feature. With Python integration, we can script the debugging process and automate it. You can even create
your own GDB command.

Sample python runnning in GDB:

```shell
(gdb) python
>print("Hello!!!")
>end
Hello!!!

(gdb) python abc="hi"
(gdb) python print(abc)
hi

(gdb) l
10		struct Classroom class_a;
11		int i = 0;
12		printf("Hello, world\n");
13		printf("i is %d\n", i);
14		i++;
15		printf("i is now %d\n", i);
(gdb) n
12		printf("Hello, world\n");
(gdb) n
Hello, world
13		printf("i is %d\n", i);
(gdb) python gdb.execute('next')          #execute gdb "next" command using Python
i is 0
14		i++;
```

GDB introduces a python module, named <code>gdb</code> which import automatically which has provides API to interface with GDB. You can check
out the API at [here](https://sourceware.org/gdb/current/onlinedocs/gdb/Basic-Python.html#Basic-Python) or type <code>python help(gdb)</code>

## Create your own command using Python 
You can create your own command using Python by inheriting <code>gdb.Command</code>

```python
# my_command.py
import gdb

class my_command(gdb.Command):
	def __init__(self):
		gdb.Command.__init__(self, 'my_command', gdb.COMMAND_NONE)
	def invoke(self, args, from_tty):
		print("Voila!!!")

my_command()
```
```shell
(gdb) source my_command.py
(gdb) my_command
Voila!!!
```

## Reverse Debugging

Wow! I never knew that we can step back when debugging. But there is a catch, it will be very slow when you want to enable reverse debugging.
Basically, GDB will store the stepping history details as the code is running, so it can be stepback to trace the variables changes. That is why this
method is slower. It is recommended to use <code>record</code> only the code portion you interested or when the duration of code execution is short

Example C code

```c
#include <stdio.h>

struct Classroom {
   int   teacher;
   int   student;
} ;

int main()
{
        struct Classroom class_a;
        int i = 0;
        printf("Hello, world\n");
        printf("i is %d\n", i);
        i++;
        printf("i is now %d\n", i);     //line 15
        return 0;
}                           
```

```shell
(gdb) start
(gdb) record          # Start recording
(gdb) b 15
Breakpoint 2 at 0x4005b7: file hello.c, line 15.
(gdb) c
Continuing.

Breakpoint 2, main () at hello.c:15
(gdb) p i
$1 = 1                # after i++, i = 1
(gdb) rs              # reverse step
(gdb) p i             
$2 = 0                # before i++, i = 0
(gdb) rc              # reverse continue
Continuing.

No more reverse-execution history.
main () at hello.c:11
11		int i = 0;
```

## Watch

Watch will suspend the program whenever the watch variable changes value or meet certain condition. Thus, you can set the program running until
the variable changes.

```shell
(gdb) watch i
Hardware watchpoint 2: i
(gdb) c
Continuing.
Hello, world
i is 0
Hardware watchpoint 2: i

Old value = 0
New value = 1
main () at hello.c:15
15		printf("i is now %d\n", i);
```

You can set the watch to break when certain condition is method

```shell
watch i if i > 10
```

## Pretty Print

This is great mode to turn on pretty print when running <code>print</code>. It is useful when you are printing large structure data, 
this option will append the structure member nicely in new line instead of concatenate in the same line.

```shell
(gdb) set print pretty off
(gdb) p class_a
$1 = {teacher = -9392, student = 32767}
(gdb) set print pretty on
(gdb) p class_a
$2 = {
  teacher = -9392, 
  student = 32767
}
```

It is recommended to set this setting in <code>.gdbint</code> file

## Dynamic Printing

Remember when we don't use debugger, we normally use <code>printf</code> to output out desired variables at console. The problem with this 
approach is we need to recompile the code whenever we insert the print out code. And sometimes we might forget to remove this debug output.

GDB offers a cool feature, which can dynamically perform printout without recompiling the code

```shell
(gdb) dprintf 15,"at line 15, i=%d\n",i   #assign dynamic printf format here
Dprintf 2 at 0x4005b7: file hello.c, line 15.
(gdb) c
Continuing.
Hello, world
i is 0
at line 15, i=1         # Dynamic printf is here
i is now 1
[Inferior 1 (process 9232) exited normally]
```

PS: <code>dprintf 15,"at line 15, i=%d\n",i</code>. Make sure no spaces between the arguments comma <code>,</code>

## Summary

The video also covers multi-threading and multi-process GDB syntax, Please watch the video to understand how he debug for multi-threading and 
multi-process program.

After watching this video, I see GDB differently, there are a lot of features that is not cover in the video. Please visit
the official GDB website to understand more on GDB.
