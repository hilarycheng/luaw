Luaw
=====
***

Luaw stands for "Lua web server". It also matches abbreviation for an air traffic controller command "Line Up And Wait" that closely resembles the way it handles multiple requests using event loop :)

Luaw is an event driven, non blocking IO based HTTP application server inspired by node.js. It uses node.js's excellent async library libuv to do non-blocking IO but instead of Javascript it uses [Lua](http://www.lua.org/) as its primary language for application development. Luaw takes advantage of Lua's first class coroutine support to avoid [callback spaghetti problem](http://callbackhell.com/). This makes writing async IO code as straight forward as writing sequential code while all the heavy-lifting of application state management is transparently handled by Lua coroutines. This mean a Luaw application developer gets best of both worlds - [scale](http://www.kegel.com/c10k.html) of event driven IO and code simplicity of blocking IO.


##Features

1. Full HTTP 1.1 support with persistent/keep-alive connections and HTTP pipelining with configurable connect and read timeouts.
2. Two ways to read and parse incoming HTTP requests,
    - Reading whole request in memory and then parsing it which is suitable for majority of applications.
    - Reading request as a stream and parsing it as it arrives in parts to minimize latency and server memory footprint for high traffic, high performance applications like HTTP proxies or HTTP load balancers.
3. Similarly on the output side it supports,
    - Buffering entire content in memory before writing it out to client with the correct "Content-Length" header, or
    - Streaming response continuously as it is generated using HTTP 1.1 chunked encoding to keep server memory pressure minimum.
4. Fully asynchronous DNS and HTTP client for making remote HTTP calls from server
5. [Sinatra](http://www.sinatrarb.com/) like web application framework that supports mapping URLs to REST resources with full support for path and query parameters.
6. Luaw template views for server side dynamic content generation . Luaw template views use streaming output with chunked encoding described above to eliminate need for huge server side memory buffers to buffer output generated by the templates.
7. Support for spawning user threads that can hook into Luaw's async IO machinery for calling multiple backend services from Luaw server in parallel.
8. Support for user timers for implementing periodic cron like jobs inside Luaw server.
9. Log4j like logging framework with configurable log levels, log file size limits and automatic log rotation that integrates with syslog out of the box.
10. [MessagePack](http://msgpack.org/) like library to efficiently serialize/deserialize arbitrarily complex data into compact and portable binary format for remote web service calls.


##How To Build
***
Let's build Luaw from its sources. Luaw depends on,

1. Lua 5.2(+)
2. libuv (v1.0.0 )

We will first build these dependencies from their sources and then finally build Luaw itself. To build these artifacts you would need [Git](http://git-scm.com/),  [Make](http://www.gnu.org/software/make/) and [autotools](http://www.gnu.org/software/automake/manual/html_node/Autotools-Introduction.html) setup on your machine.

##Building Lua 5.2
Lua sources can be downloaded from [here](http://www.lua.org/download.html). Here are the steps to download Lua 5.2 sources and build it for Linux:

    curl -R -O http://www.lua.org/ftp/lua-5.2.3.tar.gz
    tar zxf lua-5.2.3.tar.gz
    cd lua-5.2.3
    make linux test
    sudo make linux install

To build for other OSes replace "linux" from the last two make commands with the OS you are building for. For example for when building for Mac OS run,

    make macosx test
    sudo make macosx install

To see what OSes are supported run ./lua-5.2.3/src/Makefile targets


##Building libuv
Luaw uses node.js library libuv to do asynchronous, event based IO in a portable, cross platform  manner. To build libuv:

1. first clone libuv repository

        git clone https://github.com/joyent/libuv.git
        
2. Checkout latest stable release of libuv from the cloned local repository. As of this writing the latest stable release is v1.0.0 and Luaw is verified to compile and run successfully with v1.0.0 release of libuv.

        cd libuv
        git checkout tags/v1.0.0
        
3. Build libuv. Detailed instructions are [here](https://github.com/joyent/libuv#build-instructions)
        
        sh autogen.sh
        ./configure
        make
        make check
        sudo make install

## Building Luaw
With all dependencies built, now we are ready to build Luaw itself.

1. Clone Luaw repository

        git clone https://github.com/raksoras/luaw.git
        
2. Build Luaw

        cd luaw/src
        make linux
        
3. Install Luaw binary - luaw_server - in directory of your choice

        make INSTALL_ROOT=<luaw_root_dir> install
        
4. Note: On Mac running Yosemite version of Mac OS you may have to run,

        make SYSCFLAGS=-I/usr/local/include SYSLDFLAGS=-L/usr/local/lib macosx
        make INSTALL_ROOT=<luaw_root_dir> install


##Luaw directory structure

In the tree diagram below `luaw_root_dir` is a directory that you chose in the `make intsall` step above. It will act as a root for Luaw server's directory structure. The `make install` step will create following directory structure under `luaw_root_dir`

```
luaw_root_dir
 |
 |
 +--- bin              Directory that holds Luaw server binary we built
 |                     along with all necessary Lua libraries
 |
 +--- conf             Directory for server configuration
 |   |
 │   +--- server.cfg   Sample server configuration file, to be used as a starting point
 |
 +--- lib              Directory to install any third party or user supplied Lua
 |                     libraries that application may depend on.
 |
 +--- logs             Directory for server logs
 |
 +--- webapps          Directory to install Luaw webapps
```

##License

Luaw uses [MIT license](http://opensource.org/licenses/mit-license.html) for all parts of Luaw that are not externally
maintained libraries like libuv.

## Documentation

Please refer to the [project wiki](https://github.com/raksoras/luaw/wiki) for documentation regarding how to build, configure, start and program using Luaw.
