<p align="center">
  <img src="https://github.com/Donixon/vrijmiboGET/blob/main/DumpertVRIJMIBOGET.png" />
</p>
<h1>VrijMiBo Discord image poster</h1>
<p>De lads en ik kijken graag iedere vrijdag de Dumpert Vrijmibo plaatjes. Echter Dumpert laatst besloten om de NSFW plaatjes achter een inlog te gooien. Heeft niemand natuurlijk zin in. Daarom moest er iets bedacht worden om deze vrijmibo plaatjes graties te kunnen zien.<br /> In NodeRED heb ik daarvoor iets moois in elkaar gegooid.</p>
<h2>Benodigdheden</h2>
<ol>
<li>Server waar NodeRED (<a href="https://hub.docker.com/layers/nodered/node-red/latest-16/images/sha256-74230885557bac4b9983fea7316fec4588e29a6997f8f24b2d0b6b050808c3cd?context=explore">Node-engine 16+)</a> op draait. Ik heb het in Docker draaien op mijn Synology NAS. Daarbij moeten er ook nog 2 Pallets installed worden in Node-RED:
<ol>
<li>node-red-contrib-discord-advanced &ndash; Discord nodes voor messages ophalen en verzenden</li>
<li>node-red-contrib-lower &ndash; Om alle payloads lower-case te maken</li>
</ol>
</li>
<li>Discord Application voor de bot: <a href="https://discord.com/developers/applications">https://discord.com/developers/applications</a></li>
</ol>
<h2>Opzetten</h2>
<p>Als NodeRED draait en je de discord-application hebt aangemaakt (en de bot met rechten in je server hebt gehangen(minimaal rechten om te verzenden en lezen)), kan je het volgende importen in NodeRed:</p>
<p><a href="https://github.com/Donixon/vrijmiboGET/blob/main/NodeRED_Clipboard_Export_v1.txt">NodeRED_Clipboard_Export_v1.txt</a></p>

<ol>
<li>Zet je Discord token in de Discord nodes</li>
<li>Vul de commands waar hij op triggert in de node &ldquo;commands&rdquo;</li>
<li>Zet in alle function-nodes &ldquo;msg.channel&rdquo; op het channel-ID waar je wilt dat de message verzonden wordt
<ol>
<li>Channel-ID?: Rechter-muisklik op de channel in Discord en &ldquo;Copy ID&rdquo;</li>
</ol>
</li>
</ol>
<h2>Eerste gebruik</h2>
<p>De VrijMiBot ( :&rsquo;) ) heeft een URL nodig, zodat deze URL gestript kan worden en het page-ID overblijft:</p>
<ul>
<li><a href="https://www.dumpert.nl/item/100018197_5d82f342">https://www.dumpert.nl/item/100018197_5d82f342</a> &gt;&gt; 100018197_5d82f342</li>
</ul>
<p>Dit laatste stukje zet hij achter de Dumpert-API-url:</p>
<ul>
<li><a href="https://api-live.dumpert.nl/mobile_api/json/info/100018197_5d82f342">https://api-live.dumpert.nl/mobile_api/json/info/100018197_5d82f342</a></li>
</ul>
<p>Hierna schrijft hij dit in een bestand weg. Het commando voor het zetten van de Dumpert-URL staat in de commands node:</p>
<ul>
<li>dumpert url &lt;URL van de vrijmibo met plaatjes&gt;
<ul>
<li><strong>dumpert url <a href="https://www.dumpert.nl/item/100018197_5d82f342">https://www.dumpert.nl/item/100018197_5d82f342</a></strong></li>
</ul>
</li>
</ul>
<h2>Commands gebruiken</h2>
<p>Nu alles opgezet is en de Dumpert-URL van de Vrijmibo bekend is, kan je in Discord het command gooien: <strong>alle melkers </strong>. Deze zorgt ervoor dat er in de juiste channel (voorheen neergezet) alle plaatjes worden gepost die in de bovengenoemde URL stonden.</p>
<h2>Uitleg Flow</h2>
<p align="center">
  <img src="https://github.com/Donixon/vrijmiboGET/blob/main/NodeRED_Uitleg_flow_v1.png" />
</p>
<ul>
<li>Geel &ndash; Discord out
<ul>
<li>De Discord-node checkt alle messages die binnenkomen</li>
<li>De lower-case-node zet alle payloads die binnenkomen om in, najah, lower-case letters. Dit zorgt ervoor dat het niet uitmaakt hoe je het command in Discord stuurt:
<ul>
<li>Alle Melkers</li>
<li>AlLe mElKeRs</li>
<li>Hij keurt het nu allemaal goed in de volgende node</li>
</ul>
</li>
<li>In de Switch-node (command-node), staan de values die hij zoekt in de messages die binnenkomen. Mocht er iets overeenkomen (bijv. command: <strong>alle melkers</strong>), zet hij dit door naar het juiste stukje</li>
</ul>
</li>
<li>Groen &ndash; Dumpert URL
<ul>
<li>Deze nodes gaan draaien als het command <strong>dumpert url *** </strong> Hij haalt in de change-node het laatste stukje van de URL eruit en zet dit achter de Dumpet-API-URL.</li>
<li>Schrijft deze API-URL weg in een bestand zodat andere nodes deze URL kunnen gebruiken</li>
</ul>
</li>
<li>Rood &ndash; Alle melkers
<ul>
<li>Als de command <strong>alle melkers </strong>aangeroepen wordt, is de bedoeling dat alle images uit de vrijmibo gestuurd worden. De eerste node haalt de Dumpert-API-URL uit het bestand op.</li>
<li>Hierna zet een switch node msg.payload (is de output van de read-file node) om naar een msg.url zodat de HTTP-request node het begrijpt</li>
<li>HTTP-request gooit een GET naar de opgegeven API-URL in plain-tekst op de msg.payload</li>
<li>Een JSON node format dit correct</li>
<li>Hierna haalt hij de juiste array op, waar alle URL&rsquo;s in staan van de images</li>
<li>De split node zorgt ervoor dat iedere image-URL los op de msg.payload wordt gezet. Zo kunnen er meerdere images verstuurd worden.</li>
<li>Laatste function node zet het channel-ID en veranderd de msg.payload goed, zodat de URL in de payload staat</li>
</ul>
</li>
<li>Paars - Discord in
<ul>
<li>Aangezien er op de Discord-API limieten zitten hoe snel je berichten mag versturen, zit er een delay-node in de flow. Deze node zorgt ervoor dat er 1 message per 1 seconde wordt verstuurd. Dit scheelt weer wat Discord-Bans</li>
<li>De laatste Discord-node verstuurd de Dumpert-image-URL naar de correcte Discord channel</li>
</ul>
</li>
</ul>
