# Trang giải pháp — cấu trúc hiện tại

> URL mẫu: https://knxstore.vn/solution/giai-phap-khac · nguồn sitemap: https://knxstore.vn/sitemaps/brands.xml (qua robots.txt → /sitemap.xml; chứa 12 URL `/solution/{slug}`: an-ninh, casambi, dali, dmx, giai-phap-khac, homekit, hvac, knx, lutron, matter, mobus, zigbee; `/solution` gốc không có trong file này và nằm ngoài phạm vi) · số URL cùng loại: 12 · ngày khảo sát 2026-11-11

Cách đo: `curl -sSL -m 180 -A "Mozilla/5.0"` lấy HTML thô, phân tích bằng `python3 -I` trên file đã lưu; 4 request tới URL này (HTML thô, nén zstd, `Accept: text/markdown`, `.md`). Mẫu là `giai-phap-khac` ("Giải pháp khác"): trang này khác cấu trúc với 7 giải pháp có bài dài (xem mục 6 và 10).

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
| 1 | không gửi Accept-Encoding (lần lấy file phân tích) | 407,202 | 14.31 | 22.45 |
| 2 | `gzip, br, zstd` → zstd | 37,708 | 0.58 | 0.62 |
| 3 | `Accept: text/markdown` | 407,202 | không ghi | 0.74 |

Lần đầu chậm (TTFB 14.3 s, tổng 22.5 s); hai lần sau dưới 1 s. Nguyên nhân: Chưa xác định.

### 1.3 Chế độ render suy ra: dynamic SSR theo request

Bằng chứng:
- `cache-control: private, no-cache, no-store, max-age=0, must-revalidate`; không có `x-nextjs-cache`, `x-nextjs-prerender`, `x-nextjs-stale-time` (trang chủ có cả ba).
- Font và CSS preload nằm trong header `Link` (trang chủ không có `Link` header).
- Cây route trong payload: `(site)/(b2b)/solution/[slug]/__PAGE__`, tham số `["slug","giai-phap-khac","d",[]]` (segment động).
- Không có ranh giới Suspense đang chờ: toàn bộ nội dung chính nằm inline trong HTML (không có `<template id="B:…">`/`div#S:…`, khác /brands/protopixel).

### 1.4 Hành vi hiện tại với yêu cầu Markdown (chỉ ghi kết quả)

| Request | Status | content-type | Ghi chú |
|---|---|---|---|
| `GET /solution/giai-phap-khac` + `Accept: text/markdown` | 200 | `text/html; charset=utf-8` | body 407,202 B, giống bản thường; `vary` không có `Accept` |
| `GET /solution/giai-phap-khac.md` | 404 | `text/html; charset=utf-8` | body 131,427 B (trang 404 HTML) |

## 2. `<head>`

Kích thước `<head>`: 14,018 B (8 thẻ preload ảnh: 10,790 B). Font không preload trong `<head>` mà qua header `Link`.

| Thành phần | Giá trị |
|---|---|
| `<html>` | `lang="vi"`, class `onest_…__variable inter_…__variable jetbrains_mono_…__variable` |
| title (60 ký tự) | `KNXStore - Nhà phân phối thiết bị điện thông minh chính hãng` (không chứa tên giải pháp) |
| meta description (56 ký tự) | `Tham khảo ngay các thiết bị điện thông minh tại KNXStore` |
| meta author / keywords | `KNX Store` / `Giải pháp khác` |
| canonical | `https://knxstore.vn/solution/giai-phap-khac` |
| meta robots | không có |
| OG | `og:title` (= title), `og:description` (= description), `og:url` (= canonical), `og:image` `https://api.knxstore.vn/media/f29c12ba547a4caaedabbbd7612997697f5096439fbfd004b158b38d0fb4aa20.jpg` (cùng ảnh cover hero), `og:type` `website`. Không có `og:site_name`, `og:locale`, `og:image:width/height/alt` |
| Twitter | `twitter:card` `summary_large_image`, `twitter:title`, `twitter:description`, `twitter:image` (cùng ảnh cover) |
| hreflang / `link rel=alternate` | không có |
| khác | `theme-color` `#14467C`; `next-size-adjust` rỗng; `link[rel=manifest]`; 3 icon (favicon.ico, icon.svg, apple-touch-icon) giống trang chủ |
| preload | `/images/logo/knxstore-logo-tight.svg`; 8 `link[rel=preload][as=image][imageSrcSet]` (ảnh 8 card sản phẩm nổi bật: 3 avif `2018ef89…`, `719918da…`, `139aa30b…` + 5 jpg `3ff30916…`, `f326b8d3…`, `f1ed36ba…`, `f8128559…`, `d25c11f9…`); script `3ivcc6t1s04dh.js` (`fetchpriority=low`); `https://www.googletagmanager.com/gtag/js?id=G-4D9XLJDVGG` |
| CSS / script | 2 `stylesheet` (`1mmmxvxvvtkv8.css`, `1vbr8oh4939ca.css`); 11 `<script async src>` + 1 `nomodule` |

## 3. Khung trang

### 3.1 Thứ tự con trực tiếp của `<body>`

| # | Phần tử | Ghi chú |
|---|---|---|
| B0 | `div[hidden]` chứa `<!--$--><!--/$-->` | marker Suspense rỗng |
| B1 | `<script>` inline ghi cookie `knx-area=b2b` | giống trang chủ |
| B2 | `a[href=#main-content]` | skip link |
| B3 | `header.sticky.top-0.z-9999.bg-white` | 128,755 B |
| B4 | `div#main-content[tabindex=-1]` | vùng chính; không có `<main>` |
| B5 | `div.fixed.top-0.left-0.z-99999…translate-x-full[inert]` | drawer "Yêu cầu báo giá" |
| B6 | `footer.mt-auto` | 17,608 B |
| B7 | `a[data-floating-zalo]` | nút Zalo nổi |
| B8 | `<script>` (payload RSC, chunk) | |

### 3.2 Landmark và role

| Loại | Chi tiết |
|---|---|
| `<header>` / `<footer>` | 1 / 1 |
| `<main>` | 0 (nội dung chính nằm trong `div#main-content`, không có `role=main`) |
| `<aside>` | 0 |
| `<nav>` (10) | header 6, footer 3, breadcrumb 1 (`nav[aria-label=Breadcrumb]`) |
| `role` | `search` 1, `combobox` 1 (`input#b2b-header-search`), `group` 1, `region` 1 (`#b2b-trending-panel`) |

### 3.3 Header, footer, thành phần nổi

Header và footer giống hệt byte-by-byte trang chủ và /brands/protopixel (md5 trùng). Tóm tắt (chi tiết đầy đủ ở `01-home.md` mục 3):

- Header: thanh trên gồm `button[aria-label="Mở menu"]` (mobile), logo `/`, `form[role=search]` với `input#b2b-header-search` (combobox) và chip phạm vi tìm kiếm (Tất cả, Sản phẩm, Bài viết, Thương hiệu, Giải pháp, Danh mục, Visual), giỏ `a[href=/cart][aria-label="Giỏ báo giá, 0 sản phẩm"]`, CTA `/contact` "Tư vấn dự án". Thanh nav (chỉ hiện từ `lg`): 6 button (Trending, Danh mục sản phẩm, Giải pháp, Thương hiệu, Công cụ, Blog kỹ thuật) + Visual `/visual` + pill "Matter Smarthome" `/matter`. 6 panel mega menu có sẵn trong HTML với class `hidden`: trending, danh mục (12 nhóm, 59 `<a>`), giải pháp (12 mục: An ninh, Casambi, DALI, DMX, Giải pháp khác, HomeKit, HVAC, KNX, Lutron, Matter, Mobus, Zigbee, mỗi mục xuất hiện 2 lần; "Xem tất cả giải pháp" `/solution`), thương hiệu (24 logo + "Xem tất cả 75 thương hiệu"), công cụ (3), blog (9 chuyên mục + 9 bài). Không có hotline trong header.
- Footer: thông tin công ty (CÔNG TY CP TÍCH HỢP HỆ THỐNG LIÊN MINH, MST 0311846236, địa chỉ The Sun Avenue, `tel:0918918755`, `mailto:support@knxstore.vn`, Facebook, Zalo, 2 huy hiệu), 3 cột link (Danh Mục Sản Phẩm 7, Giải Pháp 7, Về chúng tôi 5) render 2 lần (mobile accordion + desktop), thanh dưới © 2026 + 4 link chính sách/bảo hành.
- Thành phần nổi: nút Zalo `a[data-floating-zalo][href=https://zalo.me/0918918755]` (52x52, `#0068ff`, `animate-ping`); drawer báo giá `div.fixed…[inert]` (trạng thái rỗng). Không thấy cookie banner, popup, back-to-top, widget chat khác trong HTML ban đầu.

## 4. Cây nội dung vùng chính

Vùng chính: `div#main-content[tabindex=-1]` (1,748 ký tự text, 43 `<a>`, 12 `<img>`). Theo thứ tự DOM:

| # | Thẻ + id/class (rút gọn) | Heading | Nội dung tóm tắt | Hiển thị mặc định | SSR |
|---|---|---|---|---|---|
| 0 | `script[type=application/ld+json]` (con đầu) | | `@graph` 4 node (mục 8) | không hiển thị | có |
| 1 | `div.bg-zinc-50 > div.mx-auto > nav[aria-label=Breadcrumb]` | | "Trang chủ" (`a[/]`) / "Giải pháp" (`span`, không phải link) / "Giải pháp khác" (`span`) | có | có |
| 2 | `section.knx-hero` (nền trang trí `div.knx-light-field[aria-hidden=true]` gồm `knx-warm`, `knx-grain`, `knx-cool`) | h1 "Giải pháp Giải pháp khác" (`h1.text-h1-mobile > span`) | 2 CTA: `a[href=/products?solution=giai-phap-khac]` "Xem sản phẩm", `a[href=/contact]` "Tư vấn dự án"; ảnh cover `div.aspect-[4/3] > img[alt=""]` (`/media/f29c12ba….jpg`, `loading=lazy`). Không có đoạn mô tả | có | có |
| 3 | `section.border-t.border-zinc-200.bg-white.py-8` > `div.grid.grid-cols-2…lg:grid-cols-4` | h2 "Thiết bị Giải pháp khác theo loại" | 14 ô danh mục `a.flex.min-h-[92px]` (tên + "N sản phẩm"), bảng 4.1 | có | có |
| 4 | `section.border-t…py-8` > `div.mt-4 > div.grid…lg:grid-cols-4` | h2 "Sản phẩm Giải pháp khác nổi bật" (3 text node: "Sản phẩm", "Giải pháp khác", "nổi bật") | `a[href=/products?solution=giai-phap-khac]` "Xem tất cả"; 8 `article` card, bảng 4.2 | có | có |
| 5 | `section.border-t…py-8` > `div.-mx-4.mt-4.flex.snap-x…sm:grid.sm:grid-cols-4` | h2 "Thương hiệu có hàng" | 12 chip `a.flex.w-[140px]` (tên hãng + "N sản phẩm"), bảng 4.3; trên mobile cuộn ngang (`overflow-x-auto snap-x`), đủ 12 chip trong DOM | có | có |
| 6 | `section.bg-zinc-50` | h2 "Bài viết kỹ thuật mới" | `a[href=/blogs]` "Xem blog"; 3 card bài: "Mở Rộng DALI-2 Ra Dimmer Triac, Relay, DMX Và Modbus TCP" 06/11/2026; "Line Segment Và Line Coupler Trong KNX: Chia Line Thế Nào" 02/11/2026; "Dự án điều khiển chiếu sáng Casambi cho nhà hàng tại 79 Mạc Thị Bưởi, TP.HCM" 29/10/2026 (ảnh `alt=""`; giống hệt 3 bài ở trang chủ, không lọc theo giải pháp) | có | có |
| 7 | `section.bg-white.pb-10` > `div.knx-hero.overflow-hidden.rounded-card.p-6` | không | `p.text-h3` "Cần tư vấn giải pháp Giải pháp khác cho dự án?" + `a[href=/contact]` "Tư vấn dự án" | có | có |

Không có trong trang: đoạn mô tả giải pháp, sơ đồ/diagram, bài dài (nội dung HTML của giải pháp), FAQ, bảng thông số, `table`, `details`, tab/accordion trong vùng chính.

### 4.1 Ô danh mục (mục 3 của bảng trên): 14 ô

| # | Tên | Số | URL |
|---|---|---|---|
| 1 | Điều khiển máy lạnh | 36 | `/dieu-khien-may-lanh` |
| 2 | Cảm biến | 34 | `/cam-bien-hien-dien` |
| 3 | Thiết bị điện | 22 | `/thiet-bi-dien` |
| 4 | Dimmer & Relay chiếu sáng | 12 | `/dieu-khien-chieu-sang` |
| 5 | Khóa cửa thông minh | 9 | `/khoa-cua-thong-minh` |
| 6 | LED Driver & Nguồn | 7 | `/led-driver-1` |
| 7 | Phụ kiện cho hệ thống | 6 | `/thiet-bi-he-thong` |
| 8 | Công tắc thông minh | 5 | `/cong-tac-thong-minh` |
| 9 | Bộ điều khiển trung tâm | 3 | `/bo-dieu-khien-trung-tam` |
| 10 | Thiết bị an ninh | 3 | `/kiem-soat-ra-vao` |
| 11 | Âm thanh đa vùng | 1 | `/am-thanh-da-vung` |
| 12 | Đèn chiếu sáng | 1 | `/den-chieu-sang` |
| 13 | Điều khiển Matter | 1 | `/san-pham-matter` |
| 14 | Led SPI | 1 | `/led-spi` |

Tổng số trong 14 ô: 141.

### 4.2 Card "Sản phẩm nổi bật": 8 card

| # | Hãng (badge) | Model | Giá hiển thị |
|---|---|---|---|
| 1 | Tongdy | TF-9 | Liên hệ |
| 2 | HUGO MULLER | Paladin 722 010 | Giá liên hệ |
| 3 | 1Home | 1Home Server Loxone | 26.710.000 đ |
| 4 | HUGO MULLER | SC-2813P4 | Liên hệ |
| 5 | HUGO MULLER | SC-2811P4 | Liên hệ |
| 6 | HUGO MULLER | P173110E | Liên hệ |
| 7 | HUGO MULLER | EA-3622P4 | Liên hệ |
| 8 | HUGO MULLER | SC-3820P230 | Liên hệ |

URL card: `/products/tram-quan-trac-chat-luong-khong-khi-ngoai-troi-tongdy-tf-9`, `/products/cam-bien-chuyen-dong-pir-gan-tran-360-hugo-muller-paladin-722-010-pir`, `/products/gateway-loxone-qua-matter-1home-server-loxone`, `/products/cong-tac-hen-gio-dien-tu-muller-sc-28-x3-pro4-paladin-172-4x3-pro4`, `/products/cong-tac-hen-gio-dien-tu-muller-sc-28-x1-pro-4`, `/products/cong-tac-hen-gio-dien-tu-am-tuong-paladin-173-110-easy`, `/products/cong-tac-hen-gio-am-tuong-muller-ea-3622p4`, `/products/cong-tac-hen-gio-dien-tu-muller-sc-38-x0-pro5`. Không có phân trang (nút "Xem tất cả" dẫn sang `/products?solution=giai-phap-khac`).

### 4.3 Chip "Thương hiệu có hàng": 12 chip

| Tên | Số sản phẩm | URL |
|---|---|---|
| Airzone | 25 | `/brands/airzone` |
| IR-TEC | 24 | `/brands/ir-tec` |
| Bticino | 10 | `/brands/bticino` |
| Legrand | 10 | `/brands/legrand` |
| Moorgen | 10 | `/brands/moorgen` |
| Vadsbo | 8 | `/brands/vadsbo` |
| HUGO MULLER | 6 | `/brands/hugo-muller` |
| OEM | 6 | `/brands/oem` |
| CP Electronics | 5 | `/brands/cp-electronics` |
| Daikin | 5 | `/brands/daikin` |
| Steinel | 5 | `/brands/steinel` |
| DALCNET | 3 | `/brands/dalcnet` |

Chip không có logo; nhãn là chữ.

## 5. Chi tiết khối đặc biệt

### 5.1 Hero (H1 lặp chữ "Giải pháp")

```html
<section class="knx-hero"><div class="knx-light-field" aria-hidden="true"><div class="knx-warm"></div><div class="knx-grain"></div><div class="knx-cool"></div></div><div class="relative z-10 mx-auto w-full …"><div class="flex flex-col gap-6 lg:flex-row …"><div class="lg:w-[52%] lg:shrink-0"><h1 class="text-h1-mobile text-ink-inverse"><span >Giải pháp Giải pháp khác</span></h1><div class="mt-6 flex …"><a … href="/products?solution=giai-phap-khac">Xem sản phẩm</a><a … href="/contact">Tư vấn dự án</a></div></div><div class="min-w-0 lg:flex-1"><div class="relative aspect-[4/3] …"><img alt="" loading="lazy" … src="/_next/image?url=%2Fmedia%2Ff29c12ba….jpg&amp;w=3840&amp;q=75"/></div></div></div></div></section>
```

Chữ "Giải pháp " được ghép trước tên giải pháp ("Giải pháp khác"), nên H1, tiêu đề các khối và câu CTA đều lặp ("Thiết bị Giải pháp khác theo loại", "Sản phẩm Giải pháp khác nổi bật", "Cần tư vấn giải pháp Giải pháp khác cho dự án?").

### 5.2 Chip thương hiệu

```html
<h2 class="text-h3 text-zinc-900">Thương hiệu có hàng</h2><div class="-mx-4 mt-4 flex snap-x gap-3 overflow-x-auto px-4 pb-2 sm:mx-0 sm:grid sm:grid-cols-4 …"><a class="flex w-[140px] shrink-0 snap-start flex-col items-center justify-center gap-1 rounded-con…" href="/brands/airzone"><span class="text-[13px] font-semibold …">Airzone</span><span class="font-mono text-chip tabular-nums text-zinc-500">25<!-- -->sản phẩm</span></a>…
```

### 5.3 CTA cuối trang

```html
<section class="bg-white pb-10"><div class="mx-auto w-full …"><div class="knx-hero overflow-hidden rounded-card p-6"><div class="knx-light-field" aria-hidden="true">…</div><div class="relative z-10 flex …"><p class="text-h3 text-ink-inverse">Cần tư vấn giải pháp Giải pháp khác cho dự án?</p><a … href="/contact">Tư vấn dự án</a></div></div></div></section>
```

### 5.4 Card sản phẩm

Cùng cấu trúc `article.group.relative.rounded-card…` như trang chủ (xem `01-home.md` mục 5.2): `div.aspect-square > img`, badge hãng + `span.model-code`, `a.after:absolute.after:inset-0` (tiêu đề, không phải heading), `div` giá, `button[aria-label="Thêm … vào yêu cầu báo giá"]`. Cả 8 ảnh eager và được preload trong `<head>`.

### 5.5 Cấu trúc payload RSC

- 1 lần `push([1,"…"])` (+ `push([0])`), giải mã 167,018 B, 100 row (99 JSON + 1 `T` là JSON-LD 5,073 B).
- 2 row layout (id `5` và `11`, 29,467 B và 29,468 B) mang prop `nav` của `HeaderB2b`, tổng 58,935 B (35% payload giải mã).
- Row id `6` (4,259 B) chứa breadcrumb, hero và khối ô danh mục; row `4e` là section sản phẩm nổi bật, `4f` thương hiệu có hàng, `50` bài viết, `51` CTA cuối; props các card sản phẩm nằm ở row `4e` và `56`–`5c`; row id `2c` là cây 404.
- Không có trường `contentHtml`, `summary`/`seoTitle` của giải pháp trong payload của trang này (0 kết quả cho `contentHtml`, `seoTitle`, `productCount`).

## 6. Nội dung chỉ có ở client hoặc KHÔNG thấy trong HTML ban đầu

| # | Nội dung | Có trong HTML ban đầu? | Bằng chứng |
|---|---|---|---|
| 1 | Bài giải pháp dài (`contentHtml`) | Không có trên trang này (không có cả trong payload RSC của trang). Với 7 giải pháp khác (knx, casambi, dmx, an-ninh, matter, dali, hvac), `contentHtml` có trong payload RSC của trang chủ (xem `01-home.md` mục 6), nhưng cách `/solution/{slug}` render nó thì Chưa xác định vì chưa tải các trang đó | `"contentHtml"` = 0 kết quả trong payload trang này |
| 2 | Danh sách sản phẩm đầy đủ của giải pháp (141 sản phẩm theo tổng 14 ô danh mục) | Không: chỉ 8 card nổi bật; còn lại ở `/products?solution=giai-phap-khac` | HTML: 8 `article` |
| 3 | Trường sản phẩm không render (`stockStatus`, `oldPrice`, `b2c`, `b2b`, `priceHidden`…) | Không | payload RSC (prop các card) |
| 4 | Gợi ý tìm kiếm | Không | `input#b2b-header-search[aria-expanded=false]`, không có `listbox` |
| 5 | Trạng thái giỏ báo giá | Chỉ trạng thái rỗng | "Giỏ báo giá, 0 sản phẩm"; "Chưa có thiết bị nào trong yêu cầu." |
| 6 | Menu mobile (sau nút "Mở menu") | Không | chỉ có `button[aria-label="Mở menu"]` |
| 7 | Cây 404 | Không (chỉ trong payload RSC) | row id `2c` ("Không tìm thấy trang") |
| 8 | Script GA4 | Không (chỉ preload) | `gtag/js?id=G-4D9XLJDVGG` |

Mọi khối chính của trang (hero, danh mục, sản phẩm, thương hiệu, blog, CTA) có text đầy đủ trong HTML ban đầu. Mega menu header có text và link trong HTML nhưng ẩn (`class=hidden`). Không thấy skeleton: 0 `<template data-dgst`, 0 `BAILOUT_TO_CLIENT_SIDE_RENDERING`, 0 `animate-pulse`, 0 Suspense đang chờ (`<!--$?-->` = 0).

## 7. Liên kết

Tổng 262 `<a>`: skip link 1, header 165, `div#main-content` 43, drawer 2, footer 50, nút Zalo 1.

| Vùng | Mẫu URL nội bộ |
|---|---|
| `div#main-content` (43; 41 href duy nhất) | `/{category-slug}` 16 (14 ô danh mục + 2 lần `/contact`); `/brands/{slug}` 12; `/products/{slug}` 8; `/{slug}.html` 3 (bài blog); `/products?solution=giai-phap-khac` 2; `/` 1; `/blogs` 1 |
| header | như trang chủ: `/{category-slug}` 62, `/solution/{slug}` 24, `/brands/{slug}` 24, `/products/{slug}` 16, `.html` 16 (14 bài blog + 2 `/tool/…/index.html`), `/blogs[/{cat}]` 10, `/search?q=` 8, còn lại 1 mỗi loại |
| footer | như trang chủ (50 `<a>`, 31 href duy nhất; 5 link ngoài: Google Maps, Facebook, Zalo, online.gov.vn, DMCA) |

- Link ngoài trong vùng chính: 0. Nút Zalo nổi: `https://zalo.me/0918918755`.
- Không có link tới các trang `/solution/{slug}` khác trong vùng chính (chỉ có trong header/footer).
- Link rel=alternate/markdown: không có.
- Tham số truy vấn lọc theo giải pháp: `/products?solution=giai-phap-khac` (2 lần).

## 8. JSON-LD

1 thẻ `script[type=application/ld+json]` (4,890 ký tự JSON, 5,108 B kèm thẻ), con đầu của `div#main-content`. Một `@graph` gồm 4 node:

| # | `@type` | `@id` | Trường có mặt |
|---|---|---|---|
| 0 | `["Organization","LocalBusiness"]` | `https://knxstore.vn/#organization` | giống trang chủ: `name`, `legalName`, `url`, `foundingDate`, `taxID`, `logo` (`https://knxstore.vn/assets/logo.png` 150x50), `image` (1200x630), `description`, `telephone`, `email`, `address`, `geo`, `hasMap`, `openingHoursSpecification` (Mon–Sat 09:00–17:30), `priceRange`, `areaServed`, `knowsAbout` (10), `contactPoint`, `sameAs` (Facebook, YouTube) |
| 1 | `CollectionPage` | `https://knxstore.vn/solution/giai-phap-khac#webpage` | `url`, `name` "Giải pháp khác", `description` "Tham khảo ngay các thiết bị điện thông minh tại KNXStore", `inLanguage`, `datePublished` và `dateModified` đều "2026-08-24T14:20:16.977Z", `isPartOf` (ref `https://knxstore.vn/#website`), `publisher` (ref `#organization`), `mainEntity` (ref `…#products`), `about` (ref `…#about`), `primaryImageOfPage` (ImageObject `…#primary-image`, `url`/`contentUrl` = `api.knxstore.vn/media/f29c12ba….jpg`, `caption` "Giải pháp khác"), `breadcrumb` (BreadcrumbList 2 `ListItem`: Trang chủ → `https://knxstore.vn/`, Giải pháp khác → URL trang) |
| 2 | `Service` | `https://knxstore.vn/solution/giai-phap-khac#about` | `name` "Giải pháp khác", `description` (như trên), `url`, `provider` (ref `#organization`), `areaServed` (`Country`: `name` "Việt Nam", `identifier` "VN") |
| 3 | `ItemList` | `https://knxstore.vn/solution/giai-phap-khac#products` | `name` "Sản phẩm Giải pháp khác tại KNXStore", `numberOfItems` 8, `itemListElement` 8 `ListItem` (`position`, `url`) |

Ghi nhận:
- `BreadcrumbList` trong JSON-LD có 2 mục (Trang chủ → Giải pháp khác); breadcrumb hiển thị có 3 mục (Trang chủ / Giải pháp / Giải pháp khác, mục giữa không phải link).
- `isPartOf` tham chiếu `https://knxstore.vn/#website` nhưng graph của trang này không có node `WebSite` với `@id` đó.
- `numberOfItems: 8` bằng số card nổi bật hiển thị; tổng số trong 14 ô danh mục là 141.
- `ListItem` chỉ có `position` và `url`; không có `name`, `image`, `offers`; không có node `Product`, `FAQPage`, `Article`.
- `description` giống meta description (câu chung, không nêu nội dung giải pháp). `datePublished` và `dateModified` trùng nhau.
- Không có trường rỗng.

## 9. Số đo

| Chỉ số | Giá trị |
|---|---|
| HTML thô | 407,202 B |
| gzip (đo local, mức 6 / 9) | 61,070 B / 60,851 B |
| zstd phía server | 37,708 B |
| brotli | Chưa xác định (không đo được) |
| `<head>` | 14,018 B |
| `<body>` | 393,009 B |
| `<header>` | 128,755 B (31.6% HTML) |
| `<footer>` | 17,608 B |
| Tổng `<script>` (17 thẻ) | 189,412 B (46.5%) |
| Script inline `self.__next_f` (2 thẻ) | 183,153 B; giải mã 167,018 B |
| Script `ld+json` (1) | 5,108 B |
| `<style>` | 0 |
| Thuộc tính `style=""` (39) | 2,027 B |
| `<svg>` inline (41) | 16,184 B |
| Thuộc tính `srcset/srcSet` (36) | 58,154 B |
| Thuộc tính `class` (906) | 82,126 B |
| HTML sau khi bỏ mọi `<script>` | 217,790 B |
| Text hiển thị `<body>` | 6,960 ký tự / 8,632 B |
| Text `div#main-content` | 1,748 ký tự / 2,154 B |
| Text ngoài vùng chính (header, footer, drawer, Zalo, skip link) | 5,212 ký tự (74.9% text hiển thị) |
| Ước lượng token (bytes/4) | HTML thô 101,800; text hiển thị 2,158; text vùng chính 538 |
| Thẻ | `<a>` 262, `<img>` 82 (72 lazy, 10 eager; 46 `alt=""`), `<svg>` 41, `<table>` 0, `<button>` 25, `<form>` 1, `<input>` 1, `<section>` 6, `<article>` 8, `<nav>` 10, `<aside>` 0, `<ul>` 31, `<li>` 173, `<details>` 3 (footer), `<template>` 0 |
| Heading | h1 ×1, h2 ×8, h3 ×18, h4–h6 ×0 |

Phân bố heading: h1 "Giải pháp Giải pháp khác" và 4 h2 ("Thiết bị Giải pháp khác theo loại", "Sản phẩm Giải pháp khác nổi bật", "Thương hiệu có hàng", "Bài viết kỹ thuật mới") trong `div#main-content`; h2 "Yêu cầu báo giá" (drawer) và 3 h2 footer; 18 h3 trong mega menu header. Card sản phẩm, ô danh mục, chip thương hiệu và CTA cuối không có thẻ heading (CTA dùng `p.text-h3`).

## 10. Chưa xác định / lưu ý

Chưa xác định:
- Cách các `/solution/{slug}` có bài dài (knx, casambi, dmx, an-ninh, matter, dali, hvac) render `contentHtml` (vị trí, heading, bảng, ảnh, FAQ, JSON-LD kèm theo): chỉ biết dữ liệu có trong payload RSC trang chủ; chưa tải các trang đó (ngoài phạm vi). Với homekit, lutron, mobus, zigbee: Chưa xác định cả có/không có `contentHtml`.
- Nguyên nhân lần tải đầu chậm (TTFB 14.3 s).
- Dung lượng brotli; menu mobile; nội dung inject sau hydrate; gợi ý tìm kiếm sau khi gõ.

Lưu ý về tính đại diện:
- `giai-phap-khac` là mục "Giải pháp khác" (gom thiết bị không thuộc giao thức cụ thể): 14 danh mục và 12 hãng hiển thị khác nhau ở mỗi giải pháp; chỉ trang này có breadcrumb JSON-LD 2 mục, `Service`, `datePublished/dateModified` trong mẫu đã tải.
- Title và description không chứa tên giải pháp, và payload không có trường `seoTitle`; trong payload trang chủ, 7 giải pháp khác có trường `seoTitle` riêng (ví dụ "Thiết bị KNX c…"). Việc title/description của trang này có phải giá trị mặc định hay không: Chưa xác định.
- H1 và các tiêu đề khối lặp chữ "Giải pháp" (kết quả ghép chuỗi "Giải pháp " + tên "Giải pháp khác"); với giải pháp tên ngắn như "KNX" có khả năng khác: Chưa xác định.
- 3 bài blog cuối trang giống hệt trang chủ (không liên quan tới giải pháp).
- Không có `<main>`; `div#main-content` là mốc duy nhất cho vùng chính. Text vùng chính chỉ chiếm 25.1% text hiển thị, phần còn lại là header/footer/drawer.
- Header, footer, drawer, nút Zalo giống hệt byte-by-byte trang chủ và /brands/protopixel.
