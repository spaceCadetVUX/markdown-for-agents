# Trang chủ đề blog (listing) — cấu trúc hiện tại

> https://knxstore.vn/blogs/an-ninh · nguồn sitemap: posts.xml · 9 URL cùng loại (`/blogs/{topic}`; `/blogs` gốc ngoài phạm vi) · khảo sát 2026-11-11

Phạm vi: chỉ mô tả hiện trạng, đo trên HTML thô lấy bằng `curl -A "Mozilla/5.0"` (không chạy trình duyệt). Để trả lời câu hỏi phân trang, đã lấy thêm 1 chủ đề lớn hơn làm mẫu bổ sung: `/blogs/kien-thuc` (kèm `?page=2`); số liệu của mẫu bổ sung được ghi rõ là "mẫu bổ sung", mọi mục còn lại đo trên `/blogs/an-ninh`.

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
| link | 6 preload: 4 font `woff2` + 2 style (`/_next/static/chunks/1mmmxvxvvtkv8.css`, `/_next/static/chunks/1vbr8oh4939ca.css`). Không có `rel=alternate` |
| x-robots-tag | Không có |
| content-encoding | Không có khi không gửi `Accept-Encoding`; `br` khi gửi `gzip, br`; `gzip` khi chỉ gửi `gzip` |
| Khác | `server: cloudflare`, `x-frame-options: SAMEORIGIN`, `x-content-type-options: nosniff`, `strict-transport-security`, `referrer-policy: same-origin`, `speculation-rules: "/cdn-cgi/speculation"`, `alt-svc: h3`. Không có `etag`, `last-modified`, `age` |

### 1.2 Chế độ render (suy ra)

Kết luận: **dynamic render theo từng request (SSR), không cache ở Cloudflare**. Bằng chứng: `cache-control: private, no-cache, no-store, max-age=0, must-revalidate` (không `s-maxage`); không có `x-nextjs-cache`/`x-nextjs-prerender`/`x-nextjs-stale-time`; `cf-cache-status: DYNAMIC`; RSC dòng `0:` có `"S":false` (ý nghĩa cờ chưa xác nhận bằng tài liệu).

- Route (từ RSC): `(site)/(b2b)/blogs/[seg]`, `seg` = `an-ninh` (khác bài blog là `(site)/[slug]`).
- Build ID `rJVjL74JjgHtrkQQTzrjV` (trùng 2 loại trang còn lại). Phiên bản Next.js: Chưa xác định.
- Danh sách bài được render phía server: 14 card có đầy đủ trong HTML ban đầu (không phải fetch phía client).

### 1.3 Hành vi hiện tại với yêu cầu Markdown (chỉ ghi kết quả)

| Request | Status | content-type | Kích thước body |
|---|---|---|---|
| `GET /blogs/an-ninh` + `Accept: text/markdown` | 200 | `text/html; charset=utf-8` | 349,744 B (bằng bản thường) |
| `GET /blogs/an-ninh.md` | 404 | `text/html; charset=utf-8` | 131,204 B |

Response 404 vẫn mang `cache-control: private, no-cache, no-store…`.

---

## 2. `<head>`

| Thành phần | Giá trị |
|---|---|
| `<title>` | `An ninh \| knxstore.vn` (tên chủ đề + hậu tố site) |
| meta description | `Nhà phân phối thiết bị KNX, Casambi, DALI và giải pháp nhà thông minh. Tra cứu thông số kỹ thuật, tải catalog, gửi yêu cầu báo giá.` (chung cho mọi chủ đề: `/blogs/kien-thuc` có cùng chuỗi; không mô tả chủ đề) |
| canonical | `https://knxstore.vn/blogs/an-ninh` |
| robots | Không có meta robots, không có `x-robots-tag` |
| `meta author` / `keywords` | Không có |
| viewport / theme-color | `width=device-width, initial-scale=1` / `#14467C` |
| OG | `og:title=An ninh` (không hậu tố), `og:description` (= meta description), `og:url`, `og:site_name=knxstore.vn`, `og:locale=vi_VN`, `og:image=https://knxstore.vn/images/og/knxstore-og.png` (ảnh chung của site), `og:type=website` |
| Twitter | `twitter:card=summary_large_image`, `twitter:title`, `twitter:description`, `twitter:image` (ảnh chung) |
| hreflang / `rel=alternate` / `rel=prev`/`next` | Không có |
| `rel=manifest` / icon | `/manifest.webmanifest`; 3 icon (`favicon.ico`, `icon.svg`, `apple-icon.png`) như trang bài |
| preload | `/images/logo/knxstore-logo-tight.svg`; 2 stylesheet `data-precedence=next`; script `3ivcc6t1s04dh.js` (`fetchPriority=low`); `googletagmanager.com/gtag/js?id=G-4D9XLJDVGG`. Không preload ảnh nội dung |
| script | 10 `<script async src>` chunk trong `<head>` (khác trang bài ở chỗ có `00_ln903pm9z8.js`, không có `2ns7…`/`39o1…`) + 1 `noModule` (`0cz1d0mv5g_q7.js`) + 1 `3ivcc6t1s04dh.js` (`id=_R_ async`, cuối `<body>`). Tổng 12 thẻ `<script src>` |
| Số bytes `<head>` | 3,284 B |

---

## 3. Khung trang

### 3.1 Landmark

| Landmark | Số lượng | Ghi chú |
|---|---|---|
| `<header>` | 1 | `header.sticky.top-0.z-9999`, 128,755 B (36.8% HTML thô) |
| `<nav>` | 10 | header 6, main 1 (`aria-label=Breadcrumb`), footer 3. Dãy chip lọc chuyên mục **không** nằm trong `<nav>` |
| `<main>` | **0** | Vùng nội dung là `div#main-content[tabindex=-1]` (đích skip-link `a[href=#main-content]`) |
| `<article>` | 0 | Card bài là `<a>` bọc, không có `<article>` |
| `<aside>` | **0** | Không có sidebar |
| `<section>` | 0 | |
| `<footer>` | 1 | 17,608 B |
| `<form>` | 2 | Tìm kiếm header; tìm bài viết (`form[role=search]` trong main) |
| `role` | `role=search` x2, `role=combobox` (`input#b2b-header-search`), `role=region aria-label=Trending`, `role=group aria-label="Tìm theo"` |

### 3.2 Header (SSR; cùng kích thước 128,755 B, 165 `<a>`, 67 `<img>`, 123 `<li>`, 18 `<h3>` như trang bài)

Thanh trên: nút `Mở menu`, logo, form tìm kiếm (phạm vi: Tất cả, Sản phẩm, Bài viết, Thương hiệu, Giải pháp, Danh mục, Visual), link giỏ `/cart`, nút "Tư vấn dự án" (`/contact`). Thanh điều hướng: Trending, Danh mục sản phẩm, Giải pháp, Thương hiệu, Công cụ, Blog kỹ thuật, `/visual`, `/matter` ("Matter Smarthome"). Sáu panel mega-menu có trong HTML, ẩn bằng class `hidden`, gồm 16 link `/products/…` kèm giá, 24 link `/brands/…`, 12 `/solution/…`, 9 `/blogs/{topic}`, 14 link bài blog. Text header: 4,190 ký tự.

### 3.3 Footer (SSR; giống trang bài)

Cột trái thông tin pháp nhân (CÔNG TY CP TÍCH HỢP HỆ THỐNG LIÊN MINH, MST 0311846236, địa chỉ, `tel:0918918755`, `mailto:support@knxstore.vn`, Facebook, Zalo, badge Bộ Công Thương + DMCA `rel="nofollow noopener noreferrer"`); 3 cột Danh Mục Sản Phẩm (7 link) / Giải Pháp (7) / Về chúng tôi (5), mỗi cột render 2 lần (accordion `details` mobile + `nav` desktop); dòng bản quyền + 4 link chính sách. 50 `<a>`, text 1,165 ký tự.

### 3.4 Thành phần nổi / phụ

| Thành phần | Selector | Trạng thái |
|---|---|---|
| Nút Zalo nổi | `a[data-floating-zalo]` → `https://zalo.me/0918918755`, fixed góc phải dưới | SSR |
| Ngăn kéo "Yêu cầu báo giá" | `div.fixed.top-0.left-0.z-99999…translate-x-full` (`h2` "Yêu cầu báo giá", "Tổng số lượng: 0") | SSR (rỗng), ẩn ngoài màn hình |
| Header sticky, skip-link | `header.sticky`, `a[href=#main-content]` | SSR |
| GA | `GoogleAnalytics` (gtag `G-4D9XLJDVGG`) | Client |

Không thấy trong HTML ban đầu và payload RSC: cookie banner, back-to-top, thanh tiến độ, popup. Chèn sau hydration hay không: Chưa xác định.

---

## 4. Cây nội dung vùng chính (theo thứ tự DOM)

Vùng chính = `div#main-content` (40,062 B, text 1,419 ký tự = 1,732 B). Không có JSON-LD trong trang. Tất cả mục dưới đây SSR, hiển thị mặc định.

| # | Thẻ + id/class (rút gọn) | Heading | Nội dung tóm tắt | Main | Hiển thị mặc định | SSR/Client |
|---|---|---|---|---|---|---|
| 1 | `div.bg-white > div.mx-auto.max-w-[1320px].px-6.pb-16.pt-6` | - | Container bao toàn bộ | Có | Hiện | SSR |
| 2 | `nav[aria-label=Breadcrumb]` | - | `Trang chủ / Blog kỹ thuật / An ninh`; link `/`, `/blogs`; mục cuối `span[aria-current=page]` | Có | Hiện | SSR |
| 3 | `div.max-w-[72ch]` > `h1.text-[32px].sm:text-[38px]` | **H1** "An ninh" | Tên chủ đề (7 ký tự) | Có | Hiện | SSR |
| 4 | `p.mt-2.5.text-[15px].text-zinc-600` | - | Mô tả chung: "Kiến thức, hướng dẫn cấu hình và tin tức sản phẩm về KNX, DALI-2, BACnet, Casambi và Matter Smarthome, viết cho kỹ sư tích hợp hệ thống và đội thi công M&E." (156 ký tự; giống hệt ở `/blogs/kien-thuc`) | Có | Hiện | SSR |
| 5 | `form[role=search][action=/search][method=get].max-w-[420px]` | - | `input[type=hidden][name=kind][value=post]`; `label.sr-only[for=blog-search]` "Tìm bài viết"; `input#blog-search[type=search][name=q]` placeholder "Tìm bài viết..." (GET `/search?kind=post&q=…`, không cần JS) | Có | Hiện | SSR |
| 6 | `div.mt-7.border-b > div[data-blog-filters].flex.flex-wrap.gap-2` | - | 10 chip là thẻ `a` (điều hướng đầy đủ, không lọc phía client): Tất cả (`/blogs`), An ninh (active: `aria-current=page`, `bg-primary text-white`), Cảm biến, Chiếu sáng, Dự án, Hướng dẫn, HVAC, Kiến thức, KNX, Tin tức (`/blogs/{topic}`) | Có | Hiện | SSR |
| 7 | `p.mb-4.mt-9.uppercase.text-primary` | Không phải heading | "14 bài viết" (nhãn đếm, `p`) | Có | Hiện | SSR |
| 8 | `div[data-blog-grid].grid.grid-cols-1.sm:grid-cols-2.lg:grid-cols-3` > 14 x `a.group.flex.h-full…` | - | 14 card bài (mục 4.1) | Có | Hiện | SSR |
| 9 | (anh em liền sau main) `div.fixed.top-0.left-0.z-99999…` | **H2** "Yêu cầu báo giá" | Ngăn kéo báo giá, ngoài `#main-content` | Không | Ẩn | SSR (rỗng) |

Không có: mô tả dài/intro của chủ đề, bộ lọc theo tag/ngày/sắp xếp, sidebar, bài nổi bật (featured), phân trang, nút "xem thêm", JSON-LD, tác giả trên card.

### 4.1 Cấu trúc 1 card

`a[href=/{slug}.html].group.flex.h-full.min-w-0.flex-col.overflow-hidden.rounded-card.border` >
- `div.relative.aspect-[4/3]` > `img.object-cover[loading=lazy][decoding=async][data-nimg=fill]` (next/image: `/_next/image?url=%2Fmedia%2F<sha256>.jpg&w=3840&q=75`, `srcSet` 10 mức 256→3840w ≈ 1,228 ký tự, `sizes="(max-width: 640px) 50vw, (max-width: 1024px) 33vw, 25vw"`, `alt` = tiêu đề; cả 14 ảnh `.jpg`, cả 14 `loading=lazy`).
- `div.flex-1.px-4.pb-4.pt-3.5` > `span.truncate.uppercase` (nhãn chuyên mục) + `p.line-clamp-2` (tiêu đề, **không** phải thẻ heading) + `span.mt-auto` "Ngày đăng <b>d/m/yyyy</b>".

Không có: mô tả/excerpt, tác giả, thời gian đọc, tag. Khoảng trắng giữa "Ngày đăng" và ngày là comment `<!-- -->`.

| # | Tiêu đề (đúng như hiển thị) | URL | Ngày |
|---|---|---|---|
| 1 | KNX & SATEL: Giải pháp kết nối an ninh hai chiều với module INT-KNX-2 | `/knx-satel-ket-noi-an-ninh-hai-chieu-voi-module-int-knx-2.html` | 4/8/2026 |
| 2 | Camera Aqara G5 Pro Hay Eve Cam? Nên Chọn Loại Nào | `/camera-chong-trom-la-gi.html` | 12/8/2026 |
| 3 | Khám phá 12 mẫu khóa cửa thông minh Moorgen ứng dụng vân tay tĩnh mạch hiện đại | `/khoa-cua-thong-minh-moorgen.html` | 24/7/2025 |
| 4 | So sánh hệ thống báo động có dây và không dây | `/so-sanh-he-thong-bao-dong-co-day-va-khong-day.html` | 23/7/2025 |
| 5 | Hệ thống chống trộm là gì? Những bộ phận nào cấu thành hệ thống? | `/he-thong-chong-trom-la-gi.html` | 1/8/2026 |
| 6 | Đặt cảm biến an ninh ở đâu là đúng? 7 vị trí bạn không thể bỏ qua | `/nhung-vi-tri-nen-lap-cam-bien-an-ninh-trong-nha.html` | 16/7/2025 |
| 7 | Thiết bị chống trộm ngoài trời | `/thiet-bi-chong-trom-ngoai-troi.html` | 11/8/2025 |
| 8 | Nguyên lý, cấu tạo của đầu báo khói | `/dau-bao-chay.html` | 21/8/2025 |
| 9 | Chuông cửa thông minh | `/chuong-cua-thong-minh.html` | 12/8/2025 |
| 10 | Tìm hiểu về cảm biến chuyển động từ thương hiệu Satel | `/cam-bien-chuyen-dong.html` | 9/5/2025 |
| 11 | Tìm hiểu về hệ thống an ninh Perfecta | `/he-thong-an-ninh-perfecta.html` | 19/2/2025 |
| 12 | Hệ thống an ninh gia đình: Lý do bạn cần lắp ngay hôm nay | `/he-thong-chong-trom-an-ninh-gia-dinh.html` | 17/7/2025 |
| 13 | Hệ thống an ninh Satel tại Triển lãm Quốc tế INTERSEC 2019 | `/trien-lam-an-toan-an-ninh-va-cuu-ho-quoc-te-intersec-2019-tai-dubai.html` | 19/2/2025 |
| 14 | Khám phá hệ thống an ninh thông minh Satel chuẩn Âu | `/part-1-giai-phap-an-ninh-thong-minh-toan-dien-nhat-cua-chau-au-duoc-the-gioi-cong-nhan-trong-nam-2018-la-gi.html` | 1/8/2026 |

Thứ tự card không đơn điệu theo ngày hiển thị (ví dụ 12/8/2026 → 24/7/2025 → 23/7/2025 → 1/8/2026). Tiêu chí sắp xếp: Chưa xác định.

### 4.2 Phân trang

- **Mẫu `/blogs/an-ninh`:** tổng "14 bài viết", đủ 14 card trong 1 trang, không có phần tử phân trang (không `nav` phân trang, không `rel=next`, không link `?page=`, không nút "xem thêm"). Trong RSC, phần tử cuối của container sau lưới là `$undefined`.
- **Mẫu bổ sung `/blogs/kien-thuc`:** nhãn "84 bài viết", **84 card đều có trong HTML ban đầu của trang đầu**, HTML thô 632,537 B (≈ 158,134 token), text hiển thị 13,301 ký tự, main text 7,787 ký tự. Không có phần tử phân trang.
- **`/blogs/kien-thuc?page=2`:** 200, 632,576 B, nội dung giống trang đầu (cùng 84 card, cùng card đầu `/giam-sat-nang-luong-trong-he-thong-knx-vi-sao-phai-do-dien-truoc-khi-tiet-kiem-dien.html` và card cuối `/bai-thuyet-trinh-10-phut-ve-he-thong-an-ninh-thong-minh-satel.html`), `canonical` vẫn là `/blogs/kien-thuc`, `<title>` vẫn `Kiến thức | knxstore.vn`. Tham số `page` không làm thay đổi danh sách.
- Hệ quả đo được: không có "bài ở trang 2" trong cả hai mẫu; toàn bộ danh sách chủ đề nằm trong một HTML.

---

## 5. Chi tiết khối đặc biệt (trích HTML)

**Chip chuyên mục (điều hướng bằng link, chip active có `aria-current`):**

```html
<div data-blog-filters="" class="flex flex-wrap gap-2"><a class="shrink-0 whitespace-nowrap rounded-control border-[1.5px] … border-zinc-300 bg-white text-zinc-700 …" href="/blogs">Tất cả</a><a aria-current="page" class="… border-primary bg-primary text-white" href="/blogs/an-ninh">An ninh</a>…
```

**Form tìm bài viết:**

```html
<form role="search" class="relative mt-4 w-full max-w-[420px]" action="/search" method="get"><input type="hidden" name="kind" value="post"/><label for="blog-search" class="sr-only">Tìm bài viết</label><svg/><input id="blog-search" type="search" placeholder="Tìm bài viết..." … name="q"/></form>
```

**Nhãn đếm + lưới + 1 card:**

```html
<p class="mb-4 mt-9 … uppercase … text-primary">14 bài viết</p><div data-blog-grid="" class="grid grid-cols-1 gap-4 sm:grid-cols-2 sm:gap-5 lg:grid-cols-3"><a class="group flex h-full min-w-0 flex-col … rounded-card border border-zinc-200 bg-white …" href="/knx-satel-ket-noi-an-ninh-hai-chieu-voi-module-int-knx-2.html"><div class="relative flex aspect-[4/3] …"><img alt="KNX &amp; SATEL: …" loading="lazy" decoding="async" data-nimg="fill" class="object-cover" sizes="(max-width: 640px) 50vw, (max-width: 1024px) 33vw, 25vw" srcSet="…" src="/_next/image?url=%2Fmedia%2Fa74e…1164.jpg&amp;w=3840&amp;q=75"/></div><div class="flex flex-1 flex-col px-4 pb-4 pt-3.5"><span class="truncate … uppercase …">An ninh</span><p class="mt-1.5 line-clamp-2 …">KNX &amp; SATEL: Giải pháp kết nối an ninh hai chiều với module INT-KNX-2</p><span class="mt-auto pt-3 …">Ngày đăng<!-- --> <b class="font-semibold text-zinc-800">4/8/2026</b></span></div></a>
```

**Breadcrumb + H1 + mô tả (payload RSC dòng `6:`):**

```
["$","h1",null,{"className":"text-[32px] font-bold …","children":"An ninh"}],["$","p",null,{"className":"mt-2.5 …","children":"Kiến thức, hướng dẫn cấu hình và tin tức sản phẩm về KNX, DALI-2, …"}]
```

---

## 6. Nội dung chỉ có ở client hoặc KHÔNG thấy trong HTML ban đầu

| Mục | Bằng chứng | Ghi chú |
|---|---|---|
| Danh sách bài (14 card) | Có đủ trong HTML ban đầu (14 `a.group…` trong `div[data-blog-grid]`) | Không phải client-only |
| Chip chuyên mục, form tìm bài | Có trong HTML; chip là `<a href>`, form là GET thuần | Hoạt động không cần JS |
| Ảnh card | Markup `<img>` có trong HTML; `loading=lazy` ở cả 14 ảnh (tải ảnh bị trì hoãn tới khi cuộn) | Markup SSR |
| Ngăn kéo báo giá, badge giỏ | Trạng thái rỗng trong HTML ("Tổng số lượng: 0", `Giỏ báo giá, 0 sản phẩm`) | Nội dung thực: Client |
| Gợi ý tìm kiếm, mở/đóng mega-menu, chọn phạm vi tìm | `role=combobox` rỗng; panel `hidden` | Client |
| Menu mobile | Không thấy markup drawer trong HTML (chỉ có nút `Mở menu`) | Chưa xác định |
| GA, cookie `knx-area=b2b` | `GoogleAnalytics` client component; script inline đầu `<body>` | Client |

Không có `<template data-dgst>`, `BAILOUT_TO_CLIENT_SIDE_RENDERING`, skeleton/`animate-pulse`, `<!--$?-->`, `<!--$!-->` (chỉ `<!--$-->` rỗng). Không có dữ liệu nội dung chủ đề chỉ nằm trong RSC: các card trong RSC (`46:`, 4,046 ký tự + các dòng card) trùng với HTML.

---

## 7. Liên kết

| Loại | Mẫu URL | Số lượng trong main | Ví dụ |
|---|---|---|---|
| Bài blog (card) | `/{slug}.html` | 14 | `/camera-chong-trom-la-gi.html` |
| Chuyên mục (chip) | `/blogs`, `/blogs/{topic}` | 10 (+1 `/blogs` ở breadcrumb) | `/blogs/cam-bien` (9 topic: an-ninh, cam-bien, chieu-sang, du-an, huong-dan, hvac, kien-thuc, knx, tin-tuc) |
| Breadcrumb | `/`, `/blogs` | 2 | |
| Tìm kiếm | `/search?kind=post&q=` (form GET) | 1 form | |
| Tác giả, sản phẩm, danh mục | - | 0 trong main (sản phẩm/danh mục/giải pháp/thương hiệu chỉ có trong header/footer) | |

Toàn trang 245 `<a>` (header 165, main 26, ngăn kéo 2, footer 50, skip-link 1, Zalo 1). Không có link ngoài trong main. Link `nofollow`: 2 badge footer. Không có `rel=alternate`/link markdown. Link card trỏ thẳng `/{slug}.html`.

---

## 8. JSON-LD

Không có. 0 thẻ `<script type="application/ld+json">`, không có chuỗi `schema.org`/`BreadcrumbList` trong HTML hoặc payload RSC (không có BreadcrumbList, CollectionPage hay ItemList cho trang này).

---

## 9. Số đo

| Chỉ số | Giá trị |
|---|---|
| HTML thô | **349,744 B** (≈ 87,436 token theo bytes/4) |
| Sau nén (server) | gzip **53,500 B**; br **32,447 B** |
| Tổng thẻ `<script>` | 15 thẻ, **157,367 B** (45.0% HTML thô) |
| - `self.__next_f` | 2 thẻ, **156,301 B**; RSC giải mã = 137,953 ký tự, 84 dòng |
| - `ld+json` | 0 |
| - `<script src>` | 12 |
| `<style>` inline | 0 |
| Text hiển thị (bỏ script/style) | **6,931 ký tự** = 8,510 B UTF-8 (≈ 2,127 token) |
| Text `#main-content` (không gồm ngăn kéo) | **1,419 ký tự** = 1,732 B (≈ 433 token); 0.5% HTML thô |
| Text header / footer | 4,190 / 1,165 ký tự |
| Bytes theo vùng | `<head>` 3,284 · header 128,755 · main 40,062 · ngăn kéo 2,585 · footer 17,608 · phần sau footer (Zalo + script) 157,043 |
| Thẻ | `<a>` 245 · `<img>` 84 (header 67, main 14, footer 3) · `<table>` 0 · `<h1>` 1 · `<h2>` 4 · `<h3>` 18 · `<h4>`-`<h6>` 0 · `<li>` 173 · `<ul>` 31 · `<details>` 3 · `<svg>` 34 · `<button>` 17 · `<form>` 2 · `<input>` 3 · `<nav>` 10 |
| Chi tiết h2 | "Yêu cầu báo giá" + 3 footer (Danh Mục Sản Phẩm, Giải Pháp, Về chúng tôi). Trong vùng nội dung chủ đề chỉ có 1 heading là H1 |
| Chi tiết h3 | 18, toàn bộ trong mega-menu header (2 thẻ rỗng) |
| Mẫu bổ sung `/blogs/kien-thuc` | HTML thô 632,537 B (≈ 158,134 token), 84 card; text 13,301 ký tự (≈ 4,085 token); main text 7,787 ký tự (≈ 2,389 token) |

Request đã dùng cho `/blogs/an-ninh`: 5 (trang, `Accept: text/markdown`, `.md`, br, gzip). Mẫu bổ sung: 2 request (`/blogs/kien-thuc`, `?page=2`).

---

## 10. Chưa xác định / lưu ý

- **Cấu hình render ở origin:** Chưa xác định (không có quyền xem source); "dynamic" suy ra từ header.
- **Tính đại diện:** `/blogs/an-ninh` có 14 bài. Chủ đề nhỏ hơn/lớn hơn có thể khác (mẫu bổ sung `/blogs/kien-thuc` có 84 bài vẫn không phân trang). 7 chủ đề còn lại (cam-bien, chieu-sang, du-an, huong-dan, hvac, knx, tin-tuc) không được kiểm tra riêng. Ngưỡng bài mà tại đó có phân trang (nếu có): Chưa xác định; tham số `page` không có tác dụng ở mẫu đã thử.
- Trang `/blogs/kien-thuc` (84 bài) cho HTML 632,537 B: kích thước HTML tăng theo số card (mỗi card kèm `srcSet` ≈ 1.2 KB), không có cơ chế cắt danh sách.
- Mô tả chủ đề (`p`) và `meta description` là chuỗi chung cho cả site/blog, không phản ánh nội dung từng chủ đề; không rõ có chủ đề nào (trong 9) có mô tả riêng hay không.
- Không có JSON-LD (kể cả BreadcrumbList) trong trang listing, dù breadcrumb hiển thị có 3 cấp.
- Card không có excerpt/tác giả; chỉ có nhãn chuyên mục, tiêu đề, "Ngày đăng", ảnh. Chưa xác định ngày hiển thị là ngày đăng hay ngày cập nhật (nhãn ghi "Ngày đăng").
- Thứ tự card không theo ngày hiển thị; tiêu chí sắp xếp Chưa xác định.
- Nhãn chuyên mục trên card ("An ninh") trùng tên chủ đề; bài thuộc nhiều chuyên mục (nếu có) không thể hiện trong HTML.
- DOM sau hydration chưa quan sát (không chạy trình duyệt); danh sách và chip đều là markup server nên không dự kiến thay đổi nội dung, nhưng điều này chưa kiểm chứng bằng trình duyệt.
- Header chiếm 36.8% HTML thô và 60% text hiển thị (4,190/6,931 ký tự) của trang; vùng nội dung chỉ 0.5% bytes HTML.
