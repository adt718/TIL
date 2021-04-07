# ECSとは
Amazon Elastic　Container Service(以下ECS)は、Dockerコンテナ環境を提供するサービスである。

# ECSの特徴
EC2インスタンス上で実行されるコンテナのことをTaskと呼び、EC2インスタンスのことはClusterと呼ぶ。
１つのCluster上で複数のTaskを実行することができる。Cluster上で動作するTaskの定義はTask Definitionで行う。
Taskの役割ごとにTask Definitionを用意し、それを基にClusterの上でTaskが起動する。
同じTaskを複数用意したい場面がある。例えば、WebサーバーTaskを複数用意し、ELBに紐付けるときなどである。
そのような場面で用いるのがServiceである。
Serviceでは、「Webサーバー用のTask DefinitionでTaskを4つ起動する」といった指定ができる。

# AWSにおけるその他のコンテナサービス
ECS以外にも、AWSには3つのコンテナ関連サービスがある。
・AWS　Fargate ・Amazon Elastic Container Service for Kubernetes(EKS)　・Amazon Elastic Container Registry(ECR)
ECSでは、Cluster用のEC2インスタンスが必要。そのため、そのEC2インスタンス自体の管理、例えばAuto　Scalingの設定などは利用者側で
意識する必要があった。

# AWS　Fargate
AWS　Fargateは、このEC2を使わずにコンテナを動かすことができるサービスである。

# Kubernetes
Kubernetesは、コンテナ管理の自動化のためのオープンソースプラットフォームである。
Amazon Elastic Container Service for Kubernetes(EKS)は、このマスターをサービスとして提供するサービスである。
マスター用のEC2インスタンスを管理する必要がなくなり、差別化を生む機能の開発により集中できるようになる。

# Amazon Elastic Container Registry(ECR)
Dockerを用いる場合、そのコンテナイメージをレジストリ(ストレージとコンテント配送サービス）で管理する必要がある。
自前で運用する場合、レジストリ自体の可用性を高める設計が必要になる。
このレジストリをサービスとして提供するのがAmazon Elastic Container Registry(ECR)である。
レジストリの管理をECRに任せることができる。レジストリへのpush/pull権限をIAMで管理することも可能である。
