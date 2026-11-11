# Markdown cho AI agent — knxstore.vn

Bộ tài liệu đặc tả để knxstore.vn (Next.js App Router) **trả Markdown sạch cho AI agent** thay vì HTML 400–600 KB. Repo này chỉ có tài liệu: hiện trạng trang web, đặc tả kết quả mong muốn theo từng loại trang, và ghi chú triển khai. Chưa có code chạy thật trong repo.

---

## 1. Kết quả mong muốn

Khi agent yêu cầu một URL của site, nhận về **một file Markdown chỉ chứa nội dung của đúng URL đó**:

| Yêu cầu | Kết quả mong muốn | Hiện trạng đo 2026-11-11 |
|---|---|---|
| `GET {url}` với `Accept: text/markdown` | `200`, `Content-Type: text/markdown; charset=utf-8`, `Vary: Accept`, body là MD theo spec của loại trang | trả HTML như thường, `vary` không có `Accept` |
| `GET {url}.md` (cho client không đặt được header) | cùng bản MD, kèm `canonical` và `noindex` | `404` |
| URL không tồn tại | `404` kèm body MD (không trả HTML) | — |
| HTML cho trình duyệt | **không đổi** (trừ `Vary`/`Link`) | — |

Bốn điều kiện chất lượng, chung cho mọi loại trang:

1. **Cùng thông tin với HTML, khác định dạng.** Không tóm tắt, không thêm thông tin không có trên trang (parity). Ngoại lệ có chủ đích được ghi rõ trong từng spec (ví dụ liên kết lọc `?solution=`).
2. **Không có chrome:** không header (~165 liên kết menu), footer, giỏ báo giá, nút Zalo nổi, nút thao tác, JSON-LD.
3. **Gọn:** MD nhỏ hơn HTML khoảng 92–99% (từ 1 KB đến 42 KB tùy loại trang, ước tính token = bytes/4).
4. **Đủ để agent đi tiếp:** mọi liên kết tuyệt đối `https://knxstore.vn/...`, danh sách có URL sản phẩm/danh mục/thương hiệu, phân trang có liên kết trang trước/sau.

Kỳ vọng đặt đúng: tối ưu trải nghiệm agent và chi phí token, **không** hứa tăng thứ hạng hay lượt trích dẫn (xem `markdown-for-agents-nextjs-direction.md` mục 3 và 3A).

### Ví dụ một bản MD hoàn chỉnh

Trang `/brands/protopixel` (MD 1,5 KB):

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

---

## 2. Cấu trúc thư mục

```
.
├── README.md                                  file này
├── markdown-for-agents-nextjs-direction.md    CƠ CHẾ: phục vụ MD thế nào (negotiation, .md, proxy.ts, cache, discovery, nghiệm thu, lộ trình)
├── site-structure/                            HIỆN TRẠNG: cấu trúc HTML từng loại trang (chỉ mô tả, không đề xuất)
│   ├── 00-index.md                            sitemap, bảng loại trang × chế độ render × container
│   └── 01…15-*.md                             mỗi file một loại trang (response, <head>, cây nội dung, payload RSC, JSON-LD, số đo)
├── product-md-spec/                           ĐẶC TẢ sản phẩm  /products/{slug}
├── product-category-md-spec/                  ĐẶC TẢ danh mục  /{slug}
├── blog-md-spec/                              ĐẶC TẢ bài blog  /{slug}.html
├── blog-topic-md-spec/                        ĐẶC TẢ chuyên mục blog  /blogs/{topic}
├── brand-md-spec/                             ĐẶC TẢ thương hiệu  /brands/{slug}
└── solution-md-spec/                          ĐẶC TẢ giải pháp  /solution/{slug}
```

Ba lớp tài liệu, đọc theo thứ tự này:

| Lớp | Trả lời câu hỏi | Nơi |
|---|---|---|
| Cơ chế | Phục vụ MD qua HTTP thế nào, cache và discovery ra sao | `markdown-for-agents-nextjs-direction.md` (viết trước khi khảo sát site; khi khác chi tiết riêng của site thì theo spec) |
| Hiện trạng | Mỗi loại trang đang render thế nào, dữ liệu nằm ở đâu | `site-structure/` (1 URL mẫu mỗi loại; số liệu rộng hơn nằm trong dev notes của từng spec) |
| Đặc tả | MD của loại trang đó phải trông thế nào | `*-md-spec/01-*.md` |

Thư mục `*-md-spec/` có tối đa 4 loại file:

| File | Vai trò | Có ở |
|---|---|---|
| `01-*-md-spec.md` | **Kết quả mong muốn**: frontmatter, thứ tự khối, quy tắc, ví dụ, quyết định còn mở. Dev đọc file này để biết phải sinh ra gì | cả 6 thư mục |
| `02-*-dev-notes.md` | Cách triển khai: dữ liệu nằm ở đâu trong payload, khảo sát nhiều URL, lỗi dữ liệu và người xử lý | `product-md-spec/`, `product-category-md-spec/` |
| `sample-*.md` | Đầu ra mẫu thật, không chỉnh tay | `product-md-spec/` (Matter, B2B), `product-category-md-spec/` (Triac, Matter) |
| (mục 7 trong file 01) | Ghi chú dev ngắn thay cho file 02 ở các spec nhỏ | `blog-`, `blog-topic-`, `brand-`, `solution-md-spec/` |

---

## 3. Loại trang → spec

| Loại trang | URL | Số URL | Spec | Nguồn dữ liệu để dựng MD | Đã kiểm chứng trên |
|---|---|---|---|---|---|
| Sản phẩm (cả skin B2B và Matter) | `/products/{slug}` | 752 | [`product-md-spec/`](product-md-spec/01-product-md-spec.md) | **Adapter đọc payload RSC**: thông số, FAQ, tài liệu PDF, biến thể chỉ có trong payload và JSON-LD, không có trong DOM cho đến khi bấm tab. HTML→MD chỉ dùng cho mô tả CMS | 2 mẫu hoàn chỉnh (B2B, Matter) + khảo sát dữ liệu 80 URL (69 B2B, 11 Matter) |
| Danh mục (B2B và Matter) | `/{slug}` | 91 | [`product-category-md-spec/`](product-category-md-spec/01-product-category-md-spec.md) | DOM + object thẻ sản phẩm trong payload (giá, SKU, tồn kho) | 14 danh mục (9 B2B, 5 Matter) và các trang `?page=N` |
| Bài blog | `/{slug}.html` | 227 | [`blog-md-spec/`](blog-md-spec/01-blog-md-spec.md) | DOM (thân bài CMS) + payload (khối sản phẩm trong bài) | 30 bài |
| Chuyên mục blog | `/blogs/{topic}` | 9 | [`blog-topic-md-spec/`](blog-topic-md-spec/01-blog-topic-md-spec.md) | DOM | 9/9 |
| Thương hiệu | `/brands/{slug}` | 71 | [`brand-md-spec/`](brand-md-spec/01-brand-md-spec.md) | DOM + payload (thẻ sản phẩm, bộ lọc) | 71/71 và mọi trang `?page=N` của 9 thương hiệu có phân trang |
| Giải pháp | `/solution/{slug}` | 12 | [`solution-md-spec/`](solution-md-spec/01-solution-md-spec.md) | DOM + payload (giá 8 thẻ nổi bật) | 12/12 |

Phân loại theo chế độ render: sản phẩm là ISR (`s-maxage=3600`); danh mục, blog, chuyên mục, thương hiệu, giải pháp là render động theo request (`cache-control: no-store`). Chi tiết ở `site-structure/00-index.md`.

### Trang không có spec riêng

Các trang này **không** có spec: nội dung nằm đủ trong HTML ban đầu và là trang điều hướng (phần thông tin riêng chỉ là H1, một đoạn mô tả và liên kết, còn lại là danh sách đã có spec ở trang con), nên để pipeline HTML→MD chung (baseline ở mục 5 của file hướng đi) xử lý.

| Trang | Lý do |
|---|---|
| `/` (trang chủ), `/matter` (landing Matter), `/matter/categories` | Trang điều hướng; mọi khối có trong HTML ban đầu |
| `/products`, `/brands`, `/blogs`, `/solution` (trang gốc) | Danh sách điều hướng; `/blogs` khác cấu trúc `/blogs/{topic}` (xem spec chuyên mục) |
| `/author/{slug}` (2 URL) | Quyết định bỏ qua |
| `/matter-search`, `/search?` | Trạng thái client; `robots.txt` đã `Disallow: /search?` |
| `/contact`, trang chính sách, `/page/about-us`, `/tool*`, `/tools/*`, `/visual*` | Trang tĩnh hoặc công cụ, không thuộc phạm vi đợt này |

Baseline phải nhận cả hai kiểu vùng nội dung chính: `div#main-content` (hầu hết trang B2B, không có `<main>`) và `<main>` (trang khu Matter).

---

## 4. Cách đọc một spec

Mọi file `01-*-md-spec.md` có cùng khung:

| Mục | Nội dung |
|---|---|
| 1. Nguyên tắc | Quy tắc áp cho cả loại trang, kèm kích thước MD đo được |
| 2. Frontmatter | Bảng key, giá trị, điều kiện xuất hiện, ưu tiên P1–P3 (P1 = agent không thể thiếu, P3 = có thì tốt) |
| 3. Body | Thứ tự khối và heading cố định, quy tắc từng khối, URL biến thể (`?page`, `?sort`…), trường hợp đặc biệt |
| 4. Không đưa vào MD | Khối bị loại và lý do |
| 5. Ví dụ hoàn chỉnh | Đầu ra thật của bộ chuyển đổi tham chiếu (có trích rút gọn thì ghi rõ) |
| 6. Quyết định còn mở | Mỗi điểm có mặc định đang dùng và người chốt |
| 7. Ghi chú cho dev | Cấu trúc DOM/payload cần bám, tham số đã kiểm, dữ liệu lệch, cách kiểm tra |

Trong dev notes, nhãn **[Đã kiểm]** là đã đo thật; **[Suy ra]** là suy từ HTML hoặc thiết kế, chưa chạy.

---

## 5. Quy ước chung của mọi spec

- **Frontmatter** YAML: key tiếng Anh, `type` theo loại trang (`product`, `category`, `article`, `blog-topic`, `brand`, `solution`), có `title`, `canonical`, `lang: vi`. `title` lấy từ H1 hiển thị, không lấy `<title>`.
- **Body** tiếng Việt: một H1; heading cố định (`## Sản phẩm`, `## FAQ`, `## Giới thiệu thương hiệu`…); danh sách sản phẩm một dòng mỗi sản phẩm: `- [tên](url) — thương hiệu · SKU · giá · nhãn tồn kho`; giá dạng `3.190.000 đ`, không giá `Liên hệ`; FAQ dạng `- **Câu hỏi** Trả lời`.
- **Nội dung CMS** (mô tả, bài viết, giới thiệu): heading hạ xuống dưới heading cố định, không nhảy quá 1 cấp; bảng GFM; ảnh `![alt](URL tuyệt đối)`; `<br>` thành ngắt dòng cứng; bỏ thẻ bao, `class`, `style`.
- **Làm sạch ký tự**: NBSP thành dấu cách thường; bỏ ký tự vô hình (U+200B…U+200F, U+202A…U+202E, U+2060, U+FEFF); escape `*`, `[`, `]`, `` ` `` trong chữ lấy từ dữ liệu.
- **Khối rỗng thì bỏ**, không ghi "Không có" hay "N/A".
- **Phân trang**: mỗi URL một bản MD, trang 2 trở đi không lặp bộ lọc và văn bản giới thiệu; có mục `## Phân trang` với liên kết trước/sau.
- **Lỗi nguồn**: lần tải đầu đôi khi trả HTML thiếu nội dung (sản phẩm 2/80, danh mục 1/14, blog 1/30, thương hiệu 4/71, chuyên mục 1 lần; giải pháp 0/12). Bộ chuyển đổi phải **báo lỗi khi thiếu H1 hoặc khối chính**, không xuất MD rỗng.
- **Tất định**: cùng HTML vào, cùng byte ra.

---

## 6. Cách dùng cho dev

1. Đọc `markdown-for-agents-nextjs-direction.md` mục 1, 4, 5, 7, 8 để nắm cơ chế (negotiation, `.md`, `proxy.ts`, `Vary: Accept`, cache, lỗi converter).
2. Với mỗi loại trang cần làm: đọc `site-structure/NN-*.md` (hiện trạng) rồi `*-md-spec/01-*.md` (đầu ra), cuối cùng mục 7 hoặc `02-*-dev-notes.md` (nguồn dữ liệu, lỗi cần tránh).
3. Dựng bộ chuyển đổi theo spec, rồi đối chiếu với HTML bằng các phép kiểm tra đã liệt kê ở mục 7 của từng spec (frontmatter khớp `canonical`/H1/meta, số dòng và thứ tự URL sản phẩm, giá, số heading/bảng/ảnh/liên kết, chuỗi từ của nội dung CMS, không còn chrome/HTML/NBSP, tất định).
4. Tiêu chí nghiệm thu tổng thể ở mục 9 của file hướng đi (HTML không đổi, parity bảng/số liệu/heading, không rò dữ liệu người dùng, giảm token, discovery, log lưu lượng agent).

Với trang DOM-based, bám theo chữ của `h2` và cấu trúc ngữ nghĩa, không bám class Tailwind.

---

## 7. Mức kiểm chứng và giới hạn

- Mọi spec đang ở trạng thái **bản nháp**, ngày 2026-11-11.
- `site-structure/` do sub-agent khảo sát **1 URL mẫu mỗi loại**; số liệu rộng hơn (nhiều URL) nằm trong dev notes và mục 7 của từng spec.
- Mỗi spec được kiểm bằng **bộ chuyển đổi tham chiếu và script kiểm tra chéo** (Python) chạy trên HTML thật, trừ sản phẩm (kiểm trên 2 mẫu hoàn chỉnh và khảo sát dữ liệu 80 URL; chưa chạy bộ chuyển đổi trên cả 11 trang Matter). Các script này nằm ở thư mục tạm của phiên làm việc, **chưa đưa vào repo**, nên đây là bằng chứng spec khớp HTML, không phải code để triển khai.
- Chưa chạy trình duyệt: mọi mô tả hành vi sau hydrate (tab, carousel, bộ lọc) là suy ra từ HTML, payload và bundle.
- Chưa xác định: điều kiện chọn skin Matter hay B2B cho `/products/{slug}`; tiêu chí chọn "sản phẩm nổi bật", thứ tự thẻ blog.

---

## 8. Điểm còn mở dùng chung nhiều spec

Mỗi spec có bảng đầy đủ ở mục 6; dưới đây là các điểm lặp lại ở nhiều loại trang.

| Điểm | Mặc định hiện tại | Người chốt |
|---|---|---|
| Liên kết lọc/sắp xếp (`?category=`, `?brand=`, `?solution=`, `sort=`) đưa vào MD dù HTML không có (danh mục, thương hiệu, giải pháp) | Có; tham số đã kiểm trên server | Tùng |
| Hotline Matter: `0378628837` (nút nổi, footer, landing) hay `0918918755` (nút "Liên hệ tư vấn" trong khung mua sản phẩm Matter) | MD sản phẩm Matter dùng số trong khung mua | Huy |
| `stockStatus` của sản phẩm B2B có trong dữ liệu nhưng trang không hiển thị | Không đưa vào MD (giữ parity với HTML) | Tùng / Huy |
| `description` bị cắt "…" hoặc là câu mặc định của site (thương hiệu 65/71, giải pháp 7/12 + 1 mặc định, blog 5/30) | Giữ nguyên chuỗi nguồn; sửa ở nguồn nếu muốn đầy đủ | Tùng / Vũ |
| Ngày cập nhật (`modified`): chỉ có trong JSON-LD, nhiều trang cùng một mốc nhập hàng loạt | Chỉ blog xuất `modified` khi khác `published`; các loại khác không xuất | Tùng |
| `canonical` của trang phân trang: danh mục tự trỏ `?page=N`, thương hiệu trỏ về URL gốc | Giữ đúng như HTML; `page`, `page_count` phân biệt | Tùng |
| Ảnh trong nội dung CMS (tốn token cho URL) | Giữ `![alt](URL)` | Tùng |
| Lỗi hiển thị ở nguồn: H1 "Giải pháp Giải pháp khác"; blog một số `alt` ảnh tự sinh; thương hiệu có `h1` trong văn bản | Giữ nguyên như HTML; sửa ở nguồn | Vũ |
