# RDS
RDSは、マネージドRDBサービスである。MySQL、MariaDB、PostgreSQL、Oracle、Microsoft SQL Serverなどのオンプレミスでも使い慣れたデータベースエンジンから
好きなものを選択できる。
RDSでは、複数のデータベースエンジンを利用できるが、それぞれのエンジンで提供されている機能のうち、RDSでは使用できない機能もある。

# 重要ポイント
・RDSはマネージドRDBサービスで、最大のメリットは運用の効率化。オンプレミスでも使い慣れたデータベースエンジンから好きなものを選択できる。
AWS独自のAuroraも選択可能である。

# RDSで使えるストレージタイプ
EBSの中でもRDSで利用可能なストレージタイプは、汎用SSD、プロビジョンドIOPS　SSD、マグネティックの３つである。
マグネティックは過去に作成したDBインスタンスの下位互換性維持のために利用可能となっているが、新しいDBインスタンスを作成するときには基本的にSSDを選択すること。
プロビジョンドIOPSは高いIOPSが求められる場合や、データ容量と比較してI/Oが多い場合に利用を検討する。
ストレージの容量は32TB（Microsoft SQL Serverは16TB）まで拡張が可能である。

# マルチAZ構成
マルチAZ構成とは、１つのリージョン内の２つのAZにDBインスタンスをそれぞれ配置し、障害発生時やメンテナンス時のダウンタイムを短くすることで高可用性を実現するサービスである。
DBインスタンス作成時にマルチAZ構成を選択するだけで、後はすべてAWSが自動でDBの冗長化に必要な環境を作成してくれる。
本番環境でRDSを利用するときはマルチAZ構成を推奨する。
#### 注意点
・書き込みが遅くなる：２つのAZ間でデータを同期するため、シングルAZ構成よりも書き込みやコミットにかかる時間が長くなる。本番環境でマルチAZ構成を利用する場合は、
性能テスト実施時にマルチAZ構成にした状態でテストしよう。AWSのコストを抑えるために開発環境はシングル構成にして、その環境を使って性能テストを実施したために
本番のマルチAZ構成だと想定した性能が出なかった、ということもある。

・フェイルオーバーには６０～１２０秒かかる：フェイルオーバーが発生した場合、RDSへの接続用FQDNのDNSレコードが、スタンバイ側のIPアドレスに書き換えられる。
異常を検知してDNSレコードの情報が書き換えられ、新しい接続先IPの情報が取得できるようになるまではDBに接続することはできない。
アプリケーション側でDB接続先IPのキャッシュを持っている場合は、RDSフェイルオーバー後にアプリケーションからRDSに接続できるようになるまで、１２０秒以上の時間が
かかることもある。

# リードレプリカ
リードレプリカとは、通常のRDSとは別に、参照専用のDBインスタンスを作成することができるサービスである。リードレプリカが利用可能なデータベースエンジンは
Aurora、MySQL、MariaDB、PostgreSQLの４つである。OracleやMicrosoft SQL Serverでは使えない。
リードレプリカを作成することで、マスターDBの負荷を抑えたり、読み込みが多いアプリケーションにおいてDBリソースのスケールアウトを容易に実現することが可能である。
例えば、マスターDBのメンテナンス時でも参照系サービスだけは停止したくないという場合に、アプリケーションの接続先をリードレプリカに変更した状態でマスターDBの
メンテナンスを実施する。
マスターとリードレプリカのデータ同期は、非同期レプリケーション方式である点は覚えておく必要がある。そのため、リードレプリカを参照するタイミングによっては、マスター側で
更新された情報が必ずしも反映されていない可能性がある。しかし、リードレプリカを作成しても、マルチAZ構成のスタンバイ側へのデータ同期のようにマスターDBのパフォーマンスに影響を
与えることはほとんどない。
# 重要ポイント
・マスターとリードレプリカのデータ同期は非同期レプリケーション方式なので、タイミングによっては、マスターの更新がリードレプリカに反映されていない事がある。

# バックアップ/リストア
・自動バックアップ：バックアップウィンドウと保持期間を指定することで、１日に１回自動的にバックアップ(DBスナップショット)を取得してくれるサービスである。
バックアップの保持期間は最大３５日である。バックアップからDBを復旧する場合は、取得したスナップショットを選択して新規RDSを作成する。稼働中のRDSにバックアップの
データを戻すことはできない。削除するDBインスタンスを再度利用する可能性がある場合は、削除時に最終スナップショットを取得するオプションを利用しよう。
・手動スナップショット：任意のタイミングでRDBのバックアップ(DBスナップショット)を取得できるサービスである。必要に応じてバックアップを取得できるが、手動スナップショットは
１リージョンあたり１００個までという取得数の制限がある。RDS単位ではなくリージョン単位の制限であることに注意。また、シングルAZ構成でスナップショットを取得する場合、
短時間のI/O中断時間があることも要注意である。この仕様は自動バックアップでも同じである。マルチAZ構成の場合はスタンバイ側のDBインスタンスには影響を与えない。
この点からも、本番環境でRDSを使うときはマルチAZ構成を推奨。
・データのリストア：RDSにデータをリストアする場合は、自動バックアップ、及び手動で取得したスナップショットから新規のRDSを作成する。スナップショット一覧から戻したいスナップショット
を選択するだけで、非常に簡単にデータをリストアできる。
・ポイントインタイムリカバリー：直近５分前から最大３５日前までの任意のタイミングの状態のRDSを新規に作成することができるサービスである。戻すことができる最大日数は自動バックアップ
の取得期間に準じる。そのため、ポイントインタイムリカバリーを使用したい場合は自動バックアップを有効にする必要がある。

# セキュリティ
データベースには個人情報などの重要な情報や機微な情報を格納することもある為、セキュリティには特に注意を払う必要がある。RDSには２つのセキュリティサービスが存在する。

・ネットワークセキュリティ：RDSはVPCに対応しているため、インターネットに接続できないAWSのVPCネットワーク内で利用可能なサービスである。DBインスタンス作成時にインターネットからの
接続を許可するオプションもあるが、デフォルトではOFFになっている。また、EC２と同様、セキュリティグループによる通信要件の制限が可能である。EC2や他のAWSサービスからRDSまでの
通信も、各データベースエンジンが提供するSSLを使った暗号化に対応している。
・データ暗号化：RDSの暗号化オプションを有効にすることで、データが保存されるストレージ(リードレプリカ用も含む)やスナップショットだけでなく、ログなどのRDSに関連するすべてのデータが
暗号化された状態で保持される。このオプションは途中から有効にすることはできない。すでにあるデータに対して暗号化を実施したい場合は、スナップショットを取得してスナップショットの
暗号化コピーを作成する。そして、作成された暗号化スナップショットからDBインスタンスを作成することで既存データの暗号化がなされる。

# Amazon Aurora
#### Auroraの構成要素
Auroraでは、DBインスタンスを作成すると同時にDBクラスタが作成される。DBクラスタは、１つ以上のDBインスタンスと、各DBインスタンスから参照するデータストレージ(クラスタボリューム)
で構成される。Auroraのデータストレージは、SSDをベースとしたクラスタボリュームである。クラスタボリュームは、単一リージョン内の３つのAZにそれぞれ２つ(計６つ)のデータコピーで
構成され、各ストレージ間のデータは自動的に同期される。また、クラスタボリューム作成時に容量を指定する必要がなく、Aurora内に保存されるデータ量に応じて最大64TBまで自動的に
拡縮する。

# Auroraレプリカ
Auroraは他のRDSと異なりマルチAZ構成オプションはない。しかし、Auroraクラスタ内に参照専用のレプリカインスタンスを作成することができる。他のRDSのリードレプリカとの
違いは、Auroraのプライマリインスタンスに障害が発生した場合にレプリカインスタンスがプライマリインスタンスに障害が発生した場合にレプリカインスタンスがプライマリインスタンスに
昇格することでフェイルオーバーを実現する点である。

# エンドポイント
通常、RDSを作成すると接続用エンドポイント(FQDN)が１つ作成され、そのFQDNを使ってデータベースに接続する。Auroraでは、次の３種類のエンドポイントが作成される。
・クラスタエンドポイント：Auroraクラスタのうち、プライマリインスタンスに接続するためのエンドポイントである。クラスタエンドポイント経由で接続した場合、データベースへの
全ての操作(参照・作成・更新・削除・定義変更)を受け付けることができる。
・読み取りエンドポイント：Auroraのクラスタのうち、レプリカインスタンスに接続するためのエンドポイントである。読み取りエンドポイント経由で接続した場合、データベースに対しては
参照のみを受け付けることができる。Auroraクラスタ内に複数のレプリカインスタンスがある場合は、読み取りエンドポイントに接続することで自動的に負荷分散が行われる。
・インスタンスエンドポイント：Auroraクラスタを構成する各DBインスタンスに接続するためのエンドポイントである。接続したDBインスタンスがプライマリインスタンスである場合は全て
操作が可能である。レプリカインスタンスである場合は参照のみ可能となる。特定のDBインスタンスに接続したいという要件の場合に使用する。

#### カスタムエンドポイント
Auroraにはもう１つ、カスタムエンドポイントと呼ばれるエンドポイントがある。これはAuroraクラスタを構成するインスタンスのうち、任意のインスタンスをグルーピングしてアクセスする
場合に使う。例えば、読み取りエンドポイントだとレプリカインスタンス全体にアクセスが分散されるが、カスタムエンドポイントを使って、レプリカインスタンスをWebサービス用とバッチ処理用
に分けることで、バッチで実行する負荷の高い参照処理がWebサービスの参照に影響を与えないような構成をとることができる。
