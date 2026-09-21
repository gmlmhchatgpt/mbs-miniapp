# Bảng mapping API ↔ Page/Function (Mini App Trái phiếu MBS trong MB BeeRich)

Nguồn dữ liệu:
- Danh sách 35 API: `Phạm vi phát triển(Phase 1).csv`
- Webapp mock: `mbs-miniapp/` (React + Vite + TypeScript, xem chi tiết route trong `src/App.tsx`)

Ghi chú ký hiệu trạng thái (theo cột "Đơn vị làm" trong file CSV gốc):
- `mới`: API cần xây mới hoàn toàn
- `sửa`: API đã có, cần chỉnh sửa/bổ sung
- `đã có`: API đã có, dùng lại không đổi

> Với các API mà file CSV gốc **chưa cung cấp endpoint cụ thể**, mục Endpoint được đánh dấu **(TBD – cần MBS/MB core xác nhận)** và path trong Swagger chỉ là đề xuất tạm để dev có bộ khung làm việc.

---

## 1. Nhóm Thông tin khách hàng & trạng thái tài khoản

| STT | Tên API | Method & Endpoint | Trạng thái | Trang/Component Frontend sử dụng | Thời điểm gọi |
|---|---|---|---|---|---|
| 1 | API lấy thông tin khách hàng | `GET /v1/customer/info` (TBD) | mới | `Home.tsx`, `open-account/BondIntro.tsx`, `open-account/Consent.tsx` (prefill CIF/CCCD/segment) | Khi vào Mini App / trước khi mở TKCK |
| 2 | API check thông tin KH | `GET /v1/customer/status` (TBD) | mới | `Home.tsx` (điều hướng `goToBond`), `open-account/BondIntro.tsx`, `link-account/Intro.tsx`, `tprl/Notice.tsx`, `AccountInfo.tsx`, `trade/BuyBond.tsx` (gatekeeper `useEffect`) | Mỗi lần cần biết `hasTKCK / isLinked / hasTPRL / ndtcnStatus` — đây là API lõi quyết định điều hướng toàn bộ luồng, tương ứng state `AppStateContext` |
| 28 | API check điều kiện kh mua TP (thông tin KH) | `GET /v1/bond/cust/info` | đã có | `tprl/Notice.tsx` (check trạng thái NĐTCN PRO trước khi mua), `trade/BuyBond.tsx` | Trước khi cho phép đặt lệnh mua |
| 29 | API check điều kiện giao dịch | `GET /v1/bond/cust/trans` | đã có | `trade/BuyBond.tsx` | Ngay trước bước xác nhận đặt lệnh mua |
| 33 | Điều chỉnh thông tin cho MSBOND | *(nội bộ core MBS, không có endpoint FE gọi trực tiếp)* | sửa | Không map trực tiếp tới FE — ảnh hưởng gián tiếp tới dữ liệu trả về ở API 1–4, 28, 29 | — |
| 35 | Hồ sơ người dùng (beerich) | *(nội bộ BeeRich core)* | sửa | `AccountInfo.tsx` (đồng bộ dữ liệu hồ sơ hiển thị) | — |

## 2. Nhóm Mở TKCK & Liên kết tài khoản

| STT | Tên API | Method & Endpoint | Trạng thái | Trang Frontend | Thời điểm gọi |
|---|---|---|---|---|---|
| 25a | API mở TKCK – thu thập thông tin | `POST /mbs-wm/open24/updateCusInfoCollect` | mới | `open-account/ConfirmInfo.tsx` → `open-account/FaceCapture.tsx` → `open-account/CreatePassword.tsx` → `open-account/RegisterService.tsx` | Gửi CCCD OCR (mặt trước/sau), ảnh khuôn mặt, mật khẩu, và các consent khi bấm "Xác nhận" ở màn Đăng ký dịch vụ |
| 18 / 25b | API OTP MBS (dùng chung cho mở TKCK) | `POST /v1/mbb/account/getSmsOTP` (đồng thời `/mbs-wm/mbb/account/getSmsOTP` khi gọi qua kênh mbs-wm) | đã có | `open-account/Otp.tsx` | Khi vào màn OTP, tự động gửi mã |
| 25c | API mở TKCK – xác nhận | `POST /mbs-wm/open24/verifyOtp` | mới | `open-account/Otp.tsx` → `open-account/Success.tsx` | Khi KH nhập đủ 6 số OTP (`OtpInput onComplete`) |
| 26a | API liên kết – kiểm tra khớp thông tin | `POST /mbb/account/checkAccountToLink` | mới | `link-account/Match.tsx`, `link-account/Mismatch.tsx` | Khi vào màn hình liên kết, so khớp Họ tên/CCCD/SĐT giữa MB & MBS |
| 16 | API lấy danh sách các tài khoản thanh toán | `GET /t24-query/v1.0/routine` | mới (MB cung cấp) | `link-account/Match.tsx` (chọn TKNH nếu KH có nhiều tài khoản), `open-account/RegisterService.tsx` (hiển thị TK liên kết mặc định) | Trước khi hiển thị danh sách TKNH để chọn |
| 18 | API OTP MBS (liên kết) | `POST /v1/mbb/account/getSmsOTP` | đã có | `link-account/Otp.tsx` | Khi vào màn SMS OTP của luồng liên kết |
| 26b | API liên kết – xác nhận OTP | `POST /mbb/account/linkOtp` | mới | `link-account/Otp.tsx` → `link-account/Success.tsx` | Khi KH nhập đủ mã OTP |

## 3. Nhóm Đăng ký TK TPRL & NĐTCN

| STT | Tên API | Method & Endpoint | Trạng thái | Trang Frontend | Thời điểm gọi |
|---|---|---|---|---|---|
| 18 | API OTP MBS | `POST /v1/mbb/account/getSmsOTP` | đã có | `tprl/Otp.tsx`, `ndtcn/Otp.tsx` | Khi vào màn OTP tương ứng |
| 27 | API mở TK TPRL | `POST /v1/accounts/care/privateBond` | đã có | `tprl/Register.tsx` → `tprl/Otp.tsx` → `tprl/Success.tsx` | Khi KH đồng ý điều khoản và bấm "Đăng ký" |
| 2 | API check thông tin KH (refresh trạng thái NĐTCN) | `GET /v1/customer/status` | mới | `ndtcn/Status.tsx`, `AccountInfo.tsx` | Khi vào màn tra cứu trạng thái hồ sơ NĐTCN |
| — | Đăng ký NĐTCN (tái sử dụng luồng BeeRich sẵn có) | *(ngoài phạm vi 35 API MBS – dùng API NĐTCN hiện có của BeeRich, chỉ thay CA bằng SMS OTP)* | đã có (BeeRich) | `ndtcn/Intro.tsx` → `ndtcn/Otp.tsx` → `ndtcn/Success.tsx` | Toàn bộ luồng đăng ký NĐTCN |

## 4. Nhóm Tài sản & Thông tin tài khoản

| STT | Tên API | Method & Endpoint | Trạng thái | Trang Frontend | Thời điểm gọi |
|---|---|---|---|---|---|
| 3 | API tài sản | `GET /v1/bond/assets` | mới | `Assets.tsx` (tổng tài sản đầu tư) | Khi vào màn Tổng tài sản, có `isLinked` |
| 4 | API chi tiết tài sản | `GET /v1/bond/asset/detail` | mới | `Assets.tsx` (khi KH bấm xem chi tiết 1 mã trái phiếu — hiện là placeholder, cần bổ sung màn chi tiết) | Khi KH chọn 1 dòng tài sản trái phiếu |
| 5 | API biểu đồ tài sản | dùng chung `GET /v1/bond/assets` | — | `Assets.tsx` (phần biểu đồ, hiện là placeholder) | Cùng lúc với API 3 |
| 34 | Beerich điều chỉnh tài sản (gọi sang MB lấy TP) | *(nội bộ, backend BeeRich gọi MB, không phải API FE gọi trực tiếp)* | mới | Ảnh hưởng gián tiếp dữ liệu hiển thị ở `Assets.tsx` | — |
| 1 | API lấy thông tin khách hàng | `GET /v1/customer/info` | mới | `AccountInfo.tsx` (khối "Thông tin cá nhân") | Khi vào màn Thông tin tài khoản |
| 2 | API check thông tin KH | `GET /v1/customer/status` | mới | `AccountInfo.tsx` (khối "Thông tin dịch vụ" + "Thông tin NĐTCN") | Khi vào màn Thông tin tài khoản |
| 6 | API lịch sử | `GET /v1/bond/trans/history` | đã có | *(chưa có màn hình riêng trong webapp hiện tại — cần bổ sung "Lịch sử giao dịch" trong `AccountInfo.tsx` hoặc trang riêng)* | Khi KH xem lịch sử giao dịch |

## 5. Nhóm Thông báo (Noti) — *ngoài phạm vi 35 API*

`Notifications.tsx` sử dụng hệ thống thông báo chung của BeeRich (không thuộc danh sách 35 API MBS). Theo mock up, hệ thống chỉ cần kiểm tra điều kiện `isLinked` (dựa vào API 2) trước khi hiển thị thông báo liên quan đến MBS.

## 6. Nhóm Bảng chào bán, sản phẩm & Giao dịch mua/bán

> Các màn hình chi tiết cho nhóm này **chưa được mô tả đầy đủ** trong file mock up gốc (chỉ có link Figma "MBS-BeeRich-030926"). Trang `trade/BuyBond.tsx` hiện tại là **placeholder/gatekeeper** cơ bản. Bảng dưới đây map API với **chức năng dự kiến** để FE dựng tiếp khi có Figma chi tiết.

| STT | Tên API | Method & Endpoint | Trạng thái | Trang/Chức năng Frontend (dự kiến) |
|---|---|---|---|---|
| 7 | API bảng chào bán | `GET /v1/bond/offerings` (TBD) | mới | Trang danh sách trái phiếu (chưa dựng) — hiển thị thẻ FIX/FLEXI theo kho |
| 8 | API trả các thông tin bộ lọc | `GET /v1/bond/filters` | sửa | Bộ lọc trên trang danh sách trái phiếu (chưa dựng) |
| 9 | API chi tiết sản phẩm | `GET /v1/bond/info` | mới | Trang chi tiết sản phẩm trái phiếu (chưa dựng) |
| 10 | API lấy list văn kiện TP | `GET /v1/bond/minio/files` | đã có | Tab văn kiện trong trang chi tiết sản phẩm (chưa dựng) |
| 11 | API lấy chi tiết nội dung văn kiện TP | `GET /v1/bond/minio/detail`, `GET /v1/minio/bond/file/page` | đã có | Màn xem chi tiết văn kiện (chưa dựng) |
| 12 | API màn hình thông tin đặt lệnh mua | `GET /v1/bond/detail` | sửa | `trade/BuyBond.tsx` (nội dung form đặt lệnh — cần thay dữ liệu tĩnh bằng API thật) |
| 13 | API minh họa dòng tiền | `GET /v1/bond/detail/data` | đã có | `trade/BuyBond.tsx` (biểu đồ minh họa dòng tiền — chưa dựng UI) |
| 14 | API minh họa chuyển nhượng flexi | `GET /v1/bond/detail/data/flexi` | chưa đánh giá | Chưa có giao diện Flexi tương ứng |
| 15 | API check mã người giới thiệu | `GET /product-sales-support/rm-management/v1.0/employee-info/list` | đã có (MB cung cấp lại) | `trade/BuyBond.tsx` (ô nhập mã giới thiệu — chưa dựng) |
| 17 | API xem trước phụ lục mua | `GET /v1/bond/appendix/contract` | đã có | Màn xem trước phụ lục hợp đồng mua (chưa dựng) |
| 18 | API OTP MBS | `POST /v1/mbb/account/getSmsOTP` | đã có | Màn OTP xác nhận giao dịch mua/bán (chưa dựng) |
| 19 | API verify OTP | `POST /v1/bond/trading/confirm` | đã có | Xác nhận OTP giao dịch mua/bán (chưa dựng) |
| 20 | API chuyển tiền (OTC & TPRL) | `POST /mb-funds-transfer/funds-transfer/v1.0/make/v1.1`, `POST /mb-funds-transfer/funds-transfer/v1.2/revert`, `POST /routing-transfer/v1.0/routing/make` | mới (MB cung cấp) | Bước chuyển tiền trong luồng đặt lệnh mua (chưa dựng) |
| 21 | API xem trước phụ lục bán | `GET /v1/bond/appendix/contract/sell` | đã có | Màn bán trước hạn (chưa dựng) |
| 22 | API mua | `POST /v1/bond/buying/register` | sửa | Xác nhận đặt lệnh mua cuối cùng (chưa dựng) |
| 23 | API lấy thông tin màn hình bán trước hạn | `GET /v1/bond/selling/detail` | đã có | Màn bán trước hạn (chưa dựng) |
| 24 | API bán trước hạn | `POST /v1/bond/selling/register` | đã có | Xác nhận bán trước hạn (chưa dựng) |
| 30 | API xem lại hợp đồng bán | `GET /v1/minio/bond/appendix/sell` | đã có | Lịch sử hợp đồng — xem lại hợp đồng đã bán (chưa dựng) |
| 31 | API xem lại hợp đồng mua | `GET /v1/minio/bond/appendix/history` | đã có | Lịch sử hợp đồng — xem lại hợp đồng đã mua (chưa dựng) |

## 7. Nhóm hạ tầng dùng chung (cross-cutting)

| STT | Tên API | Method & Endpoint | Trạng thái | Áp dụng |
|---|---|---|---|---|
| 32 | API token | `POST /v1/auth/token` (TBD) | mới | Toàn bộ Mini App — khởi tạo phiên đăng nhập/SSO khi MB chuyển KH sang MBS (tương ứng ghi chú "MB gửi thông tin truy cập, xác thực" / "MBS cấp phiên truy cập" xuất hiện ở mọi màn hình mockup) |

---

## Tổng hợp theo trang Frontend đã dựng

| Trang (route) | Các API liên quan |
|---|---|
| `Home.tsx` (`/`) | API 32 (token/SSO), API 1, API 2 |
| `open-account/*` (`/open-account/...`) | API 1, API 25a/25b/25c |
| `link-account/*` (`/link-account/...`) | API 2, API 16, API 18, API 26a, API 26b |
| `tprl/*` (`/tprl/...`) | API 2, API 28, API 18, API 27 |
| `ndtcn/*` (`/ndtcn/...`) | API 2, API 18 (OTP dùng chung), API NĐTCN của BeeRich (ngoài scope) |
| `AccountInfo.tsx` (`/account-info`) | API 1, API 2, API 6 (đề xuất bổ sung), API 35 |
| `Notifications.tsx` (`/notifications`) | Ngoài scope 35 API — chỉ cần API 2 để check điều kiện hiển thị |
| `Assets.tsx` (`/assets`) | API 3, API 4 (đề xuất bổ sung màn chi tiết), API 5, API 34 |
| `trade/BuyBond.tsx` (`/trade/buy-bond`) | API 2, API 28, API 29, và toàn bộ nhóm 6 (7–24, 30, 31) khi màn hình chi tiết được dựng đầy đủ theo Figma |

## Khuyến nghị tiếp theo cho FE

1. Bổ sung màn hình **Lịch sử giao dịch** (API 6) và **Chi tiết tài sản theo mã trái phiếu** (API 4) — hiện `AccountInfo.tsx`/`Assets.tsx` mới có placeholder.
2. Khi có Figma "MBS-BeeRich-030926", cần dựng chi tiết các màn: Danh sách/bảng chào bán trái phiếu, Chi tiết sản phẩm, Đặt lệnh mua (đủ bước chuyển tiền + OTP), Bán trước hạn — tương ứng nhóm API 7–24, 30, 31.
3. Xác nhận với MBS/MB core team các endpoint đang đánh dấu **(TBD)**: API 1 (`/v1/customer/info`), API 2 (`/v1/customer/status`), API 7 (`/v1/bond/offerings`), API 32 (`/v1/auth/token`) — đây là các API mà file CSV gốc chưa cung cấp path cụ thể.
