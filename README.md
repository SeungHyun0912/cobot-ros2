# cobot-ros2

# ROS2 Jazzy 설치 가이드 (Windows + WSL2)

> 대상 환경: Windows 10/11 + WSL2 + Ubuntu 24.04 + ROS2 Jazzy Jalisco

---

## 1. WSL2 + Ubuntu 24.04 설치

PowerShell을 **관리자 권한**으로 실행합니다.

```powershell
wsl --install -d Ubuntu-24.04
```

이미 WSL이 설치되어 있다면 배포판만 추가하고 버전을 지정합니다.

```powershell
wsl --install -d Ubuntu-24.04
wsl --set-default-version 2
```

설치 후 재부팅이 필요할 수 있습니다. 재부팅 후 Ubuntu 앱을 실행하면 최초 1회 사용자명/비밀번호를 설정합니다.

### 설치 확인

```powershell
wsl -l -v
```

`Ubuntu-24.04`가 **VERSION 2**로 표시되는지 반드시 확인합니다. (버전 1이면 ROS2 네트워킹이 정상 동작하지 않습니다.)

---

## 2. WSL 내부 기본 설정 (로케일)

Ubuntu 터미널에 진입한 뒤 실행합니다.

```bash
sudo apt update && sudo apt upgrade -y

locale  # 현재 로케일 확인

sudo apt install locales -y
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8
```

---

## 3. ROS2 Jazzy 저장소(apt) 등록

```bash
sudo apt install software-properties-common curl -y
sudo add-apt-repository universe

sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
  -o /usr/share/keyrings/ros-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | \
  sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
```

---

## 4. ROS2 Jazzy 설치

```bash
sudo apt update
sudo apt install ros-jazzy-desktop -y
```

| 패키지 | 설명 |
|---|---|
| `ros-jazzy-desktop` | RViz, 데모 패키지 등 포함된 풀 버전 |
| `ros-jazzy-ros-base` | 경량 버전 (GUI 도구 미포함) |

빌드 도구도 함께 설치합니다.

```bash
sudo apt install ros-dev-tools -y
```

---

## 5. 환경 변수 자동 설정

매번 `source`를 수동으로 입력하지 않도록 `.bashrc`에 등록합니다.

```bash
echo "source /opt/ros/jazzy/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

---

## 6. 설치 확인 (talker / listener)

**터미널 A**

```bash
ros2 run demo_nodes_cpp talker
```

**터미널 B** (새 WSL 터미널 창)

```bash
ros2 run demo_nodes_py listener
```

listener 쪽에 talker가 보낸 메시지가 출력되면 설치가 정상적으로 완료된 것입니다.

---

## ⚠️ WSL2 특유의 주의사항

### 1) GUI 앱(RViz, Gazebo)이 뜨지 않는 경우
- Windows 11 + 최신 WSL: **WSLg**가 기본 내장되어 있어 별도 X서버 없이 GUI가 실행됩니다.
- Windows 10 또는 WSLg 미지원 환경: VcXsrv 같은 별도 X서버 설치가 필요합니다.
- `wsl --update` 명령으로 WSL 자체를 최신화하면 대부분 해결됩니다.

### 2) 네트워크 discovery 문제
- WSL2는 자체 가상 네트워크(NAT)를 사용합니다.
- 같은 WSL 인스턴스 내 노드끼리는 문제없지만, **Windows 호스트나 외부 물리 머신(실제 로봇, MQTT 브로커 등)과 통신**할 때는 NAT로 인해 DDS discovery가 막힐 수 있습니다.
- 이는 실제 로봇/MQTT 브로커 연동 단계에서 반드시 다시 다뤄야 하는 항목입니다.
- Windows 11 22H2 이상에서는 `.wslconfig`에 아래 설정을 추가하면 상당 부분 해결됩니다.

```ini
# %UserProfile%\.wslconfig
[wsl2]
networkingMode=mirrored
```

### 3) 성능 이슈
- 워크스페이스는 반드시 **WSL 내부 경로**(예: `~/ros2_ws`)에 두어야 합니다.
- `/mnt/c/...` 같은 Windows 마운트 경로에 두면 `colcon` 빌드 속도가 크게 저하됩니다.

### 4) 에디터 연동
- Windows에 VSCode 설치 후 **Remote - WSL** 확장을 사용하면, Windows GUI로 WSL 내부 코드를 그대로 편집·디버깅할 수 있습니다.

---

## 다음 단계

- 워크스페이스(`ros2_ws`) 생성
- 패키지 구조 이해
- `colcon` 빌드 시스템
- Dummy Publisher/Subscriber 패키지 실습
