---
title: 【 Bash 】自用expect自动ssh登录脚本
date: 2009-11-16T18:25:00+00:00
layout: post
---
<pre class="brush: bash">#!/usr/bin/ssh

set arg0 [lindex $argv 0]
set passcode "prefix$arg0"

if {$passcode=="prefix"} {
send_user "argv0: key not found.n"
exit
}

spawn ssh -l chengang2 192.168.1.1
expect "*password:"
send "mypasswdr"
expect "*PASSCODE:"
send "$passcoder"
expect "server*"
send "192.168.10.1r"
expect "*chengang2*"
send "sudo su -r"
interact
</pre>
