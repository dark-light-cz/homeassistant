# Branka a zvonek

Stav a návrh výměny zvonku u sloupku s ovladačem žumpy. Ceny jsou orientační, s DPH, z obchodů k 21. říjnu 2026.

## Současný stav

Venku je dvoutlačítkový Commax na cihlovém sloupku. Vypadá jako [DRC-4CAN](https://www.alarmax.cz/dverni-kamerova-jednotka-commax-drc-4can), ten je ale jednotlačítkový a menší (asi 12 × 18 cm). Podle dvou jmenovek pod sebou jde o řadu **DRC-2UC**: eloxovaný hliník, zápustný nebo povrchový štít, analogový obraz CVBS, napájení 12 V z monitoru.

Tlačítka:

- horní: POKL DŮM
- dolní: PROŠKOVI

V domě i v garzonce je vnitřní videotelefon **Commax CDV-70P**. Každý má vlastní přívod 230 V. Kameru venku živí monitor po čtyřech vodičích: červená zvuk, modrá zem, žlutá +12 V, bílá obraz. DRC-2UC má kvůli dvěma bytům kolem deseti žil.

Zámek branky spíná zdroj v technické místnosti, **Urmet Domus Sch.9000/230**:

- vstup 230 V, 18 VA, 50/60 Hz
- výstup **12 V střídavých**, 1,1 A

Tlačítko otevření na monitoru jen spojí obvod zámku. V krabici za monitorem jsou k tomu nejspíš modrý a žlutozelený vodič. Napájení žumpy a zvonku je oddělené a tak má zůstat.

Vedení od domu ke sloupku měří asi **45 m**. Ve zdi za monitorem je datový kabel s barvami párů (oranžová, zelená, modrá, hnědá), použitý jako analog. Část tras pod venkovním Commaxem je zaházená maltou. Je možné, že od každého monitoru vede vlastní kabel jen ke sloupku a končí na zdi u dveří, mimo domácí ethernet.

Wi-Fi je společná pro dům i garzonku. V domě je pokrytí Zigbee.

### Poruchy

Jedna vnitřní jednotka má mrtvou napájecí část. Druhá má částečný obraz a špatný zvuk, odemčení branky většinou funguje. To sedí na délku kabelu: Commax u průřezu 0,5 mm udává rozumný obraz zhruba do 28 m, Cat5 má kolem 0,51 mm. Zámek je samostatný obvod, takže mu slabý videosignál nevadí. Stejný nový Commax na tomhle kabelu by dopadl stejně.

## Návrh

Zvonek, obraz i otevření jedou přes Home Assistant. Když HA neběží, tablet nezazvoní a branka se z něj neotevře.

Panel 2N IP Verso 2.0 s Indoor View by totéž uměl i bez HA, sestava se zvonkem a jedním panelem ale vycházela zhruba na 70 000 Kč. Tahle cesta je záměrně levnější a počítá s tím, že HA běží.

### Venku

**Reolink Video Doorbell PoE (D340P).** 5 Mpx, noční vidění, mikrofon, reproduktor, jedno tlačítko, PoE 802.3af. Do HA patří oficiální integrací Reolink, PoE varianta nemá cloud. Zvonění se bere z čidla stisku tlačítka, ne z detekce osoby.

Tlačítko na Reolinku volá dům. Pro Proškovy je vedle něj obyčejné vodotěsné tlačítko.

Přibalený zvonek na 433 MHz se nepoužívá. Reolink udává 100 m na volném prostranství. Cihlový sloupek a zdi domu to nesplní a zvonek by stejně reagoval jen na tlačítko Reolinku.

### Napájení

U vnitřního konce kabelu stačí obyčejný aktivní injektor **802.3af** (nebo 802.3at). Zvonek bere asi 7 až 12 W. Norma počítá se 100 m, úbytek na 45 m měděného Cat5e je kolem 1 až 2 V.

Urmet zvonek nenapájí. Pasivní injektor, který na kabel pustí natvrdo 24 V, taky ne.

### Síť, když kabel končí u dveří

Na zeď u dveří přijde injektor. Dlouhý kabel jde do zdířky PoE, datová zdířka do **MikroTik mAP lite (RBmAPL-2nD)**.

mAP lite se napájí přiloženým microUSB adaptérem. Do zdířky s 48 V se nezapojuje, ta patří jen kabelu do sloupku. Zařízení 802.3af sice snese, datová zdířka injektoru ale napájení nedává.

Wi-Fi režim je **station-pseudobridge-clone**. `ether1` a `wlan1` jsou v jednom bridge, DHCP server v MikroTiku je vypnutý. Domácí router pak dá adresu přímo zvonku. Režim routeru nebo CPE by kameru schoval do vlastní podsítě. 2,4 GHz na jeden stream 4 až 6 Mb/s stačí, ethernet zvonku je stejně jen 100 Mb/s.

Ze dvou kabelů (dům a garzonka) se pro kameru použije ten, u kterého je zásuvka a lepší Wi-Fi. Druhý může nést tlačítko pro Proškovy, pokud v něm zbude volný pár.

### Zámek

**Shelly Plus 1** v krabici po monitoru, kde je 230 V i vodiče od Urmetu. Napájení Shelly je ze sítě. Beznapěťový kontakt `I`/`O` nahradí tlačítko otevření na starém monitoru a dál spíná 12 V střídavých z Urmetu. Před jakoukoli prací na tom obvodu se Urmet vytáhne ze zásuvky.

Zámek je cívka. Přes kontakt patří RC odrušovač, jinak kontakt časem lepí.

Vstup `SW` se při napájení ze 230 V váže na fázi. Venkovní tlačítko se na něj nezapojuje.

### Druhé tlačítko

Volný pár v kabelu vede jen jako kontakt. V krabici u monitoru ho čte malé ESP na USB adaptéru a do HA hlásí přes ESPHome. Na 45 m je to bezpečné nízké napětí.

Když volný pár ve sloupku nebude, zvonek do domu funguje i tak. Bezdrátové Zigbee tlačítko na 45 m přes cihlu není spolehlivá náhrada.

### Vnitřní panely

Starý Android tablet nebo telefon, v domě i v garzonce. Fully Kiosk nebo aplikace Home Assistant na celou obrazovku, pořád na nabíječce. Displej může být zhasnutý.

Po stisku tlačítka automatizace rozsvítí příslušný tablet, přehraje tón a ukáže obraz. Tón domu a garzonky je jiný. Otevření branky je vypínač, který Shelly sepne na pár vteřin. Hovor jde z karty kamery přes integraci Reolink.

## Home Assistant

- integrace Reolink, lokálně, RTSP zapnuté v kameře
- Shelly Plus 1 jako vypínač zámku, pulz na dobu otevření
- ESPHome binary sensor pro tlačítko Proškovi
- Fully Kiosk: rozsvícení obrazovky a zvuk podle toho, které tlačítko přišlo

## Než se to koupí

Na sloupku stačí sundat čelo Commaxu, maltu nechat být a vyfotit svorkovnici.

1. Kolik barev datového kabelu na svorkách opravdu je a jestli jsou spoje po cestě.
2. Který kabel končí u dveří domu a který v garzonce.
3. Jestli modrý a žlutozelený vodič opravdu vedou na Urmet a zámek.

## Odhad ceny

| Kus | Cena |
| --- | --- |
| Reolink D340P | 2 800 Kč |
| PoE injektor 802.3af | 500 Kč |
| Shelly Plus 1 | 500 Kč |
| Tlačítko a ESP | 400 Kč |
| MikroTik mAP lite | 774 Kč |
| Tablety | už jsou |
| **Celkem** | **zhruba 5 000 Kč** |

mAP lite odpadne, pokud kabel ze sloupku ústí přímo u domácí sítě.
