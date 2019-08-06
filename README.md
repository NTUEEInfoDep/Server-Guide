# Server Guide
## 實體伺服器位置
- 電二136
- 聯絡人：陳彥凱先生
- 聯絡方式：不方便公開，請用 ntu mail 自行尋找

## 基本資料
- IP：140.112.18.177
- 域名：ntuee.org
- 作業系統：Arch Linux

## 一些有用的指令
| 指令       | 說明 |
|:---------:|:---:|
| ping      | 看你連不連的到某個host                      |
| ifconfig  | interface config，可以看到所有interface, IP, 或是up/down某個interface|
| ip link   |     |
| nano      |     |
| netctl    | Network manager, 可以enable/disable/start/|
| systemctl | 用來enable/start/stop/restart service     |
### 
## 曾經遇到的狀況及解法
### 伺服器斷電以後連不上 ssh
```bash
ip link #查看網路接口
```
目前ntuee網路設定的接口為 `enp1s0`.
```
cd /etc/netctl
```
在這個資料夾裡面有伺服器的對外網路設定。
```bash
nano ntuee
```
在終端機上會顯示下面資訊：
```bash
Description='A basic static ethernet connection'
Interface=enp1s0
Connection=ethernet
IP=static                     # 設定固定IP
Address=('140.112.18.177/24') # 伺服器的固定IP在140.112.18.177
Gateway='140.112.18.254'
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
1. 使用 `ifconfig`
```
ifconfig enp1s0 down
ifconfig enp1s0 up
```
2. 使用 `ip link`
```
ip link set enp1s0 down
ip link set enp1s0 up
```
重新啟動 ntuee 網路設定。
```
netctl restart ntuee
```
查看 ntuee 網路狀況。
```
netctl status ntuee
```