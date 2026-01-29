---
layout: post
title: "GitHub Actions Matrix Strategy 삽질기"
date: 2026-01-27 14:44:00 +0900
type: post
published: true
status: publish
categories:
  - Development
---

# GitHub Actions Matrix Strategy 삽질기

크롤러를 만들었다. URL 목록을 수집하고, 각 URL에서 데이터를 추출하는 2단계 작업.

URL이 수백 개라 순차 처리하면 시간이 오래 걸린다. GitHub Actions에서 병렬로 처리하고 싶었다.

---

## 초기 구조

```yaml
jobs:
  collect:
    runs-on: ubuntu-latest
    steps:
      - name: Collect URLs
        run: python crawl.py urls -o data/urls.json

      - name: Process all URLs
        run: python crawl.py process data/urls.json
```

300개 URL 처리에 2시간. 너무 느리다.

---

## Matrix Strategy 도입

GitHub Actions의 Matrix Strategy로 병렬 처리.

```yaml
jobs:
  collect-urls:
    runs-on: ubuntu-latest
    outputs:
      url_count: ${{ steps.collect.outputs.count }}
    steps:
      - name: Collect URLs
        id: collect
        run: |
          python crawl.py urls -o data/urls.json
          COUNT=$(python -c "import json; print(len(json.load(open('data/urls.json'))))")
          echo "count=$COUNT" >> $GITHUB_OUTPUT

      - name: Upload URLs artifact
        uses: actions/upload-artifact@v4
        with:
          name: urls
          path: data/urls.json

  process:
    needs: collect-urls
    runs-on: ubuntu-latest
    strategy:
      matrix:
        chunk: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
      fail-fast: false
    steps:
      - name: Download URLs
        uses: actions/download-artifact@v4
        with:
          name: urls

      - name: Process chunk
        run: |
          python crawl.py process data/urls.json \
            --chunk ${{ matrix.chunk }} \
            --total-chunks 10
```

10개의 job이 병렬로 실행된다. 각 job은 전체의 1/10만 처리.

---

## 문제 1: 동적 Matrix

URL 수에 따라 chunk 수를 동적으로 정하고 싶었다.

### 해결: fromJSON

```yaml
jobs:
  setup:
    runs-on: ubuntu-latest
    outputs:
      matrix: ${{ steps.set-matrix.outputs.matrix }}
    steps:
      - name: Set matrix
        id: set-matrix
        run: |
          # URL 수에 따라 chunk 수 결정
          URL_COUNT=300
          CHUNKS=$((URL_COUNT / 30 + 1))  # 30개씩

          # JSON 배열 생성
          MATRIX=$(python -c "import json; print(json.dumps(list(range($CHUNKS))))")
          echo "matrix=$MATRIX" >> $GITHUB_OUTPUT

  process:
    needs: setup
    strategy:
      matrix:
        chunk: ${{ fromJSON(needs.setup.outputs.matrix) }}
```

`fromJSON`으로 동적 배열을 matrix에 전달.

---

## 문제 2: Artifact 병합

각 job이 결과를 따로 저장한다. 이걸 합쳐야 한다.

### 해결: 여러 Artifact + 병합 Job

```yaml
  process:
    # ... matrix job
    steps:
      - name: Process and save
        run: python crawl.py process --chunk ${{ matrix.chunk }} -o result_${{ matrix.chunk }}.json

      - name: Upload result
        uses: actions/upload-artifact@v4
        with:
          name: result-${{ matrix.chunk }}
          path: result_${{ matrix.chunk }}.json

  merge:
    needs: process
    runs-on: ubuntu-latest
    steps:
      - name: Download all results
        uses: actions/download-artifact@v4
        with:
          pattern: result-*
          merge-multiple: true

      - name: Merge results
        run: |
          python -c "
          import json
          import glob

          all_data = []
          for f in glob.glob('result_*.json'):
              with open(f) as fp:
                  all_data.extend(json.load(fp))

          with open('final_result.json', 'w') as fp:
              json.dump(all_data, fp)
          "
```

`merge-multiple: true`로 모든 artifact를 한 폴더에 다운로드.

---

## 문제 3: Rate Limit

10개 job이 동시에 같은 사이트를 요청하면 rate limit에 걸린다.

### 해결: max-parallel

```yaml
strategy:
  matrix:
    chunk: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
  max-parallel: 3
  fail-fast: false
```

`max-parallel: 3`으로 동시 실행을 3개로 제한.

---

## 문제 4: Self-hosted Runner

크롤링 작업이 무거워서 GitHub-hosted runner 시간이 부족했다.

### 해결: Self-hosted Runner

```yaml
jobs:
  process:
    runs-on: self-hosted
    # ...
```

집에 있는 Mac Mini를 runner로 등록:

```bash
# runner 설치
./config.sh --url https://github.com/user/repo --token xxx
./run.sh
```

무제한 실행 시간 + 더 빠른 네트워크.

---

## 최종 워크플로우

```yaml
name: Crawl

on:
  schedule:
    - cron: '0 6 * * *'
  workflow_dispatch:

jobs:
  # Stage 1: URL 수집
  collect-urls:
    runs-on: ubuntu-latest
    outputs:
      matrix: ${{ steps.set-matrix.outputs.matrix }}
    steps:
      - uses: actions/checkout@v4

      - name: Collect URLs
        run: python crawl.py urls -o data/urls.json

      - name: Set matrix
        id: set-matrix
        run: |
          COUNT=$(python -c "import json; print(len(json.load(open('data/urls.json'))))")
          CHUNKS=$(( (COUNT + 29) / 30 ))
          MATRIX=$(python -c "import json; print(json.dumps(list(range($CHUNKS))))")
          echo "matrix=$MATRIX" >> $GITHUB_OUTPUT

      - uses: actions/upload-artifact@v4
        with:
          name: urls
          path: data/urls.json

  # Stage 2: 병렬 처리
  process:
    needs: collect-urls
    runs-on: ubuntu-latest
    strategy:
      matrix:
        chunk: ${{ fromJSON(needs.collect-urls.outputs.matrix) }}
      max-parallel: 5
      fail-fast: false
    steps:
      - uses: actions/checkout@v4

      - uses: actions/download-artifact@v4
        with:
          name: urls

      - name: Process chunk ${{ matrix.chunk }}
        run: |
          python crawl.py process data/urls.json \
            --chunk ${{ matrix.chunk }} \
            --total-chunks ${{ strategy.job-total }}
            -o result_${{ matrix.chunk }}.json

      - uses: actions/upload-artifact@v4
        with:
          name: result-${{ matrix.chunk }}
          path: result_${{ matrix.chunk }}.json

  # Stage 3: 결과 병합
  merge:
    needs: process
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/download-artifact@v4
        with:
          pattern: result-*
          merge-multiple: true

      - name: Merge and commit
        run: |
          python crawl.py merge result_*.json -o data/games.jsonl
          git add data/games.jsonl
          git commit -m "Update data" || true
          git push
```

---

## 성능 비교

| 방식 | 300 URLs |
|------|----------|
| 순차 처리 | 2시간 |
| Matrix (10 chunks) | 15분 |
| Matrix (10) + max-parallel 3 | 25분 (rate limit 안전) |

---

## 삽질 정리

| 문제 | 해결 |
|------|------|
| 순차 처리 느림 | Matrix strategy |
| 동적 chunk 수 | fromJSON + outputs |
| Artifact 병합 | merge-multiple |
| Rate limit | max-parallel |
| 실행 시간 제한 | Self-hosted runner |

---

## 결론

GitHub Actions Matrix Strategy는 강력하지만, 동적으로 사용하려면 삽질이 필요하다.

핵심:
1. `fromJSON`으로 동적 배열
2. Artifact로 job 간 데이터 전달
3. `max-parallel`로 rate limit 관리

> 무료 CI/CD에서 병렬 처리는 Matrix Strategy가 답이다.
