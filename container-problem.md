# Das Container-Problem

Im Folgenden möchte ich kurz beschreiben auf welche Probleme ich beim Aufsetzten einer eigenen Arbitrum-Node gestoßen bin. Die Node sollte zusammen mit einer lokal laufenden Ethereum-Rinkeby-Light-Node und einer Chainlink-Node dafür verwendet werden, Oracle-Daten auf das Arbitrum-Rinkeby-Testnet (Layer-2) zu bringen. Wieso das Ganze? Momentan gibt es noch keine öffentlich zugänglichen Arbitrum-Rinkeby Oracles, um Off-Chain-Daten in eigenen Smart-Contracts zu verwenden.

## Ausgangssituation

Für das VoFarm-Projekt, im Rahmen eines ersten Prototypen, wollen wir unsere Trading-Strategien auf dem Arbitrum-Netzwerk testen. Dafür habe ich eine Maschine, mit alter Intel-CPU und ein paar Gb Ram. Als Betriebssystem wurde Ubuntu 20.04 installiert. Zu Beginn wurde vermutet, dass dies reichen würde, um die nötigen Nodes laufen zu lassen. 

## Ablauf

Das Erste was für dieses Vorhaben benötigt wird, ist Zugang zu der Ethereum-Chain. Dafür können auf der einen Seite Dienste wie „Infura“ oder ähnliche verwendet werden. Bei solchen Diensten ist man jedoch auf eine Anzahl von Anfragen pro Tag begrenzt. Sonst müsste man Gebühren zahlen. Das wollen wir an dieser Stelle nicht. Deshalb muss ein alternativer Weg gefunden werden. Wir hosten also eine lokale Rinkeby-Light-Node am der Maschine. Über diese werden wir dann auf die Chain zugreifen. Ein Tutorial dafür ist unter diesem Link verfügbar. 
[Rinkeby-Light-Node Tutorial](https://www.rinkeby.io/#geth)

Mit dem Betreiben einer eigenen Rinkeby-Node ist der erste wichtige Schritt getan. Für das Hosten der eigenen Arbitrum-Node bietet Arbitrum ein eigenes Tutorial, welches ich an dieser Stelle verwenden werden. Das Tutorial ist unter dem folgenden Link erreichbar: 
[Arbitrum-Node Tutorial](https://developer.offchainlabs.com/docs/running_node)

Laut dem Tutorial gibt es an sich nur einen Befehl, der ausgeführt werden muss:
```
docker run --rm -it  -v /some/local/dir/arbitrum-rinkeby/:/home/user/.arbitrum/rinkeby -p 0.0.0.0:8547:8547 -p 0.0.0.0:8548:8548 offchainlabs/arb-node:v1.1.2-cffb3a0 --l1.url https://l1-rinkeby-node:8545
```

Für unsere Situation muss das Dateiverzeichnis und der L1-Provider angepasst werden. Der L1-Provider, da wir unsere eigene lokale Rinkeby-Node hosten, lautet http://127.0.0.1:8545. Der angepasste Befehl sieht also wie folgt aus:
```
docker run --rm -it  -v /home/felix/node/arbitrum-rinkeby/:/home/user/.arbitrum/rinkeby -p 0.0.0.0:8547:8547 -p 0.0.0.0:8548:8548 offchainlabs/arb-node:v1.1.2-cffb3a0 --l1.url http://127.0.0.1:8545
```

## Die ersten Probleme

Hier beginnen nun die Probleme. Das erste Mal, als der Befehl ausgeführt wird, lädt man das Docker-Image herunter. So weit so gut. Danach bricht der Prozess ab. Ohne eine direkte Fehlermeldung zu werfen. Sobald man den Befehl dann erneut ausführt, passiert nicht. Keine Fehlermeldung, keine Rückmeldung, nichts. 

An dieser Stelle beginnt die Kommunikation mit dem Arbitrum-Support, da nach langer, eigener Recherche keine eigene Lösung gefunden werden konnte. Als nächster Schritt wurden die Logs von Docker ausgelesen:

### 1st Log

Jan 05 14:19:45 node kernel: docker0: port 1(veth6dc6a85) entered blocking state

Jan 05 14:19:45 node kernel: docker0: port 1(veth6dc6a85) entered disabled state

Jan 05 14:19:45 node kernel: device veth6dc6a85 entered promiscuous mode

Jan 05 14:19:45 node kernel: eth0: renamed from vethb103ae0

Jan 05 14:19:45 node kernel: IPv6: ADDRCONF(NETDEV_CHANGE): veth6dc6a85: link becomes ready

Jan 05 14:19:45 node kernel: docker0: port 1(veth6dc6a85) entered blocking state

Jan 05 14:19:45 node kernel: docker0: port 1(veth6dc6a85) entered forwarding state

Jan 05 14:19:45 node kernel: traps: arb-node[47015] trap invalid opcode ip:7f7260bed55e sp:7ffd6ea51400 error:0 in 
librocksdb.so.6.20.3[7f7260b7a000+470000]

Jan 05 14:19:45 node kernel: docker0: port 1(veth6dc6a85) entered disabled state

Jan 05 14:19:45 node kernel: vethb103ae0: renamed from eth0

Jan 05 14:19:45 node kernel: docker0: port 1(veth6dc6a85) entered disabled state

Jan 05 14:19:45 node kernel: device veth6dc6a85 left promiscuous mode

Jan 05 14:19:45 node kernel: docker0: port 1(veth6dc6a85) entered disabled state


### 2nd Log

Jan 05 14:49:48 node dockerd[47229]: time="2022-01-05T14:49:48.160707037+01:00" level=info msg="ignoring event" container=7c49ff4e3c3c493830304d81e82d0261ccc06a75b5e10c7ba49a2d9e2985f065 module=libconta>

Diese Logs werden bei jedem Mal, wenn man probiert den Container zu starten, im gleichen Muster ausgegeben. Egal welche Parameter man bei dem Docker-Command übergibt, die Logs bleiben gleich. Mit diesen Logs wurde sich ein weiteres Mal an den Arbitrum-Support gewendet. Diese melden nach einem Tag zurück. Laut ihnen hat die Maschine, auf der ich probiert habe den Container zu starten, eine zu alte CPU.

## Fazit

Laut des Arbitrum-Supports, muss eine Maschine bestimmte Hardwareansprüche erfüllen. Sie vergleichen die Anforderungen mit Spezifikationen einer AWS VM. Um eine Node stabil laufen zu lassen, braucht man eine VM des Typen „r5d.2xlarge“. 
 
Der Versuch eine vergleichbare Maschine mittels einer zur Verfügung stehenden Azure-Subscription laufen zu lassen schlug fehl. Die Subsription ist eine reine Studentenversion. Mit einer solchen ist es nicht möglich, so starke Maschinen hochzufahren. Außerdem müssen wir davon ausgehen, dass die Maschine noch ein kleines Stück stärker sein muss, als von Arbitrum-Support angegeben, da wir zusätzlich eine Rinkeby und Chainlink-Node hosten müssen. Alles auf der gleichen Maschine. Man könnte die Nodes auch auf unterschiedliche Maschinen verteilen, das ändert aber nichts an der Tatsache, dass die Arbitrum-Node nicht lauffähig ist. Zumingest zum aktuellen Zeitpunkt mit den zur Verfügung stehenden Ressourcen.

