# Calibre_configs
credit/reference pages for commands



<a href="https://manual.calibre-ebook.com/generated/en/calibredb.html" target="_blank">CalibreDB</a> 
<a href="https://manual.calibre-ebook.com/generated/en/calibre-server.html" target="_blank">Calibre-Server</a>

#anything involving viewing/modifying the calibredb (read uses the calibredb command) will power down the calibre-server until it is finished. 



#used in GUI, stops Calibre-server and opens  CalibreGUI
 
  
```
book_add()
        {
        sudo systemctl stop calibre-server
        /usr/bin/calibre
        sudo systemctl start calibre-server
        exit
        }
```

```
book_backup()
{
screen -S calibre.backup tar -caf /disks/c/Calibre_Libraries/calibre_backup.tar.gz /disks/c/Calibre_Libraries/*
}
```


#self-explanatory
  
#Keep in mind, the book ID field is required, but will be helpful for our next funciton

```
book_search()
{
sudo systemctl stop calibre-server
clear
echo -e "\n Books"
calibredb list -s $1 -f title --sort-by title --library-path=/disks/c/Calibre_Libraries/Books
echo -e "\n Comics"
calibredb list -s $1 -f title --sort-by title --library-path=/disks/c/Calibre_Libraries/Comics
echo -e "\n Manga"
calibredb list -s $1 -f title,series --sort-by title --library-path=/disks/c/Calibre_Libraries/Manga
echo -e "\n Tech"
calibredb list -s $1 -f title --sort-by title --library-path=/disks/c/Calibre_Libraries/Tech
sudo systemctl start calibre-server
}
```

#creates a temporary directory, exports files based on book ID and creates a downloadable link using 0x0
  
#due to filesize limitations, please exclude large files

#highly rec using a self-hosted instance of 0x0

#This is unique to having a password protected calibre server, but choosing to share individual books

```
book_upload()
{
cd || exit
sudo systemctl stop calibre-server
calibredb export  "$1"  --progress --to-dir ~/calibre_exports --replace-whitespace --dont-write-opf --template="{title}"
sudo systemctl start calibre-server
cd ~/calibre_exports || exit
for file in *; do mv "$file" $(echo "$file" | sed -e 's/[^A-Za-z0-9._-]/_/g'); done &
sleep 2
clear
for i in *.mobi; do echo "$i" ; curl -F"file=@$i" https://0x0.st 2> /dev/null ; done 
for i in *.cbz; do echo "$i" ; curl -F"file=@$i" https://0x0.st 2> /dev/null ; done
for i in *.epub; do echo "$i" ; curl -F"file=@$i" https://0x0.st 2> /dev/null ; done
for i in *.pdf; do echo "$i" ; curl -F"file=@$i" https://0x0.st 2> /dev/null ; done 
cd
rm -rf  ~/calibre_exports
}
```

```
book_update()
{
        sudo systemctl stop calibre-server
        #calibredb check_library --with-library /disks/c/Calibre_Libraries/*
        echo -e "Calibre will turn off while it copies data from the following folders.
        \n/disks/c/Calibre_Libraries/update_books => /disks/c/Calibre_Libraries/Books
        \n/disks/c/Calibre_Libraries/update_comics =>  /disks/c/Calibre_Libraries/Comics
        \n/disks/c/Calibre_Libraries/update_manga =>  /disks/c/Calibre_Libraries/Manga
        \n/disks/c/Calibre_Libraries/update_tech =>  /disks/c/Calibre_Libraries/Tech"
        calibredb embed_metadata all
        sudo systemctl stop calibre-server
        sudo xvfb-run calibredb add /disks/c/Calibre_Libraries/update_books --library-path /disks/c/Calibre_Libraries/Books
        sudo xvfb-run calibredb add /disks/c/Calibre_Libraries/update_comics --library-path /disks/c/Calibre_Libraries/Comics
        sudo xvfb-run calibredb add /disks/c/Calibre_Libraries/update_manga --library-path /disks/c/Calibre_Libraries/Manga
        sudo xvfb-run calibredb add /disks/c/Calibre_Libraries/update_tech --library-path /disks/c/Calibre_Libraries/Tech
        sudo systemctl start calibre-server
        source ~/.bashrc
 }
```

 
#stops calibre server, manages current users

```
book_user()
{
         sudo systemctl stop calibre-server.service
         calibre-server --userdb /disks/c/Calibre_Libraries/users.sqlite --manage-users
         sudo systemctl daemon-reload
         sudo systemctl start calibre-server.service
         sleep 5
         source .bashrc
}

```


#Calibre systemd module
#drop into 
/etc/systemd/system/calibre-server.service

```
[Unit]
Description=calibre content server
After=network.target

[Service]
Type=simple
User=pi
Group=pi
ExecStart=/usr/bin/calibre-server "/path/to/Calibre_Libraries/Manga" "/path/to/Calibre_Libraries/Tech"  "/path/to/Calibre_Libraries/Comics" "/path/to/Calibre_Libraries/Books" --access-log /home/YOUR_USER/calibre-access --max-log-size 5 --ajax-timeout 270 --timeout 270 --enable-auth --port "####" --userdb /path/to/Calibre_Libraries/users.sqlite --enable-auth

[Install]
WantedBy=multi-user.target
```
