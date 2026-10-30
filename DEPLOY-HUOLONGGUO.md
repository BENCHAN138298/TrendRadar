# 火龙果剧场 · 轻量部署说明

本仓库已套用火龙果剧场轻量配置：只抓五家中文热榜，按选题关键词出报告，邮件推送，不调用 AI。邮箱账号写在 GitHub Secrets，不写进仓库。

## 配置摘要

| 项 | 值 |
| --- | --- |
| 时区 | `Asia/Shanghai` |
| 热榜平台 | 今日头条、bilibili 热搜、微博、抖音、知乎（`platforms.enabled: true`） |
| RSS | 关闭 |
| 报告 | `report.mode: current`，`display_mode: keyword` |
| 筛选 | `filter.method: keyword`（词表见 `config/frequency_words.txt`） |
| 调度 | `schedule.enabled: true`，预设 `morning_evening` |
| 定时 | GitHub Actions cron `0 0,12 * * *`（UTC 00:00 与 12:00 = 北京时间 08:00 与 20:00） |
| AI 分析 / 翻译 | 均关闭；`ai.api_key` 留空 |
| 通知 | 开启；邮件发件人、密码、收件人在 yaml 中留空 |
| 存储 | HTML 与 SQLite 均为 true |
| 展示区域 | 仅热榜（`hotlist`）；新增、RSS、独立展示、AI 分析均关闭 |
| 全局过滤 | 保留「震惊」 |

选题词组（`[组别名]`）：姐弟恋、妈妈的闺蜜、渣男、短剧、暧昧短剧。不含色情关键词。

## 还需要你填写的 Secrets

仓库 → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**。

必填：

| Secret | 说明 |
| --- | --- |
| `EMAIL_FROM` | 发件邮箱。**尚未设置**，等 Gmail 应用专用密码到位后再填。 |
| `EMAIL_PASSWORD` | 该邮箱的应用专用密码（不是登录密码）。**尚未设置**，同样等 Gmail 应用专用密码。 |
| `EMAIL_TO` | 收件人。设置后填 `benzenming@mail.grokbot.com`。 |

可选（Gmail 一般可留空，程序会按发件域名自动识别）：

| Secret | 说明 |
| --- | --- |
| `EMAIL_SMTP_SERVER` | 例如 `smtp.gmail.com` |
| `EMAIL_SMTP_PORT` | 例如 `587` |

不要把上述值写进 `config/config.yaml` 或提交到 git。yaml 里 `notification.channels.email` 的 `from` / `password` / `to` 必须保持空字符串。

## 每 7 天签到

`Get Hot News`（`.github/workflows/crawler.yml`）带试用期检查：从最早一次运行算起超过 7 天会自动停用该 workflow。

续期：打开 **Actions** → **Check In** → **Run workflow**。签到会清空历史运行记录并重新开始 7 天计时，不是累加。忘了签到就再跑一次 Check In，必要时到 Actions 里重新启用 `Get Hot News`。

长期不想签到，改走 Docker 部署（见主 README 的 Docker 章节）。

## Secrets 配好后如何试跑

1. 确认上面三个必填 Secret 已保存，yaml 里仍然没有邮箱和密码。
2. 打开 **Actions** → **Get Hot News**。
3. 右侧 **Run workflow**（`workflow_dispatch`）→ 选 `master` → **Run workflow**。
4. 看该次运行日志：抓取、关键词命中、邮件发送应成功；失败时先核对 Secret 名称是否与 workflow 里一致（`EMAIL_FROM`、`EMAIL_PASSWORD`、`EMAIL_TO`）。
5. 定时任务无需再改：每天北京时间 08:00、20:00 各跑一次。到期前记得 Check In。
