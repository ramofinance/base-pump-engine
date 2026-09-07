# Security

Do not commit the package key, RPC credentials, Telegram bot token, chat ID, or the private setup file.
Store runtime credentials only in GitHub Actions repository secrets.
The encrypted payload is safe to keep in the repository only while its package key remains private.
