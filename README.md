# human_follower
human_followerは2DｰLiDARと深層学習を用いた人追従機能を提供するROSパッケージです。
<br>本パッケージに含まれる機能は以下の通りです。

- 追従対象者との距離を0.5[m]~1.5[m]に維持した人追従
- ロボットの前方に侵入してきた障害物に対する緊急停止
- 人の重心座標や目標座標等の可視化

### 動作確認環境

| 環境 | バージョン |
| --- | --- |
| Ubuntu | 20.04 |
| ROS | noetic |

## Description
本機能は以下の前提条件のもとに実現されます。

- 屋内環境で利用される
- 差動二輪駆動台車を搭載したロボットである
- 2DｰLiDARがロボット前方に搭載されている
- GPUを搭載した制御PCを搭載している

本機能は、人の両脚検出器から算出された重心座標と目標座標の差を収束させるような制御を行うことで実現しています。
<br>以下、手法の詳細です。

### 人の両脚検出器の作成
人の両脚検出器は両脚部分の画像データセットを収集し、リアルタイム物体検出アルゴリズムである[YOLO](https://arxiv.org/pdf/1506.02640.pdf)を用いて作成します。前処理として、2DｰLiDARから取得した測距データを1[pixel]=0.01[m]とした大きさ500×500[pixel]の画像に変換します(Fig.1)。これは[laser_to_imageノード](scripts/laser_to_image.py)が実現しており、変換された画像はROSのImage型トピックとして配信され、rosbagで記録することでデータセットの収集を行います。
<br>本手法では人の両脚部分を1つのクラスとしてラベリングを行い、左右並行移動や回転、ノイズ付与といったデータ拡張を行うことで合計1万枚程のデータセットを作成しました。学習モデルにはYOLOv5sを用いています。

<p align="center">
  <img src="https://user-images.githubusercontent.com/45844173/220174203-4e2d7767-408c-456f-915c-e0088f67628d.png" width="50%">
</p>
<p align="center">
  Fig.1 測距データから変換された画像データ
</p>

### 追従の制御
追従制御は[follower_controlノード](scripts/follower_control.py)が実現しており、人の重心座標と目標座標の差を収束させるように、並進速度と角速度それぞれに対してPID制御を行います。重心座標は両脚検出器から出力されるBoundingBoxの交点、目標座標はロボットの正面0.5[m]としています。[follower_controlノード](scripts/follower_control.py)には、ロボット前方180[°]のsafety_dist[m]以下に障害物が侵入した際、緊急停止する処理も含まれています。
<br>また、前述したPIDの各ゲインや目標座標、safety_dist等の各値はfollower.launchからROSパラメータとして設定できます。

## Demo
直線経路・曲線経路・雑多な経路における人追従と緊急停止のデモ動画です。

https://user-images.githubusercontent.com/45844173/220177958-479912c3-afe2-4eed-853c-c494167492e4.mp4



## Requirement


## Usage


## Author
飯田一成（金沢工業大学 工学部 ロボティクス学科 出村研究室）
