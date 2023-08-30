---
title: VSCode CLI インストール後 code コマンドを叩くとエラー
tags:
  - 'vscode'
  - 'vscode tunnel'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
---

# VSCode CLI インストール後 code コマンドを叩くとエラー

```sh
cannot execute binary file: Exec format error
```

# 背景

ECS Fargate（ARM64 のアーキテクチャー）のコンテナ内で、[vscode](https://code.visualstudio.com/) を使用したい。

↓

アーキテクチャーに合っていない、不適切な`vscode cli`のファイルをダウンロードしていた。

# 解決策

## 適切な platform に合わせたファイルをダウンロードする

ARM64 の場合は下記に書き換えると OK。

```sh
curl -Lk 'https://code.visualstudio.com/sha/download?build=stable&os=cli-alpine-x64' --output vscode_cli.tar.gz
```

↓

```sh
curl -Lk 'https://code.visualstudio.com/sha/download?build=stable&os=cli-alpine-arm64' --output vscode_cli.tar.gz
```

# 参考

- [vscode tunnel](https://code.visualstudio.com/docs/remote/tunnels)
- [vscode download](https://code.visualstudio.com/download#)
