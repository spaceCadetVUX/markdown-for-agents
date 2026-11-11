# Trang sản phẩm — cấu trúc hiện tại

> https://knxstore.vn/products/sunricher-phim-bam-gan-tuong-dali-2-8-phim-sr-2422nk8-da2 · products.xml · 752 URL cùng loại (`/products/{slug}`) · khảo sát 2026-11-11

Mẫu đối chiếu: `https://knxstore.vn/products/den-led-bulb-t2-e27-anh-sang-trang-matter-aqara-lb-l02e` (chỉ so template ở mục 10).

Phương pháp: `curl` HTML thô (UA `Mozilla/5.0`, không cookie), phân tích bằng `html.parser` trên file đã lưu; giải mã payload `self.__next_f` để đối chiếu với DOM; đọc thêm 7 file JS tĩnh `/_next/static/chunks/*.js` của chính site để xác định hành vi client của tab/giá/SKU (ghi rõ "suy ra từ bundle" ở chỗ dùng). Chưa chạy trình duyệt headless nên mọi mô tả "sau hydrate" là suy ra từ source, không phải quan sát DOM sau render.

## 1. Response & rendering

Request chính: `GET` không `Accept-Encoding` (curl mặc định).

| Header | Mẫu chính (SR-2422NK8-DA2) | Mẫu đối chiếu (LB-L02E) |
|---|---|---|
| Status | 200 (HTTP/2) | 200 |
| content-type | `text/html; charset=utf-8` | `text/html; charset=utf-8` |
| cache-control | `s-maxage=3600, stale-while-revalidate=31532400` | giống |
| x-nextjs-cache | lần 1 `STALE`, lần 2 và 3 `HIT` | lần 1 `MISS` |
| x-nextjs-prerender | `1` | `1` |
| x-nextjs-stale-time | `300` | `300` |
| vary | `rsc, next-router-state-tree, next-router-prefetch, next-router-segment-prefetch, Accept-Encoding` | giống |
| cf-cache-status | `DYNAMIC` (Cloudflare không cache HTML) | `DYNAMIC` |
| link | không có | không có |
| x-robots-tag | không có | không có |
| content-encoding | không có khi không gửi Accept-Encoding; `br` khi gửi `gzip, br`; `gzip` khi gửi `gzip` | không đo |
| Header khác | `server: cloudflare`, `speculation-rules: "/cdn-cgi/speculation"`, `strict-transport-security`, `x-frame-options: SAMEORIGIN`, `x-content-type-options: nosniff`, `referrer-policy: same-origin`, `alt-svc: h3`, `nel/report-to` (Cloudflare NEL) | giống |

Kích thước trên đường truyền (mẫu chính): identity 447,188 B (lần đo khác 447,102 B), gzip 66,947 B, br 41,675 B.

Chế độ render: ISR (static prerender + revalidate theo thời gian). Bằng chứng: `x-nextjs-prerender: 1`; chuỗi `x-nextjs-cache` đi `STALE` → `HIT` (có cache entry, có stale-while-revalidate); `s-maxage=3600` cho thấy revalidate 3600 s; `stale-while-revalidate=31532400` = 31,536,000 − 3,600 (expire 1 năm); không có header `link` preload (khác trang danh mục là dynamic). HTML giữa các lần fetch chỉ khác ở cách chia/đánh số chunk RSC (số byte lệch 86 B), DOM giống nhau.

Route (đọc từ record `0:` của payload): `(site)/products/[slug]/page`, `slug` là dynamic segment, không có sibling static. Framework: Next.js App Router, bundler Turbopack (file `turbopack-0j8dnv21rrwyx.js`), Vary có `next-router-segment-prefetch`. Phiên bản chính xác: Chưa xác định (không có header nêu version).

Hành vi hiện tại với yêu cầu markdown (chỉ ghi kết quả):

| Request | Status | content-type | Ghi chú |
|---|---|---|---|
| `Accept: text/markdown` trên URL sản phẩm | 200 | `text/html; charset=utf-8` | body 447,102 B, vẫn là HTML; `vary` không chứa `Accept` |
| URL + `.md` | 404 | `text/html; charset=utf-8` | body 131,581 B (trang lỗi Next, `<html id="__next_error__">`), `x-nextjs-cache: MISS`, `cache-control: s-maxage=60, stale-while-revalidate=31535940` |

## 2. `<head>`

Mẫu chính. Head dài 5,206 ký tự; `<html lang="vi">`.

| Phần tử | Giá trị |
|---|---|
| `<title>` | `Phím bấm gắn tường DALI-2 8 phím Sunricher - SR-2422NK8-DA2` (giống H1, og:title, twitter:title, `ItemPage.name`) |
| meta description | `Phím bấm SR-2422NK8-DA2 là thiết bị điều khiển DALI-2 với 8 phím, chứng nhận DALI-2, thiết kế mỏng, hỗ trợ chức năng phản hồi và 16 đèn LED.` (= `seoDescription` trong payload) |
| canonical | `https://knxstore.vn/products/sunricher-phim-bam-gan-tuong-dali-2-8-phim-sr-2422nk8-da2` (self) |
| meta robots | không có |
| keywords | `Sunricher,Công tắc thông minh,SR-2422NK8-DA2` |
| author / theme-color | `KNX Store` / `#14467C` |
| OG | `og:title`, `og:description`, `og:url`, `og:image` = `https://api.knxstore.vn/media/b63b9aea…c373bb.png`, `og:type=website` (không phải `product`); không có `og:site_name`, `og:locale`, `product:price:*` |
| Twitter | `twitter:card=summary_large_image`, `title`, `description`, `image` |
| hreflang / `rel=alternate` | không có |
| `rel=manifest` | `/manifest.webmanifest` |
| icons | `/favicon.ico?favicon.16_cwkv0v-dvu.ico`, `/icon.svg?icon.39v289yelkh8f.svg`, `/apple-icon.png?apple-icon.264ni-uec6-1q.png` |
| preload | 4 font woff2 (`/_next/static/media/*.woff2`), logo `/images/logo/knxstore-logo-tight.svg`, ảnh chính (`<link rel=preload as=image imageSrcSet=/_next/image?url=%2Fmedia%2Fb63b9aea….png&w=640..>`), 1 JS `/_next/static/chunks/3ivcc6t1s04dh.js` (fetchPriority=low), `https://www.googletagmanager.com/gtag/js?id=G-4D9XLJDVGG` |
| stylesheet | `/_next/static/chunks/1mmmxvxvvtkv8.css`, `/_next/static/chunks/1vbr8oh4939ca.css` (`data-precedence=next`) |
| script trong head | 13 thẻ `<script src async>` (12 chunk + 1 `noModule`) |
| JSON-LD | không nằm trong `<head>`; nằm trong body (xem mục 8) |

Mẫu đối chiếu: cùng bộ thẻ, khác: title `Đèn Thông Minh Aqara Bulb T2 E27 Trắng: 2700–6500K, Matter Thread` (khác H1), description `Aqara LB-L02E E27 Tunable White: 1100lm, 2700–6500K, CRI>90, …`, preload thêm 4 icon `/images/matter-icons/{apple-home,google-home,smartthings,alexa}.png`, `og:image` `…/bca02a21…jpg`.

## 3. Khung trang

Landmark (mẫu chính): `header` (không role/aria-label), `footer`, `form[role=search]`, 10 `nav` (6 trong header, Breadcrumb, 3 trong footer), `div[role=tablist]`, `div[role=group][aria-label="Sản phẩm đề xuất"]`. Không có thẻ `<main>`, không có `role=main`, không có `<aside>`. Vùng nội dung chính là `div#main-content[tabindex=-1]` (đích của skip link `a[href="#main-content"]` "Bỏ qua, tới nội dung chính").

Thứ tự con trực tiếp của `<body>`: `div[hidden]` (rỗng, `<!--$--><!--/$-->`) → `script` (đặt cookie `knx-area=b2b`) → skip link → `header.sticky` → `div#main-content` → `div[inert].fixed` (drawer "Yêu cầu báo giá") → `footer` → `a[data-floating-zalo]` → các `<script>`.

Header (3,977 ký tự text, 165 `<a>`, toàn bộ SSR):

| Thành phần | Chi tiết |
|---|---|
| Thanh trên | `button[aria-label="Mở menu"]`; logo `a[href=/]` (img `knxstore-logo-tight.svg`); `form[role=search]` (`hidden lg:block`) với `button#b2b-header-scope` + panel `#b2b-header-scope-panel` (7 phạm vi: Tất cả, Sản phẩm, Bài viết, Thương hiệu, Giải pháp, Danh mục, Visual) và `input#b2b-header-search[type=search][name=q][role=combobox]`; nút `aria-label="Tìm kiếm"` (mobile); `a[href=/cart][aria-label="Giỏ báo giá, 0 sản phẩm"]`; `a[href=/contact]` "Tư vấn dự án" |
| Thanh menu cấp 1 | `nav[aria-label="Điều hướng chính"]`: nút Trending, Danh mục sản phẩm, Giải pháp, Thương hiệu, Công cụ, Blog kỹ thuật; link `/visual`, link `/matter` "Matter Smarthome" |
| Panel menu cấp 2 (đều `class=hidden`, có trong DOM) | `#b2b-trending-panel` (role=region, 29 link: 8 `/search?q=…`, 16 `/products/{slug}`, 5 bài `*.html`; kèm 67 `<img>` thumbnail trong toàn header), `#b2b-menu-categories` (nav "Toàn bộ danh mục sản phẩm", 59 link, ví dụ `/bo-dieu-khien-trung-tam`, `/dali-gateway`), `#b2b-menu-solutions` (25 link `/solution/{slug}`), `#b2b-menu-brands` (25 link `/brands/{slug}`, text rỗng vì là logo), `#b2b-menu-tools` (3 link: `/tool/knx-ga-planner/index.html`, `/tool/engraving/index.html`, `/tools/hvac-compatibility`), `#b2b-menu-blogs` (19 link: 10 `/blogs…` chuyên mục và 9 bài `*.html`) |
| Hotline | không có số trong header; số `0918.918.755` xuất hiện ở buy box (`tel:0918918755`), footer, JSON-LD |

Footer (1,110 ký tự, 50 `<a>`): cột trái gồm tên công ty "CÔNG TY CP TÍCH HỢP HỆ THỐNG LIÊN MINH", MST 0311846236, địa chỉ (link Google Maps), `tel:0918918755`, `mailto:support@knxstore.vn`, link Facebook, link Zalo `https://zalo.me/0918918755`, 2 badge (`online.gov.vn` Bộ Công Thương, `dmca.com`). Ba cột: "Danh Mục Sản Phẩm" (7 link), "Giải Pháp" (7 link `/solution/{slug}`), "Về chúng tôi" (5 link). Dòng cuối: "© 2026 KNXStore …" + 4 link chính sách (`/warranty-lookup`, `/chinh-sach-bao-mat.html`, `/chinh-sach-mua-hang-tai-knxstore-vn.html`, `/dieu-khoan-su-dung-website-knxstore-vn.html`). Ba cột được render hai lần: bản mobile `details` (`sm:hidden`) và bản desktop `nav` (`hidden sm:grid`), nên 19 link cột bị lặp trong DOM (38 trong tổng 50).

Thành phần nổi / ẩn:

| Thành phần | Selector | Trạng thái mặc định |
|---|---|---|
| Nút Zalo nổi | `a[data-floating-zalo][aria-label="Chat Zalo với KNXStore"]` → `https://zalo.me/0918918755`, `target=_blank`, text "Zalo" | hiển thị (fixed, `right-[26px]`) |
| Drawer yêu cầu báo giá | `div[inert].fixed.top-0.left-0.z-99999.translate-x-full`: h2 "Yêu cầu báo giá", "Chưa có thiết bị nào trong yêu cầu.", "Xem sản phẩm", "Tổng số lượng: 0", "Xem và gửi yêu cầu" (95 ký tự) | ẩn (inert, ngoài màn hình), SSR |
| Thanh mua cố định mobile | `div[inert].fixed.inset-x-0.bottom-0.lg:hidden.translate-y-full` (tên SP, "Liên hệ để nhận báo giá", nút "Thêm báo giá") | ẩn mặc định, nằm trong buy box |
| Chat widget bên thứ ba, popup, cookie banner, back-to-top | không tìm thấy trong HTML | không có |
| Google Analytics | preload `gtag/js?id=G-4D9XLJDVGG` + component `GoogleAnalytics` trong payload | script |

## 4. Cây nội dung (mẫu chính, theo thứ tự DOM)

Cột "Trong main": có nằm trong `div#main-content` hay không. Mọi khối dưới đây đều có text trong HTML ban đầu (SSR); khối nào chỉ có ở payload/JSON-LD ghi ở mục 6.

| # | Thẻ + selector rút gọn | Heading | Nội dung tóm tắt | Trong main | Hiển thị mặc định | SSR / client |
|---|---|---|---|---|---|---|
| 1 | `header.sticky.top-0.z-9999` | h3 trong panel ẩn (18 h3, 2 h3 rỗng) | menu, search, giỏ, CTA (xem mục 3) | không | thanh trên + thanh menu hiện; 6 panel `hidden` | SSR |
| 2 | `div#main-content > script[type=application/ld+json]` | – | JSON-LD 3,916 B | có | không hiển thị | SSR (và lặp trong RSC `$62`) |
| 3 | `nav[aria-label=Breadcrumb]` (span, không `ol/li`) | – | `Trang chủ` (`/`) / `Công tắc thông minh` (`/cong-tac-thong-minh`) / tên SP (`span[aria-current=page]`); dấu `/` là `span[aria-hidden]` nên textContent nối liền `Trang chủ/Công tắc thông minh/…` | có | hiện | SSR |
| 4 | `div.grid.lg:grid-cols-[62%_minmax(0,1fr)]` | – | khối sản phẩm 1,911 ký tự, chia 3 phần dưới | có | hiện | SSR |
| 4a | cột trái hàng 1: `button[aria-label^="Phóng to ảnh"] > img[fetchpriority=high][width=720][height=720]` + 2 nút `Ảnh trước`/`Ảnh sau` ("‹", "›") + 2 thumbnail `button[aria-label="Xem ảnh n trên 2"] > img[loading=lazy][alt=""]` 72x72 | – | gallery 2 ảnh PNG; ảnh chính `alt` = tên SP; `src` dạng `/_next/image?url=%2Fmedia%2F{sha256}.png&w=3840&q=75` kèm `srcset` w=640…3840 | có | ảnh 1 hiện; ảnh 2 chỉ là thumbnail; lightbox đóng | SSR (img có src/srcset); lightbox/đổi ảnh = client |
| 4b | cột phải: `a[href=/brands/sunricher]` | – | thương hiệu "Sunricher" | có | hiện | SSR |
| 4c | `p > span.model-code` (component `LiveSku`, `data-live-sku`) | – | `SKU: SR-2422NK8-DA2` | có | hiện | SSR; đổi theo variant ở client (suy ra từ bundle) |
| 4d | `h1.text-[26px]` gồm 4 `<span>` (`Phím bấm gắn tường `, `DALI-2`, ` 8 phím Sunricher - `, `SR-2422NK8-DA2`) | h1: "Phím bấm gắn tường DALI-2 8 phím Sunricher - SR-2422NK8-DA2" | tên SP; span `whitespace-nowrap` bọc token model | có | hiện | SSR |
| 4e | `div.flex.flex-wrap.gap-2 > a[href=/solution/dali]` | – | chip giải pháp "DALI" | có | hiện | SSR |
| 4f | `div#buy-new > p.text-h3` | – | "Liên hệ để nhận báo giá" (không có giá số: `price:null`) | có | hiện | SSR |
| 4g | `p` "Color" + `ul > li > button[aria-pressed=false]` | – | 2 variant: "Xám", "Trắng" (cả hai đều `aria-pressed=false` trong HTML; payload ghi "Xám" `isDefault:true`) | có | hiện | SSR; chọn = client |
| 4h | số lượng: `button[aria-label="Giảm số lượng"]`, `input[type=number][aria-label="Số lượng"]`, `button[aria-label="Tăng số lượng"]`; `button[aria-label="Thêm vào so sánh"][aria-haspopup=dialog]` | – | điều khiển số lượng, so sánh | có | hiện | SSR; hành vi client |
| 4i | `button` "Thêm vào yêu cầu báo giá"; `a[href=tel:0918918755]` "Liên hệ 0918.918.755" | – | CTA báo giá (không có nút Mua ngay, vì `quotable:true`) | có | hiện | SSR |
| 4j | `div[inert].fixed.bottom-0.lg:hidden` | – | thanh mua mobile (tên + "Liên hệ để nhận báo giá" + "Thêm báo giá") | có | ẩn | SSR |
| 4k | `ul` | – | "Xuất xứ: CN", "Giao hàng toàn quốc" | có | hiện | SSR |
| 4l | `p` "Danh mục: " + 2 `a` | – | `/cong-tac-thong-minh`, `/cong-tac-dali` | có | hiện | SSR |
| 4m | cột trái hàng 2: `section.scroll-mt-[64px] > div[role=tablist][aria-label="Nội dung sản phẩm"] > button[role=tab][aria-selected=true]` | – | chỉ 1 tab: "Mô tả chi tiết"; không có `role=tabpanel`, không có `aria-controls` | có | hiện | SSR; tab là client state |
| 4n | `div.user-html.text-body` (panel mô tả, 1,546 ký tự) | h2 "Tổng quan về phím bấm gắn tường DALI-2 8 phím Sunricher - SR-2422NK8-DA2"; h3 "Các tính năng chính của SR-2422NK8-DA2"; h3 "Sơ đồ kết nối của SR-2422NK8-DA2" | 2 `p` (650 + 114 ký tự), `ul` 10 `li` (chuẩn IEC62386-101/-103/-301/-332, 16 LED phản hồi, multimaster, cấp nguồn từ bus DALI…), `p > img[alt="Sơ đồ kết nối"]` (`/media/70f8af78….jpg`, không width/height) | có | hiện | SSR (HTML do CMS nhập, in qua `dangerouslySetInnerHTML`) |
| 5 | `section.border-t > div[role=group][aria-label="Sản phẩm đề xuất"]` | h2 "Sản phẩm đề xuất" + p "Thiết bị tương tự và cùng danh mục — cuộn ngang để xem thêm" | 16 `article` card (1,482 ký tự); 8 card "Liên hệ", 1 card "Giá liên hệ", 7 card có giá (`2.900.000 đ`…`6.460.000 đ`) | có | hiện (cuộn ngang) | SSR |
| 6 | `div[inert].fixed.translate-x-full` | h2 "Yêu cầu báo giá" | drawer báo giá rỗng | không | ẩn | SSR |
| 7 | `footer.mt-auto` | h2 ×3 | xem mục 3 | không | hiện | SSR |
| 8 | `a[data-floating-zalo]` | – | "Zalo" | không | hiện | SSR |

Không có trong trang này (mẫu chính): bảng (`<table>` = 0), thông số kỹ thuật, tài liệu tải về, video/iframe, FAQ, bài viết liên quan, form liên hệ/báo giá nhúng, mô tả ngắn (`shortDescription:null`), mã MPN (`mpn:null`).

Cấu trúc card (`article.group.relative.rounded-card`): `div.aspect-square > img[loading=lazy]` (`/_next/image?url=%2Fmedia%2F{hash}.avif|jpg&w=…`) → `div > span` thương hiệu + `span.model-code` mã model → `a[href=/products/{slug}]` (tên, bọc token model trong `span.whitespace-nowrap`; `after:absolute after:inset-0` biến cả card thành vùng bấm) → `div.mt-auto` giá (`3.190.000 đ` hoặc "Liên hệ"/"Giá liên hệ") + `button[aria-label="Thêm {tên} vào yêu cầu báo giá"]`. Card thứ 10 hiển thị mã `SR-2422NK8-DA2` nhưng trỏ tới slug `…sr-2422nkx-da2` (tên "SR-2422NKX-DA2").

## 5. Chi tiết khối đặc biệt

Tab (mẫu chính, 1 tab):

```html
<div role="tablist" aria-label="Nội dung sản phẩm" class=…><button type="button" role="tab" aria-selected="true" class=…>Mô tả chi tiết</button></div>
<div class="user-html text-body text-zinc-700"><h2>Tổng quan về phím bấm gắn tường DALI-2 …</h2> … <h3>Sơ đồ kết nối của SR-2422NK8-DA2</h3><p><img alt="Sơ đồ kết nối" src="/media/70f8af78….jpg" /></p></div>
```

Logic tab (suy ra từ bundle `/_next/static/chunks/16nkacgg7g1nb.js`, component props `{product, description, guide, variant}`): danh sách tab được dựng theo dữ liệu, `useState(tab đầu tiên)`, chỉ panel của tab đang chọn được render (`"specs"===h && …`), các panel khác không có trong DOM.

| Key | Nhãn skin b2b | Nhãn skin matter | Điều kiện hiện | DOM của panel (theo bundle) |
|---|---|---|---|---|
| description | Mô tả chi tiết | Chi tiết sản phẩm | `descriptionHtml` có | `div.user-html` + HTML thô |
| guide | không có | Hướng dẫn | `guideHtml` có (chỉ matter) | `div.user-html` |
| specs | Thông số kỹ thuật | Thông số kỹ thuật | `specs.length>0` | `dl > div.grid > dt + dd` (không phải `<table>`); mỗi dòng `name` / `valueText` (+ `unit` nếu `parseStatus==="exact"` và giá trị chưa chứa unit); nền xen kẽ trắng/zinc-50 |
| downloads | Tài liệu | Datasheet | `downloads.length>0` | `ul > li` (icon, `label ?? filename`, dung lượng B/KB/MB) + `a[target=_blank][aria-label="Tải xuống …"]`; link không an toàn → `span` "Liên kết không khả dụng" |
| faq | Câu hỏi thường gặp | FAQ | `faqs.length>0` | `div > details[open chỉ mục 0] > summary + p` |

Mẫu chính có `downloads:[]`, `specs:[]`, `faqs:[]`, `guideHtml:null` nên chỉ còn tab description; DOM thật của specs/downloads/faq ở skin b2b chưa được quan sát (mục 10).

JSON-LD (mẫu chính) nằm ở đầu `#main-content`:

```html
<div id="main-content" tabindex="-1"><script type="application/ld+json">{"@context":"https://schema.org","@graph":[{"@type":["Organization","LocalBusiness"],"@id":"https://knxstore.vn/#organization",…},{"@type":"ItemPage",…}]}</script>
```

H1 tách span:

```html
<h1 class=…><span>Phím bấm gắn tường </span><span class="whitespace-nowrap">DALI-2</span><span> 8 phím Sunricher - </span><span class="whitespace-nowrap">SR-2422NK8-DA2</span></h1>
```

Ảnh gallery (SSR):

```html
<img alt="Phím bấm gắn tường DALI-2 8 phím Sunricher - SR-2422NK8-DA2" fetchpriority="high" width="720" height="720" data-nimg="1" sizes="(max-width: 1024px) 100vw, 680px" srcSet="/_next/image?url=%2Fmedia%2Fb63b9aea….png&w=640&q=90 640w, …" src="/_next/image?url=%2Fmedia%2Fb63b9aea….png&w=3840&q=75"/>
```

Props sản phẩm trong payload RSC (không hiển thị hết ra DOM):

```json
{"id":721,"slug":"sunricher-phim-bam-gan-tuong-dali-2-8-phim-sr-2422nk8-da2","sku":"SR-2422NK8-DA2","shortDescription":null,"brand":{"slug":"sunricher","name":"Sunricher"},"b2c":false,"b2b":true,"mpn":null,"descriptionHtml":"$45","guideHtml":null,"countryOfOrigin":"CN","warrantyMonths":null,"status":"PUBLISHED","publishedAt":"2025-11-03T03:37:28.000Z","downloads":[],"specs":[],"faqs":[],"price":null,"oldPrice":null,"stockStatus":"out_of_stock","quotable":true,"variants":[{"sku":"SR-2422NK8-DA2","name":"Xám","isDefault":true,"price":null,"options":[{"name":"Color","value":"Xám"}]},{"sku":"SR-2422NK8-DA2","name":"Trắng","isDefault":false,…}],"categories":[…2],"solutions":[{"slug":"dali"}],"related":[],"bundles":[],"b2cContent":null,"skin":"b2b"}
```

## 6. Nội dung chỉ có ở client hoặc không thấy trong HTML ban đầu

Mẫu chính:

| Nội dung | Có trong HTML ban đầu? | Bằng chứng |
|---|---|---|
| Tên, SKU, thương hiệu, danh mục, giải pháp, mô tả (h2/h3/p/ul/img), breadcrumb, 16 card đề xuất, 2 variant (Xám/Trắng), ảnh gallery | Có text/thuộc tính trong DOM | đếm trong DOM sau khi bỏ script; không có `BAILOUT_TO_CLIENT_SIDE_RENDERING`, `<template data-dgst`, `animate-pulse`, `skeleton` (đều = 0) |
| Giá | Không có giá số: sản phẩm `price:null` nên DOM chỉ có "Liên hệ để nhận báo giá" | payload `"price":null,"quotable":true` |
| Trạng thái kho | Không hiển thị trong DOM | payload có `"stockStatus":"out_of_stock"`, DOM không có chữ về tồn kho |
| Chuyển variant, đổi SKU/giá/ảnh, chọn số lượng, thêm vào giỏ/so sánh, lightbox, thanh mua mobile hiện khi cuộn | Chỉ là hành vi client | `useState`/`useVariantSelection` trong bundle; thanh mua mobile `inert` + `translate-y-full` |
| Số lượng trong giỏ báo giá ("Giỏ báo giá, 0 sản phẩm", "Tổng số lượng: 0") | HTML chỉ có giá trị khởi tạo 0 | store Redux `quoteReducer` ở client (bundle) |
| Gợi ý của ô tìm kiếm header | Không | `input[role=combobox]` rỗng, không có list option trong HTML |
| Tab Thông số / Tài liệu / FAQ | Không tồn tại ở sản phẩm này (mảng rỗng) | mục 5 |
| Mega menu | Có trong DOM nhưng `hidden` (display:none) cho tới khi client bật | class `hidden` trên `#b2b-menu-*` |

Dữ liệu có trong payload `self.__next_f` nhưng không in ra DOM: `stockStatus`, `status`, `publishedAt`, cờ `b2b/b2c`, `quotable`, kích thước ảnh gốc 800x600 của 2 ảnh (`images[].width/height`), `seoTitle/seoDescription`, `countryOfOrigin` (có in "Xuất xứ: CN"), `warrantyMonths:null`.

Payload bất thường: 200,805 B trong 10 thẻ `<script>` chứa `self.__next_f` (44.9% HTML thô). Gồm khoảng: dữ liệu nav của header ~58.9 KB (record `5:` layout và record `11:` fallback not-found chứa cùng khối 28 KB, nên lặp 2 lần), card đề xuất ~40.3 KB (lặp lại nội dung đã có trong DOM), footer ~25.7 KB (footer/cột footer xuất hiện nhiều record `17`,`1f`,`26`,`2e`,`32`,`39`), mô tả HTML ~10.4 KB (lặp lại panel mô tả đã có trong DOM), route tree ~4 KB, JSON-LD ~4 KB (lặp lại `<script ld+json>`), module refs ~4.8 KB. Số liệu theo nhóm là ước tính từ phân loại record, không phải đo chính xác từng byte. Record `0:` còn chứa cả cây `notFound` (trang 404 dự phòng) trong route tree.

## 7. Liên kết

Tổng 242 `<a>` trong body: 165 header, 50 footer, 23 trong `#main-content`, 1 skip link, 1 nút Zalo nổi, 2 trong drawer báo giá (`/products` "Xem sản phẩm", `/cart` "Xem và gửi yêu cầu").

| Nhóm | Mẫu URL | Số lượng toàn trang | Trong `#main-content` |
|---|---|---|---|
| Sản phẩm | `/products/{slug}` | 32 (16 trong panel Trending của header + 16 card đề xuất) | 16 (card đề xuất) |
| Danh mục | `/{slug}` (không đuôi); nhóm này còn lẫn `/products` (drawer) và `/warranty-lookup` (footer) vì cùng dạng path | 86 (ví dụ `/dali-gateway`, `/casambi-gateway`, `/bo-dieu-khien-trung-tam`) | 3 (`/cong-tac-thong-minh` ×2, `/cong-tac-dali`) |
| Brand | `/brands/{slug}` | 25 | 1 (`/brands/sunricher`) |
| Giải pháp | `/solution/{slug}` | 39 | 1 (`/solution/dali`) |
| Bài viết / trang chính sách / công cụ dạng `.html` | `/{slug}.html` (bài blog, 3 trang chính sách ở footer), `/tool/{tên}/index.html` | 19 (5 Trending + 9 menu blog + 2 `/tool/…/index.html` + 3 footer) | 0 |
| Blog listing | `/blogs`, `/blogs/{chuyên-mục}` | 12 | 0 |
| Tìm kiếm | `/search?q=…` | 8 | 0 |
| Công cụ | `/tools/hvac-compatibility` (+ 2 link `/tool/…/index.html` đã tính ở dòng trên) | 1 | 0 |
| Khác nội bộ | `/` (3), `/cart` (2, gồm 1 trong drawer), `/contact` (3), `/visual` (1), `/matter` (1) | 10 | 1 (`/`) |
| `tel:` / `mailto:` | `tel:0918918755` (2), `mailto:support@knxstore.vn` (1) | 3 | 1 (tel) |
| Liên kết ngoài | `https://zalo.me/0918918755` (2), `https://www.facebook.com/knxstore.vn`, `https://maps.app.goo.gl/…`, `https://online.gov.vn/…`, `https://www.dmca.com/…` | 6 | 0 |

Link tới file tải (PDF/CAD/manual): 0 ở mẫu chính (không có chuỗi `.pdf`, `.dwg`, `.zip`… trong HTML). Theo bundle, link tải của sản phẩm có dữ liệu `downloads` trỏ `a[href=mediaUrl(url)][target=_blank]`; URL pattern thực tế: Chưa xác định (không có mẫu nào).

Ảnh: `/media/{sha256}.{png|jpg|avif}` (gốc, same-origin) và `/_next/image?url=%2Fmedia%2F…&w=…&q=…` (optimizer); `og:image`/JSON-LD dùng `https://api.knxstore.vn/media/{sha256}.{ext}`. Ảnh trong mô tả dùng trực tiếp `/media/…` không qua optimizer.

Về markdown/alternate: không có `<link rel=alternate>`, không có `Link:` header, không có chuỗi `text/markdown` hay `llms` trong HTML. Ngoài phạm vi 2 loại trang: `https://knxstore.vn/llms.txt` trả 200 `text/plain; charset=utf-8` 7,354 B, nhưng không được liên kết từ HTML của trang sản phẩm.

## 8. JSON-LD

Mẫu chính: 1 thẻ `<script type="application/ld+json">` (3,916 B), `@context` + `@graph` gồm 2 node. Không có thẻ thứ hai.

| Node | Trường có | Ghi chú |
|---|---|---|
| `Organization` + `LocalBusiness` (`@id` `…/#organization`) | name, legalName, url, foundingDate `2012`, taxID, logo (ImageObject 150x50), image, description, telephone `+84918918755`, email, address (PostalAddress), geo, hasMap, openingHoursSpecification (T2–T7 09:00–17:30), priceRange `50.000₫ - 200.000.000₫`, areaServed, knowsAbout (10 mục), contactPoint, sameAs (Facebook, YouTube) | giống nhau trên mọi trang đã đo |
| `ItemPage` (`@id` `…/products/{slug}#webpage`) | url, name, description (= meta description), inLanguage `vi-VN`, datePublished = dateModified = `2025-11-03T03:37:28.000Z` (= `publishedAt`), isPartOf `{@id: …/#website}`, publisher `{@id: #organization}`, primaryImageOfPage (ImageObject, url `api.knxstore.vn/media/…png`, caption), breadcrumb (BreadcrumbList 3 ListItem: Trang chủ `/`, Công tắc thông minh `/cong-tac-thong-minh`, tên SP) | thiếu: `mainEntity` |

Không có node `Product`, `Offer`, `AggregateRating`, `Review`, `FAQPage`, `Brand`, `sku`, `mpn`, `image[]`, `additionalProperty` ở mẫu chính. `isPartOf` trỏ `…/#website` nhưng không có node `WebSite` trong cùng `@graph` (tham chiếu treo, Chưa xác định node đó có ở trang khác không). Lý do mẫu chính không có `Product`: Chưa xác định (mẫu chính có `price:null`, `specs:[]`, `faqs:[]`, `b2b:true`; mẫu đối chiếu có đủ giá và có node `Product` + `FAQPage`; chưa tách được điều kiện nào quyết định).

Mẫu đối chiếu (để so): 11,077 B, 4 node: Organization, `ItemPage` (có `mainEntity` → `#product`), `Product`, `FAQPage` (chi tiết ở mục 10).

## 9. Số đo

| Chỉ số | Mẫu chính | Mẫu đối chiếu |
|---|---|---|
| HTML thô | 447,188 B | 251,362 B |
| gzip -9 (đo cục bộ) / gzip và br trên đường truyền | 66,213 B / 66,947 B và 41,675 B | 52,548 B / không đo |
| Tổng byte các thẻ `<script>` | 206,001 B (46.1%) | 168,688 B (67.1%) |
| Trong đó `self.__next_f` | 200,805 B (10 thẻ, 44.9%) | 156,328 B (8 thẻ, 62.2%) |
| Số thẻ `<script>` | 26 (12 inline, 14 src) | 24 (10 inline, 14 src) |
| Thẻ `<style>` | 0 | 0 |
| Text hiển thị (body, bỏ script/style/template; gồm cả panel `hidden` và drawer) | 8,695 ký tự, 10,736 B | 7,760 ký tự, 9,444 B |
| Text trong `#main-content` | 3,483 ký tự, 4,258 B | không có `#main-content`; các section nội dung ≈ 6,545 ký tự, 7,994 B |
| Ước lượng token (bytes/4): HTML thô | 111,797 | 62,840 |
| Ước lượng token (bytes/4): text hiển thị toàn body | 2,684 | 2,361 |
| Ước lượng token (bytes/4): text vùng nội dung | 1,064 | 1,998 |
| `<a>` | 242 | 71 |
| `<img>` | 90 (20 trong main) | 21 |
| `<table>` | 0 | 1 |
| `<h1>` / `<h2>` / `<h3>` / h4–h6 | 1 / 6 / 20 / 0 | 1 / 13 / 3 / 0 |
| `<button>` / `<form>` / `<input>` | 46 / 1 / 2 | 24 / 1 / 2 |
| `<svg>` / `<li>` / `<article>` / `<nav>` | 54 / 187 / 16 / 10 | 25 / 29 / 4 / 3 |

Heading theo thứ tự (mẫu chính): 18 h3 trong header (panel ẩn, 2 h3 rỗng) → `h1` tên SP → `h2` "Tổng quan về …" → `h3` "Các tính năng chính của SR-2422NK8-DA2" → `h3` "Sơ đồ kết nối của SR-2422NK8-DA2" → `h2` "Sản phẩm đề xuất" → `h2` "Yêu cầu báo giá" (drawer) → 3 `h2` footer (Danh Mục Sản Phẩm, Giải Pháp, Về chúng tôi). Text của header chiếm 3,977/8,695 ký tự (45.7%) do các panel menu ẩn.

## 10. Biến thể template / Chưa xác định

So sánh mẫu chính (skin B2B) với mẫu đối chiếu (Aqara LB-L02E, skin Matter):

| Hạng mục | Mẫu chính | Mẫu đối chiếu |
|---|---|---|
| Skin trong payload / cookie script | `"skin":"b2b"`, `knx-area=b2b` | `"skin":"matter"`, `knx-area=matter` |
| Cờ sản phẩm | `b2b:true, b2c:false` | `b2b:true, b2c:true` |
| Wrapper | `div#main-content[tabindex=-1]` giữa header và footer | `div.font-sans[style="--font-body:var(--font-matter)"]` bọc tất cả; không có `#main-content`; không `<main>` |
| Header | `header` (không label), search `form[role=search]` luôn trong DOM, menu 6 panel | `header[aria-label="Khu Matter Smarthome"]`, logo `/matter`, `nav[aria-label="Menu Matter"]` (mega menu `#matter-mega-menu`), search `form[role=search]` trong `#matter-search-drop[aria-hidden=true]`, nút "Trang B2B" → `/`, `aside#matter-mobile-panel[aria-hidden=true]` (menu mobile) |
| Footer | 3 cột (Danh Mục Sản Phẩm / Giải Pháp / Về chúng tôi), mỗi cột render 2 lần (mobile + desktop), 50 link | h2 "Thông tin liên hệ", "Danh mục sản phẩm" (menu riêng của Matter: Hub trung tâm, thermostat, Cảm biến Matter…), "Về chúng tôi"; 21 link; không lặp bản mobile/desktop |
| Breadcrumb | `nav[aria-label=Breadcrumb]` với `span` | `nav[aria-label="Đường dẫn"] > ol > li` ; mục 1 "Trang chủ" → `/matter` |
| H1 vs title | giống nhau | H1 `Đèn LED Bulb T2 (E27, Ánh sáng trắng) Matter Aqara - LB-L02E`; `<title>` `Đèn Thông Minh Aqara Bulb T2 E27 Trắng: 2700–6500K, Matter Thread` |
| Thương hiệu | `a[href=/brands/sunricher]` | `p` "Brand: Aqara" + `a[href=/brands/aqara]` |
| SKU | `SKU: SR-2422NK8-DA2` | `p` "Xuất xứ: CN", `p` "SKU: LB-L02E" nằm trong `#buy-new` |
| Giá | "Liên hệ để nhận báo giá" | `477.000đ`, giá cũ `530.000đ`, badge `-10%`, "(Đã bao gồm VAT) | Đặt trước" (SSR) |
| Hành động | "Thêm vào yêu cầu báo giá", gọi | "Mua ngay", "Thêm vào giỏ", "Liên hệ tư vấn" (`tel:0918918755`), so sánh |
| Mô tả ngắn / chip | không | 3 chip giao thức `Thread`, `Bluetooth`, `Zigbee` (từ `b2cContent.thuocTinh`), khối "Hoạt động cùng" (Homey, Home Assistant + 4 icon Apple Home, Google Home, SmartThings, Alexa); `shortDescription` HTML trong payload không in ra |
| Tab | 1 tab (Mô tả chi tiết) | 3 tab: Chi tiết sản phẩm / Thông số kỹ thuật / FAQ; chỉ panel đầu có trong DOM |
| Bảng trong DOM | 0 | 1 bảng nằm trong mô tả (`div.user-html.matter-html`): "So Sánh LB-L02E vs LB-L02D", 14 `tr` (1 header + 13 dòng), 3 cột (Tiêu chí / LB-L02E (Tunable White) / LB-L02D (RGB CCT)), `thead > tr > th[scope=col]`, cột đầu là `td` (không `th scope=row`), không `rowspan`/`colspan`, ô chứa `Ra>90` với `>` không escape; 3 dòng đầu: `Màu RGB | Không có | 16 triệu màu`, `Nhiệt độ màu trắng | 2700–6500K | 2000–9000K`, `Quang thông | 1100lm (6500K), 950lm (2700K) | 1100lm (6500K), 950lm (2700K)` |
| Bảng thông số kỹ thuật (tab specs) | không có dữ liệu | 14 dòng `specs` (Model, Đui bóng, Công suất 10.5W, Nguồn cấp, Quang thông, Nhiệt độ màu 2700–6500K, Điều chỉnh độ sáng 1%–100%, Góc chiếu 200°, Công suất chờ, Giao thức, Tương thích, Adaptive Lighting, MARS Technology, Kích thước Ø60 × 110 mm) chỉ có trong RSC payload và JSON-LD `additionalProperty`; text như `Ø60`, `Công suất chờ` không có trong DOM; theo bundle sẽ render `dl/dt/dd` khi tab được chọn |
| FAQ | không có | 5 Q&A chỉ có trong RSC payload và JSON-LD `FAQPage`; không có trong DOM; theo bundle sẽ render `details/summary` khi chọn tab |
| Tài liệu tải | không có (`downloads:[]`) | không có (`downloads:[]`) |
| Sản phẩm liên quan | h2 "Sản phẩm đề xuất", 16 card | h2 "Sản phẩm liên quan", 1 card (LB-L02D, `-10%`, `602.000 đ` gạch, `542.000 đ`) |
| Bài viết liên quan | không có | h2 "Bài viết liên quan", 3 `article` (tác giả "Tạ Minh Vũ", ngày `03/08/2026`, link `/{slug}.html`) |
| Zalo nổi / liên hệ | nút nổi, footer, `tel:` đều `0918918755` | nút nổi `https://zalo.me/0378628837`; footer hiển thị `0378.628.837` (`tel:0378628837`) + "Tư vấn qua Zalo" (`zalo.me/0378628837`); riêng nút "Liên hệ tư vấn" trong buy box vẫn `tel:0918918755` |
| JSON-LD | Organization + ItemPage (3,916 B) | Organization + ItemPage (`mainEntity`) + `Product` + `FAQPage` (11,077 B); vị trí: con đầu của `div.font-sans` |
| Header HTTP | như mục 1 | như mục 1 (`MISS` lần đầu) |

JSON-LD `Product` của mẫu đối chiếu: `name` (= H1), `sku` `LB-L02E`, `mpn` `LB-L02E` (payload `mpn:null` nên mpn lấy từ sku), `description` (= seoDescription), `url`, `image[]` 5 ImageObject (`api.knxstore.vn/media/…jpg`), `brand` (Brand, name, url), `manufacturer`, `category`, `additionalProperty[]` 14 PropertyValue (name, value, unitText; có dòng `Model` với `unitText:"MTAC01XXX"`), `countryOfOrigin` (Country `CN`), `offers` (Offer: `priceCurrency:"VND"`, `price:"477000"` kiểu chuỗi, `availability:"https://schema.org/BackOrder"`, `itemCondition`, `seller`, `areaServed`, `hasMerchantReturnPolicy` 30 ngày), `subjectOf` (Service Matter). Thiếu: `aggregateRating`, `review`, `gtin*`, `priceValidUntil`, `shippingDetails`. `ItemPage.breadcrumb`: mục 1 `https://knxstore.vn/matter`, mục 3 tên SP.

Metric mẫu đối chiếu (đo cùng cách): xem cột phải mục 9.

Chưa xác định:

| Điểm | Lý do |
|---|---|
| DOM thật của tab Thông số / Tài liệu / FAQ ở skin b2b | cả hai mẫu đều có `downloads:[]`; mẫu chính có `specs:[]`, `faqs:[]`. Chỉ có mô tả suy ra từ bundle (`dl/dt/dd`, `ul/li/a`, `details/summary`), chưa thấy HTML thật |
| URL pattern của file tải (PDF, CAD, manual) | không mẫu nào có `downloads` không rỗng |
| Điều kiện chọn skin Matter hay B2B cho `/products/{slug}` | chỉ có 2 mẫu; mẫu Matter có `b2c:true`, danh mục "Đèn thông minh Matter"; mẫu B2B có `b2c:false`; hàm `segmentOf` trong bundle trả `b2c` nếu `b2c` true, nhưng chưa xác nhận đó là điều kiện chọn skin trang |
| Điều kiện có/không node `Product` + `Offer` trong JSON-LD | chỉ 2 mẫu, nhiều biến đổi cùng lúc (giá, specs, faqs, skin) |
| Hành vi sau hydrate: variant, giá theo variant, tab, lightbox, gợi ý search | không chạy trình duyệt; bundle có hàm `request()` gọi `https://api.knxstore.vn/api/v1…` nhưng chưa xác định component nào gọi lúc hydrate và có thay đổi nội dung hay không |
| Trạng thái chọn variant mặc định trong DOM | HTML có cả hai nút `aria-pressed=false` dù "Xám" là `isDefault:true` |
| Cookie `knx-area` có làm đổi nội dung SSR không | chỉ thấy script đặt cookie; chưa gửi request có cookie để kiểm tra |
| Các biến thể khác trong 752 URL (sản phẩm có `priceHidden`, `clearance`, `bundles`, `related`, `guideHtml`, `warrantyMonths`, nhiều ảnh/variant, specs b2b) | chỉ khảo sát 2 URL |
| `Product` page cho slug không tồn tại | chỉ thấy `.md` → 404; không đo `/products/{slug-sai}` |
| Phiên bản Next.js chính xác | không có header/meta nêu version |
