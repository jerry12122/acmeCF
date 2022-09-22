---
title: 使用 acme.sh 與 Cloudflare 設定 Let’s Encrypt SSL 憑證自動續期
date: 2022-09-22 22:29:47
tags: Docker
categories: 建置&部屬
---

# 使用 acme.sh 與 Cloudflare 設定 Let’s Encrypt SSL 憑證自動續期

## 簡介

acme.sh 是一個在 Linux 上常用的 acme 客戶端，而 Let’s Encrypt 是免費、自動化和開放的憑證頒發機構，Cloudflare 是域名託管的平台，本教成是由三者結合搭配 Docker 來建立自動化憑證續期系統。

## 從 Github 把專案拉下來

```bash
git clone https://github.com/jerry12122/acmeCF.git
cd acmeCF
```

## 建立.env

```bash
cp env.sample .env
```

## 編輯.env

- ACME_EMAIL: 跟域名機構申請域名使用的 email，Let’s Encrypt 可以自行填寫
- CF_Token: [Clouflare 申請 API Token](https://dash.cloudflare.com/profile/api-tokens)
- CF_Zone_ID: 在 Cloudflare 域名概觀頁面取得
- CF_Account_ID: 在 Cloudflare 域名概觀頁面取得
- DOMAIN: 把要申請的域名填寫在這，格式 `-d *.example.com -d example.com`

```env
ACME_EMAIL=aaa@example.com
CF_Token=xxxxxxxxxxxxxxxxxxxxxxx
CF_Zone_ID=xxxxxxxxxxxxxxxxxxxxxx
CF_Account_ID=xxxxxxxxxxxxxxxxxxxxxx
DOMAIN=-d *.example.com -d example.com
```

## 編輯 entry.sh

將 `0 0 * * *` 改為自己想執行的時間，參考[crontab 格式](https://crontab.guru/)

```bash
#!/bin/bash
curl -sSL https://get.acme.sh | sh -s email=$ACME_EMAIL
ln -s /root/.acme.sh/acme.sh /usr/bin/acme.sh
acme.sh --set-default-ca --server letsencrypt
crontab -l > mycron
echo "0 0 * * * acme.sh  --home /acme --server letsencrypt --issue  --dns dns_cf  $DOMAIN --force >> /log/acme.log" >> mycron
crontab mycron
rm mycron
mkdir -p /log
touch /log/acme.log
tail -f /log/acme.log
```

## 編輯 docker-compose.yml

將 `/path/to/cert` 改成存放頻證的路徑

## 編譯及啟動容器

```bash
docker-compose up -d --build
```
