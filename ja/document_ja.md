# Document (ver 0.1.0)
注1: 現在、このツールには、ゲームのロード処理でランダムにdesyncを起こすという問題があります。

注2: **このツールはオフラインで利用するために作られています。オンライン(Pretendoなど)では使用しないでください。**

## 0. 説明にあたって
- 必須引数は次のように表します。`[argName:type]`
- 任意引数は次のように表します。`[argName:type=defaultValue]`

## 1. CUI Commands
CUIではコマンドを入力することで、ツールを操作できます。

注1: ファイルパスを除いて、Case-insensitiveです。

注2: 指定するファイルは、絶対パスにスペースやASCIIコードで表せない文字を含まないものにしてください。

### Macro関連コマンド

| Command | Description |
|:-----------|:-----------|
|`start`|ロード済みのマクロを実行する|
|`s`|`start`のエイリアス|
|`startwith`|`with`モードでマクロを開始(下記参照)|
|`sw`|`startwith`のエイリアス|
|`stop`|実行中のマクロを停止する|
|`x`|`stop`のエイリアス|
|`load [path:str]`|`[path]`で指定されたマクロをロードする|
|`l [path:str]`|`load [path:str]`のエイリアス|
|`pre`|ロード済みのファイルのパスを表示|
|`reload`|マクロをリロード|
|`r`|`reload`のエイリアス|
|`rs`|`reload`してから`start`する。ショートカット。|
|`rsw`|`reload`してから`startwith`|
|`dumpmacro [skip:int=0]`|ロード済みのマクロを別形式で{MACRO PATH}.dump.txtにtable形式で保存。先頭から`[skip]`フレーム無視する。|
|`dumpmacro2 [skip:int=0]`|`dumpmacro`と同じだが、RLE形式で保存|

#### `with`モードについて
マクロによる操作内容と手動の操作内容を合成する。例えば、`Y>*999`というマクロが実行されている場合、手動で`B`を入力している間は`Y>B`として解釈される。

### その他コマンド

| Command | Description |
|:-----------|:-----------|
|`cls`|CUIをクリア|
|`pwd`|{LCEMU_PATH}/macro　の絶対パスを表示|
|`seqpos [x:int] [y:int]`|Input Viewerを(`[x]`, `[y]`)に移動させる|
|`seqcfg [next:int] [render:int] [frametype:int]`|Input Viewerで表示する内容を制御する。<br>`[next]`: 先何fの入力まで表示するか。0にすると現在のフレーム分しか表示されない。<br>`[render]`:現在のフレームの入力をどう表示するか(`0`: 1行, `1`:コントローラ風)<br>`[frametype]`: frame数の表示設定(`0`:表示しない, `1`:終了時や!SHOWFRAMESコマンド時のみ, `2`: 全て表示)  |

## 2. Macro

- マクロはテキストファイルで表現されます。
- 1行が1つのコードを表します。
- コードは上の行から順に実行されます。
- 各行について、"//"の後ろは無視されます。(コメント)
- 空行は無視されます。
- 各行について、先頭と末尾の空白は無視されます。
- Case-insensitive(アルファベットの大文字小文字を区別しない)

Example: 以下の二つは全く同じ意味です。
```
M
            n*95
Y>*40
// hello
Y//B*20
//Y
                b
```
```
M
N*95
Y>*40
Y
B
```
コードには以下の三種類あります。
- "!"から始まるコード: Command
- "#"から始まるコード: Preprocessor
- それ以外: 入力情報

コードは以下の形式です。 

- `[code:str]*[repeat:int]`
- `[code:str]` (これは、`[code:str]*1`と同じ意味です。)

`[repeat]`で指定した分、codeの内容を繰り返します。

### 入力情報
入力情報は、入力するボタンを表す文字からなる文字列です。
- 間にある空白は無視されます。
- 一つの文字が一つのボタンに対応しています。
- 文字の順番は任意です。

このコードはフレームを消費します。

注: フレームを消費するとは、そのコードが実行されると、次のコードが実行されるのは1フレーム後という意味です。

Example: 以下の二つは全く同じ意味です。
```
>
>
>
Y>B
Y>B
```
```
>*3 // 3 frames
BY>*2 // 2 frames
```

#### ボタンを表す文字一覧
| Letters | Description |
|:-----------|:-----------|
|`A`,`B`,`Y`,`X`,`L`,`R`|それぞれ、A,B,Y,X,L,Rボタン|
|`P`|Plus|
|`M`|Minus|
|`J`|`B`のエイリアス(Jump)|
|`S`|`L`のエイリアス(Spin)|
|`^` or `U`|UP|
|`V` or `D`|DOWN|
|`>`|RIGHT|
|`<`|LEFT|
|`N`|NULL(何も入力しない)|

何も入力しないフレームを表現したい場合は、`N`を使ってください。空行は無視されてしまいます。`*[repeat]`も無視されます。代わりに`N*[repeat]`を使ってください。

### Preprocessor
Preprocessorは、マクロのロード時に特殊な処理を行います。`[repeat]`は使用できません。

| Preprocessor | Description |
|:-----------|:-----------|
|`#SUB [name:str]`|次の#ENDSUBまでの内容を`[name]`という名前のサブルーチンとして定義する|
|`#ENDSUB`|`#SUB`の終わりを表す|
|`#ESUB`|`#ENDSUB`のエイリアス|

### Comand
Commandは実行時に評価されます。フレームは消費しません。!callを除いて`[repeat]`が使えません。

| Command | Description |
|:-----------|:-----------|
|`!SLP [time:int=500]`|ゲームを`[time]` msの間一時停止|
|`!SLPL`|ゲームを1000 msの間一時停止|
|`!QUIT`|実行中のマクロを停止|
|`!TSTART`|Input Viewerがマクロ終了時に表示する合計フレーム数の、基点を設定|
|`!CALL [name:str]`|`[name]`という名前のサブルーチンを実行する。`[repeat]`を指定すると、その分実行される。|
|`!CALL [name:str] [dep:int]`|`!call`が再帰する場合、深さ制限`[dep]`を指定してください|
|`!SEQCFG [next:int] [render:int] [frametype:int]`|CUIの`seqcfg`と同じ|
|`!SHOWFRAMES`|Input Viewerのframetypeが1の時に、このコマンドが実行された時点のフレーム数を表示|


Example1: 以下のマクロは1フレーム分です。
```
!TSART
!SLP
!SLPL
B
```
```
B
!QUIT
// Terminated
BY>*30
<*20 
```
Example2: 以下の三つは全く同じ意味です。
```
#SUB lr
<
>
#ENDSUB

!call lr*3
```
```
!call lr_version_2*3
// Case-insensitive
#SUB LR_vErSioN_2
 <
 >
#ENDSUB
```
```
<
>
<
>
<
>
```

## 3. その他機能
### Frame Advance
メインウィンドウ(ゲームが表示されている場所)で、LEFT-SHIFTを長押しするとFrame Advanceが使えます。