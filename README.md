conversis-dcap
==============

Distributed Traffic Capture

You can provide ruby code on STDIN which will be evaluated for every captured packet.

# Dependencies
The following gems must exist locally:
* packetfu
* pcaprub

When installing dcap as a gem they will be automatically installed.

# Options

  -t --runtime RUNTIME - time to capture packets in seconds (default is 60 seconds)  
  -i --interface INTERFACE - name of the interface to capture from  
  -m --modules MODULES - list of ruby modules to load separated by , (comma)  
  -f --filter PCAPFILTER - pcap filter  
  -o --outfile OUTFILE - name of file to append output to (default is STDOUT)  
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

 * the content variable is mainly usfull when processing binary data or in combination with Conversis::Utils.hexify()
 * if no interface name is provided dcap will try to figure it out itself
 * if no code is provided on STDIN dcap will default to hexdump output
 * the sessionid is an identifier that is usfull when running several dcap in parallel on many systems

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
