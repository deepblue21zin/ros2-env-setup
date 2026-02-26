# ROS 2 Humble + Multipass on macOS (재현형 가이드)

본 리포지토리는 “완성된 설치 결과”가 아니라, `설치/문제해결/증거`를 재현 가능한 형태로 남기는 포트폴리오 기록용이다.

최종 목표
- macOS 13.7에서 Multipass로 Ubuntu 22.04 VM 생성
- ROS 2 Humble 설치 후 `ros2 --help`, `talker/listener` 동작 증빙
- 모든 실패와 해결 과정을 텍스트 로그로 기록

---

## 디렉터리 구조

```text
ros2-humble-multipass-macos13/
├─ README.md
├─ TROUBLESHOOTING.md
├─ evidence/
│  ├─ 01_brew_multipass_install.txt
│  ├─ 02_launchctl_bootstrap_ok.txt
│  ├─ 03_hash_mismatch_error.txt
│  ├─ 04_vm_launched_info.txt
│  ├─ 05_kernel_updated_uname.txt
│  ├─ 06_ros2_help_ok.txt
│  ├─ 07_talker_listener_ok.txt
│  └─ screenshots/
│     ├─ needrestart_services.png
│     └─ reboot_required.png
└─ scripts/
   ├─ install_ros2_humble.sh
   └─ qos_quick_test.sh
```

---

## 사전 조건

- macOS (예: `macOS 13.7`)
- Homebrew 설치됨
- 기본 작업 폴더: `/Users/deepblue/repos/ros2-humble-multipass-macos13`
- VM 공유 경로: `/Users/deepblue/ws/ros2_ws`(Host) ↔ `/home/ubuntu/ros2_ws`(VM)

---

## Host/VM 명령 구분표

| 작업 | 실행 위치 | 명령 |
|---|---|---|
| Multipass 관리 | macOS Host | `brew`, `multipass`, `launchctl` |
| 패키지 설치/ROS 실행 | Ubuntu VM | `apt`, `ros2`, `colcon` |
| 공유 폴더 확인 | 양쪽 모두 | Host: `~/ws/ros2_ws`, VM: `/home/ubuntu/ros2_ws` |

주의: `multipass`는 Host 전용 CLI이다.  
VM 안에서 `multipass`를 실행하면 `command not found`가 맞는 동작이다.

---

## 공통 증거 수집 규칙

- 텍스트 로그는 `script` 사용
- GUI 대화창/경고 화면처럼 문장 해석이 중요한 장면은 스크린샷 사용
- 로그는 핵심 라인만 남기고 긴 덤프는 피한다(10~40줄 중심)

### Mac `script` 권장 사용법 (안정형)

```bash
cd /Users/deepblue/repos/ros2-humble-multipass-macos13
script -a evidence/<파일명>.txt /bin/zsh -lc "<명령>"
```

※ macOS의 `script`는 GNU의 `-c`가 없으므로 `-lc`로 쉘을 직접 실행한다.

---

## 실행 절차

### Step 1) Homebrew/Multipass 기본 확인

```bash
script -a evidence/01_brew_multipass_install.txt /bin/zsh -lc 'brew --version; multipass version; multipass info'
```

### Step 2) VM 상태/메타 정보 기록

```bash
script -a evidence/04_vm_launched_info.txt /bin/zsh -lc 'multipass info; multipass list; multipass exec ubuntu2204 -- bash -lc "hostname; uname -r; ls -la /home/ubuntu/ros2_ws"'
```

### Step 3) ROS 2 도움말 및 진단 실행

```bash
script -a evidence/06_ros2_help_ok.txt /bin/zsh -lc 'multipass exec ubuntu2204 -- bash -lc "source /opt/ros/humble/setup.bash && ros2 --help && ros2 doctor --report"'
```

### Step 4) Talker/Listener 통신 확인

1. Listener 먼저 실행 (창 A)
```bash
script -a evidence/07_talker_listener_ok.txt /bin/zsh -lc 'multipass exec ubuntu2204 -- bash -lc "source /opt/ros/humble/setup.bash && ros2 run demo_nodes_cpp listener"'
```
2. Talker 실행 (창 B)
```bash
multipass exec ubuntu2204 -- bash -lc "source /opt/ros/humble/setup.bash && ros2 run demo_nodes_cpp talker"
```
3. Listener 쪽 로그가 30초 이상 메시지 수신하면 성공으로 판단

---

## 증거 파일 상태(현재 기록 기준)

- `04_vm_launched_info.txt`: 완료 (Host VM 상태 + VM 내부 기본 정보 기록)
- `06_ros2_help_ok.txt`: 완료 (`ros2 --help`, `ros2 doctor --report` 기록)
- `07_talker_listener_ok.txt`: 완료 (talker/listener 통신 로그)
- `01_brew_multipass_install.txt`: 완료 (기초 환경 확인)
- `02_launchctl_bootstrap_ok.txt`: 본 세션에서 직접 재현 로그 미기록
- `03_hash_mismatch_error.txt`: 본 세션에서 직접 재현 로그 미기록
- `05_kernel_updated_uname.txt`: 본 세션에서 직접 재현 로그 미기록

> 02/03/05는 실제 장애가 재현되지 않은 항목으로, 향후 해당 장애를 재현했을 때 증거를 추가하면 된다.

---

## Step 5: 스크린샷 정리(문서에 넣는 방법)

### 1) 캡처 시점
- `needrestart`/`reboot required` 등 GUI 경고가 필요한 화면
- 터미널 제어문이 많은 텍스트 로그로는 설명이 부족한 화면

### 2) 저장 위치
- `evidence/screenshots/YYMMDD_issue_shortname.png`
- 예: `evidence/screenshots/260226_reboot_required.png`

### 3) 증거 연결 방법
- `README.md`에는 “스크린샷 경로 + 증상 + 판단” 한 줄로 참조
- 같은 내용이 텍스트 로그에도 있으면 텍스트 우선 순위를 준다

---

## 성공 기준(KPI)

- 새 환경에서 README 순서를 따라 30분 내 재현 가능
- `evidence/04`, `06`, `07` 파일이 각각 실제 로그를 포함
- `ros2 --help`, `ros2 doctor --report`, `talker/listener` 최소 30초 성공

---

## 커밋 메시지 예시

- `docs: add reproducible multipass-ros2 setup checklist`
- `docs: add host-vm command boundary table and evidence policy`
- `test: verify ros2 cli and demo talker/listener`
- `fix(script): record command traces using macOS-compatible script invocation`

---

## 참고: Git으로 커밋하기

```bash
git add README.md TROUBLESHOOTING.md evidence scripts
git commit -m "docs: finalize reproducible ros2 multipass setup guide"
```
