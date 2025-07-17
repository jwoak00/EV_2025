2025 국제 대학생 EV 자율주행 경진대회 ― JetRacer ROS AI Kit
ROS Melodic (Ubuntu 18.04 / Python 2.7) 자율주행 노드 abc_track_node.py


**개요**

이 저장소는 2025 국제 대학생 EV 자율주행 경진대회 1/10 스케일 트랙 출전 차량용 ROS1 패키지입니다.
트랙은 세 개의 구간으로 구성되며 abc_track_node.py 하나로 세 미션을 상태-머신 방식으로 수행합니다.
| 구간      | 센서 & 미션                | 도로 조건                    |
| ------- | ---------------------- | ------------------------ |
| **A구역** | 카메라 기반 차선 인식           | 차선 폭 60 cm·흰색 5 cm       |
| **B구역** | 2-D LiDAR 기반 장애물(벽) 회피 | 벽 사이 도로 폭 1 m·벽 높이 30 cm |
| **C구역** | Odometry 기반 목표점 도달     | 이동 장애물·GPS/비콘 사용 불가      |



**핵심 기능**

A_CAM :
- 카메라 → Bird-Eye View 변환 → HLS 컬러 마스크 → 슬라이딩 윈도우 차선 검출
- 좌/우 차선 곡선 피팅·Pure Pursuit 조향

B_LIDAR :
- 전방 LiDAR(270°) 벽 거리 평균 → 벽 간 편차 기반 P 제어

C_ODOM :
- B구역 종료 시 초기 위치 저장 → 상대 목표 (x=0.8 m, y=0.8 m) 계산
- 목표점까지 선형 속도 + 각도 오차 비례 제어
- 모든 구간에서 /cmd_vel로 Ackermann 조향 형식 Twist 메시지 발행
- 디버깅용 토픽 /car/bev_image, /car/sliding_window_image, RViz Goal Marker 제공



**시스템 요구사항**

| 항목          | 버전                                          |
| ----------- | ------------------------------------------- |
| **ROS**     | Melodic Morenia                             |
| **Ubuntu**  | 18.04.5 LTS                                 |
| **JetPack** | 4.x                                         |
| Python      | 2.7.17                                      |
| 하드웨어        | JetRacer ROS AI Kit (CSI 카메라, RPLiDAR A1 등) |

필수 패키지: cv_bridge, rospy, sensor_msgs, nav_msgs, geometry_msgs, visualization_msgs, numpy, opencv-python.


**주요 파라미터(rosparam)**

| 이름                              | 기본값                                       | 설명                  |
| ------------------------------- | ----------------------------------------- | ------------------- |
| `fixed_points`                  | `[(9,479),(638,475),(517,251),(118,252)]` | BEV 변환 원근점 (픽셀)     |
| `road_width_m` / `bev_length_m` | `0.7` / `1.7`                             | 실제 도로 폭·길이          |
| `lower_hls` · `upper_hls`       | `[100,160,20]` · `[172,200,220]`          | 흰색 차선 HLS 마스크 범위    |
| `wall_threshold`                | `0.6` m                                   | B구역: 좌·우 벽 평균 거리 임계 |
| `goal_x_rel` / `goal_y_rel`     | `0.8` m                                   | C구역: 시작점 기준 목표 상대좌표 |
| 기타                              | `x_la`, `real_shift`, `kp`, `goto_*`      | 코드 주석 참고            |




**토픽/프레임**

| 방향  | 토픽                                | 타입                                 |
| --- | --------------------------------- | ---------------------------------- |
| Sub | `/csi_cam_0/image_raw/compressed` | `sensor_msgs/CompressedImage`      |
| Sub | `/scan`                           | `sensor_msgs/LaserScan`            |
| Sub | `/odom`                           | `nav_msgs/Odometry`                |
| Pub | `/cmd_vel`                        | `geometry_msgs/Twist`              |
| Pub | `/car/bev_image`                  | `sensor_msgs/Image` (디버깅)          |
| Pub | `/car/sliding_window_image`       | `sensor_msgs/Image` (디버깅)          |
| Pub | `goal_marker`                     | `visualization_msgs/Marker` (RViz) |
| TF  | `odom` 기준                         | JetRacer 기본 TF 트리 사용               |

