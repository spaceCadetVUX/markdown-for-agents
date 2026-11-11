# Trang sản phẩm skin Matter — cấu trúc hiện tại

> https://knxstore.vn/products/cam-bien-chuyen-dong-eve-motion-matter-over-thread-ipx3 · products.xml (752 URL `/products/{slug}`, `lastmod` 2026-05-26T09:01:25.000Z) · khảo sát 2026-11-11

Phương pháp: `curl` HTML thô (UA `Mozilla/5.0`, không cookie), phân tích bằng `html.parser`; giải mã payload `self.__next_f` bằng parser row của Flight (xử lý đúng row `T` theo byte length); đối chiếu DOM với payload và JSON-LD. Đọc thêm 7 file JS tĩnh `/_next/static/chunks/*.js` liệt kê trong payload để xác định hành vi client của gallery/tab/buy box (chỗ nào suy ra từ bundle đều ghi rõ). Tải thêm `/_next/image?...` và `/media/...` để kiểm tra ảnh gallery (mục 5). Có tải 1 trang B2B để so khung trang (mục 10). Số request vào chính URL này: 5 (identity, gzip, br, `Accept: text/markdown`, `.md`). Chưa chạy trình duyệt: mọi mô tả "sau hydrate" là suy ra từ HTML, payload, bundle và ảnh chụp màn hình do team cung cấp, không phải quan sát DOM sau render.

## 1. Response & rendering

| Header | Giá trị |
|---|---|
| status | 200 (HTTP/2) |
| content-type | `text/html; charset=utf-8` |
| cache-control | `s-maxage=3600, stale-while-revalidate=31532400` |
| x-nextjs-cache | `HIT` (cả 4 request GET 200); request `.md` (404): `MISS` |
| x-nextjs-prerender | `1` |
| x-nextjs-stale-time | `300` |
| vary | `rsc, next-router-state-tree, next-router-prefetch, next-router-segment-prefetch, Accept-Encoding` |
| cf-cache-status | `DYNAMIC` |
| link | không có |
| x-robots-tag | không có |
| content-encoding | không có khi không gửi `Accept-Encoding`; `gzip` khi gửi `gzip`; `br` khi gửi `br` |
| set-cookie | không có (không có header `set-cookie` ở bất kỳ response nào đã đo) |
| Header khác | `server: cloudflare`, `speculation-rules: "/cdn-cgi/speculation"`, `strict-transport-security`, `x-frame-options: SAMEORIGIN`, `x-content-type-options: nosniff`, `referrer-policy: same-origin`, `alt-svc: h3`, `nel`/`report-to` |

Kích thước trên đường truyền: identity 354,232 B; gzip 57,794 B; br 45,834 B (giải nén gzip khớp byte-by-byte với bản identity).

Chế độ render: ISR (prerender tĩnh + revalidate theo thời gian). Bằng chứng: `x-nextjs-prerender: 1`; `x-nextjs-cache: HIT`; `s-maxage=3600` (revalidate 3600 s); `stale-while-revalidate=31532400` = 31,536,000 − 3,600; record `0` của payload có `"S":true` (trang dynamic `/cong-tac-matter` có `"S":false`; chỉ ghi nhận tương quan); không có header `link`.

Route (đọc từ record `0` của payload): `(site)/products/[slug]/__PAGE__`, `slug` là dynamic segment. Wrapper skin (header + footer + router con) nằm ở record `5`, là node con của segment `[slug]` trong cây route (suy ra từ vị trí `$L5` trong record `0`), không phải ở layout `(site)`. Cùng `"b":"rJVjL74JjgHtrkQQTzrjV"` (build id) với 2 trang Matter còn lại. Phiên bản Next.js chính xác: Chưa xác định (không có header/meta nêu version); bundler Turbopack (file `turbopack-0j8dnv21rrwyx.js`).

Hành vi hiện tại với yêu cầu markdown (chỉ ghi kết quả):

| Request | Status | content-type | Ghi chú |
|---|---|---|---|
| `Accept: text/markdown` trên URL sản phẩm | 200 | `text/html; charset=utf-8` | body 354,232 B, giống hệt bản không có `Accept` (`cmp` trùng); `vary` không chứa `Accept` |
| URL + `.md` | 404 | `text/html; charset=utf-8` | body 131,577 B (trang lỗi Next), `x-nextjs-cache: MISS`, `cache-control: s-maxage=60, stale-while-revalidate=31535940` |

## 2. `<head>`

Head dài 5,756 B (không gồm `<body>`).

| Thành phần | Giá trị |
|---|---|
| `<html>` | `lang="vi"`, `class` chỉ chứa 3 biến font (`onest_…__variable inter_…__variable jetbrains_mono_…__variable`); không có class/data-attribute của skin; `<body>` không có attribute |
| title | `Cảm Biến Chuyển Động Eve Motion: Thread Matter, IPX3 \| KNXStore.vn` (66 ký tự) |
| meta description | `Eve Motion – cảm biến chuyển động không dây IPX3, góc quét 120°, tầm 9 m, hỗ trợ Matter Thread, pin AAA, bảo mật tuyệt đối.` (123 ký tự) |
| meta keywords / author | `Eve,Điều khiển Matter,Eve Motion` / `KNX Store` |
| canonical | `https://knxstore.vn/products/cam-bien-chuyen-dong-eve-motion-matter-over-thread-ipx3` (tự tham chiếu) |
| robots | không có thẻ `meta name=robots` |
| OG (5) | `og:title` (= title), `og:description` (= meta description), `og:url`, `og:image` = `https://api.knxstore.vn/media/cf9cf061…302d3c.jpg`, `og:type=website`; không có `og:site_name`, `og:locale`, kích thước ảnh |
| Twitter (4) | `twitter:card=summary_large_image`, `twitter:title`, `twitter:description`, `twitter:image` (cùng giá trị OG) |
| hreflang / `rel=alternate` | không có / không có (không có link markdown) |
| Khác | `meta theme-color #14467C`, `meta next-size-adjust`, `link rel=manifest /manifest.webmanifest`, 2 `link rel=icon` + `apple-touch-icon` |
| Preload (12) | 4 font woff2; logo `/images/logo/knxstore-logo-tight.svg`; ảnh chính (`imagesrcset` 8 candidate 640–3840, `q=90`, `imagesizes="(max-width: 1024px) 100vw, 680px"`, `fetchpriority=high`); 4 PNG `/images/matter-icons/{apple-home,google-home,smartthings,alexa}.png`; 1 script chunk (`fetchpriority=low`); `gtag/js?id=G-4D9XLJDVGG` (chỉ preload; không có thẻ `<script src=gtag>` trong HTML) |
| Stylesheet | 2: `/_next/static/chunks/1mmmxvxvvtkv8.css`, `/_next/static/chunks/1vbr8oh4939ca.css` (cùng file với trang B2B) |
| Script trong head | 12 `async` + 1 `noModule`; thêm 1 `script#_R_` ở cuối body: tổng 14 thẻ `<script src>` |
| JSON-LD | không nằm trong head; nằm trong body (mục 8) |

## 3. Khung trang

Landmark thực tế (theo thứ tự DOM):

| Phần tử | Vai trò |
|---|---|
| `div[hidden]` (con đầu `<body>`, chứa `<!--$--><!--/$-->`) | boundary rỗng của Next |
| `div.font-sans[style="--font-body:var(--font-matter)"]` | wrapper bọc toàn bộ nội dung trang (header → Zalo); không phải landmark |
| `header.sticky.left-0.top-0.z-9999[aria-label="Khu Matter Smarthome"]` | header |
| `nav[aria-label="Menu Matter"]` | menu desktop (trong header) |
| `form[role=search]` trong `div#matter-search-drop[aria-hidden=true]` | tìm kiếm (hộp thả xuống, ẩn) |
| `aside#matter-mobile-panel[aria-label="Menu khu Matter"][aria-hidden=true]` + `nav[aria-label="Menu Matter trên điện thoại"]` | menu mobile, nằm trong `div.fixed.inset-0.z-[10000]` |
| `nav[aria-label="Đường dẫn"]` | breadcrumb |
| `div[role=tablist][aria-label="Nội dung sản phẩm"]` | tab (không có `role=tabpanel`, không có `aria-controls`) |
| `div[role=group][aria-label="Sản phẩm liên quan"]` | dải card liên quan (cuộn ngang) |
| `footer.mt-auto` | footer (không có `nav` bên trong) |

Không có `<main>`, không có `role=main`, không có `#main-content`, không có skip link. Vùng nội dung chính là các khối anh em trực tiếp trong `div.font-sans`, giữa JSON-LD và drawer báo giá: `section.relative.bg-white.pb-20.pt-8` (breadcrumb + khối sản phẩm), `div.mx-auto.w-full.max-w-[1280px]` (tab + mô tả), `section.border-t` (sản phẩm liên quan), `div.bg-white > section.py-10` (bài viết liên quan).

Header (SSR, 9,358 B, 17 link):

| Vị trí | Nội dung |
|---|---|
| Logo | `a[href=/matter][aria-label="KNXStore — Matter Smarthome"]` chứa `img[src=/images/logo/knxstore-logo-tight.svg][alt=KNXStore]` + `span` text "matter" (chữ "KNXSTORE" nằm trong ảnh SVG) |
| Mục "Danh mục sản phẩm" | `button[aria-expanded=false][aria-controls=matter-mega-menu]` + `div#matter-mega-menu` (class có `invisible opacity-0`, lưới 4 cột); dữ liệu menu lấy từ prop `menu[]` (`title`, `url`, `children`) của layout (record `5`) |
| Cột 1 | Công tắc `/cong-tac-matter`; Cảm biến `/cam-bien-matter`; Camera `/camera-thong-minh` |
| Cột 2 | Động cơ rèm `/dong-co-rem-matter`; Ổ cắm `/o-cam-thong-minh-matter`; Thermostat `/dieu-khien-dan-lanh` |
| Cột 3 | Driver Matter `/driver-matter`; Gateway Matter `/gateway-matter`; nhóm "Hub trung tâm" `/hub-trung-tam` với con: "Smart hub Homey" `/hub-trung-tam-homey` (link), "Smart hub Apple Home", "Smart hub Google Home", "Smart hub Smartthings" (3 mục này là `span`, `url:null`, không phải link) |
| Cột 4 | nhóm "An ninh Matter" (`span`, không link) với con: Khóa cửa `/khoa-thong-minh-matter`, Chuông hình `/chuong-hinh` |
| Link khác trong nav | "Blogs" `/blogs`; "Liên hệ" `/contact` |
| Bên phải | "Trang B2B" `a[href=/]` (class `hidden …`, ẩn ở màn hình nhỏ); nút "Tìm kiếm" (`aria-controls=matter-search-drop`); `a[href=/cart][aria-label="Giỏ báo giá, 0 sản phẩm"]` (chỉ có `svg`, không có badge số trong HTML); nút "Mở menu" (`aria-controls=matter-mobile-panel`) |
| Hộp tìm kiếm | `form[role=search] > input#matter-search-input[type=search][name=q][placeholder="Tìm công tắc, cảm biến, hub Matter…"]` + nút submit; ẩn (`aria-hidden=true`, `tabindex=-1`) |

Footer (SSR, 7,929 B, 21 link, 3 `h2`, một bản duy nhất, không lặp mobile/desktop): logo `/matter`; cột "Thông tin liên hệ" (MST 0311846236 cấp 29/12/2025, địa chỉ Sun Avenue, `tel:0378628837` hiển thị `0378.628.837`, "Tư vấn qua Zalo" `https://zalo.me/0378628837`, `mailto:support@knxstore.vn`, Facebook, 2 ảnh huy hiệu: Bộ Công Thương `online.gov.vn` và DMCA); cột "Danh mục sản phẩm" (7 link: Hub trung tâm, `thermostat`, Cảm biến Matter, `Côn tắc thông minh` (nguyên văn trong nguồn), Động cơ rèm, Nguồn đèn Matter, Camera); cột "Về chúng tôi" (Liên hệ tư vấn `/contact`, "Thiết bị tự động hoá toà nhà" `/`, Blogs `/blogs`); thanh dưới: © 2026 + 4 link (`/warranty-lookup`, `/chinh-sach-bao-mat.html`, `/chinh-sach-mua-hang-tai-knxstore-vn.html`, `/dieu-khoan-su-dung-website-knxstore-vn.html`).

Thành phần nổi / ẩn:

| Thành phần | Selector | Trạng thái trong HTML ban đầu |
|---|---|---|
| Nút Zalo | `a[data-floating-zalo][href="https://zalo.me/0378628837"][target=_blank]`, chữ "Zalo", `aria-label="Chat Zalo với KNXStore"` | hiển thị (fixed, góc phải dưới) |
| Drawer báo giá | `div[inert].fixed.top-0.left-0.z-99999`, h2 "Yêu cầu báo giá", "Chưa có thiết bị nào trong yêu cầu.", link "Xem sản phẩm" `/products`, "Tổng số lượng: 0", link "Xem và gửi yêu cầu" `/cart` | ẩn (`inert`) |
| Thanh mua mobile | `div[inert].fixed.inset-x-0.bottom-0 … lg:hidden pointer-events-none translate-y-full` (tên SP + giá + nút "Thêm báo giá") | ẩn |
| Panel menu mobile | `aside#matter-mobile-panel` | ẩn |
| Hộp tìm kiếm | `div#matter-search-drop` | ẩn |
| Compare modal, lightbox ảnh, cart modal | không có trong HTML | client mở khi tương tác |
| Cookie banner, popup, back-to-top, widget chat khác | không thấy trong HTML ban đầu | Chưa xác định (có thể do client chèn sau) |

## 4. Cây nội dung chính (theo thứ tự DOM)

Cột "Trong main": trang này không có `<main>`/`#main-content`, nên cột ghi "vùng nội dung" (khối anh em trong `div.font-sans`) hay "khung".

| # | Thẻ + selector rút gọn | Heading | Nội dung tóm tắt | Vùng | Hiển thị mặc định | SSR / client |
|---|---|---|---|---|---|---|
| 0 | `div[hidden]` | – | boundary rỗng | khung | ẩn | SSR |
| 1 | `div.font-sans[style=--font-body:var(--font-matter)]` | – | wrapper skin | khung | – | SSR |
| 2 | `script` (117 B) | – | `document.cookie="knx-area=matter; Path=/; SameSite=Lax"` + `; Secure` nếu https | khung | – | SSR (script inline) |
| 3 | `header[aria-label="Khu Matter Smarthome"]` | không có heading | xem mục 3 | khung | thanh hiện; mega menu và tìm kiếm ẩn | SSR; mở/đóng = client |
| 4 | `div.fixed.inset-0.z-[10000] > aside#matter-mobile-panel` | – | menu mobile (`details/summary`), 15 link | khung | ẩn | SSR |
| 5 | `script[type=application/ld+json]` (11,834 B) | – | 4 node (mục 8) | khung | không hiển thị | SSR (lặp trong RSC row `6c`) |
| 6 | `section.relative.bg-white.pb-20.pt-8 > div.max-w-[1280px]` | – | breadcrumb + khối sản phẩm (27,358 B, 423 ký tự text) | vùng nội dung | hiện | SSR |
| 6a | `nav[aria-label="Đường dẫn"] > ol > li` | – | Trang chủ `/matter` › Điều khiển Matter `/san-pham-matter` › `span[aria-current=page]` tên đầy đủ; dấu ngăn là `svg` trong `span[aria-hidden]` | | hiện (tên bị cắt bằng CSS `truncate`, DOM có đủ chữ) | SSR |
| 6b | cột trái: `div.group.aspect-square > button[aria-label^="Phóng to ảnh"] > span > img` + 2 `button[data-gallery-arrow]` ("‹", "›") + dải 6 `button[aria-label="Xem ảnh n trên 6"] > img` (64×64) | – | gallery 6 ảnh (chi tiết mục 5) | | ảnh chính + 6 thumbnail; lightbox đóng | SSR (img có `src`/`srcset`); đổi ảnh, lightbox = client |
| 6c | `p` "Brand:" + `a[href=/brands/eve]` | – | thương hiệu Eve | | hiện | SSR |
| 6d | `h1.text-[22px]` | h1: "Cảm biến chuyển động Eve Motion, Matter over Thread, IPX3" | tên SP (1 text node, không chia span) | | hiện | SSR |
| 6e | `div.mb-4.5.flex > span` | – | chip "Thread" (`span`, không phải link) | | hiện | SSR |
| 6f | `div#buy-new > div > p > span.text-[30px]` | – | giá `2.772.000đ` (component client `$L46`, vẫn SSR ra HTML) | | hiện | SSR |
| 6g | `div#buy-new > div` (row `4c`) | – | "(Đã bao gồm VAT)" + `|` + chấm + "Đặt trước" (nhãn tồn kho, class `text-orange-dark`) | | hiện | SSR (server element) |
| 6h | `div#buy-new > p` | – | "Xuất xứ: **TW**" (mã ISO thô); `p[data-live-sku]` "SKU: **Eve Motion**" (component client `LiveSku`) | | hiện | SSR |
| 6i | `div` buy box: `button[aria-label="Giảm số lượng"]`, `input[type=number][min=1][max=100000][value=1]`, `button[aria-label="Tăng số lượng"]` | – | chọn số lượng | | hiện | SSR; state = client |
| 6j | `button[type=button]` "Mua ngay"; `button[type=button]` "Thêm vào giỏ"; `a[href=tel:0918918755]` "Liên hệ tư vấn"; `button[aria-haspopup=dialog][aria-label="Thêm vào so sánh"][title=…]` (chỉ có `svg`) | – | 4 CTA, không có `<form>` (form duy nhất của trang là tìm kiếm) | | hiện | SSR; "Mua ngay": dispatch `addQuoteItem` rồi `router.push("/cart")`; "Thêm vào giỏ": dispatch `addQuoteItem` rồi mở cart modal (cả hai suy ra từ bundle) |
| 6k | `div[inert].fixed.inset-x-0.bottom-0` | – | tên SP, giá, nút "Thêm báo giá" | | ẩn | SSR; hiện bằng `IntersectionObserver` (suy ra từ bundle) |
| 6l | `div.mt-5.rounded-lg` | không có heading (nhãn là `span`) "Hoạt động cùng" | 4 `img` (apple-home, google-home, smartthings, alexa; `alt` tên nền tảng, `width=64 height=28`, PNG tĩnh `/images/matter-icons/*.png`) + 2 `span` chữ "Homey", "Home Assistant"; không có link | | hiện | SSR |
| 6m | `p.mt-8.border-t` | – | "Danh mục:" + 2 link `/san-pham-matter` "Điều khiển Matter", `/cam-bien-matter` "Cảm biến thông minh Matter" | | hiện | SSR |
| 7 | `div.mx-auto.w-full.max-w-[1280px] > … > section.scroll-mt-[64px] > div[role=tablist]` | – | 3 tab: "Chi tiết sản phẩm" (`aria-selected=true`), "Thông số kỹ thuật", "FAQ" | vùng nội dung | tab 1 hiện | SSR (nút); chọn tab = `useState` client, không đổi URL/hash |
| 8 | `div.overflow-hidden.transition-[height] > div.user-html.matter-html` | h2 "Cảm Biến Chuyển Động Eve Motion: Thread Matter, Góc 120°, Tầm 9m, Dùng Được Ngoài Trời"; h2 "Nền Tảng Tương Thích Với Eve Motion"; h3 "Voice Assistant"; h3 "Tính Năng Theo Nền Tảng"; h2 "Trong Hộp Eve Motion Có Gì?" | panel của tab đang chọn: 1,835 ký tự text, 3,932 B HTML, 4 `p` mở đầu, 2 `img` (`/media/…jpg`, `alt`, `style` pixel cố định), 2 `table`, 2 `ul`, 1 `hr` | | hiện | SSR (HTML CMS in bằng `dangerouslySetInnerHTML`); tab 2 và 3 không có trong DOM |
| 9 | `section.border-t.border-zinc-200 > div[role=group]` | h2 "Sản phẩm liên quan" + p "Thiết bị tương tự và cùng danh mục — cuộn ngang để xem thêm" | 16 `article` (65,641 B; 1,428 ký tự) | | hiện (cuộn ngang) | SSR |
| 10 | `div.bg-white > section.py-10` | h2 "Bài viết liên quan"; 3 h3 | 3 `article` (bài blog): ảnh (`alt=""`), tác giả + ngày ("Tạ Minh Vũ · 03/08/2026", "Tạ Minh Vũ · 05/08/2026", bài thứ 3 chỉ có ngày "18/12/2025", không có tác giả), tiêu đề, "Đọc tiếp" | | hiện | SSR |
| 11 | `div[inert].fixed.top-0.left-0.z-99999` | h2 "Yêu cầu báo giá" | drawer báo giá rỗng | khung | ẩn | SSR |
| 12 | `footer.mt-auto` | 3 h2 | xem mục 3 | khung | hiện | SSR |
| 13 | `a[data-floating-zalo]` | – | "Zalo" | khung | hiện | SSR |
| 14 | `script#_R_` + 3 script `self.__next_f.push` + script init | – | payload RSC (200,621 B) | – | – | – |

Cấu trúc card "Sản phẩm liên quan" (`article.group.relative.rounded-card`): `div.aspect-square > img[loading=lazy][alt=tên SP]` → `span` thương hiệu + `span.model-code` mã model → `a[href=/products/{slug}]` (tên; token model có thể bọc `span.whitespace-nowrap`; `after:absolute after:inset-0` biến cả card thành vùng bấm) → giá (`1.890.000 đ`…) + `button[aria-label="Thêm {tên} vào giỏ hàng"]`. 16 card, đều có giá (990.000 đ … 13.857.000 đ), không có nhãn tồn kho trong DOM. Card thuộc nhiều thương hiệu (Remotec, Tailwind, JINK, Apple, Eve), một số không phải Matter-over-Thread. Trường `related` của object sản phẩm là `[]`, tức nguồn 16 card không phải trường này (nguồn thật: Chưa xác định).

### 4b. Đối chiếu từng khối trong ảnh chụp màn hình

| Khối trong ảnh | Text/giá trị trong HTML ban đầu? | Nguồn dữ liệu / ghi chú |
|---|---|---|
| Logo "KNXSTORE matter" | có một phần: `img[alt=KNXStore]` (SVG) + text "matter" | chữ "KNXSTORE" nằm trong file SVG, không có text node |
| "Danh mục sản phẩm" + dropdown | có: nút + `#matter-mega-menu` (invisible) với 12 link và 4 `span` không link | payload record `5` (prop `menu`) |
| "Blogs", "Liên hệ", "Trang B2B" | có (3 link `/blogs`, `/contact`, `/`) | SSR |
| Icon tìm kiếm | có nút; ô nhập nằm trong panel ẩn | SSR; gợi ý tìm kiếm = client |
| Icon giỏ hàng, badge "1" | icon có; số "1" KHÔNG có: HTML ghi `aria-label="Giỏ báo giá, 0 sản phẩm"` và không có thẻ badge | số lượng giỏ do client render sau hydrate (state giỏ lưu ở client; chi tiết lưu trữ: Chưa xác định) |
| Breadcrumb | có đủ 3 mục, tên SP đầy đủ | SSR; "…" trong ảnh là CSS `truncate` |
| Gallery: ảnh lớn + 6 thumbnail + mũi tên | có 1 `img` chính + 6 `img` thumbnail, đều có `src` và `srcset`; mũi tên là 2 `button` | payload prop `images` (6 phần tử) → SSR |
| "Brand: Eve" | có | SSR (payload `brand`) |
| H1 | có | SSR |
| Tag "Thread" | có | SSR; khớp `b2cContent.thuocTinh["Giao thức"]=["Thread"]` (mã map: Chưa xác định) |
| Giá "2.772.000đ" | có | SSR; payload `price:2772000`, JSON-LD `offers.price:"2772000"` |
| "(Đã bao gồm VAT)" | có | SSR (server element, row `4c`) |
| "● Đặt trước" | có | SSR (row `4c`); tương quan với `stockStatus:"make_to_order"` và JSON-LD `availability: BackOrder`; hàm map nằm ở server, không thấy trong bundle client |
| "Xuất xứ: TW" | có | SSR; payload `countryOfOrigin:"TW"` |
| "SKU: Eve Motion" | có | SSR; payload `sku:"Eve Motion"` (`mpn:null`) |
| Ô số lượng | có (`input value=1`) | SSR; thay đổi = client |
| "Mua ngay", "Thêm vào giỏ" | có (`button`) | SSR; handler = client |
| "Liên hệ tư vấn" | có (`a[href=tel:0918918755]`) | SSR; số lấy từ prop `hotline:"0918.918.755"` (khác số Zalo/footer `0378628837`) |
| Nút so sánh (icon) | có `button` (không có chữ; `aria-label` + `title` "Thêm vào so sánh") | SSR; modal = client (`CompareModal`) |
| "HOẠT ĐỘNG CÙNG" + 6 mục | có: 4 `img` + 2 chip chữ; nhãn viết thường trong HTML, hiển thị in hoa bằng CSS `uppercase` | SSR (row `3f`); danh sách khớp `b2cContent.thuocTinh["Nền tảng"]` (6 giá trị cùng thứ tự); map tên → icon không thấy trong bundle |
| "Danh mục: …" | có, 2 link | SSR (payload `categories`) |
| Nút nổi "Zalo" | có `a[data-floating-zalo]` | SSR |

## 5. Chi tiết khối đặc biệt

Gallery ảnh (đối chiếu số lượng): dữ liệu có 6 ảnh (prop `images` của component gallery, mỗi phần tử `url`, `alt`, `caption:null`, `width:1000`, `height:1000`, `isPrimary`, `position`; `alt` cả 6 = tên sản phẩm); JSON-LD `Product.image[]` có 6 ảnh; DOM có 7 `img` (1 ảnh chính cho ảnh số 1 + 6 thumbnail).

| Thuộc tính | Ảnh chính (`img` số 1) | 6 thumbnail |
|---|---|---|
| `alt` | tên sản phẩm | rỗng (`alt=""`) |
| `loading` | không có (mặc định eager) | `lazy` |
| `fetchpriority` | `high` (+ `<link rel=preload as=image>` trong head với cùng `imagesrcset`) | không |
| `width`/`height` | 720/720 | 64/64 |
| `sizes` | `(max-width: 1024px) 100vw, 680px` | `64px` |
| `srcset` | 8 candidate `w=640…3840&q=90` | 16 candidate `w=32…3840&q=75` |
| `src` | `/_next/image?url=%2Fmedia%2F{sha256}.jpg&w=3840&q=90` | `/_next/image?url=%2Fmedia%2F{sha256}.jpg&w=3840&q=75` |
| placeholder/blur | không có (không có `placeholder`, không có `data:` URI, không có `background-image`; `style="color:transparent"`) | idem |
| Nguồn ảnh thật | `/media/{sha256}.jpg` (payload) = `https://api.knxstore.vn/media/{sha256}.jpg` (JSON-LD, OG) | idem |

Ảnh chính và thumbnail 1 cùng `url=…cf9cf061…302d3c.jpg` (khác tham số `w`/`q`); thumbnail 2-6 là 5 hash khác (`6b9fda92…`, `31366d3e…`, `f0ca0b1e…`, `e2628f55…`, `917c4798…`). Thumbnail nằm trong `div.overflow-x-auto.snap-x`.

Kết quả tôi tự gọi các URL ảnh (ngoài trang, không cookie):

| URL | Kết quả |
|---|---|
| `/_next/image?url=%2Fmedia%2Fcf9cf061…jpg&w=2048&q=90` (ảnh chính) | 200, `image/jpeg`, 20,181 B, JPEG 1000×1000 |
| cùng URL `w=3840&q=90` | 200, 20,181 B (không upscale vượt 1000 px) |
| thumbnail 2 `…6b9fda92…jpg&w=128&q=75` | 200, 2,181 B, 128×128 |
| thumbnail 2 `w=3840&q=75` | 200, 48,744 B, 1000×1000 |
| thumbnail 6 `…917c4798…jpg&w=128&q=75` | 200, 2,853 B |
| `https://api.knxstore.vn/media/6b9fda92…jpg` | 200, 103,954 B |
| `https://knxstore.vn/media/ac5c389e…jpg` (ảnh trong mô tả) | 200, 43,739 B, `cache-control: public, max-age=31536000, immutable` |
| ảnh chính `w=2048&q=90` với `Accept: image/avif,image/webp,…` | 200, `image/webp`, 10,106 B |
| thumbnail 2 `w=128&q=75` với `Accept` như trên | 200, `image/webp`, 1,804 B |

Tôi mở ảnh chính và ảnh số 2 đã tải: cả hai là ảnh sản phẩm hợp lệ (ảnh 2 là infographic có chữ tiếng Việt "Hỗ trợ giao thức Matter…" và logo Matter/Apple Home/SmartThings/Alexa/Google Home nằm trong ảnh, nên chữ này không có trong DOM). Component gallery (module `65382` trong bundle) dùng `next/image` cho ảnh chính với `priority:true`, `quality:90`; thumbnail không có `priority`; state `useState(0)` chọn ảnh; không có `onLoad`, `placeholder` hay class `opacity-0` điều khiển hiển thị ảnh. Nguyên nhân ảnh lớn và 5/6 thumbnail hiển thị trống trong ảnh chụp: Chưa xác định (phía server mọi URL thử đều trả 200 và ảnh hợp lệ; cần Network/Console của đúng trình duyệt đã chụp).

Tab (SSR chỉ có nút, không có panel cho tab không chọn):

```html
<div role="tablist" aria-label="Nội dung sản phẩm" class="sticky top-[var(--site-header-h,56px)] z-30 flex …"><button type="button" role="tab" aria-selected="true" class="… text-[#1868c2]">Chi tiết sản phẩm</button><button type="button" role="tab" aria-selected="false" …>Thông số kỹ thuật</button><button type="button" role="tab" aria-selected="false" …>FAQ</button></div><div class="overflow-hidden  transition-[height] …"><div class="user-html matter-html"><h2>…
```

Quy tắc tạo tab (đọc từ bundle, component module `10030`): tab "Chi tiết sản phẩm" nếu `descriptionHtml`; "Hướng dẫn" nếu `guideHtml` (null ở đây nên không có); "Thông số kỹ thuật" nếu `specs.length>0`; "FAQ" nếu `faqs.length>0`; "Datasheet" nếu `downloads.length>0` (rỗng ở đây). Chỉ render nội dung của key đang chọn (`"description"===h && …`). Khi chọn: `specs` render `dl > div > dt + dd` (giá trị có thể kèm `unit` nếu `parseStatus=="exact"` và giá trị chưa chứa đơn vị); `faq` render `details/summary`, `details` đầu tiên `open`; `downloads` render `ul/li/a[target=_blank]`.

Bảng trong mô tả (HTML CMS, không class, `<th scope="col">`, không `rowspan`/`colspan`, ô cột đầu là `td`):

| Bảng | Heading liền trước | Cột × hàng | 3 dòng đầu |
|---|---|---|---|
| 1 | h3 "Voice Assistant" | 3 cột × 5 hàng (1 `thead` + 4 `tbody`) | `Voice Assistant / Hỗ trợ / Ghi chú`; `Siri (Apple Home) / Có / "Hey Siri, có ai trong phòng khách không?"`; `Google Assistant / Có / Qua Google Home` |
| 2 | h3 "Tính Năng Theo Nền Tảng" | 5 cột × 9 hàng (1 `thead` + 8 `tbody`) | `Tính năng / Apple Home + Eve app / Apple Home / Google Home / Alexa`; `Trạng thái chuyển động / Có / Có / Có / Có`; `Cảm biến ánh sáng / Có / Có / Có / Không` |

```html
<table><thead><tr><th scope="col">Voice Assistant</th><th scope="col">Hỗ trợ</th><th scope="col">Ghi chú</th></tr></thead><tbody><tr><td>Siri (Apple Home)</td><td>Có</td><td>"Hey Siri, có ai trong phòng khách không?"</td></tr>…
```

Mô tả (`descriptionHtml`): HTML thô của CMS (giữ `&nbsp;` thành `<p> </p>`, `style="height:537px; width:700px"` trên `img`, ảnh dùng `<img>` thường với `src=/media/….jpg`, không `loading`, không `width`/`height`); text trong DOM = text của payload (1,835 ký tự, khớp 100%); chuỗi này xuất hiện 2 lần trong payload (row `47` trong object sản phẩm và row `6b` truyền cho tab, mỗi row 4,031 B). Danh sách `ul` cuối và `ul`/`p` rỗng, `hr` ở cuối nguyên văn trong nguồn.

Khối "Hoạt động cùng" (HTML thô):

```html
<div class="mt-5 rounded-lg border …"><span class="mb-3.5 flex … uppercase …"><svg …></svg>Hoạt động cùng</span><div class="grid grid-cols-3 …"><span class="flex items-center justify-center"><img src="/images/matter-icons/apple-home.png" alt="Apple Home" width="64" height="28" class="… object-contain"/></span>…<span class="inline-flex … rounded-full border …">Homey</span><span class="…">Home Assistant</span></div></div>
```

Điều khiển mua (bundle, component `$L4f`, `layout:"retail"`):

```html
<button type="button" class="h-12 flex-1 rounded-full bg-[#1868c2] …">Mua ngay</button>
<button type="button" class="h-12 min-w-[150px] flex-1 rounded-full border …">Thêm vào giỏ</button>
<a href="tel:0918918755" class="… border-[#1868c2] …"><svg class="lucide lucide-phone …"/>Liên hệ tư vấn</a>
<button type="button" aria-haspopup="dialog" aria-label="Thêm vào so sánh" title="Thêm vào so sánh"><svg class="lucide lucide-git-compare-arrows …"/></button>
```

JSON-LD: đầu script là `<script type="application/ld+json">{"@context":"https://schema.org","@graph":[{"@type":["Organization","LocalBusiness"],"@id":"https://knxstore.vn/#organization",…` (chi tiết mục 8).

## 6. Nội dung chỉ có ở client / không thấy trong HTML ban đầu

Object sản phẩm trong payload có 44 trường (record `3c`, prop `product` của component client; serialize một lần, các component khác tham chiếu lại). Bảng đối chiếu từng trường:

| Trường | Giá trị mẫu | Trong DOM | Trong JSON-LD |
|---|---|---|---|
| `id` | 177 | không | không |
| `slug` | `cam-bien-chuyen-dong-eve-motion-matter-over-thread-ipx3` | chỉ qua URL/canonical | `url` |
| `name` | "Cảm biến chuyển động Eve Motion, Matter over Thread, IPX3" | có (H1, breadcrumb, aria-label, thanh mobile) | `Product.name` |
| `sku` | "Eve Motion" | có ("SKU: Eve Motion") | `sku`, `mpn` |
| `mpn` | null | không | `mpn` lấy từ `sku` |
| `shortDescription` | HTML: "Tương thích Apple Home, Google Home, SmartThings, Alexa, Homey & Home Assistant" + "Giao thức Thread" | KHÔNG (nguyên văn); thông tin tương đương hiện qua chip + "Hoạt động cùng" | không |
| `brand` | `{slug:eve,name:Eve}` | có | `brand`, `manufacturer` |
| `primaryImage` | `/media/cf9cf061…jpg` | có (ảnh chính) | `image[0]`, `primaryImageOfPage` |
| `images` | 6 phần tử | có (7 `img`) | `image[]` 6 |
| `isFeatured`, `status`, `labels`, `bundles`, `clearances`, `clearance*`, `priceHidden`, `ctaTitle`, `ctaUrl`, `visual` | false / `PUBLISHED` / `[]` / null | không | không |
| `b2c`, `b2b`, `quotable` | `true`, `true`, `false` | không | không |
| `descriptionHtml` | HTML 4,031 B | có (tab 1) | không (`description` JSON-LD là `seoDescription`) |
| `guideHtml` | null | không có tab "Hướng dẫn" | không |
| `seoTitle`, `seoDescription` | xem mục 2 | trong `<head>` (không trong body) | `ItemPage.name`, `description` |
| `countryOfOrigin` | "TW" | có | `countryOfOrigin` (Country) |
| `warrantyMonths` | null | không | không |
| `publishedAt` | 2026-05-26T09:01:25.000Z | không | `ItemPage.datePublished`/`dateModified` |
| `price` | 2772000 | có (2.772.000đ) | `offers.price:"2772000"` (chuỗi) |
| `oldPrice`, `clearanceFrom` | null | không có giá gạch/badge giảm | không |
| `priceRange` | `{from:2772000,to:2772000}` | không | không |
| `stockStatus` | `make_to_order` | có dạng nhãn "Đặt trước" | `availability: https://schema.org/BackOrder` |
| `variants` | `[]` | không có UI biến thể | không |
| `categories` | 2 phần tử | có (dòng "Danh mục", 2 link; breadcrumb chỉ dùng cái đầu) | `Product.category` chỉ có "Điều khiển Matter" |
| `solutions` | `[{slug:matter,name:Matter}]` | không | `subjectOf` (Service Matter) |
| `related` | `[]` | (16 card liên quan lấy từ nguồn khác) | không |
| `downloads` | `[]` | không (không có tab Datasheet) | không |
| **`specs`** | 11 dòng (`code`, `name`, `unit`, `rawName`, `valueText`, `parseStatus`, `position`, `priority`) | KHÔNG: tab "Thông số kỹ thuật" không có panel; tên/giá trị (ví dụ "Chống nước", "-18°C đến +55°C", "65 x 65 x 34 mm") không có trong text | `additionalProperty[]` 11 `PropertyValue` |
| **`faqs`** | 7 Q&A (`question`, `answer`, `position`) | KHÔNG: không có text câu hỏi/đáp trong DOM | `FAQPage` 7 Q&A |
| `b2cContent.noiDung.matter_banner` | `/media/740664bc…jpg` | KHÔNG (hash không có trong DOM ngoài script) | không |
| `b2cContent.noiDung.matter_application_1..3` | 3 ảnh + 3 mô tả HTML (733, 741, 823 ký tự text; row `48`, `49`, `4a`) | KHÔNG; không thấy nơi nào render (không có trong 7 chunk client đã đọc; chuỗi `matter_application`/`matter_banner`/`noiDung` không xuất hiện trong bundle) | không |
| `b2cContent.noiDung.matter_spec_description`, `matter_banner_description` | `<p>&nbsp;</p>…`, rỗng | không | không |
| `b2cContent.thuocTinh` / `b2cAttributes` (trùng nhau) | `Giao thức:[Thread]`, `Nền tảng:[6 giá trị]`, `Cần Hub?:[Có]` | một phần: chip "Thread" và 6 mục "Hoạt động cùng"; "Cần Hub?" KHÔNG hiện | không |

Các nội dung khác không có trong HTML ban đầu hoặc chỉ có sau client render:

| Nội dung | Bằng chứng | Nằm ở đâu |
|---|---|---|
| Panel tab "Thông số kỹ thuật" và "FAQ" | DOM chỉ có `div.user-html` của tab 1; bundle chỉ render key đang chọn | RSC + JSON-LD (xem trên); DOM = không |
| Số lượng trên badge giỏ ("1" trong ảnh chụp) | HTML: `aria-label="Giỏ báo giá, 0 sản phẩm"`, không có badge | client (state giỏ) |
| Lightbox phóng to ảnh, cart modal, compare modal, hộp gợi ý tìm kiếm, mở mega menu | không có trong HTML; bundle có các component | client |
| Thanh mua mobile hiện ra khi cuộn | có trong HTML nhưng `inert` + `translate-y-full`; hiện theo `IntersectionObserver` | client |
| Ảnh trong tab FAQ/spec | không áp dụng | – |
| Tab "Hướng dẫn", "Datasheet" | `guideHtml:null`, `downloads:[]` nên không tạo tab | không có ở đâu |
| Biến thể/tuỳ chọn (màu, phiên bản) | `variants:[]` | không có ở đâu |
| Rác trong payload (không có trong DOM): shell B2B | row `11` (`notFound` của layout `(site)`, tham chiếu bởi `notFound:["$L11",[]]` trong row `4`) chứa menu B2B (Trending + 5 menu + `footerColumns`, hotline) và 404 UI B2B (row `15`, `17`, `1d`…`2b`); đóng bao tham chiếu `$L…` từ row `11` gồm 20 row JSON ≈ 55,046 B (≈31% payload 177,077 B) | RSC |

## 7. Liên kết

Tổng 87 thẻ `a` (đều có `href`), phân bố:

| Vùng | Số link | Mẫu URL |
|---|---|---|
| Header | 17 | `/{slug}` (danh mục: `/cong-tac-matter`, `/cam-bien-matter`, `/camera-thong-minh`, `/dong-co-rem-matter`, `/o-cam-thong-minh-matter`, `/dieu-khien-dan-lanh`, `/driver-matter`, `/gateway-matter`, `/hub-trung-tam`, `/hub-trung-tam-homey`, `/khoa-thong-minh-matter`, `/chuong-hinh`), `/blogs`, `/contact`, `/matter` (logo), `/` (Trang B2B), `/cart` |
| Menu mobile | 15 | tương đương header (cùng 12 link danh mục, `/blogs`, `/contact`, `/` "Trang B2B"; không có logo `/matter` và `/cart`) |
| Breadcrumb | 2 | `/matter`, `/san-pham-matter` |
| Khối sản phẩm | 4 | `/brands/eve`, `tel:0918918755`, `/san-pham-matter`, `/cam-bien-matter` |
| Mô tả (tab 1) | 0 | không có link trong mô tả |
| Sản phẩm liên quan | 16 | `/products/{slug}` (16 slug khác nhau) |
| Bài viết liên quan | 9 | `/{slug}.html` (3 bài × 3 link: ảnh, tiêu đề, "Đọc tiếp") |
| Drawer báo giá | 2 | `/products`, `/cart` |
| Footer | 21 | 11 `/{slug}`, 3 `/{slug}.html` (chính sách), `/` , `tel:0378628837`, `mailto:support@knxstore.vn`, `https://zalo.me/0378628837`, `https://www.facebook.com/knxstore.vn`, `https://online.gov.vn/nen-tang/…`, `https://www.dmca.com/Protection/Status.aspx?ID=…` |
| Nút Zalo nổi | 1 | `https://zalo.me/0378628837` |

Vùng nội dung chính (khối sản phẩm + tab/mô tả + liên quan + bài viết): 4 + 0 + 16 + 9 + 2 (breadcrumb) = 31 link. Link ngoài: 4 domain (`zalo.me`, `facebook.com`, `online.gov.vn`, `dmca.com`). Link tải file: không có. Link `rel=alternate`/markdown: không có. Link tới trang B2B: `/` (Trang B2B ×2 trong header và menu mobile; footer "Thiết bị tự động hoá toà nhà"), `/products` (drawer). Link chỉ tồn tại trong JSON-LD, không có trong DOM: `https://knxstore.vn/solution/matter` (`subjectOf`), URL ảnh dạng `https://api.knxstore.vn/media/…`.

## 8. JSON-LD

1 `<script type="application/ld+json">`, 11,834 B (nội dung 11,790 B), vị trí: body, node con của `div.font-sans` ngay sau mobile panel, trước `section` nội dung; lặp lại trong RSC (row `6c`, kiểu `T`). Không có JSON-LD trong `<head>`. `@graph` gồm 4 node:

| Node | Trường chính có mặt | Thiếu/ghi chú |
|---|---|---|
| `Organization`+`LocalBusiness` (`#organization`) | name, legalName, url, foundingDate, taxID, logo, image, description, telephone `+84918918755`, email, address, geo, hasMap, openingHoursSpecification, priceRange, areaServed, knowsAbout[10], contactPoint, sameAs[2] | giống node tổ chức của trang B2B |
| `ItemPage` (`#webpage`) | url, name (= `seoTitle`), description (= `seoDescription`), inLanguage `vi-VN`, datePublished, dateModified, isPartOf, publisher, mainEntity → `#product`, primaryImageOfPage, breadcrumb (BreadcrumbList 3 mục: `/matter`, `/san-pham-matter`, SP) | `datePublished` = `dateModified` = `publishedAt` |
| `Product` (`#product`) | name, sku `Eve Motion`, mpn `Eve Motion`, description, url, image[6] ImageObject, brand (Brand + url), manufacturer, category `Điều khiển Matter`, additionalProperty[11], countryOfOrigin `TW`, offers, subjectOf (Service Matter) | không có `gtin*`, `aggregateRating`, `review`, `priceValidUntil`, `shippingDetails`, `weight`/`color`; `additionalProperty` dòng "Nguồn cấp" (`2 x AAA`) có `unitText:"V"` |
| `FAQPage` (`#faq`) | mainEntity: 7 `Question` + `acceptedAnswer.text` | text câu hỏi/đáp khớp payload, không có trong DOM |

`Offer`: `priceCurrency:"VND"`, `price:"2772000"` (chuỗi), `availability:"https://schema.org/BackOrder"`, `itemCondition: NewCondition`, `url`, `seller` (tham chiếu tổ chức), `areaServed` VN, `hasMerchantReturnPolicy` (30 ngày, `FreeReturn` cho lỗi sản phẩm, phí vận chuyển cho đổi ý, link chính sách).

Đối chiếu với DOM: tên khớp H1; SKU khớp "SKU: Eve Motion"; giá `2772000` khớp `2.772.000đ`; `BackOrder` khớp nhãn "Đặt trước"; xuất xứ `TW` khớp; 6 ảnh khớp 6 ảnh gallery (URL dạng `api.knxstore.vn/media/…`, khác dạng `/_next/image?…` trong DOM); `category` chỉ 1 trong 2 danh mục hiển thị; breadcrumb JSON-LD khớp breadcrumb DOM (chỉ dùng danh mục đầu). 11 dòng thông số và 7 FAQ chỉ khớp payload, không có trong DOM.

## 9. Số đo

| Chỉ số | Giá trị |
|---|---|
| HTML thô (identity) | 354,232 B (≈88,558 token theo bytes/4) |
| gzip / br | 57,794 B / 45,834 B |
| `<head>` | 5,756 B |
| Tổng `<script>` | 20 thẻ, 213,694 B (60.3% HTML thô) |
| — `self.__next_f` | 4 thẻ: init 60 B + 3 push 74,646 B / 76,686 B / 49,229 B = 200,621 B |
| — `ld+json` | 1 thẻ, 11,834 B |
| — script ngoài `src` | 14 thẻ (tag 1,122 B) |
| — script inline khác | 1 (cookie `knx-area`, 117 B) |
| HTML không tính script | 140,538 B (39.7%); `<style>`: 0 |
| Payload RSC đã giải mã | 177,077 B, 121 row (79 JSON, 25 import `I`, 11 hint `HL`, 6 text `T`) |
| Text trong DOM (bỏ script/style/template; gồm cả khối ẩn) | 5,227 ký tự (6,167 B ≈ 1,541 token) |
| Text vùng nội dung (bỏ header, menu mobile, drawer, footer, Zalo) | 4,010 ký tự |
| `a` / `img` / `table` / `button` | 87 / 36 / 2 / 40 |
| `h1` / `h2` / `h3` / `h4–h6` | 1 / 9 / 5 / 0 |
| `form` / `input` / `nav` / `aside` / `section` / `article` | 1 / 2 / 3 / 1 / 4 / 19 (16 card + 3 bài) |
| `ul` / `li` / `details` / `svg` / `main` | 9 / 31 / 3 (menu mobile) / 40 / 0 |
| `link` / `meta` | 19 / 16 |
| Header (HTML) | 9,358 B, 261 ký tự, 17 link |
| Menu mobile (`div.fixed.inset-0.z-[10000]`) | 6,571 B, 260 ký tự, 15 link |
| Footer | 7,929 B, 597 ký tự, 21 link |
| Drawer báo giá + nút Zalo | 2,585 B + 649 B |
| Khung (header + menu mobile + footer + drawer + Zalo) | 27,092 B (7.6% HTML thô) |
| Khối sản phẩm (`section.relative`) | 27,358 B, 423 ký tự |
| Tab + mô tả | 5,217 B, 1,872 ký tự |
| Sản phẩm liên quan | 65,641 B (18.5% HTML thô), 1,428 ký tự, 16 card ≈ 4.1 KB/card |
| Bài viết liên quan | 10,227 B, 286 ký tự |

## 10. Khác biệt so với skin B2B / Chưa xác định

So sánh có kiểm chứng với trang B2B `https://knxstore.vn/products/sunricher-phim-bam-gan-tuong-dali-2-8-phim-sr-2422nk8-da2` (tải lại hôm nay, 447,102 B):

| Hạng mục | Matter (Eve Motion) | B2B (SR-2422NK8-DA2) |
|---|---|---|
| Header HTTP/render | ISR, `x-nextjs-cache: HIT`, `s-maxage=3600` | cùng giá trị |
| `<html>`, `<body>` | `lang=vi`, 3 class font, `<body>` không attribute | giống hệt |
| Dấu hiệu skin | `div.font-sans[style="--font-body:var(--font-matter)"]` bọc cả trang; script cookie `knx-area=matter` là con của wrapper; props `skin:"matter"` (gallery, giá, SKU, compare), `variant:"matter"` (tab), `layout:"retail"`; cờ sản phẩm `b2c:true,b2b:true`, `quotable:false` | không có wrapper; script cookie `knx-area=b2b` ở đầu `<body>`; props `skin:"b2b"`; `b2c:false,b2b:true`, `quotable:true` |
| Skip link | không có | `a[href=#main-content]` "Bỏ qua, tới nội dung chính" |
| Wrapper nội dung | không có `#main-content`, không `<main>` | `div#main-content[tabindex=-1]` (90,725 B), không `<main>` |
| Header | `header[aria-label="Khu Matter Smarthome"]`, 9,358 B, 17 link, 1 `img`, 0 heading | `header.sticky.top-0.z-9999`, 128,755 B, 165 link, 67 `img`, 18 `h3` (2 rỗng), 6 panel `hidden` + `#b2b-trending-panel` |
| Menu mobile | `aside#matter-mobile-panel` (6,571 B) | không có `aside` trong trang |
| Breadcrumb | `nav[aria-label="Đường dẫn"] > ol > li`, gốc "Trang chủ" → `/matter` | `nav[aria-label=Breadcrumb]` với `span`, gốc → `/` |
| Footer | 7,929 B, 21 link, không có `nav`, 1 bản | 17,608 B, 50 link, 3 `nav[aria-label]`, mỗi cột render 2 lần |
| Nút Zalo nổi | `zalo.me/0378628837` | `zalo.me/0918918755` |
| Drawer báo giá | `div[inert].fixed.z-99999` h2 "Yêu cầu báo giá", 2,585 B (có cả ở Matter; link "Xem sản phẩm" → `/products`) | giống hệt (2,585 B) |
| Gallery | 6 ảnh, thumbnail 64 px, `alt` thumbnail rỗng | 2 ảnh, thumbnail 72 px |
| Giá và CTA | `2.772.000đ`, "(Đã bao gồm VAT)", "Đặt trước"; "Mua ngay", "Thêm vào giỏ", "Liên hệ tư vấn", so sánh | không có giá (`price:null`); "Thêm vào yêu cầu báo giá", `tel:0918918755`, so sánh |
| Biến thể | không có (`variants:[]`) | có 2 nút màu ("Xám", "Trắng") |
| Tab | 3 tab: Chi tiết sản phẩm / Thông số kỹ thuật / FAQ | 1 tab: "Mô tả chi tiết" (không có `specs`, `faqs`) |
| Bảng trong DOM | 2 `table` (trong mô tả) | 0 |
| Khối "Hoạt động cùng", chip giao thức | có | không |
| Sản phẩm liên quan | h2 "Sản phẩm liên quan", 16 card | h2 "Sản phẩm đề xuất", 16 card |
| Bài viết liên quan | có (3 bài) | không |
| JSON-LD | 4 node (Organization, ItemPage, Product, FAQPage), 11,834 B, con của wrapper, sau header | 2 node (Organization, ItemPage, không `mainEntity`/`Product`), 3,960 B, nằm trong `#main-content` |
| `<head>` | thêm 4 `preload as=image` PNG `/images/matter-icons/*` | không; còn lại cùng bộ link/meta, cùng 2 file CSS |
| Payload RSC | 177,077 B, 3 chunk `push`, wrapper skin ở row `5` (4,848 B) | 182,689 B, 7 chunk `push`; wrapper skin B2B ở row `5` (29,467 B, chứa dữ liệu nav B2B) |

Chưa xác định:

| Điểm | Lý do |
|---|---|
| Nguyên nhân ảnh chính và 5/6 thumbnail hiển thị trống trong ảnh chụp | không có bằng chứng phía server: mọi URL ảnh thử đều 200 và là ảnh hợp lệ; không chạy được trình duyệt; cần Network/Console của trình duyệt đã chụp |
| Điều kiện chọn skin Matter hay B2B cho `/products/{slug}` | chỉ có 2 mẫu; mẫu Matter có `b2c:true` và danh mục thuộc "Điều khiển Matter", mẫu B2B có `b2c:false`; wrapper nằm ở layout segment `[slug]` (suy ra từ cây route), nhưng quy tắc chọn không nhìn thấy; chưa gửi request có cookie `knx-area` để xem cookie có đổi SSR không (theo yêu cầu không gửi cookie) |
| Nơi render `b2cContent.noiDung` (`matter_banner`, `matter_application_1..3`) và `shortDescription` | không có trong DOM mẫu, không có trong 7 chunk client đã đọc; có thể thuộc component server hoặc chưa dùng |
| Hàm map `stockStatus` → nhãn ("Đặt trước"), `thuocTinh` → chip/icon | chạy ở server (row `4c`, `3a`, `3f`), không có trong bundle client; chỉ biết tương quan giá trị |
| Nguồn của 16 card "Sản phẩm liên quan" | `related:[]` trong object sản phẩm |
| DOM thật của tab Thông số/FAQ/Datasheet/Hướng dẫn | tab không chọn không có trong HTML; mô tả cách render chỉ suy ra từ bundle (`dl/dt/dd`, `details/summary`, `ul/li/a`) |
| Lưu trữ state giỏ hàng (nguồn số "1" trên badge) | bundle có `localStorage.getItem` nhưng chưa xác định khoá và component dùng |
| Cookie banner, popup, widget chat khác | không thấy trong HTML ban đầu; client có thể chèn sau |
| Mẫu 1 URL có thể không đại diện | 752 URL `/products/{slug}` gồm cả B2B và Matter; chưa khảo sát sản phẩm Matter có `variants`, `downloads`, `guideHtml`, `oldPrice`, `quotable:true`, nhiều `specs`, hoặc `stockStatus` khác (`in_stock`, `low_stock`, `backorder`, `out_of_stock` đã thấy trong payload của card nhưng chưa thấy trên trang chi tiết) |
| Phiên bản Next.js chính xác | không có header/meta nêu version |
