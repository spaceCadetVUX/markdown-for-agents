# Giải pháp — đặc tả Markdown trả về cho AI agent

> Ngày: 2026-11-11 · Phạm vi: trang giải pháp `/solution/{slug}` (12 trang trong `sitemaps/brands.xml`: an-ninh, casambi, dali, dmx, giai-phap-khac, homekit, hvac, knx, lutron, matter, mobus, zigbee) · Trạng thái: **bản nháp**.
> Ngoài phạm vi: trang gốc `/solution` (danh sách giải pháp), trang `/products?solution=…` (danh sách sản phẩm), trang chủ.
> File này mô tả **kết quả mong muốn** (MD trông như thế nào); mục 7 là ghi chú ngắn cho dev. Dựa trên đo cả 12 trang và một bộ chuyển đổi tham chiếu chạy trên cả 12 trang (mục 7). Số token là ước tính bytes/4. Cùng khung với `brand-md-spec/` và `product-category-md-spec/`; khác ở chỗ trang giải pháp là trang **tổng quan + điều hướng** (danh mục, sản phẩm nổi bật, thương hiệu) kèm một bài giới thiệu dài do CMS nhập.

---

## 1. Nguyên tắc

1. **Cùng thông tin với HTML, khác định dạng.** Không tóm tắt, không thêm thông tin không có trên trang. Ngoại lệ duy nhất: liên kết lọc `?solution=` ở mục 3.4 và 3.6 (xem quyết định 1 ở mục 6).
2. **Mỗi URL một bản MD.** Trang giải pháp không phân trang và không có tham số; mọi tham số truy vấn bị bỏ qua (mục 3.9).
3. **Chỉ nội dung của giải pháp.** Không menu, chân trang, giỏ báo giá, nút kêu gọi hành động, khối "Bài viết kỹ thuật mới" (3 bài giống hệt trang chủ, không liên quan giải pháp), JSON-LD.
4. **Dữ kiện có cấu trúc và điều hướng trước, văn xuôi dài sau**: danh mục → sản phẩm nổi bật → thương hiệu → bài giới thiệu → FAQ.
5. Khối rỗng hoặc không có trên trang thì bỏ hẳn heading/dòng; không ghi "Không có", "N/A".
6. Mọi liên kết và ảnh là URL tuyệt đối `https://knxstore.vn/...` (liên kết ngoài do CMS nhập giữ nguyên, kể cả `http://`).
7. **Gọn.** MD từ 4,8 KB đến 42,1 KB (trung vị 22,2 KB, ~5.500 token); HTML thô 407 KB đến 527 KB; giảm 91,8% đến 98,8%, trung vị 95,0%. Phần lớn dung lượng là bài giới thiệu (928 đến 5.604 từ ở 11 trang); phần frontmatter và các danh sách chỉ 2,4 KB đến 4,9 KB.

---

## 2. Frontmatter

| Key | Giá trị | Điều kiện | Ưu tiên |
|---|---|---|---|
| `type` | `solution` | luôn | P3 |
| `title` | H1 hiển thị, ví dụ `Giải pháp An ninh` | luôn | P1 |
| `description` | meta description nguyên văn | có | P2 |
| `canonical` | URL HTML chuẩn, tuyệt đối | luôn | P1 |
| `lang` | `vi` | luôn | P3 |
| `image` | ảnh cover của giải pháp (`og:image`, cũng là ảnh hero), URL tuyệt đối | có (12/12) | P3 |

- `title` lấy từ H1, **không** lấy `<title>`: `<title>` là tiêu đề SEO khác H1 ở 12/12 trang (ví dụ An ninh: "Hệ an ninh Satel Phân cấp EN 50131 và chọn đúng dòng cho dự án"; `giai-phap-khac`: tiêu đề mặc định của site "KNXStore - Nhà phân phối thiết bị điện thông minh chính hãng").
- `description` giữ nguyên chuỗi nguồn: 7/12 trang bị cắt và kết thúc bằng "…" (an-ninh, casambi, dmx, homekit, hvac, matter, zigbee); `giai-phap-khac` chỉ có câu mặc định "Tham khảo ngay các thiết bị điện thông minh tại KNXStore". Đây **không** phải đoạn mô tả hiển thị dưới H1 (mục 3.3).
- `image` luôn là ảnh riêng của giải pháp (đường dẫn `/media/...`); host `api.knxstore.vn` trong `og:image` đổi thành `knxstore.vn`.
- Không có `product_count`: trang không hiển thị tổng số sản phẩm của giải pháp (mục 3.4). Không có ngày: `datePublished`/`dateModified` chỉ có trong JSON-LD (quyết định 5).
- Key frontmatter bằng tiếng Anh; heading trong body bằng tiếng Việt. Giá trị có `: ` hoặc dấu `"` thì đặt trong dấu nháy kép.

---

## 3. Body

### 3.1 Thứ tự và heading

| # | Khối | Heading | Điều kiện hiện | Ưu tiên |
|---|---|---|---|---|
| 1 | Tên giải pháp | `#` (H1 duy nhất) | luôn | P1 |
| 2 | Breadcrumb | không heading; 1 dòng `Trang chủ > Giải pháp > {Tên}` | luôn | P3 |
| 3 | Đoạn mô tả | không heading; 1 đoạn văn ngay dưới breadcrumb | 10/12 (không có ở `giai-phap-khac`, `mobus`) | P1 |
| 4 | Thiết bị theo loại | `## Thiết bị theo loại` | luôn (12/12; 2 đến 14 mục) | P1 |
| 5 | Sản phẩm nổi bật | `## Sản phẩm nổi bật` | luôn (12/12; đúng 8 sản phẩm) | P1 |
| 6 | Thương hiệu có hàng | `## Thương hiệu có hàng` | luôn (12/12; 1 đến 12 mục) | P1 |
| 7 | Giới thiệu giải pháp | `## Giới thiệu giải pháp` | 11/12 (không có ở `giai-phap-khac`) | P1 |
| 8 | FAQ | `## FAQ` | 3/12 (an-ninh 9 câu, casambi 14, dali 8) | P1 |

Thứ tự khớp thứ tự trên trang, trừ khối "Bài viết kỹ thuật mới" (bỏ) nằm giữa bài giới thiệu và FAQ trên trang. Không có bảng "Thông tin nhanh" như các trang khác vì trang không có con số tổng nào để đưa (mục 3.4).

### 3.2 Quy tắc chung

- Link nội bộ dạng tuyệt đối. UTF-8, LF; không emoji; không HTML thô.
- Không đưa text giao diện: "Xem sản phẩm", "Tư vấn dự án", "Cần tư vấn giải pháp … cho dự án?", "Thêm vào yêu cầu báo giá".
- Không chuẩn hóa chữ của trang (giữ nguyên hoa thường của tên danh mục, thương hiệu, sản phẩm và của bài giới thiệu).
- Ký tự `*`, `[`, `]`, `` ` `` trong chữ lấy từ dữ liệu được escape bằng `\`.
- Dấu cách không ngắt (NBSP) thành dấu cách thường; ký tự định dạng vô hình (U+200B đến U+200F, U+202A đến U+202E, U+2060, U+FEFF) bị bỏ; dấu cách đơn dư cuối dòng bị bỏ (ngắt dòng cứng từ `<br>` giữ hai dấu cách). Đã gặp: U+200B trong bài giới thiệu của `dmx`, `knx`, `lutron`.

### 3.3 Breadcrumb và đoạn mô tả

```markdown
[Trang chủ](https://knxstore.vn/) > Giải pháp > An ninh

Giải pháp an ninh cho nhà ở và tòa nhà với tiêu chuẩn đến từ Châu Âu. Bảo mật cao, vận hành ổn định và hỗ trợ kỹ thuật từ KNXStore.
```

- Breadcrumb lấy đúng như trang: 3 mục; chỉ "Trang chủ" là liên kết, "Giải pháp" và mục cuối là chữ thường (trang không gắn liên kết cho "Giải pháp"; quyết định 2). Breadcrumb trong JSON-LD chỉ có 2 mục, không dùng.
- Đoạn mô tả là đoạn `p` trong khối hero dưới H1, nguyên văn (10/12). Nó khác meta description ở mọi trang có nó, nên cả hai đều giữ: meta description ở `description`, đoạn này trong body. Trang không có đoạn này (`giai-phap-khac`, `mobus`) thì bỏ, `description` là mô tả duy nhất.

### 3.4 Thiết bị theo loại

Mỗi ô danh mục trên trang một dòng, đúng thứ tự trang (giảm dần theo số đếm):

```markdown
- [Thiết bị an ninh](https://knxstore.vn/kiem-soat-ra-vao?solution=an-ninh) — 104 sản phẩm
```

| Thành phần | Quy tắc |
|---|---|
| Tên | tên danh mục nguyên văn |
| Số | số sản phẩm **của giải pháp này** thuộc danh mục đó (không phải tổng của danh mục) |
| URL | `https://knxstore.vn/{slug-danh-mục}?solution={slug-giải-pháp}`. HTML trỏ tới `/{slug-danh-mục}` không lọc nên số trên ô khác số trên trang đích; thêm `?solution=` làm hai số trùng nhau (quyết định 1) |

- Một sản phẩm có thể thuộc nhiều danh mục, nên **không cộng các số** thành tổng của giải pháp (casambi: các ô cộng được 97, `/products?solution=casambi` báo 96). Trang không hiển thị tổng, MD cũng không ghi.
- Đã kiểm bằng request thật: 16 liên kết danh mục (casambi 11, an-ninh 3, giai-phap-khac 2) trả trang danh mục có số sản phẩm đúng bằng số trên ô (kể cả 104 và 36 sản phẩm, trang có phân trang).

### 3.5 Sản phẩm nổi bật

Đúng 8 sản phẩm của trang, một dòng mỗi sản phẩm, theo thứ tự trang, rồi liên kết "Xem tất cả":

```markdown
- [{tên}](https://knxstore.vn/products/{slug}) — {thương hiệu} · {SKU} · {giá}{ (giá cũ …)}{ · nhãn tồn kho}

[Xem tất cả](https://knxstore.vn/products?solution={slug-giải-pháp})
```

| Thành phần | Quy tắc |
|---|---|
| Tên | tên sản phẩm nguyên văn |
| Thương hiệu | **có in** (khác trang thương hiệu): 8 thẻ thường thuộc nhiều hãng |
| SKU | chỉ khi thẻ hiển thị mã model; bỏ khi không có |
| Giá | `3.190.000 đ` (dấu chấm nghìn, một khoảng trắng). Lấy cận dưới của khoảng giá (`priceRange.from`) trước, `price` sau; khoảng giá thì `từ 12.540.000 đ`; không giá `Liên hệ`; giá ẩn `Giá liên hệ` |
| Giá cũ | khi giảm giá: ` (giá cũ 18.987.000 đ)` ngay sau giá; không ghi phần trăm |
| Nhãn tồn kho | chỉ khi thẻ hiển thị nhãn (`Đặt trước`, `Sắp hết hàng`, `Còn hàng`) |
| Không đưa | ảnh thẻ, nút "Thêm vào báo giá" |

- Đây là nơi duy nhất agent thấy sản phẩm của giải pháp; muốn danh sách đầy đủ thì theo liên kết "Xem tất cả" (ví dụ casambi: 96 sản phẩm, 4 trang). Tiêu chí chọn 8 sản phẩm nổi bật: Chưa xác định.
- Giá và nhãn của thẻ nằm trong payload RSC (`priceRange`, `priceHidden`, `oldPrice`…), DOM chỉ hiện giá thấp nhất. Cùng quy tắc với `product-category-md-spec/`.

### 3.6 Thương hiệu có hàng

```markdown
- [Satel](https://knxstore.vn/brands/satel?solution=an-ninh) — 101 sản phẩm
```

- Mỗi chip một dòng, đúng thứ tự trang (giảm dần theo số đếm; 1 đến 12 chip). Chip trên trang là chữ, không có logo.
- Số là số sản phẩm của hãng **trong giải pháp này**. URL thêm `?solution={slug-giải-pháp}` vào `/brands/{slug}` mà HTML trỏ tới (cùng lý do mục 3.4). Đã kiểm: 20 liên kết (casambi 12, an-ninh 5, giai-phap-khac 3) trả trang thương hiệu có số sản phẩm đúng bằng số trên chip (kể cả Satel 101).

### 3.7 Giới thiệu giải pháp

Bài giới thiệu do CMS nhập (khối `div.user-html` dưới tiêu đề "Về {Tên}", 11/12 trang). MD thay tiêu đề "Về {Tên}" bằng heading cố định `## Giới thiệu giải pháp`, rồi chuyển nội dung theo cùng quy tắc CMS của spec thương hiệu và bài blog:

| Phần tử | Kết quả |
|---|---|
| Heading `h1`…`h4` | hạ xuống dưới `## Giới thiệu giải pháp`: heading nhỏ nhất của bài thành `###`, các cấp sau theo thứ tự, không nhảy quá 1 cấp, tối đa `######`. Bài của `casambi` có `h1` (trang HTML có hai `h1`): thành `###`; bài của `hvac` dùng `h2` rồi `h3`, `h4` |
| Đoạn văn, danh sách, bảng | giữ; bảng GFM, `\|` trong ô được escape; có bảng ở 8/11 trang (18 bảng) |
| Chữ đậm, nghiêng | `**…**`, `*…*` (`hvac` dùng chữ đậm làm nhãn mục (31 chỗ)) |
| `<br>` | ngắt dòng cứng (hai dấu cách cuối dòng) |
| `<hr>` | bỏ (`hvac` có 7) |
| Ảnh | `![alt](URL)`, giữ `alt`; URL tuyệt đối; có ở 6/11 trang (62 ảnh; 4 trang có 11 đến 14 ảnh); chiếm 2,4% đến 6,7% dung lượng MD của các trang đó |
| Liên kết | giữ chữ, URL tuyệt đối; liên kết ngoài giữ nguyên (ví dụ `dali-alliance.org`, `http://art.nr/` ở `dali`) |
| Thẻ bao, `class`, `style` | bỏ, giữ chữ |

Bài có thể tự chứa một mục "Câu hỏi thường gặp" dạng heading (`matter`, `hvac`): giữ như nội dung bài, **không** gộp vào `## FAQ` (mục 3.8).

### 3.8 FAQ

Chỉ 3 trang có khối FAQ riêng (accordion `details`, nằm sau khối "Bài viết kỹ thuật mới" trên trang). Heading của trang là "Câu hỏi thường gặp về {Tên}", MD dùng `## FAQ` cho thống nhất với các loại trang khác. Mỗi câu một mục danh sách:

```markdown
- **{Câu hỏi}** {Trả lời}
```

Trả lời là văn bản thường; trả lời nhiều đoạn thì các đoạn sau thụt 2 dấu cách. Số câu trong MD bằng số `details` trên trang và bằng số câu của `FAQPage` trong JSON-LD (9, 14, 8): JSON-LD không dùng làm nguồn.

### 3.9 URL biến thể và trường hợp đặc biệt

| URL yêu cầu | Bản MD |
|---|---|
| `/solution/{slug}` (12 slug) | như mô tả trên |
| `?page=…`, `?sort=…`, `?solution=…` hoặc tham số khác | bỏ qua; MD giống hệt bản không tham số, `canonical` vẫn là URL gốc (đã kiểm trên `dmx`) |
| `/solution/{slug}/` | chuyển hướng 308 về URL không có `/` (giống HTML) |
| slug không tồn tại hoặc sai chữ hoa (`/solution/DMX`) | 404 (giống HTML) |

| Trường hợp | Kết quả |
|---|---|
| `giai-phap-khac` ("Giải pháp khác", gom thiết bị không thuộc giao thức cụ thể) | chỉ có breadcrumb và 3 danh sách (14 danh mục, 8 sản phẩm, 12 thương hiệu); không có đoạn mô tả, bài giới thiệu, FAQ; 4,8 KB |
| `mobus` | không có đoạn mô tả nhưng có bài giới thiệu (3.727 từ) |
| 9/12 trang không có khối FAQ | bỏ `## FAQ` |
| H1 lặp chữ ("Giải pháp Giải pháp khác") | giữ nguyên như HTML; sửa ở nguồn (quyết định 3) |

---

## 4. Không đưa vào MD

| Loại bỏ | Lý do |
|---|---|
| Header (menu ~165 liên kết), footer, nút Zalo nổi, giỏ/drawer báo giá | Điều hướng lặp trên mọi trang |
| Nút hero "Xem sản phẩm" (`/products?solution=…`) và "Tư vấn dự án" (`/contact`), khối CTA cuối trang | Giao diện; "Xem sản phẩm" trùng "Xem tất cả" ở mục 3.5 |
| Ảnh hero trong thân trang (`alt` rỗng, trang trí) | Đã có ở `image` |
| Khối "Bài viết kỹ thuật mới" (3 bài) | Giống hệt trên trang chủ và 12/12 trang giải pháp, không lọc theo giải pháp (quyết định 4) |
| JSON-LD (`CollectionPage`, `Service`, `ItemList`, `FAQPage`, `Organization`) | Thông tin đã có trong MD; thống nhất không đưa JSON-LD |
| `<title>` (tiêu đề SEO), `meta keywords` (chỉ lặp tên giải pháp), ngày trong JSON-LD | Không hiển thị trên trang hoặc trùng thông tin có sẵn |
| Ảnh thẻ sản phẩm, tồn kho thẻ không hiển thị | Không có thông tin ngoài tiêu đề; P1 |

---

## 5. Ví dụ hoàn chỉnh

Giải pháp không có bài giới thiệu (`/solution/giai-phap-khac`). Toàn bộ output, 4.810 B:

````markdown
---
type: solution
title: Giải pháp Giải pháp khác
description: Tham khảo ngay các thiết bị điện thông minh tại KNXStore
canonical: https://knxstore.vn/solution/giai-phap-khac
lang: vi
image: https://knxstore.vn/media/f29c12ba547a4caaedabbbd7612997697f5096439fbfd004b158b38d0fb4aa20.jpg
---

# Giải pháp Giải pháp khác

[Trang chủ](https://knxstore.vn/) > Giải pháp > Giải pháp khác

## Thiết bị theo loại

- [Điều khiển máy lạnh](https://knxstore.vn/dieu-khien-may-lanh?solution=giai-phap-khac) — 36 sản phẩm
- [Cảm biến](https://knxstore.vn/cam-bien-hien-dien?solution=giai-phap-khac) — 34 sản phẩm
- [Thiết bị điện](https://knxstore.vn/thiet-bi-dien?solution=giai-phap-khac) — 22 sản phẩm
- [Dimmer & Relay chiếu sáng](https://knxstore.vn/dieu-khien-chieu-sang?solution=giai-phap-khac) — 12 sản phẩm
- [Khóa cửa thông minh](https://knxstore.vn/khoa-cua-thong-minh?solution=giai-phap-khac) — 9 sản phẩm
- [LED Driver & Nguồn](https://knxstore.vn/led-driver-1?solution=giai-phap-khac) — 7 sản phẩm
- [Phụ kiện cho hệ thống](https://knxstore.vn/thiet-bi-he-thong?solution=giai-phap-khac) — 6 sản phẩm
- [Công tắc thông minh](https://knxstore.vn/cong-tac-thong-minh?solution=giai-phap-khac) — 5 sản phẩm
- [Bộ điều khiển trung tâm](https://knxstore.vn/bo-dieu-khien-trung-tam?solution=giai-phap-khac) — 3 sản phẩm
- [Thiết bị an ninh](https://knxstore.vn/kiem-soat-ra-vao?solution=giai-phap-khac) — 3 sản phẩm
- [Âm thanh đa vùng](https://knxstore.vn/am-thanh-da-vung?solution=giai-phap-khac) — 1 sản phẩm
- [Đèn chiếu sáng](https://knxstore.vn/den-chieu-sang?solution=giai-phap-khac) — 1 sản phẩm
- [Điều khiển Matter](https://knxstore.vn/san-pham-matter?solution=giai-phap-khac) — 1 sản phẩm
- [Led SPI](https://knxstore.vn/led-spi?solution=giai-phap-khac) — 1 sản phẩm

## Sản phẩm nổi bật

- [Trạm quan trắc chất lượng không khí ngoài trời Tongdy - TF-9](https://knxstore.vn/products/tram-quan-trac-chat-luong-khong-khi-ngoai-troi-tongdy-tf-9) — Tongdy · TF-9 · Liên hệ
- [Cảm biến chuyển động PIR gắn trần 360° Hugo Muller Paladin 722 010 pir](https://knxstore.vn/products/cam-bien-chuyen-dong-pir-gan-tran-360-hugo-muller-paladin-722-010-pir) — HUGO MULLER · Paladin 722 010 · Giá liên hệ
- [Gateway Loxone qua Matter 1Home Server Loxone](https://knxstore.vn/products/gateway-loxone-qua-matter-1home-server-loxone) — 1Home · 1Home Server Loxone · 26.710.000 đ
- [Công tắc hẹn giờ điện tử müller SC 28.x3 pro4 / paladin 172 4x3 pro4](https://knxstore.vn/products/cong-tac-hen-gio-dien-tu-muller-sc-28-x3-pro4-paladin-172-4x3-pro4) — HUGO MULLER · SC-2813P4 · Liên hệ
- [Công tắc hẹn giờ điện tử müller SC-28.x1 Pro 4](https://knxstore.vn/products/cong-tac-hen-gio-dien-tu-muller-sc-28-x1-pro-4) — HUGO MULLER · SC-2811P4 · Liên hệ
- [Công tắc hẹn giờ điện tử âm tường paladin 173 110 easy](https://knxstore.vn/products/cong-tac-hen-gio-dien-tu-am-tuong-paladin-173-110-easy) — HUGO MULLER · P173110E · Liên hệ
- [Công tắc hẹn giờ âm tường müller EA-3622P4](https://knxstore.vn/products/cong-tac-hen-gio-am-tuong-muller-ea-3622p4) — HUGO MULLER · EA-3622P4 · Liên hệ
- [Công tắc hẹn giờ điện tử müller SC 38.x0 pro5 (WiFi + Bluetooth)](https://knxstore.vn/products/cong-tac-hen-gio-dien-tu-muller-sc-38-x0-pro5) — HUGO MULLER · SC-3820P230 · Liên hệ

[Xem tất cả](https://knxstore.vn/products?solution=giai-phap-khac)

## Thương hiệu có hàng

- [Airzone](https://knxstore.vn/brands/airzone?solution=giai-phap-khac) — 25 sản phẩm
- [IR-TEC](https://knxstore.vn/brands/ir-tec?solution=giai-phap-khac) — 24 sản phẩm
- [Bticino](https://knxstore.vn/brands/bticino?solution=giai-phap-khac) — 10 sản phẩm
- [Legrand](https://knxstore.vn/brands/legrand?solution=giai-phap-khac) — 10 sản phẩm
- [Moorgen](https://knxstore.vn/brands/moorgen?solution=giai-phap-khac) — 10 sản phẩm
- [Vadsbo](https://knxstore.vn/brands/vadsbo?solution=giai-phap-khac) — 8 sản phẩm
- [HUGO MULLER](https://knxstore.vn/brands/hugo-muller?solution=giai-phap-khac) — 6 sản phẩm
- [OEM](https://knxstore.vn/brands/oem?solution=giai-phap-khac) — 6 sản phẩm
- [CP Electronics](https://knxstore.vn/brands/cp-electronics?solution=giai-phap-khac) — 5 sản phẩm
- [Daikin](https://knxstore.vn/brands/daikin?solution=giai-phap-khac) — 5 sản phẩm
- [Steinel](https://knxstore.vn/brands/steinel?solution=giai-phap-khac) — 5 sản phẩm
- [DALCNET](https://knxstore.vn/brands/dalcnet?solution=giai-phap-khac) — 3 sản phẩm
````

Giải pháp đầy đủ khối (`/solution/dali`: đoạn mô tả, bài giới thiệu 3.095 từ với 11 ảnh và 3 bảng, FAQ 8 câu). **Trích, rút gọn bằng `…`** (dòng dài bị cắt, danh sách chỉ giữ vài mục); toàn bộ output là 29.570 B (~7.400 token), HTML 492.669 B:

````markdown
---
type: solution
title: Giải pháp DALI
description: "Giải pháp DALI-2 với chứng chỉ DALI Alliance: thiết bị đa hãng hoạt động cùng bus, on/off, dimmer Triac và điều khiển rèm. KNXStore phân phối tại Việt Nam."
canonical: https://knxstore.vn/solution/dali
lang: vi
image: https://knxstore.vn/media/70fb5ae49b043f4fe4467951a17a14c5224ae6e8dea6b6f6a9b9d3045af3165f.png
---

# Giải pháp DALI

[Trang chủ](https://knxstore.vn/) > Giải pháp > DALI

Giải pháp DALI-2 (IEC 62386) là tiêu chuẩn chiếu sáng kỹ thuật số duy nhất yêu cầu chứng chỉ độc lập từ DALI Alliance, đảm bảo thiết bị từ UIT IOT, Sunricher, Helvar hoạt động cùng một bus mà không cần cấu hình đặc biệt.

## Thiết bị theo loại

- [LED Driver & Nguồn](https://knxstore.vn/led-driver-1?solution=dali) — 19 sản phẩm
- [Dimmer & Relay chiếu sáng](https://knxstore.vn/dieu-khien-chieu-sang?solution=dali) — 17 sản phẩm
- …

## Sản phẩm nổi bật

- [LED Driver DALI-2 CC 20W 350-700mA Đơn sắc / Tunable White DT8 Euchips EUPS20D-1HMC / EUP20D-2HMC-0](https://knxstore.vn/products/led-driver-dali-2-cc-20w-350-700ma-don-sac-tunable-white-dt8-euchips-eups20d-1hmc-eup20d-2hmc-0) — Euchips · EUPS20D-1HMC · Liên hệ
- [LED Driver DALI-2 CC 15W 280-450mA NFC Đơn sắc / Tunable White DT8 Euchips EUPS15D-1HNC-280 / EUPS15D-2HNC-D](https://knxstore.vn/products/led-driver-dali-2-cc-15w-280-450ma-nfc-don-sac-tunable-white-dt8-euchips-eups15d-1hnc-280-eups15d-2hnc-d) — Euchips · EUPS15D-1HNC-280 · Liên hệ
- …

[Xem tất cả](https://knxstore.vn/products?solution=dali)

## Thương hiệu có hàng

- [Sunricher](https://knxstore.vn/brands/sunricher?solution=dali) — 30 sản phẩm
- [Lunatone](https://knxstore.vn/brands/lunatone?solution=dali) — 15 sản phẩm
- …

## Giới thiệu giải pháp

### Giải pháp DALI-2 Certified Khi Đèn Từ Nhiều Hãng Hoạt Động Cùng Một Bus

Hãy tưởng tượng bạn thay driver đèn bị hỏng bằng một driver hãng khác, cắm vào bus, và hệ thống nhận ra ngay không cần cấu hình lại. Đó là thực tế của giải pháp DALI-2 Certified. Không phải…

![Hệ thống điều khiển chiếu sáng DALI là gì?](https://knxstore.vn/media/477d269dd8fba1039a670128208c9a4f9ba1512ba0450461446078a9cb65048c.jpg)

Giải pháp DALI-2 (Digital Addressable Lighting Interface, IEC 62386 thế hệ 2) cho phép từng đèn có địa chỉ riêng trên bus 2 dây, từng công tắc và cảm biến được quản lý tập trung, và toàn bộ…

…

| Tiêu chí | DALI-2 Certified | 0-10V Analog | Triac Dimmer đơn | DMX512 |
|---|---|---|---|---|
| Địa chỉ từng thiết bị | Có (64/subnet) | Không | Không | Có (512 channel) |
| Feedback từ thiết bị | Có (bi-directional) | Không | Không | Không (1 chiều) |
| Diagnostic lỗi từng điểm | Có | Không | Không | Không |

…

## FAQ

- **DALI-2 là gì?** DALI-2 là tiêu chuẩn điều khiển chiếu sáng kỹ thuật số IEC 62386 thế hệ 2, yêu cầu chứng nhận độc lập bắt buộc từ DALI Alliance cho mỗi sản phẩm. Nó cho phép từng đèn có địa chỉ ri…
- **Tôi có thể trộn driver hãng A với controller hãng B trong DALI-2 không?** Có, miễn là cả hai đều có chứng nhận DALI Alliance. Đây chính là giá trị của certification: DALI Alliance đảm bảo interope…
- **Một bus DALI-2 chứa được bao nhiêu thiết bị?** Tối đa 64 địa chỉ (short address 0-63) mỗi subnet. Tổng dòng bus tối đa 250mA. Cáp tối đa 300m với tiết diện 1.5mm². Để vượt 64 thiết bị: MAC132 (2 s…
- …
````

---

## 6. Quyết định còn mở về output

| # | Điểm | Mặc định trong spec này | Người chốt |
|---|---|---|---|
| 1 | Liên kết danh mục và thương hiệu thêm `?solution={slug}` | Có. HTML trỏ tới trang không lọc, số trên ô/chip là số trong giải pháp nên lệch với trang đích; thêm tham số làm hai số trùng nhau (36 liên kết đã kiểm, đúng 36/36). Bỏ tham số thì giữ đúng liên kết HTML nhưng agent thấy số khác | Tùng |
| 2 | Mục "Giải pháp" trong breadcrumb | Chữ thường như trang. Nếu muốn agent khám phá, trỏ tới `/solution` (đã kiểm: trả 200) | Tùng / Vũ |
| 3 | H1 và tiêu đề khối lặp chữ ở `giai-phap-khac` ("Giải pháp Giải pháp khác", "Thiết bị Giải pháp khác theo loại") | Giữ nguyên như HTML; sửa ở nguồn (không ghép "Giải pháp " trước tên đã chứa chữ này) | Vũ |
| 4 | Khối "Bài viết kỹ thuật mới" (3 bài chung) | Bỏ. Nếu muốn giữ để agent khám phá blog: thêm một dòng `[Blog kỹ thuật](https://knxstore.vn/blogs)` thay vì 3 bài lặp trên mọi trang | Tùng |
| 5 | Xuất `modified` từ JSON-LD | Không. `datePublished` là cùng một mốc (2026-08-24T14:20:16.977Z) ở cả 12 trang, có vẻ nhập hàng loạt; 10/12 trang có `dateModified` trùng mốc đó, chỉ `dali` (2026-08-27) và `hvac` (2026-10-10) khác, nên không phải tín hiệu tin cậy | Tùng |
| 6 | Ảnh trong bài giới thiệu | Giữ `![alt](URL)` (6/11 trang, 62 ảnh; tốn 2,4% đến 6,7% dung lượng ở các trang đó) | Tùng |
| 7 | `description` bị cắt "…" (7/12) hoặc mặc định (`giai-phap-khac`) | Giữ nguyên chuỗi nguồn; sửa ở nguồn nếu muốn đầy đủ | Tùng / Vũ |
| 8 | Tổng số sản phẩm của giải pháp | Không đưa (trang không hiển thị; các ô danh mục cộng lại sai vì một sản phẩm thuộc nhiều danh mục). Muốn có thì lấy từ `/products?solution={slug}` | Tùng |

---

## 7. Ghi chú cho dev (ngắn)

- **Cấu trúc**: route `(site)/(b2b)/solution/[slug]`, render động theo request (`cache-control: no-store`), mọi nội dung có trong HTML ban đầu (không Suspense đang chờ). Trong `div#main-content`: `nav` breadcrumb → `section.knx-hero` (`h1`, `p` mô tả, 2 nút, ảnh) → các `section` mỗi khối một `h2`: "Thiết bị {X} theo loại" (ô là `a` chứa "{tên}{N} sản phẩm"), "Sản phẩm {X} nổi bật" (8 `article` + liên kết "Xem tất cả"), "Thương hiệu có hàng" (chip `a`), "Về {X}" (`div.user-html`), "Bài viết kỹ thuật mới", "Câu hỏi thường gặp về {X}" (`details.group/faq`: `summary` + `div.user-html`), CTA cuối. Nên bám theo chữ của `h2`, không bám class Tailwind.
- **Dữ liệu**: nội dung bài giới thiệu và FAQ chỉ cần lấy từ DOM (payload của route không có trường `contentHtml`, `seoTitle`). Chỉ giá, SKU, `priceHidden`, `oldPrice` của 8 thẻ sản phẩm cần payload RSC, giống trang danh mục.
- **Tham số đã kiểm**: `/{danh-mục}?solution={slug}` và `/brands/{slug}?solution={slug}` trả đúng số sản phẩm bằng số trên ô/chip (36/36, gồm casambi, an-ninh, giai-phap-khac). Trang `/solution/{slug}` bỏ qua mọi tham số truy vấn.
- **Phản hồi bất thường**: 0/12 lần tải đầu trả HTML thiếu nội dung (sản phẩm 2/80, danh mục 1/14, blog 1/30, thương hiệu 4/71). Chuyển đổi vẫn phải báo lỗi khi thiếu `#main-content`, hero hoặc `h1`, không xuất MD rỗng.
- **Dữ liệu lệch nên biết**: `<title>` khác H1 ở 12/12 (SEO title riêng ở 11, tiêu đề mặc định của site ở `giai-phap-khac`); meta description bị cắt "…" ở 7/12; H1 lặp chữ ở `giai-phap-khac`; bài giới thiệu của `casambi` chứa `h1` nên trang có 2 `h1`; `hvac` dùng chữ đậm làm nhãn mục và 7 `<hr>`; breadcrumb hiển thị 3 mục nhưng breadcrumb JSON-LD chỉ 2; mục "Giải pháp" trong breadcrumb không phải liên kết; `isPartOf` trong JSON-LD tham chiếu `#website` nhưng graph không có node `WebSite` (12/12); U+200B trong bài giới thiệu của `dmx`, `knx`, `lutron`.
- **Bộ chuyển đổi tham chiếu và kiểm tra chéo** (script ở thư mục tạm của phiên: `solutionmd.py`, `solutionmd_check.py`; chưa đưa vào dự án): chạy trên cả 12 trang, 29 đến 37 kiểm tra mỗi trang, mọi kiểm tra đều đạt. Kiểm tra gồm: frontmatter (khớp `canonical`, H1, meta description, `og:image`), breadcrumb (chữ và liên kết), đoạn mô tả hero, thứ tự và không rỗng của heading cố định, danh mục và thương hiệu (tên, URL, số đếm, thứ tự khớp DOM), sản phẩm nổi bật (slug, thứ tự, tên, giá, SKU, nhãn tồn kho, thương hiệu), bài giới thiệu (chuỗi từ bằng DOM, số và chữ heading, số bảng, số ảnh, số liên kết, cấp heading), FAQ (số lượng và chữ, khớp `FAQPage`), không còn khối blog chung/CTA/JSON-LD, không còn thẻ HTML/NBSP/ký tự vô hình, dựng hai lần cho cùng byte.
