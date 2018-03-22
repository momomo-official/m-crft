# m-crft
m-crftは3Dサンドボックスゲームです。  
3DCGはDxLibを使って実装されています。  
Windowsでのみ動きます。LinuxとMacには対応していません。
（本当はMEIT Adventureという名前なのですが、m-crftの方が呼びやすいのでリポジトリ名にしました）

![Screenshot](https://raw.githubusercontent.com/momomo-official/m-crft/screenshot/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202018-03-22%2023.28.42.png)

### m-crftの特徴
* 無限の広さを持つ世界。
* パーリンノイズによる自然な地形。4つのバイオームがあります。
* Shaderによる基本的な3DCG表現。（影、光、水面反射など）
* マルチプレイ対応！チャットも可能。
* 便利なコマンドにより、ブロックによる建築がサポートされます。

## ダウンロード

実行バイナリは以下のリンクからダウンロードできます。  
[downloadlink]

## 開発環境
Visual Studio 2017 Communityとを使って開発しています。ソリューションファイルが含まれているのでCloneすればそのまま動かすことができますが、  
DxLibの設定は各自で行っていただく必要があります。Cドライブ直下にDxLibの_プロジェクトに追加すべきファイル_VC用_が置いてある場合は追加の設定は必要ありません。DxLibの設定はこちらのリンクを参照してください。  
http://dxlib.o.oo7.jp/use/dxuse_vscom2017.html  

## 操作方法
	a:          左に移動  
	s:          後ろに移動  
	d:          右に移動  
	w:          前に移動  
	e:          インベントリを開く 
	/:          コマンドモードを開く  
	f:          ダッシュキー。このキーを押しながらasdwを押すと高速移動。spaceを押すと大ジャンプ。  
	space:      ジャンプ  
	右クリック:  ブロック設置  
	左クリック:  ブロック破壊  
	マウス移動:  カメラの方向を変える  
	ホイール:    アイテムを変更 

## 実装について

### 世界の設計
Worldは9×9のChunkの集まりです。Chunkは16×16×256個のBlockの集まりです。Worldを構成するChunkの数は実行中に変更することができます。プレイヤーがWorldの中心のChunkを移動するたびに、必要なChunkを最生成、またはメモリから読み出します。

### 地形生成
地形はパーリンノイズに従って生成されます。ただ純粋にパーリンノイズを用いているのではなく、川や海に面した地形が不自然にならないよう高さを調整したり、洞窟をつくるための調整はしています。

### バイオーム

### マルチプレイ
マルチプレイはDxLibの通信機能を用いて行われています。
今のところClientがカラーライトを設置するとバグってしまいます。

### メッシュ構築
メッシュの数はレンダリング時間に大きな影響を与えます。そのため、なるべく少ないメッシュで世界を構築することが求められます。
メッシュの考えについては以下のURLが参考になるでしょう。  
https://0fps.net/2012/06/30/meshing-in-a-minecraft-game/  
m-crftでは上記URLでいう、Culling Meshを用いています。さらに少ないメッシュで世界を構築できるGreedy Meshにも挑戦中ですが、変更量が膨大になるためいつになるかわかりません。

[URL]
## Shader

### 影
影は深度バッファシャドウアルゴリズムによって実装されています。実装は[リンク]にあります。  
Cascade Shadow Mapping, Variance Shadow Mapping, Gaussian Blurを組み合わせています。  
DirectXのチュートリアル  
[https://msdn.microsoft.com/ja-jp/library/ee416307(v=vs.85).aspx]  
とリポジトリ  
[url]  
を参考にしました。  

### 水面反射
[ScreenShot]
水面反射はScreen Space Reflection(ssr)によって実装されています。  
実装は[リンク]にあります。  
よく用いられるのはカメラを水面の下に持ってきて、そこから得られる画像を水面に描画する方法ですが、この方法は水面が一定の高さにあるシーンでしか用いることができません。  
ssrを用いると画面外にある物体を水面に反射させることはできませんが、ある程度のクオリティの描画結果を得ることができます。

### さざ波
法線マップをスライドさせて水が流れているように見せています。さざ波とは言いがたいです。

### アンビエントオクルージョン
アンビエントオクルージョンは以下のURLに示すように実装されています。  
[http://0fps.wordpress.com/2013/07/03/ambient-occlusion-for-minecraft-like-worlds/]  

### ライトブルーム
太陽をまぶしく見せるための表現です。  
[ScreenShot]
以下のURLに示す考えで実装されています。
[http://maverickproj.web.fc2.com/pg45.html]

### ゴッドレイ
木漏れ日のような光の筋を表現するための技法です。
[ScreenShot]
以下のURLに示す考えで実装されています。
[http://maverickproj.web.fc2.com/pg65.html]

### カラーライト
[ScreenShot]
カラーライトは以下のURLに示す方法で実装されています。  
[https://www.seedofandromeda.com/blogs/29-fast-flood-fill-lighting-in-a-blocky-voxel-game-pt-1]
