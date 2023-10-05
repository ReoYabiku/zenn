---
title: "そのIaC、本当に自動化できてる？ 〜terraform applyはもう叩かない〜"
emoji: "🐈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["cdktf", "githubactions", "iac", "typescript", "aws"]
published: false
publication_name: "ficilcom"
---

**Terraform CDK GitHub Action** の紹介記事です。IaC管理の手間を削減できる上、コピペで利用可能なコードが公開されているので紹介していきます。

https://github.com/hashicorp/terraform-cdk-action

## CDK for Terraform（cdktf）の基礎知識
通常のTerraformでは、**HCL(Hashicorp Configuration Language)** という言語を用いてインフラを記述します。

それに対してcdktfでは、TypescriptやGoといった言語でインフラを記述することができ、**使い慣れた言語でのIaC管理**が可能になります。

さらに、利用する可能性のあるリソースを事前に生成することができます。
そのため、importして利用する際に**型によるサポート**を受けることができます。
![](/images/type_support.png)


コード上で構築したインフラを環境に反映させるには、**deployコマンド**を利用します。
```shell
cdktf deploy
```

## 2回目以降のdeployが面倒
せっかくインフラをコードで管理しているのだから、deployも自動化してしまいたいものです。

**Terraform CDK GitHub Action**を利用することで、プルリクエストを作成する時にplanして、マージする時にdeploy(apply)できるようになります。