# Networking Fundamentals

## The OSI 7-Layer Model
When you send an HTTP request, your data undergoes **Encapsulation** (on your device) and **Decapsulation** (on the server).

### Client Side: The Encapsulation Process
1.  **Layer 7 - Application:** Your browser creates an `HTTP GET` request.
2.  **Layer 6 - Presentation:** Data is wrapped in **SSL/TLS Encryption** and formatted.
3.  **Layer 5 - Session:** Manages the connection (Session IDs, Authentication, Timeouts).
4.  **Layer 4 - Transport:** Data is segmented. **TCP** adds Source/Dest ports (e.g., 443).
5.  **Layer 3 - Network:** Adds IP routing (Source IP to Destination IP). Routers operate here.
6.  **Layer 2 - Data Link:** Adds **MAC Addresses** for local delivery. Switches operate here.
7.  **Layer 1 - Physical:** Converts data into electrical/radio signals (Bits) over cables or Wi-Fi.

### Server Side: The Decapsulation Process (Reverse)
The server receives the bits and strips the headers in reverse order (L1 → L7) until the web server (like Nginx) can process the original HTTP request.

---

## TCP vs. UDP: The Transport Layer
At **Layer 4**, protocols determine how reliably your data is delivered.

| Feature | TCP (Transmission Control) | UDP (User Datagram) |
| :--- | :--- | :--- |
| **Reliability** | **Guaranteed.** Lost packets are resent. | **Best-effort.** No retransmission. |
| **Connection** | Connection-oriented (3-way handshake). | Connectionless (Fire and forget). |
| **Speed** | Slower due to overhead/error checking. | **Blazing Fast.** Minimal overhead. |
| **Ordering** | Data arrives in the correct sequence. | Data may arrive out of order. |
| **Use Case** | Web (HTTP), Email, SSH, Databases. | Gaming, Video Calls, DNS, Streaming. |

---

### Port
- **Port**: Virtual endpoint to direct data to the correct application.  
- Multiple services can run on a single device; ports help route data correctly.  
- Examples: HTTP → 80, SSH → 22, MySQL → 3306.

### Common Ports

| Port  | Service |
| ----- | ------- |
| 22    | SSH     |
| 80    | HTTP    |
| 443   | HTTPS   |
| 53    | DNS     |
| 3306  | MySQL   |
| 6379  | Redis   |
| 27017 | MongoDB |

---

## 🌍 IP Addressing: Public vs. Private

| Feature | Private IP Address | Public IP Address |
| :--- | :--- | :--- |
| **Scope** | Used within a local network (LAN). | Used across the global internet (WAN). |
| **Visibility** | Hidden from the internet (behind NAT). | Searchable and reachable globally. |
| **Cost** | Free to use. | Assigned by ISP. |
| **Example** | `192.168.1.15` | `8.8.8.8` |

---

## 🔢 IPv4 Classes & CIDR
IPv4 addresses are 32-bit numbers categorized into classes based on the first octet.

### IP Address Classes
* **Class A (1–126):** For very large networks. Default mask `255.0.0.0`.
* **Class B (128–191):** For medium networks. Default mask `255.255.0.0`.
* **Class C (192–223):** For small local networks. Default mask `255.255.255.0`.
* **Class D (224–239):** Reserved for Multicasting.
* **Class E (240–255):** Experimental/Research.
* *Note: `127.0.0.1` is reserved for loopback (localhost).*

### Subnetting (CIDR)
**Subnetting** is the process of dividing a large network into smaller, manageable networks. To find usable hosts, we use the formula: $2^{(32 - CIDR)} - 2$.


| Binary Value ($2^n$) | $2^7$ | $2^6$ | $2^5$ | $2^4$ | $2^3$ | $2^2$ | $2^1$ | $2^0$ |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Decimal Value** | **128** | **64** | **32** | **16** | **8** | **4** | **2** | **1** |

#### 1. /24 Calculation
* **Logic:** First 24 bits are ON.
* **Binary:** `11111111.11111111.11111111.00000000` → `255.255.255.0`
* **Usable Hosts:** $32 - 24 = 8$ bits. $2^8 = 256$. Subtract 2 (Network & Broadcast) = **254**.

#### 2. /16 Calculation
* **Logic:** First 16 bits are ON.
* **Binary:** `11111111.11111111.00000000.00000000` → `255.255.0.0`
* **Usable Hosts:** $32 - 16 = 16$ bits. $2^{16} = 65,536$. Subtract 2 = **65,534**.

#### 3. /28 Calculation
* **Logic:** First 28 bits are ON.
* **Binary:** `11111111.11111111.11111111.11110000` → `255.255.255.240`
* **Last Octet Math:** $128 + 64 + 32 + 16 = 240$.
* **Usable Hosts:** $32 - 28 = 4$ bits. $2^4 = 16$. Subtract 2 = **14**.

---

## DNS (Domain Name System)
DNS translates human-readable names like `google.com` into IP addresses. It works through a hierarchy:



1.  **Resolver:** Your computer asks the ISP's server for an IP.
2.  **Root Server:** Points the resolver toward the TLD (Top Level Domain) like `.com`.
3.  **TLD Server:** Points toward the Authoritative Name Server.
4.  **Authoritative Server:** Provides the final IP (A Record).

### Common DNS Record Types

| Record Type | Purpose | Example / Use Case |
| :--- | :--- | :--- |
| **A** | Maps a domain to an **IPv4** address. | `google.com` -> `142.250.190.14` |
| **AAAA** | Maps a domain to an **IPv6** address. | Used for modern 128-bit addressing. |
| **CNAME** | **Canonical Name**: Alias for another domain. | `www.app.com` -> `app.com` |
| **MX** | **Mail Exchange**: Routes email to a mail server. | Used to set up Google Workspace or Outlook. |
| **TXT** | Arbitrary text; often used for security. | SPF, DKIM, or site ownership verification. |
| **NS** | **Name Server**: Tells the internet who is authoritative. | `ns-cloud-c1.googledomains.com` |

---

## 🛠️ Essential DevOps Networking Tools

| Command | Purpose |
| :--- | :--- |
| `dig <domain>` | Check DNS records and TTL. |
| `ip addr show` | Find your local IP address and interfaces. |
| `ss -tulpn` | List all services listening on specific ports. |
| `ping <ip>` | Test Layer 3 (Network) connectivity. |
| `nc -zv <ip> <port>` | Check if a specific port is open (Netcat). |