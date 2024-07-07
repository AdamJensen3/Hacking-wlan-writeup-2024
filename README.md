### Fehler und Verbesserungen

- [ ] Bilder sortieren Dopplte raus und in ein Verzeichnis
- [ ] Handshake reinmachen
- [ ] gegenprüfen lassen
- [ ] About me




# Tutorial WPA2 Handshake bruteforcen:

## Wichtiger Hinweis:

Die gezeigeten Methoden, Befehle und Techniken dienen der Sensibilisierung und dem Selbststudium, und dürfen unter gar keinen Umständen für böse Absichten eingesetzt werden. Darüber hinaus ist es in Deutschland strafbar fremde Netzwerke, Webseite, Server oder ähnliches zu hacken, ohne schriftliche Genehmigung des Eigentümer

### Vorraussetzungen 

Hardware: Laptop/Rechner mit WLAN-Karte<br>
Software: aktuelle [Kali](https://www.kali.org/get-kali/#kali-installer-images)  Version.

### Let´s Hack

Zunächst müssen wir die WLan-Karte in den Monitormodus versetzen, denn wir benötigen diese es unser Werkzeug.
* WLAN-Karte vorbereiten
  ```sh
  sudo airmon-ng start wlan0 
  ```

![image](images/1.png "AA")
Wir werden von Kali darauf hingewiesen dass noch zwei Prozesse auf die Karte zugreifen. Uns wird auch der passende Befehl angeboten um diese beiden Prozesse zu beenden (killen)
  
  ```sh
  sudo airmon-ng check kill
  ```

![image](images/2.png "AA")

Danach steht und die Wlan-Karte voll und ganz zur Verfügung.
Nun schauen wir was wir alles in unsrer Umgebung empfangen können.
  ```sh
  sudo airodump-ng start wlan0mon
  ```
wlan0mon ist nun der Name unseres Netzwerkinterfaces welches sich im Monitor-Mode befindet daher das "mon" am Ende
![image](images/4.png "AA")

![image](images/3.png "AA")

Was sehen wir nun? Wie kann man das angezeigte interpretieren?
Wir sehen also drei Netzwerk im Reichweite.
In der ersten Spalte sehen wir die einzigartigen BSSID von den Netzwerken, gefolgt von der Spalte PWR als abkürzung für Power die Werte die wir hier sehen sind immer negativ da sich diese auf die Signalstärke in dbm beziehen. Diese Zahl ist ein Indikator dafür wie nah man an einem Netzwerkteilnehmer ist.
Beispiel ein Netzwerk mit dem Wert -35 ist näher an unserem Laptop als ein Netzwerk mit -54. Allgemein gilt beim WLan-Hacking je dichter desto besser
Die Spalte ENC steht für die Verschlüsselungsart  Welches Verschlüsselungsarten gibt es OPN= open/offen/ohne Verschlüsselung, WEP, WPA2 und WPA3
Die letzte Spalte ist die ESSID umgangssprachlich oft als SSID bzw "Wlan-Namen" bekannt. ESSID steht für Extended Service Set Identifier
Hier sehen wir erste wichtige Hinweise.
Es handelt sich um ein Gerät der Marke TP-Link und es scheint nicht Benutzer angepasst worden zu sein, da die ESSID bei allen wirklich allen TP-Link Produkten mir Wlan immer dem selben Schema folgt. Der Markenname+die letzen 4 Zeichen der MAC Adresse des Gerätes diese sehen wir auch in der ersten Spalte BSSID. Konkret also "68F0"
 
Um eine WPA2-Verschlüsselung zu erraten benötigen wir den Handshake zwischen dem Sender und Empfänger dazu benutzen wir den folgenden Befehl

  ```sh
  sudo airodump-ng start wlan0mon --bssid   C0:C9:E3:B3:68:F0 -w handshake
  ```
![image](5.png "AA")

 Mit diesem Befehl weisen wir an, dass wir nur die Datenpakte die mit der angebenen BSSID zu tun haben, also unserem Zielnetzwerk, aufgezeichnet werden sollen.
 Die gesamte Aufzeichung soll in die mit dem Parameter -w (write) in die Datei handshake geschrieben werden. Diese Bezeichnung ist nicht zwingend, man hätte es auch hugo nennen können aber gewöhnt euch gleich an die Dinge richtig zu erledigen. Das Programm erstellt uns automatisch die benötige .cap Datei. Dieser werden wir in zukünftig foglenden Anleitunge bei Wireshark wieder sehen.

 Es öffnet sich nun die folgende Anzeige mit unserem Zielnetzwerk

![image](images/6.png "AA")

Hier ist nun besonders die Stelle oben rechts für uns interessant, denn nun gilt es den Hanshake einzufangen bzw. aufzuzeichnen.
Auch hierzu gibt es wiedereinmal mehrere Ansätze. Zum Beispiel warten bis sich ein Gerät wie ein Handy mit dem Netzwerk verbindet.Man kann aber mit dem DEAUTH-Befehl verbunde Geräte kurz aus dem Netzwerk kicken. Diese verbinden sich dann direkt wieder mit dem WLAN-Sender und man bekommt den Handshake. Darum soll es hier aber jetzt nicht gehen. Zu diesen Schulungszweken habe ich ein Handy mit den Netzwerk verbunden.

![image](images/7.png "AA")

Nun können wir erkennen dass mindestens ein Handshake aufgezeichnet wurde. Nun können wir dich Aufzeichnung des WLAN-Verkers mit der Tastekombination STRG+C beenden

  ```sh
  [STRG]+[C] 
  ```

Nun kommt die Stelle die ein Skriptkiddi von einem Penetrationtester unterscheidt

"Spezialwissen" über TP-Link
- TP-Link verwendet bei allen WLAN-Produkten den WPA2 mindeststandart bei der Passwortlänge also 8 Zeichen.
- Die WLAN-Passwörter bei TP-Link bestehen in der Grundkonfiguration immer nur aus Zahlen

Kali verfügt über eine mächtige eingebaut Wordliste die Rockyou.txt aber ein Angriff auf die Verschlüsselung würde nichts bringen, da dort nicht das benötigte Passwort drin steht.

Daher generieren wir uns die passende Passwordliste mit den folgenden Befehl

  ```sh
  crunch 8 8 0123456789 -o 8er_zahlen
  ```

![image](images/8.png "AA")

Wir nutzen das Programm Crunch um uns eine Liste mit Passwörter erstellen zu lassen. Wir geben folgende dabei an: Mindestlänge des PW ist 8 Maximallänge des PW ist 8. Woraus soll das Passwort bestehen nur aus den Zahlen 0123456789 und alles soll mit dem Parameter -o (Output) in die Datei 8er-zahlen geschrieben werden. Das Programm zeigt uns die zu erwartende Dateigröße von ca. 860MB. 
Wenn man sich größer Wortlisten erstellen läst gibt das Programm zyklisch Statusupdates in der Form von "crunch: 91% completed generating output" raus

Direkt im Anschluss habe ich mir noch zu Vorführungszwecken eine anepasste Version erstellt. Die ist wesentlich kürzer und somit schneller. Damit dauert der Vorgang nur ein paar Sekunden und nicht Stunden.


  ```sh
  crunch 8 8 0123456789 -s 57880000 -e 78890000  -o 8er_zahlen_kurz
  ```

Die Option -s (start) gibt an ab Welchen Zahlenanfang bzw. Offset das Programm beginnen soll.
Die Option -e (end) gibt an bis zu welchen Zahlenwert das Programm arbeiten soll.
![image](images/9.png "AA")

Diese verkürzte und angepasste Liste ist nicht mal 1 MB groß.


Nun prüfen wir ob wir alles haben was wir beötigen, dazu tippen wir den folgenden Befehl ein.

  ```sh
  ll
  ```
![image](10.png "AA")

Ihr sollte jetzt die gleichen Dateien haben.
Am wichtigsten ist die Datei mit der Endung CAP. Dort ist der Handshake drin, den wir knacken wollen.

Nun Können wir ans "cracken" gehen.

  ```sh
  aircrack-ng handshake-01.cap -w 8er_zahlen
  ```

![image](11.png "AA")


Dieser Vorgang dauert je nach Hardware und Passwortliste von einigen Stunden bis hin zu Tagen.

![image](images/12.png "AA")

Hier sieht man nun den eigentlichen Crack-Prozess. Das Programm versucht mit jeden "Wort" aus der Wortliste den Handshake zu knacken. Da wir aber wissen, dass es eine reine 8 stellige Zahlenkombiation ist sehen wir hier das Programm dies mit Zahlen machen. In der Grafik oben wird gerade der Passwort "00017603" getestet. Wir haben bereits 0.16% der Passwortliste abgearbeitet. 



Dieser Prozess benötig viel Hardwareleistung:
Profis nutzen dafür die GPU der Grafikkarte. Hier für unsere Schulungszwecke reicht die Leistung der CPU föllig aus.
![image](14.png "AA")



![image](images/13.png "AA")

Sehr schnell passt sich geschätze Zeitdauer an. Als Faustformel kann man sagen, dass ein Passwort mit 8 Stellen (nur Zahlen) über Nacht geknackt werden kann.

![image](images/14.png "AA")

Da wir abeer hier zur Schulungszwecken direkt nach wenigen Sekunden einen Erfolg sehen wollen, nutzen wir die kleinere Wortliste "**8er_zahlen_kurz**". Somit spraren wir uns mehrere Stunden.

![image](images/15.png "AA")

Den Erfolg kann man direkt sehen, denn nach nichtmal einer Sekunde und 84,15% wurde das richtige Passwort erraten. Es ist **57886046**

![image](images/16.png "AA")



Ein kurzer Blick unter den Router verrät, dass wir das richtige Passwort haben.

Zusatzwissen: Was sagt uns dies jetzt über den Benutzer/Admin des Routes?
  
  -Der Router ist komplett default (ESSID und das Werkspasswort)
  -Die Wahrscheinlichkeit, dass die Logindaten des Routers auch noch default ist, ist sehr hoch.
  -Es gibt viele Seiten wo man die Default-Passwörter der Hersteller/Modell einsehen 


Wir notieren und den gefunden Key und starten den Rechner neu.
Da wir unsere WLAN-Karte in den Monitormode versetzt haben können wir uns nicht direkt mit dem geknackten WLAN verbinden.

Der einfachste Weg wieder alles in default zu bekommen ist ein neustart.
Einfach über die Konsole mit dem Befehl

  ```sh
  sudo reboot now
  ```

Danach können wir uns mit den verschlüsselten WLAN verbiden:

 ![image](18.png "AA")


![image](19.png "AA")


Nun sind wir verbunden. 
Trotzdem checken wir es nochmal.
Dazu in der Konsole den fogenden Befehl eingeben

  ```sh
  nmcli dev wifi
  ```

![image](20.png "AA")
 
 **Jawohl unser erster WLAN-Hack war erfolgreich!**

 In diesem Git-Repository ist mein Handshake meinr Hardware drin. Und ich erlaube euch ausdrücklich mit diesem zu spielen und zu testen. Jetzt habt Ihr alles was ihr braucht. Viel Spaß beim nachmachen.

 **Adam Jensen**

<div align="center">

Über den Author  
 ![image](images/avatar.png "AA")
 
Chief of Security (Sarif Industries)<br>
Member of Task Force 29 <br>

Discord Server
[<img src="images/discord.png">](https://discord.gg/rN4Q32EM)


</div>









