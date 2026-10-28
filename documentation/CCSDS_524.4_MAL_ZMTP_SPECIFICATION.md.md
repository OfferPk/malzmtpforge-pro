# CCSDS 524.4-B-1 & ISO 22647 MO MAL Binding to ZMTP Transport Protocol Specification

## Executive Summary

This document provides the technical architecture and wire protocol specification for **MALZMTPForge Pro**, an industrial workstation implementation of the **Consultative Committee for Space Data Systems (CCSDS) 524.4-B-1** Recommended Standard (*Mission Operations—Message Abstraction Layer Binding to ZMTP Transport*, Blue Book, Issue 1, October 2019\) and **ISO 22647**.

The CCSDS Message Abstraction Layer (MAL, CCSDS 521.0-B-2) provides high-level service interaction patterns across space agency ground infrastructures. CCSDS 524.4-B-1 specifies how MAL interactions map onto the **ZeroMQ Message Transport Protocol (ZMTP v3.0)**, delivering an asynchronous, brokerless, multi-peer architecture for high-throughput telemetry fan-out and distributed mission control networks.

---

## 1\. ZMTP Multi-Frame PDU Architecture

\+-------------------------------------------------------------------------+

\|                  ZMTP Frame 1: Routing Identity / Topic                 \|

\|  \+--------------------------------------------------------------------+  \|

\|  \| ZMTP Flags (0x01 MORE) • Length • Routing ID / Topic String Prefix \|  \|

\|  \| (e.g. "MOC\_CLIENT\_ALPHA" or "malzmtp://gateway/mc/parameter/RPM")  \|  \|

\+-------------------------------------------------------------------------+

                                    \|

\+-----------------------------------+-------------------------------------+

\|                  ZMTP Frame 2: MAL Transport Header                     \|

\|  \+--------------------------------------------------------------------+  \|

\|  \| ZMTP Flags (0x01 MORE) • Length (8b/64b)                           \|  \|

\|  \| Fixed Header (20-22 Octets):                                       \|  \|

\|  \|   \- Protocol Version (8b: 0x01)                                    \|  \|

\|  \|   \- Service Area (16b) • Service (16b) • Operation (16b)           \|  \|

\|  \|   \- Service Version (8b) • Pattern (8b) • Stage (8b)               \|  \|

\|  \|   \- 64-Bit Transaction ID (64b, Big-Endian)                         \|  \|

\|  \|   \- Flags: Error(1b) \| QoS(3b) \| Session(4b)                       \|  \|

\|  \|   \- Priority (8b)                                                  \|  \|

\|  \|   \- \[Optional\] Error Code (16b if Error Flag asserted)             \|  \|

\|  \| Variable Header:                                                   \|  \|

\|  \|   \- URI From (16b len \+ UTF-8) • URI To (16b len \+ UTF-8)          \|  \|

\|  \|   \- Domain (16b len \+ UTF-8)   • Network Zone (16b len \+ UTF-8)    \|  \|

\|  \|   \- Session Name (16b len \+ UTF-8)                                 \|  \|

\+-------------------------------------------------------------------------+

                                    \|

\+-----------------------------------+-------------------------------------+

\|                  ZMTP Frame 3: MAL Body Payload (FINAL)                 \|

\|  \+--------------------------------------------------------------------+  \|

\|  \| ZMTP Flags (0x00 FINAL) • Length • MAL Encoded Binary Payload Body \|  \|

\|  \| (Passed zero-copy directly to the application layer handler)       \|  \|

\+-------------------------------------------------------------------------+

### 1.1 Multi-Frame Message Boundaries

ZMTP encapsulates discrete logical frames within a single message. CCSDS 524.4 mandates strict 3-frame segregation:

- **Frame 1**: Routing Identity (preserving DEALER/ROUTER return peer address) or PUB/SUB Topic Prefix.  
- **Frame 2**: Complete MAL transport header.  
- **Frame 3**: MAL payload body (encoded in MAL Binary).

---

## 2\. ZeroMQ Socket Topology Mapping

| Socket Pattern | Underlying ZMQ Type | Target MAL Interaction Patterns | Operational Semantics |
| :---- | :---: | :---: | :---- |
| **ROUTER / DEALER** | `ZMQ_ROUTER` ⇄ `ZMQ_DEALER` | `REQUEST`, `SUBMIT`, `INVOKE`, `PROGRESS` | Asynchronous point-to-point command uplink and verified response routing without blocking threads. |
| **PUB / SUB** | `ZMQ_PUB` ⇄ `ZMQ_SUB` | `PUBLISH/SUBSCRIBE`, `NOTIFY` | High-rate telemetry streaming with userspace prefix matching, eliminating central broker bottlenecks. |
| **PAIR / PAIR** | `ZMQ_PAIR` ⇄ `ZMQ_PAIR` | `SEND` | Low-latency inter-process telemetry pipes and dedicated spacecraft test set links. |

---

## 3\. High-Throughput Brokerless Scaling

By eliminating central message broker daemons (e.g. RabbitMQ, ActiveMQ), MAL-ZMTP nodes establish direct peer-to-peer TCP mesh connections, supporting over 120,000 telemetry messages per second per ground node with sub-millisecond dispatch latency.