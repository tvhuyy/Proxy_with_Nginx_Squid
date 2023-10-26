## Cấu hình reverse proxy sử dụng Nginx

### Cài đặt Nginx

- Load repo và cài đặt package Nginx :
    ```
    sudo apt-get update
    sudo apt-get install nginx
    ```

### Vô hiệu hóa Default Virtual Host

- Xóa sym-link của virtual host default :
    ```
    sudo unlink /etc/nginx/sites-enabled/default
    ```

### Tạo Nginx Reverse Proxy

- Tạo một file conf mới để lưu thông tin của reverse proxy :
    ```
    vi /etc/nginx/sites-available/reverse-proxy.conf
    ```

- Thêm nội dung cho file như sau :
    ```
    server {
        listen       80;
        server_name  web.com;

        location /host81 {
            rewrite ^/host81(.*) /$1 break;
            proxy_pass http://10.11.11.2:81;
        }

        location /host82 {
            rewrite ^/host82(.*) /$1 break;
            proxy_pass http://10.11.11.2:82;
        }
    }
    ```

- Lưu ý: vì làm trên local nên cần thêm domain name vào file host :
    ```
    echo 10.88.88.21 web.com >> /etc/hosts
    ```

- Tạo sym-link tới `/etc/nginx/sites-enabled/` để nginx có thể đọc và thực thi file cấu hình :
    ```
    ln -s /etc/nginx/sites-available/reverse-proxy.conf /etc/nginx/sites-enabled/
    ```

- Kiểm tra cấu hình nginx sau thay đổi :
    ```
    root@Nginx-Node-1:~# nginx -t
    nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    nginx: configuration file /etc/nginx/nginx.conf test is successful
    ```

- Khởi động lại dịch vụ và kiểm tra trạng thái :
    ```
    root@Nginx-Node-1:~# systemctl restart nginx
    root@Nginx-Node-1:~#
    root@Nginx-Node-1:~# systemctl status nginx
    ● nginx.service - A high performance web server and a reverse proxy server
        Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
        Active: active (running) since Wed 2023-10-25 09:49:43 UTC; 1min 16s ago
        Docs: man:nginx(8)
        Process: 15238 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
        Process: 15250 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Main PID: 15257 (nginx)
        Tasks: 2 (limit: 2220)
        Memory: 2.7M
        CGroup: /system.slice/nginx.service
                ├─15257 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
                └─15258 nginx: worker process

    Oct 25 09:49:42 Nginx-Node-1 systemd[1]: Starting A high performance web server and a reverse proxy server...
    Oct 25 09:49:43 Nginx-Node-1 systemd[1]: Started A high performance web server and a reverse proxy server.
    ```

- Truy cập Apache web server thông qua Nginx Proxy :
    ```
    root@Squid-Proxy:~# curl http://web.com/host81
    <html>
    <body>
    <div style="width: 100%; font-size: 40px; font-weight: bold; text-align: center;">
    Day la web port 81 tren Apache Server Node 1
    </div>
    </body>
    </html>
    root@Squid-Proxy:~# curl http://web.com/host82
    <html>
    <body>
    <div style="width: 100%; font-size: 40px; font-weight: bold; text-align: center;">
    Day la web port 82 tren Apache Server Node 1
    </div>
    </body>
    </html>
    ```

### Cấu hình Nginx Reverse Proxy Load Balancing

- Trong mô hình này sử dụng Load Balancing sử dụng thuật toán `Round Robin` - cho phép các request được phân phối tuần tự trong 1 nhóm server. Ngoài ra có thể sử dụng các thuật toán khác như :
    - `Weighted Round Robin` : Tương tự như Round Robin nhưng thứ tự ưu tiên cho các server sẽ do người quản trị thiết lập dựa trên khả năng xử lý request của từng server.
    - `Least Connections` : Request sẽ được gửi đến server đang có ít kết nối nhất.
    - `Weighted Least Connections` : Request sẽ được gửi đến server có tốc độ phản hồi response cao nhất và ít kết nối nhất.
    - `IP Hash` : Địa chỉ IP của client sẽ được sử dụng để nhận biết server nào sẽ nhận được request từ client.

- Xóa sym-link đến file `/etc/nginx/sites-available/reverse-proxy.conf` ở bên trên.
    ```
    sudo unlink /etc/nginx/sites-enabled/reverse-proxy.conf
    ```

- Tạo file mới chứa cấu hình load balancing và reverse proxy :
    ```
    sudo vi /etc/nginx/sites-available/load-balancer.conf
    ```

- Thêm vào nội dung tương tự bên dưới :
    ```
    upstream host81 {
            server 10.11.11.2:81;
            server 10.11.11.3:81;
            }
    upstream host82 {
            server 10.11.11.2:82;
            server 10.11.11.3:82;
            }

    server {
            listen 80;

            server_name web.com;

            location /host81 {
                    rewrite ^/host81(.*) /$1 break;
                    proxy_pass http://host81;
            }

            location /host82 {
                    rewrite ^/host82(.*) /$1 break;
                    proxy_pass http://host82;
            }

    }
    ```

- Tạo sym-link tới `/etc/nginx/sites-enabled/` để nginx có thể đọc và thực thi file cấu hình :
    ```
    sudo ln -s /etc/nginx/sites-available/load-balancer.conf /etc/nginx/sites-enabled/
    ```
- Kiểm tra cấu hình nginx sau thay đổi :
    ```
    root@Nginx-Node-1:~# nginx -t
    nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    nginx: configuration file /etc/nginx/nginx.conf test is successful
    ```

- Khởi động lại dịch vụ và kiểm tra trạng thái :
    ```
    tvhuyy@Nginx-Node-1:~$ sudo systemctl restart nginx
    tvhuyy@Nginx-Node-1:~$ sudo systemctl status nginx
    ● nginx.service - A high performance web server and a reverse proxy server
        Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
        Active: active (running) since Thu 2023-10-26 01:55:22 UTC; 9s ago
        Docs: man:nginx(8)
        Process: 2770 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
        Process: 2778 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Main PID: 2783 (nginx)
        Tasks: 2 (limit: 2220)
        Memory: 2.8M
        CGroup: /system.slice/nginx.service
                ├─2783 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
                └─2784 nginx: worker process

    Oct 26 01:55:22 Nginx-Node-1 systemd[1]: nginx.service: Succeeded.
    Oct 26 01:55:22 Nginx-Node-1 systemd[1]: Stopped A high performance web server and a reverse proxy server.
    Oct 26 01:55:22 Nginx-Node-1 systemd[1]: Starting A high performance web server and a reverse proxy server...
    Oct 26 01:55:22 Nginx-Node-1 systemd[1]: Started A high performance web server and a reverse proxy server.
    ```

- Trên máy client, sử dụng câu lệnh sau để kiểm tra xem load balacing đã hoạt động đúng hay chưa :
    ```
    while sleep 0.5; do curl web.com/host81; done
    ```

    ```
    tvhuyy@Squid-Proxy:~$ while sleep 0.5; do curl web.com/host82; done
    <html>
    <body>
    <div style="width: 100%; font-size: 40px; font-weight: bold; text-align: center;">
    Day la web port 82 tren Apache Server Node 1
    </div>
    </body>
    </html>
    <html>
    <body>
    <div style="width: 100%; font-size: 40px; font-weight: bold; text-align: center;">
    Day la web port 82 tren Apache Server Node 2
    </div>
    </body>
    </html>

    <html>
    <body>
    <div style="width: 100%; font-size: 40px; font-weight: bold; text-align: center;">
    Day la web port 82 tren Apache Server Node 1
    </div>
    </body>
    </html>
    <html>
    <body>
    <div style="width: 100%; font-size: 40px; font-weight: bold; text-align: center;">
    Day la web port 82 tren Apache Server Node 2
    </div>
    </body>
    </html>
    ```

### Cấu hình HTTPS và tạo SSL CA cho Local

- Thực tế ta có thể mua cert hoặc tạo cert với công cụ `python3-certbot-nginx` cho domain của mình. Trong mô hình local này thì ta sẽ phải tạo SSL cho domain local.

#### Tạo Certificate Authority

- Đầu tiên ta cần tạo `Certificate Authority` để cung cấp chứng chỉ cho domain.
- Tạo private key :
    ```
    openssl genrsa -des3 -out rootCA.key 2048
    ```

- Nhập pass phase :
    ```
    Generating RSA private key, 2048 bit long modulus (2 primes)
    ..+++++
    ................................+++++
    e is 65537 (0x010001)
    Enter pass phrase for rootCA.key:
    Verifying - Enter pass phrase for rootCA.key:
    ```

- Sau khi đã có private key, ta tạo root certificate :
    ```
    openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1825 -out rootCA.pem
    ```

- Lệnh này sẽ cần nhập pass phase ta đã tạo ở bước trên, nhập thêm 1 số thông tin được yêu cầu :
    ```
    root@Nginx-Node-1:~# openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1825 -out rootCA.pem
    Enter pass phrase for rootCA.key:
    You are about to be asked to enter information that will be incorporated
    into your certificate request.
    What you are about to enter is what is called a Distinguished Name or a DN.
    There are quite a few fields but you can leave some blank
    For some fields there will be a default value,
    If you enter '.', the field will be left blank.
    -----
    Country Name (2 letter code) [AU]:VN
    State or Province Name (full name) [Some-State]:Ha Noi
    Locality Name (eg, city) []:Ha Noi
    Organization Name (eg, company) [Internet Widgits Pty Ltd]:TVHsvr
    Organizational Unit Name (eg, section) []:Information Technology
    Common Name (e.g. server FQDN or YOUR name) []:TVHsvr
    Email Address []:tvhuyy@gmail.com
    ```

- Sau khi xong 2 bước trên, ta sẽ có 2 file là `rootCA.key` và `rootCA.pem`. Bây giờ ta có thể tự tạo SSL.

#### Tạo HTTPS cho local site

- Tạo một private key cho domain local - mô hình này sử dụng domain là `web.com` :
    ```
    openssl genrsa -out web.com.key 2048
    ```

- Tiếp theo, tạo CSR (Certificate Signing Request) :
    ```
    openssl req -new -key web.com.key -out web.com.csr
    ```

- Nhập các thông tin tương tự như làm ở trên, phần `A challegen password` và `An optional company name` có thể bỏ trống.
    ```
    root@Nginx-Node-1:~# openssl req -new -key web.com.key -out web.com.csr
    You are about to be asked to enter information that will be incorporated
    into your certificate request.
    What you are about to enter is what is called a Distinguished Name or a DN.
    There are quite a few fields but you can leave some blank
    For some fields there will be a default value,
    If you enter '.', the field will be left blank.
    -----
    Country Name (2 letter code) [AU]:VN
    State or Province Name (full name) [Some-State]:Ha Noi
    Locality Name (eg, city) []:Ha Noi
    Organization Name (eg, company) [Internet Widgits Pty Ltd]:TVHsvr
    Organizational Unit Name (eg, section) []:Information Technology
    Common Name (e.g. server FQDN or YOUR name) []:TVHsvr
    Email Address []:tvhuyy@gmail.com

    Please enter the following 'extra' attributes
    to be sent with your certificate request
    A challenge password []:
    An optional company name []:
    ```

- Tiếp theo, tạo file config để định nghĩa Subject Alternative Name (SAN) cho SSL vừa tạo :
    ```
    vi web.com.ext
    ```

- Thêm nội dung tương tự bên dưới, Ta có thể thêm các domain khác `DNS.x = <Domain Name>` :
    ```
    authorityKeyIdentifier=keyid,issuer
    basicConstraints=CA:FALSE
    keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
    subjectAltName = @alt_names

    [alt_names]
    DNS.1 = web.com
    ```

- Tiếp theo ta tạo cert cho domain :
    ```
    openssl x509 -req -in web.com.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out web.com.crt -days 1825 -sha256 -extfile web.com.ext
    ```

- Nhập pass phase của `rootCA.pem`.
- Bây giờ ta sẽ có các file sau :
    - `web.com.key` : Private key
    - `web.com.csr` : Certificate Signing Request
    - `web.com.crt` : Signed certificate

#### Cấu hình HTTPS cho Nginx :

- Vào file `load-balancer.conf` đã tạo ở phần trên và sửa thêm như dưới đây :
    ```
    upstream host81 {
            server 10.11.11.2:81;
            server 10.11.11.3:81;
            }
    upstream host82 {
            server 10.11.11.2:82;
            server 10.11.11.3:82;
            }

    server {
            listen 80;
            return 301 https://$host$request_uri;
    }

    server {
            listen 443 ssl;

            server_name web.com;

            ssl_certificate /root/web.com.crt;
            ssl_certificate_key /root/web.com.key;
            ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
            ssl_ciphers HIGH:!aNULL:!MD5;

            location /host81 {
                    rewrite ^/host81(.*) /$1 break;
                    proxy_pass http://host81;
            }

            location /host82 {
                    rewrite ^/host82(.*) /$1 break;
                    proxy_pass http://host82;
            }

    }

    ```

- Trong đó : đoạn bên dưới cho phép server tự động redirect request của client qua sử dụng https, điều này sẽ khiến cho client sẽ luôn sử dụng giao thức https khi kết nối đến site.
    ```
    server {
            listen 80;
            return 301 https://$host$request_uri;
    }
    ```

- Kiểm tra cấu hình Nginx sau thay đổi và khởi động lại dịch vụ :
- Kiểm tra trên máy client :

    ![a](https://imgur.com/7TqdfRb.png)