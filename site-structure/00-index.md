# knxstore.vn — Tổng hợp cấu trúc hiện tại theo loại trang

> Ngày khảo sát: 2026-11-11 · Chỉ mô tả hiện trạng, không có đề xuất.
> **Cách thu thập:** 5 sub-agent (Sonnet) lấy HTML thô bằng `curl`, phân tích bằng script. 4 agent cho 9 loại trang (file 01–08 và 10; các trang công cụ và trang tĩnh đã khảo sát rồi bỏ file vì không đưa vào MD), mỗi loại **1 URL mẫu chọn ngẫu nhiên** từ sitemap; 1 agent riêng cho skin Matter (file 13–15) với URL do bạn chỉ định.
> **Mức kiểm chứng:** mục "Tự kiểm tra" là mình chạy trực tiếp. Phần còn lại lấy từ báo cáo của sub-agent và 12 file chi tiết; mình mới kiểm tra định dạng (đủ 10 mục mỗi file) và đối chiếu header cache của 5 trang, chưa đọc lại toàn bộ nội dung từng file.

---

## 1. Sitemap

`robots.txt` khai báo `Sitemap: https://knxstore.vn/sitemap.xml`; `Disallow: /admin/` và `/search?`. `sitemap.xml` là index trỏ tới 5 sitemap con **(tự kiểm tra)**:

| Sitemap | Số URL | Loại URL |
|---|---|---|
| `/sitemaps/products.xml` | 752 | `/products/{slug}` |
| `/sitemaps/categories.xml` | 91 | `/{slug}` (root, không có đuôi) |
| `/sitemaps/posts.xml` | 238 | 227 bài `/{slug}.html`, 9 trang chủ đề `/blogs/{topic}`, 2 trang tác giả `/author/{slug}` |
| `/sitemaps/brands.xml` | 83 | 71 `/brands/{slug}` + 12 `/solution/{slug}` |
| `/sitemaps/pages.xml` | 19 | Home, 4 trang gốc (`/products`, `/brands`, `/blogs`, `/solution`), `/matter`, `/matter/categories`, `/matter-search`, `/contact`, `/tools/hvac-compatibility`, 2 `/tool/*/index.html`, 3 trang chính sách `.html`, `/page/about-us`, `/visual`, 2 `/visual/*` |

Danh mục và bài blog dùng chung root namespace `/{slug}`, phân biệt bằng đuôi `.html`.

---

## 2. Bảng loại trang

| # | Loại trang | URL mẫu | Số URL cùng loại | Chế độ render | `cache-control` | Container nội dung | JSON-LD | File chi tiết |
|---|---|---|---|---|---|---|---|---|
| 01 | Trang chủ | `/` | 1 | ISR (`x-nextjs-cache: HIT`) | `s-maxage=3600` | `div#main-content` | 1 | [01-home.md](01-home.md) |
| 02 | Sản phẩm | `/products/sunricher-phim-bam-gan-tuong-dali-2-8-phim-sr-2422nk8-da2` | 752 | ISR (`x-nextjs-prerender: 1`) | `s-maxage=3600, swr=31532400` | `div#main-content` (skin B2B) | 1 (`ItemPage`) | [02-product.md](02-product.md) |
| 03 | Danh mục sản phẩm | `/dimmer-doc-lap-triac` | 91 | Dynamic SSR | `private, no-store` | `div#main-content` | 1 | [03-product-category.md](03-product-category.md) |
| 04 | Bài blog | `/matter-fabric-la-gi.html` | 227 | Dynamic SSR | `private, no-store` | `div#main-content` | 5 node (BlogPosting, FAQPage...) | [04-blog-post.md](04-blog-post.md) |
| 05 | Chủ đề blog | `/blogs/an-ninh` | 9 | Dynamic SSR | `private, no-store` | `div#main-content` | Không có | [05-blog-topic.md](05-blog-topic.md) |
| 06 | Tác giả | `/author/le-vo-quoc-huy` | 2 | Dynamic SSR | `private, no-store` | `div#main-content` | ProfilePage + Person | [06-author.md](06-author.md) |
| 07 | Thương hiệu | `/brands/protopixel` | 71 | Dynamic SSR | `no-store`, có header `Link` | `div#main-content` | 1 | [07-brand.md](07-brand.md) |
| 08 | Giải pháp | `/solution/giai-phap-khac` | 12 | Dynamic SSR | `no-store`, có header `Link` | `div#main-content` | 1 | [08-solution.md](08-solution.md) |
| 10 | Landing Matter | `/matter` | 3 (`/matter`, `/matter/categories`, `/matter-search`) | ISR | `s-maxage=3600` | Header/footer riêng khu Matter | — | [10-landing-matter.md](10-landing-matter.md) |
| 13 | Sản phẩm (skin Matter) | `/products/cam-bien-chuyen-dong-eve-motion-matter-over-thread-ipx3` | thuộc 752 URL sản phẩm | ISR (`x-nextjs-prerender: 1`) | `s-maxage=3600, swr=31532400` | Không có `#main-content`; wrapper `div.font-sans[--font-matter]` | 4 node (có Product, FAQPage) | [13-matter-product.md](13-matter-product.md) |
| 14 | Danh mục (skin Matter) | `/cong-tac-matter` | thuộc 91 URL danh mục (9 slug chứa `matter` trong sitemap) | Dynamic SSR | `private, no-store` | Không có `#main-content` | — | [14-matter-category.md](14-matter-category.md) |
| 15 | Chỉ mục danh mục Matter | `/matter/categories` | 1 | ISR | `s-maxage=3600, swr=31532400` | `<main>` (trang Matter duy nhất có) | Không có | [15-matter-categories-index.md](15-matter-categories-index.md) |

Cột render/cache của 02, 03, 04, 05, 07, 13, 14, 15 đã được **tự kiểm tra** bằng `curl -I` (khớp báo cáo). Các hàng còn lại theo báo cáo sub-agent. Số "9 slug chứa `matter`" ở hàng 14 lấy từ `categories.xml`: cam-bien-matter, cong-tac-matter, den-thong-minh-matter, dong-co-rem-matter, driver-matter, gateway-matter, khoa-thong-minh-matter, o-cam-thong-minh-matter, san-pham-matter. Đây chỉ là các slug có chữ `matter`; breadcrumb của sản phẩm Eve hiển thị "Điều khiển Matter" nên có thể còn danh mục Matter không chứa chữ này trong slug. Chưa đếm lại bằng script.

---

## 3. Đặc điểm chung (đã quan sát trên nhiều loại trang)

| Hạng mục | Hiện trạng |
|---|---|
| Hạ tầng | `server: cloudflare`; `cf-cache-status: DYNAMIC` trên mọi trang HTML đã kiểm tra (không cache HTML ở Cloudflare). **Tự kiểm tra** |
| `Vary` | `rsc, next-router-state-tree, next-router-prefetch, next-router-segment-prefetch, Accept-Encoding`, **không có `Accept`** (đo trên `/`). **Tự kiểm tra** |
| `Accept: text/markdown` | Trả `200 text/html`; không có xử lý content negotiation. Đo trên `/` (tự kiểm tra) và trên các URL mẫu khác (theo sub-agent) |
| URL `.md` | Trả 404 trên mọi URL mẫu đã thử |
| `link rel=alternate` / hreflang / meta robots | Không có trên trang chủ, brand, solution (theo agent C); blog/sản phẩm không báo cáo link alternate |
| `<main>` / `<article>` | Không có trên các trang B2B đã khảo sát; vùng nội dung là `div#main-content`. Skin Matter (sản phẩm, danh mục) cũng không có `#main-content`; `/matter/categories` là trang Matter duy nhất có `<main>` |
| Payload RSC (`self.__next_f`) | Chiếm khoảng 44–52% HTML thô (sản phẩm ~45%, blog 156–211 KB/trang, trang chủ 501 KB) |
| Header B2B | ~128,755 B, **giống hệt từng byte** trên trang chủ, brand, solution; chứa mega-menu ẩn bằng class `hidden`, chiếm 30–37% HTML thô ở trang blog. Dữ liệu menu (~59 KB) bị nhân đôi trong payload |
| Footer B2B | ~17,608 B; render hai lần (mobile + desktop) nên 19 link bị lặp |
| JSON-LD | Thường 1 script nằm trong `<body>`; blog có 5 node. Payload RSC lặp lại JSON-LD |
| Skin | Hai skin khác nhau: B2B (có `#main-content`, skip link) và Matter (header/footer riêng, không có skip link và `#main-content`). Dấu hiệu skin Matter: wrapper `div.font-sans[--font-matter]` và script đặt cookie `knx-area=matter`. Header Matter 9,4 KB / 17 link, header B2B 128,8 KB / 165 link. JSON-LD Matter 4 node (thêm Product, FAQPage), B2B 2 node. Wrapper skin nằm ở layout segment `[slug]` với sản phẩm/danh mục; với `/matter/categories` do chính page render (suy ra từ cây Flight). Payload RSC của 3 trang Matter mang ~55 KB shell B2B (từ `notFound`), chiếm 31–58% payload. **Quy tắc chọn skin: chưa xác định** (chưa thử gửi cookie) |
| `llms.txt` | **Đã tồn tại**: `/llms.txt` trả 200 `text/plain`, 7.354 B, `s-maxage=3600`; mở đầu bằng mô tả công ty, thông tin liên hệ, link B2B/B2C, mục "Danh mục sản phẩm" (23 dòng link dạng `- [..](..)`). Mình mới đọc phần đầu, chưa đọc hết. **Tự kiểm tra** |

---

## 4. Nội dung có trong dữ liệu nhưng không có trong DOM ban đầu

| Trang | Nội dung | Có ở đâu |
|---|---|---|
| Sản phẩm Matter (Aqara LB-L02E, mẫu đối chiếu) | 14 dòng thông số, 5 FAQ | JSON-LD + payload RSC; chỉ panel của tab đang chọn được render |
| Sản phẩm DALI (mẫu chính) | `specs`, `downloads`, `faqs` rỗng; JSON-LD không có `Product`/`Offer` | Không có ở đâu cả |
| Trang chủ | `contentHtml` bài dài của 7 giải pháp (~160 KB) | Chỉ trong payload RSC; slider chỉ hiện `summary` |
| `/matter` | `descriptionHtml`, `specs[]`, `seoTitle`, tồn kho (~82 KB) của 18 sản phẩm | Payload RSC; DOM chỉ có brand, tên, giá |
| Thương hiệu | Mô tả thương hiệu | Chỉ trong meta/JSON-LD; DOM để trống |
| Bài blog | Mục lục (TOC) | Client component; heading trong thân bài không có `id` trong HTML (JS gán `muc-N` sau hydration, suy từ chunk) |
| Danh mục | Toolbar lọc/sort/chuyển dạng xem | `div[hidden]#S:0` sau footer (Suspense streamed), về đúng chỗ khi script `$RC("B:0","S:0")` chạy |
| Sản phẩm Matter (Eve Motion) | `specs` (11 dòng), `faqs` (7 Q&A) | RSC + JSON-LD; tab dùng `useState`, chỉ render panel đang chọn |
| Sản phẩm Matter (Eve Motion) | `shortDescription`, `b2cContent.noiDung.matter_banner`, `matter_application_1..3` (3 ảnh + 3 đoạn HTML mô tả) | **Chỉ trong RSC**: không có trong DOM lẫn JSON-LD, và agent không thấy component client nào render ra |
| Sản phẩm Matter (Eve Motion) | Badge giỏ "1" trong ảnh chụp | Client-only: HTML chỉ có `aria-label="Giỏ báo giá, 0 sản phẩm"`, không có thẻ badge |
| Danh mục Matter (`/cong-tac-matter`) | Toolbar (lọc, "12 sản phẩm", sắp xếp, đổi dạng xem) | `div#S:0[hidden]` ở cuối `<body>`, ngoài wrapper; chuyển vào chỗ `template#B:0` khi script `$RC` chạy |

---

## 5. Phân trang và danh sách

| Trang | Hiện trạng |
|---|---|
| Danh mục | 24 card/trang, `?page=N`; ví dụ 29 sản phẩm nên trang 2 có 5 sản phẩm; sản phẩm trang 2 không có trong HTML hay payload của trang 1 |
| Chủ đề blog | Không phân trang: `/blogs/an-ninh` hiện cả 14 bài; `/blogs/kien-thuc` có 84 card trong HTML 632 KB; `?page=2` trả nội dung giống trang đầu |
| Tác giả | 82 bài của tác giả có đủ trong HTML ban đầu |
| Thương hiệu | Mẫu Protopixel chỉ có 2 sản phẩm nên chưa quan sát được cơ chế phân trang |
| Danh mục Matter | `/cong-tac-matter`: 12 card nằm trọn trong HTML, không phân trang; `/matter/categories`: 12 tile |

---

## 6. Số đo đáng chú ý

| Trang | HTML thô | Ghi chú |
|---|---|---|
| Trang chủ | 957.496 B | `__next_f` 501 KB (52%); nén zstd còn 68–104 KB. Thời gian tải thất thường: 3/7 lần đo mất 25–39 s dù TTFB 0,84 s ở các lần nhanh (nguyên nhân chưa xác định) |
| `/matter` | 552 KB | |
| `/blogs/kien-thuc` | 632 KB | 84 card |
| `/brands/protopixel` | 325 KB | Text vùng chính 217 ký tự trên 5.696 ký tự hiển thị |
| `/solution/giai-phap-khac` | 407 KB | |
| Bài blog | — | `__next_f` 156–211 KB (44–50% HTML) |
| Sản phẩm Matter (Eve Motion) | 354 KB | Script chiếm 60% |

---

## 7. Điểm bất thường trong nội dung/metadata (quan sát được, ngoài phạm vi cấu trúc)

| Trang | Quan sát |
|---|---|
| `/matter` | Slide hero thứ 3 có một câu tiếng Anh lạc đề |
| `/solution/giai-phap-khac` | H1 lặp chữ ("Giải pháp Giải pháp khác"); title và description không chứa tên giải pháp; breadcrumb JSON-LD có 2 mục, DOM có 3 mục; trang mỏng, không đại diện cho 12 trang giải pháp |
| `/products/…sr-2422nk8-da2` | `specs`, `downloads`, `faqs` đều rỗng |
| `/cong-tac-matter` | Text SEO nói "14 model" nhưng trang có 12 sản phẩm |
| Sản phẩm Matter (Eve Motion) | Gallery: dữ liệu có 6 ảnh, DOM có 7 `img` (đều có `srcset`; ảnh chính `fetchpriority=high`, thumbnail `loading=lazy`, không có placeholder/blur). Trong ảnh chụp màn hình, ảnh lớn và 5/6 thumbnail hiển thị trống. Agent gọi thử 9 URL ảnh, đều trả 200 và là ảnh hợp lệ; **nguyên nhân hiển thị trống chưa xác định**. Bản thumbnail có `alt=""` (tự kiểm tra) |

---

## 8. Chưa khảo sát / chưa xác định

- **Mẫu:** mỗi loại chỉ 1 URL (sản phẩm có thêm 1 mẫu đối chiếu; blog topic có thêm `/blogs/kien-thuc`). Chưa biết các khối như card sản phẩm nhúng, callout, list, code, iframe trong 226 bài blog còn lại; template của 90 danh mục còn lại; cách render `contentHtml` ở 11 trang giải pháp còn lại.
- **Điều kiện hệ thống:** điều kiện chọn skin B2B/Matter; điều kiện có node `Product` trong JSON-LD; tên query param khi lọc danh mục.
- **Chưa fetch:** `/visual` và 2 `/visual/*`, `/author/ta-minh-vu`, `/products`, `/brands`, `/blogs`, `/solution` (4 trang gốc).
- **DOM sau hydration:** không ai chạy trình duyệt, nên TOC thực tế, menu mobile, popup, cookie banner, hành vi tab/variant/filter chỉ suy ra từ HTML và bundle JS.
- **Hạ tầng:** cấu hình render ở origin (không có source); nguyên nhân độ trễ ngẫu nhiên của trang chủ; brotli (server trả zstd).
- **Ngoài phạm vi (không đưa vào MD):** mọi trang công cụ (`/tool/*/index.html`, `/tools/*`; đã khảo sát GA planner và HVAC compatibility rồi bỏ file), trang tĩnh (`/page/about-us`, `/contact`, 3 trang chính sách `.html`; đã khảo sát about-us rồi bỏ file), `/matter-search`, giỏ hàng, checkout, tài khoản.
- **Skin Matter:** quy tắc chọn skin Matter/B2B (chưa thử cookie `knx-area`); bundle `Toolbar` và `MatterQuoteButton` chưa đọc; chưa thử query lọc/sắp xếp/phân trang ở danh mục Matter; không tìm được trang B2B tương ứng với `/matter/categories` để so sánh; mỗi mẫu chỉ 1 URL; chưa khảo sát `/matter-search`.
