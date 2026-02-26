# TROUBLESHOOTING

목적
- 설치/운영 중 발생한 이슈를 “증상 → 원인 → 조치 → 증거” 순서로 남긴다.
- 재현성이 높은 기록을 남겨 다음 환경에서 10분 안에 같은 결론을 재현한다.

---

## 0) 증상 재현 상태 요약

- 완료: `01_brew_multipass_install`, `04_vm_launched_info`, `06_ros2_help_ok`, `07_talker_listener_ok`
- 미발생/미기록: `02_launchctl_bootstrap_ok`, `03_hash_mismatch_error`, `05_kernel_updated_uname`

## 1) `Homebrew 5.0.15`, macOS 13.7 경고

### 증상
- `brew --version` 실행 시 지원 경고(예: Tier 3 경고)가 보일 수 있음.

### 원인
- macOS 메이저 버전이 기본 지원 범위에서 벗어나 있을 수 있음.

### 조치
- 경고는 치명적 오류가 아니므로 동작 확인 후 기록만 남김.

### 근거 파일
- `evidence/01_brew_multipass_install.txt`

---

## 2) `multipass` 명령은 Host에서만 가능

### 증상
- VM 내부에서 `multipass shell` 안에 있는 상태에서 `multipass` 계열을 실행하면 실패할 수 있음.

### 원인
- `multipass` CLI는 macOS Host 전용이다.

### 조치
- VM 관련 작업은 `multipass` 호스트 명령으로 수행하고, VM은 `apt`, `ros2`만 실행.

### 근거 파일
- `README.md`의 Host/VM 구분표

---

## 3) macOS `script`에서 `illegal option -- c`

### 증상
- 다음 오류 발생:
  - `script: illegal option -- c`

### 원인
- macOS의 `script`는 GNU `script -c`를 지원하지 않음.

### 조치
- macOS 표준 방식으로 실행:
```bash
script -a evidence/<file>.txt /bin/zsh -lc "<명령>"
```

### 근거 파일
- 사용자 실행 로그 (위 조치로 재기록)

---

## 4) 증거 파일이 “없는 것처럼” 보이는 문제

### 증상
- `sed: can't read evidence/04_vm_launched_info.txt: No such file or directory`

### 원인
- 현재 쉘이 VM 내부이거나 경로가 잘못된 디렉터리에서 실행됨.
- `script`를 `evidence` 하위에서 실행하면서 `evidence/evidence/...`를 참조할 경우.

### 조치
- 항상 repo 루트 기준으로 실행:
```bash
cd /Users/deepblue/repos/ros2-humble-multipass-macos13
pwd
ls evidence
```
- 파일 경로를 절대경로로 명시해 확인:
```bash
sed -n '1,80p' /Users/deepblue/repos/ros2-humble-multipass-macos13/evidence/04_vm_launched_info.txt
```

### 근거 파일
- `evidence/04_vm_launched_info.txt`

---

## 5) 이미지 해시 불일치 (예비 항목)

### 증상
- `Hash of ... ubuntu-22.04-server-cloudimg-amd64.img does not match ...`

### 원인
- Multipass 이미지 캐시 손상/부분 다운로드

### 조치
- 캐시 정리 후 재시도:
```bash
sudo rm -rf /var/root/Library/Caches/multipassd/qemu/vault/images/jammy-20260218
multipass launch 22.04 -n ubuntu2204 -c 4 -m 8G -d 60G
```

### 근거 파일
- `evidence/03_hash_mismatch_error.txt`
- `evidence/04_vm_launched_info.txt`

### 현재 상태
- 본 세션에서는 해당 이슈가 재현되지 않아 미기록 상태다.

---

## 6) ROS 2가 아직 안 보이는 경우

### 증상
- VM에서 `ros2 --help`가 안 되거나 노드 실행 불가

### 원인
- ROS 2가 미설치이거나 세션에서 `source` 미적용

### 조치
```bash
multipass exec ubuntu2204 -- bash -lc 'source /opt/ros/humble/setup.bash; ros2 --help'
```
- `ros2`가 없어도 설치해야 함:
```bash
multipass exec ubuntu2204 -- bash -lc 'sudo apt update && sudo apt install -y ros-humble-desktop'
```

### 근거 파일
- `evidence/06_ros2_help_ok.txt`

---

## 7) Talker/Listener 메시지 미출력

### 증상
- listener 또는 talker 실행은 되는데 메시지 수신이 안 보임

### 원인
- topic/노드 실행 순서 문제 또는 환경 `source` 누락

### 조치
- listener 먼저 실행 후 talker 실행
- 각 실행 셸에서 `source /opt/ros/humble/setup.bash` 필수

### 근거 파일
- `evidence/07_talker_listener_ok.txt`
