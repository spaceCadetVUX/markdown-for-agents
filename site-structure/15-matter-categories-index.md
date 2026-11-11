# Trang chỉ mục danh mục skin Matter — cấu trúc hiện tại

> https://knxstore.vn/matter/categories · pages.xml (19 URL; nhóm Matter có 3 URL: `/matter`, `/matter/categories`, `/matter-search`; mục `/matter/categories` có `changefreq=weekly`, không có `lastmod`) · khảo sát 2026-11-11

Phương pháp: `curl` HTML thô (UA `Mozilla/5.0`, không cookie), phân tích bằng `html.parser`, giải mã payload `self.__next_f` bằng parser row của Flight. Số request vào chính URL này: 5 (identity, gzip, br, `Accept: text/markdown`, `.md`). Khảo sát gọn hơn hai file kia; không đọc bundle JS. Chưa chạy trình duyệt.

## 1. Response & rendering

| Header | Giá trị |
|---|---|
| status | 200 (HTTP/2) |
| content-type | `text/html; charset=utf-8` |
| cache-control | `s-maxage=3600, stale-while-revalidate=31532400` |
| x-nextjs-cache | `HIT` (cả 4 request GET 200); `.md` (404): không có |
| x-nextjs-prerender | `1` |
| x-nextjs-stale-time | `300` |
| vary | `rsc, next-router-state-tree, next-router-prefetch, next-router-segment-prefetch, Accept-Encoding` |
| cf-cache-status | `DYNAMIC` |
| link | không có |
| x-robots-tag | không có |
| content-encoding | không có khi không gửi `Accept-Encoding`; `gzip` / `br` khi gửi tương ứng |
| set-cookie | không có |
| Header khác | `server: cloudflare`, `speculation-rules`, `strict-transport-security`, `x-frame-options: SAMEORIGIN`, `x-content-type-options: nosniff`, `referrer-policy: same-origin`, `alt-svc: h3`, `nel`/`report-to` |

Kích thước trên đường truyền: identity 173,949 B; gzip 30,849 B; br 26,021 B.

Chế độ render: ISR (prerender tĩnh + revalidate). Bằng chứng: `x-nextjs-prerender: 1`, `x-nextjs-cache: HIT`, `s-maxage=3600` + `stale-while-revalidate=31532400`, không có header `link`, record `0` có `"S":true` (chỉ ghi nhận tương quan).

Route (record `0`): `(site)/matter/categories/__PAGE__` (segment tĩnh `matter` rồi `categories`, không phải `[slug]`). Khác hai trang Matter còn lại: wrapper skin `div.font-sans` (record `5`) nằm ở chính node `__PAGE__` trong cây Flight (cạnh `OutletBoundary`), tức do page component render cả header + `main` + footer; trên trang sản phẩm và danh mục wrapper nằm ở node layout của segment `[slug]` (kết luận suy ra từ vị trí `$L5` trong record `0`). Phiên bản Next.js chính xác: Chưa xác định.

Hành vi hiện tại với yêu cầu markdown (chỉ ghi kết quả):

| Request | Status | content-type | Ghi chú |
|---|---|---|---|
| `Accept: text/markdown` | 200 | `text/html; charset=utf-8` | body 173,949 B, giống hệt bản không có `Accept`; `vary` không chứa `Accept` |
| URL + `.md` | 404 | `text/html; charset=utf-8` | body 131,143 B (trang lỗi Next), `cache-control: private, no-cache, no-store` |

## 2. `<head>`

Head dài 3,994 B.

| Thành phần | Giá trị |
|---|---|
| `<html>`, `<body>` | `lang="vi"`, 3 class biến font; `<body>` không attribute |
| title | `Tất cả danh mục thiết bị Matter \| knxstore.vn` (45 ký tự; hậu tố `knxstore.vn` viết thường, khác `KNXStore.vn` của trang sản phẩm và `KNXStore` của trang danh mục) |
| meta description | `Toàn bộ danh mục thiết bị nhà thông minh chuẩn Matter tại KNXStore: công tắc, cảm biến, đèn, khoá, điều khiển và hơn thế nữa.` (125 ký tự) |
| meta keywords / author | không có / không có |
| canonical | `https://knxstore.vn/matter/categories` |
| robots | không có thẻ `meta name=robots` |
| OG (7) | `og:title` = `Tất cả danh mục thiết bị Matter` (không có hậu tố), `og:description`, `og:url`, `og:site_name=knxstore.vn`, `og:locale=vi_VN`, `og:image=https://knxstore.vn/images/og/knxstore-og.png`, `og:type=website` |
| Twitter (4) | `summary_large_image`, title, description, image (= `og:image`) |
| hreflang / `rel=alternate` | không có / không có |
| Preload | 4 font woff2; logo SVG; 1 script chunk (`fetchpriority=low`); `gtag/js?id=G-4D9XLJDVGG` (chỉ preload); không có preload ảnh tile |
| Stylesheet | 2 (cùng 2 file với trang sản phẩm/danh mục) |
| Khác | `theme-color #14467C`, `manifest`, icon, `next-size-adjust` |
| JSON-LD | không có |

## 3. Khung trang

Header, menu mobile, footer, Zalo, drawer báo giá giống hệt hai trang Matter còn lại về cấu trúc và kích thước (header 9,358 B / 17 link; menu mobile 6,571 B / 15 link; footer 7,929 B / 21 link; drawer báo giá 2,585 B; Zalo 649 B); chi tiết nội dung từng phần xem mục 3 của `13-matter-product.md` (đã đo lại trên trang này, số liệu trùng).

Landmark thực tế:

| Phần tử | Vai trò |
|---|---|
| `div[hidden]` (con đầu `<body>`) | boundary rỗng của Next |
| `div.font-sans[style="--font-body:var(--font-matter)"]` | wrapper skin |
| `header[aria-label="Khu Matter Smarthome"]`, `nav[aria-label="Menu Matter"]`, `form[role=search]` (trong `#matter-search-drop`), `aside#matter-mobile-panel` + `nav[aria-label="Menu Matter trên điện thoại"]` | header, menu, tìm kiếm, menu mobile |
| **`main.pt-8`** | landmark `main` duy nhất trong ba trang Matter đã khảo sát; bọc breadcrumb và `section.py-10` |
| `nav[aria-label="Đường dẫn"]` | breadcrumb (con đầu của `main`) |
| `footer.mt-auto` | footer |

Không có `#main-content`, không có skip link, không có `role=main` tường minh. Thành phần nổi: nút Zalo `a[data-floating-zalo][href=https://zalo.me/0378628837]` (hiện), drawer báo giá `div[inert].fixed.z-99999` (ẩn), menu mobile (ẩn). Không thấy cookie banner/popup/back-to-top trong HTML ban đầu.

## 4. Cây nội dung chính (theo thứ tự DOM)

Cột "Trong main": có nằm trong `main.pt-8` hay không.

| # | Thẻ + selector rút gọn | Heading | Nội dung tóm tắt | Trong main | Hiển thị mặc định | SSR / client |
|---|---|---|---|---|---|---|
| 0 | `div[hidden]` | – | boundary rỗng | không | ẩn | SSR |
| 1 | `div.font-sans` | – | wrapper skin; con đầu là `script` cookie `knx-area=matter` | không | – | SSR |
| 2 | `header[aria-label="Khu Matter Smarthome"]` | – | menu, tìm kiếm, giỏ, "Trang B2B" | không | hiện | SSR |
| 3 | `div.fixed.inset-0.z-[10000] > aside#matter-mobile-panel` | – | menu mobile | không | ẩn | SSR |
| 4 | `main.pt-8` (38,046 B, 420 ký tự text) | – | bọc 4a và 4b | – | hiện | SSR |
| 4a | `div.mx-auto.max-w-[1280px] > nav[aria-label="Đường dẫn"] > ol > li` | – | Trang chủ (`/matter`) › `span[aria-current=page]` "Tất cả danh mục" | có | hiện | SSR |
| 4b | `section.py-10 > div.mx-auto.max-w-[1280px].border-b` | h1: "Tất cả danh mục (12)" | tiêu đề (số 12 gắn trong H1) + lưới tile (36,951 B cả section) | có | hiện | SSR |
| 4c | `div.grid.grid-cols-2.gap-x-4.gap-y-5.p-1.md:grid-cols-4.md:gap-x-6` | – | 12 `a.group.flex.flex-col` (tile), không có heading con | có | hiện | SSR |
| 5 | `div.fixed.top-0.left-0.z-99999[inert]` | h2 "Yêu cầu báo giá" | drawer báo giá rỗng | không | ẩn | SSR |
| 6 | `footer.mt-auto` | 3 h2 | xem `13-matter-product.md` mục 3 | không | hiện | SSR |
| 7 | `a[data-floating-zalo]` | – | "Zalo" | không | hiện | SSR |
| 8 | `script#_R_` + script init + script `self.__next_f.push` (104,411 B) | – | payload RSC | – | – | – |

Cấu trúc 1 tile (`a.group.flex.w-full.flex-col.items-center.gap-3.rounded-2xl`; ≈3.05 KB/tile, 3,031–3,082 B): `span.h-32.w-32.md:h-40.md:w-40 > img[alt=""][loading=lazy][width=160][height=160][data-nimg=1]` (`sizes="(max-width: 767px) 128px, 160px"`, `srcset` nhiều candidate, `src` `/_next/image?url=%2Fmedia%2F{sha256}.(jpg|webp)&w=3840&q=75`) → `span.text-[15px].font-semibold` tên danh mục → `span.opacity-0` "Xem sản phẩm" + `svg` (chữ có trong DOM, chỉ hiện khi hover bằng CSS). Tile không có mô tả, không có số sản phẩm, không có badge, không có `h2`/`h3`. Không có nhóm/phân loại: một lưới phẳng 12 mục.

Danh sách 12 tile theo thứ tự DOM (đối chiếu với nhãn trong header và footer cùng trang):

| # | URL (`href`) | Nhãn tile | Nhãn trong menu header | Nhãn trong footer | Ảnh tile |
|---|---|---|---|---|---|
| 1 | `/cong-tac-matter` | Công tắc Matter | Công tắc | `Côn tắc thông minh` | `.jpg` (cùng hash `120ba5fe…` với ảnh sản phẩm đầu tiên của danh mục này) |
| 2 | `/dieu-khien-dan-lanh` | Điều khiển điều hoà máy lạnh | Thermostat | `thermostat` | `.webp` |
| 3 | `/cam-bien-matter` | Cảm biến thông minh Matter | Cảm biến | Cảm biến Matter | `.webp` |
| 4 | `/hub-trung-tam` | Hub trung tâm | Hub trung tâm (nhóm) | Hub trung tâm | `.jpg` |
| 5 | `/gateway-matter` | Gateway Matter | Gateway Matter | không có | `.jpg` |
| 6 | `/driver-matter` | Bộ điều khiển LED Matter | Driver Matter | Nguồn đèn Matter | `.jpg` |
| 7 | `/camera-thong-minh` | Camera thông minh | Camera | Camera | `.jpg` |
| 8 | `/khoa-thong-minh-matter` | Khóa thông minh Matter | Khóa cửa (trong nhóm "An ninh Matter") | không có | `.jpg` |
| 9 | `/dong-co-rem-matter` | Động cơ rèm Matter | Động cơ rèm | Động cơ rèm | `.jpg` |
| 10 | `/den-thong-minh-matter` | Đèn thông minh Matter | không có | không có | `.webp` |
| 11 | `/chuong-hinh` | Chuông hình | Chuông hình (trong nhóm "An ninh Matter") | không có | `.jpg` |
| 12 | `/o-cam-thong-minh-matter` | Ổ cắm thông minh Matter | Ổ cắm | không có | `.jpg` |

Cả 12 `href` đều có trong `categories.xml`. Danh mục cha `/san-pham-matter` ("Điều khiển Matter", là mục giữa của breadcrumb ở hai trang Matter kia) không có trong lưới này, và `/hub-trung-tam-homey` (link con trong menu header) cũng không. Thứ tự tile không theo bảng chữ cái và không trùng thứ tự menu header.

Phân trang: không có (không có link `?page=`, không có `rel=next`); toàn bộ 12 mục có trong HTML.

## 5. Chi tiết khối đặc biệt

Tile (HTML thô, rút gọn `srcset`):

```html
<a class="group flex w-full flex-col items-center gap-3 rounded-2xl border border-white bg-white px-4 py-5 text-center …" href="/cong-tac-matter"><span class="flex h-32 w-32 items-center justify-center overflow-hidden md:h-40 md:w-40"><img alt="" loading="lazy" width="160" height="160" decoding="async" data-nimg="1" class="h-32 w-32 object-contain p-1 md:h-40 md:w-40" style="color:transparent" sizes="(max-width: 767px) 128px, 160px" srcSet="/_next/image?url=%2Fmedia%2F120ba5fe…" src="/_next/image?url=%2Fmedia%2F120ba5fe….jpg&amp;w=3840&amp;q=75"/></span><span class="text-[15px] font-semibold text-[#222934]">Công tắc Matter</span><span class="flex translate-y-1 items-center gap-1 text-[12px] font-medium text-[#1868c2] opacity-0 …">Xem sản phẩm<svg …/></span></a>
```

Khung `main` và tiêu đề:

```html
<main class="pt-8"><div class="mx-auto w-full max-w-[1280px] px-4 sm:px-6 xl:px-0"><nav aria-label="Đường dẫn" class=""><ol class="flex flex-wrap items-center text-[13px]"><li …><a … href="/matter">Trang chủ</a></li><li …><span aria-hidden="true" …><svg …/></span><span aria-current="page" class="… truncate …">Tất cả danh mục</span></li></ol></nav></div><section class="py-10"><div class="mx-auto w-full max-w-[1280px] border-b … pb-15 …"><div class="mb-7"><h1 class="text-[20px] font-semibold … xl:text-[28px]">Tất cả danh mục (12)</h1></div><div class="grid grid-cols-2 gap-x-4 gap-y-5 p-1 md:grid-cols-4 md:gap-x-6"><a class="group flex …
```

Dữ liệu trong payload của tile (row `5` chứa tile đầu, row `2c`…`36` chứa 11 tile còn lại; row `4` có `notFound:["$L10",[]]`): chỉ có `href`, `className`, `src` ảnh (`/media/{sha256}.jpg|webp`), `alt:""`, `width/height:160`, tên, chữ "Xem sản phẩm"; không có trường số sản phẩm, mô tả, slug riêng ngoài `key`.

## 6. Nội dung chỉ có ở client / không thấy trong HTML ban đầu

| Nội dung | Bằng chứng | Nằm ở đâu |
|---|---|---|
| Số trên badge giỏ | HTML: `aria-label="Giỏ báo giá, 0 sản phẩm"`, không có badge | client |
| Mở mega menu, tìm kiếm, menu mobile, drawer báo giá | `aria-hidden=true`/`inert`/`invisible` trong HTML | client |
| Hiệu ứng hover "Xem sản phẩm" | chữ có trong DOM, `opacity-0` bằng CSS | CSS (không cần JS) |
| Số lượng sản phẩm theo danh mục, mô tả danh mục | không có trong HTML, không có trong payload của tile | không có ở đâu |
| Toàn bộ nội dung chính (breadcrumb, H1, 12 tile) | có trong HTML ban đầu | SSR; không có nội dung nào của trang này chỉ nằm ở client hay chỉ trong payload |
| Rác trong payload: shell B2B | row `10` (`notFound` của layout `(site)`, tham chiếu bởi `notFound:["$L10",[]]`) chứa menu B2B + footer B2B + 404 UI; đóng bao tham chiếu `$L…` gồm 20 row JSON ≈ 55,046 B (≈58% payload 94,614 B) | RSC |

Placeholder/skeleton: không có `template[data-dgst]`, không có `BAILOUT_TO_CLIENT_SIDE_RENDERING`, không có `animate-pulse`/skeleton, không có `div[hidden]` Suspense streamed (khác `/cong-tac-matter`); chỉ có `div[hidden]` rỗng đầu `<body>`.

## 7. Liên kết

Tổng 69 thẻ `a`, đều có `href`:

| Vùng | Số link | Mẫu URL |
|---|---|---|
| Header | 17 | 12 `/{slug}` danh mục, `/blogs`, `/contact`, `/matter`, `/cart`, `/` (Trang B2B) |
| Menu mobile | 15 | tương đương header (không có logo `/matter` và `/cart`) |
| Breadcrumb | 1 | `/matter` |
| Lưới tile | 12 | `/{slug}` danh mục (12 slug khác nhau, đều ở root, không đuôi `.html`) |
| Drawer báo giá | 2 | `/products`, `/cart` |
| Footer | 21 | 11 `/{slug}`, 3 `/{slug}.html`, `/`, `tel:`, `mailto:`, `zalo.me`, `facebook.com`, `online.gov.vn`, `dmca.com` |
| Nút Zalo nổi | 1 | `https://zalo.me/0378628837` |

Vùng nội dung chính (`main`): 13 link (1 breadcrumb + 12 tile). Link sản phẩm (`/products/…`), brand, bài viết, file tải, `rel=alternate`/markdown: không có. Link tới trang B2B: `/` (Trang B2B; footer "Thiết bị tự động hoá toà nhà"), `/products` (drawer).

## 8. JSON-LD

Không có. Không có thẻ `<script type="application/ld+json">` nào trong HTML (chuỗi `ld+json` không xuất hiện cả trong HTML lẫn payload RSC). Không có `BreadcrumbList`, `ItemList` hay `CollectionPage` cho trang này (so với `/cong-tac-matter` có 5 node và trang sản phẩm có 4 node).

## 9. Số đo

| Chỉ số | Giá trị |
|---|---|
| HTML thô (identity) | 173,949 B (≈43,487 token theo bytes/4) |
| gzip / br | 30,849 B / 26,021 B |
| `<head>` | 3,994 B |
| Tổng `<script>` | 16 thẻ, 105,625 B (60.7%) |
| — `self.__next_f` | 2 thẻ: init 60 B + 1 push 104,411 B = 104,471 B |
| — script ngoài `src` | 13 thẻ (tag 1,037 B) |
| — script inline khác | 1 (cookie `knx-area`, 117 B); không có `ld+json` |
| HTML không tính script | 68,324 B (39.3%); `<style>`: 0 |
| Payload RSC đã giải mã | 94,614 B, 74 row (51 JSON, 16 import, 7 hint, 0 text `T`) |
| Text trong DOM (gồm khối ẩn) | 1,637 ký tự (1,980 B ≈ 495 token) |
| Text trong `main` | 420 ký tự |
| `a` / `img` / `table` / `button` | 69 / 16 / 0 / 6 |
| `h1` / `h2` / `h3` / `h4–h6` | 1 / 4 (drawer 1 + footer 3) / 0 / 0 |
| `form` / `input` / `nav` / `aside` / `section` / `article` / `main` | 1 / 1 / 3 / 1 / 1 / 0 / 1 |
| `ul` / `li` / `details` / `svg` | 7 / 26 / 3 (menu mobile) / 29 |
| `link` / `meta` | 14 / 16 |
| Header / menu mobile / footer | 9,358 B / 6,571 B / 7,929 B |
| Drawer báo giá / Zalo | 2,585 B / 649 B |
| Khung (header + menu mobile + footer + drawer + Zalo) | 27,092 B (15.6% HTML thô) |
| `main.pt-8` | 38,046 B (21.9%), 420 ký tự, 13 link, 12 `img` |

## 10. Khác biệt so với skin B2B / Chưa xác định

Trang này không có bản tương ứng đã xác định ở skin B2B (ứng viên: `/products`, chưa tải), nên chỉ so khung trang và các điểm đã kiểm chứng với trang B2B đã tải cho file 13/14:

| Hạng mục | Trang chỉ mục Matter | Trang B2B đã tải (sản phẩm/danh mục) |
|---|---|---|
| Wrapper skin | `div.font-sans[style="--font-body:var(--font-matter)"]` + script `knx-area=matter` | không có wrapper; script `knx-area=b2b` ở đầu `<body>` |
| Skip link / `#main-content` | không có | có |
| `<main>` | có `main.pt-8` | không có |
| Vị trí wrapper skin trong cây Flight | node `__PAGE__` (page component tự render header + main + footer) | node layout `[slug]` (với trang sản phẩm B2B và Matter, và danh mục) |
| Header / footer | 9,358 B / 17 link; 7,929 B / 21 link, không `nav` | 128,755 B / 165 link; 17,608 B / 50 link, 3 `nav` |
| Nút Zalo | `zalo.me/0378628837` | `zalo.me/0918918755` |
| Breadcrumb | `nav[aria-label="Đường dẫn"] > ol > li`, gốc `/matter` | `nav[aria-label=Breadcrumb]` (span), gốc `/` |
| JSON-LD | không có | trang sản phẩm B2B có 2 node, danh mục B2B có `CollectionPage` + `BreadcrumbList`… |
| Render | ISR (HIT) | sản phẩm B2B: ISR; danh mục B2B: dynamic |
| Meta | có `og:site_name`, `og:locale`; `og:image` là ảnh chung `knxstore-og.png`; không có `keywords`/`author` | (trang sản phẩm/danh mục Matter không có `og:site_name`/`og:locale`; có `keywords`/`author`) |

Chưa xác định:

| Điểm | Lý do |
|---|---|
| Trang tương ứng ở skin B2B | chưa tải `/products` hay trang chỉ mục tương tự; không xác định được bản B2B của trang này |
| Quy tắc chọn skin cho route `/matter/*` | route nằm dưới segment tĩnh `matter`, wrapper do page tự render; chưa gửi request có cookie `knx-area`; chưa khảo sát `/matter-search` |
| Nguồn dữ liệu 12 tile (danh sách + ảnh) và quy tắc sắp xếp | payload chỉ chứa phần đã render; không thấy truy vấn hay cờ lọc "Matter" |
| Lý do `/den-thong-minh-matter` có trong lưới nhưng không có trong header/footer; `/san-pham-matter` không có trong lưới | chỉ ghi nhận sự khác biệt giữa các nhãn/menu; chưa fetch hai danh mục này |
| Nhãn không thống nhất giữa tile, menu header, footer (ví dụ "Thermostat" vs "Điều khiển điều hoà máy lạnh"; footer "Côn tắc thông minh") | chỉ ghi nhận; không rõ cố ý hay do nhập tay |
| Mẫu 1 URL | chỉ có 1 trang chỉ mục Matter; `/matter` đã có file `10-landing-matter.md`, `/matter-search` chưa khảo sát |
| Phiên bản Next.js chính xác | không có header/meta nêu version |
