# RNS Meshtastic Interface

A standalone Reticulum interface for transporting binary RNS traffic over
Meshtastic radios.

The interface uses Meshtastic's reserved `RETICULUM_TUNNEL_APP` application
port (port 76). It provides bounded binary fragmentation, sender-isolated
reassembly, CRC validation, configurable channel and modem settings, paced
transmission, and automatic reconnect handling.

This is an independent project and is not part of the official Reticulum or
Meshtastic distributions.

## Requirements

- Python 3
- [Reticulum](https://github.com/markqvist/Reticulum)
- [Meshtastic Python](https://github.com/meshtastic/python) 2.7.11 or newer

Install the runtime dependencies:

```bash
python3 -m pip install rns "meshtastic>=2.7.11"
```

On Windows, use `python` instead of `python3` if that is how Python is
installed.

## Installation

1. Locate Reticulum's storage directory. The default is `~/.reticulum`.
2. Create its `interfaces` directory if it does not already exist.
3. Copy `MeshtasticInterface.py` into that directory.

Linux and macOS:

```bash
mkdir -p ~/.reticulum/interfaces
cp MeshtasticInterface.py ~/.reticulum/interfaces/
```

Windows PowerShell:

```powershell
New-Item -ItemType Directory -Force "$HOME\.reticulum\interfaces"
Copy-Item .\MeshtasticInterface.py "$HOME\.reticulum\interfaces\"
```

If Reticulum is started with a custom storage path, place the file in the
`interfaces` directory below that path instead.

## Configuration

Add an interface section to `~/.reticulum/config`.

### Serial

```ini
[[Meshtastic]]
  type = MeshtasticInterface
  enabled = yes
  port = /dev/ttyUSB0
  channel = 0
  hop_limit = 1
```

On Windows, the serial port can for example be `COM5`. If `port`, `host`, and
`ble` are all omitted, the Meshtastic library attempts serial discovery.

### TCP

```ini
[[Meshtastic]]
  type = MeshtasticInterface
  enabled = yes
  host = meshtastic.local
  channel = 0
  hop_limit = 1
```

### Bluetooth LE

```ini
[[Meshtastic]]
  type = MeshtasticInterface
  enabled = yes
  ble = AA:BB:CC:DD:EE:FF
  channel = 0
  hop_limit = 1
```

Specify at most one of `port`, `host`, or `ble`.

## Complete configuration reference

```ini
[[Meshtastic]]
  type = MeshtasticInterface
  enabled = yes

  # Connection: select at most one. Omitting all enables serial discovery.
  port = /dev/ttyUSB0
  # host = meshtastic.local
  # ble = device_name_or_address

  # Meshtastic channel index, from 0 through 7.
  channel = 0

  # Broadcast is the default. A Meshtastic node ID enables fixed unicast.
  # destination = !12345678

  # Optional radio preset, applied to the connected radio when different.
  # All participating radios must use compatible LoRa settings.
  # modem_preset = LongFast

  # Binary framing and transmission pacing.
  # payload_size = 200
  # send_interval = 1.0

  # Incomplete-transfer lifetime and resource limits.
  # reassembly_timeout = 300
  # max_reassemblies = 64
  # max_reassemblies_per_sender = 8
  # max_packet_size = 65535
  # max_pending_packets = 128

  # Connection and reconnect behaviour.
  # connection_timeout = 30
  # reconnect_interval = 5
  # max_reconnect_interval = 60

  # Reticulum provides reliability, so Meshtastic ACKs stay disabled.
  # want_ack = no

  # 0 means direct radio range only; 1 permits at most one rebroadcast.
  # Other values are rejected to avoid multi-hop fragment flooding.
  # hop_limit = 1

  # Reported interface bitrate in bits per second.
  # bitrate = 118
```

### Operational notes

- A dedicated Meshtastic channel is recommended. The selected channel index
  must be provisioned identically on every participating radio.
- Meshtastic acknowledgements are deliberately disabled because Reticulum
  supplies its own reliability mechanisms. `want_ack = yes` is rejected.
- `hop_limit` must be `0` or `1`.
- The default and maximum Meshtastic application payload is 200 bytes. Larger
  RNS packets are fragmented and reassembled automatically.
- `send_interval` is the delay in seconds between fragments. Set it to `0` to
  disable pacing only when the radio profile and local regulations permit it.
- Setting `modem_preset` writes the selected LoRa preset to the radio if its
  current value differs.

## Wire-format compatibility

This interface uses its own versioned `RNSM` binary framing with a random
transfer ID and whole-packet CRC. It does not interoperate over the air with
the separate
[`landandair/RNS_Over_Meshtastic`](https://github.com/landandair/RNS_Over_Meshtastic)
two-byte fragment format. Every Reticulum endpoint on the same Meshtastic
transport must use this interface.

## License

See [LICENSE](LICENSE). The license notice is also included in
`MeshtasticInterface.py`.
