#### pknginxhttpserv

- controlling nginx service
```
nginx -s stop
nginx -s start
nginx -s reopen //reopen the log files
nginx -s reload //reloads the config
```
Kill all ps:
```
killall nginx
```
test validity:
```
/usr/local/nginx/sbin/nginx -t
```
generate new conf file:
```
/usr/local/nginx/sbin/nginx -t -c /home/user/test.conf
```

add additional params
```
nginx -g "timer_resolution 200ms"
```

- Adding nginx as as system service
runlevel:  
0/system is halted 1/single-user mode 2/multi-user mode(without NFS) 3/full multi-user mode 4/not used 5/gi mode 6/sys reboot
Ex
```
telinit 0 //shotdown
telinit 6 //reboot
```
service httpd start = /etc/init.d/httpd start

write a nginx script, move in init.d dir, and put it in system run level:
For debian:
```
update-rc.d -f nginx defaults
```
For redhat:
```
chkconfig nginx on
chkconfig --list nginx
```
Or use 'ntsysv' command.

##### Basic config

- time and size Units. Note: size=(G/g, M/m,K/k) time=(ms/s/m(minute)/h/d/w/M(month)/y)
```
client_max_body_size 2G;
client_body_timeout 180s;
client_body_timeout 1m30s;
client_body_timeout '1m 30s 500ms';
```
- variables (using $)
```
log_format main '$pid - $nginx_version - $remote_addr';
```
- Base module
include
if not specify an absolute path, the path should under conf folder. For examp, include sites/exmple.conf=
```
/usr/local/nginx/conf/sites/exmple.conf
```
- Testing server
httperf
```
httperf --server 192.168.1.10 --port 80 --uri /index.html --rate 300 --num-conn 3000 --num-call 1 --timeout 5
```
Upgrade nginx:
```
pid x|grep nginx|grep master (get pid)
kill -USR2 (12) 1234
kill -WINCh(28) 1234
kill -QUIT 1234
```
####case studies
create https server
```
openssl genrsa -out xx.com.key 2048
openssl req -new -key xxx.com.key -out xxx.com.csr
```
create oncloud
```
wget https://download.owncloud.org/community/owncloud-8.zip
unzip owncloud.zip
mv ./owncloud/{.[!.],}* ./
```
create crt:
```
openssl x509 -req -days 1000 -in xxx.csr -signkey xxx.key -out xxx.crt
```
Another command[ciick here](https://www.sslshopper.com/article-most-common-openssl-commands.html)
```
openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout privateKey.key -out certificate.crt
```
