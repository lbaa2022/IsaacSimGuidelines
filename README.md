# Isaac Sim 설치 및 환경 구성 가이드라인
### Isaac Sim 설치 및 통합을 위한 가이드라인 게시용 repository입니다. 
---
## 주요 링크
| Title | Link |
| ------ | ------ |
| Isaac Sim document 메인 | https://docs.omniverse.nvidia.com/ |
| Nucleus 메뉴얼 | https://docs.omniverse.nvidia.com/prod_nucleus/prod_nucleus/overview.html |
| Isaac Sim 메뉴얼 | https://docs.omniverse.nvidia.com/app_isaacsim/app_isaacsim/overview.html |
| Isaac Sim ROS workspace github | https://github.com/NVIDIA-Omniverse/IsaacSim-ros_workspaces |
| ROS2 Humble page | https://docs.ros.org/en/humble/index.html |
| Docker MoveIt2 설치 | https://moveit.picknik.ai/main/doc/how_to_guides/how_to_setup_docker_containers_in_ubuntu.html |
| MoveIt2 + Isaac Sim 연동 | https://docs.omniverse.nvidia.com/app_isaacsim/app_isaacsim/tutorial_ros2_moveit.html |
| NVIDIA-CTK 설치 | https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html |
| Moveit2_tutorial github | https://github.com/ros-planning/moveit2_tutorials |
| Topic based ROS2 control github | https://github.com/PickNikRobotics/topic_based_ros2_control/tree/main |

## 환경 설정 이슈 (Ubuntu 20.04 + Isaac Sim 2022.2.1 + ROS2 foxy)
- Ubuntu 20.04 + ROS2 foxy 버전으로 테스트 중에 topic이 보이지 않고 메시지 전달이 되지 않는 등의 문제 발생 (동일 문제가 다른 사람에게도 발생함)
- Isaac Sim 및 ROS2의 수차례 재설치 & 재부팅에도 문제 해결이 안됨
- 기 발생한 이슈 및 이미 최신 버전으로 환경을 구성한 사람들과의 통합, 향후 SW 유지보수 측면 등을 고려하여 Ubuntu 22.04 + ROS2 Humble로 환경 구성 결정

## 환경 설정 이슈 (Ubuntu 22.04 + Isaac Sim 2022.2.1 + ROS2 Humble)
- Ubuntu 22.04의 python default version은 3.10
- ROS2 Humble 설치 시, apt install을 이용하여 설치가 진행되며, python 3.10 버전을 기준으로 설치 됨
- Conda 환경에서 python version이 ROS2 Humble 설치 버전과 다를 경우, rclpy import할 때 에러 발생
- Conda 환경을 이용하여 개발할 경우, 가급적 python 버전을 ROS2 Humble 설치 python 버전과 맞추는 것을 추천(3.10)
- python 버전을 맞춘 상태에서 rclpy를 import할 때 아래의 GLIBCXX_3.4.30 에러 발생 (gcc lib이 없어서 발생)
```
ImportError: /home/tw-office/anaconda3/envs/lba/bin/../lib/libstdc++.so.6: version `GLIBCXX_3.4.30' not found (required by /opt/ros/humble/local/lib/python3.10/dist-packages/rclpy/_rclpy_pybind11.cpython-310-x86_64-linux-gnu.so)
The C extension '/opt/ros/humble/local/lib/python3.10/dist-packages/rclpy/_rclpy_pybind11.cpython-310-x86_64-linux-gnu.so' failed to be imported while being present on the system. Please refer to 'https://docs.ros.org/en/{distro}/Guides/Installation-Troubleshooting.html#import-failing-even-with-library-present-on-the-system' for possible solutions
```
- 상기 에러는 아래의 명령어를 통해 해결 가능(conda 환경을 사용한다고 가정)
```
conda install -c conda-forge gcc=12.1.0
```
- ROS2 Humble 설치 및 테스트 중에 어떤 에러(기억이...)가 발생할 수도 있는데, 아래의 명령어를 통해 해결 가능
```
pip install empy lark
```
- Empty 환경에서도 만약 GPU Fan이 심하게 도는 경우, 아래 과정을 통해 UI rendering FPS를 낮춘다
```
메뉴바 Edit > Preferences > (UI 아래쪽에서) Preferences tab > Rendering > Throttle Rendering > UI FPS Limit > 30 정도로 낮춤
```

## ROS2 연동 테스트 완료된 Isaac Sim 최종 작업 환경
- Ubuntu 22.04 + root python ver=3.10
- Anaconda python ver=3.10 
- ROS2 Humble
- Isaac Sim 2022.2.1
- NVIDIA-SMI, Driver Version 535.54.03 / CUDA Version: 12.2
- ROS2 연동 테스트 예제 [Link](https://docs.omniverse.nvidia.com/app_isaacsim/app_isaacsim/tutorial_ros2_manipulation.html)

## MoveIt2 with isaac sim
### 공식 제공되는 도커는 아래의 에러들이 발생
- Rviz 화면 렌더링이 안됨 --> mesa 업데이트 코드를 DockerFile에 추가해서 빌드하면 해결됨
- Interactive Marker가 안뜸
- Planner가 로드 안됨
- 결과적으로 topic 메시지 기반 제어 불가능

### 소스 빌드를 통해 Moveit2 + isaac sim 연동 모듈 설치
- ROS2 humble 버전은 binary 설치
- topic based ros2 control 모듈도 binary 설치
- moveit2_tutorial을 clone하여 vcs import ~~~ 를 하면 moveit2등 관련 소스가 클론 됨
- 이 상태에서 빌드 후 예제 실행

### Troubleshooting
- 상기 방법으로 moveit2_tutorial을 설치하면 cmake 컴파일 에러가 발생하는데, 아래의 방법으로 해결
```
CMake Error at /usr/share/cmake-3.22/Modules/FindPackageHandleStandardArgs.cmake:230 (message):
  Could NOT find Threads (missing: Threads_FOUND)
Call Stack (most recent call first):
```
- 상기 에러가 발생한 경우, 해당 CMakeLists.txt 파일의 상단에 아래의 코드를 추가해주면 해결됨
```
set(CMAKE_THREAD_LIBS_INIT "-lpthread")
set(CMAKE_HAVE_THREADS_LIBRARY 1)
set(CMAKE_USE_WIN32_THREADS_INIT 0)
set(CMAKE_USE_PTHREADS_INIT 1)
set(THREADS_PREFER_PTHREAD_FLAG ON)
```
