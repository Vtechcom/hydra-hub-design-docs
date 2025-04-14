# **Tổng quan**

Hydra Hub là giải pháp công nghệ  kết nối giữa client (Consumer Client) và các node Hydra (Hydra Nodes) thông qua proxy (HydraHUB) được cung cấp bởi các Providers  khác nhau

Các nhà phát triển khi có nhu cầu sử dụng node hydra để triển khai ứng dụng chỉ cần đăng ký trở thành Consumer Client và sử dụng các dịch vụ Hydra từ các các Providers một cách đơn giản và nhanh chóng triển khai. Với chi phí rẻ hơn rất nhiều khi tự vận hành, Bỏ qua những khó khăn khi phải thực hiện các công việc:

1. Đồng bộ hóa với Cardano Node phức tạp  
2. Quản lý khóa và bảo mật thủ công  
3. Giám sát và xử lý lỗi  
4. Xử lý giao tiếp giữa các Hydra Node không ổn định

**A- System Diagram của giải pháp Hydra HUB**

![Sơ đồ mô tả kiến trúc triển khai của hệ thống Hydra HUB, giải pháp kết nối giữa client (Consumer Client) và các node Hydra (Hydra Nodes) thông qua proxy.][image1]

Hình 01: Sơ đồ mô tả kiến trúc triển khai của hệ thống Hydra HUB, giải pháp kết nối giữa client (Consumer Client) và các node Hydra (Hydra Nodes) thông qua proxy.

**Diễn giải**:

1. Consumer Client:  
   * DApps (UI): Ứng dụng phi tập trung (Decentralized Apps) mà người dùng (Consumer) sử dụng.  
   * Giao tiếp với Hydra HUB Service qua WS/HTTP với API key.  
   * Ví dụ: Gửi yêu cầu GET [https://api-key-cb3ec.hydranode.io/commits](https://api-key-cb3ec.hydranode.io/commits).  
2. Hydra HUB:  
   * Hydra HUB Service: Dịch vụ trung tâm quản lý:  
     1. API keys manager: Quản lý API key.  
     2. Consumer Portal (UI): Giao diện cho người dùng quản lý thông tin.  
     3. Provider Info: Thông tin về các nhà cung cấp (Providers).  
     4. Proxy mappers: Ánh xạ yêu cầu tới các Hydra Node.  
   * Hydra Node Proxy: Proxy trung gian, giao tiếp với Hydra Nodes qua WS/HTTP  
   * Giao tiếp với Consumer Client qua [Socket.io](http://Socket.io) hoặc HTTP  
3. Providers:  
   * Provider 1 (123.2.12.3) và Provider 2 (345.2.12.3): Các cụm máy chủ hoặc nhà cung cấp dịch vụ.  
   * Mỗi provider chứa:  
     1. Hexcore Dashboard UI: Giao diện quản trị.  
     2. Cardano Node: Node của blockchain Cardano.  
     3. Hydra Nodes: Các node Hydra (10001, 10002, v.v.), có thể chạy trên Docker hoặc Kubernetes (K8s).  
   * Giao tiếp với Hydra HUB qua WS/HTTP.  
4. Hydra Nodes:  
   * Các node con (10001, 10002, v.v.) xử lý yêu cầu từ Hydra HUB Proxy.  
   * Có thể chạy trên Docker hoặc Kubernetes, phục vụ các tác vụ liên quan đến blockchain (Cardano).  
5. Luồng giao tiếp:  
   * Consumer Client gửi yêu cầu tới Hydra HUB Service (qua API key).  
   * Hydra HUB Service chuyển tiếp qua Proxy tới Hydra Nodes.  
   * Hydra Nodes xử lý và trả về kết quả qua Proxy

**![][image2]**

Hình ảnh 02: Sơ đồ sequence diagram của việc Client giao tiếp với Hydra Hub và Hydra Node

**B- Các giai đoạn triển khai Hydra HUB**

**Giai đoạn I:  Xây dựng mô hình tập trung (cơ bản):**

1. **Mục tiêu: Thử nghiệm hệ thống, tính toán tài nguyên sử dụng, tính toán các vấn đề bảo mật**  
   2. **Công việc cần triển khai:**  
      * **Hydra Hexcore v1**  
      * **Hydra Hub System: Consumer Portal, Dashboard, Admin system, …**  
      * **…**  
   3. **Provider: provider đơn lẻ do chính Hydra HUB cung cấp:**  
   4. **Consumer: Cho phép Developer/User đăng ký trở thành consumer**  
   5. **Hydra HUB: Phân bổ Node và thanh toán phí trực tiếp cho Hydra HUB**

**![][image3]**

**Hình ảnh 03: Sơ đồ kiến trúc của mô hình tập trung (Cơ bản) của Hydra Hub giai đoạn I**

**Giai đoạn II: Xây dựng mô hình tập trung với nhiều provider**

* **Mục tiêu: Cho phép các provider sử dụng Hydra Hexcore để tham gia cung cấp Hydra Node cho HUB, xây dựng thuật toán phân bổ Hydra Node từ nhiều provider**  
* **Công việc cần triển khai:**  
  * **Hydra hub system:**  
    * **Cho phép đăng ký làm provider, xây dựng quy trình đăng ký, chấp thuận**  
    * **API interface cho phép provider kết nối tới Hub System**  
    * **Xây dựng quy trình thanh toán (Consumer thanh toán, Provider nhận thưởng, …)**  
    * **…**  
  * **Hydra Hexcore:**  
    * **Giao thức kết nối với HUB**  
    * **provider identify**  
    * **….**

**![][image4]**

**Hình 04: Mô hình kiến trúc tập trung với nhiều provider của Hydra Hub giai đoạn II**

**Giai đoạn III: Phi tập trung hóa quá trình phân bổ Hydra Node**

* **Mục tiêu: tập trung quá trình phân bổ Hydra Node với các provider. Dùng hợp đồng thông minh Plutus trên Cardano để quản lý các hoạt động quan trọng, đảm bảo minh bạch và bảo mật, phù hợp với tinh thần của Cardano.**  
* **Công việc cần triển khai:**  
  * **Chuyển đổi thuật toán phân bổ Hydra Node từ offchain thành smart contract**  
  * **Chuyển đổi quy trình thanh toán (thu phí, trả thưởng) lên smart contract**

**![][image5]**

**Hình 05: Sơ đồ kiến trúc Phi tập trung hóa quá trình phân bổ Hydra Node của Hydra Hub**

**Giai đoạn IV: Phi tập trung hóa quá trình đăng ký provider**

* Mục tiêu: Dần loại bỏ sự ảnh hưởng của bên vận hành Hydra HUB tới việc lựa chọn (chấp thuận) provider  
* Công việc:  
  * Smart contract đăng ký làm Provider  
  * Đồng bộ hóa quá trình đăng ký \- đặt cọc \- thanh toán sử dụng smart contract provider  
* Mô tả ngắn:  
  * Provider cài đặt Hydra Hexcore và thiết lập cấu hình mạng  
  * Trên Dashboard UI Hexcore, gửi giao dịch đăng ký làm provider tới Smart Contract của Hydra HUB  
  * Smart contract xác nhận thông tin hợp lệ và thêm vào danh sách các provider của Hydra HUB  
  * Hydra HUB System đồng bộ dữ liệu của các provider và listing thông tin provider lên community

[image1]: ../../assets/images/image1.png
[image2]: ../../assets/images/image2.png
[image3]: ../../assets/images/image3.png
[image4]: ../../assets/images/image4.png
[image5]: ../../assets/images/image5.png
