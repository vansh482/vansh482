<!-- <div align="center">
  <img src="banner.svg" width="850" alt="Vansh Gupta — backend & distributed-systems engineer @ Smartsheet" />
</div>

<br/>

<div align="center">

<details>
<summary><code>vansh@dsys ~ % open ./terminal-session  </code></summary>

<br/>

<img src="terminal.svg" width="850" alt="vansh@distributed-systems terminal — whoami, uptime, tech stack (Java, C++, Python, Spring Boot, Kafka, AWS, Docker, Kubernetes, Terraform, MySQL, Datadog), flex (Codeforces Expert 1600+, JEE Advanced AIR 9684), git log (Dynamics 365 + Trello connectors, Kafka+RocksDB replacing Aeron), and vansh --help (linkedin.com/in/vansh-vg, vansh012345gupta@gmail.com)" />

</details>

</div>

<br/>

<div align="center">
  <code>vansh@dsys ~ % ping me</code>
  <br/><br/>
  <a href="https://www.linkedin.com/in/vansh-vg/"><img src="badge-linkedin.svg" height="46" alt="LinkedIn — vansh-vg" /></a>
  &nbsp;&nbsp;
  <a href="mailto:vansh012345gupta@gmail.com"><img src="badge-email.svg" height="46" alt="Email — vansh012345gupta@gmail.com" /></a> -->
  <!-- &nbsp;&nbsp;
  <a href="https://codeforces.com/profile/YOUR_CF_HANDLE"><img src="badge-codeforces.svg" height="46" alt="Codeforces — Expert" /></a> -->
<!-- </div> -->

<img src="banner.svg" width="850" alt="Vansh Gupta — backend & distributed-systems engineer @ Smartsheet" />

<br/>
<br/>

<details>
<summary><code>vansh@dsys ~ % ./terminal-session</code> &nbsp; <sub>click to toggle</sub></summary>

<br/>

<img src="terminal.svg" width="850" alt="vansh@distributed-systems terminal — whoami, uptime, tech stack (Java, C++, Python, Spring Boot, Kafka, AWS, Docker, Kubernetes, Terraform, MySQL, Datadog), flex (Codeforces Expert 1600+, JEE Advanced AIR 9684), git log (Dynamics 365 + ServiceNow connectors on Nango, Kafka+RocksDB replacing Aeron), and vansh --help (linkedin.com/in/vansh-vg, vansh012345gupta@gmail.com)" />

</details>

<details>
<summary><code>vansh@dsys ~ % ping me</code></summary>

<br/>

<a href="https://www.linkedin.com/in/vansh-vg/"><img src="badge-linkedin.svg" height="46" alt="LinkedIn — vansh-vg" /></a>
&nbsp;
<a href="mailto:vansh012345gupta@gmail.com"><img src="badge-email.svg" height="46" alt="Email — vansh012345gupta@gmail.com" /></a>
&nbsp;
<a href="https://codeforces.com/profile/YOUR_CF_HANDLE"><img src="badge-codeforces.svg" height="46" alt="Codeforces — Expert" /></a>

</details>

<details>
<summary><code>vansh@dsys ~ % notes</code></summary>

<br/>

<details>
<summary><code>cat aeron-to-kafka.md</code></summary>

<br/>

**Context** — at Futures First, trading servers replicated state over Aeron. Blazing transport, but replay and fault-recovery were bolted on the side.

**Change** — I moved the replication path onto an event-driven log:
- **Kafka** as the durable, strictly-ordered backbone — replayable, with producers and consumers fully decoupled.
- **RocksDB** as the embedded local state store — fast point lookups and a fast cold-start recovery.

**Trade-off** — gave up a sliver of raw latency for durability and clean recovery semantics. The right call on a backup/replication path, where *never silently lose a message* beats *shave another microsecond*.

```mermaid
%%{init: {"theme":"base","themeVariables":{"primaryColor":"#0E1627","primaryTextColor":"#DCE6F5","primaryBorderColor":"#34E7E7","lineColor":"#5B8DEF","clusterBkg":"#0B1120","clusterBorder":"#26344F","fontFamily":"monospace"}}}%%
flowchart LR
  subgraph before["before · Aeron transport"]
    a1["trading server"] <-->|"UDP / multicast"| a2["trading server"]
    a2 -. replication bolted on .-> a3[("backup")]
  end
  subgraph after["after · event-driven log"]
    p["trading servers"] -->|"append"| k[("Kafka · ordered log")]
    k --> r1["replica + RocksDB"]
    k --> r2["replica + RocksDB"]
  end
  before ==>|"migration"| after
```

</details>

<details>
<summary><code>cat colima-testcontainers.md</code></summary>

<br/>

**Symptom** — on Apple Silicon with Colima as the Docker runtime, Testcontainers can't reach the daemon and Ryuk (the cleanup sidecar) refuses to start.

**The fix everyone copy-pastes** — `TESTCONTAINERS_RYUK_DISABLED=true`. It silences the error and leaves orphaned containers piling up. Wrong fix.

**Actual cause** — Testcontainers and Ryuk look for the socket at the default path; Colima publishes it somewhere else.

```mermaid
%%{init: {"theme":"base","themeVariables":{"primaryColor":"#0E1627","primaryTextColor":"#DCE6F5","primaryBorderColor":"#34E7E7","lineColor":"#5B8DEF","clusterBkg":"#0B1120","clusterBorder":"#26344F","fontFamily":"monospace"}}}%%
flowchart LR
  subgraph problem["the mismatch"]
    t1["Testcontainers"] -. looks at default .-> s1["/var/run/docker.sock"]
    s1 --> x1["nothing here — Ryuk fails"]
    c1["Colima dockerd"] --> s2["~/.colima/default/docker.sock"]
  end
  subgraph fix["the fix"]
    t2["Testcontainers"] -->|"DOCKER_HOST"| s3["~/.colima/default/docker.sock"]
    s3 --> c2["Colima dockerd — connected"]
    t2 -->|"SOCKET_OVERRIDE"| ry["Ryuk mounts /var/run/docker.sock"]
  end
  problem ==>|"redirect the socket"| fix
```

**Real fix — keep Ryuk on, just point it at the right socket:**

```bash
export DOCKER_HOST="unix://${HOME}/.colima/default/docker.sock"
export TESTCONTAINERS_DOCKER_SOCKET_OVERRIDE="/var/run/docker.sock"
```

</details>

<details>
<summary><code>cat system-design.md</code></summary>

<br/>

Event-driven replication — the shape I keep reaching for:

```mermaid
%%{init: {"theme":"base","themeVariables":{"primaryColor":"#0E1627","primaryTextColor":"#DCE6F5","primaryBorderColor":"#34E7E7","lineColor":"#5B8DEF","clusterBkg":"#0B1120","fontFamily":"monospace"}}}%%
flowchart LR
  P[producers] -->|append| K[(Kafka · ordered log)]
  K --> C1[consumer / replica]
  K --> C2[consumer / replica]
  C1 --> R1[(RocksDB · local state)]
  C2 --> R2[(RocksDB · local state)]
  K -. replay from offset on recovery .-> C1
```

Producers append to a strictly-ordered Kafka log; each replica builds local RocksDB state and recovers by replaying from its last offset. Failure handling becomes *"replay from offset"* instead of *"hope the in-flight message survived."*

</details>

</details>
