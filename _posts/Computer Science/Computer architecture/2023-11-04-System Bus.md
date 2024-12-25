---
author: "unggu"
date: 2023-11-04 05:37:22 +0800
categories: [Computer Science, Computer Architecture]
tags: [CS, computer architecture]
render_with_liquid: true
comments: true
---
# System Bus

> 하드웨어의 구성 요소를 물리적으로 연결하는 선
> 

## 데이터 버스

- CPU ↔ 기타 장치 (Memory, I/O..,)
- 양방향 버스

## 주소 버스

- 기억 장치의 주소 / 포트 번호를 전달
- CPU와 Memory 사이에서 기억 장치 주소를 전달하는 단방향 버스

## 제어 버스

- 제어 신호를 전달
    - 주소 버스와 데이터 버스는 모든 장치가 사용하기에 이를 제어 필요
    - read, write 신호, Interrupt, Reset
- 읽기와 쓰기 모두 하기에 양방향 통신