
TCP quickscan - Rustscan:

`rustscan -a 10.129.228.102 -r 0-65535 | tee -a tcp-scan`

UDP quickscan - Rustscan:

`rustscan -a 10.129.228.102 -r 0-65535 --udp | tee -a udp-scan`

TCP Scan - Nmap:

`nmap $ip -A -sC -sV -p- -o nmap`

UDP Scan - Nmap:

`nmap $ip -sU -A -p- -oN nmap_udp_results.txt`

`nmap 10.129.228.102 -sU -A -F`  (for top ports, quick scan)

