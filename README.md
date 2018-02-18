# pknginxhttpserv

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



## HTTP Configuration
### HTTP core module
### Module directives
#### server_name
When Nginx receives an HTTP request, it matches the Host header of the request against all of the server blocks. The first server block to match this hostname is selected.
If no server block matches the desired host, Nginx selects the first server block that matches the parameters of the listen directive

#### server_name_in_redirect
If set to on, Nginx will use the first hostname specified in the server_name directive.
#### server_names_hash_max_size
#### server_names_hash_bucket_size

#### tcp_nodelay
notes
- Note that this option only applies if the sendfile directive is enabled. 
- If tcp_nopush is set to on, Nginx will attempt to transmit the entire HTTP response headers in a single TCP packet.
- Default off  

#### sendfile
On Linux, using sendfile automatically disables asynchronous IO. 

#### sendfile_max_chunk
default 0

### Paths and documents
#### alias
Context: location. Variables are accepted.
```
alias /var/www/locked/;   # do not forget trailing /
```

#### error_page
```
error_page 404 =200 /index.html; # in case of 404 error, redirect to index.html with a 200 OK response code 
```

#### if_modified_since
#### index
#### recursive_error_pages

#### try_files
```
location / {     
  try_files $uri $uri.php @proxy; 
} 
# the following is a "named location block" 
location @proxy {     
  proxy_pass 127.0.0.1:8080; 
}
```
### Client requests
#### keepalive_requests
note
- Maximum number of requests served over a single keep-alive connection.
- default 100

#### keepalive_timeout
defines the number of seconds the server will wait before closing a keep-alive connection

#### keepalive_disable
#### send_timeout
default=60

#### client_body_in_file_only client_body_in_single_buffer

#### client_body_buffer_size
Specifies the size of the buffer holding the body of client requests. If this size is exceeded, the body (or at least part of it) will be written to the disk. Note that, if the client_body_in_file_only directive is enabled, request bodies are always stored to a file on the disk, regardless of their size (whether they fit in the buffer or not).

#### client_body_temp_path
```
client_body_temp_path temp 1 2 4; # Nginx will create 3 levels of folders (first level: 1 digit, second level: 2 digits, third level: 4 digits)
```
#### client_body_timeout 
#### client_header_buffer_size
#### client_header_timeout
#### client_max_body_size
#### large_client_header_buffers

#### lingering_time
This directive defines the number of time Nginx should wait after sending this error response before closing the connection.

#### lingering_timeout
This directive defines the number of time that Nginx should wait between two read operations before closing the client connection.

#### ignore_invalid_headers
Context: ```http```, and ```server```

#### chunked_transfer_encoding
### MIME types
#### types
