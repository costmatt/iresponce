# iresponce
Automates incident response reconnaissance on the source IP addresses of an attack from a pcap file using Wireshark and Nmap

### Script: `iresponse.sh`

```bash
#!/bin/bash

# Check if necessary tools are installed
if ! command -v tshark &> /dev/null || ! command -v nmap &> /dev/null || ! command -v whois &> /dev/null; then
    echo "Please install tshark, nmap, and whois to proceed."
    exit 1
fi

# Input PCAP file
PCAP_FILE="$1"

if [ -z "$PCAP_FILE" ]; then
    echo "Usage: $0 <pcap_file>"
    exit 1
fi

# Extract unique source IPs from the pcap file using tshark
echo "[+] Extracting source IP addresses from pcap file..."
SRC_IPS=$(tshark -r "$PCAP_FILE" -T fields -e ip.src | sort | uniq)

# Check if any IPs were extracted
if [ -z "$SRC_IPS" ]; then
    echo "No source IP addresses found in the pcap file."
    exit 1
fi

# Create a directory for output
OUTPUT_DIR="incident_recon_output"
mkdir -p "$OUTPUT_DIR"

# Process each IP address
for IP in $SRC_IPS; do
    echo "[+] Reconnaissance for IP: $IP"
    
    # WHOIS lookup for IP ownership
    echo "[+] Running WHOIS for IP ownership..."
    whois "$IP" > "$OUTPUT_DIR/${IP}_whois.txt"
    
    # Geolocation using an external API (e.g., ipinfo.io or ipgeolocation.io)
    # Example: using ipinfo.io API
    echo "[+] Fetching geolocation..."
    GEO_DATA=$(curl -s "https://ipinfo.io/$IP/json")
    echo "$GEO_DATA" | jq '.' > "$OUTPUT_DIR/${IP}_geo.json"
    
    # Nmap scan
    echo "[+] Scanning IP with Nmap..."
    nmap -Pn "$IP" > "$OUTPUT_DIR/${IP}_nmap_scan.txt"
    
    echo "[+] Reconnaissance for $IP completed. Results saved in $OUTPUT_DIR."
    echo ""
done

echo "[+] Incident response reconnaissance completed. Check the $OUTPUT_DIR directory for results."

```

### Key Points:

1. **Extracting source IPs**: 
   - We use `tshark` to read the pcap file and extract the `ip.src` field (source IP addresses).
   
2. **WHOIS Lookup**:
   - We use the `whois` command to gather information about the owner of the IP address.

3. **Geolocation**:
   - In the script, we use `ipinfo.io` as an example API for fetching geolocation details. You can replace this with any other IP geolocation API (ensure that you handle API rate limits or require an API key).

4. **Nmap Scan**:
   - The `nmap` tool is used to scan the IP address for open ports and services.

### How to Use:
1. Save the script as `iresponse.sh.sh`.
2. Make it executable:
   ```bash
   chmod +x iresponse.sh.sh
   ```
3. Run the script with your pcap file as input:
   ```bash
   ./iresponse.sh example.pcap
   ```
