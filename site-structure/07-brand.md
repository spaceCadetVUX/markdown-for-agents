# Trang thương hiệu — cấu trúc hiện tại

> URL mẫu: https://knxstore.vn/brands/protopixel · nguồn sitemap: https://knxstore.vn/sitemaps/brands.xml (qua robots.txt → /sitemap.xml; file này có 83 URL = 71 `/brands/{slug}` + 12 `/solution/{slug}`; không có `/brands.xml` ở gốc, trả 404) · số URL cùng loại: 71 · ngày khảo sát 2026-11-11

Cách đo: `curl -sSL -m 180 -A "Mozilla/5.0"` lấy HTML thô, phân tích bằng `python3 -I` trên file đã lưu; 4 request tới URL này (HTML thô, nén zstd, `Accept: text/markdown`, `.md`). Mẫu chỉ có 1 thương hiệu (Protopixel, 2 sản phẩm): các phần phụ thuộc số lượng sản phẩm ghi ở mục 10.

## 1. Response & rendering

### 1.1 Header phản hồi

| Header | Giá trị |
|---|---|
| status | 200 (HTTP/2) |
| content-type | `text/html; charset=utf-8` |
| cache-control | `private, no-cache, no-store, max-age=0, must-revalidate` |
| x-nextjs-cache | không có |
| x-nextjs-prerender | không có |
| x-nextjs-stale-time | không có |
| vary | `rsc, next-router-state-tree, next-router-prefetch, next-router-segment-prefetch, Accept-Encoding` |
| cf-cache-status | `DYNAMIC` |
| link | có: 4 `rel=preload as=font` (`/_next/static/media/051742360c26797e-s.p.1bkzbscqrt8rl.woff2`, `2bbe8d2671613f1f-s.p.0k62hbripvv8p.woff2`, `83afe278b6a6bb3c-s.p.2bn3s6zvc0dyp.woff2`, `fa39153a3fc630ba-s.p.36fqqi66-3tjb.woff2`) + 2 `rel=preload as=style` (`/_next/static/chunks/1mmmxvxvvtkv8.css`, `1vbr8oh4939ca.css`) |
| x-robots-tag | không có |
| content-encoding | không có khi không gửi `Accept-Encoding`; `zstd` khi gửi `gzip, br, zstd` |
| content-length | không có |
| header khác | `server: cloudflare`, `speculation-rules: "/cdn-cgi/speculation"`, `strict-transport-security`, `x-frame-options: SAMEORIGIN`, `referrer-policy: same-origin`, `x-content-type-options: nosniff` |

### 1.2 Thời gian tải

| Lần | Điều kiện | Bytes nhận | TTFB (s) | Tổng (s) |
|---|---|---|---|---|
| 1 | không gửi Accept-Encoding | 324,526 | 0.40 | 0.50 |
| 2 | `gzip, br, zstd` → zstd | 34,320 | 0.51 | 0.55 |
| 3 | `Accept: text/markdown` | 324,526 | không ghi | 0.90 |

Không thấy hiện tượng tải chậm bất thường ở 3 lần đo.

### 1.3 Chế độ render suy ra: dynamic SSR theo request, có streaming Suspense

Bằng chứng:
- `cache-control: private, no-cache, no-store, max-age=0, must-revalidate`; không có `x-nextjs-cache`, `x-nextjs-prerender`, `x-nextjs-stale-time` (trang chủ có cả ba).
- Font và CSS preload nằm trong header `Link` (trang chủ không có `Link` header).
- Cây route trong payload: `(site)/brands/[slug]/__PAGE__`, tham số `["slug","protopixel","d",[]]` (segment động; không có route group `(b2b)` như trang chủ và /solution).
- Streaming: trong HTML có `<!--$?--><template id="B:0"></template><!--/$-->` (placeholder Suspense) và `<div hidden id="S:0">` chứa nội dung được đẩy vào sau, kèm `<script>$RC("B:0","S:0")</script>` ở cuối body.

### 1.4 Hành vi hiện tại với yêu cầu Markdown (chỉ ghi kết quả)

| Request | Status | content-type | Ghi chú |
|---|---|---|---|
| `GET /brands/protopixel` + `Accept: text/markdown` | 200 | `text/html; charset=utf-8` | body 324,526 B, giống bản thường; `vary` không có `Accept` |
| `GET /brands/protopixel.md` | 404 | `text/html; charset=utf-8` | body 130,943 B (trang 404 HTML) |

## 2. `<head>`

Kích thước `<head>`: 6,252 B (2 thẻ preload ảnh: 2,690 B). Font không preload trong `<head>` mà qua header `Link`.

| Thành phần | Giá trị |
|---|---|
| `<html>` | `lang="vi"`, class `onest_…__variable inter_…__variable jetbrains_mono_…__variable` |
| title (61 ký tự) | `ProtoPixel – Hệ thống điều khiển LED SPI và Matrix \| KNXStore` |
| meta description (154 ký tự) | `ProtoPixel cung cấp nền tảng điều khiển LED chuyên nghiệp: mapping ánh sáng, automation, tích hợp đa giao thức (DMX, KNX). Phù hợp dự án smart lighting &…` (kết thúc bằng ký tự `…`, cắt giữa cụm) |
| meta author / keywords | `KNX Store` / `Protopixel` |
| canonical | `https://knxstore.vn/brands/protopixel` |
| meta robots | không có |
| OG | `og:title` (= title), `og:description` (= description), `og:url` (= canonical), `og:image` `https://api.knxstore.vn/media/09e11967b9d1d5a9168fe4970c1ec3732bd6210fedc5a5ac32b24a5a6bb61433.jpg` (ảnh logo thương hiệu), `og:type` `website`. Không có `og:site_name`, `og:locale`, `og:image:width/height/alt` |
| Twitter | `twitter:card` `summary_large_image`, `twitter:title`, `twitter:description`, `twitter:image` (cùng ảnh logo) |
| hreflang / `link rel=alternate` | không có |
| khác | `theme-color` `#14467C`; `next-size-adjust` rỗng; `link[rel=manifest]`; 3 icon (favicon.ico, icon.svg, apple-touch-icon) giống trang chủ |
| preload | `/images/logo/knxstore-logo-tight.svg`; 2 `link[rel=preload][as=image][imageSrcSet]` (ảnh 2 card sản phẩm, `14407cfc….jpg`, `4dca2cfd….jpg`); script `3ivcc6t1s04dh.js` (`fetchpriority=low`); `https://www.googletagmanager.com/gtag/js?id=G-4D9XLJDVGG` |
| CSS / script | 2 `stylesheet` (`1mmmxvxvvtkv8.css`, `1vbr8oh4939ca.css`); 11 `<script async src>` + 1 `nomodule` |

## 3. Khung trang

### 3.1 Thứ tự con trực tiếp của `<body>`

| # | Phần tử | Ghi chú |
|---|---|---|
| B0 | `div[hidden]` chứa `<!--$--><!--/$-->` | marker Suspense rỗng |
| B1 | `<script>` inline ghi cookie `knx-area=b2b` | giống trang chủ |
| B2 | `a[href=#main-content]` | skip link "Bỏ qua, tới nội dung chính" |
| B3 | `header.sticky.top-0.z-9999.bg-white` | 128,755 B |
| B4 | `div#main-content[tabindex=-1]` | vùng chính; không có `<main>` |
| B5 | `div.fixed.top-0.left-0.z-99999…translate-x-full[inert]` | drawer "Yêu cầu báo giá" |
| B6 | `footer.mt-auto` | 17,608 B |
| B7 | `a[data-floating-zalo]` | nút Zalo nổi |
| B8 | `<script>` (payload RSC, chunk) | |
| B9 | `div#S:0[hidden]` (cuối body, sau script) | toolbar lọc/sắp xếp, xem mục 4 |
| B10 | `<script>$RC("B:0","S:0")</script>` | hoán đổi `template#B:0` bằng `div#S:0` ở client |

### 3.2 Landmark và role

| Loại | Chi tiết |
|---|---|
| `<header>` / `<footer>` | 1 / 1 |
| `<main>` | 0 (nội dung chính nằm trong `div#main-content`, không có `role=main`) |
| `<aside>` | 1: `aside[aria-label="Bộ lọc"]`, nằm trong `div#S:0` (hidden) → `div#filter-panel-drawer[role=dialog][aria-hidden=true]` |
| `<nav>` (10) | header 6 (giống trang chủ), footer 3, breadcrumb 1 (`nav[aria-label=Breadcrumb]`) |
| `role` | `search` 1, `combobox` 1 (`input#b2b-header-search`), `group` 1, `region` 1 (`#b2b-trending-panel`), `dialog` 1 (filter drawer) |

### 3.3 Header, footer, thành phần nổi

Header và footer giống hệt byte-by-byte trang chủ và /solution/giai-phap-khac (md5 trùng). Tóm tắt (chi tiết đầy đủ ở `01-home.md` mục 3):

- Header: thanh trên gồm `button[aria-label="Mở menu"]` (mobile), logo `/`, `form[role=search]` với `input#b2b-header-search` (combobox, placeholder "Tìm theo SKU, tên sản phẩm, hãng, giao thức...") và chip phạm vi tìm kiếm (Tất cả, Sản phẩm, Bài viết, Thương hiệu, Giải pháp, Danh mục, Visual), giỏ `a[href=/cart][aria-label="Giỏ báo giá, 0 sản phẩm"]`, CTA `/contact` "Tư vấn dự án". Thanh nav (chỉ hiện từ `lg`): 6 button (Trending, Danh mục sản phẩm, Giải pháp, Thương hiệu, Công cụ, Blog kỹ thuật) + link Visual `/visual` + pill "Matter Smarthome" `/matter`. 6 panel mega menu có sẵn trong HTML với class `hidden`: trending (8 từ khoá, 16 link sản phẩm, 5 bài), danh mục (12 nhóm, 59 `<a>`), giải pháp (12 mục ×2), thương hiệu (24 logo + "Xem tất cả 75 thương hiệu"), công cụ (3), blog (9 chuyên mục + 9 bài). Không có hotline trong header.
- Footer: thông tin công ty (CÔNG TY CP TÍCH HỢP HỆ THỐNG LIÊN MINH, MST 0311846236, địa chỉ The Sun Avenue, `tel:0918918755`, `mailto:support@knxstore.vn`, Facebook, Zalo, 2 huy hiệu online.gov.vn/DMCA), 3 cột link (Danh Mục Sản Phẩm 7, Giải Pháp 7, Về chúng tôi 5) render 2 lần (mobile accordion + desktop), thanh dưới với © 2026 và 4 link chính sách/bảo hành.
- Thành phần nổi: nút Zalo `a[data-floating-zalo][href=https://zalo.me/0918918755]` (52x52, `#0068ff`, `animate-ping`); drawer báo giá `div.fixed…[inert]` (trạng thái rỗng: "Chưa có thiết bị nào trong yêu cầu.", "Tổng số lượng:0"); drawer bộ lọc `div#filter-panel-drawer` + overlay `div.fixed.inset-0.z-99999[aria-hidden=true]` (trong `div#S:0`). Không thấy cookie banner, popup, back-to-top, widget chat khác trong HTML ban đầu.

## 4. Cây nội dung vùng chính

Vùng chính: `div#main-content[tabindex=-1]` (217 ký tự text) + `div#S:0` (267 ký tự) ở cuối body. Theo thứ tự DOM:

| # | Thẻ + id/class (rút gọn) | Heading | Nội dung tóm tắt | Trong `div#main-content` | Hiển thị mặc định | SSR |
|---|---|---|---|---|---|---|
| 0 | `script[type=application/ld+json]` (con đầu) | | `@graph` 4 node (mục 8) | có | không hiển thị | có |
| 1 | `div.bg-white > div.mx-auto > nav[aria-label=Breadcrumb]` | | "Trang chủ" (`a[/]`) / "Thương hiệu" (`a[/brands]`) / "Protopixel" (`span`, hiện tại) | có | có | có |
| 2 | `div.py-4.lg:py-5` (khối tên thương hiệu) | h1 "Protopixel" (`h1.text-h1-mobile`) | `span.tabular-nums` "2 sản phẩm"; logo `span.flex.h-16.w-28 > img[alt=Protopixel][width=112][height=48]` (`/media/09e11967….jpg`, lazy); `div.min-w-0.flex-1` ngay cạnh logo là div rỗng (prop `children:false` trong payload) | có | có | text có; không có đoạn mô tả thương hiệu nào |
| 3 | `template#B:0` | | placeholder Suspense (fallback `null`), trống | có | không hiển thị | có (rỗng) |
| 4 | `div.bg-white > div.mx-auto > div.min-w-0 > div.grid.grid-cols-2…lg:grid-cols-4` | không | 2 `article` card sản phẩm (bảng bên dưới) | có | có | có |
| 5 | `div#S:0[hidden]` > `div.bg-white > div.mx-auto > div.flex.flex-wrap.items-center.justify-between…border-t` | h2 "Bộ lọc" (trong drawer) | toolbar lọc/sắp xếp (bên dưới) | không: nằm ở cuối `<body>`, sau footer/script | ẩn (`hidden`) trong HTML thô; sau `$RC` được chuyển vào vị trí `template#B:0` | có text trong HTML thô (ở `div#S:0`) |

Card sản phẩm (cùng cấu trúc card ở trang chủ, mục 5.2 của `01-home.md`; ảnh eager `priority`, không lazy; 2 ảnh được preload trong `<head>`):

| # | Hãng (badge) | Model | Tên (link) | Giá hiển thị | URL |
|---|---|---|---|---|---|
| 1 | Protopixel | Gate Casambi | Bộ điều khiển Gate Casambi Protopixel – Gate Casambi | Liên hệ | `/products/bo-dieu-khien-gate-casambi-protopixel-gate-casambi` |
| 2 | Protopixel | Brain Experience | Bộ điều khiển trung tâm Brain Experience Protopixel | Liên hệ | `/products/bo-dieu-khien-trung-tam-brain-experience-protopixel` |

Toolbar `div#S:0` (nội dung):
- `button` "Bộ lọc" kèm badge số `0` (số bộ lọc đang áp).
- Drawer `div#filter-panel-drawer[role=dialog][aria-hidden=true]` (h2 "Bộ lọc", nút "Đóng") chứa `aside[aria-label="Bộ lọc"]`: checkbox "Chỉ hàng thanh lý" (mô tả: "Sản phẩm đang bán giá thanh lý, hoặc còn suất hàng demo, trôi bảo hành, đã qua sử dụng."); `details[open]` "Danh mục" với 1 `button` "Bộ điều khiển trung tâm" (2); `details[open]` "Giải pháp" với 2 `input[type=radio][name="Giải pháp"]` Casambi (1), DALI (1); nút "Xoá lọc", "Áp dụng". Payload cho thấy `facets: []`, `brands: []` (không có facet thương hiệu/thuộc tính).
- `label > select` "Sắp xếp" 6 `option`: Mặc định, Mới nhất, Tên A → Z, Tên Z → A, Giá thấp → cao, Giá cao → thấp.
- Bộ lọc dùng `button`/`input`, không có `<a>` (0 link trong `div#S:0`).

Phân trang: không thấy phần tử phân trang trong HTML hoặc payload (0 kết quả cho "phân trang", "pagination", "Tải thêm", "?page", "totalPages", "hasMore"). Với 2 sản phẩm, không xác định được hành vi khi có nhiều sản phẩm (mục 10).

## 5. Chi tiết khối đặc biệt

### 5.1 Hoán đổi Suspense (`template#B:0` và `div#S:0`)

```html
...</div></div></div></div><!--$?--><template id="B:0"></template><!--/$--><div class="bg-white"><div class="mx-auto … xl:max-w-site"><div class="min-w-0"><div class="grid grid-cols-2 …"><article …>
...(footer, nút Zalo, script)...
<div hidden id="S:0"><div class="bg-white"><div class="mx-auto …"><div class="flex flex-wrap items-center justify-between gap-3 border-t border-zinc-200 py-4">…<button type="button" …>Bộ lọc<span>0</span></button>…
<script>…$RC("B:0","S:0")</script>
```

Thứ tự trong HTML thô: toolbar lọc/sắp xếp (khối bên trên lưới sản phẩm khi hiển thị) nằm SAU footer và nút Zalo trong thứ tự DOM; thứ tự hiển thị chỉ đúng sau khi `$RC` chạy.

### 5.2 Khối tên thương hiệu (không có mô tả)

```html
<div class="py-4 lg:py-5"><div class="flex flex-wrap items-baseline gap-x-4 gap-y-1"><h1 class="text-h1-mobile text-zinc-900">Protopixel</h1><span class="text-small tabular-nums text-zinc-500">2<!-- --> sản phẩm</span></div><div class="mt-3 flex gap-4 items-start"><span class="flex h-16 w-28 shrink-0 …"><img alt="Protopixel" … class="h-12 w-auto max-w-full object-contain"/></span><div class="min-w-0 flex-1"></div></div></div>
```

Mô tả thương hiệu chỉ xuất hiện trong `<meta name=description>`, OG/Twitter và JSON-LD (`CollectionPage.description`, `Brand.description`); không có trong DOM hiển thị và không có trong payload RSC ngoài metadata và JSON-LD.

### 5.3 Card sản phẩm

Cùng cấu trúc `article.group.relative.rounded-card…` như trang chủ: `div.aspect-square > img`, hàng badge hãng (`span` gradient `#D9EAFF→#FFFFFF`) + `span.model-code`, `a.after:absolute.after:inset-0` (tiêu đề, là `<a>` chứ không phải heading), `div` giá, `button[aria-label="Thêm … vào yêu cầu báo giá"]`.

### 5.4 Cấu trúc payload RSC

- 1 lần `push([1,"…"])` (+ 1 `push([0])` khởi tạo), giải mã 131,034 B, 76 row (75 JSON + 1 `T` là JSON-LD 4,443 B).
- 2 row layout (id `5` và `11`, 29,467 B và 29,468 B) mang prop `nav` của `HeaderB2b` (dữ liệu trending/menu), tổng 58,935 B (45% payload giải mã).
- Row id `6` (4,550 B) là vùng chính: breadcrumb, khối tên, `Suspense` (`$8`, `fallback: null`) bọc component client `$L41` với prop `filter`/`chips`, rồi lưới card.
- Row id `15` (2,580 B) là cây 404 ("Không tìm thấy trang", "Trang này không tồn tại… catalogue có 753 thiết bị"), không render ra DOM.
- Row id `e` là metadata (title, meta, OG, icon).

## 6. Nội dung chỉ có ở client hoặc KHÔNG thấy trong HTML ban đầu

| # | Nội dung | Có trong HTML ban đầu? | Bằng chứng |
|---|---|---|---|
| 1 | Toolbar lọc/sắp xếp và drawer bộ lọc | Có text nhưng ở `div#S:0[hidden]` cuối `<body>`; vị trí hiển thị chỉ đúng sau `$RC("B:0","S:0")` | `template#B:0` rỗng tại chỗ; `div hidden id="S:0"` |
| 2 | Mô tả thương hiệu | Không có trong DOM hiển thị (chỉ có trong `<head>` và JSON-LD) | `div.min-w-0.flex-1` rỗng; `children:false` trong payload |
| 3 | Kết quả lọc/sắp xếp, danh sách sau khi áp bộ lọc, cơ chế cập nhật URL/trạng thái | Không | bộ lọc là `button`/`input`, không có `<a>`; cơ chế: Chưa xác định (không chạy JS) |
| 4 | Gợi ý tìm kiếm | Không | `input#b2b-header-search[aria-expanded=false]`, không có `listbox` |
| 5 | Trạng thái giỏ báo giá | Chỉ trạng thái rỗng | "Giỏ báo giá, 0 sản phẩm"; "Chưa có thiết bị nào trong yêu cầu." |
| 6 | Menu mobile (sau nút "Mở menu") | Không | chỉ có `button[aria-label="Mở menu"]` |
| 7 | Cây 404 | Không (chỉ trong payload RSC) | row id `15` |
| 8 | Các trường sản phẩm không render (`stockStatus`, `oldPrice`, `b2c`, `b2b`, `priceHidden`…) | Không | payload RSC (xuất hiện trong prop card) |
| 9 | Script GA4 | Không (chỉ preload) | `gtag/js?id=G-4D9XLJDVGG` |

Mega menu header (6 panel `hidden`) có text và link đầy đủ trong HTML ban đầu. Không thấy skeleton: 0 `<template data-dgst`, 0 `BAILOUT_TO_CLIENT_SIDE_RENDERING`, 0 `animate-pulse`; `<template id="B:0">` là placeholder Suspense, không phải skeleton có nội dung.

## 7. Liên kết

Tổng 223 `<a>`: skip link 1, header 165, `div#main-content` 4, drawer 2, footer 50, nút Zalo 1.

| Vùng | Mẫu URL nội bộ |
|---|---|
| `div#main-content` (4; 4 href duy nhất) | `/` (breadcrumb), `/brands` (breadcrumb), `/products/{slug}` ×2 |
| header | như trang chủ: `/{category-slug}` 62, `/solution/{slug}` 24, `/brands/{slug}` 24, `/products/{slug}` 16, `.html` 16 (14 bài blog + 2 `/tool/…/index.html`), `/blogs[/{cat}]` 10, `/search?q=` 8, còn lại 1 mỗi loại (`/`, `/products`, `/solution`, `/brands`, `/tools/hvac-compatibility`) |
| footer | như trang chủ (50 `<a>`, 31 href duy nhất; 5 link ngoài: Google Maps, Facebook, Zalo, online.gov.vn, DMCA) |

- Link ngoài trong vùng chính: 0. Nút Zalo nổi: `https://zalo.me/0918918755`.
- Link rel=alternate/markdown: không có.
- Danh sách sản phẩm của thương hiệu chỉ có 2 link `/products/{slug}`; cũng được liệt kê trong JSON-LD `ItemList`.
- Breadcrumb: link tới `/brands` (danh sách thương hiệu), không thuộc phạm vi khảo sát này.

## 8. JSON-LD

1 thẻ `script[type=application/ld+json]` (4,244 ký tự JSON, 4,478 B kèm thẻ), con đầu của `div#main-content`. Một `@graph` gồm 4 node:

| # | `@type` | `@id` | Trường có mặt |
|---|---|---|---|
| 0 | `["Organization","LocalBusiness"]` | `https://knxstore.vn/#organization` | giống trang chủ: `name`, `legalName`, `url`, `foundingDate`, `taxID`, `logo` (`https://knxstore.vn/assets/logo.png` 150x50), `image` (1200x630), `description`, `telephone`, `email`, `address`, `geo`, `hasMap`, `openingHoursSpecification` (Mon–Sat 09:00–17:30), `priceRange`, `areaServed`, `knowsAbout` (10), `contactPoint`, `sameAs` (Facebook, YouTube) |
| 1 | `CollectionPage` | `https://knxstore.vn/brands/protopixel#webpage` | `url`, `name` "Protopixel", `description` (cắt ở "…tích hợp đa giao thức", không có "…" và khác meta description), `inLanguage` `vi-VN`, `isPartOf` (ref `https://knxstore.vn/#website`), `publisher` (ref `#organization`), `mainEntity` (ref `…#products`), `about` (ref `…#about`), `primaryImageOfPage` (ImageObject `…#primary-image`, `url`/`contentUrl` = ảnh logo `api.knxstore.vn/media/09e11967….jpg`, `caption` "Protopixel"), `breadcrumb` (BreadcrumbList 3 `ListItem`: Trang chủ → `https://knxstore.vn/`, Thương hiệu → `https://knxstore.vn/brands`, Protopixel → URL trang) |
| 2 | `Brand` | `https://knxstore.vn/brands/protopixel#about` | `name`, `description` (như trên), `url`, `logo` (chuỗi URL ảnh, trùng `og:image`) |
| 3 | `ItemList` | `https://knxstore.vn/brands/protopixel#products` | `name` "Sản phẩm Protopixel tại KNXStore", `numberOfItems` 2, `itemListElement` 2 `ListItem` (`position`, `url`) |

Ghi nhận:
- `isPartOf` tham chiếu `https://knxstore.vn/#website` nhưng graph của trang này không có node `WebSite` với `@id` đó (trang chủ mới có).
- `ListItem` trong `ItemList` chỉ có `position` và `url`; không có `name`, `image`, `offers`. Không có node `Product`.
- `Brand` không có `sameAs`, `slogan`; không có `datePublished`/`dateModified` trên `CollectionPage`.
- Không có trường rỗng.

## 9. Số đo

| Chỉ số | Giá trị |
|---|---|
| HTML thô | 324,526 B |
| gzip (đo local, mức 6 / 9) | 55,039 B / 54,877 B |
| zstd phía server | 34,320 B |
| brotli | Chưa xác định (không đo được) |
| `<head>` | 6,252 B |
| `<body>` | 318,099 B |
| `<header>` | 128,755 B (39.7% HTML) |
| `<footer>` | 17,608 B |
| Tổng `<script>` (19 thẻ) | 150,541 B (46.4%) |
| Script inline `self.__next_f` (2 thẻ) | 143,976 B; giải mã 131,034 B |
| Script `ld+json` (1) | 4,478 B |
| `<style>` | 0 |
| Thuộc tính `style=""` (31) | 1,147 B |
| `<svg>` inline (40) | 14,672 B |
| Thuộc tính `srcset/srcSet` (27) | 48,536 B |
| Thuộc tính `class` (754) | 66,624 B |
| HTML sau khi bỏ mọi `<script>` | 173,985 B |
| Text hiển thị `<body>` | 5,696 ký tự / 7,084 B |
| Text `div#main-content` | 217 ký tự / 250 B |
| Text `div#S:0` | 267 ký tự |
| Text ngoài vùng chính (header, footer, drawer, Zalo, skip link) | 5,212 ký tự (91.5% text hiển thị) |
| Ước lượng token (bytes/4) | HTML thô 81,131; text hiển thị 1,771; text vùng chính 62 |
| Thẻ | `<a>` 223, `<img>` 73 (69 lazy, 4 eager; 42 `alt=""`), `<svg>` 40, `<table>` 0, `<button>` 24, `<form>` 1, `<input>` 4 (search, checkbox, 2 radio), `<select>` 1, `<article>` 2, `<section>` 0, `<nav>` 10, `<aside>` 1, `<ul>` 33, `<li>` 176, `<details>` 5 (3 footer + 2 trong bộ lọc), `<template>` 1 |
| Heading | h1 ×1, h2 ×5, h3 ×18, h4–h6 ×0 |

Phân bố heading: h1 "Protopixel" trong `div#main-content`; h2: "Yêu cầu báo giá" (drawer), 3 h2 footer (Danh Mục Sản Phẩm, Giải Pháp, Về chúng tôi), "Bộ lọc" (trong `div#S:0`); 18 h3 trong mega menu header. Card sản phẩm không có heading.

## 10. Chưa xác định / lưu ý

Chưa xác định:
- Phân trang, "tải thêm" và giới hạn số card cho thương hiệu có nhiều sản phẩm: mẫu chỉ có 2 sản phẩm và chưa tải thương hiệu khác (ngoài phạm vi).
- Cách bộ lọc/sắp xếp cập nhật danh sách (client hay điều hướng URL): không chạy JS.
- Cách hiển thị khi thương hiệu có mô tả: mẫu này không có mô tả hiển thị; chưa biết thương hiệu khác có.
- Cách hiển thị khi thương hiệu không có logo (trên trang chủ marquee, 47/75 hãng chỉ hiện chữ).
- Dung lượng brotli; menu mobile; nội dung inject sau hydrate.

Lưu ý về tính đại diện:
- Số card (2) và số `<img>`/bytes của vùng chính phụ thuộc vào thương hiệu; header, footer và payload layout giống nhau ở mọi trang.
- Trên trang này vùng chính chiếm 217 ký tự trong tổng 5,696 ký tự text hiển thị.
- Không có `<main>`; khi trích xuất theo landmark sẽ không bắt được vùng chính, `div#main-content` là mốc duy nhất.
- Toolbar lọc ở cuối `<body>` (ẩn) trong HTML thô.
- Title dùng "ProtoPixel" còn H1 và JSON-LD `name` dùng "Protopixel".
- Brand trong marquee trang chủ có 4 slug (`1home`, `euchips`, `lichtvision-probitsite`, `tongdy`) không thuộc 71 URL của `sitemaps/brands.xml`.
- Route của trang không thuộc group `(b2b)`, nhưng header/footer vẫn là bản B2B (cookie `knx-area=b2b`).
