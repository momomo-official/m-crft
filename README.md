# m-crft
m-crftは3Dサンドボックスゲームです。  
3DCGはDxLibを使って実装されています。  
Windowsでのみ動きます。LinuxとMacには対応していません。  
![Screenshot](https://raw.githubusercontent.com/momomo-official/m-crft/screenshot/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202018-08-19%2017.04.48.png)
  
### なぜm-crftを作ってるの？目的は？
* 3DCGの表現技法を一通り扱ってみるための題材として
	* 水がどんどんきれいになっていくのが楽しい
* ボクセルエンジンの最適化を楽しむため
* [`ももも`](https://twitter.com/mcrft_momomo)がかわいいため
  
### m-crftの特徴
* 無限の広さを持つ世界。
* パーリンノイズによる自然な地形。4つのバイオームがあります。
* Shaderによる基本的な3DCG表現。（影、光、水面反射など）
* マルチプレイ対応！チャットも可能。
* 便利なコマンドにより、ブロックによる建築がサポートされます。
  
## ダウンロード
  
実行バイナリは以下のリンクからダウンロードできます。  
[TODO:link]
  
## 開発環境
Visual Studio 2017 CommunityとDxLibを使って開発しています。  
ソリューションファイルが含まれているので、Cloneすればそのまま動かすことができます。  
ただし、DxLibの設定は各自で行っていただく必要があります。DxLibの設定はこちらの[リンク](http://dxlib.o.oo7.jp/use/dxuse_vscom2017.html)を参照してください。  
Cドライブ直下にDxLibの`プロジェクトに追加すべきファイル_VC用`が置いてあるものとしてプロジェクトの設定をしてあります。  
Cドライブ直下にない場合はDxLibの設定を行ってください。  
  
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
Worldは9×9のChunkの集まりです。Chunkは16×16×256個のBlockの集まりです。Worldを構成するChunkの数は実行中に変更することができます。プレイヤーがWorldの中心のChunkから別のChunkへと移動するたびに、必要なChunkを最生成、またはメモリから読み出します。

### 地形生成
地形はパーリンノイズに従って生成されます。ただ純粋にパーリンノイズを用いているのではなく、川や海に面した地形が不自然にならないよう高さを調整したり、洞窟をつくるための調整はしています。

### バイオーム
今のところ4つだけです。

### マルチプレイ
マルチプレイはDxLibの通信機能を用いて行われています。

### メッシュ構築
メッシュの数はレンダリング時間に大きな影響を与えます。そのため、なるべく少ないメッシュで世界を構築することが求められます。  
m-crftでは、`Greedy Mesh`というメッシュ構築アルゴリズムを用いています。  
以下のURLを参考にしました。  
[https://0fps.net/2012/06/30/meshing-in-a-minecraft-game/]  
  
## Shader
Shaderを用いて、以下のことを表現しています。  
* 影  
* 水面反射  
* さざ波  
* アンビエントオクルージョン  
* ライトブルーム・ゴッドレイ  
* フォグ  
* カラーライト  

### 影
影は、深度バッファシャドウアルゴリズムを用いて表現しています。  
* `Cascade Shadow Mapping`  
* `Variance Shadow Mapping`  
* `Gaussian Blur`  
の三つを組み合わせています。  
[DirectXのチュートリアル](https://msdn.microsoft.com/ja-jp/library/ee416307(v=vs.85).aspx)と
[リポジトリ](https://github.com/walbourn/directx-sdk-samples/tree/master/CascadedShadowMaps11)を参考にしました。  

### 水面反射
![ScreenShot](https://raw.githubusercontent.com/momomo-official/m-crft/screenshot/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202018-08-19%2017.21.16.png)
水面反射はScreen Space Reflection(ssr)を用いて表現しています。  
よく用いられるのはカメラを水面の下に持ってきて、そこから得られる画像を水面に描画する方法ですが、この方法は水面が一定の高さにあるシーンでしか用いることができません。  
ssrを用いると画面外にある物体を水面に反射させることはできませんが、ある程度のクオリティの描画結果を得ることができます。

### さざ波
動的法線マップを用いて表現しています。以下のような方法です。  
* ハイトマップを作成
* 波動方程式に基づいて波のシミュレーションを行い、ハイトマップを次の状態に更新
* ハイトマップに基づいて法線マップを作成
* 法線マップを水面にテクスチャマッピング

### アンビエントオクルージョン
`アンビエントオクルージョン`は、`遮蔽度`を表現するための技法です。  
以下のURLを参考にしました。  
[http://0fps.wordpress.com/2013/07/03/ambient-occlusion-for-minecraft-like-worlds/]
  
### ライトブルーム・ゴッドレイ
![ScreenShot](https://raw.githubusercontent.com/momomo-official/m-crft/screenshot/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202018-07-29%2021.04.04.png)  
太陽に関する表現技法です。ライトブルームは太陽をまぶしく見せる表現、ゴッドレイは木漏れ日のような表現です。  
以下のURLを参考にしました。  
[http://maverickproj.web.fc2.com/pg65.html]

### フォグ  
![ScreenShot](https://raw.githubusercontent.com/momomo-official/m-crft/screenshot/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202018-07-29%2021.19.20.png) 
遠景には距離フォグをかけています。以下のURLを参考にしました。  
[https://wgld.org/d/webgl/w060.html]

### カラーライト
![ScreenShot](https://raw.githubusercontent.com/momomo-official/m-crft/screenshot/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202018-03-22%2023.57.33.png)
以下のURLを参考にしました。  
[https://www.seedofandromeda.com/blogs/29-fast-flood-fill-lighting-in-a-blocky-voxel-game-pt-1]
