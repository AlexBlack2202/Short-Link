# Short Link Service

Một hệ thống tạo short link đơn giản, gọn nhẹ, có khả năng **High Availability (HA)** và **High Scalability** dựa trên kiến trúc **Raft (n*2+1 nodes)**.

Đây là pet project của tôi, dùng để luyện tư duy giải quyết vấn đề, khi tôi gặp bế tắt ở vấn đề khác. Tôi cố gắng giải quyết vấn đề ở mức đơn giản nhất, nhưng vẫn được hiệu quả tốt nhất. Mục tiêu là maximum performance , high TPS, High CCU. Những cái như dashboard, monitor , các bạn có thể xài các công cụ opensource khác như graylog, ELK , ... để giải quyết.

# Các vấn đề thường hay gặp với url

---

## 1. Link quá dài, khó nhớ, khó chia sẻ

**Vấn đề:**

* URL chứa nhiều tham số (query string, token) → dài và rối.
* Khi gửi qua SMS, chat, email dễ bị ngắt hoặc cắt mất phần cuối.
* Không thể gõ tay hoặc in lên ấn phẩm marketing một cách gọn gàng.

**Giải pháp với short link:**

* Rút gọn thành dạng `domain/abc123` dễ nhớ, dễ nhập.
* Giúp hiển thị đẹp trên poster, slide, QR code, SMS.

---

## 2. Không theo dõi được hiệu quả chia sẻ

**Vấn đề:**

* Không biết ai đã click, bao nhiêu lượt truy cập, từ đâu đến.
* Khó đánh giá hiệu quả chiến dịch marketing hoặc chia sẻ nội bộ.

**Giải pháp với short link:**

* Hệ thống short link log lại thông tin click (thời gian, IP, location, referrer).
* Tích hợp dashboard để theo dõi, phân tích hành vi người dùng.

---

## 3. Không linh hoạt khi cần thay đổi đích đến

**Vấn đề:**

* Khi đổi domain hoặc landing page, phải cập nhật tất cả nơi đã phát tán link.
* Không thể redirect người dùng sang nội dung mới nếu URL cũ đã được gửi ra ngoài.

**Giải pháp với short link:**

* Chỉ cần đổi URL đích trong hệ thống short link, link rút gọn cũ vẫn hoạt động.
* Cho phép redirect có điều kiện (ví dụ: theo thiết bị, thời gian).

---

## 4. URL dễ bị lỗi khi copy/paste

**Vấn đề:**

* Nhiều ứng dụng cắt bớt hoặc xuống dòng URL dài → người nhận click vào bị lỗi 404.

**Giải pháp với short link:**

* Link rút gọn ngắn gọn, không bị cắt khi truyền qua các kênh dễ lỗi.

---

## 5. Lộ cấu trúc hoặc thông tin nhạy cảm

**Vấn đề:**

* URL dài chứa ID nội bộ, tham số xác thực, tên file thật…
* Dễ bị người ngoài phân tích, khai thác.

**Giải pháp với short link:**

* Ẩn hoàn toàn URL thật khỏi người dùng.
* Giảm rủi ro lộ thông tin kỹ thuật.

---

## 6. Khó tích hợp với hệ thống kỹ thuật nội bộ

**Vấn đề:**

* Trong microservice hoặc cluster, tài nguyên được định vị bằng URL phức tạp.
* Khó migrate hoặc đổi hạ tầng mà không ảnh hưởng người dùng cuối.

**Giải pháp với short link:**

* Sử dụng short link làm “ID ánh xạ” tới tài nguyên nội bộ.
* Giúp di chuyển dữ liệu, đổi cấu trúc hạ tầng mà không cần thay đổi ở phía người dùng.

---

Nếu muốn, mình có thể viết thêm một **bảng mapping vấn đề ↔ short link feature** để bạn đưa thẳng vào proposal kiến trúc hệ thống.
Bạn có muốn mình làm bảng đó luôn không?

---


## Các phiên bản hiện có

- Phiên bản dành cho **Ubuntu / Linux**.
- Phiên bản dành cho **Mac Intel (x86_64)**.
- Phiên bản dành cho **Mac Apple Silicon (ARM64)**.

Bạn vui lòng tải và chạy phiên bản phù hợp với hệ điều hành và kiến trúc máy của mình.

---

## Tính năng chính

- **Kiến trúc n*2+1 nodes:** đảm bảo an toàn dữ liệu và hoạt động liên tục ngay cả khi một số node bị lỗi.
- **Viết (write) ở leader, đọc (read) ở các slave hoặc reader:** giúp phân phối tải hiệu quả.
- **Có thể xây dựng hệ thống HA bên trên để điều phối và cân bằng tải đọc (read).**

---

## Cách chạy hệ thống

### 1. Khởi động node mới (cluster bootstrap hoặc join)

```bash
./short_link --id=node1 --raft=127.0.0.1:12001 --bind=127.0.0.1:8081 --bootstrap
./short_link --id=node2 --raft=127.0.0.1:12002 --bind=127.0.0.1:8082 --join=127.0.0.1:8081
./short_link --id=node3 --raft=127.0.0.1:12003 --bind=127.0.0.1:8083 --join=127.0.0.1:8081
````

* Node đầu tiên dùng `--bootstrap` để tạo cluster mới.
* Các node tiếp theo dùng `--join` để tham gia cluster.

### 2. Restart các node khi cả cluster die

```bash
./short_link --id=node1 --raft=127.0.0.1:12001 --bind=127.0.0.1:8081
./short_link --id=node2 --raft=127.0.0.1:12002 --bind=127.0.0.1:8082
./short_link --id=node3 --raft=127.0.0.1:12003 --bind=127.0.0.1:8083
```

> Lưu ý: Khi restart không cần thêm `--bootstrap` hoặc `--join`, vì dữ liệu cluster đã được lưu.

---

## Sử dụng API

### Tạo short link

```bash
curl -X POST 'http://127.0.0.1:8083/shorten?url=https%3A%2F%2Fwww.thegioididong.com' -H 'Content-Type: application/json'
```

### Lấy link gốc từ short link ID

```bash
curl 'http://127.0.0.1:8082/d2bm043gbukitfpuujkg'
```

### Xóa node khỏi cluster khi node lỗi

Khi một node (ví dụ node2) bị lỗi và không thể phục hồi, để tránh tốn tài nguyên cho việc health check hoặc tự động thêm lại node vào cluster, bạn nên gọi API để **xóa node đó khỏi cluster**:

```bash
curl 'http://127.0.0.1:8083/remove-server' -H 'Content-Type: application/json' -d '{"id":"node2"}'
```

Điều này giúp cluster hoạt động trơn tru, ổn định và giảm tải cho hệ thống.

---

## Kiến trúc & Mô hình hoạt động

* **Raft protocol** đảm bảo đồng bộ dữ liệu giữa các node.
* Các thao tác **viết** chỉ thực hiện trên **Leader node**, đảm bảo tính nhất quán.
* Các thao tác **đọc** có thể thực hiện ở các node phụ (slave/reader), giúp phân tải.
* Mô hình **n\*2+1** node giúp hệ thống vẫn hoạt động ổn định ngay cả khi một số node bị lỗi.
* Có thể kết hợp với các công cụ load balancer hoặc proxy để xây dựng hệ thống **High Availability** và phân phối tải **read** hiệu quả.

---

## Liên hệ

Nếu có câu hỏi hoặc góp ý, vui lòng mở issue hoặc liên hệ trực tiếp.

---

Cảm ơn bạn đã sử dụng Short Link Service!
