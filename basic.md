# Basic ssh commands
## Connecting to a server
```
ssh [user name]@[domain name or ip address] -p [port] 
```
For example
```
ssh 12345@ntuee.org -p 55555
```

### Downloading a file from server via `scp`
```
scp -P port [user]@[ip/domain]:[path to file] <download_dir>
```
For example
```
scp -P 55555 12345@ntuee.org:/files/test.txt ./Desktop
```