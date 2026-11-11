# Bài blog — cấu trúc hiện tại

> https://knxstore.vn/matter-fabric-la-gi.html · nguồn sitemap: posts.xml · 227 URL cùng loại (`/{slug}.html` ở root) · khảo sát 2026-11-11

Phạm vi: chỉ mô tả hiện trạng, đo trên HTML thô lấy bằng `curl -A "Mozilla/5.0"` (không chạy trình duyệt, không thực thi JS). Mọi nhận định về "client-only" dựa trên HTML ban đầu + payload RSC + source chunk JS tải về.

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
| link | 6 preload: 4 font `woff2` (`/_next/static/media/051742360c26797e-s.p.*.woff2`, `2bbe8d26…`, `83afe278…`, `fa39153a…`) + 2 style (`/_next/static/chunks/1mmmxvxvvtkv8.css`, `/_next/static/chunks/1vbr8oh4939ca.css`). Không có `rel=alternate`, không có `rel=canonical` trong header |
| x-robots-tag | Không có |
| content-encoding | Không có khi không gửi `Accept-Encoding`; `br` khi gửi `gzip, br`; `gzip` khi chỉ gửi `gzip` |
| Khác | `server: cloudflare`, `x-frame-options: SAMEORIGIN`, `x-content-type-options: nosniff`, `strict-transport-security: max-age=31536000; includeSubDomains`, `referrer-policy: same-origin`, `speculation-rules: "/cdn-cgi/speculation"`, `expect-ct`, `nel`/`report-to`, `alt-svc: h3`. Không có `etag`, `last-modified`, `age`, `x-powered-by` |

### 1.2 Chế độ render (suy ra)

Kết luận: **dynamic render theo từng request (SSR), không cache ở Cloudflare**. Bằng chứng:

- `cache-control: private, no-cache, no-store, max-age=0, must-revalidate` (không có `s-maxage`, không có `stale-while-revalidate`).
- Không có `x-nextjs-cache`, `x-nextjs-prerender`, `x-nextjs-stale-time`.
- `cf-cache-status: DYNAMIC` (Cloudflare không cache; hệ quả của `no-store`, tự nó không chứng minh chế độ ở origin).
- Payload RSC dòng `0:` có `"S":false`; ý nghĩa của cờ này chưa xác nhận bằng tài liệu.
- 2 request liên tiếp trả cùng độ dài (357,724 ký tự), chỉ khác thứ tự stream (vị trí `<meta name="next-size-adjust">` và thứ tự một chunk RSC), nội dung giống nhau.
- Route (từ RSC): `(site)/[slug]`, giá trị param `slug` = `matter-fabric-la-gi.html` (gồm cả đuôi `.html`). Các segment anh em liệt kê trong RSC: `admin, api, author, blogs, brands, cart, clearance, compare, connection-check, contact, context, css, label, llms.txt, mail-success, matter, matter-search, order-success, page, products, robots.txt, search, sitemap.xml, sitemaps, solution, tools, track-order, visual, warranty, warranty-lookup`.
- Build ID `rJVjL74JjgHtrkQQTzrjV`. Chunk `turbopack-0j8dnv21rrwyx.js` cho thấy build Turbopack. Phiên bản Next.js: Chưa xác định (không có header nào lộ phiên bản).

### 1.3 Hành vi hiện tại với yêu cầu Markdown (chỉ ghi kết quả)

| Request | Status | content-type | Kích thước body |
|---|---|---|---|
| `GET /matter-fabric-la-gi.html` + `Accept: text/markdown` | 200 | `text/html; charset=utf-8` | 369,871 B (bằng bản thường) |
| `GET /matter-fabric-la-gi.html.md` | 404 | `text/html; charset=utf-8` | 131,169 B |
| `GET /matter-fabric-la-gi.md` | 404 | `text/html; charset=utf-8` | 131,159 B |

Các response 404 vẫn mang `cache-control: private, no-cache, no-store…` và `cf-cache-status: DYNAMIC`.

---

## 2. `<head>`

| Thành phần | Giá trị |
|---|---|
| `<html>` | `lang="vi"`, class biến font `onest_…__variable inter_…__variable jetbrains_mono_…__variable` |
| `<title>` | `Matter Fabric Là Gì? Mạng Tin Cậy Mã Hóa Trong Smarthome` (không có hậu tố `\| knxstore.vn`; khác H1) |
| meta description | `Matter Fabric là mạng bảo mật mã hóa liên kết thiết bị Matter qua chứng chỉ tin cậy, cho phép nhiều nền tảng cùng hoạt động. Tìm hiểu tại KNXStore.vn.` |
| canonical | `https://knxstore.vn/matter-fabric-la-gi.html` (tuyệt đối, self) |
| robots | Không có meta robots; không có `x-robots-tag` |
| `meta name=author` | `Tạ Minh Vũ` |
| `meta name=keywords` | `Kiến thức` (trùng tên chuyên mục) |
| viewport / theme-color | `width=device-width, initial-scale=1` / `#14467C` |
| OG | `og:title` (= title), `og:description` (= meta description), `og:url` (= canonical), `og:image` = `https://api.knxstore.vn/media/88dbe6556d89d940c338c40a0643787e13f5ca43dae19e9698898f18478a6077.jpg`, `og:type=article`, `article:published_time=2026-08-03T03:34:19.000Z`. Không có `og:site_name`, `og:locale`, `og:image:width/height/alt`, `article:modified_time`, `article:author`, `article:section`, `article:tag` |
| Twitter | `twitter:card=summary_large_image`, `twitter:title`, `twitter:description`, `twitter:image` (cùng ảnh OG) |
| hreflang / `rel=alternate` | Không có (kể cả không có link markdown/feed) |
| `rel=manifest` | `/manifest.webmanifest` |
| icon | `/favicon.ico?favicon.16_cwkv0v-dvu.ico` (48x48), `/icon.svg?icon.39v289yelkh8f.svg`, `/apple-icon.png?apple-icon.264ni-uec6-1q.png` (180x180) |
| preload | `/images/logo/knxstore-logo-tight.svg` (image); ảnh hero (`rel=preload as=image` + `imageSrcSet` 640→3840w, `imageSizes="(max-width: 1320px) 100vw, 1272px"`); 2 stylesheet `data-precedence=next`; 1 script `3ivcc6t1s04dh.js` (`fetchPriority=low`); `https://www.googletagmanager.com/gtag/js?id=G-4D9XLJDVGG` (as=script) |
| script | 12 `<script async src>` chunk Next trong `<head>` (`2i51…`, `3odw…`, `3ert…`, `turbopack-…`, `3fnt…`, `3dg4…`, `11xb…`, `204r…`, `3037…`, `2f_6…`, `2ns7…`, `39o1…`) + 1 `noModule` (`0cz1d0mv5g_q7.js`) + 1 `3ivcc6t1s04dh.js` (`id=_R_ async`, cuối `<body>`). Tổng 14 thẻ `<script src>` trong tài liệu |
| Số bytes `<head>` | 4,969 B |

---

## 3. Khung trang

### 3.1 Landmark

| Landmark | Số lượng | Ghi chú |
|---|---|---|
| `<header>` | 1 | `header.sticky.top-0.z-9999`, 128,755 B (34.8% HTML thô) |
| `<nav>` | 10 | header 6 (`aria-label`: Điều hướng chính; Toàn bộ danh mục sản phẩm; Danh mục giải pháp; Danh mục thương hiệu; Danh mục công cụ; Chuyên mục blog kỹ thuật), main 1 (Breadcrumb), footer 3 (Danh Mục Sản Phẩm; Giải Pháp; Về chúng tôi). Nav mục lục bài viết không có trong HTML ban đầu (xem mục 6) |
| `<main>` | **0** | Không có thẻ `<main>`. Vùng nội dung là `div#main-content[tabindex=-1]` (đích của skip-link `a[href=#main-content]`, text "Bỏ qua, tới nội dung chính") |
| `<article>` | **0** | Thân bài là `div#article-body.article-body` |
| `<aside>` | 1 | Cột phải trong main (`hidden min-[960px]:block min-[960px]:sticky`) |
| `<footer>` | 1 | 17,608 B |
| `<section>` | 2 | FAQ; chia sẻ (`section[data-share-buttons]`) |
| `role` | `role=search` (form header), `role=combobox` (`input#b2b-header-search`), `role=region aria-label=Trending` (`div#b2b-trending-panel`), `role=group aria-label="Tìm theo"` (panel chọn phạm vi tìm), `role=status aria-live=polite` (`p` trạng thái chia sẻ) |

### 3.2 Header (SSR toàn bộ, phần lớn ẩn bằng class `hidden`)

- Thanh trên: nút `button[aria-label="Mở menu"]` (mobile), logo `a[href=/] img[src=/images/logo/knxstore-logo-tight.svg]`, form tìm kiếm `role=search` (chỉ hiện ≥ `lg`): nút phạm vi `#b2b-header-scope` + panel 7 chip (Tất cả, Sản phẩm, Bài viết, Thương hiệu, Giải pháp, Danh mục, Visual) + `input#b2b-header-search`; nút tìm kiếm mobile; link giỏ báo giá `a[href=/cart][aria-label="Giỏ báo giá, 0 sản phẩm"]`; nút `a[href=/contact]` "Tư vấn dự án".
- Thanh điều hướng: 6 nút (Trending, Danh mục sản phẩm, Giải pháp, Thương hiệu, Công cụ, Blog kỹ thuật) + `a[href=/visual]` "Visual" + `a[href=/matter]` "Matter Smarthome".
- Mega-menu (6 panel `#b2b-trending-panel`, `#b2b-menu-categories`, `#b2b-menu-solutions`, `#b2b-menu-brands`, `#b2b-menu-tools`, `#b2b-menu-blogs`, tất cả class `hidden` mặc định) có trong HTML: 165 thẻ `<a>`, 67 `<img>` (66 `loading=lazy`), 123 `<li>`, 18 thẻ `<h3>`, 16 link `/products/…` (12 unique) kèm giá hiển thị dạng `4.865.000 đ`, 8 từ khóa trending, 24 link `/brands/…` (+ "Xem tất cả 75 thương hiệu"), 12 link `/solution/…`, 9 link `/blogs/{topic}`, 14 link bài blog `/{slug}.html` ("Bài mới nhất" + bài trending).
- Text hiển thị của header: 4,190 ký tự.

### 3.3 Footer (SSR)

- Cột trái: logo (`img[alt=knxstore]`), "CÔNG TY CP TÍCH HỢP HỆ THỐNG LIÊN MINH", MST 0311846236, địa chỉ (link Google Maps `target=_blank rel="noopener noreferrer"`), `tel:0918918755`, `mailto:support@knxstore.vn`, Facebook (`facebook.com/knxstore.vn`), Zalo (`zalo.me/0918918755`), 2 badge (Bộ Công Thương `online.gov.vn`, DMCA; cả hai `rel="nofollow noopener noreferrer"`).
- 3 cột: **Danh Mục Sản Phẩm** (7 link: `/bo-dieu-khien-trung-tam`, `/dieu-khien-chieu-sang`, `/dieu-khien-may-lanh`, `/cong-tac-thong-minh`, `/cam-bien-hien-dien`, `/led-driver-1`, `/kiem-soat-ra-vao`), **Giải Pháp** (7: `/solution/knx|dali|casambi|an-ninh|hvac|matter|dmx`), **Về chúng tôi** (5: `/contact`, `/blogs`, `/clearance`, `https://knxstore.vn/warranty-lookup`, `/page/about-us`).
- Mỗi cột render **2 lần** trong DOM: dạng accordion `details` (mobile, `div.sm:hidden`) và dạng `nav` + `h2` (≥ `sm`). Tổng 50 thẻ `<a>` trong footer.
- Dòng cuối: "© 2026 KNXStore — CÔNG TY CP TÍCH HỢP HỆ THỐNG LIÊN MINH . Bảo lưu mọi…" + 4 link: Tra cứu bảo hành (`/warranty-lookup`), Chính sách bảo mật (`/chinh-sach-bao-mat.html`), Chính Sách Mua Hàng (`/chinh-sach-mua-hang-tai-knxstore-vn.html`), Điều khoản sử dụng (`/dieu-khoan-su-dung-website-knxstore-vn.html`).
- Text footer: 1,165 ký tự.

### 3.4 Thành phần nổi / phụ

| Thành phần | Selector | Trạng thái |
|---|---|---|
| Nút Zalo nổi | `a[data-floating-zalo]` → `https://zalo.me/0918918755`, `fixed right-[26px]`, 52x52px, nền `#0068ff`, span `animate-ping` | SSR, hiển thị |
| Ngăn kéo "Yêu cầu báo giá" | `div.fixed.top-0.left-0.z-99999.h-screen.bg-dark/70…translate-x-full` > `h2` "Yêu cầu báo giá", "Chưa có thiết bị nào trong yêu cầu. Xem sản phẩm", "Tổng số lượng: 0", link "Xem và gửi yêu cầu"; không có `role=dialog`/`aria-modal` | SSR (trạng thái rỗng), ẩn ngoài màn hình |
| Header sticky | `header.sticky.top-0` | SSR |
| Aside sticky (CTA) | `aside…min-[960px]:sticky` | SSR; ẩn dưới 960px |
| Skip-link | `a[href=#main-content].fixed…-translate-y-[200%]` | SSR; chỉ hiện khi focus |
| GA | `GoogleAnalytics` (client component, gtag `G-4D9XLJDVGG`) | Client |

Không thấy trong HTML ban đầu và payload RSC: cookie banner, nút back-to-top, thanh tiến độ đọc, popup, chat widget khác ngoài link Zalo. Việc chúng có được chèn sau hydration hay không: Chưa xác định (không chạy trình duyệt).

Ghi chú: ngay đầu `<body>` có `<div hidden><!--$--><!--/$--></div>` và `<script>document.cookie="knx-area=b2b; Path=/; SameSite=Lax"+…</script>` (inline, ghi cookie `knx-area=b2b`).

---

## 4. Cây nội dung vùng chính (theo thứ tự DOM)

Vùng chính = `div#main-content` (43,664 B, text 8,799 ký tự). Cột "Main" = nằm trong `div#main-content`. Tất cả mục dưới đây có trong HTML ban đầu (SSR) trừ khi ghi khác.

### 4.1 Khung và thông tin đầu bài

| # | Thẻ + id/class (rút gọn) | Heading | Nội dung tóm tắt | Main | Hiển thị mặc định | SSR/Client |
|---|---|---|---|---|---|---|
| 1 | `script[type=application/ld+json]` (con đầu tiên của `#main-content`) | - | `@graph` 5 node, 7,587 ký tự (xem mục 8) | Có | Không hiển thị | SSR |
| 2 | `div.bg-white > div.mx-auto.max-w-[1320px].px-6.pb-16.pt-6` | - | Container bao toàn bộ nội dung bài | Có | Hiện | SSR |
| 3 | `nav[aria-label=Breadcrumb]` (không dùng `<ol>`, không microdata) | - | `Trang chủ / Blog kỹ thuật / Kiến thức / Matter Fabric là gì?`; 3 link `/`, `/blogs`, `/blogs/kien-thuc`; mục cuối `span[aria-current=page]`; dấu `/` là `span[aria-hidden]` | Có | Hiện | SSR |
| 4 | `a.uppercase.text-[12px]` → `/blogs/kien-thuc` | - | Nhãn chuyên mục "Kiến thức" | Có | Hiện | SSR |
| 5 | `h1.mt-2.5.max-w-[44ch].text-[28px].sm:text-[36px]` | **H1** | "Matter Fabric là gì?" (20 ký tự) | Có | Hiện | SSR |
| 6 | `div.mt-4.flex.flex-wrap…text-[13.5px]` (meta bài) | - | `span` (svg lucide-calendar + chuỗi `3/8/2026`, định dạng d/m/yyyy, khớp `published_time` 2026-08-03) · chấm tách · `span` (svg lucide-user + `a[href=/author/ta-minh-vu]` "Tạ Minh Vũ"). Không có thẻ `<time>`/`datetime`, không có ngày cập nhật, không có thời gian đọc, không có tag | Có | Hiện | SSR |
| 7 | `div.relative.mt-6.aspect-[16/9]…bg-zinc-100 > img.object-cover` | - | Ảnh đại diện, `alt="Matter Fabric là gì?"` (= H1), `data-nimg=fill`, `sizes="(max-width: 1320px) 100vw, 1272px"`, `srcSet` `/_next/image?url=%2Fmedia%2F88db…jpg&w=640…3840&q=75`, không `loading` (eager, đã preload), không `<figure>`/caption. RSC: `priority:true` | Có | Hiện | SSR |
| 8 | `div.mt-8.grid.grid-cols-1…min-[960px]:grid-cols-[minmax(0,1fr)_260px]` | - | Lưới 2 cột từ 960px: cột nội dung (`div.min-w-0`) + cột `aside` 260px; tổng text 8,288 ký tự | Có | Hiện | SSR |

### 4.2 Thân bài `div#article-body.article-body > div` (6,448 ký tự text, 8,242 B UTF-8)

Thân bài là chuỗi HTML từ CMS, chèn bằng `dangerouslySetInnerHTML` (RSC: `2e:[["$","div","0",{"dangerouslySetInnerHTML":{"__html":"$3c"}}]]`, text chunk `3c:T25ba,…` 7,955 ký tự). Không có `class`/`id` trên `p/h2/h3/table/td/th/img` (trừ `th[scope=col]`). Whitespace gốc của CMS giữ nguyên: 75 cặp CRLF + 54 tab trong toàn file. Không có `id` trên heading nào trong thân bài.

| # | Thẻ | Heading | Nội dung tóm tắt | Ký tự |
|---|---|---|---|---|
| 9.1 | `p` | - | Đoạn dẫn: Matter Fabric là mạng kết nối khép kín, chỉ thiết bị đã xác thực bằng chứng chỉ mới tham gia; mỗi nền tảng một Fabric; Multi-Admin | 423 |
| 9.2 | `h2` | **H2** Matter Fabric Là Gì: Mạng Kết Nối Khép Kín Chỉ Cho Thiết Bị Đã Xác Thực | | 71 |
| 9.3-9.5 | `p` x3 | - | Định nghĩa; nghĩa từ "fabric" và phân biệt với Thread; ví dụ nhóm chat mã hóa | 347 / 475 / 223 |
| 9.6 | `p > img` | - | `alt="Matter Fabric là gì 1"`, `src=/media/4f06…d521f.jpg`, `style="height:394px; width:700px"` | 0 |
| 9.7 | `h3` | **H3** Mỗi Thiết Bị Là Một Node, Xác Thực Bằng Chứng Chỉ Riêng | | 55 |
| 9.8-9.9 | `p` x2 | - | Node và chứng chỉ; dẫn vào bảng ("Fabric thường được quản lý bởi chính nền tảng bạn dùng…:") | 430 / 86 |
| 9.10 | `table` | - | Bảng nền tảng - tên Fabric (chi tiết mục 4.4) | 141 |
| 9.11 | `p` | - | Home Assistant và các hệ mở tự tạo Fabric riêng | 216 |
| 9.12 | `p > img` | - | `alt="Matter Fabric là gì 2"` | 0 |
| 9.13 | `h2` | **H2** Một Thiết Bị Thuộc Nhiều Fabric Cùng Lúc Nhờ Chế Độ Multi-Admin | | 63 |
| 9.14-9.16 | `p` x3 | - | Multi-Admin; ví dụ Alexa rồi Apple Home; Matter 1.4 yêu cầu tối thiểu 5 Fabric. Đoạn 9.16 có link `/5-buoc-thiet-lap-nha-thong-minh-theo-chuan-matter.html` | 174 / 534 / 433 |
| 9.17 | `p > img` | - | `alt="Matter Fabric là gì 3"` | 0 |
| 9.18 | `h2` | **H2** Matter Fabric Không Phụ Thuộc Thread, Nhưng Vẫn Bị Giới Hạn Bởi Mạng Thread Phía Dưới | | 85 |
| 9.19-9.21 | `p` x3 | - | Fabric là lớp logic; nhầm lẫn Fabric/Thread; Thread Border Router (link `/thread-border-router-la-gi.html`) | 322 / 308 / 848 |
| 9.22 | `h2` | **H2** Vì Sao Hiểu Matter Fabric Giúp Bạn Tránh Rắc Rối Khi Dùng Nhiều Nền Tảng Cùng Lúc | | 81 |
| 9.23-9.25 | `p` x3 | - | Hai tình huống rắc rối; ví dụ cụ thể; kết (link `/matter-la-gi-cam-nang-toan-tap-ve-giao-thuc-matter.html`, `/thiet-bi-nao-tuong-thich-voi-matter.html`) | 216 / 588 / 308 |
| 9.26 | `p > img` | - | `alt="Matter Fabric là gì 4"` | 0 |

Outline heading trong thân bài (đầy đủ): H2 (4) + H3 (1), không có H4-H6:

```
H2 Matter Fabric Là Gì: Mạng Kết Nối Khép Kín Chỉ Cho Thiết Bị Đã Xác Thực
  H3 Mỗi Thiết Bị Là Một Node, Xác Thực Bằng Chứng Chỉ Riêng
H2 Một Thiết Bị Thuộc Nhiều Fabric Cùng Lúc Nhờ Chế Độ Multi-Admin
H2 Matter Fabric Không Phụ Thuộc Thread, Nhưng Vẫn Bị Giới Hạn Bởi Mạng Thread Phía Dưới
H2 Vì Sao Hiểu Matter Fabric Giúp Bạn Tránh Rắc Rối Khi Dùng Nhiều Nền Tảng Cùng Lúc
```

Thống kê thẻ trong thân bài: `p` 20 (16 có text, 4 chỉ chứa `img`), `h2` 4, `h3` 1, `table` 1, `img` 4, `a` 4. Không có: `ul/ol/li`, `blockquote`, `pre/code`, `figure/figcaption`, `iframe/video`, `strong/em/b/i`, `br`, `span`, `div` lồng (chỉ 1 `div` bọc ngoài).

### 4.3 Các khối sau thân bài

| # | Thẻ + id/class (rút gọn) | Heading | Nội dung tóm tắt | Main | Hiển thị mặc định | SSR/Client |
|---|---|---|---|---|---|---|
| 10 | `section.mt-10.border-t.border-zinc-200.pt-6` > `ul.flex.flex-col.gap-3` > 5 x `li > details.group.rounded-card…` | **H2** "Câu hỏi thường gặp"; mỗi câu hỏi là **H3** trong `summary` | 5 cặp Q/A; `summary` = `h3.font-semibold` + `span[aria-hidden]` (icon +/x); đáp án `p.border-t.border-zinc-200.py-4` (311/296/235/218/294 ký tự). Câu hỏi và đáp án khớp từng ký tự với JSON-LD FAQPage | Có | Đáp án nằm trong DOM nhưng `details` đóng (không có thuộc tính `open`; RSC `"open":false`) | SSR |
| 11 | `section[data-share-buttons].mt-8.border-t` | **H2** "Chia sẻ bài viết" | `a` Facebook (`facebook.com/sharer/sharer.php?u=<URL mã hóa>`), `button` Instagram, `a` LinkedIn (`linkedin.com/sharing/share-offsite/?url=…`), `button` "Chép link", `p.min-h-5[role=status]` rỗng. Link ngoài `target=_blank rel="noopener noreferrer"` | Có | Hiện | SSR (HTML); hành vi nút = Client |
| 12 | `aside.hidden.min-[960px]:block.min-[960px]:sticky.top-[calc(var(--site-header-h,64px)_+_24px)]` | - | Chỉ chứa khối CTA (mục 4.5). Vị trí đặt TOC có trong RSC nhưng trống trong HTML | Có | Ẩn dưới 960px; sticky từ 960px | SSR (CTA); TOC = Client |
| 13 | `div.mt-14` > `p.uppercase.text-primary` + `div.grid.sm:grid-cols-2.lg:grid-cols-3` | Không dùng thẻ heading (`p`) "Bài viết liên quan" | 4 card `a.group.flex.h-full…` (mục 4.6) | Có | Hiện | SSR |
| 14 | `div.fixed.top-0.left-0.z-99999…translate-x-full` | **H2** "Yêu cầu báo giá" | Ngăn kéo báo giá (mục 3.4), nằm ngoài `#main-content` (anh em liền sau) | Không | Ẩn | SSR (rỗng); trạng thái = Client |

Không có trong trang: khối tác giả cuối bài (chỉ có byline link ở đầu bài), bình luận, form/CTA đăng ký, tag/chủ đề, card sản phẩm nhúng, callout/box lưu ý, video/iframe (YouTube...), TOC trong HTML ban đầu.

### 4.4 Bảng trong thân bài

- 1 bảng, không `class`, không `caption`. Cấu trúc: `thead > tr > th[scope=col]` x2 ("Nền tảng", "Tên Fabric tương ứng"); `tbody > tr` x4, mỗi hàng `td` x2. Ô đầu mỗi hàng là `td` (không phải `th[scope=row]`). Không có `colspan/rowspan`.
- Dữ liệu: Amazon Alexa → Amazon Fabric; Apple Home → Apple Fabric; Google Home → Google Fabric; Samsung SmartThings → Samsung Fabric.
- Không có wrapper cuộn ngang (`overflow-x`) quanh bảng trong HTML.

### 4.5 CTA trong aside

`div.mt-4.rounded-card.border.border-zinc-200.bg-canvas.p-5`: `p` "Cần tư vấn thiết bị hoặc giải pháp cho công trình của bạn?" · `a[href=tel:0918918755]` "0918.918.755" · `a[href=/contact]` "Tư vấn dự án".

### 4.6 Card bài liên quan (4 card)

Cấu trúc 1 card: `a[href=/{slug}.html].group.flex.h-full.min-w-0.flex-col…` > `div.aspect-[4/3]` > `img.object-cover[loading=lazy]` (next/image fill, `/_next/image?url=%2Fmedia%2F…avif&w=256…3840&q=75`, `alt` = tiêu đề) · `div.px-4.pb-4.pt-3.5` > `span.truncate.uppercase` (chuyên mục "Kiến thức") + `p.line-clamp-2` (tiêu đề) + `span` "Ngày đăng <b>d/m/yyyy</b>". Không có mô tả, tác giả, thời gian đọc.

| # | Tiêu đề | Ngày hiển thị |
|---|---|---|
| 1 | Giám Sát Năng Lượng Trong Hệ Thống KNX: Vì Sao Phải Đo Điện Trước Khi Tiết Kiệm Điện | 13/10/2026 |
| 2 | IC Là Gì? Cấu Tạo, Chức Năng Và Vai Trò Trong Smarthome | 17/8/2026 |
| 3 | NFC là gì? Vì sao smarthome hiện đại đều tích hợp công nghệ này? | 5/3/2026 |
| 4 | SPI Là Gì? Ứng Dụng SPI LED Controller Trong LED RGBW | 17/8/2026 |

---

## 5. Chi tiết khối đặc biệt (trích HTML)

**Bảng (thân bài, giữ nguyên tab/xuống dòng của CMS):**

```html
<table>
	<thead>
		<tr>
			<th scope="col">Nền tảng</th>
			<th scope="col">Tên Fabric tương ứng</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>Amazon Alexa</td>
			<td>Amazon Fabric</td>
		</tr>
		<!-- 3 hàng còn lại cùng dạng -->
```

**Ảnh trong thân bài (không `figure`, không `loading`, kích thước bằng inline style):**

```html
<p><img alt="Matter Fabric là gì 1" src="/media/4f0655961741ae201173208522bf55710288a2fb98f5af5766801eee537d521f.jpg" style="height:394px; width:700px" /></p>
```

**FAQ (`details` đóng, đáp án có sẵn trong DOM):**

```html
<li><details class="group rounded-card border border-zinc-200 bg-white px-5 …"><summary class="flex cursor-pointer list-none … [&::-webkit-details-marker]:hidden"><h3 class="font-semibold text-zinc-900">Matter Fabric khác Matter Controller như thế nào?</h3><span aria-hidden="true" …><svg/></span></summary><p class="border-t border-zinc-200 py-4 leading-relaxed text-zinc-700">Matter Fabric là mạng kết nối khép kín gồm các thiết bị đã được xác thực, …</p></details></li>
```

**Aside trong HTML ban đầu (không có TOC):**

```html
<aside class="hidden min-[960px]:sticky … min-[960px]:block min-[960px]:self-start"><div class="mt-4 rounded-card border border-zinc-200 bg-canvas p-5"><p …>Cần tư vấn thiết bị hoặc giải pháp cho công trình của bạn?</p><a href="tel:0918918755" …>0918.918.755</a><a class="…" href="/contact">Tư vấn dự án</a></div></aside>
```

**TOC: payload RSC (dòng `36:`) cho thấy aside có client component nhận `bodyId`:**

```
36:["$","aside",null,{"className":"hidden min-[960px]:sticky …","children":[["$","$L39",null,{"bodyId":"article-body"}],["$","div",null,{"className":"mt-4 rounded-card …
39:I[38380,[…],"default"]
```

**TOC: logic client (module 38380 trong `/_next/static/chunks/2ns7gl8cadig5.js`, đọc từ source, không phải DOM quan sát):**

```js
let r=[...document.getElementById(bodyId).querySelectorAll("h2, h3")] ; // gán id nếu rỗng:
""===e.id&&(e.id=`muc-${t+1}`)  // → <nav aria-label="Mục lục bài viết"><p>Mục lục</p><ul><li><a href="#muc-N">…
```

Suy ra từ code + outline: sau hydration, TOC có 5 mục (`#muc-1` H2, `#muc-2` H3 thụt `pl-6`, `#muc-3`, `#muc-4`, `#muc-5`), render `null` nếu không có heading; làm nổi mục đang đọc bằng scroll listener; click gọi `history.replaceState` + smooth scroll.

**JSON-LD (trích BlogPosting):**

```json
{"@type":"BlogPosting","@id":"https://knxstore.vn/matter-fabric-la-gi.html#article","headline":"Matter Fabric là gì?","datePublished":"2026-08-03T03:34:19.000Z","dateModified":"2026-08-03T03:34:19.000Z","author":{"@id":"https://knxstore.vn/author/ta-minh-vu#author"},"articleSection":"Kiến thức","mainEntityOfPage":{"@id":"…/matter-fabric-la-gi.html#webpage"}}
```

**Nút chia sẻ (`section[data-share-buttons]`):**

```html
<a href="https://www.facebook.com/sharer/sharer.php?u=https%3A%2F%2Fknxstore.vn%2Fmatter-fabric-la-gi.html" target="_blank" rel="noopener noreferrer" class="inline-flex h-10 …">Facebook</a><button type="button" class="inline-flex h-10 …">Instagram</button>
```

---

## 6. Nội dung chỉ có ở client hoặc KHÔNG thấy trong HTML ban đầu

| Mục | Bằng chứng | Ghi chú |
|---|---|---|
| **TOC (mục lục)** | Chuỗi "Mục lục" xuất hiện 0 lần trong HTML; aside chỉ có CTA; RSC `36:` có `$L39 {bodyId:"article-body"}`; module 38380 dùng `useState`/`useEffect`, trả `null` khi chưa có heading | Hoàn toàn Client; không tồn tại trong HTML ban đầu |
| **`id` của heading thân bài** | 0 `id` trên `h2/h3` trong `#article-body`; code gán `muc-${index+1}` sau hydration | Anchor `#muc-N` chỉ có sau khi JS chạy |
| Nút "Instagram", "Chép link" | `button type=button` không `onclick` trong HTML; component `$L38` (module 85392) nhận `{url,title}`; `p[role=status]` rỗng trong SSR | Markup SSR có, hành vi và thông báo trạng thái là Client |
| Ngăn kéo báo giá, badge giỏ | HTML chứa trạng thái rỗng ("Tổng số lượng: 0", `aria-label="Giỏ báo giá, 0 sản phẩm"`) | Nội dung giỏ thực tế: Client; chưa xác định nguồn state |
| Gợi ý tìm kiếm, mở/đóng mega-menu, chọn phạm vi tìm | `role=combobox` rỗng; panel `hidden` | Client |
| Menu mobile (`button[aria-label="Mở menu"]`) | Không thấy markup drawer menu mobile trong HTML | Chưa xác định (có thể dựng khi bấm nút) |
| Google Analytics (gtag `G-4D9XLJDVGG`) | `GoogleAnalytics` client component + `<link rel=preload as=script>` | Client |
| Cookie `knx-area=b2b` | Script inline đầu `<body>` | Client |
| Dữ liệu chỉ có trong RSC, không render ra HTML | Props `nav` của `HeaderB2b` (id, sku, shortDescription sản phẩm; excerpt, publishedAt, author.bio của bài trending) | Chỉ thuộc menu header, không liên quan thân bài |

Nội dung bài **không** client-only: thân bài, bảng, ảnh, FAQ, bài liên quan, breadcrumb, byline, JSON-LD đều có trong HTML ban đầu. Không có `<template data-dgst>`, không có `BAILOUT_TO_CLIENT_SIDE_RENDERING`, không có skeleton/`animate-pulse` (ngoài hiệu ứng `animate-ping` của nút Zalo), không có `<!--$?-->` hay `<!--$!-->` (chỉ có `<!--$-->` rỗng).

---

## 7. Liên kết

| Loại | Mẫu URL | Ví dụ trong trang |
|---|---|---|
| Bài blog | `/{slug}.html` | `/thread-border-router-la-gi.html`, `/5-buoc-thiet-lap-nha-thong-minh-theo-chuan-matter.html` |
| Blog gốc / chuyên mục | `/blogs`, `/blogs/{topic}` | `/blogs/kien-thuc` (9 topic trong menu: an-ninh, cam-bien, chieu-sang, du-an, huong-dan, hvac, kien-thuc, knx, tin-tuc) |
| Tác giả | `/author/{slug}` | `/author/ta-minh-vu` |
| Sản phẩm | `/products/{slug}` | chỉ trong mega-menu header (16 link, 12 unique) |
| Danh mục sản phẩm | `/{category-slug}` | `/bo-dieu-khien-trung-tam`, `/dali-gateway`, `/cong-tac-knx` |
| Giải pháp | `/solution/{x}` | `/solution/knx`, `/solution/matter` (12) |
| Thương hiệu | `/brands/{x}` | 24 link + `/brands` |
| Công cụ | `/tool/{x}/index.html`, `/tools/{x}` | `/tool/knx-ga-planner/index.html`, `/tools/hvac-compatibility` |
| Khác | `/search?q=`, `/cart`, `/contact`, `/visual`, `/matter`, `/clearance`, `/warranty-lookup`, `/page/about-us` | |
| Trang chính sách cùng dạng `.html` | `/chinh-sach-bao-mat.html`, `/chinh-sach-mua-hang-tai-knxstore-vn.html`, `/dieu-khoan-su-dung-website-knxstore-vn.html` | Dạng `/{slug}.html` ở root cũng được dùng cho các trang này |

Số liệu: toàn trang 236 `<a>` (header 165, main 17, drawer 2, footer 50, skip-link 1, nút Zalo 1). Trong thân bài: 4 link, cả 4 là link nội bộ tới bài blog, anchor text mô tả, không có link ngoài. Trong main: 2 link ngoài (Facebook/LinkedIn sharer, `target=_blank rel="noopener noreferrer"`, không `nofollow`). Link `nofollow`: chỉ 2 badge ở footer (`online.gov.vn`, `dmca.com`). Không có `rel=alternate`, không có link/URL markdown nào trong `<head>`, header `Link` hoặc thân trang.

---

## 8. JSON-LD

1 thẻ `<script type="application/ld+json">` (con đầu tiên của `div#main-content`, 8,372 B kèm thẻ; nội dung 7,587 ký tự), 1 `@context` + `@graph` gồm 5 node. Cùng nội dung cũng có trong RSC (dòng `41:T2088,…`).

| # | `@type` | `@id` | Trường có mặt | Thiếu / rỗng / lưu ý |
|---|---|---|---|---|
| 1 | `["Organization","LocalBusiness"]` | `https://knxstore.vn/#organization` | name, legalName, url, foundingDate, taxID, logo (ImageObject, `@id #logo`, 150x50), image (1200x630), description, telephone, email, address (PostalAddress), geo, hasMap, openingHoursSpecification, priceRange, areaServed, knowsAbout (10 mục), contactPoint, sameAs (Facebook, YouTube) | `logo.url` là `/assets/logo.png` (dạng khác với `/images/logo/knxstore-logo-tight.svg` đang dùng trên trang) |
| 2 | `WebPage` | `…/matter-fabric-la-gi.html#webpage` | url, name, description, inLanguage `vi-VN`, datePublished, dateModified, isPartOf, publisher, mainEntity (→ `#article`), author (→ `ta-minh-vu#author`), primaryImageOfPage (ImageObject `#primary-image`, caption), breadcrumb (BreadcrumbList 4 ListItem) | `isPartOf` trỏ `https://knxstore.vn/#website`: không có node `WebSite` nào trong `@graph`. Breadcrumb mục 2 tên `Tin tức` (`/blogs`) trong khi breadcrumb hiển thị là `Blog kỹ thuật` |
| 3 | `BlogPosting` | `…#article` | headline (`Matter Fabric là gì?` = H1), name (= title), description, url, inLanguage, datePublished, dateModified, isPartOf, publisher, author (→ Person), image (chuỗi URL `api.knxstore.vn/media/88db…jpg`), articleSection `Kiến thức`, keywords `Kiến thức`, about (Thing `Kiến thức` + url), mainEntityOfPage | `dateModified` = `datePublished` (`2026-08-03T03:34:19.000Z`). Không có `wordCount`, `timeRequired`, `thumbnailUrl`, `speakable`. `image` là chuỗi URL, không phải ImageObject |
| 4 | `Person` | `https://knxstore.vn/author/ta-minh-vu#author` | name, url, image (ImageObject + caption), jobTitle `System Integration & Marketing Engineer`, description, worksFor (→ Organization) | `description` chứa ký tự xuống dòng (`\n`) và khoảng trắng kép; không có `sameAs`, `knowsAbout`, `email` |
| 5 | `FAQPage` | `…#faq` | mainEntity 5 `Question` + `acceptedAnswer.Answer.text` | Khớp từng ký tự với FAQ hiển thị |

Không có: `Article` riêng ngoài `BlogPosting`, `HowTo`, `Table`, `ImageObject` cho ảnh thân bài, `Product`/`ItemList`.

---

## 9. Số đo

| Chỉ số | Giá trị |
|---|---|
| HTML thô | **369,871 B** (≈ 92,467 token theo bytes/4) |
| Sau nén (server, `Accept-Encoding`) | gzip **65,394 B**; br **39,814 B** (gzip mức 6 tự nén lại: 65,972 B) |
| Tổng thẻ `<script>` | 18 thẻ, **180,674 B** (48.8% HTML thô) |
| - `self.__next_f` | 2 thẻ, **171,066 B** (1 thẻ khởi tạo `push([0])` 43 B + 1 thẻ dữ liệu); nội dung RSC giải mã = 148,479 ký tự, 80 dòng |
| - `ld+json` | 1 thẻ, 8,372 B |
| - `<script src>` | 14 |
| `<style>` inline | 0 |
| Text hiển thị (bỏ script/style) | **14,346 ký tự** = 18,024 B UTF-8 (≈ 4,506 token theo bytes/4) |
| Text `#main-content` (không gồm ngăn kéo báo giá) | 8,799 ký tự = 11,203 B (≈ 2,800 token) |
| Text riêng thân bài `#article-body` | **6,448 ký tự** = 8,242 B (≈ 2,060 token); HTML thân bài 7,837 ký tự / 9,639 B |
| Text FAQ | ≈ 1,699 ký tự |
| Text header / footer | 4,190 / 1,165 ký tự |
| Bytes theo vùng | `<head>` 4,969 · header 128,755 · main (gồm JSON-LD) 43,664 · ngăn kéo báo giá 2,585 · footer 17,608 · phần sau footer (nút Zalo + script) 171,808 |
| Thẻ | `<a>` 236 · `<img>` 79 (header 67, main 9, footer 3) · `<table>` 1 · `<h1>` 1 · `<h2>` 10 · `<h3>` 24 · `<h4>`-`<h6>` 0 · `<li>` 178 · `<ul>` 32 · `<details>` 8 (5 FAQ + 3 footer) · `<svg>` 45 · `<button>` 19 · `<form>` 1 (tìm kiếm header) · `<nav>` 10 · `<iframe>`/`<video>`/`<figure>`/`<pre>`/`<blockquote>` 0 |
| Chi tiết h2 | 4 thân bài + "Câu hỏi thường gặp" + "Chia sẻ bài viết" + "Yêu cầu báo giá" + 3 footer = 10 |
| Chi tiết h3 | 18 (header mega-menu, 2 thẻ rỗng) + 1 thân bài + 5 câu hỏi FAQ = 24 |
| Tỷ lệ | Thân bài = 2.2% HTML thô (bytes); text `main` = 3.0% |

Request đã dùng: 8 (trang chính, `Accept: text/markdown`, `.html.md`, `.md`, br, gzip, 2 chunk JS để đọc logic TOC/share).

---

## 10. Chưa xác định / lưu ý

- **Cấu hình render ở origin** (revalidate, `generateStaticParams`, `dynamic`): Chưa xác định, không có quyền xem source. Kết luận "dynamic" chỉ dựa trên header và payload. Không loại trừ khả năng header `x-nextjs-*` bị lược ở tầng proxy.
- **DOM sau hydration:** không chạy trình duyệt. TOC (5 mục) và id `muc-N` suy ra từ code module 38380 + outline thân bài, chưa quan sát trực tiếp.
- **Tính đại diện của 1 mẫu:** bài mẫu không có card sản phẩm nhúng, CTA/form, callout, blockquote, code, list, iframe/video, link ngoài trong thân bài. 226 bài còn lại có thể chứa các khối này; cách chúng render (server hay client, class gì): Chưa xác định.
- Thân bài là HTML CMS thô (class/id rỗng, `style` inline cho ảnh, CRLF/tab). Bài viết cũ hơn có thể có markup khác (ví dụ mẫu khác trong cùng loại chưa được kiểm tra).
- Ngày hiển thị `3/8/2026` là d/m/yyyy (khớp `published_time` 2026-08-03). `dateModified` trùng `datePublished`: không xác định được bài đã sửa hay chưa. Không có ngày cập nhật hiển thị.
- Tên chuyên mục không nhất quán giữa breadcrumb hiển thị (`Blog kỹ thuật`) và JSON-LD (`Tin tức`); `<title>` bài không có hậu tố `| knxstore.vn` trong khi trang chuyên mục/tác giả có.
- Ảnh trong thân bài dùng đường dẫn `/media/…` trực tiếp (không qua `/_next/image`), ảnh OG/JSON-LD dùng host `api.knxstore.vn/media/…`.
- Header chiếm 34.8% HTML thô và 29% text hiển thị; 24 link thương hiệu, 16 link sản phẩm kèm giá và 14 link bài blog trong mega-menu có mặt trong DOM nhưng ẩn bằng class `hidden`.
- Payload RSC chứa 2 bản gần giống nhau của cây layout/header (dòng `5:` 28,069 và `11:` 28,070 ký tự) và nhiều khối footer lặp, cộng với bản sao thân bài (`3c:`, 7,955 ký tự) và JSON-LD (`41:`, 7,692 ký tự): cùng một nội dung xuất hiện cả trong HTML lẫn RSC.
- Số URL "227 cùng loại" lấy theo yêu cầu, không tự kiểm lại từ `posts.xml`. Có trang chính sách cùng dạng `/{slug}.html` (xem mục 7); chưa xác định chúng có nằm trong `posts.xml` hay không.
