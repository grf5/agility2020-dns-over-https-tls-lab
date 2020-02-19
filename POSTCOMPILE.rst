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
