---
title: "Local PC 종속 운영을 Kubernetes로 옮긴 이유와 전환 범위"
summary: "ML 서비스만 옮기는 대신 PostgreSQL·Kafka·Gateway 등 운영 의존성을 함께 구성하고, 내부 DB를 독립 PostgreSQL Pod로 분리했습니다."
category: "운영 구조의 전환"
company: "현대오토에버 · MLOps 플랫폼"
project: "MLOps 배포·E2E 검증·운영 리포팅"
journey: "platform-operations"
relatedCases: ["platform-operations"]
period: "2026년 경험 · 성과 범위 2026.08 기준"
role: "MLOps 운영 · Backend · Kubernetes 이관 및 구성·리소스 조정"
tags: ["Kubernetes", "Helm", "PostgreSQL", "Kafka", "OpenSearch", "MLOps"]
order: 6
draft: false
publishedAt: "2026-11-09"
---
## 모델을 실행하는 곳과, 플랫폼을 운영하는 곳을 함께 바꿨습니다

현대오토에버에서 Local PC에 종속돼 있던 CoreNect 운영 구조를 HMG Cloud Kubernetes 환경으로 이관했습니다. ML 서비스 8종 이상을 옮기고, 플랫폼을 구성하는 데이터 저장·메시징·외부 요청 경로를 함께 다뤘습니다.

이 작업의 중심은 모델의 추론 성능을 높이는 것이 아니었습니다. **ML 서비스를 실행하는 데 필요한 의존성을 포함해 운영 구조를 옮기는 일**이었습니다. 모델 컨테이너만 기동했다고 플랫폼의 전환이 끝났다고 보지 않았습니다.

## 서비스와 함께 움직여야 하는 의존성이 있었습니다

PostgreSQL, Kafka·Kafdrop, OpenSearch·Dashboard, CoreNect Core Studio, External Gateway, Harbor, Loki·Prometheus, ML 서비스 Pod를 단계적으로 구성했습니다. 모델 실행 외에 요청을 받고 처리 흐름을 연결하며 결과와 운영 정보를 다루는 구성요소들이었습니다.

| 이관에서 다룬 범위 | 수행한 내용 |
|---|---|
| ML 실행 경로 | ML 서비스 8종 이상 이관 |
| 데이터·메시징·플랫폼 구성 | PostgreSQL·Kafka·OpenSearch·Core Studio 등 구성 |
| 외부 요청과 운영 기반 | External Gateway·Harbor·Loki·Prometheus 등 구성 |
| 배치와 자원 | 관련 구성요소 노드 재배치 및 리소스 조정 |

이 표는 담당한 전환 범위입니다. 서비스 수와 인프라 구성요소 수를 합산하거나, 모든 구성의 고가용성 검증을 뜻하는 표로 해석하지 않습니다.

## 내부 DB를 독립된 PostgreSQL Pod로 분리했습니다

CoreNect Core 내부 DB 구조를 독립 PostgreSQL Pod 구조로 분리했습니다. 플랫폼 구성요소와 DB의 실행 경계를 구분해 장애 격리성을 개선한 변경이었습니다.

다만 실행 경계를 분리했다는 사실과 데이터의 고가용성·복구가 자동 보장된다는 사실은 다릅니다. 이 글에서는 DB 복제 구성, 자동 Failover, 백업 복구의 목표 시간이나 데이터 무손실 전환을 완료했다고 주장하지 않습니다.

```text
실행 경계의 개념도

Local PC 종속 운영
    → HMG Cloud Kubernetes 환경의 플랫폼 구성
    → ML 서비스와 운영 의존성 이관
    → 내부 DB를 독립 PostgreSQL Pod로 분리
```

실제 네트워크·노드 구성도나 이관 작업의 모든 순서를 재현한 그림은 아닙니다. 바꾼 운영 경계를 요약한 것입니다.

## 전환 범위와 확인 범위를 같은 숫자로 묶지 않았습니다

ML 서비스 이관과 함께 pykafka reader·writer 버전 변경을 검증하고, 관련 구성요소의 노드 재배치와 리소스 조정을 수행했습니다. 이관한 서비스 수는 8종 이상입니다.

다른 글에서 다룬 개발계 CI/CD 9개 서비스, E2E 동기 검증 4개 서비스와는 집계 대상이 다릅니다. 이관·배포 자동화·응답 검증을 하나의 ‘완료 서비스 수’로 합치지 않습니다. 당시 전환 성과를 현재 시점의 전체 운영 상태나 무중단 이관 보장으로 확대하지도 않습니다.

## 플랫폼 운영의 단위를 넓힌 경험이었습니다

이 작업을 통해 운영의 대상을 API 서버 한 개가 아니라, 서비스를 실행하고 요청·결과를 연결하는 의존성까지 넓혀 보게 됐습니다. 운영 구조를 옮길 때는 무엇을 옮겼는지와 어느 경계를 분리했는지, 무엇을 확인했는지를 각각 남겨야 합니다.

현재 다시 전환을 준비한다면 의존성별 준비 상태, 데이터 이동·복구 기준, 전환 후 확인과 되돌릴 조건을 더 명시적으로 정리하겠습니다. 이는 당시 해당 절차를 모두 자동화했다는 설명이 아니라 후속 설계 기준입니다.
