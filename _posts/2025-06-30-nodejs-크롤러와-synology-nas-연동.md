---
layout: post
title: "Node.js 크롤러와 Synology NAS 연동"
date: 2025-06-30 21:51:00 +0900
type: post
published: true
status: publish
categories:
  - Development
---

# Node.js 크롤러와 Synology NAS 연동

유료 ROM 사이트를 구독하고 있는데, 매번 웹에서 다운로드하기 귀찮았다. 크롤러를 만들어서 NAS에 자동으로 다운로드하게 만들었다.

---

## 구조

```
1. 크롤링: 사이트에서 게임 목록 수집
2. 파싱: 다운로드 링크 추출
3. 다운로드: Synology Download Station에 작업 추가
```

---

## 크롤링 구현

```javascript
const puppeteer = require('puppeteer');

async function crawlGames(category) {
    const browser = await puppeteer.launch();
    const page = await browser.newPage();

    // 로그인
    await page.goto('https://site.com/login');
    await page.type('#username', process.env.SITE_USER);
    await page.type('#password', process.env.SITE_PASS);
    await page.click('#login-btn');
    await page.waitForNavigation();

    // 게임 목록 페이지
    await page.goto(`https://site.com/${category}`);

    // 게임 링크 추출
    const games = await page.evaluate(() => {
        return Array.from(document.querySelectorAll('.game-item')).map(el => ({
            title: el.querySelector('.title').textContent,
            link: el.querySelector('a').href,
            image: el.querySelector('img').src
        }));
    });

    await browser.close();
    return games;
}
```

Puppeteer로 로그인이 필요한 사이트도 크롤링.

---

## 데이터 저장 (NoSQL 파일)

간단하게 JSON Lines 형식으로 저장.

```javascript
const fs = require('fs');
const path = require('path');

class Database {
    constructor(category) {
        this.path = path.join('db', category, 'titles.nosql');
    }

    load() {
        if (!fs.existsSync(this.path)) return [];
        return fs.readFileSync(this.path, 'utf8')
            .split('\n')
            .filter(line => line.trim())
            .map(line => JSON.parse(line));
    }

    save(items) {
        const data = items.map(item => JSON.stringify(item)).join('\n');
        fs.writeFileSync(this.path, data);
    }

    append(item) {
        fs.appendFileSync(this.path, JSON.stringify(item) + '\n');
    }
}
```

---

## Synology Download Station API

Synology NAS의 Download Station은 REST API를 제공한다.

### 인증

```javascript
const axios = require('axios');

class SynologyAPI {
    constructor(host, protocol = 'https') {
        this.baseUrl = `${protocol}://${host}`;
        this.sid = null;
    }

    async login(username, password) {
        const resp = await axios.get(`${this.baseUrl}/webapi/auth.cgi`, {
            params: {
                api: 'SYNO.API.Auth',
                version: 3,
                method: 'login',
                account: username,
                passwd: password,
                session: 'DownloadStation',
                format: 'cookie'
            }
        });

        if (resp.data.success) {
            this.sid = resp.data.data.sid;
        }
        return resp.data.success;
    }
}
```

### 다운로드 작업 추가

```javascript
async addDownload(url, destination) {
    const resp = await axios.get(`${this.baseUrl}/webapi/DownloadStation/task.cgi`, {
        params: {
            api: 'SYNO.DownloadStation.Task',
            version: 1,
            method: 'create',
            uri: url,
            destination: destination,
            _sid: this.sid
        }
    });

    return resp.data.success;
}

async listTasks() {
    const resp = await axios.get(`${this.baseUrl}/webapi/DownloadStation/task.cgi`, {
        params: {
            api: 'SYNO.DownloadStation.Task',
            version: 1,
            method: 'list',
            _sid: this.sid
        }
    });

    return resp.data.data.tasks;
}
```

---

## 전체 흐름

```javascript
async function main() {
    const category = process.argv[2];
    const db = new Database(category);

    // 1. 크롤링
    console.log('Crawling...');
    const games = await crawlGames(category);
    console.log(`Found ${games.length} games`);

    // 2. 기존 데이터와 비교
    const existing = db.load();
    const existingTitles = new Set(existing.map(g => g.title));
    const newGames = games.filter(g => !existingTitles.has(g.title));
    console.log(`New games: ${newGames.length}`);

    // 3. 다운로드 링크 추출
    for (const game of newGames) {
        const downloadLinks = await extractDownloadLinks(game.link);
        game.downloads = downloadLinks;
        db.append(game);
    }

    // 4. Synology NAS에 다운로드 작업 추가
    if (process.env.SYNO_HOST) {
        const syno = new SynologyAPI(process.env.SYNO_HOST, process.env.SYNO_PROTOCOL);
        await syno.login(process.env.SYNO_USER, process.env.SYNO_PASS);

        for (const game of newGames) {
            for (const link of game.downloads) {
                await syno.addDownload(link.url, `/volume1/Games/${category}`);
                console.log(`Added: ${game.title}`);
            }
        }
    }
}
```

---

## HTML 리포트 생성

다운로드 가능한 게임 목록을 HTML로 생성.

```javascript
function generateHTML(games, category) {
    const template = `
<!DOCTYPE html>
<html>
<head>
    <title>${category} Games</title>
    <style>
        .game { display: flex; margin: 10px; padding: 10px; border: 1px solid #ddd; }
        .game img { width: 100px; margin-right: 10px; }
        .game h3 { margin: 0; }
    </style>
</head>
<body>
    <h1>${category} Games (${games.length})</h1>
    ${games.map(game => `
        <div class="game">
            <img src="${game.image}" alt="${game.title}">
            <div>
                <h3>${game.title}</h3>
                <p>Downloads: ${game.downloads?.length || 0}</p>
            </div>
        </div>
    `).join('')}
</body>
</html>
    `;

    fs.writeFileSync(`db/${category}/index.html`, template);
}
```

---

## GitHub Actions 자동화

매일 자동으로 크롤링:

```yaml
name: Crawl

on:
  schedule:
    - cron: '0 0 * * *'
  workflow_dispatch:

jobs:
  crawl:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 18

      - run: npm install

      - name: Crawl SWITCH
        env:
          SITE_USER: ${{ secrets.SITE_USER }}
          SITE_PASS: ${{ secrets.SITE_PASS }}
        run: npm run crawl SWITCH

      - name: Commit changes
        run: |
          git add db/
          git commit -m "Update DB" || true
          git push
```

---

## Docker 지원

NAS에서 직접 실행할 수 있도록 Docker 이미지 생성:

```dockerfile
FROM node:18-slim

# Puppeteer 의존성
RUN apt-get update && apt-get install -y \
    chromium \
    --no-install-recommends

ENV PUPPETEER_EXECUTABLE_PATH=/usr/bin/chromium

WORKDIR /app
COPY package*.json ./
RUN npm install

COPY . .

CMD ["npm", "run", "crawl", "SWITCH"]
```

```bash
docker build -t game-crawler .
docker run -e SITE_USER=xxx -e SITE_PASS=xxx game-crawler
```

---

## 삽질 포인트

| 문제 | 해결 |
|------|------|
| 로그인 세션 유지 | Puppeteer 쿠키 |
| Synology API 인증 | sid 토큰 저장 |
| 대용량 파일 | Download Station에 위임 |
| 증분 크롤링 | JSON Lines + Set 비교 |

---

## 결론

크롤러 + NAS 연동으로 완전 자동화:
1. GitHub Actions가 매일 크롤링
2. 새 게임 발견 시 DB 업데이트
3. (선택) Synology Download Station에 자동 추가

> 자동화는 한 번 설정하면 계속 일해준다.
