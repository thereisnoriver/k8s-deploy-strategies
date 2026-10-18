# Original Study Plan (Not Completed)

This document preserves the original Korean 14-day experiment outline. The entries below are **proposed activities**, not a record of completed work.

##1주차: 롤링 업데이트 기초 + 장애 실험
| 요일    | 목표                         | 세부 내용                                                                                       |
| ----- | -------------------------- | ------------------------------------------------------------------------------------------- |
| Day 1 | 실험 환경 구성                   | - 포트 변경 (8080 → 8090)<br>- v1 앱 배포<br>- 기본 `/version` 응답 확인                                 |
| Day 2 | 롤링 업데이트 흐름 익히기             | - `kubectl set image`로 v2 배포<br>- 응답 `"v1"` → `"v2"` 변하는지 확인                                |
| Day 3 | 파드 수 늘려 순차적 롤링 확인          | - `replicas: 4~5`<br>- `maxUnavailable`, `maxSurge` 조정<br>- `watch get pods`, `curl`로 동작 확인 |
| Day 4 | readinessProbe 없이 실패 유도 실험 | - v2에 `Random()` 기반 에러 삽입<br>- 일부 파드 실패 발생<br>- `curl`, `get pods`로 확인                      |
| Day 5 | `readinessProbe` 추가 후 재실험  | - 응답 안정화 확인<br>- readiness 동작 검증                                                            |
| Day 6 | 전체 파드 죽는 경우 실험             | - `System.exit(1)` 삽입<br>- `CrashLoopBackOff` 확인<br>- `rollout undo` 실습                     |
| Day 7 | 일부만 죽는 경우 실험               | - 일부 파드만 죽는 조건 조작<br>- 라벨/리소스 편차 등 실험<br>- 대응 전략 검토                                         |

##2주차: 고급 시나리오 + 문제 해결 실험
| 요일     | 목표               | 세부 내용                                                               |
| ------ | ---------------- | ------------------------------------------------------------------- |
| Day 8  | 실무 장애 사례 재현 ①    | - ConfigMap 변경 중 롤링 실패<br>- live 변경 문제 실험                           |
| Day 9  | 실무 장애 사례 재현 ②    | - readinessProbe은 OK인데 실제 에러 응답<br>- 로그 파싱, `startupProbe` 필요 여부 검토 |
| Day 10 | 리소스 편차 시나리오 실험   | - CPU 제한 조절<br>- 응답 지연 유도 + timeout 발생                              |
| Day 11 | 롤링 업데이트 튜닝 실험    | - `maxSurge`, `maxUnavailable` 값 실험<br>- 가용성/속도 트레이드오프              |
| Day 12 | 실험 결과 수집 + 로그 정리 | - `docs/01_rolling.md` 정리<br>- 실험별 명령어, 로그, 결과 정리                   |
| Day 13 | 블로그 초안 작성        | - 실험 요약, 기술적 교훈, 문제 해결 과정 정리                                        |
| Day 14 | 마무리 + 피드백 반영     | - 기록 마감<br>- 포트폴리오, 이력서 문장화 준비                                      |
