# Chuyên mục blog — đặc tả Markdown trả về cho AI agent

> Ngày: 2026-11-11 · Phạm vi: trang chuyên mục `/blogs/{topic}` (9 trang: `an-ninh`, `cam-bien`, `chieu-sang`, `du-an`, `huong-dan`, `hvac`, `kien-thuc`, `knx`, `tin-tuc`) · Trạng thái: **bản nháp**.
> Ngoài phạm vi: bài blog (xem `blog-md-spec/`) và trang tác giả. Trang `/blogs` (gốc) **không** thuộc phạm vi: cùng giao diện nhưng khác cấu trúc (11 bài mới nhất theo ngày, không có nhãn đếm, một số thẻ không có nhãn chuyên mục).
> File này mô tả **kết quả mong muốn** (MD trông như thế nào); mục 7 là ghi chú ngắn cho dev. Dựa trên đo cả 9 chuyên mục và một bộ chuyển đổi tham chiếu chạy trên cả 9 trang (mục 7). Số token là ước tính bytes/4. Cùng khung với `product-md-spec/`, `product-category-md-spec/`, `blog-md-spec/`.

---

## 1. Nguyên tắc

1. **Cùng thông tin với HTML, khác định dạng.** Không tóm tắt, không thêm thông tin không có trên trang.
2. **Danh sách bài đầy đủ, không cắt, đúng thứ tự trang.** Trang không phân trang (mọi bài của chuyên mục nằm trong một HTML, tới 84 bài).
3. **Chỉ nội dung chuyên mục.** Không menu, chân trang, giỏ báo giá, ô tìm bài, ảnh thẻ bài.
4. Khối rỗng hoặc giá trị `null` thì bỏ hẳn heading/dòng; không ghi "Không có", "N/A".
5. Mọi liên kết là URL tuyệt đối `https://knxstore.vn/...`.
6. **Gọn.** MD từ 1,8 KB đến 14,5 KB (~460 đến ~3.600 token); HTML thô 310 KB đến 633 KB; giảm 97,7% đến 99,4%.

---

## 2. Frontmatter

| Key | Giá trị | Điều kiện | Ưu tiên |
|---|---|---|---|
| `type` | `blog-topic` | luôn | P3 |
| `title` | tên chuyên mục (giống H1) | luôn | P1 |
| `description` | meta description nguyên văn | có | P3 |
| `canonical` | URL HTML chuẩn, tuyệt đối | luôn | P1 |
| `lang` | `vi` | luôn | P3 |
| `post_count` | số bài của chuyên mục (nhãn "N bài viết" trên trang) | luôn | P1 |

- `description` là chuỗi chung của cả site, giống hệt ở 9/9 chuyên mục ("Nhà phân phối thiết bị KNX, Casambi, DALI và giải pháp nhà thông minh. Tra cứu thông số kỹ thuật, tải catalog, gửi yêu cầu báo giá."), không mô tả từng chuyên mục. Xem mục 6, điểm 1.
- Không có `image` (ảnh `og:image` là ảnh chung của site) và không có ngày.
- Key frontmatter bằng tiếng Anh; heading trong body bằng tiếng Việt (giống trang). Giá trị có `: ` hoặc dấu `"` thì đặt trong dấu nháy kép.

---

## 3. Body

### 3.1 Thứ tự và heading

| # | Khối | Heading | Điều kiện hiện | Ưu tiên |
|---|---|---|---|---|
| 1 | Tên chuyên mục | `#` (H1 duy nhất) | luôn | P1 |
| 2 | Breadcrumb | không heading; 1 dòng `Trang chủ > Blog kỹ thuật > {Chuyên mục}` | luôn | P3 |
| 3 | Đoạn giới thiệu | không heading; một đoạn văn | luôn | P3 |
| 4 | Thông tin nhanh | không heading; bảng 2 cột | luôn | P1 |
| 5 | Chuyên mục khác | `## Chuyên mục khác` | luôn | P2 |
| 6 | Bài viết | `## Bài viết` | luôn | P1 |

### 3.2 Quy tắc chung

- Link nội bộ dạng tuyệt đối; UTF-8, LF; không emoji; không HTML thô.
- Không đưa text giao diện ("Tìm bài viết...", "Yêu cầu báo giá", "Bỏ qua, tới nội dung chính").
- Không chuẩn hóa chữ của trang (giữ nguyên hoa thường, dấu câu của tiêu đề bài).
- Ký tự `*`, `[`, `]`, `` ` `` trong chữ lấy từ dữ liệu được escape bằng `\`.
- Dấu cách không ngắt (NBSP) thành dấu cách thường; ký tự định dạng vô hình (U+200B đến U+200F, U+202A đến U+202E, U+2060, U+FEFF) bị bỏ; dấu cách đơn dư cuối dòng bị bỏ (ngắt dòng cứng từ `<br>` giữ hai dấu cách).

### 3.3 Breadcrumb, đoạn giới thiệu, thông tin nhanh

Breadcrumb: các mục trước mục cuối là liên kết, mục cuối là chữ thường:

```markdown
[Trang chủ](https://knxstore.vn/) > [Blog kỹ thuật](https://knxstore.vn/blogs) > An ninh
```

Đoạn giới thiệu là đoạn văn trang đặt ngay dưới H1, chung cho mọi chuyên mục: "Kiến thức, hướng dẫn cấu hình và tin tức sản phẩm về KNX, DALI-2, BACnet, Casambi và Matter Smarthome, viết cho kỹ sư tích hợp hệ thống và đội thi công M&E." (giữ nguyên chữ, escape ký tự đặc biệt nếu có).

Thông tin nhanh là bảng 2 cột (cột tiêu đề rỗng) với một dòng:

| Dòng | Giá trị |
|---|---|
| Số bài viết | `14` (bằng `post_count`; luôn bằng số bài trong mục Bài viết) |

### 3.4 Chuyên mục khác

Các chuyên mục mà trang hiển thị dạng nút lọc (thực chất là liên kết), **trừ chuyên mục đang xem**, đúng thứ tự trang; mục đầu là "Tất cả" trỏ `/blogs`:

```markdown
- [Tất cả](https://knxstore.vn/blogs)
- [Cảm biến](https://knxstore.vn/blogs/cam-bien)
- [Chiếu sáng](https://knxstore.vn/blogs/chieu-sang)
…
```

Danh sách này giống nhau ở mọi chuyên mục (9 chuyên mục + "Tất cả"), nên mỗi trang có 9 dòng.

### 3.5 Bài viết

Mỗi bài một dòng, đúng thứ tự trang, không cắt:

```markdown
- [Camera Aqara G5 Pro Hay Eve Cam? Nên Chọn Loại Nào](https://knxstore.vn/camera-chong-trom-la-gi.html) — 12/8/2026
- [RCU Khách Sạn Cho Chủ Đầu Tư Và ME: Phân Loại Và Cách Chọn.](https://knxstore.vn/rcu-khach-san-cho-chu-dau-tu-va-me-phan-loai-va-cach-chon.html) — HVAC · 22/10/2026
```

| Thành phần | Quy tắc |
|---|---|
| Tiêu đề | nguyên văn, là liên kết tới `/{slug}.html` |
| Ngày | `d/m/yyyy` đúng như thẻ trên trang (không đệm số 0; là ngày UTC, giống ngày hiển thị trong bài) |
| Nhãn chuyên mục | **chỉ khi khác tên chuyên mục đang xem**, đặt trước ngày, cách bằng ` · `. Ở 8/9 chuyên mục mọi thẻ có nhãn trùng tên chuyên mục nên bỏ; riêng `/blogs/knx` có 2/12 bài mang nhãn khác ("HVAC", "Kiến thức") |
| Thứ tự | đúng thứ tự trang; trang không sắp theo ngày đăng (cũng không theo ngày cập nhật, đã kiểm 14 bài của `an-ninh`) |
| Không đưa | ảnh, mô tả, tác giả (thẻ không có các thông tin này) |

### 3.6 URL biến thể

| URL yêu cầu | Bản MD |
|---|---|
| `/blogs/{topic}` | bản MD của chuyên mục |
| `/blogs/{topic}?page=N` | giống hệt bản gốc, `canonical` vẫn là URL gốc (site bỏ qua `page`: trả 200 cùng một danh sách) |
| `/blogs/{topic}/` | chuyển hướng 308 về URL không có `/` (giống HTML) |
| chuyên mục không tồn tại, viết hoa (`/blogs/AN-NINH`) | 404 (giống HTML) |

---

## 4. Không đưa vào MD

| Loại bỏ | Lý do |
|---|---|
| Header (menu ~165 liên kết), footer, nút Zalo nổi, giỏ/drawer báo giá | Điều hướng lặp trên mọi trang |
| Ô "Tìm bài viết" (form GET `/search?kind=post&q=`) | Giao diện; đường dẫn tìm kiếm bị `robots.txt` chặn (`Disallow: /search?`), không đưa vào MD |
| Ảnh thẻ bài | Không có thông tin ngoài tiêu đề (alt = tiêu đề) |
| `og:image` (ảnh chung của site) | Không riêng cho chuyên mục |
| JSON-LD | Trang không có JSON-LD |

---

## 5. Ví dụ hoàn chỉnh

Chuyên mục nhỏ nhất (`/blogs/du-an`, 4 bài). Toàn bộ output, 1.849 B:

````markdown
---
type: blog-topic
title: Dự án
description: Nhà phân phối thiết bị KNX, Casambi, DALI và giải pháp nhà thông minh. Tra cứu thông số kỹ thuật, tải catalog, gửi yêu cầu báo giá.
canonical: https://knxstore.vn/blogs/du-an
lang: vi
post_count: 4
---

# Dự án

[Trang chủ](https://knxstore.vn/) > [Blog kỹ thuật](https://knxstore.vn/blogs) > Dự án

Kiến thức, hướng dẫn cấu hình và tin tức sản phẩm về KNX, DALI-2, BACnet, Casambi và Matter Smarthome, viết cho kỹ sư tích hợp hệ thống và đội thi công M&E.

| | |
|---|---|
| Số bài viết | 4 |

## Chuyên mục khác

- [Tất cả](https://knxstore.vn/blogs)
- [An ninh](https://knxstore.vn/blogs/an-ninh)
- [Cảm biến](https://knxstore.vn/blogs/cam-bien)
- [Chiếu sáng](https://knxstore.vn/blogs/chieu-sang)
- [Hướng dẫn](https://knxstore.vn/blogs/huong-dan)
- [HVAC](https://knxstore.vn/blogs/hvac)
- [Kiến thức](https://knxstore.vn/blogs/kien-thuc)
- [KNX](https://knxstore.vn/blogs/knx)
- [Tin tức](https://knxstore.vn/blogs/tin-tuc)

## Bài viết

- [Dự án điều khiển chiếu sáng Casambi cho nhà hàng tại 79 Mạc Thị Bưởi, TP.HCM](https://knxstore.vn/du-an-dieu-khien-chieu-sang-casambi-cho-nha-hang-tai-79-mac-thi-buoi-tp-hcm.html) — 29/10/2026
- [Casambi điều khiển DMX cho không gian tiệc cưới Sunwah Pearl](https://knxstore.vn/casambi-dieu-khien-dmx-cho-khong-gian-tiec-cuoi-sunwah-pearl.html) — 6/7/2026
- [Casambi Case Study | Dự án chiếu sáng tại Vegi Resort](https://knxstore.vn/casambi-case-study-du-an-chieu-sang-vegi-resort.html) — 13/5/2026
- [Casambi Case Study | Hệ thống chiếu sáng văn phòng The Access Group](https://knxstore.vn/casambi-case-study-he-thong-chieu-sang-van-phong-the-access-group.html) — 14/8/2026
````

Chuyên mục lớn nhất (`/blogs/kien-thuc`, 84 bài) cho MD 14,5 KB (~3.600 token) so với HTML 632,5 KB.

---

## 6. Quyết định còn mở về output

| # | Điểm | Mặc định trong spec này | Người chốt |
|---|---|---|---|
| 1 | Có giữ `description` (meta) trong frontmatter | Có, vì cùng quy ước các loại trang khác; nhưng chuỗi này chung cả site nên bỏ đi cũng không mất thông tin riêng của chuyên mục | Tùng |
| 2 | Có giữ đoạn giới thiệu chung dưới H1 | Có, vì trang hiển thị; đoạn này giống nhau ở mọi chuyên mục (~160 ký tự) | Tùng |
| 3 | Nhãn chuyên mục trên từng dòng bài | Chỉ khi khác tên chuyên mục (tiết kiệm token, không mất thông tin) | Tùng |
| 4 | Ngày dạng `d/m/yyyy` như trang | Giữ như trang (có thể gây nhầm ngày/tháng); không xuất ISO vì thẻ chỉ có ngày | Tùng |
| 5 | Mục "Chuyên mục khác" lặp giống nhau ở 9 trang (9 dòng, 461 B) | Giữ, vì là đường đi giữa các chuyên mục | Tùng |

---

## 7. Ghi chú cho dev (ngắn)

- **Mọi nội dung có sẵn trong HTML ban đầu**, không phải client-only: tên, đoạn giới thiệu, nhãn đếm, chip chuyên mục, danh sách bài. Thẻ bài là `a.group` (không phải `article`) nằm trong `div[data-blog-grid]`: nhãn chuyên mục là `span` đầu, tiêu đề là `p`, ngày là chữ "Ngày đăng" kèm `b` d/m/yyyy. Chip nằm trong `div[data-blog-filters]` (liên kết `a`, chip đang xem có `aria-current=page`). Nhãn đếm là `p` "N bài viết". Trang không có `<main>`; vùng nội dung là `div#main-content`.
- **Số đếm khớp danh sách** ở 9/9 chuyên mục (14, 11, 57, 4, 7, 23, 84, 12, 20 bài). Tổng 232 lớn hơn 227 bài trong `sitemaps/posts.xml` vì 9 bài xuất hiện ở hơn 1 chuyên mục.
- **Không phân trang**: mọi bài trong một HTML; `?page=N` bị bỏ qua. Nếu site sau này thêm phân trang, dùng cùng quy tắc với trang danh mục sản phẩm (`page`, `page_count`, mục Phân trang).
- **Thứ tự thẻ không theo ngày đăng và không theo ngày cập nhật** (đã kiểm 14 bài của `an-ninh` qua JSON-LD của từng bài); tiêu chí chưa xác định.
- **Không có JSON-LD**, không `rel=prev/next`; `<title>` dạng `{Tên} | knxstore.vn`, `og:title` chỉ là tên.
- **HTML đã đo**: 310.444 B (`du-an`) đến 632.537 B (`kien-thuc`); kích thước tăng theo số thẻ vì mỗi ảnh thẻ có `srcSet` dài (~1,2 KB).
- **Bộ chuyển đổi tham chiếu và kiểm tra chéo** (script ở thư mục tạm của phiên: `topicmd.py`, `topicmd_check.py`; chưa đưa vào dự án): chạy trên cả 9 chuyên mục, mỗi trang đạt cả 21 kiểm tra (frontmatter khớp `canonical`, H1, `og:title`, meta description; breadcrumb; đoạn giới thiệu; số đếm = số thẻ = `post_count`; chip khớp DOM trừ chip đang xem; số dòng, thứ tự URL, tiêu đề, ngày của bài; nhãn chuyên mục chỉ khi khác; không còn thẻ HTML/NBSP/chuỗi giao diện; dựng hai lần cho cùng byte). Với `/blogs` (gốc), bộ chuyển đổi báo lỗi có chủ đích vì khác cấu trúc.
- **Phản hồi bất thường** (HTML thiếu nội dung ở lần tải đầu, đã gặp ở trang sản phẩm, danh mục và bài): không gặp ở 10 lần tải lần này; vẫn nên báo lỗi khi thiếu `h1` hoặc `[data-blog-grid]`, không xuất MD rỗng.
