# human_follower
human_followerは2DｰLiDARと深層学習を用いた人追従機能を提供するROSパッケージです。
<br>本パッケージに含まれる機能は以下の通りです。

- 追従対象者との距離を0.5[m]~1.5[m]に維持した人追従
- ロボットの前方に侵入してきた障害物に対する緊急停止
- 人の重心座標や目標座標等の可視化

## Description
本機能は以下の前提条件のもとに実現されます。

- 屋内環境で利用される
- 差動二輪駆動台車を搭載したロボットである
- 2DｰLiDARがロボット前方に搭載されている
- GPUを搭載した制御PCを搭載している

本機能は、人の両脚検出器から算出された重心座標と目標座標の差を収束させるような制御を行うことで実現しています。

### 人の両脚検出器の作成
人の両脚検出器は両脚部分の画像データセットを収集し、リアルタイム物体検出アルゴリズムである[YOLO](https://arxiv.org/pdf/1506.02640.pdf)を用いて作成します。前処理として、2DｰLiDARから取得した[LaserScan型](http://docs.ros.org/en/noetic/api/sensor_msgs/html/msg/LaserScan.html)の測距データを1[pixel]=0.01[m]とした大きさ500×500[pixel]の画像に変換します。これは[laser_to_imageノード](scripts/laser_to_image.py)が実現しており、変換された画像はROSの[Image型](http://docs.ros.org/en/noetic/api/sensor_msgs/html/msg/Image.html)トピックとして配信され、[rosbag](http://wiki.ros.org/ja/rosbag)で記録することでデータセットの収集を行います。
<br>本手法では人の両脚部分を1つのクラスとしてラベリングを行い、左右並行移動や回転、ノイズ付与といったデータ拡張を行うことで合計1万枚程のデータセットを作成しました。学習モデルにはYOLOv5sを用いています。私が作成した重みファイルが[human_follower/weights](./weights)にあるのでデモとして使ってみてください。

### 追従の制御
追従制御は[follower_controlノード](scripts/follower_control.py)が実現しており、人の重心座標と目標座標の差を収束させるように、並進速度と角速度それぞれに対してPID制御を行います。重心座標は両脚検出器から出力される[BoundingBox](https://github.com/mats-robotics/detection_msgs/tree/main/msg)の交点、目標座標はロボットの正面0.5[m]としています。[follower_controlノード](scripts/follower_control.py)には、ロボット前方180[°]のsafety_dist[m]以下に障害物が侵入した際、緊急停止する処理も含まれています。
<br>また、前述したPIDの各ゲインや目標座標、safety_dist等の各値は[follower.launch](./launch/follower.launch)からROSパラメータとして設定できます。

<p align="center">
  <img src="https://user-images.githubusercontent.com/45844173/220180336-7dfbf791-a63d-453f-b686-8caf2908135a.png" width="50%">
</p>
<p align="center">
  重心座標や目標座標を可視化した画像
</p>

## Demo
<details>
<summary>直線経路・曲線経路・雑多な経路における人追従と緊急停止のデモ動画です</summary>

https://user-images.githubusercontent.com/45844173/220177958-479912c3-afe2-4eed-853c-c494167492e4.mp4
</details>


## Requirement

| 環境 | バージョン |
| --- | --- |
| Ubuntu | 20.04 |
| ROS | noetic |

本パッケージでは、[mats-robotics](https://github.com/mats-robotics)氏の[yolov5_ros](https://github.com/mats-robotics/yolov5_ros)というYOLOv5のROSラッパーを利用しています。インストールは[mats-robottics/yolov5_ros](https://github.com/mats-robotics/yolov5_ros/blob/main/launch/yolov5.launch)を参考にしてください。


## Usage

### 人追従
本パッケージを実行する前に、2DｰLiDARを起動してください。
<br>※2DｰLiDARから配信される測距データはLaserScan型である必要があります。

2DｰLiDARを起動したらfollower.launchを起動してください。

~~~
roslaunch human_follower follower.launch
~~~

次にyolov5.launchを起動します。

~~~
roslaunch yolov5_ros yolov5.launch
~~~

## Author
飯田一成（金沢工業大学 工学部 ロボティクス学科 出村研究室）
