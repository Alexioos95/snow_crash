What do we have?

	$ ls -la
	total 12
	dr-xr-x---+ 1 level00 level00  100 Mar  5  2016 .
	d--x--x--x  1 root    users    340 Aug 30  2015 ..
	-r-xr-x---+ 1 level00 level00  220 Apr  3  2012 .bash_logout
	-r-xr-x---+ 1 level00 level00 3518 Aug 30  2015 .bashrc
	-r-xr-x---+ 1 level00 level00  675 Apr  3  2012 .profile

After checking the 3 files, there is nothing noteworthy, aside of 3 useful alias for `ls` in the `.bashrc`.

	alias ll='ls -alF'
	alias la='ls -A'
	alias l='ls -CF'

So let's explore all the VM... After searching, we found a file titled `john` in `/usr/sbin`.

	$ cd /usr/sbin; cat john
	cdiiddwpgswtgt

It contains a string, definitely a password.

	$ su flag00
	Password: 
	su: Authentication failure

It isn't a password in itself, so it probably has been altered. We tried the most popular cipher, the Caesar cipher, with the help of https://www.dcode.fr/caesar-cipher and found the right sequence : shift of 11 to the right (or 15 to the left) gives `nottoohardhere`.

	$ su flag00
	Password: 
	Don't forget to launch getflag !
	$ getflag
	Check flag.Here is your token : x24ti5gi3x0ol2eh4esiuxias
