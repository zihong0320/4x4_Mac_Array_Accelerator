# 4x4_Mac_Array_Accelerator 설계
4x4 Matrix Multiplication Unit

## 0. Summary
   - 2개의 Weight를 저장하는 Mac구조와 Weight Stationary 기법을 적용한 4x4 Systolic Array 구조의 행렬 곱셈 가속기(Multiplication Unit)
   - INPUT, WEIGHT, OUTPUT 각각의 독립된 메모리를 제어하여 최대 8x8 크기의 두 행렬 곱셈을 자동으로 수행하고 결과를 저장하는 하드웨어 시스템으로 구성


## 1. Instruction

   ### 1.1 Systolic Array 개요
   
   ### 1.2 변수 정의 (T, N, M)


## 2. 하드웨어 아키텍처

   ### 2.1 전체 시스템 블록 다이어그램 (Block Diagram)
   <img width="1493" height="700" alt="image" src="https://github.com/user-attachments/assets/4300ab39-9542-4c9d-8f96-e28b33989663" />
   본 설계에서는 Datapath와 Control Unit을 명확히 분리하여 확장성과 Control 효율을 극대화


   ### 2.2 기존 구조의 한계점 및 개선책
   - 단점 (기존 구조): 메모리 접근 단위(8개 데이터, 64비트)와 실제 연산 유닛의 처리 단위(4개 데이터, 32비트)가 불일치하여 Dataflow 제어가 복잡해지고, 다량의 임시 레지스터 선언이 강제됨

   - 해결책 (개선된 구조): 1개 PE 내에 2개의 Weight를 저장하는 구조와 Weight Stationary 기법을 적용. 한 번의 메모리 접근(64비트)으로 8개의 Weight를 각 MAC에 미리 저장함으로써 Control Unit의 상태(State)를 줄이고 전체 Dataflow를 단순화



   ### 2.3 MAC, MAC ROW, MAC ARRAY 구조
   Datapath는 아래의 방식으로 인스턴스화되어 구성됨.
   $$\text{doublePE} \xrightarrow{\times 4} \text{PErow} \xrightarrow{\times 4} \text{PEarray (Datapath)}$$

   #### 2.3.1 MAC
   <img width="827" height="418" alt="image" src="https://github.com/user-attachments/assets/3776c6cb-2104-4ecc-a0b6-2f0181fe8c5f" />

   MAC 구조 : 하나의 MAC에 2개의 Weight를 저장해 한 번에 최대 16비트를 로딩하도록 구현


   #### 2.3.2 MAC ROW
   <img width="1863" height="608" alt="image" src="https://github.com/user-attachments/assets/0e23be81-7c6e-4a69-ac4d-8ec65082f8f0" />

   MAC Row 구조 및 입력 제어: Weight Stationary 방식에 맞춰 한 번에 최대 64비트 Weight를 프리로딩(Pre-loading)

   Input 데이터는 한 사이클씩 딜레이되어 입력됨

   EN_INPUT1(상위 32비트 Weight 연산용) 또는 EN_INPUT2(하위 32비트 Weight 연산용) 신호가 활성화되면 Clock의 Positive Edge에 맞춰 다음 MAC으로 신호가 전달됨

      2.3.2-1 MAC ROW 동작과정
   <img width="1555" height="591" alt="image" src="https://github.com/user-attachments/assets/7d7bf4fa-7e18-47a5-8fb0-608d5e6ee413" />
      <img width="1288" height="541" alt="image" src="https://github.com/user-attachments/assets/23225857-b5ea-49b6-9976-8f609054650c" />
      <img width="1288" height="541" alt="image" src="https://github.com/user-attachments/assets/dab45b5c-6c1d-41be-bce7-3842e4cb62d2" />
      <img width="1288" height="541" alt="image" src="https://github.com/user-attachments/assets/e83b8444-4e78-452f-ab8b-36d33b2162b4" />
      <img width="1288" height="497" alt="image" src="https://github.com/user-attachments/assets/024edca0-c4fe-4833-91a6-125c8ad71944" />
      <img width="1558" height="591" alt="image" src="https://github.com/user-attachments/assets/85c81b55-5f10-423e-9a4c-8e40b3f3ae36" />
      <img width="1288" height="541" alt="image" src="https://github.com/user-attachments/assets/f9962a26-ed44-4c9e-a094-244aa9c8307c" />
      <img width="1288" height="541" alt="image" src="https://github.com/user-attachments/assets/e0997878-2fb3-4300-ad05-4c5fb82dbfbb" />
      <img width="1288" height="541" alt="image" src="https://github.com/user-attachments/assets/56156df0-67e5-47dc-803c-fa1496270ca2" />
      <img width="1288" height="497" alt="image" src="https://github.com/user-attachments/assets/3d96ae51-f1d0-49df-8d43-518e00502c34" />




   #### 2.3.3 MAC ARRAY
   <img width="1967" height="882" alt="image" src="https://github.com/user-attachments/assets/bd4ff8c5-5b2a-43c8-883d-72fa8ae883bc" />
   
   - MAC Array 구조 및 출력 동기화:
   
         - 4개의 MAC Row을 통해 Array를 형성
         
         - 64비트 Partial Sum(IN_OUT)을 16비트씩 분할하여 PErow별로 한 사이클씩 밀어서 입력받음
         
         - 타이밍에 맞춰 입력을 지연시켰기 때문에 출력(OUT_OUTPUT) 역시 행마다 한 사이클씩 밀려서 출력됨


      **2.3.2-2 MAC ARRAY 출력
      <img width="1520" height="802" alt="image" src="https://github.com/user-attachments/assets/2a1cedd6-47e2-440c-933e-3314a55f0d5e" />
      


  ## 3. 파이프라인 타이밍 설계 (Pipeline Timing)
      Weight가 먼저 고정(Stationary)된 상태에서 $4 \times 4$ 행렬 곱셈 결과가 최종적으로 정렬되어 출력되기까지 총 7 Cycle이 소요
      
   ### 3.1 Latency 분석
      - MAC 단일 유닛: 1 Cycle (입력 데이터가 출력으로 나오는 데 걸리는 시간)
      - MAC Row (1행): 4 Cycle (첫 4개 입력 데이터의 합산 결과가 도출되는 시간)
      - MAC Array (전체): 7 Cycle (최종 $4 \times 4$ 행렬 곱셈 출력이 완료되는 시간)
      
   ### 3.2 행(Row)간 데이터 전달 매커니즘
      IN_I_ROW와 활성화 신호(EN_I)가 아래 행으로 내려갈 때 내부 레지스터에 의해 정확히 1 Cycle씩 지연(Shift)됨. 이로 인해 각 행의 연산 완료 시점이 다르게 나타나게 됨.
      - UPErow1 출력: Cycle 4에 최초 결과 도출 ($1 \times 4$ 완료)
      - UPErow2 출력: Cycle 5에 최초 결과 도출 ($2 \times 4$ 완료)
      - UPErow3 출력: Cycle 6에 최초 결과 도출 ($3 \times 4$ 완료)
      - UPErow4 출력: Cycle 7에 최초 결과 도출 ($4 \times 4$ 완료)이를 64비트 버스로 동시에 정렬하여 출력하기 위해 하단에 동기화 레지스터를 배치하여 타이밍을 무조건 Cycle 7로 통일
   
   
   ### 3.3 Valid 신호 동기화
       UPErow1의 유효 신호인 VAL_o[0] 역시 레지스터를 통해 3 Cycle 지연되어, 데이터 정렬과 정확히 일치하는 Cycle 7 시점에 OUT_VALID로 출력
   
   
   ## 4. Control Unit(FSM)
   <img width="1797" height="717" alt="image" src="https://github.com/user-attachments/assets/7e6f28f1-fca4-4918-b88e-c744d2f0c04f" />
   
   - INPUT_MEM: 64비트씩 최대 2회 로드 (EN_INPUT1에 따라 상/하위 32비트 Truncate 분기)
   
   - WEIGHT_MEM: 한 번에 최대 64비트 로드
   
   - 제어 레지스터
     - EN_INPUT1_PEarray_reg와 EN_INPUT2_PEarray_reg는 연산 진행을 위한 signal(IN_I와 연산할 weight 선택, IN_OUT과의 덧셈)
     - EN_WEIGHT_SELECT_PEarray_reg는 PEarray에서 4개의 PErow 중 하나를 선택
     - EN_WEIGHT_PEarray_reg는 N에 맞게 Weight_row를 활성화시키는 signal
    
      ### 4.1 동작 경우의 수 및 FSM
     #### ① N≤4, M≤4
        <img width="1452" height="725" alt="image" src="https://github.com/user-attachments/assets/7b694f85-eaad-4002-a0cc-7d35fabe1678" />
         <INPUT, WEIGHT 관련 FSM>



        <img width="1518" height="810" alt="image" src="https://github.com/user-attachments/assets/e59a3861-1276-43e2-b663-8e7e2d599696" />
        <OUTPUT 관련 FSM>


        <img width="775" height="516" alt="image" src="https://github.com/user-attachments/assets/f88f6458-1f0e-4d6f-a241-64d031089220" />

        <img width="451" height="545" alt="image" src="https://github.com/user-attachments/assets/212a11a6-499c-4173-95c0-9f49ee89dfa4" />

         - N≤4, M≤4의 경우, ADDR_W를 증가시켜, 최대 상위 32비트 Weight_row를 받아 저장시킨 후, ADDR_I를 증가시켜 최대 상위 32비트 INPUT을 받아와서 연산시킴.
         - output address를 0에서 2씩 최대 14까지 증가시켜서 OUT_MEM에 결과값을 저장함.


     #### ② N>4, M≤4
        <img width="1654" height="831" alt="image" src="https://github.com/user-attachments/assets/8c0e8edd-22d9-49c8-8566-11cd37eb8755" />
        <INPUT, WEIGHT 관련 FSM>

        <img width="1096" height="414" alt="image" src="https://github.com/user-attachments/assets/6fb504e4-0506-4377-94a9-8852ea5f022f" />
        <남은 하위 Input_row와 연산을 다시 진행할 때의 Block diagram>


        <img width="1772" height="816" alt="image" src="https://github.com/user-attachments/assets/7a9b9c1a-2f3d-4af0-a6f1-9feeabf35cc5" />
        <OUTPUT 관련 FSM>

        <img width="451" height="547" alt="image" src="https://github.com/user-attachments/assets/510652cb-d9ac-4866-be8a-db6c7bd33d37" />
        <img width="445" height="544" alt="image" src="https://github.com/user-attachments/assets/9ef961f7-95d6-4c6e-ad70-2f067362ac10" />

         - N>4, M≤4인 경우에는 ADDR_W를 증가시켜 최대 상위 64비트 Weight_row를 받아 저장시킨 후, ADDR_I를 증가시켜 상위 32비트 INPUT_row와 w1, w2, w3, w4와 연산하고, ADDR_I를 초기화한 후 다시 ADDR_I를 증가시켜 다른 Weight_row(w5, w6, w7, w8)와 다시 들어온 최대 하위 32비트 INPUT_row를 연산시킴.
         - 처음 하위 Input_row와 연산을 진행할 때의 나온 결과를, ADDR_O를 2씩 증가시켜, OUT_MEM에 저장하고 하위 32비트 Input_row와 연산을 진행할 때는 처음 저장된 output을 다시 IN_OUT으로 넣어서 함께 partial sum 연산함.
         - output address는 0부터 2씩 최대 14까지 증가시켜서 값 넣어주고 다시 연산 후 0부터 2씩 최대 14까지 증가시켜 같은 address에 output 값을 업데이트함.



      #### ③ N≤4, M>4
         <img width="1884" height="707" alt="image" src="https://github.com/user-attachments/assets/3ae0727e-9837-4aa1-9eae-ceeb4dc79dd4" />
         <INPUT, WEIGHT 관련 FSM>

         <img width="1842" height="825" alt="image" src="https://github.com/user-attachments/assets/9ff80f1d-1fea-4ddf-8beb-03dba0ad2f18" />
         <OUTPUT 관련 FSM>
     

         <img width="1410" height="501" alt="image" src="https://github.com/user-attachments/assets/c5e261be-fe7b-4db0-91c0-252a34a7732a" />
         <ADDR_W를 5부터 M까지 증가시킨 후, 다시 기존의 상위 최대 32비트 input과 연산을 진행할 때의 Block diagram>


         <img width="491" height="598" alt="image" src="https://github.com/user-attachments/assets/26f969d1-3e31-45d1-a7d5-bcd28711868b" />
         <img width="493" height="599" alt="image" src="https://github.com/user-attachments/assets/dc2dbc36-813a-4cfc-b7d6-42b2b4c562eb" />


         - N≤4, M>4인 경우에는 ADDR_W를 4까지 증가시켜 상위 32비트 Weight_row를 받아 저장시킨 후, ADDR_I를 증가시켜 최대 상위 32비트 INPUT_row와 연산시킴. 그 후, ADDR_W를 5부터 M까지 증가시켜 Weight_row를 바꿔주고, 다시 초기화하여 ADDR_I를 증가시켜 전에 연산한 INPUT_row와 동일한 값을 갖는 INPUT_row와 연산시킴.
      - output address는 0부터 2씩 최대 14까지 먼저 증가시켜 저장함.
      - 그 후, 새로운 Weight_row와의 연산은 output address를 1부터 2씩 증가해 최대 15까지 증가시켜 OUT_MEM에 저장함.


         #### ④ N>4, M>4
         <img width="2002" height="628" alt="image" src="https://github.com/user-attachments/assets/afb282c9-5c11-4628-abc3-3d89c1a1e220" />
         <INPUT, WEIGHT 관련 FSM>

         <img width="2002" height="628" alt="image" src="https://github.com/user-attachments/assets/24b6a027-5e0f-4f1e-bda2-d803264b0a60" />
         <OUTPUT 관련 FSM>


         <img width="493" height="597" alt="image" src="https://github.com/user-attachments/assets/cdf520d1-40bd-4d91-850e-141a26ba7cbd" />
         <img width="491" height="600" alt="image" src="https://github.com/user-attachments/assets/80e295fa-b81a-4202-8c4c-039673f4cede" />
         <img width="495" height="597" alt="image" src="https://github.com/user-attachments/assets/4d1a2ef3-dffd-4366-999e-891d30288d54" />
         <img width="493" height="597" alt="image" src="https://github.com/user-attachments/assets/cab2026c-e20b-4161-b5d0-2d331633f3cd" />

         - N>4, M≤4의 경우를 먼저 진행시킴.
         - 그 후, M>4인 경우에 대해 ADDR_W를 5부터 M까지 증가시켜 Weight_row를 바꿔주고, 다시 N>4, M≤4인 경우를 순차적으로 진행시킴



   ## 5. 검증 및 합성 결과 (Verification & Synthesis)

   ### 5.1 시뮬레이션 결과 검증 ($T, N, M = 8$ 예시)
      하드웨어 설계의 정확성을 검증하기 위해 C 언어 기반의 Golden Reference Model을 구축하고 Testbench 결과와 비교 분석하였습니다.


         <img width="1806" height="333" alt="image" src="https://github.com/user-attachments/assets/1e515b6c-2c82-4bbe-a468-e33e44fc7ed2" />
         <8x8 행렬 곱셈>

         <img width="914" height="571" alt="image" src="https://github.com/user-attachments/assets/601bd0ee-be22-4a89-a5ac-33ba2ee17bd9" />
         <Testbench code 결과>

         <img width="988" height="575" alt="image" src="https://github.com/user-attachments/assets/9c26812b-f674-4091-b73e-8c8b115e3b22" />
         <Golden Reference Model 결과>


   ### 5.2 Golden model code
         <img width="574" height="678" alt="image" src="https://github.com/user-attachments/assets/d2fcc537-1a51-4a8c-9b18-0cc6c93d1d4b" />

         <img width="502" height="678" alt="image" src="https://github.com/user-attachments/assets/85e3ab21-0213-439a-9fd3-747de154c81a" />


         - Testbench 파형 결과: 모든 Case에서 Golden Reference 데이터와 100% 일치하는 정확한 연산 결과 출력을 확인

   ### 5.3 논리 합성(Logic Synthesis) 결과
         제공된 .sdc (Synopsys Design Constraints) 파일을 기반으로 합성을 진행한 결과 이상 없이 Netlist가 생성됨
         <img width="827" height="620" alt="image" src="https://github.com/user-attachments/assets/45e65fcf-aa5b-4e98-81c7-b5558fe7b809" />
         <sdc 파일>
      
         <img width="1104" height="651" alt="image" src="https://github.com/user-attachments/assets/0afd7836-a3a8-4f24-979e-671f628df2f2" />
         <img width="1179" height="652" alt="image" src="https://github.com/user-attachments/assets/34a7955f-43fe-47b2-8b50-1b05bbf1d14e" />
         <Synthesis 결과>

         - 최대 동작 주파수 (Fmax): 76.37 MHz (예상)

         - 하드웨어 리소스 사용량:

            - Embedded Multiplier 9-bit elements: 16개

            - Total Logic Elements: 1,857개 (고효율/저면적 설계 달성)

         - 보라색 warning들의 명칭은 critical warning인데, 첫 번째는 285개의 I/O port를 board에 연결을 안 해서 나온 warning이고, 나머지 4개의 uncertainty warning은 sdc에서 uncertainty를 따로 지정하지 않아 나온 waring으로 추측됨
     
   



