conversis-dcap
==============

Distributed Traffic Capture

You can provide ruby code on STDIN which will be evaluated for every captured packet.

# Options

  -t --runtime RUNTIME - time to capture packets in seconds (default is 60 seconds)  
  -i --interface INTERFACE - name of the interface to capture from  
  -m --modules MODULES - list of ruby modules to load separated by , (comma)  
  -f --filter PCAPFILTER - pcap filter  
  -o --outfile OUTFILE - name of file to append output to (default is STDOUT), this will daemonize the process
  -p --promisc - turn on promisc mode  
  -s --sessionid SESSIONID - a unique identifier to prepend all text with (default is a random string)  
  -r --regex REGEX - a regex to search for (ignored if code is input on STDIN)  
  -x --hex - output data in hex only  

# Variables
These variables can be used in your code  
  **protocol**					the IP protocol used (e.g. TCP/UDP/ICMP)  
  **source_address**			the IP a packet originated from  
  **source_port**				the port a packet originated from  
  **destination_address**		the destination IP of the packet  
  **destination_port**			the destination port of the packet  
  **content**					the payload of the packet (8bit)  
  **ascii_content**				the payload of the packet stripped of any non-ascii characters  
  **packet**            the raw packetfu object in all it's glory


* the content variable is mainly usfull when processing binary data or in combination with Conversis::Utils.hexify()
* if no interface name is provided dcap will try to figure it out itself
* if no code is provided on STDIN dcap will default to hexdump output
* the sessionid is an identifier that is usfull when running several dcap in parallel on many systems
* if an outfile is specified (-o) dcap will fork and detach itself from the terminal (i.e. daemonize)

# Dependencies
The following gems must exist locally:
* packetfu
* pcaprub

When installing dcap as a gem they will be automatically installed.

# Examples

  Hexdump captured packets on the default interface
   ```bash
  $ sudo dcap
  ```

  Run for one hour. Apply the code in mycode.rb for every packet. Load the openssl module. Sniff port 80 only.
   ```bash
  $ cat mycode.rb | sudo dcap -m openssl -f 'port 80' -t 3600

  $ cat mycode.rb
  [".{10}\x6C\x75\x6B\x61\x73.{10}", "neil"].each do |sig|
    hit = content.scan(/#{sig}/i) || nil
    puts "from %s to %s [%s]" % [source_address, destination_address, hit[0]] unless hit.size.zero?
  end
  ```

  Capture on interface en0. Output payload in hex. Output packets matching the regex in syslog format. Write output to a logfile.
  ```bash
  $ sudo dcap -i en0 -x -r '.{10}\x6C\x75\x6B\x61\x73.{10}' -o /var/log/dcap.log
  ```

  Capture on interface en0. Set a custom session identifier. Match a regex and output to a logfile.
  ```bash
  $ sudo dcap -i en0  -s 'myCaptureSession001' -r '.*lukas.*' -o /var/log/dcap.log
  ```

  The following code would sniff for HTTP GET requests and extract the requested URI out of the packet
  ```bash
  $ cat http.dcap | sudo dcap -i en0

  $ cat http.dcap
  /GET (?<url>.*?) HTTP.*/ =~ ascii_content
  puts "from %s getting url %s" % [destination_address, url] if url
  ```

  Capture RTP traffic and hexdump all the packets to a file in syslog format.
  ```bash
  $ sudo dcap -i en0 -x -r '.*' -f 'udp[1] & 1 != 1 && udp[3] & 1 != 1 && udp[8] & 0x80 == 0x80 && length < 250' -o /var/log/dcap.log
  ```

  Set interface to promisc mode (only useful on a mirror port) and display local IPs trying to connect to certain remote ports (useful for identifying machines infected with worms)
  ```bash
  $ echo 'puts source_address' | sudo dcap -i eth0 -p  -f '(dst port 135 or dst port 445 or dst port 1433) and tcp[tcpflags] & (tcp-syn) != 0 and tcp[tcpflags] & (tcp-ack) = 0 and src net 10.16.0.0/16'
  ```
