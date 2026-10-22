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

Mọi node cần hai token (mỗi token ít nhất 16 ký tự), thiếu thì node không khởi động:

```bash
export SHORTLINK_CLUSTER_TOKEN=$(openssl rand -hex 32)   # giống nhau trên mọi node
export SHORTLINK_API_TOKEN=$(openssl rand -hex 32)       # client dùng để gọi /shorten
./short_link --id=node1 --raft=127.0.0.1:12001 --bind=127.0.0.1:8081 --bootstrap
./short_link --id=node2 --raft=127.0.0.1:12002 --bind=127.0.0.1:8082 --join=127.0.0.1:8081
./short_link --id=node3 --raft=127.0.0.1:12003 --bind=127.0.0.1:8083 --join=127.0.0.1:8081
```

* Node đầu tiên dùng `--bootstrap` để tạo cluster mới.
* Các node tiếp theo dùng `--join` để tham gia cluster. `--join` có thể trỏ tới bất kỳ node nào, request sẽ được chuyển về leader.
* Node mới được thêm dạng non-voter, leader chờ node bắt kịp dữ liệu (tối đa 30 giây) rồi mới đưa thành voter. Node không bắt kịp sẽ bị gỡ ra và lệnh join trả lỗi.
* Thay một node hỏng bằng máy mới (cùng `--id`, địa chỉ khác) chỉ cần chạy lại với `--join`; leader tự gỡ bản ghi cũ.
* Node mất ổ đĩa: xoá thư mục dữ liệu của nó (nếu còn) rồi chạy lại với `--join`. Trước khi raft khởi động, node nhờ leader gỡ bản ghi cũ, rồi mới join lại và bắt kịp dữ liệu; nhờ vậy một node rỗng không bao giờ được bỏ phiếu.

### Chạy nhiều máy (production)

```bash
export SHORTLINK_CLUSTER_TOKEN='<chuỗi bí mật dùng chung cho mọi node>'
export SHORTLINK_API_TOKEN='<token cho client tạo link>'
./short_link --id=node1 --raft=10.0.0.1:12001 --bind=:8080 --public-url=https://s.example.com --allowed-hosts=thegioididong.com --bootstrap
./short_link --id=node2 --raft=10.0.0.2:12001 --bind=:8080 --public-url=https://s.example.com --allowed-hosts=thegioididong.com --join=10.0.0.1:8080
```

| Flag | Ý nghĩa |
|------|---------|
| `--cluster-token` / `SHORTLINK_CLUSTER_TOKEN` | Bắt buộc. Xác thực kết nối raft giữa các node (bắt tay HMAC, token không gửi qua mạng) và bảo vệ `/join`, `/remove-server`, `/cluster/*`, `/metrics`. Nên dùng biến môi trường thay vì flag để token không hiện trong `ps`. |
| `--api-token` / `SHORTLINK_API_TOKEN` | Token cho `/shorten`, có thể nhiều token cách nhau bằng dấu phẩy. Bắt buộc, trừ khi chạy với `--allow-anonymous-shorten`. |
| `--allow-anonymous-shorten` | Cho phép ai cũng tạo được link. Không nên bật: kẻ xấu có thể dùng domain của bạn để chuyển hướng tới trang lừa đảo. |
| `--allowed-hosts` | Danh sách domain mà link được phép trỏ tới, cách nhau bằng dấu phẩy; mỗi domain cho phép cả subdomain của nó. Mặc định: mọi domain. |
| `--http-advertise` | Địa chỉ HTTP mà node khác dùng để gọi tới node này. Mặc định: host của `--raft` + port của `--bind`. |
| `--public-url` | Domain dùng trong short link trả về. Nên đặt khi chạy thật; mặc định là `http://<http-advertise>` của leader. |
| `--raft-tls-cert`, `--raft-tls-key`, `--raft-tls-ca` | Mã hoá cổng raft bằng TLS hai chiều. Chứng chỉ của mỗi node phải có IP/hostname của `--raft` trong SAN và được ký bởi cùng CA. Không đặt thì cổng raft vẫn được xác thực bằng token nhưng không mã hoá. |
| `--cache-mb` | Block cache của Pebble (mặc định `1024`). RAM dùng ≈ cache + khoảng 300 MB, không tăng theo số link. |
| `--default-ttl` | Thời hạn mặc định của link khi request không truyền `ttl`/`expires_at` (ví dụ `720h`). Mặc định `0` = không hết hạn. |
| `--expire-sweep-interval` | Chu kỳ leader xoá link đã hết hạn (mặc định `30s`). |

Giữa các node, HTTP nội bộ (chuyển request về leader, `/join`, `/cluster/*`) chưa được mã hoá và có gửi cluster token. Hãy đặt các node trong mạng nội bộ, và chặn cổng raft lẫn cổng HTTP từ bên ngoài; chỉ mở HTTP qua load balancer.

### 2. Restart các node khi cả cluster die

```bash
./short_link --id=node1 --raft=127.0.0.1:12001 --bind=127.0.0.1:8081
./short_link --id=node2 --raft=127.0.0.1:12002 --bind=127.0.0.1:8082
./short_link --id=node3 --raft=127.0.0.1:12003 --bind=127.0.0.1:8083
```

> Lưu ý: Khi restart không cần thêm `--bootstrap` hoặc `--join`, vì dữ liệu cluster đã được lưu. Nếu vẫn truyền `--join` thì leader chỉ cập nhật lại thông tin node; join lỗi mà node đã có dữ liệu cluster thì node vẫn chạy tiếp như bình thường.
>
> Node bị remove trong lúc đang tắt sẽ tự phát hiện sau khoảng 10 giây (hỏi các node khác) và trả `503`. Muốn đưa nó trở lại thì chạy lại với `--join`.

### Dừng node

Nên dừng bằng `Ctrl+C` hoặc `kill` (SIGTERM). Node sẽ nhường leader cho node khác (nếu đang là leader) trong lúc vẫn còn nhận request, chờ request đang chạy xong rồi mới tắt raft và đóng dữ liệu.

Dữ liệu link nằm trên đĩa (Pebble), nên node khởi động lại đọc được ngay, kể cả khi chưa liên lạc được leader. Sau khi crash (`kill -9`, mất điện), node đọc được dữ liệu đã ghi xuống đĩa; link chưa kịp có thì trả `503` thay vì `404` cho tới khi node liên lạc lại được leader.

### Nâng cấp từ phiên bản cũ

Dừng **tất cả** node, thay binary rồi khởi động lại với `SHORTLINK_CLUSTER_TOKEN` và `SHORTLINK_API_TOKEN` (không chạy lẫn phiên bản cũ và mới trong cùng cluster: cổng raft bây giờ yêu cầu bắt tay bằng token). Dữ liệu cũ được chuyển sang Pebble tự động ở lần khởi động đầu, từ snapshot JSON hoặc raft log của các bản trước; link cũ giữ nguyên ID. Lần khởi động đầu có thể lâu hơn, tuỳ lượng dữ liệu.

Build từ source: code giờ gồm nhiều file nên dùng `go build .` thay vì `go build main.go`.

---

## Sử dụng API

### Tạo short link

Chỉ nhận `POST`, cần header `Authorization: Bearer <api token>`. Tham số `url` (bắt buộc, chỉ `http`/`https`, tối đa 2048 ký tự, domain phải nằm trong `--allowed-hosts` nếu có đặt) có thể truyền qua query, form hoặc JSON body.

```bash
curl -X POST 'http://127.0.0.1:8083/shorten?url=https://www.thegioididong.com' -H "Authorization: Bearer $SHORTLINK_API_TOKEN"
```

Link có thời hạn: truyền `ttl` (số giây hoặc dạng `30m`, `24h`) **hoặc** `expires_at` (RFC3339 hoặc unix giây), tối đa 10 năm:

```bash
curl -X POST 'http://127.0.0.1:8083/shorten' -H "Authorization: Bearer $SHORTLINK_API_TOKEN" -d 'url=https://www.thegioididong.com&ttl=24h'
curl -X POST 'http://127.0.0.1:8083/shorten' -H "Authorization: Bearer $SHORTLINK_API_TOKEN" \
  -H 'Content-Type: application/json' -H 'Accept: application/json' \
  -d '{"url":"https://www.thegioididong.com","expires_at":"2026-12-31T23:59:59+07:00"}'
```

Mã lỗi: `401` sai/thiếu token, `403` domain không được phép, `413` body quá 8 KB, `503` đang đổi leader (thử lại).

Mặc định trả về short link dạng text. Gửi `Accept: application/json` để nhận `{"id","short_url","url","expires_at"}`.
ID được sinh ngẫu nhiên (tối đa 11 ký tự base62) nên không đoán hay liệt kê được link.
Khi cluster đang không có leader, API trả `503` kèm `Retry-After`; client nên thử lại.

### Lấy link gốc từ short link ID

```bash
curl 'http://127.0.0.1:8082/d2bm043gbukitfpuujkg'
```

* `302`: chuyển hướng tới link gốc.
* `404`: không tồn tại.
* `410`: link đã hết hạn. Leader xoá hẳn link hết hạn theo chu kỳ `--expire-sweep-interval`, sau đó sẽ trả `404`.
* `503`: node này không còn là thành viên cluster (đã bị remove), hoặc chưa xác nhận được với leader là link có tồn tại hay không; client nên thử lại hoặc thử node khác.

Follower không thấy link (ví dụ link vừa tạo) sẽ đối chiếu với leader rồi đọc lại. Các lần đối chiếu được gom lại, tối đa 50 lần/giây cho mỗi follower, nên một lượng lớn request với ID không tồn tại không đổ dồn lên leader.

### Health check

```bash
curl 'http://127.0.0.1:8082/health'
```

Trả `200` khi node là voter của cluster và đang có leader. Trả `503` khi node đã bị remove, bị tách khỏi cluster, hoặc đang join và bắt kịp dữ liệu. Dùng endpoint này cho load balancer.

### Metrics

```bash
curl -H "Authorization: Bearer $SHORTLINK_CLUSTER_TOKEN" 'http://127.0.0.1:8082/metrics'
```

Định dạng text của Prometheus, gồm số request theo route và mã trả về, số link, dung lượng dữ liệu trên đĩa, trạng thái raft (leader, term, index, thời gian từ lần liên lạc leader cuối).

### Xóa node khỏi cluster khi node lỗi

Khi một node (ví dụ node2) bị lỗi và không thể phục hồi, để tránh tốn tài nguyên cho việc health check hoặc tự động thêm lại node vào cluster, bạn nên gọi API để **xóa node đó khỏi cluster**:

```bash
curl 'http://127.0.0.1:8083/remove-server' -H "Authorization: Bearer $SHORTLINK_CLUSTER_TOKEN" -H 'Content-Type: application/json' -d '{"id":"node2"}'
```

Trước khi xoá, leader kiểm tra các voter còn lại vẫn đủ quorum; một node chỉ được tính là còn sống khi nó đang nhận heartbeat từ leader. Nếu không đủ (ví dụ xoá một node đang sống trong khi node khác đã chết hoặc bị cắt mạng) thì API trả `409` để tránh làm chết cả cluster. Hãy xoá node đã chết trước. Chỉ dùng `{"id":"node2","force":true}` khi chắc chắn mình biết hậu quả.

Join và remove được leader xử lý lần lượt, nên remove một node đang join sẽ không bị lệnh join đó ghi đè.

Xoá chính leader: leader nhường quyền cho node khác trước rồi leader mới thực hiện việc xoá, nên cluster không bị gián đoạn ghi.

Điều này giúp cluster hoạt động trơn tru, ổn định và giảm tải cho hệ thống.

---

## Kiến trúc & Mô hình hoạt động

* **Raft protocol** đảm bảo đồng bộ dữ liệu giữa các node.
* Các thao tác **viết** chỉ thực hiện trên **Leader node**, đảm bảo tính nhất quán.
* Các thao tác **đọc** có thể thực hiện ở các node phụ (slave/reader), giúp phân tải.
* Mô hình **n\*2+1** node giúp hệ thống vẫn hoạt động ổn định ngay cả khi một số node bị lỗi.
* Có thể kết hợp với các công cụ load balancer hoặc proxy để xây dựng hệ thống **High Availability** và phân phối tải **read** hiệu quả.
* Mỗi node lưu toàn bộ link trên đĩa bằng **Pebble** (thư mục `<raft-dir>/<id>/fsm-*`). Snapshot của raft được ghi dạng stream (nén snappy) và không làm nghẽn lệnh ghi.

### Ước lượng tài nguyên

Đo với 5 triệu link (URL dài khoảng 90 ký tự):

| | Mỗi link | Ước tính 1 tỷ link |
|---|---|---|
| Dữ liệu trên đĩa mỗi node | ≈ 87 byte | ≈ 90 GB |
| File snapshot | ≈ 30 byte | ≈ 30 GB |
| RAM | không tăng theo số link | cache (`--cache-mb`) + khoảng 300 MB |

Thời gian ghi snapshot (chạy nền) và restore (khi thêm node mới) đo được khoảng 2 giây và 4 giây cho 5 triệu link, tức khoảng 7 phút và 14 phút cho 1 tỷ link. Nên dùng ổ SSD/NVMe và dự trù dung lượng đĩa gấp đôi (dữ liệu và snapshot).

---

## Liên hệ

Nếu có câu hỏi hoặc góp ý, vui lòng mở issue hoặc liên hệ trực tiếp.

---

Cảm ơn bạn đã sử dụng Short Link Service!
