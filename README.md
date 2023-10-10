# draft-jabley-dnsop-addr-query

A QTYPE to request all available addresses at once. Based on the TYPE65535 functionality implemented on Cloudflare's authoritative DNS servers.

    [monster:~]% dig @barbara.ns.cloudflare.com strandkip.nl TYPE65535 +dnssec +multiline +norec
    
    ; <<>> DiG 9.18.19 <<>> @barbara.ns.cloudflare.com strandkip.nl TYPE65535 +dnssec +multiline +norec
    ; (6 servers found)
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 32279
    ;; flags: qr aa; QUERY: 1, ANSWER: 6, AUTHORITY: 0, ADDITIONAL: 1
    
    ;; OPT PSEUDOSECTION:
    ; EDNS: version: 0, flags: do; udp: 1232
    ;; QUESTION SECTION:
    ;strandkip.nl.		IN TYPE65535
    
    ;; ANSWER SECTION:
    strandkip.nl.		300 IN A 104.18.6.53
    strandkip.nl.		300 IN A 104.18.7.53
    strandkip.nl.		300 IN AAAA 2606:4700::6812:635
    strandkip.nl.		300 IN AAAA 2606:4700::6812:735
    strandkip.nl.		300 IN RRSIG A 13 2 300 (
    				20231011122058 20231009102058 34505 strandkip.nl.
    				zHvYVrTWFX+If6UYy7tQYGxZAr6IkIdUHts27PAMI2e4
    				CHF1HnR2sehtBHVz8oJdt1R6wU6cWoICbnwVlcJ+wA== )
    strandkip.nl.		300 IN RRSIG AAAA 13 2 300 (
    				20231011122058 20231009102058 34505 strandkip.nl.
    				AGQtxW5iCzVIgMEtndTEIMDaxPR7kfVOTxApO+HZHdnk
    				saMhbiIqLrbBGO4VNXzFP5vEEwmvYhgukAdRwJW9nA== )
    
    ;; Query time: 3 msec
    ;; SERVER: 2606:4700:50::adf5:3af8#53(barbara.ns.cloudflare.com) (UDP)
    ;; WHEN: Tue Oct 10 11:20:58 UTC 2023
    ;; MSG SIZE  rcvd: 345
    
    [monster:~]% 
