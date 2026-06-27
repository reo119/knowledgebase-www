# 배포 가이드 — 서버 개발자용

> 이 repo(`knowledgebase-www`)는 **컨태그 홈페이지의 빌드된 정적본**입니다. 맥미니가 자동 발행하고, 서버는 **pull해서 정적 서빙**만 하면 됩니다. (이 문서는 볼트 내부 `99_System/homepage_deploy.md`의 서버측 발췌·자립본)

## 0. 개념
- repo 루트 = 사이트 루트 (정적 HTML/CSS/JS). 빌드 도구·Node **불필요**.
- 갱신은 맥미니가 push → 서버는 `git pull`만.

## 1. 사전 확정 (서버 개발자/OWNER)
| 항목 | 예시 |
|---|---|
| 도메인 | `home.contag.co.kr` |
| docroot | `/var/www/contag-home` |
| 웹서버 | nginx 또는 apache |

> ⚠ 도메인이 정해지면 맥미니 측에서 `baseUrl`을 반영해 재빌드합니다(RSS·사이트맵·OG 절대경로). 도메인 확정을 운영자에게 알려주세요.

## 2. 최초 1회 (셋업)
```bash
# repo는 public이라 인증 불요
sudo git clone https://github.com/reo119/knowledgebase-www.git /var/www/contag-home
```

**nginx** (`/etc/nginx/conf.d/contag-home.conf`):
```nginx
server {
    listen 80;
    server_name home.contag.co.kr;          # ← 실제 도메인
    root /var/www/contag-home;               # ← docroot
    index index.html;
    location / {
        try_files $uri $uri.html $uri/ /404.html;   # .html 매핑 + 404 폴백
    }
    location /static/ { expires 7d; }
}
```
```bash
sudo nginx -t && sudo systemctl reload nginx
```

**apache** (vhost):
```apache
<VirtualHost *:80>
    ServerName home.contag.co.kr
    DocumentRoot /var/www/contag-home
    <Directory /var/www/contag-home>
        Require all granted
        DirectoryIndex index.html
        Options +MultiViews          # .html 확장자 생략 매핑
        ErrorDocument 404 /404.html
    </Directory>
</VirtualHost>
```
> HTTPS는 기존 서버 인증서(certbot 등) 정책에 server_name 추가.

## 3. 갱신 시 (반복 — 맥미니가 발행할 때마다)
```bash
cd /var/www/contag-home
sudo git pull        # 최신 정적본 수신. 정적 파일 교체뿐이라 웹서버 재시작 불필요
```

## 4. 검증
```bash
curl -I http://home.contag.co.kr/             # 200
curl -s http://home.contag.co.kr/ | grep 컨태그
# 브라우저: index·services·capabilities·portfolio·contact·404
```

## 5. 롤백
```bash
cd /var/www/contag-home
sudo git log --oneline -5
sudo git reset --hard <이전 커밋>
```

## 6. 주의
- 이 repo는 **자동 생성**입니다. 직접 수정하지 마세요(다음 발행 시 덮어씀).
- SPA지만 페이지마다 실제 `.html`이 있어 직접 URL 접근 OK(위 `try_files`/`MultiViews`).

---
문의: 운영자(맥미니) 측. 콘텐츠/디자인 변경은 운영자가 발행합니다.
