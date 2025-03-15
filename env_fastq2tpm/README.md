# 水野班基本環境2025
  
## 必要なもの
VScode (すでに実装済みと思われる)  
このenv

## 水野班基本環境2025の解説
- numpy1.26.4、numpy2.1.3の二つのdockerを構築
- 水野班で使用していると思われるライブラリは基本搭載(openbabel, openmmは除く)
- devcontainer * docker composeにすることで従来の使用感を再現

## 手順
### ①VSCodeを使用したリモート接続
(2023年度の吉開さんの定例調査を参考)  

#### 1. ローカルPCのVSCodeにRemote SSHの拡張機能をインストール
拡張機能からRemote-SSHという拡張機能をインストールする。  
また, そもそもホストにDockerがインストールされていない場合はインストールしておく。

#### 2. SSH鍵の作成
SSHの接続方法は公開鍵方式とパスワード方式がありますが, 公開鍵方式を使う場合はSSH鍵を作成する必要があります。parentなど踏み台サーバを経由する場合, ローカル→踏み台 の鍵をローカルPCに作成する必要があります。  

***Remote SSHではフォルダを開き直す度にパスワードが要求されるので, 踏み台を経由する場合は公開鍵方式でローカル→踏み台のみパスワード有が良いと思います。***

#### 3. ローカル → 計算ノードへSSH接続するconfigを作成
Remote-SSHではSSH configに書かれている項目を読み込み, その中から接続先を選択するので,　SSH configを書く必要があります。 ```~/.ssh/config```を作成してその中に設定を書いてください。  

オススメの記法は ***parentまで*** を別に記載し、それを参照させて計算機に飛ばす方法です。


##### 例1. parentまでアクセスする
```
# ~/.ssh/config
Host <適当な名前(name1)>
  Hostname <parentのIPアドレス>
  Port <parentのSSHポート>
  User <parentのユーザ名>
  IdentityFile <2.で作成したSSH秘密鍵のパス>
```

##### 例2. config内のparentへの情報(例1)を使用して計算機へアクセスする
```
# ~/.ssh/config
Host <適当な名前(name2)>
  UserKnownHostsFile=/dev/null
  Hostname <計算ノードのIPアドレス>
  Port <計算ノードのSSHポート>
  User <計算ノードのユーザ>
  ProxyCommand ssh -W %h:%p (例1のname1)
```
特にUserKnownHostsFile=/dev/nullは大切である。
例えば、一つ目の親機の直下の192.???.??.12と2つ目の親機の直下の192.???.??.12が割り当てられた場合、KnownHostsFileへの保存はIPを基に行われるため、conflictし、接続でエラーが起こる。

#### 4. VSCodeからSSH接続
```ctrl + B``` で表示、非表示ができる普段エクスプローラーが開かれている左端のバーに、リモートエクスプローラーがある。  
ここの、一番上の選択肢が```リモート（トンネル/SSH）```になっていると、```~/.ssh/config```に登録したマシンが出てくるので、初回はそのマシンの名前を押す。すると、そのマシンのホームディレクトリに接続できる。

(2回目以降は、そのマシンのトグルに接続頻度が高いディレクトリ(?)が表示される。そこで、自分が設置したenv fileを選択するとよい。)

### ②VSCode dev containerの利用
dev containerはVSCodeの拡張機能の1つで, ホストのDockerに関する操作を行う。  
dev containerを使うと, VSCode内でdockerを立て, その中でVSCodeを操作できるようにする。  
これにより, dockerの作成~コードの実行をVSCodeだけで完結することができる。  
建てたdocker内で.ipynbをインタラクティブに実行することも可能。

#### 1. dev containerのインストール
左サイドバーの拡張機能マーク→"Dev Containers"と検索してインストールすればよい。

#### 2. env直下へ移動 
例えば、/mnt/cluster/HDD/ohto 直下にこのenvを置くときには、/mnt/cluster/HDD/ohtoよりも上流のfolderを開き、envを設置、envのfolderを開く。  
(folderを開くとは、vscodeの上部の検索窓で、>open folderとうち、そのfolderを開くことである。)

#### 3. Dockerの作成
左下の"><"のようなマークをクリックし, "Reopen in Container"を選択する。  
ctn_numpy1とctn_numpy2の選択肢が出るので、使用するnumpyのversionに合わせて選択する。  
するとDockerイメージのビルドが始まり, 続いてDockerが実行され, フォルダが開かれる。  
(どちらのdockerも構築されたうえで、選択したdockerの内部に入る。)

### Jupyter notebookで環境が認識されない場合
最初にdevcontainerを建てた時は, 作成したpython環境が認識されず, ipynbを開いても「カーネルを選択」の選択肢に表示されない場合がある。  
その場合はまず仮想環境をactivateした状態でターミナルなどで```ipykernel```, ```jupyter```をインストールし, その後以下のコマンドを実行する。
```
ipython kernel install --user --name=<env name> --display-name=<env name>
```
すると入力した```<env name>```でカーネルに表示されるようになる。  
「カーネルの選択」→「別のカーネルを選択...」→「既存のJupyterサーバー...」にある場合もある。

### 構築が遅いな？と思ったら
不要なもの (いらないrequirements.txtのライブラリ、rapids関係、そもそも使わないctn_numpy*) を削除していってください。  
特にparent4の直下で実行する場合には非常に時間がかかる可能性があります。

### エラーが起きたら
エラーが起きると、続いて出ている "Exist code ~" の数字が大切です。
また、ポップアップで "Edit devcontainer.json ~" が出てくるので、それを開くとlogファイルを見ることができます。
具体的にdocker fileのどのコマンドで失敗しているかを見て、検索してください。