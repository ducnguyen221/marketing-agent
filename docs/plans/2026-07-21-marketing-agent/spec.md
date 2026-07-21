# SPEC — marketing-agent

> Ngày: 2026-07-21 · Trạng thái: **chờ duyệt Gate 1** · Thiết kế & task: [plan.md](plan.md)

## 1. Vấn đề

Ba mảng tồn tại rời nhau, mỗi mảng thiếu đúng cái mảng kia có:

| Mảng | Có | Thiếu |
|---|---|---|
| Khoá **MKA-101 Marketing Analytics** | Giáo trình 6 buổi, case KPIM Bank, 60 post + 2 CSV Facebook khớp khoá 60/60 | 8 bảng dữ liệu → **chặn S02, S03, S05 và 2 trang dashboard của S06** |
| Kênh **Duc Nguyen AI** | 51.01 rất chắc: 7 tầng funnel, North Star `subs/1.000 engaged views`, schema đo 17 trường (§12.6), luật winner "vượt median ≥2 KPI" | 52.02 là **prose Markdown**: không `asset_id`, không `live_url`, không `platform_native_id`, không baseline → **luật winner không tính được** |
| Hạ tầng `.news` / `.tts` | Đủ chiều đẩy: sinh → render → YouTube → Facebook → Excel → Telegram. `video_id`/`short_id`/`fb_post_id` đã lưu bền trong sidecar `*.published.json` | **Chiều đọc ngược bằng 0** — không một lời gọi insights/analytics nào |

Hệ quả: campaign chạy thật mỗi ngày nhưng **không ai biết cái gì hiệu quả**; lớp học có phương pháp nhưng thiếu dữ liệu để luyện.

## 2. Ý tưởng lõi — repo là ENGINE, không phải kho nội dung

`marketing-agent` chỉ chứa **skill · agent · workflow · script · reference template**. Không chứa nội dung của ai cả, trừ **một bộ mẫu duy nhất**.

```
        marketing-agent (engine, public, không secret)
                        │
                install.ps1 hỏi người dùng
                        │
        ┌───────────────┼────────────────────────────┐
        ▼               ▼                            ▼
  content/KPIM/    content/<slug>/            <đường dẫn ngoài>
  ★ bộ mẫu         instance của người         vd Brain/... cho
  commit vào repo  cài (gitignored)           Duc Nguyen AI
  phục vụ MKA-101
```

- **`content/KPIM/`** — instance mẫu **duy nhất** đi kèm repo: dữ liệu mock KPIM Bank + toàn bộ campaign do AI sinh, phục vụ khoá học. Đây chính là bản sao của những gì làm cho Duc Nguyen AI nhưng bằng dữ liệu mô phỏng → public an toàn, học viên clone về chạy được ngay.
- **Instance thật** (Duc Nguyen AI, Nghe Tiên Truyện, KPIM Page…) — người cài tự chọn vị trí lúc install; **không bao giờ vào git**.

Một engine, nhiều instance. Mỗi cải tiến schema/measure/dashboard làm một lần, hưởng ở mọi instance — kể cả bộ mẫu đem đi dạy.

## 3. Yêu cầu

### 3.1 Repo chứa gì

| Thư mục | Nội dung | Vào git? |
|---|---|---|
| `skills/` | skill cross-tool (Claude/Codex/Antigravity) | ✅ |
| `agents/` | subagent persona: strategist · producer · QA reviewer · insight analyst | ✅ |
| `workflows/` | pipeline xác định: campaign-launch · weekly-measure · course-dataset-build | ✅ |
| `scripts/` | calendar · publish · collect · model · courseware | ✅ |
| `templates/` | **schema canonical**, khung instance, prompt playbook, checklist QA, `.pbip` mẫu | ✅ |
| `content/KPIM/` | bộ mẫu khoá học (mock, không PII) | ✅ |
| `content/<slug>/` | instance người cài | ❌ gitignore |
| `.env`, token | | ❌ không bao giờ |

### 3.2 Installer phải hỏi những gì

1. **Tên instance** — mặc định theo tên kênh hoặc chủ đề nội dung muốn triển khai (người cài tự đặt).
2. **Nơi đặt content** — mặc định `content/<slug>` trong repo; hoặc đưa đường dẫn ngoài (Duc Nguyen AI → Brain).
3. **Kênh** — YouTube / Facebook Page / cả hai.
4. **Nguồn số liệu** — file export tay / API / **cả hai** (đường tay là dự phòng khi API hỏng, đồng thời là bài lab S04).
5. **Mức tự trị** — `suggest` (mặc định) / `auto-safe` / `full`.
6. **Ngôn ngữ + brand voice profile** (giọng, luật cấm lộ tên tool nội bộ).

→ scaffold folder từ `templates/instance/`, ghi `instance.yml`, đăng ký vào registry cục bộ (gitignored).

### 3.3 Chức năng end-to-end

campaign brief → calendar máy đọc → AI sinh nội dung → **approval gate** → publish YouTube + Facebook → ghi ngược `platform_native_id`/`live_url` → thu số liệu (2 đường) → star schema → Power BI 5 trang → báo cáo AI + quyết định 28 ngày.

### 3.4 Phi chức năng

- **Public-safe**: engine công khai; token/dữ liệu thật không bao giờ vào repo.
- **Sandbox first**: `--dry-run` là **mặc định**, không phải tuỳ chọn — khoá học cấm đăng thật trong lab.
- **Ba mức tự trị** (OpcOS core.md): `full` chỉ bật được bằng tay tại máy, có xác nhận tương tác.
- **Không phá cái đang chạy**: 6 scheduled task của `.news` không bị đụng ở mọi pha.
- **Không lộ tên tool nội bộ** ra output công khai (kế thừa luật kênh).

## 4. Ranh giới (không làm)

- Không viết lại pipeline `.news` — marketing-agent **wrap**, không thay.
- Không build n8n thay học viên; repo xuất dữ liệu/định dạng để n8n tiêu thụ (đó là bài lab S02/S03).
- Không PII, không dữ liệu khách hàng thật ở bất kỳ nhánh nào.

## 5. Giả định (ASSUMPTION — cần xác nhận)

1. **A1** — Facebook của Duc Nguyen AI là **Page** do Đức quản trị (Graph API không đọc được profile cá nhân).
2. **A2** — Kênh YouTube do Đức sở hữu → chạy lại `--auth` thêm `yt-analytics.readonly` được (retention/impressions chỉ có với kênh mình sở hữu).
3. **A3** — Campaign 52.02 (13/07→09/08/2026) **đang chạy dở**; phải đối chiếu asset nào đã đăng thật trước khi cấp `asset_id`.
4. **A4** — Giữ nguyên 60 post KPIM Bank; dữ liệu sinh bù phải khớp khoá `fb_post_id` sẵn có.

## 6. Rủi ro

| Rủi ro | Mức | Giảm thiểu |
|---|---|---|
| Repo public + có khả năng đăng thật → lộ token | **Cao** | `content/*` gitignore trừ `KPIM`; chỉ `.env.example`; pre-commit chặn pattern token |
| Học viên clone về chạy nhầm lên trang thật | Cao | dry-run mặc định; `full` phải bật tay + xác nhận |
| Baseline median chưa tồn tại → luật "winner" chưa tính được ở Pha 1 | Trung bình | Ghi nợ tường minh; dashboard hiện "chưa đủ mẫu", **không hiện số giả** |
| Xung đột định danh 3 hệ tên (brief / xlsx / CSV tiếng Việt) | Trung bình | `crosswalk.yml` là artifact bắt buộc Pha 1 |
| Hai hệ pillar chưa ánh xạ (A..G của 51.01 vs 1..6 của 52.01) | Trung bình | Chốt A..G canonical cho kênh, thêm cột crosswalk |
| Đụng nhầm `.news` làm chết 6 task đang chạy | Cao | Pha 3 chỉ **copy** adapter, không sửa file gốc |
| Ghi dữ liệu đo hằng ngày vào Brain gây nhiễu git/lint/quarterly review | Trung bình | Tách `content_root` (markdown → Brain) khỏi `data_root` (CSV → ngoài Brain) — **chờ Đức chốt** |
