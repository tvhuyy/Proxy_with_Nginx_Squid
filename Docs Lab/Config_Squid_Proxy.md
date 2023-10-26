## Cấu hình Forward Proxy sử dụng Squid

### Cài đặt Squid

- Load repo và cài đặt package Squid :
    ```
    sudo apt-get update
    sudo apt-get install squid
    ```

- Khởi động `Squid` và cho phép service khởi động cùng hệ thống :
    ```
    sudo systemctl start squid
    sudo systemctl enable squid
    ```

- Kiểm tra trạng thái service của `Squid` :
    ```
    sudo systemctl status squid
    ```

### Cấu hình Squid Proxy

- Cần quan tâm một số file đặc biệt sau :
    - File config của squid : `/etc/squid/squid.conf`
    - File Access log của squid : `/var/log/squid/access.log`

- File config của squid chứa rất nhiều chú thích, ta chỉ cần quan tâm vào đoạn chính như dưới đây :
    ```
    # Recommended minimum configuration:
    #

    # Example rule allowing access from your local networks.
    # Adapt to list your (internal) IP networks from where browsing
    # should be allowed
    acl localnet src 0.0.0.1-0.255.255.255  # RFC 1122 "this" network (LAN)
    acl localnet src 10.0.0.0/8             # RFC 1918 local private network (LAN)
    acl localnet src 100.64.0.0/10          # RFC 6598 shared address space (CGN)
    acl localnet src 169.254.0.0/16         # RFC 3927 link-local (directly plugged) machines
    acl localnet src 172.16.0.0/12          # RFC 1918 local private network (LAN)
    acl localnet src 192.168.0.0/16         # RFC 1918 local private network (LAN)
    acl localnet src fc00::/7               # RFC 4193 local private network range
    acl localnet src fe80::/10              # RFC 4291 link-local (directly plugged) machines

    acl SSL_ports port 443
    acl Safe_ports port 80          # http
    acl Safe_ports port 21          # ftp
    acl Safe_ports port 443         # https
    acl Safe_ports port 70          # gopher
    acl Safe_ports port 210         # wais
    acl Safe_ports port 1025-65535  # unregistered ports
    acl Safe_ports port 280         # http-mgmt
    acl Safe_ports port 488         # gss-http
    acl Safe_ports port 591         # filemaker
    acl Safe_ports port 777         # multiling http
    acl CONNECT method CONNECT
    ```

- Theo mặc định thì nó đã có sẵn các dải mạng local và port thường dùng trong các access list `localnet` và `Safe_ports`.
- Nếu địa chỉ IP mà ta cho phép đi qua proxy server chưa có thì có thể thêm vào với cấu trúc :
    ```
    acl localnet src x.x.x.x/x
    ```

- Tương tự với các port mà ta cho phép chưa có trong danh sách :
    ```
    acl Safe_ports port x
    ```

- Mặc định thì bên nó chưa allow cho acl localnet bên trên, ta cần thêm dòng `http_access allow localnet` để cho phép các dải mạng đó.
- Cũng theo mặc định thì port của squid proxy sẽ là `3128` ta có thể tùy chỉnh nó tại dòng `http_port 3128`, ở mô hình này ta đổi thành port `9999`.
- Khởi động lại `Squid` service để áp dụng những thay đổi :
    ```
    systemctl restart squid
    ```

- Kiểm tra bằng cách sử dụng trình duyệt trên máy client và xem log request đến trên Nginx Proxy server. Trên máy Client ta cần thêm proxy :

    ![a](https://imgur.com/Mr6jaCr.png)

- Log trên Nginx server :
    ```
    root@Nginx-Node-1:~# tail -f /var/log/nginx/access.log
    10.88.88.23 - - [26/Oct/2023:06:46:59 +0000] "GET /host82 HTTP/1.1" 301 178 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/116.0"
    10.88.88.23 - - [26/Oct/2023:06:47:03 +0000] "GET /host82 HTTP/1.1" 200 155 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/116.0"
    10.88.88.23 - - [26/Oct/2023:06:47:04 +0000] "GET /favicon.ico HTTP/1.1" 404 134 "https://web.com/host82" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/116.0"
    10.88.88.23 - - [26/Oct/2023:06:51:16 +0000] "GET /host81 HTTP/1.1" 301 178 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/116.0"
    10.88.88.23 - - [26/Oct/2023:06:51:22 +0000] "GET /host81 HTTP/1.1" 200 155 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/116.0"
    10.88.88.23 - - [26/Oct/2023:06:51:22 +0000] "GET /favicon.ico HTTP/1.1" 404 134 "https://web.com/host81" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/116.0"
    10.88.88.23 - - [26/Oct/2023:06:59:13 +0000] "GET /host81 HTTP/1.1" 200 156 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/116.0"
    10.88.88.23 - - [26/Oct/2023:07:01:54 +0000] "GET /host81 HTTP/1.1" 200 155 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/116.0"
    10.88.88.23 - - [26/Oct/2023:07:02:30 +0000] "GET /host81 HTTP/1.1" 200 156 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/116.0"
    10.88.88.23 - - [26/Oct/2023:07:02:37 +0000] "GET /host81 HTTP/1.1" 200 155 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/116.0"
    ```

- Nginx server chỉ bắt được các kết nối đến từ địa chỉ IP của Squid Server, không biết được client nào.

### Cấu hình block website với Squid Proxy

- Tạo một file acl chứa các web trong danh sách đen :
    ```
    vi /etc/squid/blacklist_site.acl
    ```

- Thêm domain cần block :
    ```
    web.com
    ```

- Mở file config của squid và thêm 2 dòng sau :
    ```
    acl blacklist_url dstdomain "/etc/squid/blacklist_site.acl"
    http_access deny blacklist_url
    ```

- Khởi động lại squid và kiểm tra :

    ![a](https://imgur.com/vXJ1eFr.png)

- Ta có thể truy cập bất kì trang web nào trừ `web.com` đã nằm trong blacklist.
- Check log access của Squid để thấy rõ hơn `tail /var/log/squid/access.log` :
    ```
    1698305346.559      0 10.22.22.2 TCP_DENIED/403 3985 CONNECT web.com:443 - HIER_NONE/- text/html
    1698305398.748 170364 10.22.22.2 TCP_TUNNEL/200 7869 CONNECT youtube.com:443 - HIER_DIRECT/172.217.31.14 -
    ```
