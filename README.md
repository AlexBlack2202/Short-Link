# Short Link Service

Một hệ thống tạo short link đơn giản, gọn nhẹ, có khả năng **High Availability (HA)** và **High Scalability** dựa trên kiến trúc **Raft (n*2+1 nodes)**.

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
