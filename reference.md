물론이죠! 아래는 **"AutoPark Vision" 포트폴리오**를 전체 구조 설명 → 본인 파트 강조 → 기술 및 협업 정리 순으로 구성한 최종 버전입니다. 기존 Face-Arm 양식 스타일을 최대한 반영했어요.

---

# AutoPark Vision: AI 기반 스마트 자율주차 로봇 시스템

**ROS 2 + YOLOv8 + OCR 기반으로, 차량 번호판 인식 → 차량 분류 → 자율 주차 및 출차까지 수행하는 자율주차 시스템을 구축했습니다.**
**전기차 수요 증가와 주차 공간 부족 문제 해결을 목표로, TurtleBot 기반의 `인식-판단-행동` 전체 파이프라인을 자동화한 자율주행 로봇 프로토타입을 개발했습니다.**

* 📅 **개발 기간**: 2025.06.10 \~ 06.21 (12일)
* 🧑‍🤝‍🧑 **인원 구성**: 7명
  **(본인 역할: YOLO 기반 주차공간 탐지, 시스템 경량화, 주차금지 사인 인식 및 경고 시스템 구현)**
* 🛠 **사용 기술**: ROS 2, YOLOv8, EasyOCR, OAK-D Pro, TurtleBot4, RPLiDAR, InfluxDB, PyQt
* ✅ **주요 기능**: 차량 번호판 인식 및 분류, 정밀 주차/출차, 실시간 GUI 및 DB 연동, 리소스 경량화, 경고음 안내

---

## 📑 프로젝트 개요

* **주제**: AutoPark Vision (AI 기반 스마트 주차 로봇)
* **목표**: 전기차 증가와 인력 부족 문제 대응을 위한 무인 자율주차 시스템 구축
* **전체 구조 요약**:

| 주요 기능       | 기술 구성                                |
| ----------- | ------------------------------------ |
| 차량 인식       | YOLOv8 + EasyOCR → 번호판 인식 및 차량 분류    |
| 주차 공간 탐지    | YOLOv8 → 주차 가능 공간 및 금지 표지판 인식        |
| 정밀 주차/출차    | OAK-D Depth + TF 변환 + Nav2 자율주행      |
| GUI 및 DB 연동 | PyQt GUI + InfluxDB 기록               |
| 시스템 경량화     | 이미지 압축 + RaspberryPi 실시간 부하 측정 도구 개발 |

---

## 💡 본인 주요 기여

### 1️⃣ YOLOv8 기반 주차 공간 탐지 모델 구축

* 주차 가능 영역, 금지 사인 등을 인식하는 커스텀 YOLOv8 모델 학습
* 여러 버전(YOLOv5, v8 등) 비교 후 성능과 속도 고려해 `YOLOv8n` 선정
* 입력 이미지 크기 `(320x320)`로 리사이즈하여 리소스 최적화

### 2️⃣ 시스템 부하 경량화 및 실시간 측정 도구 개발

* SLAM + Navigation 부하를 고려하여 OAK-D 설정(FPS, 해상도 등) 최적화
* 이미지 압축(`compressed image`) 구조로 전환해 네트워크 트래픽 90% 절감
* 기존 `htop`, `iftop` 대신 **직접 Python 기반 리소스 측정 도구(`raspi_monitor.py`) 구현**
  → CPU 사용량, 네트워크 트래픽, RAM 사용률을 `CSV`로 정량 분석 가능

### 3️⃣ 주차금지 사인 탐지 및 경고음 출력 기능 구현

* 차량이 설정 거리 안에 접근했을 때 YOLO로 주차금지 표지판 인식 시
  → **실시간 경고음(Beep) 출력** 로직 추가
* 입출차 프로세스 과정별 threading 방식 경고음 출력으로 어떤 명령 수행중인지 확인

---

## ⚙️ 사용 장비 및 기술 스택

| 분류       | 구성 요소                              |
| -------- | ---------------------------------- |
| 하드웨어     | TurtleBot4, OAK-D Pro, RPLIDAR     |
| 프레임워크    | ROS 2 (Humble), Nav2, SLAM         |
| 인식 모델    | YOLOv8 (Ultralytics), EasyOCR      |
| GUI 및 DB | PyQt (입출차 관리), InfluxDB (시계열 기록)   |
| 최적화      | 이미지 압축 전송, RaspberryPi 리소스 측정 스크립트 |

---

## 🧭 전체 시스템 프로세스

1. **차량 진입 감지**
2. **번호판 및 차종 인식 (YOLO + OCR)**
3. **주차 공간 탐색 (YOLO 주차 구역 + 금지 사인 탐지)**
4. **Depth + TF로 정밀 주차 좌표 계산 → Nav2로 이동**
5. **입차 완료 → GUI/DB 기록 저장**
6. **출차 요청 시 위치 탐색 → 차량 이동 및 도킹**

---

## 📂 핵심 코드

### `detect_parking.py`

* YOLOv8로 주차 공간 및 금지 사인 탐지
* Depth 기반 거리 계산 후 TF 변환 → 좌표 산출

### `raspi_monitor.py`

* 실시간 CPU, PID, RAM, 네트워크 트래픽 측정
* CSV 저장으로 성능 실험 데이터 확보
  → htop/iftop보다 정확하고 비교 가능한 데이터 확보 가능

```python
# 측정 항목: 전체 CPU%, 특정 프로세스 CPU%, MEM%, TX, RX
self.csv_writer.writerow(["timestamp", "CPU%", "PID_CPU%", "Mem%", "TX_Mb", "RX_Mb"])
```

---

## 🤝 협업 및 통합 개발

* OCR 파트와 데이터 연동 구조 조율
* Navigation 파트와 위치 좌표 형식 및 기준점 통일
* GUI 팀과 차량 정보/경고음 트리거 인터페이스 정의
* 전체 모듈 통합 테스트 주도, 각기 다른 기능을 하나의 시스템으로 연결

---

## 📊 도전 과제 & 해결 방법

| 문제                    | 해결 방법                          |
| --------------------- | ------------------------------ |
| YOLO 연산 부담            | YOLOv8n 선택 + 이미지 축소 (320x320)  |
| 실시간 CPU 측정의 불안정성      | `raspi_monitor.py` 커스텀 스크립트 개발 |
| SLAM/TF 연산 중 TF 좌표 오차 | 기준점 조정 및 오프셋 보정                |
| 네트워크 전송 부하            | 압축 이미지 전송(compressed image) 적용 |
| 주차 금지 사인 대응 필요        | 거리 기반 경고음(beep) 출력 로직 구현       |

---

## ✅ 성과 및 결과물

* YOLO + OCR + Depth + Navi 통합 파이프라인 성공
* 경량화 적용 후 라즈베리파이 안정 작동 및 실시간 제어 가능
* 주차금지 사인에 반응하는 경고음 로직 정상 동작
* CSV 기반 리소스 로그 기록을 통해 실험 비교 가능

---

## 📌 개인적 성찰 및 배운 점

* 리소스가 제한된 환경에서 경량화 설계와 성능 측정의 중요성을 체감
* 복수 센서 정보(이미지, Depth, TF 등)의 융합 처리 및 좌표 변환 감각 향상
* 협업 시 인터페이스 정의와 모듈 연계 구조 설계의 중요성 확인
* 단순 YOLO 학습을 넘어, 실제 ROS 시스템 속 연동을 고려한 통합 설계 경험 확보

---

## 🔄 향후 개선 아이디어

* 다양한 조명 환경에서 OCR 인식률 향상
* 실외 환경 확장을 위한 GPS 및 외부 지도 연동
* GUI 기반 모바일 앱 전환 및 관리자 알림 기능 탑재
* SLAM 자동 갱신 및 딥러닝 기반 경로 최적화 기술 도입 검토

---

정밀주차 마킹
좋아요. 말씀하신 흐름은 **정밀 주차 좌표 마킹 시점 제어**를 위해 `BasicNavigator`의 `getFeedback()`과 `isTaskComplete()`를 활용하여 **waypoint 이동 중 주차 위치에서만 좌표 마킹이 이루어지도록** 구현한 것이 핵심입니다.

이는 포트폴리오에서 **“불필요한 객체 탐지를 방지하는 조건부 정밀 제어 로직”** 으로 정리할 수 있습니다.

---

## 📌 포트폴리오에 반영할 설명 예시

### 🔧 정밀 주차 위치 마킹 시점 제어 (`detect_ps_front.py` + `sc_follow_waypoints.py` 연계)

> 주차 구역을 YOLO로 탐지한 후, Depth 정보와 TF를 이용해 `map` 좌표로 변환하여 주차 좌표를 `/detect/object_map_pose`로 발행합니다.
> 그러나 주행 중간에 탐지되는 다른 객체들이 잘못 마킹되는 문제를 방지하기 위해, **좌표 마킹은 Navigator가 해당 waypoint에 도달한 이후로 제한**하였습니다.

#### ✅ 구현 방식

* `sc_follow_waypoints.py`에서 `BasicNavigator` 사용:

  ```python
  self.navigator = BasicNavigator()
  ...
  while not self.navigator.isTaskComplete():
      feedback = self.navigator.getFeedback()
      # 현재 waypoint index와 예상 도착 시간 확인
  ```

* `detect_ps_front.py`에서는 별도 모듈로 동작하며,

  * `/detect/object_info`를 구독하여 YOLO 탐지 객체 수신
  * 오직 waypoint 도달 이후에만 객체를 선택하여 정밀 좌표로 변환
  * `/detect/object_map_pose`로 주차 좌표 발행

#### ✅ 효과

* 중간 경로의 무의미한 탐지(전봇대, 간판, 주변 차량 등)를 필터링
* 실제 주차 위치에 도달한 시점에서만 정밀 좌표 발행이 이루어지므로, **시스템의 오탐 가능성을 최소화**

---

## 🔁 포트폴리오 문장 추가 예시 (3-2 혹은 4번 항목 아래)

```md
- waypoint 주행 중간의 잘못된 마킹을 방지하기 위해, `BasicNavigator.getFeedback()`과 `isTaskComplete()`를 활용하여 **주차 waypoint 도달 후**에만 `/detect/object_map_pose`로 정밀 좌표를 발행하도록 제어
- 이로 인해 YOLO가 탐지한 객체 중에서도 **주차 사인 객체만을 안정적으로 추출**할 수 있도록 구현
```

---

필요하면 이 로직이 반영된 `sc_follow_waypoints.py` 쪽 코드도 함께 정리해드릴 수 있어요.
👉 **포트폴리오 완성용으로 전체 정리 원하시나요?** (최신 코드 흐름 + 역할 정리 포함해서요)

---
좋은 질문이에요. 요약하자면:

> ✅ **`BasicNavigator`는 내부적으로 `NavigateToPose` 액션 클라이언트를 포함하고 있으며, 사용자가 별도로 액션 클라이언트를 정의하지 않아도 자동으로 사용됩니다.**

---

## 🔍 자세히 설명하면:

### ✅ 1. `BasicNavigator`는 Nav2의 Python 래퍼 클래스

`BasicNavigator`는 `nav2_simple_commander` 패키지에 포함된 클래스이며, 사용자가 **편하게 Nav2의 액션 서버 (`NavigateToPose`)와 상호작용**할 수 있도록 다음을 자동화해줍니다:

* 액션 클라이언트 생성 (`NavigateToPose.action`)
* 목표 위치 전달
* 상태(feedback) 확인 (`getFeedback()`)
* 완료 여부 확인 (`isTaskComplete()`)

```python
from nav2_simple_commander.basic_navigator import BasicNavigator

navigator = BasicNavigator()
navigator.goToPose(pose)  # 내부적으로 NavigateToPose 액션 요청
```

→ 이때, 내부적으로 `rclpy.action.ActionClient`를 생성하고, `/navigate_to_pose`라는 액션 이름과 `NavigateToPose` 메시지 타입을 사용해 **Nav2 액션 서버와 통신**합니다.

---

### ✅ 2. 그래서 import만 해도 "NavigateToPose" 액션을 자동으로 사용

`BasicNavigator`는 아래와 같은 작업을 내부적으로 수행합니다:

| 작업          | 설명                                          |
| ----------- | ------------------------------------------- |
| 액션 클라이언트 생성 | `rclpy.action.ActionClient(NavigateToPose)` |
| 목표 pose 전송  | `goToPose(pose)` 호출 시 액션 서버에 요청 전송          |
| 중간 피드백 확인   | `getFeedback()` 호출 시 내부 액션 상태 확인            |
| 완료 여부 확인    | `isTaskComplete()`로 성공/실패 여부 확인 가능          |

---

## 🔧 결론

> ✅ `BasicNavigator`는 `NavigateToPose` 액션 서버에 자동으로 연결되므로, `import`와 `goToPose()` 같은 메서드 호출만으로도 **Nav2의 액션 구조를 활용하는 셈**입니다.
> 따라서, 직접 액션 메시지를 선언하거나 클라이언트를 따로 구성할 필요 없이도 **액션 기반 시점 제어, 피드백 확인, 완료 판단 등의 기능을 사용할 수 있습니다.**

---

### 💡 포트폴리오에 쓸 수 있는 요약 문장

> `BasicNavigator`를 통해 별도의 액션 클라이언트 구현 없이도 Nav2의 `NavigateToPose` 액션 서버와 연동하여, 주행 피드백 및 완료 여부를 확인하며 정밀 주차 마킹 시점을 정밀 제어함.

필요하시면 이걸 기반으로 짧은 문단 정리나 다이어그램도 만들어드릴게요.


---
ref2
물론입니다. 아래는 주신 내용을 정리·보완하여 양식 통일성 있게 작성한 최종 포트폴리오 본입니다.

---

# AutoPark Vision: AI 기반 스마트 자율주차 로봇 시스템

**ROS 2 + YOLOv8 + OCR 기반으로, 차량 번호판 인식 → 차량 분류 → 주차 및 출차까지 수행하는 자율주차 시스템을 구현했습니다.**
전기차 수요 증가와 주차 공간 부족 문제를 해결하고자, `인식-판단-행동` 전 과정을 자동화한 TurtleBot 기반 주차 로봇 프로토타입을 구축했습니다.

* 📅 **개발 기간**: 2025.06.25 \~ 2025.07.04 (10일)
* 🧑‍🤝‍🧑 **팀 인원**: 7명
  *(본인 역할: YOLO 학습(주차공간 객체 탐지), 시스템 성능 측정 및 경량화, 경고음 기능 통합)*
* 🛠 **사용 기술**: ROS 2, YOLOv8, EasyOCR, OAK-D Pro, TurtleBot4, RPLiDAR, PyQt, InfluxDB
* ✅ **주요 기능**: 차량 번호판 인식 및 분류, 정밀 주차/출차, 실시간 GUI 및 DB 연동, 시스템 경량화

---

## 📌 프로젝트 개요

* **주제**: AutoPark Vision - AI 기반 스마트 자율주차 로봇 시스템
* **목표**: 주차 수요 증가 및 인건비 부담 문제를 해결하기 위한 실증형 자율주차 로봇 프로토타입 개발

### 👥 팀원 역할

| 이름      | 역할                                           |
| ------- | -------------------------------------------- |
| **이세현** | **YOLO 학습(주차 공간), 경량화 및 부하 측정, 경고음 통합 구현**   |
| 강인우     | OCR 기반 번호판 인식, Depth-TF 기반 주차 위치 추정, Navi 연동 |
| 이형연     | GUI 구현, OCR-GUI 통합, InfluxDB 구조 설계           |
| 정서윤     | \[팀장] SLAM 및 Navigation, GUI-SLAM 통합, 경고음 통합 |
| 나승원     | 번호판 인식용 YOLO 모델 학습                           |
| 이동기     | 차량 분류 및 사전 학습 데이터셋 구성                        |
| 홍진      | 자료 조사, 영상 콘텐츠 제작                             |

---

## 🗓 프로젝트 일정

| 기간             | 작업 내용                      |
| -------------- | -------------------------- |
| 06.25 \~ 06.26 | 시스템 기획 및 역할 분담             |
| 06.27 \~ 06.29 | YOLO 학습, OCR 파이프라인 구성      |
| 06.30 \~ 07.01 | SLAM + Navi 연동, GUI 연동 개발  |
| 07.01 \~ 07.02 | 시스템 통합 및 주차 프로세스 연동 테스트    |
| 07.03 \~ 07.04 | 최종 발표, 리허설, 시연 영상 및 보고서 제출 |

---

## 1. 개발 배경 및 목표

전기차와 무인화 수요의 급증에 따라 주차 공간의 효율적 관리 및 자동화를 필요로 하는 수요가 증가하고 있습니다.
`AutoPark Vision`은 ROS 기반 TurtleBot4를 활용하여, 번호판 인식 → 차량 분류 → 자율 주차 및 출차까지 **전과정 자동화**를 목표로 구축한 스마트 주차 시스템입니다.

---

## 2. 사용 장비 및 기술 스택

| 분류       | 내용                                                     |
| -------- | ------------------------------------------------------ |
| OS 및 플랫폼 | Ubuntu 22.04, ROS 2 Humble, RPi4                       |
| 로봇 하드웨어  | TurtleBot4, RPLiDAR A1, OAK-D Pro, Raspberry Pi 4      |
| 객체 탐지    | YOLOv8 (주차 공간 및 번호판 객체 감지)                             |
| 문자 인식    | EasyOCR + 정규표현식 필터링 (`\d{2,3}[가-힣]\d{4}`)              |
| 위치 인식    | OAK-D Depth + TF 변환 + SLAM 기반 위치 좌표계 변환                |
| 주행 제어    | Nav2, SLAM, `BasicNavigator` (액션 기반 Pose 이동 및 피드백 추적)  |
| 인터페이스    | PyQt 기반 GUI, InfluxDB 시계열 DB 연동                        |
| 최적화      | 압축 이미지 전송(image\_transport/compressed), 시스템 부하 측정 및 로깅 |

---

## 3. 시스템 전체 흐름

1. **차량 인식 및 분류**
   `detect_car_info.py`에서 YOLOv8로 차량 번호판 감지 → EasyOCR로 번호 추출 및 필터링 → 차량 유형 분류
2. **정보 전송 및 GUI 연동**
   추출 정보는 `/car/info` 토픽으로 발행 → `parking_gui.py`에서 실시간 표시
3. **주차 위치 탐색**
   `detect_parking.py`에서 YOLO + Depth 기반으로 주차 구역(P 마크) 인식
   → `detect_ps_front.py`에서 TF 좌표 변환 및 `PoseStamped` 마킹
   → `go_to_pose_blocking()`으로 액션 피드백 감지 후 최종 waypoint 등록
4. **자율주차 및 복귀**
   `sc_follow_waypoints.py`에서 SLAM 기반 경로 주행 → 주차 위치 정밀 이동
   → 180도 회전 후 경고음 출력 → 시작 지점으로 복귀 및 도킹 수행

---

## 4. 주요 기능 상세

### 🚘 차량 번호판 인식 및 차량 분류

* 번호판 객체 탐지 (YOLOv8 → 4 Class 분류: 일반/EV/장애인 등)
* 번호판 내 문자열 추출 (EasyOCR + 정규표현식 필터)
* 결과는 `{ "type": "ev", "car_plate": "12가1234" }` 형태로 가공 및 전송

### 📏 정밀 주차 좌표 추정 (Depth + TF)

* YOLO 탐지 중심 좌표의 Depth 추정 (중앙 3x3 window 평균)
* `pixel_to_3d()`로 카메라 좌표계로 복원 후 `do_transform_point()` 사용
* 로봇 기준으로 -0.5m offset 적용 → map 기준 좌표로 변환
* `PoseStamped`로 `/detect/object_map_pose` 발행 + RViz 마커 시각화

### 🧭 SLAM 기반 입출차 제어

* `BasicNavigator`를 활용한 액션 기반 이동 (waypoint 도달 여부 추적)
* `getFeedback()`을 통해 도중 탐지된 P 사인 무시하고 도착 후만 마킹
* 경로 실행 후 복귀 및 도킹까지 자율 수행

### 🖥 GUI 및 DB 연동

* PyQt 기반 주차 버튼, 실시간 상태 확인, DB 연동 UI 제공
* InfluxDB에 차량 입출차 시간, 위치, 차종, 번호 기록
* 차량 번호 검색 및 시간 필터 기능 제공

---

## 5. 핵심 코드 예시

### `detect_ps_front.py`

```python
# waypoint 도착 후 마킹하기 위해 액션 피드백 확인
while not self.navigator.isTaskComplete():
    feedback = self.navigator.getFeedback()
    ...
# 도착 완료 후 YOLO 좌표 → Depth 추정 → TF 변환 → PoseStamped 발행
```

### `detect_car_info.py`

```python
# YOLO 객체 탐지 후 번호판 영역 crop
# EasyOCR → 정규표현식 필터 적용
if re.match(r"\d{2,3}[가-힣]\d{4}", plate_text):
    ...
```

---

## 6. 도전 과제 및 해결

| 도전 과제       | 해결 방법                                            |
| ----------- | ------------------------------------------------ |
| OCR 인식률 저하  | EasyOCR 결과에 정규표현식 필터 적용                          |
| Depth 오차    | 중심 window(3x3) 평균값 + baselink offset 보정          |
| 시스템 부하 및 지연 | YOLOv8n, 압축 이미지(compressed) 전송, CPU/Net 부하 로깅    |
| 탐지 중복 방지    | 액션 피드백 기반 waypoint 도달 시점만 마킹 (BasicNavigator 활용) |

---

## 7. 협업 및 역할 정리

* YOLO 학습 후 주차 공간 객체 탐지 정확도 향상
* 시스템 부하 측정 코드를 직접 작성해 로그 기반 경량화 검증 진행
* Depth-TF 좌표 변환 과정에서 위치 오차 보정 로직 제안 및 구현
* OCR → GUI → Navi 전체 연동 과정에서 인터페이스 구조 통합

---

## 8. 성과 및 결과물

* 번호판 인식률 90% 이상, 정밀 주차 좌표 오차 ±10cm 내 도달
* GUI + OCR + Navi + TF + YOLO 통합 파이프라인 완성
* 시스템 부하 측정 및 경량화로 CPU/네트워크 사용량 최대 90% 절감
* 입차 \~ 출차 전과정 자동화 시연 성공

---

## 9. 개인적 성장 및 느낀 점

* 센서 융합과 SLAM 기반 로봇의 통합 구조 설계 역량 향상
* 자율주행 제어 시스템에서 액션 서버 및 피드백 활용에 대한 이해
* 시스템 성능을 수치 기반으로 분석하고 개선하는 경험 확보

---

## 🔧 향후 개선 및 확장 방향

* SLAM 맵 자동 갱신 기능 도입으로 환경 변화 대응력 향상
* 차량 등록제 기반 예약 주차 기능 연동
* CCTV + YOLO 연동 통한 보안 강화 및 진입차량 감시
* 음성 명령 기반 입출차 기능으로 고령자 접근성 향상
* 세대별 주차 공간 등록 기반 주차 분쟁 해소 기능 탑재

---

이 포트폴리오는 마크다운 또는 PDF 형식으로 제출/활용 가능합니다. 수정·보완할 부분 있으면 알려주세요!

