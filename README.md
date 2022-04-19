# certbot-nginx
使用 certbot 去 Let’s Encrypt 更新憑證到nginx
## 1. 下載 Certbot
``` bash
$ sudo apt-get update -y
$ sudo apt-get install certbot
$ apt-get install python3-certbot-nginx
```

## 2. 設定 Nginx
1. 去nginx設定資料夾`/etc/nginx/conf.d` 加入一個要綁SSL網域的檔案`mydomain.com.conf`

2. 檔案內容如下，其中`server_name` 就外面對應到nginx的domain。這裡綁定`mydomain.com`和 `mydomain.com`並指到 `/var/www/html`靜態網頁目錄
```bash
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    root /var/www/html;
    server_name mydomain.com www.mydomain.com;
}
```
3. 測試並重啟nginx
```bash
$ sudo nginx -t         #測試設定是否有問題
$ sudo nginx -s reload  #重啟nginx
```


## 3.取得憑證
使用 nginx 執行 certbot，並使用-d指定我們希望的網域名稱，對其發行 certificates。


```bash
$ sudo certbot --nginx -d mydomain.com -d www.mydomain.com
```

驗證domain是否正確，要先去DNS設定好到這台主機的public IP。
```bash
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator nginx, Installer nginx
Requesting a certificate for www.mydomain.com
Performing the following challenges:
http-01 challenge for www.mydomain.com
Waiting for verification...
Cleaning up challenges
```
更新`mydomain.com.conf`SSL區段
```bash
Deploying Certificate to VirtualHost /etc/nginx/conf.d/mydomain.com.conf
Redirecting all traffic on port 80 to ssl in /etc/nginx/conf.d/mydomain.com.conf

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Congratulations! You have successfully enabled https://www.mydomain.com
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

顯示目前憑證和私要目錄，還有到期期限
```bash
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/mydomain.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/mydomain.com/privkey.pem
   Your certificate will expire on 2022-07-18. To obtain a new or
   tweaked version of this certificate in the future, simply run
   certbot again with the "certonly" option. To non-interactively
   renew *all* of your certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

## 更新 SSL certificates

測試取得憑證是否正常
```bash
$ sudo cerbot renew --dry-run 
```
> 移除--dry-run就正式取得

## 確認憑證狀態
```bash
$ sudo certbot certificates
```

會出現類似下方訊息

```bash
Saving debug log to /var/log/letsencrypt/letsencrypt.log
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Found the following certs:
  Certificate Name: xxxx.xxxx.com
    Domains: xxxx.xxxxxx.com
    Expiry Date: xxxx-xxxx-xxxx 03:44:52+00:00 (VALID: 89 days)
    Certificate Path: /etc/letsencrypt/live/xxxxxx.xxxxxx.com/fullchain.pem
    Private Key Path: /etc/letsencrypt/live/xxxxxx.xxxxxx.com/privkey.pem
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```
上述資訊可看出憑證距離到期還有多久的時間

## 設置排程工作
```bash
sudo crontab -e
```

## 更新憑證
```bash
$ certbot -q renew --deploy-hook 'service nginx reload'
```

## 寫入排程規則
下方指令以固定每月1號進行 renew SSL 憑證為例，其中 --quiet 表不產生輸出結果
```bash
0 0 1 * * /usr/bin/certbot renew --quiet 
```

## 系統Timer更新
查詢系統timer
```bash
$ systemctl list-timers --all
```
複製service檔案
```bash
$ sudo mv certbot.service /lib/systemd/system/certbot.service
$ sudo mv certbot.timer /lib/systemd/system/certbot.timer
```

