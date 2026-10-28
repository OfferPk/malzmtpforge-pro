  
ZMTP  
MALZMTPForge Pro v1.0.0  
CCSDS 524.4-B-1 & ISO 22647 MO MAL Binding to ZMTP Transport Workstation  
CCSDS 524.4-B-1 ZeroMQ v3.0 Brokerless Multi-Peer

UTC: \--:--:--

📦 Multi-Frame Encoder 🔍 Multi-Frame Dissector 📡 Pub/Sub Topic Stream 🔌 ZMQ Socket Topology 🛡️ 10-Point Normative Auditor

🔀  
3 Frames  
ZMTP Message Architecture

⚡  
\>120k msg/s  
Brokerless Throughput

📏  
162 Octets  
Multi-Frame Dimension

⚙️ ZMTP & MAL Header Configuration  
CCSDS 524.4 §3

---

Frame 1: Routing Identity Envelope / Topic

URI From

URI To

Service Area

Service ID

Operation ID

Interaction Pattern REQUEST (Pattern 3\) SEND (Pattern 1\) SUBMIT (Pattern 2\) INVOKE (Pattern 4\) PROGRESS (Pattern 5\) PUBSUB (Pattern 6\)

Interaction Stage

Frame 3: MAL Body Payload

Encode Multi-Frame Message

📄 On-The-Wire Multi-Frame Hex Stream  
162 Octets

---

■ Frame 1: Identity Envelope (MORE=1) ■ Frame 2: MAL Header (MORE=1) ■ Frame 3: Body (MORE=0)

> **ZMTP Multi-Frame Structure (§3.2):**

> * Frame 1 (Flags: 0x01 MORE): Preserves ROUTER socket return path or PUB/SUB topic  
> * Frame 2 (Flags: 0x01 MORE): Standard 20-byte fixed header \+ 5 length-prefixed strings  
> * Frame 3 (Flags: 0x00 FINAL): Message payload passed zero-copy to application layer

🔍 ZMTP Multi-Frame Stream Dissector  
Load Sample Multi-Frame

---

Hexadecimal ZMTP Byte Stream 01 10 4D 4F 43 5F 43 4C 49 45 4E 54 5F 41 4C 50 48 41 01 81 01 00 02 00 01 00 01 01 03 01 00 00 00 00 68 F5 2C 1A 21 0A 00 2D 6D 61 6C 7A 6D 74 70 3A 2F 2F 6D 6F 63 2D 64 61 72 6D 73 74 61 64 74 2E 65 73 61 2E 69 6E 74 3A 35 35 35 35 2F 63 6F 6E 73 75 6D 65 72 00 2D 6D 61 6C 7A 6D 74 70 3A 2F 2F 73 74 61 74 69 6F 6E 2D 63 65 62 72 65 72 6F 73 2E 65 73 61 2E 69 6E 74 3A 35 35 35 35 2F 70 72 6F 76 69 64 65 72 00 16 4D 69 73 73 69 6F 6E 2E 4C 75 6E 61 72 2E 47 61 74 65 77 61 79 00 0B 4F 70 65 72 61 74 69 6F 6E 61 6C 00 07 4C 49 56 45 5F 30 31 00 23 00 21 53 54 41 52 54 5F 52 41 44 41 52 5F 43 41 4C 49 42 52 41 54 49 4F 4E 5F 53 45 51 55 45 4E 43 45

Dissect ZMTP Frames

#### 

Frame 1: Identity Envelope (ZMTP Delimiter)

| Routing Identity | MOC\_CLIENT\_ALPHA |
| :---- | :---- |
| ZMTP Frame Flags | 0x01 (MORE=1) |
| Envelope Type | ROUTER Return Peer Address |

#### **Frame 2: MAL Transport Header**

| Service Area & Service | Area 2, Service 1 |
| :---- | :---- |
| Operation & Version | Op 1, Version 1 |
| Pattern & Stage | REQUEST (3) \| Stage 1 |
| Transaction ID | 1760889882 |

#### 

Routing & Endpoints

| URI From | malzmtp://moc-darmstadt.esa.int:5555/consumer |
| :---- | :---- |
| URI To | malzmtp://station-cebreros.esa.int:5555/provider |
| Domain | Mission.Lunar.Gateway |
| Network Zone | Operational |

#### **Frame 3: MAL Body Payload (FINAL)**

**"START\_RADAR\_CALIBRATION\_SEQUENCE"**

📡 ZeroMQ PUB/SUB Telemetry Broadcaster (Brokerless)  
Broadcast Telemetry Frame

---

Demonstrates zero-broker PUB/SUB distribution. Subscribers filter messages on Frame 1 topic prefixes directly in userspace memory with zero central messaging server overhead.

Click 'Broadcast Telemetry Frame' to simulate instantaneous multi-subscriber fan-out.

🔌 CCSDS 524.4 ZMTP Socket Topology Mapping  
---

| Socket Pattern | Underlying ZMQ Type | MAL Interaction Patterns | Architectural Purpose   |
| :---- | :---- | :---- | :---- |
| ROUTER / DEALER | ZMQ\_ROUTER \<=\> ZMQ\_DEALER | REQUEST, SUBMIT, INVOKE, PROGRESS | Asynchronous multi-client bidirectional request/reply with explicit envelope routing. |
| PUB / SUB | ZMQ\_PUB \<=\> ZMQ\_SUB | PUBLISH / SUBSCRIBE, NOTIFY | High-rate telemetry stream fan-out, parameter updates, and broadcast alerts. |
| PAIR / PAIR | ZMQ\_PAIR \<=\> ZMQ\_PAIR | SEND | Direct inter-thread communication or dedicated point-to-point command pipes. |

🛡️ CCSDS 524.4-B-1 & ISO 22647 10-Point Normative Auditor  
10/10 PASS (100%)

---

| Rule ID | Requirement Title | Normative Focus | Status   |
| :---- | :---- | :---- | :---- |
| MALZMTP-001 | ZMTP Multi-Frame Framing & Envelope Segregation | 3-frame ZMTP structure verified: Frame 1 (ID), Frame 2 (Hdr), Frame 3 (Body) | PASS |
| MALZMTP-002 | Router/Dealer Point-to-Point Socket Binding | ROUTER/DEALER envelope routing preserved with client identity frame intact | PASS |
| MALZMTP-003 | Pub/Sub Topic Distribution & Prefix Filtering | ZeroMQ PUB/SUB topic prefix mapped in Frame 1 for subscriber-side filtering | PASS |
| MALZMTP-004 | MO Service Area, Service, Op & Version ID | Service Area (2), Service (1), Op (5), and Version (1) packed in Frame 2 | PASS |
| MALZMTP-005 | Multi-Stage MAL Interaction Pattern Progression | REQUEST/RESPONSE stage progression (Stage 1 \-\> Stage 2\) verified | PASS |
| MALZMTP-006 | 64-Bit Transaction ID Correlation Across Hops | 64-bit Big-Endian unsigned integer preserved without bit inversion | PASS |
| MALZMTP-007 | Dual-URI Addressing (malzmtp:// Scheme Format) | malzmtp:// URI scheme format verified with length prefix serialization | PASS |
| MALZMTP-008 | Quality of Service (QoS) & Priority Mapping | QoS level (TIMELY=4) and message priority (90) packed in header bitfield | PASS |
| MALZMTP-009 | Error Signaling & Extended Exception Codes | Error flag bit asserted and 16-bit error code unpacked from Frame 2 header | PASS |
| MALZMTP-010 | Zero-Copy Body Encapsulation & Boundary Safety | Payload body isolated in Frame 3 with clean ZMTP final-frame delimiter | PASS |

