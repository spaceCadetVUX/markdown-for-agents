# Landing Matter (B2C) — cấu trúc hiện tại

> URL mẫu: https://knxstore.vn/matter · nguồn sitemap: pages.xml (thực tế `https://knxstore.vn/sitemaps/pages.xml`, 19 URL) · số URL cùng nhóm: 3 (`/matter`, `/matter/categories`, `/matter-search`; chỉ khảo sát `/matter`) · ngày khảo sát 2026-11-11

## 1. Response & rendering

| Header | Giá trị |
|---|---|
| status | 200 |
| content-type | `text/html; charset=utf-8` |
| cache-control | `s-maxage=3600, stale-while-revalidate=31532400` |
| x-nextjs-cache | `STALE` ở request đầu tiên; `HIT` ở các request sau (Accept markdown, gzip, br) |
| x-nextjs-prerender | `1` |
| x-nextjs-stale-time | `300` |
| vary | `rsc, next-router-state-tree, next-router-prefetch, next-router-segment-prefetch, Accept-Encoding` |
| cf-cache-status | `DYNAMIC` |
| link | Không có |
| x-robots-tag | Không có |
| content-encoding | `br` (gửi `gzip, br`); `gzip` (chỉ gửi `gzip`); không nén khi không gửi Accept-Encoding |

**Chế độ render suy ra: ISR (prerender tĩnh + revalidate).** Bằng chứng: `x-nextjs-prerender: 1`; `x-nextjs-cache` chuyển `STALE` → `HIT`; `cache-control: s-maxage=3600, stale-while-revalidate=31532400` (revalidate 3600 s); Cloudflare không cache (`DYNAMIC`) nên cache nằm ở Next. Dữ liệu sản phẩm/bài viết lấy từ backend (ví dụ `og:image` trỏ `api.knxstore.vn`); thời điểm lấy dữ liệu (build hay runtime): Chưa xác định.

Request phụ (chỉ ghi kết quả):

| Request | Status | content-type | Ghi chú |
|---|---|---|---|
| `Accept: text/markdown` → `/matter` | 200 | `text/html; charset=utf-8` | body 551,956 B; `x-nextjs-cache: HIT` |
| `/matter.md` | 404 | `text/html; charset=utf-8` | trang not-found của Next, 131,133 B; `cache-control: private, no-cache, no-store, max-age=0, must-revalidate` |

Kích thước: HTML thô 551,978 B (request 1, STALE) / 551,956 B (các request sau); wire `br` 86,123 B; wire `gzip` 95,102 B; gzip level 6 đo cục bộ 95,843 B. Khác biệt 22 B giữa 2 response chỉ nằm ở router state trong RSC payload: request STALE chứa `"c":["","matter?_rsc=uLDbtvszpF-zVyfK"]`, request HIT chứa `"c":["","matter"]`.

## 2. `<head>`

| Thành phần | Giá trị |
|---|---|
| html | `lang="vi"`; class chứa 3 biến font (`onest_*`, `inter_*`, `jetbrains_mono_*`) |
| title | `Thiết bị nhà thông minh Matter chính hãng \| KNXStore.vn` |
| meta description | `Công tắc, động cơ rèm, cảm biến, ổ cắm, hub chuẩn Matter chính hãng. Tương thích Apple Home, Google Home, Alexa. Tư vấn miễn phí, giao hàng toàn quốc.` |
| canonical | `https://knxstore.vn/matter` |
| robots | Không có thẻ meta robots |
| Open Graph | `og:title`, `og:description` (trùng title/description), `og:url=https://knxstore.vn/matter`, `og:site_name=knxstore.vn`, `og:locale=vi_VN`, `og:type=website`, `og:image=https://api.knxstore.vn/media/0119edf7565dace9f515219397416e8c9d63e737c562aa55be779fd2d44db72f.avif` |
| Twitter | `summary_large_image`, title, description, image (cùng ảnh avif) |
| hreflang / rel=alternate | Không có |
| preload | 4 font woff2 (`/_next/static/media/*.woff2`); `/images/logo/knxstore-logo-tight.svg`; 9 ảnh (`imagesrcset` qua `/_next/image`: 1 ảnh hero w=640... và 8 ảnh sản phẩm w=256...); script `/_next/static/chunks/3ivcc6t1s04dh.js` (fetchpriority low); `https://www.googletagmanager.com/gtag/js?id=G-4D9XLJDVGG` |
| stylesheet | 2: `/_next/static/chunks/1mmmxvxvvtkv8.css`, `1vbr8oh4939ca.css` |
| khác | `theme-color #14467C`; `link rel=manifest /manifest.webmanifest`; icon `favicon.ico`, `icon.svg`, `apple-icon.png`; `meta name=next-size-adjust` |

## 3. Khung trang

Thứ tự con trực tiếp của `<body>`: `div[hidden]` (chỉ chứa comment `<!--$--><!--/$-->`) → `div.font-sans[style="--font-body:var(--font-matter)"]` → 8 thẻ `<script>` cuối. Trong `div.font-sans`:

| # | Phần tử | Nội dung | Ghi chú |
|---|---|---|---|
| 1 | `script` inline (100 B) | `document.cookie="knx-area=matter; Path=/; SameSite=Lax"+(https? "; Secure")` | Khu vực "matter"; payload RSC (boundary not-found) chứa thêm script tương tự đặt `knx-area=b2b` |
| 2 | `header.sticky.left-0.top-0.z-9999[aria-label="Khu Matter Smarthome"]` (17 link) | Logo `a[href=/matter][aria-label="KNXStore — Matter Smarthome"]` (img `/images/logo/knxstore-logo-tight.svg` + chữ "matter"); `nav[aria-label="Menu Matter"]` có nút "Danh mục sản phẩm" (`aria-expanded=false`, `aria-controls=matter-mega-menu`) mở `div#matter-mega-menu` (4 cột, 12 link: Công tắc, Cảm biến, Camera, Động cơ rèm, Ổ cắm, Thermostat, Driver Matter, Gateway Matter, Hub trung tâm, Smart hub Homey, Khóa cửa, Chuông hình), + link "Blogs" `/blogs`, "Liên hệ" `/contact`; "Trang B2B" `/`; nút tìm kiếm; link giỏ `a[href=/cart][aria-label="Giỏ báo giá, 0 sản phẩm"]`; nút mở menu mobile | trong header còn `div#matter-search-drop[aria-hidden=true]` chứa `form[role=search]` với `input#matter-search-input[type=search][name=q][placeholder="Tìm công tắc, cảm biến, hub Matter…"]` và `label.sr-only` "Tìm thiết bị Matter" |
| 3 | `div.fixed.inset-0.z-[10000].overflow-hidden` > `aside#matter-mobile-panel[aria-label="Menu khu Matter"][aria-hidden=true]` | Menu mobile (15 link, lặp lại mega menu bằng `details.group/mcat`, `details.group/mnav` × 2) | ẩn mặc định |
| 4 | `main` | xem mục 4 | 62 link, 2,491 ký tự text |
| 5 | `div[inert].fixed.top-0.left-0.z-99999.translate-x-full` | Drawer "Yêu cầu báo giá": h2 "Yêu cầu báo giá", nút đóng, "Chưa có thiết bị nào trong yêu cầu.", link "Xem sản phẩm" (`/products`), "Tổng số lượng:0", link "Xem và gửi yêu cầu" (`/cart`) | `inert`, nằm ngoài màn hình; nội dung phụ thuộc trạng thái client |
| 6 | `footer.mt-auto.border-t.border-[#e5e7eb].bg-white` (21 link) | Logo; 3 `h2`: "Thông tin liên hệ", "Danh mục sản phẩm", "Về chúng tôi"; MST `0311846236`...; địa chỉ; `tel:0378628837`; `https://zalo.me/0378628837` "Tư vấn qua Zalo"; `mailto:support@knxstore.vn`; Facebook; 2 ảnh badge (online.gov.vn, DMCA); 7 link danh mục (`/hub-trung-tam` "Hub trung tâm", `/dieu-khien-dan-lanh` "thermostat", `/cam-bien-matter`, `/cong-tac-matter` "Côn tắc thông minh", `/dong-co-rem-matter`, `/driver-matter` "Nguồn đèn Matter", `/camera-thong-minh`); `/contact`, `/` "Thiết bị tự động hoá toà nhà", `/blogs`; thanh dưới: © 2026..., `/warranty-lookup`, `/chinh-sach-bao-mat.html`, `/chinh-sach-mua-hang-tai-knxstore-vn.html`, `/dieu-khoan-su-dung-website-knxstore-vn.html` | |
| 7 | `a.group.fixed.right-[26px].bottom-[calc(8.75rem+env(safe-area-inset-bottom))][aria-label="Chat Zalo với KNXStore"]` | `href=https://zalo.me/0378628837`, text "Zalo" | nút Zalo nổi |

- Landmark: `header` 1, `nav` 2 (`Menu Matter`, `Menu Matter trên điện thoại`), `main` 1, `aside` 1, `footer` 1, `form[role=search]` 1. Không có role=dialog/alert.
- **Header/footer của /matter là bộ riêng của khu Matter (B2C), khác chrome B2B.** Bằng chứng trong payload RSC của chính trang này: boundary not-found (rows `10`, `14`–`2a`) chứa header B2B (dữ liệu nav "Trending"...) và footer B2B `footer.mt-auto.border-t.border-zinc-200` với Zalo `0918918755` và link `/solution/*`; trong khi DOM của `/matter` dùng footer `border-[#e5e7eb]` với Zalo/tel `0378628837` và link "Trang B2B" quay về `/`. Chưa lấy DOM của một trang B2B khác để so sánh trực tiếp.
- Thành phần nổi: nút Zalo nổi (mục 7 bảng trên), drawer báo giá (inert), Google Analytics `G-4D9XLJDVGG` (component `GoogleAnalytics` trong payload + preload gtag). Không thấy cookie banner/popup trong HTML.

## 4. Cây nội dung trong `<main>` (theo thứ tự DOM)

`<main>` có 7 con trực tiếp. **Toàn bộ nội dung bên dưới có trong HTML ban đầu (SSR/prerender).** Mọi thẻ nằm trong `<main>`.

| # | Thẻ (rút gọn) | Heading | Nội dung tóm tắt | Hiển thị mặc định | SSR/client |
|---|---|---|---|---|---|
| 1 | `script[type=application/ld+json]` (con đầu tiên của `main`, 5,668 B) | — | JSON-LD (mục 8) | không hiển thị | SSR |
| 2 | `section > div > div.group.relative.overflow-hidden.bg-[#f9fafb]` (hero carousel) | slide 1: `h1.text-[22px]...` "Thiết bị nhà thông minh Matter chính hãng"; slide 2: `h2` "Cảm biến môi trường"; slide 3: `h2` "Công tắc thông minh" | `ul.flex.overflow-x-auto.snap` gồm 3 `li.snap-start.w-full`, mỗi `li` có 1 `img.object-cover` (alt = tiêu đề slide; ảnh 1 không `loading`, ảnh 2-3 `loading="lazy"`), lớp phủ gradient aria-hidden, khối chữ. Slide 1: `span` "KNXStore.vn/matter", `p` "Tương thích Apple Home, Google Home và Alexa. Điều khiển ngay trong app bạn đã quen dùng, không cần cài thêm app mới.", `a[href=/matter-search]` "Khám phá thiết bị Matter". Slide 2: `a[href=/cam-bien-matter]` "Xem thiết bị" (không có p). Slide 3: `p` "Advanced imaging performance with a 200MP AI camera with Enhanced image quality." (tiếng Anh), `a[href=/cong-tac-matter]` "Xem thiết bị". Điều khiển: 2 `button` "Ảnh trước"/"Ảnh sau" (`opacity-0` đến khi hover/focus), 3 `button` "Tới ảnh N" (`aria-current`) | slide 1 hiển thị; slide 2-3 cuộn ngang (scroll-snap) | SSR (3 slide đều có trong HTML); chuyển slide là client |
| 3 | `section.py-10 > div.mx-auto.w-full.max-w-[1280px].border-b` | `h2` "Khám phá theo danh mục" | `div.grid.grid-cols-2` 8 `a.group` (ảnh `alt=""` + tên + "Xem sản phẩm"): `/cong-tac-matter` "Công tắc Matter", `/dieu-khien-dan-lanh` "Điều khiển điều hoà máy lạnh", `/cam-bien-matter` "Cảm biến thông minh Matter", `/hub-trung-tam` "Hub trung tâm", `/gateway-matter` "Gateway Matter", `/driver-matter` "Bộ điều khiển LED Matter", `/camera-thong-minh` "Camera thông minh", `/khoa-thong-minh-matter` "Khóa thông minh Matter"; nút `a[href=/matter/categories]` "Xem tất cả danh mục" | hiển thị | SSR |
| 4 | `section.py-15 > div.mx-auto...px-4` | `h2` "Sản phẩm mới" + `a[href=/matter-search]` "Xem tất cả" | `div.grid.grid-cols-2` **8** `article.group/card` (bảng 4a) | hiển thị | SSR (props client component kèm theo, xem mục 6) |
| 5 | `section.bg-[#f9fafb].py-15` | `h2` "Sản phẩm nổi bật" | **10** `article.group/card` (bảng 4a) + `div.mt-12 > a[href=/matter-search]` "Xem tất cả" | hiển thị | SSR (props client component kèm theo) |
| 6 | `section.py-15 > ... div.relative.overflow-hidden.rounded-[14px].bg-[#0f1b45]` | `h2.text-[20px].font-bold` "Cần tư vấn giải pháp Matter Smarthome?" | `p` "Đội ngũ kỹ thuật KNXStore hỗ trợ chọn thiết bị, thiết kế hệ thống và báo giá theo đúng nhu cầu của bạn."; `a[href=tel:0378628837]` "0378.628.837"; `a[href=https://zalo.me/0378628837]` "Liên hệ qua Zalo" | hiển thị | SSR |
| 7 | `section.py-10` | `h2` "Bài viết mới nhất" + `a[href=/blogs]` "Xem tất cả bài viết" | `div.grid.grid-cols-1.sm:grid-cols-2` **3** `article.group/post` (bảng 4b) | hiển thị | SSR |

Không có `h4`–`h6`. Heading trong `<main>`: h1 ×1, h2 ×7 (2 h2 của slide hero 2-3 + 5 h2 section), h3 ×21 (18 tên sản phẩm + 3 tiêu đề bài). Ngoài `<main>`: 4 h2 (1 ở drawer báo giá "Yêu cầu báo giá", 3 ở footer: "Thông tin liên hệ", "Danh mục sản phẩm", "Về chúng tôi"); tổng toàn trang h2 = 11.

### 4a. Danh sách sản phẩm trong HTML ban đầu (18 thẻ)

Cấu trúc mỗi thẻ: `article.group/card` > `a[aria-hidden=true][tabindex=-1][href=/products/<slug>]` bọc `img[alt=""][data-nimg=fill]` (srcset `/_next/image?...&w=256..1200`) + `div` gồm `span.uppercase` (brand), `h3 > a[href=/products/<slug>]` (tên), `span` giá (`<số>đ`), `button[aria-label="Thêm <tên> vào ..."][type=button]` (thêm vào giỏ báo giá; là client component). Mỗi sản phẩm có 2 link `/products/<slug>` (36 link tổng).

| Mục | Brand | Tên | Giá |
|---|---|---|---|
| Mới 1 | Micro Air | Bảng điều khiển điều hòa Matter MicroAir 81MTAC01, 12 hãng | 3.995.000đ |
| Mới 2 | Remotec | Bộ điều khiển máy lạnh Matter IR Remotec – MXT-300 | 1.890.000đ |
| Mới 3 | Tailwind | Cảm biến ô tô Moko Technology H2 Tailwind | 990.000đ |
| Mới 4 | Tailwind | Bộ điều khiển cửa gara và cổng thông minh Tailwind iQ3 2.1 | 4.900.000đ |
| Mới 5 | JINK | Động cơ rèm thông minh chuẩn Matter Thread JINK - Jink Motor Air | 4.500.000đ |
| Mới 6 | Apple | Loa Apple Homepod Mini | 3.990.000đ |
| Mới 7 | JINK | Công tắc thông minh 4 phím Thread Matter Skin Glass JINK - S1-04 SKIN | 2.850.000đ |
| Mới 8 | JINK | Công tắc thông minh 4 phím Thread Matter Aluminium JINK - S1-04 ALUMI | 2.850.000đ |
| Nổi bật 1 | Eve | Động cơ rèm Apple Home Eve MotionBlinds, Bluetooth/Matter over Thread | 11.086.000đ |
| Nổi bật 2 | Eve | Cảm biến chuyển động Eve Motion, Matter over Thread, IPX3 | 2.772.000đ |
| Nổi bật 3 | Eve | Camera ngoài trời IP55 Eve Outdoor Cam Wi-Fi (2.4 GHz) | 13.857.000đ |
| Nổi bật 4 | Eve | Ổ cắm thông minh Eve Energy Matter over Thread | 2.218.000đ |
| Nổi bật 5 | Homey | Bộ điều khiển trung tâm Homey Pro - Matter & Thread, Z-Wave, Zigbee, Wi-Fi, BLE, IR | 16.900.000đ |
| Nổi bật 6 | Aqara | Núm xoay không dây H2 (EU), Trắng Matter Aqara - KD-R01D-W | 1.028.000đ |
| Nổi bật 7 | Aqara | Đèn LED Bulb T2 (E27, RGB CCT) Matter Aqara - LB-L02D | 542.000đ |
| Nổi bật 8 | JINK | Công tắc thông minh 3 phím Thread Matter JINK - Z75-06 | 2.990.000đ |
| Nổi bật 9 | Micro Air | Bảng điều khiển điều hòa Daikin Matter Wifi S21, P1P2 - 81MTAC01-DAI | 3.995.000đ |
| Nổi bật 10 | Eve | Cảm biến chống ngập nước Eve Water Guard, Matter over Thread | 5.543.000đ |

Trong HTML ban đầu mỗi thẻ chỉ có brand, tên, giá, ảnh (alt rỗng) và link; không có mô tả, SKU, tồn kho, thông số (xem mục 6).

### 4b. Bài viết mới nhất (3 thẻ `article.group/post`)

| Tác giả · ngày | Tiêu đề (`h3 > a`) | URL |
|---|---|---|
| Tạ Minh Vũ · 03/08/2026 | Camera An Ninh Matter Đã Có Chưa? Cập Nhật Sau Matter 1.5 | `/vi-sao-hien-tai-chua-co-camera-an-ninh-tuong-thich-matter.html` |
| Tạ Minh Vũ · 05/08/2026 | Hub Matter Nào Đáng Mua? So Sánh Aqara Hub M3, Homey Pro Và Các Loại Phổ Biến | `/top-7-hub-matter-cho-nha-thong-minh.html` |
| (không tác giả) · 18/12/2025 | Thread Group vượt mốc 1.000 sản phẩm đạt chứng nhận Thread | `/thread-group-vuot-moc-1-000-san-pham-dat-chung-nhan-thread.html` |

Mỗi thẻ có ảnh (`alt=""`), ngày, tiêu đề, link "Đọc tiếp" (3 link/thẻ = 9 link `.html`); không có đoạn trích nội dung bài.

## 5. Chi tiết khối đặc biệt

Thẻ sản phẩm (rút gọn từ HTML thô):

```html
<article class="group/card relative min-w-0 rounded-lg ..."><a tabindex="-1" aria-hidden="true" href="/products/bang-dieu-khien-dieu-hoa-matter-microair-81mtac01-12-hang"><img alt="" data-nimg="fill" srcSet="/_next/image?url=%2Fmedia%2Fb5b44b...jpg&amp;w=256&amp;q=75 256w, ..."/></a>
<div class="min-w-0 px-2.5 pb-3.5 pt-0.5 ..."><span class="mb-1.5 block text-[12px] uppercase ...">Micro Air</span><h3 class="mb-3.5 line-clamp-2 ..."><a class="hover:underline" href="/products/...">Bảng điều khiển điều hòa Matter MicroAir 81MTAC01, 12 hãng</a></h3>
... <span class="min-w-0 text-[15px] font-semibold ...">3.995.000đ</span> ... <button aria-label="Thêm Bảng điều khiển điều hòa Matter MicroAir 81MTAC01, 12 h..." type="button">
```

Props client component trong payload RSC (nút thêm vào giỏ báo giá, module `16268`; đối tượng `product` đầy đủ, không có trong DOM):

```text
40:I[16268,[...],"default"]
{"product":{"id":177,"slug":"cam-bien-chuyen-dong-eve-motion-matter-over-thread-ipx3","name":"...","sku":"Eve Motion","shortDescription":"Tương thích <span class=\"fw-bold\" ...>Apple Home, Google Home, ...</span><br>\r\nGiao thức ...","brand":{...},"b2c":true,"b2b":true,"descriptionHtml":"$60","seoTitle":"...","images":[{...}],"specs":[{"code":"goc-phat-hien","name":"Góc phát hiện","unit":"°","valueText":"120°","parseStatus":"exact",...}],"bundles":[...],"related":[],"downloads":[]}}
```

Hero (HTML thô, slide 1):

```html
<span class="mb-3 block ...">KNXStore.vn/matter</span><h1 class="mb-3 text-[22px] font-semibold leading-tight text-white sm:text-[40px]">Thiết bị nhà thông minh Matter chính hãng</h1><p class="text-[14px] leading-[1.5] text-[#fafafa]">Tương thích Apple Home, Google Home và Alexa. ...</p>
```

## 6. Nội dung chỉ có ở client / không thấy trong HTML ban đầu

Không thấy skeleton/placeholder chờ hydrate: không có `BAILOUT_TO_CLIENT_SIDE_RENDERING`, `<template data-dgst`, `animate-pulse`, `skeleton`; 0 thẻ `<template>`. Nội dung chính của `<main>` (hero, danh mục, 18 sản phẩm, CTA, 3 bài viết) **đều có trong HTML ban đầu**.

Dữ liệu có trong payload RSC (`self.__next_f.push`, 6 lần push, 386,149 B trong 6 thẻ script; decode ra 233,025 B / 100 row) nhưng **không có trong DOM**:

| Dữ liệu | Vị trí | Quy mô | Bằng chứng không có trong DOM |
|---|---|---|---|
| Đối tượng `product` đầy đủ của 18 sản phẩm (props của client component nút thêm giỏ, module `16268`) | ví dụ rows `43`–`48`, `4b` | 8 sản phẩm "mới" dùng object nhẹ (id, slug, name, sku, shortDescription, brand, primaryImage, b2c/b2b, stockStatus, price, oldPrice, priceRange, clearance...); 10 sản phẩm "nổi bật" dùng object đầy đủ (+ `descriptionHtml`, `specs[]`, `images[]`, `seoTitle`, `seoDescription`, `countryOfOrigin`, `warrantyMonths`, `bundles`, `related`, `downloads`, `status`, `publishedAt`) | chuỗi "SmartThings" (trong `shortDescription`) và tên đề mục mô tả (ví dụ "Bộ động cơ nâng cấp điều khiển rèm") không xuất hiện trong HTML ngoài thẻ script; trạng thái tồn kho (`in_stock`, `out_of_stock`, `low_stock`, `make_to_order`) không hiển thị trong DOM |
| HTML mô tả dài của 10 sản phẩm nổi bật (`descriptionHtml`) và 3 trường `matter_application_{1,2,3}_description` | 13 row dạng `T` (`49`, `60`, `62`, `64`, `65`, `67`, `68`, `69`, `6a`–`6e`) | tổng 81,987 B (khoảng 35% payload đã decode); từng row 1,358–10,893 B, mở đầu bằng `<h2>...` (ví dụ "Homey Pro Là Gì: Hub Smarthome Hợp Nhất 8 Giao Thức Trong Một Thiết Bị") | `Homey Pro Là Gì` không có trong HTML ngoài script |
| Dữ liệu chrome B2B của boundary not-found: mega menu "Trending", 8 từ khóa trending, danh sách sản phẩm trong nav, footer B2B | rows `10`, `14`–`2a` | 55,046 B (23.6% payload) | không render trên /matter; header/footer thật của trang là bản Matter |
| JSON-LD (lặp lại row `5f`) | row `5f` (5,668 B) | | cũng có dạng `<script type=application/ld+json>` trong DOM |

Chỉ phát sinh sau tương tác client (không khảo sát được vì không chạy JS): đổi slide hero, mở mega menu/menu mobile, ô tìm kiếm trong header (gợi ý/điều hướng `?q=`), số lượng và nội dung drawer báo giá, nút "Thêm ... vào ..." trên thẻ sản phẩm.

## 7. Liên kết

| Vùng | Số `<a>` |
|---|---|
| Header | 17 |
| Aside menu mobile | 15 |
| `<main>` | 62 |
| Drawer báo giá | 2 (`/products`, `/cart`) |
| Footer | 21 |
| Nút Zalo nổi | 1 |
| Tổng | 118 |

Phân loại 62 link trong `<main>`: `/products/<slug>` 36; `/<slug>.html` (bài viết) 9; `/matter-search` 3; danh mục gốc (`/cam-bien-matter` 2, `/cong-tac-matter` 2, `/dieu-khien-dan-lanh`, `/hub-trung-tam`, `/gateway-matter`, `/driver-matter`, `/camera-thong-minh`, `/khoa-thong-minh-matter` mỗi cái 1) 10; `/matter/categories` 1; `/blogs` 1; `tel:0378628837` 1; `https://zalo.me/0378628837` 1.

- Mẫu URL nội bộ: sản phẩm `/products/<slug>`; danh mục `/<slug>` (không có tiền tố); bài viết `/<slug>.html`; liệt kê `/matter-search`, `/matter/categories`, `/blogs`, `/cart`, `/contact`, `/warranty-lookup`. Tất cả URL nội bộ là root-relative.
- Link ngoài (toàn trang): `zalo.me/0378628837` ×3, `facebook.com/knxstore.vn`, `online.gov.vn/...`, `dmca.com/...`.
- `link rel=alternate`, link markdown: Không có.

## 8. JSON-LD

1 script `application/ld+json` (5,668 B), là con đầu tiên của `<main>`, dạng `@graph` 4 node:

| @type | @id | Trường có mặt | Trường rỗng/thiếu |
|---|---|---|---|
| `["Organization","LocalBusiness"]` | `https://knxstore.vn/#organization` | name, legalName ("Công ty CP Tích Hợp Hệ Thống Liên Minh"), url, foundingDate "2012", taxID, logo (ImageObject `.../assets/logo.png`, 150×50), image (ImageObject `.../assets/knxstore.png`, 1200×630), description, telephone `+84918918755`, email, address (có postalCode "700000"), geo, hasMap, openingHoursSpecification (Mon–Sat 09:00–17:30), priceRange "50.000₫ - 200.000.000₫", areaServed, knowsAbout (10 mục), contactPoint (sales), sameAs (Facebook, YouTube) | Không có giá trị rỗng |
| `CollectionPage` | `https://knxstore.vn/matter#webpage` | url, name, description, inLanguage "vi-VN", isPartOf `{@id: https://knxstore.vn/#website}`, publisher, about (Thing "Matter", sameAs csa-iot.org), hasPart (2 @id ItemList) | Không có breadcrumb |
| `ItemList` | `.../matter#new-products` | name "Sản phẩm Matter mới tại KNXStore", numberOfItems 8, itemListElement 8 `ListItem` (`position`, `url`) | `ListItem` chỉ có `position` + `url` (không có name, image, `item`) |
| `ItemList` | `.../matter#featured-products` | name "Sản phẩm Matter nổi bật tại KNXStore", numberOfItems 10, 10 `ListItem` (`position`, `url`) | như trên |

- Thiếu node: `WebSite` (`https://knxstore.vn/#website` được tham chiếu bởi `isPartOf` nhưng không được định nghĩa trong graph của trang này); không có `BreadcrumbList`; không có `Product`/`Offer` cho 18 sản phẩm; không có `Article`/`BlogPosting` cho 3 bài viết.
- Điện thoại trong JSON-LD (`+84918918755`) khác số hiển thị/liên kết trong DOM của trang (`0378.628.837`); ghi nhận hiện trạng.

## 9. Số đo

| Chỉ số | Giá trị |
|---|---|
| HTML thô | 551,978 B |
| Sau nén | br 86,123 B; gzip 95,102 B (wire); gzip level 6 cục bộ 95,843 B |
| Tổng byte `<script>` inline | 391,960 B (22 thẻ script, 13 có `src`) |
| `self.__next_f` | 386,149 B (6 lần push; decode 233,025 B) |
| JSON-LD | 5,668 B |
| `<style>` inline | 0 (2 stylesheet ngoài) |
| Text hiển thị (bỏ script/style) toàn `<body>` | 3,708 ký tự = 4,365 B UTF-8 (gồm menu ẩn/mobile/drawer) |
| Text hiển thị trong `<main>` | 2,491 ký tự = 2,913 B UTF-8 |
| `<a>` / `<img>` / `<table>` | 118 / 36 / 0 |
| `<h1>`...`<h6>` (toàn trang) | 1 / 11 / 21 / 0 / 0 / 0 |
| `<form>` / `<input>` / `<button>` | 1 / 1 / 29 |
| `<details>` / `<svg>` | 3 / 48 |
| Token ước lượng (bytes/4) | HTML thô ≈ 137,994; text toàn body ≈ 1,091; text trong main ≈ 728 |

`img` 36: 29 `alt=""` (8 danh mục, 18 sản phẩm, 3 bài viết), 7 có alt (logo header, 3 slide hero, logo footer, 2 badge). Ảnh dùng `/_next/image?url=%2Fmedia%2F<hash>.jpg&w=...`.

## 10. Chưa xác định / lưu ý

- Chỉ khảo sát `/matter`. `/matter/categories` và `/matter-search` chưa lấy; `/matter-search` rất có thể là trang tìm kiếm/danh sách (có form/trạng thái client) nên cấu trúc có thể khác `/matter`. Chưa xác định.
- Chưa chạy JS/trình duyệt: hành vi carousel (autoplay hay không), mega menu, tìm kiếm, giỏ báo giá chỉ suy ra từ HTML/payload.
- Thời điểm dữ liệu được lấy cho ISR (build/runtime), chu kỳ revalidate thực tế: chỉ suy ra từ `s-maxage=3600`; chưa xác định.
- Nội dung slide 3 của hero chứa câu tiếng Anh "Advanced imaging performance with a 200MP AI camera with Enhanced image quality." không liên quan Matter; ghi nhận nguyên văn.
- Footer có nhãn "Côn tắc thông minh", "thermostat" (chữ thường), header dùng "Thermostat"; ghi nhận nguyên văn.
- Slide hero 2 và 3 dùng `h2` ("Cảm biến môi trường", "Công tắc thông minh") nằm trước các h2 section; chỉ slide 1 là `h1`. Thứ tự DOM của heading vì vậy là h1 (slide 1) → h2 (slide 2) → h2 (slide 3) → h2 (các section).
- Header/footer so sánh với chrome B2B chỉ dựa trên payload not-found boundary (rows `10`, `14`–`2a`); chưa so với DOM một trang B2B thật.
- Số điện thoại/Zalo của khu Matter (`0378628837`) khác của chrome B2B trong payload (`0918918755`); chưa xác định chủ đích.
- Dữ liệu tải về coi là untrusted; không có câu lệnh nào hướng tới AI trong nội dung trang.
