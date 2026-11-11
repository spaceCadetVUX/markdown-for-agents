# Trang danh mục sản phẩm skin Matter — cấu trúc hiện tại

> https://knxstore.vn/cong-tac-matter · categories.xml (91 URL `/{slug}` ở root, không đuôi `.html`) · khảo sát 2026-11-11

Phương pháp: `curl` HTML thô (UA `Mozilla/5.0`, không cookie), phân tích bằng `html.parser`; giải mã payload `self.__next_f` bằng parser row của Flight; đối chiếu DOM với payload và JSON-LD. Có tải thêm 1 biến thể `?view=list` và 1 trang danh mục B2B (`/dimmer-doc-lap-triac`) để so khung trang. Số request vào chính URL này: 5 (identity, gzip, br, `Accept: text/markdown`, `.md`) + 1 (`?view=list`). Chưa chạy trình duyệt: mọi mô tả "sau hydrate" là suy ra từ HTML và payload.

## 1. Response & rendering

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
| link | có: 4 `rel=preload as="font"` woff2 (`051742360c26797e-s.p…`, `2bbe8d2671613f1f-s.p…`, `83afe278b6a6bb3c-s.p…`, `fa39153a3fc630ba-s.p…`) |
| x-robots-tag | không có |
| content-encoding | không có khi không gửi `Accept-Encoding`; `gzip` / `br` khi gửi tương ứng |
| set-cookie | không có |
| Header khác | `server: cloudflare`, `speculation-rules`, `strict-transport-security`, `x-frame-options: SAMEORIGIN`, `x-content-type-options: nosniff`, `referrer-policy: same-origin`, `alt-svc: h3`, `nel`/`report-to` |

Kích thước trên đường truyền: identity 303,087 B; gzip 51,259 B; br 39,651 B.

Chế độ render: dynamic (SSR theo request, không cache). Bằng chứng: `cache-control: private, no-cache, no-store` (không có `s-maxage`); không có `x-nextjs-cache`/`x-nextjs-prerender`; có header `link` preload font (trang ISR không có header này, font preload nằm trong `<head>`); record `0` của payload có `"S":false` (các trang ISR có `"S":true`; chỉ ghi nhận tương quan). Có Suspense streaming: khối toolbar được stream sau (xem mục 4 và 6). Trang danh mục B2B `/dimmer-doc-lap-triac` cũng có `cache-control` no-store, `"S":false` và cùng cơ chế `B:0`/`S:0`.

Route (record `0` của payload): `(site)/[slug]/__PAGE__`, `slug` là dynamic segment đứng cạnh các route tĩnh: `admin, api, author, blogs, brands, cart, clearance, compare, connection-check, contact, context, css, label, llms.txt, mail-success, matter, matter-search, order-success, page, products, robots.txt, search, sitemap.xml, sitemaps, solution, tools, track-order, visual, warranty, warranty-lookup`. Wrapper skin Matter (header + footer + router con) là record `5`, node con của segment `[slug]` (suy ra từ vị trí `$L5` trong record `0`). Phiên bản Next.js chính xác: Chưa xác định.

Hành vi hiện tại với yêu cầu markdown (chỉ ghi kết quả):

| Request | Status | content-type | Ghi chú |
|---|---|---|---|
| `Accept: text/markdown` | 200 | `text/html; charset=utf-8` | body 303,087 B, giống hệt bản không có `Accept`; `vary` không chứa `Accept` |
| URL + `.md` | 404 | `text/html; charset=utf-8` | body 131,151 B (trang lỗi Next), `cache-control: private, no-cache, no-store` |
| `?view=list` | 200 | `text/html; charset=utf-8` | 304,693 B; `canonical` vẫn là `/cong-tac-matter`; không có `meta robots` |

## 2. `<head>`

Head dài 11,852 B (lớn hơn trang sản phẩm vì có 6 `preload` ảnh).

| Thành phần | Giá trị |
|---|---|
| `<html>`, `<body>` | `lang="vi"`, 3 class biến font; `<body>` không attribute |
| title | `Công Tắc Matter Thông Minh – Aqara, JINK, Thread \| KNXStore` (59 ký tự) |
| meta description | `công tắc Matter tại KNXStore — Aqara H2 , JINK Thread kính cường lực, núm xoay không dây. Tương thích Apple HomeKit, Google Home & Alexa. Lắp thay công tắc…` (156 ký tự; bắt đầu bằng chữ thường, kết thúc bằng `…` tức bị cắt; có khoảng trắng thừa trước dấu phẩy) |
| meta keywords / author | `Công tắc Matter` / `KNX Store` |
| canonical | `https://knxstore.vn/cong-tac-matter` |
| robots | không có thẻ `meta name=robots` |
| OG (5) | `og:title` = `Công Tắc Matter Thông Minh – Aqara, JINK, Thread \| KNXStore`, `og:description` (= meta description), `og:url`, `og:image` = `https://api.knxstore.vn/media/abda0616…347c98.jpg`, `og:type=website`; không có `og:site_name`, `og:locale` |
| Twitter (4) | `summary_large_image`, title, description, image |
| hreflang / `rel=alternate` | không có / không có |
| Preload | 6 `<link rel=preload as=image imagesrcset=… imagesizes="(max-width: 480px) 50vw, (max-width: 1024px) 33vw, 290px">` (6 card đầu, `w=256…`, `q=75`); logo SVG; 1 script chunk (`fetchpriority=low`); `gtag/js?id=G-4D9XLJDVGG` (chỉ preload). Không có preload font trong `<head>` (font nằm ở header HTTP `link`) |
| Stylesheet | 2 (cùng file với trang sản phẩm: `1mmmxvxvvtkv8.css`, `1vbr8oh4939ca.css`) |
| Khác | `theme-color #14467C`, `manifest`, icon, `next-size-adjust` |
| JSON-LD | nằm trong body (mục 8) |

## 3. Khung trang

Header, menu mobile, footer, Zalo và drawer báo giá giống hệt trang sản phẩm skin Matter về cấu trúc và kích thước (header 9,358 B / 17 link; menu mobile 6,571 B / 15 link; footer 7,929 B / 21 link; drawer báo giá 2,585 B; Zalo 649 B). Nội dung chi tiết các mục này: xem mục 3 của `13-matter-product.md` (đã đo lại trên trang này, số liệu trùng).

Landmark thực tế:

| Phần tử | Vai trò |
|---|---|
| `div[hidden]` (con đầu `<body>`) | boundary rỗng của Next |
| `div.font-sans[style="--font-body:var(--font-matter)"]` | wrapper skin |
| `header[aria-label="Khu Matter Smarthome"]`, `nav[aria-label="Menu Matter"]`, `form[role=search]` (trong `#matter-search-drop`), `aside#matter-mobile-panel` + `nav[aria-label="Menu Matter trên điện thoại"]` | header, menu, tìm kiếm, menu mobile |
| `nav[aria-label="Đường dẫn"]` | breadcrumb (nằm trong thanh tiêu đề trang, cùng khối với H1) |
| `footer.mt-auto` | footer |
| `div#filter-panel-drawer[role=dialog][aria-modal=true][aria-label="Bộ lọc sản phẩm"][aria-hidden=true]` + `aside[aria-label="Bộ lọc"]` | drawer bộ lọc (nằm trong `div#S:0`, xem dưới) |

Không có `<main>`, không có `role=main`, không có `#main-content`, không có skip link. Nội dung chính là các khối anh em trong `div.font-sans`: `div.overflow-hidden.shadow-breadcrumb` (H1 + breadcrumb), `section.overflow-hidden.pb-20.bg-white` (lưới), `section.border-t.border-zinc-200` (văn bản SEO + FAQ). Thành phần nổi: nút Zalo `a[data-floating-zalo][href=https://zalo.me/0378628837]`; drawer báo giá `div[inert].fixed.z-99999`; menu mobile; drawer bộ lọc (ẩn). Không thấy cookie banner/popup/back-to-top trong HTML ban đầu.

## 4. Cây nội dung chính (theo thứ tự DOM)

Cột "Vùng": trang không có `<main>`, nên ghi "vùng nội dung" cho khối anh em trong wrapper, "khung" cho header/footer/nổi.

| # | Thẻ + selector rút gọn | Heading | Nội dung tóm tắt | Vùng | Hiển thị mặc định | SSR / client |
|---|---|---|---|---|---|---|
| 0 | `div[hidden]` | – | boundary rỗng | khung | ẩn | SSR |
| 1 | `div.font-sans` | – | wrapper skin; con đầu là `script` cookie `knx-area=matter` | khung | – | SSR |
| 2 | `header[aria-label="Khu Matter Smarthome"]` | – | menu, tìm kiếm, giỏ, "Trang B2B" | khung | hiện | SSR |
| 3 | `div.fixed.inset-0.z-[10000] > aside#matter-mobile-panel` | – | menu mobile | khung | ẩn | SSR |
| 4 | `script[type=application/ld+json]` (11,796 B) | – | 5 node (mục 8) | khung | không hiển thị | SSR (lặp trong RSC row `56`) |
| 5 | `div.overflow-hidden.shadow-breadcrumb > div.border-t > div.max-w-[1280px]` (965 B) | h1: "Công tắc Matter" | H1 + `nav[aria-label="Đường dẫn"] > ol > li`: Trang chủ `/matter` / Điều khiển Matter `/san-pham-matter` / `span[aria-current=page]` Công tắc Matter; dấu ngăn là chữ `/` trong `span[aria-hidden]` (nên text nối "Trang chủ/Điều khiển Matter/Công tắc Matter") | vùng nội dung | hiện | SSR |
| 6 | `section.overflow-hidden.pb-20.bg-white > div.max-w-[1280px].pt-10` | – | chứa placeholder Suspense `<!--$?--><template id="B:0"></template><!--/$-->` rồi `div.mt-6 > div.grid` | vùng nội dung | placeholder không hiển thị gì (fallback `null`) | SSR + streaming |
| 6a | `div.grid.grid-cols-2.items-stretch.gap-5.lg:grid-cols-3.xl:grid-cols-4` | mỗi card có 1 `h3` | 12 `article.group/card` (51,308 B cả section, 1,146 ký tự text) | vùng nội dung | hiện | SSR |
| 7 | `section.border-t.border-zinc-200.bg-white.py-8` (18,335 B) | h2: "Về Công tắc Matter" | khối văn bản SEO + FAQ | vùng nội dung | hiện | SSR |
| 7a | `div.user-html.mt-4.text-body.text-zinc-700` (8,999 B HTML, 6,280 ký tự text) | h2 ×5: "Công tắc Matter là gì?", "Lắp công tắc Matter có cần đi lại dây không?", "Thread và Zigbee — hai loại công tắc Matter trong cùng danh mục", "Các dòng công tắc Matter có tại KNXStore", "Mua công tắc Matter và hỗ trợ lắp đặt tại KNXStore"; h3 ×1: "Tích hợp vào hệ smarthome và kịch bản tự động" | 32 `p`, 21 `strong`, 1 `ul` (4 `li`), 1 `img[alt="matter protocol"]` (`/media/c8d97567….jpg`, `style="height:485px; width:900px"`, không `loading`); các tên dòng sản phẩm (Aqara H2, JINK S1…) là `p` chứ không phải heading | vùng nội dung | hiện | SSR; text khớp row `4c` của payload (kiểu `T`, 9,022 B) |
| 7b | `div.mt-8 > section.rounded-card > h2 + ul > li > details.group/faq` | h2: "Câu hỏi thường gặp về công tắc matter" | 10 `details` (đều đóng, không có `open`); `summary` chứa câu hỏi + `span` "+"; đáp án là `div.user-html` nằm trong `details` (có đầy đủ trong HTML) | vùng nội dung | câu hỏi hiện, đáp án gập | SSR; mở/đóng = hành vi native của `details` |
| 8 | `div.fixed.top-0.left-0.z-99999[inert]` | h2 "Yêu cầu báo giá" | drawer báo giá rỗng | khung | ẩn | SSR |
| 9 | `footer.mt-auto` | 3 h2 | xem `13-matter-product.md` mục 3 | khung | hiện | SSR |
| 10 | `a[data-floating-zalo]` | – | "Zalo" | khung | hiện | SSR |
| 11 | `script` (init), `script#_R_`, script `self.__next_f.push` (172,565 B) | – | payload RSC | – | – | – |
| 12 | `div#S:0[hidden]` — con trực tiếp của `<body>`, nằm sau `div.font-sans` và sau các script | h2 "Bộ lọc" (trong drawer) | toolbar bị stream: nút "Bộ lọc" (+ badge "0"), drawer bộ lọc, "12 sản phẩm", nút chuyển dạng xem, `select` sắp xếp (7,751 B, 258 ký tự text) | – | ẩn (`hidden`) cho đến khi script `$RC("B:0","S:0")` chuyển vào vị trí `template#B:0` | HTML có sẵn nhưng nằm ngoài vị trí logic; chuyển chỗ = client (inline script `$RC`) |
| 13 | script `$RC` (inline, ~853 B) + `requestAnimationFrame(...$RT...)` | – | điều phối Suspense | – | – | – |

Toolbar `div#S:0` (nội dung thật, theo thứ tự DOM):

| Phần tử | Nội dung |
|---|---|
| `button[aria-expanded=false][aria-controls=filter-panel-drawer]` | "Bộ lọc" + badge "0" |
| `div#filter-panel-drawer[role=dialog][aria-hidden=true]` | h2 "Bộ lọc"; `label > input[type=checkbox]` "Chỉ hàng thanh lý" (mô tả: "Sản phẩm đang bán giá thanh lý, hoặc còn suất hàng demo, trôi bảo hành, đã qua sử dụng."); `details[open]` "Thương hiệu" với `fieldset > ul > li > label > input[type=radio][name=Thương hiệu]`: Aqara 6, JINK 5, Sunricher 1; nút "Xoá lọc" (disabled), "Áp dụng" |
| `span` | "12 sản phẩm" |
| Chuyển dạng xem | `a[aria-label="Xem dạng lưới"][aria-current=true][href=/cong-tac-matter]`, `a[aria-label="Xem dạng bảng"][href=/cong-tac-matter?view=list]` |
| Sắp xếp | `label > span.sr-only "Sắp xếp"` + `select` 6 `option`: "Mặc định" (value rỗng, selected), "Mới nhất" `newest`, "Tên A → Z" `name-asc`, "Tên Z → A" `name-desc`, "Giá thấp → cao" `price-asc`, "Giá cao → thấp" `price-desc` |

Props của toolbar trong payload (row `6`): `showViewToggle:true`, `facets:[]`, `optionFilters:[]`, `clearance:{hidden:false,position:0}`, `brands:[{slug,name,count}]` ×3, `solutions:[{slug:"matter",name:"Matter",count:12}]`, `info:["12"," sản phẩm"]`. Component `Toolbar` bọc trong Suspense `fallback:null`.

12 card sản phẩm (theo thứ tự DOM; "stockStatus" lấy từ payload, "Nhãn DOM" là chữ trong card):

| # | Brand (DOM) | Mã model/SKU (payload) | Giá | Giá gạch / badge | stockStatus → nhãn DOM | `loading` ảnh |
|---|---|---|---|---|---|---|
| 1 | JINK | S1-04 SKIN-GREY | 2.850.000đ | – | `low_stock` → "Sắp hết hàng" | eager |
| 2 | JINK | S1-04 ALUMI-WHITE | 2.850.000đ | – | `low_stock` → "Sắp hết hàng" | eager |
| 3 | JINK | S1-04 AG-WHITE | 2.850.000đ | – | `low_stock` → "Sắp hết hàng" | eager |
| 4 | JINK | Z75-06 EGG-GREY | 2.990.000đ | – | `low_stock` → "Sắp hết hàng" | eager |
| 5 | Aqara | KD-R01D-W | 1.028.000đ | 1.142.000đ / −10% | `make_to_order` → "Đặt trước" | eager |
| 6 | Aqara | WS-K08D | 1.077.000đ | 1.196.000đ / −10% | `make_to_order` → "Đặt trước" | eager |
| 7 | Aqara | WS-K07D | 996.000đ | 1.106.000đ / −10% | `make_to_order` → "Đặt trước" | lazy |
| 8 | Aqara | WS-K04E | 1.109.000đ | 1.232.000đ / −10% | `make_to_order` → "Đặt trước" | lazy |
| 9 | Aqara | WS-K03E | 955.000đ | 1.061.000đ / −10% | `make_to_order` → "Đặt trước" | lazy |
| 10 | Aqara | WS-K02E | 955.000đ | 1.061.000đ / −10% | `make_to_order` → "Đặt trước" | lazy |
| 11 | JINK | Z75-06 EGG 3B-GREY | 2.990.000đ | – | `make_to_order` → "Đặt trước" | lazy |
| 12 | Sunricher | SR-MT2835RAC-NK4 | 1.541.000đ | – | `make_to_order` → "Đặt trước" | lazy |

Cấu trúc 1 card (`article.group/card.relative.rounded-lg`): `a[tabindex=-1][aria-hidden=true][href=/products/{slug}] > img[alt=""][data-nimg=fill]` (ảnh; `sizes="(max-width: 480px) 50vw, (max-width: 1024px) 33vw, 290px"`, `src` `/_next/image?url=%2Fmedia%2F{sha256}.jpg&w=3840&q=75`) → `span.uppercase` brand (chữ lưu nguyên: "JINK", "Aqara", "Sunricher"; viết hoa bằng CSS) → `h3 > a[href=/products/{slug}]` (tên đầy đủ) → khối giá: `span.text-[15px]` giá, `span.line-through` giá cũ, `span` badge "−10%" (ký tự U+2212) → `span` trạng thái ("Sắp hết hàng" có kèm `svg`; "Đặt trước" không) → `button[type=button][aria-label="Thêm {tên} vào giỏ báo giá"]` (component client `MatterQuoteButton`; hành vi chưa đọc bundle). Không có badge giao thức (Thread/Zigbee), không có mô tả ngắn, không có SKU trong card; `shortDescription` (HTML có "Tương thích … Giao thức …") và `sku` nằm trong object `product` của nút giỏ trong payload (18 trường: `id, slug, name, sku, shortDescription, brand, primaryImage, isFeatured, b2c, b2b, stockStatus, price, oldPrice, priceRange, clearance, priceHidden, clearanceFrom, clearanceCount`), không hiện trong DOM.

Phân trang: không có. 12 sản phẩm đều có trong HTML; không có `?page=` trong link, không có `rel=next`; tổng số "12 sản phẩm" chỉ xuất hiện trong toolbar (`S:0`), `ItemList.numberOfItems:12` trong JSON-LD và badge `solutions[].count:12` trong payload. Kích thước trang (page size) khi danh mục có nhiều sản phẩm hơn: Chưa xác định (trang B2B mẫu `/dimmer-doc-lap-triac` có 24 `article` trong HTML và 4 link `?page=`, nhưng đó là skin B2B).

`?view=list`: trả đủ 12 card như bản mặc định, không có `table`; khác biệt chỉ ở class lưới (`grid-cols-1 items-stretch gap-3.5` thay cho `grid-cols-2 … xl:grid-cols-4`) và `aria-current` của 2 nút chuyển dạng xem; `canonical` vẫn là URL gốc.

Danh mục con / liên quan: không có chip danh mục con, không có khối "danh mục liên quan" trong trang.

## 5. Chi tiết khối đặc biệt

Placeholder Suspense và toolbar bị stream (HTML thô):

```html
<div class="max-w-[1280px] … pt-10"><!--$?--><template id="B:0"></template><!--/$--><div class="mt-6"><div class="grid grid-cols-2 items-stretch gap-5 lg:grid-cols-3 lg:gap-6 xl:grid-cols-4"><article class="group/card …">…
…(cuối body, sau script payload)…
<div hidden id="S:0"><div class="bg-white"><div class="mx-auto w-full …"><div class="flex flex-wrap items-center justify-between gap-3 border-t …"><div class="flex flex-wrap items-center gap-4"><button type="button" aria-expanded="false" aria-controls="filter-panel-drawer" …>
…$RC("B:0","S:0")</script>
```

Vị trí (tính theo ký tự trong file HTML): `template#B:0` ở 39,393; `div#S:0` ở 116,954; script `$RC("B:0","S:0")` ở 125,412.

Card sản phẩm (rút gọn):

```html
<article class="group/card relative min-w-0 rounded-lg bg-white shadow-[…]"><a tabindex="-1" aria-hidden="true" class="relative flex aspect-[240/258] …" href="/products/cong-tac-thong-minh-4-phim-thread-matter-skin-glass"><img alt="" decoding="async" data-nimg="fill" sizes="(max-width: 480px) 50vw, (max-width: 1024px) 33vw, 290px" srcSet="/_next/image?url=%2Fmedia%2F120ba5fe…" src="/_next/image?url=%2Fmedia%2F120ba5fe….jpg&amp;w=3840&amp;q=75"/></a><div class="min-w-0 px-2.5 …"><span class="mb-1.5 block text-[12px] uppercase …">JINK</span><h3 class="mb-3.5 line-clamp-2 …"><a class="hover:underline" href="/products/cong-tac-thong-minh-4-phim-thread-matter-skin-glass">Công tắc thông minh 4 phím Thread Matter Skin Glass JINK - S1-04 SKIN</a></h3>…<span class="… text-[#1868c2] …">2.850.000đ</span>…<span class="mt-1 inline-flex … text-[#8a6d1f]"><svg …/>Sắp hết hàng</span>…<button type="button" aria-label="Thêm … vào giỏ báo giá" …>
```

FAQ (HTML thô, đáp án nằm trong `details`):

```html
<li class="border-b border-zinc-200"><details class="group/faq"><summary class="flex cursor-pointer list-none items-center justify-between gap-4 py-3 …">Công tắc Matter có cần hub riêng không?<span aria-hidden="true" class="… group-open/faq:rotate-45">+</span></summary><div class="user-html pb-4 text-body text-zinc-700">Tùy giao thức kết nối của công tắc. …</div></details></li>
```

JSON-LD: bắt đầu `<script type="application/ld+json">{"@context":"https://schema.org","@graph":[{"@type":["Organization","LocalBusiness"],"@id":"https://knxstore.vn/#organization",…` (chi tiết mục 8).

## 6. Nội dung chỉ có ở client / không thấy trong HTML ban đầu

| Nội dung | Bằng chứng | Nằm ở đâu |
|---|---|---|
| Toolbar (bộ lọc, "12 sản phẩm", chuyển dạng xem, sắp xếp) | có trong HTML nhưng trong `div#S:0[hidden]` ở cuối `<body>`, ngoài `div.font-sans`; vị trí logic (`template#B:0`) rỗng cho tới khi `$RC` chạy | HTML (ẩn) + RSC row `6`; sau `$RC` = client |
| Trạng thái "đang lọc/đang sắp xếp" (`?sort=`, `?brand=`… ) | DOM mặc định: badge "0", `select` ở "Mặc định"; hành vi lọc/sắp xếp thực tế: Chưa xác định (chưa đọc bundle `Toolbar`, chưa thử query) | client |
| Mở drawer bộ lọc, mở menu/tìm kiếm/giỏ | `aria-hidden=true`/`inert` trong HTML | client |
| Số trên badge giỏ | HTML: `aria-label="Giỏ báo giá, 0 sản phẩm"`, không có badge | client |
| `sku`, `shortDescription` (tương thích nền tảng, giao thức) của 12 sản phẩm | chỉ có trong object `product` truyền cho `MatterQuoteButton` (payload) | RSC; DOM = không; JSON-LD = không |
| Ảnh đại diện danh mục `abda0616…347c98.jpg` | chỉ có trong `og:image`/`twitter:image` và JSON-LD `primaryImageOfPage`; không có `img` nào trong body | head + JSON-LD |
| Object danh mục (tên, mô tả, `seoTitle`, parent…) | không được serialize thành object trong payload; chỉ thấy phần đã render (H1, breadcrumb, `T` row HTML SEO, FAQ) | – |
| Rác trong payload: shell B2B | row `11` (`notFound` của layout `(site)`, tham chiếu bởi `notFound:["$L11",[]]` trong row `4`) chứa menu B2B + footer B2B + 404 UI; đóng bao tham chiếu `$L…` gồm 20 row JSON ≈ 55,046 B (≈35% payload 156,415 B) | RSC |
| Gợi ý tìm kiếm, tương tác `MatterQuoteButton` | bundle client (chưa đọc `MatterQuoteButton`) | client |

Dữ liệu có trong cả DOM và JSON-LD: FAQ (10/10 Q&A khớp nguyên văn), thứ tự 12 sản phẩm (JSON-LD `ItemList` khớp thứ tự DOM). Văn bản SEO có trong DOM (và RSC row `4c`) nhưng không có trong JSON-LD.

## 7. Liên kết

Tổng 84 thẻ `a`, đều có `href`:

| Vùng | Số link | Mẫu URL |
|---|---|---|
| Header | 17 | như trang sản phẩm: 12 `/{slug}` danh mục, `/blogs`, `/contact`, `/matter`, `/cart`, `/` (Trang B2B) |
| Menu mobile | 15 | tương đương header |
| Breadcrumb | 2 | `/matter`, `/san-pham-matter` |
| Lưới sản phẩm | 24 | `/products/{slug}` (mỗi card 2 link tới cùng slug: ảnh + tên) = 12 sản phẩm |
| Văn bản SEO + FAQ | 0 | không có link trong `div.user-html` hay FAQ |
| Drawer báo giá | 2 | `/products`, `/cart` |
| Footer | 21 | như trang sản phẩm |
| Nút Zalo nổi | 1 | `https://zalo.me/0378628837` |
| Toolbar (`S:0`) | 2 | `/cong-tac-matter` (dạng lưới), `/cong-tac-matter?view=list` (dạng bảng) |

Vùng nội dung chính (breadcrumb + lưới + toolbar): 2 + 24 + 2 = 28 link. Link ngoài: `zalo.me`, `facebook.com`, `online.gov.vn`, `dmca.com`. Không có link tới trang danh mục khác trong vùng nội dung (kể cả danh mục cha ngoài breadcrumb), không có link brand (`/brands/…`), không có link bài viết trong vùng nội dung, không có link tải file, không có `rel=alternate`/markdown. Link tới trang B2B: `/` (Trang B2B, footer "Thiết bị tự động hoá toà nhà"), `/products` (drawer).

## 8. JSON-LD

1 `<script type="application/ld+json">`, 11,796 B (nội dung 11,752 B), vị trí: body, con của `div.font-sans` ngay sau menu mobile, trước thanh tiêu đề; lặp trong RSC (row `56`, kiểu `T`). Không có JSON-LD trong `<head>`. `@graph` gồm 5 node:

| Node | Trường chính | Ghi chú |
|---|---|---|
| `Organization`+`LocalBusiness` | giống node tổ chức của trang sản phẩm | – |
| `CollectionPage` (`#webpage`) | url, name "Công tắc Matter", description (= meta description, kết thúc bằng `…`), inLanguage, datePublished = dateModified = `2026-08-24T14:20:16.977Z`, isPartOf, publisher, mainEntity → `#products`, about → `#about`, primaryImageOfPage (`api.knxstore.vn/media/abda0616…jpg`, caption), breadcrumb (BreadcrumbList 3 mục: `/matter`, `/san-pham-matter`, `/cong-tac-matter`) | khớp breadcrumb DOM |
| `Service` (`#about`) | name, description (cùng chuỗi bị cắt), url, category "Điều khiển Matter", provider, areaServed VN | – |
| `ItemList` (`#products`) | name "Sản phẩm Công tắc Matter tại KNXStore", `numberOfItems: 12`, `itemListElement` 12 `ListItem` chỉ có `position` và `url` | không có `name`, `image`, `offers` trong item; thứ tự khớp DOM |
| `FAQPage` (`#faq`) | `mainEntity` 10 `Question` + `acceptedAnswer.text` | khớp 10/10 FAQ trong DOM |

Không có node `Product`, `Offer`, `aggregateRating`. Đối chiếu DOM: số sản phẩm 12 khớp; văn bản SEO trong DOM nói "KNXStore hiện có 14 model công tắc Matter" (lệch với 12 sản phẩm trong lưới/`ItemList`), "giá từ 955.000đ" khớp giá thấp nhất trong lưới (955.000đ).

## 9. Số đo

| Chỉ số | Giá trị |
|---|---|
| HTML thô (identity) | 303,087 B (≈75,771 token theo bytes/4) |
| gzip / br | 51,259 B / 39,651 B |
| `<head>` | 11,852 B |
| Tổng `<script>` | 20 thẻ, 186,596 B (61.6%) |
| — `self.__next_f` | 2 thẻ: init 60 B + 1 push 172,565 B = 172,625 B |
| — `ld+json` | 1 thẻ, 11,796 B |
| — script ngoài `src` | 14 thẻ (tag 1,122 B) |
| — script inline khác | 3 (cookie 117 B; `$RT`; `$RC`/`$RB`/`$RV` ≈ 853 B; tổng 1,053 B) |
| Payload RSC đã giải mã | 156,415 B, 96 row (65 JSON, 22 import, 7 hint, 2 text `T`: JSON-LD 11,752 B và SEO HTML 9,022 B) |
| Text trong DOM (gồm khối ẩn) | 12,954 ký tự (16,339 B ≈ 4,084 token) |
| Text vùng nội dung (bỏ header, menu mobile, drawer, footer, Zalo, `S:0`) | 11,479 ký tự |
| `a` / `img` / `table` / `button` | 84 / 17 / 0 / 22 |
| `h1` / `h2` / `h3` / `h4–h6` | 1 / 12 / 13 / 0 |
| `form` / `input` / `nav` / `aside` / `section` / `article` | 1 / 5 / 3 / 2 / 3 / 12 |
| `ul` / `li` / `details` / `summary` / `svg` / `main` | 10 / 44 / 14 (3 menu mobile + 10 FAQ + 1 bộ lọc) / 14 / 45 / 0 |
| `select` / `option` | 1 / 6 (trong `S:0`) |
| `link` / `meta` | 16 / 16 |
| Header / menu mobile / footer | 9,358 B / 6,571 B / 7,929 B |
| Drawer báo giá / Zalo | 2,585 B / 649 B |
| Khung (header + menu mobile + footer + drawer + Zalo) | 27,092 B (8.9% HTML thô) |
| Thanh tiêu đề (H1 + breadcrumb) | 965 B, 58 ký tự |
| Lưới sản phẩm (`section`) | 51,308 B (≈4.2 KB/card), 1,146 ký tự |
| Văn bản SEO + FAQ (`section`) | 18,335 B, 10,275 ký tự (SEO 6,280 + FAQ 3,977) |
| Toolbar `div#S:0` | 7,751 B, 258 ký tự |

## 10. Khác biệt so với skin B2B / Chưa xác định

So sánh có kiểm chứng với trang danh mục B2B `https://knxstore.vn/dimmer-doc-lap-triac` (tải hôm nay, 535,241 B) và với trang sản phẩm B2B đã tải cho file 13:

| Hạng mục | Matter (`/cong-tac-matter`) | B2B (`/dimmer-doc-lap-triac`) |
|---|---|---|
| Header HTTP / render | dynamic, `no-store`, `link` preload font, `"S":false` | cùng kiểu |
| Wrapper và skin | `div.font-sans[style="--font-body:var(--font-matter)"]`, script `knx-area=matter` trong wrapper, không `#main-content`, không skip link | script `knx-area=b2b` ở đầu `<body>`, skip link `a[href=#main-content]`, `div#main-content` (121,102 B) |
| Header | 9,358 B, 17 link | 128,755 B, 165 link, 67 `img` |
| Footer | 7,929 B, 21 link, không `nav` | 17,608 B, 50 link, 3 `nav` |
| Breadcrumb / H1 | H1 trong thanh tiêu đề riêng cùng khối với breadcrumb; `nav[aria-label="Đường dẫn"] > ol > li`, gốc `/matter` | `nav[aria-label=Breadcrumb]` (span) rồi H1 + "29 sản phẩm" trong `div.py-4`; gốc `/` |
| Chip danh mục con | không có | có (4 chip: Dimmer DALI, Dimmer DMX, Điều khiển 0-10V, Điều khiển ON/OFF) |
| Card | `article.group/card` (brand viết hoa, tên `h3`, giá + giá gạch + badge, nhãn tồn kho, nút giỏ) | `article.group.relative.rounded-card.border` |
| Số card trong HTML / phân trang | 12 (toàn bộ), không có `?page=` | 24 `article`, có 4 link `?page=` (tổng 29 sản phẩm) |
| Toolbar | `div#S:0[hidden]` cuối body, bộ lọc brand 3 mục, sắp xếp 6 option, 2 dạng xem | cũng `div#S:0[hidden]` (15,077 B) + `$RC` |
| Văn bản SEO + FAQ | có (6,280 + 3,977 ký tự, 10 FAQ) | `details` ×10 và 1 `table` trong trang mẫu (nội dung không so sánh) |
| JSON-LD | 5 node: Organization, CollectionPage, Service, ItemList (12 url), FAQPage (10) | có `CollectionPage` + `BreadcrumbList` + …; nội dung node chưa so sánh chi tiết |
| `<head>` | 6 preload ảnh `imagesrcset` (card đầu) | chưa so sánh chi tiết |

Chưa xác định:

| Điểm | Lý do |
|---|---|
| Hành vi sau `$RC` của toolbar (lọc theo brand, sắp xếp, query `?sort=`, `?brand=`) | chưa chạy trình duyệt, chưa đọc bundle `Toolbar`, chưa thử các query |
| Page size và kiểu phân trang khi danh mục Matter có nhiều hơn 12 sản phẩm | `/cong-tac-matter` chỉ có 12; chưa khảo sát `/san-pham-matter`, `/cam-bien-matter`… (chỉ B2B mẫu thấy 24/trang) |
| Điều kiện chọn skin Matter hay B2B cho `/{slug}` | wrapper ở layout segment `[slug]` (suy ra từ cây route); quy tắc chọn không nhìn thấy; chưa gửi request có cookie `knx-area` |
| Cấu trúc danh mục cha (ví dụ `/san-pham-matter`) và danh mục con | chưa fetch |
| Hành vi `MatterQuoteButton` | chưa đọc bundle |
| Object danh mục không được serialize | chỉ thấy output đã render; các trường `seoTitle`, mô tả, parent không xuất hiện thành key trong payload |
| Nguồn của "14 model" trong văn bản SEO | văn bản CMS nhập tay lệch với 12 sản phẩm hiện có; chưa xác định có phải sản phẩm ẩn/chưa publish |
| Mẫu 1 URL có thể không đại diện | 91 URL danh mục gồm cả B2B và Matter; template danh mục có sản phẩm đã hết hàng, `priceHidden`, `clearance`, nhiều facet (`facets`/`optionFilters` khác rỗng) chưa quan sát |
| Phiên bản Next.js chính xác | không có header/meta nêu version |
