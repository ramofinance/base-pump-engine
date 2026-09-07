# RAMO Smart Money Base v1.6 — Progressive Live Mode

نسخه V1.6 برای این ساخته شده که لازم نباشد تا پایان Backfill سی‌روزه برای اولین سیگنال صبر کنیم.

## تغییر اصلی V1.6

- مانیتور Real-Time از همان ابتدای Run فعال است.
- Candidate Walletها هر 5 دقیقه دوباره تحلیل می‌شوند.
- تا قبل از تکمیل تاریخچه، فقط Walletهایی که شرایط سخت‌گیرانه‌تر را پاس کنند با وضعیت `PROVISIONAL` فعال می‌شوند.
- Alertهای زودهنگام در تلگرام با عنوان `EARLY SMART MONEY ALERT (PROVISIONAL)` مشخص می‌شوند.
- یک Warm-up یک‌باره 24 ساعته به‌صورت **newest-first** اجرا می‌شود تا Walletهای فعال اخیر زودتر دیده شوند.
- سپس Backfill اصلی 30 روزه دقیقاً از checkpoint قبلی ادامه پیدا می‌کند.
- Warm-up اخیر checkpoint اصلی را جلو نمی‌اندازد و gap ایجاد نمی‌کند.
- تمام Tradeهای قبلی، Candidateها و SQLite قبلی حفظ می‌شوند.

## شروط پیش‌فرض Provisional

Wallet تا وقتی Backfill کامل نشده فقط زمانی Smart Money موقت می‌شود که حداقل:

- 35 معامله معتبر BUY/SELL
- 8 معامله بسته‌شده و قابل ارزیابی
- ROI تحقق‌یافته حداقل 25%
- Win Rate حداقل 60%
- Profit Factor حداقل 1.8
- Score حداقل 78/100
- Max Drawdown حداکثر 35%
- سهم بزرگ‌ترین برد از کل سود حداکثر 55%
- Consistency حداقل 0.45

این Thresholdها عمداً از حالت نهایی سخت‌تر هستند تا تاریخچه ناقص باعث سیگنال ضعیف نشود.

## Final Smart Money

پس از کامل شدن Backfill سی‌روزه، وضعیت Walletها از نو با قوانین Final محاسبه می‌شود. برای جلوگیری از Lucky Wallet در نسخه V1.6 علاوه بر Thresholdهای اصلی، حداقل 5 خروج بسته، Score حداقل 68 و محدودیت Drawdown/تمرکز سود نیز اعمال می‌شود.

## GitHub Actions

Secretهای قبلی بدون تغییر:

- `BASE_RPC_URL`
- `TELEGRAM_BOT_TOKEN`
- `TELEGRAM_CHAT_ID`
- `SMART_MONEY_PACKAGE_KEY`

هیچ Secret جدیدی لازم نیست.

## لاگ‌هایی که باید ببینید

```text
RAMO Smart Money Base v1.6.0 starting
V1.6 live monitor anchored at current Base tip ...
Wallet refresh complete | mode=provisional ...
V1.6 HOT WARMUP starting newest-first ...
HOT WARMUP PROGRESS | ...
V1.6 progressive mode active ...
```

اگر Wallet شرایط provisional را پاس کند:

```text
PROVISIONAL SMART MONEY wallet=0x... score=... ROI=... trades=... closed=...
```

و از همان لحظه خریدهای جدید آن Wallet در Base مانیتور می‌شود.
