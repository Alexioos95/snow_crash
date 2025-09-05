What do we have?

	$ ll
	total 20
	dr-x------ 1 level13 level13  120 Mar  5  2016 ./
	d--x--x--x 1 root    users    340 Aug 30  2015 ../
	-r-x------ 1 level13 level13  220 Apr  3  2012 .bash_logout*
	-r-x------ 1 level13 level13 3518 Aug 30  2015 .bashrc*
	-rwsr-sr-x 1 flag13  level13 7303 Aug 30  2015 level13*
	-r-x------ 1 level13 level13  675 Apr  3  2012 .profile*

A `level13` binary with rights of owner. `Ghidra`:

	void main(void)

	{
	__uid_t _Var1;
	undefined4 uVar2;
	
	_Var1 = getuid();
	if (_Var1 != 0x1092) {
		_Var1 = getuid();
		printf("UID %d started us but we we expect %d\n",_Var1,0x1092);
						/* WARNING: Subroutine does not return */
		exit(1);
	}
	uVar2 = ft_des("boe]!ai0FB@.:|L6l@A?>qJ}I");
	printf("your token is %s\n",uVar2);
	return;
	}

	char * ft_des(char *param_1)

	{
	char cVar1;
	char *pcVar2;
	uint uVar3;
	char *pcVar4;
	byte bVar5;
	uint local_20;
	int local_1c;
	int local_18;
	int local_14;
	
	bVar5 = 0;
	pcVar2 = strdup(param_1);
	local_1c = 0;
	local_20 = 0;
	do {
		uVar3 = 0xffffffff;
		pcVar4 = pcVar2;
		do {
		if (uVar3 == 0) break;
		uVar3 = uVar3 - 1;
		cVar1 = *pcVar4;
		pcVar4 = pcVar4 + (uint)bVar5 * -2 + 1;
		} while (cVar1 != '\0');
		if (~uVar3 - 1 <= local_20) {
		return pcVar2;
		}
		if (local_1c == 6) {
		local_1c = 0;
		}
		if ((local_20 & 1) == 0) {
		if ((local_20 & 1) == 0) {
			for (local_14 = 0; local_14 < "0123456"[local_1c]; local_14 = local_14 + 1) {
			pcVar2[local_20] = pcVar2[local_20] + -1;
			if (pcVar2[local_20] == '\x1f') {
				pcVar2[local_20] = '~';
			}
			}
		}
		}
		else {
		for (local_18 = 0; local_18 < "0123456"[local_1c]; local_18 = local_18 + 1) {
			pcVar2[local_20] = pcVar2[local_20] + '\x01';
			if (pcVar2[local_20] == '\x7f') {
			pcVar2[local_20] = ' ';
			}
		}
		}
		local_20 = local_20 + 1;
		local_1c = local_1c + 1;
	} while( true );
	}

Lot of obfuscation, but the program checks for an UID, exit if it's not the good one, and continue into a (de?)hashing function. Our UID isn't the one wanted.

	$ ./level13
	UID 2013 started us but we we expect 4242

So we just copy the hashing function into our own program, to see what the result of the input is.

	$ nano /tmp/e.c; cd /tmp; cc /tmp/e.c; ./a.out
	#include <stdio.h>
	#include <stdlib.h>
	#include <limits.h>
	#include <stdbool.h>
	#include <string.h>

	char * ft_des(char *param_1)

	{
	char cVar1;
	char *pcVar2;
	unsigned int uVar3;
	char *pcVar4;
	unsigned char bVar5;
	unsigned int local_20;
	int local_1c;
	int local_18;
	int local_14;

	bVar5 = 0;
	pcVar2 = strdup(param_1);
	local_1c = 0;
	local_20 = 0;
	do {
			uVar3 = 0xffffffff;
			pcVar4 = pcVar2;
			do {
			if (uVar3 == 0) break;
			uVar3 = uVar3 - 1;
			cVar1 = *pcVar4;
			pcVar4 = pcVar4 + (unsigned int)bVar5 * -2 + 1;
			} while (cVar1 != '\0');
			if (~uVar3 - 1 <= local_20) {
			return pcVar2;
			}
			if (local_1c == 6) {
			local_1c = 0;
			}
			if ((local_20 & 1) == 0) {
			if ((local_20 & 1) == 0) {
					for (local_14 = 0; local_14 < "0123456"[local_1c]; local_14 = local_14 + 1) {
					pcVar2[local_20] = pcVar2[local_20] + -1;
					if (pcVar2[local_20] == '\x1f') {
							pcVar2[local_20] = '~';
					}
					}
			}
			}
			else {
			for (local_18 = 0; local_18 < "0123456"[local_1c]; local_18 = local_18 + 1) {
					pcVar2[local_20] = pcVar2[local_20] + '\x01';
					if (pcVar2[local_20] == '\x7f') {
					pcVar2[local_20] = ' ';
					}
			}
			}
			local_20 = local_20 + 1;
			local_1c = local_1c + 1;
	} while( true );
	}

	int main(void)
	{
		printf("%s\n", ft_des("boe]!ai0FB@.:|L6l@A?>qJ}I"));
		return 0;
	}

This give us `2A31L79asukciNyi8uppkEuSx`. From there, we tried to log into `flag13`, but it didn't work... That's only after trying to search some kind of exploit or things to change under `gdb` for a while that we randomly tried to log directly onto `level14` with the string... and it worked...

	$ su level14
	Password: 
	$

Very weird... Oh, well. Whatever.
