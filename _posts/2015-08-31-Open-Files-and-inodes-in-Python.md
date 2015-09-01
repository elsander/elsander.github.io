---
layout: post
title: Open Files and inodes in Python and Beyond

---

Today I was trying to solve a SQLalchemy bug in Python when I
discovered a strange behavior. I would set up logging to file, run
some SQLalchemy code, and look at the log. Then I would open the file in emacs and
clear out the log if there wasn't anything interesting. But then, when
I ran more SQLalchemy, the logging wouldn't work anymore. Python
didn't throw an error, it just stopped logging altogether. Even if I
tried to reset my logging by setting up a new session, I couldn't get
any more log statements without quitting and reopening ipython
altogether.

I paired with my fellow RCer
[John Xia](http://www.johnxiaisontheinter.net/) to solve this
problem. I assumed 
it was something specific to the logging module, but we discovered
that the problem ran deeper and was much stranger than I had
originally realized. You can get precisely the same behavior just by
opening a file in Python and then editing it manually. But even
stranger, `echo`-ing to the file from the terminal does not produce
this behavior. Try this to see what I mean (definitely works on Linux,
and probably will also work on a Mac):

In Python:

    f = open('tmp', 'a')
    f.write('Python!')
	f.flush()

Ok, now we have our little file. Now from the command line, run:

	echo "echo!" >> tmp

You can `cat` the file to see that it has everything we've added so
far. Now go back to Python:

    f.write('Python again!')
    f.flush()

So far, so good. You can `cat` the file and you'll have some text from
Python, some text from echo, and more text from Python. Python doesn't
care that `echo` has appended to a file it's using. Now, open it in
emacs or vim (this will work with most advanced editors, so something
like Sublime should also be fine). In your text editor, add a line,
then save and close. If you `cat` it, that line will appear there too,
no big surprise. But now in Python:

    f.write('Python is confused!')
    f.flush()

Try looking at your file. Python didn't add anything! Where did that
line go? From now on, `f` is basically useless to you. Python will
happily allow you to use it, and won't throw an error, but nothing
else you write will show up in the file. Bizarre!

This file connection is useless to us now, so let's close and reopen
it.

    f.close()
	f = open('tmp', 'a')

So what's happening here? Why does Python get confused by emacs but
not `echo`? To understand this, try using `lsof` to see what processes
are running. By default, `lsof` will give you *all* processes running
on the computer, so try running something like `lsof | grep
'path/to/folder/containing/tmp'`. There's a lot of information here, but
look for python or ipython in leftmost column, you'll see something
like this:

    ipython   10560   username   5w   REG   8,21   35   8919545    /path/to/tmp/tmp

The last three columns are what you care about. From right to left,
you have to path to your file, the inode, and the offset (for other
processes, it may be the size instead). The offset
is essentially telling you how many characters have been written to
file. The inode is how Linux stores information about the file,
including metadata and blocks of memory used to store the file. For
me, ipython sees that the file `tmp` is using inode 8919545. Now if we
write to that file using `echo`:

	echo 'hi' > tmp
	lsof | grep path/to/tmp

should give us

    ipython   10560   username   5w   REG   8,21   37   8919545    /path/to/tmp/tmp
    
The offset is now 37, because we've written two characters. So
everything seems to be working. But now let's open the file in emacs,
add a character, save, and then run the `lsof` command again. Now we should
see something like this:

    ipython   10560   username   5w   REG   8,21   37   8919545    /path/to/tmp/tmp~
    emacs     10680   username   5w   REG   8,21   4096 8918334    /path/to/tmp

There are a few important things to notice here. First, the offset for
ipython hasn't changed, which means it hasn't seen the change we made
in emacs. Next, the inode for ipython hasn't changed, but emacs is
using a different one. Finally, ipython is working with some sort of
deleted or temporary file, as noted by the ~ at the end of the file
name.

It turns out that most editors used today create temporary files, so
that if the program crashes, you don't lose your work. This is why
editors like emacs leave ~ files all over the place. The temporary
files are assigned different inodes, so when you save the file you're
editing, you end up assigning it a new inode. But ipython doesn't know
that this has happened! It's still connected to the old inode, so when
you write using this old inode connection, the change doesn't show up
in your current copy of the file. You're just editing old
memory. Unlike emacs, commands like `echo` edit your file inline, so
the inode doesn't change. This lets Python keep a working connection
to the file. There are some older editors that do this too. Vi edits
inline, but vim creates swap files, which means that you'll cause
problems editing in vim but not vi! 

This isn't an issue that's specific to Python. You can do it in C as
well! Here's some code (courtesy of John Xia) that will do it.

	#include <stdlib.h>
	#include <unistd.h>
	#include <fcntl.h>
	#include <stdio.h>

	int main (int argc, char *argv) {
	  int fd = open("f",
					O_WRONLY | O_CREAT | O_TRUNC,
					S_IRUSR | S_IWUSR | S_IRGRP | S_IROTH);
	  printf("Writing \"write from c\\n\"...\n");
	  write(fd, "write from c\n", 13);
	  printf("Do you want me to run `vim +wq g` (this will change the inode) (y/n)? ");
	  int doVim = getchar();
	  if (doVim == 'y') {
		system("vim +wq f");
	  }
	  printf("Do you want me to run `vi +wq g` (this will edit in-place and not change the inode) (y/n)? ");
	  getchar();
	  int doVi = getchar();
	  if (doVi == 'y') {
		system("vi +wq f");
	  }
	  printf("\nWriting \"write from c!\"...\n");
	  write(fd, "write from c!", 13);
	  close(fd);
	  return 0;
	}

This was an interesting problem to look into, because nothing throws
an error. The (fairly common-sense) moral of the story is, don't edit
files that you're editing with a program. Tying it back to Python's
logging module, it seems that the logger simply opens a file
connection and leaves it open. Viewing the file should be fine, but
editing the text could break it. If you really need to edit it while
your code is running, you can use an inline editor like vi as a workaround.
