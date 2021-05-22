---
title: Spawning a TTY Shell
author: Udesh
date: 2020-05-16
categories: [Pentesting]
---
## Spawning a TTY Shell - Break out of Jail or limited shell

You should almost always upgrade your shell after taking control of an apache or www user. you may encounter limited shells that use rbash and only allow you to execute a single command per session. you can overcome this by executing an SSH shell to your localhost.

**Python:**
```shell
python -c 'import pty; pty.spawn("/bin/sh")'
```
**Perl:**
```perl
perl -e 'exec "/bin/sh";'
```
**ruby:**
```ruby
ruby: exec "/bin/sh"
```
**lua:**
```lua
lua: os.execute('/bin/sh')
```

------------
Other command:

    echo os.system('/bin/bash')
	/bin/sh -i
