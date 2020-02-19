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

BIG-IP Configuration Review
---------------------------

Launch your RDP client and connect to the Windows Jump Host.

|A screen shot of a computer Description automatically generated|

Click “No” to close the network discovery prompt.

Click on the Firefox icon to launch the browser.

Three tabs will open up. The first tab is the UI of the BIG-IP. Let’s
login using **admin** for our username and **f5agility2020** as our
password.

|A screenshot of a cell phone Description automatically generated|

You should see the license screen initially. Let’s take a look at the
configuration before we proceed with testing the proxy.

System Configuration
~~~~~~~~~~~~~~~~~~~~

Resource Provisioning
^^^^^^^^^^^^^^^^^^^^^

First, let’s look at how the platform’s modules are provisioned.
Navigate to **System** -> **Resource Provisioning** in the menu. You
will see that we have LTM and iRulesLX provisioned. We’ll need both of
these modules for handling DNS connections and translating between DNS
and HTTPS.

|A screenshot of a computer Description automatically generated|

NTP
^^^

Next, let’s look at a few key system settings necessary for overall
system health. Navigate to **System** -> **Configuration** -> **Device**
-> **NTP**. It’s important that NTP is configured and working properly,
especially when a BIG-IQ is paired or managed by BIG-IQ.

|A screenshot of a social media post Description automatically
generated|

DNS
^^^

Navigate to **System** -> **Configuration** -> **Device** -> **DNS**

Because we’re using FQDNs in our iRules and DNS pools, we’ll need DNS
resolvers that the system can use.

**If you didn’t want to use public DNS servers in a certain environment,
you could simply assign static addresses to pool members and resolvers
to alleviate this requirement.**

|A screenshot of a computer Description automatically generated|

Network Configuration
~~~~~~~~~~~~~~~~~~~~~

The BIG-IP sits in two VLANs with self-IPs in each. One side serves up
the DNS VIPs and the other is used to reach DNS servers. If you wish to
view this portion of the config, you can click on the respective
sections under the Network menu. Please do not make any changes.

|A screenshot of a cell phone Description automatically generated|

Local Traffic Manager (LTM)
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Let’s now look at the portion of the configuration that is performing
the heavy lifting – the LTM configuration.

Nodes
^^^^^

Navigate to **Local Traffic** -> **Nodes** and look at the node list.
Here, we’re resolving dns.google and automatically creating pool members
based on the records returned.

|A screenshot of a social media post Description automatically
generated|

Pools
^^^^^

If you’ll kindly navigate to **Local Traffic** -> **Pools**, you will
see three pools. While the backend nodes are identical between them, the
ports used for each are not. You’ll see a pool for DNS over HTTPS
(doh_dns.google) that uses port 443, a pool for DNS over TLS
(dot_dns.google) that utilizes port 853 and finally a pool that uses
port 53 for traditional DNS services (traditional_dns.google). If you’re
not familiar with LTM pools, click through each pool to see how the
service ports are specified.

|A screenshot of a computer Description automatically generated|

iRulesLX
^^^^^^^^

iRulesLX engine based on Node.js is the mechanism that we will leverage
to handle DNS over HTTPS translations. DoH requests either arrive at the
BIG-IP in an HTTPS POST with a binary payload or a base64url- encoded
GET request parameter. We’ll need to transpose the data from these
requests and translate into a traditional DNS request (DoH-to-DNS). We
can also take a traditional DNS request and encapsulate it into a DoH
request using iRulesLX.

Workspaces
''''''''''

If you’ll navigate to **Local Traffic** -> **iRules** -> **LX
Workspaces**, you can see the two rules for handling conversions in
their respective direction.

|A screenshot of a social media post Description automatically
generated|

DNS to DoH Proxy
                

Click on the *DNS_to_DoH_Proxy* item under the *rules* section of
**Workspace Files**. The first rule, *DNS_to_DoH_Proxy*, has two
components. The classic iRule, which is written in TCL, is used to nab
data from the incoming payload and pass it off to iRulesLX. The
ILX::init function is called and the entire UDP payload is simply passed
to iRulesLX using base64 encoding. Once the request is processed, the
response will be returned to this iRule, which will be base64 decoded
and passed to the client.

|A screenshot of a social media post Description automatically
generated|

Click on the *index.js* file under the *dns_over_https* section of
**Workspace Files**. The iRulesLX portion takes the DNS packet’s payload
and sends it to a remote DoH server as a binary payload using the HTTP
POST method. The response, which will also be binary, gets base64
encoded and passed back to the TCL portion of the iRule, which then
sends the request back to the client.

|A screenshot of a computer Description automatically generated|

DoH to DNS Proxy
                

Navigate back to the iRulesLX Workspace list (**Local Traffic** ->
**iRules** -> **iRulesLX Workspaces**) and view the *DoH_to_DNS_Proxy*
iRule. Click on the *DoH_to_DNS_Proxy* item under the *rules* section of
**Workspace Files**. This conversion is a more intensive task.

First, POST and GET are both valid DoH request methods, but have
different payloads. POST payloads are binary and GET requests are
base64url encoded in the URI request, so we need to treat them
separately.

Since POST has the request in the actual HTTP payload, we’ll have to
grab that information, perform base64 encoding and pass it along to
iRulesLX to process.

For GET requests, we can simply send the base64url-encoded GET
parameter. In both cases, we’ll also have to wait for a response from
the iRulesLX engine, which is handled in this portion of the iRule as
well.

There is a slight distinction between base64 and base64url encoding! For
more information, see https://en.wikipedia.org/wiki/Base64.

|A screenshot of a computer Description automatically generated|

Click on the *index.js* item under *DoH_to_DNS_Proxy* section of
**Workspace Files**. For the iRulesLX portion, the script has several
steps it must perform.

First, we need to get the DoH request into a traditional DNS request
packet. Not only that, but we need check for truncated responses from
UDP requests and resend them as TCP requests. Once we have a response
from the DNS server, we’ll need to encode it to pass back to TCL so the
final response can be returned to the server.

The process intensive iRule can take advantage of the BIG-IPs native SSL
and TCP protocol accelerations, greatly increasing the volume of
requests that can be handled.

|A screenshot of a computer Description automatically generated|

Plugins
'''''''

Navigate to **Local Traffic** -> **iRules** -> **LX Plugins**. This is
where a workspace is mapped to a plug-in. This allows you to make
changes to the workspace without committing those changes immediately.

|A screenshot of a cell phone Description automatically generated|

Virtual Servers 
^^^^^^^^^^^^^^^

Finally, let’s take a look at the virtual servers handling incoming
requests. Navigating to **Local Traffic** -> **Virtual Servers** will
bring up the list.

Notice that we have 5 scenarios to cover in order to handle DNS
translations in either direction.

First, the DNS-to-DoH virtual server handles incoming traditional DNS
requests and encapsulates them to a backend DoH server. The next two
rules handle DNS-to-DoT for both inbound TCP and UDP requests. An
example use case for these proxies would be offering encrypted DNS
services to client software/hardware that doesn’t support DoH/DoT.

The next two rules handle inbound DoH and DoT requests, respectively. An
example use case for these proxies would be for offering DoH/DoT to
clients/customers/etc. without the need for modifying existing DNS
infrastructure.

|A screenshot of a computer Description automatically generated|

Proxying DNS over HTTPS Queries to Traditional DNS 
--------------------------------------------------

Certificate Requirements for DoH/DoT Virtual Servers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

DNS over HTTPS requires a valid server-side certificate. In our lab, we
created a self-signed CA certificate as well as a self-signed
certificate for the server. We loaded those certificates in your Firefox
browser so that the browser will trust the BIG-IP DoH resolver.

In a real-world scenario, you would need a certificate signed by a
well-known certificate authority and loaded into the BIG-IP and attached
to the client-ssl profile in use for DoH/DoT listeners. Most DoH
clients, including Firefox, will not trust a DoH server if the
certificate is not signed by a known certificate authority.

Test Drive
~~~~~~~~~~

Now, let’s generate some traffic and see the translations in real-time.

Firefox Configuration
~~~~~~~~~~~~~~~~~~~~~

For this test, we’re going to use Firefox as our DoH client. Click the
second tab in Firefox to view the about:config page. On the top of that
page, you’ll see a search box. Enter *trr* and press enter to see the
DoH (trusted recursive resolver) configuration.

|A screenshot of a computer Description automatically generated|

We’ve pre-configured a few things for you. First, we set
*network.trr.uri* to our custom virtual server URL. We also enabled
*network.trr.useGET* as it’s a bit faster than using POST, but you’re
welcome to test using POST as well. We set *network.trr.mode* to 3,
which means we want Firefox to **only** use DoH. This will not be a
typical configuration as Firefox defaults to traditional DNS when a DoH
request fails. That explains the differing timeout values just below
that setting. The *network.dns.skipTRR-when-parental-control-enabled*
disables Firefox’s feature that disables DoH when parental control via
DNS is sensed on the network.

**Please keep in mind that these settings are changing as Firefox
continues testing DoH. The ink on the RFC is still wet, technically, and
those heavily involved in encrypted DNS are still working out the
nuances.**

Firefox Network Utilities
~~~~~~~~~~~~~~~~~~~~~~~~~

Clicking on the third tab in Firefox will open the networking tools page
within the browser. This is a great way to see if DoH (TRR in
Mozilla-speak) is working. Click on *DNS Lookup* to bring up the DNS
query tool.

|A screenshot of a social media post Description automatically
generated|

Entering a URL and clicking *Resolve* will show the A/AAAA records
returned for that FQDN.

|A screenshot of a social media post Description automatically
generated|

If you then click on *DNS*, you’ll be presented with a table of the
current in-browser DNS cache. Click on *Refresh* to update the view. You
can see in the output below that TRR was *true* for the queries sent,
meaning DoH was used to resolve those hostnames.

|A screenshot of a cell phone Description automatically generated|

DoH in Action
~~~~~~~~~~~~~

Open a new tab and browse to a website. Return to the third tab and
click *Refresh* to see the updated DNS cache table.

|A screenshot of a cell phone Description automatically generated|

BIG-IP Statistics and Logging
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Back in the first tab on the F5 web UI, navigate to **Statistics** ->
**Module Statistics** -> **Local Traffic**. Make sure that *Virtual
Servers* is selected in the *Statistics Type* drop-down. Observe the
traffic statistics on the DoH-to-DNS virtual server.

|A screenshot of a computer Description automatically generated|

Change the *Statistics Type* to iRulesLX and you can see how many RPC
connections have been made.

|A screenshot of a computer Description automatically generated|

Change the drop-down to *Pools*. You should notice that the back-end
pools show 0 connections. Why? Because iRulesLX is talking to the
back-end DoH resolvers directly. You could point your DoH iRule to a
local VIP with a DNS pool for better performance, stability, etc. but
that is outside the scope of this lab.

|A screenshot of a computer screen Description automatically generated|

Navigate to **System** -> **Logs** -> **Local Traffic**. Notice that
some useful information is being logged to help show the parsing and
querying that is taking place behind the scenes.

|A screenshot of a computer Description automatically generated|

Packet Capture
~~~~~~~~~~~~~~

Finally, minimize *Firefox* to reveal the CLI shortcuts on the desktop:

|A screen shot of a smart phone Description automatically generated|

Let’s open the BIG-IP DNS Proxy link to bring up the BIG-IP’s CLI. Once
running, let’s start a capture that will show us both sides of the DoH
proxy:

tcpdump -nni 0.0 (host 10.1.1.4 and host 10.1.10.100 and port 443) or
(host 8.8.4.4 or host 8.8.8.8 and port 53)

Once running, maximize *Firefox* and perform another DNS lookup. View
the HTTPS and DNS traffic in the packet capture output. The output below
shows my queries to f5.com, f5agility.com and disney.com.

|A screenshot of a computer Description automatically generated|

Stop your capture before moving to the next section. This concludes the
DoH-to-DNS proxy portion of the lab.

Proxying DNS over TLS Queries to Traditional DNS

DoT-to-DNS is a bit more simplistic. We’re simply taking the existing
DNS request and encapsulating it in TLS. If you review the virtual
server configuration, you’ll notice that we’re simply using a client-SSL
profile and a backend pool. No iRule magic needed here; just classic
BIG-IP high-performance SSL offloading.

**The client-SSL profile on this virtual server specifies that SSL/TLS
termination should occur on the client side of the connection.**

Virtual Server Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Maximize *Firefox*. Notice the virtual server for DoT-to-DNS is very
simple:

|A screenshot of a cell phone Description automatically generated|

Clicking on *Resources* tab on the top navigation bar will show that the
virtual server has a simple pool and no iRules attached.

|A screenshot of a cell phone Description automatically generated|

.. _test-drive-1:

Test Drive
~~~~~~~~~~

Let’s return to the desktop and launch the Lab DNS Server client. You’ll
be automagically logged in. Let’s run a DNS over TLS query:

kdig +tls @10.1.10.100 www.f5.com

|A screenshot of a computer screen Description automatically generated|

Viewing Statistics
~~~~~~~~~~~~~~~~~~

You can then see statistics on the virtual server by navigating to
**Statistics** -> **Module Statistics** -> **Local Traffic** and
selecting *Virtual Servers* in the drop-down list.

|A screenshot of a computer screen Description automatically generated|

Because this virtual server is taking advantage of backend pools, you
will see statistics under the *Pools* statistics type as well.

|A screenshot of a computer Description automatically generated|

Because we don’t have any type of logging configured for that virtual
server, you won’t see any information in **System** -> **Logs** for this
traffic. Conventional F5 logging/statistics practices can be used for
these connections, so we’ll move on.

.. _packet-capture-1:

Packet Capture 
~~~~~~~~~~~~~~

Maximize the BIG-IP CLI window. Execute the follow tcpdump command:

tcpdump -nni 0.0 port 53 or port 853

Return to the Ubuntu Jump Host and re-run your **kdig** command. Observe
the front and back-end connections using port 853 and 53, respectively.

|A picture containing window Description automatically generated|

Stop your capture before moving on to the next section. This concludes
the DoT-to-DNS portion of the lab.

Proxying traditional DNS to DNS over TLS

In this section of the lab, we’re going to run DoT in the opposite
direction, taking traditional DNS requests and translating them into DoT
requests. This is done as simply as the DoT-to-DNS; we simply take the
incoming DNS connection (UDP or TCP) and encapsulate it in TLS using a
server-side SSL profile.

.. _test-drive-2:

Test Drive
~~~~~~~~~~

On the Ubuntu jump host, issue the following command:

kdig @10.1.10.101 `www.yahoo.com <http://www.yahoo.com>`__

You should receive a successful response as shown below:

|A screenshot of a cell phone Description automatically generated|

.. _viewing-statistics-1:

Viewing Statistics
~~~~~~~~~~~~~~~~~~

Back in the BIG-IP web UI, you will see that the VIP is receiving
connections:

|A screenshot of a computer screen Description automatically generated|

Issuing the same command with TCP will increment the counters on the
corresponding virtual server:

kdig +tcp @10.1.10.101 `www.f5.com <http://www.f5.com>`__

|A screenshot of a computer Description automatically generated|

Again, nothing super-fancy is happening in this configuration.
Conventional F5 logging methods can be used for this traffic so we won’t
cover that in this lab.

.. _packet-capture-2:

Packet Capture
~~~~~~~~~~~~~~

We can see the 53/853 exchange on a packet capture using the same
**tcpdump** command we used in the DoT-to-DNS section, as the IP/ports
are simply being switched around:

tcpdump -nni 0.0 (host 10.1.20.10 or 10.1.1.6) and (port 53 or port 853)

You will see the 53 and 853 connections in the output, as shown below.

|image36|

Stop your capture before moving on to the next section. This concludes
the DNS-to-DoT section.

Proxying traditional DNS queries to DNS over HTTPS
--------------------------------------------------

Finally, let’s look at converting a DNS query to a DoH request.

.. _test-drive-3:

Test Drive
~~~~~~~~~~

We’ll once again use **kdig** as we’re simply generating a traditional
DNS request.

kdig @10.1.10.102 `www.f5agility.com <http://www.f5agility.com>`__

You’ll get a response as shown below:

|A close up of a screen Description automatically generated|

.. _viewing-statistics-2:

Viewing Statistics
~~~~~~~~~~~~~~~~~~

Back on the BIG-IP, we’ll see connections on the DNS-to-DoH virtual
server:

|A screenshot of a computer Description automatically generated|

If we set the statistics type to *iRulesLX*, we’ll see RPC connections
on the iRule for this translation:

|A screenshot of a computer Description automatically generated|

.. _packet-capture-3:

Packet Capture
~~~~~~~~~~~~~~

Running a packet capture, we can see the front-end udp/53 requests being
translated to DoH requests:

tcpdump -nni 0.0 (host 10.1.10.102 and port 53) or (host 8.8.4.4 or host
8.8.8.8 and port 443)

**If your packet capture is “noisy,” removing the HTTPS monitor from the
“doh_google.dns” pool will stop the intermittent queries.**

Notice that a port 53 request comes in, a HTTPS connection is set up and
the query is passed, then the port 53 response is sent to the client
before the HTTPS connection is torn down.

|image40|

This concludes the hands-on portion of the lab. Feel free to explore and
test the environment if there is time remaining.

Additional Resources
--------------------

The following resources will allow you to explore DoH and DoT more, and
setup this functionality in your own environment.

-  RFC8484: DNS over HTTPS: https://tools.ietf.org/html/rfc8484

-  RFC7858: DNS over TLS: https://tools.ietf.org/html/rfc7858

-  Github repository with iRules and sample configuration:
   https://github.com/grf5/DoHDotiRulesLX

.. |A screenshot of a cell phone Description automatically generated| image:: media/image1.png
   :width: 7.5in
   :height: 5.29969in
.. |A screen shot of a computer Description automatically generated| image:: media/image2.png
   :width: 7.5in
   :height: 4.6875in
.. |A screenshot of a cell phone Description automatically generated| image:: media/image3.png
   :width: 7.5in
   :height: 4.6875in
.. |A screenshot of a computer Description automatically generated| image:: media/image4.png
   :width: 7.5in
   :height: 4.47917in
.. |A screenshot of a social media post Description automatically generated| image:: media/image5.png
   :width: 7.5in
   :height: 4.48438in
.. |A screenshot of a computer Description automatically generated| image:: media/image6.png
   :width: 7.5in
   :height: 4.4775in
.. |A screenshot of a cell phone Description automatically generated| image:: media/image7.png
   :width: 2.39879in
   :height: 2.88051in
.. |A screenshot of a social media post Description automatically generated| image:: media/image8.png
   :width: 7.5in
   :height: 4.47917in
.. |A screenshot of a computer Description automatically generated| image:: media/image9.png
   :width: 7.5in
   :height: 4.47917in
.. |A screenshot of a social media post Description automatically generated| image:: media/image10.png
   :width: 7.5in
   :height: 3.89006in
.. |A screenshot of a social media post Description automatically generated| image:: media/image11.png
   :width: 7.5in
   :height: 4.47917in
.. |A screenshot of a computer Description automatically generated| image:: media/image12.png
   :width: 7.5in
   :height: 4.47396in
.. |A screenshot of a computer Description automatically generated| image:: media/image13.png
   :width: 7.5in
   :height: 4.47917in
.. |A screenshot of a computer Description automatically generated| image:: media/image14.png
   :width: 7.5in
   :height: 4.54167in
.. |A screenshot of a cell phone Description automatically generated| image:: media/image15.png
   :width: 7.5in
   :height: 4.47917in
.. |A screenshot of a computer Description automatically generated| image:: media/image16.png
   :width: 7.5in
   :height: 4.47917in
.. |A screenshot of a computer Description automatically generated| image:: media/image17.png
   :width: 7.5in
   :height: 4.47917in
.. |A screenshot of a social media post Description automatically generated| image:: media/image18.png
   :width: 7.5in
   :height: 4.47917in
.. |A screenshot of a social media post Description automatically generated| image:: media/image19.png
   :width: 7.5in
   :height: 3.19271in
.. |A screenshot of a cell phone Description automatically generated| image:: media/image20.png
   :width: 7.5in
   :height: 3.74479in
.. |A screenshot of a cell phone Description automatically generated| image:: media/image21.png
   :width: 7.5in
   :height: 2.85417in
.. |A screenshot of a computer Description automatically generated| image:: media/image22.png
   :width: 7.5in
   :height: 3.51563in
.. |A screenshot of a computer Description automatically generated| image:: media/image23.png
   :width: 7.5in
   :height: 3.46314in
.. |A screenshot of a computer screen Description automatically generated| image:: media/image24.png
   :width: 7.5in
   :height: 3.48958in
.. |A screenshot of a computer Description automatically generated| image:: media/image25.png
   :width: 7.5in
   :height: 4.47396in
.. |A screen shot of a smart phone Description automatically generated| image:: media/image26.png
   :width: 2.75in
   :height: 6.40278in
.. |A screenshot of a computer Description automatically generated| image:: media/image27.png
   :width: 7.5in
   :height: 4.55208in
.. |A screenshot of a cell phone Description automatically generated| image:: media/image28.png
   :width: 7.5in
   :height: 10in
.. |A screenshot of a cell phone Description automatically generated| image:: media/image29.png
   :width: 7.5in
   :height: 6.98222in
.. |A screenshot of a computer screen Description automatically generated| image:: media/image30.png
   :width: 7.5in
   :height: 4.76136in
.. |A screenshot of a computer screen Description automatically generated| image:: media/image31.png
   :width: 7.5in
   :height: 3.45313in
.. |A screenshot of a computer Description automatically generated| image:: media/image32.png
   :width: 7.5in
   :height: 3.51563in
.. |A picture containing window Description automatically generated| image:: media/image33.png
   :width: 7.5in
   :height: 4.49479in
.. |A screenshot of a cell phone Description automatically generated| image:: media/image34.png
   :width: 7.5in
   :height: 4.37598in
.. |A screenshot of a computer screen Description automatically generated| image:: media/image35.png
   :width: 7.5in
   :height: 3.49479in
.. |A screenshot of a computer Description automatically generated| image:: media/image36.png
   :width: 7.5in
   :height: 3.46875in
.. |image36| image:: media/image37.png
   :width: 7.5in
   :height: 4.47396in
.. |A close up of a screen Description automatically generated| image:: media/image38.png
   :width: 7.5in
   :height: 2.99202in
.. |A screenshot of a computer Description automatically generated| image:: media/image39.png
   :width: 7.5in
   :height: 3.50243in
.. |A screenshot of a computer Description automatically generated| image:: media/image40.png
   :width: 7.5in
   :height: 3.59375in
.. |image40| image:: media/image41.png
   :width: 7.5in
   :height: 1.45278in
