# Markdown for AI Agents trên Next.js — Hướng đi tính năng

> Ngày: 2026-11-11 · Phạm vi: định hướng kỹ thuật chung, **chưa** khảo sát cấu trúc knxstore.vn.
> Quy ước nhãn: **[Nguồn]** = đã đọc từ tài liệu gốc/bài viết; **[Đề xuất]** = suy luận thiết kế của mình, cần kiểm chứng; **[Chưa rõ]** = không tìm được bằng chứng.

---

## 1. Kết luận nhanh

| Quyết định | Hướng chọn | Lý do chính |
|---|---|---|
| Cơ chế phục vụ | **Content negotiation trên cùng URL** (`Accept: text/markdown`) + **URL tường minh `.md`** cho client không set được header | Negotiation là chuẩn chung (Cloudflare, Vercel, Sanity, Next.js docs đều làm); `.md` bù cho agent không gửi Accept |
| Nguồn sinh Markdown | **Hybrid**: pipeline HTML→MD chung làm mặc định cho *mọi* page type, cộng *adapter* theo page type cho trang giá trị cao (sản phẩm, blog) | Yêu cầu "tất cả loại trang" → cần baseline generic; độ chính xác thông số kỹ thuật → cần override từ data |
| Hook vào Next.js | `proxy.ts` (Next 16, Node runtime) với `matcher.has` header `accept`, rewrite sang route nội bộ | Parse được `q-value`; chỉ chạy khi request có `text/markdown`, không tốn cho traffic thường |
| Cache | `Vary: Accept` trên **cả HTML lẫn MD**, cache MD riêng, invalidate chung tag với HTML | Tránh trả nhầm representation |
| Discovery | Thứ tự: (1) `Accept` negotiation, (2) `Link` header + `<link rel="alternate" type="text/markdown">` trỏ tới URL `.md` tường minh, (3) `sitemap.md`; `llms.txt` để cuối | Crawler chỉ lấy `.md` qua link/URL tường minh, không negotiate; coding agent mới gửi `Accept` (mục 3A) |
| Kỳ vọng | Tối ưu **agent experience + token cost**, không hứa uplift ranking/citation | Chưa có bằng chứng định lượng (mục 3 và 3A) |
| Đo lường | Log `User-Agent`, `Accept`, path ngay từ PoC; quyết định mở rộng dựa trên số liệu của chính site | Số liệu bên ngoài chỉ là chỉ báo (một site, một tháng) |

---

## 2. Cơ chế chuẩn (baseline từ Cloudflare)

Cloudflare "Markdown for Agents" (changelog 2026-02-12) là implementation tham chiếu. Đáng sao chép/đối chiếu:

| Hạng mục | Cloudflare làm | Ghi chú cho bản tự dựng |
|---|---|---|
| Trigger | `Accept: text/markdown` | **[Nguồn]** |
| `Content-Type` | `text/markdown; charset=utf-8` | **[Nguồn]** |
| `Vary` | thêm `Accept`, giữ giá trị origin | **[Nguồn]** |
| Header bị xóa | `Content-Length`, `Content-Encoding`, `Transfer-Encoding`, `Content-Range`, `ETag`, `Last-Modified` | **[Nguồn]** Bản tự dựng nên **tự tính ETag từ hash MD** để hỗ trợ `If-None-Match` → 304 (**[Đề xuất]**) |
| Header giữ nguyên | `Cache-Control`, `Set-Cookie`, CSP, HSTS, CORS... | **[Nguồn]** Với MD route, **không** nên có `Set-Cookie` (**[Đề xuất]**) |
| `x-markdown-tokens` | Ước lượng token, ví dụ `725` | **[Nguồn]** Có trong blog; trang reference chỉ nhắc token ở analytics. Công thức: bytes/4, làm tròn xuống, không dùng tokenizer thật |
| `Content-Signal` | Mặc định `ai-train=yes, search=yes, ai-input=yes` nếu origin không đặt; origin đặt thì ưu tiên origin | **[Nguồn]** → Đây là **quyết định chính sách** của KNXStore, không nên để mặc định |
| Cấu trúc output | (1) YAML frontmatter `title`/`description`/`image` từ meta (`name=` thắng `og:`), (2) body đã bỏ header/footer/nav/script/style, (3) JSON-LD gộp trong một code block `json` ở cuối | **[Nguồn]** |
| Giới hạn | 6 MiB HTML sau giải nén; chỉ HTML | **[Nguồn]** |
| Outcome | `full` / `partial` / `completed_unknown` / `error` / `unfinished` | **[Nguồn]** Nên có metric tương tự để đo chất lượng conversion |
| Ví dụ giảm token | Một bài blog 16.180 → 3.150 token (~80%) | **[Nguồn]** blog Cloudflare, một mẫu duy nhất |
| Client thực sự gửi header | Claude Code, OpenCode; không có danh sách đầy đủ | **[Nguồn]** |
| Cloudflare tự nhận | Beta; chuyển đổi "when possible" | **[Nguồn]** |

Số liệu Vercel (500 KB → 3 KB, giảm 99,37%) là **bytes của một bài blog**, không phải token; không dùng làm kỳ vọng chung.

---

## 3. Bằng chứng về giá trị (để đặt kỳ vọng đúng)

| Chủ đề | Dữ liệu | Nguồn |
|---|---|---|
| llms.txt có được đọc? | Ahrefs, 137K domain: ~28% có file, **97% số file không nhận request nào** trong 05/2026; retrieval bot chỉ ~1% request, phần lớn là coding agent, training crawler và SEO audit tool. Ahrefs lưu ý mẫu thiên về khách hàng kỹ thuật | Ahrefs, SEJ |
| Khảo sát khác | SEmator top 100K: 8,6% có file; Otterly: 84 request llms.txt / >62.100 lượt AI bot | SEOmator, Optimizely roundup (thứ cấp) |
| Google | Trang *AI features* của Search Central: *"You don't need to create new machine readable files, AI text files, or markup to appear in these features."* Phần nói thẳng "Markdown/llms.txt" chỉ thấy qua tóm tắt bên thứ ba về guide ngày 15/05/2026 | Google (trích nguyên văn), SEJ (thứ cấp) |
| Phạm vi của Google | Chỉ AI Overviews / AI Mode, không áp dụng cho ChatGPT, Claude, Perplexity... | SEJ (thứ cấp) |
| Mueller | llms.txt "not done for search", "crutch" cho coding tool | Ahrefs (thứ cấp) |

**Hệ quả [Đề xuất]:**
- Giá trị thực tế nằm ở **agent đọc trực tiếp URL** (coding agent, MCP fetch, pipeline RAG của khách hàng/SI) và giảm token, không phải SEO Google.
- `llms.txt` ưu tiên thấp; `Accept` negotiation + `.md` + `sitemap.md` đáng làm trước.
- Rủi ro cloaking: giữ **parity nội dung** giữa HTML và MD (cùng thông tin, khác định dạng). **[Chưa rõ]** chưa tìm được tuyên bố chính thức của Google về Markdown representation, nên parity là ràng buộc cứng.

---

## 3A. Discovery — agent tự phát hiện bản Markdown bằng cách nào, và có hiệu quả không

### 3A.1 Các cơ chế

| Cơ chế | Agent phải làm gì | Tình trạng |
|---|---|---|
| `Accept: text/markdown` trên cùng URL | Tự gửi header | Checkly test 7 coding agent (đăng 19/02/2026): **Claude Code (2.1.38), Cursor (2.4.28), OpenCode (1.2.5) có gửi**; **Codex, Gemini CLI, GitHub Copilot, Windsurf không gửi** (Gemini CLI, Windsurf gửi `*/*`). **[Nguồn]** |
| `<link rel="alternate" type="text/markdown" href="...">` trong `<head>` | Đọc HTML, theo link | Là đường mà crawler thực tế đã đi (3A.2). Vercel gắn link này trên mọi trang docs. **[Nguồn]** |
| HTTP `Link: <...>; rel="alternate"; type="text/markdown"` | Đọc response header | llms.txt v2 (Jeremy Howard, 10/08/2026) đưa vào; dùng được cả file không phải HTML, đặt được ở server/CDN. Spec còn draft, cú pháp có thể đổi; bài SEJ không nêu cú pháp header chính xác nên cần đối chiếu llmstxt.org. **[Nguồn]** |
| `rel="describedby"` → llms.txt | Theo link từ trang tới file llms.txt bao trùm trang đó | Thêm vào ở llms.txt v2. **[Nguồn]** |
| URL `.md` tường minh (`/x.html.md` hoặc `/x.md`) | Biết trước URL hoặc theo link | llms.txt v2 chính thức hóa cả hai dạng. Vercel: URL `.md` "durable và shareable". **[Nguồn]** |
| `sitemap.md`, `llms.txt`, `llms-full.txt` | Chủ động tìm file | Gần như không được AI bot đọc (3A.2). Chrome Lighthouse (hạng mục Agentic Browsing) có check llms.txt nhưng không check các link relation mới. **[Nguồn]** |
| Phát hiện agent qua User-Agent | Không cần làm gì | Vercel docs: agent nhận ra mà không nêu ưu tiên định dạng thì nhận Markdown (trái với bài blog Vercel nói không dùng UA sniffing). **[Đề xuất] không dùng:** UA tự khai, dễ bị coi là cloaking, khó audit |
| Cloudflare Agent Readiness scanner | Gửi `Accept: text/markdown` để chấm điểm | Theo SEJ, chỉ ~3,9% site pass hạng mục này; Cloudflare nói đây là đề xuất của họ, không phải chuẩn IETF (dù là HTTP content negotiation chuẩn). **[Nguồn thứ cấp]** |

### 3A.2 Số liệu sử dụng thực tế

**Log Cloudflare của Dries Buytaert** (dri.es, đăng 05/03/2026; một tháng; kích thước site không nêu). **[Nguồn]**

| Bot | Hãng | Tổng request | `.md` | % `.md` |
|---|---|---|---|---|
| GPTBot | OpenAI | 3.385 | 1.177 | **34,8%** |
| OAI-SearchBot | OpenAI | 5.722 | 1.300 | **22,7%** |
| Amazonbot | Amazon | 16.872 | 1.840 | 10,9% |
| Claude-User | Anthropic | 94 | 7 | 7,4% |
| Meta AI | Meta | 9.011 | 485 | 5,4% |
| ClaudeBot | Anthropic | 7.144 | 149 | 2,1% |
| PerplexityBot | Perplexity | 467 | 1 | 0,2% |
| ChatGPT-User | OpenAI | 13.864 | 8 | 0,1% |
| Bytespider, CCBot | ByteDance, CommonCrawl | 1.190 / 530 | 0 | 0% |

- **Content negotiation:** 0 request `Accept: text/markdown` từ mọi crawler. Mọi bot lấy Markdown đều đi qua URL `.md` tường minh. Việc bot **tìm thấy** URL `.md` nhờ link auto-discovery là nhận định của tác giả; bài không báo cáo "alternate link" như một kênh đo riêng.
- **Tải crawl:** tăng khoảng 7% sau khi thêm Markdown; bot lấy cả hai định dạng chứ không thay HTML bằng MD.
- **Trích dẫn:** ~1.241 trang bị crawl cho mỗi một lượt trích dẫn; tác giả không thấy bằng chứng Markdown cải thiện việc được trích dẫn.
- **llms.txt:** 52 request trên site của tác giả, toàn bộ từ SEO audit tool; fleet Acquia ~5.000/400 triệu request (0,001%), gần như toàn SEO tool.
- **Kết luận của tác giả:** nếu thêm Markdown dễ thì thử; nếu tốn công thật thì dồn công vào nội dung. llms.txt là "a solution looking for a problem".

Một nguồn khác (kết quả tìm kiếm, **chưa mở bài gốc**) cho biết Claude Code gửi header trên gần như mọi request trong một khảo sát log ~1.400 request và nhận Markdown 98/100 lần; một khảo sát khác báo 0 request `Accept: text/markdown` từ AI crawler trên ~1,28 triệu request. Chỉ dùng làm chỉ báo.

### 3A.3 Giới hạn của bằng chứng

- Buytaert là **một site, một tháng**, đo trước khi llms.txt v2 chính thức hóa các link relation.
- Checkly chỉ đo **header được gửi đi**, không đo agent có xử lý Markdown trả về hay có theo `rel="alternate"` hay không; mỗi agent chỉ test một phiên bản.
- Theo một bảng so sánh thứ cấp, Codex CLI fetch HTML trước rồi theo link alternate sang `.md`; **[Chưa rõ]** không có tài liệu gốc của OpenAI hay Anthropic xác nhận.
- Phần lớn nguồn là blog của vendor SEO/analytics, có thể có lợi ích riêng.

### 3A.4 Hệ quả cho thiết kế **[Đề xuất]**

1. **Hai kênh phục vụ hai nhóm khác nhau:** `Accept` negotiation cho coding agent/fetch tool; `Link` header + `<link rel=alternate>` + URL `.md` cho crawler. Chỉ làm negotiation thì bỏ sót nhóm crawler; chỉ làm `.md` thì bỏ sót nhóm coding agent.
2. **URL `.md` phải ổn định:** crawler lưu và quay lại URL này, nên cần đủ header `canonical`/`noindex` (mục 7.2) để không nhân đôi index.
3. **Lợi ích thực tế nằm ở coding agent và fetch tool** (khách là SI dùng Claude Code/Cursor để tra datasheet). Với crawler OpenAI/Amazon, bằng chứng chỉ cho thấy họ *lấy* Markdown, chưa cho thấy điều đó giúp được *trích dẫn*.
4. **Chi phí/lợi ích:** tự dựng (converter, cache, invalidate, test parity) tốn công thật. Nên giữ phạm vi PoC nhỏ, đo bằng log của chính site trước khi mở rộng.
5. **Không đầu tư sớm vào `llms.txt`/`llms-full.txt`:** dữ liệu cho thấy chủ yếu SEO tool đọc. Nếu làm thì chỉ để rẻ và đồng bộ với `sitemap.md`.

---

## 4. Cơ chế trong Next.js (đã đối chiếu docs)

### 4.1 Thứ tự xử lý request **[Nguồn: nextjs.org, cập nhật 2026-09/11]**

1. `headers` (next.config)
2. `redirects` (next.config)
3. **Proxy** (`proxy.ts`)
4. `beforeFiles` rewrites
5. Filesystem (public, `_next/static`, page tĩnh không dynamic)
6. `afterFiles` rewrites
7. Dynamic routes (`[slug]`)
8. `fallback` rewrites

Redirect chạy trước nên 301 của trang không bị ảnh hưởng. Rewrite `has`/`missing` hỗ trợ `type: header` với `value` dạng regex.

### 4.2 Hai cách hook

| | A. `rewrites()` + `has: header accept` | B. `proxy.ts` + `matcher.has` |
|---|---|---|
| Nguồn tham chiếu | Vercel blog, Vercel KB, Sanity course | Next.js docs; Vercel KB (qua search) nhắc `proxy.ts` nhưng bài fetch được dùng rewrites |
| Match Accept | Regex `(.*)text/markdown(.*)` — khớp mọi chuỗi chứa `text/markdown`, **không xử lý `q`** | Tự parse `Accept` đầy đủ |
| Chi phí | Thấp, khai báo tĩnh | Proxy chạy Node runtime; hạn chế bằng `matcher.has` để chỉ chạy khi có `text/markdown` **[Đề xuất]** |
| Thêm header (`Vary`, `Link`) | Qua `headers()` config | Qua `NextResponse` |
| Khuyến nghị | Đủ cho PoC | **Chọn cho bản chính thức** nếu cần policy q-value, logging, `.md` + negotiation thống nhất một chỗ |

Snippet gốc của Vercel (tham chiếu, không phải code của dự án):

```ts
{
  source: `${prefix}/:path*`,
  has: [{ type: 'header', key: 'accept', value: '(.*)text/markdown(.*)' }],
  destination: `${prefix}/md/:path*`,
}
```

### 4.3 `.md` suffix

Sanity ghi nhận Next.js không cho dynamic segment nằm trong tên file (không có `[slug].md/route.ts`), nên `.md` phải xử lý bằng rewrite `/:path*.md` → route nội bộ. Vercel KB dùng `destination: /docs/:path*.md` nhưng không nói rõ file handler nằm đâu và **không xử lý root path** (suy luận: `/docs` → `/docs/.md`). Cần test riêng trang chủ và các URL có trailing slash.

### 4.4 Policy parse Accept **[Đề xuất]**

| Accept | Kết quả |
|---|---|
| `text/markdown` | MD |
| `text/markdown, text/html;q=0.8` | MD |
| `text/html, text/markdown;q=0.1` | HTML (q thấp hơn; regex của rewrite sẽ trả sai ở case này) |
| `*/*` | HTML (curl mặc định, browser fallback) |
| `text/html,application/xhtml+xml,...` (browser) | HTML |
| `text/markdown;q=0` | HTML |

Một test suite mirror từ Supabase docs chọn policy khác: `text/markdown` tường minh thắng kể cả khi `text/html` có q cao hơn. Đây là lựa chọn policy, cần chốt (browser không gửi `text/markdown` nên rủi ro thực tế thấp).

---

## 5. Nguồn sinh Markdown — quyết định cốt lõi

| Chiến lược | Ưu | Nhược | Phù hợp |
|---|---|---|---|
| **A. Data → MD** (serializer theo page type từ DB/CMS) | Chính xác tuyệt đối; bảng thông số giữ nguyên cấu trúc; nhẹ | Mỗi page type một serializer; dễ **drift** so với HTML; không phủ trang custom | Sản phẩm, bài blog |
| **B. HTML → MD** (convert từ HTML đã render) | Phủ mọi page type bằng một pipeline; parity với HTML theo thiết kế | Phụ thuộc chất lượng converter; mất ngữ nghĩa nếu markup kém; phải lấy được HTML | Baseline cho mọi trang |
| **C. Hybrid** | B bảo đảm coverage, A override nơi cần chính xác | Hai đường code | **Đề xuất** |

Vercel dùng A (rich text CMS → MD, đồng bộ qua shared slug và remote cache Next 16); Cloudflare dùng B (edge convert). Skeptrune dùng B ở build time (`@wcj/html-to-markdown-cli` chạy sau `astro build`).

### 5.1 Cách lấy HTML cho chiến lược B trong Next.js **[Đề xuất, chưa thử]**

| Cách | Mô tả | Rủi ro |
|---|---|---|
| Self-fetch lúc request | Route MD gọi nội bộ HTML của chính URL đó, convert, cache | Cần header bypass chống lặp (`proxy` không rewrite request nội bộ); **không forward Cookie/Authorization**; thêm latency lần đầu |
| Đọc HTML đã build (static/ISR) | Lấy từ `.next/server/app/*.html` sau build | Chỉ phủ trang prerender; phụ thuộc layout nội bộ của `.next` (không phải API công khai) |
| Convert trong CI sau `next build` | Crawl site đã build, sinh file MD | Không phủ trang dynamic; cần cơ chế invalidate khi ISR revalidate |

Self-fetch + cache là hướng thực tế nhất để phủ "mọi loại trang", với điều kiện chốt được mô hình cache (mục 7).

### 5.2 Chọn converter

| Thư viện | Bảng | Ghi chú |
|---|---|---|
| `turndown` | **Core không render bảng**: mỗi ô thành một đoạn văn riêng, mất quan hệ hàng/cột (Thunderbit, 4 mẫu, 0 table row). Cần `turndown-plugin-gfm`; không strip `<script>/<style>`; cần DOM (jsdom/linkedom) | Không dùng nguyên bản cho trang sản phẩm |
| `rehype-remark` (unified) + `remark-gfm` | Bảng GFM được; **bảng có merged cell không biểu diễn được** trong GFM; cần `rehype-minify-whitespace` trước khi convert | **[Đề xuất]** ưu tiên: AST-based, không cần DOM, chèn được plugin (bỏ nav, absolutize link, giữ JSON-LD) |
| `node-html-markdown` | Tác giả công bố nhanh hơn turndown 1,57× (số của vendor); không có sanitize | **[Chưa rõ]** hỗ trợ bảng, chưa có dữ liệu kiểm chứng |
| `markdownify` / `html2text` / `markitdown` (Python) | markdownify, markitdown render bảng tốt nhất trong test Thunderbit; html2text là GPL-3.0 | Chỉ liên quan nếu tách service Python |

Số liệu Thunderbit là bài đo trên 4 mẫu, một bên thứ ba. Không đủ để kết luận cho nội dung KNX; cần benchmark trên trang thật ở bước sau.

Các ca khó cần quyết định trước: bảng merged cell / lồng bảng (gợi ý: fallback giữ `<table>` HTML hoặc flatten theo rule cố định), nội dung trong tab/accordion bị ẩn, ảnh sơ đồ đấu nối (giữ `alt` + caption, không mô tả lại), nội dung render phía client.

---

## 6. Đặc tả output Markdown **[Đề xuất, tham chiếu Cloudflare]**

```
---
title:
description:
canonical:        # URL HTML chuẩn
lang:
lastmod:
type:             # product | article | category | page ...
image:
---
# H1 ...
<main content only: bỏ header/footer/nav/cookie banner/related widget>

<!-- JSON-LD gộp ở cuối, code block json -->
```

- Giữ **JSON-LD** (Product/Article/FAQ): chứa dữ kiện có cấu trúc, Cloudflare cũng giữ.
- Link nội dung trỏ về **canonical HTML URL** tuyệt đối (tránh nhân đôi URL); `sitemap.md` mới liệt kê URL `.md`. Vercel làm ngược (sitemap link `.md`), đây là điểm cần chốt.
- Heading hierarchy, bảng, code block, số/đơn vị (ví dụ "24 V DC", "0,5 A") phải giữ nguyên, kiểm bằng test.
- Giới hạn input (tham chiếu Cloudflare: 6 MiB) và trạng thái `partial` rõ ràng thay vì cắt im lặng.

---

## 7. Cache, header, discovery

### 7.1 Cache

- Sanity: `Vary: Accept` → mỗi URL có ~2 bản cache; tránh được nếu dùng URL tường minh `/markdown/*` (không cần Vary). Vercel blog **không** đề cập Vary/CDN key.
- **[Đề xuất] Rủi ro phân mảnh cache:** `Accept` của các browser khác nhau về chuỗi, nên `Vary: Accept` thô trên HTML có thể làm tụt hit ratio ở CDN/proxy. Nếu có CDN/reverse proxy phía trước: chuẩn hóa `Accept` thành 2 nhóm (markdown / còn lại) trong cache key. Nếu chỉ Node thuần: không phải vấn đề.
- Cache bản MD theo URL canonical; invalidate chung tag với HTML để tránh drift (Vercel dùng shared slug + remote cache).
- Mẫu `Cache-Control` từ Sanity (tham khảo): bài viết `public, max-age=60, stale-while-revalidate=300`; mục/sitemap `public, max-age=300, stale-while-revalidate=600`.

### 7.2 Header

| Response | Header |
|---|---|
| MD | `Content-Type: text/markdown; charset=utf-8`, `Vary: Accept`, `Cache-Control`, `ETag` (hash MD), `X-Markdown-Tokens` (ước lượng), `Content-Signal` (theo chính sách), `Link: <canonical>; rel="canonical"` |
| MD tại URL `.md` | thêm `X-Robots-Tag: noindex` để không index trùng **[Đề xuất; Sanity không đề cập canonical/noindex]** |
| HTML | thêm `Vary: Accept` và `Link: <url.md>; rel="alternate"; type="text/markdown"` |

### 7.3 Discovery

Chi tiết và bằng chứng ở mục 3A. Thứ tự ưu tiên triển khai:

1. `Accept: text/markdown` negotiation (coding agent: Claude Code, Cursor, OpenCode).
2. `Link: <url.md>; rel="alternate"; type="text/markdown"` trên response HTML, cộng `<link rel="alternate" type="text/markdown" href="...">` trong `<head>` (đường crawler đã đi theo log Buytaert). Cú pháp `Link` header cần đối chiếu llmstxt.org vì spec v2 còn draft.
3. URL `.md` tường minh, `Content-Type: text/markdown`, kèm `canonical` và `noindex` (7.2).
4. `/sitemap.md` (Vercel, Next.js docs đều có).
5. `/llms.txt`, `rel="describedby"`: ưu tiên thấp; chính nextjs.org phục vụ `/docs/llms.txt` và `/sitemap.md` và trả MD kèm frontmatter khi fetch.

Không dùng UA sniffing để chọn định dạng.

---

## 8. Phạm vi trang, bảo mật, vận hành

| Hạng mục | Quy tắc **[Đề xuất]** |
|---|---|
| Loại trừ | `/api`, `_next`, ảnh/asset, `sitemap.xml`, `robots.txt`, giỏ hàng, checkout, tài khoản, kết quả search nội bộ, mọi trang cá nhân hóa |
| Dữ liệu người dùng | Convert chạy như **anonymous**; không forward cookie/token; cache key không chứa session |
| Status code | Giữ 404/410; **MD 404 trả MD** (skeptrune ghi nhận trang 404 vẫn trả HTML là điểm yếu) |
| Redirect | Đã xử lý trước proxy/rewrite (thứ tự 4.1) |
| `HEAD` request | Trả header, không convert (Cloudflare không tạo span cho HEAD) |
| Lỗi converter | Fallback: trả HTML bình thường hoặc 406/500 có kiểm soát, ghi log; không trả MD rỗng |
| Rate/CPU | Convert là CPU-bound: cache + giới hạn kích thước + timeout |
| Giám sát | Log `User-Agent`, `Accept`, path, outcome, input/output bytes, thời gian convert (không chặn response) |

---

## 9. Tiêu chí nghiệm thu (đề xuất, đo được)

| # | Tiêu chí | Cách đo |
|---|---|---|
| 1 | HTML cho browser **không đổi** (trừ `Vary`/`Link`) | diff HTML trước/sau |
| 2 | Parity bảng: số hàng/cột MD = HTML | script đếm trên mẫu trang sản phẩm |
| 3 | Parity số liệu kỹ thuật (giá trị + đơn vị) | trích regex số+đơn vị cả hai bản, so tập |
| 4 | Parity heading và số link nội dung | đếm |
| 5 | Không còn nav/footer/cookie banner/widget | kiểm tay + selector |
| 6 | Accept parsing đúng bảng 4.4 | bộ test curl |
| 7 | Không rò dữ liệu người dùng | test với cookie đăng nhập: MD giống bản anonymous |
| 8 | Giảm token so với HTML | `bytes/4` và tokenizer thật; mục tiêu tham khảo ≥ 70% (Cloudflare ~80% trên 1 mẫu), chốt số sau khi đo |
| 9 | Overhead khi cache hit | đo p50/p95; đặt ngưỡng sau PoC |
| 10 | Discovery hoạt động | `curl -I` trên HTML có `Link` header đúng; `<head>` có `rel="alternate"`; URL `.md` trả 200 `text/markdown` và có `canonical`, `noindex` |
| 11 | Đo được lưu lượng agent thật | log theo `User-Agent`, `Accept`, path, kênh vào (negotiation vs `.md`), tỷ lệ MD/HTML theo bot; sau 30 ngày đối chiếu với số liệu Buytaert (mục 3A.2) để quyết định mở rộng hay dừng |

---

## 10. Câu hỏi mở cần chốt trước PoC

1. **Hosting:** Vercel hay self-host (Node/Docker)? Có CDN/reverse proxy phía trước không (đã loại Cloudflare conversion, nhưng còn dùng Cloudflare làm CDN thường không)?
2. **Mô hình render:** tỷ lệ SSG/ISR/SSR của các page type; có nội dung chỉ render phía client không (ví dụ product widget chèn bằng JS)?
3. **Nguồn dữ liệu:** sản phẩm/blog lưu ở đâu, có sẵn dạng cấu trúc để làm adapter (chiến lược A)?
4. **Ngôn ngữ:** một hay nhiều locale; MD có theo locale không?
5. **Content-Signal:** KNXStore cho phép `ai-train`, `search`, `ai-input` ở mức nào?
6. **Link trong MD:** trỏ canonical HTML hay `.md`?
7. **Parse Accept:** q-value chuẩn hay "text/markdown tường minh thì thắng"?

---

## 11. Lộ trình đề xuất

| Giai đoạn | Nội dung | Đầu ra |
|---|---|---|
| 0 | Chốt mục 10 | Quyết định ghi lại |
| 1 | Khảo sát cấu trúc knxstore.vn (page types, rendering mode, data source) | Bảng page type × chiến lược A/B |
| 2 | PoC trên 3 page type (sản phẩm, blog, danh mục) với `proxy.ts` + self-fetch + `rehype-remark`/`remark-gfm`, **kèm logging UA/Accept/kênh vào từ đầu** | Kết quả theo mục 9 |
| 3 | Adapter cho page type cần chính xác; `.md`, `Link` header, `<link rel=alternate>`, `sitemap.md` | Bản chạy staging |
| 4 | Giám sát và rollout theo path; sau 30 ngày đánh giá lại bằng log (tiêu chí 11): mở rộng, giữ nguyên hoặc dừng | Dashboard outcome/token/UA + quyết định ghi lại |

---

## Nguồn

- Cloudflare — [Markdown for Agents (reference)](https://developers.cloudflare.com/fundamentals/reference/markdown-for-agents/) · [changelog 2026-02-12](https://developers.cloudflare.com/changelog/2026-02-12-markdown-for-agents) · [blog](https://www.blog.cloudflare.com/markdown-for-agents/)
- Vercel — [Making agent-friendly pages with content negotiation](https://vercel.com/blog/making-agent-friendly-pages-with-content-negotiation) · [How to serve documentation for agents](https://vercel.com/kb/guide/how-to-serve-documentation-for-agents)
- Sanity — [Markdown Routes with Next.js: production-ready](https://www.sanity.io/learn/course/markdown-routes-with-nextjs/production-ready-markdown-routes)
- Skeptrune — [Respect content negotiation to serve markdown instead of HTML to LLMs](https://dev.to/skeptrune/respect-content-negotiation-to-serve-markdown-instead-of-html-to-llms-2ldn)
- Next.js — [rewrites](https://nextjs.org/docs/app/api-reference/config/next-config-js/rewrites) · [proxy.js](https://nextjs.org/docs/app/api-reference/file-conventions/proxy)
- Ahrefs — [llms.txt study (137K sites)](https://ahrefs.com/blog/llmstxt-study/) · [SEJ summary](https://www.searchenginejournal.com/97-of-llms-txt-files-got-no-requests-ahrefs-data-shows/)
- Google — [AI features and your website](https://developers.google.com/search/docs/appearance/ai-features) · tóm tắt guide 15/05/2026: [SEJ](https://www.searchenginejournal.com/googles-new-ai-search-guide-calls-aeo-and-geo-still-seo/)
- Thunderbit — [So sánh thư viện HTML→Markdown](https://thunderbit.com/zh-Hans/blog/html-to-markdown-libraries-compared)
- Discovery (mục 3A): Checkly — [State of content negotiation for AI agents (02/2026)](https://www.checklyhq.com/blog/state-of-ai-agent-content-negotation/) · SEJ — [llms.txt v2](https://www.searchenginejournal.com/llms-txt-v2-formal-markdown-linking-ai-agents/586119/) · Vercel — [Markdown and Agent Discovery](https://vercel.com/docs/agent-resources/markdown-access) · Dries Buytaert — [Markdown, llms.txt and AI crawlers](https://dri.es/node/6121) · Cloudflare — [Agent Readiness](https://blog.cloudflare.com/agent-readiness/) (điểm adoption 3,9% lấy từ tóm tắt SEJ, thứ cấp)
