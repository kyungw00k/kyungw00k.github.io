---
layout: post
title: "Go로 APKPure 클라이언트 만들기"
date: 2025-10-21 14:18:00 +0900
type: post
published: true
status: publish
categories:
  - Development
---

# Go로 APKPure 클라이언트 만들기

Android 앱 자동화 도구를 만들다가, APK 파일을 자동으로 다운로드할 필요가 생겼다. Play Store는 인증이 복잡해서, APKPure를 사용하기로 했다.

Python으로 된 [apkeep](https://github.com/EFForg/apkeep)이 있지만, Go 프로젝트에 임베딩하고 싶어서 Go로 포팅했다.

---

## APKPure 구조 분석

APKPure는 웹사이트와 API 두 가지 경로가 있다.

### 웹사이트 구조

```
https://apkpure.com/kr/instagram/com.instagram.android
                    ↑       ↑              ↑
                 locale   app name    package ID
```

버전 목록 페이지:
```
https://apkpure.com/kr/instagram/com.instagram.android/versions
```

### 다운로드 API

```
POST https://api.apkpure.com/v3/download_link
Content-Type: application/json

{
  "packageName": "com.instagram.android",
  "versionCode": 123456789
}
```

응답:
```json
{
  "download_link": "https://download.apkpure.com/...",
  "file_size": 50000000
}
```

---

## Go 클라이언트 구현

### 기본 구조

```go
type Client struct {
    httpClient *http.Client
    options    DownloadOptions
}

type DownloadOptions struct {
    Arch      string // arm64-v8a, armeabi-v7a, x86_64
    Language  string // ko-KR, en-US
    OSVersion string // Android API level
}

func NewClient(opts DownloadOptions) *Client {
    return &Client{
        httpClient: &http.Client{Timeout: 30 * time.Second},
        options:    opts,
    }
}
```

### 버전 목록 가져오기

```go
func (c *Client) ListVersions(packageID string) ([]Version, error) {
    url := fmt.Sprintf(
        "https://apkpure.com/%s/%s/versions",
        c.options.Language, packageID,
    )

    resp, err := c.httpClient.Get(url)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()

    doc, err := goquery.NewDocumentFromReader(resp.Body)
    if err != nil {
        return nil, err
    }

    var versions []Version
    doc.Find(".ver-item").Each(func(i int, s *goquery.Selection) {
        versions = append(versions, Version{
            Code: s.AttrOr("data-versioncode", ""),
            Name: s.Find(".ver-item-n").Text(),
            Size: s.Find(".ver-item-s").Text(),
        })
    })

    return versions, nil
}
```

goquery로 HTML 파싱. APKPure 페이지 구조가 바뀌면 깨질 수 있다.

### 다운로드 링크 획득

```go
func (c *Client) GetDownloadLink(packageID string, versionCode string) (string, error) {
    payload := map[string]string{
        "packageName": packageID,
        "versionCode": versionCode,
    }

    body, _ := json.Marshal(payload)
    resp, err := c.httpClient.Post(
        "https://api.apkpure.com/v3/download_link",
        "application/json",
        bytes.NewReader(body),
    )
    if err != nil {
        return "", err
    }
    defer resp.Body.Close()

    var result struct {
        DownloadLink string `json:"download_link"`
    }
    json.NewDecoder(resp.Body).Decode(&result)

    return result.DownloadLink, nil
}
```

### 파일 다운로드 (프로그레스 지원)

```go
func (c *Client) Download(url string, dest string, progress func(int64, int64)) error {
    resp, err := c.httpClient.Get(url)
    if err != nil {
        return err
    }
    defer resp.Body.Close()

    file, err := os.Create(dest)
    if err != nil {
        return err
    }
    defer file.Close()

    totalSize := resp.ContentLength
    var downloaded int64

    buf := make([]byte, 32*1024)
    for {
        n, err := resp.Body.Read(buf)
        if n > 0 {
            file.Write(buf[:n])
            downloaded += int64(n)
            if progress != nil {
                progress(downloaded, totalSize)
            }
        }
        if err == io.EOF {
            break
        }
        if err != nil {
            return err
        }
    }

    return nil
}
```

---

## XAPK/Split APK 처리

최신 앱은 Split APK 형태로 배포된다. APKPure는 이걸 XAPK라는 형식으로 묶어서 제공.

### XAPK 구조

```
app.xapk (ZIP 파일)
├── manifest.json
├── base.apk
├── config.arm64_v8a.apk
├── config.ko.apk
└── config.xxhdpi.apk
```

### XAPK 처리

```go
func ExtractXAPK(xapkPath string, destDir string) ([]string, error) {
    reader, err := zip.OpenReader(xapkPath)
    if err != nil {
        return nil, err
    }
    defer reader.Close()

    var apks []string
    for _, file := range reader.File {
        if strings.HasSuffix(file.Name, ".apk") {
            destPath := filepath.Join(destDir, file.Name)
            extractFile(file, destPath)
            apks = append(apks, destPath)
        }
    }

    return apks, nil
}
```

---

## CLI 도구

```go
func main() {
    app := &cli.App{
        Name:  "apkpure",
        Usage: "Download APKs from APKPure",
        Commands: []*cli.Command{
            {
                Name:  "download",
                Usage: "Download an APK",
                Flags: []cli.Flag{
                    &cli.StringFlag{Name: "app", Aliases: []string{"a"}, Required: true},
                    &cli.StringFlag{Name: "arch", Value: "arm64-v8a"},
                },
                Action: func(c *cli.Context) error {
                    client := apkpure.NewClient(apkpure.DownloadOptions{
                        Arch: c.String("arch"),
                    })

                    return client.DownloadLatest(
                        c.String("app"),
                        c.Args().First(),
                    )
                },
            },
            {
                Name:  "list",
                Usage: "List available versions",
                Flags: []cli.Flag{
                    &cli.StringFlag{Name: "app", Aliases: []string{"a"}, Required: true},
                },
                Action: func(c *cli.Context) error {
                    client := apkpure.NewClient(apkpure.DownloadOptions{})
                    versions, err := client.ListVersions(c.String("app"))
                    if err != nil {
                        return err
                    }
                    for _, v := range versions {
                        fmt.Printf("%s\t%s\t%s\n", v.Code, v.Name, v.Size)
                    }
                    return nil
                },
            },
        },
    }

    app.Run(os.Args)
}
```

### 사용 예시

```bash
# 최신 버전 다운로드
apkpure -a com.instagram.android ./downloads

# 특정 버전 다운로드
apkpure -a com.instagram.android@150.0.0.0 ./downloads

# 버전 목록
apkpure list -a com.instagram.android
```

---

## 라이브러리로 사용

```go
import "github.com/kyungw00k/apkpure-go/pkg/apkpure"

func main() {
    client := apkpure.NewClient(apkpure.DownloadOptions{
        Arch:     "arm64-v8a",
        Language: "ko-KR",
    })

    // 최신 버전 다운로드
    err := client.DownloadLatest("com.instagram.android", "./instagram.apk")

    // 프로그레스 콜백
    err = client.DownloadWithProgress(
        "com.instagram.android",
        "./instagram.apk",
        func(downloaded, total int64) {
            fmt.Printf("\r%.1f%%", float64(downloaded)/float64(total)*100)
        },
    )
}
```

---

## 삽질 포인트

| 문제 | 해결 |
|------|------|
| HTML 구조 변경 | goquery 셀렉터를 유연하게 |
| XAPK 형식 | ZIP으로 처리 |
| Rate limiting | 요청 간 딜레이 |
| 아키텍처별 APK | options에 arch 지정 |

---

## 결론

APKPure API는 공식 문서가 없어서 리버스 엔지니어링이 필요했다.

Go로 만들어서:
- 다른 Go 프로젝트에 쉽게 임베딩
- 단일 바이너리 배포
- 병렬 다운로드에 유리

> 공식 API가 없으면 직접 만들면 된다.
