
# CAN-based Driver Behavior Analysis System


<img width="500" height="400" alt="image" src="https://github.com/user-attachments/assets/19767d39-d7c6-476d-a1ff-11ada8712abc" />
<img width="500" height="400" alt="image" src="https://github.com/user-attachments/assets/0379f442-2345-4963-afed-392d6f3383bd" />

https://github.com/user-attachments/assets/672f6e51-8fd5-48cc-b3f2-25eaf2b3af77

본 프로젝트는 차량 **OBD-II 기반 CAN 통신**을 통해 주행 데이터를 수집하고,  
운전 행동을 분석하여 **운전자 성향(캐릭터)** 및 **주행 점수(SCORE)** 를 실시간으로 시각화하는  
**임베디드 차량 데이터 분석 시스템**입니다.


단순 PID 조회 방식이 아닌 **UDS + ISO-TP 기반 ECU 내부 데이터 접근 구조**를 직접 구현하여 확장된 차량 신호를 처리하도록 설계했습니다.

---

## 프로젝트 개요
<img width="900" height="444" alt="image" src="https://github.com/user-attachments/assets/fd5d0479-3698-4ce0-a1df-26d1ba9e898b" />

차량 OBD-II 포트를 통해 CAN 프레임을 수신하고,  
ISO-TP 멀티프레임 재조립 및 UDS 응답 Payload 파싱을 통해  
속도, RPM, 조향각, 브레이크 데이터를 실시간으로 추출합니다.

수집된 주행 데이터는 필터링 및 변화량 분석을 거쳐  
운전 행동 지표로 변환되며,  
점수 계산 알고리즘을 통해 운전자 성향을 분류하고 TFT 디스플레이로 출력됩니다.

---
## Hardware 구성
<img width="900" height="444" alt="image" src="https://github.com/user-attachments/assets/b8395bdf-d8b7-4a94-907f-644b1decf1c0" />


| 구분 | 부품명 | 역할 |
| :--- | :--- | :--- |
| **Vehicle Interface** | **OBD-II Port** | 차량 CAN 데이터 수신 |
| **CAN Controller** | **MCP2515** | CAN 프레임 송수신 처리 |
| **Main Board** | **Arduino Nano Mini** | 데이터 파싱 및 점수 로직 처리 |
| **Display** | **2.2" TFT LCD (ILI9341)** | 실시간 주행 정보 UI 출력 |
| **Storage (Option)** | **microSD Module** | 캐릭터 이미지 리소스 저장 |

---
## CAN Data (UDS + ISO-TP)
<img width="900" height="444" alt="image" src="https://github.com/user-attachments/assets/ecbd03a5-29ca-4214-a8cc-6aaa1ef22d7b" />

### 1. ECU Message Selection

- KIA NIRO CAN message table 기반 분석
- ABS / ESP ECU 대상
- UDS Service `0x22 (ReadDataByIdentifier)` 사용

ISO-TP 멀티프레임을 재조립한 뒤 Byte Offset 기반 필드 매핑을 통해 실제 물리값으로 변환합니다.

### 2. Request Frame Format

**Request ID : `0x7D1`**

| Field | Value |
|------|------|
| Service ID | `0x22` |
| DID | `0xC101` |

### 3. Response Payload Mapping

**Response ID : `0x7D9`**

| Payload Index | Signal | Description |
|---------------|--------|-------------|
| [10] | Vehicle Speed | 속도(km/h) |
| [33], [34] | Steering Angle | 핸들 조향각(High / Low byte) |
| [37] | Brake Pedal | 브레이크 |
---

## Driving Behavior Logic
<img width="900" height="444" alt="image" src="https://github.com/user-attachments/assets/e2542a1e-7b76-433e-a3b5-cb66b1fccdb6" />

### Risk Behavior Penalty (위험 운전 감점)

- 급가속 (Speed 증가율 / RPM 상승률 기반)
- 급제동 (Brake 변화량 + 주행 속도 조건)
- 급조향 (조향각 변화량)
- 고RPM 지속 운전
- 복합 이벤트 동시 발생 시 가중 감점 적용

### Flow Interference Penalty (교통 흐름 방해 감점)

- 불필요한 저속 유지
- 평균 대비 과도한 감속
- 감속 후 회복 지연

### Traffic Jam Protection Logic (정체 구간 보호)

- 정체 구간 판단 시 저속 감점 비활성화
- 위험 이벤트 기준 완화 적용
- 정체 해제 시 정상 평가 모드 복귀

---
## Driver Type Classification

캐릭터 UI를 SD Card 기반 외부 리소스로 분리하여 **사용자 맞춤형 이미지 커스터마이징 가능**


<img width="500" height="400" alt="image" src="https://github.com/user-attachments/assets/2ae22458-fb11-4eb7-b7ea-cd8b07eaa3eb" />


