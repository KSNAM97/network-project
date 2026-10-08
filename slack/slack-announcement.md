📢 [프로젝트] 최종 설계 확정 — HQ/DC/BR1/BR2 + ISP 이중화 + DMVPN

■ 구조
사이트(HQ 65001 / DC 65002 / BR1 65003 / BR2 65004) 내부는 OSPF 전용,
ISP-1 · ISP-2는 라우터별 개별 AS의 eBGP 전용 (PE-A/P1/P2/PE-B, 마름모 4대).
사이트↔ISP는 정적경로+재분배, DC↔BR1/BR2는 DMVPN Phase3(mGRE over IPsec)으로
Private Internet(203.0.113.0/24) 경유. Private Internet에는 화이트리스트 ACL 적용.
DC 내부 스위치 6대(L3 4대 + 서버용 L2 2대)만 Arista cEOS, 나머지는 전부 Cisco.
DC 서버 VLAN(50)은 DNS1(주 용도 DHCP)과 DB1 두 대로 구성합니다 (DB 종류·DHCP 방식·접근 포트는 미확정).

■ 역할 분담
- Net-Cloud(남기석): DC 전체, HQ, ISP-1, ISP-2 (eBGP 8세션, 정적경로+track), Private Internet, DNS/DB 서버, QoS
- Policy(남인탁): BR1, BR2, DMVPN Spoke+IPsec, ACL/컨벤션/문서

■ 컨벤션
- VLAN: 10=HQ-USER-A, 20=HQ-USER-B, 50=DC-SERVER(DNS1, DB1), 100=MGMT, 999=NATIVE
  (BR1·BR2는 각 VLAN 10)
- 인터페이스/IP 매핑은 docs/network-project-final.md 참고, 실제 케이블링과 반드시 대조
- 브랜치: feature/<역할>-<내용>, PR 후 Policy 담당 승인을 받아 main에 merge
- 커밋 메시지: <Jira키>: [역할][VLAN태그] 내용
  예) KAN-14: [Network][vlan-dc-server] DNS1 ACL 추가
  Jira 프로젝트 키는 KAN이고, 커밋 맨 앞에 키를 적으면 이슈에 자동 연결됩니다.
- 컨피그 파일: configs/<사이트>/<장비명>.cfg 로 저장
  변경 전에 장비에서 copy running-config flash:<장비명>-backup-<날짜>.cfg 로 백업

■ 리소스 주의
7200 대당 256MB, IOU 대당 500MB 기준 확인됨. cEOS(DC 6대)는 아직 미실측이라
각자 작업 전 담당 사이트만 켜는 것을 기본으로 합니다. 실측치는 W1 안에 공유 예정.

■ 목표
6주, "완주"보다 "각자 결과물 하나씩" 우선. 취업활동으로 잠시 손 놓아도 괜찮습니다.

■ 문서·링크
- GitHub: KSNAM97/network-project
  README.md, docs/network-project-final.md(전체 설계), docs/schedule-2person.md(일정)
- Jira: KAN 프로젝트 (티켓 목록은 jira/jira-tickets-3person.csv 기준으로 등록)
#team-net-cloud #team-policy #dev-issues
