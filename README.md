shadowsocks-libuv
=================
[![Build Status](https://travis-ci.org/dndx/shadowsocks-libuv.png?branch=master)](https://travis-ci.org/dndx/shadowsocks-libuv)

shadowsocks is a lightweight tunnel proxy which can help you get through firewalls. 

protol made by [clowwindy](https://raw.github.com/clowwindy/), libuv port by [dndx](https://github.com/dndx)

This is only a **server**, it should works with any shadowsocks client. 

Current version: 0.2

This is an [Open Source](http://opensource.org/licenses/MIT) project and released under [The MIT License](http://opensource.org/licenses/MIT)

## Features
* Super fast and low resource consume (thanks to [libuv](https://github.com/joyent/libuv)), it can run very smoothly on almost any VPS. 
* Fully compatible to other port of shadowsocks. 
* Support the latest RC4 encryption method. 
* Fully IPv6 Ready

## About IPv6 Support
Instead of create two separate file descriptor for IPv4 and IPv6, shadowsocks-libuv only create one. The reason it works is that since Linux kernel 2.4.21 and 2.6, we can use `IN6ADDR_ANY` (aka. `::0`) to accept connection from both IPv4 and IPv6 stack. Those connections who comes from IPv4 stack will be mapped to [IPv4-mapped IPv6 addresses](https://en.wikipedia.org/wiki/IPv6#IPv4-mapped_IPv6_addresses) automaticly. For example, IPv4 address `192.168.1.2` will be mapped to `::ffff:192:168:1:2` and will work whether your actual machine have IPv6 link or not. 

If you want your shadowsocks listen on specified IPv4 address, just make it listen on something like `::ffff:192:168:1:2`. 

When connect to remote server, shadowsocks will prefer to use the IPv6 address if both your server and remote supports IPv6. This will work even your connection to the server is using IPv4. Thus you can use shadowsocks as an IPv4 to IPv6 or IPv6 to IPv4 tunnel. 

### Diagram

	+------+    IPv4    +------+    IPv4    +------+
	|Client| <---OR---> |Server| <---OR---> |Remote|
	+------+    IPv6    +------+    IPv6    +------+
Client is any compatible shadowsocks client

Server is shadowsocks-libuv or other compatible server

Remote is the service you are trying to access

## Attention
Please open an issue if you encounter any bugs. Be sure to attach the error message so I can identify it. 

## How to Build
	$ yum install openssl-devel
	$ git clone --recursive https://github.com/dndx/shadowsocks-libuv.git
	$ cd shadowsocks-libuv/
	$ vim config.h
	$ make

Note that you need to rebuild it every time you modify config.h, just run `make` again and it will do rest of the work. 

Tested and confirmed to work on:

* Mac OS X 10.8.2 x64 using Clang 4.1
* CentOS 5.8 x86 using GCC 4.1.2
* CentOS 6.3 x64 using GCC 4.4.6
* Ubuntu Linux 12.04 using GCC 4.6.x and Clang 3.1.x (Travis Environment)

## How to Use
After you build shadowsocks successfully, you can rename the file `server` and move it to anywhere you want. 

Also there are command line options that can be used to override settings in config.h:

	$ ./server -?
	./server: invalid option -- ?
	Shadowsocks Version:0.2 libuv(0.9) Written by Dndx(idndx.com)
	Usage: ./server [-l listen] [-p port] [-k keyfile] [-f pidfile] [-m rc4|shadow]

	Options:
      -l : Override the listening IP
	  -p : Override the listening port
	  -k : Override the listening password
	  -f : Override the pidfile path
	  -m : Override the encryption method
If you want to run server in background, you can:

	$ nohup /path/to/server &

Note this will redirect all log generated by program to `nohup.out`. If you do not like any log to be saved, you can run:

	$ nohup /path/to/server > /dev/null 2>&1 &

This will throw all the message to a black hole. 

I suggest the first way. If your program does not run as expected, log file will be very important for me to identify the bug. 

## Known Issues
### Build Failed
	src/unix/linux/syscalls.h:74: error: expected specifier-qualifier-list before ‘__u64’
1. First, make sure you have the latest kernel-headers by running `yum install kernel-headers`
2. Try make again, if it still complains, see next
3. `cd` to shadowsocks-libuv and `$ vim libuv/config-unix.mk`
4. At about line 22, you will see `CSTDFLAG=--std=c89 -pedantic -Wall -Wextra -Wno-unused-parameter`
5. Change it to `CSTDFLAG=--std=gnu99 -pedantic -Wall -Wextra -Wno-unused-parameter`
6. Save the file and run `make` again

### Can not using static link
Thanks to [@cyfdecy](https://github.com/cyfdecyf)

Do not use `-static` to compile this project. The problem is that `libuv` is using `pthread` and `getaddrinfo(3)` for async DNS resolve. But this will cause `getaddrinfo(3)` Segment Fault. 

More information can be found at [issue #3](https://github.com/dndx/shadowsocks-libuv/issues/3)

### CPU 100%
There is a possibility that the process will sometimes consume 100% of CPU and can not be restored. The cause of problem is not clear but I am working for a solution. Including re-design the way connection is handled, please stay tuned. 


## Performance
I did not fully benchmark it yet, but according to my use on [TinyVZ](http://tinyvz.com/) (OpenVZ 128M RAM and CentOS 5.8 x86). When streaming YouTube 1080p vedio at about 20Mbit/s bandwidth, it use at most 3% of RAM (RSS about 7500) and almost no CPU time. During webpage browse it use less than 0.7% RAM (RSS about 1164). Which can be considered effective. 

##Contributors
* [@messense](https://github.com/messense) Logger Color
* [@clowwindy](https://github.com/clowwindy)
	* Test Case and Protol Maker of shadowsocks
	* [getopt(3) issue](https://github.com/dndx/shadowsocks-libuv/pull/4)
* [@cyfdecyf](https://github.com/cyfdecyf) Warning about static link in [issue #3](https://github.com/dndx/shadowsocks-libuv/issues/3)
* [@madeye](https://github.com/madeye) [IPv6 Support](https://github.com/dndx/shadowsocks-libuv/pull/8)

I appreciate all the people who have made this project better. Contribution is always welcome! 

## TODO List
* ~~Fully IPv6 Support~~ (accomplished)
* ~~RC4 Crypto Support~~ (accomplished)
* Multi Port Support
* Client Implement
* …to be continued…