<div align="center">
  <img src="assets/img/logo_earthbeat.png" alt="EarthBeat logo" width="200" height="200">
</div>

<p>
  EarthBeat e' un progetto per governare elettrovalvole e sensori agricoli usando Raspberry Pi 3 B+, con un nodo master che orchestra anche se stesso come primo slave. La rete mesh e' un obiettivo del progetto, ma verra' introdotta dopo un primo MVP locale basato su script minimi per configurazione, lettura sensori e attivazione relay.
</p>

<h2>Obiettivo</h2>

<p>
  Il sistema deve permettere di configurare un Raspberry come master o slave tramite due pin GPIO, leggere dati da sensori di umidita' e pH del terreno, comandare elettrovalvole tramite relay e applicare fail-safe locali. In una fase successiva lo smartphone dovra' potersi collegare alla rete esposta dai Raspberry e raggiungere il master anche passando da uno slave, senza access point fisici esterni.
</p>

<h3>Vincoli iniziali</h3>

<ul>
  <li>Raspberry Pi 3 B+.</li>
  <li>Solo Wi-Fi integrato, senza chiavette Wi-Fi esterne.</li>
  <li>Raspberry Pi OS, senza OpenWrt.</li>
  <li>Meno dipendenze possibili.</li>
  <li>Funzionamento offline, senza necessita' di Internet.</li>
  <li>Ruolo master/slave selezionato tramite due pin GPIO.</li>
  <li>Il master e' anche un nodo operativo locale, quindi controlla relay e sensori come gli slave.</li>
</ul>

<h2>Approccio iniziale</h2>

<p>
  La prima versione non parte dalla rete mesh. Parte invece dal controllo affidabile del singolo nodo: configurazione, GPIO, relay, sensori, log e fail-safe. Questa scelta permette di validare subito la parte fisica del progetto e di ridurre le variabili prima di introdurre il layer rete.
</p>

<p>
  Il software applicativo deve restare indipendente dalla rete: master, slave, letture e comandi devono poter funzionare prima in locale, poi via collegamento diretto, e infine sopra una mesh vera.
</p>

<h2>Nota tecnica sulla mesh futura</h2>

<p>
  La mesh resta un requisito di progetto, ma non e' la prima milestone. Quando verra' introdotta, la versione desiderata provera' a ottenere una mesh vera usando lo stesso adattatore Wi-Fi integrato dei Raspberry Pi. Questo e' il punto piu' rischioso del progetto: una singola interfaccia Wi-Fi deve gestire sia il collegamento mesh tra nodi sia, idealmente, l'accesso dello smartphone.
</p>

<h3>Roadmap realistica della rete futura</h3>

<ol>
  <li>Prima validare una mesh tra Raspberry con <code>batman-adv</code> su Raspberry Pi OS.</li>
  <li>Poi verificare se la stessa interfaccia puo' esporre anche un accesso utilizzabile per smartphone.</li>
  <li>Se questa configurazione non sara' stabile, mantenere invariata l'architettura applicativa e rivedere solo il layer rete.</li>
</ol>

<p>
  Se in futuro si passa a chiavette Wi-Fi, OpenWrt o altra topologia, API, MQTT e logica dei nodi non devono cambiare in modo sostanziale.
</p>

<h2>Architettura prevista a regime</h2>

<pre><code>Smartphone
   |
Nodo mesh raggiungibile
   |
Mesh Wi-Fi tra Raspberry
   |
Master orchestratore</code></pre>

<p>
  Ogni nodo esegue lo stesso software e decide il ruolo all'avvio leggendo due GPIO.
</p>

<table>
  <thead>
    <tr>
      <th>GPIO A</th>
      <th>GPIO B</th>
      <th>Ruolo</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>0</td>
      <td>Slave mesh</td>
    </tr>
    <tr>
      <td>0</td>
      <td>1</td>
      <td>Master + slave locale</td>
    </tr>
    <tr>
      <td>1</td>
      <td>0</td>
      <td>Manutenzione</td>
    </tr>
    <tr>
      <td>1</td>
      <td>1</td>
      <td>Setup/reset configurazione</td>
    </tr>
  </tbody>
</table>

<h3>Master</h3>

<ul>
  <li>Partecipa alla rete dei nodi, prima in locale/test e poi in mesh.</li>
  <li>Espone la web app per smartphone.</li>
  <li>Esegue il broker MQTT locale.</li>
  <li>Mantiene registry dei nodi.</li>
  <li>Salva configurazioni, stato e storico.</li>
  <li>Controlla relay e sensori locali.</li>
</ul>

<h3>Slave</h3>

<ul>
  <li>Partecipa alla rete dei nodi, prima in locale/test e poi in mesh.</li>
  <li>Legge sensori locali.</li>
  <li>Comanda relay locali.</li>
  <li>Pubblica telemetria al master.</li>
  <li>Riceve comandi dal master.</li>
  <li>Deve andare in stato sicuro se perde la connessione.</li>
</ul>

<h2>Stack iniziale</h2>

<p>Scelte conservative per ridurre dipendenze:</p>

<ul>
  <li>Sistema operativo: Raspberry Pi OS Lite.</li>
  <li>Backend master: Python.</li>
  <li>Script minimi: Python standard library dove possibile.</li>
  <li>API/web UI: FastAPI oppure Flask, da introdurre dopo il prototipo hardware.</li>
  <li>Messaggi master/slave: MQTT con Mosquitto, da introdurre dopo i test locali.</li>
  <li>Persistenza: SQLite.</li>
  <li>GPIO: <code>gpiozero</code> o libreria GPIO disponibile sulla versione di Raspberry Pi OS scelta.</li>
  <li>ADC per sensori analogici: ADS1115 o MCP3008.</li>
  <li>Rete mesh futura: <code>batman-adv</code> e strumenti Linux standard.</li>
</ul>

<p>
  Per l'MVP si rimandano MQTT, web UI e mesh. Prima devono esistere script locali affidabili per configurare il nodo, leggere i dati e attivare i relay in sicurezza.
</p>

<h2>Roadmap</h2>

<h3>Fase 0 - Repository e documentazione</h3>

<ul>
  <li>Inizializzare repository Git.</li>
  <li>Documentare vincoli, architettura, rischi e roadmap.</li>
  <li>Preparare handoff per riprendere il progetto con Codex.</li>
</ul>

<p><strong>Stato:</strong> in corso.</p>

<h3>Fase 1 - Script minimi di configurazione nodo</h3>

<p>
  <strong>Obiettivo:</strong> creare una base software minima per identificare e configurare un nodo Raspberry senza dipendere dalla rete.
</p>

<p><strong>Attivita':</strong></p>

<ul>
  <li>Creare struttura iniziale <code>src/</code>, <code>scripts/</code> e <code>config/</code>.</li>
  <li>Definire un file di configurazione locale semplice, per esempio JSON o TOML.</li>
  <li>Creare uno script per leggere i due GPIO di ruolo.</li>
  <li>Creare uno script per stampare configurazione, hostname e ruolo rilevato.</li>
  <li>Definire convenzioni per <code>node_id</code>, nome zona e modalita' master/slave.</li>
  <li>Preparare documentazione per installazione su Raspberry Pi OS Lite.</li>
</ul>

<p><strong>Criterio di successo:</strong></p>

<pre><code>python scripts/node_status.py
# node_id=earthbeat-master
# role=master_local_slave</code></pre>

<p>
  Il nodo deve riconoscere il proprio ruolo e stampare uno stato leggibile senza servizi di rete.
</p>

<h3>Fase 2 - Attivazione relay</h3>

<p>
  <strong>Obiettivo:</strong> comandare in sicurezza almeno un relay collegato a una futura elettrovalvola.
</p>

<p><strong>Attivita':</strong></p>

<ul>
  <li>Definire mappa GPIO relay.</li>
  <li>Creare script <code>relay_on.py</code>, <code>relay_off.py</code> e <code>relay_pulse.py</code>.</li>
  <li>Imporre una durata massima per ogni apertura temporizzata.</li>
  <li>Garantire stato sicuro all'avvio e alla chiusura dello script.</li>
  <li>Loggare ogni cambio stato relay.</li>
</ul>

<p><strong>Criterio di successo:</strong></p>

<pre><code>python scripts/relay_pulse.py --relay valve_1 --seconds 5</code></pre>

<p>
  Il relay si attiva per il tempo richiesto e poi torna spento anche in caso di errore gestito.
</p>

<h3>Fase 3 - Lettura sensori</h3>

<p>
  <strong>Obiettivo:</strong> leggere umidita' e pH dal nodo locale con output normalizzato.
</p>

<p><strong>Attivita':</strong></p>

<ul>
  <li>Definire modello ADC: ADS1115 o MCP3008.</li>
  <li>Creare script per lettura grezza ADC.</li>
  <li>Creare script per lettura umidita' normalizzata.</li>
  <li>Creare script per lettura pH con parametri di calibrazione.</li>
  <li>Produrre output testuale e JSON.</li>
  <li>Segnalare valori fuori range o sensore non valido.</li>
</ul>

<p><strong>Criterio di successo:</strong></p>

<pre><code>python scripts/read_sensors.py --json
{
  "soil_moisture": 42.5,
  "ph": 6.8
}</code></pre>

<h3>Fase 4 - Nodo locale completo</h3>

<p>
  <strong>Obiettivo:</strong> unire ruolo, relay e sensori in un solo comando locale di stato e controllo.
</p>

<p><strong>Attivita':</strong></p>

<ul>
  <li>Creare comando unico <code>earthbeat-node</code> o script equivalente.</li>
  <li>Mostrare stato ruolo, sensori e relay.</li>
  <li>Eseguire comandi relay con timeout obbligatorio.</li>
  <li>Salvare log locali.</li>
  <li>Preparare test senza hardware usando backend mock.</li>
</ul>

<p><strong>Criterio di successo:</strong></p>

<ul>
  <li>Il nodo legge sensori.</li>
  <li>Il nodo apre/chiude una valvola.</li>
  <li>Al riavvio torna in stato sicuro.</li>
</ul>

<h3>Fase 5 - Master locale e prima interfaccia</h3>

<p>
  <strong>Obiettivo:</strong> trasformare il master in orchestratore locale, mantenendolo anche come primo slave.
</p>

<p><strong>Attivita':</strong></p>

<ul>
  <li>Creare servizio master locale.</li>
  <li>Usare SQLite per configurazione, stato e storico.</li>
  <li>Esporre API o CLI per leggere lo stato del master.</li>
  <li>Comandare i relay locali del master.</li>
  <li>Preparare una prima web UI minimale solo dopo che CLI e script sono stabili.</li>
</ul>

<h3>Fase 6 - Comunicazione master/slave</h3>

<p>
  <strong>Obiettivo:</strong> collegare slave reali al master.
</p>

<p><strong>Attivita':</strong></p>

<ul>
  <li>Definire protocollo MQTT o HTTP partendo dagli output JSON gia' creati.</li>
  <li>Implementare heartbeat.</li>
  <li>Implementare pubblicazione sensori.</li>
  <li>Implementare comandi relay.</li>
  <li>Gestire timeout e perdita connessione.</li>
</ul>

<p><strong>Topic MQTT proposti:</strong></p>

<pre><code>earthbeat/nodes/&lt;node_id&gt;/status
earthbeat/nodes/&lt;node_id&gt;/sensors
earthbeat/nodes/&lt;node_id&gt;/commands
earthbeat/nodes/&lt;node_id&gt;/events</code></pre>

<h3>Fase 7 - Automazioni</h3>

<p>
  <strong>Obiettivo:</strong> rendere il sistema utile senza intervento continuo.
</p>

<p><strong>Attivita':</strong></p>

<ul>
  <li>Soglie umidita' per zona.</li>
  <li>Soglie pH.</li>
  <li>Finestre orarie.</li>
  <li>Durata massima irrigazione.</li>
  <li>Blocco per sensore non valido.</li>
  <li>Modalita' manuale/automatica.</li>
  <li>Log decisioni automatiche.</li>
</ul>

<h3>Fase 8 - Rete mesh e accesso smartphone</h3>

<p>
  <strong>Obiettivo:</strong> introdurre la mesh vera solo dopo che il nodo locale e la comunicazione master/slave funzionano.
</p>

<p><strong>Attivita':</strong></p>

<ul>
  <li>Installare Raspberry Pi OS Lite su almeno 3 nodi.</li>
  <li>Configurare hostname univoci, per esempio <code>earthbeat-master</code>, <code>earthbeat-node-01</code>, <code>earthbeat-node-02</code>.</li>
  <li>Abilitare <code>batman-adv</code>.</li>
  <li>Configurare interfaccia Wi-Fi in modalita' ad-hoc/mesh compatibile.</li>
  <li>Creare interfaccia <code>bat0</code>.</li>
  <li>Assegnare IP statici.</li>
  <li>Verificare ping diretto e multi-hop.</li>
  <li>Testare accesso smartphone verso il master passando da un nodo vicino.</li>
  <li>Misurare stabilita', latenza e riconvergenza.</li>
</ul>

<p><strong>Criterio di successo:</strong></p>

<pre><code>Smartphone -&gt; nodo vicino -&gt; mesh -&gt; master web UI</code></pre>

<h3>Fase 9 - Robustezza di campo</h3>

<p>
  <strong>Obiettivo:</strong> preparare il sistema per uso reale.
</p>

<p><strong>Attivita':</strong></p>

<ul>
  <li>Watchdog servizi.</li>
  <li>Backup configurazione.</li>
  <li>Recovery mode via GPIO.</li>
  <li>Pagina diagnostica rete, quando la mesh sara' attiva.</li>
  <li>Qualita' link mesh, quando la mesh sara' attiva.</li>
  <li>Protezione da relay bloccato.</li>
  <li>Gestione perdita alimentazione.</li>
  <li>Procedure di aggiornamento.</li>
</ul>

<h2>Sicurezza e fail-safe</h2>

<p>Le elettrovalvole devono chiudersi in modo sicuro in caso di:</p>

<ul>
  <li>riavvio Raspberry;</li>
  <li>perdita connessione master;</li>
  <li>processo applicativo bloccato;</li>
  <li>sensore non valido;</li>
  <li>durata apertura oltre limite;</li>
  <li>alimentazione instabile.</li>
</ul>

<p>
  Il progetto deve trattare ogni comando di apertura valvola come temporaneo: ogni apertura deve avere una durata massima esplicita.
</p>

<h2>Domande aperte</h2>

<ul>
  <li>Numero di nodi per il primo prototipo.</li>
  <li>Distanza reale tra i nodi.</li>
  <li>Tipo elettrovalvole: 12V DC, 24V AC, 230V AC o altro.</li>
  <li>Modello sensori umidita' e pH.</li>
  <li>ADC scelto.</li>
  <li>Se la web UI deve funzionare solo in LAN o anche esportare dati quando Internet e' disponibile.</li>
</ul>

<h2>Stato corrente</h2>

<p>
  Il progetto e' nella fase di definizione. La prima milestone tecnica vera e' la creazione degli script minimi per configurare il nodo, leggere sensori e attivare relay in sicurezza. La rete mesh verra' introdotta dopo.
</p>

# Analisi dei Costi per la fase Pilota - Progetto EarthBeat
* La fase pilota considera l'impianto agricolo formato da 48 elettrovalvole collegate da 6 nodi (8 Elettrovalvole a nodo). In questa fase implementeremo anche i sensori di umidità e PH.
  
| Pezzo | Quantità | Prezzo Totale | Prezzo/pz | Link acquisto |
| :--- | :---: | :---: | :---: | :--- |
| **Elettrovalvola** | 50 | 686,00 € | 13,72 € | [Alibaba Link](https://www.alibaba.com/product-detail/1-Solenoid-Valve-24V-AC-_or_1601308197575.html) |
| **Raspberry Pi 3B PLUS generazione B** | 6 | 213,00 € | 35,50 € | [Alibaba Link](https://www.alibaba.com/product-detail/Raspberry-Pi-3B-PLUS-Generation-B_1600634669456.html) |
| **Modulo Relè a 8 Canali 24V** | 6 | 15,00 € | 2,50 € | [Alibaba Link](https://www.alibaba.com/product-detail/1-2-4-6-8-Channel_1601668673709.html) |
| **ADS1115 16 Bit ADC Module** | 6 | 9,00 € | 1,50 € | [Alibaba Link](https://www.alibaba.com/product-detail/Hot-Selling-16-Bit-I2C-ADS1115_62061259873.html) |
| **Sensori umidità** | 12 | 17,00 € | 1,42 € | [Alibaba Link](https://www.alibaba.com/product-detail/Capacitive-soil-moisture-sensor-Corrosion-Resistant_1601395418812.html) |
| **Sensori PH** | 6 | 6,00 € | 1,00 € | [Alibaba Link](https://www.alibaba.com/product-detail/TSLJSLY-RS485-4-20mA-Soil-Temperature_1601420836516.html) |
| **Trasformatori** | 6 | 78,00 € | 13,00 € | [Alibaba Link](https://www.alibaba.com/product-detail/IP67-Constant-Voltage-12V-24V-Transformer_1601444158296.html) |
| **Case** | 6 | 36,00 € | 6,00 € | [Alibaba Link](https://www.alibaba.com/product-detail/Electric-Cabinet-Fiberglass-Fiberglass-Project-Box_1601300142413.html) |
| **Cavi** | 246 | 344,40 € | 1,40 € | [Amazon Link](https://www.amazon.it/Zenitech-Alimentazione-Apparecchi-AllAbrasione/dp/B0716D7XZY/) |
| **7" Touchscreen Display (DSI)** | 1 | 43,00 € | 43,00 € | [Alibaba Link](https://www.alibaba.com/product-detail/2025-New-7inch-MIPI-DSI-800x480_62236144922.html?spm=a2700.prosearch.normal_offer.d_title.54b567af4orVeG&priceId=d9df9ef9f2f04c6298a859c767feca77) |
| **Modulo RTC DS3231 (Orologio)** | 1 | 2,00 € | 2,00 € | [Alibaba Link](https://www.alibaba.com/product-detail/AI-KSEN-New-and-Original-Clock_1601588222941.html?spm=a2700.prosearch.normal_offer.7.3be367afuuDF6K&selectedCarrierCode=SEMI_MANAGED_STANDARD%40%40STANDARD&priceId=fbf95cd7832a41909ad3c42087c1762b) |
| **MicroSD 32GB High Endurance** | 1 | 12,00 € | 12,00 € | [Amazon Link](https://www.amazon.it/SanDisk-Endurance-microSDHC-adattatore-Monitoraggio/dp/B07P14QHB7/) |
| **TOTALE** | | **1.476,40 €** | | |

---

**Note sul preventivo:**
* I costi tengono conto anche di eventuale spedizione per singolo pz; non è stata testata l'ipotesi di fare un'unica spedizione (potrebbero ridursi ulteriormente i costi).
* Da confermare i singoli pezzi elettronici (forse i sensori del PH non sono quelli adatti al caso)
