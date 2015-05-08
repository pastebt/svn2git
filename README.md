# svn2git
backup working svn repository to remote git server, keeping commit message


## Init

We use svn project name "mip" as sample
First get all author mapping list
```bash
cd svn/project/trunk/mip
svn log --xml | grep author | sort -u
```
edit users.txt

then clone from svn
```bash
git svn clone http://your_svn_server/your_svnroot/project/trunk/mip --authors-file=users.txt --no-metadata
```

let say you want to keep backup at https://YourName@bitbucket.org/YourName/mip.git

```bash
git remote add origin  https://YourName@bitbucket.org/YourName/mip.git
git push -u origin --all
git push -u origin --tags
```

## Update

You have commit something into svn, now you want to backup to git server. 
First you run
```bash
svn up
svn diff
```
to make sure local workspace is up to date

get last revision of svn, sxxxxxx
```bash
svn info|grep "Last Changed Rev"|cut -d ' ' -f4
```

get last revision git have, gxxxxxx
```bash
tail -1 .git/logs/refs/remotes/git-svn | cut -f2 | tr -d 'r'
```

now reverse workspace to git point:
```bash
svn merge -r sxxxxx:gxxxxx .
```

You are ready to sycn local git master
```bash
git svn fetch
git rebase remotes/git-svn master
```

finally push to remote
```bash
git push -u origin
```

# simple systemd usage

Assume one service at /home/swl/svr.py

```python
#! /usr/bin/python

import os
import sys
import urllib2
from BaseHTTPServer import BaseHTTPRequestHandler, HTTPServer


class MyHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        if self.path == '/stop_server':
            self.server.running = False
        dat = "query " + self.path + "\n"
        self.send_response(200)
        self.send_header("Content-Length", len(dat))
        self.end_headers()
        self.wfile.write(dat)


def start(port):
    #if os.fork() > 0:
    #    sys.exit(0)
    print >> sys.stderr, "serving at port ", port
    httpd = HTTPServer(("", port), MyHandler)
    httpd.running = True
    while httpd.running:
        httpd.handle_request()


def stop(port):
    try:
        urllib2.urlopen("http://127.0.0.1:%d/stop_server" % port).read()
    except:
        pass


def usage():
    print 'Usage:', sys.argv[0], 'start|stop [port]'
    sys.exit(1)


def main():
    if len(sys.argv) not in (2, 3):
        usage()
    try:
        port = int(sys.argv[2])
    except:
        port = 8000

    if sys.argv[1] == 'start':
        start(port)
    elif sys.argv[1] == 'stop':
        stop(port)
    else:
        usage()


if __name__ == '__main__':
    main()
```

And we have service file mytstsvr.service :
```ini
[Unit]
Description=My Test Server
After=network.target

[Service]
#Type=forking   # if you add this, then your service should fork
ExecStart=/home/swl/svr.py start 8000
ExecStop=/home/swl/svr.py stop 8000

[Install]
WantedBy=multi-user.target
```

We can run command in /home/swl:
First link service into system path
```bash
sudo systemctl enable /home/swl/mytstsvr.service
```
then start it
```bash
sudo systemctl start mytstsvr
```
