
# Installing go
**==In order to install go we can easily use this tool==** [https://github.com/Dewalt-arch/pimpmykali](https://github.com/Dewalt-arch/pimpmykali)

We run it and we choose `3 - Fix Golang`


## Finding subdomains with Assetfinder

**==We can download this tool from==** [https://github.com/tomnomnom/assetfinder](https://github.com/tomnomnom/assetfinder)

Or easily running:
```bash
go get -u github.com/tomnomnom/assetfinder
```


To use it we can do:
```bash
assetfinder --subs-only DOMAIN >> sub.txt
```

But in general is better to run, without `--subsonly` it and then filter the result.



So let's write a bash script:
```bash
#!/bin/bash

url = $1

#create url folder
if [ ! -d "$url" ]; then
	mkdir $url
fi

#create recon folder
if [ ! -d "$url/recon" ]; then
	mkdir $url/recon
fi

echo "[+] Getting subdomains with assetfinder..."

assetfinder $url >> $url/recon/assetfinder.txt

cat $url/recon/assetfinder.txt | grep $1 >> $url/recon/final_assets.txt

rm $url/recon/assetfinder.txt
```

This will give us a final txt file.

To run it we can do:
```bash
chmod +x script.sh

./script.sh
```


# Finding subdomains with Amass

**==We can download it from==** [https://github.com/owasp-amass/amass](https://github.com/owasp-amass/amass)
Just follow the part `From Source`:
``` bash
go install -v github.com/owasp-amass/amass/v4/...@master
```

To run it we can easily do:
```bash
amass enum -d DOMAIN
```

So update the script:
```bash
#!/bin/bash

url = $1

#create url folder
if [ ! -d "$url" ]; then
	mkdir $url
fi

#create recon folder
if [ ! -d "$url/recon" ]; then
	mkdir $url/recon
fi

echo "[+] Getting subdomains with assetfinder..."

assetfinder $url >> $url/recon/assetfinder.txt

cat $url/recon/assetfinder.txt | grep $1 >> $url/recon/final_assets.txt

rm $url/recon/assetfinder.txt

echo "[+] Getting subdomains with Amass..."

amass enum -d $url >> $url/recon/fin.txt

sort -u $url/recon/fin.txt >> $url/recon/final_assets.txt

rm $url/recon/fin.txt

```



# Finding active domains with Httprobe

We can download it from [https://github.com/tomnomnom/httprobe](https://github.com/tomnomnom/httprobe)

Easily in this way:
```bash
go install github.com/tomnomnom/httprobe@latest
```

At this point we use `final.txt` and do :
```bash
cat DOMAIN/recon/final.txt | httprobe -s -p https:443 | sed "s/https\?:\/\///" | tr -d ":443"
```
- this will execute the command and restore the file


Let's modify the script:
```bash
#!/bin/bash

url = $1

#create url folder
if [ ! -d "$url" ]; then
	mkdir $url
fi

#create recon folder
if [ ! -d "$url/recon" ]; then
	mkdir $url/recon
fi

echo "[+] Getting subdomains with assetfinder..."

assetfinder $url >> $url/recon/assetfinder.txt

cat $url/recon/assetfinder.txt | grep $1 >> $url/recon/final_assets.txt

rm $url/recon/assetfinder.txt

# echo "[+] Getting subdomains with Amass..."

# amass enum -d $url >> $url/recon/fin.txt

# sort -u $url/recon/fin.txt >> $url/recon/final_assets.txt

# rm $url/recon/fin.txt

echo "[+] testing for reachability"

cat $url/recon/final.txt | sort -u | httprobe -s -p https:443 | sed "s/https\?:\/\///" | tr -d ":443" >> alive.txt

```


So we have now alive subdomains.
We can filter on `dev` `test` `admin` to see if we reach something.



# Screenshots with GoWitness
We can download it from [https://github.com/sensepost/gowitness](https://github.com/sensepost/gowitness)

**==Before installing it we need to run first:==**
```bash
go get -u gorm.io/gorm
```

Then we can install it.

To perform screenshots we can use:
```bash
gowitness single https://DOMAIN
```

The result will be:
- ![[Pasted image 20241223143737.png]]



# Automating the enumeration process

We can download a script written by another pentester from [https://github.com/thatonetester/sumrecon](https://github.com/thatonetester/sumrecon)



A modified version is:
```bash
#!/bin/bash	
url=$1
if [ ! -d "$url" ];then
	mkdir $url
fi
if [ ! -d "$url/recon" ];then
	mkdir $url/recon
fi
#    if [ ! -d '$url/recon/eyewitness' ];then
#        mkdir $url/recon/eyewitness
#    fi
if [ ! -d "$url/recon/scans" ];then
	mkdir $url/recon/scans
fi
if [ ! -d "$url/recon/httprobe" ];then
	mkdir $url/recon/httprobe
fi
if [ ! -d "$url/recon/potential_takeovers" ];then
	mkdir $url/recon/potential_takeovers
fi
if [ ! -d "$url/recon/wayback" ];then
	mkdir $url/recon/wayback
fi
if [ ! -d "$url/recon/wayback/params" ];then
	mkdir $url/recon/wayback/params
fi
if [ ! -d "$url/recon/wayback/extensions" ];then
	mkdir $url/recon/wayback/extensions
fi
if [ ! -f "$url/recon/httprobe/alive.txt" ];then
	touch $url/recon/httprobe/alive.txt
fi
if [ ! -f "$url/recon/final.txt" ];then
	touch $url/recon/final.txt
fi

echo "[+] Harvesting subdomains with assetfinder..."
assetfinder $url >> $url/recon/assets.txt
cat $url/recon/assets.txt | grep $1 >> $url/recon/final.txt
rm $url/recon/assets.txt

#echo "[+] Double checking for subdomains with amass..."
#amass enum -d $url >> $url/recon/f.txt
#sort -u $url/recon/f.txt >> $url/recon/final.txt
#rm $url/recon/f.txt

echo "[+] Probing for alive domains..."
cat $url/recon/final.txt | sort -u | httprobe -s -p https:443 | sed 's/https\?:\/\///' | tr -d ':443' >> $url/recon/httprobe/a.txt
sort -u $url/recon/httprobe/a.txt > $url/recon/httprobe/alive.txt
rm $url/recon/httprobe/a.txt

echo "[+] Checking for possible subdomain takeover..."

if [ ! -f "$url/recon/potential_takeovers/potential_takeovers.txt" ];then
	touch $url/recon/potential_takeovers/potential_takeovers.txt
fi

subjack -w $url/recon/final.txt -t 100 -timeout 30 -ssl -c ~/go/src/github.com/haccer/subjack/fingerprints.json -v 3 -o $url/recon/potential_takeovers/potential_takeovers.txt

echo "[+] Scanning for open ports..."
nmap -iL $url/recon/httprobe/alive.txt -T4 -oA $url/recon/scans/scanned.txt

echo "[+] Scraping wayback data..."
cat $url/recon/final.txt | waybackurls >> $url/recon/wayback/wayback_output.txt
sort -u $url/recon/wayback/wayback_output.txt

echo "[+] Pulling and compiling all possible params found in wayback data..."
cat $url/recon/wayback/wayback_output.txt | grep '?*=' | cut -d '=' -f 1 | sort -u >> $url/recon/wayback/params/wayback_params.txt
for line in $(cat $url/recon/wayback/params/wayback_params.txt);do echo $line'=';done

echo "[+] Pulling and compiling js/php/aspx/jsp/json files from wayback output..."
for line in $(cat $url/recon/wayback/wayback_output.txt);do
	ext="${line##*.}"
	if [[ "$ext" == "js" ]]; then
		echo $line >> $url/recon/wayback/extensions/js1.txt
		sort -u $url/recon/wayback/extensions/js1.txt >> $url/recon/wayback/extensions/js.txt
	fi
	if [[ "$ext" == "html" ]];then
		echo $line >> $url/recon/wayback/extensions/jsp1.txt
		sort -u $url/recon/wayback/extensions/jsp1.txt >> $url/recon/wayback/extensions/jsp.txt
	fi
	if [[ "$ext" == "json" ]];then
		echo $line >> $url/recon/wayback/extensions/json1.txt
		sort -u $url/recon/wayback/extensions/json1.txt >> $url/recon/wayback/extensions/json.txt
	fi
	if [[ "$ext" == "php" ]];then
		echo $line >> $url/recon/wayback/extensions/php1.txt
		sort -u $url/recon/wayback/extensions/php1.txt >> $url/recon/wayback/extensions/php.txt
	fi
	if [[ "$ext" == "aspx" ]];then
		echo $line >> $url/recon/wayback/extensions/aspx1.txt
		sort -u $url/recon/wayback/extensions/aspx1.txt >> $url/recon/wayback/extensions/aspx.txt
	fi
done

rm $url/recon/wayback/extensions/js1.txt
rm $url/recon/wayback/extensions/jsp1.txt
rm $url/recon/wayback/extensions/json1.txt
rm $url/recon/wayback/extensions/php1.txt
rm $url/recon/wayback/extensions/aspx1.txt
#echo "[+] Running eyewitness against all compiled domains..."
#python3 EyeWitness/EyeWitness.py --web -f $url/recon/httprobe/alive.txt -d $url/recon/eyewitness --resolve
```
- `waybackurls` shows how the website was in the past


# New resources
There are two channel useful to learn more:
- [https://www.youtube.com/watch?v=uKWu6yhnhbQ](https://www.youtube.com/watch?v=uKWu6yhnhbQ)
- [https://www.youtube.com/watch?v=MIujSpuDtFY&list=PLKAaMVNxvLmAkqBkzFaOxqs3L66z2n8LA](https://www.youtube.com/watch?v=MIujSpuDtFY&list=PLKAaMVNxvLmAkqBkzFaOxqs3L66z2n8LA)

