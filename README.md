# EarthBeat

EarthBeat e' un progetto per governare elettrovalvole e sensori agricoli usando Raspberry Pi 3 B+ collegati tra loro in una rete Wi-Fi mesh software, con un nodo master che orchestra anche se stesso come primo slave.

## Obiettivo

Il sistema deve permettere a uno smartphone di collegarsi alla rete esposta dai Raspberry e raggiungere il master anche passando da uno slave, senza access point fisici esterni. Il master deve mostrare una web app locale per controllare tutti i nodi, leggere umidita' e pH del terreno, comandare relay ed eseguire automazioni.

Vincoli iniziali:

- Raspberry Pi 3 B+.
- Solo Wi-Fi integrato, senza chiavette Wi-Fi esterne.
- Raspberry Pi OS, senza OpenWrt.
- Meno dipendenze possibili.
- Funzionamento offline, senza necessita' di Internet.
- Ruolo master/slave selezionato tramite due pin GPIO.
- Il master e' anche un nodo operativo locale, quindi controlla relay e sensori come gli slave.

## Nota tecnica sulla mesh

La versione iniziale prova a ottenere una mesh vera usando lo stesso adattatore Wi-Fi integrato dei Raspberry Pi. Questo e' il punto piu' rischioso del progetto: una singola interfaccia Wi-Fi deve gestire sia il collegamento mesh tra nodi sia, idealmente, l'accesso dello smartphone.

Roadmap realistica:

1. Prima validare una mesh tra Raspberry con `batman-adv` su Raspberry Pi OS.
2. Poi verificare se la stessa interfaccia puo' esporre anche un accesso utilizzabile per smartphone.
3. Se questa configurazione non sara' stabile, mantenere invariata l'architettura applicativa e rivedere solo il layer rete.

Il software applicativo deve quindi essere progettato in modo indipendente dalla rete: se in futuro si passa a chiavette Wi-Fi, OpenWrt o altra topologia, API, MQTT e logica dei nodi non devono cambiare in modo sostanziale.

## Architettura prevista

```text
Smartphone
   |
Nodo mesh raggiungibile
   |
Mesh Wi-Fi tra Raspberry
   |
Master orchestratore
```

Ogni nodo esegue lo stesso software e decide il ruolo all'avvio leggendo due GPIO.

```text
GPIO A | GPIO B | Ruolo
0      | 0      | Slave mesh
0      | 1      | Master + slave locale
1      | 0      | Manutenzione
1      | 1      | Setup/reset configurazione
```

### Master

- Partecipa alla mesh.
- Espone la web app per smartphone.
- Esegue il broker MQTT locale.
- Mantiene registry dei nodi.
- Salva configurazioni, stato e storico.
- Controlla relay e sensori locali.

### Slave

- Partecipa alla mesh.
- Legge sensori locali.
- Comanda relay locali.
- Pubblica telemetria al master.
- Riceve comandi dal master.
- Deve andare in stato sicuro se perde la connessione.

## Stack iniziale

Scelte conservative per ridurre dipendenze:

- Sistema operativo: Raspberry Pi OS Lite.
- Rete mesh: `batman-adv` e strumenti Linux standard.
- Backend master: Python.
- API/web UI: FastAPI oppure Flask, da scegliere dopo il prototipo rete.
- Messaggi master/slave: MQTT con Mosquitto.
- Persistenza: SQLite.
- GPIO: `gpiozero` o libreria GPIO disponibile sulla versione di Raspberry Pi OS scelta.
- ADC per sensori analogici: ADS1115 o MCP3008.

Per l'MVP si puo' anche rimandare MQTT e usare HTTP tra master e slave, ma MQTT resta preferibile appena ci sono piu' nodi perche' semplifica heartbeat, telemetria e comandi.

## Roadmap

### Fase 0 - Repository e documentazione

- Inizializzare repository Git.
- Documentare vincoli, architettura, rischi e roadmap.
- Preparare handoff per riprendere il progetto con Codex.

Stato: in corso.

### Fase 1 - Prova rete mesh minima

Obiettivo: dimostrare che tre Raspberry Pi 3 B+ comunicano in multi-hop senza AP esterni.

Attivita':

- Installare Raspberry Pi OS Lite su 3 nodi.
- Configurare hostname univoci, per esempio `earthbeat-master`, `earthbeat-node-01`, `earthbeat-node-02`.
- Abilitare `batman-adv`.
- Configurare interfaccia Wi-Fi in modalita' ad-hoc/mesh compatibile.
- Creare interfaccia `bat0`.
- Assegnare IP statici.
- Verificare ping diretto e multi-hop.
- Misurare stabilita', latenza e riconvergenza.

Criterio di successo:

```text
node-02 -> node-01 -> master
```

Il master deve essere raggiungibile anche quando il nodo piu' lontano non vede direttamente il master.

### Fase 2 - Accesso smartphone

Obiettivo: collegare uno smartphone da un punto qualsiasi della rete.

Attivita':

- Verificare se la configurazione con singola Wi-Fi permette un SSID accessibile al telefono.
- Testare raggiungibilita' della web UI del master da smartphone.
- Documentare limitazioni reali di roaming, banda e stabilita'.

Criterio di successo:

```text
Smartphone -> nodo vicino -> mesh -> master web UI
```

Se non e' fattibile con una sola interfaccia Wi-Fi, questa fase deve produrre una decisione tecnica documentata, non una riscrittura applicativa.

### Fase 3 - Nodo hardware locale

Obiettivo: controllare un nodo singolo senza rete.

Attivita':

- Leggere due GPIO per determinare il ruolo.
- Controllare almeno un relay.
- Leggere un sensore umidita'.
- Leggere pH tramite ADC.
- Esporre stato locale.
- Definire fail-safe per relay e valvole.

Criterio di successo:

- Il nodo legge sensori.
- Il nodo apre/chiude una valvola.
- Al riavvio torna in stato sicuro.

### Fase 4 - Master orchestratore

Obiettivo: creare il primo controllo centralizzato.

Attivita':

- Web UI locale.
- API per nodi e smartphone.
- Database SQLite.
- Registro nodi.
- Stato online/offline.
- Comandi manuali per valvole.
- Storico misure.

Criterio di successo:

- Lo smartphone vede tutti i nodi registrati.
- Lo smartphone comanda una valvola locale sul master.
- Lo smartphone visualizza umidita' e pH del master.

### Fase 5 - Comunicazione master/slave

Obiettivo: collegare slave reali al master.

Attivita':

- Definire protocollo MQTT o HTTP.
- Implementare heartbeat.
- Implementare pubblicazione sensori.
- Implementare comandi relay.
- Gestire timeout e perdita connessione.

Topic MQTT proposti:

```text
earthbeat/nodes/<node_id>/status
earthbeat/nodes/<node_id>/sensors
earthbeat/nodes/<node_id>/commands
earthbeat/nodes/<node_id>/events
```

### Fase 6 - Automazioni

Obiettivo: rendere il sistema utile senza intervento continuo.

Attivita':

- Soglie umidita' per zona.
- Soglie pH.
- Finestre orarie.
- Durata massima irrigazione.
- Blocco per sensore non valido.
- Modalita' manuale/automatica.
- Log decisioni automatiche.

### Fase 7 - Robustezza di campo

Obiettivo: preparare il sistema per uso reale.

Attivita':

- Watchdog servizi.
- Backup configurazione.
- Recovery mode via GPIO.
- Pagina diagnostica rete.
- Qualita' link mesh.
- Protezione da relay bloccato.
- Gestione perdita alimentazione.
- Procedure di aggiornamento.

## Sicurezza e fail-safe

Le elettrovalvole devono chiudersi in modo sicuro in caso di:

- riavvio Raspberry;
- perdita connessione master;
- processo applicativo bloccato;
- sensore non valido;
- durata apertura oltre limite;
- alimentazione instabile.

Il progetto deve trattare ogni comando di apertura valvola come temporaneo: ogni apertura deve avere una durata massima esplicita.

## Domande aperte

- Numero di nodi per il primo prototipo.
- Distanza reale tra i nodi.
- Tipo elettrovalvole: 12V DC, 24V AC, 230V AC o altro.
- Modello sensori umidita' e pH.
- ADC scelto.
- Se la web UI deve funzionare solo in LAN o anche esportare dati quando Internet e' disponibile.

## Stato corrente

Il progetto e' nella fase di definizione. La prima milestone tecnica vera e' la validazione della mesh su Raspberry Pi OS usando il solo Wi-Fi integrato.
