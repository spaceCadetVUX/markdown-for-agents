# Trang chủ — cấu trúc hiện tại

> URL mẫu: https://knxstore.vn/ · nguồn sitemap: https://knxstore.vn/sitemaps/pages.xml (qua robots.txt → /sitemap.xml; 19 URL trong file, trang chủ là 1 URL) · số URL cùng loại: 1 · ngày khảo sát 2026-11-11

Cách đo: `curl -sSL -m 180 -A "Mozilla/5.0"` lấy HTML thô, phân tích bằng `python3 -I` (html.parser + regex) trên file đã lưu. Tổng 9 request tới trang chủ (1 lấy HTML thô, 5 đo lại thời gian/nén, 1 `Accept: text/markdown`, 2 thử `.md`). Phần "RSC" là payload `self.__next_f.push` đã giải mã.

## 1. Response & rendering

### 1.1 Header phản hồi (request lấy HTML thô)

| Header | Giá trị |
|---|---|
| status | 200 (HTTP/2) |
| content-type | `text/html; charset=utf-8` |
| cache-control | `s-maxage=3600, stale-while-revalidate=31532400` |
| x-nextjs-cache | `HIT` |
| x-nextjs-prerender | `1` |
| x-nextjs-stale-time | `300` |
| vary | `rsc, next-router-state-tree, next-router-prefetch, next-router-segment-prefetch, Accept-Encoding` |
| cf-cache-status | `DYNAMIC` |
| link | không có |
| x-robots-tag | không có |
| content-encoding | không có khi client không gửi `Accept-Encoding`; `gzip` khi gửi `gzip`; `zstd` khi gửi `gzip, br, zstd` (server chọn zstd, không thấy `br`) |
| content-length | không có |
| header khác | `server: cloudflare`, `speculation-rules: "/cdn-cgi/speculation"`, `strict-transport-security: max-age=31536000; includeSubDomains`, `x-frame-options: SAMEORIGIN`, `referrer-policy: same-origin`, `x-content-type-options: nosniff`, `alt-svc: h3=":443"` |

### 1.2 Thời gian tải và kích thước thật (7 lần GET toàn bộ HTML, tuần tự)

| Lần | Accept-Encoding gửi đi | Bytes nhận | TTFB (s) | Tổng (s) |
|---|---|---|---|---|
| 1 | không gửi (lần lấy file phân tích) | 957,496 | 0.48 | 0.84 |
| 2 | `gzip, br, zstd` → zstd | 68,133 | 1.49 | 39.24 |
| 3 | `identity` | 957,496 | 2.94 | 25.24 |
| 4 | `gzip` | 143,943 | 3.76 | 25.51 |
| 5 | `identity` | 957,496 | 0.43 | 0.84 |
| 6 | `zstd` | 104,143 | 0.50 | 0.90 |
| 7 | `Accept: text/markdown`, không gửi Accept-Encoding | 957,496 | không ghi | 3.81 |

- Không lần nào timeout với `-m 120`/`-m 180`; mọi lần đều nhận đủ 957,496 B (identity) và HTML kết thúc bằng `</body></html>`.
- Tổng thời gian dao động: 0.84–0.90 s (3 lần), 3.81 s (1 lần), 25.2–25.5 s (2 lần), 39.2 s (1 lần). TTFB của các lần chậm vẫn chỉ 1.5–3.8 s, nghĩa là thời gian tăng nằm ở giai đoạn truyền body, không phải chờ byte đầu. Nguyên nhân: Chưa xác định (không có dữ liệu phía server/Cloudflare).
- Kích thước zstd khác nhau giữa hai lần đo (68,133 B và 104,143 B) cho cùng 957,496 B thô.

### 1.3 Chế độ render suy ra: static prerender có revalidate (ISR), phục vụ từ cache của Next

Bằng chứng:
- `x-nextjs-cache: HIT` + `x-nextjs-prerender: 1` ở cả 7 lần.
- `cache-control: s-maxage=3600, stale-while-revalidate=31532400` (31532400 = 31536000 − 3600), đúng dạng header Next.js ISR với revalidate 3600 s.
- `x-nextjs-stale-time: 300`.
- Không có header `Link` per-request (font/CSS preload nằm trong `<head>` thay vì `Link` header, khác với /brands và /solution).
- HTML cùng kích thước 957,496 B ở mọi lần.
- Cây route trong payload: `(site)/(b2b)/__PAGE__`.
- Cloudflare không cache HTML (`cf-cache-status: DYNAMIC`); cache nằm ở lớp Next.js.

### 1.4 Hành vi hiện tại với yêu cầu Markdown (chỉ ghi kết quả)

| Request | Status | content-type | Ghi chú |
|---|---|---|---|
| `GET /` + `Accept: text/markdown` | 200 | `text/html; charset=utf-8` | body 957,496 B, giống bản thường; `vary` không có `Accept` |
| `GET /.md` | 404 | `text/html; charset=utf-8` | body 131,121 B (trang 404 HTML) |
| `GET /index.md` | 404 | `text/html; charset=utf-8` | body 131,131 B |

## 2. `<head>`

Kích thước `<head>`: 10,541 B (5 thẻ preload ảnh chiếm 6,497 B). `<title>` và các meta SEO nằm cuối `<head>` (sau preload/script, sau `meta[name=next-size-adjust]`).

| Thành phần | Giá trị |
|---|---|
| `<html>` | `lang="vi"`, class `onest_…__variable inter_…__variable jetbrains_mono_…__variable` |
| charset / viewport | `utf-8` / `width=device-width, initial-scale=1` |
| title (61 ký tự) | `KNX, DALI, Casambi, Matter: Thiết bị chính hãng \| KNXStore.vn` |
| meta description (153 ký tự) | `Phân phối chính hãng thiết bị KNX, DALI-2, Casambi, DMX, Matter và điều khiển HVAC tại Việt Nam. Hỗ trợ kỹ thuật, T&C, showroom TP.HCM. Gọi 0918 918 755.` |
| canonical | `https://knxstore.vn` (không có `/` cuối) |
| meta robots | không có |
| OG | `og:title` (= title), `og:description` (= description), `og:url` `https://knxstore.vn`, `og:site_name` `knxstore.vn`, `og:locale` `vi_VN`, `og:image` `https://api.knxstore.vn/media/0119edf7565dace9f515219397416e8c9d63e737c562aa55be779fd2d44db72f.avif`, `og:type` `website`. Không có `og:image:width/height/alt` |
| Twitter | `twitter:card` `summary_large_image`, `twitter:title`, `twitter:description`, `twitter:image` (cùng ảnh .avif) |
| hreflang | không có |
| `link rel=alternate` | không có (kể cả markdown/RSS) |
| khác | `meta[name=theme-color]` `#14467C`; `meta[name=next-size-adjust]` content rỗng; `link[rel=manifest]` `/manifest.webmanifest`; icon `/favicon.ico?favicon.16_cwkv0v-dvu.ico` (48x48), `/icon.svg?icon.39v289yelkh8f.svg`, `apple-touch-icon` `/apple-icon.png?apple-icon.264ni-uec6-1q.png` (180x180) |
| preload font | 4 file `/_next/static/media/*.woff2` (`as=font crossorigin type=font/woff2`) |
| preload ảnh | `/images/logo/knxstore-logo-tight.svg` + 5 `link[rel=preload][as=image][imageSrcSet]`: ảnh cover slide KNX (`e351bcf3….jpg`) và 4 ảnh avif của 4 card đầu khối "Mới ra mắt" (`dab0abe2…`, `527a6269…`, `b5a77f48…`, `f52cee4c…`); mỗi srcset 8–10 biến thể rộng 256–3840 |
| preload script | `/_next/static/chunks/3ivcc6t1s04dh.js` (`fetchpriority=low`); `https://www.googletagmanager.com/gtag/js?id=G-4D9XLJDVGG` (`as=script`) |
| CSS | 2 `link[rel=stylesheet][data-precedence=next]`: `/_next/static/chunks/1mmmxvxvvtkv8.css`, `1vbr8oh4939ca.css` |
| script | 11 `<script async src>` (4 không `crossorigin`, gồm `turbopack-0j8dnv21rrwyx.js`; 7 có `crossorigin`) + 1 `<script nomodule src=/_next/static/chunks/0cz1d0mv5g_q7.js>` |

## 3. Khung trang

### 3.1 Thứ tự con trực tiếp của `<body>`

| # | Phần tử | Ghi chú |
|---|---|---|
| B0 | `<script>` inline `document.cookie="knx-area=b2b; Path=/; SameSite=Lax"…` | ghi cookie `knx-area=b2b` |
| B1 | `a.fixed.left-4.top-4.z-[10000]…[href=#main-content]` | skip link "Bỏ qua, tới nội dung chính" (ẩn bằng `-translate-y-[200%]` đến khi focus) |
| B2 | `header.sticky.top-0.z-9999.bg-white` | 128,755 B, 165 `<a>`, 67 `<img>` |
| B3 | `div#main-content[tabindex=-1] > main` | trang chủ là trang duy nhất trong 3 loại có thẻ `<main>` |
| B4 | `div.fixed.top-0.left-0.z-99999…translate-x-full[inert]` | drawer "Yêu cầu báo giá" |
| B5 | `footer.mt-auto.border-t.border-zinc-200` | 17,608 B, 50 `<a>` |
| B6 | `a[data-floating-zalo][href=https://zalo.me/0918918755]` | nút Zalo nổi |
| B7+ | 10 `<script>` inline `self.__next_f` + script chunk | payload RSC |

### 3.2 Landmark và role

| Loại | Chi tiết |
|---|---|
| `<header>` | 1 |
| `<main>` | 1, bọc trong `div#main-content[tabindex=-1]` |
| `<aside>` | 0 |
| `<footer>` | 1 |
| `<nav>` (9) | header 6: `nav[aria-label="Điều hướng chính"]` + 5 nav mega (`aria-label`: "Toàn bộ danh mục sản phẩm", "Danh mục giải pháp", "Danh mục thương hiệu", "Danh mục công cụ", "Chuyên mục blog kỹ thuật"); footer 3: "Danh Mục Sản Phẩm", "Giải Pháp", "Về chúng tôi" |
| `role` | `search` ×2 (form header, form hero), `combobox` ×2 (`input#b2b-header-search`, `input#hero-search`), `group` ×2, `region` ×1 (`div#b2b-trending-panel[aria-label=Trending]`), `tablist` ×1 + `tab` ×1 |
| `<form>` / `<input>` | 2 / 2 (cả hai `type=search name=q`) |

### 3.3 Header (`header.sticky`)

Gồm 2 khối, cả header giống hệt byte-by-byte trên /brands/protopixel và /solution/giai-phap-khac (md5 trùng).

Khối 1, thanh trên (`div.border-b.border-zinc-200`, cao 56/64 px):
- `button[aria-label="Mở menu"]` (menu mobile). Nội dung drawer mobile không có trong HTML ban đầu.
- Logo: `a[href=/][aria-label="knxstore.vn — trang chủ"] > img[src=/images/logo/knxstore-logo-tight.svg][alt=knxstore][width=176][height=28]`.
- Search: `form[role=search]` (ẩn dưới `lg`) gồm `button#b2b-header-scope[aria-label="Tìm theo: Tất cả"]`, panel `div#b2b-header-scope-panel[role=group]` ("Tìm theo · chọn được nhiều": 7 chip `button[aria-pressed]` Tất cả, Sản phẩm, Bài viết, Thương hiệu, Giải pháp, Danh mục, Visual), `input#b2b-header-search[role=combobox][placeholder="Tìm theo SKU, tên sản phẩm, hãng, giao thức..."]`. Có thêm `button[aria-label="Tìm kiếm"]` cho mobile.
- Giỏ: `a[href=/cart][aria-label="Giỏ báo giá, 0 sản phẩm"]`.
- CTA: `a[href=/contact]` "Tư vấn dự án".
- Không có hotline/số điện thoại trong header.

Khối 2, thanh nav + mega menu (`div.relative.hidden.lg:block`, ẩn dưới `lg`):
- Thanh nav `nav[aria-label="Điều hướng chính"]`: 6 `button` (Trending, Danh mục sản phẩm, Giải pháp, Thương hiệu, Công cụ, Blog kỹ thuật), `a[href=/visual]` "Visual", pill `a[href=/matter]` "Matter Smarthome".
- 6 panel, tất cả có sẵn trong HTML với class `hidden` (display:none), hiện khi tương tác:

| Panel | Nội dung trong HTML |
|---|---|
| `div#b2b-trending-panel[role=region]` | "Từ khóa được tìm kiếm nhiều nhất": 8 link `/search?q=…` (DALI Casambi, Điều khiển KNX, Cảm biến hiện diện DALI, Điêu khiển điều hoà, Điều khiển Dali, Satel, Gateway Matter, Intesis). "Sản phẩm được tìm kiếm nhiều nhất": 16 link `/products/{slug}` (12 sản phẩm + 4 lặp lại CBU-DA-1P, Lithernet, CBU-DCS, Xpress), kèm giá và 21 `<img>`. 5 link bài viết `*.html` (1 lặp: Smart Thermostat) |
| `div#b2b-menu-categories` | 12 nhóm `h3 > a` + 46 link con + "Xem tất cả sản phẩm" `/products` (59 `<a>`). Nhóm: Bộ điều khiển trung tâm (5 con), Dimmer & Relay chiếu sáng (5), Điều khiển máy lạnh (6), Công tắc thông minh (6), Cảm biến (5), LED Driver & Nguồn (5), Thiết bị an ninh (6), Phụ kiện cho hệ thống (3), Thiết bị điện (3), Màn hình điều khiển hệ thống (0), Âm thanh đa vùng (2), Dịch vụ (0). Các panel menu (trừ Trending) có thêm chữ "Cuộn để xem thêm" |
| `div#b2b-menu-solutions` | 12 giải pháp xuất hiện 2 lần (danh sách "Theo công nghệ" + thẻ "Giải pháp …"): An ninh, Casambi, DALI, DMX, Giải pháp khác, HomeKit, HVAC, KNX, Lutron, Matter, Mobus (nguyên văn trong HTML), Zigbee; "Xem tất cả giải pháp" `/solution`; 12 `<img>` |
| `div#b2b-menu-brands` | 24 link `/brands/{slug}` có logo (satel, sunricher, legrand, airzone, kanonbus, hugo-muller, ir-tec, intesis, cool-automation, casambi, lunatone, micro-air, ofler, uitiot, 1home, core, helvar, euchips, steinel, siqitech, enertex-bayern-gmbh, tongdy, atios, protopixel) + "Xem tất cả 75 thương hiệu" `/brands` |
| `div#b2b-menu-tools` | KNX GA Planner `/tool/knx-ga-planner/index.html`, Công cụ khắc chữ `/tool/engraving/index.html`, Tra cứu gateway điều hoà `/tools/hvac-compatibility` |
| `div#b2b-menu-blogs` | 9 chuyên mục `/blogs/{an-ninh,cam-bien,chieu-sang,du-an,huong-dan,hvac,kien-thuc,knx,tin-tuc}`; 9 bài "Bài mới nhất" (ảnh + tiêu đề); "Xem tất cả bài viết" `/blogs` |

### 3.4 Footer

- Cột trái: logo; "CÔNG TY CP TÍCH HỢP HỆ THỐNG LIÊN MINH"; "MST: 0311846236 do sở kế hoạch đầu tư Thành phố Hồ Chí Minh cấp ngày: 29/12/2025"; địa chỉ "Văn phòng số SAV5-01.02, Lầu 1, Tháp 5, The Sun Avenue, 28 Mai Chí Thọ, Phường Bình Trưng, TP.HCM" (link Google Maps); `tel:0918918755` hiển thị "0918.918.755"; `mailto:support@knxstore.vn`; Facebook; Zalo; 2 ảnh huy hiệu (online.gov.vn, DMCA; `rel="nofollow noopener noreferrer"`).
- 3 cột link: "Danh Mục Sản Phẩm" (7 link: /bo-dieu-khien-trung-tam, /dieu-khien-chieu-sang, /dieu-khien-may-lanh, /cong-tac-thong-minh, /cam-bien-hien-dien, /led-driver-1, /kiem-soat-ra-vao), "Giải Pháp" (7 link `/solution/{knx,dali,casambi,an-ninh,hvac,matter,dmx}`), "Về chúng tôi" (/contact, /blogs, /clearance, `https://knxstore.vn/warranty-lookup`, /page/about-us). Cả 3 cột được render 2 lần trong DOM: bản mobile `div.border-t.sm:hidden` (accordion `details`) và bản desktop `div.hidden.sm:grid` (`nav` + `h2`), nên mỗi link xuất hiện 2 lần (50 `<a>` / 31 href duy nhất).
- Thanh dưới: "© 2026 KNXStore — CÔNG TY CP TÍCH HỢP HỆ THỐNG LIÊN MINH. Bảo lưu mọi quyền." + 4 link: /warranty-lookup, /chinh-sach-bao-mat.html, /chinh-sach-mua-hang-tai-knxstore-vn.html, /dieu-khoan-su-dung-website-knxstore-vn.html.

### 3.5 Thành phần nổi

| Thành phần | Selector | Ghi chú |
|---|---|---|
| Nút Zalo | `a[data-floating-zalo].group.fixed.right-[26px].bottom-[calc(8.75rem+env(safe-area-inset-bottom))].z-999` | tròn 52x52, nền `#0068ff`, chữ "Zalo", `motion-safe:animate-ping`; `target=_blank rel="noopener noreferrer"` |
| Drawer báo giá | `div.fixed.top-0.left-0.z-99999…translate-x-full[inert]` | HTML sẵn: h2 "Yêu cầu báo giá", "Chưa có thiết bị nào trong yêu cầu.", "Xem sản phẩm", "Tổng số lượng:0", `a[href=/cart]` "Xem và gửi yêu cầu" |
| Skip link | `a[href=#main-content]` | |
| Cookie banner, popup, back-to-top, widget chat khác | không thấy trong HTML ban đầu | `grep`: `back-to-top`, `Lên đầu`, `scroll-top`, `chatbot`, `tawk`, `messenger`, `crisp`, `livechat` đều 0 kết quả; "cookie" chỉ khớp script ghi cookie `knx-area`; "popup"/"modal" chỉ khớp `aria-haspopup` (nút scope search) và class `modal-content` của drawer báo giá |
| GA4 | `link[rel=preload][as=script]` tới `gtag/js?id=G-4D9XLJDVGG` | component client `GoogleAnalytics` (tên export trong payload RSC); không có script gtag inline trong HTML ban đầu |

## 4. Cây nội dung vùng chính

Vùng chính: `div#main-content[tabindex=-1] > main`. Có 8 `<section>` và 1 `<script>` JSON-LD là con trực tiếp, theo thứ tự DOM. Cột "SSR" ghi có text trong HTML ban đầu hay không. Heading trong header (h3 mega menu, 12 cái) và footer (h2, 3 cái) nằm ngoài `<main>`.

| # | Thẻ + id/class (rút gọn) | Heading | Nội dung tóm tắt | Trong `<main>` | Hiển thị mặc định | SSR |
|---|---|---|---|---|---|---|
| M0 | `script[type=application/ld+json]` (con đầu của `<main>`) | | 1 `@graph` 4 node (mục 8) | có | không hiển thị | có |
| M1 | `section[data-home-hero].relative.isolate.z-1.bg-[radial-gradient…]` | h1 "Nhà phân phối thiết bị tự động hóa tòa nhà" (là badge pill `h1.mb-7.inline-flex…rounded-full`); h2 "Tìm thiết bị tự động hóa tòa nhà" | p "Tra cứu theo SKU, hãng hoặc giao thức trong kho phân phối B2B."; search card (`form[role=search]`, `input#hero-search[role=combobox]`, nút "Tra cứu", `div[role=group][aria-label="Tìm theo"]` 7 chip: Tất cả, Sản phẩm, Bài viết, Thương hiệu, Giải pháp, Danh mục, Visual); `ul` 5 link nhanh: KNX `/solution/knx`, DALI-2 `/solution/dali`, Casambi `/solution/casambi`, Matter `/solution/matter`, Modbus `/search?q=Modbus`; số liệu "14+ năm kinh nghiệm", "810+ sản phẩm", "70+ thương hiệu" | có | có | text có; ô search là component client (`HeroSearchCard`), kết quả gợi ý không có trong HTML |
| M2 | `section.bg-white` > `div.knx-marquee > div.knx-marquee-track[style=--knx-marquee-duration:112.5s]` | không (`p.text-small` "Hãng phân phối chính hãng") | 2 `ul`, mỗi `ul` 75 `li > a[href=/brands/{slug}]` (28 có `<img>` logo, 47 chỉ chữ tên hãng). `ul` thứ hai `aria-hidden=true` (bản nhân đôi cho marquee, link `tabindex=-1`) | có | có (chạy CSS animation) | có, đủ 75 hãng ×2 |
| M3 | `section.bg-zinc-50` > `div.grid.lg:grid-cols-2` (component client `SolutionSlider`) | h2 theo slide: KNX, Casambi, DMX, An ninh, Matter, DALI, HVAC | Trái: 7 `div.absolute.inset-0` chứa ảnh cover (slide 1 `aria-hidden=false`, 6 slide sau `invisible aria-hidden=true`, `alt=""`) + 2 nút `button[aria-label="Giải pháp trước"/"Giải pháp sau"]`. Phải: `div.grid[aria-live=off]` chứa 7 khối chồng nhau bằng `[grid-area:1/1]`, mỗi khối có h2 + p (summary 131–283 ký tự) + `a` "Xem giải pháp" (`/solution/{knx,casambi,dmx,an-ninh,matter,dali,hvac}`) + bộ đếm "n / 7" | có | slide 1 hiện; slide 2–7 `invisible opacity-0 aria-hidden=true` | có: cả 7 slide có text đầy đủ trong HTML (không chỉ slide đầu) |
| M4 | `section.bg-white[aria-labelledby=home-featured-heading]` > `div.grid.grid-cols-2…lg:grid-cols-4` | h2#home-featured-heading "Sản phẩm nổi bật" | 12 `article` card (bảng 4.1) | có | có | có |
| M5 | `section.bg-white` > `div[role=tablist][aria-label="Bộ lọc sản phẩm nổi bật"]` + `a[href=/products]` "Xem tất cả" + `div.grid…lg:grid-cols-4` | không | tablist chỉ có 1 `span[role=tab][aria-selected=true]` "Mới ra mắt" (không phải button, không có tab thứ hai); 8 `article` card (bảng 4.2) | có | có | có |
| M6 | `section.bg-white` > `div.grid.sm:grid-cols-3` | không | 3 ô số: "819 sản phẩm trong catalogue", "75 thương hiệu phân phối", "12 giải pháp và giao thức" | có | có | có (số đã có trong HTML) |
| M7 | `section.bg-zinc-50` | h2 "Bài viết kỹ thuật mới" | `a[href=/blogs]` "Xem blog"; 3 card `a.group.flex.flex-col` (ảnh `aspect-video` `alt=""`, tiêu đề `span.clamp-2`, ngày dạng `dd/mm/yyyy` trong `span`, không dùng `<time>`): "Mở Rộng DALI-2 Ra Dimmer Triac, Relay, DMX Và Modbus TCP" 06/11/2026; "Line Segment Và Line Coupler Trong KNX: Chia Line Thế Nào" 02/11/2026; "Dự án điều khiển chiếu sáng Casambi cho nhà hàng tại 79 Mạc Thị Bưởi, TP.HCM" 29/10/2026 | có | có | có |
| M8 | `section.bg-white` > `ul.mt-6.max-w-[860px]` | h2 "Câu hỏi thường gặp" | 10 `li > details.group/faq > summary + div.user-html`; `details` đầu có thuộc tính `open`, 9 cái còn lại đóng; nội dung trả lời có đủ trong HTML | có | câu 1 mở, câu 2–10 đóng (nội dung vẫn trong DOM) | có |

Ngoài `<main>` (cùng cấp `div#main-content` trong body): header (3.3), drawer báo giá, footer (3.4), nút Zalo.

### 4.1 Khối "Sản phẩm nổi bật" (M4): 12 card

Cấu trúc 1 card: xem mục 5.2. Dữ liệu:

| # | Hãng | Model (SKU) | Giá hiển thị |
|---|---|---|---|
| 1 | Olfer | CBU-DA-1P | 4.865.000 đ |
| 2 | Casambi | CBU-A2D | 4.128.000 đ |
| 3 | Casambi | XPRESS-LR-B | 6.486.000 đ |
| 4 | Kanonbus | KTS-D3 | 30.261.000 đ |
| 5 | Atios | ADE-KD | 28.380.000 đ |
| 6 | Daikin | BRC1E63 | 2.100.000 đ |
| 7 | Core | CR-CG-DK-KNX-01 | 6.640.000 đ |
| 8 | CP Electronics | EBDSPIR-CB-DD | 14.680.000 đ |
| 9 | Intesis | IN770AIRXXSO000-DAIKIN | 34.997.000 đ |
| 10 | Kanonbus | KTS0-DALI | 8.630.000 đ |
| 11 | Casambi | CBU-DCS | 4.422.000 đ |
| 12 | Atios | ADE-R12 | 32.780.000 đ |

### 4.2 Khối "Mới ra mắt" (M5): 8 card

| # | Hãng | Model | Giá hiển thị |
|---|---|---|---|
| 1 | Euchips | EUPS20D-1HMC | Liên hệ |
| 2 | Euchips | EUPS15D-1HNC-280 | Liên hệ |
| 3 | Euchips | EUP12D-1HMC | Liên hệ |
| 4 | Euchips | LCP75D-4W24V | Liên hệ |
| 5 | Euchips | LCP75D-2W24V | Giá liên hệ |
| 6 | Euchips | LCP75D-1H24V | Giá liên hệ |
| 7 | Lunatone | Switch Cross | Liên hệ |
| 8 | Lunatone | DALI-2 IoT4 Gateway | Liên hệ |

Cả hai khối không có phân trang, không có bộ lọc chức năng (tab duy nhất chỉ là nhãn). Link "Xem tất cả" trỏ `/products`.

## 5. Chi tiết khối đặc biệt

### 5.1 Slider giải pháp (M3): toàn bộ slide có trong DOM

```html
<div class="grid" aria-live="off"><div aria-hidden="false" class="flex flex-col justify-center [grid-area:1/1] …"><h2 class="text-h3 text-zinc-900 lg:text-h2">KNX</h2><p class="mt-5 text-body text-zinc-700">Tìm hiểu hệ thống KNX: …</p><a class="mt-6 inline-flex h-11 …" href="/solution/knx">Xem giải pháp</a><p class="mt-4 font-mono …">1<!-- --> / <!-- -->7</p></div>
<div aria-hidden="true" class="flex flex-col justify-center [grid-area:1/1] …invisible opacity-0"><h2 …>Casambi</h2><p …>Giải pháp chiếu sáng Casambi: …</p><a tabindex="-1" … href="/solution/casambi">Xem giải pháp</a>…
```

Payload RSC của cùng slider (component `SolutionSlider`, module `$L90`) nhận prop `solutions[]` có thêm `contentHtml` (xem mục 6). Phần này không render ra DOM.

### 5.2 Card sản phẩm (dùng ở M4, M5)

```html
<article class="group relative rounded-card border … flex h-full flex-col p-3"><div class="relative aspect-square …"><img alt="CBU-DA-1P - Bộ điều khiển DALI Casambi Olfer" loading="lazy" data-nimg="fill" sizes="(max-width: 640px) 50vw, (max-width: 1024px) 33vw, 270px" srcSet="/_next/image?url=%2Fmedia%2F…jpg&amp;w=256&amp;q=75 256w, … 3840w" src="/_next/image?url=…&amp;w=3840&amp;q=75"/></div>
<div class="flex flex-wrap items-baseline …"><span class="rounded-none bg-[linear-gradient(90deg,#D9EAFF_0%,#FFFFFF_100%)] …">Olfer</span><span class="model-code font-mono …">CBU-DA-1P</span></div>
<a class="text-zinc-900 after:absolute after:inset-0 clamp-fluid mt-1" href="/products/casambi-bo-dieu-khien-dali-cbu-da-1p"><span class="whitespace-nowrap">CBU-DA-1P</span><span> - Bộ điều khiển DALI Casambi Olfer</span></a>
<div class="mt-auto pt-4"><div class="flex gap-2 min-h-[32px] items-end justify-between"><div class="min-w-0"><div class="text-[16px] font-semibold … tabular-nums …">4.865.000 đ</div></div><button type="button" aria-label="Thêm CBU-DA-1P - … vào yêu cầu báo giá" …><svg …/></button></div></div></article>
```

Lưu ý khi đọc text: tiêu đề card không phải heading, là `a` có nhiều `span` (model nằm trong `span.whitespace-nowrap`; model code xuất hiện cả ở `span.model-code` phía trên). Với card ADE-R12 tiêu đề bị tách `…Atios Smartcore - ` + `ADE-R12-1` + `.6` và giá `32.780.000 đ` nằm trong `div` riêng. Nút giỏ có `aria-label` dạng "…vào yêu cầu báo giá" (36 lần trong HTML) hoặc "…vào giỏ hàng" (2 card Atios ADE-KD, ADE-R12).

### 5.3 Marquee thương hiệu (M2)

```html
<div class="knx-marquee"><div class="knx-marquee-track" style="--knx-marquee-duration:112.5s"><ul class="flex shrink-0 items-center gap-3 pr-3"><li><a class="flex h-[56px] w-[120px] … " href="/brands/1home"><img alt="1Home" loading="lazy" width="120" height="40" … src="/_next/image?url=%2Fmedia%2F5238…avif&amp;w=3840&amp;q=75"/></a></li>…<li><a … href="/brands/weinzierl">Weinzierl</a></li></ul><ul … aria-hidden="true">…bản nhân đôi…</ul></div></div>
```

75 slug theo thứ tự alphabet (1home … weinzierl). Có 4 slug trong marquee không nằm trong `sitemaps/brands.xml`: `1home`, `euchips`, `lichtvision-probitsite`, `tongdy`.

### 5.4 Hero search (M1)

`input#hero-search[type=search][name=q][role=combobox][aria-expanded=false][aria-autocomplete=list][autocomplete=off]` placeholder "Tìm theo SKU, tên sản phẩm, hãng, giao thức...". Không có `listbox` hay danh sách gợi ý trong HTML ban đầu.

### 5.5 Mega menu (header)

```html
<div id="b2b-menu-categories" class="hidden"><nav aria-label="Toàn bộ danh mục sản phẩm" class="w-full border-b …"><div class="relative"><div class="mx-auto grid max-h-[480px] w-full grid-cols-2…"><div class="px-6"><h3 class="border-b … text-menu-parent…" aria-hidden="false"><a href="/bo-dieu-khien-trung-tam">Bộ điều khiển trung tâm</a></h3><ul class="mt-2 space-y-0.5"><li><a href="/dali-gateway">Bộ điều khiển DALI</a></li>…
```

### 5.6 FAQ (M8)

```html
<li class="border-b border-zinc-200"><details open="" class="group/faq"><summary class="flex cursor-pointer list-none …"><span class="text-card-title text-zinc-900">KNXStore cung cấp những loại thiết bị tự động hóa nào?</span><span aria-hidden="true" …>+</span></summary><div class="user-html pb-5 pr-8 text-body text-zinc-600">KNXStore cung cấp thiết bị tự động hóa cho nhà ở và tòa nhà thương mại: …</div></details></li>
```

Ký tự `+` (span `aria-hidden`) nằm trong `summary`, sẽ lẫn vào text nếu lấy toàn bộ `textContent` của `summary`.

### 5.7 Nút Zalo

```html
<a href="https://zalo.me/0918918755" target="_blank" rel="noopener noreferrer" aria-label="Chat Zalo với KNXStore" title="Chat Zalo" data-floating-zalo="" class="group fixed right-[26px] bottom-[calc(8.75rem+env(safe-area-inset-bottom))] z-999 …"><span aria-hidden="true" class="absolute inset-0 rounded-full bg-[#0068ff] opacity-40 motion-safe:animate-ping"></span><span class="relative text-[13px] font-bold tracking-tight">Zalo</span></a>
```

### 5.8 Cấu trúc payload RSC (nguyên nhân HTML nặng)

- `self.__next_f.push`: 9 lần gọi `push([1,"…"])` trong 10 thẻ `<script>` (1 thẻ là khởi tạo `push([0])`), tổng 501,705 B trong HTML, giải mã ra 437,791 B ở 248 row (8 row kiểu `T` = text, 240 row kiểu JSON).
- 7 row `T` (id `91`–`97`, 160,310 B) là `contentHtml` (HTML bài dài) của 7 giải pháp trong slider.
- 2 row layout gần giống hệt nhau (id `5` và `10`, 29,467 B và 29,468 B, chỉ khác số tham chiếu row) mang prop `nav` của component client `HeaderB2b` (dữ liệu trending, menu, blog).
- Footer xuất hiện trong nhiều row (id `16`, `2d` + các row con `1d`, `25`, `31`, `38`).

## 6. Nội dung chỉ có ở client hoặc KHÔNG thấy trong HTML ban đầu

| # | Nội dung | Có trong HTML ban đầu? | Bằng chứng |
|---|---|---|---|
| 1 | `contentHtml` của 7 giải pháp (KNX, Casambi, DMX, An ninh/Satel, Matter, DALI, HVAC): bài dài 4,425–26,640 ký tự text mỗi bài, chứa h1–h4 (Casambi có 1 `<h1>`), `table` (0–4/bài; KNX 0, DMX 1, Matter 2, HVAC 2, Casambi 3, DALI 3, An ninh 4), ảnh (KNX, Casambi, An ninh, DALI), mục FAQ (Matter, An ninh, HVAC) | Không. Chỉ có trong payload RSC (prop `solutions[].contentHtml` = `"$91"`…`"$97"`), không render ra DOM trang chủ | row `T` id 91–97, tổng 160,310 B; slide chỉ hiển thị `summary` |
| 2 | Prop khác của `solutions[]`: `productCount` (KNX 183, Casambi 96, DMX 33, An ninh 110, Matter 79, DALI 87, HVAC 55), `seoTitle`, `coverImage`, `position` | Không (chỉ `slug`, `name`, `summary` được render) | payload RSC, `"productCount"` |
| 3 | Trường sản phẩm không render: `stockStatus`, `oldPrice`, `shortDescription`, `b2c`, `b2b`, `clearance`, `isFeatured`, `priceRange`, `priceHidden` (mỗi trường xuất hiện 44–52 lần) | Không | payload RSC |
| 4 | Gợi ý tìm kiếm (combobox header và hero) | Không | `aria-expanded=false`, không có node `listbox` |
| 5 | Trạng thái giỏ báo giá (số lượng, danh sách) | Chỉ trạng thái rỗng: "Giỏ báo giá, 0 sản phẩm", "Chưa có thiết bị nào trong yêu cầu.", "Tổng số lượng:0". Nguồn dữ liệu giỏ: Chưa xác định | HTML + không chạy JS |
| 6 | Menu mobile (drawer sau nút "Mở menu") | Không | chỉ có `button[aria-label="Mở menu"]`, không có `role=dialog` trên trang chủ |
| 7 | Cây 404 ("Không tìm thấy trang", "Trang này không tồn tại… catalogue có 753 thiết bị") | Không | nằm trong payload RSC (slot `notFound` của layout), không ở DOM |
| 8 | Script GA4 (gtag) và bất kỳ widget nào chèn sau hydrate | Không | chỉ có thẻ preload tới `gtag/js?id=G-4D9XLJDVGG`; component `GoogleAnalytics` là client. Phần inject sau hydrate: Chưa xác định (không chạy JS) |
| 9 | Hành vi slider (autoplay, đổi slide), animation đếm số của `HeroYears` | Chỉ có trạng thái ban đầu (slide 1, "14+") | `aria-live=off`; hành vi: Chưa xác định |
| 10 | Dữ liệu menu mega đầy đủ | Có trong DOM nhưng ẩn (`class=hidden`) | mục 3.3 |
| 11 | Các panel/tab ẩn: slide 2–7, `ul` marquee thứ hai (`aria-hidden`), 9 câu FAQ đóng | Có text trong DOM | mục 4 |

Không thấy placeholder/skeleton: 0 `<template data-dgst`, 0 `BAILOUT_TO_CLIENT_SIDE_RENDERING`, 0 `animate-pulse`, 0 class chứa `skeleton`; không có ranh giới Suspense đang chờ (`<!--$?-->` = 0). 149 `loading="lazy"` là thuộc tính ảnh, không phải skeleton.

## 7. Liên kết

Tổng 406 `<a>`: skip link 1, header 165, `main` 187, drawer 2, footer 50, nút Zalo 1.

| Vùng | Mẫu URL nội bộ (số lượng) |
|---|---|
| `main` (187; 108 href duy nhất) | `/brands/{slug}` 150 (marquee 2×75); `/products/{slug}` 20 (12 nổi bật + 8 mới ra mắt); `/solution/{slug}` 11 (5 link nhanh hero: 4 `/solution/…` + `/search?q=Modbus`; 7 slide); `/{slug}.html` 3 (bài blog); `/search?q=` 1; `/products` 1; `/blogs` 1 |
| header (165; 148 duy nhất) | `/{category-slug}` 62; `/solution/{slug}` 24; `/brands/{slug}` 24; `/products/{slug}` 16; `/{slug}.html` 16 (14 bài blog + 2 trang công cụ `/tool/knx-ga-planner/index.html`, `/tool/engraving/index.html`); `/blogs[/{cat}]` 10; `/search?q=` 8; `/tools/hvac-compatibility` 1; `/`, `/products`, `/solution`, `/brands` mỗi loại 1 |
| footer (50; 31 duy nhất) | `/{category-slug}` 19 (+2 dạng tuyệt đối `https://knxstore.vn/warranty-lookup`); `/solution/{slug}` 14; `/{slug}.html` 3 (trang chính sách); `/blogs` 2; `/page/*` 2; `/` 1; `tel:` 1; `mailto:` 1; ngoài 5 |

- Link ngoài (5, đều ở footer, trừ Zalo nổi): `https://maps.app.goo.gl/1avSdM6EdwmTHqgg8`, `https://www.facebook.com/knxstore.vn`, `https://zalo.me/0918918755`, `https://online.gov.vn/nen-tang/870e6c15-f313-4206-a9b9-c4a5df24097a` (nofollow), `https://www.dmca.com/Protection/Status.aspx?ID=3aee6f49-5771-490c-97ee-858bd3323144` (nofollow). Nút Zalo nổi (1 `<a>` ngoài footer) cùng `https://zalo.me/0918918755`.
- Link nội bộ dùng dạng tương đối (`/…`), riêng 2 link footer dùng tuyệt đối `https://knxstore.vn/warranty-lookup`.
- Không có `link rel=alternate`, không có link/URL markdown ở bất kỳ đâu.
- Sitemap: `/robots.txt` có `Disallow: /admin/`, `Disallow: /search?`, `Sitemap: https://knxstore.vn/sitemap.xml` (index gồm pages, products, categories, brands, posts).

## 8. JSON-LD

1 thẻ `script[type=application/ld+json]` (7,439 ký tự JSON, 8,447 B kèm thẻ), đặt làm con đầu của `<main>`. Một `@graph` gồm 4 node:

| # | `@type` | `@id` | Trường có mặt |
|---|---|---|---|
| 0 | `WebSite` | `https://knxstore.vn/#website` | `url` (`https://knxstore.vn/`), `name` ("CÔNG TY CP TÍCH HỢP HỆ THỐNG LIÊN MINH"), `inLanguage` `vi-VN`, `publisher` (ref `#organization`), `potentialAction` `SearchAction` với `target` là `EntryPoint` (`urlTemplate: https://knxstore.vn/search?q={search_term_string}`), `query-input` |
| 1 | `["Organization","LocalBusiness"]` | `https://knxstore.vn/#organization` | `name` "KNXStore.vn", `legalName`, `url`, `foundingDate` "2012", `taxID` "0311846236", `logo` (ImageObject `https://knxstore.vn/assets/logo.png` 150x50, `@id` `#logo`), `image` (ImageObject `https://knxstore.vn/assets/knxstore.png` 1200x630), `description`, `telephone` "+84918918755", `email`, `address` (PostalAddress, `postalCode` "700000"), `geo` (10.785148, 106.748704), `hasMap`, `openingHoursSpecification` (Monday–Saturday 09:00–17:30), `priceRange` "50.000₫ - 200.000.000₫", `areaServed` "VN", `knowsAbout` (10 mục), `contactPoint` (sales, `availableLanguage`), `sameAs` (Facebook, YouTube) |
| 2 | `WebSite` | `https://knxstore.vn/#website` (trùng `@id` với node 0) | `url` (`https://knxstore.vn`, không `/` cuối), `name` "KNXStore.vn", `inLanguage`, `publisher`, `about` (ref `#organization`), `potentialAction` `SearchAction` với `target` là chuỗi URL |
| 3 | `["WebPage","FAQPage"]` | `https://knxstore.vn/#webpage` | `url`, `name` ("KNXStore.vn - Thiết bị KNX, Casambi và DALI - Phân phối chính hãng tại Việt Nam"), `description`, `inLanguage`, `datePublished` "2012-01-01T00:00:00+07:00", `dateModified` "2026-10-10T00:00:00+07:00", `isPartOf`, `about`, `publisher`, `primaryImageOfPage` (ImageObject 1200x630), `mainEntity`: 10 `Question`/`acceptedAnswer` |

Ghi nhận:
- Node 0 và 2 cùng `@id` nhưng khác `name`, `url` và dạng `target` của `SearchAction`.
- Không có node `BreadcrumbList`, `ItemList`, `Product`, `Brand`.
- 10 câu hỏi trong `FAQPage` khớp 10 câu FAQ hiển thị; 8 câu trả lời trùng chữ, 2 câu khác dấu câu (câu "tương thích với Apple HomeKit…" dùng `,` trong JSON-LD và ` - ` trong DOM; câu "đặt mua sản phẩm…" thiếu dấu `.` sau `support@knxstore.vn` trong DOM).
- Không có trường rỗng; `description` của Organization kết thúc bằng khoảng trắng thừa.
- Ảnh logo/image trong JSON-LD trỏ `https://knxstore.vn/assets/…`, còn `og:image` trỏ `https://api.knxstore.vn/media/…avif`. Việc các URL `/assets/logo.png`, `/assets/knxstore.png` có tồn tại: Chưa xác định (chưa tải).

## 9. Số đo

| Chỉ số | Giá trị |
|---|---|
| HTML thô | 957,496 B |
| gzip (đo local, mức 6 / 9) | 144,625 B / 143,816 B |
| gzip phía server (`Accept-Encoding: gzip`) | 143,943 B |
| zstd phía server | 68,133 B và 104,143 B (2 lần đo) |
| brotli | Chưa xác định (không đo được: python thiếu module `brotli`, server trả zstd khi client gửi `br`) |
| `<head>` | 10,541 B |
| `<body>` | 946,780 B |
| `<header>` | 128,755 B (13.4%) |
| `<main>` | 294,979 B |
| `<footer>` | 17,608 B |
| Tổng `<script>` (25 thẻ) | 511,303 B (53.4%) |
| Script inline `self.__next_f` (10 thẻ) | 501,705 B (52.4%); giải mã 437,791 B |
| Script `ld+json` (1) | 8,447 B |
| Script `src` ngoài (13 thẻ) | 1,037 B (chỉ tính thẻ) |
| `<style>` | 0 |
| Thuộc tính `style=""` (121) | 5,510 B |
| `<svg>` inline (59) | 24,320 B |
| Thuộc tính `srcset/srcSet` (110) | 182,214 B (19.0%) |
| Thuộc tính `class` (1,370) | 138,980 B (14.5%) |
| HTML sau khi bỏ mọi `<script>` | 446,193 B |
| Text hiển thị `<body>` (bỏ script/style) | 12,902 ký tự / 15,622 B |
| Text trong `div#main-content` | 7,690 ký tự / 9,144 B |
| Text ngoài vùng chính (header, footer, drawer, nút Zalo, skip link) | 5,212 ký tự (40.4% text hiển thị) |
| Ước lượng token (bytes/4) | HTML thô 239,374; text hiển thị 3,905; text vùng chính 2,286 |
| Thẻ | `<a>` 406, `<img>` 156 (149 `loading=lazy`, 7 eager; 52 `alt=""`), `<svg>` 59, `<table>` 0, `<button>` 47, `<form>` 2, `<input>` 2, `<section>` 8, `<article>` 20, `<nav>` 9, `<ul>` 35, `<li>` 338, `<details>` 13 (10 FAQ + 3 footer), `<iframe>` 0, `<video>` 0 |
| Heading | h1 ×1, h2 ×15, h3 ×18, h4–h6 ×0 |

Phân bố heading theo vùng: h1 và 11 h2 trong `<main>` (hero 1 h1 + 1 h2, 7 h2 slide, "Sản phẩm nổi bật", "Bài viết kỹ thuật mới", "Câu hỏi thường gặp"); 1 h2 "Yêu cầu báo giá" trong drawer; 3 h2 trong footer (Danh Mục Sản Phẩm, Giải Pháp, Về chúng tôi); 18 h3 trong mega menu header (trong đó 2 h3 có chữ rỗng). Card sản phẩm, khối "Mới ra mắt", khối số liệu và marquee không có thẻ heading.

## 10. Chưa xác định / lưu ý

Chưa xác định:
- Nguyên nhân tổng thời gian tải ngẫu nhiên 25–39 s (3/7 lần), trong khi TTFB nhỏ. Dữ liệu chỉ từ phía client.
- Dung lượng brotli.
- Cấu trúc menu mobile (không có trong HTML); hành vi sau hydrate (gợi ý tìm kiếm, nguồn dữ liệu giỏ báo giá, autoplay slider, animation số liệu, script inject sau tải); chưa chạy JS nên không biết có widget/popup nào xuất hiện sau đó.
- Việc `contentHtml` có được render ở `/solution/{slug}` hay không: chưa tải trang đó (ngoài phạm vi).
- Thời điểm bản prerender được tạo (không có header `age`/`date` phản ánh việc này; `date` chỉ là thời điểm phản hồi).
- Sự tồn tại của `/assets/logo.png`, `/assets/knxstore.png` (JSON-LD).

Lưu ý về tính đại diện và số liệu:
- Số liệu trong cùng trang không thống nhất: hero "810+ sản phẩm" và "70+ thương hiệu"; khối M6 "819 sản phẩm", "75 thương hiệu", "12 giải pháp"; payload RSC (cây 404) "753 thiết bị"; `sitemaps/brands.xml` có 71 URL `/brands/{slug}` (marquee có 75, trong đó 4 slug không có trong sitemap).
- Trang này là snapshot ISR (revalidate 3600 s theo header); card nổi bật/mới ra mắt, 3 bài blog và số liệu có thể đổi sau mỗi lần revalidate.
- Header, footer, drawer và nút Zalo giống hệt byte-by-byte trên /brands/protopixel và /solution/giai-phap-khac; chúng chiếm 5,212 ký tự text và khoảng 146 KB HTML ở mỗi trang.
- Menu "Giải pháp" ghi nhãn "Mobus" (nguyên văn); H1 là badge pill, H2 mới là câu tiêu đề lớn của hero.
- Khối "Mới ra mắt" có tablist 1 tab không tương tác, không heading.
