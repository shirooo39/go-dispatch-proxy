# Go dispatch proxy

A SOCKS5 load balancing proxy to combine multiple internet connections into one. Works on Windows and Linux (partial support, see below). [Reported to work on macOS](https://github.com/extremecoders-re/go-dispatch-proxy/issues/1). Written in pure Go with no additional dependencies.

It can also be used as a transparent proxy to load balance multiple SSH tunnels.

## Rationale

The idea for this project came from [dispatch-proxy](https://github.com/Morhaus/dispatch-proxy) which is written in NodeJS.
[NodeJS is not entirely hard disk friendly considering the multitude of files it creates even for very simple programs](https://medium.com/@jdan/i-peeked-into-my-node-modules-directory-and-you-wont-believe-what-happened-next-b89f63d21558). I needed something which was light & portable, preferably a single binary without polluting the entire drive.

## Installation

No installation required. Grab the latest binary for your platform from the CI server or from [releases](https://github.com/extremecoders-re/go-dispatch-proxy/releases) and start speeding up your internet connection!

https://ci.appveyor.com/project/extremecoders-re/go-dispatch-proxy/build/artifacts

[![Build status](https://ci.appveyor.com/api/projects/status/nll4hvpdjlfsp7mu?svg=true)](https://ci.appveyor.com/project/extremecoders-re/go-dispatch-proxy/build/artifacts)

## Usage

The example below are shown on Windows. The steps are similar for other platforms.

### 1 - Load balance connections

The primary purpose of the tool is to combine multiple internet connections into one. For this we need to know the IP addresses of the interface we wish to combine. You can obtain the IP addresses using the `ipconfig` (`ifconfig` on linux) command. Alternatively run `go-dispatch-proxy -list`.

```
D:\>go-dispatch-proxy.exe -list
--- Listing the available adresses for dispatching
[+] Mobile Broadband Connection , IPv4:10.81.201.18
[+] Local Area Connection, IPv4:192.168.1.2
```

Start `go-dispatch-proxy` specifying the IP addresses of the load balancers obtained in the previous step. Optionally, along with the IP address you may also provide the contention ratio(after by the @ symbol). If no contention ratio is specified, it's assumed as 1.

### 2 - Load balance SSH tunnels

The tool can load balance multiple SSH tunnels. See Example 3 for usage.

**Example 1**

SOCKS proxy running on localhost at default port. Contention ratio is specified.
```
D:\>go-dispatch-proxy.exe 10.81.201.18@3 192.168.1.2@2
[INFO] Load balancer 1: 10.81.201.18, contention ratio: 3
[INFO] Load balancer 2: 192.168.1.2, contention ratio: 2
[INFO] SOCKS server started at 127.0.0.1:8080
```

**Example 2**

SOCKS proxy running on a different interface at a custom port. Contention ratio is not specified.

```
D:\>go-dispatch-proxy.exe -lhost 192.168.1.2 -lport 5566 10.81.177.215 192.168.1.100
[INFO] Load balancer 1: 10.81.177.215, contention ratio: 1
[INFO] Load balancer 2: 192.168.1.100, contention ratio: 1
[INFO] SOCKS server started at 192.168.1.2:5566
```

Out of 5 consecutive connections, the first 3 are routed to `10.81.201.18` and the remaining 2 to `192.168.1.2`. The SOCKS server is started by default on `127.0.0.1:8080`. It can be changed using the `-lhost` and `lport` directive.

Now change the proxy settings of your browser, download manager etc to point to the above address (eg `127.0.0.1:8080`). Be sure to add this as a SOCKS v5 proxy and NOT as a HTTP/S proxy.

**Example 3**

The tool can be used to load balance multiple SSH tunnels. In this mode, go-dispatch-proxy acts as a transparent load balancing proxy. 

First, setup the tunnels. 

```
D:\> ssh -D 127.0.0.1:7777 user@192.168.1.100
D:\> ssh -D 127.0.0.1:7778 user@192.168.1.101
```

Here we are setting up two SSH tunnels to remote hosts `192.168.1.100`, and `192.168.1.101` on local ports `7777` and `7778` respectively. The IP address (`127.0.0.1`) if omitted defaults to localhost. The `-D` option stands for dynamic port forwarding.

Next, launch go-dispatch-proxy using the `-tunnel` argument. 

```
D:\> go-dispatch-proxy.exe -tunnel 127.0.0.1:7777 127.0.0.1:7778
```

Both the IP and port must be mentioned while specifying the load balancer addresses. 

Optionally, the listening host, port and contention ratio can also be specified like in example 2.

```
D:\> go-dispatch-proxy.exe -lport 5555 -tunnel 127.0.0.1:7777@1 127.0.0.1:7778@3
```

The `lport` if not specified defaults to 8080. This is the port where you need to point your web browser, download manager etc. Be sure to add this as a SOCKS v5 proxy.


## Compiling (For Development)

Ensure that Go is installed and available on the system path.

```sh
$ git clone https://github.com/extremecoders-re/go-dispatch-proxy.git
$ cd go-dispatch-proxy

# Compile for Windows x86
$ GOOS=windows GOARCH=386 go build

# Compile for Windows x64
$ GOOS=windows GOARCH=amd64 go build

# Compile for Linux x86
$ GOOS=linux GOARCH=386 go build

# Compile for Linux x64
$ GOOS=linux GOARCH=amd64 go build

# Compile for macos x64
$ GOOS=darwin GOARCH=amd64 go build
```

## Partial Linux Support

Go-dispatch-proxy supports linux only if used as a transparent proxy (tunnel mode) to load balance SSH tunnels.

Go-dispatch-proxy can't load balance internet connections on linux. You can run this tool on linux but it will not function as expected, traffic will not be load balanced across the available interfaces. This is because of how linux works.

Go-dispatch-proxy works by specifying the IP address (binding) to be used for each outgoing connection. Unfortunately on linux **IP address binding != interface binding**. Linux uses the route which has the lowest metric inspite of specifying the source IP address. See [this](http://wiki.treck.com/Appendix_C:_Strong_End_System_Model_/_Weak_End_System_Model) for further information.

One workaround on linux is to use [`SO_BINDTODEVICE`](https://linux.die.net/man/7/socket) while creating the socket. However this reqires root to work and currently not implemented.

## Credits

- [dispatch-proxy](https://github.com/Morhaus/dispatch-proxy): A SOCKS5/HTTP load balancing proxy written in NodeJS.

## License

Licensed under MIT
