## Tổng quan về Proxy

- Proxy là một ứng dụng hoặc máy chủ đóng vai trò trung gian giữa máy client yêu cầu tài nguyên và server cung cấp tài nguyên đó.
- Proxy được chia làm 2 loại : `Forward Proxy` và `Reverse proxy`.
- Về bản chất, cách thức hoạt động của cả 2 loại là gần như nhau, điều quan trọng để phân biệt chúng là hướng dữ liệu được truyền tải.

### Forward Proxy

- Forward proxy là một ứng dụng, server có nhiệm vụ trung chuyển data từ browser (client) đến web server (server).

#### Các tính năng của Forward Proxy

- **Allow/Deny request** : check các request có hợp lệ hay không theo các rule được đặt ra để quyết định forward hoặc block request này.
- **Log/Monitor request** : Theo dõi và logging request của client.
- **Cahce response** : Cache lại các request lặp lại thường xuyên để response lại cho client mà không cần resquest đến server, giúp giảm latency, tăng performence, tối ưu bandwidth,...
- **Anonymous** : Tính năng này thường xuyên được sử dụng, client sẽ kết nối đến server thông qua proxy server giúp cho server đích không thể biết được request này đến từ đâu. Thực tế nó thường được sử dụng để truy cập vào các web bị các tổ chức chặn hoặc lướt web một cách an toàn hơn.

### Reverse Proxy

- Hiểu đơn giản thì reverse proxy sẽ ngược lại với forward proxy. Nhận request từ clien hoặc forward proxy và gửi nó đến server.

    ![a](https://imgur.com/TwfwyxP.png)

- Với forward proxy thì server không thể biết request được gửi từ đâu, còn với reverse proxy thì client sẽ không biết request thực sự được gửi đến server nào.
- Mô hình khi kết hợp cả 2 loại proxy sẽ như sau :

    ![a](https://imgur.com/rF1qyP0.png)

#### Các tính năng chính của Reverse proxy

- **Load balancing** : Phân tán các request đến nhiều server khác nhau để giảm latency, cân tải phục vụ lượng request lớn,...
- **Security** : Có thể coi nó là `waf` (web application firewall) có nhiệm vụ chặn các truy cập khả nghi đến server. `Rate limit`, chặn DDOS một phần bằng cách giới hạn số lượng request lên server, giới hạn số lượng request đến từ 1 IP, chặn các IP chỉ định,...
- **Caching response** : tương tự forward proxy, khi có request đến có thể response lại luôn mà không cần gửi request đến server.
- **Port Forwarding** : Tiếp nhận request đến 1 IP duy nhất sau đó điều hướng đến đúng server chứa tài nguyên mà client đang request.
- **HTTPS** : Tạo cert SSL cho các web server thiếu xác thực, mã hóa kết nối giữa client và reverse proxy bằng TLS.


## Lab cấu hình Nginx Reverse Proxy và Squid Forward Proxy

- Tài liệu này sẽ hướng dẫn config các chức năng cơ bản của proxy như :
    - Nginx reverse proxy :
        - Proxy pass forward port.
        - Load balancing web server.
        - HTTPS.
    - Squid forward proxy :
        - Anonymous Client.
        - Allow/Deny request.
        - Log/Monitor request.

- Mô hình tổng thể :

    ![a](https://imgur.com/9Hqmmym.png)

- Với mô hình trên ta sẽ hoàn thành các yêu cầu :
    - Truy cập `web.com/host81` -> nội dung trên Apache server port 81.
    - Truy cập `web.com/host82` -> nội dung trên Apache server port 82.
    - Các lượt truy cập sẽ được đẩy luân phiên giữa 2 node Apache server 1 và 2.
    - Chuyển hướng truy cập của client từ http sang https.
    - Client truy cập được website thông qua Squid Proxy.
    - Cho phép Client truy cập đến `web.com/host81` và từ chối truy cập đến `web.com/host82`.
    - Proxy theo dõi được client nào truy cập đến đâu.

- Toàn bộ thiết bị trên bài lab sử dụng Ubuntu 20.04/22.04.
- Docs Lab các thành phần :

1. [Config Apache2](./Docs%20Lab/Config_Apache2.md)
2. [Config Nginx Proxy](./Docs%20Lab/Config_Nginx_Proxy.md)