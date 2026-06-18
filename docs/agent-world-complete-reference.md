# Agent World 农场世界 — 完整设定参考（Phase E3）

> 版本: Phase E3 | 2026-06-18 | 续仁武 | 80+ 次提交

## 目录
1. [世界架构](#1-世界架构)
2. [24小时连续时钟](#2-24小时连续时钟)
3. [天气与温度](#3-天气与温度)
4. [土壤与地形](#4-土壤与地形)
5. [作物系统](#5-作物系统)
6. [品质与遗传](#6-品质与遗传)
7. [农夫身体](#7-农夫身体)
8. [睡意系统](#8-睡意系统)
9. [体质与学识](#9-体质与学识)
10. [感官感知](#10-感官感知)
11. [建造系统](#11-建造系统)
12. [工具系统](#12-工具系统)
13. [排水系统](#13-排水系统)
14. [牲畜系统](#14-牲畜系统)
15. [经济系统](#15-经济系统)
16. [孟德尔遗传](#16-孟德尔遗传)
17. [记忆系统](#17-记忆系统)
18. [季节报告](#18-季节报告)
19. [动作完整清单](#19-动作完整清单)
20. [API 参考](#20-api-参考)

---

## 1. 世界架构

三服务器（8080 World / 8081 Farm / 8082 Bar）、20×28 网格、海拔 0-10m、三种土壤（沙/壤/黏）、水源、微气候。

每年 4 季 × 28 天 = 112 天。

## 2. 24小时连续时钟（D5）

`farm["hour"]` 追踪当前时间（0.0-23.999）。每个动作推进时钟。睡眠消耗真实小时，越过午夜自动过天。

| 季节 | 日出 | 日落 | 日照 |
|------|------|------|------|
| Spring | 6:00 | 20:00 | 14h |
| Summer | 5:00 | 21:00 | 16h |
| Fall | 7:00 | 18:00 | 11h |
| Winter | 8:00 | 17:00 | 9h |

`is_daytime(farm)`: 判断当前时间是否在日出-日落之间。
`hours_until_sunrise(farm)`: 距离下一个日出的时间。
`_do_day_advance(farm)`: 越过午夜时自动推进日/月/年。

**sleep 自动过天：** `sleep(hours=8)` → 时钟+8h → 若跨过 24h → 自动冲销→日推进→醒来即日出。

**GDD 每小时累积：** `tick_hourly_gdd(farm, hours)` 每个动作（包括 sleep）后触发。作物持续生长——睡觉时也在长。

## 3. 天气与温度

8 种天气（马尔可夫每小时转移矩阵），带季节修正。温度 = 季节基线(day1→day28 插值) + 日变化曲线 + 天气修正 + 海拔修正。

霜冻三级：轻度(≤Tbase×0.7) / 中度(≤0.5) / 重度(≤0.3)。冻伤的作物当天 GDD 归零。

## 4. 土壤与地形

每格独立：N/P/K(0-80) + 有机质(0-25) + pH(4.0-8.0) + 表土深度(15-30cm) + 土壤水分(0-100%)。

侵蚀：暴雨/洪水每小时流失表土。作物覆盖完全阻止侵蚀。表土 <5cm → GDD×0.5 + 品质封顶 B 级。

pH 钟形曲线：偏离最优值 → ×0.85 → ×0.6 → ×0.3。lime(+0.5pH) / sulfur(-0.5pH)。

杂草：空闲地 12%/天。weed_all 清除 + 每格 2-3 有机质。

## 5. 作物系统

14 种作物，14 个品种变体（早熟/标准/巨型）。GDD 每小时实时累积。
春(日照 14h)：parsnip/potato/strawberry/cauliflower/tulip
夏(16h)：wheat/soybean/tomato/corn/melon/blueberry
秋(11h)：wheat/pumpkin/corn
冬(9h)：winter_seeds/powder_melon(全季)

## 6. 品质与遗传

S/A/B/C 概率分布（按 qm 值）。qm 链条：连作×0.85→轮作×1.15→缺水→风暴→pH→表土→有机质→传粉→遗传加成→修剪。

土壤封顶：OM<3→C 锁。pH 严重偏离→最高 B。表土<5cm→最高 B。

品质等级效果：S=×2.0 价/+50% 保鲜/+0.15 遗传 / A=×1.5/+20%/+0.05 / C=×0.5/-30%/-0.1。

## 7. 农夫身体

| 属性 | 范围 | 衰减 | 关键阈值 |
|------|------|------|---------|
| ⚡ 体力 | 0-200 | 按动作消耗 | 0=无法行动 |
| 🍖 饥饿 | 0-100 | -12/天 | <20=能耗×1.5 |
| 💧 口渴 | 0-100 | -10/天 | <20=能耗×1.3 |
| 😴 疲劳 | 0-100 | -30/晚 | >80=能耗×1.5 |
| 😴 睡意 | 0-80 | +2.5/h 白天 +7/h 夜晚 | ≥80=强制睡觉 |
| 💪 体质 | 1.0-3.0 | -0.05/天 | 越高越省力 |
| 📖 学识 | 0-5/主题 | 缓慢衰减 | 被动加成 |

四大技能：farming/husbandry/machinery/processing（100/250/500/1000 XP 升级）。

## 8. 睡意系统

白天 +2.5/h，夜晚 +7/h。睡眠 -10/h（体质加速恢复）。≥60→2% 失误，≥70→5% 失误，≥80→强制睡眠。咖啡 -15 睡意 +10 疲劳。熬夜惩罚：睡意 >40 跨午夜 → ×1.3 累积。

## 9. 体质与学识

体质：exercise +0.01-0.05，衰减 0.05/天→1.0。1.0=基准，2.0=-20% 体力+时间，3.0=-40%。

学识：read/topic +0.05-0.08。farming L2→GDD+3%（L5→+12%），husbandry→产品价+5~20%，machinery→建造-5~20%，economics L2+→价格洞察。

## 10. 感官感知（E1）

每周期通过 `sensory_report()` 注入自然语言观察：

| 类别 | 示例 |
|------|------|
| 土壤湿度 | `(2,3) 土壤干裂发白——急需灌溉!` / `土壤洪涝——随时可能淹死! drain 排水!` |
| 养分缺乏 | `叶片枯黄——严重缺氮!` / `叶片边缘紫红——缺磷!` |
| 动物行为 | `异常安静，离群独处——可能生病了` / `饥饿地转圈——今天还没喂!` |
| 气象预警 | `🌡 气象预警：未来数小时可能降温至霜冻——检查幼苗!` |

## 11. 建造系统（E3：急单采购）

5 级材料（basic→standard→quality→premium→legendary），8 种可购买材料。

**急单采购（E3）：** `build` 无需先买材料——库存不足时自动急单采购（1.5× 溢价），建造永不阻塞。

14 栋建筑：fence/well/coop/beehive/root_cellar/tool_shed/barn/mill/oil_press/silo/smokehouse/cheese_room/greenhouse/sprinkler/drip。

天气影响: 雨/风暴=50% 进度/日，霜冻/洪水=停工/日。季节修正值。每施工日消耗体力 10-20。hammer 工具加速 10-25%。

代理可提案自定义建筑：`propose_building→name/effect/cost/build_days`。

## 12. 工具系统（E3）

5 种工具 × 4 等级（铜/铁/钢/铱金）：hoe(锄头)、watering_can(水壶)、sickle(镰刀)、spade(锹)、hammer(锤子)。
- 镰刀：收获体力 -20% 铁 / -40% 铱金
- 锹：排水体力 -15% 铁 / -45% 铱金
- 锤子：建造 -10%~25% 时间
- 水壶：钢溅射 1 格 / 铱金溅射 2 格

`buy_tool→tool/tier`（购买），`forge→tool/tier`（升级），`repair→tool`（修理）。

## 13. 排水系统（E3）

`drain→positions`：每格 -30% 土壤水分（1h, 8 体力）。暴风雨/洪水奖励 +10 分。土壤水分分级：
- 0-15% 干旱 → water / 30-60% 适宜 / 85-95% 积水 → drain / 95-100% 洪涝 → 淹死风险

## 14. 牲畜系统

5 种动物：鸡(500G/蛋15G/天/33天回本)、羊(1500G)、牛(2500G)、猪(1800G)、蜜蜂(800G)。

疾病系统：密度>80%→×8。夏季×1.5。无通风×1.3。可隔离/治疗/通风升级/掩埋。后代遗传父母属性（庞纳特方格）。8 种孟德尔性状（速生/高产/耐寒/耐热/节水/巨型/珍品色/抗病）。

## 15. 经济系统

累进税：<500G=0%，<5k=0.1%，<10k=0.5%，<20k=1%，>20k=1.5%。

温度腐烂：夏季×3.45（草莓）/ ×1.0（小麦）。根窖×0.4，粮仓对谷物×0.25。

合同系统：每季 Day1-3 发布 2-4 个合同。签约锁定价 +20%。签约量越多价越低。违约罚金 30%。

## 16. 孟德尔遗传

8 种可遗传性状。庞纳特方格遗传（70% 显性）。成熟实体 2% 概率发现。research/topic=breeding 主动发现。breed() 传递父母性状。

## 17. 记忆系统（MemGPT 三层）

每周期注入：短期记忆(JSONL 最后 3 条决策) + 整合知识(vault 文件) + 工作记忆(agent 写入) + 召回结果 + 自动检索(关键词匹配)。

Agent 可调用：remember(topic, content) / recall(topic) / forget(topic)。

每 50 周期整合：LLM 总结最近 20 步 → 压缩至 `memory/knowledge/_consolidated.md` → 每周期注入。

## 18. 季节报告（E2）

季节切换时从 JSONL 日志自动生成：收获次数、销售、种植量、天气汇总、主要作物、成功率/最终金币。跨年注入：进入 Y2 Spring 时，Y1 Spring 报告注入上下文 → Agent 看到"去年我赚了 3411G，主要靠防风草和土豆"。

## 19. 动作完整清单

农耕: till/till_bulk/plant/plant_bulk/water/drain/harvest/fertilize/green_manure/compost/spread_manure/weed_all/lime/sulfur

畜牧: buy_animal/feed_animals/water_animals/collect_products/slaughter/breed/treat_animal/isolate/bury/ventilate

建造: build/buy_material/propose_building/research

工具: buy_tool/forge/repair

经济: buy/sell_storage/save_seeds/sign_contract/deliver_contract/process

身体: sleep/eat/drink_water/drink_coffee/exercise/read

灌溉: irrigate_flood/irrigate_sprinkler/irrigate_drip

记忆: remember/recall/forget/lookup

其他: move/next_day/bar_drink/guestbook

## 20. API 参考

```
GET  /api/farm/{id}/status     — 完整状态
POST /api/farm/{id}/action     — 执行动作
POST /api/farm/{id}/next-day   — 跳转下一天
GET  /api/game/config           — 游戏配置
GET  /api/market/prices         — 市场价格
GET  /api/market/contracts      — 合同
POST /api/agents/register       — 注册
POST /api/agents/verify         — 验证
```

状态响应包含：season, day, year, hour, weather, gold, score, energy, crops[], sensory_observations[], farmer{}, buildings[], storage[], weed_count, avg_topsoil, weather_notes, 等。

---

*Phase A→E3 80+ 次提交 | Agent World 本地模拟器 v7.0 | 2026-06-18*
