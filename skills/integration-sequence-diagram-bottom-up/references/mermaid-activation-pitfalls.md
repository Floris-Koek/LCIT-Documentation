# Mermaid activation-shorthand: een concrete valkuil (en de fix)

Dit document beschrijft één specifieke, makkelijk te maken fout met de `+`/`-` activatie-shorthand uit `standards.md` sectie 3.5, die een renderfout oplevert zodra je `alt`/`else` combineert met een activatie die vóór het blok is gestart. Lees dit vóórdat je een diagram opstelt met een `alt`/`else`-blok waarin de "buitenste" participant (degene die de hele interactie startte) antwoordt in meerdere branches.

## De onderliggende regel

- `A ->>+ B: <actie>` activeert **B**, de ontvanger van het bericht.
- `B -->>- A: <actie>` deactiveert **B**, de VERZENDER van dit antwoord — niet A. Het `-`-teken betekent "deactiveer wie dit bericht verzendt", niet "deactiveer de naam die na de pijl staat."

Dit is precies zoals je het intuïtief zou verwachten: A vraagt iets aan B, B wordt "aan het werk gezet" (actief), B antwoordt en is dan weer vrij (inactief).

## De valkuil

Mermaid bouwt de activatie-balken op door de berichten sequentieel te verwerken zoals ze in de code staan — inclusief de inhoud van ZOWEL de `if`- als de `else`-tak van een `alt`-blok. Het ziet een `alt`/`else` dus niet als "precies één van deze paden gebeurt" voor de activatie-boekhouding; het verwerkt gewoon alle regels in volgorde.

Gevolg: als een participant vóór het `alt`-blok is geactiveerd, en zowel de `if`-tak als de `else`-tak eindigen met een `-->>-` die diezelfde participant deactiveert, dan sluit de eerste tak die activatie al af. Als de tweede tak vervolgens ook proberen te deactiveren, is er niets meer om te sluiten:

```
Error: Trying to inactivate an inactive participant (<naam>)
```

### Voorbeeld van de foute versie

```
bartrack ->>+ bartrack_ea: POST /v1/create-order

alt remarks valid
    bartrack_ea ->>+ order_pa: POST /internal/v1/orders/create-order
    order_pa    -->>- bartrack_ea: 200 ok
    bartrack_ea -->>- bartrack: 200 ok (order created)   %% deactiveert bartrack_ea
else remarks invalid
    bartrack_ea -->>- bartrack: 400 Bad Request           %% probeert bartrack_ea NÓG een keer te deactiveren -> fout
end
```

### De fix

Gebruik binnen de branches gewone (niet-deactiverende) pijlen voor het branch-specifieke antwoord, en plaats één losse `deactivate <participant>`-regel na het `end` van het `alt`-blok — die regel sluit de activatie exact één keer, onafhankelijk van welke branch daadwerkelijk "gebeurde":

```
bartrack ->>+ bartrack_ea: POST /v1/create-order

alt remarks valid
    bartrack_ea ->>+ order_pa: POST /internal/v1/orders/create-order
    order_pa    -->>- bartrack_ea: 200 ok
    bartrack_ea -->> bartrack: 200 ok (order created)
else remarks invalid
    bartrack_ea -->> bartrack: 400 Bad Request
end

deactivate bartrack_ea
```

## Wanneer dit GEEN probleem is

Een activatie die volledig **binnen** één enkele branch wordt geopend én gesloten — dus zowel de `+` als de bijbehorende `-` staan in dezelfde `if`-tak, of allebei in dezelfde `else`-tak, zonder dat de activatie van buiten het blok afkomstig is — heeft deze fix niet nodig. Elke branch beheert dan zijn eigen, volledig onafhankelijke activatie-paar; er is geen gedeelde toestand om over te struikelen.

## Controle vóór opleveren

Voor elke participant die vóór een `alt`/`else`-blok wordt geactiveerd met `+`:

1. Tel hoe vaak die participant met `-` wordt gedeactiveerd binnen de branches van dat blok.
2. Is dat aantal `> 1`? Vervang alle `-->>-`/`->>-`-deactivaties binnen de branches door gewone pijlen, en voeg één losse `deactivate <participant>` toe direct na het `end`.
3. Is dat aantal exact `1`, en gebeurt die deactivatie in elke mogelijke branch (dus niet alleen in de `if`-tak terwijl de `else`-tak "doorloopt" zonder antwoord)? Dan is de huidige vorm meestal al correct — maar controleer of ELKE branch daadwerkelijk een pad naar die ene deactivatie heeft, anders loop je tegen het omgekeerde probleem aan (een activatie die nooit wordt gesloten).

Deze check geldt niet alleen voor `alt`/`else` — hetzelfde patroon (gedeelde activatie, meerdere potentiële sluitpunten) kan ook optreden bij `opt`, al is dat zeldzamer omdat een `opt` maar één pad heeft naast "niets doen".
