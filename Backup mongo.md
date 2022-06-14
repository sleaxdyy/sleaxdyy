### Tạo file bash backup mongo
### Thay đổi $1,2,3,4,5,6 note đã ghi 
```console
#!/bin/bash

# variables
backupPath="/srv/backup/mongo" # duong dan backup cho mongodb
database=$1 # ten database mongo can backup
mongodbUser=$2 # mongodb user has access to backup database
mongodbPassword=$3 # mat khau mongodb cho $mongodbUser
container=$4 # ten mongodb container
gdriveID=$5
dateFormat=$(date '+%Y-%m-%d')
interface="$6"
fileNum=$(ls $backupPath | wc -l)

# oldest dir of backupPath
oldestFile=$(find $backupPath -type f | sort | head -n 1)

if [[ $fileNum -ge 1 ]]; then
  rm -f $oldestFile
fi

# gen backup folder in container
docker exec -i $container /usr/bin/mongodump --username $mongodbUser --password $mongodbPassword --authenticationDatabase $database --db $database --out /dump

# copy backup folder to host
existPath=$backupPath/"$database"_"$dateFormat"
mkdir $existPath/
docker cp $container:/dump/$database $existPath
cd "$backupPath" && tar -czf "$database"_"$dateFormat.tar.gz" "$database"_"$dateFormat" && rm -rf "$existPath"

# gen info for telemessage
existFile=$backupPath/"$database"_"$dateFormat".tar.gz
size=$(ls -lhrt $existFile | awk '{print $5}')

ip=$(/usr/sbin/ifconfig $interface | grep -Eo 'inet (addr:)?([0-9]*\.){3}[0-9]*' | grep -Eo '([0-9]*\.){3}[0-9]*' | grep -v '127.0.0.1')

message="Backup Failed!!!"$'\n'"Time: $dateFormat"
# gen telemessage
if [[ -e "$existFile" ]]; then
  date="$(date +"%e %b %Y, %a %r")"
  sync="$(/usr/sbin/drive sync upload --keep-local --delete-extraneous --no-progress $backupPath/$database $gdriveID | tail -1)"
  message="Backup successfully!!!"$'\n'"Address: $ip"$'\n'"Type: mongodb"$'\n'"Database: $database"$'\n'"Path: $existFile"$'\n'"Size: $size"$'\n'"Time: $dateFormat"$'\n'"$sync"
fi

# send notify
sh /srv/bkscript/telegram-send.sh "$message"
```
### Chuẩn bị file bash gửi massage tới telegram
```console
vi /srv/bkscript/telegram-send.sh
```
### Thay đổi $1,2 theo note đã ghi
```console
#!/bin/bash
GROUP_ID="$1" #ID cua group da them chat bot telegram
BOT_TOKEN="$2" #Token cua chat bot telegram

# this 3 checks (if) are not necessary but should be convenient
if [ "$1" == "-h" ]; then
  echo "Usage: `basename $0` \"text message\""
  exit 0
fi

if [ -z "$1" ]
  then
    echo "Add message text as second arguments"
    exit 0
fi

if [ "$#" -ne 1 ]; then
    echo "You can pass only one argument. For string with spaces put it on quotes"
    exit 0
fi

curl -s --data "text=$1" --data "chat_id=$GROUP_ID" 'https://api.telegram.org/bot'$BOT_TOKEN'/sendMessage' > /dev/null
```