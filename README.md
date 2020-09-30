# LazyPT

紀錄一下自己在 Penetration Testing 會用到的工具跟專案目錄結構。

對，我把 GitHub 當雲端硬碟在用，爽啦。

## EntryPoints.txt

甲方給的網站入口點清單，一行一個站點，需要手動製作。
廠商給什麼這裏就貼什麼，**請確保一定要有開頭協定、End Of File 不能有空行**。


預期格式列舉：
```
http://example.com
http://example.com/
http://example.com/dir
http://example.com/dir/
http://example.com/dir/Login.php
http://example.com/dir/Login.php/
http://example.com/dir/Login.php?action=xxx
http://12.34.56.78
http://12.34.56.78/
http://12.34.56.78/dir
http://12.34.56.78/dir/
http://12.34.56.78/dir/Login.php
http://12.34.56.78/dir/Login.php/
http://12.34.56.78/dir/Login.php?action=xxx
...
...
...
(Different protocol acceptable)
```

## Parse the EntryPoints.txt

```
# 行尾補上斜線(純域名、有get參數 不補斜線)
< 暫時想不到最佳解，待補 >
# 留下 Hostname
cut -d'/' -f3,3 EntryPoints.txt | tee EntryPoints_Host.txt
# 留下 Protocol 與 Hostname
cut -d'/' -f1,2,3 EntryPoints.txt | tee EntryPoints_Protocol.txt
# 留下 Hostname 與 Path
cut -d'/' -f3- EntryPoints.txt | tee EntryPoints_Path.txt
# 留下 Hostname，但把 Port 拿掉 (如果有 Port 的話)
cut -d'/' -f3,3 EntryPoints.txt | cut -d':' -f1 | tee EntryPoints_Host_NoPort.txt
```

## Parse the result of gobuster.sh

只留下 URL
```
cat ./AutoCommand/gobuster/result.txt | awk '{print $2}' | tee ./AutoCommand/gobuster/url.txt
```

砍掉 Response 302, 404 的 URL
```
cat ./AutoCommand/gobuster/result.txt | awk -F, '!/(Status: 302)/' | awk -F, '!/(Status: 404)/' | awk '{print $2}' | tee ./AutoCommand/gobuster/available_url.txt
```


## gobuster.sh

```
./AutoCommandScripts/gobuster.sh
```

要用 Kali 跑，因為會讀取 `/usr/share/wordlists/dirb/big.txt`。

```
gobuster dir --wildcard --verbose --expanded --threads 15 -w /usr/share/wordlists/dirb/big.txt -u <標的> -o ./AutoCommand/gobuster/<標的域名>.txt
```

結果將輸出在 `./AutoCommandResult/gobuster`。

## http_options.sh

```
./AutoCommandScripts/http_options.sh
```

```
curl -X OPTIONS -IL -m 30 <標的入口點>
```

結果將輸出在 `./AutoCommandResult/http_options`。


## nmap_portDiscover.sh

```
sudo ./AutoCommandScripts/nmap_portDiscover.sh
```

```
nmap -sV -Pn -T5 --top-ports 2048 <標的域名> -oN ./AutoCommand/nmap_portDiscover/<標的域名>.txt
```

結果將輸出在 `./AutoCommandResult/nmap_portDiscover`。

## nmap_ssl.sh

```
sudo ./AutoCommandScripts/nmap_ssl.sh
```

```
nmap -p 80,443 --script ssl-enum-ciphers,ssl-cert <標的域名> -oN ./AutoCommand/nmap_ssl/<標的域名>.txt
```

結果將輸出在 `./AutoCommandResult/nmap_ssl`。

## ResponseHeader.sh

```
./AutoCommandScripts/ResponseHeader.sh
```

```
curl -IL -m 30 <標的入口點> | tee ./AutoCommand/ResponseHeader/<標的域名>.txt
```

結果將輸出在 `./AutoCommandResult/ResponseHeader`。

## sslscan.sh

```
./AutoCommandScripts/sslscan.sh
```

```
sslscan --verbose --no-colour <標的域名> | tee ./AutoCommand/sslscan/<標的域名>.txt
```

結果將輸出在 `./AutoCommandResult/sslscan`。

## testssl_runner.sh

testssl 的專案資料夾要放在家目錄。

```
./AutoCommandScripts/testssl_runner.sh
```

```
~/testssl.sh/testssl.sh <標的域名> | tee ./AutoCommand/testssl/<標的域名>.txt
```

結果將輸出在 `./AutoCommandResult/testssl_runner`。