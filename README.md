# pknginxhttpserv4ed

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



## 3. HTTP Configuration
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
#### types_hash_max_size
#### types_hash_bucket_size


### Limits and restrictions
#### limit_except
```
location /admin/ { 
    limit_except GET { 
      allow 192.168.1.0/24; 
      deny all; 
    } 
} 
```
#### limit_rate
```
limit_rate 500k;
```
This will limit connection transfer rates to 500 kilobytes per second. If a client opens two connections, the client will be allowed 2 mul 500 kilobytes.

#### limit_rate_after
Defines the number of data transferred before the limit_rate directive takes effect.

#### satisfy
context: location

#### internal
The specified resource cannot be accessed by external requests.

### File processing and caching
#### disable_symlinks
#### directio
#### directio_alignment
#### open_file_cache
#### open_file_cache_errors

#### read_ahead

### Other directives
#### log_not_found
#### log_subrequest
#### merge_slashes
By default, if the client attempts to access http://website.com//documents/ (note the // in the middle of the URI), Nginx will return a 404 Not Found HTTP error. If you enable this directive, the two slashes will be merged into one and the location pattern will be matched.

#### resolver
#### resolver_timeout

#### server_tokens
Context: ```http```, ```server```, and ```location```  
hide nginx versions

#### underscores_in_headers
#### variables_hash_max_size
#### variables_hash_bucket_size
#### post_action
Context: ```http```, ```server```, and ```location``` ```if```  
Defines a post-completion action, a URI that will be called by Nginx after the request has been completed.

### Using HTTP/2
If install nginx via package this module is most likely enabled.
If compiled yourself, make sure you compiled Nginx using the ```--with_http_v2_module```  configure flag.
```
listen 443 ssl http2;
```
### http2_chunk_size

Sets the maximum size of chunks into which the response body is sliced.
Default value: 8k

### http2_body_preread_size

### http2_idle_timeout





### Request headers
```
$http_host
$http_user_agent
$http_referer
$http_x_forwarded_for
```
### Response headers
```
$sent_http_content_type
$sent_http_content_length
```

### Nginx generated

(need update)

### The location block
#### The ~ modifier
With operating systems such as Microsoft Windows, ~ and ~* are both case-insensitive, as the OS uses a case-insensitive filesystem.
#### The ~* modifier
The requested URI must be a case-insensitive match for the specified regular expression:
#### The ^~ modifier
Similar to the no-symbol behavior, the location URI must begin with the specified pattern. The difference is that, if the pattern is matched, Nginx stops searching for other patterns (read the Search order and priority section).

#### Search order and priority
Order:
- location blocks with the = modifier: If the specified string exactly matches the requested URI, Nginx retains the location block
- location blocks with no modifier: If the specified string exactly matches the requested URI, Nginx retains the location block
- location blocks with the ^~ modifier: If the specified string matches the beginning of the requested URI, Nginx retains the location block
- location blocks with the ~ or ~* modifier: If the regular expression matches the requested URI, Nginx retains the location block
- location blocks with no modifier: If the specified string matches the beginning of the requested URI, Nginx retains the location block


## Module Configuration
### Rewrite module

### Quantifiers
### Captures

#### error_page
