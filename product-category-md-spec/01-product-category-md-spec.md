# Trang danh mục sản phẩm — đặc tả Markdown trả về cho AI agent

> Ngày: 2026-11-11 · Phạm vi: `/{slug}` (91 URL trong `sitemaps/categories.xml`, cả skin B2B và Matter) · Trạng thái: **bản nháp**.
> File này chỉ mô tả **kết quả mong muốn** (MD trông như thế nào). Cách triển khai, nguồn dữ liệu, bằng chứng và lỗi dữ liệu: `02-product-category-md-dev-notes.md`.
> Dựa trên khảo sát 14 danh mục (9 B2B, 5 Matter) và 2 mẫu hoàn chỉnh (`sample-dimmer-triac.md` skin B2B, `sample-cong-tac-matter.md` skin Matter). Số token là ước tính bytes/4, chưa đo bằng tokenizer. Cùng khung với `product-md-spec/01-product-md-spec.md`.

---

## 1. Nguyên tắc

1. **Cùng thông tin với HTML, khác định dạng.** Không tóm tắt, không diễn giải, không thêm thông tin không có trên trang.
2. **Mỗi URL một bản MD, chỉ chứa phần của URL đó.** Trang 2 không chứa sản phẩm trang 1; không gộp các trang phân trang vào một file.
3. **Danh sách sản phẩm của trang đầy đủ, không cắt.** (Giới hạn do phân trang của chính site: 24 sản phẩm mỗi trang ở B2B.)
4. **Khối lặp giữa các trang (bộ lọc, giới thiệu, FAQ) chỉ ở trang 1.** HTML lặp chúng nguyên văn ở mọi trang, nên bỏ ở trang 2 trở đi làm MD nhỏ hơn mà không mất dữ kiện.
5. **Chỉ nội dung danh mục.** Không menu, chân trang, giỏ báo giá, nút thao tác, JSON-LD.
6. Khối rỗng hoặc giá trị `null` thì bỏ hẳn heading/dòng; không ghi "Không có", "N/A".
7. Mọi lối đi tiếp (danh mục liên quan, bộ lọc, phân trang, sản phẩm) là liên kết tuyệt đối `https://knxstore.vn/...`.
8. **Gọn.** Trang 1 của danh mục 29 sản phẩm: ~4.200 token (HTML thô ~133.800), giảm 96,9%. Trang 2 chỉ ~530 token.

---

## 2. Frontmatter

| Key | Giá trị | Điều kiện | Ưu tiên |
|---|---|---|---|
| `type` | `category` | luôn | P3 |
| `title` | tên danh mục (giống H1) | luôn | P1 |
| `description` | meta description nguyên văn | có | P2 |
| `canonical` | URL HTML chuẩn, tuyệt đối; trang N ≥ 2 có `?page=N` | luôn | P1 |
| `lang` | `vi` | luôn | P3 |
| `product_count` | tổng số sản phẩm của danh mục (mọi trang), không phải số dòng của trang | có | P1 |
| `page` | số trang hiện tại (mặc định 1) | luôn | P1 |
| `page_count` | tổng số trang (1 khi không có phân trang) | luôn | P1 |
| `query` | tham số `brand`, `solution`, `sort` của URL yêu cầu, ví dụ `brand=vadsbo&sort=price-asc` | chỉ khi URL có các tham số này | P2 |
| `image` | ảnh đại diện (`og:image`), URL tuyệt đối | có | P3 |

- `description` giữ nguyên chuỗi nguồn, kể cả khi đã bị cắt và kết thúc bằng "…" (5/14 danh mục khảo sát; nguồn không có bản đầy đủ).
- Không có ngày (`published`, `modified`): xem mục 6, điểm 3.
- Key frontmatter bằng tiếng Anh; heading trong body bằng tiếng Việt (giống trang).
- Giá trị có `: ` hoặc dấu `"` thì đặt trong dấu nháy kép (YAML hợp lệ).
- `canonical` của URL có `brand`, `solution`, `sort`, `view` vẫn là URL danh mục gốc (đúng như HTML); `query` cho biết đó là danh sách đã lọc hoặc sắp xếp.

---

## 3. Body

### 3.1 Thứ tự và heading

| # | Khối | Heading | Điều kiện hiện | Ưu tiên |
|---|---|---|---|---|
| 1 | Tên danh mục | `#` (H1 duy nhất) | luôn | P1 |
| 2 | Breadcrumb | không heading; 1 dòng `A > B > C` | luôn | P2 |
| 3 | Thông tin nhanh | không heading; bảng 2 cột | luôn (có ít nhất một dòng) | P1 |
| 4 | Danh mục liên quan | `## Danh mục liên quan` | có chip danh mục | P2 |
| 5 | Bộ lọc và sắp xếp | `## Bộ lọc và sắp xếp` | chỉ trang 1, có sản phẩm | P2 |
| 6 | Sản phẩm | `## Sản phẩm` | có sản phẩm | P1 |
| 7 | Phân trang | `## Phân trang` | `page_count > 1` | P1 |
| 8 | Giới thiệu danh mục | `## Giới thiệu danh mục` | chỉ trang 1, có văn bản | P2 |
| 9 | FAQ | `## FAQ` | chỉ trang 1, có câu hỏi | P2 |

Thứ tự đặt dữ kiện có cấu trúc và điều hướng trước, văn xuôi dài (giới thiệu, FAQ) sau; khi agent cắt ngắn nội dung thì phần quan trọng nhất còn lại.

### 3.2 Quy tắc chung

- Link nội bộ dạng tuyệt đối; giữ nguyên `?page=N` như HTML.
- UTF-8, LF; không emoji; không HTML thô (ngoại lệ duy nhất: `<br>` trong ô bảng).
- Không đưa text giao diện: "Thêm vào yêu cầu báo giá", "So sánh", "Bộ lọc 0", "Xoá bộ lọc", "Xem dạng lưới/bảng", "Không có sản phẩm nào khớp bộ lọc".
- Không chuẩn hóa số và đơn vị; giữ nguyên chữ nguồn (kể cả lỗi chính tả và số liệu không khớp phần khác của trang).
- Ký tự đặc biệt trong chữ lấy từ dữ liệu (`*`, `[`, `]`, `` ` ``) được escape bằng `\`.
- Dấu cách không ngắt (NBSP) thành dấu cách thường; ký tự định dạng vô hình (U+200B đến U+200F, U+202A đến U+202E, U+2060, U+FEFF) bị bỏ; dấu cách đơn dư cuối dòng bị bỏ (ngắt dòng cứng từ `<br>` giữ hai dấu cách).

### 3.3 Breadcrumb và Thông tin nhanh

Breadcrumb: một dòng, các mục trước mục cuối là liên kết, mục cuối là chữ thường:

```markdown
[Trang chủ](https://knxstore.vn/) > [Dimmer & Relay chiếu sáng](https://knxstore.vn/dieu-khien-chieu-sang) > Dimmer Triac
```

Skin Matter: mục đầu "Trang chủ" trỏ `https://knxstore.vn/matter` (đúng như trang). Danh mục cấp một chỉ có 2 mục (`Trang chủ > Tên`).

Thông tin nhanh là bảng 2 cột (cột tiêu đề rỗng), các dòng:

| Dòng | Giá trị | Điều kiện |
|---|---|---|
| Số sản phẩm | `29` | có `product_count` |
| Trang | `1 trên 2` | `page_count > 1` |
| Xem ở trang 1 | `bộ lọc, giới thiệu danh mục và FAQ (https://knxstore.vn/dimmer-doc-lap-triac)` | `page ≥ 2` |

### 3.4 Danh mục liên quan

```markdown
- [Dimmer DALI](https://knxstore.vn/dimmer-doc-lap-dali)
```

- Các liên kết danh mục mà trang hiển thị ngay dưới tên danh mục (chip), đúng thứ tự trang. Là danh mục con nếu có; không thì các danh mục cùng cấp; danh mục cấp một không có sản phẩm thì hiện toàn bộ 12 danh mục cấp một.
- Không gồm liên kết trong breadcrumb. Skin Matter không có chip nên bỏ mục.
- Hiện ở mọi trang phân trang (nhỏ, giúp điều hướng).

### 3.5 Bộ lọc và sắp xếp

Chỉ ở trang 1 và khi danh mục có sản phẩm. Ba dòng:

```markdown
- Thương hiệu: [Vadsbo](https://knxstore.vn/dimmer-doc-lap-triac?brand=vadsbo) (7) · [Lite-Puter](…?brand=lite-puter) (4) · …
- Giải pháp: [Giải pháp khác](…?solution=giai-phap-khac) (8) · [Casambi](…?solution=casambi) (5) · …
- Sắp xếp: thêm `sort=` vào URL: `newest` (Mới nhất), `name-asc` (Tên A → Z), `name-desc` (Tên Z → A), `price-asc` (Giá thấp → cao), `price-desc` (Giá cao → thấp). Mặc định không có tham số. Kết hợp được, ví dụ `?brand={slug}&sort=price-asc`.
```

- Thứ tự như trang (giảm dần theo số đếm). Số trong ngoặc là số sản phẩm của cả danh mục với giá trị đó. Dòng nào không có dữ liệu thì bỏ (Matter chỉ có giải pháp "Matter").
- `{base}` là URL danh mục không có `?page`. Liên kết lọc dùng đúng tham số mà site chạy ở server (đã kiểm, xem file dev mục 2.4).

### 3.6 Sản phẩm

Mỗi sản phẩm của trang một dòng, đúng thứ tự trang, không cắt:

```markdown
- [{tên}](https://knxstore.vn/products/{slug}) — {thương hiệu} · {SKU} · {giá}{ (giá cũ …)} · {nhãn tồn kho}
```

| Thành phần | Quy tắc |
|---|---|
| Tên | tên sản phẩm nguyên văn |
| Thương hiệu | bỏ khi sản phẩm không có thương hiệu |
| SKU | **chỉ khi card trên trang hiển thị mã** (card B2B có, card Matter không); bỏ khi sản phẩm không có SKU |
| Giá | `3.190.000 đ` (dấu chấm nghìn, một khoảng trắng; trang Matter viết dính "2.850.000đ", MD luôn có khoảng trắng). Sản phẩm có khoảng giá: `từ 600.000 đ` (trang chỉ hiện cận dưới). Không giá: `Liên hệ`. Giá ẩn: `Giá liên hệ`. Không ghi "(đã bao gồm VAT)" vì card không hiển thị |
| Giá cũ | khi sản phẩm giảm giá: ` (giá cũ 18.987.000 đ)` ngay sau giá; không ghi phần trăm giảm |
| Nhãn tồn kho | chỉ khi card hiển thị nhãn (skin Matter): `Đặt trước`, `Sắp hết hàng`, `Còn hàng`. Card B2B không hiển thị tồn kho nên không in |

Không đưa ảnh card.

### 3.7 Phân trang

```markdown
- Trang trước: https://knxstore.vn/dimmer-doc-lap-triac
- Trang sau: https://knxstore.vn/dimmer-doc-lap-triac?page=2
- Các trang: [1](https://knxstore.vn/dimmer-doc-lap-triac) · [2](https://knxstore.vn/dimmer-doc-lap-triac?page=2)
```

- `Trang trước` khi `page ≥ 2`; `Trang sau` khi chưa phải trang cuối. Trang 1 là URL gốc không có `?page`.
- `?page=N` vượt số trang: 404 (giống HTML). `?page=0` hoặc không phải số: trang 1 (giống HTML).

### 3.8 Giới thiệu danh mục

Văn bản mô tả nằm dưới lưới sản phẩm (do CMS nhập), chuyển sang Markdown theo đúng quy tắc mô tả của trang sản phẩm:

| Phần tử | Kết quả |
|---|---|
| Heading `h2`, `h3` | hạ 1 cấp (`h2`→`###`, `h3`→`####`, tối đa `######`) để nằm dưới `## Giới thiệu danh mục` |
| Đoạn văn | giữ nguyên |
| Danh sách | `-` hoặc `1.`; danh sách lồng nhau thụt lề |
| Bảng | bảng Markdown (GFM), số hàng/cột bằng trang; `\|` trong ô được escape; xuống dòng trong ô thành `<br>`; ô tiêu đề rỗng giữ rỗng |
| Ảnh | `![alt](URL gốc)`, giữ `alt` nguyên văn, không tự sinh mô tả |
| Link | tuyệt đối hóa link tương đối |
| Thẻ bao, `class`, `style` | bỏ, giữ chữ |

Tiêu đề "Về {tên danh mục}" mà trang đặt phía trên văn bản được thay bằng heading cố định `## Giới thiệu danh mục` (tên danh mục đã có ở H1).

### 3.9 FAQ

Mỗi Q&A một mục danh sách, không dùng heading; đáp án nhiều đoạn thì thụt lề 2 dấu cách:

```markdown
- **Dimmer Triac là gì?** Dimmer Triac là thiết bị điều chỉnh độ sáng đèn bằng cách cắt pha sóng sin …
```

### 3.10 URL biến thể

| URL yêu cầu | Bản MD |
|---|---|
| `/{slug}` | trang 1 |
| `/{slug}?page=N` (N ≥ 2, trong khoảng) | trang N: không có Bộ lọc, Giới thiệu, FAQ; có dòng "Xem ở trang 1" |
| `/{slug}?page=N` ngoài khoảng | 404 |
| `/{slug}?brand=…`, `?solution=…`, `?sort=…` (kết hợp được) | trang 1 của danh sách đã lọc/sắp xếp; `canonical` là URL gốc; có `query` |
| `/{slug}?view=list` | như trang 1 (chỉ khác bố cục lưới/bảng); tham số lạ bị bỏ qua |

### 3.11 Trường hợp đặc biệt

| Trường hợp | Kết quả |
|---|---|
| Danh mục 0 sản phẩm | `product_count: 0`; bỏ Bộ lọc và Sản phẩm; giữ Danh mục liên quan (12 danh mục cấp một) và Giới thiệu; không in thông báo "Không có sản phẩm nào khớp bộ lọc" (sai nghĩa) |
| Không có văn bản giới thiệu hoặc FAQ | bỏ mục (có 2/14 danh mục không có văn bản; 8/14 không có FAQ) |
| Chỉ một trang | không có mục Phân trang; `page_count: 1`; không có dòng "Trang" |

---

## 4. Không đưa vào MD

| Loại bỏ | Lý do |
|---|---|
| Header (menu ~165 liên kết), footer, nút Zalo nổi, giỏ/drawer báo giá | Điều hướng lặp trên mọi trang, không phải nội dung danh mục |
| Nút "Thêm vào báo giá", so sánh, chuyển lưới/bảng, "Bộ lọc 0", "Xoá bộ lọc" | Chỉ là giao diện |
| JSON-LD (`CollectionPage`, `Service`, `ItemList`, `FAQPage`, `Organization`) | Thông tin đã có trong MD; thống nhất không đưa JSON-LD (giống trang sản phẩm) |
| Ảnh card, tồn kho của card B2B, `isFeatured`, `shortDescription`, cờ thanh lý | Không hiển thị trên trang |
| Ngày xuất bản / cập nhật | Chưa có ngày thật (mục 6, điểm 3) |

---

## 5. Ví dụ

Hai mẫu hoàn chỉnh, không có placeholder:

- Skin B2B, 29 sản phẩm chia 2 trang, có chip, bộ lọc, giới thiệu, FAQ: `sample-dimmer-triac.md` (trang 1).
- Skin Matter, 12 sản phẩm một trang, card có nhãn tồn kho, không SKU, không chip: `sample-cong-tac-matter.md`.

Trang 2 của danh mục B2B (`/dimmer-doc-lap-triac?page=2`, 5 sản phẩm, 2.116 B):

````markdown
---
type: category
title: Dimmer Triac
description: Dimmer Triac chính hãng, điều khiển độ sáng đèn LED mượt, tích hợp KNX, Casambi, DALI, DMX. Tư vấn chọn đúng driver tương thích, báo giá ngay.
canonical: https://knxstore.vn/dimmer-doc-lap-triac?page=2
lang: vi
product_count: 29
page: 2
page_count: 2
image: https://knxstore.vn/media/1eec55b4679baf5165d5a32bb4af07035960c1ce1b0e591c1b738174dcf254bd.jpg
---

# Dimmer Triac

[Trang chủ](https://knxstore.vn/) > [Dimmer & Relay chiếu sáng](https://knxstore.vn/dieu-khien-chieu-sang) > Dimmer Triac

| | |
|---|---|
| Số sản phẩm | 29 |
| Trang | 2 trên 2 |
| Xem ở trang 1 | bộ lọc, giới thiệu danh mục và FAQ (https://knxstore.vn/dimmer-doc-lap-triac) |

## Danh mục liên quan

- [Dimmer DALI](https://knxstore.vn/dimmer-doc-lap-dali)
- [Dimmer DMX](https://knxstore.vn/dmx)
- [Điều khiển 0-10V](https://knxstore.vn/dimmer-doc-lap-0-10v)
- [Điều khiển ON/OFF](https://knxstore.vn/dieu-khien-chieu-sang-on-off)

## Sản phẩm

- [Bộ điều khiển chiếu sáng triac KNX Kanonbus - KTE45BD](https://knxstore.vn/products/kanonbus-bo-dieu-khien-chieu-sang-triac-knx-kte45bd) — Kanonbus · KTE45BD · 8.729.000 đ
- [Dimmer độc lập Triac - WALL-DIMMER-230V-TRIAC](https://knxstore.vn/products/ai-control-dimmer-doc-lap-triac) — OEM · WALL-DIMMER-230V-TRIAC · 2.100.000 đ (giá cũ 2.500.000 đ)
- [Bộ điều khiển RF Lutron - RRK-R6ANS-240](https://knxstore.vn/products/lutron-bo-dieu-khien-rf) — Lutron · RRK-R6ANS-240 · 4.617.600 đ
- [Dimmer điều chỉnh độ sáng có dây RA2 Lutron - RRK-R25NE-240](https://knxstore.vn/products/lutron-dimmer-dieu-chinh-do-sang-dong-ra2-co-day) — Lutron · RRK-R25NE-240 · 4.617.600 đ
- [Dimmer xoay cho đèn led Vadsbo - VD300](https://knxstore.vn/products/vadsbo-dimmer-xoay-cho-den-led-vd300) — Vadsbo · VD300 · 1.965.000 đ

## Phân trang

- Trang trước: https://knxstore.vn/dimmer-doc-lap-triac
- Các trang: [1](https://knxstore.vn/dimmer-doc-lap-triac) · [2](https://knxstore.vn/dimmer-doc-lap-triac?page=2)
````

Danh mục 0 sản phẩm (`/hang-thanh-ly`): chỉ có frontmatter (`product_count: 0`), H1, breadcrumb, bảng (1 dòng "Số sản phẩm | 0"), `## Danh mục liên quan` (12 liên kết) và `## Giới thiệu danh mục`.

---

## 6. Quyết định còn mở về output

| # | Điểm | Mặc định trong spec này | Người chốt |
|---|---|---|---|
| 1 | Liên kết lọc `?brand=`, `?solution=` trong mục Bộ lọc | Có. Đây là phần thêm so với HTML (trang chỉ có ô chọn không có URL), dùng tham số đã kiểm. Bỏ liên kết (chỉ giữ tên và số đếm) giảm mục này từ 1.816 B xuống 581 B (trang 1 mẫu: −7,4%) | Tùng |
| 2 | Dòng "Sắp xếp" (5 giá trị `sort`) | Có | Tùng |
| 3 | Ngày | Không xuất `published` (14/14 danh mục cùng ngày 2026-08-24, nhiều khả năng mốc nhập hàng loạt) và `modified` (`dateModified` khác ở 9/14 nhưng chưa xác nhận là ngày sửa thật). Thêm lại khi xác nhận | Tùng / Vũ |
| 4 | SKU chỉ khi card hiển thị mã (Matter không có SKU) | Có | Tùng |
| 5 | Nhãn tồn kho ở card Matter | Có, đúng như trang | Tùng |
| 6 | Trang ≥ 2 không lặp Bộ lọc, Giới thiệu, FAQ | Có | Tùng |
| 7 | Hạ heading trong văn bản giới thiệu 1 cấp (`###`) | Có (giống trang sản phẩm) | Tùng |
| 8 | FAQ dạng danh sách, không heading | Có (giống trang sản phẩm) | Tùng |
| 9 | Danh mục 0 sản phẩm: giữ 12 danh mục liên quan | Có | Tùng |
