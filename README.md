Rewards

<div align="center">

一个基于 PHP + MySQL 的积分管理系统

A points management system built with PHP + MySQL

Made with love by WYX

https://img.shields.io/badge/License-MIT-yellow.svg
https://img.shields.io/badge/PHP-7.2%2B-777BB4.svg
https://img.shields.io/badge/MySQL-5.7%2B-4479A1.svg
https://img.shields.io/badge/Platform-InfinityFree%20%7C%20Shared%20Hosting-blue.svg

中文 · English

</div>

---

中文

简介

Rewards 是一个基于 PHP + MySQL 的积分管理系统，提供完整的用户积分体系、金融功能、市场交易、娱乐玩法、社交互动以及后台管理能力。系统采用前后端分离的轻量架构，前端为原生 HTML/CSS/JavaScript，后端为 PHP 7.2 + MySQL/MariaDB，适合部署在共享主机或自有服务器上。

演示：https://wyx-rewards.kesug.com

功能特性

用户端

· 账号系统：注册、登录、改密、强制改名、账号状态管理
· 积分系统：积分获取、消费、明细流水、排行榜
· 签到：每日签到、连续签到倍率、签到日历
· 金融：股市（实时波动、黑天鹅事件）、银行（存取款、贷款、利息）、转账、银行卡
· 市场：官方商品、用户挂单、套装、购物车、优惠券、收藏、改价申请
· 道具：文件预览、下载、出售、道具码核销
· 娱乐：抽奖、转盘、刮刮乐、彩票、博彩（炸金花）
· 社交：好友、赠送、消息中心、传信（阅后即焚）
· 活动与工具：成就勋章、倒计日、搜索、反馈

管理端

· 用户管理、市场管理、股市管理、抽奖管理、彩票管理、博彩管理
· 交易管理、银行管理、码管理、道具核销、反馈管理
· 发消息、公告管理、成就勋章、系统设置、权限分配
· 操作日志、SQL 控制台、数据库备份、系统自检

技术栈

层 技术
前端 HTML5 + CSS3 + 原生 JavaScript
弹窗库 SweetAlert2
天气图标 QWeather Icons
PDF 渲染 PDF.js
音频封面 jsmediatags
后端 PHP 7.2
数据库 MySQL / MariaDB
数据库引擎 InnoDB
字符集 utf8mb4
托管 InfinityFree 或任意支持 PHP + MySQL 的主机

项目结构

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

实现原理

1. 请求流程

1. 用户打开页面，加载 common.js。
2. common.js 检查登录状态（本地存储 1 年 + 会话存储）。
3. 页面 onload 调用 request 请求 api.php。
4. api.php 根据 action 参数分发到对应处理逻辑。
5. 后端操作数据库，返回 JSON。
6. 前端渲染结果。

2. 核心机制

积分流水

每一笔积分变动写入 points_log，记录变动金额、变动后余额、类型、说明、订单号、时间，便于对账。

订单号

格式为 前缀 + 年月日时分秒 + 3位随机数，例如 TR20261001120000123。不同业务使用不同前缀：

前缀 类型 表
TR 转账 transfers
PU 商城购买 user_purchases
MT 市场交易 market_trades
ST 股票交易 stock_trades
BK 银行操作 bank_records
RD 兑换码 redeem_records
LOT 抽奖 lottery_records
ML 市场挂单 market_listings
FB 反馈 feedback
MSG 消息 messages
SIGN 签到 sign_log
ADJ 管理员调整 points_log
EP 传信 ephemeral_messages
WH 转盘 wheel_records
SC 刮刮乐 scratch_cards
LB 彩票 lottery_tickets
NT 公告 notices
IC 道具兑换码 item_codes
BG 勋章兑换码 badge_codes
BN 套装兑换码 item_codes
IT 道具使用码 item_use_codes

事务保护

转账、购买、改名、抽奖等多表操作使用数据库事务，任一步失败全部回滚：

```php
$pdo->beginTransaction();
try {
    // 扣除转出方积分
    // 加上收款方积分
    // 写入转账记录
    $pdo->commit();
} catch (Exception $e) {
    $pdo->rollBack();
    throw $e;
}
```

密码安全

使用 bcrypt 加密，同一密码每次哈希结果不同：

```php
$hash = password_hash($password, PASSWORD_BCRYPT);
// 验证
if (password_verify($input, $hash)) { ... }
```

权限验证

· isAdminUser 检查等级 ≥ 4
· isMMU 检查主管理员（is_mmu = 1）
· hasPermission 检查具体权限（MMU 直接通过，否则查权限表）

模块化扩展

成就、勋章、套装、道具均以文件夹形式组织，放入对应目录即可自动识别。

3. 数据库设计

表清单

用户相关：

表名 说明
users 用户主表
admin_permissions 管理员权限

积分相关：

表名 说明
points_log 积分流水
sign_log 签到记录

金融相关：

表名 说明
transfers 转账记录
user_bankcards 用户银行卡
bank_accounts 银行账户
bank_records 银行流水
bank_rates 当前利率
rate_history 利率历史
stocks 股票
stock_holdings 股票持仓
stock_trades 股票交易
stock_history 股票价格历史

市场相关：

表名 说明
products 官方商品
market_listings 市场挂单
market_trades 市场成交
cart_items 购物车
user_purchases 用户购买记录
favorites 收藏
product_bundles 套装
bundle_items 套装内商品
bundle_folders 套装文件夹信息

兑换相关：

表名 说明
coupons 优惠券
user_coupons 用户优惠券
redeem_codes 积分兑换码
redeem_records 兑换记录
item_codes 道具兑换码

道具相关：

表名 说明
user_items 用户道具
item_use_codes 道具使用码

娱乐相关：

表名 说明
lotteries 抽奖活动
lottery_records 抽奖记录
lottery_tickets 彩票
lottery_draws 开奖记录
game_rooms 博彩房间
game_players 博彩玩家
game_actions 博彩动作
user_chips 用户筹码
wheels 转盘
wheel_prizes 转盘奖品
wheel_records 转盘记录
user_wheel_quota 用户转盘额度
scratch_cards 刮刮卡

社交相关：

表名 说明
friends 好友关系
friend_requests 好友申请
messages 站内消息
ephemeral_messages 传信

成就相关：

表名 说明
user_achievements 用户成就
user_badges 用户勋章
badge_codes 勋章兑换码

其他：

表名 说明
countdowns 倒计日
feedback 反馈
notices 公告
system_settings 系统设置
operation_logs 操作日志

自定义

1. 添加成就

在 /Runfiles/achievements/ 下新建文件夹，例如 first_sign/，放入三个文件：

```
/Runfiles/achievements/first_sign/
├── config.json
├── condition.php
└── icon.png
```

config.json：

```json
{
    "name": "初次签到",
    "description": "完成第一次签到",
    "icon": "icon.png",
    "points_reward": 10,
    "is_hidden": 0
}
```

condition.php：系统把当前用户名传入，返回 true 或 false。

2. 添加勋章

在 /Runfiles/badges/ 下新建文件夹，例如 sign_badge/，放入：

```
/Runfiles/badges/sign_badge/
├── config.json
└── icon.png
```

config.json 配置名称、描述、图标、稀有度。

3. 添加套装

在 /Runfiles/bundles/ 下新建文件夹，例如 starter_pack/：

```
/Runfiles/bundles/starter_pack/
├── config.json
├── cover.png
├── 文件1.pdf
└── 文件2.mp3
```

config.json：

```json
{
    "name": "套装名称",
    "description": "说明",
    "price": 100,
    "cover": "cover.png"
}
```

然后在管理后台 → 市场管理 → 套装管理点击"重新扫描"。

4. 添加道具模块

在 /Runfiles/items/ 下新建文件夹，例如 item_key/，放入 icon.png 作为图标。若没有图标，系统显示默认图标 item_default.png。

5. 修改系统设置

登录主管理员账号，进入 管理 → 系统设置，可配置：

· 功能开关（签到、转账、股市、银行、市场、抽奖、转盘、彩票、博彩、刮刮乐、好友、赠送、成就、消息、天气、每日一言、时光沙漏、倒计日、背景音乐、搜索、传信、购物车、优惠券、维护模式等）
· 积分相关（签到最小/最大积分、提现手续费率、刮刮乐单价、传信单价）
· 刮刮乐概率（大奖、中奖、小奖概率与金额）
· 背景音乐 URL

6. 修改前端样式

主题色存储在 users 表的 theme 字段，前端使用 CSS 变量 --primary 和 --primary-2 应用。自定义颜色时，系统自动计算 --primary-2（HSL 亮度 -20%）。

7. 修改数据库连接

编辑 api.php：

```php
$host = 'localhost';
$dbname = 'your_database';
$user = 'your_username';
$pass = 'your_password';
```

8. 修改 SQL 控制台密码

```php
$sql_console_password = 'wyx_sql_2026';  // 改成你自己的强密码
```

部署

环境要求

· PHP 7.2 或更高
· MySQL 5.7 / MariaDB 10.2 或更高
· 支持 InnoDB 引擎和 utf8mb4 字符集
· Web 服务器：Apache / Nginx（需支持 PHP）
· 可选：支持 .htaccess 的主机（用于保护 data 目录）

InfinityFree 部署（推荐）

1. 注册 InfinityFree 账号，创建一个网站
2. 得到域名 yourname.infinityfreeapp.com 或绑定自己的域名
3. 通过 File Manager 或 FTP 进入 htdocs/ 目录
4. 上传项目所有文件
5. 创建数据库：控制面板 → MySQL Databases，创建一个数据库，字符集选 utf8mb4
6. 导入表结构：用 phpMyAdmin 导入项目提供的 SQL 文件
7. 配置数据库连接：编辑 api.php，填入数据库信息
8. 设置目录权限：确保 /data/、/files/、/Runfiles/ 可写
9. 保护 data 目录：在 /data/ 下放置 .htaccess：

```
Deny from all
```

10. 创建主管理员：

```sql
UPDATE users SET level = 4, is_mmu = 1 WHERE username = 'your_admin';
```

11. 访问系统：打开域名，用主管理员账号登录，进入 管理 → 系统设置 完成初始化

其他主机部署

任何支持 PHP + MySQL 的虚拟主机都可以，步骤相同。注意：

· 目录权限：755（文件 644）
· 确保 data/、files/、Runfiles/ 可写
· 若用 Nginx，确保静态资源能被直接访问

Docker 部署（本地测试）

```dockerfile
FROM php:8.2-apache
RUN docker-php-ext-install pdo pdo_mysql
COPY . /var/www/html/
RUN chmod -R 755 /var/www/html
```

```bash
docker build -t rewards .
docker run -d -p 8080:80 \
  -v $(pwd)/data:/var/www/html/data \
  rewards
```

访问 http://localhost:8080，按提示配置数据库。

常见部署问题

问题 原因 解决
无法连接到服务器 api.php 路径或数据库配置错误 检查数据库连接信息
登录后闪退回登录页 本地存储被禁用或跨域 检查浏览器设置与域名配置
图片不显示 Runfiles 目录权限或路径错误 检查目录权限与文件路径
备份失败 data 目录不可写 设置目录写权限
SQL 控制台无法进入 密码错误或不是 MMU 检查密码与 is_mmu 字段
中文乱码 数据库字符集不是 utf8mb4 修改数据库与表字符集

常见问题

Q: 为什么我买的道具不能使用？

普通道具和文件商品不一样。文件商品有文件可以下载；普通道具没有文件，需要"使用"生成道具码，等待管理员核销。

Q: 为什么我看不到天气？

天气功能依赖城市设置。没设置城市会提示去设置；城市名无法识别会提示重新设置。

Q: 为什么我的股票跌了？

股市每 2 秒更新一次，正常波动 ±3%，黑天鹅事件时下跌 20%-50%。这是设计如此，用于模拟真实市场风险。

Q: 为什么存款利息没有变化？

利息在登录时结算。一直不退出登录，利息不会实时增加。退出后重新登录会一次性结算。

Q: 忘记密码怎么办？

系统没有自助找回密码的功能。请联系管理员重置密码。

Q: 为什么我的账户被冻结了？

账户被冻结一般是违反了系统规则，具体原因需要联系管理员。冻结状态下无法登录。

Q: 为什么我不能购买自己的商品？

市场设计如此，防止用户通过自买自卖刷积分。

Q: 炸金花为什么不能单人玩？

炸金花是多人游戏，至少需要 2 人才能开始。

开源协议

MIT License。详见 LICENSE 文件。

致谢

· SweetAlert2 - 弹窗库
· PDF.js - PDF 渲染
· jsmediatags - 音频封面解析
· QWeather Icons - 天气图标

---

English

Introduction

Rewards is a points management system built with PHP + MySQL. It provides a complete point system, financial features, marketplace trading, entertainment modules, social interactions, and a full admin backend. The system uses a lightweight separated front-end and back-end architecture: the front end is native HTML/CSS/JavaScript, and the back end is PHP 7.2 + MySQL/MariaDB. It is suitable for shared hosting or self-hosted servers.

Demo: https://wyx-rewards.kesug.com

Features

User Side

· Account system: registration, login, password change, forced rename, account status management
· Points system: earning, spending, transaction log, leaderboard
· Check-in: daily check-in, consecutive check-in multiplier, check-in calendar
· Finance: stock market (real-time fluctuation, black swan events), bank (deposit, withdraw, loan, interest), transfer, bank cards
· Marketplace: official products, user listings, bundles, shopping cart, coupons, favorites, price change requests
· Items: file preview, download, sell, item code redemption
· Entertainment: lottery, wheel, scratch cards, daily lottery, gambling (Zha Jin Hua)
· Social: friends, gifting, message center, ephemeral messages (burn after reading)
· Activities and tools: achievements and badges, countdown, search, feedback

Admin Side

· User management, marketplace management, stock management, lottery management, daily lottery management, gambling management
· Trade management, bank management, code management, item redemption, feedback management
· Send messages, notice management, achievements and badges, system settings, permission assignment
· Operation logs, SQL console, database backup, system self-check

Tech Stack

Layer Technology
Front end HTML5 + CSS3 + native JavaScript
Dialog library SweetAlert2
Weather icons QWeather Icons
PDF rendering PDF.js
Audio cover jsmediatags
Back end PHP 7.2
Database MySQL / MariaDB
Database engine InnoDB
Charset utf8mb4
Hosting InfinityFree or any host supporting PHP + MySQL

Project Structure

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

How It Works

1. Request Flow

1. The user opens a page and loads common.js.
2. common.js checks login status (local storage for 1 year + session storage).
3. The page onload calls request to api.php.
4. api.php dispatches to the corresponding handler based on the action parameter.
5. The back end operates the database and returns JSON.
6. The front end renders the result.

2. Core Mechanisms

Points Log

Every point change is written to points_log, recording amount, balance after change, type, description, order number, and time, for reconciliation.

Order Number

Format is prefix + YYYYMMDDHHMMSS + 3 random digits, for example TR20261001120000123. Different businesses use different prefixes:

Prefix Type Table
TR Transfer transfers
PU Mall purchase user_purchases
MT Market trade market_trades
ST Stock trade stock_trades
BK Bank operation bank_records
RD Redeem code redeem_records
LOT Lottery lottery_records
ML Market listing market_listings
FB Feedback feedback
MSG Message messages
SIGN Check-in sign_log
ADJ Admin adjustment points_log
EP Ephemeral message ephemeral_messages
WH Wheel wheel_records
SC Scratch card scratch_cards
LB Daily lottery lottery_tickets
NT Notice notices
IC Item redeem code item_codes
BG Badge redeem code badge_codes
BN Bundle redeem code item_codes
IT Item use code item_use_codes

Transaction Protection

Multi-table operations such as transfer, purchase, rename, and lottery use database transactions. Any failure rolls back everything:

```php
$pdo->beginTransaction();
try {
    // Deduct sender points
    // Add receiver points
    // Write transfer record
    $pdo->commit();
} catch (Exception $e) {
    $pdo->rollBack();
    throw $e;
}
```

Password Security

Passwords are hashed with bcrypt. The same password produces a different hash each time:

```php
$hash = password_hash($password, PASSWORD_BCRYPT);
// Verify
if (password_verify($input, $hash)) { ... }
```

Permission Checks

· isAdminUser checks level >= 4
· isMMU checks main admin (is_mmu = 1)
· hasPermission checks specific permissions (MMU passes directly, otherwise checks the permission table)

Modular Extension

Achievements, badges, bundles, and items are organized as folders. Dropping a folder into the corresponding directory makes it auto-recognized.

3. Database Design

Table List

User-related:

Table Description
users User main table
admin_permissions Admin permissions

Points-related:

Table Description
points_log Points log
sign_log Check-in records

Finance-related:

Table Description
transfers Transfer records
user_bankcards User bank cards
bank_accounts Bank accounts
bank_records Bank records
bank_rates Current rates
rate_history Rate history
stocks Stocks
stock_holdings Stock holdings
stock_trades Stock trades
stock_history Stock price history

Marketplace-related:

Table Description
products Official products
market_listings Market listings
market_trades Market trades
cart_items Shopping cart
user_purchases User purchases
favorites Favorites
product_bundles Bundles
bundle_items Bundle items
bundle_folders Bundle folder info

Redeem-related:

Table Description
coupons Coupons
user_coupons User coupons
redeem_codes Point redeem codes
redeem_records Redeem records
item_codes Item redeem codes

Item-related:

Table Description
user_items User items
item_use_codes Item use codes

Entertainment-related:

Table Description
lotteries Lottery activities
lottery_records Lottery records
lottery_tickets Daily lottery tickets
lottery_draws Draw records
game_rooms Gambling rooms
game_players Gambling players
game_actions Gambling actions
user_chips User chips
wheels Wheels
wheel_prizes Wheel prizes
wheel_records Wheel records
user_wheel_quota User wheel quota
scratch_cards Scratch cards

Social-related:

Table Description
friends Friendships
friend_requests Friend requests
messages Internal messages
ephemeral_messages Ephemeral messages

Achievement-related:

Table Description
user_achievements User achievements
user_badges User badges
badge_codes Badge redeem codes

Others:

Table Description
countdowns Countdowns
feedback Feedback
notices Notices
system_settings System settings
operation_logs Operation logs

Customization

1. Add an Achievement

Create a folder under /Runfiles/achievements/, for example first_sign/, with three files:

```
/Runfiles/achievements/first_sign/
├── config.json
├── condition.php
└── icon.png
```

config.json:

```json
{
    "name": "First Check-in",
    "description": "Complete your first check-in",
    "icon": "icon.png",
    "points_reward": 10,
    "is_hidden": 0
}
```

condition.php: The system passes the current username in, and the code returns true or false.

2. Add a Badge

Create a folder under /Runfiles/badges/, for example sign_badge/, with:

```
/Runfiles/badges/sign_badge/
├── config.json
└── icon.png
```

config.json defines name, description, icon, and rarity.

3. Add a Bundle

Create a folder under /Runfiles/bundles/, for example starter_pack/:

```
/Runfiles/bundles/starter_pack/
├── config.json
├── cover.png
├── file1.pdf
└── file2.mp3
```

config.json:

```json
{
    "name": "Bundle Name",
    "description": "Description",
    "price": 100,
    "cover": "cover.png"
}
```

Then go to Admin -> Marketplace -> Bundle Management and click "Rescan".

4. Add an Item Module

Create a folder under /Runfiles/items/, for example item_key/, and put icon.png inside as the icon. If no icon exists, the system shows the default icon item_default.png.

5. Change System Settings

Log in as the main admin, go to Admin -> System Settings, and configure:

· Feature toggles (check-in, transfer, stock, bank, marketplace, lottery, wheel, daily lottery, gambling, scratch cards, friends, gifting, achievements, messages, weather, daily quote, time hourglass, countdown, BGM, search, ephemeral messages, cart, coupons, maintenance mode, etc.)
· Points settings (min/max check-in points, withdrawal fee rate, scratch card price, ephemeral message price)
· Scratch card probabilities (big, medium, small prize probabilities and amounts)
· Background music URL

6. Change Front-end Style

The theme color is stored in the theme field of the users table. The front end applies it via CSS variables --primary and --primary-2. For custom colors, the system automatically computes --primary-2 (HSL lightness -20%).

7. Change Database Connection

Edit api.php:

```php
$host = 'localhost';
$dbname = 'your_database';
$user = 'your_username';
$pass = 'your_password';
```

8. Change SQL Console Password

```php
$sql_console_password = 'wyx_sql_2026';  // Change to your own strong password
```

Deployment

Requirements

· PHP 7.2 or higher
· MySQL 5.7 / MariaDB 10.2 or higher
· InnoDB engine and utf8mb4 charset support
· Web server: Apache / Nginx (with PHP support)
· Optional: a host supporting .htaccess (to protect the data directory)

InfinityFree Deployment (Recommended)

1. Register at InfinityFree and create a site
2. Get your domain yourname.infinityfreeapp.com or bind a custom domain
3. Enter htdocs/ via File Manager or FTP
4. Upload all project files
5. Create database: Control Panel -> MySQL Databases, create a database with utf8mb4 charset
6. Import schema: Use phpMyAdmin to import the provided SQL file
7. Configure database connection: Edit api.php and fill in the database info
8. Set directory permissions: Ensure /data/, /files/, /Runfiles/ are writable
9. Protect the data directory: Place a .htaccess in /data/:

```
Deny from all
```

10. Create the main admin:

```sql
UPDATE users SET level = 4, is_mmu = 1 WHERE username = 'your_admin';
```

11. Access the system: Open your domain, log in with the main admin account, and go to Admin -> System Settings to finish initialization

Other Hosts

Any PHP + MySQL shared hosting works, with the same steps. Notes:

· Directory permissions: 755 (files 644)
· Ensure data/, files/, Runfiles/ are writable
· On Nginx, ensure static assets are directly accessible

Docker (Local Testing)

```dockerfile
FROM php:8.2-apache
RUN docker-php-ext-install pdo pdo_mysql
COPY . /var/www/html/
RUN chmod -R 755 /var/www/html
```

```bash
docker build -t rewards .
docker run -d -p 8080:80 \
  -v $(pwd)/data:/var/www/html/data \
  rewards
```

Visit http://localhost:8080 and configure the database as prompted.

Common Deployment Issues

Issue Cause Solution
Cannot connect to server Wrong api.php path or database config Check database connection info
Login flashes back to login page Local storage disabled or cross-domain Check browser settings and domain config
Images not showing Runfiles directory permission or wrong path Check directory permissions and file paths
Backup fails data directory not writable Set directory write permission
Cannot enter SQL console Wrong password or not MMU Check password and is_mmu field
Chinese garbled text Database charset not utf8mb4 Change database and table charset

FAQ

Q: Why can't I use the item I bought?

Normal items and file products are different. File products have files to download; normal items have no file and need to be "used" to generate an item code, waiting for admin redemption.

Q: Why can't I see the weather?

Weather depends on city settings. If no city is set, it prompts you to set one; if the city name is unrecognized, it prompts you to reset it.

Q: Why did my stock drop?

The stock market updates every 2 seconds, with normal fluctuation of ±3%, and black swan events causing 20%-50% drops. This is by design, simulating real market risk.

Q: Why hasn't my deposit interest changed?

Interest is settled at login. If you stay logged in, interest doesn't increase in real time. Log out and back in to settle it all at once.

Q: What if I forget my password?

There is no self-service password recovery. Please contact the admin to reset it.

Q: Why is my account frozen?

Accounts are usually frozen for violating system rules. Contact the admin for details. Frozen accounts cannot log in.

Q: Why can't I buy my own product?

This is by design, to prevent users from farming points through self-trading.

Q: Why can't I play Zha Jin Hua alone?

Zha Jin Hua is a multiplayer game requiring at least 2 players.

License

MIT License. See LICENSE.

Credits

· SweetAlert2 - Dialog library
· PDF.js - PDF rendering
· jsmediatags - Audio cover parser
· QWeather Icons - Weather icons
