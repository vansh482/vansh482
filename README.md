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



<div align="center">
  <img src="banner.svg" width="850" alt="Vansh Gupta — backend & distributed-systems engineer @ Smartsheet" />
</div>

<br/>

<div align="center">

<details>
<summary><code>vansh@dsys ~ % ./terminal-session</code> &nbsp; <sub>click to toggle</sub></summary>

<br/>

<img src="terminal.svg" width="850" alt="vansh@distributed-systems terminal — whoami, uptime, tech stack (Java, C++, Python, Spring Boot, Kafka, AWS, Docker, Kubernetes, Terraform, MySQL, Datadog), flex (Codeforces Expert 1600+, JEE Advanced AIR 9684), git log (Dynamics 365 + ServiceNow connectors on Nango, Kafka+RocksDB replacing Aeron), and vansh --help (linkedin.com/in/vansh-vg, vansh012345gupta@gmail.com)" />

</details>

</div>

<br/>

<div align="center">
  <code>vansh@dsys ~ % ping me</code>
  <br/><br/>
  <a href="https://www.linkedin.com/in/vansh-vg/"><img src="badge-linkedin.svg" height="46" alt="LinkedIn — vansh-vg" /></a>
  &nbsp;&nbsp;
  <a href="mailto:vansh012345gupta@gmail.com"><img src="badge-email.svg" height="46" alt="Email — vansh012345gupta@gmail.com" /></a>
  &nbsp;&nbsp;
  <a href="https://codeforces.com/profile/YOUR_CF_HANDLE"><img src="badge-codeforces.svg" height="46" alt="Codeforces — Expert" /></a>
</div>

<br/>
<br/>

<div align="center"><code>vansh@dsys ~ % ls ./field-notes</code></div>

<br/>

<details>
<summary><code>cat ~/field-notes/aeron-to-kafka.md</code></summary>

<br/>

**Context** — at Futures First, trading servers replicated state over Aeron. Blazing transport, but replay and fault-recovery were bolted on the side.

**Change** — I moved the replication path onto an event-driven log:
- **Kafka** as the durable, strictly-ordered backbone — replayable, with producers and consumers fully decoupled.
- **RocksDB** as the embedded local state store — fast point lookups and a fast cold-start recovery.

**Trade-off** — gave up a sliver of raw latency for durability and clean recovery semantics. The right call on a backup/replication path, where *never silently lose a message* beats *shave another microsecond*.

</details>

<details>
<summary><code>cat ~/field-notes/colima-testcontainers.md</code></summary>

<br/>

**Symptom** — on Apple Silicon with Colima as the Docker runtime, Testcontainers can't reach the daemon and Ryuk (the cleanup sidecar) refuses to start.

**The fix everyone copy-pastes** — `TESTCONTAINERS_RYUK_DISABLED=true`. It silences the error and leaves orphaned containers piling up. Wrong fix.

**Actual cause** — Testcontainers and Ryuk look for the socket at the default path; Colima publishes it somewhere else.

**Real fix — keep Ryuk on, just point it at the right socket:**

```bash
export DOCKER_HOST="unix://${HOME}/.colima/default/docker.sock"
export TESTCONTAINERS_DOCKER_SOCKET_OVERRIDE="/var/run/docker.sock"
```

</details>

<details>
<summary><code>cat ~/field-notes/system-design.md</code></summary>

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
