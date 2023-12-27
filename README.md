# Mail User Agent

> Questo repository contiene il testo (ma *non* il codice di supporto, o i test)
> del **progetto d'esame** della sessione di gennaio/febbraio 2024
> dell'insegnamento di "Programmazione II".
>
> Fino alla chiusura delle iscrizioni, l'accesso a questo repository sarà
> consentito a tutti gli utenti di questa istanza di GitLab. 
>
> Agli studenti **regolarmente iscritti al SIFA** verrà approntata una copia
> personale (completa del codice di supporto e dei test) di questo repository
> con cui potranno effettuare la consegna della loro soluzione. 
> 
> Per **chiedere chiarimenti** sul testo, o **segnalare errori o imprecisioni**,
> usi *esclusivamente* la sezione
> [Issues](https://gitlab.di.unimi.it/prog2/projects/01-mua/-/issues) di questo
> repository; eventuali richieste pervenute per email saranno ignorate.

## Descrizione

Scopo della prova è progettare e implementare una gerarchia di oggetti utili a
realizzare una versione semplificata di un *Mail User Agent* (MUA), ovvero una
applicazione (basata su interfaccia utente di tipo testuale) in grado di
comporre e memorizzare su disco dei *messaggi email*.

L'aspetto cruciale del lavoro è che, come è ben noto, al fine di poter essere
trasmesso, ogni messaggio deve essere *codificato* in una sequenza di *byte*
secondo un formato molto preciso, descritto da una serie di RFC ("Request For
Comments", documenti normativi emessi dall'"Internet Society" e dai suoi
organismi, come la "Internet Engineering Task Force"). Sebbene non è richiesto
che l'applicazione sia in grado di trasmettere o ricevere messaggi, è
viceversa obbligatorio che i messaggi vengano memorizzati sul disco in forma
*codificata*.

Per portare a termine il lavoro dovrà decidere quali classi (concrete o
astratte) e quali interfacce implementare. Per ciascuna di esse **dovrà
descrivere** (preferibilmente in formato Javadoc, ma comunque solo attraverso
commenti presenti nel codice) le scelte relative alla **rappresentazione** dello
stato (con particolare riferimento all'*invariante di rappresentazione* e alla
*funzione di astrazione*) e ai **metodi** (con particolare riferimento a
*pre/post-condizioni* ed *effetti collaterali*, soffermandosi ad illustrare le
ragioni della *correttezza* solo per le implementazioni che riterrà più
critiche). Osservi che l'esito di questa prova, che le consentirà di accedere o
meno all'orale, si baserà tanto su questa documentazione quanto sul codice
sorgente.

> Presti grande attenzione ai **test** (che può eseguire con il comando `gradle
> test`, come illustrato nelle istruzioni per [eseguire i test in locale e
> individuare i
> difetti](https://gitlab.di.unimi.it/prog2#eseguire-i-test-in-locale-e-individuare-i-difetti)).
> Anche la **documentazione** deve essere *corretta*, in particolare il comando
> `gradle javadoc` deve terminare senza riportare problemi. **In presenza anche
> di un solo errore (nei test o nella documentazione), il progetto non sarà
> valutato**.
>
> Presti anche attenzione agli **errori di compilazione**: il contenuto dei file
> che il compilatore si rifiuta di compilare **non sarà affatto esaminato**; se
> incontrasse errori di compilazione che non è in grado di correggere valuti la
> possibilità di racchiudere le porzioni di codice che li causano all'interno di
> commenti — resta inteso che tale codice commentato non sarà valutato.

### I messaggi

Un **messaggio** è costituito da una o più *parti*; ciascuna **parte** comprende
alcune *intestazioni* (come ad esempio l'oggetto, il mittente, i destinatari…) e
un *corpo* che è il suo vero e proprio contenuto.

Una **intestazione** è caratterizzata dal *tipo* e da un *valore*, esempi di
*intestazioni* sono:

* il *mittente*, il cui valore è un *indirizzo*;
* i *destinatari*, il cui valore è una lista di, *indirizzi*;
* la *data* di composizione del messaggio, il cui valore è un oggetto di tipo
  [`ZonedDateTime`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/time/ZonedDateTime.html),
* l'*oggetto* del messaggio, il cui valore è una stringa. 

Come vedremo, un messaggio può avere anche altre intestazioni, ma quelle
elencate qui sopra *devono* necessariamente essere sempre presenti.

Per finire, un **indirizzo** è costituito da tre stringhe: il *display name*
(che è opzionale), il *locale* e il *dominio*.

### L'applicazione e le mailbox

Un **MUA** gestisce una collezione di **mailbox** (il cui contenuto è
memorizzato sul disco) ciascuna delle quali contiene una raccolta di messaggi;
l'applicazione deve consentire di:

* *elencare* le *mailbox* (in ordine alfabetico crescente del loro nome),
* *selezionare* una delle *mailbox* e, una volta fatto:
    * *elencare* i messaggi che contiene (in ordine cronologico decrescente),
    * *visualizzare* uno specifico messaggio,
    * *cancellare* uno specifico messaggio,
    * *comporre* un nuovo messaggio (ed aggiungerlo alla mailbox).

l'applicazione ha una semplice *interfaccia grafica testuale* che, leggendo un
*comando* alla volta, permette di mostrare elenchi (di mailbox e messaggi) in
**tabelle** e di visualizzare i messaggi in apposite **schede**.

Potete vedere un esempio d'uso dell'applicazione nel seguente video:

<a href="https://asciinema.org/a/TgmrX3qAK6VAAz0iyO0Ix4X5X" target="_blank"><img src="https://asciinema.org/a/TgmrX3qAK6VAAz0iyO0Ix4X5X.svg" width=800/></a>


### La codifica e la memorizzazione delle email

Il formato con cui una mail può essere codificata, in genere per la trasmissione
via rete e nel caso specifico del progetto per la memorizzazione su disco, è
descritto delle RFC:

* [Internet Message Format](https://tools.ietf.org/html/rfc5322) che descrive
  gli aspetti base del formato,
* [MIME Format of Internet Message Bodies](https://tools.ietf.org/html/rfc2045)
  che descrive come gestire i messaggi composti da più contenuti (come testo e
  HTML) e
* [MIME Message Header Extensions for Non-ASCII Text](https://tools.ietf.org/html/rfc2047)
  che illustra (tra l'altro) come poter usare le lettere accentate senza che
  compaiano strani punti di domanda o altre amenità.

Tali documenti normativi sono molto complessi e prevedono un numero elevatissimo
di casi ed eccezioni. Ai fini di questo progetto, potrà basarsi sulla seguente
presentazione semplificata che riduce notevolmente la variabilità che dovrà
gestire.

#### Le codifiche ASCII e UTF-8 Base64

Per prima cosa, occorre osservare che sebbene le stringhe Java siano Unicode
(che consente, tra l'altro, che contengano lettere accentate e caratteri di
alfabeti non "latini"), la codifica dei messaggi prevede viceversa
esclusivamente l'uso della codifica ASCII.

Per poter codificare una stringa che contiene caratteri non ASCII in questo
progetto verrà usata la codifica [Base64](https://en.wikipedia.org/wiki/Base64)
della codifica UTF-8 della stringa. 

Poiché si tratta di nozioni non banali che richiedono l'uso di API complesse del
JDK, è fornito un pacchetto di *classi di utilità*.

Ciò che deve sapere è che al fine di poter memorizzare i dati, essi dovranno
essere del tipo `ASCIICharSequence` (una implementazione di
[`CharSequence`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/CharSequence.html),
una versione semplificata del tipo stringa). Il metodo statico `isAscii` di tale
classe le permetterà di capire se una delle stringhe che intende memorizzare non
è ASCII, in tal caso potrà usare i metodi di codifica e decodifica della classe
`Base64Encoding` che consentono di passare da stringhe a `ASCIICharSequence` e
viceversa.

#### La memorizzazione su disco

Al fine di semplificare la memorizzazione dei messaggi codificati su disco, il
pacchetto di utilità contiene la classe `Storage` che permette di gestire delle
`Storage.Box` (costruite a partire da una gerarchia di *directory* sul disco) e
contenenti elenchi di `Storage.Entry` (corrispondenti ai *file* nella
directory); potrà usare le *box* per rappresentare le *mailbox* e le *entry* per
scrivere e leggere la codifica delle email nelle *box*.

#### La codifica delle intestazioni

La codifica di ciascuna *intestazione* è data dalla codifica del suo *tipo*
seguito da quella del suo *valore*. Alcuni esempi sono dati dalle linee
seguenti:

    From: Luca Prigioniero <prigioniero@di.unimi.it>
    To: Massimo Santini <santini@di.unimi.it>, random@google.com
    Subject: Un esempio
    Date: Wed, 6 Dec 2023 19:22:29 +0100

dove la codifica del tipo corrisponde alla parte a sinistra dei due punti,
mentre quella del valore è quella a destra. 

Particolare attenzione va prestata al caso in cui il valore non sia ASCII, per
semplicità nel nostro caso questo sarà possibile solo nel caso dell'*oggetto*;
se ad esempio (il valore del)l'oggetto del messaggio fosse "Questa è una prova"
(che contiene il carattere non ASCII "è"), la codifica della relativa
intestazione sarebbe

    Subject: =?utf-8?B?UXVlc3RhIMOoIHVuYSBwcm92YQ==?=

dove il valore `=?utf-8?B?UXVlc3RhIMOoIHVuYSBwcm92YQ==?=` è una *encoded-word*
ottenuta grazie al metodo statico `Base64Encoding.encodeWord`

La *data* è un altro esempio che richiede attenzione, perché il formato esatto
della sua rappresentazione testuale è dato dall'RFC 1123; per questa ragione nel
pacchetto di utilità è presente la classe `DateEncoding` che permette di
trasformare una
[`ZonedDateTime`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/time/ZonedDateTime.html)
in una stringa (ASCII) e viceversa, secondo il formato stabilito dall'RFC di cui
sopra.

Per finire, il valore delle intestazioni relative a mittente e destinatari
sono dati dalla codifica degli *indirizzi*, alcuni esempi sono

    Luca Prigioniero <prigioniero@di.unimi.it>
    "Massimo prof. Santini" <santini@di.unimi.it>
    info@unimi.it

Come si nota, il *display name* (che è opzionale, come mostra il terzo esempio) è
codificato come una stringa (se contiene più di uno spazio è racchiuso tra
virgolette, come mostra il secondo esempio); le parti *locale* e *dominio* sono
codificate come due stringhe concatenate (separate dal carattere `@`)
eventualmente racchiuse tra `<` e `>` se è presente anche il *display name*
(come mostrano i primi due esempi).

**Attenzione**: per semplificare i test la codifica delle intestazioni deve
sempre rispettare sempre questo *ordine* fissato:

    From
    To
    Subject
    Date
    MIME-Version
    Content-Type
    Content-Transfer-Encoding

e all'inizio del messaggio devono sempre essere presenti le prime quattro
intestazioni.

**Nota bene**: gli RFC specificano che la codifica di una intestazione non può
superare una data lunghezza, per evitare che ciò accada la codifica è sottoposta
a *folding* (ossia viene suddivisa su righe consecutive di lunghezza limitata);
per semplicità in questo progetto tale vincolo è rimosso e può assumere che
valga sempre che la codifica di ogni intestazione è contenuta su una sola riga
(per quanto lunga).

#### La codifica del corpo

Il *corpo* (di ciascuna *parte*) segue le intestazioni, è separato dalle
precedenti con una *riga vuota*; il tipo di codifica del corpo è dato
dall'intestazione `Content-Type` e, se il corpo non contiene caratteri non
ASCII, un esempio di codifica è

    Content-Type: text/plain; charset="us-ascii"

    Questo messaggio costituisce un semplice esempio.

dove il `charset` è appunto quello ASCII; se viceversa il corpo contiene
caratteri non ASCII, un esempio di codifica è 

    Content-Type: text/plain; charset="utf-8"
    Content-Transfer-Encoding: base64

    UXVlc3RhIHBhcnRlIMOoIHVuIGVzZW1waW8=

dove, oltre al `Content-Type` (che questa volta ha un `charset` pari a UTF-8) è
presente una intestazione di tpo `Content-Transfer-Encoding` con valore `base64`
corrispondente alla codifica scelta.

Di nuovo, nel pacchetto di utilità la class `Base64Encoding` sono messi a disposizione dei metodi per
passare da stringhe (non ASCII) a `ASCIICharSequence` corrispondenti alla loro
codifica in Base64 e UTF-8.

##### I messaggi con più parti

Talvolta può essere utile avere diverse "alternative" per il contenuto di un
messaggio. 

Ad esempio, potrebbe essere utile avere una versione HTML da mostrare in client
che siano in grado di visualizzarla (come i client web o grafici), ma inviare
anche una versione testuale pura per i client non in grado di interpretare
l'HTML (come i client "a caratteri"), oppure una versione con, o senza,
caratteri non ASCII.

Le RFC specificano un modo per codificare questa circostanza. Per prima cosa è
necessario aggiungere

    MIME-Version: 1.0
    Content-Type: multipart/alternative; boundary=frontier

alle intestazioni, quindi si possono includere le varie *parti* alternative del
messaggio a patto di separarle con una riga contenente un *boundary* (che in
questo progetto assumeremo sia sempre uguale a `frontier`).

Un esempio di codifica di un messaggio con l'oggetto contenente caratteri non
ASCII, una parte testuale (contenente caratteri non ASCII) e una HTML è data da

    From: Massimo Santini <santini@di.unimi.it>
    To: prigioniero@di.unimi.it, "Ing. Carlo Rossi" <cr@architettura.it>
    Subject: =?utf-8?B?UXVlc3RhIMOoIHVuYSBwcm92YQ==?=
    Date: Wed, 6 Dec 2023 19:22:29 +0100
    MIME-Version: 1.0
    Content-Type: multipart/alternative; boundary=frontier

    This is a message with multiple parts in MIME format.
    --frontier
    Content-Type: text/plain; charset="utf-8"
    Content-Transfer-Encoding: base64

    UXVlc3RhIHBhcnRlIMOoIHVuIGVzZW1waW8=
    --frontier
    Content-Type: text/html; charset="utf-8"
    Content-Transfer-Encoding: base64

    UXVlc3RhIHBhcnRlIMOoIGluIDxzdHJvbmc+aHRtbDwvaHRtbD4h
    --frontier--

Si può osservare che il testo `This is a message with multiple parts in MIME
format.` non è propriamente parte del messaggio, ma viene aggiunto per quei
client che non sono in grado di comprendere la codifica dei messaggi con parti
alternative; inoltre, nel caso della parte in HTML, il valore del `Content-Type` è
`text/html` e non più `text/plain`.

#### Un esempio

Di seguito è mostrato come usare `Storage` ed `EntryDecoding` per leggere e
iniziare il processo di decodifica di un messaggio. Per prima cosa è necessario
leggere il contenuto codificato del file, ad esempio `tests/mbox/test-219c4395`:

```
Storage  S = new Storage("tests/mbox");
Box box = S.boxes().get(1);
Entry entry = box.entries().get(1);
ASCIICharSequence sequence = entry.content();
```

Ora `sequence` contiene esattamente il messaggio codificato; usando
`EntryDecoding.decode` si ottiene un elenco di tre `Fragment`; ogni frammento ha
un elenco di *raw header* che sono coppie di nomi e valori delle intestazioni e
un *raw body* che è il corpo, così come è codificato (il tutto rappresentato
tramite `ASCIICharSequence`). Il codice 

```
List<Fragment> fragments = EntryEncoding.decode(sequence);
for (Fragment fragment : fragments) {
  System.out.println("Fragment\n\tRaw headers:");
  for (List<ASCIICharSequence> rawHeader : fragment.rawHeaders())
    System.out.println("\t\tRaw type = " + rawHeader.get(0) + ", value = " + rawHeader.get(1));
  System.out.println("\tRaw body: \n\t\t" + fragment.rawBody().toString().split("\n")[0] + "\n");
}
```
produce l'output

```
Fragment
	Raw headers:
		Raw type = from, value = Gioacchino Costanzi <adelmo01@sagnelli-letta.net>
		Raw type = to, value = "Sig.ra Patrizia Pisano" <elianamazzanti@abatantuono.it>, Lilla Manolesso <ocaruso@casalodi-bataglia.org>, Matilda Ovadia-Cipolla <serrigo@giacometti.org>, fiorenzo61@forza-bompiani.com
		Raw type = subject, value = Core sicura discreta
		Raw type = date, value = Tue, 5 Dec 2023 10:00:17 +0100
		Raw type = mime-version, value = 1.0
		Raw type = content-type, value = multipart/alternative; boundary=frontier
	Raw body: 
		This is a message with multiple parts in MIME format.

Fragment
	Raw headers:
		Raw type = content-type, value = text/plain; charset="us-ascii"
	Raw body: 
		Ullam nihil voluptas ipsa. Optio ea sint cumque assumenda mollitia.

Fragment
	Raw headers:
		Raw type = content-type, value = text/html; charset="utf-8"
		Raw type = content-transfer-encoding, value = base64
	Raw body: 
		PGh0bWw+QXNwZXJuYXR1ciB2b2x1cHRhdGUgYXNwZXJpb3Jlcy4gQXV0IHNpdCBuaXNpIGFsaWFz
```

a partire dal quale, usando opportunamente `Base64Encoding`, `AddressEncoding` e
`DateEncoding` è possibile proseguire con la decodifica; per esempio,
l'invocazione del metodo `AddressEncoding.decode` sui destinatari restituisce la
lista

```
[["Sig.ra Patrizia Pisano", "elianamazzanti", "abatantuono.it"], ["Lilla Manolesso", "ocaruso", "casalodi-bataglia.org"], ["Matilda Ovadia-Cipolla", "serrigo", "giacometti.org"], ["", "fiorenzo61", "forza-bompiani.com"]]
```

dalla quale è quindi possibile istanziare gli indirizzi utili a definire il
valore dell'intestazione destinatri così finalmente decodificata.

### L'interfaccia grafica testuale

Per realizzare la visualizzazione mostrata in precedenza può avvalersi delle
classi `UITable`, `UICard` e `UIInteract` del pacchetto di utilità fornito
assieme al progetto. In particolare:

* `UITable` permette di visualizzare una "tabella" di stringhe (composte anche
  da più linee ciascuna), un esempio d'uso delle tabelle è la visualizzazione di
  elenchi (di mailbox, o messaggi);
* `UICard` permette di visualizzare una "scheda" di stringhe (composte anche da
  più linee ciascuna), un esempio d'uso delle schede è la visualizzazione dei
  messaggi;
* `UIInteract` permette di costruire il
  [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop) su
  cui è basata l'applicazione.

Osservi che l'uso dell'ultimo componente è molto utile perché in grado di
sopprimere in modo automatico i prompt che non devono essere emessi dai client
di test, ma soltanto durante l'uso interattivo dell'applicazione.

## Cosa è necessario implementare

Dovrà implementare una gerarchia di oggetti utili a rappresentare:

* gli **indirizzi** (assumendo che non contengano caratteri non ASCII);
* le **intestazioni** più comuni (almeno il *mittente*, i *destinatari*, la
  *data* e l'*oggetto*); assuma che l'unica intestazione che possa contenere
  caratteri non ASCII sia l'oggetto;
* diverse **parti** di un messaggio, almeno il testo contenente solo caratteri
  ASCII, il testo comprendente anche caratteri non ASCII e l'HTML (assumendo
  che contenga sempre caratteri non ASCII);
* i **messaggi** costituiti da una sola parte di testo (contenente solo
  caratteri ASCII), o più parti;
* le **mailbox** costituite da una collezione di messaggi;
* l'**applicazione** che permette di gestire le mailbox tramite l'interfaccia
  testuale.

Al fine di evitare confusione con il pacchetto di utilità e con i client
(descritti nella prossima sezione), è consigliabile che le classi della sua
soluzione siano contenute un un pacchetto (e suoi eventuali sotto-pacchetti) a
parte, ad esempio nel pacchetto `mua` i cui sorgenti dovranno essere nella
directory `src/main/java/mua`.

#### Il pacchetto di utilità

Il codice del pacchetto di utilità fornito dal docente è contenuto nella directory
`src/main/java/utils`, per generare la *documentazione* del pacchetto può usare
il comando `gradle utilsJavadoc` (che genera la documentazione in `utils-docs`).

L'uso delle classi di tale pacchetto è inteso come un *supporto* per risolvere
alcuni problemi di natura più "tecnica", ma è assolutamente legittimo ignorarne
in tutto, o in parte, l'esistenza e procedere in modo del tutto autonomo. Il
codice contenuto nel pacchetto però **non deve essere modificato, o copiato in
altre classi del progetto**.

### I client di test

Per guidarla nel processo di sviluppo in `src/main/java/clients/` le sono
forniti molti test sotto forma di "client" che dovrà implementare (secondo le
specifiche che troverà nel Javadoc di ciascuno di essi) facendo uso delle classi
della sua soluzione (ed eventualmente di quelle del pacchetto di utilità); per
ciascun client, nella sottodirectory di `tests/clients/` avente nome uguale a
quello della classe client, si trovano un corredo di file di input e output che
le permetteranno di verificare il corretto funzionamento del suo codice.

Le viene inoltre fornita una mailbox di test `tests/mbox` che contiene alcuni
messaggi di esempio

```
tests/mbox/
├── first
│   ├── test-37524974
│   ├── test-9a611d5b
│   └── third
│       ├── test-3e05d11e
│       ├── test-4e0f2ae1
│       └── test-92659d74
├── second
│   └── test-a0cbd389
├── test-219c4395
├── test-311a171a
└── test-bcca12f2
```

Alcuni dei test forniti scrivono e cancellano messaggi in tale directory; in
caso che un malfunzionamento alteri il contenuto di tale mailbox può
ripristinarne il contenuto originale con il comando `git checkout -- tests/mbox`
(da eseguire dopo aver eventualmente cancellato la directory per eliminare file
generati, ma non correttamente cancellati, dai test).

Si ricorda che **il progetto non sarà valutato a meno che tutti i test diano
esito positivo** e che il contenuto dei sorgenti che presentano errori di
compilazione non sarà affatto esaminato.

### Codice di condotta

Dovendo svolgere il progetto a casa non le vengono imposte particolari
restrizioni delle quali sarebbe peraltro difficile verificare il rispetto.

Le è pertanto **consentito** di avvalersi:

* di qualunque risorsa disponibile in rete,
* di strumenti di supporto basati sull'AI (come [GitHub
  Copilot](https://github.com/features/copilot), o
  [ChatGPT](https://chat.openai.com/)),
* del *confronto* con altri studenti, o professionisti,

sia per la *progettazione* che per l'*implementazione* e *documentazione* del
codice. Ogni supporto che la aiuti a apprendere e dominare gli obiettivi
culturali dell'insegnamento è benvenuto!

D'altro canto le viene  **formalmente richiesto di elencare** (nella
documentazione del codice) in modo chiaro ed esaustivo **ogni risorsa di cui si
è avvalso al di fuori di quelle esplicitamente indicate come materiale
didattico** dell'insegnamento.

Si sottolinea che consegnando il progetto lei dichiara di fatto di esserne
l'**unico autore**, assumendosi la piena responsabilità dell'originalità del
codice e della documentazione che esso include, nonché della completezza e
veridicità del suddetto elenco. 

Durante la discussione orale, eventuali incertezze nell'*illustrare*,
*giustificare* o *modificare* il materiale consegnato non potranno che essere a
lei esclusivamente addebitate e, come tali, **valutate negativamente**.

### Note legali e copyright

Ai sensi della Legge n. 633/1941 e successive modificazioni, l'autore si
riserva, in ogni forma e modo nei limiti fissati dalla legge, il diritto
esclusivo di pubblicare e di utilizzare il materiale contenuto nel presente
repository.

Più specificatamente, è fatto **divieto di riprodurre, trascrivere, comunicare
al pubblico, distribuire, tradurre, elaborare e modificare il presente
materiale**, in tutto o in parte, senza specifica autorizzazione scritta
dell'autore.
