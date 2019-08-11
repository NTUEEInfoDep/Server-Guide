# 關於憑證

所有設定檔都放在這裡

```bash
cd /etc/letsencrypt # Let's Encrypt
vim cli.ini         # 設定檔本人
```

## 實際去發憑證要用`certbot`(`/usr/bin/certbot`)

```bash
cd /home/ntuee/production
docker-compose stop nginx                    # nginx 會佔用 80，會讓certbot失敗
sudo certbot certonly --standalone --dry-run # 先用--dry-run試試看看可不可以成功(否則失敗的話有冷卻時間)
certbot certonly --standalone                # 實際去發憑證
```

## Renew(還沒測試過)

```bash
cd /home/ntuee/production
docker-compose stop nginx                    # nginx 會佔用 80，會讓certbot失敗
sudo certbot renew --dry-run # 先用--dry-run試試看看可不可以成功(否則失敗的話有冷卻時間)
certbot renew
```

## 如何新增 Subdomain

目標是別人 nslookup xxx.ntuee.org 的時候會找到 ntuee.org，再讓 nginx 導到 site-enabled 的 server 下

1. 加入 CNAME

   1. 到[Omnis](https://www.omnis.com/manage/login)登入，帳號是`ntuee`
   2. 選擇`Records Wizard`

      ![首頁畫面](static/img/cert/index.png)

   3. 選擇`Add a subdomain to this domain name`

      ![Records Wizard](static/img/cert/wiz.png)

   4. 選擇`Create a CNAME to an existing host name`，把 subdomain 指向`ntuee.org`

      ![CNAME](static/img/cert/cname.png)

2. 重發憑證

```bash
vim /etc/letsencrypt/cli.ini # 在domains 加入要新增的 domain
# 接著用上面certonly的流程跑
```
