# コンテナハンズオン環境

このプロジェクトは、コンテナ技術の要素技術を学習するためのEC2環境をAWS CloudFormationで構築します。
EC2インスタンスへの接続はSSH接続を使用します。

## スタックの展開方法

### 1. AWSクレデンシャルの設定
```bash
export AWS_ACCESS_KEY_ID=your_access_key
export AWS_SECRET_ACCESS_KEY=your_secret_key
export AWS_SESSION_TOKEN=your_session_token  # 必要に応じて
```

### 2. SSHキーペアの作成とインポート
```bash
cd container_handson

# 新しいSSHキーペアを生成
ssh-keygen -t rsa -b 2048 -f container-handson-key.pem -N ""

# 公開鍵をAWSにインポート
aws ec2 import-key-pair \
  --key-name "Container-Handson-keypair-new" \
  --public-key-material fileb://container-handson-key.pem.pub \
  --region ap-northeast-1

# 秘密鍵の権限を設定
chmod 600 container-handson-key.pem
```

### 3. CloudFormationスタックの作成
```bash
aws cloudformation create-stack \
  --stack-name container-handson-stack \
  --template-body file://ec2-stack.yaml \
  --capabilities CAPABILITY_NAMED_IAM \
  --region ap-northeast-1
```

### 4. 作成完了の確認
```bash
aws cloudformation wait stack-create-complete \
  --stack-name container-handson-stack \
  --region ap-northeast-1
```

### 5. 出力情報の確認
```bash
aws cloudformation describe-stacks \
  --stack-name container-handson-stack \
  --region ap-northeast-1 \
  --query 'Stacks[0].Outputs' \
  --output table
```

## EC2へのログイン方法

### SSH接続
スタック作成後、以下のコマンドでEC2インスタンスに接続できます：

```bash
ssh -i container-handson-key.pem ubuntu@<PUBLIC_IP>
```

**パブリックIPの確認方法**:
```bash
aws cloudformation describe-stacks \
  --stack-name container-handson-stack \
  --region ap-northeast-1 \
  --query 'Stacks[0].Outputs[?OutputKey==`InstancePublicIP`].OutputValue' \
  --output text
```

## スタックの削除方法

### 1. CloudFormationスタックの削除
```bash
aws cloudformation delete-stack \
  --stack-name container-handson-stack \
  --region ap-northeast-1
```

### 2. 削除完了の確認
```bash
aws cloudformation wait stack-delete-complete \
  --stack-name container-handson-stack \
  --region ap-northeast-1
```

## インストール済みパッケージ

この環境には以下のパッケージがプリインストールされています：

- **Docker**: コンテナランタイム（docker-ce, docker-compose-plugin, docker-buildx-plugin）
- **cgroup-tools**: cgroupの操作ツール
- **util-linux**: unshare, nsenterコマンド
- **iproute2**: ipコマンド
- **iptables**: ネットワーク制御
- **libcap**: Capability操作ライブラリ
- **cgdb**: デバッガー
- **build-essential**: ビルドツール

## 参考資料

- [コンテナ技術入門 - 仮想化との違いを知り、要素技術を触って学ぼう](https://en-ambi.com/itcontents/entry/2019/02/05/103000)
