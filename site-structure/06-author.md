# Trang tác giả — cấu trúc hiện tại

> https://knxstore.vn/author/le-vo-quoc-huy · nguồn sitemap: posts.xml · 2 URL cùng loại (`/author/{slug}`) · khảo sát 2026-11-11

Phạm vi: chỉ mô tả hiện trạng, đo trên HTML thô lấy bằng `curl -A "Mozilla/5.0"` (không chạy trình duyệt). Chỉ khảo sát `le-vo-quoc-huy`; URL còn lại của loại này (`/author/ta-minh-vu`, theo link trong trang bài) không được fetch.

---

## 1. Response & rendering

### 1.1 Header response (GET, không `Accept-Encoding`)

| Header | Giá trị |
|---|---|
| status | `200` (HTTP/2) |
| content-type | `text/html; charset=utf-8` |
| cache-control | `private, no-cache, no-store, max-age=0, must-revalidate` |
| x-nextjs-cache | Không có |
| x-nextjs-prerender | Không có |
| x-nextjs-stale-time | Không có |
| vary | `rsc, next-router-state-tree, next-router-prefetch, next-router-segment-prefetch, Accept-Encoding` (không có `Accept`) |
| cf-cache-status | `DYNAMIC` |
| link | 4 preload font `woff2` (`051742360c26797e-s.p.*`, `2bbe8d26…`, `83afe278…`, `fa39153a…`); khác 2 trang còn lại, không có 2 preload style trong header (2 stylesheet vẫn có trong `<head>`) |
| x-robots-tag | Không có |
| content-encoding | Không có khi không gửi `Accept-Encoding`; `br` khi gửi `gzip, br`; `gzip` khi chỉ gửi `gzip` |
| Khác | `server: cloudflare`, `x-frame-options: SAMEORIGIN`, `x-content-type-options: nosniff`, `strict-transport-security`, `referrer-policy: same-origin`, `speculation-rules: "/cdn-cgi/speculation"`, `alt-svc: h3`. Không có `etag`, `last-modified`, `age` |

### 1.2 Chế độ render (suy ra)

Kết luận: **dynamic render theo từng request (SSR), không cache ở Cloudflare**. Bằng chứng: `cache-control: private, no-cache, no-store, max-age=0, must-revalidate`; không có `x-nextjs-cache`/`x-nextjs-prerender`/`x-nextjs-stale-time`/`s-maxage`; `cf-cache-status: DYNAMIC`; RSC dòng `0:` có `"S":false` (ý nghĩa cờ chưa xác nhận bằng tài liệu).

- Route (từ RSC): `(site)/(b2b)/author/[slug]`, `slug` = `le-vo-quoc-huy`.
- Build ID `rJVjL74JjgHtrkQQTzrjV`. Phiên bản Next.js: Chưa xác định.
- Hồ sơ và toàn bộ 82 item bài viết là Server Component (RSC liệt kê 82 tham chiếu `$L17…$L68` trong `ul`), có đủ trong HTML ban đầu.

### 1.3 Hành vi hiện tại với yêu cầu Markdown (chỉ ghi kết quả)

| Request | Status | content-type | Kích thước body |
|---|---|---|---|
| `GET /author/le-vo-quoc-huy` + `Accept: text/markdown` | 200 | `text/html; charset=utf-8` | 423,422 B (bằng bản thường) |
| `GET /author/le-vo-quoc-huy.md` | 404 | `text/html; charset=utf-8` | 131,212 B |

Response 404 vẫn mang `cache-control: private, no-cache, no-store…`.

---

## 2. `<head>`

| Thành phần | Giá trị |
|---|---|
| `<title>` | `Lê Võ Quốc Huy — Sales Engineer \| knxstore.vn` |
| meta description | `Lê Võ Quốc Huy Chuyên viên tư vấn B2B tại KNXStore.vn. Hỗ trợ System Integrator & ME Contractor triển khai KNX, Casambi, DALI-2, BACnet cho tòa nhà thương mại…` (159 ký tự, kết thúc bằng `…`; tên và câu giới thiệu dính liền, không có dấu phân cách) |
| canonical | `https://knxstore.vn/author/le-vo-quoc-huy` |
| robots | Không có meta robots, không có `x-robots-tag` |
| `meta author` | `Lê Võ Quốc Huy` |
| `meta keywords` | `KNX TP / KNX IP DALI-2 Lighting Control BACnet / Modbus Integration Building Management System (BMS) M&E Project Consulting Commercial & Hospitality Automation` (159 ký tự, một chuỗi liền, không có dấu phẩy) |
| viewport / theme-color | `width=device-width, initial-scale=1` / `#14467C` |
| OG | `og:title=Lê Võ Quốc Huy — Sales Engineer` (không hậu tố), `og:description` (= meta description), `og:url`, `og:image=https://api.knxstore.vn/media/97b725830514056a9de009c65ee513e0dfc89b8e538adb71b25533ed7c766777.jpg` (avatar), `og:type=profile`. Không có `og:site_name`, `og:locale`, `profile:*` |
| Twitter | `twitter:card=summary` (không phải `summary_large_image`), `twitter:title`, `twitter:description`, `twitter:image` (avatar) |
| hreflang / `rel=alternate` | Không có |
| `rel=manifest` / icon | `/manifest.webmanifest`; 3 icon như các trang khác |
| preload | `/images/logo/knxstore-logo-tight.svg`; 2 stylesheet `data-precedence=next`; script `3ivcc6t1s04dh.js` (`fetchPriority=low`); `googletagmanager.com/gtag/js?id=G-4D9XLJDVGG`. Không preload avatar |
| script | 10 `<script async src>` chunk trong `<head>` + 1 `noModule` + 1 `3ivcc6t1s04dh.js` cuối `<body>` = 12 thẻ `<script src>` (cùng bộ chunk với trang chủ đề: có `00_ln903pm9z8.js`) |
| Số bytes `<head>` | 3,713 B |

---

## 3. Khung trang

### 3.1 Landmark

| Landmark | Số lượng | Ghi chú |
|---|---|---|
| `<header>` | 1 | `header.sticky.top-0.z-9999`, 128,755 B (30.4% HTML thô) |
| `<nav>` | 10 | header 6, main 1 (`aria-label=Breadcrumb`), footer 3 |
| `<main>` | **0** | Vùng nội dung là `div#main-content[tabindex=-1]` (đích skip-link) |
| `<article>` | 0 | |
| `<aside>` | 0 | |
| `<section>` | 1 | `section.overflow-hidden.bg-[#f3f4f6].pb-20` bọc thẻ hồ sơ + danh sách bài |
| `<footer>` | 1 | 17,608 B |
| `<form>` | 1 | Tìm kiếm header |
| `role` | `role=search`, `role=combobox` (`input#b2b-header-search`), `role=region aria-label=Trending`, `role=group aria-label="Tìm theo"` |

### 3.2 Header và footer (SSR; cùng kích thước và số thẻ với 2 loại trang còn lại)

- Header 128,755 B: nút `Mở menu`, logo, form tìm kiếm (phạm vi 7 chip), giỏ `/cart`, nút `/contact` "Tư vấn dự án", thanh điều hướng (Trending, Danh mục sản phẩm, Giải pháp, Thương hiệu, Công cụ, Blog kỹ thuật, `/visual`, `/matter`). 6 panel mega-menu có trong HTML nhưng ẩn bằng class `hidden`: 16 link `/products/…` kèm giá, 24 `/brands/…`, 12 `/solution/…`, 9 `/blogs/{topic}`, 14 link bài blog; 165 `<a>`, 67 `<img>`, 18 `<h3>`. Text 4,190 ký tự.
- Footer 17,608 B: thông tin pháp nhân (CÔNG TY CP TÍCH HỢP HỆ THỐNG LIÊN MINH, MST 0311846236), liên hệ, Facebook/Zalo, 2 badge (`nofollow`), 3 cột (Danh Mục Sản Phẩm 7 link, Giải Pháp 7, Về chúng tôi 5; mỗi cột render 2 lần: accordion `details` + `nav`), 4 link chính sách. 50 `<a>`, text 1,165 ký tự.

### 3.3 Thành phần nổi / phụ

| Thành phần | Selector | Trạng thái |
|---|---|---|
| Nút Zalo nổi | `a[data-floating-zalo]` → `https://zalo.me/0918918755` | SSR |
| Ngăn kéo "Yêu cầu báo giá" | `div.fixed.top-0.left-0.z-99999…translate-x-full` (`h2` "Yêu cầu báo giá") | SSR (rỗng), ẩn ngoài màn hình |
| Header sticky, skip-link | | SSR |
| GA | `GoogleAnalytics` (gtag `G-4D9XLJDVGG`) | Client |

Không thấy cookie banner, back-to-top, thanh tiến độ, popup trong HTML ban đầu và RSC; chèn sau hydration: Chưa xác định.

---

## 4. Cây nội dung vùng chính (theo thứ tự DOM)

Vùng chính = `div#main-content` (58,526 B, text 20,425 ký tự = 25,025 B). Tất cả mục SSR, hiển thị mặc định (trừ ghi chú).

| # | Thẻ + id/class (rút gọn) | Heading | Nội dung tóm tắt | Main | SSR/Client |
|---|---|---|---|---|---|
| 1 | `script[type=application/ld+json]` (con đầu tiên của `#main-content`) | - | `@graph` 3 node: Organization/LocalBusiness, ProfilePage, Person (4,553 ký tự; mục 8) | Có | SSR |
| 2 | `div.bg-white > div.mx-auto.px-4.sm:px-6.lg:px-8.xl:max-w-site` > `nav[aria-label=Breadcrumb]` | - | `Trang chủ / Lê Võ Quốc Huy`; 1 link `/`; mục cuối `span[aria-current=page]` | Có | SSR |
| 3 | `div.pb-5.lg:pb-6 > h1.text-h1-mobile` | **H1** "Lê Võ Quốc Huy" | Tên tác giả (14 ký tự) | Có | SSR |
| 4 | `section.overflow-hidden.bg-[#f3f4f6].pb-20 > div.mx-auto.max-w-[870px].px-4.pt-10.sm:px-8.xl:px-0` | - | Nền xám, cột nội dung 870px | Có | SSR |
| 5 | `div.rounded-xl.bg-white.p-6.shadow-1.sm:p-10 > div.flex.flex-col.gap-6.sm:flex-row.sm:items-start` | - | Thẻ hồ sơ (732 ký tự text): avatar + thông tin | Có | SSR |
| 5a | `img.h-32.w-32.shrink-0.rounded-full.object-cover` | - | Avatar, `alt="Lê Võ Quốc Huy"`, `loading=lazy`, `width=128 height=128`, `srcSet` 1x/2x (`/_next/image?url=%2Fmedia%2F97b7…jpg&w=128|256&q=75`) | Có | SSR |
| 5b | `p.text-xl.font-semibold.text-dark` | Không phải heading | "Lê Võ Quốc Huy" (lặp lại H1, dùng `p`) | Có | SSR |
| 5c | `p.mt-1.text-dark-4` | - | "Sales Engineer" (chức danh) | Có | SSR |
| 5d | `p.mt-4.whitespace-pre-line.leading-relaxed.text-dark-3` | - | Tiểu sử 496 ký tự, 3 đoạn ngăn bằng ký tự `\n` trong **một** thẻ `p` (hiển thị xuống dòng nhờ `whitespace-pre-line`): kinh nghiệm cùng System Integrator/ME Contractor; tư vấn KNX, Casambi, DALI-2, BACnet, Modbus cho văn phòng, khách sạn, tòa nhà thương mại, khu công nghiệp; liên hệ qua Zalo Hotline | Có | SSR |
| 5e | `div.mt-5` > `p.font-medium` "Lĩnh vực chuyên môn" + `ul.flex.flex-wrap.gap-2` > **1** `li.rounded-full.bg-blue-light-5` | Không phải heading | `li` duy nhất chứa nguyên chuỗi 159 ký tự "KNX TP / KNX IP DALI-2 Lighting Control BACnet / Modbus Integration Building Management System (BMS) M&E Project Consulting Commercial & Hospitality Automation" (các mục chuyên môn không được tách thành từng `li`) | Có | SSR |
| 5f | `ul.mt-5.flex.flex-wrap.gap-4` > 3 `li > a.text-blue` | - | LinkedIn (`linkedin.com/in/huy-le-vo-quoc-177ba8309/`), Facebook (`facebook.com/levoquochuy3011`), Website (`qhsmart.vn/`); `target=_blank rel="noopener noreferrer nofollow"` | Có | SSR |
| 6 | `div.mt-10 > h2.mb-4.text-lg.font-semibold.text-dark` | **H2** "Bài viết của Lê Võ Quốc Huy (82)" | Tiêu đề kèm số bài; trong HTML các đoạn "Bài viết của ", tên, " (", `82`, ")" tách bởi comment `<!-- -->` | Có | SSR |
| 7 | `ul.flex.flex-col.gap-3` > 82 x `li > a.block.rounded-xl.border.border-gray-3.bg-white.p-5[href=/{slug}.html]` | Không dùng heading cho tiêu đề bài | 82 item (mục 4.1) | Có | SSR |
| 8 | (anh em liền sau main) `div.fixed.top-0.left-0.z-99999…` | **H2** "Yêu cầu báo giá" | Ngăn kéo báo giá, ngoài `#main-content` | Không | SSR (rỗng) |

Không có: bảng, code, blockquote, FAQ, video, card sản phẩm, TOC, chuyên mục/tag trên item, ảnh thumbnail trong danh sách (0 `<img>` trong danh sách), phân trang, bộ lọc, bình luận, nút chia sẻ, địa chỉ email/điện thoại cá nhân của tác giả.

### 4.1 Danh sách bài (82 item, tất cả có trong HTML ban đầu)

Cấu trúc 1 item: `li > a[href=/{slug}.html].block.rounded-xl.border.border-gray-3.bg-white.p-5.transition.hover:border-blue-light-4` >
- `p.font-medium.text-dark` — tiêu đề.
- `p.mt-1.line-clamp-2.text-custom-sm.text-dark-4` — mô tả/excerpt (có ở 81/82 item; độ dài 127–548 ký tự, trung bình ≈ 167 ký tự).
- `p.mt-2.text-custom-xs.text-dark-5` — ngày d/m/yyyy (không có nhãn, không có `<time>`).

Item thiếu mô tả: #10 `/dta116a51-vs-dvc-p-00-01-adapter-daikin-sang-modbus-nao-cho-du-an.html` ("DTA116A51 vs DVC-P-00-01 Adapter Daikin Sang Modbus Nào Cho Dự Án", 24/7/2026): không có thẻ `p` mô tả.

Trích 5 item đầu và 3 item cuối (đúng thứ tự DOM):

| # | Tiêu đề | URL | Ngày |
|---|---|---|---|
| 1 | Mở Rộng DALI-2 Ra Dimmer Triac, Relay, DMX Và Modbus TCP | `/mo-rong-dali-2-ra-dimmer-triac-relay-dmx-va-modbus-tcp.html` | 6/11/2026 |
| 2 | Line Segment Và Line Coupler Trong KNX: Chia Line Thế Nào | `/line-segment-va-line-coupler-trong-knx-chia-line-the-nao.html` | 2/11/2026 |
| 3 | RCU Khách Sạn Cho Chủ Đầu Tư Và ME: Phân Loại Và Cách Chọn. | `/rcu-khach-san-cho-chu-dau-tu-va-me-phan-loai-va-cach-chon.html` | 22/10/2026 |
| 4 | Apple HomeView Là Gì? Màn Hình Thông Minh Siri AI Điều Khiển Cả Căn Nhà | `/apple-homeview-la-gi-man-hinh-thong-minh-siri-ai-dieu-khien-ca-can-nha.html` | 14/10/2026 |
| 5 | Giám Sát Năng Lượng Trong Hệ Thống KNX: Vì Sao Phải Đo Điện Trước Khi Tiết Kiệm Điện | `/giam-sat-nang-luong-trong-he-thong-knx-vi-sao-phai-do-dien-truoc-khi-tiet-kiem-dien.html` | 13/10/2026 |
| 80 | Hệ thống KNX là gì? Ứng dụng từ nhà ở đến tòa nhà | `/he-thong-ung-dung-knx.html` | 10/8/2026 |
| 81 | Cảm Biến Cho Nhà Máy: Modbus RTU, DALI Casambi, BACnet/IP | `/viec-ket-noi-giua-cac-cam-bien-khong-con-kho.html` | 19/8/2026 |
| 82 | DALI Là Gì? Kiến Trúc Hệ Thống Điều Khiển Chiếu Sáng DALI | `/dali.html` | 7/8/2026 |

Khoảng ngày hiển thị: 23/3/2026 đến 6/11/2026. Danh sách giảm dần theo ngày ở phần đầu nhưng không đơn điệu toàn bộ (≥ 28 chỗ đảo thứ tự, đo trên 81 item có mô tả). Tiêu chí sắp xếp: Chưa xác định. Một số slug không khớp tiêu đề hiện tại (ví dụ `/viec-ket-noi-giua-cac-cam-bien-khong-con-kho.html` với tiêu đề "Cảm Biến Cho Nhà Máy: …").

Không có phân trang: số ghi ở H2 (82) bằng số `li` trong HTML (82).

---

## 5. Chi tiết khối đặc biệt (trích HTML)

**Thẻ hồ sơ (rút gọn):**

```html
<div class="rounded-xl bg-white p-6 shadow-1 sm:p-10"><div class="flex flex-col gap-6 sm:flex-row sm:items-start"><img alt="Lê Võ Quốc Huy" loading="lazy" width="128" height="128" … class="h-32 w-32 shrink-0 rounded-full object-cover" src="/_next/image?url=%2Fmedia%2F97b7…6777.jpg&amp;w=256&amp;q=75"/><div class="min-w-0"><p class="text-xl font-semibold text-dark xl:text-heading-5">Lê Võ Quốc Huy</p><p class="mt-1 text-dark-4">Sales Engineer</p><p class="mt-4 whitespace-pre-line leading-relaxed text-dark-3">Quốc Huy là chuyên viên tư vấn giải pháp tự động hóa tòa nhà …
Huy chuyên tư vấn và hỗ trợ kỹ thuật cho các dự án …
Để được tư vấn dự án hoặc báo giá, …</p>
```

**Lĩnh vực chuyên môn (một `li` duy nhất) và link mạng xã hội:**

```html
<ul class="flex flex-wrap gap-2"><li class="rounded-full bg-blue-light-5 px-3 py-1 text-custom-sm text-blue-dark">KNX TP / KNX IP DALI-2 Lighting Control BACnet / Modbus Integration Building Management System (BMS) M&amp;E Project Consulting Commercial &amp; Hospitality Automation</li></ul><ul class="mt-5 flex flex-wrap gap-4"><li><a href="https://www.linkedin.com/in/huy-le-vo-quoc-177ba8309/" target="_blank" rel="noopener noreferrer nofollow" class="text-blue hover:underline">LinkedIn</a></li>…
```

**H2 danh sách bài (chuỗi tách bằng comment):**

```html
<h2 class="mb-4 text-lg font-semibold text-dark">Bài viết của <!-- -->Lê Võ Quốc Huy<!-- --> (<!-- -->82<!-- -->)</h2>
```

**1 item danh sách:**

```html
<li><a class="block rounded-xl border border-gray-3 bg-white p-5 transition hover:border-blue-light-4" href="/mo-rong-dali-2-ra-dimmer-triac-relay-dmx-va-modbus-tcp.html"><p class="font-medium text-dark">Mở Rộng DALI-2 Ra Dimmer Triac, Relay, DMX Và Modbus TCP</p><p class="mt-1 line-clamp-2 text-custom-sm text-dark-4">DALI-2 không phải hệ kín: đi qua 5 đường mở rộng gồm relay DT7, dimmer triac, DMX512, BMS Modbus TCP và Apple Home. …</p><p class="mt-2 text-custom-xs text-dark-5">6/11/2026</p></a></li>
```

**JSON-LD Person (trích):**

```json
{"@type":"Person","@id":"https://knxstore.vn/author/le-vo-quoc-huy#author","name":"Lê Võ Quốc Huy","url":"https://knxstore.vn/author/le-vo-quoc-huy","jobTitle":"Sales Engineer","knowsAbout":["KNX TP / KNX IP DALI-2 Lighting Control BACnet / Modbus Integration …"],"worksFor":{"@id":"https://knxstore.vn/#organization"},"sameAs":["https://www.linkedin.com/in/huy-le-vo-quoc-177ba8309/","https://www.facebook.com/levoquochuy3011","https://qhsmart.vn/"]}
```

---

## 6. Nội dung chỉ có ở client hoặc KHÔNG thấy trong HTML ban đầu

| Mục | Bằng chứng | Ghi chú |
|---|---|---|
| Hồ sơ tác giả + 82 bài | Có đủ trong HTML ban đầu; RSC cho thấy các item là Server Component | Không client-only |
| Avatar | `<img loading=lazy>` có trong HTML | Markup SSR; tải ảnh bị trì hoãn |
| Ngăn kéo báo giá, badge giỏ | Trạng thái rỗng trong HTML ("Tổng số lượng: 0", `Giỏ báo giá, 0 sản phẩm`) | Nội dung thực: Client |
| Gợi ý tìm kiếm, mở/đóng mega-menu | `role=combobox` rỗng; panel `hidden` | Client |
| Menu mobile | Không thấy markup drawer trong HTML | Chưa xác định |
| GA, cookie `knx-area=b2b` | `GoogleAnalytics` client component; script inline đầu `<body>` | Client |

Không có `<template data-dgst>`, `BAILOUT_TO_CLIENT_SIDE_RENDERING`, skeleton/`animate-pulse`, `<!--$?-->`, `<!--$!-->`.

---

## 7. Liên kết

| Loại | Mẫu URL | Số lượng trong main | Ví dụ |
|---|---|---|---|
| Bài blog | `/{slug}.html` | 82 (cả 82 đều kết thúc `.html`) | `/dali.html` |
| Breadcrumb | `/` | 1 | |
| Mạng xã hội / website ngoài | `https://www.linkedin.com/in/…`, `https://www.facebook.com/…`, `https://qhsmart.vn/` | 3, tất cả `target=_blank rel="noopener noreferrer nofollow"` | |
| Chuyên mục, sản phẩm, tác giả khác | - | 0 trong main | |

Toàn trang 305 `<a>` (header 165, main 86, ngăn kéo 2, footer 50, skip-link 1, Zalo 1). Link `nofollow`: 3 link ngoài của tác giả + 2 badge footer. Không có `rel=alternate`/link markdown. URL `/author/{slug}` của tác giả được trang bài trỏ tới (`/author/ta-minh-vu` từ trang bài mẫu), trang này không có link ngược tới trang tác giả khác.

---

## 8. JSON-LD

1 thẻ `<script type="application/ld+json">` (con đầu tiên của `div#main-content`, 4,885 B kèm thẻ; nội dung 4,553 ký tự), `@graph` 3 node. Nội dung cũng có trong RSC (`8d:T12e9,…`).

| # | `@type` | `@id` | Trường có mặt | Thiếu / rỗng / lưu ý |
|---|---|---|---|---|
| 1 | `["Organization","LocalBusiness"]` | `https://knxstore.vn/#organization` | Như trang bài: name, legalName, url, foundingDate, taxID, logo, image, description, telephone, email, address, geo, hasMap, openingHoursSpecification, priceRange, areaServed, knowsAbout, contactPoint, sameAs (Facebook, YouTube) | Giống hệt node tổ chức ở các trang khác |
| 2 | `ProfilePage` | `…/author/le-vo-quoc-huy#webpage` | url, name, description (= meta description), inLanguage `vi-VN`, isPartOf, publisher, mainEntity (→ `#author`), primaryImageOfPage (ImageObject không có `caption`), breadcrumb (BreadcrumbList 2 ListItem: Trang chủ, Lê Võ Quốc Huy) | `isPartOf` trỏ `https://knxstore.vn/#website`: không có node `WebSite` trong `@graph`. Không có `dateCreated`/`dateModified`. Không có danh sách bài (`hasPart`/`ItemList`) |
| 3 | `Person` | `…/author/le-vo-quoc-huy#author` | name, url, image (ImageObject + caption), jobTitle `Sales Engineer`, description (bio đầy đủ, chứa `\n`), knowsAbout (mảng **1 phần tử** là cả chuỗi 159 ký tự), worksFor (→ Organization), sameAs (3 URL: LinkedIn, Facebook, qhsmart.vn) | Không có `email`, `telephone`, `alumniOf`, `award`. `knowsAbout` không tách từng chủ đề |

Meta `description` của trang và `ProfilePage.description` là cùng một chuỗi (tên + câu giới thiệu dính liền), khác với `Person.description` (bio đầy đủ).

---

## 9. Số đo

| Chỉ số | Giá trị |
|---|---|
| HTML thô | **423,422 B** (≈ 105,855 token theo bytes/4) |
| Sau nén (server) | gzip **74,743 B**; br **46,588 B** |
| Tổng thẻ `<script>` | 16 thẻ, **217,037 B** (51.3% HTML thô) |
| - `self.__next_f` | 2 thẻ, **211,086 B**; RSC giải mã = 184,606 ký tự, 148 dòng |
| - `ld+json` | 1 thẻ, 4,885 B |
| - `<script src>` | 12 |
| `<style>` inline | 0 |
| Text hiển thị (bỏ script/style) | **25,961 ký tự** = 31,833 B UTF-8 (≈ 7,958 token) |
| Text `#main-content` (không gồm ngăn kéo) | **20,425 ký tự** = 25,025 B (≈ 6,256 token); 5.9% HTML thô |
| Trong đó danh sách 82 bài | ≈ 19,796 ký tự; thẻ hồ sơ 732 ký tự; H1 + breadcrumb ≈ 41 ký tự |
| Text header / footer | 4,190 / 1,165 ký tự |
| Bytes theo vùng | `<head>` 3,713 · header 128,755 · main (gồm JSON-LD) 58,526 · ngăn kéo 2,585 · footer 17,608 · phần sau footer (Zalo + script) 211,828 |
| Thẻ | `<a>` 305 · `<img>` 71 (header 67, avatar 1, footer 3) · `<table>` 0 · `<h1>` 1 · `<h2>` 5 · `<h3>` 18 · `<h4>`-`<h6>` 0 · `<li>` 259 · `<ul>` 34 · `<details>` 3 · `<svg>` 33 · `<button>` 17 · `<form>` 1 · `<nav>` 10 |
| Chi tiết h2 | "Bài viết của Lê Võ Quốc Huy (82)" + "Yêu cầu báo giá" + 3 footer |
| Chi tiết h3 | 18, toàn bộ trong mega-menu header (2 thẻ rỗng) |

Request đã dùng: 5 (trang, `Accept: text/markdown`, `.md`, br, gzip).

---

## 10. Chưa xác định / lưu ý

- **Cấu hình render ở origin:** Chưa xác định (không có quyền xem source); "dynamic" suy ra từ header.
- **Tính đại diện:** chỉ khảo sát 1 trong 2 URL tác giả. Trang `/author/ta-minh-vu` không được fetch; từ JSON-LD Person trong trang bài mẫu, tác giả đó có `jobTitle` "System Integration & Marketing Engineer", `description` chứa `\n`, không có `sameAs`; cấu trúc hồ sơ trên trang (ví dụ có avatar, mạng xã hội, "Lĩnh vực chuyên môn" hay không) Chưa xác định.
- Trang này (và có thể cả loại tác giả) dùng bộ class khác với trang blog/chủ đề (`text-dark`, `bg-blue-light-5`, `shadow-1`, `text-h1-mobile`), phản ánh khác biệt giao diện giữa các loại trang.
- Một số điểm dữ liệu trông chưa được tách trường: "Lĩnh vực chuyên môn" là 1 chuỗi liền trong cả HTML, `meta keywords` và JSON-LD `knowsAbout`; `meta description` nối tên với câu giới thiệu không có dấu phân cách. Chưa xác định đây là dữ liệu nhập gốc hay do template.
- Ngày hiển thị trong danh sách: chưa xác định là ngày đăng hay ngày cập nhật; không có nhãn, không có thẻ `<time>`.
- Thứ tự danh sách không đơn điệu theo ngày; tiêu chí sắp xếp Chưa xác định.
- 82 bài của tác giả này hiển thị 1 trang duy nhất không phân trang; với tác giả có nhiều bài hơn, hành vi Chưa xác định (không có mẫu so sánh).
- DOM sau hydration chưa quan sát (không chạy trình duyệt); toàn bộ nội dung trang tác giả là markup server nên không dự kiến phụ thuộc client, nhưng chưa kiểm chứng bằng trình duyệt.
- Header chiếm 30.4% HTML thô và 16% text hiển thị (4,190/25,961 ký tự); mega-menu có mặt trong DOM nhưng ẩn bằng class `hidden`.
