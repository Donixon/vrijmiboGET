# VrijMiBo Discord image poster
De lads en ik kijken graag iedere vrijdag de Dumpert Vrijmibo plaatjes. Echter Dumpert laatst besloten om de NSFW plaatjes achter een inlog te gooien. Heeft niemand natuurlijk zin in. Daarom moest er iets bedacht worden om deze vrijmibo plaatjes graties te kunnen zien. 
In NodeRED heb ik daarvoor iets moois in elkaar gegooid. 
## Benodigdheden
1.	Server waar NodeRED (Node-engine 16+) op draait. Ik heb het in Docker draaien op mijn Synology NAS. Daarbij moeten er ook nog 2 Pallets installed worden in Node-RED:
a.	node-red-contrib-discord-advanced – Discord nodes voor messages ophalen en verzenden
b.	node-red-contrib-lower – Om alle payloads lower-case te maken
2.	 Discord Application voor de bot: https://discord.com/developers/applications

## Opzetten
Als NodeRED draait en je de discord-application hebt aangemaakt (en de bot met rechten in je server hebt gehangen(minimaal rechten om te verzenden en lezen)), kan je het volgende importen in NodeRed: 
`` 
<NodeRED_Clipboard_Export_v1.txt>
`` 
1.	Zet je Discord token in de Discord nodes
2.	Vul de commands waar hij op triggert in de node “commands” 
3.	Zet in alle function-nodes “msg.channel” op het channel-ID waar je wilt dat de message verzonden wordt
a.	Channel-ID?: Rechter-muisklik op de channel in Discord en “Copy ID”
Eerste gebruik
De VrijMiBot ( :’) ) heeft een URL nodig, zodat deze URL gestript kan worden en het page-ID overblijft:
•	https://www.dumpert.nl/item/100018197_5d82f342 >> 100018197_5d82f342
Dit laatste stukje zet hij achter de Dumpert-API-url:
•	https://api-live.dumpert.nl/mobile_api/json/info/100018197_5d82f342
Hierna schrijft hij dit in een bestand weg. Het commando voor het zetten van de Dumpert-URL staat in de commands node:
•	dumpert url <URL van de vrijmibo met plaatjes>
o	dumpert url https://www.dumpert.nl/item/100018197_5d82f342

## Commands gebruiken
Nu alles opgezet is en de Dumpert-URL van de Vrijmibo bekend is, kan je in Discord het command gooien: alle melkers . Deze zorgt ervoor dat er in de juiste channel (voorheen neergezet) alle plaatjes worden gepost die in de bovengenoemde URL stonden. 

## Uitleg Flow
![alt text](https://github.com/Donixon/vrijmiboGET/blob/main/NodeRED_Uitleg_flow_v1.png?raw=true)
* **Geel – Discord out**
    * De Discord-node checkt alle messages die binnenkomen
    * De lower-case-node zet alle payloads die binnenkomen om in, najah, lower-case letters. Dit zorgt ervoor dat het niet uitmaakt hoe je het command in Discord stuurt:
        	Alle Melkers
        	AlLe mElKeRs
        	Hij keurt het nu allemaal goed in de volgende node
    *	In de Switch-node (command-node), staan de values die hij zoekt in de messages die binnenkomen. Mocht er iets overeenkomen (bijv. command: **alle melkers**), zet hij dit door naar het juiste stukje
* **Groen – Dumpert URL**
    * Deze nodes gaan draaien als het command **dumpert url xxx** binnenkomt. Hij haalt in de change-node het laatste stukje van de URL eruit en zet dit achter de Dumpet-API-URL. 
    * Schrijft deze API-URL weg in een bestand zodat andere nodes deze URL kunnen gebruiken
* **Rood – Alle melkers**
    * Als de command **alle melkers** aangeroepen wordt, is de bedoeling dat alle images uit de vrijmibo gestuurd worden. De eerste node haalt de Dumpert-API-URL uit het bestand op.
    * Hierna zet een switch node msg.payload (is de output van de read-file node) om naar een msg.url zodat de HTTP-request node het begrijpt
    * HTTP-request gooit een GET naar de opgegeven API-URL in plain-tekst op de msg.payload
    * Een JSON node format dit correct
    * Hierna haalt hij de juiste array op, waar alle URL’s in staan van de images
    * De split node zorgt ervoor dat iedere image-URL los op de msg.payload wordt gezet. Zo kunnen er meerdere images verstuurd worden.
    * Laatste function node zet het channel-ID en veranderd de msg.payload goed, zodat de URL in de payload staat
* **Paars -  Discord in**
    * Aangezien er op de Discord-API limieten zitten hoe snel je berichten mag versturen, zit er een delay-node in de flow. Deze node zorgt ervoor dat er 1 message per 1 seconde wordt verstuurd. Dit scheelt weer wat Discord-Bans
    * De laatste Discord-node verstuurd de Dumpert-image-URL naar de correcte Discord channel
