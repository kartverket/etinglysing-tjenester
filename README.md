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
Dataene for grunnboken er modellert med boblearkitektur. En boblemodell er en teknisk implementasjon av domenemodellen, der domenet er oppdelt i bobler som representerer de viktigste domeneklassene. Bobleobjektene er l�st koblet ved at de refererer til hverandre via koblinger p� id-niv� og ikke p� objekt-niv�. Dokumentasjon av denne modellen (UML-modell) er tilgjengelig via denne linken: https://etgltest.grunnbok.no/grunnbok/modell/grunnbok-domene-v2-modell/index.html 

I den delen av modellen som omhandler dokumenter og rettsstiftelser er det kun data som er endelig registrert i grunnboken som inng�r.

## 2. InformasjonService
### Transfer
I tr�d med v�r boblearkitektur, s� returnerer s�ketjenestene i utgangspunktet bare id-ene for det man ber om. Selve dataene m� man deretter hente med en egen tjeneste, StoreService. Men siden InformasjonService, i motsetning til alle andre bobletjenester, tar hensyn til forel�pige registreringer, s� kan man ikke benytte StoreService for � hente ut data, da den ikke vil se de forel�pige registreringene. Derfor legger vi med en s�kalt transfer som inneholder dataene for de boblene vi anser som relevante for � forst� og presentere svaret, slik de er if�lge de forel�pige registreringene. Vi legger ogs� med informasjon om hvilke bobler som har blitt p�virket av forel�pige registreringer.

Der det st�r at tjenestene bygger en vanlig transfer, s� best�r denne i tillegg til de rettsstiftelsene som allerede har blitt listet for hver tjeneste, av dokumentene for alle disse rettsstiftelsene, alle dokumenter som ellers refereres til fra disse rettsstiftelsene, alle registerenhetsrettene som refereres til fra disse rettsstiftelsene (hefter i eller realkoblet), alle registerenheter som refereres til direkte fra rettsstiftelsene, alle borettslags som refereres til direkte fra rettsstiftelsene, alle personer som refereres til direkte fra rettsstiftelsene, alle seksjonssameieandeler som refereres til direkte fra rettsstiftelsene, alle rettsstiftelser som er referert til fra rettsstiftelser som allerede er funnet, dokumentene til disse rettsstiftelsene, alle registerenhetsrettsandelene de opprinnelige rettsstiftelsene hefter i, alle personer som er rettighetshaver til disse andelene, alle registerenheter som er realkoblet til disse andelene, alle registerenhetsrettene disse andelene er i, alle registerenhetene for disse registerenhetsrettene, borettslag for alle borettslagsandeler funnet s� langt, adresser og personer for alle borettslag funnet s� langt, kommuner for alle matrikkelenheter og adresser funnet s� langt, og alle koder brukt i alt som er funnet s� langt.

### findHeftelser
#### Kort beskrivelse av tjenesten:
Tjenesten gir oversikt over alle tinglyste heftelser p� en registerenhet, samt alle tinglyste heftelser i disse heftelsene, rettsendringer som endrer disse heftelsene og anmerkninger som anmerker disse heftelsene. Denne tjenesten leverer ogs� forel�pig registrerte heftelser p� registerenheten det s�kes p�.

Heftelsene er representert ved klassen Heftelser (med alle subklasser) i domenemodellen. 

S�keparameter for denne tjenesten er id for registerenheten man skal finne heftelser for.

#### Mer om forel�pig registreringer (teknisk):
Registerenheten trenger ikke v�re endelig registrert.

Tjenesten finner f�rst alle forel�pig registrerte meldinger som systemet har registrert at p�virker denne registerenheten og tar inn de beregnede effektene fra disse. Deretter finner den i den predikerte tilstanden hvilke registerenhetsretter som finnes p� registerenheten. S� finner den alle andeler i registerenhetsrettene som i henhold til prediksjonen er aktive. S�ket avsluttes s� med � finne alle heftelser som er predikert � hefte i disse andelene.

Det som returneres er id-ene til disse heftelsene, samt den predikerte tilstanden til heftelsene, alle heftelser som hefter i dem, alle anmerkninger p� alt dette, alle rettsendringer p� alt dette igjen rekursivt, og s� en vanlig transfer basert p� dette.

### findTidligereHeftelser
#### Kort beskrivelse av tjenesten:
Tjenesten gir oversikt over alle tidligere tinglyste (historiske) heftelser p� en registerenhet, samt alle tidligere tinglyste heftelser i disse heftelsene, tidligere rettsendringer som endrer disse heftelsene, og tidligere anmerkninger som anmerket disse heftelsene. 
Denne tjenesten leverer ogs� heftelser som ikke har blitt historiske, men hvor det er forel�pig registrert en sletting p� heftelsen, som vil gj�re heftelsen historisk n�r den forel�pige registrerte sletting blir godkjent. Det samme gjelder heftelser som ikke har blitt slettet, men som har blitt historiske som f�lge av at registerenheten heftelsen hefter i er historisk/utg�tt.

Heftelsene er representert ved klassen Heftelser (med alle subklasser) i domenemodellen.  

S�keparameter for denne tjenesten er id for registerenheten man skal finne tidligere (historiske) heftelser for.

#### Mer om forel�pig registreringer (teknisk):
Registerenheten trenger ikke v�re endelig registrert.

Tjenesten finner f�rst alle forel�pig registrerte meldinger som systemet har registrert at p�virker denne registerenheten og tar inn de beregnede effektene fra disse. Deretter finner den i den predikerte tilstanden hvilke registerenhetsretter som finnes p� registerenheten. S� finner den alle andeler som i henhold til prediksjonen er i registerenhetsrettene, b�de aktive og historiske. S�ket avsluttes s� med � finne alle heftelser som er predikert � hefte historisk i disse andelene.

Det som returneres er id-ene til disse heftelsene, samt den predikerte tilstanden til heftelsene, alle heftelser som hefter i dem, alle anmerkninger p� alt dette, alle rettsendringer p� alt dette igjen rekursivt, og s� en vanlig transfer basert p� dette.

### findOverdragelserAvRegisterenhetsrett
#### Kort beskrivelse av tjenesten:
Tjenesten gir oversikt over alle tinglyste (og fortsatt gjeldende) overdragelser av registerenhetsretter, samt alle rettsendringer som endrer disse overdragelsene, og anmerkninger som anmerker disse overdragelsene. 

F�rstegangsregistrering av registerenhetsretter (eksempelvis ved f�rstegangsregistrering av rettighetshavere til registerenhetsrett etter opprettelse av matrikkelenhet fra tidligere uregistrert grunn), og overdragelser der en registerenhet har f�tt andel (og fortsatt har andel) i et realsameie/jordsameie blir ogs� levert i denne tjenesten. 

Ogs� alle forel�pig registrerte overdragelser vil bli levert i denne tjenesten.

Overdragelsene er representert ved klassen OverdragelseAvRegisterenhetsrett (med alle subklasser) i domenemodellen. 

S�keparameter for denne tjenesten er id for registerenheten man skal finne overdragelser for.

#### Mer om forel�pig registreringer (teknisk):
Registerenheten trenger ikke v�re endelig registrert.

Tjenesten finner f�rst alle forel�pig registrerte meldinger som systemet har registrert at p�virker denne registerenheten og tar inn de beregnede effektene fra disse. Deretter finner den i den predikerte tilstanden alle overdragelser hvor noen av de nye andelene er aktive andeler i registerenhetsretter i denne registerenheten, og alle overdragelser hvor noen av de nye andelene er aktive andeler realkoblet til denne registerenheten.

Det som returneres er id-ene til disse overdragelsene, samt den predikerte tilstanden til overdragelsene, alle anmerkninger p� disse, alle rettsendringer p� alt dette igjen rekursivt, og s� en vanlig transfer basert p� dette.

### findTidligereOverdragelserAvRegisterenhetsrett
#### Kort beskrivelse av tjenesten:
Tjenesten gir oversikt over alle tinglyste (men ikke lenger gjeldende) overdragelser av registerenhetsretter (for eksempel fordi registerenhetsretten er overdratt videre), samt alle rettsendringer som endrer disse overdragelsene, og anmerkninger som anmerker disse overdragelsene. 

F�rstegangsregistrering av registerenhetsretter (som ikke lenger er gjeldende), og overdragelser der en registerenhet har f�tt andel (men ikke lenger har samme andel) i et realsameie/jordsameie blir ogs� levert i denne tjenesten. 

Ogs� overdragelser som ikke lenger er gjeldende, som f�lge av en forel�pig registrert overdragelse p� registerenhetsretten, blir levert i denne tjenesten. Det samme gjelder overdragelser som ikke lenger er gjeldende som f�lge av at registerenhetsretten er historisk/utg�tt.   

Overdragelsene er representert ved klassen OverdragelseAvRegisterenhetsrett (med alle subklasser) i domenemodellen.

S�keparameter for denne tjenesten er id for registerenheten man skal finne tidligere (ikke lenger gjeldende) overdragelser for.

#### Mer om forel�pig registreringer (teknisk):
Registerenheten trenger ikke v�re endelig registrert.

Tjenesten finner f�rst alle forel�pig registrerte meldinger som systemet har registrert at p�virker denne registerenheten og tar inn de beregnede effektene fra disse. Deretter finner den i den predikerte tilstanden alle overdragelser hvor noen av de nye andelene er historiske andeler i registerenhetsretter i denne registerenheten, og alle overdragelser hvor noen av de nye andelene er historiske andeler realkoblet til denne registerenheten.

Det som returneres er id-ene til disse overdragelsene, samt den predikerte tilstanden til overdragelsene, alle anmerkninger p� disse, alle rettsendringer p� alt dette igjen rekursivt, og s� en vanlig transfer basert p� dette.

### findRegisterenhetsendringer
#### Kort beskrivelse av tjenesten:
Tjenesten gir oversikt over alle tinglyste registerenhetsendringer p� en registerenhet, samt alle rettsendringer som endrer disse registerenhetsendringene. Tjenesten leverer alle registerenhetsendringer, uavhengig av om registerenheten er aktiv eller historisk, samt forel�pig registrerte registerenhetsendringer.

Registerenhetsendring er representert ved klassen Registerenhetsendring (med alle subklasser) i domenemodellen.  

S�keparameter for denne tjenesten er id for registerenheten man skal finne registerenhetsendringer for. 

#### Mer om forel�pig registreringer (teknisk):
Registerenheten trenger ikke v�re endelig registrert.

Tjenesten finner f�rst alle forel�pig registrerte meldinger som systemet har registrert at p�virker denne registerenheten og tar inn de beregnede effektene fra disse. 
Hvis registerenheten er en borettslagsandel, s� finner den alle borettslagsandelsendringer hvor borettslagsandelen er predikert til � ha rollen til eller fra. Hvis registerenheten er en matrikkelenhet, s� finner den alle matrikkelenhetsendringer hvor matrikkelenheten er predikert til ha rollen til eller fra. Er matrikkelenheten i tillegg en seksjon, s� s�kes det ogs� opp alle seksjonssameieendringer hvor en seksjonssameieandel for seksjonen er predikert til � ha rollene ny, utg�tt, ny for tilpasning av br�k eller utg�tt for tilpasning av br�k.

Det som returneres er id-ene til disse registerenhetsendringene, samt den predikerte tilstanden til registerenhetsendringene, alle rettsendringer p� dette rekursivt, og s� en vanlig transfer basert p� dette.

### findOverdragelserAvRegisterenhetsrettForPerson
#### Kort beskrivelse av tjenesten:
Tjenesten gir oversikt over rettsstiftelser der en person har blitt (og fortsatt er) rettighetshaver til registerenhetsrett p� registerenheter. Enten i spesifikke kommuner eller i hele landet, avhengig av valgte s�keparameter. Tjenesten leverer ogs� forel�pig registrerte overdragelser for en person. 

Overdragelse av registerenhetsrett for person er representert ved klassene EierskifteMatrikkelenhet, EtableringAvFesterett, OverdragelseAvFesterett, EierskifteBorettslagsandel, EtableringAvEiendomsrettBorettslagsandel, Borett2_13 og EierskifteFraBorett2_13 i domenemodellen. 

S�keparameter for denne tjenesten er id for person man �nsker � finne overdragelser for, samt eventuelle lister over id-er til kommuner dersom man �nsker � begrense s�ket. Dersom kommune er angitt, vil man ikke f� treff p� overdragelser av borettslagsandeler. Dersom kommune ikke er angitt vil man f� treff p� overdragelser som gjelder hele landet, for b�de matrikkelenheter og borettslagsandeler. 

#### Mer om forel�pig registreringer (teknisk):
Tjenesten finner f�rst alle forel�pig registrerte meldinger som systemer har registrert at p�virker denne personen eller noen av den registerenhetsrettsandelene personen allerede eier. Den beregnede effekten fra disse meldingene tas inn. Deretter s�kes alle aktive andeler som personen if�lge prediksjonen har opp. Dersom kommune-id-er er angitt, s� filtreres de andelene som ikke er i noen av kommunen vekk. Til slutt finnes alle overdragelser hvor gjenv�rende registerenhetsrettsandeler har rollen ny.

Det som returneres er id-ene til disse overdragelsene, samt den predikerte tilstanden til overdragelsene, og s� en vanlig transfer basert p� dette.

### findRettigheterForPerson
#### Kort beskrivelse av tjenesten:
Tjenesten gir oversikt over rettsstiftelser der en person har blitt (og fortsatt er) rettighetshaver (herunder panthaver og saks�ker) i heftelser. Enten i spesifikke kommuner eller i hele landet, avhengig av valgte s�keparameter. Tjenesten leverer ogs� forel�pig registrerte rettigheter for en person.

Rettigheter for person er representert ved klassene Leieavtale, Fremleieavtale, Avtale, Servitutt, ServituttPengeheftelse, AnnenRettighet, HeftelseMedBeloep, HeftelseIAnnenRett og AnnenHeftelse i domenemodellen. 

S�keparameter for denne tjenesten er id for person man �nsker � finne rettigheter for, samt eventuelle lister over id-er til kommuner dersom man �nsker � begrense s�ket. Dersom kommune er angitt, vil man ikke f� treff p� rettigheter som hefter i borettslagsandeler. Dersom kommune ikke er angitt vil man f� treff p� rettigheter som gjelder hele landet, for b�de matrikkelenheter og borettslagsandeler. 

#### Mer om forel�pig registreringer (teknisk):
Tjenesten finner f�rst alle forel�pig registrerte meldinger som systemet har registrert at p�virker denne personen og tar inn de beregnede effektene fra disse. Deretter finner den alle rettsstiftelser hvor personen er predikert til � v�re aktiv rettighetshaver eller aktiv saks�ker. Rettsstiftelsene siles s� ned til de som er Leieavtale, Fremleieavtale, AnnenRettighet, AnnenHeftelse, Avtale, Servitutt, ServituttPengeheftelse, HeftelseMedBeloep eller HeftelseIAnnenRett. Dersom kommunefilter er angitt, s� siles de ytterligere ned til de rettsstiftelsene som hefter i matrikkelenheter eller andel i s� dann i en av de kommunene, eller som hefter i en slik rettsstiftelse og er begrenset til registerenhetsrett i en slik matrikkelenhet.
Det som returneres er id-ene til disse rettsstiftelsene, samt den predikerte tilstanden til rettsstiftelsene, og s� en vanlig transfer basert p� dette.

### findRettigheterForRegisterenhet
#### Kort beskrivelse av tjenesten:
Tjenesten gir oversikt over rettsstiftelser der registerenhetsretter p� en registerenhet har blitt (og fortsatt er) rettighetshaver i heftelser. Enten i spesifikke kommuner eller i hele landet, avhengig av valgte s�keparameter. Tjenesten leverer ogs� forel�pig registrerte rettigheter for en registerenhet.

Rettigheter for registerenhet er representert ved klassene Servitutt, ServituttPengeheftelse, Avtale og AnnenHeftelse i domenemodellen. 

S�keparameter for denne tjenesten er id for registerenhet man �nsker � finne rettigheter for, samt eventuelle lister over id-er til kommuner dersom man �nsker � begrense s�ket. Dersom kommune er angitt, vil man ikke f� treff p� rettigheter som hefter i borettslagsandeler. Dersom kommune ikke er angitt vil man f� treff p� rettigheter som gjelder hele landet, for b�de matrikkelenheter og borettslagsandeler. 

#### Mer om forel�pig registreringer (teknisk):
Tjenesten finner f�rst alle forel�pig registrerte meldinger som systemet har registrert at p�virker realkoblinger mot rettigheter for denne registerenheten og tar inn de beregnede effektene fra disse. Deretter finner den alle rettsstiftelser hvor registerenheten er predikert til � v�re aktivt realkoblet. Rettsstiftelsene siles s� ned til de som er AnnenHeftelse, Avtale, Servitutt eller ServituttPengeheftelse. Dersom kommunefilter er angitt, s� siles de ytterligere ned til de rettsstiftelsene som hefter i matrikkelenheter eller andel i s� dann i en av de kommunene, eller som hefter i en slik rettsstiftelse og er begrenset til registerenhetsrett i en slik matrikkelenhet.

Det som returneres er id-ene til disse rettsstiftelsene, samt den predikerte tilstanden til rettsstiftelsene, og s� en vanlig transfer basert p� dette.

### findRettsstiftelserForDokument
#### Kort beskrivelse av tjenesten:
Tjenesten gir oversikt over de rettsstiftelsene et dokument inneholder. Ogs� forel�pig registrerte rettsstiftelser i et dokument vil bli levert i denne tjenesten.

S�keparameter for denne tjenesten id til dokumentet man �nsker � finne rettsstiftelser for.  

#### Mer om forel�pig registreringer (teknisk):
Dokumentet trenger ikke v�re endelig registrert.

Tjenesten finner f�rst alle forel�pig registrerte meldinger som systemet har registrert at p�virker dokumentet og tar inn de beregnede effektene fra disse. Deretter finner den alle rettsstiftelsen i dokumentet.

Det som returneres er id-ene til disse rettsstiftelsene, samt den predikerte tilstanden til rettsstiftelsene, og s� en vanlig transfer basert p� dette.

### findRettsstiftelse
#### Kort beskrivelse av tjenesten:
Tjenesten gir informasjon om en gitt rettsstiftelse. Ogs� forel�pig registrerte rettsstiftelser blir levert i denne tjenesten.

S�keparameter for denne tjenesten er id til rettsstiftelsen man �nsker � finne. 

#### Mer om forel�pig registreringer (teknisk):
Rettsstiftelsen trenger ikke v�re endelig registrert.

Tjenesten finner f�rst alle forel�pig registrerte meldinger som systemet har registrert at p�virker rettsstiftelsen og tar inn de beregnede effektene fra disse. Deretter henter den ut rettsstiftelsen.

Det som returneres er id-ene til denne rettsstiftelsen, som ikke er s� veldig spennende, samt den predikerte tilstanden til rettsstiftelsen, og s� en vanlig transfer basert p� dette.

### findBorettslagsandelerForBorettslag
#### Kort beskrivelse av tjenesten:
Tjenesten gir oversikt over alle borettslagsandeler i et borettslag. Ogs� forel�pige registrerte borettslagsandeler blir levert i denne tjenesten.

S�keparameter for denne tjenesten er id til borettslaget man �nsker � finne borettslagsandelene for.  

#### Mer om forel�pig registreringer (teknisk):
Borettslaget trenger ikke v�re endelig registrert.

Tjenesten finner f�rst alle forel�pig registrerte meldinger som systemet har registrert at p�virker borettslaget eller dens borettslagsandeler og tar inn de beregnede effektene fra disse. Deretter finner den alle borettslagsandeler som prediksjonen sier finnes for borettslaget.

Det som returneres er id-ene til disse borettslagsandelene, samt den predikerte tilstanden til borettslagsandelene, borettslaget, borettslagets person, borettslagsandelenes adresser og adressenes kommuner, samt alle koder brukt i alt som er funnet s� langt.

## 3. GrunnboksutskriftService
### ubekreftetGrunnboksutskriftPdf
#### Kort beskrivelse av tjenesten:
Tjenesten gir oversikt over all tinglyst informasjon (som fortsatt er gjeldende) som er registrert p� en registerenhet. 

Tjenesten leverer en fullstendig grunnboksutskrift for registerenheten, i pdf-format. Dersom det er forel�pige registreringer p� registerenheten, vil ogs� disse fremg� av grunnboksutskriften.

S�keparameter for denne tjenesten er id til registerenheten man �nsker � finne grunnboksutskrift for. 

### ubekreftetHistoriskGrunnboksutskriftPdf
#### Kort beskrivelse av tjenesten:
Tjenesten gir oversikt over all tinglyst informasjon (som ikke lenger er gjeldende) som er registrert p� en registerenhet.
 
Tjenesten leverer en fullstendig historisk grunnboksutskrift for registerenheten, i pdf-format. Dersom det er forel�pige registreringer p� registerenheten, som setter rettsstiftelser historisk, vil disse rettsstiftelsene fremg� av grunnboksutskriften. 

S�keparameter for denne tjenesten er id til registerenheten man �nsker � finne historisk grunnboksutskrift for. 

## 4. PersonService
### findPersoner
#### Kort beskrivelse av tjenesten:
Tjenesten gir oversikt over personer i databasen som oppfyller angitte s�kerkriterier. 

S�keparameter for denne tjenesten er navn, fysisk person (J/N), juridisk person (J/N) samt f�dsels�r. 

Teksten som oppgis som navn kan ikke v�re tom. Det blir sjekket mot for- og etternavn/organisasjonsnavn etter en passende algoritme. Dersom fysisk person er angitt vil resultatet inneholde fysiske personer og personer med l�penummer. Dersom juridisk person er angitt vil resultatet inneholde juridiske personer og personer med l�penummer. Dersom b�de fysisk og juridisk person er valgt vil resultatet inneholde fysiske personer, juridiske personer og personer med l�penummer. (Som hovedregel skal personer registreres med f�dselsnummer eller organisasjonsnummer i grunnboken, men av ulike grunner registreres noen personer med automatisk genererte l�penumre).

## 5. EndringsloggService
Form�let med endringsloggen er prim�rt � tilby en mekanisme der andre datasystemer kan holdes ajour med grunnboken i sanntid. N�r bobler i domenemodellen opprettes, oppdateres eller slettes registreres det endringshendelser i databasen. Endringsloggen tilbyr tjenester som, basert p� disse endringshendelsene, gir oversikt over hvilke bobler som har endret seg siden et gitt endringsnummer.

Tjenesten leverer ogs� forel�pige registreringer.

## 6. NedlastingService
For effektivt � komme i gang med � holde lokale systemer ajour med grunnboken via endringslogg, tilbys en nedlastingstjeneste. Tjenesten er basert p� nedlasting av de ulike typer bobler i domenemodellen i fortl�pende batcher, inntil alle er lastet ned. Ved � ta vare p� siste endringsnummer f�r nedlastingen startes kan deretter, n�r alle forekomster av alle aktuelle typer bobler er lastet ned, endringsloggen leses fra dette endringsnummeret for ajourhold.

Tjenesten leverer ogs� forel�pig registreringer.

## 7. IdentService
Denne tjenesten kan brukes for � hente ut id-ene til de ulike bobleobjektene i databasene, ved � benytte seg av det aktuelle objektets ident. Id-en er det som m� benyttes for � hente ut det aktuelle objektet ved bruk av StoreService, og ogs� som innparameter for andre s�ketjenester. Eksempelvis, ved s�k p� matrikkelenhetident vil id-en til den aktuelle matrikkelenheten bli levert, og ved s�k p� dokumentident vil id-en til dokumentet bli levert.  

Tjenesten leverer kun endelige registreringer.

S�keparameter for denne tjenesten er identen til objektet man vil ha id til.

## 8. StoreService
Denne tjenesten kan brukes for � hente selve bobleobjektet fra databasen, ved � benytte seg av det aktuelle objektets id. Den n�dvendige id-en kan hentes ut ved bruk av IdentService, og id-er leveres ogs� ut fra bl.a. s�ketjenester og endringslogg. Det kan ogs� hentes objekter for en liste av id-er. Eksempelvis, ved s�k p� rettsstiftelsesid vil den aktuelle rettsstiftelsen bli levert. 

Tjenesten leverer kun endelige registreringer.  

S�keparameter for denne tjenesten er id-en til objektet man vil hente.  
