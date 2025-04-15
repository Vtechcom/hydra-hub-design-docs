# **Tổng quan**

Hydra Hub là giải pháp công nghệ kết nối giữa client (Consumer Client) và các node Hydra (Hydra Nodes) thông qua proxy (HydraHUB) được cung cấp bởi các Providers khác nhau.

Các nhà phát triển khi có nhu cầu sử dụng node hydra để triển khai ứng dụng chỉ cần đăng ký trở thành Consumer Client và sử dụng các dịch vụ Hydra từ các các Providers một cách đơn giản và nhanh chóng triển khai. Với chi phí rẻ hơn rất nhiều khi tự vận hành, bỏ qua những khó khăn khi phải thực hiện các công việc:

1. Đồng bộ hóa với Cardano Node phức tạp
2. Quản lý khóa và bảo mật thủ công
3. Giám sát và xử lý lỗi
4. Xử lý giao tiếp giữa các Hydra Node không ổn định

## **A- System Diagram của giải pháp Hydra HUB**

![Sơ đồ mô tả kiến trúc triển khai của hệ thống Hydra HUB, giải pháp kết nối giữa client (Consumer Client) và các node Hydra (Hydra Nodes) thông qua proxy.][image1]

**Hình 01**: Sơ đồ mô tả kiến trúc triển khai của hệ thống Hydra HUB, giải pháp kết nối giữa client (Consumer Client) và các node Hydra (Hydra Nodes) thông qua proxy.

**Diễn giải**:

1. **Consumer Client**:
    * DApps (UI): Ứng dụng phi tập trung (Decentralized Apps) mà người dùng (Consumer) sử dụng.
    * Giao tiếp với Hydra HUB Service qua WS/HTTP với API key.
    * Ví dụ: Gửi yêu cầu GET [https://api-key-cb3ec.hydranode.io/commits](https://api-key-cb3ec.hydranode.io/commits).
2. **Hydra HUB**:
    * **Hydra HUB Service**: Dịch vụ trung tâm quản lý:
        1. API keys manager: Quản lý API key.
        2. Consumer Portal (UI): Giao diện cho người dùng quản lý thông tin.
        3. Provider Info: Thông tin về các nhà cung cấp (Providers).
        4. Proxy mappers: Ánh xạ yêu cầu tới các Hydra Node.
    * **Hydra Node Proxy**: Proxy trung gian, giao tiếp với Hydra Nodes qua WS/HTTP
    * Giao tiếp với Consumer Client qua [Socket.io](http://Socket.io) hoặc HTTP
3. **Providers**:
    * Provider 1 (123.2.12.3) và Provider 2 (345.2.12.3): Các cụm máy chủ hoặc nhà cung cấp dịch vụ.
    * Mỗi provider chứa:
        1. Hexcore Dashboard UI: Giao diện quản trị.
        2. Cardano Node: Node của blockchain Cardano.
        3. Hydra Nodes: Các node Hydra (10001, 10002, v.v.), có thể chạy trên Docker hoặc Kubernetes (K8s).
    * Giao tiếp với Hydra HUB qua WS/HTTP.
4. **Hydra Nodes**:
    * Các node con (10001, 10002, v.v.) xử lý yêu cầu từ Hydra HUB Proxy.
    * Có thể chạy trên Docker hoặc Kubernetes, phục vụ các tác vụ liên quan đến blockchain (Cardano).
5. **Luồng giao tiếp**:
    * Consumer Client gửi yêu cầu tới Hydra HUB Service (qua API key).
    * Hydra HUB Service chuyển tiếp qua Proxy tới Hydra Nodes.
    * Hydra Nodes xử lý và trả về kết quả qua Proxy

![][image2]

**Hình ảnh 02**: Sơ đồ sequence diagram của việc Client giao tiếp với Hydra Hub và Hydra Node

## **B- Các giai đoạn triển khai Hydra HUB**

### **Giai đoạn I: Xây dựng mô hình tập trung (cơ bản)**:

1. **Mục tiêu**: Thử nghiệm hệ thống, tính toán tài nguyên sử dụng, tính toán các vấn đề bảo mật
2. **Công việc cần triển khai**:
    * **Hydra Hexcore v1**
    * **Hydra Hub System**: Consumer Portal, Dashboard, Admin system, …
    * **…**
3. **Provider**: provider đơn lẻ do chính Hydra HUB cung cấp.
4. **Consumer**: Cho phép Developer/User đăng ký trở thành consumer.
5. **Hydra HUB**: Phân bổ Node và thanh toán phí trực tiếp cho Hydra HUB.

![][image3]

**Hình ảnh 03**: Sơ đồ kiến trúc của mô hình tập trung (Cơ bản) của Hydra Hub giai đoạn I

### **Giai đoạn II: Xây dựng mô hình tập trung với nhiều provider**

* **Mục tiêu**: Cho phép các provider sử dụng Hydra Hexcore để tham gia cung cấp Hydra Node cho HUB, xây dựng thuật toán phân bổ Hydra Node từ nhiều provider.
* **Công việc cần triển khai**:
    * **Hydra hub system**:
        * Cho phép đăng ký làm provider, xây dựng quy trình đăng ký, chấp thuận.
        * API interface cho phép provider kết nối tới Hub System.
        * Xây dựng quy trình thanh toán (Consumer thanh toán, Provider nhận thưởng, …).
        * **…**
    * **Hydra Hexcore**:
        * Giao thức kết nối với HUB.
        * provider identify.
        * **….**

![][image4]

**Hình 04**: Mô hình kiến trúc tập trung với nhiều provider của Hydra Hub giai đoạn II

### **Giai đoạn III: Phi tập trung hóa quá trình phân bổ Hydra Node**

* **Mục tiêu**: Tập trung quá trình phân bổ Hydra Node với các provider. Dùng hợp đồng thông minh Plutus trên Cardano để quản lý các hoạt động quan trọng, đảm bảo minh bạch và bảo mật, phù hợp với tinh thần của Cardano.
* **Công việc cần triển khai**:
    * Chuyển đổi thuật toán phân bổ Hydra Node từ offchain thành smart contract.
    * Chuyển đổi quy trình thanh toán (thu phí, trả thưởng) lên smart contract.

![][image5]

**Hình 05**: Sơ đồ kiến trúc Phi tập trung hóa quá trình phân bổ Hydra Node của Hydra Hub

### **Giai đoạn IV: Phi tập trung hóa quá trình đăng ký provider**

* **Mục tiêu**: Dần loại bỏ sự ảnh hưởng của bên vận hành Hydra HUB tới việc lựa chọn (chấp thuận) provider.
* **Công việc**:
    * Smart contract đăng ký làm Provider.
    * Đồng bộ hóa quá trình đăng ký - đặt cọc - thanh toán sử dụng smart contract provider.
* **Mô tả ngắn**:
    * Provider cài đặt Hydra Hexcore và thiết lập cấu hình mạng.
    * Trên Dashboard UI Hexcore, gửi giao dịch đăng ký làm provider tới Smart Contract của Hydra HUB.
    * Smart contract xác nhận thông tin hợp lệ và thêm vào danh sách các provider của Hydra HUB.
    * Hydra HUB System đồng bộ dữ liệu của các provider và listing thông tin provider lên community.

## **C- Các rủi ro bảo mật và giải pháp dự kiến để xử lý khi triển khai Hydra Hub**

**Câu hỏi 1: Điều gì xảy ra nếu hợp đồng thông minh của Hydra HUB có lỗ hổng bảo mật, cho phép kẻ tấn công thao túng việc phân bổ Node hoặc đánh cắp phí thanh toán?**

**Giải pháp:**

1. Kiểm toán hợp đồng thông minh:  
   * Thuê các công ty kiểm toán uy tín hoặc Runtime Verification kiểm tra mã Plutus trước khi triển khai  
   * Sử dụng công cụ tự động như Marlowe (nếu phù hợp) để mô hình hóa và kiểm tra lỗi logic hợp đồng  
2. Kiểm tra bảo mật liên tục:  
   * Triển khai chương trình săn lỗi trên các nền tảng như Immunefi, với phần thưởng bằng ADA hoặc token của Hydra HUB  
   * Thực hiện kiểm toán định kỳ sau mỗi lần nâng cấp hợp đồng  
3. Cơ chế khóa an toàn (Timelock):  
   * Áp dụng timelock cho các thay đổi quan trọng (như cập nhật hợp đồng hoặc rút phí), tạo cơ hội cho cộng đồng phát hiện hành vi bất thường  
   * Ví dụ: Các thay đổi hợp đồng cần chờ 48 giờ trước khi có hiệu lực  
4. Giới hạn quyền truy cập:  
   * Chỉ cho phép các địa chỉ ví đã xác minh (của Provider và Customer) tương tác với hợp đồng  
   * Áp dụng chữ ký đa bên cho các giao dịch nhạy cảm như rút quỹ của Hydra HUB

**Câu hỏi 2: Làm thế nào để ngăn chặn tấn công DDoS vào máy chủ off-chain và bảo vệ hoạt động của Hydra HUB?**

Giải pháp:

1. Sử dụng CDN và chống DDoS:  
   * Triển khai CDN như Cloudflare hoặc Akamai để phân tán lưu lượng và lọc yêu cầu độc hại  
   * Kích hoạt tính năng Rate Limiting của Cloudflare để giới hạn số lượng yêu cầu từ mỗi IP  
2. Tăng cường khả năng mở rộng:  
   * Áp dụng kiến trúc microservices và container (Docker, K8s) để phân tán tải qua nhiều máy chủ  
   * Sử dụng hệ thống cân bằng tải để phân phối yêu cầu hợp lý  
3. Dự phòng (Failover):  
   * Duy trì máy chủ dự phòng ở nhiều khu vực địa lý (US, EU, Asia) để chuyển hướng khi cần  
   * Thiết lập cơ chế tự động chuyển đổi DNS khi phát hiện tấn công  
4. Giám sát và ứng phó:  
   * Triển khai hệ thống giám sát thời gian thực để phát hiện lưu lượng bất thường  
   * Xây dựng quy trình ứng phó khẩn cấp để giảm thiểu tác động nhanh chóng

**Câu hỏi 3: Điều gì xảy ra nếu cơ sở dữ liệu off-chain bị xâm nhập, dẫn đến rò rỉ thông tin Node hoặc giả mạo danh sách Node?**

Giải pháp:

1. Mã hóa dữ liệu:  
   * Mã hóa thông tin nhạy cảm bằng AES-256 hoặc tương đương trước khi lưu vào cơ sở dữ liệu  
   * Bảo vệ khóa mã hóa trong môi trường an toàn như AWS Key Management Service  
2. Kiểm soát truy cập nghiêm ngặt:  
   * Áp dụng xác thực mạnh (OAuth 2.0) và phân quyền chi tiết để kiểm soát truy cập cơ sở dữ liệu  
   * Chỉ cho phép Backend truy vấn cơ sở dữ liệu thông qua API bảo mật  
3. Sao lưu và khôi phục:  
   * Thực hiện sao lưu hàng ngày và lưu trữ tại nhiều vị trí an toàn với mã hóa (như AWS S3)  
   * Xây dựng quy trình khôi phục nhanh để giảm thiểu thời gian gián đoạn khi dữ liệu bị xóa hoặc sửa đổi  
4. Kiểm tra tính toàn vẹn:  
   * Đồng bộ định kỳ thông tin giữa cơ sở dữ liệu và hợp đồng thông minh để phát hiện thay đổi bất thường  
   * Áp dụng hàm băm để xác minh tính toàn vẹn của danh sách Node trước khi gửi cho Customer

**Câu hỏi 4: Làm thế nào để đảm bảo Provider không cung cấp Hydra Node độc hại hoặc kém chất lượng gây gián đoạn Head của Customer?**

Giải pháp:

1. Cơ chế đặt cọc (Staking):  
   * Yêu cầu Provider đặt cọc ADA khi đăng ký Node. Nếu Node bị phát hiện độc hại hoặc kém chất lượng, số ADA đặt cọc sẽ bị tịch thu  
   * Ví dụ: Đặt cọc 100 ADA cho mỗi Node và mất 50% nếu Node không đạt yêu cầu  
2. Giám sát hiệu suất độc lập:  
   * Monitoring Module sử dụng nhiều nguồn dữ liệu để đánh giá hiệu suất Node (như kiểm tra tốc độ, độ trễ từ nhiều điểm), thay vì chỉ dựa vào báo cáo của Provider  
   * So sánh hiệu suất giữa các Node trong cùng Head để phát hiện bất thường  
3. Hệ thống đánh giá và phạt:  
   * Hợp đồng thông minh tự động phạt Provider khi Node không đạt tiêu chuẩn (ví dụ: độ trễ lớn hơn 1 giây, thời gian hoạt động dưới 99%)  
   * Customer có thể gửi phản hồi về chất lượng Node để hợp đồng thông minh xem xét xử phạt  
4. Xác minh danh tính Provider:  
   * Yêu cầu Provider xác minh danh tính (KYC đơn giản) hoặc liên kết với địa chỉ ví Cardano có lịch sử hoạt động tốt  
   * Giới hạn số lượng Node tối đa mỗi Provider có thể đăng ký để ngăn chặn tấn công Sybil

**Câu hỏi 5: Làm thế nào để ngăn chặn Customer gửi yêu cầu giả mạo hoặc không thanh toán phí, gây ảnh hưởng đến Provider và Hydra HUB?**

Giải pháp:

1. Xác thực yêu cầu:  
   * Yêu cầu Customer xác thực thông qua ví Cardano (như Lace) trước khi tạo Head.  
   * Backend xác minh chữ ký số của yêu cầu để đảm bảo nguồn gốc từ Customer hợp lệ.  
2. Thanh toán trước (Pre-payment):  
   * Hợp đồng thông minh yêu cầu Customer thanh toán phí trước khi được cấp Node (ví dụ: 5 ADA cho Head với 3 Node).  
   * Áp dụng cơ chế hoàn tiền (escrow) khi Head không được tạo thành công do lỗi hệ thống.  
3. Giới hạn tỷ lệ yêu cầu (Rate Limiting):  
   * Backend giới hạn số lượng yêu cầu từ mỗi Customer (tối đa 10 yêu cầu/phút).  
   * Hợp đồng thông minh từ chối các yêu cầu bất thường lặp lại từ cùng một ví.  
4. Phạt hành vi xấu:  
   * Hợp đồng thông minh có thể tạm khóa ví (blacklist) của Customer khi phát hiện yêu cầu giả mạo hoặc không thanh toán.  
   * Lưu trữ lịch sử hành vi của Customer trên blockchain để Provider đánh giá độ tin cậy.

**Câu hỏi 6: Điều gì xảy ra nếu giao tiếp giữa Backend và hợp đồng thông minh bị chặn hoặc giả mạo, dẫn đến phân bổ Node sai hoặc mất phí?**

Giải pháp:

1. Bảo mật API:  
   * Áp dụng HTTPS với chứng chỉ SSL/TLS để mã hóa giao tiếp giữa Backend và API.  
   * Xác thực yêu cầu API bằng khóa bí mật và chữ ký HMAC để đảm bảo tính toàn vẹn.  
2. Kiểm tra tính toàn vẹn:  
   * Backend ký số tất cả yêu cầu gửi đến hợp đồng thông minh, và hợp đồng sẽ kiểm tra chữ ký trước khi xử lý.  
   * Lưu hash của yêu cầu trên blockchain để đối chiếu khi có tranh chấp.  
3. Dự phòng API:  
   * Sử dụng nhiều nhà cung cấp API (Blockfrost, Ogmios và Cardano Node cục bộ) để đảm bảo tính sẵn sàng.  
   * Tự động chuyển đổi sang API dự phòng khi phát hiện API chính bị chặn hoặc chậm.  
4. Giám sát giao tiếp:  
   * Monitoring Module ghi lại nhật ký mọi yêu cầu API và phát hiện các mẫu bất thường (như yêu cầu lặp lại từ nguồn không xác định).  
   * Cảnh báo tức thì nếu giao tiếp giữa Backend và hợp đồng thông minh bị gián đoạn quá 30 giây.

**Câu hỏi 8: Làm thế nào để đảm bảo rằng Monitoring Module không bị thao túng để báo cáo sai hiệu suất của Hydra Node, dẫn đến phân bổ hoặc phạt không công bằng?**

Giải pháp:

1. Nguồn dữ liệu đa dạng:  
   * Monitoring Module thu thập dữ liệu từ nhiều nguồn (như kiểm tra từ các máy chủ khác nhau, phản hồi từ Customer, dữ liệu từ Head) để đảm bảo tính chính xác.  
   * Sử dụng giao thức đồng thuận nhẹ để so sánh kết quả giám sát từ nhiều điểm.  
2. Xác minh on-chain:  
   * Ghi lại các chỉ số hiệu suất quan trọng (như độ trễ trung bình) lên hợp đồng thông minh, cho phép cộng đồng hoặc Customer kiểm tra.  
   * Hợp đồng thông minh tự động từ chối báo cáo hiệu suất bất thường (ví dụ: độ trễ âm).  
3. Cơ chế tranh chấp:  
   * Cho phép Provider và Customer gửi tranh chấp nếu họ không đồng ý với báo cáo hiệu suất, được hợp đồng thông minh xử lý dựa trên dữ liệu lịch sử.  
   * Ví dụ: Nếu Provider bị phạt do báo cáo sai, họ có 24 giờ để cung cấp bằng chứng (như nhật ký Node).  
4. Bảo mật Monitoring Module:  
   * Triển khai module giám sát trên các máy chủ tách biệt, với tường lửa và kiểm soát truy cập nghiêm ngặt.  
   * Mã hóa dữ liệu hiệu suất trước khi truyền từ Hydra Node đến Monitoring Module, sử dụng giao thức như TLS.

**Tổng kết Các giải pháp được sử dụng để bảo mật phòng chống rủi ro bao gồm:**

* **Kiểm toán và bảo mật hợp đồng thông minh để ngăn lỗ hổng.**  
* **Chống DDoS và dự phòng cho máy chủ off-chain.**  
* **Mã hóa và kiểm soát truy cập cho cơ sở dữ liệu.**  
* **Cơ chế đặt cọc và giám sát để đảm bảo chất lượng Node.**  
* **Xác thực và giới hạn yêu cầu để ngăn hành vi xấu từ Customer.**  
* **Bảo mật API và giao diện Frontend để tránh giả mạo hoặc đánh cắp dữ liệu.**  
* **Giám sát độc lập và minh bạch để đảm bảo báo cáo hiệu suất chính xác.**

**Các biện pháp này sẽ giúp Hydra HUB trở thành một nền tảng an toàn, minh bạch, và hiệu quả, tiện dụng cho các nhà phát triển với Hydra với chi phí thấp góp phần thúc đẩy sự phát triển của các ứng dụng.**


[image1]: ../../assets/images/image1.png
[image2]: ../../assets/images/image2.png
[image3]: ../../assets/images/image3.png
[image4]: ../../assets/images/image4.png
[image5]: ../../assets/images/image5.png
