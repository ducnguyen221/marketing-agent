# PLAN — marketing-agent

> Ngày: 2026-07-21 · Trạng thái: **chờ duyệt Gate 1** · Spec: [spec.md](spec.md)

## 1. Cây repo (engine thuần)

```
Code/marketing-agent/
├─ README.md                     # marketing-agent là gì · cài thế nào · chạy thế nào
├─ install.ps1 / install.sh      # hỏi người dùng → scaffold instance → đăng ký registry
├─ .env.example                  # KHÔNG có giá trị thật
├─ .gitignore                    # content/* trừ content/KPIM · *.token.json · .env
│
├─ skills/                       # cross-tool (Claude · Codex · Antigravity)
│  ├─ campaign-plan/SKILL.md         # brief → calendar máy đọc, chấm TOS/HRS
│  ├─ content-produce/SKILL.md       # calendar → hero/shorts/fb/community theo prompt playbook
│  ├─ approval-gate/SKILL.md         # checklist QA + máy trạng thái duyệt
│  ├─ metrics-collect/SKILL.md       # nạp số liệu 2 đường về cùng schema
│  ├─ campaign-report/SKILL.md       # insight có bằng chứng + quyết định 28 ngày
│  └─ course-dataset/SKILL.md        # sinh bộ dữ liệu giảng dạy từ schema
│
├─ agents/                       # persona subagent
│  ├─ campaign-strategist.md · content-producer.md
│  ├─ qa-reviewer.md            · insight-analyst.md
│
├─ workflows/                    # pipeline xác định (fan-out có kiểm soát)
│  ├─ campaign-launch.js         # brief → 30 asset draft → QA song song → gate
│  ├─ weekly-measure.js          # collect → model → insight → Telegram
│  └─ course-dataset-build.js    # sinh + validate bộ dữ liệu khoá học
│
├─ scripts/
│  ├─ calendar/   parse_markdown_campaign.py · validate_calendar.py · assign_asset_id.py
│  ├─ publish/    youtube.py · facebook.py          (dry-run MẶC ĐỊNH)
│  ├─ collect/    from_export.py · from_api_youtube.py · from_api_facebook.py · backfill_sidecar.py
│  ├─ model/      build_star.py · build_pbip.py
│  └─ courseware/ generate_kpim_bank.py
│
├─ templates/                    # reference — thứ engine đọc, người dùng không sửa
│  ├─ schema/     dim_campaign.yml · dim_asset.yml · dim_kpi_target.yml
│  │              fact_metrics_daily.yml · fact_lead.yml · dim_pillar.yml
│  │              enums.yml · crosswalk.yml
│  ├─ instance/   khung folder scaffold lúc install
│  ├─ prompts/    playbook theo pillar × định dạng
│  ├─ checklists/ QA 14 mục · approval checklist · risk checklist
│  └─ powerbi/    .pbip mẫu 5 trang + bộ measure
│
├─ content/                      # brief · calendar · draft · approval · published
│  └─ KPIM/                      # ★ instance mẫu DUY NHẤT vào git — khoá MKA-101
├─ data/                         # số liệu đo + star schema + .pbip     (gitignore trừ KPIM)
│  └─ KPIM/
├─ reports/                      # exec summary · báo cáo tuần         (gitignore trừ KPIM)
│  └─ KPIM/
│
└─ docs/plans/2026-07-21-marketing-agent/
```

**Luật `.gitignore` — `KPIM` là instance duy nhất được whitelist ở cả 3 gốc:**

```gitignore
content/*   !content/KPIM/
data/*      !data/KPIM/
reports/*   !reports/KPIM/
```

## 1A. Tiến độ

| Pha | Trạng thái |
|---|---|
| **Pha 0** — bộ khung repo | ✅ xong · `git init`, `.gitignore`, `README`, `.env.example`, `install.ps1` (BOM + syntax OK, chạy thật OK) |
| **1.1** schema canonical | ✅ `templates/schema/model.yml` · `enums.yml` |
| **1.2** crosswalk 4 hệ tên | ✅ `templates/schema/crosswalk.yml` |
| **1.3** pillar A–G + crosswalk | ✅ `<Brain>/ducnguyen-ai/01_brand/pillars.csv` · G thiếu `target_share` (chờ Đức chốt) |
| **1.4** parse 52.02 → calendar | ✅ 31 asset (tài liệu khai 30 — xem qa_findings §C1) |
| **1.5** bổ sung trường thiếu | ✅ 39 cột, gồm `verification_open`, `has_commercial_cta`, `doc_ref` |
| **1.6** validator | ✅ `scripts/calendar/validate_calendar.py` — 0 error, 4 nhóm warning |
| **1.7** đối chiếu đã-đăng-thật | ⏸️ **chặn** — cần Đức xác nhận tuần 1 đã đăng hay chưa (qa_findings §D) |
| **1.8** sửa lỗi dữ liệu 52.02 | ⏸️ **cố ý không tự sửa** — 2/4 heading hỏng không suy được nguyên bản; đã ra punch list |
| **Pha 1B** bộ dữ liệu KPIM | ✅ xong · `generate_kpim_bank.py` deterministic; 8 bảng + 40 bài kế hoạch bổ sung; 60/60 và 29/29 khoá nối verify đạt |

**Gitignore verify (2026-07-21):** 27 file KPIM (1,6 MB) được git theo dõi · **0 file** của instance thật, `instances.json` hay `.env` lọt ra.

## 1C. UAT — 12 bước, chạy trên bộ mẫu KPIM (2026-07-21)

Chạy trên `content/KPIM/02_campaigns/CMP-2026-Q1/CMP-2026-Q1.xlsx` — **không đụng chiến dịch thật**.

| # | Bước | Kỳ vọng | Kết quả |
|---|---|---|---|
| 1 | `status` | Liệt kê trạng thái + cổng + việc theo stage | ✅ 25 asset, 0/25 tick cả 3 cổng |
| 2 | `list --stage draft` khi chưa duyệt | Trả 0 asset, chỉ rõ ai cần tick cột nào | ✅ "5 asset đúng trạng thái nhưng CHƯA tick approve_topic" |
| 3 | `approve --gate approve_topic` | Tick + ghi vòng duyệt vào `05_Approval` | ✅ P-Q1-18 vòng 1 |
| 4 | `list --stage draft` sau khi duyệt | Trả đúng 1 asset | ✅ |
| 5 | `content` ghi 3 mảnh nội dung | Vào `04_Content`, đếm ký tự | ✅ script 1.845 · caption 357 · hook 75 |
| 6 | **Agent tự tick cổng** | **PHẢI bị từ chối** | ✅ exit 1 — "cột của NGƯỜI, script không được ghi" |
| 7 | `set status/verification_open` | Ghi được (cột của agent) | ✅ |
| 8 | **Đăng khi chưa `approve_final`** | **PHẢI chặn** | ✅ exit 1 |
| 9 | **Đăng khi còn `[KIỂM CHỨNG]` mở** | **PHẢI chặn** dù đã duyệt cuối | ✅ exit 1 |
| 10 | Dry-run sau khi đóng kiểm chứng | Chạy được, ghi `06_Publish_Log` | ✅ exit 0 |
| 11 | **`mode=live` khi `autonomy=suggest`** | **PHẢI chặn** | ✅ exit 1 |
| 12 | `status` tổng kết | Phản ánh đúng thay đổi | ✅ approve_topic 1/25 · approve_final 1/25 |

**Nội dung demo sinh ra:** `content/KPIM/02_campaigns/CMP-2026-Q1/drafts/P-Q1-18-{script,caption}.md`
— có `[KIỂM CHỨNG]` chưa đóng và mục QA tự nhận chưa đạt, đúng như `content-write.md` yêu cầu.

**Sửa phát sinh khi UAT:** calendar bộ mẫu KPIM dùng tên cột lệch canonical
(`title`, `platform_primary`, `pillar_code`) → đã chuẩn hoá về `title_draft`, `platform`,
`pillar_primary` ở cả 4 quý.

## 1B. Quyết định đã chốt (2026-07-21)

| Điểm | Quyết định |
|---|---|
| Tên repo | `marketing-agent` |
| Repo chứa gì | engine thuần: skill · agent · workflow · script · template; **một** instance mẫu `KPIM` |
| `content_root` của Duc Nguyen AI | `Brain/50_CONTENT/52_IDEAS/ducnguyen-ai/<campaign_id>/` — theo precedent `51_BRAND/ducnguyen-ai`, subfolder theo brand bên trong category nên **không tốn slot** (50_CONTENT đã đủ 9/9) |
| `data_root` + `report_root` | `marketing-agent/data/<slug>/` và `reports/<slug>/` — **không đẩy lên GitHub**. Người cài repo cũng được tạo vào folder của họ lúc install |
| Nguồn số liệu | làm **cả hai**: file export tay + API |
| Dữ liệu khoá học | giữ nguyên KPIM Bank 60 post, sinh bù 8 bảng thiếu |

## 2. Khung một instance (scaffold từ `templates/instance/`)

```
<instance>/
├─ instance.yml            # tên · kênh · platform id · autonomy · content_root · data_root
├─ 01_brand/               # strategy · pillar · persona · brand voice   (DNA: trỏ Brain 51_BRAND)
├─ 02_campaigns/<campaign_id>/
│  ├─ brief.yml            # objective · big idea · persona · KPI target · ràng buộc tuân thủ
│  ├─ calendar.csv         # ★ 1 dòng = 1 asset
│  ├─ research/            # research brief theo schema 51.01 §8.2
│  └─ drafts/              # nội dung AI sinh: hero · shorts · fb · community
├─ 03_approvals/           # ai duyệt · lúc nào · checklist pass/fail
├─ 04_published/           # platform_native_id · live_url ghi ngược
├─ 05_data/                # raw export tay + fact_metrics_daily      ← data_root
├─ 06_dashboard/           # .pbip
└─ 07_reports/             # exec summary · quyết định 28 ngày
```

`instance.yml` tách **`content_root`** (markdown, có thể nằm trong Brain) khỏi **`data_root`** (CSV cập nhật hằng ngày, nên nằm ngoài Brain để không gây nhiễu git/lint/quarterly review). Mặc định hai cái trùng nhau; instance Duc Nguyen AI tách ra.

## 3. Star schema canonical

| Bảng | Khoá | Vai trò |
|---|---|---|
| `dim_campaign` | `campaign_id` | brief: objective · big idea · persona · funnel goal · ngân sách · ràng buộc |
| `dim_asset` | `asset_id` | **trung tâm** — 1 dòng = 1 asset. FK `campaign_id`, `parent_asset_id`, `research_brief_id` |
| `dim_kpi_target` | (`campaign_id`,`channel`,`pillar`,`metric`,`period`) | target + `baseline_median` + `baseline_source` |
| `fact_metrics_daily` | (`asset_id`,`platform`,`date`) | 17 trường theo 51.01 §12.6 |
| `fact_lead` | `lead_id` | 7 nguồn; FK `campaign_id`, `asset_id` |
| `dim_pillar` | `pillar_code` | A..G canonical + crosswalk sang 1..6 và sang `content_pillar` của KPIM Bank |
| `dim_date`, `dim_channel` | | chuẩn |

**Quy ước `asset_id`** (ổn định, không đổi khi chèn/xoá nội dung):
`{BRAND}-{CAMPAIGN}-W{tuần}-{LOẠI}{số}` → `DNA-C01-W1-HERO`, `DNA-C01-W1-SHORT2`, `DNA-C01-W1-FB3`, `DNA-C01-W1-YTC1`.

## 4. Các pha

### Pha 0 — Bộ khung repo
`git init` · cây thư mục · `.gitignore` · `README.md` · `.env.example` · `install.ps1` hỏi 6 câu ở spec §3.2 và scaffold từ `templates/instance/`.

### Pha 1 — Schema + Content calendar chuẩn hoá ⬅️ *bắt đầu ở đây*

| # | Task | Output |
|---|---|---|
| 1.1 | Chốt `templates/schema/*.yml` + `enums.yml` | schema canonical |
| 1.2 | **`crosswalk.yml`** — ánh xạ 3 hệ tên (`asset_id`↔`post_id`↔`ID bài viết`; `pillar`↔`content_group`↔`Nhãn tùy chỉnh`; `publish_date`↔`planned_date`↔`Thời gian đăng`) | hết xung đột định danh |
| 1.3 | Ánh xạ pillar A..G (51.01) ↔ 1..6 (52.01) | `dim_pillar.csv` |
| 1.4 | `parse_markdown_campaign.py`: 52.02 → `calendar.csv` 30 asset, **tách ô gộp** (`Short 3 + FB post 3`), cấp `asset_id` | calendar máy đọc |
| 1.5 | Bổ sung trường thiếu: `publish_time` · `status` · `approved_by/at` · `tos_score` · `hrs_score` · `live_url` · `platform_native_id` · `cta_type` · `has_commercial_cta` · `parent_asset_id` | đủ để đo |
| 1.6 | `validate_calendar.py` — enum · ngày ISO · hashtag theo kênh (≤3 YT / ≤6 Shorts / 6–13 FB) · CTA thương mại chỉ tập #3 · `[KIỂM CHỨNG]` chưa đóng · **grep tên tool nội bộ = rỗng** | gate chất lượng chạy được |
| 1.7 | Đối chiếu asset **đã đăng thật** (đọc sidecar `.news` + kiểm kênh) → điền `actual_publish_datetime` + native_id | calendar khớp thực tế |
| 1.8 | Sửa 5 lỗi dữ liệu trong 52.02 (hashtag `#AItperary`, 3 heading Unicode hỏng, 6 chỗ `[KIỂM CHỨNG]` tuần 4) | 52.02 sạch |

**DoD Pha 1:** `validate_calendar.py` chạy sạch trên 30 asset; mọi asset có `asset_id` + `status` + `pillar` + `funnel_stage` hợp lệ; crosswalk cover 100% cột của cả 3 hệ.

### Pha 1B — `content/KPIM` (song song, sau 1.2)

| # | Task | Output |
|---|---|---|
| 1B.1 | `generate_kpim_bank.py` — sinh **8 bảng thiếu**, khớp khoá `fb_post_id` 60/60 sẵn có | bộ dữ liệu MKA-101 đầy đủ |
| 1B.2 | `lead_sources` — **7 file rời schema lệch nhau** (app · hotline · form · social inbox · website · chi nhánh · QR) để bài append đa nguồn có nghĩa | gỡ blocker S05 |
| 1B.3 | `kpi_target` theo tháng × kênh × pillar + cột `cost` cấp post | gỡ blocker trang KPI vs Actual |
| 1B.4 | `content_calendar` có `caption` · `media_link` · `schedule_date` · **`approval_status` nhiều trạng thái** (hiện 60/60 đều "Đã đăng" nên IF Approved của n8n không có gì để lọc) | gỡ blocker S02, S03 |
| 1B.5 | `youtube_export` cho 29 video có `yt_video_id` | trang Channel Performance mới so sánh được 2 kênh |
| 1B.6 | `execution_log` · `content_review` · `cta_lead_bridge` | đủ artifact S02/S03/S05 |
| 1B.7 | Campaign mẫu do AI sinh trọn vẹn cho KPIM Bank (brief → 15 bài → duyệt → publish mô phỏng → số liệu → dashboard) | bản sao đầy đủ của quy trình Duc Nguyen AI, bằng dữ liệu mock |
| 1B.8 | Data dictionary + README instructor + khung sample answer | `09_OUTPUTS_SUBMISSION/instructor_only` hết rỗng |

**Nguyên tắc sinh dữ liệu:** có tín hiệu thật để học viên tìm ra (Video > Ảnh về engagement; pillar Promo CTR cao nhưng lead kém; một kênh lệch hẳn), có nhiễu thật để phải làm sạch (ngày lệch định dạng, trùng lead, thiếu `media_link`), **không PII**.

### Pha 2 — Content + approval gate
Sinh nháp AI theo brief (tái dùng `claude -p` headless như `.news`), ghi vào calendar ở `need_review`; máy trạng thái chặn publish nếu chưa `approved`; log ai duyệt/lúc nào.

### Pha 3 — Publish adapter
**Copy** (không sửa) `youtube_upload.py` + `post_facebook.py` từ `.news/engine` vào `scripts/publish/`, thêm: dry-run mặc định, ghi ngược `platform_native_id` + `live_url` vào calendar, kiểm tra mức tự trị. `.news` giữ nguyên bản của nó — 6 task đang chạy không bị đụng.

### Pha 4 — Collect (làm **cả hai đường**)
- **4a `from_export.py`** — đọc YouTube Studio CSV + Meta Business Suite export → `fact_metrics_daily`. Không cần auth. Chính là bài lab S04, đồng thời là đường dự phòng.
- **4b `from_api_*.py`** — YouTube Analytics API (`yt-analytics.readonly`) + FB Page Insights (`read_insights`), chạy theo lịch qua `notify-run.ps1`. Nạp về **cùng schema** với 4a.
- **`backfill_sidecar.py`** — quét `.news/*/daily-out/*/*.published.json` (đã có `video_id`/`short_id`/`fb_post_id`) → có ngay lịch sử để tính baseline median.

### Pha 5 — Model + Dashboard
`.pbip` 5 trang + measure: engagement rate · CTR · CTA rate · **`subs_per_1000_engaged_views` (North Star)** · KPI variance · so-với-median. Dùng `powerbi-mcp` sẵn có. Chưa đủ 10 mẫu/định dạng → hiện "chưa đủ mẫu", **không hiện số giả**.

### Pha 6 — Report + lịch
AI viết exec summary + insight theo bảng chẩn đoán 51.01 §13 (8 cặp triệu chứng→hành động), quyết định 28 ngày, đẩy Telegram qua `notify-run.ps1`.

## 5. Nợ kỹ thuật ghi nhận trước

1. **Baseline median chưa tồn tại** — luật "winner ≥2 KPI vượt median" chỉ tính được sau Pha 4 backfill. Pha 1–3 không bịa số baseline.
2. **Pillar G không có tỷ trọng** trong bảng §14.2 của 51.01 (chỉ A–F = 100%) trong khi 52.02 dùng A+G cho tuần 4 → cần Đức chốt.
3. **`append_excel_log.py:21` hardcode sai đường dẫn Desktop** (`OneDrive\Desktop` trong khi thật là `OneDrive - KPIM Joint Stock Company\Desktop`) → adapter mới resolve động qua `[Environment]::GetFolderPath('Desktop')`.
4. **Cadence lệch tài liệu**: comment nói 2 ngày/lần, Task Scheduler đang chạy hằng ngày cho cả AI và Data.

## 6. Gate

- **Gate 1 (hiện tại)** — duyệt spec + plan trước khi viết code.
- **Gate 2** — trước mọi thao tác outward-facing: bật `full`, đăng thật, push repo public, commit vào kho KPIM.

---

## 2C. TÁI TẠO theo bản chuẩn KPIM 30_MARKETING (2026-07-23)

Model 18-sheet/dim_asset tự nghĩ ở các đợt trước là SAI. Đọc trực tiếp bản chuẩn của KPIM
Academy (`30_MARKETING/agent/AGENTS.md`, `templates/`, `output-styles/tobi-post.md`,
`31_CAMPAIGNS/01_CAMPAIGNS/01_Tobi_Posts/*` + `scripts/PIPELINE_CONTRACT.md`) và tái tạo
đúng y:

- **Mô hình:** 1 campaign = 1 folder + 1 file `.md` (hồ sơ, từ CAMPAIGN_TEMPLATE 15 mục) +
  1 file `.xlsx` **5 sheet**: Campaign / Post / Result / Engagement / Assets.
- **3 cổng duyệt là CỘT trong sheet Post** (approve_topic/content/final), không phải sheet riêng.
- **5 stage:** topics → draft → media → atlas → publish, gating y hệt PIPELINE_CONTRACT.
- **Bê nguyên:** CAMPAIGN_TEMPLATE.md, CONTENT_TEMPLATE.md, tobi-post.md, compa-class-blog.md,
  multichannel-style.md, EMAIL_NEWSLETTER, RECYCLING_PLAN. Viết AGENTS.md cho engine.
- **Bộ mẫu:** dựng lại `01_Tobi_Posts` (hồ sơ + workbook 5 sheet + asset TOBI-001) — UAT vòng
  đời đầy đủ qua 3 cổng đạt.

**Xoá:** 4 workbook 18-sheet Q1–Q4 + campaigns cũ; schema/{model,enums,crosswalk}.yml;
scripts validate_calendar/build_summary/index_assets/new_campaign/from_export/generate_kpim_bank;
templates yml cũ; knowledge BRAND_VOICE/MULTICHANNEL_MATRIX/SCORING/DIAGNOSTICS (trùng
output-styles hoặc tham chiếu schema chết); toàn bộ data/KPIM bank cũ.
