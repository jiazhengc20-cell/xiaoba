# 咕噜咕噜大作战 · 考研复习每日打卡

从 `xiaoba`（小八复习工作台）重做的版本：保留「陪伴角色 + 今日任务 + 番茄钟 + 打卡日历」这套骨架，
换掉了原有的实现方式，做成一个可维护、可部署的工程。角色图和完成音效沿用原素材。

## 跑起来

```bash
npm install
npm run dev     # http://localhost:5288
npm run build   # 产物在 dist/
npm run preview # 预览构建结果
```

`dist/` 是纯静态文件。资源全部用相对路径（`base: './'`），所以放在任何子目录都能跑，
包括 GitHub Pages 的 `用户名.github.io/仓库名/`。

## 手机上用

1. 把仓库推到 GitHub（源码进 `main`）。
2. 仓库 **Settings → Pages → Build and deployment → Source** 选 **GitHub Actions**。
   `.github/workflows/deploy.yml` 会自动 `npm ci && npm run build` 并发布，之后每次 push 都会重新部署。
3. 等 Actions 跑完（约 1 分钟），访问 `https://<用户名>.github.io/<仓库名>/`。
4. 手机浏览器打开这个地址 → 加到主屏幕：
   - **iPhone Safari**：底部分享按钮 → 「添加到主屏幕」
   - **安卓 Chrome / Edge**：右上角菜单 → 「添加到主屏幕」或「安装应用」

加完之后图标就是小八，点开是全屏的（`display: standalone`），看起来跟原生 App 差不多。

> 数据存在手机浏览器本地，不会跨设备同步。换手机前先在「设置 → 数据 → 导出备份」存一份 JSON。

## 和原版的区别

| 方面 | 原版 | 现在 |
|---|---|---|
| 结构 | 6 个几乎重复的单文件 HTML，逻辑分散 | 单一 Vite + React 工程，组件拆分，样式走 token |
| 日期 | `toISOString()` 取 UTC 日期，东八区早 8 点前会错一天，每月 1 号打卡在日历上不显示 | 统一本地年月日拼 key，`src/lib/date.js` 一处收口 |
| 连续打卡 | 把所有完成过的天数加起来（其实是累计） | 从今天往回逐日判断，真的连续 |
| 专注时长 | 数的是番茄钟次数，却显示成小时 | 记录真实秒数，只统计走完的整轮 |
| 计时器 | `setInterval` 逐秒递减，切后台被节流，刷新即丢 | 以 `endAt` 时间戳为准，切后台/刷新都能续上，跨天也记得记回原日期 |
| 任务 | 硬编码在 JS 数组里，没有时间概念 | 带时间段的每日时间表，按时刻排序、当前时段高亮，可增删改，表可编辑 |
| 日历 | 只能看当月 | 可翻月、点某天看详情、可补打卡 |
| 角色素材 | 16 张原始图片（含 2.4MB 的 PNG），首屏会卡 | 沿用原图，统一压到 640px 内并抠掉白底，16 张共 216KB；完成音效沿用原 aac |
| 数据 | 三个文件三套 localStorage key，互不相通 | 一个 key，支持导出/导入 JSON 备份 |
| 适配 | 固定 480px，桌面端就是中间一条手机 | 移动端底部导航 / 桌面端侧栏，两套布局 |

## 目录

```
src/
  main.jsx              入口
  App.jsx               布局与导航
  lib/
    date.js             本地日期工具（时区修正都在这里）
    store.jsx           状态、reducer、持久化、派生统计、时间表解析
    sound.js            完成音效（原 aac，解码失败退回合成提示音）
  components/
    Creature.jsx        角色图（rest / idle / study / progress / done）
    Icon.jsx            线性图标集
    TodayView.jsx       今日任务 + 备注 + 完成庆祝
    FocusTimer.jsx      专注计时
    CalendarView.jsx    打卡日历 + 补打卡
    StatsView.jsx       统计
    SettingsView.jsx    设置 / 周计划 / 导入导出
  styles/
    tokens.css          设计 token（颜色、圆角、阴影、间距、动效）
    app.css             布局与组件样式
scripts/                视觉与交互验证脚本（puppeteer-core 驱动本机 Edge）
```

## 数据

全部存在浏览器 `localStorage` 的 `gulugulu.desk.v1` 下，不上传任何服务器。

```js
{
  settings: { examDate, dailyGoalMin, subjects[], plan{0..6} },
  days:  { 'YYYY-MM-DD': { tasks: [{id,title,subjectId,start,end,done,doneAt}], note } },
  focus: { 'YYYY-MM-DD': 秒数 },
  timer: null | { totalSec, endAt, running, leftSec, date }
}
```

## 默认时间表

周一到周日通用，写在 `src/lib/store.jsx` 的 `DAY_SCHEDULE`，也可以在「设置 → 每日时间表」里
按 `08:30-09:00 科目|任务` 一行一条地改。

| 时间 | 科目 | 内容 |
|---|---|---|
| 08:30–09:00 | 英语 | 阅读 |
| 09:00–12:00 | 数学 | 套卷 / 真题（限时） |
| 14:00–15:30 | 数学 | 专题训练 |
| 15:30–18:00 | 专业课 | 背诵 1 小时 + 复习 1 小时（第 2–3 章） |
| 18:00–19:00 | 政治 | 20 分钟小题 + 30 分钟课 |
| 20:30–21:00 | 英语 | 作文 |
| 21:30–22:30 | 专业课 | 背诵 |
| 23:00–24:00 | 专业课 | 背诵巩固 |

换设备或清缓存前，去「设置 → 数据 → 导出备份」存一份 JSON。

## 还没做

- 多端同步（需要账号 + 后端；现在只有本地存储和手动导入导出）
- 离线缓存（有 manifest 和图标，能加到主屏幕，但断网打不开）
- 任务拖拽排序、子任务、重复规则
- 时段到点提醒通知
- 时间冲突检测（同一时刻排了两件事时给提示）

## 验证

```bash
node scripts/shoot.mjs      # 5 个页签 × 移动/桌面，顺带查横向溢出
node scripts/verify.mjs     # 灌测试数据，跑勾选/完成弹窗/计时器/资源 200 的冒烟
node scripts/check-plan.mjs # 时间表文本 <-> 数据的往返验证
node scripts/check-pages.mjs # 把 dist 摆到子目录里跑一遍，模拟 GitHub Pages
```

截图输出在 `shots/`。脚本用本机 Edge，不需要下载 Chromium。
