## DIFY 安裝
```
## 下載 Dify 原始碼
cd ~/
git clone https://github.com/langgenius/dify.git

## 設定環境變數檔案（啟用 HTTPS）
cd ~/dify/docker/
cp .env.example .env
sed -i 's/NGINX_HTTPS_ENABLED=false/NGINX_HTTPS_ENABLED=true/g'  .env

# 下載自簽憑證與私鑰至 nginx 的 ssl 目錄
wget https://raw.githubusercontent.com/c00cjz00/dify/refs/heads/main/ssl/dify.crt -O nginx/ssl/dify.crt
wget https://raw.githubusercontent.com/c00cjz00/dify/refs/heads/main/ssl/dify.key -O nginx/ssl/dify.key

## 自訂 dify 前端，建立新的 web image
cd ~/dify
rm -rf web_demo
cp -rf web web_demo

## 將所有介面上的「Dify」字樣替換為「NCHC」
sed -i 's/Dify/NCHC/g'  ./web_demo/app/components/base/chat/chat-with-history/index.tsx
sed -i 's/Dify/NCHC/g'  ./web_demo/app/components/workflow/constants.ts
sed -i 's/Dify/NCHC/g'  ./web_demo/i18n/zh-Hant/*ts
sed -i 's/Dify/NCHC/g'  ./web_demo/i18n/zh-Hans/*ts
sed -i 's/Dify/NCHC/g'  ./web_demo/i18n/languages.json
sed -i 's/Dify/NCHC/g'  ./web_demo/i18n/en-US/*ts
sed -i 's/Dify/NCHC/g'  ./web_demo/app/layout.tsx
sed -i 's/test - Dify/test - NCHC/g' ./web_demo/hooks/use-document-title.spec.ts
sed -i 's/Dify/NCHC/g'  ./web_demo/hooks/use-document-title.ts
sed -i "s/const isCreatedByMe = params.get('isCreatedByMe') === 'true'/const isCreatedByMe = params.get('isCreatedByMe') !== 'false'/g" "./web_demo/app/(commonLayout)/apps/hooks/useAppsQueryState.ts"

## 下載自訂 Logo 並替換預設圖示
mkdir -p demo
wget "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQ2fTd47USCjvhs57atGeo2iTke1IpPODNtqw&s" -O ./demo/logo-site.png
wget "https://www.svgrepo.com/download/530572/accelerate.svg" -O ./demo/logo.svg
cp ./demo/logo.svg ./web_demo/public/logo/logo.svg
cp ./demo/logo-site.png ./web_demo/public/logo/logo.png
cp ./demo/logo-site.png ./web_demo/public/logo/logo-site-dark.png
cp ./demo/logo-site.png ./web_demo/public/logo/logo-embedded-chat-avatar.png
cp ./demo/logo-site.png ./web_demo/public/logo/logo-embedded-chat-header.png

## 建立新的 Docker image
cd ~/dify/web_demo
docker build -t="c00cjz00/dify-web:1.4.3" .

## 修改 docker-compose.yaml，使用自訂前端 image
cd ~/dify
sed -i 's$langgenius/dify-web:1.4.3$c00cjz00/dify-web:1.4.3$g' ./docker/docker-compose.yaml

## 啟動 Dify 系統（包含後端與前端）
cd ~/dify/docker
docker compose down                # 關閉現有容器（若有）
docker compose up -d              # 背景啟動所有服務

## 修改提示詞（prompts.py）為繁體中文版本
sleep 30
sudo apt update
sudo apt install opencc -y        # 安裝 OpenCC 工具（簡繁轉換）
mkdir -p demo
docker cp docker-api-1:/app/api/core/llm_generator/prompts.py demo/prompts_old.py  # 從容器中匯出舊檔案
opencc -i demo/prompts_old.py -o demo/prompts.py -c s2twp.json                # 轉換為繁體中文（台灣）
sed -i 's/Chinese/繁體中文(zh-TW)/g'  demo/prompts.py                               # 修改語言描述
sed -i 's/你今天咋樣/你今天如何/g'  demo/prompts.py                               # 修改提示語句內容
docker cp demo/prompts.py docker-api-1:/app/api/core/llm_generator/prompts.py     # 複製回容器
docker restart docker-api-1                                                            # 重新啟動 API 容器以套用更改

## 修改invite_member_mail_template_en-US.html
mkdir -p demo
docker cp docker-worker-1:/app/api/templates/invite_member_mail_template_en-US.html demo/invite_member_mail_template_en-US.html  # 從容器中匯出舊檔案
docker cp docker-worker-1:/app/api/tasks/mail_invite_member_task.py demo/mail_invite_member_task.py
sed -i 's$href="{{ url }}"$ref="https://dify2025.biobank.org.tw{{url}}"$g'  demo/invite_member_mail_template_en-US.html           
sed -i 's$<img src="https://assets.dify.ai/images/logo.png" alt="Dify Logo">$<img src="https://www.nchc.org.tw/UploadImage/SiteLayout/638784916208044068_thumb.png" alt="NCHC Logo">$g'  demo/invite_member_mail_template_en-US.html
sed -i 's$ Dify $ NCHC $g' demo/mail_invite_member_task.py
docker cp demo/invite_member_mail_template_en-US.html docker-worker-1:/app/api/templates/invite_member_mail_template_en-US.html   # 複製回容器
docker cp demo/invite_member_mail_template_en-US.html docker-worker-1:/app/api/templates/invite_member_mail_template_zh-CN     # 複製回容器
docker cp demo/mail_invite_member_task.py docker-worker-1:/app/api/tasks/mail_invite_member_task.py     # 複製回容器

docker restart docker-worker-1                                                            # 重新啟動 API 容器以套用更

```

## 開啟連線
https://dify2025.biobank.org.tw
