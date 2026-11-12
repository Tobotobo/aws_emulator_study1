# aws_emulator_study1

## 概要

* 公式の AWS ローカルエミュレーターが出たので試してみる

AWS Emulator  
https://github.com/aws/aws-emulator  
AWS Emulator は、AWS API のローカルエミュレータです。AWS を呼び出すことなく、自分のマシン、開発サーバー、または継続的インテグレーション (CI) 環境でソフトウェアをテストするために使用できます。AWS アカウントは不要です。

## 詳細

Getting started with Docker  
https://docs.aws.amazon.com/emulator/latest/userguide/getting-started-docker.html  


Dockerfile
```Dockerfile
FROM public.ecr.aws/amazonlinux/amazonlinux:2023-minimal
ADD https://downloads.aws.com/aws-emulator/packages/aws-emulator/LTS/latest/aws-emulator-x86_64.rpm /tmp/aws-emulator.rpm
RUN rpm -i /tmp/aws-emulator.rpm && rm /tmp/aws-emulator.rpm \
    && mkdir -p /home/emulator && chown 1000 /home/emulator
USER 1000
ENV HOME=/home/emulator
EXPOSE 9297
ENTRYPOINT ["aws-emulator", "-b", "0.0.0.0"]
```

```sh
wslc build -t aws-emulator .
```

```sh
wslc network create aws-emulator-network

wslc run -d `
  --name aws-emulator `
  -p "127.0.0.1:9297:9297" `
  -v ".\emulator:/home/emulator/.aws/emulator" `
  --network aws-emulator-network `
  aws-emulator -p 9297 --telemetry off
```

以下で ok が返るまで待つ
```sh
curl.exe -sf --cacert ./emulator/9297/ca.pem https://127.0.0.1:9297/__admin/health
```

StackPort  
https://stackport.cloud/  
ローカルエミュレーターと実際のAWSアカウントに対応した、汎用的なAWSリソースブラウザ。  
S3、DynamoDB、Lambda、SQS、IAM、EC2、CloudWatch Logs、Secrets Manager専用のUIを使用して、35種類のAWSサービスにわたるリソースを閲覧、検査、管理できます。

https://hub.docker.com/r/davireis/stackport/tags

```sh
wslc run -d `
  --name stackport `
  --network aws-emulator-network `
  -p "127.0.0.1:8080:8080" `
  -v "./emulator:/root/.aws/emulator" `
  -e AWS_ENDPOINT_URL=https://aws-emulator:9297 `
  -e AWS_SHARED_CREDENTIALS_FILE=/root/.aws/emulator/9297/emulator_credentials `
  -e AWS_CONFIG_FILE=/root/.aws/emulator/9297/config `
  -e AWS_CA_BUNDLE=/root/.aws/emulator/9297/ca.pem `
  -e AWS_EC2_METADATA_DISABLED=true `
  -e AWS_DISABLE_HOST_PREFIX_INJECTION=true `
  -e AWS_REGION=us-east-1 `
  -e AWS_DEFAULT_REGION=us-east-1 `
  davireis/stackport:0.4.6
```

```sh
wslc run -d `
  --name stackport `
  --network aws-emulator-network `
  -p "127.0.0.1:8080:8080" `
  -v "./emulator:/root/.aws/emulator" `
  -e AWS_ENDPOINT_URL=https://aws-emulator:9297 `
  -e AWS_SHARED_CREDENTIALS_FILE=/root/.aws/emulator/9297/emulator_credentials `
  -e AWS_CA_BUNDLE=/root/.aws/emulator/9297/ca.pem `
  -e AWS_EC2_METADATA_DISABLED=true `
  -e AWS_DISABLE_HOST_PREFIX_INJECTION=true `
  -e AWS_REGION=us-east-1 `
  -e AWS_DEFAULT_REGION=us-east-1 `
  davireis/stackport:0.4.6
```

http://localhost:8080

```sh
wslc run -d `
  --name ubuntu `
  --network aws-emulator-network `
  -v "./emulator:/root/.aws/emulator" `
  ubuntu:26.04 `
  sleep infinity
```

```sh
wslc exec -it ubuntu bash
apt update
apt install -y curl iputils-ping
curl -sf --cacert /root/.aws/emulator/9297/ca.pem https://aws-emulator:9297/__admin/health
curl -v --cacert /root/.aws/emulator/9297/ca.pem https://aws-emulator:9297/__admin/health
```

```sh
wslc rm -f ubuntu
```
