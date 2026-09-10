# KGP Transport Protocol (KTP)

A reliable transport protocol implemented from scratch over UDP in C, providing reliable and ordered data delivery despite packet loss and network unreliability.

KTP implements core transport-layer mechanisms including Go-Back-N ARQ, sliding-window flow control, sequence numbering, retransmission, connection termination, concurrent socket handling, ring-buffer-based packet buffering, and shared-memory IPC.

---

## Overview

UDP provides fast, connectionless communication but does not guarantee:

- Reliable delivery
- Packet ordering
- Duplicate detection
- Retransmission of lost packets
- Flow control
- Connection management

KTP addresses these limitations by building a reliable transport layer on top of UDP.

The implementation was designed to explore how transport protocols such as TCP provide reliability while giving direct control over packet transmission, buffering, acknowledgements, retransmissions, and concurrency.

---

## Key Features

- Reliable data transfer over UDP
- Ordered delivery using sequence numbers
- Go-Back-N ARQ for loss recovery
- Sliding-window flow control
- Packet retransmission
- Ring-buffer-based packet buffering
- Concurrent socket processing using POSIX threads
- Shared-memory IPC for communication between components
- FIN/FAK-based connection termination
- Fixed-size 512-byte packet handling
- Robust operation under high packet-loss conditions

---

## Protocol Design

### 1. Sequence Numbering

Each data packet is assigned a sequence number.

The receiver uses sequence numbers to:

- Detect missing packets
- Identify duplicate packets
- Maintain packet ordering
- Deliver data in the correct order

---

### 2. Sliding Window

KTP uses a sliding-window mechanism to allow multiple packets to be in flight simultaneously.

Instead of waiting for an acknowledgement after every packet, the sender can transmit multiple packets within the current window.

This improves throughput while maintaining reliable delivery.

---

### 3. Go-Back-N ARQ

KTP uses **Go-Back-N Automatic Repeat reQuest (ARQ)** for retransmission.

When packet loss is detected, the sender retransmits the affected packet and subsequent packets in the current transmission window.

This allows the protocol to recover from packet loss while maintaining ordered delivery.

---

### 4. Packet Buffering

A ring buffer is used to efficiently manage packets waiting for transmission, acknowledgement, or processing.

The buffer-based design provides predictable memory usage and supports concurrent packet processing.

---

### 5. Connection Termination

KTP implements a FIN/FAK handshake for controlled connection termination.

This allows both sides to explicitly coordinate the end of a transfer instead of simply terminating the underlying UDP sockets.

---

## Architecture

The protocol consists of multiple components responsible for different stages of communication:

```text
                    ┌───────────────────┐
                    │   Application     │
                    └─────────┬─────────┘
                              │
                              ▼
                    ┌───────────────────┐
                    │   KTP Interface   │
                    └─────────┬─────────┘
                              │
                              ▼
                    ┌───────────────────┐
                    │   Send / Receive  │
                    │      Buffers      │
                    └─────────┬─────────┘
                              │
                              ▼
                    ┌───────────────────┐
                    │ Reliability Layer │
                    │                   │
                    │ Sequence Numbers  │
                    │ Sliding Window    │
                    │ Go-Back-N ARQ     │
                    │ Retransmission    │
                    └─────────┬─────────┘
                              │
                              ▼
                    ┌───────────────────┐
                    │       UDP         │
                    └─────────┬─────────┘
                              │
                    Unreliable Network
                              │
                              ▼
                    ┌───────────────────┐
                    │       UDP         │
                    └───────────────────┘
