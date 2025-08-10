# Wireshark Capture & Analysis

This repository contains a network capture and screenshots for analysis. Below are simple steps to install Wireshark on Windows, capture traffic, filter HTTP/HTTPS, and a concise analysis of the provided capture.

## Checklist

- [x] Install Wireshark (steps below)
- [x] Capture local network traffic (how-to below)
- [x] Filter HTTP traffic and analyze packets (results below; no plaintext HTTP observed, HTTPS/QUIC analyzed instead)
- [ ] Deliver wireshark_capture.pcap (see conversion steps from the provided .pcapng)
- [x] README.md explaining the captured packets (this file)

---

## Install Wireshark (Windows)

1. Download Wireshark from [wireshark.org/download](https://www.wireshark.org/download.html)
2. During setup, keep defaults and ensure these options are selected:
   - Install Npcap (packet capture driver)
   - Install TShark (command‑line tools)
3. Finish installation and reboot if prompted.

Optional (command line via winget):

```powershell
winget install WiresharkFoundation.Wireshark --accept-source-agreements --accept-package-agreements
```

## Capture traffic

- Open Wireshark, pick your active interface (Wi‑Fi/Ethernet with packets increasing).

- Click Start capturing.

- Generate some traffic (browse a site, e.g., <http://neverssl.com> for HTTP or any site for HTTPS).

- Stop capture and save: File → Save As…

Tips:

- To capture only web traffic while capturing: set capture filter to `tcp port 80 or tcp port 443`.
- For post‑capture filtering, use display filters (below).

## Display filters you can use

- Plain HTTP (port 80): `http or tcp.port == 80`
- HTTP requests specifically: `http.request`
- TLS/HTTPS SNI for a given host: `tls.handshake.extensions_server_name contains "example.com"`
- QUIC (HTTP/3): `quic or udp.port == 443`
- DNS: `dns`
- BitTorrent DHT: `bittorrent.dht or udp.port == 6881`
- ICMP/ICMPv6 (ping, ND): `icmp or icmpv6`
- ARP: `arp`

## Convert .pcapng to .pcap (deliverable)

The provided capture is `wireshark_capture.pcapng`. If your assignment requires legacy `.pcap`, convert it using either GUI or CLI.

- GUI: File → Save As… → Save as type: "pcap - Wireshark/tcpdump/..." → Save as `wireshark_capture.pcap`.
- CLI (requires TShark installed during setup):

```powershell
# Convert pcapng → pcap
"$Env:ProgramFiles\Wireshark\tshark.exe" -r "wireshark_capture.pcapng" -F pcap -w "wireshark_capture.pcap"
```

Note: If `tshark` isn’t found, install Wireshark with TShark or use the GUI method above.

---

## What’s inside this capture (high‑level)

Files provided:

- Capture: `wireshark_capture.pcapng`
- Text export: `wireshark_capture.txt` (used for the summary below)
- Screenshots: multiple `Screenshot (...).png`

Observed protocols and activities (based on `wireshark_capture.txt`):

- ARP from the gateway (192.168.29.1) resolving local hosts.
- ICMP echo requests/replies (ping) between 192.168.29.59 and 203.74.222.4 (e.g., frames ~74–77, 150–161, 197–205).
- Extensive BitTorrent DHT (bittorrent.dht) queries/responses over UDP/6881 and IPv6 to many nodes from host 192.168.29.59 (e.g., frames ~9–73 and throughout). Each `Get_peers` includes an `Info_hash`.
- DNS lookups to the local DNS (192.168.29.1) and via IPv6 to the router (e.g., frames 78–83, 110–113, 129–133, 165–178, 281–287, 289–315, 356–361, many later).
- TLS/HTTPS connections to Microsoft/Azure/Akamai properties using SNI values like:
  - `login.live.com` (frames 360–376+)
  - `browser.events.data.microsoft.com` / `browser.events.data.msn.com` (frames 168–176, 369–371+, 555–591)
  - `www.bing.com`, `r.bing.com`, `assets.msn.com`, `c.msn.com`, `r.msftstatic.com` (frames 287+, 316–335+, 382+, 391–401+, 453–501+, etc.)
- QUIC (HTTP/3) traffic to Akamai edges for Bing/MSN (e.g., frames 287–313+ using QUIC Initial/Handshake/Protected Payload).
- IGMP v3 membership reports and general queries (frames 50, 144, 276).
- IPv6 Neighbor Solicitation/Advertisement (frames 40–41, 228–229) and Router Advertisement (frame 235).

No plaintext HTTP was seen in the provided sample (modern browsers redirect to HTTPS or use QUIC). HTTP/2 runs over TLS and is not labeled `http` without decryption.

### Notable examples with display filters

- Ping conversation: filter `icmp && ip.addr == 203.74.222.4` to see request/reply RTTs.
- DHT chatter: filter `bittorrent.dht or udp.port == 6881` to isolate P2P queries like `Get_peers` and responses.
- Microsoft sign‑in: `tls.handshake.extensions_server_name contains "login.live.com"` shows the TLS handshake with SNI.
- Bing/MSN assets: try `tls.handshake.extensions_server_name matches "(bing|msn)\.com"`.
- QUIC/HTTP3: `quic && ipv6` highlights QUIC Initial/Handshake packets to Akamai (e.g., `2600:140f:...`).
- DNS for Bing/MSN: `dns && (dns.qry.name contains "bing.com" or dns.qry.name contains "msn.com")`.

### If you specifically need HTTP (port 80)

The capture contains no classic HTTP over port 80. To produce HTTP traffic for learning/testing:

- Visit <http://neverssl.com/> or run a local server on port 80.
- Then filter with `http` or `tcp.port == 80` and examine `GET/POST` requests and responses.

### Privacy note

TLS and QUIC encrypt payloads. You can still identify destinations (via SNI, IPs, DNS) and timings, but not content. To decrypt TLS in Wireshark, you’d need server keys or client secrets (e.g., SSLKEYLOGFILE) and proper setup.

---

## Screenshots

Below are inline references to the provided images for visual context (open them in GitHub):

- ![Screenshot 175](<Screenshot (175).png>)
- ![Screenshot 176](<Screenshot (176).png>)
- ![Screenshot 177](<Screenshot (177).png>)
- ![Screenshot 178](<Screenshot (178).png>)
- ![Screenshot 179](<Screenshot (179).png>)
- ![Screenshot 180](<Screenshot (180).png>)
- ![Screenshot 181](<Screenshot (181).png>)

---

## What this traffic suggests

- Normal LAN noise: ARP, IGMP, IPv6 ND.
- Routine system traffic to Microsoft services (sign‑in, telemetry endpoints, Bing/MSN assets) over TLS/QUIC.
- Active BitTorrent DHT queries/responses from 192.168.29.59. If unexpected, check for P2P software or apps using WebTorrent.

---

## Appendix: quick how‑tos

- Follow a TLS/TCP stream: right‑click a packet → Follow → TCP Stream.
- Resolve names: View → Name Resolution (or keep off to focus on raw IPs).
- Export a subset: set a display filter → File → Export Specified Packets…

---

## Deliverables

- `wireshark_capture.pcapng` (provided)
- `README.md` (this file)
- To produce `wireshark_capture.pcap`, use the conversion steps above and commit the resulting file next to this README.
