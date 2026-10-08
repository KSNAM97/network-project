# 멀티벤더 하이브리드 네트워크 프로젝트 — 최종 설계

## 1. 전체 구조

```
HQ(65001) ─┬─ ISP-1 ─┐
           └─ ISP-2 ─┤
DC(65002) ─┬─ ISP-1 ─┼─ (ISP 내부는 eBGP, 사이트는 OSPF)
           └─ ISP-2 ─┤
BR1        ─┬─ ISP-1 ─┤
           └─ ISP-2 ─┤
BR2        ─┬─ ISP-1 ─┘
           └─ ISP-2 ─┘

DC ←── DMVPN Phase3(mGRE over IPsec, Private Internet 경유) ──→ BR1, BR2
```

- 사이트(HQ/DC/BR1/BR2) 내부: OSPF 전용, BGP 없음
- ISP-1/ISP-2: 라우터별 개별 AS의 eBGP 전용, OSPF 없음
- 사이트 ↔ ISP: 라우팅 프로토콜 없음, 정적 경로 + OSPF 재분배(track)
- Private Internet(DMVPN 언더레이): 별도 OSPF 프로세스(2), ACL 화이트리스트
- DC만 Arista cEOS, 나머지 전부 Cisco (7200 / IOU)

## 2. 사이트 · AS

| 사이트 | AS | 대역 | 비고 |
|---|---|---|---|
| HQ | 65001 | 10.1.0.0/16 | OSPF 프로세스1 Area0 |
| DC | 65002 | 10.2.0.0/16 | OSPF 프로세스1 Area10, DMVPN Hub(DC-R2) |
| BR1 | 65003 | 10.3.0.0/16 | OSPF 프로세스1 Area20, DMVPN Spoke(BR1-R1) |
| BR2 | 65004 | 10.4.0.0/16 | OSPF 프로세스1 Area30, DMVPN Spoke(BR2-R2) |
| ISP-1 | PE-A 65111 / P1 65112 / P2 65113 / PE-B 65114 | 100.1.0.0/16 | eBGP 전용 |
| ISP-2 | PE-A 65121 / P1 65122 / P2 65123 / PE-B 65124 | 100.2.0.0/16 | eBGP 전용 |
| Private Internet | 없음(BGP 미사용) | 203.0.113.0/24 | OSPF 프로세스2 Area100, DMVPN 언더레이 전용 |
| DMVPN 터널 | - | 172.16.0.0/24 | OSPF 프로세스1 Area0 |

폐기된 AS: 65000, 65006, 65007, 65010, 65020 (설계 변경 과정에서 대체됨)

## 3. VLAN

| 사이트 | VLAN | 이름 | 서브넷 | 게이트웨이 이중화 |
|---|---|---|---|---|
| HQ | 10 | HQ-USER-A | 10.1.10.0/24 | HSRP (HQ-SW-2 .2 / HQ-SW-3 .3, VIP .1) |
| HQ | 20 | HQ-USER-B | 10.1.20.0/24 | HSRP |
| DC | 50 | SERVER (DNS1, DB1) | 10.2.50.0/24 | VARP/VRRP (DC-SW3-L3 .2 / DC-SW4-L3 .3, VIP .1) |
| BR1 | 10 | BR1-USER | 10.3.10.0/24 | HSRP |
| BR2 | 10 | BR2-USER | 10.4.10.0/24 | HSRP |
| 전 사이트 | 100 | MGMT | 10.X.100.0/24 | - |
| 전 사이트 | 999 | NATIVE | IP 없음 | 모든 트렁크 native |

## 3-1. GNS3 장비명 대응표

이 문서와 컨피그 파일(`configs/<사이트>/<장비명>.cfg`), 장비 hostname은 GNS3 장비명을 기준으로 합니다. 아래 표는 초기 설계에서 쓰던 이름(설계 장비명)과의 대응이며, 예전 자료를 볼 때 참고합니다. `확인 필요` 표시는 도면 배치로 맞춘 값이라 케이블링과 대조해 확정합니다.

| 사이트 | GNS3 장비명 (기준) | 초기 설계명 | 역할 | 비고 |
|---|---|---|---|---|
| Private Internet | PrivI-net | PI | DMVPN 언더레이 | |
| HQ | HQ-R1 / HQ-R2 | R11 / R12 | 라우터 | |
| HQ | HQ-SW-1 | SW110 | L3 스위치 | 확인 필요 |
| HQ | HQ-SW-2 / HQ-SW-3 | SW101 / SW102 | L3 스위치(HSRP) | 확인 필요 |
| DC | DC-R1 / DC-R2 | R23 / R24 | 라우터(PI 연결) | R24가 DMVPN Hub, 확인 필요 |
| DC | DC-R3 / DC-R4 | R21 / R22 | 라우터(ISP 연결) | 확인 필요 |
| DC | DC-SW1-L3 / DC-SW2-L3 | SW211 / SW212 | L3 스위치(Arista cEOS) | 확인 필요 |
| DC | DC-SW3-L3 / DC-SW4-L3 | SW201 / SW202 | L3 스위치(Arista cEOS, VARP) | 확인 필요 |
| DC | DC-SW1-L2 / DC-SW2-L2 | SW-SVR1 / SW-SVR2 | 서버용 L2 스위치(Arista cEOS) | 확인 필요 |
| DC | (GNS3에 없음) | DNS1 / DB1 | 서버(VLAN50) | 노드 생성 필요 |
| ISP-1 | ISP-1-PE-A / ISP-1-P1 / ISP-1-P2 / ISP-1-PE-B | PE-A / P1 / P2 / PE-B | eBGP | |
| ISP-2 | ISP-2-PE-A / ISP-2-P1 / ISP-2-P2 / ISP-2-PE-B | PE-A / P1 / P2 / PE-B | eBGP | |
| BR1 | BR1-R1 / BR1-R2 | R1 / R2 | 라우터(R1이 DMVPN Spoke) | |
| BR1 | BR1-SW1-L3 / BR1-SW2-L3 / BR1-SW3-L3 | SW1 / SW2 / SW3 | 스위치(SW3은 호스트 연결) | |
| BR1 | (GNS3에 없음) | H1 / H2 | 호스트 | 노드 생성 필요 |
| BR2 | BR2-R1 / BR2-R2 | R1 / R2 | 라우터(R2가 DMVPN Spoke) | |
| BR2 | BR2-SW1-L3 / BR2-SW2-L3 | SW1 / SW2 | L3 스위치 | 확인 필요 |
| BR2 | BR2-SW3-L2 | SW3 | 호스트 연결 L2 스위치 | GNS3 장비명 변경 필요(현재 BR2-SW1-L2) |
| BR2 | (GNS3에 없음) | H1 / H2 | 호스트 | 노드 생성 필요 |

## 4. 인터페이스 · 포트 매핑 (제안값 — 실제 케이블링과 대조 필요)

### HQ
| A | B | 서브넷/용도 |
|---|---|---|
| HQ-SW-2 e1/0 | HQ-R1 Gi0/0 | 10.1.0.0/30 |
| HQ-SW-2 e1/1 | HQ-R2 Gi0/0 | 10.1.0.4/30 |
| HQ-SW-3 e1/0 | HQ-R1 Gi1/0 | 10.1.0.8/30 |
| HQ-SW-3 e1/1 | HQ-R2 Gi1/0 | 10.1.0.12/30 |
| HQ-R1 Gi2/0 | HQ-R2 Gi2/0 | 10.1.0.16/30 |
| HQ-R1 Gi3/0 | ISP-1-PE-A Gi2/0 | 100.1.11.0/30 |
| HQ-R2 Gi3/0 | ISP-2-PE-A Gi2/0 | 100.2.12.0/30 |
| HQ-R2 Gi4/0 | PrivI-net Gi2/0 | 203.0.113.8/30 |
| HQ-SW-1 e1/0-1 | HQ-SW-2 e0/0-1 (Po1, VLAN10/20/100) |
| HQ-SW-1 e1/2-3 | HQ-SW-3 e0/0-1 (Po2) |
| HQ-SW-2 e0/2-3 | HQ-SW-3 e0/2-3 (Po3) |
| **QoS 대상** | HQ-R1 Gi0/0, Gi1/0 / HQ-R2 Gi0/0, Gi1/0 |

### DC
| A | B | 서브넷/용도 |
|---|---|---|
| DC-R1 Gi0/0 | DC-SW1-L3 Et1 | 10.2.0.0/30 |
| DC-R1 Gi1/0 | DC-SW2-L3 Et1 | 10.2.0.4/30 |
| DC-R2 Gi0/0 | DC-SW1-L3 Et2 | 10.2.0.8/30 |
| DC-R2 Gi1/0 | DC-SW2-L3 Et2 | 10.2.0.12/30 |
| DC-SW1-L3 Et3 | DC-SW2-L3 Et3 | 10.2.0.16/30 |
| DC-SW1-L3 Et4-5 | DC-SW3-L3 Et1 / DC-SW4-L3 Et1 |
| DC-SW2-L3 Et4-5 | DC-SW3-L3 Et2 / DC-SW4-L3 Et2 |
| DC-SW3-L3 Et3-4 | DC-R3 Gi0/0 / DC-R4 Gi0/0 |
| DC-SW4-L3 Et3-4 | DC-R3 Gi1/0 / DC-R4 Gi1/0 |
| DC-R3 Gi2/0 | DC-R4 Gi2/0 | 10.2.0.52/30 |
| DC-R1 Gi2/0 | PrivI-net Gi0/0 | 203.0.113.0/30 |
| DC-R2 Gi2/0 | PrivI-net Gi1/0 | 203.0.113.4/30 (Tunnel0 source) |
| DC-R3 Gi3/0 | ISP-1-PE-A Gi3/0 | 100.1.21.0/30 |
| DC-R4 Gi3/0 | ISP-2-PE-A Gi3/0 | 100.2.22.0/30 |
| DC-SW3-L3 Et5-6 | DC-SW4-L3 Et5-6 (Po1, VLAN50/100) |
| DC-SW3-L3/DC-SW4-L3 Et7-8 | DC-SW1-L2 / DC-SW2-L2 (트렁크) |
| DC-SW1-L2 Et3 | DNS1 (VLAN50 access) |
| DC-SW2-L2 Et3 | DB1 (VLAN50 access) |
| **QoS 대상** | DC-R3 Gi0/0,Gi1/0 / DC-R4 Gi0/0,Gi1/0 / DC-R1 Gi0/0,Gi1/0 / DC-R2 Gi0/0,Gi1/0 |

### BR1 (BR2는 10.3→10.4, 장비명 BR2-*, ISP 링크 하단 표 참고)
| A | B | 서브넷/용도 |
|---|---|---|
| BR1-SW1-L3 e0/0-1 | BR1-R1 Gi0/0 / BR1-R2 Gi0/0 |
| BR1-SW2-L3 e0/0-1 | BR1-R1 Gi1/0 / BR1-R2 Gi1/0 |
| BR1-R1 Gi2/0 | BR1-R2 Gi2/0 | 10.3.0.16/30 |
| BR1-R1 Gi3/0 | ISP-1-PE-B Gi2/0 | 100.1.31.0/30 |
| BR1-R2 Gi3/0 | ISP-2-PE-B Gi2/0 | 100.2.32.0/30 |
| BR1-R1 Gi4/0 | PrivI-net Gi3/0 | 203.0.113.12/30 (Tunnel0 source, DMVPN Spoke) |
| BR1-SW1-L3 e1/0-1 | BR1-SW2-L3 e1/0-1 (Po1, VLAN10/100) |
| BR1-SW1-L3/BR1-SW2-L3 → BR1-SW3-L3 | 트렁크 |
| H1/H2 | BR1-SW3-L3 (VLAN10 access) |
| **QoS 대상** | BR1-R1 Gi0/0,Gi1/0 / BR1-R2 Gi0/0,Gi1/0 |

### BR2 (BR1과 동일 패턴, 장비명 BR2-*, 스포크는 BR2-R2)
| 링크 | 서브넷 |
|---|---|
| BR2-R1 Gi3/0 ↔ ISP-1-PE-B Gi3/0 | 100.1.41.0/30 |
| BR2-R2 Gi3/0 ↔ ISP-2-PE-B Gi3/0 | 100.2.42.0/30 |
| BR2-R2 Gi4/0 ↔ PrivI-net Gi4/0 | 203.0.113.16/30 (Tunnel0 source, DMVPN Spoke) |

### ISP-1 (ISP-2는 같은 구조이며 장비명 ISP-2-*, 100.1→100.2, AS +10)
| A | B | 서브넷 |
|---|---|---|
| ISP-1-PE-A Gi0/0 | ISP-1-P1 Gi0/0 | 100.1.0.0/30 |
| ISP-1-PE-A Gi1/0 | ISP-1-P2 Gi0/0 | 100.1.0.4/30 |
| ISP-1-PE-B Gi0/0 | ISP-1-P1 Gi1/0 | 100.1.0.8/30 |
| ISP-1-PE-B Gi1/0 | ISP-1-P2 Gi1/0 | 100.1.0.12/30 |
| ISP-1-P1 Gi2/0 | ISP-1-P2 Gi2/0 | 100.1.0.16/30 |

### DMVPN
| 장비 | Tunnel0 | NBMA(source) | 역할 |
|---|---|---|---|
| DC-R2 | 172.16.0.1/24 | 203.0.113.5 | Hub, priority 255, mGRE over IPsec transport |
| BR1-R1 | 172.16.0.11/24 | 203.0.113.13 | Spoke, priority 0, shortcut |
| BR2-R2 | 172.16.0.12/24 | 203.0.113.17 | Spoke, priority 0, shortcut |

## 5. 정책

### Private Internet ACL (화이트리스트, 5개 링크 전부)
- permit OSPF(224.0.0.5/6), GRE/ESP/UDP500(203.0.113.0/24 간), ICMP(임시)
- deny ip any any log

### QoS (사이트↔스위치 라우터 인터페이스, 총 20포트)
- 위치: HQ-R1/HQ-R2, DC-R1~DC-R4, BR1-R1/BR1-R2, BR2-R1/BR2-R2 — 각 Gi0/0, Gi1/0
- 제외: 라우터간 링크, ISP 방향, PI 방향, Tunnel0
- 클래스: CONTROL(DSCP CS6, 10%) / BUSINESS(DNS 53→AF31, 30%) / class-default(fair-queue)
- 부모 정책: shape average (병목 검증용 예시값, 실측 시 조정)

### 접근 통제
- DC VLAN50(DNS1): 각 사이트 → UDP/TCP 53만 허용, 그 외 차단 (DHCP 허용 포트는 DHCP 설계 확정 후 추가)
- DC VLAN50(DB1): DB 종류 확정 후 허용 포트 추가 (그 전까지 사이트 → DB1 차단 유지)

## 6. 빌드 순서 (6주, 2인 체제 확정)

| 주차 | 내용 |
|---|---|
| W1 | AS/IP/VLAN 확정, GNS3 환경 세팅, 이미지별 RAM 실측(cEOS 포함), 노드 배치 |
| W2 | 사이트 내부 L2/L3 (VLAN, 트렁크, EtherChannel, HSRP/VARP, 사이트 내부 OSPF) |
| W3 | 사이트↔ISP 정적경로+재분배, ISP 내부 eBGP 풀메시, PI ACL |
| W4 | DMVPN(mGRE over IPsec), DNS 서비스, QoS 정책, 접근통제 ACL |
| W5 | 이중화 장애 실측(FHRP, EtherChannel, ISP 장애, DMVPN Hub 장애), 트러블슈팅 로그 |
| W6 | PPT 제작, 리허설 |

## 7. 미확정 항목
- HQ의 DMVPN 스포크 편입 여부 (편입 시 Area 번호 재조정 필요)
- DC 서버 VLAN50: DNS1(주 용도 DHCP)과 DB1 두 대로 구성 (DNS 이중화 없음). DB 종류, IP, 허용 포트, DHCP 방식, 접근 정책은 미확정
- 스위치(IOU L2/cEOS) QoS 지원 여부 실측
