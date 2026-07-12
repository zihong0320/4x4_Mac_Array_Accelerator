# 4x4_Mac_Array_Accelerator 설계
4x4 Matrix Multiplication Unit

Summary
2개의 Weight를 저장하는 Mac구조와 Weight Stationary 기법을 적용한 4x4 Systolic Array 구조의 행렬 곱셈 가속기(Multiplication Unit)
INPUT, WEIGHT, OUTPUT 각각의 독립된 메모리를 제어하여 최대 8x8 크기의 두 행렬 곱셈을 자동으로 수행하고 결과를 저장하는 하드웨어 시스템으로 구성


1. Instruction

1.1 Systolic Array 개요
1.2 변수 정의 (T, N, M)

2. 하드웨어 아키텍처

2.1 전체 시스템 블록 다이어그램 (Block Diagram)
<img width="1493" height="700" alt="image" src="https://github.com/user-attachments/assets/4300ab39-9542-4c9d-8f96-e28b33989663" />
본 설계는 Datapath와 Control Unit을 명확히 분리하여 확장성과 제어 효율성을 극대화했습니다.
