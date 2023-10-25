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
