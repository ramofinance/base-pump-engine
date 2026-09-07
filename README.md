# RAMO Smart Money Base — Protected GitHub Runner

این Repository نسخه GitHub Actions سیستم Smart Money روی **Base Mainnet** است.
هسته Python داخل `smart_money_payload.enc` رمزگذاری شده و فقط در Runner موقت GitHub با Secret خصوصی باز می‌شود.

## Secrets اجباری

در:

`Repository > Settings > Secrets and variables > Actions > New repository secret`

این چهار Secret را بساز:

1. `SMART_MONEY_PACKAGE_KEY`
2. `BASE_RPC_URL`
3. `TELEGRAM_BOT_TOKEN`
4. `TELEGRAM_CHAT_ID`

مقدار `SMART_MONEY_PACKAGE_KEY` داخل فایل خصوصی‌ای است که همراه بسته تحویل شده و **نباید روی GitHub آپلود شود**.

## DEX Routerهای اختیاری

Aerodrome Router به صورت پیش‌فرض داخل برنامه تنظیم شده است. اگر آدرس‌های تأییدشده سایر Routerها را داری، می‌توانی این Secrets را نیز اضافه کنی:

- `DEX_UNISWAP_V2_ROUTER`
- `DEX_UNISWAP_V3_ROUTER`
- `DEX_UNISWAP_UNIVERSAL_ROUTER`
- `DEX_BASESWAP_ROUTER`

خالی بودن آنها مانع اجرای Aerodrome نمی‌شود.

## نحوه اجرا

Workflow با نام:

`RAMO Smart Money Base 24x7`

روزی چهار بار اجرا می‌شود. هر پنجره حدود 340 دقیقه کار می‌کند و پیش از timeout سخت GitHub به صورت Graceful متوقف می‌شود.

SQLite و checkpointها در `app/data` قرار دارند و بین Runnerها با GitHub Actions Cache منتقل می‌شوند. بنابراین:

- Historical Discovery از آخرین Batch ادامه پیدا می‌کند.
- Candidate Walletها و Tradeها حفظ می‌شوند.
- Smart Money Scoreها حفظ می‌شوند.
- `monitor_last_block` حفظ می‌شود.
- Alert cooldown حفظ می‌شود.

## اجرای دستی برای تست

به تب `Actions` برو، workflow را باز کن و `Run workflow` را بزن.

در اولین اجرا Historical Discovery آغاز می‌شود. با RPC استاندارد، اسکن 30 روز Base سنگین است و ممکن است طی چند پنجره GitHub تکمیل شود. این رفتار عمدی است؛ هر Batch checkpoint می‌شود و اجرای بعدی از همان نقطه ادامه می‌دهد.

تا زمانی که Discovery اولیه Catch-up نشده، ممکن است Live Monitoring شروع نشود. پس برای تست سریع می‌توان بعداً Lookback را موقتاً کمتر کرد یا Discovery Provider را با Indexer جایگزین کرد.

## امنیت

فقط محتویات پوشه `UPLOAD_TO_GITHUB` را روی GitHub قرار بده. پوشه `PRIVATE_DO_NOT_UPLOAD` و فایل کلید خصوصی را Commit نکن.

## Disclaimer

این پروژه ابزار تحلیل داده‌های on-chain است. Smart Money classification و Telegram alert تضمین سود یا توصیه خرید/فروش نیستند.
