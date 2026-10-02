# 기여 가이드

## 브랜치 전략

```
main                  ← 항상 동작 검증된 상태만
feature/network-*
feature/cloud-*
feature/policy-*
```

작업은 각자의 `feature/<역할>-<내용>` 브랜치에서 진행하고, PR을 올려
Policy 담당의 Review 승인 후 `main`에 merge합니다.

## 브랜치 보호 설정 (레포 관리자가 1회 설정)

```
Settings → Branches → Add rule → main
- Require pull request before merging
- Require approvals (1명 이상)
```

## 커밋 메시지 규칙

```
<Jira키>: [역할][VLAN태그] 내용
```

예시:
```
KAN-14: [Network][vlan-dc-server] DNS1/2 ACL 추가
KAN-21: [Cloud][vlan-hq-user-a] HQ HSRP 설정
KAN-33: [Policy][vlan-br1-user] DMVPN Spoke 구성
```

### VLAN 태그

| VLAN | 태그 |
|---|---|
| 10 (HQ-USER-A) | `vlan-hq-user-a` |
| 20 (HQ-USER-B) | `vlan-hq-user-b` |
| 50 (DC-SERVER) | `vlan-dc-server` |
| BR1 10 | `vlan-br1-user` |
| BR2 10 | `vlan-br2-user` |
| 100 (MGMT) | `vlan-mgmt` |
| 999 (NATIVE) | `vlan-native` |

### 역할 라벨

3인 체제: `Network` / `Cloud` / `Policy`
2인 체제: `Net-Cloud` / `Policy`

## Jira 연동

1. Jira 프로젝트 → Apps → **GitHub for Jira** 설치 후 본 레포 연결 (1회 설정)
2. 커밋 메시지 맨 앞에 Jira 이슈 키(`KAN-1` 등)를 정확히 포함해야 자동 연결됩니다.
3. CSV Import 후 실제 발급된 이슈 번호를 Jira Board에서 직접 확인하고
   팀에 공지합니다. Import 순서와 실제 번호가 어긋날 수 있습니다.
4. PR 제목에도 같은 이슈 키를 포함하면 PR-이슈 연결도 됩니다.

## 컨피그 커밋 규칙

- 변경 전 반드시 장비에서 `copy running-config flash:<장비명>-backup-<날짜>.cfg`로 백업 후 진행합니다.
- 위험도 "상" 작업(Trunk/Native VLAN 변경, 라우팅 프로세스 제거, 터널 재구성 등)은
  PR 설명에 영향 범위와 롤백 명령을 함께 적습니다.
- 컨피그 파일은 `configs/<사이트>/<장비명>.cfg` 형식으로 저장합니다.
