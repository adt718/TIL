# EBS
EBSは、AWSが提供するブロックストレージサービスである。EC2のOS領域として利用したり、追加ボリュームとして複数のEBSをEC2に
アタッチすることもできる。RDSのデータ保存用にも使用する。
EBSは単一のEC2にのみアタッチ可能なサービスであるため、複数のEC2インスタンスから同時にアタッチするといった使い方はできない。
また、EBSは作成時にAZを指定するため、指定したAZに作成されたEC2インスタンスからのみアタッチ可能である。

# 重要ポイント
EBSはブロックストレージサービスで、EC2のOS領域、EC2の追加ボリューム、RDSのデータ保存領域などに使用する。

# EBSのボリュームタイプ
EBSのボリュームタイプはSSDタイプで2種類、HDDタイプで2種類の4つである。
・汎用SSD(gp2)
・プロビジョンドIOPS　SSD(io1)
・スループット最適化HDD(st1)
・Cold HDD(sc1)
旧世代のマグネティックと呼ばれるHDDのストレージタイプもあるが、新規で作成するときはマグネティックタイプは使わずに、現行のボリュームタイプ
から最適なものを選ぶようにしよう！

# 汎用SSD(gp2)
EBSの中で最も一般的な、SSDをベースとしたボリュームタイプである。EC2インスタンスを作成する際のデフォルトボリュームタイプとしても利用されている。
性能の指標としてIOPS(1秒あたりに処理できるI/Oアクセスの数)を用い、3IOPS/GB(最低100IOPS)から最大10000IOPS/ボリュームまで、容量に応じたベースライン性能がある。
このベースライン性能はEBS利用時間の99%で満たされるように設計されている。また、1TB未満のボリュームには、一時的なIOPSの上昇に対応できるようにバースト機能が
用意されており、容量に応じて一定期間3000IOPSまで性能を向上させることができる。

# プロビジョンドIOPS SSD(io1)
プロビジョンドIOPSはEBSの中で最も高性能な、SSDをベースとしたボリュームタイプである。
RDSやEC2インスタンスでデータベースサーバーを構成する場合など、高いIOP性能が求められる際に利用する。io1は最大50IOPS/GB、もしくは最大64000IOPS/ボリューム
まで、容量に応じたベースライン性能がある。このベースライン性能はEBS利用時間の99.9％で満たされるように設計されている。また、スループットもボリュームあたり
最大1000MB/秒まで出るようになっており、IOPS負荷の高いユースケースと、高いスループットが必要なユースケースの両方に適したストレージタイプである。

# スループット最適化HDD(st1)
スループット最適化はHDDをベースとしたスループット重視のボリュームタイプである。ログデータに対する処理やバッチ処理のインプット用のファイルなど、
大容量ファイルを高速に読み取るようなユースケースに適している。スループット(MB/秒)を性能指標として用いており、1TBあたり40MB/秒、最大スループットは
ボリュームあたり500MB/秒のベースライン性能がある。このベースライン性能はEBS利用時間の99%で満たされるように設計されている。

# Cold HDD(sc1)
Cold HDDは4つのストレージタイプの中でストレージとしての性能はそれほど高くないが、最も低コストなボリュームタイプである。利用頻度があまりなく、
アクセス時の性能もそれほど求められないデータをシーケンシャルにアクセスするようなユースケースやアーカイブ領域の用途に適している。
1TBあたり12MB/秒、ボリュームあたり、最大250MB/秒のベースライン性能がある。

# ベースライン性能とバースト性能
プロビジョンドIOPS以外のストレージタイプには、ストレージの容量に応じてベースライン性能がある。
これらのストレージタイプには、ベースライン性能とは異なり、処理量の一時的な増加に対応可能なバースト性能という指標もある。
バースト性能はあくまで一時的な処理量の増加への対応に使われることを想定したものと理解しておき、バースト性能に頼ったサイジングはしないようにしよう。
# 重要ポイント
バースト性能は処理量の一時的な増加に対応する能力を示すものなので、これに頼ったサイジングはすべきでない。

# EBSの拡張・変更
注意点
1.EBSボリュームに対して変更作業を行った場合、同一のEBSボリュームへの変更作業は6時間以上開ける必要がある。
2.現行世代以外のEC2インスタンスタイプで使用中のEBSボリュームに対する変更作業では、インスタンスの停止やEBSのデタッチが必要になる場合がある。
# 容量拡張
全てのタイプのEBSは1ボリュームあたりの最大容量が16TBである。ディスク容量が不足したら必要に応じてサイズを何度でも拡張できる。
オンラインで使用中のEBSボリュームを拡張した後は、EC2インスタンス上でOSに応じたファイルシステムの拡張作業を別途実施して、OS側でも認識できるようにした方がいい。
注意点
1.拡張はできるが縮小はできない。一時的なデータ容量の増加などの要件に対しては、ボリュームの拡張ではなく、新規EBSごと削除するといった方法を検討しよう！

# ボリュームタイプの変更
4つの現行世代タイプ間でのタイプ変更が可能である。gp2タイプで作成したが、IOPSが不足することが分かったためio1タイプに変更したい、といった
要件に対応できる。また、io1タイプで指定したIOPSが足りない場合に追加のプロビジョニングを実施することも可能である。
注意点
1.プロビジョンドIOPSタイプで指定したIOPS値については、増減のどちらの変更も可能である。
2.IOPSの変更には最大24時間かかる場合がある。変更期間中はボリュームのステータスが「Modifying」になっている。ステータスが「Complete」になっていれば完了。

# 可用性・耐久性
EBSは内部的にAZ内の複数の物理ディスクに複製が行われており、AWS内で物理的な故障が発生した場合でも利用者が意識することはほとんどない。
SLAは月あたりの利用可能時間が99.99%と設定されている。
また、EBSにはスナップショット機能もあるため、定期的にバックアップを取得することで必要な時点の状態に戻すことが可能である。
データのリストアは、スナップショットから新規EBSボリュームを作成し、EC2インスタンスにアタッチすることで実現できる。

# セキュリティ
EBSには、ストレージ自体を暗号化するオプションがある。暗号化オプションを有効にすると、ボリュームが暗号化されるだけではなく、暗号化されたボリュームから
取得したスナップショットも暗号化される。暗号化処理はEC2インスタンスが稼働するホストで実施されるため、EBS間をまたぐデータ通信時のデータも暗号化された状態となる。
すでに作成済みのEBSボリュームを暗号化したい場合は、次の手順を踏む
1.EBSボリュームのスナップショットを取得
2.スナップショットを暗号化
3.暗号化されたスナップショットから新規EBSボリュームを作成
4.EC2インスタンスにアタッチしているEBSボリュームを入れ替え
既存のブートボリュームを暗号化する場合は、スナップショットではなくAMIを取得して、AMIコピー時に暗号化を実施したのち、コピーされたAMIからEC2インスタンス
を作成することで暗号化が可能である。