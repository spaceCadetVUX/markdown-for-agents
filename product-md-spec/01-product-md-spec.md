# Trang sản phẩm — đặc tả Markdown trả về cho AI agent

> Ngày: 2026-11-11 · Phạm vi: `/products/{slug}` (752 URL) · Trạng thái: **bản nháp**.
> File này chỉ mô tả **kết quả mong muốn** (MD trông như thế nào). Cách triển khai, nguồn dữ liệu, bằng chứng và lỗi dữ liệu: `02-product-md-dev-notes.md`.
> Dựa trên khảo sát 80/752 URL và 2 mẫu hoàn chỉnh (`sample-weinzierl-886.md` skin B2B, `sample-lb-l02e.md` skin Matter). Số token là ước tính bytes/4, chưa đo bằng tokenizer.

---

## 1. Nguyên tắc

1. **Cùng thông tin với HTML, khác định dạng.** Không tóm tắt, không diễn giải, không thêm thông tin không có trên trang.
2. **Đầy đủ theo dữ liệu sản phẩm, không theo những gì HTML ban đầu đã render.** Nội dung nằm trong tab (Thông số kỹ thuật, FAQ, Hướng dẫn, Tài liệu) phải có trong MD.
3. **Chỉ nội dung sản phẩm.** Không đưa menu, chân trang, giỏ báo giá, nút thao tác, widget. Ngoại lệ duy nhất: một dòng thông tin người bán ở cuối mục Đặt hàng / báo giá.
4. **Gọn.** Hai mẫu hoàn chỉnh: LB-L02E ~3.100 token (HTML thô ~62.840), Weinzierl 886 ~4.600 token (HTML thô ~128.400); giảm 95–96%. Sản phẩm ít nội dung nhỏ hơn nhiều.
5. Khối rỗng hoặc giá trị `null` thì bỏ hẳn heading/dòng; không ghi "Không có", "N/A".

---

## 2. Frontmatter

| Key | Giá trị | Điều kiện | Ưu tiên |
|---|---|---|---|
| `type` | `product` | luôn | P3 |
| `title` | tên sản phẩm (giống H1) | luôn | P1 |
| `description` | mô tả SEO (meta description) | có | P2 |
| `canonical` | URL HTML chuẩn, tuyệt đối | luôn | P1 |
| `lang` | `vi` | luôn | P3 |
| `brand` | tên thương hiệu | luôn | P1 |
| `sku` | mã SKU | luôn | P1 |
| `mpn` | mã MPN | khi có | P2 |
| `image` | ảnh chính, URL tuyệt đối | có | P2 |
| `published` | ngày xuất bản (ISO 8601) | có | P3 |
| `price`, `currency` | số, `VND` | chỉ khi giá khác null | P1 |

- `published` là ngày xuất bản, **không** phải ngày cập nhật. Không thêm `lastmod` cho đến khi có ngày cập nhật thật.
- Key frontmatter bằng tiếng Anh; heading trong body bằng tiếng Việt (giống trang).
- Giá trị có `: ` (dấu hai chấm + khoảng trắng) hoặc dấu `"` thì đặt trong dấu nháy kép (YAML hợp lệ); ví dụ `description` của mẫu Matter.
- `sku` và `mpn` **luôn** đặt trong dấu nháy kép, vì YAML sẽ hiểu `5256` là số và làm mất số 0 đầu (ví dụ SKU `003543` thành `3543`).
- `image` dùng host `api.knxstore.vn` (giống `og:image`); ảnh trong body dùng `knxstore.vn/media/…`. Cả hai cùng trỏ tới một file.

---

## 3. Body

### 3.1 Thứ tự và heading

| # | Khối | Heading | Điều kiện hiện | Ưu tiên |
|---|---|---|---|---|
| 1 | Tên sản phẩm | `#` (H1 duy nhất) | luôn | P1 |
| 2 | Breadcrumb | không heading; 1 dòng `A > B > C`, link tuyệt đối | luôn | P2 |
| 3 | Thông tin nhanh | không heading; bảng 2 cột | luôn | P1 |
| 4 | Biến thể | `## Biến thể` | có hơn 1 variant | P1 |
| 5 | Mô tả | `## Mô tả` | có mô tả | P1 |
| 6 | Thông số kỹ thuật | `## Thông số kỹ thuật` | có dòng thông số | P1 |
| 7 | Tài liệu | `## Tài liệu` | có tài liệu tải | P1 |
| 8 | Hướng dẫn | `## Hướng dẫn` | có hướng dẫn (skin Matter) | P2 |
| 9 | FAQ | `## FAQ` | có câu hỏi | P2 |
| 10 | Hình ảnh | `## Hình ảnh` | từ 2 ảnh trở lên; tối đa 8 (mục 3.11) | P2 |
| 11 | Sản phẩm đề xuất | `## Sản phẩm đề xuất` | có sản phẩm đề xuất; tối đa 8 | P2 |
| 12 | Bài viết liên quan | `## Bài viết liên quan` | chỉ skin Matter; tối đa 3 | P3 |
| 13 | Đặt hàng / báo giá | `## Đặt hàng / báo giá` | luôn | P2 |

Thứ tự khối 5–9 theo thứ tự tab trên trang (Mô tả, Hướng dẫn, Thông số, Tài liệu, FAQ). Xem mục 6, điểm 4.

### 3.2 Quy tắc chung

- Link nội bộ dạng tuyệt đối `https://knxstore.vn/...`; giữ nguyên `tel:`, `mailto:` và link ngoài.
- UTF-8, LF, giữ tiếng Việt có dấu; không emoji; không HTML thô (ngoại lệ: bảng có ô gộp, mục 3.4).
- Không đưa text giao diện vào body: "Thêm vào yêu cầu báo giá", "Giảm/Tăng số lượng", "Thêm vào so sánh", "Bỏ qua, tới nội dung chính", "Giao hàng toàn quốc".
- Không chuẩn hóa số và đơn vị: giữ nguyên `24 V DC`, `0,5 A`, `Ø60 × 110 mm`.
- SKU của sản phẩm đề xuất lấy từ chính sản phẩm đó, không lấy từ chữ hiển thị trên card.

### 3.3 Thông tin nhanh (bảng 2 cột)

| Dòng | Giá trị | Điều kiện |
|---|---|---|
| Thương hiệu | `[Sunricher](https://knxstore.vn/brands/sunricher)` | luôn |
| Danh mục | tất cả danh mục của sản phẩm, mỗi mục một link | luôn |
| Giải pháp | `[DALI](https://knxstore.vn/solution/dali)` | chỉ khi trang hiển thị giải pháp (mẫu B2B có; mẫu Matter không) |
| Giá | Hiển thị đúng như HTML. B2B không có giá: `Liên hệ để nhận báo giá`. B2B có giá: `13.880.000 đ (giá đã bao gồm VAT)`. Matter: `477.000đ (giá cũ 530.000đ, -10%, đã bao gồm VAT), đặt trước` | luôn |
| Xuất xứ | Giữ nguyên chữ trang hiển thị (`CN`, `Germany`) | có |
| Bảo hành | `{n} tháng` | có |
| Giao thức | `Thread, Bluetooth, Zigbee` | chỉ skin Matter |
| Hoạt động cùng | `Apple Home, Google Home, SmartThings, Alexa, Homey, Home Assistant` | chỉ skin Matter |

Tình trạng tồn kho chỉ xuất hiện khi trang đang hiển thị (mẫu Matter: "đặt trước" nằm trong dòng Giá); trang không hiển thị thì không đưa vào (mục 6, điểm 1).

### 3.4 Mô tả

| Phần tử trong mô tả | Kết quả trong MD |
|---|---|
| Heading `h2`, `h3` | **Hạ 1 cấp** (`h2`→`###`, `h3`→`####`, tối đa `######`) để nằm dưới `## Mô tả` |
| Đoạn văn | giữ nguyên |
| Danh sách | `-` hoặc `1.`; mỗi mục một dòng |
| Ảnh | `![alt](https://knxstore.vn/media/{sha256}.{ext})`; giữ `alt` nguyên văn (mẫu: `Sơ đồ kết nối`); `alt` rỗng thì giữ rỗng `![](URL)`; dùng URL ảnh gốc; không tự sinh mô tả ảnh |
| Bảng | Bảng Markdown (GFM) với số hàng, số cột bằng trên trang; ký tự `|` trong ô được escape thành `\|`. Bảng có ô gộp (`rowspan`/`colspan`) thì giữ nguyên `<table>` HTML |
| Link | tuyệt đối hóa link tương đối; giữ chữ hiển thị |
| Định dạng và thẻ bao (`span`, `class`, `style`) | bỏ, giữ chữ; giữ khoảng trắng quanh mã model trong tên |
| `script`, `style`, `iframe` | bỏ |

### 3.5 Biến thể

Bảng `| Biến thể | Tùy chọn | SKU | Giá |`, đánh dấu `(mặc định)` cho variant mặc định. Ghi đúng dữ liệu, không tự sửa SKU.

```markdown
| Biến thể | Tùy chọn | SKU | Giá |
|---|---|---|---|
| Xám (mặc định) | Color: Xám | SR-2422NK8-DA2 | Liên hệ |
| Trắng | Color: Trắng | SR-2422NK8-DA2 | Liên hệ |
```

### 3.6 Thông số kỹ thuật

Bảng 2 cột `| Thông số | Giá trị |`, giữ thứ tự gốc, không lọc dòng nào (kể cả `Model`). Giá trị là chữ giá trị của dòng thông số; nối thêm đơn vị chỉ khi đơn vị đã được xác thực (`parseStatus` = `exact`) và chữ giá trị chưa chứa đơn vị đó.

### 3.7 Tài liệu

```markdown
- [{tên hiển thị}](https://knxstore.vn/media/docs/{sha256}.pdf) — PDF, {dung lượng}
```

- Tên hiển thị = nhãn tài liệu nếu có, không thì tên file (hầu hết tài liệu hiện chỉ có tên file, ví dụ `SR-2421-MT2_instruction_KNXStore.pdf`).
- Dung lượng làm tròn theo cơ số 1.024 (1 KB = 1.024 byte): `450 KB`, `1,3 MB`. Agent dùng con số này để quyết định có tải file hay không (file khảo sát từ 28 KB đến 17 MB).
- Heading luôn là `## Tài liệu` (kể cả skin Matter gọi là "Datasheet").
- Tài liệu có liên kết không khả dụng thì ghi `{tên} — liên kết không khả dụng`, không gắn URL.
- Sản phẩm không có tài liệu thì bỏ cả mục.

### 3.8 FAQ

Mỗi Q&A một mục danh sách, không dùng heading:

```markdown
- **{câu hỏi}** {câu trả lời}
```

Câu trả lời nhiều đoạn thì thụt lề 2 dấu cách.

### 3.9 Sản phẩm đề xuất và Bài viết liên quan

```markdown
- [{tên}](https://knxstore.vn/products/{slug}) — {Brand} · {SKU} · {giá | Liên hệ | Giá liên hệ}
```

- Thứ tự như trên trang, tối đa 8. `{Brand}` và `{SKU}` viết đúng như trên card (ví dụ `DALCNET`, `Art.Nr. 89452937-3`).
- Giá định dạng giống HTML: `3.190.000 đ`, `Liên hệ`, `Giá liên hệ`; sản phẩm giảm giá ghi `542.000 đ (giá cũ 602.000 đ, -10%)`.
- Bài viết liên quan (Matter): `- [{tiêu đề}](https://knxstore.vn/{slug}.html) — {tác giả}, {dd/mm/yyyy}`, tối đa 3. Bài không có tác giả thì chỉ ghi ngày.

### 3.10 Đặt hàng / báo giá

| Trường hợp | Nội dung |
|---|---|
| Sản phẩm báo giá (mẫu B2B, kể cả khi trang có hiển thị giá) | `- Gửi yêu cầu báo giá: hotline [0918.918.755](tel:0918918755)`, `- Zalo: https://zalo.me/0918918755`, `- Liên hệ tư vấn dự án: https://knxstore.vn/contact` |
| Sản phẩm có giá, bán trực tiếp (mẫu Matter) | `- Mua trực tiếp trên trang sản phẩm: {canonical}`, `- Liên hệ tư vấn: [{hotline}](tel:…)`; hotline đúng như khung mua của trang đó |

Các dòng liên hệ viết dạng danh sách `-` (để không bị gộp thành một đoạn khi render). Sau các dòng đó thêm 1 đoạn riêng cho người bán: `CÔNG TY CP TÍCH HỢP HỆ THỐNG LIÊN MINH · MST 0311846236 · support@knxstore.vn`. Mục này thay cho nút "Thêm vào yêu cầu báo giá" mà agent không bấm được.

### 3.11 Hình ảnh

```markdown
- [Ảnh {n} trên {tổng}](https://knxstore.vn/media/{sha256}.{ext}) — {rộng}×{cao}
```

- Chỉ hiện khi sản phẩm có từ 2 ảnh trở lên (ảnh duy nhất đã có ở frontmatter `image`). Thứ tự như thư viện ảnh trên trang; ảnh 1 là ảnh chính.
- Tối đa 8 dòng; nhiều hơn thì thêm dòng `- còn {n} ảnh khác`.
- Ghi thêm `, {alt}` sau kích thước chỉ khi `alt` có và khác tên sản phẩm.
- Dùng URL ảnh gốc, kèm kích thước để agent biết độ phân giải trước khi tải.

---

## 4. Không đưa vào MD

| Loại bỏ | Lý do |
|---|---|
| Header (menu lớn ~165 link, ô tìm kiếm), footer (cột danh mục, giải pháp, chính sách, huy hiệu) | Điều hướng lặp trên mọi trang, không phải nội dung sản phẩm (header ~3,977 ký tự, footer ~1,110 ký tự) |
| Giỏ/drawer báo giá, thanh mua mobile, nút Zalo nổi, điều khiển số lượng, nút so sánh | Chỉ là giao diện |
| JSON-LD dán nguyên khối | Thông tin đã có ở frontmatter và body; node Organization lặp trên mọi trang (~980 token) |
| Cờ nội bộ (`b2b`, `b2c`, `quotable`, `status`); tình trạng tồn kho khi trang không hiển thị | Không hiển thị trên trang (tồn kho: xem 3.3) |

---

## 5. Ví dụ

### 5.1 Mẫu hoàn chỉnh (không có placeholder)

- Skin B2B, đủ Mô tả, Thông số, Tài liệu (PDF), FAQ, Sản phẩm đề xuất: `sample-weinzierl-886.md`.
- Skin Matter, đủ Mô tả, Thông số, FAQ, Sản phẩm đề xuất, Bài viết liên quan: `sample-lb-l02e.md`.

### 5.2 Trường hợp tối giản: có biến thể, không giá, thiếu Thông số / Tài liệu / FAQ (mẫu SR-2422NK8-DA2; phần chưa lấy nội dung để `<…>`)

````markdown
---
type: product
title: Phím bấm gắn tường DALI-2 8 phím Sunricher - SR-2422NK8-DA2
description: Phím bấm SR-2422NK8-DA2 là thiết bị điều khiển DALI-2 với 8 phím, chứng nhận DALI-2, thiết kế mỏng, hỗ trợ chức năng phản hồi và 16 đèn LED.
canonical: https://knxstore.vn/products/sunricher-phim-bam-gan-tuong-dali-2-8-phim-sr-2422nk8-da2
lang: vi
brand: Sunricher
sku: "SR-2422NK8-DA2"
image: https://api.knxstore.vn/media/b63b9aea….png
published: 2025-11-03
---
# Phím bấm gắn tường DALI-2 8 phím Sunricher - SR-2422NK8-DA2

[Trang chủ](https://knxstore.vn/) > [Công tắc thông minh](https://knxstore.vn/cong-tac-thong-minh) > Phím bấm gắn tường DALI-2 8 phím Sunricher - SR-2422NK8-DA2

| | |
|---|---|
| Thương hiệu | [Sunricher](https://knxstore.vn/brands/sunricher) |
| Danh mục | [Công tắc thông minh](https://knxstore.vn/cong-tac-thong-minh), [<tên>](https://knxstore.vn/cong-tac-dali) |
| Giải pháp | [DALI](https://knxstore.vn/solution/dali) |
| Giá | Liên hệ để nhận báo giá |
| Xuất xứ | CN |

## Biến thể

| Biến thể | Tùy chọn | SKU | Giá |
|---|---|---|---|
| Xám (mặc định) | Color: Xám | SR-2422NK8-DA2 | Liên hệ |
| Trắng | Color: Trắng | SR-2422NK8-DA2 | Liên hệ |

## Mô tả

### Tổng quan về phím bấm gắn tường DALI-2 8 phím Sunricher - SR-2422NK8-DA2
<2 đoạn văn …>

#### Các tính năng chính của SR-2422NK8-DA2
- <chuẩn IEC62386-101/-103/-301/-332 …>
- <16 LED phản hồi …>
- <… tổng 10 mục>

#### Sơ đồ kết nối của SR-2422NK8-DA2
![Sơ đồ kết nối](https://knxstore.vn/media/70f8af78….jpg)

## Hình ảnh
(… 2 ảnh, mỗi dòng `[Ảnh n trên 2](URL) — rộng×cao`)

## Sản phẩm đề xuất
- [<tên>](https://knxstore.vn/products/<slug>) — <Brand> · <SKU> · <giá>
(… tối đa 8)

## Đặt hàng / báo giá
- Gửi yêu cầu báo giá: hotline [0918.918.755](tel:0918918755)
- Zalo: https://zalo.me/0918918755
- Liên hệ tư vấn dự án: https://knxstore.vn/contact

CÔNG TY CP TÍCH HỢP HỆ THỐNG LIÊN MINH · MST 0311846236 · support@knxstore.vn
````

Mẫu này không có khối Thông số, Tài liệu, FAQ vì sản phẩm chưa có dữ liệu các mục đó, và không có `price`, `currency` vì chưa có giá.

---

## 6. Quyết định còn mở về output

| # | Điểm | Mặc định trong spec này | Người chốt |
|---|---|---|---|
| 1 | Tình trạng tồn kho có vào MD không | Chỉ khi trang hiển thị (Matter: "đặt trước"). Mẫu B2B có `out_of_stock` trong dữ liệu nhưng trang không hiển thị nên không đưa vào; chờ xác nhận dữ liệu đúng | Tùng / Huy |
| 2 | Có giữ JSON-LD trong MD không | Không. Nếu cần thì chỉ một `Product` JSON-LD dựng từ dữ liệu sản phẩm, không có Organization | Tùng |
| 3 | Số sản phẩm đề xuất tối đa | 8 | Tùng |
| 4 | Thứ tự khối: theo tab trang hay đưa Thông số + Tài liệu lên trước Mô tả | Theo tab trang; cân nhắc đưa Thông số lên trước nếu agent cắt ngắn nội dung dài | Tùng |
| 5 | Hạ heading trong mô tả 1 cấp | Có | Tùng |
| 6 | FAQ dạng danh sách, không heading | Có | Tùng |
| 7 | `price`, `currency` ở frontmatter khi có giá | Có (để agent/RAG đọc có cấu trúc); body vẫn hiển thị như HTML | Tùng / Vũ |
| 8 | Ảnh thư viện sản phẩm | **Đã đưa** (3.11): URL ảnh gốc + kích thước, tối đa 8, chỉ khi có từ 2 ảnh. Khảo sát: 21/81 trang chỉ có 1 ảnh, trung vị 3 ảnh, nhiều nhất 21 | Tùng |
| 9 | Chân trang và dòng "Xem toàn bộ" (`/products`, `/brands`, `/solution`, `/llms.txt`) | **Không đưa.** Đã thử ở bản trước (~575 token mỗi trang), bỏ theo quyết định của Tùng. Có thể thêm lại sau dạng 1 dòng liên kết mục lục vào cuối mục Đặt hàng / báo giá | Tùng |
