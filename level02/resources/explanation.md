What do we have?

	$ ll
	total 24
	dr-x------ 1 level02 level02  120 Mar  5  2016 ./
	d--x--x--x 1 root    users    340 Aug 30  2015 ../
	-r-x------ 1 level02 level02  220 Apr  3  2012 .bash_logout*
	-r-x------ 1 level02 level02 3518 Aug 30  2015 .bashrc*
	----r--r-- 1 flag02  level02 8302 Aug 30  2015 level02.pcap
	-r-x------ 1 level02 level02  675 Apr  3  2012 .profile*

A `.pcap` file! Looking online, it is a record of packets transaction. It's recommended to use a tool such as `wireshark` to analyze it, so we'll use the `tshark` of `wireshark` for it. First, let's transfer the file to the host so that we can give it to the Docker.

	$ scp -P 4242 level02@192.168.56.101:/home/user/level02/level02.pcap ./
		_____                      _____               _     
		/ ____|                    / ____|             | |    
		| (___  _ __   _____      _| |     _ __ __ _ ___| |__  
		\___ \| '_ \ / _ \ \ /\ / / |    | '__/ _` / __| '_ \ 
		____) | | | | (_) \ V  V /| |____| | | (_| \__ \ | | |
		|_____/|_| |_|\___/ \_/\_/  \_____|_|  \__,_|___/_| |_|
															
	Good luck & Have fun

			192.168.56.101 
	level02@192.168.56.101's password: 
	level02.pcap 100% 8302 5.8MB/s 00:00

Then, let's use `tshark` to follow the transactions, in ASCII.

	$ docker pull cincan/tshark
	Using default tag: latest
	latest: Pulling from cincan/tshark
	2408cc74d12b: Pull complete 
	51d192427449: Pull complete 
	28515114a011: Pull complete 
	5edd0b038d9d: Pull complete 
	Digest: sha256:9cf3985977320cde1b19e9cbb3130c03c5668ff7b561ec97bfff28c850383104
	Status: Downloaded newer image for cincan/tshark:latest
	docker.io/cincan/tshark:latest

	$ docker run --user root  --rm -v ${PWD}:/samples cincan/tshark -r /samples/level02.pcap -q -z "follow,tcp,ascii,0"
	===================================================================
	Follow: tcp,ascii
	Filter: tcp.stream eq 0
	Node 0: 59.233.235.218:39247
	Node 1: 59.233.235.223:12121
		3
	..%
	3
	..%
		18
	..&..... ..#..'..$
	18
	..&..... ..#..'..$
		24
	.. .....#.....'.........
	67
	.. .38400,38400....#.SodaCan:0....'..DISPLAY.SodaCan:0......xterm..
		18
	........"........!
	74
	........"..".....b........b.....B.
	..............................1.......!
		7
	.."....
	7
	.."....
		15
	..!..........."
	9
	........"
		41
	.."................
	.....................
		75

	Linux 2.6.38-8-generic-pae (::ffff:10.1.1.2) (pts/10)

	..wwwbugs login: 
	1
	l
		2
	.l
	1
	e
		2
	.e
	1
	v
		2
	.v
	1
	e
		2
	.e
	1
	l
		2
	.l
	1
	X
		2
	.X
	1

		1
	.
		13
	.
	Password: 
	1
	f
	1
	t
	1
	_
	1
	w
	1
	a
	1
	n
	1
	d
	1
	r
	1
	.
	1
	.
	1
	.
	1
	N
	1
	D
	1
	R
	1
	e
	1
	l
	1
	.
	1
	L
	1
	0
	1
	L
	1

		3
	.

		1
	.
		35
	.
	Login incorrect
	wwwbugs login: 
	===================================================================

If we clean the output a little bit:

	Linux 2.6.38-8-generic-pae (::ffff:10.1.1.2) (pts/10)

	wwwbugs login: levelX
	Password: ft_wandr...NDRel.L0L

	Login incorrect
	wwwbugs login:

Let's try the password

	$ su flag02
	Password: 
	su: Authentication failure

It doesn't work. The man of `tshark` says `ascii: ASCII output with dots for non-printable characters`. So, the dots are not dots, but non-printable characters. By passing the output into hexa, we confirm that they are `DEL` chars (`7f` in Hexa in the Ascii table).

	$ docker run --user root  --rm -v ${PWD}:/samples cincan/tshark -r /samples/level02.pcap -q -z "follow,tcp,hex,0"
	===================================================================
	Follow: tcp,hex
	Filter: tcp.stream eq 0
	Node 0: 59.233.235.218:39247
	Node 1: 59.233.235.223:12121
		00000000  ff fd 25                                          ..%
	00000000  ff fc 25                                          ..%
		00000003  ff fb 26 ff fd 18 ff fd  20 ff fd 23 ff fd 27 ff  ..&.....  ..#..'.
		00000013  fd 24                                             .$
	00000003  ff fe 26 ff fb 18 ff fb  20 ff fb 23 ff fb 27 ff  ..&.....  ..#..'.
	00000013  fc 24                                             .$
		00000015  ff fa 20 01 ff f0 ff fa  23 01 ff f0 ff fa 27 01  .. ..... #.....'.
		00000025  ff f0 ff fa 18 01 ff f0                           ........
	00000015  ff fa 20 00 33 38 34 30  30 2c 33 38 34 30 30 ff  .. .3840 0,38400.
	00000025  f0 ff fa 23 00 53 6f 64  61 43 61 6e 3a 30 ff f0  ...#.Sod aCan:0..
	00000035  ff fa 27 00 00 44 49 53  50 4c 41 59 01 53 6f 64  ..'..DIS PLAY.Sod
	00000045  61 43 61 6e 3a 30 ff f0  ff fa 18 00 78 74 65 72  aCan:0.. ....xter
	00000055  6d ff f0                                          m..
		0000002D  ff fb 03 ff fd 01 ff fd  22 ff fd 1f ff fb 05 ff  ........ ".......
		0000003D  fd 21                                             .!
	00000058  ff fd 03 ff fc 01 ff fb  22 ff fa 22 03 01 00 00  ........ ".."....
	00000068  03 62 03 04 02 0f 05 00  00 07 62 1c 08 02 04 09  .b...... ..b.....
	00000078  42 1a 0a 02 7f 0b 02 15  0f 02 11 10 02 13 11 02  B....... ........
	00000088  ff ff 12 02 ff ff ff f0  ff fb 1f ff fa 1f 00 b1  ........ ........
	00000098  00 31 ff f0 ff fd 05 ff  fb 21                    .1...... .!
		0000003F  ff fa 22 01 03 ff f0                              .."....
	000000A2  ff fa 22 01 07 ff f0                              .."....
		00000046  ff fa 21 03 ff f0 ff fb  01 ff fd 00 ff fe 22     ..!..... ......"
	000000A9  ff fd 01 ff fb 00 ff fc  22                       ........ "
		00000055  ff fa 22 03 03 e2 03 04  82 0f 07 e2 1c 08 82 04  .."..... ........
		00000065  09 c2 1a 0a 82 7f 0b 82  15 0f 82 11 10 82 13 11  ........ ........
		00000075  82 ff ff 12 82 ff ff ff  f0                       ........ .
		0000007E  0d 0a 4c 69 6e 75 78 20  32 2e 36 2e 33 38 2d 38  ..Linux  2.6.38-8
		0000008E  2d 67 65 6e 65 72 69 63  2d 70 61 65 20 28 3a 3a  -generic -pae (::
		0000009E  66 66 66 66 3a 31 30 2e  31 2e 31 2e 32 29 20 28  ffff:10. 1.1.2) (
		000000AE  70 74 73 2f 31 30 29 0d  0a 0a 01 00 77 77 77 62  pts/10). ....wwwb
		000000BE  75 67 73 20 6c 6f 67 69  6e 3a 20                 ugs logi n:
	000000B2  6c                                                l
		000000C9  00 6c                                             .l
	000000B3  65                                                e
		000000CB  00 65                                             .e
	000000B4  76                                                v
		000000CD  00 76                                             .v
	000000B5  65                                                e
		000000CF  00 65                                             .e
	000000B6  6c                                                l
		000000D1  00 6c                                             .l
	000000B7  58                                                X
		000000D3  00 58                                             .X
	000000B8  0d                                                .
		000000D5  01                                                .
		000000D6  00 0d 0a 50 61 73 73 77  6f 72 64 3a 20           ...Passw ord:
	000000B9  66                                                f
	000000BA  74                                                t
	000000BB  5f                                                _
	000000BC  77                                                w
	000000BD  61                                                a
	000000BE  6e                                                n
	000000BF  64                                                d
	000000C0  72                                                r
	000000C1  7f                                                .
	000000C2  7f                                                .
	000000C3  7f                                                .
	000000C4  4e                                                N
	000000C5  44                                                D
	000000C6  52                                                R
	000000C7  65                                                e
	000000C8  6c                                                l
	000000C9  7f                                                .
	000000CA  4c                                                L
	000000CB  30                                                0
	000000CC  4c                                                L
	000000CD  0d                                                .
		000000E3  00 0d 0a                                          ...
		000000E6  01                                                .
		000000E7  00 0d 0a 4c 6f 67 69 6e  20 69 6e 63 6f 72 72 65  ...Login  incorre
		000000F7  63 74 0d 0a 77 77 77 62  75 67 73 20 6c 6f 67 69  ct..wwwb ugs logi
		00000107  6e 3a 20                                          n:
	===================================================================

Cleaned:

	000000B9  66                                                f
	000000BA  74                                                t
	000000BB  5f                                                _
	000000BC  77                                                w
	000000BD  61                                                a
	000000BE  6e                                                n
	000000BF  64                                                d
	000000C0  72                                                r
	000000C1  7f                                                .
	000000C2  7f                                                .
	000000C3  7f                                                .
	000000C4  4e                                                N
	000000C5  44                                                D
	000000C6  52                                                R
	000000C7  65                                                e
	000000C8  6c                                                l
	000000C9  7f                                                .
	000000CA  4c                                                L
	000000CB  30                                                0
	000000CC  4c                                                L

So, replacing the dots by a `DEL` (`Backspace` in terminal): `ft_waNDReL0L`

	$ su flag02
	Password: 
	Don't forget to launch getflag !
	$ getflag
	Check flag.Here is your token : kooda2puivaav1idi4f57q8iq
