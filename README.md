# ASP.NET Core 8.0 hosted on ECS

これは、ASP.NET Core 8.0 を Amazon ECS 上で動かすためのサンプルプロジェクトです。

## ローカルでの実行方法

```
dotnet run
または
dotnew watch
```

## Docker での実行方法

```
docker build -t aspnetapp:latest .
docker run -it --rm -p 5000:8080 aspnetapp:latest
```

https://github.com/dotnet/dotnet-docker/tree/main/samples/aspnetapp

## Github Actions での CI/CD

### 後続のステップに値を渡す

後続のステップに値を渡す場合は、以下のようにする

```
echo "image_id=${{ env.REGISTRY }}/${{ env.REPOSITORY }}:${{ env.IMAGE_TAG }}" >> $GITHUB_OUTPUT
```

下記記事にもあるように、`set-output` は非推奨になったので、`$GITHUB_OUTPUT` を使う

[GitHub Actions: Deprecating save-state and set-output commands](https://github.blog/changelog/2022-10-11-github-actions-deprecating-save-state-and-set-output-commands/)

### Github Actions 拡張機能について

拡張機能を使うと マーケットプレイスの関数を利用する際に、Repository のリンクが表示されるため、便利
Repository Secrets を利用する場合は、ログインを行うことで正しく表示されるようになる

### ECS へのデプロイについて

今回のケースでは Github Actions での Cluster, Service, TaskDefinition の作成は行なっていない。
あらかじめ AWS コンソール上で作成しておく必要がある。

### ecs-task-def.json への値の埋め込み

`ecs-task-def.json` を Git で管理する場合、次の点に注意する必要がある。

- AWS アカウントなどの機密情報の管理
- 動的に変化する値の管理

そのため、以下のようにして、`ecs-task-def.json` に値を埋め込むようにする。

```
sed -i 's|\[AWS_ACCOUNT_ID\]|${{ env.AWS_ACCOUNT_ID }}|g' ./deploy/ecs-task-def.json
sed -i 's|\[IMAGE_ID\]|${{ steps.build-image.outputs.image_id }}|g' ./deploy/ecs-task-def.json
```

### AWS の ロール

今回の Github Actions を実行させる場合、下記ポリシーをアタッチしたロールを作成する必要がある。
[AWS_ACCOUNT_ID] は、自身のアカウントの ID に置き換える。

```json
{
	"Version": "2012-10-17",
	"Statement": [
    {
			"Action": "ecr:GetAuthorizationToken",
			"Effect": "Allow",
			"Resource": "*"
		},
		{
			"Action": [
				"ecr:UploadLayerPart",
				"ecr:PutImage",
				"ecr:InitiateLayerUpload",
				"ecr:CompleteLayerUpload",
				"ecr:BatchCheckLayerAvailability"
			],
			"Effect": "Allow",
			"Resource": "arn:aws:ecr:ap-northeast-1:[AWS_ACCOUNT_ID]:repository/ecs-sample"
		}
		{
			"Sid": "VisualEditor0",
			"Effect": "Allow",
			"Action": [
				"ecs:DeregisterTaskDefinition",
				"ecs:RegisterTaskDefinition",
				"ecs:ListTaskDefinitions",
				"ecs:DescribeTaskDefinition",
				"ecs:DescribeServices",
				"ecs:UpdateService"
			],
			"Resource": "*"
		},
		{
			"Effect": "Allow",
			"Action": "iam:PassRole",
			"Resource": "arn:aws:iam::[AWS_ACCOUNT_ID]:role/ecsTaskExecutionRole"
		}
	]
}

```
