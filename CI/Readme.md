# Rust, CircleCI, CodeCov, tarpaulin, Slack 連携
勉強がてらRustプロジェクトのクラウドサービスの連携をしてみた。  
githubにpushすると自動でテストして結果をSlackに届けてくれる。  
CircleCI ver.2は各Jobがコンテナ内で実行されるので、Job間でファイルを共有するにはpersist_to_workspaceとattach_workspaceを使う必要があるという点に注意が必要。
ついでにtarpaulinを使って出したカバレッジをcodecovにアップロードするように設定。Rustの場合は、`xd009642/tarpaulin`コンテナを使って以下のスクリプトを実行する。
```bash
cargo tarpaulin -v --out Xml
bash <(curl -s https://codecov.io/bash)
```

最終的に作成したconfig.ymlは以下の通り
```yaml
version: 2
jobs:
  init:
    docker:
      - image: circleci/rust
    steps:
      - checkout
      - run:
          name: update rust version
          command: rustup --version > rustup-version
      - persist_to_workspace:
          root: .
          paths:
            - .

  test:
    docker:
      - image: circleci/rust
    steps:
      - attach_workspace:
          at: .
      - run:
          name: test
          command: cargo test

  build:
    docker:
      - image: circleci/rust
    steps:
      - attach_workspace:
          at: .
      - run:
          name: build
          command: cargo build

  coverage:
    docker:
      - image: xd009642/tarpaulin
    steps:
      - attach_workspace:
          at: .
      - run:
          name: execute tarpaulin
          command: cargo tarpaulin -v --out Xml
      - run:
          name: upload coverage to codecov
          command: bash <(curl -s https://codecov.io/bash)

workflows:
  version: 2
  all:
    jobs:
      - init
      - build:
          requires:
            - test
      - test:
          requires:
            - init
      - coverage:
          requires:
            - test
```

Gitea, Drone.io, OpenFaaS, RocketChatのフルオンプレミスもいつかやってみたい・・・

