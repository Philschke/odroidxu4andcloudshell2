# Odroid XU4 & Cloudshell2 als NAS-Server
Meine Hardware: Odroid XU4, Cloudshell2, MMC 32GB, 2x WD RED 6TB, RAID1

## Distro
Da der Odroid mal einen NAS Server werden soll, brauchen wir [OMV](https://www.openmediavault.org/). Leider baut OMV nicht für alle Computer ein passendes Image. Daher müssen wir es später händisch installieren. Es gibt ein Installationsskript, welches auf dem Basis Debian Image basiert. Daher können wir leider nicht das offizielle Ubuntu OS vom Odroid nehmen. Lösung: Armbian.
Wichtig dabei ist die Kernel-Version. Denn nur mit der 4.14 Kernel lässt sich alles installieren. Zu dem Image kommt ihr hier: Also ist das zu installierende Betriebssystem Armbian Buster Legacy 4.14. [Download](https://armbian.hosthatch.com/archive/odroidxu4/archive/Armbian_20.05.2_Odroidxu4_buster_legacy_4.14.180.img.xz) des Images.
Dieses Image wie alle anderen auch installieren.

## Lüfter und LCD Display
Da Armbian Debian als Basis hat und alle Pakete des Lüfters und des LCD Displays Ubuntu orientiert sind, müssen wir etwas tricksen. Folgerndermaßen: 

**Kyle PPA hinzufügen**
```bash
odroid@odroidxu4:~$ sudo add-apt-repository ppa:kyle1117/ppa
 
More info: https://launchpad.net/~kyle1117/+archive/ubuntu/ppa
Press [ENTER] to continue or ctrl-c to cancel adding it

gpg: keybox '/tmp/tmpf11e1bb0/pubring.gpg' created
gpg: /tmp/tmpf11e1bb0/trustdb.gpg: trustdb created
gpg: key 3028C3C96AD57103: public key "Launchpad PPA for KYLE LEE" imported
gpg: Total number processed: 1
gpg:               imported: 1
gpg: no valid OpenPGP data found.
```
**Key manuell hinzufügen**
```bash
odroid@odroidxu4:~$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 3028C3C96AD57103
Executing: /tmp/apt-key-gpghome.WU8NDps3LZ/gpg.1.sh --keyserver keyserver.ubuntu.com --recv-keys 3028C3C96AD57103
gpg: key 3028C3C96AD57103: public key "Launchpad PPA for KYLE LEE" imported
gpg: Total number processed: 1
gpg:               imported: 1
```
**Distro Namen in ein Ubuntu System ändern**
```bash
odroid@odroidxu4:~$ sudo sed -i 's/jammy/bionic/g' /etc/apt/sources.list.d/kyle1117-ubuntu-ppa-jammy.list 
```
**System und Distro auf den aktuellen Stand bringen**
```bash
odroid@odroidxu4:~$ sudo apt update
odroid@odroidxu4:~$ sudo apt upgrade
odroid@odroidxu4:~$ sudo apt full-upgrade
odroid@odroidxu4:~$ sudo apt autoremove
```
**LCD und Lüfter Pakete installieren**
```bash
odroid@odroidxu4:~$ apt install cloudshell-lcd cloudshell2-fan odroid-cloudshell 
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  i2c-tools libi2c0
```
**Anpassen des Interface Namens von eth auf enx**
```bash
odroid@odroidxu4:~$ sed -i 's/\/eth/\/enx/g' /bin/cloudshell-lcd 
```
**Einmal Reboot um Ergebnisse zu sehen**

## Open Media Vault 5
**Es gibt ein Script, was einem bei der Installation von OMV hilft. Einfach folgenden Befehl ausführen(Kann etwas länger dauern mit der Installation):**
```bash
wget -O - https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/install | sudo bash
```
## Zusatz: Installation der grafischen LCD Anzeige von [AreaScout](https://forum.odroid.com/viewtopic.php?f=147&p=246480#p246480)
*Achtung: Hierbei handelt es sich um externen Inhalt und man sollte dabei immer vorsichtig sein. Bitte vor der Ausführung kontrollieren ob es wirklich noch das richtige ist.*

**Herunterladen der Datei**
```bash
odroid@odroidxu4:~$ sudo wget --no-check-certificate http://www.areascout.at/cloudshell2-monitoring_1.0.23-1_armhf.deb
```
**Installieren**
```bash
odroid@odroidxu4:~$ sudo apt install./cloudshell2-monitoring_1.0.23-1_armhf.deb
```
**Symbolischen Link zur einfacheren Konfiguration im Home Verzeichnis erstellen**
```bash
odroid@odroidxu4:/tmp/ich/bin/nicht/im/home$ cd
odroid@odroidxu4:~$ ln -s /etc/cloudshell2-monitoring/cloudshell2-monitoring.conf cloudshell2-monitoring.conf
# Zum Öffnen der Konfigurationsdatei nun einfach folgenden Befehl ausführen (mit vi oder bash)
odroid@odroidxu4:~$ vi cloudshell2-monitoring.conf
```
*Weitere Hinweise zu Konfiguration o.ä. findet ihr in dem [Forumbeitrag](https://forum.odroid.com/viewtopic.php?f=147&p=246480#p246480) von AreaScout. In dieser Konfiguration lassen sich auch die Lüftereinstellungen anpassen.*

**Deinstallation des alten LCD Skriptes, damit nur noch das neue angezeigt wird**
```bash
odroid@odroidxu4:~$ sudo apt-get remove cloudshell-lcd
```
