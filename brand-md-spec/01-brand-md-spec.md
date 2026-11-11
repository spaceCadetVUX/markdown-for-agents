# Thương hiệu — đặc tả Markdown trả về cho AI agent

> Ngày: 2026-11-11 · Phạm vi: trang thương hiệu `/brands/{slug}` (71 trang trong `sitemaps/brands.xml`) · Trạng thái: **bản nháp**.
> Ngoài phạm vi: trang danh sách `/brands`, các URL `/solution/{slug}` (cũng nằm trong `brands.xml`, 12 URL), trang tác giả.
> File này mô tả **kết quả mong muốn** (MD trông như thế nào); mục 7 là ghi chú ngắn cho dev. Dựa trên đo cả 71 trang và một bộ chuyển đổi tham chiếu chạy trên cả 71 trang (mục 7). Số token là ước tính bytes/4. Cùng khung với `product-category-md-spec/` (trang thương hiệu gần như cùng cấu trúc với trang danh mục).

---

## 1. Nguyên tắc

1. **Cùng thông tin với HTML, khác định dạng.** Không tóm tắt, không thêm thông tin không có trên trang.
2. **Mỗi URL một bản MD, chỉ chứa phần của URL đó.** Trang 2 không chứa sản phẩm trang 1.
3. **Danh sách sản phẩm của trang đầy đủ, không cắt** (24 sản phẩm mỗi trang, thương hiệu có hơn 24 sản phẩm thì chia trang).
4. **Khối lặp giữa các trang (bộ lọc, giới thiệu thương hiệu) chỉ ở trang 1.**
5. **Chỉ nội dung thương hiệu.** Không menu, chân trang, giỏ báo giá, ngăn kéo bộ lọc, nút thao tác, JSON-LD.
6. Khối rỗng hoặc giá trị `null` thì bỏ hẳn heading/dòng; không ghi "Không có", "N/A".
7. Mọi liên kết và ảnh là URL tuyệt đối `https://knxstore.vn/...`.
8. **Gọn.** MD từ 1,1 KB đến 30,7 KB (trung vị 8 KB, ~2.000 token); HTML thô 144 KB đến 554 KB; giảm 93,6% đến 99,6%, trung vị 97,8%.

---

## 2. Frontmatter

| Key | Giá trị | Điều kiện | Ưu tiên |
|---|---|---|---|
| `type` | `brand` | luôn | P3 |
| `title` | tên thương hiệu (giống H1) | luôn | P1 |
| `description` | meta description nguyên văn | có | P2 |
| `canonical` | URL HTML chuẩn, tuyệt đối | luôn | P1 |
| `lang` | `vi` | luôn | P3 |
| `product_count` | tổng số sản phẩm của thương hiệu (mọi trang) | có | P1 |
| `page` | số trang hiện tại (mặc định 1) | luôn | P1 |
| `page_count` | tổng số trang (1 khi không phân trang) | luôn | P1 |
| `query` | tham số `category`, `solution`, `sort` của URL yêu cầu, ví dụ `category=thiet-bi-he-thong&sort=price-asc` | chỉ khi URL có các tham số này | P2 |
| `image` | logo thương hiệu (`og:image`), URL tuyệt đối | chỉ khi thương hiệu có logo riêng (24/71) | P3 |

- `description` giữ nguyên chuỗi nguồn: 57/71 thương hiệu có mô tả riêng nhưng bị cắt và kết thúc bằng "…"; 8/71 chỉ có câu mặc định của site ("Thiết bị {Tên} chính hãng phân phối tại knxstore.vn — xem thông số kỹ thuật, tài liệu, so sánh và gửi yêu cầu báo giá cho dự án."); 6/71 có mô tả đầy đủ.
- `image` chỉ xuất khi `og:image` là ảnh riêng của thương hiệu (đường dẫn `/media/...`). Thương hiệu không có logo thì `og:image` là ảnh chung của site (47/71) nên bỏ khóa này.
- `canonical` của **mọi** trang phân trang (`?page=N`) là URL gốc của thương hiệu (HTML không tự trỏ về `?page=N`); `page` và `page_count` phân biệt các trang. `canonical` của URL có `category`, `solution`, `sort` cũng là URL gốc, `query` cho biết đó là danh sách đã lọc hoặc sắp xếp.
- Không có ngày. Key frontmatter bằng tiếng Anh; heading trong body bằng tiếng Việt. Giá trị có `: ` hoặc dấu `"` thì đặt trong dấu nháy kép.

---

## 3. Body

### 3.1 Thứ tự và heading

| # | Khối | Heading | Điều kiện hiện | Ưu tiên |
|---|---|---|---|---|
| 1 | Tên thương hiệu | `#` (H1 duy nhất) | luôn | P1 |
| 2 | Breadcrumb | không heading; 1 dòng `Trang chủ > Thương hiệu > {Tên}` | luôn | P3 |
| 3 | Thông tin nhanh | không heading; bảng 2 cột | luôn | P1 |
| 4 | Bộ lọc và sắp xếp | `## Bộ lọc và sắp xếp` | chỉ trang 1, có sản phẩm | P2 |
| 5 | Sản phẩm | `## Sản phẩm` | có sản phẩm | P1 |
| 6 | Phân trang | `## Phân trang` | `page_count > 1` | P1 |
| 7 | Giới thiệu thương hiệu | `## Giới thiệu thương hiệu` | chỉ trang 1, thương hiệu có văn bản giới thiệu (51/71) | P2 |

Thứ tự đặt dữ kiện có cấu trúc và điều hướng trước, văn xuôi dài (giới thiệu) sau.

### 3.2 Quy tắc chung

- Link nội bộ dạng tuyệt đối; giữ nguyên `?page=N` như HTML. UTF-8, LF; không emoji; không HTML thô (ngoại lệ duy nhất: `<br>` trong ô bảng).
- Không đưa text giao diện: "Thêm vào yêu cầu báo giá", "Bộ lọc 0", "Xoá lọc", "Áp dụng", "Chỉ hàng thanh lý".
- Không chuẩn hóa chữ của trang (giữ nguyên hoa thường của tên thương hiệu và tiêu đề sản phẩm).
- Ký tự `*`, `[`, `]`, `` ` `` trong chữ lấy từ dữ liệu được escape bằng `\`.
- Dấu cách không ngắt (NBSP) thành dấu cách thường; ký tự định dạng vô hình (U+200B đến U+200F, U+202A đến U+202E, U+2060, U+FEFF) bị bỏ; dấu cách đơn dư cuối dòng bị bỏ (ngắt dòng cứng từ `<br>` giữ hai dấu cách). Đã gặp: NBSP trong tên sản phẩm và `alt` ảnh, U+200E đứng đầu tên hoặc mã SKU, U+200B trong văn bản giới thiệu.

### 3.3 Breadcrumb và Thông tin nhanh

```markdown
[Trang chủ](https://knxstore.vn/) > [Thương hiệu](https://knxstore.vn/brands) > Protopixel
```

Liên kết lấy đúng như trang: mục "Trang chủ" trỏ `/` ở hầu hết thương hiệu, nhưng trỏ `/matter` ở 6/71 thương hiệu.

Thông tin nhanh là bảng 2 cột (cột tiêu đề rỗng):

| Dòng | Giá trị | Điều kiện |
|---|---|---|
| Số sản phẩm | `2` | có `product_count` |
| Trang | `1 trên 5` | `page_count > 1` |
| Xem ở trang 1 | `bộ lọc và giới thiệu thương hiệu ({URL gốc})` | `page ≥ 2` |

### 3.4 Bộ lọc và sắp xếp

Chỉ ở trang 1 và khi thương hiệu có sản phẩm. Ba dòng:

```markdown
- Danh mục: [Thiết bị an ninh](https://knxstore.vn/brands/satel?category=kiem-soat-ra-vao) (95) · [Phụ kiện cho hệ thống](…?category=thiet-bi-he-thong) (6) · …
- Giải pháp: [An ninh](https://knxstore.vn/brands/satel?solution=an-ninh) (101) · [KNX](…?solution=knx) (7) · …
- Sắp xếp: thêm `sort=` vào URL: `newest` (Mới nhất), `name-asc` (Tên A → Z), `name-desc` (Tên Z → A), `price-asc` (Giá thấp → cao), `price-desc` (Giá cao → thấp). Mặc định không có tham số. Kết hợp được, ví dụ `?category={slug}&sort=price-asc`.
```

- Thứ tự như trang (giảm dần theo số đếm). Số trong ngoặc là số sản phẩm của cả thương hiệu với giá trị đó; một sản phẩm có thể thuộc nhiều giá trị, hoặc không thuộc giá trị nào, nên **không cộng các số đếm** thành tổng.
- Dòng nào không có dữ liệu thì bỏ. Liên kết lọc dùng đúng tham số mà server chạy (đã kiểm: `category`, `solution`, `sort`, kết hợp được; xem mục 7).
- **Không đưa** bộ lọc thuộc tính kỹ thuật mà thương hiệu lớn có (ví dụ Satel: IP rating, Giao thức, Nhiệt độ hoạt động, Khối lượng, Dòng tiêu thụ tối đa): chưa biết tên tham số URL nên agent không dùng được.

### 3.5 Sản phẩm

Mỗi sản phẩm của trang một dòng, đúng thứ tự trang, không cắt:

```markdown
- [{tên}](https://knxstore.vn/products/{slug}) — {SKU} · {giá}{ (giá cũ …)}{ · nhãn tồn kho}
```

| Thành phần | Quy tắc |
|---|---|
| Tên | tên sản phẩm nguyên văn |
| Thương hiệu | **không in**: mọi thẻ trên trang thuộc đúng thương hiệu của trang (đã kiểm trên trang 1 của cả 71 thương hiệu). Chỉ in nếu gặp thẻ của thương hiệu khác |
| SKU | chỉ khi thẻ hiển thị mã model; bỏ khi sản phẩm không có SKU |
| Giá | `3.190.000 đ` (dấu chấm nghìn, một khoảng trắng). Lấy cận dưới của khoảng giá (`priceRange.from`) trước, `price` sau; khoảng giá thì `từ 12.540.000 đ`; không giá `Liên hệ`; giá ẩn `Giá liên hệ`. Không ghi "(đã bao gồm VAT)" vì thẻ không hiển thị |
| Giá cũ | khi giảm giá: ` (giá cũ 18.987.000 đ)` ngay sau giá; không ghi phần trăm |
| Nhãn tồn kho | chỉ khi thẻ hiển thị nhãn (`Đặt trước`, `Sắp hết hàng`, `Còn hàng`) |
| Không đưa | ảnh thẻ, nút "Thêm vào báo giá" |

### 3.6 Phân trang

Giống trang danh mục: 24 sản phẩm mỗi trang; mục chỉ hiện khi `page_count > 1` (9/71 thương hiệu: Airzone 36, HUGO MULLER 32, Intesis 30, IR-TEC 32, Kanonbus 34, Legrand 60, Moorgen 32, Satel 110, Sunricher 61 sản phẩm).

```markdown
- Trang trước: https://knxstore.vn/brands/satel
- Trang sau: https://knxstore.vn/brands/satel?page=3
- Các trang: [1](https://knxstore.vn/brands/satel) · [2](https://knxstore.vn/brands/satel?page=2) · …
```

`Trang trước` khi `page ≥ 2`; `Trang sau` khi chưa phải trang cuối. Trang 1 là URL gốc không có `?page`.

### 3.7 Giới thiệu thương hiệu

Văn bản giới thiệu (nếu có) nằm dưới lưới sản phẩm, do CMS nhập; trang đặt phía trên nó tiêu đề "Về {Tên}", MD thay bằng heading cố định `## Giới thiệu thương hiệu`. Chuyển sang Markdown theo cùng quy tắc nội dung CMS của spec danh mục và bài blog:

| Phần tử | Kết quả |
|---|---|
| Heading `h1`, `h2`, `h3`… | hạ xuống dưới `## Giới thiệu thương hiệu`: heading nhỏ nhất thành `###`, các cấp sau theo thứ tự, không nhảy quá 1 cấp (có thương hiệu có `h1` trong văn bản: Kanonbus) |
| Đoạn văn, danh sách, bảng | giữ; bảng GFM, `\|` được escape |
| Ảnh | `![alt](URL)`, giữ `alt`; URL tuyệt đối (ảnh nằm ở `/media/...` hoặc `/assets/image/upload/...`); có ở 38/51 phần giới thiệu |
| Liên kết | giữ chữ, URL tuyệt đối (có liên kết tới sản phẩm và bài blog ở 12/51 phần giới thiệu) |
| Thẻ bao, `class`, `style` | bỏ, giữ chữ |

### 3.8 URL biến thể

| URL yêu cầu | Bản MD |
|---|---|
| `/brands/{slug}` | trang 1 |
| `/brands/{slug}?page=N` (1 < N ≤ số trang) | trang N; `canonical` vẫn là URL gốc; không có Bộ lọc và Giới thiệu; có dòng "Xem ở trang 1" |
| `?page=N` vượt số trang, hoặc `?page=2` của thương hiệu ≤ 24 sản phẩm | 404 (giống HTML) |
| `?page=0`, `?page=abc` | trang 1 (giống HTML) |
| `?category=…`, `?solution=…`, `?sort=…` (kết hợp được, kể cả với `page`) | danh sách đã lọc/sắp xếp; `canonical` là URL gốc; có `query` |
| `?view=list` | như trang 1 |
| thương hiệu không tồn tại | 404; `/brands/{slug}/` chuyển hướng 308 về URL không có `/` (giống HTML) |

### 3.9 Trường hợp đặc biệt

| Trường hợp | Kết quả |
|---|---|
| Thương hiệu 1 sản phẩm (17/71) | bình thường: `product_count: 1`, một dòng sản phẩm, không Phân trang |
| Không có văn bản giới thiệu (20/71) | bỏ mục Giới thiệu thương hiệu; mô tả duy nhất là `description` trong frontmatter |
| Không có logo (47/71) | bỏ `image` |
| `title` trang khác tên thương hiệu ("ProtoPixel – Hệ thống điều khiển LED… \| KNXStore" so với H1 "Protopixel") | dùng H1 làm `title`; bỏ `<title>` |

---

## 4. Không đưa vào MD

| Loại bỏ | Lý do |
|---|---|
| Header (menu ~165 liên kết), footer, nút Zalo nổi, giỏ/drawer báo giá | Điều hướng lặp trên mọi trang |
| Ngăn kéo bộ lọc, nút "Bộ lọc 0", "Chỉ hàng thanh lý", bộ lọc thuộc tính kỹ thuật | Giao diện; bộ lọc thuộc tính chưa có URL dùng được |
| Logo trong thân trang | Đã có ở `image` (alt chỉ lặp tên thương hiệu) |
| JSON-LD (`CollectionPage`, `Brand`, `ItemList`, `Organization`) | Thông tin đã có trong MD; thống nhất không đưa JSON-LD |
| Ảnh thẻ sản phẩm, tồn kho thẻ không hiển thị | Không có thông tin ngoài tiêu đề; P1 |

---

## 5. Ví dụ hoàn chỉnh

Thương hiệu nhỏ, có logo, không có văn bản giới thiệu (`/brands/protopixel`, 2 sản phẩm). Toàn bộ output, 1.570 B:

````markdown
---
type: brand
title: Protopixel
description: "ProtoPixel cung cấp nền tảng điều khiển LED chuyên nghiệp: mapping ánh sáng, automation, tích hợp đa giao thức (DMX, KNX). Phù hợp dự án smart lighting &…"
canonical: https://knxstore.vn/brands/protopixel
lang: vi
product_count: 2
page: 1
page_count: 1
image: https://knxstore.vn/media/09e11967b9d1d5a9168fe4970c1ec3732bd6210fedc5a5ac32b24a5a6bb61433.jpg
---

# Protopixel

[Trang chủ](https://knxstore.vn/) > [Thương hiệu](https://knxstore.vn/brands) > Protopixel

| | |
|---|---|
| Số sản phẩm | 2 |

## Bộ lọc và sắp xếp

- Danh mục: [Bộ điều khiển trung tâm](https://knxstore.vn/brands/protopixel?category=bo-dieu-khien-trung-tam) (2)
- Giải pháp: [Casambi](https://knxstore.vn/brands/protopixel?solution=casambi) (1) · [DALI](https://knxstore.vn/brands/protopixel?solution=dali) (1)
- Sắp xếp: thêm `sort=` vào URL: `newest` (Mới nhất), `name-asc` (Tên A → Z), `name-desc` (Tên Z → A), `price-asc` (Giá thấp → cao), `price-desc` (Giá cao → thấp). Mặc định không có tham số. Kết hợp được, ví dụ `?category={slug}&sort=price-asc`.

## Sản phẩm

- [Bộ điều khiển Gate Casambi Protopixel – Gate Casambi](https://knxstore.vn/products/bo-dieu-khien-gate-casambi-protopixel-gate-casambi) — Gate Casambi · Liên hệ
- [Bộ điều khiển trung tâm Brain Experience Protopixel](https://knxstore.vn/products/bo-dieu-khien-trung-tam-brain-experience-protopixel) — Brain Experience · Liên hệ
````

Thương hiệu có phân trang và văn bản giới thiệu dài (`/brands/satel`, 110 sản phẩm, 5 trang): trang 1 là 17,8 KB (HTML 528,9 KB); mỗi trang sau chỉ gồm frontmatter, thông tin nhanh, 24 dòng sản phẩm và mục Phân trang. Thương hiệu có phần giới thiệu dài nhất (`/brands/airzone`, 36 sản phẩm): trang 1 là 30,7 KB.

---

## 6. Quyết định còn mở về output

| # | Điểm | Mặc định trong spec này | Người chốt |
|---|---|---|---|
| 1 | Liên kết lọc `?category=`, `?solution=` và dòng Sắp xếp | Có. Là phần thêm so với HTML (trang chỉ có ô chọn không có URL), dùng tham số đã kiểm. Bỏ liên kết (chỉ giữ tên và số đếm) làm mục này nhỏ đi khoảng 53% (Satel: 1.159 B còn 540 B) | Tùng |
| 2 | Bộ lọc thuộc tính kỹ thuật của thương hiệu lớn (IP rating, giao thức…) | Không đưa (chưa biết tham số URL) | Tùng / Vũ |
| 3 | Bỏ tên thương hiệu khỏi dòng sản phẩm | Bỏ (trùng với H1 ở mọi thẻ) | Tùng |
| 4 | `canonical` của trang phân trang | Giống HTML: về URL gốc, phân biệt bằng `page` (khác trang danh mục, nơi HTML tự trỏ `?page=N`) | Tùng |
| 5 | `description` mặc định hoặc bị cắt "…" | Giữ nguyên chuỗi nguồn; sửa ở nguồn nếu muốn đầy đủ | Tùng / Vũ |
| 6 | Ảnh trong phần giới thiệu | Giữ dạng `![alt](URL)` (38/51 phần giới thiệu có ảnh, tốn token cho URL) | Tùng |

---

## 7. Ghi chú cho dev (ngắn)

- **Cấu trúc gần như trang danh mục**: breadcrumb → `h1` + số "N sản phẩm" → (thanh phân trang) → lưới `article` → khối `section` "Về {Tên}" với `div.user-html`; thanh công cụ lọc/sắp xếp nằm trong `div#S:0[hidden]` cuối `<body>`. Route `(site)/brands/[slug]`. Không có chip danh mục, không có FAQ. Mọi nội dung (danh sách, mô tả) có trong HTML ban đầu.
- **Dữ liệu bộ lọc** nằm trong props của khối lọc trong payload: `facets` (bộ lọc thuộc tính: `checkbox` có `options` và số đếm, `range` có `min`/`max`, kèm `coverage`), `brands: []` (rỗng), `solutions[]` và `categories[]` (`slug`, `name`, `count`). Thẻ sản phẩm là cùng kiểu object như trang danh mục.
- **Tham số đã kiểm** (trên `satel`, `sunricher`): `?category={slug}` và `?solution={slug}` trả đúng số thẻ bằng số đếm (6, 7, 2 khi kết hợp); `?sort=price-asc` đổi thứ tự; `?page=2…5` đủ 24/24/24/24/14 thẻ, `?page=6` trả 404; `?ip-rating=IP20` không có tác dụng. Với 9 thương hiệu phân trang, tổng thẻ các trang bằng đúng `product_count` (36, 32, 30, 32, 34, 60, 32, 110, 61).
- **Phản hồi bất thường**: 4/71 lần tải đầu trả HTML thiếu nội dung (52 KB đến 217 KB; 2 trong số đó không có `h1` và thẻ nào), tải lại ngay trả đủ. Chuyển đổi phải báo lỗi khi thiếu `h1` hoặc nhãn "N sản phẩm", không xuất MD rỗng.
- **Dữ liệu lệch nên biết**: `<title>` khác H1 ở nhiều thương hiệu (dạng riêng "{Tên} – … | KNXStore" hoặc mặc định "{Tên} | knxstore.vn" ở 8/71); meta description mặc định hoặc cắt "…" (65/71), JSON-LD `Brand.description` thường không có; một số thương hiệu có breadcrumb "Trang chủ" trỏ `/matter` (6/71); `og:image` là ảnh chung của site khi không có logo (47/71); mục giới thiệu của một thương hiệu (Kanonbus) chứa `h1`. 5/71 trang có NBSP hoặc ký tự vô hình trong dữ liệu (Intesis: U+00A0 trong tên sản phẩm; Steinel: U+00A0 trong `alt` ảnh; Eve và Homey: U+200E đứng đầu tên hoặc SKU; Lite-Puter: U+200B trong văn bản giới thiệu): bộ chuyển đổi phải chuẩn hóa như quy tắc ở mục 3.2.
- **Bộ chuyển đổi tham chiếu và kiểm tra chéo** (script ở thư mục tạm của phiên: `brandmd.py`, `brandmd_check.py`; chưa đưa vào dự án): chạy trên cả 71 trang thương hiệu và trên mọi trang `?page=N` của 9 thương hiệu phân trang; mọi kiểm tra áp dụng đều đạt (frontmatter khớp `canonical`, H1, meta description; breadcrumb; `product_count`; số dòng, thứ tự URL, tên, giá, SKU, nhãn tồn kho của sản phẩm; số liên kết lọc; `page_count` và `Trang sau`; trang ≥ 2 không lặp bộ lọc và giới thiệu; số heading, số bảng, chuỗi từ của phần giới thiệu; không còn thẻ HTML/NBSP/chuỗi giao diện; dựng hai lần cho cùng byte).
