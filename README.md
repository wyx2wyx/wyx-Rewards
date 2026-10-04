# Rewards

<div align="center">

*积分管理系统 · 使用手册*

A points management system · User Manual

版本 Version: V0.4.1

开发 Developer: wyx

许可 License: MIT

</div>

---

## 简体中文

### 项目简介

Rewards 是一个基于 PHP + MySQL 的积分管理系统，提供完整的用户积分体系、金融功能、市场交易、娱乐玩法、社交互动以及后台管理能力。系统采用前后端分离的轻量架构，前端为原生 HTML/CSS/JavaScript，后端为 PHP 7.2 + MySQL/MariaDB，适合部署在共享主机或自有服务器上。

### 功能概览

*用户端*

- 账号系统：注册、登录、改密、强制改名、账号状态管理
- 积分系统：积分获取、消费、明细流水、排行榜
- 签到：每日签到、连续签到倍率、签到日历
- 金融：股市（实时波动、黑天鹅事件）、银行（存取款、贷款、利息）、转账、银行卡
- 市场：官方商品、用户挂单、套装、购物车、优惠券、收藏、改价申请
- 道具：文件预览、下载、出售、道具码核销
- 娱乐：抽奖、转盘、刮刮乐、彩票、博彩（炸金花）
- 社交：好友、赠送、消息中心、传信（阅后即焚）
- 活动与工具：成就勋章、倒计日、搜索、反馈

*管理端*

- 用户管理、市场管理、股市管理、抽奖管理、彩票管理、博彩管理
- 交易管理、银行管理、码管理、道具核销、反馈管理
- 发消息、公告管理、成就勋章、系统设置、权限分配
- 操作日志、SQL 控制台、数据库备份、系统自检

### 技术栈

| 层 | 技术 |
| --- | --- |
| 前端 | HTML5 + CSS3 + 原生 JavaScript |
| 弹窗库 | SweetAlert2 |
| 天气图标 | QWeather Icons |
| PDF 渲染 | PDF.js |
| 音频封面 | jsmediatags |
| 后端 | PHP 7.2 |
| 数据库 | MySQL / MariaDB |
| 数据库引擎 | InnoDB |
| 字符集 | utf8mb4 |
| 托管 | InfinityFree 或任意支持 PHP + MySQL 的主机 |

### 项目结构

```
/
├── index.html                  登录页
├── dashboard.html              首页
├── features.html               功能与活动
├── settings.html               个人中心
├── admin.html                  管理后台
├── rename.html                 强制改名
├── notices.html                公告
├── stock.html                  股市
├── bank.html                   银行
├── transfer.html               转账
├── bankcard.html               我的银行卡
├── items.html                  我的道具
├── market.html                 市场
├── cart.html                   购物车
├── lottery.html                抽奖
├── wheel.html                  转盘
├── scratch.html                刮刮乐
├── lottery_bet.html            彩票
├── game.html                   博彩
├── friends.html                好友
├── gift.html                   赠送
├── messages.html               消息中心
├── ephemeral.html              传信
├── points_log.html             积分明细
├── my_purchases.html           我的购买
├── order_search.html           订单查询
├── feedback.html               意见反馈
├── achievements.html           成就勋章
├── countdown.html              倒计日
├── search.html                 搜索
├── help.html                   帮助中心
├── manual.html                 使用手册
├── about.html                  关于
├── selfcheck.html              系统自检
├── coupons.html                我的优惠券
├── favorites.html              我的收藏
├── api.php                     后端 API
├── common.js                   公共 JS
├── data.json                   静态文案
├── manifest.json               PWA 配置
├── sw.js                       Service Worker
├── favicon.ico                 网站图标
├── data/                       数据备份目录
├── files/                      文件商品
└── Runfiles/                   素材
    ├── feature-photo/          界面图标
    ├── covers/                 封面
    ├── bundles/                套装
    ├── Puke/                   扑克牌
    ├── items/                  道具模块
    ├── achievements/           成就模块
    ├── badges/                 勋章模块
    ├── avatars/                用户头像
    └── ...
```

### 实现原理

*请求流程*

1. 用户打开页面，加载 common.js。
2. common.js 检查登录状态（本地存储 1 年 + 会话存储）。
3. 页面 onload 调用 request 请求 api.php。
4. api.php 根据 action 参数分发到对应处理逻辑。
5. 后端操作数据库，返回 JSON。
6. 前端渲染结果。

*核心机制*

- *积分流水*：每一笔积分变动写入 points_log，记录变动金额、变动后余额、类型、说明、订单号、时间，便于对账。
- *订单号*：格式为 前缀 + 年月日时分秒 + 3位随机数，例如 TR20261001120000123。不同业务使用不同前缀。
- *事务保护*：转账、购买、改名、抽奖等多表操作使用数据库事务，任一步失败全部回滚。
- *密码安全*：使用 bcrypt 加密，同一密码每次哈希结果不同。
- *权限验证*：isAdminUser 检查等级 ≥ 4，isMMU 检查主管理员，hasPermission 检查具体权限。
- *模块化扩展*：成就、勋章、套装、道具均以文件夹形式组织，放入对应目录即可自动识别。

### 如何自定义

*1. 添加成就*

在 /Runfiles/achievements/ 下新建文件夹，例如 first_sign/，放入三个文件：

```
/Runfiles/achievements/first_sign/
├── config.json
├── condition.php
└── icon.png
```

config.json：

```
{
    "name": "初次签到",
    "description": "完成第一次签到",
    "icon": "icon.png",
    "points_reward": 10,
    "is_hidden": 0
}
```

condition.php：系统把当前用户名传入，返回 true 或 false。

*2. 添加勋章*

在 /Runfiles/badges/ 下新建文件夹，例如 sign_badge/，放入：

```
/Runfiles/badges/sign_badge/
├── config.json
└── icon.png
```

config.json 配置名称、描述、图标、稀有度。

*3. 添加套装*

在 /Runfiles/bundles/ 下新建文件夹，例如 starter_pack/：

```
/Runfiles/bundles/starter_pack/
├── config.json
├── cover.png
├── 文件1.pdf
└── 文件2.mp3
```

config.json：

```
{
    "name": "套装名称",
    "description": "说明",
    "price": 100,
    "cover": "cover.png"
}
```

然后在管理后台 → 市场管理 → 套装管理点击“重新扫描”。

*4. 添加道具模块*

在 /Runfiles/items/ 下新建文件夹，例如 item_key/，放入 icon.png 作为图标。若没有图标，系统显示默认图标 item_default.png。

*5. 修改系统设置*

登录主管理员账号，进入 管理 → 系统设置，可配置：

- 功能开关（签到、转账、股市、银行、市场、抽奖、转盘、彩票、博彩、刮刮乐、好友、赠送、成就、消息、天气、每日一言、时光沙漏、倒计日、背景音乐、搜索、传信、购物车、优惠券、维护模式等）
- 积分相关（签到最小/最大积分、提现手续费率、刮刮乐单价、传信单价）
- 刮刮乐概率（大奖、中奖、小奖概率与金额）
- 背景音乐 URL

*6. 修改前端样式*

主题色存储在 users 表的 theme 字段，前端使用 CSS 变量 --primary 和 --primary-2 应用。自定义颜色时，系统自动计算 --primary-2（HSL 亮度 -20*）。

### 如何部署

*环境要求*

- PHP 7.2 或更高
- MySQL 5.7 / MariaDB 10.2 或更高
- 支持 InnoDB 引擎和 utf8mb4 字符集
- Web 服务器：Apache / Nginx（需支持 PHP）
- 可选：支持 .htaccess 的主机（用于保护 data 目录）

*部署步骤*

*1. 上传文件*

把项目所有文件上传到网站根目录（与 index.html 同级）。

*2. 创建数据库*

在 MySQL 中创建一个数据库，字符集选择 utf8mb4。

*3. 导入表结构*

导入项目提供的 SQL 文件（或运行安装脚本），创建所有必需的表。必需的表包括：

```
users, admin_permissions, points_log, sign_log,
transfers, user_bankcards, bank_accounts, bank_records,
bank_rates, rate_history, stocks, stock_holdings, stock_trades, stock_history,
products, market_listings, market_trades, cart_items, user_purchases,
favorites, product_bundles, bundle_items, bundle_folders,
coupons, user_coupons, redeem_codes, redeem_records, item_codes,
user_items, item_use_codes,
lotteries, lottery_records, lottery_tickets, lottery_draws,
game_rooms, game_players, game_actions, user_chips,
wheels, wheel_prizes, wheel_records, user_wheel_quota, scratch_cards,
friends, friend_requests, messages, ephemeral_messages,
user_achievements, user_badges, badge_codes,
countdowns, feedback, notices, system_settings, operation_logs
```

*4. 配置数据库连接*

编辑 api.php，修改数据库连接信息：

```
$host = 'localhost';
$dbname = 'your_database';
$user = 'your_username';
$pass = 'your_password';
```

*5. 设置 SQL 控制台密码*

在 api.php 中找到控制台密码配置，修改默认值：

```
$sql_console_password = 'wyx_sql_2026';
```

建议改成你自己的强密码。

*6. 设置目录权限*

确保以下目录可写：

```
/data/          备份目录
/files/         文件商品
/Runfiles/      素材目录
```

*7. 保护 data 目录*

在 /data/ 下放置 .htaccess，禁止外部访问：

```
Deny from all
```

Nginx 用户可在配置中添加：

```
location /data/ {
    deny all;
}
```

*8. 创建主管理员*

在 users 表中创建第一个账号，把 level 设为 4 或更高，把 is_mmu 设为 1：

```
UPDATE users SET level = 4, is_mmu = 1 WHERE username = 'your_admin';
```

*9. 访问系统*

打开浏览器访问你的域名，使用主管理员账号登录，进入 管理 → 系统设置 完成初始化配置。

*10. 配置自动备份与自检（可选）*

主管理员登录时，系统会自动检查是否需要备份（超过 7 天）和自检（超过 7 天），无需额外配置。也可以手动触发。

*常见部署问题*

| 问题 | 原因 | 解决 |
| --- | --- | --- |
| 无法连接到服务器 | api.php 路径或数据库配置错误 | 检查数据库连接信息 |
| 登录后闪退回登录页 | 本地存储被禁用或跨域 | 检查浏览器设置与域名配置 |
| 图片不显示 | Runfiles 目录权限或路径错误 | 检查目录权限与文件路径 |
| 备份失败 | data 目录不可写 | 设置目录写权限 |
| SQL 控制台无法进入 | 密码错误或不是 MMU | 检查密码与 is_mmu 字段 |
| 中文乱码 | 数据库字符集不是 utf8mb4 | 修改数据库与表字符集 |

### 订单号前缀对照表

| 前缀 | 类型 | 表 |
| --- | --- | --- |
| TR | 转账 | transfers |
| PU | 商城购买 | user_purchases |
| MT | 市场交易 | market_trades |
| ST | 股票交易 | stock_trades |
| BK | 银行操作 | bank_records |
| RD | 兑换码 | redeem_records |
| LOT | 抽奖 | lottery_records |
| ML | 市场挂单 | market_listings |
| FB | 反馈 | feedback |
| MSG | 消息 | messages |
| SIGN | 签到 | sign_log |
| ADJ | 管理员调整 | points_log |
| EP | 传信 | ephemeral_messages |
| WH | 转盘 | wheel_records |
| SC | 刮刮乐 | scratch_cards |
| LB | 彩票 | lottery_tickets |
| NT | 公告 | notices |
| IC | 道具兑换码 | item_codes |
| BG | 勋章兑换码 | badge_codes |
| BN | 套装兑换码 | item_codes |
| IT | 道具使用码 | item_use_codes |

*订单号格式：* 前缀 + 年月日时分秒 + 3 位随机数

*例如：* TR20261001120000123

---

## English

### Introduction

Rewards is a points management system built with PHP + MySQL. It provides a complete point system, financial features, marketplace trading, entertainment modules, social interactions, and a full admin backend. The system uses a lightweight separated front-end and back-end architecture: the front end is native HTML/CSS/JavaScript, and the back end is PHP 7.2 + MySQL/MariaDB. It is suitable for shared hosting or self-hosted servers.

### Features

*User Side*

- Account system: registration, login, password change, forced rename, account status management
- Points system: earning, spending, transaction log, leaderboard
- Check-in: daily check-in, consecutive check-in multiplier, check-in calendar
- Finance: stock market (real-time fluctuation, black swan events), bank (deposit, withdraw, loan, interest), transfer, bank cards
- Marketplace: official products, user listings, bundles, shopping cart, coupons, favorites, price change requests
- Items: file preview, download, sell, item code redemption
- Entertainment: lottery, wheel, scratch cards, daily lottery, gambling (Zha Jin Hua)
- Social: friends, gifting, message center, ephemeral messages (burn after reading)
- Activities and tools: achievements and badges, countdown, search, feedback

*Admin Side*

- User management, marketplace management, stock management, lottery management, daily lottery management, gambling management
- Trade management, bank management, code management, item redemption, feedback management
- Send messages, notice management, achievements and badges, system settings, permission assignment
- Operation logs, SQL console, database backup, system self-check

### Tech Stack

| Layer | Technology |
| --- | --- |
| Front end | HTML5 + CSS3 + native JavaScript |
| Dialog library | SweetAlert2 |
| Weather icons | QWeather Icons |
| PDF rendering | PDF.js |
| Audio cover | jsmediatags |
| Back end | PHP 7.2 |
| Database | MySQL / MariaDB |
| Database engine | InnoDB |
| Charset | utf8mb4 |
| Hosting | InfinityFree or any host supporting PHP + MySQL |

### Project Structure

```
/
├── index.html                  Login page
├── dashboard.html              Home
├── features.html               Features and activities
├── settings.html               User center
├── admin.html                  Admin panel
├── rename.html                 Forced rename
├── notices.html                Notices
├── stock.html                  Stock market
├── bank.html                   Bank
├── transfer.html               Transfer
├── bankcard.html               My bank cards
├── items.html                  My items
├── market.html                 Marketplace
├── cart.html                   Shopping cart
├── lottery.html                Lottery
├── wheel.html                  Wheel
├── scratch.html                Scratch cards
├── lottery_bet.html            Daily lottery
├── game.html                   Gambling
├── friends.html                Friends
├── gift.html                   Gifting
├── messages.html               Message center
├── ephemeral.html              Ephemeral messages
├── points_log.html             Points log
├── my_purchases.html           My purchases
├── order_search.html           Order search
├── feedback.html               Feedback
├── achievements.html           Achievements and badges
├── countdown.html              Countdown
├── search.html                 Search
├── help.html                   Help center
├── manual.html                 User manual
├── about.html                  About
├── selfcheck.html              System self-check
├── coupons.html                My coupons
├── favorites.html              My favorites
├── api.php                     Backend API
├── common.js                   Common JS
├── data.json                   Static text
├── manifest.json               PWA config
├── sw.js                       Service Worker
├── favicon.ico                 Site icon
├── data/                       Backup directory
├── files/                      File products
└── Runfiles/                   Assets
    ├── feature-photo/          UI icons
    ├── covers/                 Covers
    ├── bundles/                Bundles
    ├── Puke/                   Playing cards
    ├── items/                  Item modules
    ├── achievements/           Achievement modules
    ├── badges/                 Badge modules
    ├── avatars/                User avatars
    └── ...
```

### How It Works

*Request Flow*

1. The user opens a page and loads common.js.
2. common.js checks login status (local storage for 1 year + session storage).
3. The page onload calls request to api.php.
4. api.php dispatches to the corresponding handler based on the action parameter.
5. The back end operates the database and returns JSON.
6. The front end renders the result.

*Core Mechanisms*

- *Points log*: Every point change is written to points_log, recording amount, balance after change, type, description, order number, and time, for reconciliation.
- *Order number*: Format is prefix + YYYYMMDDHHMMSS + 3 random digits, for example TR20261001120000123. Different businesses use different prefixes.
- *Transaction protection*: Multi-table operations such as transfer, purchase, rename, and lottery use database transactions. Any failure rolls back everything.
- *Password security*: Passwords are hashed with bcrypt. The same password produces a different hash each time.
- *Permission checks*: isAdminUser checks level >= 4, isMMU checks main admin, hasPermission checks specific permissions.
- *Modular extension*: Achievements, badges, bundles, and items are organized as folders. Dropping a folder into the corresponding directory makes it auto-recognized.

### Customization

*1. Add an Achievement*

Create a folder under /Runfiles/achievements/, for example first_sign/, with three files:

```
/Runfiles/achievements/first_sign/
├── config.json
├── condition.php
└── icon.png
```

config.json:

```
{
    "name": "First Check-in",
    "description": "Complete your first check-in",
    "icon": "icon.png",
    "points_reward": 10,
    "is_hidden": 0
}
```

condition.php: The system passes the current username in, and the code returns true or false.

*2. Add a Badge*

Create a folder under /Runfiles/badges/, for example sign_badge/, with:

```
/Runfiles/badges/sign_badge/
├── config.json
└── icon.png
```

config.json defines name, description, icon, and rarity.

*3. Add a Bundle*

Create a folder under /Runfiles/bundles/, for example starter_pack/:

```
/Runfiles/bundles/starter_pack/
├── config.json
├── cover.png
├── file1.pdf
└── file2.mp3
```

config.json:

```
{
    "name": "Bundle Name",
    "description": "Description",
    "price": 100,
    "cover": "cover.png"
}
```

Then go to Admin -> Marketplace -> Bundle Management and click "Rescan".

*4. Add an Item Module*

Create a folder under /Runfiles/items/, for example item_key/, and put icon.png inside as the icon. If no icon exists, the system shows the default icon item_default.png.

*5. Change System Settings*

Log in as the main admin, go to Admin -> System Settings, and configure:

- Feature toggles (check-in, transfer, stock, bank, marketplace, lottery, wheel, daily lottery, gambling, scratch cards, friends, gifting, achievements, messages, weather, daily quote, time hourglass, countdown, BGM, search, ephemeral messages, cart, coupons, maintenance mode, etc.)
- Points settings (min/max check-in points, withdrawal fee rate, scratch card price, ephemeral message price)
- Scratch card probabilities (big, medium, small prize probabilities and amounts)
- Background music URL

*6. Change Front-end Style*

The theme color is stored in the theme field of the users table. The front end applies it via CSS variables --primary and --primary-2. For custom colors, the system automatically computes --primary-2 (HSL lightness -20*).

### Deployment

*Requirements*

- PHP 7.2 or higher
- MySQL 5.7 / MariaDB 10.2 or higher
- InnoDB engine and utf8mb4 charset support
- Web server: Apache / Nginx (with PHP support)
- Optional: a host supporting .htaccess (to protect the data directory)

*Steps*

*1. Upload Files*

Upload all project files to the web root (same level as index.html).

*2. Create Database*

Create a MySQL database with utf8mb4 charset.

*3. Import Schema*

Import the provided SQL file (or run the install script) to create all required tables. Required tables include:

```
users, admin_permissions, points_log, sign_log,
transfers, user_bankcards, bank_accounts, bank_records,
bank_rates, rate_history, stocks, stock_holdings, stock_trades, stock_history,
products, market_listings, market_trades, cart_items, user_purchases,
favorites, product_bundles, bundle_items, bundle_folders,
coupons, user_coupons, redeem_codes, redeem_records, item_codes,
user_items, item_use_codes,
lotteries, lottery_records, lottery_tickets, lottery_draws,
game_rooms, game_players, game_actions, user_chips,
wheels, wheel_prizes, wheel_records, user_wheel_quota, scratch_cards,
friends, friend_requests, messages, ephemeral_messages,
user_achievements, user_badges, badge_codes,
countdowns, feedback, notices, system_settings, operation_logs
```

*4. Configure Database Connection*

Edit api.php and update the database connection info:

```
$host = 'localhost';
$dbname = 'your_database';
$user = 'your_username';
$pass = 'your_password';
```

*5. Set SQL Console Password*

In api.php, find the console password config and change the default:

```
$sql_console_password = 'wyx_sql_2026';
```

Use a strong password of your own.

*6. Set Directory Permissions*

Make sure the following directories are writable:

```
/data/          Backup directory
/files/         File products
/Runfiles/      Asset directory
```

*7. Protect the data Directory*

Place a .htaccess in /data/ to deny external access:

```
Deny from all
```

For Nginx users, add to the config:

```
location /data/ {
    deny all;
}
```

*8. Create the Main Admin*

Create the first account in the users table, set level to 4 or higher, and set is_mmu to 1:

```
UPDATE users SET level = 4, is_mmu = 1 WHERE username = 'your_admin';
```

*9. Access the System*

Open your domain in a browser, log in with the main admin account, and go to Admin -> System Settings to finish initialization.

*10. Optional: Auto Backup and Self-check*

When the main admin logs in, the system automatically checks whether backup (over 7 days) and self-check (over 7 days) are needed. No extra config required. You can also trigger them manually.

*Common Deployment Issues*

| Issue | Cause | Solution |
| --- | --- | --- |
| Cannot connect to server | Wrong api.php path or database config | Check database connection info |
| Login flashes back to login page | Local storage disabled or cross-domain | Check browser settings and domain config |
| Images not showing | Runfiles directory permission or wrong path | Check directory permissions and file paths |
| Backup fails | data directory not writable | Set directory write permission |
| Cannot enter SQL console | Wrong password or not MMU | Check password and is_mmu field |
| Chinese garbled text | Database charset not utf8mb4 | Change database and table charset |

### Order Number Prefix Table

| Prefix | Type | Table |
| --- | --- | --- |
| TR | Transfer | transfers |
| PU | Mall purchase | user_purchases |
| MT | Market trade | market_trades |
| ST | Stock trade | stock_trades |
| BK | Bank operation | bank_records |
| RD | Redeem code | redeem_records |
| LOT | Lottery | lottery_records |
| ML | Market listing | market_listings |
| FB | Feedback | feedback |
| MSG | Message | messages |
| SIGN | Check-in | sign_log |
| ADJ | Admin adjustment | points_log |
| EP | Ephemeral message | ephemeral_messages |
| WH | Wheel | wheel_records |
| SC | Scratch card | scratch_cards |
| LB | Daily lottery | lottery_tickets |
| NT | Notice | notices |
| IC | Item redeem code | item_codes |
| BG | Badge redeem code | badge_codes |
| BN | Bundle redeem code | item_codes |
| IT | Item use code | item_use_codes |

*Order number format:* prefix + YYYYMMDDHHMMSS + 3 random digits

*Example:* TR20261001120000123

---

## License

This project is open-sourced under the MIT License.

MIT License permits:

- Free use
- Free modification
- Free distribution
- Commercial use

The only requirement: retain the copyright notice and license notice.

## Contact

- Developer: wyx
- Project: wyx-rewards.kesug.com
- Manual: manual.html
- Help center: help.html

## Acknowledgements

Thanks to all users who use and provide feedback.
