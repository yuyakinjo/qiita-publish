---
title: CircleCIで何をデプロイしたかをわかるGitタグを作成
tags:
  - Bash
  - Git
  - CircleCI
  - SourceTree
  - gitkraken
private: false
updated_at: '2018-12-10T00:59:25+09:00'
id: 097cdcfaea8ea95a9e85
organization_url_name: ca-adv
slide: false
---
CircleCIの設定の話よりも、Gitタグ名の運用をメインに記していきます:santa:

CircleCIでGitタグをイベントにCIを回していく記事はありますが、じゃあ実際にどういうタグ名にするか
という記事は見かけなかった為、実際にタグを運用してみての話をしていきたいと思います:santa:



# 対象読者
- CircleCIを使用している
- CircleCIの環境内でshellを使う
- Gitタグにブランチ名を使用したいと思っている

# Gitタグ運用の背景
自分の場合、Gitタグを生成する目的は、Gitタグを使用してGitタグ生成時までロールバックするのが目的でした:santa:
なのでタグ名の詳細は特に細かいことを考えず、当初は下記のようなタグを生成していました:fish:

```bash

NOW=$(date +'%Y_%m%d'); # echo $NOW => "2018_0830"
TAG_NAME=${CIRCLE_BRANCH}_$NOW

# echo $TAG_NAME => master_2018_0830
```

その時は、いつなんのブランチのjobが実行完了したかを把握すればいいやという感覚で下記のようなタグ名にしていたのですが


```bash

master_2018_0830
```


- マージやデプロイされる度にタグ名がどんどん増えていく（当たり前）
- 何をいつデプロイしたのか忘れる→いつ時点までロールバックすれば良いのかコミット内容全て見返すハメに
- デプロイ待ちを可視化したい

ロールバックをするという事態は起きていないのですが、より意味のあるGitタグを生成したい思いタグ名を見直す事にしました:santa:


# CircleCIの流れ

まずは自分のCIの流れを簡単に紹介しておくと

### masterブランチへプルリク成功後

1. ビルド  
↓
2. テスト  
↓
3. Gitタグを生成

```bash

"master_2018_0830"　#こんな名称
```

### prod、devブランチへマージ成功後

1. デプロイ  
↓
2. Gitタグを生成

```bash

"prod_2018_0830"　#こんな名称
"dev_2018_0830"　#devの場合はこんな名称
```

そしてmaster→prod、stagingへマージするタイミングは人間の判断でしたい時とかあったので手動でmaster→prod、stgにマージしたタイミングです:santa:

### それ以外のブランチは、プッシュの度に

1. ビルド  
↓
2. テスト

が走ります

CicleCIのconfigの内容がメインではないので、config.yml設定内容は最後に置いておきました！

# 結果
ともかく、まずは結果から

## 変更前

```bash

# masterブランチにマージ成功後
master_2018_0830

# prodブランチにマージ成功後

prod_2018_0830

# stgブランチにマージ成功後

stg_2018_0830
```

## 上記のように運用してみてわかった弱点

- マージやデプロイされる度にタグ名がかさばっていく
- 何をいつデプロイしたのか忘れる
- デプロイされていないものがわからない

上記を解決するため下記のようにタグ名を変更しました:santa:

## 変更後

```bash

# masterブランチにマージ成功後
"master←←←issues/#100-hogehoge_2018_0928"  というGitタグを作成
"#100_prod_no_deploy"　も新たにタグ生成
"#100_stg_no_deploy"　も新たにタグ生成


# productionブランチにマージ成功後
prod←←←issues/#100_2018_0928

"#100_prod_no_deploy"　を削除


# stagingブランチにマージ成功後
stg←←←issues/#100_2018_0928

"#100_stg_no_deploy"　を削除

```

# 変更のポイント
1. " / "を入れると、GitHubのGUI上でフォルダ形式になってくれる
2. どういうissueがいつどのブランチにマージされたかがわかる
3. masterにプルリクがマージ後にno_deployを生成して何のissueがデプロイされていないかわかる

画像を使用して説明すると、[GitKraken](https://www.gitkraken.com/)や[SourceTree](https://ja.atlassian.com/software/sourcetree)などをGitのGUIを使用しているわかる機能ですが
<img width="394" alt="スクリーンショット 2018-12-09 23.27.28.png" src="https://qiita-image-store.s3.amazonaws.com/0/183059/f17a3575-3768-1e7e-56b5-8d28e1c77057.png">
上が"/"が入っていないタグで、どんどんリストが多くなっていきます
下がタグ名に"/"が入っているタグで、"/"から切れた状態でフォルダ化されます

タグ名をクリックして展開すると下記のようになります:santa:

<img width="392" alt="スクリーンショット 2018-12-09 23.27.36.png" src="https://qiita-image-store.s3.amazonaws.com/0/183059/3787e423-4356-65ce-c4a9-690f2ebdc85e.png">

# CircleCI上でマージしたissue名を抽出するには
さて、ここが結構苦悩したというか、自分の経験のなさ故にハマった場所なのですが、どうやったらCircleCIでマージされたブランチ名を取得しようか模索したのですが
結論からいうと```git log```コマンドで取得することがわかりました:santa:

ツイッターでどうやってマージされたブランチ名を取得するんじゃーって悩んでいたら、公式のCircleCIのアカウントの方からアドバイスをいただいたのでそれを参考にしました:santa:

この場を借りて感謝申し上げます:santa:
本当にありがとうございます！

下記がmasterブランチにマージ後、CircleCIのdockerで実行するシェルになります(わかりにくいですね💦)

```create-mastertag.sh
#!/bin/bash

# want to create git tag ex. 「 pr_name←←←issues/#100-hogehoge_2018_0928 」 and 「 #100_no_deploy 」
NOW=$(date +'%Y_%m%d');                                                    # echo $NOW => "2018_0830"
pr_repo_name=$( git log --oneline --merges | awk '{print $7}' | head -1 ); # echo $pr_repo_name => CyberAgent/issues/#100-hogehoge
pr_name=${pr_repo_name#*/};                                                # echo $pr_name => issues/#100-hogehoge
issue_no=$( echo $pr_name | (sed 's/.*\(#[0-9]\{1,100\}\).*/\1/') );       # echo $issue_no => #100
tag_name="master←←←${pr_name}_${NOW}";                                     # echo $tag_name => master←←←issues/#100-hogehoge_2018_0928
dev_no_deploy_tag="${issue_no}_dev_no_deploy";                             # echo $no_deploy_tag => #100_dev_no_deploy
prod_no_deploy_tag="${issue_no}_prod_no_deploy";                           # echo $no_deploy_tag => #100_prod_no_deploy

git tag $tag_name && git tag $dev_no_deploy_tag && git tag $prod_no_deploy_tag && git push origin --tags;
```

変数の中身のサンプルをコメント化しているのであらかた見当はつくと思いますが

マージされた直後のブランチ名を取得しているのは5行目の箇所です

```bash

pr_repo_name=$( git log --oneline --merges | awk '{print $7}' | head -1 ); # echo $pr_repo_name => CyberAgent/issues/#100-hogehoge
```


1. ```git log```コマンドでマージのみのlogを取得
2. ```awk```コマンドで空白で区切られた7番目のlogだけにする
3. ```head -1```でマージ直前のブランチ名のみのラインを取得


CircleCIの```${TAG_NAME}```を使用するだとか、いろんな方法を試しましたが、うまく行かず上記の取得方法に落ち着きました:santa:

その後、下記がprod(or dev)ブランチにマージ後、CircleCIのdockerで実行するシェルになります

```bash

#!/bin/bash

# create gittag ex. dev←←←issues/#100,#101,#102,_2018_0928
NOW=$(date +'%Y_%m%d')

# no_deployを含むタグの#numberだけを連結
for nodeploytag in $(git tag -l '*dev_no_deploy'); do                       # nodeploytag => #100_dev_no_deploy
  tagnumber=$( echo $nodeploytag | (sed 's/.*\(#[0-9]\{1,100\}\).*/\1/') ); # tagnumber => #100
  tagnumbers+="$tagnumber";                                                 # tagnumbers => #100,#101・・・
  git tag -d "${nodeploytag}";                                              # #100_dev_no_deployのタグをローカルで削除
  git push --delete origin "${nodeploytag}";                                # #100_dev_no_deployのタグを削除
done;

tag_name="dev←←←issues/${tagnumbers}_${NOW}";                               # echo $tag_name => dev←←←issues/#100,#101,#102,_2018_0928

git tag $tag_name && git push origin --tags
```

デプロイ待ちのブランチが複数あることを想定して、forでnodeploytagを連結していってます
devはprodをdevに置換するだけです。プロセスは全く同じです:santa:

# 最終的に
[GitKraken](https://www.gitkraken.com/)ではこういう風にタグが可視化されました（[SourceTree](https://ja.atlassian.com/software/sourcetree)も同じようになるはず）

![スクリーンショット 2018-12-10 0.37.15.png](https://qiita-image-store.s3.amazonaws.com/0/183059/e5cfdeaa-6931-23f2-f79c-9443e6fb65eb.png)

いつ何がmasterにマージされて、何がdevやprodにマージされていないのかが可視化され、かつタグがかさばらないようになりました:santa:
```*no_deproy```は本来あるべきではないので、"/"で区切らなくても良いと思いました:fish:


# 参考までにCircleCIの設定
```config.yml
version: 2.1

executors:
  container:
    working_directory: ~/hogehoge
    docker:
      - image: circleci/node:8.11-browsers
        environment:
          TZ: Asia/Tokyo
          TERM: xterm

jobs:
  install-and-build:
    executor:
      name: container
    steps:
      - checkout
      - restore_cache:
          name: Restore Yarn Package Cache
          key: dependency-cache-{{ checksum "yarn.lock" }}{{ .Branch }}
      - run: yarn install
      - run: yarn build:prod
      - save_cache:
          name: Save Yarn Package Cache
          key: dependency-cache-{{ checksum "yarn.lock" }}{{ .Branch }}
          paths:
            - ~/.cache
            - ./node_modules
      - persist_to_workspace:
          root: .
          paths:
            - .

  test:
    executor:
      name: container
    steps:
      - attach_workspace:
          at: .
      - run:
          command: yarn serve
          background: true
      - run:
          command: yarn cypress:install
      - run:
          command: yarn test:wait

  master-jobs:
    executor:
      name: container
    steps:
      - checkout
      - attach_workspace:
          at: .
      - run: ./scripts/create-mastertag.sh

  production-jobs:
    executor:
      name: container
    steps:
      - checkout
      - attach_workspace:
          at: .
      - run: ./scripts/deploy-prod.sh

  staging-jobs:
    executor:
      name: container
    steps:
      - checkout
      - attach_workspace:
          at: .
      - run: ./scripts/deploy-dev.sh

workflows:
  build-and-test:
    jobs:
      - install-and-build
      - test:
          requires:
            - install-and-build
          requires:
            - test
      - master-jobs:
          requires:
            - test
          filters:
            branches:
              only: master
      - production-jobs:
          filters:
            branches:
              only: production
      - staging-jobs:
          filters:
            branches:
              only: staging

```

