# RAMO Smart Money Base v1.1 — Protected GitHub Runner

این Repository نسخه GitHub Actions سیستم Smart Money روی **Base Mainnet** است. هسته Python داخل `smart_money_payload.enc` رمزگذاری شده و فقط داخل Runner موقت GitHub با Secret خصوصی باز می‌شود.

## آپدیت از نسخه قبلی

اگر همین Repository قبلی را داری، Secretها را تغییر نده. فقط این فایل‌ها را با نسخه v1.1 جایگزین کن:

1. `smart_money_payload.enc`
2. `.github/workflows/smart-money-24x7.yml`
3. `README.md`

کلید `SMART_MONEY_PACKAGE_KEY` در v1.1 همان کلید بسته قبلی است، بنابراین Secret فعلی GitHub معتبر می‌ماند.

SQLite/Cache قبلی هم حفظ می‌شود. V1.1 checkpoint قدیمی Aerodrome را مهاجرت می‌دهد و از همان پیشرفت استفاده می‌کند. Routerهای جدید Uniswap/Aerodrome چون قبلاً اسکن نشده‌اند، Lookback خودشان را از ابتدا Backfill می‌کنند.

## Secrets اجباری

همان چهار Secret قبلی کافی است:

- `SMART_MONEY_PACKAGE_KEY`
- `BASE_RPC_URL`
- `TELEGRAM_BOT_TOKEN`
- `TELEGRAM_CHAT_ID`

## DEXهای فعال پیش‌فرض در v1.1

آدرس‌های زیر از مستندات رسمی Deployment/Security پروتکل‌ها گرفته شده‌اند و بدون Secret اضافی فعال می‌شوند:

- Uniswap V2 Router02 — Base
- Uniswap V3 SwapRouter02 — Base
- Uniswap Universal Router — Base
- Uniswap Universal Router 2.1.1 — Base
- Aerodrome classic Router
- Aerodrome Universal Router
- Aerodrome Slipstream SwapRouter

BaseSwap همچنان اختیاری است و فقط اگر `DEX_BASESWAP_ROUTER` را خودت تنظیم کنی فعال می‌شود.

منابع آدرس‌ها:

- Uniswap Base deployments: https://developers.uniswap.org/docs/protocols/v3/deployments/v3-base-deployments
- Uniswap v2 deployments: https://developers.uniswap.org/docs/protocols/v2/deployments
- Uniswap v4 deployments: https://developers.uniswap.org/docs/protocols/v4/deployments
- Aerodrome security/contracts: https://aerodrome.finance/security

## لاگ تشخیصی جدید

هر 10,000 بلاک یک گزارش INFO می‌بینی، مثل:

```text
DISCOVERY PROGRESS | run_blocks=10000 router_txs=1842 successful_router_txs=1810 run_BUY=604 run_SELL=497 run_UNKNOWN=709 run_unique_wallets=736 DB_trades=1810 DB_BUY=604 DB_SELL=497 DB_UNKNOWN=709 DB_unique_wallets=736 candidates=48
DEX ROUTER CALLS | aerodrome=620 aerodrome_slipstream=251 uniswap_universal=533 uniswap_v3=438
```

با این گزارش خیلی زود مشخص می‌شود سیستم واقعاً DEX transaction، BUY/SELL و Candidate Wallet پیدا می‌کند یا نه.

## اجرای GitHub

Workflow با نام `RAMO Smart Money Base 24x7` روزی چهار بار اجرا می‌شود و هر پنجره حدود 340 دقیقه کار می‌کند. SQLite و checkpointها در `app/data` با GitHub Actions Cache بین Runها حفظ می‌شوند.

پس از جایگزینی فایل‌ها:

`Actions → RAMO Smart Money Base 24x7 → Run workflow`

را اجرا کن.

## امنیت

فقط محتویات `UPLOAD_TO_GITHUB` را روی Repository قرار بده. فایل‌های داخل `PRIVATE_DO_NOT_UPLOAD` را Commit نکن.

## Disclaimer

این پروژه ابزار تحلیل داده‌های on-chain است. Smart Money classification و Telegram alert تضمین سود یا توصیه خرید/فروش نیستند.
