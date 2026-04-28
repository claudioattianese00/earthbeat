<div align="center">
  <img src="assets/img/logo_earthbeat.png" alt="EarthBeat logo" width="200" height="200">
  <h1>EarthBeat</h1>
</div>

<p>
  EarthBeat e' un progetto per governare elettrovalvole e sensori agricoli usando Raspberry Pi 3 B+ collegati tra loro in una rete Wi-Fi mesh software, con un nodo master che orchestra anche se stesso come primo slave.
</p>

<h2>Obiettivo</h2>

<p>
  Il sistema deve permettere a uno smartphone di collegarsi alla rete esposta dai Raspberry e raggiungere il master anche passando da uno slave, senza access point fisici esterni. Il master deve mostrare una web app locale per controllare tutti i nodi, leggere umidita' e pH del terreno, comandare relay ed eseguire automazioni.
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

<h2>Nota tecnica sulla mesh</h2>

<p>
  La versione iniziale prova a ottenere una mesh vera usando lo stesso adattatore Wi-Fi integrato dei Raspberry Pi. Questo e' il punto piu' rischioso del progetto: una singola interfaccia Wi-Fi deve gestire sia il collegamento mesh tra nodi sia, idealmente, l'accesso dello smartphone.
</p>

<h3>Roadmap realistica della rete</h3>

<ol>
  <li>Prima validare una mesh tra Raspberry con <code>batman-adv</code> su Raspberry Pi OS.</li>
  <li>Poi verificare se la stessa interfaccia puo' esporre anche un accesso utilizzabile per smartphone.</li>
  <li>Se questa configurazione non sara' stabile, mantenere invariata l'architettura applicativa e rivedere solo il layer rete.</li>
</ol>

<p>
  Il software applicativo deve quindi essere progettato in modo indipendente dalla rete: se in futuro si passa a chiavette Wi-Fi, OpenWrt o altra topologia, API, MQTT e logica dei nodi non devono cambiare in modo sostanziale.
</p>

<h2>Architettura prevista</h2>

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
  <li>Partecipa alla mesh.</li>
  <li>Espone la web app per smartphone.</li>
  <li>Esegue il broker MQTT locale.</li>
  <li>Mantiene registry dei nodi.</li>
  <li>Salva configurazioni, stato e storico.</li>
  <li>Controlla relay e sensori locali.</li>
</ul>

<h3>Slave</h3>

<ul>
  <li>Partecipa alla mesh.</li>
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
  <li>Rete mesh: <code>batman-adv</code> e strumenti Linux standard.</li>
  <li>Backend master: Python.</li>
  <li>API/web UI: FastAPI oppure Flask, da scegliere dopo il prototipo rete.</li>
  <li>Messaggi master/slave: MQTT con Mosquitto.</li>
  <li>Persistenza: SQLite.</li>
  <li>GPIO: <code>gpiozero</code> o libreria GPIO disponibile sulla versione di Raspberry Pi OS scelta.</li>
  <li>ADC per sensori analogici: ADS1115 o MCP3008.</li>
</ul>

<p>
  Per l'MVP si puo' anche rimandare MQTT e usare HTTP tra master e slave, ma MQTT resta preferibile appena ci sono piu' nodi perche' semplifica heartbeat, telemetria e comandi.
</p>

<h2>Roadmap</h2>

<h3>Fase 0 - Repository e documentazione</h3>

<ul>
  <li>Inizializzare repository Git.</li>
  <li>Documentare vincoli, architettura, rischi e roadmap.</li>
  <li>Preparare handoff per riprendere il progetto con Codex.</li>
</ul>

<p><strong>Stato:</strong> in corso.</p>

<h3>Fase 1 - Prova rete mesh minima</h3>

<p>
  <strong>Obiettivo:</strong> dimostrare che tre Raspberry Pi 3 B+ comunicano in multi-hop senza AP esterni.
</p>

<p><strong>Attivita':</strong></p>

<ul>
  <li>Installare Raspberry Pi OS Lite su 3 nodi.</li>
  <li>Configurare hostname univoci, per esempio <code>earthbeat-master</code>, <code>earthbeat-node-01</code>, <code>earthbeat-node-02</code>.</li>
  <li>Abilitare <code>batman-adv</code>.</li>
  <li>Configurare interfaccia Wi-Fi in modalita' ad-hoc/mesh compatibile.</li>
  <li>Creare interfaccia <code>bat0</code>.</li>
  <li>Assegnare IP statici.</li>
  <li>Verificare ping diretto e multi-hop.</li>
  <li>Misurare stabilita', latenza e riconvergenza.</li>
</ul>

<p><strong>Criterio di successo:</strong></p>

<pre><code>node-02 -&gt; node-01 -&gt; master</code></pre>

<p>
  Il master deve essere raggiungibile anche quando il nodo piu' lontano non vede direttamente il master.
</p>

<h3>Fase 2 - Accesso smartphone</h3>

<p>
  <strong>Obiettivo:</strong> collegare uno smartphone da un punto qualsiasi della rete.
</p>

<p><strong>Attivita':</strong></p>

<ul>
  <li>Verificare se la configurazione con singola Wi-Fi permette un SSID accessibile al telefono.</li>
  <li>Testare raggiungibilita' della web UI del master da smartphone.</li>
  <li>Documentare limitazioni reali di roaming, banda e stabilita'.</li>
</ul>

<p><strong>Criterio di successo:</strong></p>

<pre><code>Smartphone -&gt; nodo vicino -&gt; mesh -&gt; master web UI</code></pre>

<p>
  Se non e' fattibile con una sola interfaccia Wi-Fi, questa fase deve produrre una decisione tecnica documentata, non una riscrittura applicativa.
</p>

<h3>Fase 3 - Nodo hardware locale</h3>

<p>
  <strong>Obiettivo:</strong> controllare un nodo singolo senza rete.
</p>

<p><strong>Attivita':</strong></p>

<ul>
  <li>Leggere due GPIO per determinare il ruolo.</li>
  <li>Controllare almeno un relay.</li>
  <li>Leggere un sensore umidita'.</li>
  <li>Leggere pH tramite ADC.</li>
  <li>Esporre stato locale.</li>
  <li>Definire fail-safe per relay e valvole.</li>
</ul>

<p><strong>Criterio di successo:</strong></p>

<ul>
  <li>Il nodo legge sensori.</li>
  <li>Il nodo apre/chiude una valvola.</li>
  <li>Al riavvio torna in stato sicuro.</li>
</ul>

<h3>Fase 4 - Master orchestratore</h3>

<p>
  <strong>Obiettivo:</strong> creare il primo controllo centralizzato.
</p>

<p><strong>Attivita':</strong></p>

<ul>
  <li>Web UI locale.</li>
  <li>API per nodi e smartphone.</li>
  <li>Database SQLite.</li>
  <li>Registro nodi.</li>
  <li>Stato online/offline.</li>
  <li>Comandi manuali per valvole.</li>
  <li>Storico misure.</li>
</ul>

<p><strong>Criterio di successo:</strong></p>

<ul>
  <li>Lo smartphone vede tutti i nodi registrati.</li>
  <li>Lo smartphone comanda una valvola locale sul master.</li>
  <li>Lo smartphone visualizza umidita' e pH del master.</li>
</ul>

<h3>Fase 5 - Comunicazione master/slave</h3>

<p>
  <strong>Obiettivo:</strong> collegare slave reali al master.
</p>

<p><strong>Attivita':</strong></p>

<ul>
  <li>Definire protocollo MQTT o HTTP.</li>
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

<h3>Fase 6 - Automazioni</h3>

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

<h3>Fase 7 - Robustezza di campo</h3>

<p>
  <strong>Obiettivo:</strong> preparare il sistema per uso reale.
</p>

<p><strong>Attivita':</strong></p>

<ul>
  <li>Watchdog servizi.</li>
  <li>Backup configurazione.</li>
  <li>Recovery mode via GPIO.</li>
  <li>Pagina diagnostica rete.</li>
  <li>Qualita' link mesh.</li>
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
  Il progetto e' nella fase di definizione. La prima milestone tecnica vera e' la validazione della mesh su Raspberry Pi OS usando il solo Wi-Fi integrato.
</p>
