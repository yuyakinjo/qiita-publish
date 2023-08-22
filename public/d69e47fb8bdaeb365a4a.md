---
title: AWS CDKでAuroraServeless V2がサポートされたので書き換えてみた
tags:
  - AWS
  - Aurora
  - CDK
private: false
updated_at: '2023-06-19T20:03:14+09:00'
id: d69e47fb8bdaeb365a4a
organization_url_name: null
slide: false
---

# 突然ですが`deprecated`していいですか

最近、CDKでデプロイしていたら、

```
[WARNING] aws-cdk-lib.aws_rds.DatabaseClusterProps#instanceProps is deprecated.
  - use writer and readers instead
  This API will be removed in the next major release.
```

と出てきたので、RDSの書き方変わるのかなーとChangeLogを漁りに行ったら、

https://github.com/aws/aws-cdk/blob/main/CHANGELOG.v2.md

直近、`2.82.0`でRDSの変更があって

`rds: support Aurora Serverless V2 instances`の文字が！！！待望！！


# Before

通常は下記のような、Auroraの書き方をすると

```typescript
   const rds = new DatabaseCluster(this, DatabaseCluster.name, {
      engine: DatabaseClusterEngine.auroraMysql({ version: AuroraMysqlEngineVersion.VER_3_03_0 }),
      instanceProps: {
        vpc,
        vpcSubnets: { subnetType: SubnetType.PRIVATE_WITH_EGRESS },
      },
      defaultDatabaseName,
      removalPolicy: RemovalPolicy.DESTROY,
    });
```

現在はインスタンスタイプが、`db.t3.medium`のリーダーインスタンス・ライターインスタンスが作成されます。

# After

今後はこうなるようです。

```typescript
    const rds = new DatabaseCluster(this, DatabaseCluster.name, {
      engine: DatabaseClusterEngine.auroraMysql({ version: AuroraMysqlEngineVersion.VER_3_03_0 }),
      vpc,
      vpcSubnets: { subnetType: SubnetType.PRIVATE_WITH_EGRESS },
      instanceUpdateBehaviour: InstanceUpdateBehaviour.ROLLING,
      writer: ClusterInstance.provisioned("WriterInstance", {
        instanceType: InstanceType.of(InstanceClass.T3, InstanceSize.MEDIUM),
        enablePerformanceInsights: true,
      }),
      readers: [
        ClusterInstance.provisioned("ReaderInstance", {
          instanceType: InstanceType.of(InstanceClass.T3, InstanceSize.MEDIUM),
          enablePerformanceInsights: true,
        }),
      ],
      defaultDatabaseName,
      removalPolicy: RemovalPolicy.DESTROY,
    });
```

## ポイント

- writer・readersでインスタンスを定義

## serveless v2用に書き換え

writerだけ`serverlessV2`に書き換えるとこうなりました。

```typescript
const rds = new DatabaseCluster(this, DatabaseCluster.name, {
      engine: DatabaseClusterEngine.auroraMysql({ version: AuroraMysqlEngineVersion.VER_3_03_0 }),
      vpc,
      vpcSubnets: { subnetType: SubnetType.PRIVATE_WITH_EGRESS },
      instanceUpdateBehaviour: InstanceUpdateBehaviour.ROLLING,
+     writer: ClusterInstance.serverlessV2("Writer", { scaleWithWriter: true, enablePerformanceInsights: true }),
      defaultDatabaseName,
      removalPolicy: RemovalPolicy.DESTROY,
    });
```

## servelessv2用に書き換えたのをデプロイしてみました。

まず、Beforeで書いたテンプレートをデプロイしておき
次に、servelessv2用に書き換えたのをデプロイしてみました。


### 1. servelessv2インスタンスが`リーダーインスタンス`で作られる

![rds-writer-phase-1-2023-06-19 19.03.58.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/183059/c3e2bd17-9a82-0607-4557-8c73c7019311.png)


### 2. servelessv2インスタンスが`リーダーインスタンス`→`ライターインスタンス`に昇格し、`リーダーインスタンス`が削除

![rds-writer-phase-2-2023-06-19 19.13.40.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/183059/bfe4094f-9ad4-4e37-991f-fc620519712b.png)

### 3. servelessv2インスタンスのみになった

![rds-writer-phase-3-2023-06-19 19.41.51.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/183059/7ec37a92-1d07-0199-8135-7f2304686916.png)

ローリングデプロイもバッチリでした。
