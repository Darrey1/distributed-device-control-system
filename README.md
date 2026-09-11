# Distributed Device Control System

A distributed device control system built with **Go, gRPC, Protocol Buffers, MQTT, and PostgreSQL**.

The system connects backend services to physical devices through edge gateways, handling device registration, telemetry, command delivery, acknowledgements, reconnection, and state synchronization.

> **The goal is simple: send commands to devices reliably, even when the network is not.**

## Architecture

```text
                    ┌─────────────────────┐
                    │     Client / API    │
                    └──────────┬──────────┘
                               │
                              gRPC
                               │
                    ┌──────────▼──────────┐
                    │   Control Service   │
                    │                     │
                    │ Commands            │
                    │ Device State        │
                    │ Acknowledgements    │
                    └──────────┬──────────┘
                               │
                         Message / State
                               │
                    ┌──────────▼──────────┐
                    │     Edge Gateway    │
                    │                     │
                    │ Device Connection   │
                    │ Reconnection        │
                    │ Local State         │
                    └──────────┬──────────┘
                               │
                              MQTT
                               │
                 ┌─────────────┴─────────────┐
                 │                           │
          ┌──────▼──────┐             ┌──────▼──────┐
          │   Device    │             │   Device    │
          │   ESP32     │             │   Simulator │
          └─────────────┘             └─────────────┘
```

## What it demonstrates

* Device registration and authentication
* Device state management
* Telemetry ingestion
* Remote command dispatch
* Command acknowledgements
* Device reconnection
* State synchronization
* Communication between distributed services
* Edge gateway to device communication
* Persistent device and command state
* Failure handling across disconnected components

## Why this project exists

Controlling a device is easy when everything is online.

The interesting problems start when:

* a device disconnects while a command is being processed
* the gateway loses its connection to the backend
* the same command is delivered more than once
* a device reconnects with stale state
* the backend restarts while commands are in flight
* telemetry arrives out of order
* the network disappears temporarily

This project explores the infrastructure required to handle those situations instead of assuming the network is always available.

## Core flow

### 1. Device registration

A device connects through an edge gateway and registers its identity and capabilities with the backend.

```text
Device → Gateway → Control Service → Database
```

The backend maintains the device's current state and connection information.

### 2. Command dispatch

A client sends a command to a registered device.

```text
Client
  │
  ▼
Control Service
  │
  ▼
Edge Gateway
  │
  ▼
Device
  │
  ▼
Acknowledgement
```

Commands are tracked so the system knows whether they were delivered, acknowledged, or still pending.

### 3. Reconnection

When a device or gateway disconnects, the system does not assume that the last known state is still correct.

After reconnection, the gateway can synchronize its state with the backend and continue communicating with the device.

```text
Connected
    │
    ▼
Disconnected
    │
    │   network failure
    ▼
Reconnecting
    │
    ▼
State Sync
    │
    ▼
Connected
```

## Technology

| Component             | Technology       |
| --------------------- | ---------------- |
| Backend services      | Go               |
| Service communication | gRPC             |
| API contracts         | Protocol Buffers |
| Device communication  | MQTT             |
| Persistence           | PostgreSQL       |
| Edge communication    | MQTT             |
| Containerization      | Docker           |

## Project structure

```text
.
├── cmd/
│   ├── api/
│   ├── gateway/
│   └── ...
├── internal/
│   ├── device/
│   ├── command/
│   ├── gateway/
│   └── ...
├── proto/
├── migrations/
├── deployments/
├── docker-compose.yml
└── README.md
```

## Getting started

### Requirements

* Go
* Docker
* Docker Compose
* MQTT broker

### Clone

```bash
git clone https://github.com/YOUR_USERNAME/distributed-device-control.git
cd distributed-device-control
```

### Start dependencies

```bash
docker compose up -d
```

### Run the services

```bash
go run ./cmd/...
```

> The exact commands may change as the project evolves. See the individual service directories for service-specific instructions.

## Example command

A client can request an action on a device through the control service:

```text
Client
  │
  │  "turn_on"
  ▼
Control Service
  │
  │  command
  ▼
Gateway
  │
  │  MQTT
  ▼
Device
  │
  │  acknowledgement
  ▼
Gateway
  │
  ▼
Control Service
```

The important part is not simply sending the message. The system tracks the command throughout its lifecycle.

## Reliability considerations

This project focuses on the problems that appear when distributed systems interact with unreliable networks and physical devices.

### Connection failures

Devices can disappear without warning.

The gateway handles reconnection rather than treating a temporary connection failure as permanent device failure.

### Command delivery

Commands have an explicit lifecycle rather than being treated as fire-and-forget messages.

```text
PENDING
   │
   ▼
SENT
   │
   ├──────────────► FAILED
   │
   ▼
ACKNOWLEDGED
```

### State synchronization

The backend and gateway may temporarily disagree about the state of a device.

Reconnection provides an opportunity to synchronize state instead of blindly assuming the previous state is still valid.

### Distributed failures

The system is designed around the assumption that individual components can fail independently.

A backend service can restart.

A gateway can disconnect.

A device can disappear.

The network can fail.

The system should recover without requiring every component to restart at the same time.

## Testing

Run the test suite with:

```bash
go test ./...
```

For race detection:

```bash
go test -race ./...
```

## What I am exploring

This project is part of my work around **distributed systems, edge infrastructure, and device communication**.

The next areas I am exploring include:

* Offline command handling
* Durable command queues
* Stronger delivery guarantees
* Device capability models
* Local automation at the edge
* Additional device protocols
* Observability and distributed tracing
* Secure device identity and authentication
* Scaling gateway connections across large device fleets

## Status

**Active development**

