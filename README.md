# phat_trien_ung_dung_tren_ma_nguon_mo_4
# Môn: Phát triển ứng dụng với mã nguồn mở-TEE0421

Lớp: 58KTPM

**Bài tập 04:**  
# KHAI THÁC N8N ĐỂ TỰ ĐỘNG ĐĂNG BÀI LÊN WORDPRESS
# 
## deadline : 23h59 ngày 25 tháng 5 năm 2026.

### SỬ DỤNG KẾT QUẢ ĐÃ LÀM Ở BÀI TẬP 3, BỔ SUNG VÀO DOCKER COMPOSE ĐỂ CÓ THÊM SERVICE 8N8:

1. SỬ DỤNG DOCKER TRÊN UBUNTU ĐỂ TẠO 1 file **docker-compose.yml** chứa: 
- Mariadb: sử dụng **image: mariadb:latest** để làm hệ quản trị csdl cho wordpress, thêm các biến môi trường: TZ: "Asia/Ho_Chi_Minh", MARIADB_ROOT_PASSWORD, MARIADB_DATABASE, MARIADB_USER, MARIADB_PASSWORD (giá trị tuỳ ý)
- Phpmyadmin: sử dụng **image: phpmyadmin:latest** để đăng nhập vào mariadb rồi tạo csdl trống (chỉ để xem, ko cần tạo bảng từ đây, wordpress sẽ làm hết), khai báo biến môi trường: PMA_HOST: <tên service mariadb>, PMA_ARBITRARY: 1
- WordPress: sử dụng **image: wordpress:latest**, truyền các tham số môi trường cho wordpress là các thông tin truy cập csdl mariadb, tạo bởi Phpmyadmin, khai báo biến môi trường:  WORDPRESS_DB_HOST: <tên service mariadb>, WORDPRESS_DB_NAME, WORDPRESS_DB_USER, WORDPRESS_DB_PASSWORD (giá trị theo mariadb đã khai báo)
- Cloudflared: sử dụng **image: cloudflare/cloudflared:latest** , full command và token lấy từ dashboard của cloudflare, dùng AI chuyển sang dạng docker compose
- N8n : sử dụng **image: n8nio/n8n:latest**, nhớ truyền biến môi trường WEBHOOK_URL theo sub-domain đã add router cho cloudflared tunnel (ví dụ: WEBHOOK_URL=https://k58-n8n.tdh.io.vn/ )

2. Yêu cầu: sau khi có 5 service này trong file docker-compose.yml :
- pull các images về và chạy chúng (up -d)
- Kiểm tra các service đã running ok (ko bị restart liên tục)
- Cấu hình cloudflare tunnel add router để public wordpress lên sub-domain1 (dùng để truy cập wordpress)
- Cấu hình cloudflare tunnel add router để public Phpmyadmin lên sub-domain2 (dùng để truy cập phpmyadmin)
- Cấu hình cloudflare tunnel add router để public n8n này lên sub-domain3 (dùng để truy cập và cấu hình n8n)
- Truy cập sub-domain2 để quan sát xem cơ sở dữ liệu chưa có bảng nào!
- Truy cập sub-domain1 để cài đặt wordpress (làm theo hướng dẫn của wordpress)
- Truy cập sub-domain2 để quan sát xem cơ sở dữ liệu có những bảng dữ liệu nào sau khi cài wp
- Tạo 1 bài viết trong wordpress giới thiệu về bản thân sinh viên: thông tin cá nhân, sở thích, ... bài viết có thể chứa hình ảnh, âm thanh, video, ...
- Tạo 1 bài viết trong wordpress giới thiệu về nhữn kiến thức mà em đã học được ở môn **Phát triển ứng dụng với mã nguồn mở**
- Truy cập sub-domain3 để cấu hình n8n:
  + tạo tài khoản admin : nhớ điền đúng email
  + Send me a Licence key, bước này điền đủ thông tin, làm chậm sẽ thấy mục gửi License key về mail (n8n sẽ gửi email KEY cho dùng), check email để lấy KEY
  + Activate License key: vào trang chủ => SETTING (góc dưới trái) => Usage and plan => Enter activation key: paste key từ email vào đây => Activate => sẽ nhận đc thông báo (góc dưới phải) Your Registered Community Edition has been successfully activated.
  + Create workflow  (home page => overview => Create workflow)
  + Add trigger node: tìm node: Telegram => OnMessage  ; cấu hình Credential: Set up Credential => cần Nhập Access Token
    + Access Token thì lấy ở Telegram qua việc chát với @BotFather
    + Cần chát với bot @BotFather để đẻ ra bot mới của riêng mình. bot này sẽ là nơi nhận lệnh (promt) để AI sinh html => n8n sẽ dùng html này để đăng bài lên wp
    + Sau khi tạo bot mới cần copy lấy Token, và chát lần đầu với bot mới này, nội dung bất kỳ (bước này quan trọng!)
  + Add (nối tiếp vào sau node Telegram Trigger) node: AI Google Gemini => Message a model => Set up Credential => cần Nhập API KEY
    + Lấy API KEY tại trang: https://aistudio.google.com  => https://aistudio.google.com/api-keys
    + cần tạo project mới, sẽ lấy được API KEY
    + Nhập API Key lên giao diện n8n
    + kéo thả **nội dung đã chát** với bot của telegram (phía bên trái) vào **nội dung phần PROMPT** kết quả được {{ $json.message.text }}, cần gõ thêm vào sau {{ $json.message.text }} để promt dài hơn : vd ({{ $json.message.text }}. Kết quả sinh ra ở định dạng HTML+CSS để tôi dùng HTML+CSS này tạo bài viết cho wordpress.)
    + Turn on Output Content as JSON : để kết quả trả về dạng json
    + Có thể thử nghiệm các thành phần khác trong Options (add Options: System message, ...) => đưa ra cái nào đáng dùng?
  + Add (nối tiếp vào sau node Message a model) node: Code in JavaScript
    + Code js ở dạng này, có thể phải thay đổi tuỳ theo json AI trả về.
```
// 1. lấy dữ liệu gốc
const rawText = $input.first().json.content.parts[0].text;

// 2. Chuyển đổi chuỗi (đã được bọc JSON) thành Object trong JavaScript
const cleanData = JSON.parse(rawText);

// 3. Trả về kết quả định dạng lại gọn gàng cho n8n sử dụng
return {
  title: cleanData.post_title,
  content: cleanData.post_content
};
```

  + Add (nối tiếp vào sau node Code in JavaScript) node: WordPress => Create a Post
    + Set up Credential: vào wp tại url: https://sub-domain1/wp-admin  => vào mục Tài Khoản => chọn user đã tạo lúc setup wordpress => Mật khẩu ứng dụng => Nhập n8n và bấm "Thêm mật khẩu ứng dụng" => copy chuỗi 24 kí tự : Đây là mật khẩu ứng dụng => paste vào mục Password của n8n Credential
    + Wordpress URL: điền giá trị https://sub-domain1/   (giá trị này cũng khai báo trong biến môi trường WEBHOOK_URL của n8n)
    + Ignore SSL Issues (Insecure): TURN ON
    + Cấu hình node Create a Post: bấm nút Execute previous nodes để thấy trường giá trị của node trước trả về, kéo nội dung phần title (bên trái) vào trường title, tương tự kéo nội dung content vào content
    + Add field (Thêm thuộc tính): Status == Publish (bài đăng sẽ ở trạng thái xuất bản ngay lập tức, mặc định nó ở giá trị Draft bản nháp)
+ PUBLISH flow (góc trên phải) Nút này thực hiện việc xuất bản flow <=> flow sẽ tự động thực thi khi thoả mãn điều kiện trigger
   
+ Kết quả cuối cùng cần đặt được:
  + từ điện thoại, chát với telegram bot
  + nội dung chát được tự động gửi tới node Telegram trigger => Gửi tới Google Gemini Message a model (bản chất là gửi Prompt) : Nhận về json kết quả của Prompt => Gửi sang node Code in JavaScript để tách tiêu đề và nội dung => gửi đến node WordPress để Create a Post(đăng bài) với tiêu đề và nội dung từ node trước gửi sang.
  + f5 wordpress để thấy bài viết mới đã lên sóng.

+ Chụp ảnh quá trình thao tác/cấu hình/các kết quả trung gian đạt được
+ Nhận xét thành quả đạt được!!!


demo kết quả cuối cùng:

chát với bot:

<img width="471" height="264" alt="image" src="https://github.com/user-attachments/assets/7c439503-63b4-4529-bbec-78fa1d4933d6" />

flow automation của n8n (nhìn bên ngoài):

<img width="1319" height="389" alt="image" src="https://github.com/user-attachments/assets/abbdc5af-952f-4d50-8fba-0cafc7334212" />


bài tự động đăng trên wp:

<img width="750" height="817" alt="image" src="https://github.com/user-attachments/assets/4f7c0cec-292f-4973-9eb0-1534189cdb18" />



## BÀI LÀM
- sửa docker-compose.yml (QUAN TRỌNG NHẤT)
Mở file:
cd ~/wordpress-lab
nano docker-compose.yml
- THÊM 2 SERVICE: n8n + cloudflared
<img width="761" height="940" alt="image" src="https://github.com/user-attachments/assets/a3e9ad16-b027-4844-a7ef-f6a3901d37e4" />

- SAU KHI SỬA XONG
> - Chạy lại:
docker compose down
docker compose up -d
<img width="1655" height="733" alt="image" src="https://github.com/user-attachments/assets/15a6a919-68ae-4ba8-a338-e46f3c5ec316" />


- TRUY CẬP CÁC SERVICE
1. WordPress
- https://wp.hoangthixuantrang.id.vn/wp-admin
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/3366a8ea-1aad-4829-9706-ff67f83f3580" />

-  https://wp.hoangthixuantrang.id.vn/
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/451a7639-d293-4698-83e9-1cab4d37ef11" />

2. phpMyAdmin
http://localhost:8081
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/68630c12-b9c1-4aac-acc4-7918c93fdab8" />

3. n8n
http://localhost:5678
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d94e83b7-e633-4a5e-af22-999911ef746d" />


- Cấu hình cloudflare tunnel add router để public WordPress, phpMyAdmin, n8n
  > - sửa file:
nano ~/.cloudflared/config.yml
<img width="746" height="908" alt="image" src="https://github.com/user-attachments/assets/be9664ff-17a9-4e3c-9a2d-7bf79d2426b7" />

- TẠO CLOUDFLARE TUNNEL
- <img width="883" height="237" alt="image" src="https://github.com/user-attachments/assets/e662028b-2a17-4ded-96f8-d024c2af6d8c" />

- GẮN DNS VÀO TUNNEL
  <img width="1466" height="208" alt="image" src="https://github.com/user-attachments/assets/7cd3ed32-5100-466e-8f8f-0bd1def27d79" />

- sau đó chạy lại file docker
- <img width="1463" height="717" alt="image" src="https://github.com/user-attachments/assets/5e63f0f7-f44c-44db-ac02-3f642b8cc9c6" />

- CẤU HÌNH ingress
<img width="746" height="908" alt="image" src="https://github.com/user-attachments/assets/be9664ff-17a9-4e3c-9a2d-7bf79d2426b7" />

- TEST NỘI BỘ DOCKER
<img width="1460" height="339" alt="image" src="https://github.com/user-attachments/assets/0ee58019-e237-4a2d-b978-50a4596b2796" />
- TEST CHUẨN DOCKER NETWORK
- chạy alpine container trong cùng network: docker run --network wordpress-lab_default alpine
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/2deb4012-64cc-4a9c-8b61-9983ae39c3c2" />

- Public HTTPS access OK
- N8N : https://n8n.hoangthixuantrang.id.vn
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/a0389b98-2e93-4b31-8d6a-5528388cee99" />

- phpMyAdmin : https://db.hoangthixuantrang.id.vn/
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/ebd2f133-b55e-408a-87a7-d6b9580ead47" />

------------------------------
- truy cập n8n để cấu hình
-  https://n8n.hoangthixuantrang.id.vn
-  tạo tài khoản admin : nhớ điền đúng email
-  Send me a Licence key, bước này điền đủ thông tin, làm chậm sẽ thấy mục gửi License key về mail (n8n sẽ gửi email KEY cho dùng), check email để lấy KEY
<img width="1179" height="2556" alt="image" src="https://github.com/user-attachments/assets/6022e93b-4793-459f-89da-1d3e9bd879e8" />
- sau đó Activate License key: vào trang chủ => SETTING (góc dưới trái) => Usage and plan => Enter activation key: paste key từ email vào đây => Activate => sẽ nhận đc thông báo (góc dưới phải) Your Registered Community Edition has been successfully activated.

- Thêm node Telegram
Access Token thì lấy ở Telegram qua việc chát với @BotFather
<img width="590" height="1280" alt="image" src="https://github.com/user-attachments/assets/90ecbdd2-d353-463b-aba2-ce55f1470054" />

- Cần chat với bot @BotFather để đẻ ra bot mới của riêng mình. bot này sẽ là nơi nhận lệnh (promt) để AI sinh html => n8n sẽ dùng html này để đăng bài lên wp
- <img width="1179" height="725" alt="image" src="https://github.com/user-attachments/assets/e03b5e9a-eac6-4499-8973-416cfcca92a5" />

- Sau khi tạo bot mới cần copy lấy Token, và chát lần đầu với bot mới này, nội dung bất kỳ (bước này quan trọng!)
  <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/8a10fb08-0094-4d6b-a0e4-3256a3d37452" />

- Lấy API KEY tại trang: https://aistudio.google.com => https://aistudio.google.com/api-keys

Cần tạo project mới, sẽ lấy được API KEY

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/aa2187d9-f5a7-466f-9e09-d8154af7fcb4" />

- Nhập API Key lên giao diện n8n

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/1ab52a9a-a873-4b16-a4c9-054f06fccff5" />


<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/2b74cd49-b362-48b8-abd2-aaea3aea9969" />


- Add (nối tiếp vào sau node Message a model) node: Code in JavaScript

Code js ở dạng này, có thể phải thay đổi tuỳ theo json AI trả về.
// 1. lấy dữ liệu gốc
const rawText = $input.first().json.content.parts[0].text;

// 2. Chuyển đổi chuỗi (đã được bọc JSON) thành Object trong JavaScript
const cleanData = JSON.parse(rawText);

// 3. Trả về kết quả định dạng lại gọn gàng cho n8n sử dụng
return {
 title: cleanData.post_title,
 content: cleanData.post_content
};  

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/c27eb8d1-4fec-446f-85c3-6618f83ccbfa" />

Thêm node Code (JavaScript)

Bấm dấu + bên phải Message a model
Tìm: Code
Chọn: Code in JavaScript
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/66da2796-6342-47d1-b7d0-182e4ba3d5ba" />


- Thêm node WordPress
  > - Bấm dấu + bên phải node Code
      > - Tìm :WordPress
  >     - Chọn  :Create a Post
  Set up Credential: vào wp tại url: https://wp.nguyenthilinhk58.id.vn/wp-admin => vào mục Tài Khoản => chọn user đã tạo lúc setup - wordpress => Mật khẩu ứng dụng => Nhập n8n và bấm "Thêm mật khẩu ứng dụng" => copy chuỗi 24 kí tự : Đây là mật khẩu ứng dụng => paste vào mục Password của n8n Credential
- Wordpress URL: điền giá trị https://wp.hoangthixuantrang.id.vn/ (giá trị này cũng khai báo trong biến môi trường WEBHOOK_URL của n8n)

Ignore SSL Issues (Insecure): TURN ON

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/5336e885-6d30-4a3a-8cf9-7d834e43bb53" />
# KẾT QUẢ BÀI LÀM
- từ điện thoại, chat với telegram bot
- nội dung chát được tự động gửi tới node Telegram trigger => Gửi tới Google Gemini Message a model (bản chất là gửi Prompt) : Nhận về json kết quả của Prompt => Gửi sang node Code in JavaScript để tách tiêu đề và nội dung => gửi đến node WordPress để Create a Post(đăng bài) với tiêu đề và nội dung từ node trước gửi sang.

<img width="590" height="1280" alt="image" src="https://github.com/user-attachments/assets/584139e1-dd1c-4c8b-ab58-49390e79e6a8" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/85bc1532-3793-4943-9383-9ae5a4541f86" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/fd8174c3-4628-4c22-af4e-72b7511ceacc" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/08ef582e-7264-4b3e-bcd9-c501a59a5cb9" />

## Nhận xét thành quả đạt được

Sau khi hoàn thành bài thực hành, em đã xây dựng thành công hệ thống WordPress chạy trên Docker với các dịch vụ MariaDB, phpMyAdmin, Cloudflare Tunnel và n8n. Các container được quản lý tập trung bằng Docker Compose, giúp việc triển khai và vận hành trở nên thuận tiện.

Hệ thống Cloudflare Tunnel hoạt động ổn định, cho phép công khai các dịch vụ WordPress, phpMyAdmin và n8n ra Internet thông qua các subdomain riêng mà không cần mở cổng trên router. Cơ sở dữ liệu MariaDB được WordPress tự động khởi tạo và quản lý thành công.

Đặc biệt, em đã cấu hình thành công workflow tự động hóa trên n8n. Khi người dùng gửi nội dung đến Telegram Bot, dữ liệu sẽ được chuyển đến Google Gemini để sinh nội dung bài viết, sau đó n8n xử lý dữ liệu và tự động đăng bài lên WordPress. Toàn bộ quy trình từ Telegram → Gemini AI → n8n → WordPress được thực hiện tự động mà không cần thao tác thủ công.

Thông qua bài thực hành này, em đã củng cố được kiến thức về:

Docker và Docker Compose.
Quản trị dịch vụ trên Ubuntu Linux.
Cloudflare Tunnel và Reverse Proxy.
WordPress và MariaDB.
Tích hợp API của Telegram và Google Gemini.
Xây dựng hệ thống tự động hóa bằng n8n.
Quy trình triển khai ứng dụng mã nguồn mở trên môi trường thực tế.

Kết quả đạt được cho thấy các công nghệ mã nguồn mở có thể kết hợp hiệu quả để xây dựng một hệ thống quản trị nội dung và tự động hóa hiện đại, dễ triển khai và có khả năng mở rộng cao. Đây là kinh nghiệm thực tế hữu ích cho việc học tập và phát triển các dự án sau này.
