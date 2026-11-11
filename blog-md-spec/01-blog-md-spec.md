# Bài blog — đặc tả Markdown trả về cho AI agent

> Ngày: 2026-11-11 · Phạm vi: bài blog `/{slug}.html` (227 bài trong `sitemaps/posts.xml`) · Trạng thái: **bản nháp**.
> Ngoài phạm vi (cần spec riêng nếu làm): trang chuyên mục `/blogs/{topic}` (9 trang) và trang tác giả `/author/{slug}` (2 trang).
> File này mô tả **kết quả mong muốn** (MD trông như thế nào); mục 7 là ghi chú ngắn cho dev. Dựa trên khảo sát 30 bài và một bộ chuyển đổi tham chiếu chạy trên cả 30 bài (mục 7). Số token là ước tính bytes/4, chưa đo bằng tokenizer. Cùng khung với `product-md-spec/` và `product-category-md-spec/`.

---

## 1. Nguyên tắc

1. **Cùng thông tin với HTML, khác định dạng.** Không tóm tắt, không diễn giải, không thêm thông tin không có trên trang.
2. **Thân bài đầy đủ và đúng thứ tự**, kể cả các khối sản phẩm trang chèn giữa các đoạn văn.
3. **Chỉ nội dung bài.** Không menu, chân trang, nút chia sẻ, khối tư vấn bên phải, mục lục (chỉ dựng ở client), JSON-LD.
4. Khối rỗng hoặc giá trị `null` thì bỏ hẳn heading/dòng; không ghi "Không có", "N/A".
5. Mọi liên kết và ảnh là URL tuyệt đối `https://knxstore.vn/...`.
6. Không đưa tác giả khi trang không hiển thị tác giả.
7. **Gọn.** Trung vị ~3.600 token mỗi bài (HTML thô trung vị ~94.000 token); giảm 92,4% đến 99,3%, trung vị 96,4%.

---

## 2. Frontmatter

| Key | Giá trị | Điều kiện | Ưu tiên |
|---|---|---|---|
| `type` | `article` | luôn | P3 |
| `title` | tên bài (giống H1) | luôn | P1 |
| `description` | meta description nguyên văn | có | P2 |
| `canonical` | URL HTML chuẩn, tuyệt đối | luôn | P1 |
| `lang` | `vi` | luôn | P3 |
| `category` | tên chuyên mục (từ breadcrumb) | có | P2 |
| `author` | tên tác giả | chỉ khi trang hiển thị tác giả | P2 |
| `published` | ngày đăng, ISO 8601 UTC nguyên văn (ví dụ `2026-08-03T03:34:19.000Z`) | có | P1 |
| `modified` | ngày cập nhật, ISO 8601 UTC | chỉ khi khác `published` | P3 |
| `image` | ảnh đại diện (`og:image`), URL tuyệt đối | có | P3 |

- `description` giữ nguyên chuỗi nguồn, kể cả khi bị cắt và kết thúc bằng "…" (5/30 bài).
- `published` giữ cả giờ vì trang hiển thị ngày theo **UTC**, không theo giờ Việt Nam (bài đăng 22:34 UTC ngày 8/4/2025 hiển thị "8/4/2025" dù là 9/4 theo giờ Việt Nam).
- `modified` chỉ có trong dữ liệu công khai (JSON-LD), trang không hiển thị: xem mục 6, điểm 1.
- Key frontmatter bằng tiếng Anh; heading trong body bằng tiếng Việt (giống trang). Giá trị có `: ` hoặc dấu `"` thì đặt trong dấu nháy kép (YAML hợp lệ).

---

## 3. Body

### 3.1 Thứ tự và heading

| # | Khối | Heading | Điều kiện hiện | Ưu tiên |
|---|---|---|---|---|
| 1 | Tên bài | `#` (H1 duy nhất) | luôn | P1 |
| 2 | Breadcrumb | không heading; 1 dòng `A > B > C > Tên bài` | luôn | P2 |
| 3 | Thông tin nhanh | không heading; bảng 2 cột | luôn (có ít nhất dòng "Ngày đăng") | P1 |
| 4 | Nội dung bài | heading của chính bài (`##`, `###`…) | luôn | P1 |
| 5 | Sản phẩm trong bài | không heading; dòng `**Sản phẩm trong bài:**` + danh sách, **xen giữa nội dung bài đúng vị trí trang** | bài có khối sản phẩm | P2 |
| 6 | FAQ | `## FAQ` | bài có FAQ | P2 |
| 7 | Bài viết liên quan | `## Bài viết liên quan` | có | P3 |

### 3.2 Quy tắc chung

- Link nội bộ dạng tuyệt đối; giữ nguyên `tel:`, `mailto:` và link ngoài.
- UTF-8, LF; không emoji; không HTML thô (ngoại lệ duy nhất: `<br>` trong ô bảng).
- Không chuẩn hóa nội dung: giữ nguyên chữ, số, đơn vị của bài (kể cả lỗi chính tả, chữ in đậm cắt giữa từ như `c**ần**`).
- Ký tự `*`, `[`, `]`, `` ` `` trong chữ lấy từ dữ liệu được escape bằng `\`.
- Dấu cách không ngắt (NBSP) thành dấu cách thường; ký tự định dạng vô hình (U+200B đến U+200F, U+202A đến U+202E, U+FEFF) bị bỏ (đã gặp ký tự U+200E đứng đầu một mã SKU).

### 3.3 Breadcrumb và Thông tin nhanh

Breadcrumb lấy đúng chữ trên trang (`Trang chủ > Blog kỹ thuật > {Chuyên mục} > {Tên bài}`); các mục trước mục cuối là liên kết, mục cuối là chữ thường:

```markdown
[Trang chủ](https://knxstore.vn/) > [Blog kỹ thuật](https://knxstore.vn/blogs) > [Kiến thức](https://knxstore.vn/blogs/kien-thuc) > Matter Fabric là gì?
```

Thông tin nhanh là bảng 2 cột (cột tiêu đề rỗng):

| Dòng | Giá trị | Điều kiện |
|---|---|---|
| Chuyên mục | `[Kiến thức](https://knxstore.vn/blogs/kien-thuc)` | có |
| Tác giả | `[Tạ Minh Vũ](https://knxstore.vn/author/ta-minh-vu)` | trang hiển thị tác giả (19/30 bài) |
| Ngày đăng | `3/8/2026` (đúng như trang: ngày/tháng/năm, không đệm số 0, ngày UTC) | có |

### 3.4 Nội dung bài

Thân bài là HTML do CMS nhập, chuyển theo bảng sau. Heading giữ **đúng cấp của bài** vì thân bài là phần chính của trang (trang cũng đặt các `h2` của bài ngang với các mục FAQ, bài liên quan).

| Phần tử | Kết quả |
|---|---|
| Heading `h2`, `h3`, … | giữ cấp (`h2`→`##`, `h3`→`###`); không nhảy quá 1 cấp so với heading trước (sau H1, heading đầu tiên là `##`); nếu bài có `h1` thì mọi heading xuống thêm 1 cấp (chưa gặp ở 30 bài). Heading chứa ảnh: chữ giữ trong heading, ảnh tách thành đoạn ngay sau. Heading chứa liên kết: giữ liên kết (`### [Slim Dual Luna Pet](URL)`). `br` trong heading thành dấu cách |
| Đoạn văn | giữ nguyên; đoạn rỗng bỏ |
| Danh sách | `-` hoặc `1.`; danh sách lồng nhau thụt lề |
| Bảng | bảng Markdown (GFM), số hàng và số cột bằng trang; bảng không có `thead` thì hàng đầu làm hàng tiêu đề; `\|` trong ô được escape; xuống dòng trong ô thành `<br>`; bảng CMS dùng làm FAQ vẫn là bảng thường |
| Ảnh | `![alt](URL gốc)`; giữ `alt` nguyên văn (alt thường do CMS tự sinh dạng "{tên bài} 1", "{tên bài} 2"); URL `/media/...` thành `https://knxstore.vn/media/...`; bỏ `style` (chiều rộng, cao) |
| Liên kết | giữ chữ hiển thị. `/x.html` thành tuyệt đối. Liên kết tương đối **không có `/` đầu và không có đuôi** (ví dụ `sensor-la-gi`, đã gặp 4 liên kết ở 1 bài) thành `https://knxstore.vn/sensor-la-gi.html` (site chuyển hướng 308 sang `.html`; đã kiểm 4/4). Bỏ `target`, `rel` |
| Trích dẫn `blockquote` | dòng bắt đầu bằng `> ` |
| Video `iframe` (YouTube) | một dòng riêng `[Video: {title của iframe}]({src})`, giữ nguyên `src` |
| `strong`, `em`, `br` | `**…**`, `*…*`, ngắt dòng cứng |
| `span`, `div`, `class`, `style`, `id` | bỏ, giữ chữ |
| `script`, `style` | bỏ |

### 3.5 Sản phẩm trong bài

Bài mới thường chèn các khối thẻ sản phẩm (1 đến 9 thẻ mỗi khối, phổ biến 1 đến 4) giữa các đoạn văn. Mỗi khối thành một đoạn `**Sản phẩm trong bài:**` cùng danh sách, đặt đúng vị trí giữa hai đoạn văn như trên trang:

```markdown
**Sản phẩm trong bài:**

- [Đèn LED Bulb T2 (E27, RGB CCT) Matter Aqara - LB-L02D](https://knxstore.vn/products/den-led-bulb-t2-e27-rgb-cct-matter-aqara-lb-l02d) — Aqara · LB-L02D · 542.000 đ (giá cũ 602.000 đ)
- [Bộ điều khiển trung tâm Homey Pro - Matter & Thread, Z-Wave, Zigbee, Wi-Fi, BLE, IR](https://knxstore.vn/products/bo-dieu-khien-trung-tam-homey-pro-matter-thread-z-wave-zigbee-wi-fi-ble-ir) — Homey · HOMEY-PRO-EU-03 · 16.900.000 đ
```

| Thành phần | Quy tắc |
|---|---|
| Tên, thương hiệu | tên sản phẩm nguyên văn; thương hiệu của sản phẩm |
| SKU | chỉ khi thẻ hiển thị mã model (thẻ trong bài có mã) |
| Giá | giống dòng sản phẩm của trang danh mục: `3.190.000 đ` (dấu chấm nghìn, một khoảng trắng); khoảng giá `từ 600.000 đ`; không giá `Liên hệ`; giá ẩn `Giá liên hệ`; giá cũ ` (giá cũ 602.000 đ)`; không ghi phần trăm giảm |
| Không đưa | ảnh, nút "Thêm vào báo giá", nhãn tồn kho |
| Thứ tự | thứ tự thẻ trong khối; mỗi khối một đoạn riêng; sản phẩm xuất hiện ở nhiều khối thì ghi ở từng khối |

### 3.6 FAQ

Nếu trang có mục "Câu hỏi thường gặp" riêng (dưới thân bài), mỗi Q&A một mục danh sách, không dùng heading; đáp án nhiều đoạn thì thụt lề 2 dấu cách:

```markdown
## FAQ

- **Matter Fabric khác Matter Controller như thế nào?** Matter Fabric là mạng kết nối khép kín gồm các thiết bị đã được xác thực, …
```

Một số bài tự viết phần hỏi đáp ngay trong thân bài (ví dụ dưới dạng bảng, hoặc heading "FAQ – Câu hỏi thường gặp về …"): phần đó thuộc nội dung bài, chuyển theo 3.4, không gộp vào mục `## FAQ`.

### 3.7 Bài viết liên quan

Danh sách các bài trang gợi ý ở cuối (thường 4 bài; 30/30 bài có), đúng thứ tự trang:

```markdown
- [IC Là Gì? Cấu Tạo, Chức Năng Và Vai Trò Trong Smarthome](https://knxstore.vn/ic-la-gi-vai-tro-cua-ic-trong-thiet-bi-smarthome.html) — Kiến thức · 17/8/2026
```

Mỗi dòng: tiêu đề (liên kết), chuyên mục, ngày đăng đúng như thẻ trên trang. Không có tác giả, mô tả hay ảnh.

---

## 4. Không đưa vào MD

| Loại bỏ | Lý do |
|---|---|
| Header (menu ~165 liên kết), footer, nút Zalo nổi, giỏ/drawer báo giá | Điều hướng lặp trên mọi trang |
| Nút chia sẻ (Facebook, Instagram, LinkedIn, "Chép link") | Giao diện |
| Khối "Cần tư vấn thiết bị…" (hotline, "Tư vấn dự án") ở cột bên | Khối chung mọi bài, không phải nội dung bài |
| Mục lục | Chỉ dựng ở client sau khi trang chạy; không có trong HTML ban đầu |
| Ảnh đại diện đầu bài | Đã có ở frontmatter `image` (alt chỉ lặp tên bài) |
| JSON-LD (`BlogPosting`, `WebPage`, `FAQPage`, `Person`, `Organization`) | Thông tin đã có trong MD; thống nhất không đưa JSON-LD |
| `meta keywords` (= tên chuyên mục), `meta author` ("KNX Store" khi bài không hiển thị tác giả) | Trùng hoặc là giá trị mặc định của site |

---

## 5. Ví dụ hoàn chỉnh

Bài ngắn nhất trong mẫu khảo sát (`/bai-thuyet-trinh-10-phut-ve-he-thong-an-ninh-thong-minh-satel.html`: bài cũ, không tác giả, có `modified`, thân bài chỉ có vài đoạn và một video). Toàn bộ output, 2.344 B:

````markdown
---
type: article
title: Bài thuyết trình 10 phút về hệ thống an ninh thông minh Satel
description: Bài thuyết trình 10 phút về Hệ Thống An Ninh Thông Minh Satel bởi diễn giả Phạm Đức Độ. Đây là một hệ thống đạt tiêu chuẩn Châu Âu được phân phối bởi KNXStore.
canonical: https://knxstore.vn/bai-thuyet-trinh-10-phut-ve-he-thong-an-ninh-thong-minh-satel.html
lang: vi
category: Kiến thức
published: 2025-04-08T07:55:00.000Z
modified: 2026-10-05T06:37:17.588Z
image: https://knxstore.vn/media/79ba8ab5ba60912a847c2c6276e89a41ab5ee28040915f519d676ea2ef8f24e8.jpg
---

# Bài thuyết trình 10 phút về hệ thống an ninh thông minh Satel

[Trang chủ](https://knxstore.vn/) > [Blog kỹ thuật](https://knxstore.vn/blogs) > [Kiến thức](https://knxstore.vn/blogs/kien-thuc) > Bài thuyết trình 10 phút về hệ thống an ninh thông minh Satel

| | |
|---|---|
| Chuyên mục | [Kiến thức](https://knxstore.vn/blogs/kien-thuc) |
| Ngày đăng | 8/4/2025 |

Diễn giả: Phạm Đức Độ

Ngày 24 tháng 8 năm 2018.

Tại: Khách sạn Kỳ Hòa 238 Đường 3 tháng 2, quận 10, Thành phố Hồ Chí Minh

Một hệ thống đạt tiêu chuẩn Châu Âu,

Chúng tôi chuyên cung cấp và phân phối sỉ lẻ đến các:

\+ Đại lý

\+ Các công ty tích hợp hệ thống điện thông minh

[Video: YouTube video player](https://www.youtube.com/embed/5ug7ZUzXgyc?si=Y79z6rwITmHVlbj4)

## Bài viết liên quan

- [Giám Sát Năng Lượng Trong Hệ Thống KNX: Vì Sao Phải Đo Điện Trước Khi Tiết Kiệm Điện](https://knxstore.vn/giam-sat-nang-luong-trong-he-thong-knx-vi-sao-phai-do-dien-truoc-khi-tiet-kiem-dien.html) — Kiến thức · 13/10/2026
- [IC Là Gì? Cấu Tạo, Chức Năng Và Vai Trò Trong Smarthome](https://knxstore.vn/ic-la-gi-vai-tro-cua-ic-trong-thiet-bi-smarthome.html) — Kiến thức · 17/8/2026
- [NFC là gì? Vì sao smarthome hiện đại đều tích hợp công nghệ này?](https://knxstore.vn/nfc-la-gi-vi-sao-smarthome-hien-dai-deu-tich-hop-cong-nghe-nay.html) — Kiến thức · 5/3/2026
- [SPI Là Gì? Ứng Dụng SPI LED Controller Trong LED RGBW](https://knxstore.vn/spi-la-gi-ung-dung-spi-led-controller-trong-led-rgbw.html) — Kiến thức · 17/8/2026
````

Bài điển hình dài hơn (`/matter-fabric-la-gi.html`: có tác giả, 5 mục FAQ, 4 ảnh, 1 bảng) là 12,9 KB (HTML 369,8 KB, giảm 96,5%). Bài nhiều khối sản phẩm (`/cac-thanh-phan-giao-thuc-matter.html`: 3 khối, 11 sản phẩm, 6 mục FAQ) là 21,2 KB (HTML 467,2 KB, giảm 95,5%).

---

## 6. Quyết định còn mở về output

| # | Điểm | Mặc định trong spec này | Người chốt |
|---|---|---|---|
| 1 | Có xuất `modified` | Có, khi khác `published`. Dữ liệu công khai (JSON-LD) nhưng trang không hiển thị; khác `published` ở 8/30 bài cũ, giá trị rơi vào 2026-08-30 đến 2026-10-10 (có vẻ cập nhật hàng loạt). Bỏ nếu muốn khớp chặt với những gì trang hiển thị | Tùng / Vũ |
| 2 | Khối sản phẩm đặt xen giữa nội dung (đúng vị trí trang) hay gom thành một mục cuối bài | Xen giữa. Giữ ngữ cảnh "sản phẩm này đi kèm đoạn nào"; gom cuối thì gọn hơn nhưng mất vị trí | Tùng |
| 3 | Ngày đăng trong bảng và bài liên quan | Giữ dạng `d/m/yyyy` như trang (có thể gây nhầm ngày/tháng); ngày chính xác ở `published` | Tùng |
| 4 | Heading của bài giữ nguyên cấp (`##`, `###`) thay vì hạ 1 cấp | Giữ nguyên, vì thân bài là phần chính (khác mô tả sản phẩm nằm dưới mục `## Mô tả`) | Tùng |
| 5 | Khối tư vấn ở cột bên (hotline, "Tư vấn dự án") | Không đưa | Tùng |
| 6 | Breadcrumb theo trang ("Blog kỹ thuật") thay vì theo JSON-LD ("Tin tức") | Theo trang | Tùng |

---

## 7. Ghi chú cho dev (ngắn)

- **Nội dung có sẵn trong HTML ban đầu**, không giấu sau tab: thân bài, bảng, ảnh, FAQ (đáp án có trong `details` đóng), bài liên quan, breadcrumb, byline. Chỉ mục lục là client-only (heading không có `id` trong HTML). Payload RSC của bài chỉ chứa dữ liệu menu và bài liên quan, không có đối tượng "bài viết" đầy đủ; nội dung bài lấy từ DOM (hoặc từ nguồn CMS mà trang dùng).
- **Cấu trúc thân bài** (`div#article-body`): các con trực tiếp xen kẽ giữa khối văn bản (`div` không class, HTML CMS) và khối sản phẩm (`div.my-8`, chứa 1 đến 9 `article`, trung vị 3 cùng loại thẻ sản phẩm của trang danh mục). 16/30 bài có khối sản phẩm (tổng 39 khối, 122 thẻ; nhiều nhất 33 thẻ trong một bài).
- **Dữ liệu thẻ sản phẩm** có trong payload (cùng 18 khóa như thẻ ở trang danh mục: `price`, `priceRange`, `oldPrice`, `sku`, `brand`…); giá DOM là cận dưới `priceRange.from`. Khi không tìm thấy object theo `slug`, dùng chữ của chính thẻ (đã làm trong bộ chuyển đổi tham chiếu).
- **Phản hồi bất thường** **[Đã kiểm]**: 1/30 lần tải đầu trả HTML 101.730 B **không có `#article-body` và không có `h1`** (tải lại ngay trả đủ 414.326 B); 4 request khác lỗi kết nối. Chuyển đổi phải báo lỗi khi thiếu `#article-body`, không được xuất MD rỗng.
- **Dữ liệu lệch nên biết**: breadcrumb trên trang là "Blog kỹ thuật" nhưng breadcrumb JSON-LD đặt tên "Tin tức"; meta description bị cắt "…" ở 5/30 bài; `meta author` là "KNX Store" khi bài không hiển thị tác giả (11/30); một SKU có ký tự vô hình (U+200E) đứng đầu; 4 liên kết CMS thiếu đuôi `.html`; `alt` ảnh do CMS tự sinh; một số bài cũ có `<table>` thiếu `thead` (2/30), `iframe` YouTube (1/30), `blockquote` (1/30).
- **Cơ cấu mẫu khảo sát** (30 bài): chuyên mục Kiến thức 13, Chiếu sáng 12, Cảm biến 3, An ninh 2; 19 bài hiển thị tác giả; 17 bài có FAQ; 11 bài có bảng; 4 bài có danh sách đánh số; không bài nào có heading `h4` trở lên; 8 bài có `dateModified` khác `datePublished`.
- **Bộ chuyển đổi tham chiếu và kiểm tra chéo** (script ở thư mục tạm của phiên làm việc: `blogmd.py`, `blogmd_check.py`; chưa đưa vào dự án): chạy trên cả 30 bài, mọi kiểm tra áp dụng đều đạt (25 đến 28 kiểm tra mỗi bài). Kiểm tra gồm: frontmatter (khớp `canonical`, H1, meta description, ngày UTC), breadcrumb và thông tin nhanh khớp DOM, số heading và cấp, số bảng và số hàng, số ảnh và `alt`, số liên kết, chuỗi từ của thân bài bằng DOM, khối sản phẩm (slug, thứ tự, giá), FAQ (số lượng và chữ, khớp `FAQPage`), bài liên quan, không còn thẻ HTML/NBSP/ký tự vô hình, dựng hai lần cho cùng byte.
