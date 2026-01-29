---
layout: post
title: "Art Run Generator 실패담"
date: 2026-01-03 16:02:00 +09:00
type: post
published: true
status: publish
categories:
  - Project
---

# Art Run Generator 실패담

지도 위에 그림을 그리며 달리는 "아트런"이라는 게 있다. GPS를 켜고 달리면 궤적이 지도에 남는데, 이걸 이용해서 하트 모양이나 동물 모양을 만드는 것이다.

> "이거 자동으로 코스 추천해주는 서비스 만들면 재밌겠다"

3시간 정도 작업했다. 결과적으로는 실패했지만, 왜 이런 서비스가 없는지 이해하게 됐다.

---

## 아이디어

1. 사용자가 현재 위치를 입력
2. 원하는 형상 선택 (하트, 별, 강아지 등)
3. 목표 거리 선택 (5km, 10km, 15km, 20km)
4. OSMnx로 주변 도로 네트워크를 가져와서 형상에 맞게 경로 생성
5. 카카오맵에 경로 표시

간단해 보였다.

---

## 기술 스택

- **Frontend**: 카카오맵 SDK (지도 표시, Polyline 경로)
- **Backend**: FastAPI + OSMnx (도로 네트워크 처리)
- **LLM**: Zhipu AI GLM-4.7 (커스텀 형상 생성)

OSMnx는 OpenStreetMap 데이터를 Python에서 다룰 수 있게 해주는 라이브러리다. 도로 네트워크를 그래프로 가져와서 최단 경로 계산 같은 걸 할 수 있다.

---

## 구현 과정

### 1. 형상 좌표 정의

하트, 별, 삼각형 등의 형상을 정규화된 좌표로 정의했다.

```python
def _create_heart_shape(self) -> List[Tuple[float, float]]:
    """하트 형상 좌표 (정규화)"""
    return [
        (0.0, 0.5),   # 상단 중앙
        (0.3, 0.8),   # 오른쪽 위
        (0.5, 0.5),   # 오른쪽
        (0.0, -0.5),  # 하단
        (-0.5, 0.5),  # 왼쪽
        (-0.3, 0.8),  # 왼쪽 위
        (0.0, 0.5),   # 상단 중앙
    ]
```

### 2. 도로 네트워크 다운로드

사용자 위치 기준으로 주변 도로를 가져왔다.

```python
graph = ox.graph_from_point(
    center_point,
    dist=dist_m,
    network_type="walk",
    simplify=True,
)
```

### 3. 형상을 도로에 매핑

여기서 문제가 시작됐다. 형상의 각 꼭짓점을 가장 가까운 도로 노드에 매핑하고, 노드들을 최단 경로로 연결하는 방식이었다.

```python
for lat, lng in absolute_coords:
    nearest_node = ox.distance.nearest_nodes(graph, lng, lat)
    road_nodes.append(nearest_node)

for i in range(len(road_nodes) - 1):
    path = nx.shortest_path(graph, road_nodes[i], road_nodes[i+1], weight="length")
```

---

## 왜 실패했나

### 도로는 형상대로 없다

가장 근본적인 문제. 하트 모양을 그리려면 하트 모양대로 도로가 있어야 한다. 하지만 현실 세계의 도로는 그렇지 않다.

- 형상의 꼭짓점을 도로에 매핑하면, 실제 도로 위치로 이동한다
- 하트 상단의 볼록한 부분이 도로가 없는 곳이면, 엉뚱한 곳으로 매핑된다
- 최단 경로로 연결하면 형상이 완전히 뭉개진다

### 거리 조절의 딜레마

목표 거리에 맞추려면 형상을 확대/축소해야 하는데:
- 작게 하면: 도로 밀집 지역에서 형상이 뭉개짐
- 크게 하면: 도로가 없는 지역이 포함되어 우회 경로가 생김

### LLM으로 해결하려 했으나

"토끼 모양"이라고 입력하면 LLM이 적절한 도로 노드를 선택하도록 시도했다. 주변 도로 교차점 목록을 주고 "이 중에서 토끼 모양이 되는 노드들을 선택해"라고 요청.

```python
selected_node_ids = llm_service.select_nodes_for_shape(
    request.description,
    key_nodes,
    request.distance
)
```

결과는 더 엉망이었다. LLM이 좌표의 공간적 배치를 이해하지 못했다.

---

## 깨달은 점

1. **아트런은 수동으로 경로를 찾는 게 맞다**: 지도를 보고 "저 도로가 귀처럼 생겼네"라고 사람이 판단하는 것
2. **자동화가 안 되는 이유**: 도로 패턴과 원하는 형상의 매칭은 근본적으로 어려움
3. **기존 서비스가 없는 이유**: 기술적 한계, 그리고 수요도 많지 않을 것

---

## 남은 것들

코드는 동작한다. 다만 결과가 "형상"처럼 보이지 않을 뿐.

- Frontend: 카카오맵 표시, 경로 시각화
- Backend: OSMnx 도로 네트워크 처리, 경로 생성 API
- 미리 정의된 6개 형상 (삼각형, 다이아몬드, 별, 하트, 화살표, 집)
- LLM 기반 커스텀 형상 (작동은 하지만 결과가...)

```
artrun-generator/
├── backend/
│   ├── main.py
│   └── services/
│       ├── osmnx_service.py
│       ├── shape_library.py
│       └── llm_service.py
└── frontend/
    ├── index.html
    ├── css/style.css
    └── js/
        ├── api.js
        ├── map.js
        └── route.js
```

---

## 다른 접근법?

만약 다시 시도한다면:

1. **역방향 접근**: 형상을 먼저 정의하지 말고, 도로 패턴에서 형상을 "발견"하는 방식
2. **특정 지역 특화**: "강남역 주변에서 가능한 아트런 코스" 같이 지역별로 미리 계산
3. **사용자 참여**: 사람들이 발견한 아트런 코스를 공유하는 커뮤니티

어쨌든, 이번 프로젝트는 여기서 중단.

> "왜 이런 서비스가 없는지 알게 됐다"

가끔은 실패에서 배우는 게 더 많다.
