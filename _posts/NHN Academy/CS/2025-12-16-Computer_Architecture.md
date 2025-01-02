---
author: 김규형
date: 2023-09-12 14:10:00 +0800
categories: [NHN Academy, CS]
tags: [CS, NHN Acedemy]
render_with_liquid: true
---

# Computer Architecture (23.09.04)

## CPU의 동작

### Memory

- **보조 기억장치 ( HDD, SDD ), 주 기억장치( RAM ), Cache Memory**

### Cache Memory

- L1 → L2 → L3 → RAM 순으로 저장 장소 탐색

### Register

- 저장된 주소의 데이터 저장, 로드등의 정보 저장

### CPU의 4가지 동작

- Fetch (인출)
- Decode (해석)
- Execute (실행)
- Store (저장)

## 프로그램의 동작원리

### 16bit vs 32 bit vs 64 bit

- xx 비트 → 한번에 xx개의 비트를 처리량을 뜻함. ( 32 → 4, 64 → 8 )

### Native Code의 동작

- PC(Program counter) 가 주소를 전달, AP(Address Register) 가 명령어 해석 BR(Buffer Regitster) 이 Fetch IR(Instruction Register)가 CU(Control Uinit)에 전달 그리고 ALU에서 각자 명령 수행 (ADD, LOAD, STORE) 등등…
