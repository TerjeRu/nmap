# █▓▒▒░░░ NMAP: A PRACTICAL SCANNING MANUAL ░░░▒▒▓█

> Nmap (Network Mapper) is the industry standard for network discovery and security auditing. It uses raw IP packets to determine what hosts are available on a network, what services those hosts are offering, what operating systems they are running, and what type of packet filters/firewalls are in use. This manual provides a structured methodology for using Nmap effectively.
>
> Launch your Kali Linux virtual machine. All commands are to be executed from the terminal.

## 壱 - HOST DISCOVERY: MAPPING THE TERRAIN

> Before you can analyze a target, you must confirm it is online. Host discovery is the first phase, designed to identify live hosts on the network. This is more complex than a simple ping, as firewalls often block ICMP traffic.

### ► **LIVE EXERCISE: Initial Reconnaissance**

1. **Simple Ping Scan:** This is the most basic discovery method. It sends an ICMP echo request.

   * `nmap -sn 192.168.1.0/24`

   * `-sn`: No port scan. This flag tells Nmap to only perform host discovery.

   * `192.168.1.0/24`: Replace with your target network range. The output will be a list of hosts that responded.

2. **ARP Scan (Local Network):** On a local Ethernet network, ARP scans are faster and more reliable than ICMP pings. Nmap does this automatically for local subnets.

   * `sudo nmap -sn 192.168.1.0/24`

   * Note: On local networks, Nmap often uses ARP requests for discovery, which are highly effective.

3. **No Ping (Assume Host is Up):** If you know a host is online but blocking pings, you can skip the discovery phase and proceed directly to port scanning.

   * `nmap -Pn 192.168.1.50`

   * `-Pn`: Tells Nmap to skip the host discovery stage entirely. This is essential for scanning hosts protected by firewalls that drop discovery probes.

## 弐 - PORT SCANNING: PROBING FOR ENTRYPOINTS

> Once a host is identified, the next step is to discover which services it is offering. This is done by scanning its ports. A port's state (open, closed, or filtered) provides critical information.

### ► **LIVE EXERCISE: Identifying Open Doors**

1. **Default Scan (TCP SYN Scan):** This is the default and most popular scan type. It's fast, stealthy, and works on any compliant TCP stack. It sends a TCP packet with the SYN flag set. A SYN/ACK response indicates the port is open.

   * `sudo nmap -sS scanme.nmap.org`

   * `-sS`: Specifies a TCP SYN scan. Requires root privileges to craft raw packets.

2. **TCP Connect Scan:** If you don't have root privileges, Nmap defaults to this. It uses the operating system's `connect()` system call to establish a full connection. It's noisier and slower than a SYN scan.

   * `nmap -sT scanme.nmap.org`

   * `-sT`: Specifies a TCP Connect scan.

3. **UDP Scan:** Many important services run on UDP (e.g., DNS, DHCP, SNMP). UDP scanning is slower and more difficult than TCP scanning.

   * `sudo nmap -sU scanme.nmap.org`

   * `-sU`: Specifies a UDP scan.

4. **Targeted Port Scan:** Scanning all 65,535 ports is time-consuming. You can specify which ports to scan.

   * `nmap -p 80,443 scanme.nmap.org` (Scans only ports 80 and 443)

   * `nmap -p 1-1000 scanme.nmap.org` (Scans ports in the range 1 to 1000)

   * `nmap -F scanme.nmap.org` (Fast scan - scans the 100 most common ports)

## 参 - SERVICE & VERSION DETECTION: IDENTIFYING THE SOFTWARE

> Knowing a port is open is useful. Knowing what software is running on that port, and its version number, is critical for vulnerability analysis.

### ► **LIVE EXERCISE: Fingerprinting Services**

1. **Version Detection:** This scan sends a series of probes to open ports to determine the application protocol and version.

   * `sudo nmap -sV scanme.nmap.org`

   * `-sV`: Enables version detection.

2. **Controlling Intensity:** Version scanning intensity can be adjusted. A higher intensity is more likely to identify services but is slower and noisier.

   * `sudo nmap -sV --version-intensity 7 scanme.nmap.org`

   * `--version-intensity`: A value from 0 (light) to 9 (try all probes). The default is 7.

3. **Combining with a Targeted Scan:** For efficiency, combine version detection with a specific port list or a fast scan.

   * `sudo nmap -sV -F scanme.nmap.org`

## 四 - OS DETECTION: FINGERPRINTING THE HOST

> Nmap can analyze responses to a series of TCP/IP probes to make an educated guess about the target's operating system. This information helps in tailoring potential exploits.

### ► **LIVE EXERCISE: Identifying the Operating System**

1. **Standard OS Detection:**

   * `sudo nmap -O scanme.nmap.org`

   * `-O`: Enables OS detection. Requires at least one open and one closed port on the target to be effective.

2. **Aggressive Mode:** The `-A` flag is a convenient shortcut that enables OS detection, version detection, script scanning, and traceroute simultaneously.

   * `sudo nmap -A scanme.nmap.org`

   * `-A`: A common choice for comprehensive, albeit noisy, scanning.

## 五 - TIMING & STEALTH: CONTROLLING THE SCAN

> The speed and intensity of a scan can be adjusted to evade Intrusion Detection Systems (IDS) or to simply speed up a scan on a slow network.

### ► **LIVE EXERCISE: Pacing the Scan**

1. **Timing Templates:** Nmap has templates for controlling timing, from slow and stealthy to fast and aggressive.

   * `sudo nmap -T4 scanme.nmap.org`

   * `-T<0-5>`: Sets the timing template.

     * `T0`: Paranoid (very slow, for IDS evasion)

     * `T1`: Sneaky

     * `T2`: Polite

     * `T3`: Normal (default)

     * `T4`: Aggressive (assumes a fast, reliable network)

     * `T5`: Insane (can overwhelm targets)

2. **Manual Timing Controls:** For granular control, you can set specific timeouts.

   * `sudo nmap --max-rtt-timeout 100ms --initial-rtt-timeout 200ms scanme.nmap.org`

   * These options control how long Nmap waits for a response before giving up on a probe.

## 六 - NMAP SCRIPTING ENGINE (NSE): AUTOMATING THE HUNT

> The NSE is Nmap's most powerful feature. It allows users to write (and use pre-written) scripts to automate a wide variety of networking tasks, from advanced discovery to vulnerability detection.

### ► **LIVE EXERCISE: Unleashing Scripts**

1. **Default Scripts:** The `-sC` flag runs a set of default scripts that are considered safe and useful for discovery.

   * `sudo nmap -sC scanme.nmap.org`

   * `-sC`: Equivalent to `--script=default`.

2. **Running Script Categories:** Scripts are grouped into categories. You can run all scripts in a specific category.

   * `sudo nmap --script=vuln 192.168.1.50`

   * `--script=vuln`: Runs all scripts in the `vuln` category to check for known vulnerabilities. Use with caution and only on systems you have permission to test. Other categories include `discovery`, `dos`, `exploit`, `auth`, etc.

3. **Running a Specific Script:** You can call a single script by name.

   * `sudo nmap -p 80 --script=http-title scanme.nmap.org`

   * `http-title`: This script grabs the title from the web page on open port 80.

4. **Getting Help with a Script:** To learn what a script does and what arguments it takes:

   * `nmap --script-help "http-*"` (Shows help for all scripts starting with "http-")

## 七 - OUTPUT FORMATS: SAVING THE RESULTS

> Saving scan results is crucial for documentation and for processing with other tools. Nmap supports several output formats.

### ► **LIVE EXERCISE: Recording the Data**

1. **Multiple Formats:** The best practice is to save in all major formats at once.

   * `sudo nmap -A -oA scan_results scanme.nmap.org`

   * `-oA <basename>`: Outputs to all formats:

     * `scan_results.nmap` (Normal, human-readable format)

     * `scan_results.gnmap` (Grepable format, for easy parsing with command-line tools)

     * `scan_results.xml` (XML format, best for parsing with other programs)

2. **Grepable Output Example:** Use the `.gnmap` file to quickly extract information.

   * `grep "Host:" scan_results.gnmap | cut -d' ' -f2` (Extracts all live IP addresses from the scan)

> This manual covers the core functionality of Nmap. Mastery requires practice and a deep understanding of the underlying network protocols. Use this knowledge responsibly and ethically.
>
> **// END OF LINE //**
