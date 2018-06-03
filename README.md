# これは何？
- Packerを使ったEC2（RHRL7）の構築テンプレートです。

## 何ができるの？
- Cloudinitの設定ファイルを置き換えます。
	- ロケールの変更
	- タイムゾーンの変更
	- 詳細は`cloud-init_cfg/jp.cfg`を参照
- 指定した設定ファイルを含めた、AMIのカスタムイメージが作成されます。
- 実行AWSアカウント内にAMIが作成され、作成された指定した別AWSアカウントに（プライベートイメージとして）共有されます
- EBSは、20GBのルートデバイスと、20GBの追加デバイスが作成されます。

```
"launch_block_device_mappings": [
	{
		"volume_type": "gp2",
        "device_name": "/dev/sda1",
        "volume_size": 20,
        "delete_on_termination": "true"
    }
],
"ami_block_device_mappings": [
	{
    	"volume_type": "gp2",
        "device_name": "/dev/sdb",
        "volume_size": 20,
        "delete_on_termination": "true"
    }
]
```

# Packerの仕組み(概要)
`Template`と呼ばれるテンプレートファイルをJSON形式で定義します。  
AWSの場合、`Template`を元に、EC2インスタンスが立ち上がり、定義されたコマンド・操作が実行されます。  
EC2インスタンスを元にAMIが作成され、起動されたEC2は削除されます。  
EC2起動時に利用されるキーペアも自動的に作成され、AMI削除後には自動的に削除されます。  
Packerを利用する事で、上記の一連の流れを自動化できます。


# 使い方
## 事前準備
- Packerのインストール
	- [こちら](https://www.packer.io/docs/install/index.html)を参照 

- Homebrewの場合
	- `$ brew install packer`

## パラメータの指定
- 利用環境に合わせて`setup.json`の`variables`セクションの各値を書き換えます。

```
"variables": {
	"name": "rhel_no_ami_dayo",
	"region": "ap-northeast-1",
	"ami-id": "ami-XXXXX",
	"sg-id": "sg-XXXXX",
	"vpc-id":"vpc-XXXXX",
	"subnet-id":"subnet-XXXXX",
	"cloud-init_cfg": "jp.cfg",
	"shared-aws-account": "XXXXXXXXXXX"
}
```

- name
	- 作成されるAMI名です。
- region
	- 作成するAWSリージョン
- ami-id
	- 元となるAMI-ID
- sg-id
	- AMIの元となるEC2が起動時に利用するセキュリティグループ 
- vac-id
	- AMIの元となるEC2が起動時に配置されるVPC
- subnet-id
	- AMIの元となるEC2が起動時に配置されるサブネット
- cloud-init_cfg
	- `cloud-init_cfg/`以下にあるCloudinit用ファイル
- shared-aws-account
	- AMI作成後に共有するAWSアカウント


##　実行

- バリデーションチェック
	- `$ packer validate setup.json`
- 実行
	- `$ AWS_PROFILE=XXXXXX packer build setup.json`
	- AWS CLIにて、アクセスキー/シークレットキーが設定されている事を想定