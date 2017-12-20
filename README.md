![Statens Kartverk](https://github.com/kartverket/etinglysing-systembeskrivelse/blob/master/doc/kartverket.png)


## Innholdsfortegnelse
1. [Domenemodell](#1-domenemodell)	
2. [InformasjonService](#2-informasjonservice)
    + [Transfer](#transfer)	
    + [findHeftelser](#findheftelser)	
    + [findTidligereHeftelser](#findtidligereheftelser)	
    + [findOverdragelserAvRegisterenhetsrett](#findoverdragelserAvregisterenhetsrett)
    + [findTidligereOverdragelserAvRegisterenhetsrett](#findtidligereoverdragelserAvregisterenhetsrett)	
    + [findRegisterenhetsendringer](#findregisterenhetsendringer)
    + [findOverdragelserAvRegisterenhetsrettForPerson](#findoverdragelseravregisterenhetsrettforperson)	
    + [findRettigheterForPerson](#findrettigheterforperson)
    + [findRettigheterForRegisterenhet](#findrettigheterforregisterenhet)
    + [findRettsstiftelserForDokument](#findrettsstiftelserfordokument)
    + [findRettsstiftelse](#findrettsstiftelse)
    + [findBorettslagsandelerForBorettslag](#findborettslagsandelerforborettslag)	
3. [GrunnboksutskriftService](#3-grunnboksutskriftservice)	
    + [ubekreftetGrunnboksutskriftPdf](#ubekreftetgrunnboksutskriftpdf)
    + [ubekreftetHistoriskGrunnboksutskriftPdf](#ubekreftethistoriskgrunnboksutskriftpdf)	
4. [PersonService](#4-personservice)	
    + [findPersoner](#findpersoner)
5. [EndringsloggService](#5-endringsloggservice)	
6. [NedlastingService](#6-nedlastingservice)	
7. [IdentService](#7-identservice)	
8. [StoreService](#8-storeservice)	


## 1. Domenemodell
Dataene for grunnboken er modellert med boblearkitektur. En boblemodell er en teknisk implementasjon av domenemodellen, der domenet er oppdelt i bobler som representerer de viktigste domeneklassene. Bobleobjektene er løst koblet ved at de refererer til hverandre via koblinger på id-nivå og ikke på objekt-nivå. Dokumentasjon av denne modellen (UML-modell) er tilgjengelig via denne linken: https://etgltest.grunnbok.no/grunnbok/modell/grunnbok-domene-v2-modell/index.html 

I den delen av modellen som omhandler dokumenter og rettsstiftelser er det kun data som er endelig registrert i grunnboken som inngår.

## 2. InformasjonService
### Transfer
I tråd med vår boblearkitektur, så returnerer søketjenestene i utgangspunktet bare id-ene for det man ber om. Selve dataene må man deretter hente med en egen tjeneste, StoreService. Men siden InformasjonService, i motsetning til alle andre bobletjenester, tar hensyn til foreløpige registreringer, så kan man ikke benytte StoreService for å hente ut data, da den ikke vil se de foreløpige registreringene. Derfor legger vi med en såkalt transfer som inneholder dataene for de boblene vi anser som relevante for å forstå og presentere svaret, slik de er ifølge de foreløpige registreringene. Vi legger også med informasjon om hvilke bobler som har blitt påvirket av foreløpige registreringer.

Der det står at tjenestene bygger en vanlig transfer, så består denne i tillegg til de rettsstiftelsene som allerede har blitt listet for hver tjeneste, av dokumentene for alle disse rettsstiftelsene, alle dokumenter som ellers refereres til fra disse rettsstiftelsene, alle registerenhetsrettene som refereres til fra disse rettsstiftelsene (hefter i eller realkoblet), alle registerenheter som refereres til direkte fra rettsstiftelsene, alle borettslags som refereres til direkte fra rettsstiftelsene, alle personer som refereres til direkte fra rettsstiftelsene, alle seksjonssameieandeler som refereres til direkte fra rettsstiftelsene, alle rettsstiftelser som er referert til fra rettsstiftelser som allerede er funnet, dokumentene til disse rettsstiftelsene, alle registerenhetsrettsandelene de opprinnelige rettsstiftelsene hefter i, alle personer som er rettighetshaver til disse andelene, alle registerenheter som er realkoblet til disse andelene, alle registerenhetsrettene disse andelene er i, alle registerenhetene for disse registerenhetsrettene, borettslag for alle borettslagsandeler funnet så langt, adresser og personer for alle borettslag funnet så langt, kommuner for alle matrikkelenheter og adresser funnet så langt, og alle koder brukt i alt som er funnet så langt.

### findHeftelser
#### Kort beskrivelse av tjenesten:
Tjenesten gir oversikt over alle tinglyste heftelser på en registerenhet, samt alle tinglyste heftelser i disse heftelsene, rettsendringer som endrer disse heftelsene og anmerkninger som anmerker disse heftelsene. Denne tjenesten leverer også foreløpig registrerte heftelser på registerenheten det søkes på.

Heftelsene er representert ved klassen Heftelser (med alle subklasser) i domenemodellen. 

Søkeparameter for denne tjenesten er id for registerenheten man skal finne heftelser for.

#### Mer om foreløpig registreringer (teknisk):
Registerenheten trenger ikke være endelig registrert.

Tjenesten finner først alle foreløpig registrerte meldinger som systemet har registrert at påvirker denne registerenheten og tar inn de beregnede effektene fra disse. Deretter finner den i den predikerte tilstanden hvilke registerenhetsretter som finnes på registerenheten. Så finner den alle andeler i registerenhetsrettene som i henhold til prediksjonen er aktive. Søket avsluttes så med å finne alle heftelser som er predikert å hefte i disse andelene.

Det som returneres er id-ene til disse heftelsene, samt den predikerte tilstanden til heftelsene, alle heftelser som hefter i dem, alle anmerkninger på alt dette, alle rettsendringer på alt dette igjen rekursivt, og så en vanlig transfer basert på dette.

### findTidligereHeftelser
#### Kort beskrivelse av tjenesten:
Tjenesten gir oversikt over alle tidligere tinglyste (historiske) heftelser på en registerenhet, samt alle tidligere tinglyste heftelser i disse heftelsene, tidligere rettsendringer som endrer disse heftelsene, og tidligere anmerkninger som anmerket disse heftelsene. 
Denne tjenesten leverer også heftelser som ikke har blitt historiske, men hvor det er foreløpig registrert en sletting på heftelsen, som vil gjøre heftelsen historisk når den foreløpige registrerte sletting blir godkjent. Det samme gjelder heftelser som ikke har blitt slettet, men som har blitt historiske som følge av at registerenheten heftelsen hefter i er historisk/utgått.

Heftelsene er representert ved klassen Heftelser (med alle subklasser) i domenemodellen.  

Søkeparameter for denne tjenesten er id for registerenheten man skal finne tidligere (historiske) heftelser for.

#### Mer om foreløpig registreringer (teknisk):
Registerenheten trenger ikke være endelig registrert.

Tjenesten finner først alle foreløpig registrerte meldinger som systemet har registrert at påvirker denne registerenheten og tar inn de beregnede effektene fra disse. Deretter finner den i den predikerte tilstanden hvilke registerenhetsretter som finnes på registerenheten. Så finner den alle andeler som i henhold til prediksjonen er i registerenhetsrettene, både aktive og historiske. Søket avsluttes så med å finne alle heftelser som er predikert å hefte historisk i disse andelene.

Det som returneres er id-ene til disse heftelsene, samt den predikerte tilstanden til heftelsene, alle heftelser som hefter i dem, alle anmerkninger på alt dette, alle rettsendringer på alt dette igjen rekursivt, og så en vanlig transfer basert på dette.

### findOverdragelserAvRegisterenhetsrett
#### Kort beskrivelse av tjenesten:
Tjenesten gir oversikt over alle tinglyste (og fortsatt gjeldende) overdragelser av registerenhetsretter, samt alle rettsendringer som endrer disse overdragelsene, og anmerkninger som anmerker disse overdragelsene. 

Førstegangsregistrering av registerenhetsretter (eksempelvis ved førstegangsregistrering av rettighetshavere til registerenhetsrett etter opprettelse av matrikkelenhet fra tidligere uregistrert grunn), og overdragelser der en registerenhet har fått andel (og fortsatt har andel) i et realsameie/jordsameie blir også levert i denne tjenesten. 

Også alle foreløpig registrerte overdragelser vil bli levert i denne tjenesten.

Overdragelsene er representert ved klassen OverdragelseAvRegisterenhetsrett (med alle subklasser) i domenemodellen. 

Søkeparameter for denne tjenesten er id for registerenheten man skal finne overdragelser for.

#### Mer om foreløpig registreringer (teknisk):
Registerenheten trenger ikke være endelig registrert.

Tjenesten finner først alle foreløpig registrerte meldinger som systemet har registrert at påvirker denne registerenheten og tar inn de beregnede effektene fra disse. Deretter finner den i den predikerte tilstanden alle overdragelser hvor noen av de nye andelene er aktive andeler i registerenhetsretter i denne registerenheten, og alle overdragelser hvor noen av de nye andelene er aktive andeler realkoblet til denne registerenheten.

Det som returneres er id-ene til disse overdragelsene, samt den predikerte tilstanden til overdragelsene, alle anmerkninger på disse, alle rettsendringer på alt dette igjen rekursivt, og så en vanlig transfer basert på dette.

### findTidligereOverdragelserAvRegisterenhetsrett
#### Kort beskrivelse av tjenesten:
Tjenesten gir oversikt over alle tinglyste (men ikke lenger gjeldende) overdragelser av registerenhetsretter (for eksempel fordi registerenhetsretten er overdratt videre), samt alle rettsendringer som endrer disse overdragelsene, og anmerkninger som anmerker disse overdragelsene. 

Førstegangsregistrering av registerenhetsretter (som ikke lenger er gjeldende), og overdragelser der en registerenhet har fått andel (men ikke lenger har samme andel) i et realsameie/jordsameie blir også levert i denne tjenesten. 

Også overdragelser som ikke lenger er gjeldende, som følge av en foreløpig registrert overdragelse på registerenhetsretten, blir levert i denne tjenesten. Det samme gjelder overdragelser som ikke lenger er gjeldende som følge av at registerenhetsretten er historisk/utgått.   

Overdragelsene er representert ved klassen OverdragelseAvRegisterenhetsrett (med alle subklasser) i domenemodellen.

Søkeparameter for denne tjenesten er id for registerenheten man skal finne tidligere (ikke lenger gjeldende) overdragelser for.

#### Mer om foreløpig registreringer (teknisk):
Registerenheten trenger ikke være endelig registrert.

Tjenesten finner først alle foreløpig registrerte meldinger som systemet har registrert at påvirker denne registerenheten og tar inn de beregnede effektene fra disse. Deretter finner den i den predikerte tilstanden alle overdragelser hvor noen av de nye andelene er historiske andeler i registerenhetsretter i denne registerenheten, og alle overdragelser hvor noen av de nye andelene er historiske andeler realkoblet til denne registerenheten.

Det som returneres er id-ene til disse overdragelsene, samt den predikerte tilstanden til overdragelsene, alle anmerkninger på disse, alle rettsendringer på alt dette igjen rekursivt, og så en vanlig transfer basert på dette.

### findRegisterenhetsendringer
#### Kort beskrivelse av tjenesten:
Tjenesten gir oversikt over alle tinglyste registerenhetsendringer på en registerenhet, samt alle rettsendringer som endrer disse registerenhetsendringene. Tjenesten leverer alle registerenhetsendringer, uavhengig av om registerenheten er aktiv eller historisk, samt foreløpig registrerte registerenhetsendringer.

Registerenhetsendring er representert ved klassen Registerenhetsendring (med alle subklasser) i domenemodellen.  

Søkeparameter for denne tjenesten er id for registerenheten man skal finne registerenhetsendringer for. 

#### Mer om foreløpig registreringer (teknisk):
Registerenheten trenger ikke være endelig registrert.

Tjenesten finner først alle foreløpig registrerte meldinger som systemet har registrert at påvirker denne registerenheten og tar inn de beregnede effektene fra disse. 
Hvis registerenheten er en borettslagsandel, så finner den alle borettslagsandelsendringer hvor borettslagsandelen er predikert til å ha rollen til eller fra. Hvis registerenheten er en matrikkelenhet, så finner den alle matrikkelenhetsendringer hvor matrikkelenheten er predikert til ha rollen til eller fra. Er matrikkelenheten i tillegg en seksjon, så søkes det også opp alle seksjonssameieendringer hvor en seksjonssameieandel for seksjonen er predikert til å ha rollene ny, utgått, ny for tilpasning av brøk eller utgått for tilpasning av brøk.

Det som returneres er id-ene til disse registerenhetsendringene, samt den predikerte tilstanden til registerenhetsendringene, alle rettsendringer på dette rekursivt, og så en vanlig transfer basert på dette.

### findOverdragelserAvRegisterenhetsrettForPerson
#### Kort beskrivelse av tjenesten:
Tjenesten gir oversikt over rettsstiftelser der en person har blitt (og fortsatt er) rettighetshaver til registerenhetsrett på registerenheter. Enten i spesifikke kommuner eller i hele landet, avhengig av valgte søkeparameter. Tjenesten leverer også foreløpig registrerte overdragelser for en person. 

Overdragelse av registerenhetsrett for person er representert ved klassene EierskifteMatrikkelenhet, EtableringAvFesterett, OverdragelseAvFesterett, EierskifteBorettslagsandel, EtableringAvEiendomsrettBorettslagsandel, Borett2_13 og EierskifteFraBorett2_13 i domenemodellen. 

Søkeparameter for denne tjenesten er id for person man ønsker å finne overdragelser for, samt eventuelle lister over id-er til kommuner dersom man ønsker å begrense søket. Dersom kommune er angitt, vil man ikke få treff på overdragelser av borettslagsandeler. Dersom kommune ikke er angitt vil man få treff på overdragelser som gjelder hele landet, for både matrikkelenheter og borettslagsandeler. 

#### Mer om foreløpig registreringer (teknisk):
Tjenesten finner først alle foreløpig registrerte meldinger som systemer har registrert at påvirker denne personen eller noen av den registerenhetsrettsandelene personen allerede eier. Den beregnede effekten fra disse meldingene tas inn. Deretter søkes alle aktive andeler som personen ifølge prediksjonen har opp. Dersom kommune-id-er er angitt, så filtreres de andelene som ikke er i noen av kommunen vekk. Til slutt finnes alle overdragelser hvor gjenværende registerenhetsrettsandeler har rollen ny.

Det som returneres er id-ene til disse overdragelsene, samt den predikerte tilstanden til overdragelsene, og så en vanlig transfer basert på dette.

### findRettigheterForPerson
#### Kort beskrivelse av tjenesten:
Tjenesten gir oversikt over rettsstiftelser der en person har blitt (og fortsatt er) rettighetshaver (herunder panthaver og saksøker) i heftelser. Enten i spesifikke kommuner eller i hele landet, avhengig av valgte søkeparameter. Tjenesten leverer også foreløpig registrerte rettigheter for en person.

Rettigheter for person er representert ved klassene Leieavtale, Fremleieavtale, Avtale, Servitutt, ServituttPengeheftelse, AnnenRettighet, HeftelseMedBeloep, HeftelseIAnnenRett og AnnenHeftelse i domenemodellen. 

Søkeparameter for denne tjenesten er id for person man ønsker å finne rettigheter for, samt eventuelle lister over id-er til kommuner dersom man ønsker å begrense søket. Dersom kommune er angitt, vil man ikke få treff på rettigheter som hefter i borettslagsandeler. Dersom kommune ikke er angitt vil man få treff på rettigheter som gjelder hele landet, for både matrikkelenheter og borettslagsandeler. 

#### Mer om foreløpig registreringer (teknisk):
Tjenesten finner først alle foreløpig registrerte meldinger som systemet har registrert at påvirker denne personen og tar inn de beregnede effektene fra disse. Deretter finner den alle rettsstiftelser hvor personen er predikert til å være aktiv rettighetshaver eller aktiv saksøker. Rettsstiftelsene siles så ned til de som er Leieavtale, Fremleieavtale, AnnenRettighet, AnnenHeftelse, Avtale, Servitutt, ServituttPengeheftelse, HeftelseMedBeloep eller HeftelseIAnnenRett. Dersom kommunefilter er angitt, så siles de ytterligere ned til de rettsstiftelsene som hefter i matrikkelenheter eller andel i så dann i en av de kommunene, eller som hefter i en slik rettsstiftelse og er begrenset til registerenhetsrett i en slik matrikkelenhet.
Det som returneres er id-ene til disse rettsstiftelsene, samt den predikerte tilstanden til rettsstiftelsene, og så en vanlig transfer basert på dette.

### findRettigheterForRegisterenhet
#### Kort beskrivelse av tjenesten:
Tjenesten gir oversikt over rettsstiftelser der registerenhetsretter på en registerenhet har blitt (og fortsatt er) rettighetshaver i heftelser. Enten i spesifikke kommuner eller i hele landet, avhengig av valgte søkeparameter. Tjenesten leverer også foreløpig registrerte rettigheter for en registerenhet.

Rettigheter for registerenhet er representert ved klassene Servitutt, ServituttPengeheftelse, Avtale og AnnenHeftelse i domenemodellen. 

Søkeparameter for denne tjenesten er id for registerenhet man ønsker å finne rettigheter for, samt eventuelle lister over id-er til kommuner dersom man ønsker å begrense søket. Dersom kommune er angitt, vil man ikke få treff på rettigheter som hefter i borettslagsandeler. Dersom kommune ikke er angitt vil man få treff på rettigheter som gjelder hele landet, for både matrikkelenheter og borettslagsandeler. 

#### Mer om foreløpig registreringer (teknisk):
Tjenesten finner først alle foreløpig registrerte meldinger som systemet har registrert at påvirker realkoblinger mot rettigheter for denne registerenheten og tar inn de beregnede effektene fra disse. Deretter finner den alle rettsstiftelser hvor registerenheten er predikert til å være aktivt realkoblet. Rettsstiftelsene siles så ned til de som er AnnenHeftelse, Avtale, Servitutt eller ServituttPengeheftelse. Dersom kommunefilter er angitt, så siles de ytterligere ned til de rettsstiftelsene som hefter i matrikkelenheter eller andel i så dann i en av de kommunene, eller som hefter i en slik rettsstiftelse og er begrenset til registerenhetsrett i en slik matrikkelenhet.

Det som returneres er id-ene til disse rettsstiftelsene, samt den predikerte tilstanden til rettsstiftelsene, og så en vanlig transfer basert på dette.

### findRettsstiftelserForDokument
#### Kort beskrivelse av tjenesten:
Tjenesten gir oversikt over de rettsstiftelsene et dokument inneholder. Også foreløpig registrerte rettsstiftelser i et dokument vil bli levert i denne tjenesten.

Søkeparameter for denne tjenesten id til dokumentet man ønsker å finne rettsstiftelser for.  

#### Mer om foreløpig registreringer (teknisk):
Dokumentet trenger ikke være endelig registrert.

Tjenesten finner først alle foreløpig registrerte meldinger som systemet har registrert at påvirker dokumentet og tar inn de beregnede effektene fra disse. Deretter finner den alle rettsstiftelsen i dokumentet.

Det som returneres er id-ene til disse rettsstiftelsene, samt den predikerte tilstanden til rettsstiftelsene, og så en vanlig transfer basert på dette.

### findRettsstiftelse
#### Kort beskrivelse av tjenesten:
Tjenesten gir informasjon om en gitt rettsstiftelse. Også foreløpig registrerte rettsstiftelser blir levert i denne tjenesten.

Søkeparameter for denne tjenesten er id til rettsstiftelsen man ønsker å finne. 

#### Mer om foreløpig registreringer (teknisk):
Rettsstiftelsen trenger ikke være endelig registrert.

Tjenesten finner først alle foreløpig registrerte meldinger som systemet har registrert at påvirker rettsstiftelsen og tar inn de beregnede effektene fra disse. Deretter henter den ut rettsstiftelsen.

Det som returneres er id-ene til denne rettsstiftelsen, som ikke er så veldig spennende, samt den predikerte tilstanden til rettsstiftelsen, og så en vanlig transfer basert på dette.

### findBorettslagsandelerForBorettslag
#### Kort beskrivelse av tjenesten:
Tjenesten gir oversikt over alle borettslagsandeler i et borettslag. Også foreløpige registrerte borettslagsandeler blir levert i denne tjenesten.

Søkeparameter for denne tjenesten er id til borettslaget man ønsker å finne borettslagsandelene for.  

#### Mer om foreløpig registreringer (teknisk):
Borettslaget trenger ikke være endelig registrert.

Tjenesten finner først alle foreløpig registrerte meldinger som systemet har registrert at påvirker borettslaget eller dens borettslagsandeler og tar inn de beregnede effektene fra disse. Deretter finner den alle borettslagsandeler som prediksjonen sier finnes for borettslaget.

Det som returneres er id-ene til disse borettslagsandelene, samt den predikerte tilstanden til borettslagsandelene, borettslaget, borettslagets person, borettslagsandelenes adresser og adressenes kommuner, samt alle koder brukt i alt som er funnet så langt.

## 3. GrunnboksutskriftService
### ubekreftetGrunnboksutskriftPdf
#### Kort beskrivelse av tjenesten:
Tjenesten gir oversikt over all tinglyst informasjon (som fortsatt er gjeldende) som er registrert på en registerenhet. 

Tjenesten leverer en fullstendig grunnboksutskrift for registerenheten, i pdf-format. Dersom det er foreløpige registreringer på registerenheten, vil også disse fremgå av grunnboksutskriften.

Søkeparameter for denne tjenesten er id til registerenheten man ønsker å finne grunnboksutskrift for. 

### ubekreftetHistoriskGrunnboksutskriftPdf
#### Kort beskrivelse av tjenesten:
Tjenesten gir oversikt over all tinglyst informasjon (som ikke lenger er gjeldende) som er registrert på en registerenhet.
 
Tjenesten leverer en fullstendig historisk grunnboksutskrift for registerenheten, i pdf-format. Dersom det er foreløpige registreringer på registerenheten, som setter rettsstiftelser historisk, vil disse rettsstiftelsene fremgå av grunnboksutskriften. 

Søkeparameter for denne tjenesten er id til registerenheten man ønsker å finne historisk grunnboksutskrift for. 

## 4. PersonService
### findPersoner
#### Kort beskrivelse av tjenesten:
Tjenesten gir oversikt over personer i databasen som oppfyller angitte søkerkriterier. 

Søkeparameter for denne tjenesten er navn, fysisk person (J/N), juridisk person (J/N) samt fødselsår. 

Teksten som oppgis som navn kan ikke være tom. Det blir sjekket mot for- og etternavn/organisasjonsnavn etter en passende algoritme. Dersom fysisk person er angitt vil resultatet inneholde fysiske personer og personer med løpenummer. Dersom juridisk person er angitt vil resultatet inneholde juridiske personer og personer med løpenummer. Dersom både fysisk og juridisk person er valgt vil resultatet inneholde fysiske personer, juridiske personer og personer med løpenummer. (Som hovedregel skal personer registreres med fødselsnummer eller organisasjonsnummer i grunnboken, men av ulike grunner registreres noen personer med automatisk genererte løpenumre).

## 5. EndringsloggService
Formålet med endringsloggen er primært å tilby en mekanisme der andre datasystemer kan holdes ajour med grunnboken i sanntid. Når bobler i domenemodellen opprettes, oppdateres eller slettes registreres det endringshendelser i databasen. Endringsloggen tilbyr tjenester som, basert på disse endringshendelsene, gir oversikt over hvilke bobler som har endret seg siden et gitt endringsnummer.

Tjenesten leverer også foreløpige registreringer.

## 6. NedlastingService
For effektivt å komme i gang med å holde lokale systemer ajour med grunnboken via endringslogg, tilbys en nedlastingstjeneste. Tjenesten er basert på nedlasting av de ulike typer bobler i domenemodellen i fortløpende batcher, inntil alle er lastet ned. Ved å ta vare på siste endringsnummer før nedlastingen startes kan deretter, når alle forekomster av alle aktuelle typer bobler er lastet ned, endringsloggen leses fra dette endringsnummeret for ajourhold.

Tjenesten leverer også foreløpig registreringer.

## 7. IdentService
Denne tjenesten kan brukes for å hente ut id-ene til de ulike bobleobjektene i databasene, ved å benytte seg av det aktuelle objektets ident. Id-en er det som må benyttes for å hente ut det aktuelle objektet ved bruk av StoreService, og også som innparameter for andre søketjenester. Eksempelvis, ved søk på matrikkelenhetident vil id-en til den aktuelle matrikkelenheten bli levert, og ved søk på dokumentident vil id-en til dokumentet bli levert.  

Tjenesten leverer kun endelige registreringer.

Søkeparameter for denne tjenesten er identen til objektet man vil ha id til.

## 8. StoreService
Denne tjenesten kan brukes for å hente selve bobleobjektet fra databasen, ved å benytte seg av det aktuelle objektets id. Den nødvendige id-en kan hentes ut ved bruk av IdentService, og id-er leveres også ut fra bl.a. søketjenester og endringslogg. Det kan også hentes objekter for en liste av id-er. Eksempelvis, ved søk på rettsstiftelsesid vil den aktuelle rettsstiftelsen bli levert. 

Tjenesten leverer kun endelige registreringer.  

Søkeparameter for denne tjenesten er id-en til objektet man vil hente.  
