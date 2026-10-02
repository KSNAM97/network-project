📢 [프로젝트] 최종 설계 확정 — HQ/DC/BR1/BR2 + ISP 이중화 + DMVPN

■ 구조
사이트(HQ 65001 / DC 65002 / BR1 65003 / BR2 65004) 내부는 OSPF 전용,
ISP-1 · ISP-2는 라우터별 개별 AS의 eBGP 전용 (PE-A/P1/P2/PE-B, 마름모 4대).
사이트↔ISP는 정적경로+재분배, DC↔BR1/BR2는 DMVPN(mGRE over IPsec)으로
Private Internet(203.0.113.0/24) 경유. Private Internet에는 화이트리스트 ACL 적용.
DC 내부 스위치 4대만 Arista cEOS, 나머지는 전부 Cisco.

■ 역할 분담
- Network(본인): DC 전체, Private Internet, DNS 서버, QoS
- Cloud: HQ, ISP-1, ISP-2 (eBGP 8세션, 정적경로+track)
- Policy: BR1, BR2, DMVPN Spoke+IPsec, ACL/컨벤션/문서

■ 컨벤션
- VLAN: 10=HQ-USER-A, 20=HQ-USER-B, 50=DC-SERVER(DNS), 100=MGMT, 999=NATIVE
- 인터페이스/IP 매핑은 network-project-final.md 참고, 실제 케이블링과 반드시 대조
- 커밋 메시지: [파트] 내용

■ 리소스 주의
7200 대당 256MB, IOU 대당 500MB 기준 확인됨. cEOS(DC 6대)는 아직 미실측이라
각자 작업 전 담당 사이트만 켜는 것을 기본으로 합니다. 실측치는 W1 안에 공유 예정.

■ 목표
6주, "완주"보다 "각자 결과물 하나씩" 우선. 취업활동으로 잠시 손 놓아도 괜찮습니다.

첨부: network-project-final.md, jira-tickets.csv
#team-network #team-cloud #team-policy #dev-issues
