# 일정 — 2인 체제 확정, 3.5주 (Net-Cloud 통합 / Policy)

기간: 2026-10-12(월) ~ 2026-11-04(수, 반일). 평일 17.5일이며, 기존 6주(30일) 계획을 압축한 일정입니다.

## 역할 분담

| 역할 | 담당자 | 담당 사이트 |
|---|---|---|
| Net-Cloud (통합) | 남기석 | DC, HQ, ISP-1, ISP-2, Private Internet, DNS·DB, QoS |
| Policy | 남인탁 | BR1, BR2, DMVPN(Hub+Spoke), 접근통제 ACL |

⚠️ 통합 담당자 업무량이 전체의 약 75%(24개 티켓 중 18개)를 차지합니다.
BR1/BR2 중 하나를 Policy로 넘기는 재배분안을 권장하며, 본 일정은 재배분 적용 기준입니다.

## VLAN 태그

| VLAN | 태그 |
|---|---|
| 10 | `vlan-hq-user-a` |
| 20 | `vlan-hq-user-b` |
| 50 | `vlan-dc-server` |
| BR1 10 | `vlan-br1-user` |
| BR2 10 | `vlan-br2-user` |
| 100 | `vlan-mgmt` |
| 999 | `vlan-native` |

## 일별 목표

| 주 | 일 | Net-Cloud | Policy |
|---|---|---|---|
| 1주 (10/12~10/16) | 월 | AS/IP/VLAN 표 리뷰, GNS3/Jira/Slack/Git 세팅 | 〃 |
| | 화 | 7200/cEOS/IOU RAM 실측, 노드 배치, 기본 링크 up/up | BR1+BR2 노드 배치, 기본 링크 up/up |
| | 수 | DC VLAN/트렁크(Arista), DC VARP | BR1 VLAN/트렁크, HSRP |
| | 목 | HQ VLAN/트렁크, HSRP, Po1-3 | BR2 VLAN/트렁크, HSRP |
| | 금 | 사이트 내부 OSPF, DC/HQ 내부통신 검증 | 사이트 내부 OSPF, BR1/BR2 내부통신 검증 |
| 2주 (10/19~10/23) | 월 | ISP-1, ISP-2 내부 eBGP 8세션 | BR1/BR2↔ISP 정적경로+track |
| | 화 | PE 고객 대역 광고, DC/HQ↔ISP 정적경로+재분배 | 〃 |
| | 수 | PI 기본연결+OSPF(프로세스2), PI ACL 화이트리스트 | 전 사이트 상호도달 확인 |
| | 목 | DMVPN Hub(DC-R2) crypto | DMVPN Spoke(BR1, BR2) |
| | 금 | DNS1·DB1 배치+검증 | Phase3 shortcut 검증 |
| 3주 (10/26~10/30) | 월 | QoS 20포트 중 DC/HQ 12대 | QoS 20포트 중 BR 8대 |
| | 화 | 접근통제 ACL, 통합 테스트 | 〃 |
| | 수 | HSRP/VARP, EtherChannel 실측 | 〃 |
| | 목 | ISP 장애→DMVPN 전환, DMVPN Hub 장애 실측 | 〃 |
| | 금 | 트러블슈팅 로그 정리 | 〃 |
| 3.5주 (11/2~11/4) | 월 | PPT(DC/HQ/ISP/이중화) | PPT(BR/DMVPN) |
| | 화 | PPT 마무리, 리허설 | 〃 |
| | 수(반일) | 예비 | 〃 |

## 기존 주차 표기와의 대응

Jira 이슈 제목 앞의 [W1]~[W6]은 기존 6주 계획의 주차 표기입니다. 이슈 제목은 그대로 두고, 아래 표로 실제 일정과 맞춥니다.

| 이슈 표기 | 실제 일정 |
|---|---|
| [W1] | 1주 월~화 |
| [W2] | 1주 수~금 |
| [W3] | 2주 월~수 |
| [W4] | 2주 목~금, 3주 월~화 |
| [W5] | 3주 수~금 |
| [W6] | 3.5주 |

이슈별 마감일은 Jira의 Due date와 `jira/jira-tickets-3person.csv`의 `Due Date` 열에 같은 값으로 입력되어 있습니다. 마감일은 위 일별 목표의 해당 작업 날짜 기준이며, 마지막 이슈(PPT·리허설)는 11-04입니다.

## 리스크
- 기간이 기존 6주의 약 60%로 줄어 예비일이 거의 없습니다. 2주(ISP·DMVPN)와 3주 월~화(QoS·ACL)에 통합 담당자 업무가 몰려 있어 지연 시 가장 먼저 영향받는 구간입니다.
- 지연 발생 시 컷 순서: ① QoS 스위치 검증 생략(라우터만) ② ISP-2 생략, ISP-1+DMVPN만
