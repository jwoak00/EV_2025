2025 국제 대학생 EV 자율주행 경진대회 ― JetRacer ROS AI Kit
ROS Melodic (Ubuntu 18.04 / Python 2.7) 자율주행 노드 abc_track_node.py


# ABC Track Node (ROS Melodic / Jetson Nano)

자율주행 **JetRacer / JetBot**를 위한 “ABC 구역” 파이프라인 노드입니다.

| Zone | 센서 · 전략 | 목적 |
|------|------------|------|
| **A** | CSI-카메라 + Bird-Eye 변환 → 슬라이딩 윈도우 차선 검출 → Pure Pursuit | 차선 주행 |
| **B** | 2-D LiDAR | 복도(corridor) 중심 유지 |
| **C** | `/odom` | 상대 목표점(로봇 local → odom)으로 이동 |

<p align="center">
  <img src="https://img.shields.io/badge/Ubuntu-18.04-E95420?logo=ubuntu" />
  <img src="https://img.shields.io/badge/ROS-Melodic-22314E?logo=ros" />
  <img src="https://img.shields.io/badge/Python-2.7-blue" />
</p>

---

## 📂 패키지 구조
```text
abc_track/
├── abc_track_node.py        # ★ 본 노드
├── launch/                  # abc_track.launch 등
└── package.xml / CMakeLists.txt
```

---

## 주요 기능
| Zone | 핵심 로직 | 주요 파라미터 |
|------|-----------|---------------|
| **A-CAM** | • CameraInfo → undistort<br>• Perspective(BEV) transform `fixed_points`<br>• HLS 차선 마스킹 → 원형 필터(표지 제거)<br>• Sliding window(18 윈도우) → 차선 3차·1차 근사<br>• Pure Pursuit (`x_la`, `real_shift`)로 조향 산출 | `lower_hls` `upper_hls` `nwindows` `margin` |
| **B-LIDAR** | • ±100° 영역 평균 거리 비교<br>• P 제어(`kp`)로 회전하여 중심 유지 | `wall_threshold` `kp` |
| **C-ODOM** | • 모드 진입 시 현재 pose → 목표점 (world)<br>• 도착까지 속도·각속도 제어 | `goal_x_rel` `goal_y_rel` `arrive_dist_thresh` |
<img width="1920" height="978" alt="image" src="https://github.com/user-attachments/assets/b6f11b8f-9b67-4f78-9d77-ecac97b5f8f5" />

---

##  의존성
```bash
sudo apt install ros-melodic-{cv-bridge,compressed-image-transport,laser-proc} \
                 ros-melodic-roslint  # 선택
sudo apt install python-numpy python-opencv
```
> JetPack 4.x / CUDA 10.2 환경에서 테스트됨.

---

##  빌드 · 설치
```bash
cd ~/catkin_ws/src
git clone <repo-url>
cd ..
catkin_make -DCATKIN_WHITELIST_PACKAGES="abc_track"
source devel/setup.bash
```

---

##  실행 예시
```bash
roslaunch abc_track abc_track.launch
```
*예시 launch 파일에서*  
- 카메라 드라이버(`/csi_cam_0`)  
- LiDAR 드라이버(`/scan`)  
- `robot_state_publisher` / `ekf_localization` 등이 함께 구동된다고 가정합니다.

---

##  토픽 I/O

| 방향 | 토픽 | 타입 | 설명 |
|------|------|------|------|
| Sub | `/csi_cam_0/image_raw/compressed` | `sensor_msgs/CompressedImage` | A-zone 원본 이미지 |
| Sub | `/csi_cam_0/camera_info` | `sensor_msgs/CameraInfo` | 내부 파라미터 |
| Sub | `/scan` | `sensor_msgs/LaserScan` | B-zone LiDAR |
| Sub | `/odom` | `nav_msgs/Odometry` | C-zone 위치 |
| Pub | `/cmd_vel` | `geometry_msgs/Twist` | 전체 주행 속도·조향 |
| Pub | `/car/bev_image` | `sensor_msgs/Image` | 디버그: BEV |
| Pub | `/car/sliding_window_image` | `sensor_msgs/Image` | 디버그: 윈도우 |
| Pub | `/goal_marker` | `visualization_msgs/Marker` | RViz 목표 표시 |

---

##  주요 파라미터 (rosparam)

| 이름 | 기본값 | 영역 | 설명 |
|------|--------|------|------|
| `fixed_points` | `[(9,479),(638,475),(517,251),(118,252)]` | A | BEV 원근변환 4점 |
| `lower_hls` / `upper_hls` | `[100,160,20]` / `[172,200,220]` | A | 차선 HLS 범위 |
| `x_la` | `0.57` | A | Pure Pursuit look-ahead [m] |
| `wall_threshold` | `0.6` m | B | 좌우 벽 거리 임계 |
| `kp` | `0.7` | B | P 제어 gain |
| `goal_x_rel`, `goal_y_rel` | `0.8`, `0.8` m | C | 초기 pose 기준 목표점 |
| `arrive_dist_thresh` | `0.25` m | C | 도착 판정 거리 |

---

##  RViz 빠른 시각화
```bash
rviz -d abc_track/rviz/abc_debug.rviz
```
*BEV / SlidingWindow / Goal Marker 레이어 포함.*

---



