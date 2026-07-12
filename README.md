# 4x4_Mac_Array_Accelerator 설계
4x4 Matrix Multiplication Unit

0. Summary
2개의 Weight를 저장하는 Mac구조와 Weight Stationary 기법을 적용한 4x4 Systolic Array 구조의 행렬 곱셈 가속기(Multiplication Unit)
INPUT, WEIGHT, OUTPUT 각각의 독립된 메모리를 제어하여 최대 8x8 크기의 두 행렬 곱셈을 자동으로 수행하고 결과를 저장하는 하드웨어 시스템으로 구성


1. Instruction

1.1 Systolic Array 개요
1.2 변수 정의 (T, N, M)


2. 하드웨어 아키텍처

2.1 전체 시스템 블록 다이어그램 (Block Diagram)
<img width="1493" height="700" alt="image" src="https://github.com/user-attachments/assets/4300ab39-9542-4c9d-8f96-e28b33989663" />
본 설계는 Datapath와 Control Unit을 명확히 분리하여 확장성과 제어 효율성을 극대화했습니다.


2.2 기존 구조의 한계점 및 개선책
단점 (기존 구조): 메모리 접근 단위(8개 데이터, 64비트)와 실제 연산 유닛의 처리 단위(4개 데이터, 32비트)가 불일치하여 Dataflow 제어가 복잡해지고, 다량의 임시 레지스터 선언이 강제됨.

해결책 (개선된 구조): 1개 PE 내에 2개의 Weight를 저장하는 구조와 Weight Stationary 기법을 융합. 한 번의 메모리 접근(64비트)으로 8개의 Weight를 각 MAC에 미리 저장함으로써 Control Unit의 상태(State)를 줄이고 전체 Dataflow를 단순화함.



2.3 MAC, MAC ROW, MAC ARRAY 구조
Datapath는 아래의 방식으로 인스턴스화되어 구성됩니다.
$$\text{doublePE} \xrightarrow{\times 4} \text{PErow} \xrightarrow{\times 4} \text{PEarray (Datapath)}$$


