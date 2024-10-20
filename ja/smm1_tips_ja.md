# SMM1 TIPS
## 1. マクロの安定化
マクロの実行結果はマクロ開始時の状況によって異なります。

作るモードにおいて、最も安定してマクロを実行させる方法は、マイナスボタンを押すところもマクロデータに含めることです。この方法でdesyncを起こしたことはありません。以下のようになります。
```
M
N*95 // スタートするまで待つ

!tstart
B // 最初のフレーム
```
スタート地点のドアに入る場合は1f早くできます: `N*94 + U`


きろくロボットのコースで遊ぶ場合は、以下のように、ポーズして"スタートからやり直す"を押すところを含めるのが安定します。しかし、"スタートからやり直す"を押してからスタートするまでの間が一定ではないようなので、desyncを起こすことがあります。

```
P*1 // pause
N*21 //　menuが開かれるまで待つ
D*1
A*1 // スタートからやり直す
N*107 // スタートするまで待つ

!tstart
B // 最初のフレーム
```

`N*107`の場所でdesyncすることがあります。`N*108`にするなど、環境に合わせてうまく待ち時間調整してください。

### ドアの待ち時間
コースやバージョンによって異なるようです。(要検証)

以下を参考に-2f~0f程度調整してください。
#### ドア/Pドア
```
U // ドアに入る
N*104 // 待つ
B // 最初のフレーム
```
#### 鍵ドア
```
U // ドアに入る
N*160 // 待つ
B // 最初のフレーム
```

#### 開始時にアイテムを持つ方法
コース開始時やドアから出た直後に、マリオと重なっているアイテムを持つには、`Y*107`などのように、上記の`N`の部分を`Y`にしてください。この場合、`!tstart`をどうするべきかは任せます。

注: 実際は`N*77` + `Y*30`のように、後半の一部だけ`Y`にしても動作すると思います。

## 2. Behavior value(挙動値)
コース内のオブジェクトの挙動を決める値で、コースのセーブデータに含まれています。古いコースを作るモードで開く場合は、開いた後すぐにmemory searcherを使ってこの値を固定してください。そうでないと、Pスイッチなどの挙動が変わってしまいます。
### アドレス検索方法(1.47)

1. 新しいコースをつくり、一つ以上ブロックを置いた状態で`7`をint32で検索する
2. きろくロボットを開き、コース一覧が表示された状態で`0`でfilter
3. きろくロボットから新しいコースをつくり、何も操作しないで`0`でfilterする
4. 一つ以上ブロックを置いた状態で`7`でfilter
 
これで一意に特定できるはずです。

## 3. IGT
残り時間タイマーの内部値を使うと、ドア・土管・パワーアップ・パワーダウンの時間を除いたIGTを、以下の計算方法で得ることができます。

1. T = コースのTimeLimit(500など)
2. V = タイマーの内部値
3. FRAMES = (T * 65536 - V) / 1092
4. IGT = FRAMES / 60

###  アドレス検索方法(1.47)
1. 新しいコースをつくり、"あそぶ"を一度も押さないで`32768000`をint32で検索する
2. "あそぶ"を押して、すぐに"つくる"を押して作るモードに戻り、`65470464`でfilter
3. この時点で二つのアドレスに絞られる。もう一度"あそぶ"を押して、時間経過で減少する方がタイマー。

これでタイマーの内部値のアドレスを一意に特定できるはずです。
      