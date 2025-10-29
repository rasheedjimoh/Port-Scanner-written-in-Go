# 🔍 Custom Multithreaded Port Scanner (Go)

## Overview
A compact, **multithreaded TCP port scanner** implemented in Go.  
Designed for rapid discovery of open TCP ports across single IPs or IP ranges. This is a practical, no-frills scanner intended for lab use, red-team exercises, and learning about concurrent network probing.

---

## Key Features
- ⚡ **Concurrency-first design** using goroutines and `sync.WaitGroup` for parallel port checks.  
- 🔁 **Range & single-target support** — accept a single IP, multiple space-separated IPs, or an IP range (e.g., `192.168.1.1-192.168.1.100`).  
- ⏱️ **Timeout-controlled connections** (`DialTimeout`) to avoid long hangs on filtered ports.  
- 🧹 **Lightweight and dependency-free** — only uses Go standard library (`net`, `sync`, `time`, `bufio`, etc.).  
- 📈 **Simple runtime reporting**: prints each open port and total elapsed scan time.

---

## Why this script is useful
- Great for quickly validating network exposure during lab exercises or small internal scans.  
- A clear, educational example of how to combine Go’s networking primitives with concurrency patterns for high-throughput tasks.  
- Easy to read and extend — ideal as a starter for custom scanning tools (e.g., targeted scans, service fingerprinting, or integration in pentest toolchains).

---

## Usage (how it works)
1. Run the compiled binary.  
2. When prompted, enter targets as:
   - A single IP: `10.0.0.5`  
   - Multiple IPs: `10.0.0.5 10.0.0.6`  
   - A range: `192.168.1.1-192.168.1.100`  
3. The scanner launches concurrent goroutines to test all TCP ports (1–65535) for each target and prints open ports as they’re discovered. At the end, the script prints total run time.

---

## Example output
```

Starting port scan on targets: [192.168.1.10]
192.168.1.10 Port 22 is open.
192.168.1.10 Port 80 is open.
Port scan completed in 4.23s

```

---

## Performance & limits
- Scans every TCP port (1–65535) for each target — extremely parallel by default. This can saturate CPU, network, and local file descriptors on large ranges.  
- Timeout is currently 10s per port — a balance between speed and reliability, but tunable.  
- For large-scale scanning, consider rate-limiting, a worker-pool pattern, or scanning a prioritized subset of ports first (common ports, top-n lists).

---

## Safety & ethics (must-read)
- **Only scan systems you own or have explicit permission to test.** Unauthorized scanning can be disruptive and illegal.  
- Use in controlled lab environments or under a signed engagement scope when testing customer networks.

---

## Extension ideas (next-level improvements)
- Add **worker pool** to limit concurrent connections and manage resource usage.  
- Allow **custom port lists** or top-ports mode (e.g., top 100 common ports).  
- Integrate **service banner grabbing** on open ports for quick fingerprinting.  
- Add **rate limiting**, per-host concurrency limits, and progress reporting.  
- Export results to CSV/JSON for automated ingestion into reporting tools.

---

## Implementation note
This scanner is intentionally minimal and educational — it demonstrates Go concurrency, use of `net.DialTimeout`, basic IP range parsing, and straightforward output handling. Treat it as a foundation to build more robust scanning tools.

---

**Author:** Rasheed Jimoh  
**Language:** Go (Golang)  
**Intended use:** Lab / learning / authorized penetration testing only.
```
package main

import (
 "bufio"
 "fmt"
 "net"
 "os"
 "strconv"
 "strings"
 "sync"
 "time"
)

func scanPort(address string, port int, wg *sync.WaitGroup) {
 defer wg.Done()
 conn, err := net.DialTimeout("tcp", fmt.Sprintf("%s:%d",
address, port), time.Second*10)
 if err != nil {
 return
 }
 conn.Close()
 fmt.Println(address, "Port", port, "is open.")
}

func main() {
 fmt.Println("Enter IPs or a range of IPs (e.g. 192.168.1.1-
192.168.1.100)")
 scanner := bufio.NewScanner(os.Stdin)
 scanner.Scan()
 input := scanner.Text()
 var ipAddresses []string
 if strings.Contains(input, "-") {
 ipRange := strings.Split(input, "-")
 startIP := net.ParseIP(ipRange[0])
 endIP := net.ParseIP(ipRange[1])
 start := net.IPToBigInt(startIP)
 end := net.IPToBigInt(endIP)
 for i := start; i <= end; i++ {
 ipAddresses = append(ipAddresses, net.BigIntToIP
(i).String())
 }
 } else {
 ipAddresses = strings.Split(input, " ")
 }
 start := time.Now()
 fmt.Println("Starting port scan on targets:", ipAddresses)
 var wg sync.WaitGroup
 for _, address := range ipAddresses {
 for port := 1; port <= 65535; port++ {
 wg.Add(1)
 go scanPort(address, port, &wg)
 }
 }
 wg.Wait()
 elapsed := time.Since(start)
 fmt.Println("Port scan completed in", elapsed)
}
