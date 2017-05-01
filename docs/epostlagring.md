Forslag til

# Hvordan lagre epost i en Noark 5-struktur

av Petter Reinholdtsen.

Denne beskrivelsen er basert på spesifikasjonen for [Noark 5 versjon
3.1](http://www.arkivverket.no/arkivverket/Offentleg-forvalting/Noark/Noark-5)
og [Noark 5 Tjenestegresesnitt versjon 1.0
beta](https://samdok.com/leveranser/leveranser-2016/noark5-tjenestegrensesnitt-v-1-0-beta/).

Her følger en beskrivelse om hvordan epost kan lagres i en Noark
5-struktur via et Noark 5 Tjenstegrensesnitt på en slik måte at en kan
gjenskape eposttråder og sjekke kryptosignaturer basert på S/MIME
eller OpenPGP er brukt, samt kontrollere [Domain Keys Identified
Message](http://dkim.org/)-signaturer ([IETF RFC
6376](https://tools.ietf.org/html/rfc4180)).

Epostformatet er standardisert i [IETC RFC
822](https://tools.ietf.org/html/rfc822) og etterfølgende RFCer ([RFC
2822](https://tools.ietf.org/html/rfc2822) og [RFC
5322](https://tools.ietf.org/html/rfc5322)).  Jeg bruker videre i
teksten RFC 822 om dette formatet, for enkelhets skyld.  En epost
består i grove trekk av tekstlinjer, først tekstlinjer som utgjør
epostens hode/metadata, så en blank linje og tekstlinjer som utgjør
epostens kropp/innhold.  Epostenkroppens format og tegnsett er oppgitt
i hodet.  Tegnsett for tittelfeltet (Subject) er oppgitt som del av
tittelfeltet.  Epostens kropp kan bestå av flere deler (såkalte
MIME-vedlegg), som kan hentes ut separat og ha hvert sitt format og
tegnsett.  MIME-vedlegg er standardisert og beskrevet i utvidelser til
IETF RFC 5322, se
[IETF RFC 2045](https://tools.ietf.org/html/rfc2045),
[IETF RFC 2046](https://tools.ietf.org/html/rfc2046),
[IETF RFC 2047](https://tools.ietf.org/html/rfc2047),
[IETF RFC 2049](https://tools.ietf.org/html/rfc2049),
[IETF RFC 4288](https://tools.ietf.org/html/rfc4288) og
[IETF RFC 4289](https://tools.ietf.org/html/rfc4289) henvist til i
IETF RFC 5322.

En mye brukt måte å lagre epost på er som en fil per epost, slik for
eksempel [Maildir-](https://en.wikipedia.org/wiki/Maildir) og
[MH-lagringsformatene](https://en.wikipedia.org/wiki/MH_Message_Handling_System)
gjør.  En annen mye brukt lagringsmåte er flere epost i samme fil slik
for eksempel [mbox-formatet](https://en.wikipedia.org/wiki/Mbox)
([IETC RFC 4155](https://tools.ietf.org/html/rfc4155)) gjør.  I bunnen
av alle disse lagringsformatene ligger formatbeskrivelsen fra RFC 822,
som er det et epostprogram er laget for å tolke.

Epost kan lagres i Noark i epostens opprinnelige format.  I tillegg
kan det være lurt å hente ut enkeltdelene i eposten der det er aktuelt
(f.eks. ved PDF-vedlegg), slik at disse filene er tilgjengelig separat
og kan omformes til PDF/A for langtidslagring.  Verdien i
format-feltet må være unikt og representere Internett-epost.  En god
verdi å bruke her er 'RFC822'.

Eposter kan enten lagres som basisregistrering eller journalpost.
Førstnevnte er for epost som ikke skal med i den offentlige journalen.

Epost inneholder følgende hodefelter relevant for lagring i Noark 5:

 * Subject:
 * From:
 * To:
 * Cc:
 * Reply-To:
 * Sender:
 * Date:
 * In-Reply-To:
 * References:
 * Message-ID:
 * Content-Type:
 * Content-Transfer-Encoding:
 * Content-Disposition:

For a kunne effektivt finne alle epostene i en eposttråd er det lurt å
ta gjøre det mulig å søke ut epost basert på Message-ID-verdien som
var i eposten som startet tråden.  Dette er den Message-ID-verdien som
ligger først i References i epost der dette feltet finnes og i
In-Reply-To-verdien for epost som er svar på første epost.  Det er
desverre ikke noe åpenbart felt i Noark 5 for å lagre denne verdien
for å søke den ut senere. Mitt forslag er å bruke 'oppbevaringssted
tilhørende basisregistrering og journalpost, da dette feltet er ubrukt
for elektroniske dokumenter, og en kan tenke seg at epost er lagret
bak stedet merket med Message-ID-verdien.

FIXME: Bestem og beskriv hvordan epost skal lagres:

Hver arkiverte epost får en basisregistrering eller journalpost, alt
etter om eposten skal være journalført eller ikke.  Feltet "forfatter"
registreres ikke i basisregistreringsdelen, for å unngå
dobbeltregistrering i korrespondansepart og dokumentbeskrivelse.

```
registrering / basisregistrering / journalpost: {
  # registrering

  # basisregistrering / journalpost
  "tittel"         : subject,
  "oppbevaringssted" til første message-id i References?

  # Journalpost
  "dokumentDato"   : date
}
```

For Journalpost registreres det en korrespondansepart for hver adresse
oppgitt i en av From/Reply-To/Sender, To og Cc.  Epostadressedelen
(dvs. det mellom &lt; og &gt;) legges i epostadresse-attributtet til
kontaktinformasjons-feltet, mens adressens tilknyttede navn legges inn
i feltet "navn".

Når det gjelder avsender er det flere eposthodefelt som er aktuelle.
Den første med verdi av følgende felt legges inn med
korrespondansetype "avsender: Sender, From:

```
korrespondansepart: {
  "korrespondansepartType" : "avsender"
  "navn" : sender / from
  "kontaktinformasjon.epostadresse" : sender / from
}
```

Når det gjelder mottaker, så er det et eposthodefelt som er aktuelt,
"To".  Dette feltet kan inneholde flere epostadresser oppdelt med
komma.  Det lages en korrespondansepart for hver epostadresse:

```
korrespondansepart: {
  "korrespondansepartType" : "mottaker"
  "navn" : to
  "kontaktinformasjon.epostadresse" : to
}
```

Når det gjelder kopimottaker, er det også bare ett eposthodefelt som
er aktuelt, "Cc".  Dette feltet kan inneholde flere epostadresser
oppdelt med komma.  Det lages en korrespondansepart for hver
epostadresse:

```
korrespondansepart: {
  "korrespondansepartType" : "kopimottaker"
  "navn" : cc
  "kontaktinformasjon.epostadresse" : cc
}
```

Selve eposten lagres som et dokumentobjekt med tilhørende
dokumentbeskrivelse, som i dagens REST-API må opprettes i motsatt
rekkefølge:

```
dokumentbeskrivelse: {
  "tittel"         : subject,
  "forfatter"      : from / sender / reply-to,
  "tilknyttetRegistreringSom" : "Hoveddkokument"
}

dokumentobjekt: {
  "format"         : "RFC822",
  "mimeType"       : "message/rfc822",
  "filnavn"        : message-id,
}
```

Hvis eposten består av flere vedlegg, så bør det lages separate
dokumentbeskrivelse-/dokumentobjekt-instanser for vedleggene.  Det bør
være mulig å angi av det av disse vedleggene som er følgebrevet /
selve meldingen får tilknyttetRegistreringSom "Hoveddokument".


## Eksempler

### Eksempel uten vedlegg

Her er to eksempel-epost med henvisning som utgjør en liten eposttråd.

```
From: Ola Nordmann <ola@example.com>
Subject: =?iso-8859-1?q?Valg=20av=20ny=20konge,=20=F8nsker=20ny=20konge=20velkommen?=
To: Kari Nordmann <kari@example.com>
Cc: Bob-Jonny Nordmann <bob@example.com>
Date: Sun, 1 Jan 2017 12:30:00 +0100
Content-Type: text/plain; charset="UTF-8"
Message-ID: <20170101123000.AB23269@example.com>

Som avtalt i Storskogen har vi vurdert og valgt ny konge.  Det ble
Bob-Jonny.  Vi gleder oss alle til å bli hans lojale undersåtter.

-- 
Vennlig hilsen
Ola Nordmann
```

Har her droppet alle eposthodefelter som ikke er releante for dette
forslaget.  Svareposten ser så slik ut:

```
From: Bob-Jonny Nordmann <bob@example.com>
Subject: Re: =?iso-8859-1?q?Valg=20av=20ny=20konge,=20=F8nsker=20ny=20konge=20velkommen?=
To: Ola Nordmann <ola@example.com>
Cc: Kari Nordmann <kari@example.com>
Date: Mon, 2 Jan 2017 09:00:00 +0100
In-Reply-To: <20170101123000.AB23269@example.com>
References: <20170101123000.AB23269@example.com>
Content-Type: text/plain; charset="UTF-8"
Message-ID: <20170102090000.AB23269@example.com>

Jeg takker for tilliten dere viser meg ved valget, og ble veldig glad
for beskjeden.  Men jeg må desverre melde at jeg abdiserer, da Sverige
la inn et bedre tilbud.
-- 
Vennlig hilsen
Bob-Jonny Nordmann
```

FIXME vurder om oppbevaringssted skal brukes til første References-verdi og/eller In-Reply-To.

Disse to epostene legges inn i tjenestegresesnittet på følgende måte.
Det er viktig her å huske at eventuell koding av ikke-ASCII-tegn i
tittelen må oversettes til UTF-8 før lagring i tråd med beskrivelsen i
[IETF RFC 2047](https://tools.ietf.org/html/rfc2047).  Først den
første eposten:

```
registrering / basisregistrering / journalpost: {
  "tittel"         : "Valg av ny konge, ønsker ny konge velkommen",
  "dokumentDato"   : "2017-01-01T12:30+0100"
}

korrespondansepart: {
  "korrespondansepartType" : "avsender"
  "navn" : "Ola Nordmann",
  "kontaktinformasjon.epostadresse" : "ola@example.com"
}

korrespondansepart: {
  "korrespondansepartType" : "mottaker"
  "navn" : "Kari Nordmann",
  "kontaktinformasjon.epostadresse" : "kari@example.com"
}

korrespondansepart: {
  "korrespondansepartType" : "kopimottaker"
  "navn" : "Bob-Jonny Nordmann"
  "kontaktinformasjon.epostadresse" : "bob@example.com"
}

dokumentbeskrivelse: {
  "tittel"         : "Valg av ny konge, ønsker ny konge velkommen",
  "forfatter"      : "Ola Nordmann"
  "tilknyttetRegistreringSom" : "Hoveddkokument"
}

dokumentobjekt: {
  "format"         : "RFC822",
  "mimeType"       : "message/rfc822",
  "filnavn"        : "<20170101123000.AB23269@example.com>",
}
```

Selve eposten med hode og kropp lastes så opp som del av
dokumentobjekt.  Har her ignorert filstoerrelse og andre felt som ikke
er spesielt for epostarkivering.

Arkivering av svar-eposten ser så slik ut.  Merk at det RFC
5322-standardiserte (del 3.6.4) subject-prefikset 'Re: ' fjernes fra
tittelen



```
registrering / basisregistrering / journalpost: {
  "tittel"         : "Valg av ny konge, ønsker ny konge velkommen",
  "dokumentDato"   : "2017-01-02T09:00+0100"
}

korrespondansepart: {
  "korrespondansepartType" : "avsender"
  "navn" : "Bob-Jonny Nordmann",
  "kontaktinformasjon.epostadresse" : "bob@example.com"
}

korrespondansepart: {
  "korrespondansepartType" : "mottaker"
   "navn" : "Ola Nordmann",
  "kontaktinformasjon.epostadresse" : "ola@example.com"
}

korrespondansepart: {
  "korrespondansepartType" : "kopimottaker"
  "navn" : "Kari Nordmann",
  "kontaktinformasjon.epostadresse" : "kari@example.com"
}

dokumentbeskrivelse: {
  "tittel"         : "Valg av ny konge, ønsker ny konge velkommen",
  "forfatter"      : "Bob-Jonny Nordmann",
  "tilknyttetRegistreringSom" : "Hoveddkokument"
}

dokumentobjekt: {
  "format"         : "RFC822",
  "mimeType"       : "message/rfc822",
  "filnavn"        : "<20170102090000.AB23269@example.com>"
}
```

### Eksempel med vedlegg

Her er en eksempel-epost med vedlegg.

```
From: Ola Nordmann <ola@example.com>
Subject: =?iso-8859-1?q?Valg=20av=20ny=20konge,=20=F8nsker=20ny=20konge=20velkommen?=
To: Kari Nordmann <kari@example.com>
Cc: Bob-Jonny Nordmann <bob@example.com>
Date: Sun, 1 Jan 2017 12:40:00 +0100
MIME-Version: 1.0
Content-Type: multipart/mixed;
	boundary="----=_NextPart_000_0BF562E34FB3"
Message-ID: <20170101123000.AB23342@example.com>

This is a multipart message in MIME format.

------=_NextPart_000_0BF562E34FB3
Content-Type: text/plain;
	charset="UTF-8"
Content-Transfer-Encoding: quoted-printable

Som avtalt i Storskogen har vi vurdert og valgt ny konge.  Det ble
Bob-Jonny.  Vi gleder oss alle til å bli hans lojale undersåtter.
Valgresultatet er vedlagt.

-- 
Vennlig hilsen
Ola Nordmann
------=_NextPart_000_0BF562E34FB3
Content-Type: application/xml;
	name="voteringsresultat.xml"
Content-Disposition: attachment;
	filename="voteringsresultat.xml"

<?xml version='1.0' encoding='UTF-8' ?>
<votes>
  <candidate name="Albert Nordengen" count="1">
  <candidate name="Harald" count="5">
  <candidate name="Bob-Jonny Nordmann" count="4829492">
</votes>
------=_NextPart_000_0000_01D2B7B6.BFD90F10--
```

Denne eposten består av to deler, først tekst/følgebrev, og deretter
et XML-vedlegg.  Her lagres begge vedleggene ut som separate
vedleggsdokumenter i Noark 5-strukturen i tillegg til eposten i RFC
822-format.

Merk at ved bruk av Content-Type multipart/alternative skal det være
flere ulike varianter av samme innhold i eposten, slik at en henter ut
den av variantene for lagring som dokumentvedlegg i Noark 5-strukturen
som er mest egnet for arkivering (f.eks. text/plain i stedet for
text/html).  Ofte har dog de ulike "variantene" ikke samme innhold,
f.eks. er det ikke uvanlig at text/plain-utgaven ber leseren om å se
på text/html-utgaven, der epostens innhold befinner seg.  I slike
tilfeller er det HTML-utgaven som bør hentes ut som vedleggsdokument.

Der vedlegget er sendt ved bruk av Content-Transfer-Encoding
(f.eks. base64 eller quoted-printable) så skal innholdet "pakkes ut"
slik at innholdet i vedlegget lagres uten innpakking.

Noark 5-strukturen for denne eposten blir da slik:


```
registrering / basisregistrering / journalpost: {
  "tittel"         : "Valg av ny konge, ønsker ny konge velkommen",
  "dokumentDato"   : "2017-01-01T12:40+0100"
}

korrespondansepart: {
  "korrespondansepartType" : "avsender"
  "navn" : "Ola Nordmann",
  "kontaktinformasjon.epostadresse" : "ola@example.com"
}

korrespondansepart: {
  "korrespondansepartType" : "mottaker"
  "navn" : "Kari Nordmann",
  "kontaktinformasjon.epostadresse" : "kari@example.com"
}

korrespondansepart: {
  "korrespondansepartType" : "kopimottaker"
  "navn" : "Bob-Jonny Nordmann"
  "kontaktinformasjon.epostadresse" : "bob@example.com"
}
```

Deretter følger eposten i RFC822-format som hoveddokument, og de to
MIME-vedleggene som separate vedlegg med dokumentbeskrivelse og
dokumentobjekt:

```
dokumentbeskrivelse: {
  "tittel"         : "Valg av ny konge, ønsker ny konge velkommen",
  "forfatter"      : "Ola Nordmann"
  "tilknyttetRegistreringSom" : "Hoveddkokument"
}

dokumentobjekt: {
  "format"         : "RFC822",
  "mimeType"       : "message/rfc822",
  "filnavn"        : "<20170101123000.AB23342@example.com>",
}

dokumentbeskrivelse: {
  "tittel"         : "Valg av ny konge, ønsker ny konge velkommen",
  "forfatter"      : "Ola Nordmann"
  "tilknyttetRegistreringSom" : "Vedlegg"
}

dokumentobjekt: {
  "format"         : "TXT",
  "mimeType"       : "text/plain; charset=\"UTF-8\"",
}

dokumentbeskrivelse: {
  "tittel"         : "voteringsresultat.xml",
  "tilknyttetRegistreringSom" : "Vedlegg"
}

dokumentobjekt: {
  "format"         : "XML",
  "mimeType"       : "application/xml",
  "filnavn"        : "voteringsresultat.xml",
}
```

FIXME hva bør tittel for formater som ikke inneholder en tittel være,
jamfør XML-eksempelet over?

FIXME bør tekst-vedlegget arve tittel fra subjekt, eller få en annen
tittel?  Kanskje hente fra MIME-feltet Content-Description?  Eller bør
det inn i dokumentbeskrivelses-feltet 'beskrivelse'?

FIXME hvordan identifiseres hovedvedlegget i en rekursiv
MIME-struktur?  Kun se på første nivå?