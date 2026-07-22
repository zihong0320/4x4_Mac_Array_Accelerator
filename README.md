# 🚀 4x4 MAC Array Accelerator

> **2개의 Weight를 저장하는 MAC 구조와 Weight Stationary 기법을 적용한 4x4 Systolic Array 기반 행렬 곱셈 가속기**

---

## 📌 0. Summary

### 🎯 Abstract
* **Multi-Weight MAC Structure & Weight Stationary Method**: 1개 PE 내에 2개의 Weight를 저장하는 구조 및 Weight Stationary 기법을 적용한 $4 \times 4$ Systolic Array 행렬 곱셈 가속기(Multiplication Unit)
* **Flexible Memory Control**: INPUT, WEIGHT, OUTPUT에 각각 독립된 메모리 제어 로직을 배치하여 최대 $8 \times 8$ 크기의 두 행렬 곱셈을 자동으로 수행하고 결과를 정렬·저장하는 하드웨어 시스템 설계

### 🛠️ Tool & Language & Technique
* **EDA & Tools:** ModelSim, Synopsys Design Compiler (Logic Synthesis)
* **Languages:** Verilog HDL, C Language
* **Technique:** Systolic Array (Weight Stationary), Dataflow Optimization, Golden Reference Model

### 👨‍💻 Main Contributions
* 2개의 Weight를 저장하는 MAC 구조 및 Weight Stationary 기법 적용
* Datapath & Pipeline Timing 설계
* C 언어 기반 Golden Reference Model 구축 및 기능 검증

---

## 📚 1. Instruction & Background

### 1.1 Systolic Array 개요

<p align="center">
  <img src="https://github.com/user-attachments/assets/d76ddf4b-6d12-475d-b9a3-a9bac64a9ad7" width="70%" alt="Dataflow Systolic Array">
</p>

* **Temporal Architecture (SIMD/SIMT):** CPU와 GPU에서 사용하는 연산 구조
* **Spatial Architecture (Dataflow Processing):** TPU, NPU 등 DNN Accelerator에서 사용하는 연산 구조
* **Processing Element (PE):**
  * Data reuse를 극대화하기 위해 PE를 활용하여 연산 배열을 구성함
  * PE는 MAC 연산을 수행하는 연산자(ALU)와 재사용 데이터를 저장하는 Local Memory로 구성됨

<p align="center">
  <img src="https://github.com/user-attachments/assets/68173c43-2577-4690-8469-de1c022bf8d5" width="70%" alt="Weight vs Output Stationary">
</p>

* **(a) Weight Stationary (본 프로젝트 채택):** Weight 데이터를 PE 내부 레지스터에 고정시키고 Input 데이터를 스티리밍하여 연산 수행
* **(b) Output Stationary:** Partial Sum(출력 결과)을 PE 내부에 고정시키고 Input과 Weight를 이동시키며 연산 수행

---

### 1.2 변수 정의 (T, N, M)

<p align="center">
  <img src="https://github.com/user-attachments/assets/a5d2b361-01e5-438a-939c-f73101924b61" width="90%" alt="Variable Definition">
</p>

| Variable | Description |
| :---: | :--- |
| **$T$** | Input Matrix의 행(Row) 크기 / Output Matrix의 행(Row) 크기 |
| **$N$** | Input Matrix의 열(Column) 크기 = Weight Matrix의 행(Row) 크기 |
| **$M$** | Weight Matrix의 열(Column) 크기 / Output Matrix의 열(Column) 크기 |

---

## 🏗️ 2. Hardware Architecture

### 2.1 전체 시스템 블록 다이어그램 (Block Diagram)

<p align="center">
  <img src="https://github.com/user-attachments/assets/4300ab39-9542-4c9d-8f96-e28b33989663" width="85%" alt="System Block Diagram">
</p>

* Datapath와 Control Unit을 명확히 분리하여 확장성과 제어 효율성을 극대화했습니다.

---

### 2.2 기존 구조의 한계점 및 개선책

* **기존 구조의 한계:**
  메모리 접근 단위(8개 데이터, 64비트)와 실제 연산 유닛의 처리 단위(4개 데이터, 32비트)가 불일치하여 Dataflow 제어가 복잡해지고, 다량의 임시 레지스터 선언이 강제됨.
* **개선된 구조 (Dual-Weight Pre-loading):**
  1개 PE 내에 2개의 Weight를 저장하는 구조와 Weight Stationary 기법을 적용. 한 번의 메모리 접근(64비트)으로 8개의 Weight를 각 MAC에 미리 저장함으로써 Control Unit의 상태(State)를 줄이고 전체 Dataflow를 단순화함.

---

### 2.3 MAC, MAC ROW, MAC ARRAY 구조

Datapath는 아래의 계층적 구조로 인스턴스화되어 구성됩니다:

$$\text{doublePE (MAC)} \xrightarrow{\times 4} \text{PErow (MAC Row)} \xrightarrow{\times 4} \text{PEarray (Datapath)}$$

#### 2.3.1 MAC (Processing Element)
<p align="center">
  <img src="https://github.com/user-attachments/assets/3776c6cb-2104-4ecc-a0b6-2f0181fe8c5f" width="50%" alt="MAC Structure">
</p>

* 하나의 MAC에 2개의 Weight를 저장해 한 번에 최대 16비트를 로딩하도록 구현함.

#### 2.3.2 MAC ROW
<p align="center">
  <img src="https://github.com/user-attachments/assets/0e23be81-7c6e-4a69-ac4d-8ec65082f8f0" width="85%" alt="MAC ROW Structure">
</p>

* Weight Stationary 방식에 맞춰 한 번에 최대 64비트 Weight를 프리로딩(Pre-loading)함.
* Input 데이터는 한 사이클씩 딜레이되어 입력됨.
* `EN_INPUT1`(상위 32비트 Weight 연산용) 또는 `EN_INPUT2`(하위 32비트 Weight 연산용) 신호가 활성화되면 Clock의 Positive Edge에 맞춰 다음 MAC으로 신호가 전달됨.

##### 💡 MAC ROW 동작 과정
<p align="center">
  <img src="https://github.com/user-attachments/assets/7d7bf4fa-7e18-47a5-8fb0-608d5e6ee413" width="60%"><br>
  <img src="https://github.com/user-attachments/assets/23225857-b5ea-49b6-9976-8f609054650c" width="55%">
  <img src="https://github.com/user-attachments/assets/23225857-b5ea-49b6-9976-8f609054650c" width="55%"><br>
  <img src="https://github.com/user-attachments/assets/e83b8444-4e78-452f-ab8b-36d33b2162b4" width="55%">
  <img src="https://github.com/user-attachments/assets/024edca0-c4fe-4833-91a6-125c8ad71944" width="55%"><br>
  <img src="https://github.com/user-attachments/assets/85c81b55-5f10-423e-9a4c-8e40b3f3ae36" width="55%">
  <img src="https://github.com/user-attachments/assets/f9962a26-ed44-4c9e-a094-244aa9c8307c" width="55%"><br>
  <img src="https://github.com/user-attachments/assets/e0997878-2fb3-4300-ad05-4c5fb82dbfbb" width="55%">
  <img src="https://github.com/user-attachments/assets/56156df0-67e5-47dc-803c-fa1496270ca2" width="55%"><br>
  <img src="https://github.com/user-attachments/assets/3d96ae51-f1d0-49df-8d43-518e00502c34" width="55%">
</p>

#### 2.3.3 MAC ARRAY
<p align="center">
  <img src="https://github.com/user-attachments/assets/bd4ff8c5-5b2a-43c8-883d-72fa8ae883bc" width="90%" alt="MAC Array Structure">
</p>

* 4개의 MAC Row를 세로로 수직 배치하여 Array 형성.
* 64비트 Partial Sum(`IN_OUT`)을 16비트씩 분할하여 PErow별로 한 사이클씩 밀어서 입력받음.
* 타이밍에 맞춰 입력을 지연시켰기 때문에 출력(`OUT_OUTPUT`) 역시 행마다 한 사이클씩 밀려서 출력됨.

##### 💡 MAC ARRAY 출력 동기화
<p align="center">
  <img src="https://github.com/user-attachments/assets/2a1cedd6-47e2-440c-933e-3314a55f0d5e" width="60%" alt="MAC Array Output">
</p>

---

## ⏱️ 3. Pipeline Timing & Latency

Weight가 먼저 고정(Stationary)된 상태에서 $4 \times 4$ 행렬 곱셈 결과가 최종적으로 정렬되어 출력되기까지 **총 7 Cycle**이 소요됩니다.

### 3.1 Latency 분석
* **MAC 단일 유닛:** `1 Cycle` (입력 데이터가 출력으로 나오는 데 걸리는 시간)
* **MAC Row (1행):** `4 Cycle` (첫 4개 입력 데이터의 합산 결과가 도출되는 시간)
* **MAC Array (전체):** `7 Cycle` (최종 $4 \times 4$ 행렬 곱셈 출력이 완료되는 시간)

### 3.2 행(Row)간 데이터 전달 매커니즘
* `IN_I_ROW`와 활성화 신호(`EN_I`)가 아래 행으로 내려갈 때 내부 레지스터에 의해 정확히 1 Cycle씩 지연(Shift)됨.
  * `UPErow1` 출력: Cycle 4에 최초 결과 도출 ($1 \times 4$ 연산 완료)
  * `UPErow2` 출력: Cycle 5에 최초 결과 도출 ($2 \times 4$ 연산 완료)
  * `UPErow3` 출력: Cycle 6에 최초 결과 도출 ($3 \times 4$ 연산 완료)
  * `UPErow4` 출력: Cycle 7에 최초 결과 도출 ($4 \times 4$ 연산 완료)
* 이를 64비트 버스로 동시에 정렬하여 출력하기 위해 하단에 동기화 레지스터를 배치하여 타이밍을 무조건 **Cycle 7**로 통일함.

### 3.3 Valid 신호 동기화
* `UPErow1`의 유효 신호인 `VAL_o[0]` 역시 레지스터를 통해 3 Cycle 지연되어, 데이터 정렬과 정확히 일치하는 Cycle 7 시점에 `OUT_VALID`로 출력됨.

---

## 🔄 4. Control Unit (FSM)

<p align="center">
  <img src="https://github.com/user-attachments/assets/7e6f28f1-fca4-4918-b88e-c744d2f0c04f" width="85%" alt="Control Unit FSM">
</p>

* **INPUT_MEM:** 64비트씩 최대 2회 로드 (`EN_INPUT1`에 따라 상/하위 32비트 Truncate 분기)
* **WEIGHT_MEM:** 한 번에 최대 64비트 로드
* **주요 제어 레지스터:**
  * `EN_INPUT1_PEarray_reg`, `EN_INPUT2_PEarray_reg`: `IN_I`와 연산할 Weight 선택 및 `IN_OUT`과의 덧셈 제어
  * `EN_WEIGHT_SELECT_PEarray_reg`: PEarray 내부 4개의 PErow 중 하나를 선택
  * `EN_WEIGHT_PEarray_reg`: $N$ 크기에 맞게 Weight_row를 활성화하는 신호

---

### 4.1 동작 경우의 수 및 FSM Detailed Flow

#### ① $N \le 4, M \le 4$
<p align="center">
  <img src="https://github.com/user-attachments/assets/7b694f85-eaad-4002-a0cc-7d35fabe1678" width="80%"><br>
  <sub>&lt;INPUT, WEIGHT 관련 FSM&gt;</sub>
</p>
<p align="center">
  <img src="https://github.com/user-attachments/assets/e59a3861-1276-43e2-b663-8e7e2d599696" width="80%"><br>
  <sub>&lt;OUTPUT 관련 FSM&gt;</sub>
</p>
<p align="center">
  <img src="https://github.com/user-attachments/assets/f88f6458-1f0e-4d6f-a241-64d031089220" width="50%">
  <img src="https://github.com/user-attachments/assets/212a11a6-499c-4173-95c0-9f49ee89dfa4" width="30%">
</p>

* `ADDR_W`를 증가시켜 최대 상위 32비트 Weight_row를 저장시킨 후, `ADDR_I`를 증가시켜 최대 상위 32비트 INPUT을 받아와서 연산함.
* Output Address를 0에서 2씩 최대 14까지 증가시켜 `OUT_MEM`에 결과값을 저장함.

---

#### ② $N > 4, M \le 4$
<p align="center">
  <img src="https://github.com/user-attachments/assets/8c0e8edd-22d9-49c8-8566-11cd37eb8755" width="80%"><br>
  <sub>&lt;INPUT, WEIGHT 관련 FSM&gt;</sub>
</p>
<p align="center">
  <img src="https://github.com/user-attachments/assets/6fb504e4-0506-4377-94a9-8852ea5f022f" width="60%"><br>
  <sub>&lt;남은 하위 Input_row와 연산을 다시 진행할 때의 Block Diagram&gt;</sub>
</p>
<p align="center">
  <img src="https://github.com/user-attachments/assets/7a9b9c1a-2f3d-4af0-a6f1-9feeabf35cc5" width="80%"><br>
  <sub>&lt;OUTPUT 관련 FSM&gt;</sub>
</p>
<p align="center">
  <img src="https://github.com/user-attachments/assets/451262cb-d9ac-4866-be8a-db6c7bd33d37" width="35%">
  <img src="https://github.com/user-attachments/assets/9ef961f7-95d6-4c6e-ad70-2f067362ac10" width="35%">
</p>

* `ADDR_W`를 증가시켜 상위 64비트 Weight_row를 받아 저장한 뒤, `ADDR_I`를 증가시켜 상위 32비트 INPUT_row와 ($W_1 \sim W_4$)를 연산함.
* 이후 `ADDR_I`를 초기화하여 다시 증가시키면서 하위 Weight_row($W_5 \sim W_8$)와 하위 32비트 INPUT_row를 연산함.
* 하위 Input_row 연산 시에는 이전에 1차 저장되었던 Output 값을 `IN_OUT`으로 피드백받아 Partial Sum을 누적 계산하여 업데이트함.

---

#### ③ $N \le 4, M > 4$
<p align="center">
  <img src="https://github.com/user-attachments/assets/3ae0727e-9837-4aa1-9eae-ceeb4dc79dd4" width="80%"><br>
  <sub>&lt;INPUT, WEIGHT 관련 FSM&gt;</sub>
</p>
<p align="center">
  <img src="https://github.com/user-attachments/assets/18428257-b5ea-49b6-9976-8f609054650c" width="80%"><br>
  <sub>&lt;OUTPUT 관련 FSM&gt;</sub>
</p>
<p align="center">
  <img src="https://github.com/user-attachments/assets/c5e261be-fe7b-4db0-91c0-252a34a7732a" width="70%"><br>
  <sub>&lt;ADDR_W를 5부터 M까지 증가시킨 후 동일 Input과 연산 시 Block Diagram&gt;</sub>
</p>
<p align="center">
  <img src="https://github.com/user-attachments/assets/49126f96-95d6-4c6e-ad70-2f067362ac10" width="35%">
  <img src="https://github.com/user-attachments/assets/dc2dbc36-813a-4cfc-b7d6-42b2b4c562eb" width="35%">
</p>

* `ADDR_W`를 4까지 증가시켜 상위 32비트 Weight_row를 로드한 뒤 INPUT_row와 연산하고, 1차 결과는 `ADDR_O` (0, 2, 4...)에 저장함.
* 이후 `ADDR_W`를 5부터 $M$까지 증가시켜 Weight_row를 교체하고 동일한 INPUT_row와 다시 연산하여, 결과는 `ADDR_O` 홀수 번지(1, 3, 5...)에 저장함.

---

#### ④ $N > 4, M > 4$
<p align="center">
  <img src="https://github.com/user-attachments/assets/afb282c9-5c11-4628-abc3-3d89c1a1e220" width="85%"><br>
  <sub>&lt;INPUT, WEIGHT 관련 FSM&gt;</sub>
</p>
<p align="center">
  <img src="https://github.com/user-attachments/assets/24b6a027-5e0f-4f1e-bda2-d803264b0a60" width="85%"><br>
  <sub>&lt;OUTPUT 관련 FSM&gt;</sub>
</p>
<p align="center">
  <img src="https://github.com/user-attachments/assets/cdf520d1-40bd-4d91-850e-141a26ba7cbd" width="23%">
  <img src="https://github.com/user-attachments/assets/80e295fa-b81a-4202-8c4c-039673f4cede" width="23%">
  <img src="https://github.com/user-attachments/assets/4d1a2ef3-dffd-4366-999e-891d30288d54" width="23%">
  <img src="https://github.com/user-attachments/assets/cab2026c-e20b-4161-b5d0-2d331633f3cd" width="23%">
</p>

* $N > 4, M \le 4$ 케이스를 우선 수행하여 Partial Sum 누적을 완료함.
* 그 후 $M > 4$ 영역에 대해 `ADDR_W`를 5부터 $M$까지 증가시키면서 동일한 누적 연산 프로세스를 재귀적으로 반복 진행함.

---

## 📊 5. Verification & Synthesis Results

### 5.1 시뮬레이션 결과 검증 ($T, N, M = 8$ 예시)

C 언어 기반의 Golden Reference Model을 자체 제작하여 Verilog Testbench 결과와 $1:1$ 비트 일치 여부를 검증했습니다.

<p align="center">
  <img src="https://github.com/user-attachments/assets/1e515b6c-2c82-4bbe-a468-e33e44fc7ed2" width="80%"><br>
  <sub>&lt;8x8 행렬 곱셈 검증 케이스&gt;</sub>
</p>

| ModelSim Testbench Result | C Golden Reference Result |
| :---: | :---: |
| <img src="https://github.com/user-attachments/assets/601bd0ee-be22-4a89-a5ac-33ba2ee17bd9" width="100%"> | <img src="https://github.com/user-attachments/assets/988575f3-85e3-4028-8755-125c8ad71944" width="100%"> |

* **검증 결과:** 모든 Testcase에서 Golden Reference 데이터와 **100% 일치하는 연산 결과**를 확인했습니다.

---

### 5.2 Golden Reference Model Code
<p align="center">
  <img src="https://github.com/user-attachments/assets/d2fcc537-1a51-4a8c-9b18-0cc6c93d1d4b" width="45%">
  <img src="https://github.com/user-attachments/assets/85e3ab21-0213-439a-9fd3-747de154c81a" width="45%">
</p>

---

### 5.3 논리 합성(Logic Synthesis) 결과

제공된 `.sdc` (Synopsys Design Constraints) 파일을 기반으로 Design Compiler 합성을 수행하였습니다.

<p align="center">
  <img src="https://github.com/user-attachments/assets/827620f3-45ef-45fc-aa5b-4e9881c7b555" width="50%"><br>
  <sub>&lt;SDC Constraints File&gt;</sub>
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/1104651f-43fe-47b2-8b50-1b05bbf1d14e" width="48%">
  <img src="https://github.com/user-attachments/assets/1179652f-43fe-47b2-8b50-1b05bbf1d14e" width="48%"><br>
  <sub>&lt;Synthesis Resource & Timing Results&gt;</sub>
</p>

* **최대 동작 주파수 ($F_{\max}$):** **76.37 MHz** (Estimated)
* **하드웨어 리소스 사용량:**
  * **Embedded Multiplier 9-bit elements:** 16개
  * **Total Logic Elements:** **1,857개** (고효율/저면적 구조 달성)

> **💡 Critical Warnings 분석 및 해소**
> 1. **285개 Unconnected I/O Port Warning:** 실제 보드 핀 매핑(Pin Assignment) 전단계 합성 진행으로 인해 발생한 미연결 경고.
> 2. **4개 Clock Uncertainty Warning:** `.sdc` 상에서 Uncertainty 제약 조건이 미지정되어 기본값 할당에 의해 발생한 경고로 하드웨어 연산에는 지장 없음.
