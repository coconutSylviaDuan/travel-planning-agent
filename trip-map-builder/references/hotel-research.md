# 酒店比价调研工作流

## 前提

酒店是出行开销大头，同一酒店在不同平台价差可达 20-30%。本工作流的目标是：
在已确定住宿区域后，跨平台比价找到最优可预订选项，而非"推荐一家好酒店"。

**核心原则**：先定区域，再比价格。不要让酒店位置绑架行程路线。

---

## 平台策略

### 国内目的地

| 平台 | 定位 | 优势 | 搜索入口 |
|------|------|------|----------|
| **携程** | 全品类 OTA 老大 | 酒店库存最全、评价量最大、企业协议价 | `hotels.ctrip.com` |
| **飞猪** | 阿里系 OTA | 信用住免押金、88VIP 折扣、支付宝积分抵扣 | `hotel.fliggy.com` |
| **去哪儿** | 性价比导向 | 价格通常最低、比价功能强、学生/年轻用户多 | `hotel.qunar.com` |

### 海外目的地

| 平台 | 定位 | 优势 | 搜索入口 |
|------|------|------|----------|
| **Booking.com** | 全球最大 OTA | 库存最全、免费取消多、Genius 会员折扣 | `booking.com` |
| **Agoda** | 亚太区王者 | 日韩东南亚价格常有优势、会员专属价 | `agoda.com` |
| **Airbnb** | 民宿/公寓 | 适合多人出行、长住、有厨房需求 | `airbnb.com` |

### 补充平台（特殊场景）

| 平台 | 场景 |
|------|------|
| **Trip.com**（携程国际版） | 海外酒店用中文界面搜，有时比 Booking 便宜 |
| **Klook** | 日本/东南亚酒店+门票套餐 |
| **Jalan / 一休** | 日本本地平台，日式旅馆/温泉酒店库存更全 |
| **Hotels.com** | 住10送1，适合高频出行 |

---

## 搜索策略

### 第一步：确定搜索条件

从行程规划中提取：

```
- 入住/退房日期：YYYY-MM-DD ~ YYYY-MM-DD
- 住宿区域：{具体区域/地铁站}（如"新宿站步行5分钟"、"外滩附近"）
- 人数：{成人}人，{儿童}人（如有）
- 预算范围：¥{min}-{max}/晚
- 房型偏好：大床房/双床房/套房/整租公寓
- 特殊需求：早餐、停车场、无障碍设施、允许吸烟等
```

### 第二步：构造搜索 URL

各平台搜索 URL 格式：

```bash
# 携程
https://hotels.ctrip.com/hotel/{city-pinyin}?checkin={date}&checkout={date}

# 飞猪
https://hotel.fliggy.com/search.htm?city={cityCode}&checkIn={date}&checkOut={date}

# 去哪儿
https://hotel.qunar.com/city/{city-pinyin}/?fromDate={date}&toDate={date}

# Booking.com
https://www.booking.com/searchresults.html?ss={city}&checkin={date}&checkout={date}&group_adults={n}

# Agoda
https://www.agoda.com/search?city={cityId}&checkIn={date}&checkOut={date}&adults={n}

# Airbnb
https://www.airbnb.com/s/{location}/homes?checkin={date}&checkout={date}&adults={n}
```

### 第三步：筛选排序

所有平台统一筛序：

```
排序：按价格从低到高（或按评分+价格综合排序）
筛选：
  - 评分 ≥ 8.0（Booking/Agoda）、≥ 4.0（携程/飞猪/去哪儿）
  - 评价量 ≥ 100（信号可靠度）
  - 可免费取消优先
  - 位置在目标区域内（看地图确认）
  - 排除：评分高但评价量 < 30 的（可能是刷的）
```

### 第四步：比价输出

挑出 3-5 个符合条件的酒店，逐一到其他平台查同酒店同日期价格：

| 酒店名称 | 携程 | 飞猪 | 去哪儿 | Booking | Agoda | Airbnb | 最优 |
|----------|------|------|--------|---------|-------|--------|------|
| 酒店A | ¥680 | ¥650 | ¥620 | — | — | — | **去哪儿 ¥620** |
| 酒店B | — | — | — | ¥850 | ¥820 | — | **Agoda ¥820** |

**国内版本**：携程 → 飞猪 → 去哪儿（三平台交叉比价）
**海外版本**：Booking → Agoda → Airbnb → Trip.com（四平台交叉比价）

---

## 判断维度

### 硬指标

| 维度 | 数据来源 | 判断标准 |
|------|----------|----------|
| 价格 | 各平台搜索结果 | 在同区域同档位中是否有竞争力 |
| 位置 | 地图坐标 | 到最近地铁站/景点步行时间 ≤ 10分钟（城市酒店） |
| 评分 | 用户评价 | Booking/Agoda ≥ 8.0；携程/飞猪 ≥ 4.0 |
| 评价量 | 用户评价总数 | ≥ 100 才可靠，≥ 500 信号强 |
| 取消政策 | 预订条款 | 免费取消优先，至少入住前 24h 可退 |
| 含早 | 房型详情 | 是否含早、早餐价格、周边早餐便利度 |

### 软指标

| 维度 | 数据来源 | 关注点 |
|------|----------|--------|
| 真实体验 | 小红书 / 评价 | 房间大小、隔音、热水、空调、电梯、前台服务 |
| 近期口碑 | 各平台排序"最新" | 最近 3 个月的差评内容（装修噪音、服务变差等） |
| 卫生 | 差评关键词 | 虫子、异味、床单不干净 |
| 周边配套 | Google Maps / 高德 | 便利店、药店、餐厅密度 |
| 安全 | 评价 + 街区信息 | 晚上回酒店的路是否安全（女生独自出行尤其重要） |

### 特殊场景判断

| 场景 | 优先平台 | 优先房型 |
|------|----------|----------|
| 带爸妈出行 | 携程/Booking（库存稳） | 双床房，含早，有电梯 |
| 情侣/蜜月 | 小红书种草 + Booking 验证 | 大床房，风景好，氛围感 |
| 多人/家庭（≥4人） | Airbnb（整租公寓） | 两居室以上，有厨房和洗衣机 |
| 穷游/背包 | 去哪儿/Agoda | 青旅床位或胶囊 |
| 商务出差 | 携程企业版/Booking Business | 大床房，近会议地点，含发票 |
| 日本温泉旅馆 | Jalan / 一休 | 和室+半食宿（一泊二食） |

---

## 实战流程（整合到行程规划中）

酒店调研在 **Plan 阶段确认住宿区域后**立即执行，在餐厅调研之前：

```
Plan（确认区域）
    ↓
Hotel Research（本文件）
    - 国内：携程/飞猪/去哪儿三平台比价
    - 海外：Booking/Agoda/Airbnb/Trip.com 四平台比价
    ↓
Restaurant Research（大众点评 or Google Maps）
    ↓
Build（生成地图）
```

### 输出格式

每区域输出 3 个候选酒店，标注最优平台和价格：

```md
## 住宿区域：新宿（东京）

### 主推：格拉斯丽新宿酒店（Hotel Gracery Shinjuku）
- Booking：¥1,280/晚 · Agoda：¥1,220/晚 · Airbnb：无 · Trip.com：¥1,300/晚
- ✅ 最优：**Agoda ¥1,220**（会员价，免费取消，入住前1天可退）
- 评分：8.6（5000+ reviews）
- 位置：新宿站东口步行3分钟，歌舞伎町入口
- 评价要点：房间偏小但位置无敌，高层景观好，前台有中文服务
- 适合：情侣/单人，行李少
- 不适合：带大箱子、需要大空间

### 备选1：JR九州酒店Blossom新宿
- Booking：¥980/晚 · Agoda：¥950/晚 · Trip.com：¥1,000/晚
- ✅ 最优：**Agoda ¥950**（免费取消）
- 评分：9.0（3000+ reviews）
- 位置：新宿站南口步行4分钟，安静街区
- 评价要点：房间比Gracery大，设计感强，安静，早餐优秀
- 适合：商务出行、带爸妈、需要安静环境

### 备选2：Airbnb 新宿区整租公寓
- Airbnb：¥680/晚（2人入住，整租1DK）
- 位置：新宿三丁目步行8分钟
- 评价要点：空间大，有厨房洗衣机，适合长住
- 适合：住 3 天以上、需要做饭洗衣
```

---

## 浏览器自动化（可选）

如果手动比价效率低，可走 CDP 方案：

### 国内平台

```bash
# 携程 — 直接用搜索 URL，不走输入框
Page.navigate → hotels.ctrip.com/hotel/shanghai2?checkin=2026-07-10&checkout=2026-07-12

# 提取酒店卡片
document.querySelectorAll('.hotel-item').forEach(card => {
  name: card.querySelector('.hotel-name')?.innerText,
  price: card.querySelector('.price')?.innerText,
  rating: card.querySelector('.rating-score')?.innerText,
  reviewCount: card.querySelector('.review-count')?.innerText
})
```

### 海外平台

```bash
# Booking.com
Page.navigate → booking.com/searchresults.html?ss=Tokyo&checkin=2026-07-10&checkout=2026-07-12&group_adults=2

# 提取
document.querySelectorAll('[data-testid="property-card"]').forEach(card => {
  name: card.querySelector('[data-testid="title"]')?.innerText,
  price: card.querySelector('[data-testid="price-and-discounted-price"]')?.innerText,
  rating: card.querySelector('[data-testid="review-score"]')?.innerText?.split(' ')[0],
  reviewCount: card.querySelector('[data-testid="review-score"]')?.innerText?.split(' ')[2]
})
```

---

## 常见坑

- **"评分最高 = 最值得住"** ── 9.5 分但只有 20 条评价，大概率刷的。优先评价量 ≥ 100。
- **"Booking 价格最低"** ── 不一定。Agoda 在亚太常有专属折扣，Airbnb 多人分摊可能更便宜。
- **"只看价格不看取消政策"** ── 不可取消的便宜酒店如果行程变动就是纯亏。
- **"位置看文字不看地图"** ── "步行5分钟到地铁"可能是直线距离，实际要绕一大圈。
- **"国外酒店也上携程搜"** ── 携程国际版 Trip.com 在海外可能有优势，但 Booking 库存更全。
- **"Airbnb 只看总价"** ── 清洁费+服务费可能让标价翻倍，一定要看到"总价"再比。
- **"不看差评只看好评"** ── 好评千篇一律，差评才是那个酒店真正的下限。
