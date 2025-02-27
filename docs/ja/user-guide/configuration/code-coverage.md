---
layout: main
title: コードカバレッジ
category: User Guide
menu: menu_ja
toc:
    - title: コードカバレッジ
      url: "#コードカバレッジ"
    - title: SonarQube
      url: "#sonarqube"
    - title: セルフホスト型のSonarQubeを利用する
      url: "#セルフホスト型のsonarqubeを利用する"
      subitem: true
    - title: GitHub PR decoration
      url: "#github-pull-request-decoration"
      subitem: true
---
# コードカバレッジ

ビルド完了後、コードカバレッジ率がUI上に表示されるようになり、コードカバレッジデータがアップロードされます。

![Coverage in build detail page](../../../user-guide/assets/coverage.png)

現在、カバレッジ bookend では [SonarQube](https://github.com/screwdriver-cd/coverage-sonar) をサポートしています。どのカバレッジプラグインがサポートされているかは Screwdriver クラスタ管理者に確認してください。

## SonarQube

Sonar properties を `sonar-project.properties` ファイルや、screwdriver.yaml 内に `$SD_SONAR_OPTS` の環境変数として設定することができます。`sonar.sources` のプロパティは必須で、ソースのパスを指定します。

### sonar-project.properties

SonarQube を使用するために、`sonar-project.properties` ファイルをソースコードのルートに追加して、そこに設定を追加していきます。

以下は [Javascript example](https://github.com/screwdriver-cd-test/sonar-coverage-example-javascript) の `sonar-project.properties` の例です:
```
sonar.sources=index.js
sonar.javascript.lcov.reportPaths=artifacts/coverage/lcov.info
```

`reportPath` プロパティは使用する言語によって変わります。正しい指定の仕方は [SonarQube documentation](https://docs.sonarqube.org/latest/instance-administration/plugin-version-matrix) を確認してください。

### $SD_SONAR_OPTS

設定を `$SD_SONAR_OPTS` の環境変数で指定することも出来ます。

`screwdriver.yaml` の例:

```yaml
shared:
  environment:
    SD_SONAR_OPTS: '-Dsonar.sources=lib -Dsonar.javascript.lcov.reportPaths=artifacts/coverage/lcov.info'
jobs:
  main:
    requires: [~pr, ~commit]
    image: node:14
    steps:
      - install: npm install
      - test: npm test
```

#### 注意

- `sonar-project.properties` と `$SD_SONAR_OPTS` で同じプロパティを設定していた場合、`$SD_SONAR_OPTS` の設定が優先されます。
- Screwdriver は次のプロパティ(`sonar.host.url`, `sonar.login`, `sonar.projectKey`, `sonar.projectName`, `sonar.projectVersion`, `sonar.links.scm`, `sonar.links.ci`)を自動で設定します。**`sonar.sources` は自分で設定する必要があります。**
- プライベートパイプラインではデフォルトでカバレッジが送信されません。送信したい場合は、`SD_ALLOW_PRIVATE_COVERAGE_SEND`を`true`に設定してください。

### セルフホスト型のSonarQubeを利用する

環境変数 `$SD_SELF_SONAR_HOST` に Sonar ホストの URL を設定することで、Screwdriver クラスタに設定されているものではないホストにコードカバレッジをアップロードすることができます。  
`$SD_SELF_SONAR_HOST` を使用する場合、そのホストの admin の User Token を環境変数 `$SD_SELF_SONAR_ADMIN_TOKEN` に設定する必要があります。

`screwdriver.yaml` の例:

```yaml
jobs:
  main:
    requires: [~pr, ~commit]
    image: node:14
    steps:
      - install: npm install
      - test: npm test
  environment:
    SD_SELF_SONAR_HOST: 'http://YOUR_SONAR_URL'
  secrets:
    - SD_SELF_SONAR_ADMIN_TOKEN
```

#### 注意

- `SD_SELF_SONAR_HOST` にコードカバレッジをアップロードした場合、UI上でのコードカバレッジ率の表示は `N/A` となります。

#### 関連リンク
- [SonarQube properties](https://docs.sonarqube.org/latest/analysis/analysis-parameters)
- [Java example](https://github.com/screwdriver-cd-test/sonar-coverage-example-java)
- [Javascript example](https://github.com/screwdriver-cd-test/sonar-coverage-example-javascript)
- [Examples from the SonarQube website](https://github.com/SonarSource/sonar-scanning-examples)
- [SonarQube docs](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner/)
- [SonarQube environment variables](../environment-variables#カバレッジsonar)

### GitHub pull request decoration
ScrewdriverのクラスタがSonar Enterpriseを利用している場合、GitHubでのチェックに[Pull Request decoration](https://docs.sonarqube.org/latest/analyzing-source-code/pull-request-analysis/)を利用することができます。この機能が有効な場合、リポジトリにSonar PR Checks用のGitHub appを追加することで利用することが出来ます。サポート状況の詳細はScrewdriverクラスタ管理者に確認してください。
