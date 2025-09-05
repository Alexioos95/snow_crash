What do we have?

	$ ll
	total 24
	dr-x------ 1 level09 level09  140 Mar  5  2016 ./
	d--x--x--x 1 root    users    340 Aug 30  2015 ../
	-r-x------ 1 level09 level09  220 Apr  3  2012 .bash_logout*
	-r-x------ 1 level09 level09 3518 Aug 30  2015 .bashrc*
	-rwsr-sr-x 1 flag09  level09 7640 Mar  5  2016 level09*
	-r-x------ 1 level09 level09  675 Apr  3  2012 .profile*
	----r--r-- 1 flag09  level09   26 Mar  5  2016 token 

Again, a binary `level09`, with the rights to launch as owner, and a `token` file.

`Ghidra`:

	size_t main(int param_1,int param_2)

	{
	char cVar1;
	bool bVar2;
	long lVar3;
	size_t sVar4;
	char *pcVar5;
	int iVar6;
	int iVar7;
	uint uVar8;
	int in_GS_OFFSET;
	byte bVar9;
	uint local_120;
	undefined1 local_114 [256];
	int local_14;
	
	bVar9 = 0;
	local_14 = *(int *)(in_GS_OFFSET + 0x14);
	bVar2 = false;
	local_120 = 0xffffffff;
	lVar3 = ptrace(PTRACE_TRACEME,0,1,0);
	if (lVar3 < 0) {
		puts("You should not reverse this");
		sVar4 = 1;
	}
	else {
		pcVar5 = getenv("LD_PRELOAD");
		if (pcVar5 == (char *)0x0) {
		iVar6 = open("/etc/ld.so.preload",0);
		if (iVar6 < 1) {
			iVar6 = syscall_open("/proc/self/maps",0);
			if (iVar6 == -1) {
			fwrite("/proc/self/maps is unaccessible, probably a LD_PRELOAD attempt exit..\n",1,0x46,
					stderr);
			sVar4 = 1;
			}
			else {
			do {
				do {
				while( true ) {
					sVar4 = syscall_gets(local_114,0x100,iVar6);
					if (sVar4 == 0) goto LAB_08048a77;
					iVar7 = isLib(local_114,&DAT_08048c2b);
					if (iVar7 == 0) break;
					bVar2 = true;
				}
				} while (!bVar2);
				iVar7 = isLib(local_114,&DAT_08048c30);
				if (iVar7 != 0) {
				if (param_1 == 2) goto LAB_08048996;
				sVar4 = fwrite("You need to provied only one arg.\n",1,0x22,stderr);
				goto LAB_08048a77;
				}
				iVar7 = afterSubstr(local_114,"00000000 00:00 0");
			} while (iVar7 != 0);
			sVar4 = fwrite("LD_PRELOAD detected through memory maps exit ..\n",1,0x30,stderr);
			}
		}
		else {
			fwrite("Injection Linked lib detected exit..\n",1,0x25,stderr);
			sVar4 = 1;
		}
		}
		else {
		fwrite("Injection Linked lib detected exit..\n",1,0x25,stderr);
		sVar4 = 1;
		}
	}
	LAB_08048a77:
	if (local_14 == *(int *)(in_GS_OFFSET + 0x14)) {
		return sVar4;
	}
						/* WARNING: Subroutine does not return */
	__stack_chk_fail();
	LAB_08048996:
	local_120 = local_120 + 1;
	uVar8 = 0xffffffff;
	pcVar5 = *(char **)(param_2 + 4);
	do {
		if (uVar8 == 0) break;
		uVar8 = uVar8 - 1;
		cVar1 = *pcVar5;
		pcVar5 = pcVar5 + (uint)bVar9 * -2 + 1;
	} while (cVar1 != '\0');
	if (~uVar8 - 1 <= local_120) goto code_r0x080489ca;
	putchar((int)*(char *)(local_120 + *(int *)(param_2 + 4)) + local_120);
	goto LAB_08048996;
	code_r0x080489ca:
	sVar4 = fputc(10,stdout);
	goto LAB_08048a77;
	}

And we have the rights to read `token`.

	$ cat token
	f4kmm6p|=pnDBDu{

So, the binary has a lot of checks for reverse engineering, but in the end, just do some operations on each characters of the input, which, we guess, created the string in `token` that we have to reverse. The code is clearly obfuscated to be very hard to read, so we just randomly tested its behavior.

	$ ./level09 abc
	ace
	$ ./level09 123
	135
	$ ./level09 aaa
	abc
	$ ./level09 111
	123

Seems like it just adds the index of the character in the string to the `char` itself, in reality. Let's do a C program that will reverse that:

	$ << 'end' cat > /tmp/e.c
	#include <stdio.h>
	#include <unistd.h>
	#include <string.h>

	int main(int argc, char **argv)
	{
		int i = 0;
		char c;

		while (i < strlen(argv[1]))
		{
			c = argv[1][i] - i;
			write(1, &c, 1);
			i++;
		}
		write(1, "\n", 1);
		return 0;
	}
	end

And run it with the content of `token`.

	$ cd /tmp; cc /tmp/e.c; /tmp/a.out $(cat ~/token)
	f3iji1ju5yuevaus41q1afiuq

We haven't launched `getflag`, so we log into `flag09` for it.

	$ su flag09
	Password:
	Don't forget to launch getflag !
	$ getflag
	Check flag.Here is your token : s5cAJpM8ev6XHw998pRWG728z
