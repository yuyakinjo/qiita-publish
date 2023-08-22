---
title: AngularJSでimgタグが壊れた時、ngOnを使う
tags:
  - AngularJS1.x
private: false
updated_at: '2020-03-04T11:33:43+09:00'
id: 0ceccfa3d2c6cadc946b
organization_url_name: ca-adv
slide: false
---
# TL;DR

- imgタグが壊れた時のエラーハンドルを[ngOn](https://code.angularjs.org/snapshot-stable/docs/api/ng/directive/ngOn)を使用する

# 環境

- AngularJS 1.7.9(ドキュメント見ると、1.7以上から)

# imgタグが壊れた時、ngOnなしの場合

userIdを渡せば、ユーザー画像を表示するようなコンポーネントがあるとします

例えば、アカウント削除でuserIdが返ってこないとか、オフラインとか何らかの理由でリクエストが失敗すると
このような代替画像を表示させるはずです
↓
![nophoto.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/183059/83c65dd2-cff7-ad20-6925-604d2dbc6b88.png)


imgタグのsrcにセットした画像が取得できない場合、
・テンプレート側で壊れた画像を補う
・CSS側で補う
・JSで補う

など、特にベストプラクティスとかありそうもないので、特段の理由なくコントローラーでエラーハンドルしてました

html側でimgタグの`onerror`も試したりしましたが、
答えが出なかったので、パッと見て何したいかかわかるコードで対応してました・・・

(new imageしてインスタンス作るまでするか？とか葛藤はありましたが)

# しかし問題点

AngularのライフサイクルかJSにうまく乗れておらず、下記のような↓一瞬壊れた画像が表示されたあとで、代替画像が表示されていました、一瞬だったので目をつむっていました・・・

![スクリーンショット 2020-01-24 16.16.35.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/183059/d434b0e6-6fcd-0cd5-71e1-1089bc2bb1c2.png)


# [ngOn](https://docs.angularjs.org/api/ng/directive/ngOn)を使って克服
AngularJS1.7以上から実装された[ngOn](https://docs.angularjs.org/api/ng/directive/ngOn)で下記のように変更しました:santa:

```user-image.pug
img.user-image(ng-on-error="ctrl.onError()" ng-src="{{ ctrl.userImageUrl }}" alt="ユーザー画像")
```

```user-image.component.ts
const noPhoto = require('_Images/no-photo.jpg');

class UserImageComponent {
  private userId: string; //bindings
  private userImageUrl: string; //view bind

  $onChanges = changes => {
    if (!changes.userId.currentValue) return;
    const url = `https://hoge/thumbnail/${this.userId}.jpg`;
    this.userImageUrl = url;
  };

  onError = () => {
    this.userImageUrl = noPhoto;
  };
}

export const userImage = {
  template: require('./user-image.pug'),
  controllerAs: 'ctrl',
  controller: UserImageComponent,
  bindings: {
    userId: '<',
  },
};
```

## ポイント

1. テンプレート側で`ng-on-error`をカスタム属性で付与し、imgタグが壊れた時に`onError`が発動するように変更
2. onErrorで代替画像に差し替える

だけ:santa:

これで圧倒的に直感的になり、画像も壊れたのが表示されなくなりました:santa:


Angularにも同様のやつあるんですかね？:santa:

# 参考

[ngOn](https://docs.angularjs.org/api/ng/directive/ngOn)

# あとがき

`$onInit`時に、userIdがundefindで渡ってくるときがあるので、`$onChanges`を使っています:santa:
