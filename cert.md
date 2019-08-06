# 關於憑證

所有設定檔都放在這裡

```bash
cd /etc/letsenctypt # Let's Encrypt
vim cli.ini
```

實際去發憑證要用`certbot`(`/usr/bin/certbot`)

```bash
certbot certonly --standalone --dry-run # nginx 會佔用 443，會讓certbot失敗，先用--dry-run試試看看可不可以成功(否則失敗的話有冷卻時間)
certbot certonly --standalone # 據說是過期的時候重新發的
certbot renew # 快過期的時候renew
```
