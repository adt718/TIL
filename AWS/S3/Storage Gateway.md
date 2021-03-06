# Storage Gateway
Storage Gatewayは、オンプレミスにあるデータをクラウドへ連携するための受け口を提供するサービスである。Storage Gatewayを使って連携されたデータの
保存先には、S3やGlacierといった、耐久性が高く低コストなストレージが利用される。Storage GatewayのキャッシュストレージとしてEBSが使われる。

このように、Storage Gatewayはサービスとして独自のストレージを持っているわけではない。ストレージを組み合わせて、オンプレミスとAWS間のデータ連携を
容易にするためのインターフェイスを連携するサービスだと考えられる。

オンプレミスとのハイブリッド環境であり、参照頻度が高いデータはオンプレミスの高速ストレージに保存し、参照頻度が低いデータやバックアップデータは
Storage Gatewayを利用してクラウドに保管するといった使い分けもできる。そのため、利用目的を明確にすることで、大容量のデータを効率的に管理できる。

# 重要ポイント
Storage Gatewayは、オンプレミスにあるデータをクラウドへ連携するためのインターフェイスを提供するサービス。独自のストレージを持たず、S3、Glacier、
EBSなどを利用する。

# Storage Gatewayのタイプ
Storage Gatewayには次の３種類のゲートウェイタイプが用意されている。
#### ファイルゲートウェイ
#### ボリュームゲートウェイ
#### テープゲートウェイ

ボリュームゲートウェイには２つのボリューム管理方法がある。
#### キャッシュ型ボリューム
#### 保管型ボリューム

# ファイルゲートウェイ
S3をクライアントサーバーからNFSマウントして、あたかもファイルシステムのように扱うことができるタイプのゲートウェイである。作成されたファイルシステムは
非同期であるが、ほぼリアルタイムでS3にアップロードされる。アップロードされたファイルは１ファイルごとにS3のオブジェクトとして扱われるため、保存された
データにS3のAPIを利用してアクセスすることも可能である。注意点として、データの書き込みや読み込みの速度がローカルディスクに比べて遅いことが挙げられる。

# ボリュームゲートウェイ
データをS3に保存することはファイルゲートウェイと同じだが、各ファイルをオブジェクトとして管理するのではなく、S3のデータ保存領域全体を１つのボリューム
として管理する。そのため、S3に保存されたデータにS3のAPIを利用してアクセスすることはできない。クライアントサーバーからこのタイプのゲートウェイへの接続方式は
NFSではなく、isCSI接続になる。ボリュームはスナップショットを取得することができるため、スナップショットからEBSを作成し、EC２インスタンスにアタッチすることで、
スナップショットを取得した時点までのデータにアクセスできるようになる。

#### キャッシュ型ボリューム

頻繁に利用するデータはStorage Gateway内のキャッシュディスクに保存して高速にアクセスすることを可能とし、すべてのデータを保存するストレージとしてS3を利用する
タイプのボリュームゲートウェイである。データ量が増えたとしてもローカルディスクを拡張する必要がなく、効率的に大容量データを管理できる。
キャッシュ上に存在しないデータにアクセスする場合はS3から取得する必要があるため、データ読み込みの速度がシステム上問題になる場合には適さない。

キャッシュ型ボリュームボリュームゲートウェイでは、キャッシュボリュームとアップロードバッファボリュームにストレージを使用する。
オンプレミスの場合は仮想アプライアンスが実行される環境にあるストレージを使用し、AWSにStorage Gatewayを構成する場合にはEBSを利用する。
キャッシュ型ボリュームは頻繁に使用するデータに対して高速にアクセスするためのもので、アップロードバッファボリュームは、S3にアップロードするデータを
一時的に保管するためのものである。

#### 保管型ボリューム
すべてのデータを保存するストレージ(プライマリストレージ)としてローカルストレージを利用し、データを定期的にスナップショット形式でS3へ転送するタイプのボリュームゲートウェイ
である。S3へ転送されたスナップショットはEBSとしてリストア可能なため、必要に応じてEC2インスタンスにアタッチすることでデータを参照することができる。
全てのデータがローカルストレージに保存されるため、データへのアクセス速度はStorage Gateway導入によって変化することはない。
オンプレミスのデータを定期的にクラウドへバックアップする用途に適している。

# テープゲートウェイ
テープデバイスの代替としてS3やGlacierにデータをバックアップするタイプのゲートウェイである。物理のテープカートリッジを入れ替えたり遠隔地にオフサイト保存する
といったことをする必要がなくなる。サードパーティ製のバックアップアプリケーションと組み合わせることができるため、すでにバックアップにテープデバイスを利用している
場合は、比較的簡単にStorage Gatewayへの移行が可能である。

# セキュリティ
Storage Gatewayのセキュリティ要素には次の３つがある。
#### CHAP認証
クライアントからStorage GatewayにiSCSIで接続する際に、CHAP認証を設定することができる。CHAP認証を設定することで、不正なクライアントからのなりすましを
防止でき、また、通信の盗聴といった脅威に対するリスクを軽減できる。
#### データ暗号化
Storage GatewayではAWS KMSを使ってデータの暗号化が可能である。暗号化されるタイミングはデータが保管されるときであるため、S3に保存されるタイミングで
暗号化される。また、暗号化されたボリュームから取得したスナップショットも同じキーで暗号化されている。
#### 通信の暗号化
オンプレミス環境からStorage Gatewayを経由してS3にデータを転送する際にはHTTPSが使用されるため、通信時のデータ内容は暗号化される。
