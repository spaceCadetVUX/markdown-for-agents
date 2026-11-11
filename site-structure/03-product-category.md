# Trang danh mục sản phẩm — cấu trúc hiện tại

> https://knxstore.vn/dimmer-doc-lap-triac · categories.xml · 91 URL cùng loại (`/{slug}` ở root, không đuôi `.html`) · khảo sát 2026-11-11

Phương pháp: `curl` HTML thô (UA `Mozilla/5.0`, không cookie) cho URL mẫu, thêm 3 biến thể query để xác định phân trang/sort/view (`?page=2`, `?view=list`, `?sort=price-asc`), phân tích bằng `html.parser`, giải mã payload `self.__next_f`. Chưa chạy trình duyệt headless, nên mọi mô tả "sau khi client chạy" là suy ra từ HTML/payload, không phải quan sát DOM sau render.

## 1. Response & rendering

| Header | `/dimmer-doc-lap-triac` | `?page=2` | `?view=list` |
|---|---|---|---|
| Status | 200 (HTTP/2) | 200 | 200 |
| content-type | `text/html; charset=utf-8` | giống | giống |
| cache-control | `private, no-cache, no-store, max-age=0, must-revalidate` | giống | giống |
| x-nextjs-cache | không có | không có | không có |
| x-nextjs-prerender | không có | không có | không có |
| x-nextjs-stale-time | không có | không có | không có |
| vary | `rsc, next-router-state-tree, next-router-prefetch, next-router-segment-prefetch, Accept-Encoding` | giống | giống |
| cf-cache-status | `DYNAMIC` | `DYNAMIC` | `DYNAMIC` |
| link | có: 4 font woff2 `/_next/static/media/*.woff2` (`rel=preload; as="font"; crossorigin; type="font/woff2"`) + 2 CSS `/_next/static/chunks/1mmmxvxvvtkv8.css`, `/_next/static/chunks/1vbr8oh4939ca.css` (`rel=preload; as="style"`) | có | không đo |
| x-robots-tag | không có | không có | không có |
| content-encoding | không có khi không gửi Accept-Encoding; `br` khi gửi `gzip, br` | không đo | không đo |
| Header khác | `server: cloudflare`, `speculation-rules: "/cdn-cgi/speculation"`, `strict-transport-security`, `x-frame-options: SAMEORIGIN`, `x-content-type-options: nosniff`, `referrer-policy: same-origin`, `alt-svc: h3` | giống | giống |

`?sort=price-asc` cũng trả 200 và có hiệu lực ở server (24 card sắp tăng dần theo giá, `<option value="price-asc" selected>` trong HTML).

Kích thước body: `/dimmer-doc-lap-triac` 535,241 B (identity), br 52,721 B trên đường truyền; `?page=2` 396,398 B; `?view=list` 572,738 B; `?sort=price-asc` 535,918 B.

Chế độ render: dynamic (render theo từng request, không có tầng ISR/static). Bằng chứng: `cache-control: private, no-cache, no-store`; vắng cả ba header `x-nextjs-cache`, `x-nextjs-prerender`, `x-nextjs-stale-time` (trang sản phẩm ISR có đủ ba header này); có header `link` preload (trang sản phẩm ISR không có); `?page`, `?view`, `?sort` đều cho nội dung khác nhau ở server. Lý do kỹ thuật khiến route này dynamic (đọc `searchParams`, `cookies()`, fetch `no-store`…): Chưa xác định, không có cách thấy từ phía client.

Route (từ record `0:` của payload): `(site)/[slug]/page`, `slug` là dynamic segment. Danh sách sibling static ở root đọc được trong payload: `admin, api, author, blogs, brands, cart, clearance, compare, connection-check, contact, context, css, label, llms.txt, mail-success, matter, matter-search, order-success, page, products, robots.txt, search, sitemap.xml, sitemaps, solution, tools, track-order, visual, warranty, warranty-lookup`.

Streaming: có 1 Suspense boundary `B:0` (`<!--$?--><template id="B:0"></template><!--/$-->`, fallback `null`), nội dung đẩy xuống sau cùng trong `<div hidden id="S:0">` kèm script `$RC("B:0","S:0")` (mục 4, 6).

Hành vi hiện tại với yêu cầu markdown (chỉ ghi kết quả):

| Request | Status | content-type | Ghi chú |
|---|---|---|---|
| `Accept: text/markdown` | 200 | `text/html; charset=utf-8` | body 535,241 B, vẫn là HTML; `vary` không chứa `Accept` |
| URL + `.md` (`/dimmer-doc-lap-triac.md`) | 404 | `text/html; charset=utf-8` | body 131,161 B (trang lỗi Next), `cache-control: private, no-cache, no-store` |

## 2. `<head>`

Head 14,216 ký tự (dài hơn trang sản phẩm do 8 thẻ preload ảnh). `<html lang="vi">`.

| Phần tử | `/dimmer-doc-lap-triac` | `?page=2` |
|---|---|---|
| `<title>` | `Dimmer Triac Chính Hãng, Điều Khiển Đèn LED \| KNXStore.vn` | `… \| KNXStore.vn — trang 2` |
| meta description | `Dimmer Triac chính hãng, điều khiển độ sáng đèn LED mượt, tích hợp KNX, Casambi, DALI, DMX. Tư vấn chọn đúng driver tương thích, báo giá ngay.` | giống hệt trang 1 |
| canonical | `https://knxstore.vn/dimmer-doc-lap-triac` | `https://knxstore.vn/dimmer-doc-lap-triac?page=2` |
| meta robots | không có | không có |
| keywords | `Dimmer Triac` | – |
| OG | `og:title` (= title), `og:description`, `og:url` (= canonical), `og:image` `https://api.knxstore.vn/media/1eec55b4…bd.jpg`, `og:type=website` | `og:url` = `…?page=2`, còn lại như trang 1 |
| Twitter | `summary_large_image`, title, description, image | – |
| hreflang / `<link rel=alternate>` | không có | không có |
| `<link rel=prev/next>` trong head | không có | không có (`rel="next"`/`rel="prev"` chỉ nằm trên thẻ `<a>` phân trang trong body) |
| `rel=manifest`, icons, `theme-color #14467C`, `author KNX Store` | giống trang sản phẩm | – |
| preload | 4 font woff2, logo `/images/logo/knxstore-logo-tight.svg`, 8 ảnh card đầu (`<link rel=preload as=image imageSrcSet=/_next/image?url=%2Fmedia%2F{hash}.{jpg|png}&w=256…>`), 1 JS `3ivcc6t1s04dh.js` (fetchPriority=low), `gtag/js?id=G-4D9XLJDVGG` | – |
| script trong head | 13 `<script src async>` (12 chunk + 1 `noModule`); khác trang sản phẩm ở 2 chunk cuối (`2ns7gl8cadig5.js`, `39o1ddnu25rel.js`) | – |

`?view=list` có canonical `https://knxstore.vn/dimmer-doc-lap-triac` (bỏ query). `?sort=price-asc` cũng canonical về URL gốc. Chỉ `?page=N` giữ trong canonical.

## 3. Khung trang

Landmark: `header`, `footer`, `form[role=search]`, `nav` (6 trong header, 1 Breadcrumb, 1 `Phân trang`, 3 footer = 11), `aside[aria-label="Bộ lọc"]` (trong drawer lọc), `div[role=dialog][aria-modal=true]#filter-panel-drawer`. Không có `<main>`; vùng nội dung chính là `div#main-content[tabindex=-1]` (đích skip link "Bỏ qua, tới nội dung chính").

Thứ tự con trực tiếp của `<body>`: `div[hidden]` rỗng → `script` (cookie `knx-area=b2b`) → skip link → `header.sticky` → `div#main-content` → `div[inert].fixed` (drawer "Yêu cầu báo giá") → `footer` → `a[data-floating-zalo]` → `script` → `script#_R_` → `div[hidden]#S:0` → các `<script>`.

Header, footer, nút Zalo nổi, drawer báo giá: giống hệt trang sản phẩm skin B2B (header 165 `<a>`, 6 panel menu `hidden`, footer 50 `<a>` với 3 cột render 2 lần mobile/desktop, Zalo `https://zalo.me/0918918755`). Không có chat widget bên thứ ba, popup, cookie banner, back-to-top.

Khác biệt khung so với trang sản phẩm: nằm ngoài `#main-content` ở cuối `<body>` có thêm `div[hidden]#S:0` (toolbar lọc/sort, mục 4). Drawer lọc `#filter-panel-drawer` cũng nằm trong `#S:0`, không phải trong `#main-content`.

## 4. Cây nội dung (theo thứ tự DOM)

Cột "Trong main": nằm trong `div#main-content` hay không.

| # | Thẻ + selector rút gọn | Heading | Nội dung tóm tắt | Trong main | Hiển thị mặc định | SSR / client |
|---|---|---|---|---|---|---|
| 1 | `header.sticky` | 18 h3 trong panel ẩn | như trang sản phẩm | không | thanh trên + menu hiện, 6 panel `hidden` | SSR |
| 2 | `div#main-content > script[type=application/ld+json]` | – | JSON-LD 9,519 B (mục 8) | có | không hiển thị | SSR |
| 3 | `div.bg-white > nav[aria-label=Breadcrumb]` | – | `Trang chủ` (`/`) / `Dimmer & Relay chiếu sáng` (`/dieu-khien-chieu-sang`) / `Dimmer Triac` (span hiện tại) | có | hiện | SSR |
| 4 | `div.py-4 > div > h1.text-h1-mobile` + `span.tabular-nums` | h1: "Dimmer Triac" | tên danh mục + "29 sản phẩm" (tổng, không phải số card trong trang) | có | hiện | SSR |
| 5 | `div.flex.flex-wrap.gap-2.5.pb-5` chứa 4 `a` | – | chip liên kết: `Dimmer DALI` → `/dimmer-doc-lap-dali`; `Dimmer DMX` → `/dmx`; `Điều khiển 0-10V` → `/dimmer-doc-lap-0-10v` (chữ chia 2 span); `Điều khiển ON/OFF` → `/dieu-khien-chieu-sang-on-off` | có | hiện | SSR |
| 6 | `<!--$?--><template id="B:0"></template><!--/$-->` | – | placeholder Suspense, rỗng | có | không hiển thị | SSR (placeholder) |
| 7 | `div.bg-white > div.grid.grid-cols-2.sm:grid-cols-3.lg:grid-cols-4` | – | lưới 24 `article` (1,956 ký tự); card gồm ảnh, thương hiệu, mã, tên, giá, nút thêm báo giá (mục 5) | có | hiện | SSR |
| 8 | `nav[aria-label="Phân trang"].mt-8` (cùng `div.bg-white` với lưới) | – | `Trước` (span `aria-disabled`), `1` (`a[aria-current=page]`, `/dimmer-doc-lap-triac`), `2` (`/dimmer-doc-lap-triac?page=2`), `Sau` (`a[rel=next][aria-label="Trang sau"]`, `?page=2`) | có | hiện | SSR |
| 9 | `section.border-t.bg-white.py-8 > div.mx-auto > h2 + div.user-html.mt-4` | h2 "Về Dimmer Triac" | văn bản SEO dưới lưới: `div.user-html` 5,714 ký tự gồm 18 `p`, 6 h2 con, 4 h3 con, 2 `ul`, 1 `table`; không có link, không có ảnh. Toàn `section` (gồm cả FAQ ở dòng 10) = 7,130 ký tự | có | hiện | SSR |
| 10 | `div.mx-auto > div.mt-8 > section.rounded-card > h2 + ul > li > details.group/faq` (con cuối của section ở dòng 9) | h2 "Câu hỏi thường gặp về dimmer triac" | 5 Q&A (1,401 ký tự): `summary` (câu hỏi + `span` "+") và `div.user-html` (đáp án); mọi `details` đóng (không `open`) nhưng đáp án có trong DOM | có | câu hỏi hiện, đáp án thu gọn | SSR |
| 11 | `div[inert].fixed.translate-x-full` | h2 "Yêu cầu báo giá" | drawer rỗng | không | ẩn | SSR |
| 12 | `footer.mt-auto` | h2 ×3 | như trang sản phẩm | không | hiện | SSR |
| 13 | `a[data-floating-zalo]` | – | nút "Zalo" | không | hiện | SSR |
| 14 | `div[hidden]#S:0` (offset 280,720 / 522,786 ký tự trong HTML, sau footer) | h2 "Bộ lọc" (trong drawer) | toolbar: nút "Bộ lọc 0", drawer lọc, nút xem lưới/bảng, `select` sort (chi tiết dưới bảng) | không | `hidden` trong HTML thô; vị trí hiển thị thực là chỗ `<template id="B:0">` sau khi `$RC` chạy | SSR (trong HTML) nhưng chỉ vào đúng vị trí sau khi script `$RC` chạy |

Toolbar `#S:0` (395 ký tự):

| Thành phần | Chi tiết |
|---|---|
| Nút lọc | `button[aria-expanded=false][aria-controls=filter-panel-drawer]` "Bộ lọc" + badge `0` |
| Drawer lọc | `div#filter-panel-drawer[role=dialog][aria-modal=true][aria-hidden=true]`, class `fixed … -translate-x-full pointer-events-none`; `aside[aria-label="Bộ lọc"]` gồm: checkbox "Chỉ hàng thanh lý"; `details[open]` "Thương hiệu" với `fieldset > ul > li > label > input[type=radio][name="Thương hiệu"]` + số đếm: Vadsbo 7, Lite-Puter 4, Sunricher 3, DALCNET 2, EBELONG 2, Legrand 2, Lutron 2, OEM 2, Casambi 1, GVS 1, Kanonbus 1, Maintronic 1, Satel 1 (tổng 29); `details` "Giải pháp" 7 radio: Giải pháp khác 8, Casambi 5, KNX 5, DMX 4, Zigbee 3, DALI 2, Lutron 2; nút "Xoá lọc", "Áp dụng" |
| Chuyển dạng xem | `a[aria-label="Xem dạng lưới"]` → `/dimmer-doc-lap-triac`; `a[aria-label="Xem dạng bảng"]` → `/dimmer-doc-lap-triac?view=list` |
| Sort | `<select>`: `""` Mặc định, `newest` Mới nhất, `name-asc` Tên A → Z, `name-desc` Tên Z → A, `price-asc` Giá thấp → cao, `price-desc` Giá cao → thấp |

Bộ lọc dùng `input` radio/checkbox không có `href`/`name` dạng query; tên query param khi áp dụng lọc: Chưa xác định. Sort/page/view là query param xử lý ở server (đã kiểm: `?sort=price-asc`, `?page=2`, `?view=list`).

Cấu trúc 1 card (24 card/trang, `article.group.relative.rounded-card.border`):

| Phần | Selector | Dữ liệu |
|---|---|---|
| Ảnh | `div.aspect-square > img[alt=<tên SP>][decoding=async][data-nimg=fill]` + `srcset` w=256…3840 + `sizes="(max-width: 640px) 50vw, (max-width: 1024px) 33vw, 270px"`; 8 card đầu không có `loading` (khớp 8 thẻ `<link rel=preload as=image>` trong head), các card sau `loading=lazy` | `src` dạng `/_next/image?url=%2Fmedia%2F{sha256}.jpg&w=3840&q=75`; không có `fetchpriority`; `alt` luôn có (0 ảnh alt rỗng trong main) |
| Badge giảm giá | `span.absolute.left-2.top-2` ("-50%") | chỉ ở card có giá cũ |
| Thương hiệu + mã | `span` (nền gradient) + `span.model-code` | ví dụ `Maintronic` / `CRD400` |
| Tên + link | `a.clamp-fluid[href=/products/{slug}]` với `after:absolute after:inset-0` (cả card là vùng bấm); token model bọc `span.whitespace-nowrap` | ví dụ "Bộ điều khiển dimmer Triac Casambi Maintronic - CRD400" |
| Giá | `div.mt-auto` > `div.text-[16px].font-semibold` (giá hiện tại); card giảm giá có thêm `div.line-through` (giá cũ) đứng trước giá mới trong thứ tự DOM | định dạng `9.375.000 đ`; không có giá → "Liên hệ" (2 card) |
| Nút | `button[aria-label="Thêm {tên} vào yêu cầu báo giá"]` | hành vi client |

Thống kê 24 card trang 1: 22 card có giá số, 2 card "Liên hệ" (Legrand 002654, Sunricher SR-2303AC), 2 card có badge `-50%` (GVS KA/D 0415.S.1: `18.987.000 đ` → `9.494.000 đ`; Vadsbo LD220: `2.150.000 đ` → `1.075.000 đ`). Trang 2: 5 card, 5 có giá, 1 badge `-16%` (OEM WALL-DIMMER-230V-TRIAC `2.500.000 đ` → `2.100.000 đ`).

Dạng xem bảng (`?view=list`): vẫn 24 `article`, grid đổi thành `div.grid.grid-cols-1.gap-2.5`, mỗi card thêm `div.product-row-check > label > input[type=checkbox]` ("So sánh {tên}"), class `product-row-title`, `product-row-price`; cùng trường dữ liệu như dạng lưới (ảnh, thương hiệu, mã, tên, giá), không có cột thông số, không có `<table>` mới (số `<table>` vẫn 1, là bảng trong văn bản SEO).

Phân trang: link `?page=N` (render ở server, mỗi trang là một URL riêng, 200). Sản phẩm trang 2 (5 slug: `kanonbus-bo-dieu-khien-chieu-sang-triac-knx-kte45bd`, `ai-control-dimmer-doc-lap-triac`, `lutron-bo-dieu-khien-rf`, `lutron-dimmer-dieu-chinh-do-sang-dong-ra2-co-day`, `vadsbo-dimmer-xoay-cho-den-led-vd300`) không có trong HTML cũng không có trong payload của trang 1 (0 lần xuất hiện). Page size suy ra = 24 (24 + 5 = 29). Không có load-more/infinite scroll trong HTML.

Trang 2 lặp nguyên văn: h1 "Dimmer Triac", "29 sản phẩm", chip, toolbar `#S:0`, văn bản SEO 7,130 ký tự (giống từng ký tự trang 1), FAQ 5 Q&A, JSON-LD `FAQPage`.

## 5. Chi tiết khối đặc biệt

Suspense placeholder và nội dung streamed:

```html
...</a></div></div></div><!--$?--><template id="B:0"></template><!--/$--><div class="bg-white"><div class="grid grid-cols-2 …"><article …>
…(footer, nút Zalo, script)…
<div hidden id="S:0"><div class="bg-white"><div class="mx-auto …"><div class="flex flex-wrap items-center justify-between gap-3 border-t …"><button type="button" aria-expanded="false" aria-controls="filter-panel-drawer" …>Bộ lọc</button> …
…$RC("B:0","S:0")</script>
```

Phân trang:

```html
<nav aria-label="Phân trang" class="mt-8"><div class="flex max-w-full items-center gap-2 overflow-x-auto"><span aria-disabled="true" …>Trước</span><a aria-current="page" … href="/dimmer-doc-lap-triac">1</a><a … href="/dimmer-doc-lap-triac?page=2">2</a><a rel="next" aria-label="Trang sau" … href="/dimmer-doc-lap-triac?page=2">Sau</a></div></nav>
```

Bảng trong văn bản SEO (1 bảng, `div.user-html > table`, 3 cột, 8 `tr` = 1 header + 7 dòng, 3 `th[scope=col]`, 21 `td`, không `rowspan`/`colspan`, cột đầu là `td`, HTML thụt tab):

```html
<table><thead><tr><th scope="col">Giao thức</th><th scope="col">Đặc điểm</th><th scope="col">Thương hiệu tiêu biểu</th></tr></thead>
<tbody><tr><td>Độc lập, không giao thức</td><td>Điều khiển trực tiếp bằng nút nhấn hoặc núm xoay trên thiết bị, không tích hợp hệ thống trung tâm</td><td>Vadsbo, EBELONG, OEM</td></tr>
<tr><td>KNX</td><td>Tích hợp vào bus KNX, cấu hình group address qua ETS, …</td><td>Kanonbus, Satel, Legrand, GVS</td></tr> …
```

7 dòng dữ liệu: Độc lập; KNX; Casambi BLE; DALI; DMX; Zigbee; RF độc quyền (Lutron).

FAQ:

```html
<li class="border-b border-zinc-200"><details class="group/faq"><summary class="flex cursor-pointer …">Dimmer Triac là gì?<span aria-hidden="true">+</span></summary><div class="user-html pb-4 …">Dimmer Triac là thiết bị điều chỉnh độ sáng đèn …</div></details></li>
```

Props card trong payload RSC (mỗi card, một số trường không in ra DOM):

```json
{"id":310,"slug":"dimmer-dieu-chinh-do-sang-led-casambi-retrofit-crd-ins-casambi-crd400","name":"Bộ điều khiển dimmer Triac Casambi Maintronic - CRD400","sku":"CRD400","shortDescription":null,"brand":{"slug":"maintronic","name":"Maintronic"},"primaryImage":"/media/243b12a6….jpg","isFeatured":false,"b2c":false,"b2b":true,"stockStatus":"make_to_order","price":9375000,"oldPrice":null,"priceRange":{"from":9375000,"to":9375000},"clearance":false,"priceHidden":false,"clearanceFrom":null,"clearanceCount":0}
```

`stockStatus` nhận các giá trị `in_stock`, `low_stock`, `make_to_order`, `out_of_stock` trong payload của trang, không hiển thị ở card.

## 6. Nội dung chỉ có ở client hoặc không thấy trong HTML ban đầu

| Nội dung | Có trong HTML ban đầu? | Bằng chứng |
|---|---|---|
| H1, "29 sản phẩm", breadcrumb, chip, 24 card, phân trang, văn bản SEO, FAQ (cả đáp án) | Có, SSR | đếm trong DOM; `<template data-dgst`, `BAILOUT_TO_CLIENT_SIDE_RENDERING`, `animate-pulse`, `skeleton` đều = 0 |
| Toolbar lọc/sort/chuyển dạng xem | Có trong HTML nhưng không ở vị trí hiển thị: nằm trong `div[hidden]#S:0` sau footer; vào đúng chỗ khi script `$RC("B:0","S:0")` chạy | `<!--$?--><template id="B:0"></template><!--/$-->` + `<div hidden id="S:0">` + `$RC(...)` |
| Facet có số đếm (13 thương hiệu, 7 giải pháp) | Có text trong `#S:0` (hidden); không có trong `#main-content` | mục 4 |
| Kết quả sau khi chọn bộ lọc | Không; radio/checkbox không có URL, lọc chạy ở client (tên param Chưa xác định) | không có `href` trong `fieldset` |
| Sản phẩm trang 2 (5 sản phẩm) | Không có trong HTML/payload của trang 1; chỉ có ở URL `?page=2` | 0 lần xuất hiện của 5 slug ở trang 1 |
| Thêm vào báo giá, so sánh, số lượng giỏ | Chỉ hành vi client (nút có, state không) | `button[aria-label="Thêm … vào yêu cầu báo giá"]`; drawer "Tổng số lượng: 0" |
| Gợi ý tìm kiếm header, mega menu | Mega menu có trong DOM `hidden`; gợi ý search không có | như trang sản phẩm |
| `stockStatus`, `isFeatured`, `priceRange`, `clearance*`, `shortDescription` của từng card | Chỉ trong payload RSC | mục 5 |
| Mô tả/giới thiệu ở đầu trang | Không có; văn bản mô tả nằm dưới lưới | `main-content` không có đoạn text giữa h1 và lưới |

Payload: `self.__next_f` 233,399 B trong 2 thẻ `<script>` (43.6% HTML thô; một push lớn chứa gần như toàn bộ dữ liệu). Thành phần ước tính theo nhóm record: dữ liệu nav header ~58.9 KB (khối 28 KB xuất hiện 2 lần: layout thật và fallback not-found), dữ liệu card ~56.1 KB (24 card, lặp lại nội dung DOM + các trường không in ra), footer ~25.7 KB, JSON-LD ~9.6 KB (lặp lại thẻ ld+json), HTML văn bản SEO + FAQ ~8.5 KB, route tree ~3.9 KB, module refs ~4.2 KB, phần còn lại (facet, props khác) ~45 KB. Đây là ước tính từ phân loại record, không phải đo chính xác từng byte.

## 7. Liên kết

Tổng 254 `<a>` trong body: 165 header, 50 footer, 33 trong `#main-content`, 1 skip link, 1 Zalo nổi, 2 trong drawer báo giá (`/products`, `/cart`), 2 trong toolbar `#S:0` (nút lưới/bảng).

| Nhóm | Mẫu URL | Toàn trang | Trong `#main-content` |
|---|---|---|---|
| Sản phẩm | `/products/{slug}` | 40 (24 card + 16 trong panel Trending header) | 24 |
| Danh mục / khác dạng `/{slug}` | `/{slug}` không đuôi | 93 (header 61, footer 21, main 8 gồm 1 breadcrumb cha + 4 chip + 3 link phân trang, còn lại gồm nút lưới/bảng, drawer) | 8 |
| Brand | `/brands/{slug}` | 24 (header) | 0 |
| Giải pháp | `/solution/{slug}` | 38 (header 24, footer 14) | 0 |
| Bài viết / trang chính sách / công cụ `.html` | `/{slug}.html`, `/tool/{tên}/index.html` | 19 | 0 |
| Blog listing | `/blogs…` | 12 | 0 |
| Tìm kiếm | `/search?q=…` | 8 | 0 |
| Khác nội bộ | `/` (3), `/cart` (2), `/contact` (3), `/visual` (1), `/matter` (1), `/tools/hvac-compatibility` (1) | 11 | 1 (`/`) |
| `?page=` | `/dimmer-doc-lap-triac?page=2` | 2 (số "2" và "Sau") | 2 (đã tính trong nhóm `/{slug}`) |
| Liên ngoài | `zalo.me` ×2, `facebook.com`, `maps.app.goo.gl`, `online.gov.vn`, `dmca.com`; `tel:`, `mailto:` | 6 + 2 | 0 |

Link trong văn bản SEO: 0 (không có link nội bộ, không có ảnh). Link tới file tải: 0 (không có `.pdf`, `.dwg`, `.zip`… trong HTML). Pattern ảnh: `/_next/image?url=%2Fmedia%2F{sha256}.{jpg|png|avif}&w=…&q=…` (card), `/media/{sha256}.{ext}` (gốc); `og:image` và JSON-LD dùng `https://api.knxstore.vn/media/…`.

Về markdown/alternate: không có `<link rel=alternate>`, không có `Link:` header kiểu alternate (header `link` chỉ chứa preload font/CSS), không có chuỗi `text/markdown`. Chuỗi `llms.txt` chỉ xuất hiện như tên route static trong cây route của payload; `https://knxstore.vn/llms.txt` trả 200 `text/plain; charset=utf-8` 7,354 B (đo riêng), không được liên kết từ HTML danh mục.

## 8. JSON-LD

1 thẻ `<script type="application/ld+json">` (9,519 B), con đầu tiên của `div#main-content`; `@context` + `@graph` gồm 5 node.

| Node | Trường có | Ghi chú |
|---|---|---|
| `Organization` + `LocalBusiness` | như trang sản phẩm | giống nhau trên mọi trang đã đo |
| `CollectionPage` (`@id` `…/dimmer-doc-lap-triac#webpage`) | url, name `Dimmer Triac`, description (= meta description), inLanguage `vi-VN`, datePublished = dateModified = `2026-08-24T14:20:16.977Z`, isPartOf `{@id …/#website}`, publisher, `mainEntity` → `#products`, `about` → `#about`, primaryImageOfPage (ImageObject, url `api.knxstore.vn/media/1eec55b4…jpg`, caption `Dimmer Triac`), breadcrumb (BreadcrumbList 3 ListItem: Trang chủ `/`, Dimmer & Relay chiếu sáng `/dieu-khien-chieu-sang`, Dimmer Triac) | `isPartOf` trỏ `#website` nhưng không có node `WebSite` trong `@graph` (tham chiếu treo) |
| `Service` (`#about`) | name, description, url, category `Dimmer & Relay chiếu sáng`, provider, areaServed (Country Việt Nam, identifier VN) | |
| `ItemList` (`#products`) | name `Sản phẩm Dimmer Triac tại KNXStore`, `numberOfItems: 24`, `itemListElement[24]` mỗi phần tử chỉ có `@type`, `position` (1–24), `url` | không có `name`, `image`, `item`, giá cho từng phần tử; `numberOfItems` = số card trang hiện tại, không phải 29 |
| `FAQPage` (`#faq`) | `mainEntity` 5 `Question` (name + `acceptedAnswer.text`) | khớp 5 `details` trong DOM |

Không có `Product`, `Offer`, `AggregateRating`, `Review` trong trang danh mục.

Trang 2 (`?page=2`): cùng 5 node; `CollectionPage.name/description` không đổi (không ghi số trang); `ItemList.numberOfItems: 5`, `position` 1–5 (đánh lại từ 1); `FAQPage` lặp lại.

## 9. Số đo

| Chỉ số | Trang 1 (`/dimmer-doc-lap-triac`) | `?page=2` | `?view=list` | `?sort=price-asc` |
|---|---|---|---|---|
| HTML thô | 535,241 B | 396,398 B | 572,738 B | 535,918 B |
| gzip -9 (đo cục bộ) | 76,180 B | 66,064 B | 77,745 B | 76,463 B |
| br trên đường truyền | 52,721 B | không đo | không đo | không đo |
| Tổng byte thẻ `<script>` | 245,134 B (45.8%) | 185,971 B (46.9%) | 257,203 B (44.9%) | 245,462 B (45.8%) |
| Trong đó `self.__next_f` (trang 1) | 233,399 B (2 thẻ, 43.6%) | – | – | – |
| Text trong DOM (bỏ script/style/template; gồm panel `hidden`, drawer, `#S:0`) | 14,828 ký tự, 18,305 B | 13,265 ký tự, 16,572 B | 16,298 ký tự, 19,986 B | 14,836 ký tự |
| Text trong `#main-content` | 9,221 ký tự, 11,341 B | 7,658 ký tự, 9,608 B | 10,680 ký tự, 13,005 B | 9,229 ký tự |
| Ước lượng token (bytes/4): HTML thô | 133,810 | 99,099 | 143,184 | 133,979 |
| Ước lượng token: text toàn body | 4,576 | 4,143 | 4,996 | 4,577 |
| Ước lượng token: text `#main-content` | 2,835 | 2,402 | 3,251 | 2,836 |

Thẻ (trang 1): `<a>` 254, `<img>` 94 (67 trong header, 24 card, 3 footer), `<table>` 1, `<h1>` 1, `<h2>` 13, `<h3>` 22, h4–h6 0, `<button>` 45, `<input>` 22, `<form>` 1, `<svg>` 64, `<li>` 204, `<article>` 24, `<nav>` 11, `<details>` 10 (5 FAQ trong main, 2 nhóm lọc trong `#S:0`, 3 footer mobile), `<section>` 2, `<script>` 20 (6 inline, 14 src).

Phân bố text trong `#main-content` (9,221 ký tự = 125 breadcrumb/h1/chip + 1,966 lưới card + 7,130 section văn bản): văn bản SEO `div.user-html` 5,714 ký tự (62%), FAQ 1,401 ký tự (15%, nằm trong cùng section), lưới 24 card 1,966 ký tự (21%).

Thứ tự heading: 18 h3 trong header (panel ẩn) → `h1` "Dimmer Triac" → `h2` "Về Dimmer Triac" → h2 con trong văn bản SEO: "Dimmer Triac là gì và hoạt động theo nguyên lý nào", "Ưu và nhược điểm của Dimmer Triac", "Phân loại Dimmer Triac theo giao thức điều khiển", "Cách chọn Dimmer Triac phù hợp với đèn LED và hệ thống đang dùng", "Một số dòng Dimmer Triac tiêu biểu tại KNXStore" (kèm 4 h3: "Dòng độc lập, giá phổ thông", "Dòng tích hợp KNX", "Dòng tích hợp Casambi BLE", "Dòng tích hợp DALI và DMX"), "Vì sao nên mua Dimmer Triac tại KNXStore" → `h2` "Câu hỏi thường gặp về dimmer triac" → (ngoài main) `h2` "Yêu cầu báo giá" (drawer), 3 `h2` footer, `h2` "Bộ lọc" (trong `#S:0`). Trong văn bản SEO, `h2` "Về Dimmer Triac" và 6 `h2` con cùng cấp (không có h2 → h3 lồng cho 6 mục đó).

## 10. Biến thể template / Chưa xác định

Chỉ khảo sát 1 danh mục (+ 3 biến thể query). Các điểm khác biệt đã thấy giữa các biến thể URL của cùng danh mục:

| Biến thể | Khác biệt đã quan sát |
|---|---|
| `?page=2` | 5 card thay vì 24; title thêm " — trang 2"; canonical `?page=2`; `Trước` thành link `rel=prev`, `Sau` thành span `aria-disabled`; JSON-LD `ItemList` 5 phần tử; văn bản SEO, FAQ, h1, "29 sản phẩm" lặp nguyên |
| `?view=list` | grid 1 cột, card có checkbox so sánh và class `product-row-*`; canonical về URL gốc; link phân trang giữ `view=list` (`?view=list&page=2`) |
| `?sort=price-asc` | thứ tự card theo giá tăng, `option[value=price-asc][selected]`; canonical về URL gốc |

Chưa xác định:

| Điểm | Lý do |
|---|---|
| Template của 90 danh mục còn lại (danh mục cha có danh mục con, danh mục không có sản phẩm, danh mục không có văn bản SEO/FAQ, danh mục skin Matter như `/den-thong-minh-matter`) | chỉ khảo sát 1 URL; `/den-thong-minh-matter` chỉ thấy qua breadcrumb của trang sản phẩm Matter, chưa fetch |
| Quan hệ của 4 chip phía trên lưới (con hay anh em của danh mục) | HTML chỉ có link; `Dimmer DMX` trỏ `/dmx`, khác kiểu slug của 3 chip còn lại |
| Hành vi khi `?page` vượt tổng số trang, `?page=0`, query lạ | không thử |
| Tên query param và hành vi khi áp dụng bộ lọc thương hiệu/giải pháp/thanh lý | filter là input radio/checkbox không có URL; không chạy JS |
| Lý do route dynamic (đọc cookies/searchParams hay `no-store`) | chỉ thấy header |
| Hành vi sau `$RC` (vị trí toolbar, trạng thái drawer) | không chạy trình duyệt; suy ra từ `<template id="B:0">` + `$RC("B:0","S:0")` |
| Số card/trang là cấu hình cố định 24 hay đổi theo điều kiện | chỉ thấy 24 (trang 1) và 5 (trang 2, đủ 29) |
| Cookie `knx-area` (script đặt `knx-area=b2b`) có làm đổi nội dung SSR không | không gửi request có cookie |
| `<title>`/h1 của danh mục lấy từ trường nào (h1 `Dimmer Triac`; JSON-LD `Service.category` là tên danh mục cha `Dimmer & Relay chiếu sáng`) | chỉ thấy kết quả render |
| Phiên bản Next.js chính xác | không có header nêu version |
