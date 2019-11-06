# Docker guide

## 大部分的專案都放在`/home/ntuee/production`
- 就算把自己加到`ntuee`group好像還是沒有x權限

## 為什麼要用Docker
- 分離環境，避免不同專案之間dependency衝突
- 方便架構網路，所有專案可以各自使用同一個port而不衝突

## 架構
伺服器上使用docker的架構如下
  - 能夠從外部看到的port只有nginx佔用的80/tcp, 443/tcp (http, https)
  - 其他專案的server各自跑在自己的container裡面
  - nginx的DNS可以找到其他專案的server
### Nginx
- 外面看到的所有DNS查詢都指向伺服器ip (140.112.18.177)
- 從`http://xxx.ntuee.org`連到伺服器後，會被轉送到`https://xxx.ntuee.org` （設定在`/home/ntuee/production/nginx/nginx.conf`）
- 接著 nginx 會轉送到各自 subdomain 的 server（`nginx.conf` include了`../conf.d/sites-enabled/*`）
- `sites-enabled`裡面的所有設定檔都是`../sites-available`裡面設定檔的symlink
### 設定檔
一種是serve static file 的 server，另一種是php/node.js開的server
  - #### static file
    這種時候不需要container，以make.ntuee.org為例：
    - `location /`定義了外面 GET: `https://make.ntuee.org` 的 Response
    - `root [directory]`代表client GET /的時候要回傳的檔案路徑
      - 放在`/home/ntuee/production/nginx/static`下
    - try_files 代表剛好是`/`的時候要回傳的那個檔案
  - #### server
    這時候就會用docker來跑server，以course.ntuee.org為例：
    - proxy_pass代表實際的request會被送到`http://course:8000`
    - 其他都是proxy的header
  - 所有的domain都列在certificate裡面(目前不是用wildcard certificate)

## Docker基礎
### Docker 專有名詞

| 名詞      | 解釋                                                        |
| --------- | ----------------------------------------------------------- |
| image     | 基本上就是程式的環境，所有library、程式都應該在image設定好  |
| container | 可以想像成用image在電腦裡又再開一台機器（雖然跟虛擬機不同） |
| service   | 基本上跟container差不多，多個service可以組合在一起          |

### Image

首先要先有Image，Image的來源有兩種：

1. Docker Hub

   - 存Image的地方，可以找到很多官方的Image

2. Dockerfile
  
   - 當你在Docker Hub上找不到想要的Image，你可以在別人的Image基礎上加上新的東西，這時候就可以寫Dockerfile

#### docker pull
從Docker Hub下載image到電腦裡
```bash
docker pull image名稱[:tag名稱]
```
建議pull的時候加上tag名稱（通常加上版本號會比較好），不加的話預設是latest，你還要自己去確認是哪個版本的
Docker Hub上可以看到Image有哪些tag
- latest不代表會自動更新成最新版，他只是一個名字，pull下來的時候是哪個版本就是哪個版本

#### Dockerfile

- `FROM`代表你想從哪個image開始加東西
- 可以用`RUN`來跑cmd，安裝想要的dependency
    例如：
        RUN apt-get update && apt-get install -y xxx
    或是：
        RUN cd /frontend && npm install
- [官方說明文件](https://docs.docker.com/engine/reference/builder/)裡面有更多可以用的指令
- 用`docker build`就可以從Dockerfile產生image了

#### container特性
1. container可以start, stop, restart，裡面的資料不會不見
2. 當container被recreate的時候，資料會被清空
3. 可以用volume讓主機檔案跟container連動，或是在container被刪除（或recreate）時保留資料
4. 只有expose的port會被外面看到
5. 有時候改設定之後要restart才會生效

#### docker 常用指令

| 指令             | 內容                                       |
| ---------------- | ------------------------------------------ |
| docker image     | ls, rm, prune(, inspect , build, pull)     |
| docker container | ls(, start, stop, restart, exec, inspect)  |
| docker network   | ls, rm, prune, create(, inspect)           |
| docker logs      | 可以看到container的output，--tail -f很好用 |
| docker ps        | 跟docker container ls一樣                  |
| docker cp        | container和主機之間複製檔案                |
| docker exec      | 跟docker container exec差不多              |

- 所有括號後的command都有對應的top-level指令

  ​	例如docker image inspect  docker inspect
- `docker system prune`很危險，使用前請注意有沒有想要保留的container是stopped狀態

## docker-compose

dockerfile通常都只有一個程式，如果我想用node build前端，python當後端怎麼辦？

1. 寫一個Dockerfile，產生專門的Image
   - 這樣很容易變成每個project都要寫一個dockerfile
2. 用兩個image產生node和python container，再手動用docker network連起來
   - 不太方便
3. 用docker-compose

docker-compose其實就是把上面第二個方法標準化，把如何組合多個container的設定寫在.yml檔裡(docker-compose.yml)，組合出一個完整的程式

### docker-compose.yml

docker-compose.yml所在資料夾名稱會是project名稱

基本上就是把所有個別的Dockerfile寫在`services:`底下，可以在`networks:`定義額外的interface

設定完之後用`docker-compose up -d`（在docker-compose.yml所在資料夾），可以發現`docker ps`多了好幾個container。（在version: "3"下，container命名是`project名稱_service名稱_序號`）

所有的service可以直接用service名稱當domain找到其他service，像nginx就可以用`http://course`看到service course所在的ip

- 要記得expose port給nginx看
- 這是因為所有同project的service預設會連到`projcet名稱_default`這個interface
- 如果要連接兩個docker-compose出來的東西，可以在`networks:`自己開一個bridge連

| 指令                      | 內容                                                         |
| ------------------------- | ------------------------------------------------------------ |
| docker-compose up         | 產生出container，如果已經存在就只重新產生跟設定有變的container |
| docker-compose restart    | 就是restart所有sevice，也可以只restart某個特定的service      |
| docker-compose start/stop | 跟start/stop所有serviec，也可以指start/stop特定的service     |

- docker-compose的指令都是打service，不是打他的container名稱
- docker exec那些才是打container名稱
- 最好在docker-compose.yml同目錄執行docker-compose

## 備註

- autocompletion:
- 可以用 zsh plugin (包含在 oh-my-zsh 裡面)
  - docker-compose
  - docker