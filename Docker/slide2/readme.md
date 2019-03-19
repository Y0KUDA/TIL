layout: true
class: middle

---
class: center,middle
# 引き篭もり開発環境構築
---

## SaaS怖い
* セキュリティ
* 仕様変更
* ライセンス変更
* 利用料
* 汚いコードを公開したくない

---

## 怖い例
* Github
* Travis CI
* Circle CI
* Slack

---

### 引きこもれる開発環境の条件
* OSSでSelf Hosting可能

---

### 今回導入したソフトウェア
* Gitリポジトリマネージャ
* CIツール

---

### Gitリポジトリマネージャ
* Gitlab
* Gogs
* Gitea

---

#### Gitlab
Community EditionはOSS  
Self Hosting可能

Pros

* 信頼の実績

Cons

* メモリ爆喰
* 起動が遅い

---
background-image: url(./gogs-lg2.png)
background-size: contain
#### Gogs
Go言語で開発されたGRM

Pros

* 軽い
* ロゴがキュート

Cons

* メンテナが一人

---

#### Gitea
GogsをフォークしたGRM

Pros

* 軽い
* メンテナは18人いるらしい

よさそうなのでGiteaを導入していく。

---

#### Gitea導入手順

---

1.docker-compose.yamlを作成
```yaml
version: '2'
services:
  web:
    image: gitea/gitea:1.3.2
    volumes:
      - ./data:/data
    ports:
      - "3000:3000"
      - "22:22"
    depends_on:
      - db
    restart: always
  db:
    image: mariadb:10
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=changeme
      - MYSQL_DATABASE=gitea
      - MYSQL_USER=gitea
      - MYSQL_PASSWORD=changeme
    volumes:
      - ./db/:/var/lib/mysql
```

---

2.コンテナ起動

```sh
 docker-compose up
```

---

#### CIツール
* Jenkins
* Drone.io
* Travis CI
* CircleCI

Jenkinsは複雑すぎる。  
巷で流行りのDrone.ioを採用してみる

---
```yaml
  drone-server:
    image: drone/drone:0.8
    ports:
      - 4000:8000
      - 9000
    links:
      - gitea
    volumes:
      - ./drone/drone:/var/lib/drone/
    restart: always
    environment:
      - DRONE_OPEN=true
      - DRONE_HOST=http://drone-server:4000
      - DRONE_GITEA=true
      - DRONE_GITEA_URL=http://gitea:3000
      - DRONE_GITEA_PRIVATE_MODE=true
      - DRONE_SECRET=drone_secret
      - DRONE_MAX_PROCS=3

  drone-agent:
    image: drone/agent:0.8
    command: agent
    restart: always
    depends_on:
      - drone-server
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - DRONE_SERVER=drone-server:9000
      - DRONE_SECRET=drone_secret

```

---
##### docker-compose.ymlをマージして連携
---

# 以下実演

---