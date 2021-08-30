Personal Calibre_configs


<a href="#bashrcportable-functions" 
target="_blank">Bash/Portable Functions</a>

<a href="#gui" 
target="_blank">GUI</a>


<a href="#setup-scripts" 
target="_blank">Setup Scripts</a>

<a href="#setup-scripts" 
target="_blank">SystemD Module</a>


<a href="https://manual.calibre-ebook.com/generated/en/calibredb.html" target="_blank">CalibreDB</a> 
<a href="https://manual.calibre-ebook.com/generated/en/calibre-server.html" target="_blank">Calibre-Server</a>

#anything involving viewing/modifying the calibredb (read uses the calibredb command) will power down the calibre-server until it is finished. 



<h2>BASHRC/Portable Functions</h2>

```
book_backup()
{
screen -S calibre.backup tar -caf /disks/c/Calibre_Libraries/calibre_backup.tar.gz /disks/c/Calibre_Libraries/*
}
```

```
book_list_manga()
{
        clear
        sudo systemctl stop calibre-server
        calibredb list -f series --with-library /disks/c/Calibre_Libraries/Manga | awk '{print $2,$3,$4,$5,$6}'  | sort | uniq
        sudo systemctl start calibre-server
}
```

```
book_list_tech()
{
        clear
        sudo systemctl stop calibre-server
        calibredb list -f title --with-library /disks/c/Calibre_Libraries/Tech | awk '{print $2,$3,$4,$5,$6}'  | sort | uniq
        sudo systemctl start calibre-server
}
```


A script to add multiple books via the CLI, asks for author/series.
Used for Manga library at the moment, will implement a 'list' of existing  libraries.
Have yet to add in covers/comments
```
book_meta_add()
{
  clear
  GREEN='\033[0;32m'
  NC='\033[0m' # No Color
  echo -e "${GREEN}Run this in the folder containing the books you want to add. \n If you're not in it, press CTRL-C to exit. You're currently in \n $(pwd)${NC}"
  read -rp "What library is this going into?  Manga Tech Books Comics" library
  read -rp "What is the author?" author
  read -rp "What is the series this book belongs to?" series
  sudo systemctl stop calibre-server
  calibredb add *.{epub,azw3,azw4,book,cbc,cbr,cbz,chm,djv,djvu,doc,docx,fb2,htm,html,htmlz,iba,lit,lrf,lrs,lrx,markdown,ncx,opf,oxps,pdb,pdf,pdr,pml,pobi,prc,rar,rtf,snb,tcr,txt,xhtml,xps,zip} -a "$author" -s "$series" --library-path /disks/c/Calibre_Libraries/$library
  sudo systemctl start calibre-server
  echo "Don't forget to add covers/comments in the GUI when needed"
}
```



<a href="https://manual.calibre-ebook.com/generated/en/calibre-server.html#cmdoption-calibre-server-manage-users" target="_blank">Searches Books</a> 

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


<a href="https://manual.calibre-ebook.com/generated/en/calibredb.html#adding-from-folders" target="_blank">Docs</a> 

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


#creates a temporary directory, exports files based on book ID and creates a downloadable link using http://0x0.st/ (please adhere to the rules involving CopyRight)
  
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

Manages current <a href="https://manual.calibre-ebook.com/generated/en/calibre-server.html#cmdoption-calibre-server-manage-users" target="_blank">users</a> 

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



<h2>GUI</h2>

used in GUI terminal, stops Calibre-server and opens  CalibreGUI
 
  
```
book_add()
        {
        sudo systemctl stop calibre-server
        /usr/bin/calibre
        sudo systemctl start calibre-server
        exit
        }
```






<h2>Setup Script(s)</h2>

#manually build python unrardll
```
(
sudo apt update
sudo apt install -y build-essential python-pip wget
mkdir unrarsrc
cd unrarsrc
wget https://rarlab.com/rar/unrarsrc-5.6.8.tar.gz
tar -xvf unrarsrc-5.6.8.tar.gz
make -C unrar lib
sudo make -C unrar install-lib
sudo pip install --global-option=build_ext --global-option="-I$(pwd)" unrardll
)

```

<h2>SystemD Module</h2>

/etc/systemd/system/calibre-server.service

I use this to tell Calibre where my different libraries are. While this isn't required, it helps keeps different types of libraries sorted/organized. 


See the official docs for more info

<a href="https://manual.calibre-ebook.com/server.html#creating-a-service-for-the-calibre-server-on-a-modern-linux-system" target="_blank">docs</a> 

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
