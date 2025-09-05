What do we have?

	$ ll
	total 12
	dr-xr-x---+ 1 level00 level00  100 Mar  5  2016 .
	d--x--x--x  1 root    users    340 Aug 30  2015 ..
	-r-xr-x---+ 1 level00 level00  220 Apr  3  2012 .bash_logout
	-r-xr-x---+ 1 level00 level00 3518 Aug 30  2015 .bashrc
	-r-xr-x---+ 1 level00 level00  675 Apr  3  2012 .profile

Nothing... We need to explore the VM again... but it's so bothersome, and we don't know where to look, really. By looking at the `john` file used for the previous level, we noticed that he was modified on the 5th March 2016.

	$ ll
	----r--r--  1 flag00  flag00      15 Mar  5  2016 john

So, we used `find` to search for other such files, from the root of the disk, and in recursive.

	$ find / -type f -newermt "2016-03-05" ! -newermt "2016-03-06" 2>/dev/null
	/etc/apache2/sites-available/level05.conf
	/etc/init.d/level11_start
	/etc/issue.bak
	/etc/issue.bak.old
	/etc/issue.net.bak
	/etc/issue.net.bak.old
	/etc/passwd
	/etc/remastersys.conf
	/lib/plymouth/themes/ubuntu-text/ubuntu-text.plymouth
	/usr/bin/remastersys
	/usr/sbin/john
	/usr/sbin/openarenaserver
	/usr/share/gdb/python/gdb/command/__init__.pyc
	/usr/share/gdb/python/gdb/command/pretty_printers.pyc
	/usr/share/gdb/python/gdb/command/prompt.pyc
	/usr/share/gdb/python/gdb/prompt.pyc
	/var/cache/apt/pkgcache.bin
	/var/cache/apt/srcpkgcache.bin
	/var/cache/man/index.db
	/var/lib/apt/extended_states
	/var/lib/dpkg/lock
	/var/lib/dpkg/status
	/var/lib/dpkg/status-old
	/var/lib/dpkg/triggers/Lock
	/var/lib/update-notifier/dpkg-run-stamp
	/var/lib/update-notifier/updates-available
	/rofs/etc/apache2/sites-available/level05.conf
	/rofs/etc/hostname
	/rofs/etc/init.d/level11_start
	/rofs/etc/issue.bak
	/rofs/etc/issue.bak.old
	/rofs/etc/issue.net.bak
	/rofs/etc/issue.net.bak.old
	/rofs/etc/network/interfaces
	/rofs/etc/passwd
	/rofs/etc/remastersys.conf
	/rofs/lib/plymouth/themes/ubuntu-text/ubuntu-text.plymouth
	/rofs/usr/bin/remastersys
	/rofs/usr/sbin/john
	/rofs/usr/sbin/openarenaserver
	/rofs/usr/share/gdb/python/gdb/command/__init__.pyc
	/rofs/usr/share/gdb/python/gdb/command/pretty_printers.pyc
	/rofs/usr/share/gdb/python/gdb/command/prompt.pyc
	/rofs/usr/share/gdb/python/gdb/prompt.pyc
	/rofs/var/cache/apt/pkgcache.bin
	/rofs/var/cache/apt/srcpkgcache.bin
	/rofs/var/cache/man/index.db
	/rofs/var/lib/apt/extended_states
	/rofs/var/lib/dpkg/lock
	/rofs/var/lib/dpkg/status
	/rofs/var/lib/dpkg/status-old
	/rofs/var/lib/dpkg/triggers/Lock
	/rofs/var/lib/update-notifier/dpkg-run-stamp
	/rofs/var/lib/update-notifier/updates-available

`/etc/passwd` is a very eye catching file, and when we `cat` it:

	$ cat /etc/passwd
	[...]
	flag00:x:3000:3000::/home/flag/flag00:/bin/
	flag01:42hDRfypTqqnw:3001:3001::/home/flag/flag01:/bin/
	flag02:x:3002:3002::/home/flag/flag02:/bin/bash 
	[...]

The string `42hDRfypTqqnw` has nothing to do there, and is extremely suspicious; definitely a password.

	$ su flag00
	Password: 
	su: Authentication failure

It isn't a password in itself, so it has been altered. After trying all popular cipher and hashing algorithms, we went more deep into it, and used `John The Ripper` to brute-force it.

	$ git clone https://github.com/openwall/john -b bleeding-jumbo john-jumbo; cd john-jumbo/src; ./configure && make -s clean && make -sj$(nproc); cd ../run
	$ echo 'flag01:42hDRfypTqqnw' > hash.txt
	$ ./john hash.txt
	Using default input encoding: UTF-8
	Loaded 1 password hash (descrypt, traditional crypt(3) [DES 256/256 AVX2])
	Will run 12 OpenMP threads
	Note: Passwords longer than 8 truncated (property of the hash)
	Proceeding with single, rules:Single
	Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
	Almost done: Processing the remaining buffered candidate passwords, if any.
	Warning: Only 663 candidates buffered for the current salt, minimum 3072 needed for performance.
	0g 0:00:00:00 DONE 1/3 (2025-07-10 08:00) 0g/s 44200p/s 44200c/s 44200C/s flag01..Flag0159
	Proceeding with wordlist:./password.lst
	Enabling duplicate candidate password suppressor using 256 MiB
	abcdefg (flag01)
	1g 0:00:00:00 DONE 2/3 (2025-07-10 08:01) 5.263g/s 395021p/s 395021c/s 395021C/s 123456..gravitat
	Use the "--show" option to display all of the cracked passwords reliably
	Session completed.

It was a DES hash of `abcdefg`.

	$ su flag01
	Password: 
	Don't forget to launch getflag !
	$ getflag
	Check flag.Here is your token : f2av5il02puano7naaf6adaaf
