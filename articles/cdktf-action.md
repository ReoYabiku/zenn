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

**Terraform CDK GitHub Action**を利用することで、プルリクエストの作成時にplanして、マージ時にdeploy(apply)できるようになります。

以下に、手順を説明します。
## 準備
インフラの環境は、すでにcdktfで管理されているものとします。

cdktfに馴染みがなく、実際に手を動かしたい方は、**公式のチュートリアル**を参考にしてください。

https://developer.hashicorp.com/terraform/tutorials/cdktf/cdktf-build

### Terraform CloudからAPI Tokenを取得する
(トップページ) > User Settings > Tokensから、API Tokenを取得することができます。

CLIで`terraform login`を実行するときにもこのページでAPI Tokenを取得するので、すでに生成されたTokenがあるかもしれません。

このTokenを、`TF_API_TOKEN`としてリポジトリの環境変数に登録します。

![](/images/terraform_token.png)

### plan・deploy用のyamlファイルを作成する
**Terraform CDK Action** のREADMEに、Plan, Apply, Synthesize用のサンプルyamlファイルが用意されています。

例えばPlanのyamlファイルはこのようになっており、`stackName`を書き換えることでそのまま利用できます。

https://github.com/hashicorp/terraform-cdk-action

```yaml
name: "Comment a Plan on a PR"

on: [pull_request]

permissions:
  contents: read
  pull-requests: write

jobs:
  terraform:
    name: "Terraform CDK Diff"
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - uses: actions/setup-node@v3
        with:
          node-version: 18

      - name: Install dependencies
        run: yarn install

      - name: Generate module and provider bindings
        run: npx cdktf-cli get

      # Remove this step if you don't have any
      - name: Run unit tests
        run: yarn test

      - name: Run Terraform CDK
        uses: hashicorp/terraform-cdk-action@v0.1
        with:
          cdktfVersion: 0.17.0
          terraformVersion: 1.5.2
          mode: plan-only
          stackName: my-stack
          terraformCloudToken: ${{ secrets.TF_API_TOKEN }}
          githubToken: ${{ secrets.GITHUB_TOKEN }}
```

:::message
2023年10月現在、「`TF_API_TOKEN`を利用できない」というバグが報告されています。
yamlファイルをこのように書き換えることで、正常に動作します。
```yaml
      - name: Run Terraform CDK Plan
        uses: hashicorp/terraform-cdk-action@v0.1.0
        with:
          terraformVersion: 1.3.0
          cdktfVersion: 0.15.5
          stackName: your-stack
          mode: plan-only
          terraformCloudToken: ${{ secrets.TF_API_TOKEN }} # Useless
          githubToken: ${{ secrets.GITHUB_TOKEN }}
        env:
          TF_TOKEN_app_terraform_io: ${{ secrets.TF_API_TOKEN }}
```
:::

## 使ってみる
あとはPRを作成し、マージするだけです。

自動的にplanが実行され、マージするとその変更が実環境に反映されました。

![](/images/plan_and_apply.png)

## おわりに