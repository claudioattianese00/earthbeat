# EarthBeat Handoff per Codex

## Contesto

Il progetto EarthBeat serve a costruire una rete di Raspberry Pi 3 B+ per controllare elettrovalvole via relay e leggere sensori di umidita' e pH del terreno. Il master orchestra tutti i nodi, ma deve funzionare anche come primo slave locale.

L'utente vuole una mesh vera a regime: uno smartphone dovra' potersi collegare da qualunque nodo disponibile e raggiungere il master attraverso la rete tra Raspberry, senza access point fisici esterni.

Vincolo aggiornato: per il momento tutto deve lavorare sulla stessa Wi-Fi integrata dei Raspberry, senza chiavette esterne e senza OpenWrt. Preferire meno dipendenze possibili.

Priorita' aggiornata: la mesh va introdotta dopo. La roadmap deve partire dalla creazione minimale degli script per configurazione nodo, lettura dati e attivazione relay.

## Decisioni prese

- Repository Git inizializzato nel workspace.
- Branch principale: `main`.
- Prima documentazione in `README.md`.
- Questo file serve per riprendere il lavoro in qualsiasi momento.
- Sistema target iniziale: Raspberry Pi OS Lite.
- Rete target futura: mesh software con strumenti Linux e `batman-adv`, non OpenWrt.
- Architettura applicativa separata dal layer rete, cosi' si puo' sviluppare prima il nodo locale e introdurre la mesh in seguito senza riscrivere master/slave.
- Prima milestone tecnica: script minimi per ruolo GPIO, relay, sensori e fail-safe locale.

## Assunzioni operative

- Ogni Raspberry esegue lo stesso software.
- Due GPIO decidono il ruolo all'avvio.
- Il master avvia anche i servizi centrali.
- Tutti i nodi controllano hardware locale.
- Ogni comando valvola deve avere durata massima e fail-safe.
- Il funzionamento deve essere offline-first.

Mappa ruolo proposta:

```text
GPIO A | GPIO B | Ruolo
0      | 0      | Slave mesh
0      | 1      | Master + slave locale
1      | 0      | Manutenzione
1      | 1      | Setup/reset configurazione
```

## Rischio principale

Il rischio tecnico centrale a regime resta la rete: ottenere una mesh vera e anche accesso smartphone usando una sola interfaccia Wi-Fi integrata sul Raspberry Pi 3 B+ potrebbe essere instabile o non supportato bene dai driver.

Pero' non va affrontato per primo. La prossima milestone deve essere applicativa/hardware locale: configurazione nodo, lettura sensori e relay sicuri.

## Prossimo lavoro consigliato

1. Creare struttura minima `src/`, `scripts/`, `config/` e `docs/`.
2. Creare script per leggere i due GPIO di ruolo.
3. Creare script per attivare/disattivare relay con timeout obbligatorio.
4. Creare script per leggere sensori, con output testuale e JSON.
5. Aggiungere backend mock per testare su computer senza GPIO reali.
6. Solo dopo, introdurre master locale, comunicazione tra nodi e infine mesh.

## Convenzioni proposte

Comandi/script iniziali:

```text
scripts/node_status.py
scripts/relay_on.py
scripts/relay_off.py
scripts/relay_pulse.py
scripts/read_sensors.py
```

Output sensori proposto:

```json
{
  "node_id": "earthbeat-master",
  "role": "master_local_slave",
  "soil_moisture": 42.5,
  "ph": 6.8,
  "relays": {
    "valve_1": "off"
  }
}
```

Hostname:

```text
earthbeat-master
earthbeat-node-01
earthbeat-node-02
```

IP mesh:

```text
10.42.0.1   master
10.42.0.11  node-01
10.42.0.12  node-02
```

Servizi master futuri:

```text
Web UI/API    http://10.42.0.1
MQTT          10.42.0.1:1883
SQLite        file locale sul master
```

Topic MQTT proposti:

```text
earthbeat/nodes/<node_id>/status
earthbeat/nodes/<node_id>/sensors
earthbeat/nodes/<node_id>/commands
earthbeat/nodes/<node_id>/events
```

## Struttura repository proposta

Da creare quando si inizia il codice:

```text
earthbeat/
  README.md
  HANDOFF.md
  docs/
    hardware.md
    gpio-map.md
    network-mesh-rpi-os.md
  config/
    node.example.json
  scripts/
    install/
    node_status.py
    relay_on.py
    relay_off.py
    relay_pulse.py
    read_sensors.py
    network/
  src/
    earthbeat/
      common/
      node/
      master/
      hardware/
  tests/
```

## Principi di implementazione

- Prima validare hardware locale, poi rete.
- Tenere master e slave nello stesso pacchetto software.
- Evitare dipendenze finche' non servono.
- Preferire file di configurazione semplici e leggibili.
- Non rendere permanente un comando valvola senza timeout.
- Scrivere log comprensibili per debugging sul campo.
- Documentare ogni comando di setup testato su Raspberry reale.

## Checklist ripresa lavoro

Quando Codex riprende il progetto:

1. Leggere `README.md`.
2. Leggere questo `HANDOFF.md`.
3. Controllare `git status`.
4. Chiedere se l'utente ha gia' hardware disponibile per test GPIO, relay e sensori.
5. Se si lavora sul codice, partire dagli script locali minimi.
6. Non assumere che la mesh sia gia' validata.
7. Se si lavora sulla rete, documentare comandi e risultati in `docs/`.

## Stato attuale

Solo documentazione iniziale e repository Git. Nessun codice applicativo ancora creato.
