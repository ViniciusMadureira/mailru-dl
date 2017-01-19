#!/bin/bash
# Author: Vinícius Madureira <contato@viniciusmadureira.com>
# Version: 0.1
# Date: January 17, 2017
# Script: Bash MailRu Downloader (mailru-dl)
# Dependencies: GNU bash 4.3.42(1); GNU coreutils 8.24; curl 7.43.0; GNU Wget 1.18; GNU grep 2.22; GNU sed 4.2.2
: '

 ███▄ ▄███▓ ▄▄▄      ▓█████▄ ▓█████     ▄▄▄▄ ▓██   ██▓    █     █░███▄    █  ███▄ ▄███▓
▓██▒▀█▀ ██▒▒████▄    ▒██▀ ██▌▓█   ▀    ▓█████▄▒██  ██▒   ▓█░ █ ░█░██ ▀█   █ ▓██▒▀█▀ ██▒
▓██    ▓██░▒██  ▀█▄  ░██   █▌▒███      ▒██▒ ▄██▒██ ██░   ▒█░ █ ░█▓██  ▀█ ██▒▓██    ▓██░
▒██    ▒██ ░██▄▄▄▄██ ░▓█▄   ▌▒▓█  ▄    ▒██░█▀  ░ ▐██▓░   ░█░ █ ░█▓██▒  ▐▌██▒▒██    ▒██ 
▒██▒   ░██▒ ▓█   ▓██▒░▒████▓ ░▒████▒   ░▓█  ▀█▓░ ██▒▓░   ░░██▒██▓▒██░   ▓██░▒██▒   ░██▒
░ ▒░   ░  ░ ▒▒   ▓▒█░ ▒▒▓  ▒ ░░ ▒░ ░   ░▒▓███▀▒ ██▒▒▒    ░ ▓░▒ ▒ ░ ▒░   ▒ ▒ ░ ▒░   ░  ░
░  ░      ░  ▒   ▒▒ ░ ░ ▒  ▒  ░ ░  ░   ▒░▒   ░▓██ ░▒░      ▒ ░ ░ ░ ░░   ░ ▒░░  ░      ░
░      ░     ░   ▒    ░ ░  ░    ░       ░    ░▒ ▒ ░░       ░   ░    ░   ░ ░ ░      ░   
       ░         ░  ░   ░       ░  ░    ░     ░ ░            ░            ░        ░   
                      ░                      ░░ ░                                      

'
function hideDownloadVerbose() {
  local flagBar=false caractere count carriegeReturn=$'\r' newLine=$'\n'
    while IFS='' read -d '' -r -n 1 caractere; do
        if $flagBar; then
            printf '%s' "$caractere"
        else
            if [[ $caractere != $carriegeReturn && $caractere != $newLine ]]; then
                count=0
            else
                ((count++))
                if [[ count -gt 1 ]]; then
                    flagBar=true
                fi
            fi
        fi
    done
}

function download() {
	page="http://"$(echo "$1" | grep -Po "(?i)my\..*")
	title="$2"
	jsonUrl="http://videoapi.my.mail.ru/videos/"$(echo "$page" | sed -e 's/http:\/\/my.mail.ru\///g' -e 's/\/video\//\//g' -e 's/\.html/\.json/g')
	jsonContent=$(curl --silent --cookie-jar cookies.txt  $jsonUrl)
	jsonVideos=$(echo $jsonContent | grep -Pio "\"videos.*]")
	IFS=' ' read -r -a resolutions <<< $(echo $jsonVideos | grep -Pio "\d+p")
	IFS=' ' read -r -a videos <<< $(echo $jsonVideos | grep -Pio "(?U)http.*region=(?-U)\d+")
	choice=$((${#resolutions} + 1))
	while [[ $choice -lt 1 || $((${#resolutions} - 1)) -le $choice || -z $choice ]]; do
		reset
		echo -e "Download MailRu\n""Title: $title\n""Choose a video resolution:"
		for index in ${!resolutions[@]}; do
			echo -e "["$((index + 1))"] ${resolutions[$index]}"
		done
		echo -e "Type a number between brackets like [1]:"
		read choice
	done
	echo -e "Downloading $title.mp4..."
	state=8
	while [[ !state -eq 0 ]]; do
		wget -c --progress=bar:force ${videos[$((choice - 1))]} --load-cookies="cookies.txt" -O "$title.mp4" 2>&1 | hideDownloadVerbose
		state=$(echo $?)
	done
	rm -rf "cookies.txt"
}

function resetVideos() {
	reset
	itemsTags=("$@")
	urls=()
	titles=()
	durations=()
	text=""
	for index in "${!itemsTags[@]}"; do
		urls[$index]=$(echo -e "${itemsTags[$index]}" | grep -Po "http.*.html")
		titles[$index]=$(echo -e "${itemsTags[$index]}" | grep -Po "(?U)title.*<" | sed 's/.*>\|<.*//g')
		durations[$index]=$(echo -e "${itemsTags[$index]}" | grep -Po "(?U)duration.*<" | sed 's/.*>\|<.*//g')
		text="$text\e[96m"$(($index + 1))"###\e[92m${titles[$index]}###\e[93m ${durations[$index]}\n"
	done
}

function printVideos() {
	if [ -n "$1" ]; then
		echo -e " \e[1m\e[34m\bIndex###    Title### Duration\e[21m\n$text" | column --table --separator "###"
		tput sgr0
	fi
}

baseUrl="http://m.my.mail.ru/video/search"
offset=0
urls=()
titles=()
durations=()
text=""
while [ true ]; do
	htmlContent=$(curl -s --data "st=search&q=$1&sameFromAjax=1&offset=$offset" "$baseUrl" -H 'Accept-Language:en-US,en;q=0.8,pt;q=0.6')
	if [[ $offset -eq 0 && -z "$htmlContent" ]]; then
		echo -e "No video found to \"$1\"."
		exit 0
	fi
	items=$(echo -e "$htmlContent" | grep -Poz "(?U)<li class=\"list-item \">.*<\/li>" | sed -e 's/\$/ /g' -e 's/\|#[0-9]\+//g')
	IFS=$ read -r -a itemsTags <<< $(echo -e "$items" | sed 's/li>/li>$/g')
	resetVideos "${itemsTags[@]}"
	printVideos "$text"
	limit=${#itemsTags[@]}
	if [ $limit -gt 0 ]; then
		key=-1
		while [[ $key -lt 1 || $key -gt $limit ]]; do
			echo -e "Type a index for download a video or press <Enter> to continue searching.\nTo quit press <q>."
			read key
			if [[ "$key" == "" ]]; then
				offset=$(echo -e "$htmlContent" | grep -Po "\[nextpage=\d+\]" | sed -e 's/[^0-9]//g')
				break
			elif [[ "$key" =~ ^[Qq]$ ]]; then
				exit 0
			elif [[ "$key" =~ ^[0-9]+$ && $key -ge 1 && $key -le $limit ]]; then
				index=$(($key - 1))
				download "${urls[$index]}" "${titles[$index]}" "${durations[$index]}"
				exit 0
			else
				key=-1
			fi
		done	
	else
		echo -e "The list has reached its end and will be restarted..."
		sleep 2
		offset=0			
	fi
done
exit 0
