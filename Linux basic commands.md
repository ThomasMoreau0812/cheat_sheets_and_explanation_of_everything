# 🐧 Ubuntu 26 Command Cheat Sheet

Een praktische cheat sheet met de belangrijkste Ubuntu/Linux commands.

> 💡 Klik op het kopieer-icoontje bij een command om het direct te kopiëren.

---

## 📑 Contents

- [📁 Files & Directories](#-files--directories)
- [📖 Files bekijken](#-files-bekijken)
- [🔎 Zoeken](#-zoeken)
- [📦 Software & APT](#-software--apt)
- [⚙️ Services](#️-services)
- [🌐 Networking](#-networking)
- [👤 Users](#-users)
- [🔐 Permissions](#-permissions)
- [💾 Disk & Storage](#-disk--storage)
- [🧠 Processes](#-processes)
- [📜 Logs](#-logs)
- [🔑 SSH](#-ssh)
- [🗜️ Archives](#️-archives)
- [⌨️ Shortcuts](#️-shortcuts)
- [⭐ Commands om eerst te leren](#-commands-om-eerst-te-leren)
- [⚠️ Dangerous Commands](#️-dangerous-commands)

---

# 📁 Files & Directories

## Huidige directory

```bash
pwd
````

 ## Bestanden bekijken

```
ls
```

```
ls -la
```

 ## Directory veranderen

```
cd folder
```

```
cd ..
```

```
cd ~
```

```
cd -
```

 ## Directory maken

```
mkdir folder
```

 ## Bestand maken

```
touch file.txt
```

 ## Bestand kopiëren

```
cp file.txt backup.txt
```

 ## Directory kopiëren

```
cp -r folder backup
```

 ## Bestand verplaatsen / hernoemen

```
mv old.txt new.txt
```

```
mv file.txt folder/
```

 ## Bestand verwijderen

```
rm file.txt
```

 ## Directory verwijderen

```
rm -r folder
```

 > ⚠️ `rm` verwijdert bestanden direct.

---

 # 📖 Files bekijken

 ## Bestand volledig bekijken

```
cat file.txt
```

 ## Bestand pagina voor pagina bekijken

```
less file.txt
```

 ## Eerste regels bekijken

```
head file.txt
```

 ## Laatste regels bekijken

```
tail file.txt
```

 ## Logbestand live bekijken

```
tail -f logfile.log
```

 ## Bestand bewerken

```
nano file.txt
```

 ### Nano shortcuts

 | Shortcut | Functie |
| --- | --- |
| `CTRL + O` | Opslaan |
| `CTRL + X` | Afsluiten |
| `CTRL + W` | Zoeken |
| `CTRL + K` | Regel knippen |
| `CTRL + U` | Plakken |

---

 # 🔎 Zoeken

 ## Bestand zoeken

```
find . -name "file.txt"
```

 ## Configuratiebestanden zoeken

```
find /etc -name "*.conf"
```

 ## Tekst zoeken

```
grep "error" file.txt
```

 ## Recursief zoeken

```
grep -r "error" /var/log/
```

 ## Locatie van een command vinden

```
which python3
```

---

 # 📦 Software & APT

 ## Package lists updaten

```
sudo apt update
```

 ## Packages upgraden

```
sudo apt upgrade
```

 ## Software installeren

```
sudo apt install package
```

 Voorbeeld:

```
sudo apt install git curl htop
```

 ## Software verwijderen

```
sudo apt remove package
```

 ## Software + configuratie verwijderen

```
sudo apt purge package
```

 ## Ongebruikte packages verwijderen

```
sudo apt autoremove
```

 ## Package zoeken

```
apt search package
```

 ## Package informatie bekijken

```
apt show package
```

---

 # ⚙️ Services

 ## Status bekijken

```
systemctl status service
```

 ## Service starten

```
sudo systemctl start service
```

 ## Service stoppen

```
sudo systemctl stop service
```

 ## Service herstarten

```
sudo systemctl restart service
```

 ## Service automatisch starten

```
sudo systemctl enable service
```

 ## Automatisch starten uitschakelen

```
sudo systemctl disable service
```

 ### Voorbeeld met SSH

```
systemctl status ssh
```

```
sudo systemctl restart ssh
```

---

 # 🌐 Networking

 ## IP-adressen bekijken

```
ip addr
```

 Kort:

```
ip a
```

 ## Routing bekijken

```
ip route
```

 ## Internet testen

```
ping 8.8.8.8
```

 ## DNS testen

```
ping google.com
```

 ## Luisterende ports bekijken

```
ss -tulpn
```

 ## HTTP request

```
curl https://example.com
```

 ## Bestand downloaden

```
wget https://example.com/file
```

 ## Lokaal IP bekijken

```
hostname -I
```

 ## DNS informatie

```
resolvectl status
```

---

 # 👤 Users

 ## Huidige gebruiker

```
whoami
```

 ## User ID en groups

```
id
```

 ## Ingelogde gebruikers

```
who
```

 ## Command uitvoeren als administrator

```
sudo command
```

 Voorbeeld:

```
sudo apt update
```

 ## Wachtwoord veranderen

```
passwd
```

 ## Andere gebruiker

```
su - username
```

---

 # 🔐 Permissions

 Linux gebruikt:

```
r = read
w = write
x = execute
```

 ## Permissions bekijken

```
ls -l
```

 ## Bestand executable maken

```
chmod +x script.sh
```

 ## Permissions instellen

```
chmod 644 file.txt
```

```
chmod 755 script.sh
```

 ## Owner veranderen

```
sudo chown user:group file.txt
```

 ### Veelgebruikte permissions

 | Permission | Betekenis |
| --- | --- |
| `644` | `rw-r--r--` |
| `755` | `rwxr-xr-x` |
| `700` | `rwx------` |
| `777` | `rwxrwxrwx` |

> ⚠️ Gebruik `chmod 777` liever niet.

---

 # 💾 Disk & Storage

 ## Beschikbare diskruimte

```
df -h
```

 ## Grootte van een directory

```
du -sh folder
```

 ## Grootte van alles in huidige directory

```
du -sh *
```

 ## Disks en partitions

```
lsblk
```

 ## Mounted filesystems

```
mount
```

---

 # 🧠 Processes

 ## Alle processen

```
ps aux
```

 ## Live processen

```
top
```

 ## Htop

```
htop
```

 Installeren:

```
sudo apt install htop
```

 ## Process zoeken

```
pgrep firefox
```

 ## Process stoppen

```
kill PID
```

 Voorbeeld:

```
kill 1234
```

 ## Process forceren te stoppen

```
kill -9 PID
```

 > ⚠️ Gebruik `kill -9` alleen wanneer normaal stoppen niet werkt.

---

 # 📜 Logs

 ## Alle system logs

```
journalctl
```

 ## Logs van huidige boot

```
journalctl -b
```

 ## Logs live volgen

```
journalctl -f
```

 ## Logs van een service

```
journalctl -u ssh
```

 ## Service logs van huidige boot

```
journalctl -u ssh -b
```

---

 # 🔑 SSH

 ## Verbinden met een andere computer

```
ssh username@192.168.1.100
```

 ## Bestand kopiëren

```
scp file.txt username@192.168.1.100:/home/username/
```

 ## Directory kopiëren

```
scp -r folder username@192.168.1.100:/home/username/
```

---

 # 🗜️ Archives

 ## `.tar.gz` maken

```
tar -czf backup.tar.gz folder/
```

 ## `.tar.gz` uitpakken

```
tar -xzf backup.tar.gz
```

 ## ZIP maken

```
zip -r backup.zip folder/
```

 ## ZIP uitpakken

```
unzip backup.zip
```

---

 # 🖥️ System Commands

 ## Systeeminformatie

```
uname -a
```

 ## Computernaam

```
hostname
```

 ## Datum en tijd

```
date
```

 ## Hoelang draait het systeem?

```
uptime
```

 ## Reboot

```
sudo reboot
```

 ## Shutdown

```
sudo poweroff
```

---

 # 🧹 Terminal

 ## Terminal leegmaken

```
clear
```

 ## Command history

```
history
```

 ## Laatste command opnieuw uitvoeren

```
!!
```

 ## Laatste command met sudo

```
sudo !!
```

---

 # ⌨️ Shortcuts

 | Shortcut | Functie |
| --- | --- |
| `CTRL + C` | Command stoppen |
| `CTRL + Z` | Command pauzeren |
| `CTRL + D` | Shell afsluiten |
| `CTRL + L` | Terminal leegmaken |
| `CTRL + R` | History doorzoeken |
| `TAB` | Autocomplete |
| `↑` | Vorig command |
| `↓` | Volgend command |

---

 # ⭐ Commands om eerst te leren

 Als je net begint met Linux, leer deze eerst:

```
pwd
ls -la
cd
mkdir
touch
cp
mv
rm
cat
less
nano
grep
find
sudo
apt
systemctl
ip a
ssh
df -h
du -sh
ps
kill
journalctl
```

---

 # 🧠 Command Map

```
FILES
├── ls
├── cd
├── cp
├── mv
├── rm
└── find

TEXT
├── cat
├── less
├── grep
├── head
├── tail
└── nano

SOFTWARE
└── apt

SERVICES
└── systemctl

PROCESSES
├── ps
├── top
├── htop
└── kill

NETWORK
├── ip
├── ss
├── ping
├── curl
├── wget
└── ssh

STORAGE
├── df
├── du
└── lsblk

PERMISSIONS
├── chmod
├── chown
└── sudo

LOGS
└── journalctl
```

---

 # ⚠️ Dangerous Commands

 Wees voorzichtig met:

```
rm -rf
```

```
sudo rm
```

```
chmod -R 777
```

```
sudo dd
```

 > ⚠️ Controleer altijd wat een command doet voordat je het uitvoert met `sudo`, `rm` of andere destructieve opties.

---

 # 📚 Quick Reference

 | Taak | Command |
| --- | --- |
| Current directory | `pwd` |
| Files bekijken | `ls -la` |
| Directory veranderen | `cd` |
| Directory maken | `mkdir` |
| Bestand maken | `touch` |
| Kopiëren | `cp` |
| Verplaatsen | `mv` |
| Verwijderen | `rm` |
| Bestand bekijken | `cat` |
| Bestand bewerken | `nano` |
| Zoeken | `find` |
| Tekst zoeken | `grep` |
| Installeren | `sudo apt install` |
| Packages updaten | `sudo apt update` |
| Packages upgraden | `sudo apt upgrade` |
| Service status | `systemctl status` |
| IP bekijken | `ip a` |
| Processes | `ps aux` |
| Disk space | `df -h` |
| Folder size | `du -sh` |
| Logs | `journalctl` |
| SSH | `ssh user@host` |
| Reboot | `sudo reboot` |
| Shutdown | `sudo poweroff` |

---

 ## 🚀 Happy Linux-ing!

 > **Learn the command → understand the command → run the command.**

`````

**Let op:** in dit antwoord kan de chatweergave zelf nog steeds iets aan de Markdown tonen. Maar de inhoud die je naar GitHub kopieert bevat nu alleen standaard Markdown. De cruciale vorm is bijvoorbeeld:

````text
```bash
sudo apt update
`````

```

Dus **geen `id=...` achter `bash`**.
```
