---
name: trip-map-builder
description: >
  End-to-end trip planning: gather user constraints, build a reference
  itinerary, research locations and dining signals via 大众点评 + 小红书 (China)
  or Google Maps + 小红书 (Overseas), then generate an interactive mobile-first
  map page (Leaflet + timeline) and optionally deploy to Vercel. Auto-detects
  destination type and switches research strategy. Use when user asks to plan a
  trip, create an itinerary, research restaurants, build a trip map page, or
  says "行程规划", "行程地图", "trip map", "plan my trip", "做个行程". Covers the
  full pipeline from scattered inputs to a deployable reference map.
---

# Trip Map Builder

Pipeline: **Plan → Detect → Hotel → Restaurant → Build**.

The output is a **reference itinerary**, not a script the traveler must obey.
During the trip, weather, current location, fatigue, and hunger can override
the original plan.

## Shared memory

Before planning or building, read `~/.trip-map-builder/MEMORY.md` if it
exists. Use it only for durable traveler context:

- pace preference
- food and drink preferences
- budget habits
- payment and navigation preferences
- previously generated trip outputs
- recurring constraints and unresolved follow-ups

If the file does not exist, continue normally. Do not block on memory setup.

Do not store raw screenshots, passport data, booking codes, full chat logs, or
other sensitive/private material.

After each completed trip plan, research pass, or map build, update
`~/.trip-map-builder/MEMORY.md` with only durable facts:

```md
# Trip Map Builder Memory

## Traveler Defaults
- Departure city:
- Pace:
- Food preferences:
- Hotel budget: ¥{min}-{max}/night (domestic) / ${min}-{max}/night (overseas)
- Hotel type preference: hotel / serviced apartment / Airbnb / hostel
- Budget habits:
- Payment preference:
- Navigation preference:
- Language preference:

## Past Trips
| Trip | Dates | Destination | Output | Notes |
|------|-------|-------------|--------|-------|

## Reusable Preferences
-

## Open Threads
-
```

## Phase 1: Plan the itinerary

Read `references/trip-planning.md` for the full methodology.

Core sequence:

1. **Extract hard constraints** — dates, flight times, terminals, hotel location
2. **Group user's wishlist** — city-easy / needs-reservation / far-suburbs / pass-through
3. **Cut high-risk items first** — too far, holiday-crowded, weather-dependent. Say what was cut and why.
4. **Arrange by area** — one main area per day, first day light, last day close to airport
5. **Fill in meals** — daily-area candidates first, 大众点评 + 小红书 signals second, fame last.
6. **Add tickets & transport** — only critical ones (museum tickets, airport transfer)
7. **Write reference doc** — conclusion first, then daily plan, weather-sensitive spots, meal areas, and what was cut

Key principles:
- Not everything the user listed fits. Delete for them.
- One area per day. One reservation-required spot per day max.
- Itineraries should be smooth, not packed.
- The plan gives coordinates for later adjustment; it does not pretend reality will follow the timeline.
- All user-facing questions follow the **4-beat format**: Re-ground → Simplify → Recommend → Options. See `references/trip-planning.md` § 用户交互 for examples and anti-patterns.

## Destination Detection (between Phase 1 and Phase 2)

After the itinerary plan is confirmed, detect the destination type:

| Destination | Trigger examples | Research strategy |
|-------------|------------------|-------------------|
| **China** (中国大陆) | 上海、北京、成都、杭州... | 大众点评 + 小红书（Phase 2A） |
| **Overseas** (海外) | 东京、巴黎、曼谷、纽约、港澳台... | Google Maps + 小红书（Phase 2B） |

Detection logic:
1. Check if destination is in mainland China → China track
2. Otherwise (Hong Kong, Macau, Taiwan, Japan, Korea, Southeast Asia, Europe, Americas, Oceania, etc.) → Overseas track
3. If uncertain (border cities, ambiguous names), ask user for clarification

---

## Phase 2: Hotel Research (cross-platform price comparison)

Read `references/hotel-research.md` for the full methodology.

Hotel research runs **immediately after confirming accommodation areas** in Phase 1,
before restaurant research. Hotel location anchors the daily route — restaurants
are chosen within walking/dining distance of the hotel area.

### China track

| Platform | Search entry |
|----------|-------------|
| 携程 (Ctrip) | `hotels.ctrip.com` |
| 飞猪 (Fliggy) | `hotel.fliggy.com` |
| 去哪儿 (Qunar) | `hotel.qunar.com` |

Strategy: search all three platforms for the same hotel + same dates, pick the
lowest price. Also note: 飞猪 credit-waiver deposit, 88VIP discounts, Alipay
points redemption may create hidden advantages.

### Overseas track

| Platform | Search entry |
|----------|-------------|
| Booking.com | `booking.com` |
| Agoda | `agoda.com` |
| Airbnb | `airbnb.com` |
| Trip.com (Ctrip international) | `trip.com` |

Strategy: search all four platforms. Agoda often beats Booking in Asia-Pacific.
Airbnb wins for groups ≥4 or stays ≥3 nights (full apartment, kitchen, laundry).
Trip.com sometimes beats Booking when paying in CNY.

### Output format

For each accommodation area, output 3 candidates:

```md
### 主推：Hotel Name
- 携程 ¥680 · 飞猪 ¥650 · 去哪儿 ¥620 → ✅ 最优：去哪儿 ¥620
- 评分 4.5（2000+ reviews）· 免费取消 · 含早
- 位置：XX站步行5分钟
- 评价要点：{2-3 sentence verdict}
- 适合/不适合：{scenario fit}
```

### Scenario routing

| Travel scenario | Primary platform | Room type priority |
|-----------------|-----------------|-------------------|
| 带爸妈 | 携程 / Booking | 双床房，含早，有电梯 |
| 情侣/蜜月 | 小红书→Booking 验证 | 大床房，景观好 |
| 多人(≥4) / 长住(≥3晚) | Airbnb | 整租公寓，厨房洗衣机 |
| 穷游/背包 | 去哪儿 / Agoda | 青旅/胶囊 |
| 商务出差 | 携程企业版 | 大床房，近会议地点 |
| 日本温泉旅馆 | Jalan / 一休 | 和室+一泊二食 |

---

## Phase 3A: Restaurant Research via 大众点评 + 小红书 (China)

Read `references/dianping-research.md` for the 大众点评 OpenCLI workflow.
Read `references/xhs-research.md` for the 小红书 OpenCLI + CDP workflow.

For restaurants, use 大众点评 as the main Chinese dining signal for taste,
queue risk, value, and obvious traps. Use 小红书 to supplement atmosphere,
recent experience, photo-worthiness, and soft warnings. Do not bend a whole
day around a famous restaurant unless it is already on the route.

小红书 core sequence:

1. Launch Chrome with `--remote-debugging-port=9223`
2. Connect via OpenCLI's `CDPBridge`
3. Navigate to `xiaohongshu.com/search_result?keyword=<encoded>` (never simulate input box)
4. Intercept `POST /api/sns/web/v1/search/notes` response
5. Pick top 2-3 notes by relevance, open detail pages
6. Extract via DOM: `#detail-title`, `#detail-desc`, `.author-container .username`
7. Compress to one decision-useful sentence per store, write back to local `.md`

Filtering rules:
- Keep: specific store name, address, dish, personal experience, repeated keywords
- Drop: generic area roundups, reposts, pure emotion, "氛围很好" x3
- Output: store name + one representative link + 2-3 sentence verdict

---

## Phase 3B: Restaurant Research via Google Maps + 小红书 (Overseas)

Read `references/overseas-research.md` for the full methodology.

**Key difference from China track**: 大众点评 is unreliable overseas. Use
Google Maps as the primary hard-signal source (ratings, review volume, price
level, opening hours, popular times, photos). 小红书 still provides the soft
layer (Chinese traveler experience, atmosphere, warnings).

Sequence:
1. **小红书 first** — search itinerary-level content ("{city} 3天2夜 行程"), extract restaurant candidates and area recommendations. Use same CDP workflow as Phase 2A.
2. **Google Maps verify** — for each candidate restaurant, check rating, review count, price level, opening hours, and recent reviews. Filter out weak signals.
3. **Write back** — each meal: 2-3 candidates with GMaps hard data + 小红书 soft notes.

Google Maps data points to collect per restaurant:
- Rating (1-5) and total review count
- Price level ($-$$$$)
- Opening hours (note: lunch breaks in Japan, late dining in Spain, etc.)
- Popular times (avoid peak queues)
- Recent reviews (sort by newest, watch for repeated complaints)

Navigation links in the final map page should use Google Maps for overseas
destinations (高德/百度 maps have limited overseas coverage):
```
https://www.google.com/maps/dir/?api=1&destination={lat},{lng}&travelmode={mode}
```

Additional overseas considerations to include in trip output:
- Payment methods (local + Alipay/WeChat coverage)
- Language tips
- Tipping culture (US 15-20%, Japan/China none, etc.)
- Reservation culture (Japan: book ahead for good restaurants)
- Local dining time norms (Spain dinner at 9pm, etc.)

## Phase 4: Build the map page

1. Copy `assets/template.html` → `index.html`
2. Fill `HOTEL` object and `DAYS` array with structured data from Phase 1+2
3. Each location needs: name, lat/lng, type, time, desc; optional: budget, detail, pay, xhs, reserve, gmap, airbnb
4. Fill `overviewContent()` with trip summary, payment warnings
5. Apply design system — default template uses Apple style, but can switch to any style from awesome-design-md

Location types: `food` | `spot` | `drink` | `hotel` | `transport`

Payment chip values: `1` = confirmed yes (green), `0.5` = maybe (orange), omit = not shown

### Hotel booking links (`airbnb` field)

For locations with `type: 'hotel'`, add an `airbnb` field with a pre-filled search URL.
The template will render a **🏠 订房** button (Airbnb red `#FF385C`) in the location card's
navigation bar, next to the 🧭 导航 and 📅 预约 buttons.

```js
{
  name: '基督城 Airbnb', type: 'hotel',
  airbnb: 'https://www.airbnb.com/s/Christchurch-New-Zealand/homes?checkin=2026-09-29&checkout=2026-09-30&adults=3',
  gmap: 'Merivale+Christchurch+Airbnb'
}
```

URL format for Airbnb search links:
```
https://www.airbnb.com/s/{city}-{country}/homes?checkin={YYYY-MM-DD}&checkout={YYYY-MM-DD}&adults={N}
```

The template also renders a **`宿`** badge (orange) on hotel-type cards for quick visual identification in the timeline.

### Design system (optional)

Default template uses Apple design system (SF Pro, light theme, frosted glass).

To use a different style, grab a `DESIGN.md` from [awesome-design-md](https://github.com/VoltAgent/awesome-design-md):

```bash
# Browse available design systems
# Apple, Vercel, Linear, Stripe, Notion, Airbnb, Nike, Spotify, etc.
curl -O https://raw.githubusercontent.com/VoltAgent/awesome-design-md/main/design-md/<brand>/DESIGN.md
```

Then adjust `template.html`'s `:root` CSS variables (colors, fonts, spacing, border-radius) to match the chosen DESIGN.md tokens.

### Deploy (optional)

```bash
git init && git add . && git commit -m "trip map"
gh repo create REPO --public --source=. --push
# Import from vercel.com/new — auto-deploys on push
```

## Dependencies

| Tool | Purpose | Install |
|------|---------|---------|
| [OpenCLI](https://github.com/jackwener/OpenCLI) | 大众点评 adapter + 小红书调研 | `npm install -g @jackwener/opencli` |
| Chrome/Chromium | 浏览器 + 远程调试 | 已有 |
| [Leaflet.js](https://leafletjs.com) | 地图渲染（本地 `lib/` 目录，非 CDN） | 下载到 `lib/leaflet.css` + `lib/leaflet.js` |
| [gh CLI](https://cli.github.com) | GitHub 仓库创建（可选） | `brew install gh` |

## Resources

- `references/trip-planning.md` — itinerary planning methodology, input/output templates, selection principles, common pitfalls
- `references/hotel-research.md` — cross-platform hotel price comparison (携程/飞猪/去哪儿 for China, Booking/Agoda/Airbnb for overseas), scenario routing, writeback format
- `references/dianping-research.md` — 大众点评 OpenCLI search/shop workflow, dining decision signals, writeback format (China destinations)
- `references/xhs-research.md` — OpenCLI installation, Chrome CDP setup, 小红书 search workflow, API details, filtering criteria
- `references/overseas-research.md` — Google Maps + 小红书 workflow for non-China destinations, rating interpretation, navigation links
- `assets/template.html` — single-file HTML map template (Leaflet + Apple design system)
- [awesome-design-md](https://github.com/VoltAgent/awesome-design-md) — 60+ brand design systems (Apple, Vercel, Stripe, Linear, etc.) for alternative styling

## 常见问题排查

生成 `index.html` 后本地预览空白或报错时，按以下顺序排查：

### 1. CSS 变量失效 → 页面空白

**症状**：页面完全空白，F12 控制台无报错。

**原因**：`<style>` 中写了 `::root {` 而非 `:root {`（双冒号是伪元素语法，CSS 变量定义只在 `:root` 上生效）。

**修复**：搜索 `::root` 替换为 `:root`。

### 2. Leaflet CDN 被墙 → 地图不加载

**症状**：F12 Network 显示 `blocked:origin` 或 `net::ERR_CONNECTION_TIMED_OUT`。

**原因**：`unpkg.com` 在中国大陆被墙。

**修复**：不要从 CDN 引用 Leaflet。将 `leaflet.css` 和 `leaflet.js` 下载到 `lib/` 目录，引用本地文件：

```bash
# 下载 Leaflet 到本地
mkdir -p lib
curl -o lib/leaflet.css https://unpkg.com/leaflet@1.9.4/dist/leaflet.css
curl -o lib/leaflet.js  https://unpkg.com/leaflet@1.9.4/dist/leaflet.js
```

HTML 引用改为：
```html
<link rel="stylesheet" href="lib/leaflet.css" />
<script src="lib/leaflet.js"></script>
```

模板 `assets/template.html` 已内置注释说明此流程，无需手动改。

### 3. 单引号截断 JS 字符串 → SyntaxError

**症状**：`Uncaught SyntaxError: Unexpected identifier` 指向某个位置数据行。

**原因**：地点名称中包含 ASCII 单引号（如 `Peter's Lookout`），在 JS 字符串字面量中未转义，导致字符串提前终止。

**修复**：将数据中的 ASCII 单引号 `'` 替换为 `\'` 转义：
```js
// ❌ 错误
desc: 'Peter's Lookout is a viewpoint...'
// ✅ 正确
desc: 'Peter\\'s Lookout is a viewpoint...'
```

**预防**：生成数据时对所有字符串字段做单引号转义检查。
