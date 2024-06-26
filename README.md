# Traffic Forward

[![Static Badge](https://img.shields.io/badge/traffic_forward-v0.1.8-blue)](https://crates.io/crates/traffic_forward)

This is a tool that allows you to easily create traffic forwarding rules through this command, which is implemented through the iptables tool.

## What is this

It create TCP/UDP rules for three chains:

```bash
# Chians: PREROUTING、POSTROUTING、FORWARD
sudo traffic_forward add 193.23.11.3:3333 9988
```

Equivalent to:

```bash
# 192.168.17.131 is the server host's IP address, automatically fetched.
iptables -t nat -I PREROUTING -p tcp --dport 9988 -j DNAT --to-destination 193.23.11.3:3333
iptables -t nat -I POSTROUTING -d 193.23.11.3 -p tcp --dport 3333 -j SNAT --to-source 192.168.17.131
iptables -t filter -I FORWARD -d 193.23.11.3 -p tcp --dport 3333
iptables -t filter -I FORWARD -s 193.23.11.3 -p tcp --dport 3333
iptables -t nat -I PREROUTING -p udp --dport 9988 -j DNAT --to-destination 193.23.11.3:3333
iptables -t nat -I POSTROUTING -d 193.23.11.3 -p udp --dport 3333 -j SNAT --to-source 192.168.17.131
iptables -t filter -I FORWARD -d 193.23.11.3 -p udp --dport 3333
iptables -t filter -I FORWARD -s 193.23.11.3 -p udp --dport 3333
```

## Prerequisite

* Must be a Linux operating environment.
* Iptables is already installed (most systems will default to installation).
* The server has enabled traffic forwarding.【95% of people did not take this step】
* The server already has a `Rust` environment.

## How to enable server to enable traffic forwarding

> Just for your reference, the ideas are the same

``` shell
# A value of 0 indicates that the forwarding function is not activated, while a value of 1 means it is activated.
cat /proc/sys/net/ipv4/ip_forward

# Temporarily enable forwarding function
echo 1 > /proc/sys/net/ipv4/ip_forward
```

## illustrate

This tool only replaces writing `iptables` rules and solves multiple configurations with one command. As `iptables` is used, `root` account operations are required.

## Installation

```bash
cargo install traffic_forward
```

Upon successful installation, you will see the following path:

```bash
...
 Compiling clap v4.5.4
   Compiling ron v0.8.1
   Compiling traffic_forward v0.1.0
    Finished release [optimized] target(s) in 2m 17s
  Installing /home/youre_account/.cargo/bin/traffic_forward
   Installed package `traffic_forward v0.1.0` (executable `traffic_forward`)
```

【Non-root Account】 Add a soft link to `/usr/bin`：

``` bash
# This is only for your reference, please operate according to your actual path
sudo ln -s /home/youre_account/.cargo/bin/traffic_forward /usr/bin/traffic_forward
```

View version:

``` bash
traffic_forward --version
traffic_forward X.X.X
```

Or:

``` bash
sudo traffic_forward --version
traffic_forward X.X.X
```

---

## Usage

* Add forwarding：

```bash
# Use local port 5555 to forward to 192.102.11.44:8000
sudo traffic_forward add 192.102.11.44:8000 5555
```

* List of existing rules:

``` bash
sudo traffic_forward list
0.0.0.0:5555 -> 192.102.11.44:8000
```

* Query traffic consumption for a specific IP:

``` bash
sudo traffic_forward query 192.102.11.44
```

* Delete forwarding rule for a specific IP:

``` bash
sudo traffic_forward delete 192.102.11.44
Delete completed
```

* Enable the wep interface:

``` bash
# traffic_forward web <token> [port]
 sudo traffic_forward 1234 8080
```

---

## Enable the wep interface

```bash
# traffic_forward web <token> [port]
traffic_forward web 1234 8080
```

Add

* uri: iptables/add
* method: post
* request

``` bash
{
 "target_ip": "192.168.50.50",
 "target_port": "4488",
 "local_port": "4433",
}
```

* response

``` bash
{
 "code": 1,
 "msg": "Success",
 "data": null
}
```

Delete

* uri: iptables/del
* method: post
* request

``` json
{
 "target_ip": "192.168.50.50"
}
```

* response

``` json
{
 "code": 1,
 "msg": "Success",
 "data": null
}
```

List

* uri: iptables/list
* method: get
* request: empty (change next version will)
* response

``` json
{
 "code": 1,
 "msg": "Success",
 "data": {
  "list": [
   {
    "ip": "192.168.50.50",
    "target_port": "4488",
    "local_port": "4433"
   }
  ]
 }
}
```

## Usage examples

``` bash
$ sudo traffic_forward -V
traffic_forward 0.1.2

$ sudo traffic_forward add 192.168.11.11:3389 2233
Add completed

$ sudo traffic_forward list
0.0.0.0:5000 -> 192.145.2.22:323
0.0.0.0:2233 -> 192.168.11.11:3389

$ sudo traffic_forward query 192.168.11.11
Up: 0 KB 
Down: 0 KB

$ sudo traffic_forward delete 192.168.11.11
Delete completed

$ sudo traffic_forward query 192.168.11.11
Query error: No matching IP found

$ sudo traffic_forward list
0.0.0.0:5000 -> 192.145.2.22:323

# When use API 
sudo ./traffic_forward web 1234 8080

# Request test
curl -H"token: 1234" localhost:8080/iptables/list
{"code":1,"msg":"Success","data":{"list":[{"ip":"xxx.xx.xx.xx","target_port":"12581","local_port":"49204"}]}}
```

## License

MIT OR Apache-2.0
