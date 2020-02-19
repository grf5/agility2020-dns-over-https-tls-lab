Lab Goals
~~~~~~~~~

The lab consists of four sections:

-  Proxying DNS over HTTPS queries to traditional DNS servers

   -  In this section, you will use Mozilla Firefox as a DoH client to
      browse the web using encrypted DNS through the BIG-IP using DNS
      over HTTPS

-  Proxying DNS over TLS queries to traditional DNS servers

   -  In this section, you will use the kdig utility as a DoT client to
      perform queries through the BIG-IP using DNS over TLS

-  Proxying traditional DNS queries to DNS over HTTPS servers

   -  In this section, you will use the nslookup/dig utilities to send
      traditional DNS queries through the BIG-IP to Google's DoH service

-  Proxying traditional DNS queries to DNS over TLS servers

   -  In this section, you will use the nslookup/dig utilities to send
      traditional DNS queries through the BIG-IP to Google's DoT service


.. toctree::
   :maxdepth: 1
   :glob:

   module*/module*
   resources*