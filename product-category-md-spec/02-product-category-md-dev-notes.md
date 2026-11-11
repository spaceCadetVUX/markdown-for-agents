# Trang danh mục → Markdown — ghi chú cho dev

> Ngày: 2026-11-11 · Đi kèm `01-product-category-md-spec.md` (kết quả mong muốn) và 2 mẫu đầu ra `sample-dimmer-triac.md` (B2B), `sample-cong-tac-matter.md` (Matter).
> File này gồm: dữ liệu danh mục nằm ở đâu, kết quả khảo sát, nguồn của từng trường, lỗi dữ liệu và việc còn thiếu.
> Nguồn: `site-structure/03-product-category.md`, `site-structure/14-matter-category.md`, `markdown-for-agents-nextjs-direction.md`, và các lần kiểm tra bằng `curl` ngày 2026-11-11 (UA `Mozilla/5.0`, không chạy JS).
> Nhãn: **[Đã kiểm]** = đã đo trong phiên làm việc; **[Suy ra]** = suy ra từ HTML hoặc ý tưởng thiết kế, chưa chạy thật.

---

## 1. Kết luận kỹ thuật

Khác trang sản phẩm: danh mục **không giấu nội dung sau tab**. Danh sách sản phẩm, FAQ (cả đáp án) và văn bản giới thiệu đều có text trong DOM ban đầu. Nhưng payload RSC của danh mục **không có đối tượng "category"** (không có prop `product` hay tương đương); trang được dựng thành cây JSX phía server.

Vẫn nên dựng MD từ dữ liệu server (kết quả các hàm mà `page.tsx` của route `(site)/[slug]` gọi), không parse HTML, vì:
- Giá card: DOM chỉ hiện cận dưới của khoảng giá; có card `price: null` nhưng `priceRange` có giá (lỗi #1 mục 5). Đọc DOM sẽ dễ ghi sai.
- Bộ lọc (facet) và sắp xếp nằm trong khối `div[hidden]` ngoài `#main-content`.
- HTML đôi khi không có payload (mục 2.5).

Tên hàm và model phía server: chưa xác định (chỉ quan sát từ phía client). Bộ chuyển đổi tham chiếu đã dựng bằng DOM + object card + props bộ lọc + JSON-LD (mục 4) để chứng minh spec làm được.

---

## 2. Dữ liệu nằm ở đâu và kết quả kiểm tra

### 2.1 Mỗi dữ kiện nằm ở đâu **[Đã kiểm]**

| Dữ kiện | DOM ban đầu | Payload RSC | JSON-LD / `<head>` |
|---|---|---|---|
| Tên | `h1` | cây JSX | `CollectionPage.name`; `<title>` thường khác (riêng ở 11/14 danh mục) |
| Số sản phẩm | B2B: số cạnh `h1`. Matter: thanh công cụ cuối trang (`div[hidden]#S:0`) | không tách | `ItemList.numberOfItems` là số card của trang, không phải tổng |
| Breadcrumb | `nav` (Matter: đứng sau `h1`) | không tách | `CollectionPage.breadcrumb` |
| Chip danh mục | liên kết giữa `h1` và lưới | prop `chips` | không |
| Sản phẩm của trang | `article` (B2B 24, Matter 12) | object card 18 khóa | `ItemList` chỉ `position`, `url` |
| Facet + số đếm | `#S:0` (hidden) | props bộ lọc `brands[]`, `solutions[]` (`slug`, `name`, `count`) | không |
| Sắp xếp | `select option` trong `#S:0` | không | không |
| Văn bản giới thiệu | `div.user-html` trong `section` dưới lưới | dòng text HTML | không |
| FAQ | `details` (đáp án có trong DOM dù đóng) | cây JSX | `FAQPage` (6/6 khớp DOM) |
| Mô tả ngắn | không | cùng chuỗi | `CollectionPage.description` = meta (5/14 kết thúc "…") |
| Ngày | không | không | `datePublished` 14/14 cùng ngày; `dateModified` khác ở 9/14 |
| Ảnh đại diện | không | không | `og:image`, `primaryImageOfPage` (ảnh mặc định của site ở một số danh mục) |

### 2.2 Cách đọc payload **[Đã kiểm]**

1. Giải mã `self.__next_f.push([1,"…"])`, nối lại (như file dev của trang sản phẩm).
2. **Object card**: `{id, slug, name, sku, shortDescription, brand, primaryImage, isFeatured, b2c, b2b, stockStatus, price, oldPrice, priceRange, clearance, priceHidden, clearanceFrom, clearanceCount}`. Cùng kiểu object cũng xuất hiện ở khối Trending của header (16 sản phẩm dùng chung mọi trang), nên phải khớp theo `slug` của các `article` trong lưới, không lấy "object đầu tiên".
3. **Facet**: tìm `"optionFilters"`, ngay sau là `"brands":[…]` và `"solutions":[…]` (`FeaturedFilters`).
4. Văn bản giới thiệu lấy từ DOM (`div.user-html` đầu tiên không nằm trong `article`/`details`); FAQ từ `details`.

### 2.3 Khảo sát 14 danh mục (9 B2B, 5 Matter) **[Đã kiểm]**

| Danh mục | Skin | Sản phẩm | Trang | Giới thiệu | FAQ | Chip | Facet (TH/GP) | Meta cắt "…" |
|---|---|---|---|---|---|---|---|---|
| `/cam-bien-chat-luong-khong-khi` | B2B | 12 | 1 | có | 0 | 5 | 5/3 | không |
| `/cam-bien-matter` | Matter | 12 | 1 | có | 5 | 0 | 4/1 | không |
| `/combo-san-pham-1` | B2B | 5 | 1 | không | 0 | 1 | 1/1 | không |
| `/cong-tac-matter` | Matter | 12 | 1 | có | 10 | 0 | 3/1 | có |
| `/cong-tac-thong-minh` | B2B | 82 | 4 | có | 0 | 6 | 23/9 | có |
| `/den-chieu-sang` | B2B | 8 | 1 | có | 0 | 2 | 2/3 | không |
| `/den-thong-minh-matter` | Matter | 2 | 1 | có | 5 | 0 | 1/1 | không |
| `/dich-vu-casambi-4c` | B2B | 0 | 1 | có | 0 | 12 | 0/0 | có |
| `/dieu-khien-chieu-sang` | B2B | 109 | 5 | có | 0 | 5 | 24/8 | có |
| `/dimmer-doc-lap-triac` | B2B | 29 | 2 | có | 5 | 4 | 13/7 | không |
| `/dong-co-rem-matter` | Matter | 2 | 1 | có | 5 | 0 | 2/1 | không |
| `/gateway-matter` | Matter | 3 | 1 | có | 4 | 0 | 2/1 | không |
| `/hang-thanh-ly` | B2B | 0 | 1 | có | 0 | 12 | 0/0 | không |
| `/phu-kien` | B2B | 9 | 1 | không | 0 | 2 | 7/3 | có |

Tổng hợp: có văn bản giới thiệu 12/14 (B2B 7/9, Matter 5/5); có FAQ 6/14 (B2B 1/9, Matter 5/5); có phân trang 3/14; rỗng 2/14; chip 9/9 B2B, 0/5 Matter. Chip là danh mục con nếu có (`/dieu-khien-chieu-sang` → 5 con), không thì các danh mục cùng cấp (`/dimmer-doc-lap-triac` → 4 anh em), danh mục cấp một rỗng → toàn bộ danh mục cấp một.

Khác nhau giữa hai skin **[Đã kiểm]**:

| Hạng mục | B2B | Matter |
|---|---|---|
| `#main-content` | có | không |
| Số sản phẩm | cạnh `h1` | thanh công cụ cuối trang |
| Card | có mã model (`span.model-code`), không nhãn tồn kho, giá `9.375.000 đ` | không mã, có nhãn tồn kho, giá `2.850.000đ`, ảnh `alt` rỗng |
| Facet | thương hiệu + giải pháp | thương hiệu + giải pháp (chỉ "Matter") |
| Breadcrumb gốc | `Trang chủ` → `/` | `Trang chủ` → `/matter` |

Nhãn tồn kho trên card Matter (31 card): `make_to_order` → "Đặt trước" (25), `low_stock` → "Sắp hết hàng" (5), `in_stock` → "Còn hàng" (1). Card B2B (82 card đã xem): payload có `make_to_order`, `backorder`, `out_of_stock`, `low_stock`, `in_stock` hoặc null, nhưng DOM không hiển thị nhãn nào.

### 2.4 URL biến thể đã thử **[Đã kiểm]**

| URL | Kết quả |
|---|---|
| `/{slug}?page=N` | trang N; canonical `?page=N`; title thêm " — trang N". Danh mục 82 sản phẩm: trang 1 24, trang 2 24, trang 3 24, trang 4 10 (tổng 82); 109 sản phẩm → 5 trang; 29 → 2 trang |
| `?page=5` (danh mục 4 trang), `?page=2` của danh mục Matter 12 sản phẩm | 404 ("Không tìm thấy trang") |
| `?page=0`, `?page=abc` | 200, nội dung trang 1, canonical gốc |
| `?sort=name-asc`, `newest`, `name-desc`, `price-desc` (và `price-asc` ở khảo sát trước) | 200, thứ tự đổi, `<option selected>` đúng giá trị, canonical gốc. Thứ tự mặc định giống `newest` ở `/dimmer-doc-lap-triac` |
| `?brand=vadsbo`, `?brand=jink` (Matter), `?solution=knx`, `?solution=giai-phap-khac`, `?solution=matter` | 200, số card bằng số đếm facet (7, 5, 5, 8, 12), canonical gốc |
| `?brand=vadsbo&sort=price-asc` | 200, 7 card sắp tăng |
| `?brand=khong-co` | 200, 0 card |
| `?view=list&page=2` | trang 2 dạng bảng; canonical `?page=2` |
| `?clearance=1` | 200, vẫn 24 card (tên tham số của bộ lọc "Chỉ hàng thanh lý" chưa xác định) |

HTML chỉ lộ bộ lọc dưới dạng `input` radio/checkbox không có `href`, nên liên kết lọc trong MD là phần thêm so với HTML (spec mục 6, điểm 1).

### 2.5 Phản hồi bất thường **[Đã kiểm]**

Lần tải đầu của `/cong-tac-thong-minh` trả HTML 411.844 B không có RSC payload (nhưng đủ 24 card trong DOM); 5 lần tải lại đều 562.981 B có payload. Trước đó 2/80 lần tải đầu trang sản phẩm cũng không có payload. Nguyên nhân chưa rõ. Code đọc dữ liệu server thì không bị ảnh hưởng; mọi cách đọc payload từ HTML cần có đường dự phòng.

---

## 3. Nguồn dữ liệu cho từng trường

| Trường trong MD | Nguồn | Ghi chú |
|---|---|---|
| `title`, H1 | `h1` | decode entity (`&amp;`), collapse whitespace |
| `description` | meta description (= `CollectionPage.description` = payload) | không cắt, không thêm "…" |
| `canonical` | `<link rel=canonical>` | tự trỏ; giữ `?page=N`, bỏ `brand`, `solution`, `sort`, `view` |
| `product_count` | số đứng ngay trước chữ "sản phẩm" | B2B cạnh `h1`; Matter trong thanh công cụ |
| `page`, `page_count` | `?page=N` trong canonical; số lớn nhất trong `nav[aria-label="Phân trang"]` | B2B: `ceil(product_count / 24)` |
| `query` | tham số `brand`, `solution`, `sort` của URL yêu cầu | |
| `image` | `og:image` | host `api.knxstore.vn` đổi thành `knxstore.vn` |
| Breadcrumb | `CollectionPage.breadcrumb.itemListElement` | khớp `nav` |
| Danh mục liên quan | prop `chips` hoặc các `a` giữa `h1` và khối đầu tiên | loại `a` nằm trong `nav` (breadcrumb Matter đứng sau `h1`); tên chip có thể bị chia thành nhiều `span` |
| Bộ lọc | props `brands[]`, `solutions[]` | liên kết `{base}?brand={slug}` |
| Sắp xếp | `select option` | giá trị khác rỗng |
| Dòng sản phẩm | object card, theo thứ tự `article` | xem lỗi #1 về giá |
| SKU | `card.sku` khi `article` có `span.model-code` | |
| Nhãn tồn kho | nhãn trong `article` (Matter) | |
| Giới thiệu danh mục | `div.user-html` (mô tả CMS) | |
| FAQ | `details` (`summary` + `div.user-html`) | bỏ `span[aria-hidden]` ("+") |

---

## 4. Ghi chú converter và kiểm tra

Văn bản giới thiệu và đáp án FAQ là HTML do CMS nhập; dùng cùng bộ chuyển đổi của trang sản phẩm (`rehype-remark` + `remark-gfm` **[Đề xuất]**). Điểm riêng đã gặp ở 14 danh mục **[Đã kiểm]**:

- Văn bản bắt đầu bằng `h2` của CMS, thường có `h3` lồng; sau khi hạ 1 cấp thành `###` và `####`. Không có `h1` trong 14/14.
- Bảng có `th` rỗng ở cột đầu (11 cột ở `/hang-thanh-ly`) và `strong` trong `th`: giữ `**…**`, ô rỗng giữ rỗng. Ô có `br` thành `<br>`.
- Danh sách `ol > li > (p, ul)` (`/dich-vu-casambi-4c`): mục có số, danh sách con thụt 3 khoảng trắng.
- Ảnh có `alt` (`matter protocol` ở `/cong-tac-matter`): giữ `alt`.
- Dùng `html.parser` với `convert_charrefs`; chuỗi `&amp;` trong H1 cần decode.

Kiểm tra cần có (parity HTML↔MD): `title` = H1; `product_count` = số trong DOM; số dòng sản phẩm = số `article` và đúng thứ tự slug; giá từng dòng = giá trong DOM; SKU chỉ khi card có mã; nhãn tồn kho khớp DOM; chip khớp DOM; tổng số đếm thương hiệu (cộng sản phẩm không thương hiệu) = `product_count`; số heading, số bảng và chuỗi từ của văn bản giới thiệu = DOM; số FAQ = DOM = `FAQPage`; `page_count` và `Trang sau` khớp thanh phân trang; trang ≥ 2 không có Bộ lọc/Giới thiệu/FAQ; không còn thẻ HTML, `&nbsp;`, chuỗi giao diện; sinh hai lần cho cùng byte.

Bộ chuyển đổi tham chiếu và bộ kiểm tra đã chạy trên cả 14 danh mục (trang 1), 4 trang của `/cong-tac-thong-minh` và 2 mẫu: tất cả kiểm tra áp dụng đều đạt (27 đến 32 kiểm tra mỗi trang). Script nằm ở thư mục tạm của phiên làm việc (`.../scratchpad/scripts/catmd2.py`, `catmd2_check.py`), chưa đưa vào dự án.

Số đo (bytes/4, chưa đo tokenizer): `/dimmer-doc-lap-triac` trang 1: HTML 535.241 B, MD 16.705 B (giảm 96,9%); trang 2: 396.398 B → 2.116 B (99,5%); `/cong-tac-matter`: 303.087 B → 17.352 B (94,3%); `/hang-thanh-ly`: 332.635 B → 6.109 B (98,2%). Phân bố trang 1 mẫu: frontmatter + đầu trang 644 B; Danh mục liên quan 269 B; Bộ lọc và sắp xếp 1.817 B; Sản phẩm 4.578 B; Phân trang 194 B; Giới thiệu 7.428 B; FAQ 1.775 B.

---

## 5. Lỗi dữ liệu làm MD sai nếu không xử lý

| # | Vấn đề | Hệ quả | Hướng xử lý | Người chốt |
|---|---|---|---|---|
| 1 | Card có `price: null` nhưng `priceRange` có giá (ví dụ Moorgen MRE675 trong `/cong-tac-thong-minh?page=2`: `priceRange` 7.800.000, DOM hiện `7.800.000 đ`; Casambi XPRESS-LR-B ở trang 3) **[Đã kiểm]** | Suy "Liên hệ" từ `price` null sẽ ghi sai giá | Giá lấy từ `priceRange.from` trước, rồi mới `price`; "Liên hệ" chỉ khi cả hai null | Dev |
| 2 | Có sản phẩm không có thương hiệu (1 trong 12 sản phẩm của `/cam-bien-chat-luong-khong-khi`) | Dòng thiếu thương hiệu; tổng facet thương hiệu nhỏ hơn số sản phẩm (11/12) | Bỏ thương hiệu khỏi dòng; không coi là lỗi | Tùng |
| 3 | Một sản phẩm thuộc 2 giải pháp: tổng số đếm giải pháp lớn hơn số sản phẩm ở 2/14 (`/cong-tac-thong-minh` 83/82, `/dieu-khien-chieu-sang` 110/109) | Cộng số đếm không ra tổng | Số đếm là số sản phẩm theo từng giá trị, không cộng | Dev |
| 4 | Card B2B có `sku` null (1 card trong `/cong-tac-thong-minh`, 1 trong `/dieu-khien-chieu-sang`) | Dòng thiếu SKU | Bỏ SKU khỏi dòng | Dev |
| 5 | Meta description bị cắt "…" ở nguồn (5/14: `/cong-tac-matter`, `/cong-tac-thong-minh`, `/dich-vu-casambi-4c`, `/dieu-khien-chieu-sang`, `/phu-kien`); payload, JSON-LD, meta cùng chuỗi cụt | `description` trong MD cụt | Sửa tại nguồn (lưu bản đầy đủ) | Tùng / Vũ |
| 6 | Văn bản `/cong-tac-matter` nói "14 model công tắc Matter", lưới 12 sản phẩm | MD chép nguyên văn nên có hai con số | Sửa nội dung CMS | Tùng |
| 7 | Danh mục rỗng hiển thị "Không có sản phẩm nào khớp bộ lọc" (sai nghĩa) cùng liên kết "Xoá bộ lọc" | Không (MD không in) | Sửa chữ hiển thị trên trang | Vũ |
| 8 | `datePublished` cùng ngày 2026-08-24 ở 14/14 danh mục (4/4 trang đọc đến mili-giây cùng `14:20:16.977Z`); `dateModified` khác ở 9/14 | MD không xuất ngày | Xác nhận `dateModified` có phải ngày sửa thật không | Vũ |
| 9 | Chip `Dimmer DMX` trỏ `/dmx` (khác kiểu slug của 3 chip còn lại) | Không | Giữ nguyên | Tùng |

---

## 6. Chưa xác định

| Điểm | Lý do |
|---|---|
| Kích thước trang của skin Matter | 5/5 danh mục Matter có ≤ 12 sản phẩm, chưa thấy phân trang; `?page=2` trả 404 |
| Kết hợp `page` với `brand`/`solution` | Chưa thử |
| Nhiều giá trị `brand` trong một URL | Chưa thử |
| Nhãn tồn kho của `out_of_stock`, `backorder` ở card Matter | Chưa thấy mẫu |
| Tên tham số của bộ lọc "Chỉ hàng thanh lý" | `?clearance=1` không đổi kết quả |
| `/san-pham-matter` (danh mục cha của danh mục Matter) | Trả 200, chưa đọc cấu trúc; chưa xác nhận cùng template danh mục |
| Công thức `<title>` khi không có `seoTitle` | 11/14 `<title>` riêng; 2/14 dạng `{tên} \| knxstore.vn`; 1/14 chỉ là tên |
| Tên hàm/model phía server của danh mục | Chỉ quan sát từ phía client |
| Hành vi sau hydrate (vị trí toolbar, drawer lọc) | Chưa chạy trình duyệt headless |
