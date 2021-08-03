---
published: false
title: What I have learned this week
---

# Linux
## Install Cyrus IMAP server with Postfix
- do not use sendmail, there are some issues with it (especially with virtual domains) which are hard to debug
- postfix is simpler to configure and debug

## Show active listening ports
- `netstat planet`
  - `p` - show PID and name of program
  - `l` - show only listening sockets
  - `a` - all
  - `n` - show numerical address 
  - `e` - show extended information
  - `t` - TCP

## Capture packets 
- `tcpdump`
- <https://danielmiessler.com/study/tcpdump/#host>
