### Cấu hình web server sử dụng Apache2

- Trong bài này ta sẽ sử dụng `Apache` để dựng web server cơ bản (vì nó đơn giản dễ cấu hình :>).
- Mô hình này có 2 Node Apache để cân tải, cấu hình giống nhau chỉ khác IP. Có thể sử dụng 2 docker container sẽ nhẹ nhàng hơn.

#### Cài đặt web server Apache :

- Ta có thể cài trực tiếp từ repo của Ubuntu. Đầu tiên ta cần load lại các thay đổi mới nhất trên config repo của bản phân phối :
    ```
    sudo apt-get update
    ```

- Cài đặt package `apache2`
    ```
    sudo apt-get install apache2
    ```

#### Kiểm tra cấu hình của firewall

- Thực tế thì ta cần phải allow service và allow port để có thể sử dụng dịch vụ nhưng trường hợp này thì mình lược bớt nên disable firewall.
- `ufw`    
    ```
    sudo ufw disable
    ```

- Firewalld
    ```    
    sudo systemctl stop firewalld
    sudo systemctl disable firewalld
    ```

- Selinux
    ```
    vi /etc/sysconfig/selinux
    SELINUX=disabled
    ```

#### Kiểm tra trạng thái web server

- `sudo systemctl status apache2` :

    ```
    ● apache2.service - The Apache HTTP Server
        Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor preset: enabled)
        Active: active (running) since Wed 2023-10-25 14:01:21 +07; 16min ago
        Docs: https://httpd.apache.org/docs/2.4/
        Process: 553 ExecStart=/usr/sbin/apachectl start (code=exited, status=0/SUCCESS)
        Main PID: 598 (apache2)
        Tasks: 55 (limit: 1010)
        Memory: 7.3M
            CPU: 313ms
        CGroup: /system.slice/apache2.service
                ├─598 /usr/sbin/apache2 -k start
                ├─600 /usr/sbin/apache2 -k start
                └─601 /usr/sbin/apache2 -k start

    Thg 10 25 14:01:19 Apache-Node-1 systemd[1]: Starting The Apache HTTP Server...
    Thg 10 25 14:01:21 Apache-Node-1 systemd[1]: Started The Apache HTTP Server.
    ```

#### Thiết lập Virtual Host

- Ta sẽ tạo 2 virtual host giống nhau trên mỗi Node.
- Tạo thư mục chứa index :
    ```
    sudo mkdir /var/www/apache-web-81 /var/www/apache-web-82
    ```

- Phân quyền cho thư mục đảm bảo cho phép chủ sở hữu quyền đọc, ghi, thực thi và quyền đọc, thực thi đối với group và những user khác :
    ```
    sudo chmod -R 755 /var/www/apache-web-81 /var/www/apache-web-82
    ```

- Tạo file index.html :
    ```
    sudo vi /var/www/apache-web-81/index.html
    ```

- Bên trong file thêm đoạn HTML :
    ```
    <html>
    <body>
    <div style="width: 100%; font-size: 40px; font-weight: bold; text-align: center;">
    Day la web port 81 tren Apache Server Node 1
    </div>
    </body>
    </html>
    ```

- Lưu file lại. Làm tương tự với virtual host còn lại và trên node còn lại.
- Để tách biệt virtual host cho phép apache directive chính xác, ta nên tạo một file conf mới trong `/etc/apache2/sites-available/` :
    ```
    sudo vi /etc/apache2/sites-available/apache-web.conf
    ```

- Thêm vào đoạn code như dưới đây :
    ```
    <VirtualHost 10.11.11.2:81>
            DocumentRoot /var/www/apache-web-81
            ServerName 10.11.11.2
            ServerAdmin admin@10.11.11.2
            ErrorLog /var/log/apache2/apacheweb.error.log
            CustomLog /var/log/apache2/apacheweb.access.log combined
    </VirtualHost>

    <VirtualHost 10.11.11.2:82>
            DocumentRoot /var/www/apache-web-82
            ServerName 10.11.11.2
            ServerAdmin admin@10.11.11.2
            ErrorLog /var/log/apache2/apacheweb.error.log
            CustomLog /var/log/apache2/apacheweb.access.log combined

    </VirtualHost>
    ```

- Trên Node 2 ta làm tương tự, chỉ thay đổi IP.
- Sử dụng công cụ `a2ensite` để enable file :
    ```
    sudo a2ensite apache-web.conf
    ```

- Tiếp theo ta tạo symlink đến thư mục `/etc/nginx/site-enabled` để enable virtual host (trên thực tế, có thể tạo file trực tiếp vào thư mục này mà không cần phải tạo vào site-available, nhưng khuyến khích không nên làm trực tiếp như thế. Để lúc mình muốn disable virtual host chỉ cần xóa symlink là xong mà không hề ảnh hưởng đến file virtual host gốc) :

    ```
    sudo ln -s /etc/apache2/sites-available/apache-web.conf /etc/apache2/sites-enabled/
    ```

- Kiểm tra lỗi cấu hình nếu có :
    ```
    tvhuyy@Apache-Node-1:~$ sudo apache2ctl configtest
    Syntax OK
    ```

- Restart lại Apache để áp dụng các thay đổi :
    ```
    sudo systemctl restart apache2
    ```

- Ta có thể truy cập thử để kiểm tra :
    ```
    tvhuyy@Apache-Node-1:~$ curl http://10.11.11.2:81
    <html>
    <body>
    <div style="width: 100%; font-size: 40px; font-weight: bold; text-align: center;">
    Day la web port 81 tren Apache Server Node 1
    </div>
    </body>
    </html>
    tvhuyy@Apache-Node-1:~$ curl http://10.11.11.2:82
    <html>
    <body>
    <div style="width: 100%; font-size: 40px; font-weight: bold; text-align: center;">
    Day la web port 82 tren Apache Server Node 1
    </div>
    </body>
    </html>
    ```


### Tài liệu tham khảo :

[VietNix](https://vietnix.vn/cach-cai-dat-web-server-apache-tren-ubuntu-20-04/)