# Trang sản phẩm → Markdown — ghi chú cho dev

> Ngày: 2026-11-11 · Đi kèm `01-product-md-spec.md` (kết quả mong muốn) và 2 mẫu đầu ra `sample-lb-l02e.md` (Matter), `sample-weinzierl-886.md` (B2B).
> File này gồm: vì sao cần đọc dữ liệu thay vì convert HTML, dữ liệu nằm ở đâu, kết quả khảo sát, lỗi dữ liệu và việc còn thiếu.
> Nguồn: `site-structure/02-product.md`, `markdown-for-agents-nextjs-direction.md`, và các lần kiểm tra bằng `curl` ngày 2026-11-11 (UA `Mozilla/5.0`, không chạy JS).
> Nhãn: **[Đã kiểm]** = đã đo trong phiên làm việc; **[Suy ra]** = suy ra từ bundle hoặc ý tưởng thiết kế, chưa chạy thật.

---

## 1. Kết luận kỹ thuật

Trang sản phẩm **không thể chỉ dùng HTML→MD**. Thông số kỹ thuật, FAQ và link tài liệu PDF không có trong HTML ban đầu (mục 2). Converter đọc HTML sẽ mất đúng phần giá trị nhất với kỹ sư.

→ Cần **adapter đọc cùng object `product` mà trang dùng**: dựng frontmatter, bảng, danh sách từ dữ liệu; chỉ dùng converter HTML→MD cho riêng `descriptionHtml`. Phần còn lại của site dùng pipeline HTML→MD làm baseline (hybrid, mục 5 file hướng đi).

---

## 2. Dữ liệu nằm ở đâu và kết quả kiểm tra

### 2.1 Payload là gì, đọc thế nào

Next.js App Router nhúng dữ liệu trang vào HTML qua các thẻ `<script>self.__next_f.push([1,"…"])</script>` (RSC payload), để client hydrate. Component tab nhận props `{product, description, guide, variant}`. Object `product` chứa đủ `specs`, `faqs`, `downloads`, `descriptionHtml`, nhưng component chỉ render panel của tab đang chọn **[Suy ra từ bundle]**. JSON-LD cũng dựng từ cùng object. Vì vậy dữ liệu của mọi tab có trong payload, nhưng không có trong văn bản HTML ban đầu.

Cách đọc đã dùng khi khảo sát **[Đã kiểm]**:
1. Lấy các chuỗi trong `self.__next_f.push([1,"…"])`, giải mã chuỗi JSON, nối lại.
2. Tìm `"product":{`, cắt object bằng đếm ngoặc (có xử lý chuỗi và dấu `\`).
3. Một số trường là **tham chiếu**: `"descriptionHtml":"$48"` trỏ tới khối văn bản `48:T<độ dài hex>,…` (độ dài tính bằng byte UTF-8); `variants`, `images` ở một số trang là `"$6:1:…"`, cần tìm mảng thật (ví dụ theo `url` ảnh chính).

### 2.2 Kết quả đo trang Matter LB-L02E **[Đã kiểm]**

| Kiểm tra | Kết quả |
|---|---|
| Tab trong HTML | 3 tab (Chi tiết sản phẩm / Thông số kỹ thuật / FAQ); không có `role=tabpanel` |
| `<dl>`, `<dt>` (nơi specs sẽ render) | 0 |
| 5 câu hỏi FAQ trong văn bản hiển thị | 0/5; có đủ trong payload và JSON-LD |
| Nhãn spec riêng biệt (Đui bóng, Nguồn cấp, Góc chiếu, Công suất chờ, Kích thước) | không có trong văn bản hiển thị; có trong payload và JSON-LD |
| Payload `specs` | 14 dòng `{code, name, unit, rawName, valueText, parseStatus, position, priority}` |
| Payload `faqs` | 5 dòng `{question, answer, position}` |

Mẫu B2B SR-2422NK8-DA2: `"specs":[]`, `"faqs":[]`, `"downloads":[]`; chỉ 1 tab "Mô tả chi tiết".

### 2.3 Dòng `Model` có đơn vị sai ngay ở dữ liệu gốc **[Đã kiểm]**

Payload: `{"code":"model","name":"Model","unit":"MTAC01XXX","valueText":"LB-L02E","parseStatus":"failed",…}`. JSON-LD chép lại thành `unitText`. Quy tắc nối đơn vị: chỉ nối `unit` khi `parseStatus === "exact"` và `valueText` chưa chứa unit (cùng rule với DOM theo bundle). Ví dụ `Nguồn cấp`: `parseStatus:"partial"`, `unit:"V"`, `valueText:"100–240V AC, 50/60 Hz"` thì không nối. Không lấy specs từ JSON-LD `additionalProperty`.

### 2.4 Khảo sát 80 URL ngẫu nhiên **[Đã kiểm]**

Danh sách URL: `https://knxstore.vn/sitemap.xml` → `sitemaps/products.xml` (752 URL; `/products.xml` trả 404). Mẫu 80 URL (10,6%), seed cố định, mỗi URL tải 1 lần.

| Chỉ số | Kết quả (n = 80) |
|---|---|
| Skin B2B / Matter | 69 / 11 |
| Có `downloads` | 42 trang (52,5%), tổng 52 file, **tất cả là PDF** |
| Số file mỗi trang | 1 file: 37 trang; 2 file: 3; 4 file: 1; 5 file: 1 |
| Có `specs` | 34 (42,5%) |
| Có `faqs` | 34 (42,5%) |
| `price` khác null | 70 (87,5%) |
| `mpn` khác null | 0 |
| `guideHtml` khác null | 0 |

- **`downloads[]`**: `{"url":"/media/docs/{sha256}.pdf","filename":"…","label":null,"sizeBytes":461255,"position":0}`. `label` khác null ở 1/52 file. Dung lượng nhỏ nhất 28.782 B, trung vị 785.829 B, lớn nhất 17.196.625 B. Kiểm 1 file: `https://knxstore.vn/media/docs/…pdf` và `https://api.knxstore.vn/media/docs/…pdf` đều trả 200, `application/pdf`, `content-length` bằng `sizeBytes`.
- **Link PDF trong HTML ban đầu**: 1/42 trang (link nằm trong mô tả CMS); 41/42 trang chỉ có trong payload (hiện khi bấm tab "Tài liệu" ở B2B, "Datasheet" ở Matter). Tab thấy trong mẫu: B2B gồm Mô tả chi tiết / Thông số kỹ thuật / Tài liệu / Câu hỏi thường gặp (tab nào có tùy dữ liệu); Matter gồm Chi tiết sản phẩm / Thông số kỹ thuật / FAQ / Datasheet.
- **Ảnh thư viện** `images[]` (`url`, `alt`, `caption`, `width`, `height`, `isPrimary`, `position`): trên 81 trang (80 mẫu + LB-L02E): 21 trang 1 ảnh, 10 trang 2 ảnh, 12 trang 3 ảnh, 9 trang 4 ảnh, còn lại 5–21 ảnh; trung vị 3. `alt` rỗng 1/363 ảnh.
- **Phản hồi bất thường**: 2/80 lần tải đầu (`cool-automation-bo-dieu-khien-tich-hop-dieu-hoa-trung-tam`, `legrand-cong-knx-ip-quan-ly-he-thong-knx-003543`) trả HTML 133.210 B và 252.799 B không có RSC payload; tải lại ngay trả bình thường (461.695 B và 443.037 B). Nguyên nhân chưa rõ. Adapter cần đường fallback khi thiếu `product`.
- **Kích thước HTML lệch lớn**: thường 250–500 KB; trang `bo-chuyen-doi-giao-thuc-dieu-khien-dali-dmx512-64-dia-chi-dalcnet-dgd-gateway` là 2.740.298 B (~2,6 MiB), dưới giới hạn 6 MiB tham chiếu của Cloudflare nhưng đáng đo khi tính thời gian convert.

Nếu sau này thêm lại dòng liên kết mục lục vào MD: `/llms.txt` (200, `text/plain`, 7.354 B), `/products`, `/brands`, `/solution` đều trả 200; trang HTML sản phẩm hiện không trỏ tới `llms.txt`.

---

## 3. Nguồn dữ liệu cho từng trường

Theo `02-product.md` và khảo sát trên (props trong payload RSC; bundle `/_next/static/chunks/16nkacgg7g1nb.js`).

| Trường trong MD | Nguồn | Ghi chú |
|---|---|---|
| `title`, H1 | `h1` (4 `span`), lấy textContent | Matter: H1 khác `<title>` |
| `description` | `seoDescription` | = meta description |
| `canonical` | canonical trong `<head>` | tuyệt đối |
| `brand`, `sku`, `mpn` | `brand.name`, `sku`, `mpn` | `mpn` null ở cả 80 trang mẫu |
| `price`, `currency` | `price`, giá cũ `oldPrice` | VND |
| Danh mục, giải pháp | `categories[]`, `solutions[]` | Breadcrumb chỉ hiện 1 danh mục; Matter có `solutions` trong dữ liệu nhưng trang không hiển thị |
| Mô tả | `descriptionHtml` | HTML do CMS nhập |
| Thông số | `specs[]` | xem 2.3 |
| Tài liệu | `downloads[]` | ghép `url` với `https://knxstore.vn` (mục 2.4) |
| FAQ | `faqs[]` | |
| Hướng dẫn | `guideHtml` | chỉ Matter; 0/80 trang có |
| Biến thể | `variants[]` (`name`, `isDefault`, `price`, `options[]`) | |
| Xuất xứ, bảo hành | `countryOfOrigin`, `warrantyMonths` | |
| Giao thức, Hoạt động cùng | `b2cContent.thuocTinh` | chỉ Matter; trường `Cần Hub?` có trong dữ liệu nhưng trang không hiển thị |
| Ảnh chính, Hình ảnh | `images[]`; `primaryImage` | URL gốc `/media/…` |
| `published` | `publishedAt` | |
| Sản phẩm đề xuất | card đề xuất trong DOM (B2B 16 card), `related` | lấy SKU/giá từ sản phẩm đích |
| Bài viết liên quan | `posts[]` trong payload (Matter) | `authorName`/`author`, `publishedAt` |
| Hotline, Zalo | khung mua của trang | xem lỗi #5 |

---

## 4. Ghi chú converter và kiểm tra

Converter cho `descriptionHtml`. Gợi ý `rehype-remark` + `remark-gfm` (file hướng đi mục 5.2) **[Đề xuất, chưa thử]**:

- Chạy `rehype-minify-whitespace` trước khi convert; giữ khoảng trắng quanh `span.whitespace-nowrap` (token model trong H1).
- Bảng: GFM; bảng có `rowspan`/`colspan` thì giữ `<table>`. Bảng mẫu Matter (14 hàng × 3 cột, `thead > tr > th[scope=col]`, ô chứa `Ra>90` không escape) phải đủ số hàng/cột sau convert.
- Breadcrumb là `span`, không phải `ol/li`; textContent nối liền `Trang chủ/Công tắc thông minh/…`. Cần rule riêng.
- `img` trong mô tả không có `width/height`; dùng URL `/media/…` gốc, không dùng `_next/image`.
- `turndown` core không render bảng và không strip `script/style` (file hướng đi mục 5.2).

Kiểm tra cần có (parity HTML↔MD): số hàng/cột bảng; giá trị + đơn vị số liệu; số heading (tính theo cấp đã hạ 1 bậc); số link nội dung; số dòng thông số, FAQ, tài liệu bằng với dữ liệu. Dùng `sample-lb-l02e.md` và `sample-weinzierl-886.md` làm đầu ra đối chiếu.

---

## 5. Lỗi dữ liệu làm MD sai nếu không xử lý

| # | Vấn đề | Hệ quả | Hướng xử lý | Người chốt |
|---|---|---|---|---|
| 1 | Hai variant "Xám" và "Trắng" cùng SKU `SR-2422NK8-DA2` | Agent không phân biệt được mã đặt hàng | Cần SKU riêng theo variant. Trước mắt MD ghi đúng dữ liệu (spec 3.5) | Tùng / Vũ |
| 2 | Card đề xuất thứ 10 trên trang SR-2422NK8-DA2 hiển thị mã `SR-2422NK8-DA2` nhưng trỏ slug `…sr-2422nkx-da2` | MD liệt kê sai SKU | Lấy SKU từ dữ liệu sản phẩm đích (đã kiểm 8/8 card của trang Weinzierl: khớp) | Dev |
| 3 | `stockStatus` có trong payload nhưng chỉ skin Matter hiển thị: Matter `make_to_order` ↔ "Đặt trước" trên HTML **[Đã kiểm]**; B2B `out_of_stock` không hiện trong DOM | Đưa trường B2B vào MD phá parity HTML↔MD (rủi ro cloaking, file hướng đi mục 3) | Chỉ ánh xạ khi HTML đang hiển thị; B2B không đưa cho đến khi xác nhận dữ liệu đúng và hiển thị | Tùng / Huy |
| 4 | `dateModified` = `datePublished` = `publishedAt` | Chưa có ngày cập nhật thật | Không xuất `lastmod` nếu API không có `updatedAt`; cần xác nhận API có trường này không | Vũ |
| 5 | Hotline khác giữa hai skin: B2B `0918918755`; Matter `0378628837` (nút nổi + footer), riêng nút "Liên hệ tư vấn" trong khung mua Matter vẫn `0918918755`. MD Matter hiện dùng số trong khung mua | MD có thể ghi sai số | Chốt chủ ý hay lỗi | Huy |
| 6 | Mẫu B2B SR-2422NK8-DA2 `specs: []`, thông số thật nằm trong `ul` của mô tả. Đo trên 80 URL: chỉ 42,5% có `specs` | Mục Thông số vắng ở ~57% sản phẩm | Không bịa bảng thông số từ mô tả; cân nhắc nhập `specs` cho sản phẩm B2B | Tùng |
| 7 | Dòng `Model` có `unit` sai (mục 2.3) | Giá trị sai nếu nối unit vô điều kiện | Rule `parseStatus === "exact"` | Dev |
| 8 | JSON-LD mẫu B2B: `isPartOf` trỏ `#website` nhưng không có node `WebSite`; không có `Product` | Không dùng JSON-LD làm nguồn dữ liệu | Dựng từ object `product` | Dev |
| 9 | Hai sản phẩm đề xuất cùng tên "Gateway điều khiển điều hòa VRF đa thương hiệu tích hợp BMS qua RS485 Micro Air", slug khác nhau (`…rs485` SKU `MicroAir B25`; `…rs485-micro-air` SKU `B25`), cùng xuất hiện trong danh sách đề xuất của trang Weinzierl 886 **[Đã kiểm]** | Danh sách đề xuất có 2 dòng gần trùng nhau | Xác nhận có phải sản phẩm trùng trong catalog | Tùng / Vũ |

---

## 6. Chưa xác định

| Điểm | Lý do |
|---|---|
| Tỉ lệ `bundles`, `warrantyMonths`, nhiều variant trên 752 URL | Khảo sát 80 URL mới đo `downloads`, `specs`, `faqs`, `mpn`, `price`, `guideHtml`, ảnh |
| Định dạng tài liệu ngoài PDF (CAD, zip) | 52/52 file trong mẫu đều là PDF |
| Nguyên nhân phản hồi không có payload (2/80 lần tải đầu) | Chưa tái hiện được |
| Điều kiện chọn skin B2B/Matter | Chỉ có 2 mẫu đầu; chưa xác nhận hàm `segmentOf` quyết định skin |
| Điều kiện có node `Product` + `Offer` trong JSON-LD | Nhiều biến đổi cùng lúc giữa 2 mẫu |
| Hành vi sau hydrate | Chưa chạy trình duyệt headless |
