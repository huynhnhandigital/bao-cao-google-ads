# Báo cáo hiệu quả Digital Ads · Microlife (trang web)

Trang `index.html` đọc trực tiếp Google Sheet **[ML] Báo cáo Google Ads – tự động** bằng tài khoản Google của người xem.

Trang có 3 phần:

- **Tổng thể:** chi phí và lượt hiển thị của Google Ads và Facebook Ads cộng chung; lượt nhấp và kết quả đặt cạnh nhau vì mỗi nền tảng định nghĩa khác nhau.
- **Google Ads:** số liệu từ các tab `GA_` do Google Ads Script ghi.
- **Facebook Ads:** số liệu từ các tab `FB_` do Apps Script `facebook-sang-sheets.gs` ghi.

Trên trang có thể đổi khoảng ngày và lọc theo nhóm sản phẩm. Phần Google Ads lọc thêm được theo tài khoản, chiến dịch và loại chiến dịch; phần Facebook Ads lọc được theo chiến dịch và mục tiêu Meta. Mỗi phần có thẻ KPI so với kỳ trước, biểu đồ xu hướng, phân bổ chi phí, bảng chiến dịch kèm đánh giá và tổng hợp theo tháng.

## Nguồn Facebook Ads (Apps Script trong Sheet)

1. Trong Google Sheet: **Tiện ích mở rộng → Apps Script**, dán nội dung `facebook-sang-sheets.gs`, bấm Lưu, rồi tải lại Sheet.
2. Tạo token Meta: Meta Business Settings → **System users** → gán tài khoản quảng cáo với quyền xem hiệu quả → **Generate new token**, chọn quyền `ads_read`, hạn **Never**.
3. Trong Sheet, chọn menu **Facebook Ads → Nhập / đổi token Meta** rồi dán token. Token chỉ nằm trong Script Properties của dự án Apps Script, không nằm trong ô nào của Sheet, nên người chỉ có quyền xem Sheet không thấy.
4. Chọn **Facebook Ads → Đồng bộ ngay**, rồi **Bật lịch chạy hằng ngày** (khoảng 6–7 giờ sáng).

Script chỉ đọc số liệu từ Meta, mỗi lần chạy lấy lại 30 ngày gần nhất. Tài khoản quảng cáo, ngày bắt đầu và phiên bản Graph API chỉnh trong tab `FB_Cấu hình`. Muốn xem phân bổ theo sản phẩm, điền cột Nhóm sản phẩm trong `FB_Ánh xạ chiến dịch`.

**Về quyền xem:**

- Trong repo không có số liệu nào. Ai mở trang cũng phải bấm **Đăng nhập Google**.
- Chỉ tài khoản được chia sẻ Sheet mới thấy số.
- Trang chỉ xin quyền **đọc** bảng tính (`spreadsheets.readonly`) và không lưu lại số liệu hay token.
- Số liệu mới nhất là lần các script chạy gần nhất (Google Ads khoảng 5–6 giờ sáng, Facebook Ads khoảng 6–7 giờ sáng mỗi ngày), không theo thời gian thực.

---

## Bước 1 – Đưa trang lên GitHub Pages

1. Vào github.com, bấm **New repository**, đặt tên, ví dụ `bao-cao-google-ads`.
   - Với tài khoản GitHub miễn phí, repo phải để **Public** thì mới bật được Pages.
   - Repo chỉ chứa giao diện, số liệu vẫn nằm trong Sheet riêng tư.
2. Trong repo, bấm **Add file → Upload files**, kéo thả `index.html` và `README.md`, rồi bấm **Commit changes**.
3. Vào **Settings → Pages**.
   - Ở mục *Source* chọn **Deploy from a branch**.
   - Branch chọn **main**, thư mục **/ (root)**, rồi bấm **Save**.
4. Sau khoảng 1 phút, trang có địa chỉ dạng `https://<tên-github>.github.io/bao-cao-google-ads/`.
   - Lúc này trang báo *“Chưa cấu hình đăng nhập Google”*. Đó là đúng, vì chưa làm Bước 2.

## Bước 2 – Tạo mã đăng nhập Google (OAuth Client ID, miễn phí)

Đăng nhập console.cloud.google.com bằng tài khoản quản lý Sheet.

1. **Tạo dự án:** bấm chọn dự án ở thanh trên cùng, chọn **New project**, đặt tên ví dụ `bao-cao-ads`, rồi bấm **Create**.
2. **Bật Google Sheets API:** vào **APIs & Services → Library**, tìm **Google Sheets API** và bấm **Enable**.
3. **Thiết lập màn hình đăng nhập:** vào **Google Auth Platform** (tên cũ: *OAuth consent screen*).
   - **Branding:** tên ứng dụng (ví dụ *Báo cáo Google Ads*) và email hỗ trợ.
   - **Audience:**
     - Nếu công ty dùng Google Workspace và mọi người xem đều có email công ty, chọn **Internal**.
     - Nếu không, chọn **External**, giữ trạng thái **Testing**, rồi thêm email từng người xem vào **Test users** (tối đa 100 người).
   - **Data access:** bấm **Add or remove scopes** và thêm `https://www.googleapis.com/auth/spreadsheets.readonly`.
4. **Tạo Client ID:** vào **Clients** (hoặc *Credentials → Create credentials → OAuth client ID*).
   - *Application type:* **Web application**.
   - *Authorized JavaScript origins:* điền `https://<tên-github>.github.io`. Chỉ điền phần gốc, không có đường dẫn repo và không có dấu `/` ở cuối.
   - Bấm **Create**, rồi sao chép **Client ID**, dạng `1234567890-abc...apps.googleusercontent.com`.

Client ID không phải mật khẩu, nên đặt công khai trong file HTML được. Chỉ những địa chỉ khai báo ở *Authorized JavaScript origins* mới dùng được Client ID này.

## Bước 3 – Dán Client ID vào trang

1. Trong repo GitHub, mở `index.html` và bấm biểu tượng bút chì (**Edit**).
2. Tìm phần `CONFIG` ở đầu đoạn script và dán Client ID vào giữa hai dấu nháy:

   ```js
   CLIENT_ID: '1234567890-abc...apps.googleusercontent.com',
   ```

3. Bấm **Commit changes**. Khoảng 1 phút sau, tải lại trang và sẽ thấy nút **Đăng nhập Google**.

## Bước 4 – Cấp quyền xem cho từng người

1. Mở Google Sheet báo cáo, bấm **Chia sẻ**, thêm email người xem với quyền **Người xem**.
2. Nếu ở Bước 2 bạn chọn *External / Testing*, thêm cùng email đó vào **Test users**.
3. Người xem mở trang, bấm **Đăng nhập Google** và chọn đúng tài khoản đó.
   - Ở chế độ Testing, Google hiện cảnh báo *“Google hasn't verified this app”*. Bấm **Continue** là được, vì đây là ứng dụng nội bộ của bạn.

Muốn thu hồi quyền của ai, chỉ cần bỏ chia sẻ Sheet với người đó (và xoá họ khỏi Test users).

---

## Lỗi thường gặp

| Thông báo trên trang | Cách xử lý |
|---|---|
| Chưa cấu hình đăng nhập Google | Chưa dán `CLIENT_ID` (Bước 3). |
| Cửa sổ Google báo `origin_mismatch` hoặc `redirect_uri_mismatch` | Địa chỉ trang chưa có trong *Authorized JavaScript origins*, hoặc bị điền thừa đường dẫn hay dấu `/`. |
| Google Sheets API chưa được bật | Làm lại Bước 2, mục 2, trong đúng dự án chứa Client ID. |
| Tài khoản chưa được chia sẻ quyền xem | Chia sẻ Sheet cho email đó (Bước 4). |
| Không tìm thấy Google Sheet | Kiểm tra `SPREADSHEET_ID` trong `CONFIG`. |
| Không mở được cửa sổ đăng nhập | Cho phép cửa sổ bật lên (popup) với trang này. |

Mở file `index.html` trực tiếp trên máy (`file://…`) thì không đăng nhập được, vì Google chỉ chấp nhận địa chỉ đã khai báo. Hãy xem qua địa chỉ GitHub Pages.

## Nguồn số liệu và cách tính

Trang đọc 5 tab do script Google Ads ghi:

- `GA_Dữ liệu ngày`
- `GA_Chuyển đổi theo action`
- `GA_Ánh xạ chuyển đổi`
- `GA_Cấu hình`
- `GA_Nhật ký`

và 3 tab do Apps Script Facebook ghi: `FB_Dữ liệu ngày`, `FB_Ánh xạ chiến dịch`, `FB_Nhật ký`. Khi Sheet chưa có các tab `FB_`, trang vẫn chạy với riêng Google Ads và ghi chú rõ điều đó.

Trang tự tính lại mọi chỉ số theo đúng quy tắc của tab GA_Dashboard trong Sheet:

- Kỳ trước có cùng số ngày với kỳ đang xem.
- Ô nào thiếu số thì ghi “Chưa có dữ liệu”, không điền 0.
- Lead và Purchase chỉ được tính khi conversion action đã được xác nhận loại.
- Chi phí không bị cộng lặp khi ghép số chuyển đổi theo action.
- Ngưỡng đánh giá chiến dịch lấy từ GA_Cấu hình.
- Ký hiệu ₫ chỉ hiện khi tài khoản dùng VND.

Nếu đổi tên tab hoặc tên cột trong Sheet, trang sẽ báo rõ thiếu cột nào.
