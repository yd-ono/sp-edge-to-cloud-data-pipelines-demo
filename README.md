# Solution Pattern: Edge-to-Core Data Pipelines for AI/ML

本プロジェクトは、[このプロジェクト](https://github.com/brunoNetId/sp-edge-to-cloud-data-pipelines-demo)のforkです。

本プロジェクトでは、AI/MLの実装における、エッジからコアへのデータ・パイプラインのソリューション・パターンを提供します。
具体的には、エッジ・デバイスが画像データを生成し、コア・データセンターやクラウドでAI/MLモデルのトレーニングを実行する前に、エッジでデータを収集・処理・保存する必要があるシナリオにおけるアーキテクチャ・ソリューションを提供します。

このソリューション・パターンには、トレーニング・データを取得し、新しいMLモデルをトレーニングし、それらをエッジへデプロイして提供し、クライアントが推論リクエストを送信するためのサービスを公開する、データの一連の動きを示すリソースが含まれています。

## Home page

このデモの全容を知るには、ソリューション・パターンのホームページをご覧ください。以下のリンクからご覧いただけます。

- [**Solution Pattern Home Page**](https://redhat-solution-patterns.github.io/solution-pattern-edge-to-cloud-pipelines/solution-pattern-edge-to-core-pipelines/index.html)


## Tested with

* RH OpenShift 4.12.12
* RHODF 4.12.11 provided by Red Hat
* RHOAI 2.8.0 provided by Red Hat
* RHO Pipelines 1.10.4 provided by Red Hat
* AMQ-Streams 2.6.0-1 provided by Red Hat
* AMQ Broker 7.11.6 provided by Red Hat
* Red Hat build of Apache Camel 4
* Camel K 1.10.6 provided by Red Hat
* RH Service Interconnect 1.4.4-rh-1 provided by Red Hat


## Deployment instructions

### 2. Provision an OpenShift environment

1. RHDSにて[**Solution Pattern - Edge to Core Data Pipelines for AI/ML**](https://demo.redhat.com/catalog?item=babylon-catalog-prod/community-content.com-edge-to-core.prod&utm_source=webapp&utm_medium=share-link)をデプロイする
2. RHDS にアクセスできない場合、OpenShift 環境が最低限利用可能であることを確認し、前提条件となる製品バージョンを満たして Red Hat OpenShift AI をインストールします (製品バージョンを調べるには、「_Tested with_」セクションを参照してください)。

<br/>

### 2. Deploy the Solution Pattern

以降の手順は、以下の環境が存在することを前提としています。

* ローカル環境に_Docker_、_Podman_、または`ansible-playbook`がインストールされている。
* RHDSを使用してOCPクラスタ（OCP 4.12 + RHOAI 2.8でテスト済み）をプロビジョニングし、bastionサーバが利用可能である。

<br/>


#### Install the demo

1. GitHubリポジトリをclone

    ```sh
    git clone https://github.com/brunoNetId/sp-edge-to-cloud-data-pipelines-demo.git
    ```

1. プロジェクトのrootディレクトリへcd

    ```sh
    cd sp-edge-to-cloud-data-pipelines-demo
    ```

    <br/>

2. _Docker_ または _Podman_ が動作している場合
    
    1. `KUBECONFIG` ファイルを構成する (OpenShiftクラスタへログイン後に kube-demo の詳細が設定されます)。

        ```sh
        export KUBECONFIG=./ansible/kube-demo
        ```

    2.OpenShift クラスタへログイン ( `oc` コマンド )

        ```sh
        oc login --username="admin" --server=https://(...):6443 --insecure-skip-tls-verify=true
        ```

        `--server` は、お使いのOpenShiftクラスタのAPIエンドポイントのURLへ置き換えてください。

    2. Ansible Playbookを実行

        1. Dockerを使う場合:
        
            ```sh
            docker run -i -t --rm --entrypoint /usr/local/bin/ansible-playbook \
            -v $PWD:/runner \
            -v $PWD/ansible/kube-demo:/home/runner/.kube/config \
            quay.io/agnosticd/ee-multicloud:v0.0.11  \
            ./ansible/install.yaml
            ```
        
        2. Podmanを使う場合:
        
            ```sh
            podman run -i -t --rm --entrypoint /usr/local/bin/ansible-playbook \
            -v $PWD:/runner \
            -v $PWD/ansible/kube-demo:/home/runner/.kube/config \
            quay.io/agnosticd/ee-multicloud:v0.0.11  \
            ./ansible/install.yaml
            ```
    <br/>

2'. Ansible Playbook（ローカルマシンにインストール済み）で実行する場合

    1. OpenShift クラスタへログイン ( `oc` コマンド )

        例: \
        ```sh
        oc login --username="admin" --server=https://(...):6443 --insecure-skip-tls-verify=true
        ```
        (`--server` は、お使いのOpenShiftクラスタのAPIエンドポイントのURLへ置き換えてください。)

    2. 以下のプロパティを設定:
        ```
        TARGET_HOST="lab-user@bastion.b9ck5.sandbox1880.opentlc.com"
        ```
    3. Ansible Playbookを実行
        ```sh
        podman run -i -t --rm --entrypoint /usr/local/bin/ansible-playbook \
        -v $PWD:/runner \
        -v $PWD/ansible/kube-demo:/home/runner/.kube/config \
        quay.io/agnosticd/ee-multicloud:v0.0.11  \
        -i $TARGET_HOST,ansible/inventory/openshift.yaml ./ansible/install.yaml
        ```

<br/>

### 3. エッジ環境を追加でデプロイする方法

2のインストールを行うと、デフォルトで以下のゾーンがデプロイされています。
 - `edge1`: 推論処理を行うエッジ環境
 - `central`: MLモデルを学習するコア・データセンター

<br>

本ソリューションパターンのアーキテクチャでは、下図のように、多くのエッジ環境をコア・データセンターに接続することができます：

![image](docs/images/01-full-architecture.png)

新しい _Edge_ 環境をデプロイするために、上記と同じコマンドを使用します。ただし、以下の環境変数パラメータを追加してください。
- `-e EDGE_NAME=[your-edge-name]`

例えば、以下のパラメータ定義を使用します。
```sh no-copy
... ./ansible/install.yaml -e EDGE_NAME=zone2
```
新しい namespace `edge-zone2` が作成されると、そこへ、全ての _Edge_ アプリケーションとインテグレーションがデプロイされます。 

<br/>

### 3. Undeploy the Solution Pattern

もしアンインストールしたい場合は、上記のインストールコマンドの末尾の`./install.yaml`を以下に差し替えてください。
 - `./uninstall.yaml`
