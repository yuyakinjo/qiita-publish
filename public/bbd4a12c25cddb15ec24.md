---
title: aws cdk v2 でクラスターモードオフなelasticacheを作成
tags:
  - TypeScript
  - CDK
private: false
updated_at: '2022-10-04T19:24:17+09:00'
id: bbd4a12c25cddb15ec24
organization_url_name: ca-adv
slide: false
---
# 環境

```bash
cdk --version
2.44.0 (build bf32cb1)
```

# 結論
aws cdk でelasticacheを作成するときに

- クラスターモード オフ
- マルチAZ オン

を満たす一例は下記です。

```cdk.ts
    const subnetGroup = new CfnSubnetGroup(this, 'redis-cluster-subnet-group', {
      cacheSubnetGroupName: 'redis-private-subnet',
      subnetIds: vpc.privateSubnets.map(({ subnetId }) => subnetId),
      description: 'private subnet',
    });

    const redis = new CfnReplicationGroup(this, 'redis-cluster', {
      engine: 'Redis',
      cacheNodeType: 'cache.t4g.micro',
      engineVersion: '6.2',
      replicasPerNodeGroup: 1,
      numNodeGroups: 1,
      replicationGroupDescription: 'cdk setup',
      cacheSubnetGroupName: subnetGroup.cacheSubnetGroupName,
      multiAzEnabled: true,
    });

    redis.addDependsOn(subnetGroup);
```

`numNodeGroups`が2以上になると、クラスターモードが`オン`になり、`プライマリエンドポイント`・`リーダーエンドポイント`が発行されませんでした。

上記のデプロイ後、管理画面でみるとこうなります↓
すでに削除して、リソースはないのですが、念の為、ARNとか黒塗りにしてます。

![スクリーンショット 2022-10-04 18.57.26.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/183059/e8e99bf4-0fff-adcc-3387-da0ab032a047.png)|
|:-:|

elasticacheのcdkってちょっちわかりにくいですよね。。。
