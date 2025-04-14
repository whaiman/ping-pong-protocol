# Ping-Pong Communication Protocol

**PPCP** - Ping-Pong Communication Protocol is a lightweight, secure stream-oriented zero-configuration protocol designed for remote ad-hoc execution and data transfer.

## The Motivation

The idea that sparked this protocol was simple: _how do you control a computer on the other side of the when its IP address constantly changes, and without having to manually enter an encryption key every time?_

PPCP is specifically designed for seamless device communication within a local area network (LAN). It ensures endpoints can effortlessly find each other without static IP routing, while maintaining a mathematically secure, encrypted data channel.

## Core Features

- **Zero-Configuration Discovery:** Clients broadcast a PING to discover servers across local subnets dynamically.
- **Ephemeral Key Exchange:** Perfect Forward Secrecy generated on the fly using ECDH (Elliptic Curve Diffie-Hellman P-256).
- **Mutual Authentication:** HMAC-SHA256 proofs leveraging an out-of-band pre-shared key protect against Man-In-The-Middle attacks.
- **Encrypted Transport:** All command and file payloads are framed and protected via AES-256-GCM.
- **Robust Feature Set:** Supports multiplexed bi-directional heartbeats, remote shell execution, and base64 chunked file transfers.

## Documentation & Architecture

The architecture of PPCP is heavily documented. Please refer to the following documents for protocol details:

1. **[RFC Specification (docs/rfc.md)](docs/rfc.md)** - The complete formal protocol specification detailing message schemas, the state machine, capability negotiations, and exact payload formatting.
2. **[Sequence Diagram (docs/diagram.md)](docs/diagram.md)** - A full visual breakdown of the connection lifecycle phases, including the Key Exchange payload arrays.

## Code Structure

```text
ping-pong-protocol/
├── ppcp/
│   ├── core/           - Dataclasses, Message schemas, State Machine Enums
│   ├── crypto/         - PBKDF2 cryptography, AES-GCM engine, HMAC derivation
│   ├── discovery/      - Network scanners and Ping/Pong payloads
│   ├── handshake/      - High-level Orchestrator bridging Key Exchange -> Auth
│   ├── key_exchange/   - Ephemeral ECDH pair generator, HKDF derivations
│   ├── transport/      - TCP/UDP socket configurations and Frame packaging
│   ├── control/        - Execution routers, File-transfer multiplexers
│   └── utils/          - Serialization, Timestamping
├── docs/
│   ├── rfc.md          - Full Protocol Specification
│   └── diagram.md      - Mermaid visual architecture
└── README.md
```

## Quick Start (Pseudo-code)

**Server (`server.py`)**

```python
from ppcp import create_tcp_server, server_accept

pre_shared_key = b"super-secret-key"
listener = create_tcp_server("0.0.0.0", 9000)

while True:
    sock, addr = listener.accept()
    # PING -> ECDH -> AUTH. Returns a completely encrypted session!
    session = server_accept(sock, pre_shared_key)
    print(f"Connected to {session.client_id}!")
```

**Client (`client.py`)**

```python
from ppcp import find_server, client_connect

pre_shared_key = b"super-secret-key"
# 1. Scans the LAN to find the server dynamically
server_socket = find_server(port=9000)

# 2. Performs full cryptographic handshake
session = client_connect(server_socket, pre_shared_key)
# Ready to send encrypted commands and files...
```

## License

This project is licensed under the GNU Affero General Public License v3.0 (AGPL-3.0). See the [LICENSE](LICENSE) file for details.
