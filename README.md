# Show me love

In de deze manual laten we 2 NodeMCU's met elkaar communiceren via Adafruit IO. Node 1 heeft een drukknop en stuurt een signaal naar een feed. Node 2 leest die feed uit en laat een ledstrip roze branden. Als extra challenge stuurt node 2 met zijn FLASH knop ook iets terug, waarna het blauwe ledje op node 1 gaat branden.

Ik heb de opdracht alleen gedaan. Daarom heb ik twee Adafruit IO accounts gebruikt: **USSCallister** voor node 1 en **SESCallister** voor node 2.

## Benodigdheden

* 2x NodeMCU 
* Drukknop module met de pinnen VCC, OUT en GND
* NeoPixel ledstrip 
* Jumper kabels
* 2x USB kabel
* Arduino IDE met de libraries **Adafruit IO Arduino** en **Adafruit NeoPixel**
* Een hotspot op 2,4 GHz
* Twee Adafruit IO accounts

## Zo werkt het

| Richting | Zender | Feed | Ontvanger |
|---|---|---|---|
| Heen | Groene knop op node 1 | `ada-love` | Ledstrip op node 2 |
| Terug | FLASH knop op node 2 | `love-back` | Ingebouwde led op node 1 |

Beide feeds zijn van USSCallister en gedeeld met SESCallister met Read + Write rechten.

## Stap 1: Account en feed aanmaken

1. Ga naar io.adafruit.com en maak een account aan via **Sign Up**.

![Welcome pagina Adafruit IO](fotos/Screenshot%202026-09-30%20103503.png)
![Sign Up pagina](fotos/Screenshot%202026-09-30%20103614.png)

2. Klik bovenaan op **IO**.

![IO in de menubalk](fotos/Screenshot%202026-09-30%20103903.png)

3. Ga naar **Feeds** en klik op **New Feed**.

![Feedoverzicht voor het aanmaken](fotos/Screenshot%202026-09-30%20103943.png)

4. Geef de feed een naam. Ik koos **Ada Love**.

![Nieuwe feed aanmaken](fotos/Screenshot%202026-09-30%20104004.png)

5. Adafruit maakt van de naam automatisch een **Feed Key**: `ada-love`.

![Feedoverzicht met Ada Love](fotos/Screenshot%202026-09-30%20104019.png)

> **Let op:** de naam van de feed (Ada Love) en de Feed Key (`ada-love`) zijn niet hetzelfde. In de code gebruik je altijd de Feed Key.

## Stap 2: Drukknop aansluiten op node 1

| Knop | NodeMCU |
|---|---|
| OUT | D0 |
| GND | G |
| VCC | 3V |

![Drukknop met labels](fotos/Foto1.jpg)
![Drukknop schuin](fotos/Foto7.jpg)
![Bedrading node 1](fotos/Foto9.jpg)
![Node 1 met knop](fotos/Foto6.jpg)

### Fout 1: Andere drukknop dan in de opdracht

**Probleem:** de opdracht beschrijft een knop met de pinnen S, min en plus. Mijn knop heeft VCC, OUT en GND.

**Oplossing:** het is dezelfde soort knop met andere namen. OUT is het signaal (S) en gaat naar D0. GND gaat naar G. VCC is de plus en gaat naar 3V. Gebruik 3V en niet VIN, want de pinnen van de NodeMCU kunnen maar 3,3 V aan.

## Stap 3: Voorbeeld openen en de knop pin aanpassen

Open **File > Examples > Adafruit IO Arduino > adafruitio_06_digital_in**.

![Examples menu met digital_in](fotos/Screenshot%202026-09-30%20104258.png)

Verander `BUTTON_PIN` van `5` naar `D0`.

Voor:

![BUTTON_PIN 5](fotos/Screenshot%202026-09-30%20105135.png)

Na:

![BUTTON_PIN D0](fotos/Screenshot%202026-09-30%20105150.png)

Het voorbeeld heeft twee tabbladen. Je wifi en Adafruit gegevens vul je in bij `config.h`.

![Tabbladen van het voorbeeld](fotos/Screenshot%202026-09-30%20105208.png)

## Stap 4: config.h invullen

Vul de naam en het wachtwoord van je hotspot in. Let op: de naam is hoofdlettergevoelig.

Voor:

![Wifi standaard](fotos/Screenshot%202026-09-30%20105309.png)

Na:

![Wifi ingevuld](fotos/Screenshot%202026-09-30%20105335.png)

Vul daarna je username en je **Adafruit IO Key** in.

![Username en key standaard](fotos/Screenshot%202026-09-30%20105406.png)

### Fout 2: Feed Key ingevuld in plaats van de Adafruit IO Key

**Probleem:** bij `IO_KEY` vulde ik `ada-love` in. Dat is de Feed Key en niet de key van mijn account.

![Feed Key in het overzicht](fotos/Screenshot%202026-09-30%20105441.png)
![Foute key in config.h](fotos/Screenshot%202026-09-30%20105459.png)

Het uploaden ging goed, maar in de serial monitor verschenen alleen puntjes. Als ik op de knop drukte gebeurde er niets.

![Uploaden met de foute key](fotos/Screenshot%202026-09-30%20105626.png)
![Serial monitor met alleen puntjes](fotos/Screenshot%202026-09-30%20105711.png)

**Oorzaak:** het voorbeeld wacht in `setup()` tot de verbinding met Adafruit IO gelukt is. Met een verkeerde key lukt dat nooit, dus de code in `loop()` die de knop uitleest wordt nooit uitgevoerd.

**Oplossing:** de juiste key vind je via het gele sleutelicoon rechtsboven op io.adafruit.com. Deze key begint altijd met `aio_`.

![Feedoverzicht](fotos/Screenshot%202026-09-30%20105856.png)
![Sleutelicoon My Key](fotos/Screenshot%202026-09-30%20105935.png)
![Juiste key in config.h](fotos/Screenshot%202026-09-30%201100081.png)

> **Tip:** zet je echte key nooit zichtbaar op GitHub. Dan kan iedereen je account gebruiken. Blur hem of genereer na afloop een nieuwe key.

## Stap 5: Uploaden en testen

Upload de code en open de serial monitor op **115200 baud**. Druk een paar keer op de knop. Je ziet nu `sending button` met een 1 of een 0.

![Knop indrukken](fotos/Foto4.jpg)
![Serial monitor node 1](fotos/Screenshot%202026-09-30%20110111.png)

## Stap 6: Data controleren in Adafruit IO

### Fout 3: De data kwam in een feed `digital` terecht

**Probleem:** de verbinding werkte, maar mijn feed Ada Love bleef leeg. Er was ineens een nieuwe feed `digital` waarin de 0'en en 1'en wel binnenkwamen.

![Lege feed Ada Love](fotos/Screenshot%202026-09-30%20110156.png)
![Data in de feed digital](fotos/Screenshot%202026-09-30%20110213.png)

**Oorzaak:** in het voorbeeld staat de feednaam `digital`. Als de code naar een feed stuurt die nog niet bestaat, maakt Adafruit IO die automatisch aan.

**Oplossing:** verander de feednaam naar je eigen Feed Key.

Voor:

![Feed digital in de code](fotos/Screenshot%202026-09-30%20110358.png)

Na:

![Feed ada-love in de code](fotos/Screenshot%202026-09-30%20110414.png)

Nu komt de data in de goede feed binnen:

![Data in Ada Love](fotos/Screenshot%202026-09-30%20110535.png)

De feed `digital` kun je daarna verwijderen.

## Stap 7: De feed delen met een ander account

1. Open de feed Ada Love en klik op **Sharing**.

![Not shared yet](fotos/Screenshot%202026-09-30%20111048.png)

2. Vul bij **Email or Username** het andere account in en kies **Read + Write**. Met alleen Read kan de ander niet naar je feed schrijven.

![Sharing Settings](fotos/Screenshot%202026-09-30%20111133.png)

3. Klik op **Send Invitation**. De uitnodiging staat nu op Pending.

![Uitnodiging Pending](fotos/Screenshot%202026-09-30%20112236.png)

4. Het andere account krijgt een mail en klikt op **Approve**.

![Review Sharing Invitation](fotos/Screenshot%202026-09-30%20112904.png)

### Fout 4: De uitnodiging kon niet geaccepteerd worden

**Probleem:** de link in de mail stuurde me steeds door naar de welcome page of naar mijn publieke pagina.

**Oorzaak 1:** ik opende de link in een browser waarin ik nog ingelogd was als USSCallister. De uitnodiging was voor SESCallister.

**Oorzaak 2:** het account SESCallister was wel aangemaakt, maar Adafruit IO was nog niet geactiveerd. Daardoor had het account geen Feeds menu en kon de uitnodiging nergens landen.

![IO menu zonder feeds](fotos/Screenshot%202026-09-30%20112414.png)

**Oplossing:**
1. Open een incognitovenster en log in met het tweede account.
2. Ga naar io.adafruit.com en klik onder **The Internet of Things for Everyone** op **Get Started**. Pas dan wordt Adafruit IO geactiveerd.
3. Open de link opnieuw of ga naar **Privacy & Sharing** en klik op **Approve**.

## Stap 8: De gedeelde feed uitlezen op node 2

Open **File > Examples > Adafruit IO Arduino > adafruitio_21_feed_read**.

![Examples menu met feed_read](fotos/Screenshot%202026-09-30%20113105.png)

Vul bij `FEED_OWNER` de username in van de eigenaar van de feed.

Voor:

![FEED_OWNER standaard](fotos/Screenshot%202026-09-30%20113142.png)

Na:

![FEED_OWNER ingevuld](fotos/Screenshot%202026-09-30%20113213.png)

Vul bij de feed de Feed Key in.

Voor:

![Feed standaard](fotos/Screenshot%202026-09-30%20113221.png)

Na:

![Feed ada-love](fotos/Screenshot%202026-09-30%20113249.png)

### Fout 5: config.h van het nieuwe voorbeeld niet ingevuld

**Probleem:** node 2 liet alleen puntjes zien en ontving niets als ik op de knop van node 1 drukte.

![Serial monitor node 2 met puntjes](fotos/Screenshot%202026-09-30%20113818.png)

**Oorzaak:** elk voorbeeld heeft zijn eigen `config.h`. In het nieuwe voorbeeld stonden nog de standaardwaardes.

**Oplossing:** vul in `config.h` de gegevens van het **tweede account** in: de username en de key van SESCallister. USSCallister staat alleen bij `FEED_OWNER`.

Voor:

![config.h standaard](fotos/Screenshot%202026-09-30%20113831.png)

Na:

![config.h ingevuld voor node 2](fotos/Screenshot%202026-09-30%201139321.png)

Nu ontvangt node 2 de waardes van node 1:

![Node 2 ontvangt data](fotos/Screenshot%202026-09-30%20114032.png)

## Stap 9: De ledstrip laten reageren

| Ledstrip | NodeMCU |
|---|---|
| DIN | D1 |
| GND | G |
| 5V | 3V |

![Bedrading node 2](fotos/Foto5.jpg)

Voeg bovenaan de NeoPixel library en de ledstrip toe:

```cpp
#include <Adafruit_NeoPixel.h>
Adafruit_NeoPixel pixels(16, D1, NEO_GRB + NEO_KHZ800);
```

![NeoPixel include](fotos/Screenshot%202026-09-30%20114645.png)

Start de ledstrip in `setup()`:

```cpp
pixels.begin();
```

![pixels.begin in setup](fotos/Screenshot%202026-09-30%20114705.png)

Laat de strip in `handleMessage` reageren op de ontvangen waarde:

```cpp
if (data->toInt() == 1) {
  pixels.fill(pixels.Color(255, 0, 100));
} else {
  pixels.clear();
}
pixels.show();
```

![handleMessage met ledstrip](fotos/Screenshot%202026-09-30%20114750.png)
![Serial monitor node 2](fotos/Screenshot%202026-09-30%20114924.png)

Resultaat: zolang je de knop van node 1 indrukt, brandt de ledstrip op node 2 roze. Er zit een kleine vertraging in, omdat alles via Adafruit IO gaat.

![Ledstrip uit](fotos/Foto3.jpg)
![Ledstrip aan](fotos/Foto12.jpg)

## Extra challenge: heen en weer

Ik had maar 1 drukknop en geen tweede output. Daarom gebruik ik op node 2 de ingebouwde **FLASH knop** (D3) en op node 1 het ingebouwde **blauwe ledje** (`LED_BUILTIN`).

![FLASH knop op node 2](fotos/Foto8.jpg)

### A: Tweede feed aanmaken en delen

Maak als USSCallister een feed **Love back** aan (Feed Key `love-back`) en deel die met SESCallister met Read + Write.

![Feed Love back aanmaken](fotos/Screenshot%202026-09-30%20115212.png)

### B: Node 2 laat de FLASH knop versturen

Bovenaan, bij de andere feed:

```cpp
AdafruitIO_Feed *backFeed = io.feed("love-back", "USSCallister");
bool vorige = HIGH;
```

![backFeed op node 2](fotos/Screenshot%202026-09-30%20115427.png)

In `setup()`:

```cpp
pinMode(D3, INPUT_PULLUP);
```

![pinMode D3](fotos/Screenshot%202026-09-30%20115450.png)

In `loop()`, onder `io.run();`:

```cpp
bool nu = digitalRead(D3);
if (nu != vorige) {
  backFeed->save(nu == LOW ? 1 : 0);
  vorige = nu;
}
```

![loop met FLASH knop](fotos/Screenshot%202026-09-30%20115534.png)

De code stuurt alleen iets als de knop verandert. Zo blijf je onder het limiet van 30 berichten per minuut van een gratis account.

> **Opmerking:** de FLASH knop geeft 0 als je hem indrukt en 1 als je hem loslaat. Daarom draait `nu == LOW ? 1 : 0` de waarde om. Houd de FLASH knop niet ingedrukt tijdens het opstarten, want dan gaat de NodeMCU in uploadmodus.

### C: Node 1 laat het blauwe ledje reageren

Bovenaan, onder de feed `ada-love`:

```cpp
AdafruitIO_Feed *backFeed = io.feed("love-back");
```

![backFeed op node 1](fotos/Screenshot%202026-09-30%20115840.png)

In `setup()`, direct na `io.connect();`:

```cpp
pinMode(LED_BUILTIN, OUTPUT);
digitalWrite(LED_BUILTIN, HIGH);
backFeed->onMessage(handleBack);
```

![setup van node 1](fotos/Screenshot%202026-09-30%20115849.png)

Onderaan een nieuwe functie:

```cpp
void handleBack(AdafruitIO_Data *data) {
  if (data->toInt() == 1) {
    digitalWrite(LED_BUILTIN, LOW);
  } else {
    digitalWrite(LED_BUILTIN, HIGH);
  }
}
```

![handleBack functie](fotos/Screenshot%202026-09-30%20115907.png)

> **Opmerking:** het ingebouwde ledje werkt omgekeerd. `LOW` is aan en `HIGH` is uit. Dat is geen fout, zo is het ledje op de NodeMCU aangesloten.

Resultaat: druk je op de FLASH knop van node 2, dan gaat het blauwe ledje op node 1 branden.

![FLASH knop indrukken](fotos/Foto10.jpg)

![Blauw ledje aan](fotos/Foto11.jpg)

## Bronnen

* Opdracht Show me love, DfETsr IoT, HvA
* Adafruit, Sharing a Feed: https://learn.adafruit.com/adafruit-io-basics-feeds/sharing-a-feed
* Adafruit IO Arduino library en de voorbeelden 06, 20 en 21: https://github.com/adafruit/Adafruit_IO_Arduino
* Adafruit NeoPixel library: https://github.com/adafruit/Adafruit_NeoPixel
* Random Nerd Tutorials, ESP8266 Pinout Reference (FLASH knop op D3 en ingebouwde led): https://randomnerdtutorials.com/esp8266-pinout-reference-gpios/
* Claude (Anthropic), hulp met troubleshooting en code
