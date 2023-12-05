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

// TODO 動いたらまとめる

https://zenn.dev/kou_pg_0131/articles/gh-actions-oidc-aws

https://zenn.dev/kou_pg_0131/articles/gh-actions-ecr-push-image
