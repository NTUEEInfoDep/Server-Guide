# Server Guide
## 連上伺服器
```bash
ssh eeinfo@ntuee.org -p 22222
```
## 實體伺服器位置

- 電二 136
- 聯絡人：陳彥凱先生
- 聯絡方式：不方便公開，請用 ntu mail 自行尋找
- 伺服器長這樣（寫這篇文的時候是在機櫃後面）
  ![伺服器本人](./static/img/DSC_0958.JPG)

## 基本資料

- IP：140.112.18.177
- 域名：ntuee.org
- 作業系統：Arch Linux

## 一些有用的指令

|   指令    | 說明                                                                      |
| :-------: | :------------------------------------------------------------------------ |
|   ping    | 看你連不連的到某個 host                                                   |
| ifconfig  | interface config，可以看到所有 interface, IP, 或是 up/down 某個 interface |
|  ip link  | 管理 network device                                                       |
|   nano    | 機房的 terminal 怪怪的，用 vim 有時後進不去 Insert Mode                   |
|   less    | 機房的 terminal 不能往回看紀錄，用 less 很有用                            |
|  netctl   | Network manager, 可以 enable/disable/start 某個 profile                   |
| systemctl | 用來 enable/start/stop/restart service                                    |

## 曾經遇到的狀況及解法

### 伺服器斷電以後連不上 ssh

```bash
ip link #查看網路接口
```

目前 ntuee 網路設定的接口為 `enp1s0`.
(ethernet pci bus 1 slot 0，用 `lspci` 可以看到所有 pci device)

```bash
cd /etc/netctl # 移動到 netctl 的設定目錄，其他伺服器可能是用 NetworkManager 或是 dhcpcd
```

在這個資料夾裡面有伺服器的對外網路設定檔案（在我們的伺服器裡面叫 ntuee）。

```bash
nano ntuee
```

nano 的用法：

下面會看到`^O`, `^X`之類的，只要按下`Ctrl-O`, `Ctrl-X`就可以做對應的動作

存檔：`^O`之後按`Enter`就可以存檔了

在終端機上會顯示下面資訊：

```bash
Description='A basic static ethernet connection'
Interface=enp1s0              # 選擇哪個interface來用
Connection=ethernet
IP=static                     # 設定固定IP
Address=('140.112.18.177/24') # 伺服器的固定IP在140.112.18.177
Gateway='140.112.18.254'      # 大概是機房某一臺電腦
DNS=('8.8.8.8')               # Google 的 DNS server

## For IPv6 autoconfiguration
#IP6=stateless

## For IPv6 static address configuration
#IP6=static
#Address6=('1234:5678:9abc:def::1/64' '1234:3456::123/96')
#Routes6=('abcd::1234')
#Gateway6='1234:0:123::abcd'
```

先關掉網路接口再重啟。我們用 `ifconfig`成功，但兩種方法都可以試試（？）。
實際上`lspci`會看到兩個 Ethernet controller，後來我們把 `enp2s0` 關掉，打開 `enp1s0`

1. 使用 `ifconfig`

```bash
ifconfig enp1s0 down
ifconfig enp1s0 up
```

2. 使用 `ip link`

```bash
ip link set enp1s0 down
ip link set enp1s0 up
```

重新啟動 ntuee 網路設定 (network profile)。

```bash
netctl restart ntuee
```

（optional？worth a try）

```bash
netctl disable ntuee
netctl enable ntuee
```

查看 ntuee 網路狀況。

```bash
netctl status ntuee
```

正常運作的話在 Active 的欄位應該要顯示綠色的 *active*。

查看 IP routing table

```bash
route
```

```
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         _gateway        0.0.0.0         UG    0      0        0 enp1s0
140.112.18.0    0.0.0.0         255.255.255.0   U     0      0        0 enp1s0

應該要最下面兩行
```

### 重新設定網路

把 netctl 的預設 profile 複製來改

```bash
cd /etc/netctl
cp /examples/ethernet-static <new config file>
```

在 `new config file` 裡面設定 interface, dns, gateway 等資訊。

## Some References

- 伺服器沒有對外網路：
  - https://unix.stackexchange.com/questions/80493/arch-linux-connect-network-is-unreachable
- `ip` command
  - https://www.zybuluo.com/ghostfn1/note/120631

##
