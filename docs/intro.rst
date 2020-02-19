Lab Overview
------------

Introduction
~~~~~~~~~~~~

In this lab, you will see DNS over HTTPS (DoH) and DNS over TLS (DoT) queries proxied in both directions. That is, traditional DNS queries will be proxied to backend DoT/DoH servers, as well as DoH/DoT queries being proxied to traditional DNS servers.

DoT is "simpler" to proxy as the original DNS protocol is simply encapsulated in TLS using client-SSL (DoT-to-DNS) or server-ssl (DNS-to-DoT) profiles.

DoH is a bit more complex as we must take the DNS request and encapsulate it into a binary HTTPS payload (DNS-to-DoH) or extract the binary payload and convert it into a traditional DNS query (DoH-to-DNS).

We will use the power of iRulesLX to handle this advanced task. Native functionality will be available in a near-future release of TMOS so that the iRules will not be necessary.

