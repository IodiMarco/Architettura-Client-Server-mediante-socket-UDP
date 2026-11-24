# Servizio meteo client-server UDP

Applicazione client-server scritta in C che simula un servizio meteo tramite datagrammi UDP. Il client invia una richiesta per una città italiana e il server restituisce un dato meteo casuale, oppure un codice di errore.

Il progetto è portabile su macOS, Linux e Windows (con Winsock) e contiene due progetti Eclipse CDT separati: uno per il client e uno per il server.

## Funzionamento

1. Il server apre una socket UDP e rimane in ascolto, per impostazione predefinita, sulla porta `56700`.
2. Il client risolve l'host indicato tramite DNS, invia un datagramma con il tipo di dato richiesto e la città.
3. Il server registra mittente, tipo e città; valida la richiesta e restituisce un datagramma di risposta.
4. Il client effettua il reverse lookup dell'indirizzo del server e stampa il risultato in italiano.

Le richieste sono indipendenti: non viene aperta una connessione persistente tra client e server.

## Dati disponibili

| Codice | Dato restituito | Intervallo generato |
| --- | --- | --- |
| `t` | Temperatura | da -10.0 a 40.0 °C |
| `h` | Umidità | da 20.0 a 100.0 % |
| `w` | Vento | da 0.0 a 100.0 km/h |
| `p` | Pressione | da 950.0 a 1050.0 hPa |

Il valore viene generato casualmente a ogni richiesta valida.

Le città riconosciute, senza distinzione tra maiuscole e minuscole, sono: Bari, Roma, Milano, Napoli, Torino, Palermo, Genova, Bologna, Firenze e Venezia.

## Protocollo UDP

Il codice non trasmette direttamente le strutture C: serializza manualmente i campi in buffer separati.

**Richiesta** (65 byte):

```text
+--------+---------------------------+
|  type  | city (64 byte, terminata) |
+--------+---------------------------+
```

**Risposta** (9 byte):

```text
+----------------------+--------+-------------------+
| status (uint32_t)    |  type  | value (float)     |
+----------------------+--------+-------------------+
```

I campi numerici `status` e `value` sono convertiti in network byte order. I possibili stati sono:

- `0`: richiesta completata;
- `1`: città non disponibile;
- `2`: richiesta non valida.

## Compilazione

È possibile importare le cartelle `client-project` e `server-project` in Eclipse CDT oppure compilare dal terminale.

Su macOS/Linux:

```bash
cc -Wall -Wextra -o server-project/server-project server-project/src/main.c
cc -Wall -Wextra -o client-project/client-project client-project/src/main.c
```

Su Windows usare MinGW e collegare Winsock:

```bash
gcc -o server-project.exe server-project/src/main.c -lws2_32
gcc -o client-project.exe client-project/src/main.c -lws2_32
```

## Avvio

Avviare prima il server in un terminale:

```bash
./server-project/server-project
```

Per usare una porta diversa:

```bash
./server-project/server-project -p 56701
```

In un secondo terminale, inviare una richiesta dal client:

```bash
./client-project/client-project -r "t bari"
```

Sintassi completa del client:

```text
./client-project/client-project [-s server] [-p port] -r "tipo città"
```

- `-s server`: hostname o indirizzo IPv4 del server; valore predefinito `localhost`.
- `-p port`: porta UDP del server; valore predefinito `56700`.
- `-r`: richiesta obbligatoria nel formato `tipo città`.

Esempi:

```bash
./client-project/client-project -r "h napoli"
./client-project/client-project -s 127.0.0.1 -r "w milano"
./client-project/client-project -s localhost -p 56701 -r "p roma"
```

Esempio di risposta:

```text
Ricevuto risultato dal server localhost (ip 127.0.0.1). Bari: Temperatura = 22.5°C
```

Sul server viene mostrato un log per ogni datagramma ricevuto, ad esempio:

```text
Richiesta ricevuta da localhost (ip 127.0.0.1): type='t', city='bari'
```

Per fermare il server usare `Ctrl+C`.

## Validazione ed errori

Il client rifiuta localmente una richiesta senza uno spazio tra tipo e città, con un tipo composto da più di un carattere, con tabulazioni o con un nome città oltre 63 caratteri.

Il server risponde con `Richiesta non valida` se il tipo non è `t`, `h`, `w` o `p`, oppure se la città contiene tabulazioni o caratteri non alfanumerici (sono ammessi anche spazio, apostrofo e trattino). Per una città valida ma non presente nell'elenco, risponde con `Città non disponibile`.

## Struttura

```text
.
├── client-project/
│   └── src/
│       ├── main.c       # parsing argomenti, invio richiesta e stampa risposta
│       └── protocol.h   # protocollo, costanti e compatibilità piattaforme
├── server-project/
│   └── src/
│       ├── main.c       # ricezione, validazione e generazione dati meteo
│       └── protocol.h   # protocollo condiviso con il client
└── Assegnazione.md      # testo completo della consegna
```

## Limiti attuali

UDP non garantisce consegna, ordine o ritrasmissione dei datagrammi. Inoltre il client attende la risposta senza timeout: se il server non è raggiungibile, può rimanere in attesa.

## Licenza

Consultare [LICENSE](LICENSE).
