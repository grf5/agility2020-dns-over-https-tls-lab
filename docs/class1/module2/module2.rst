Proxying DNS over TLS Queries to Traditional DNS
------------------------------------------------

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

|image11.png|

Clicking on *Resources* tab on the top navigation bar will show that the
virtual server has a simple pool and no iRules attached.

|image12.png|

.. _test-drive-1:

Test Drive
~~~~~~~~~~

Let’s return to the desktop and launch the Lab DNS Server client. You’ll
be automagically logged in. Let’s run a DNS over TLS query:

kdig +tls @10.1.10.100 www.f5.com

|image13.png|

Viewing Statistics
~~~~~~~~~~~~~~~~~~

You can then see statistics on the virtual server by navigating to
**Statistics** -> **Module Statistics** -> **Local Traffic** and
selecting *Virtual Servers* in the drop-down list.

|image14.png|

Because this virtual server is taking advantage of backend pools, you
will see statistics under the *Pools* statistics type as well.

|image15.png|

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

|image16.png|

Stop your capture before moving on to the next section. This concludes
the DoT-to-DNS portion of the lab.




.. |image1.png| image:: media/image1.png
   :width: 7.5in
   :height: 5.29969in
.. |image2.png| image:: media/image2.png
   :width: 7.5in
   :height: 4.6875in
.. |image3.png| image:: media/image3.png
   :width: 7.5in
   :height: 4.6875in
.. |image4.png| image:: media/image4.png
   :width: 7.5in
   :height: 4.47917in
.. |image5.png| image:: media/image5.png
   :width: 7.5in
   :height: 4.48438in
.. |image6.png| image:: media/image6.png
   :width: 7.5in
   :height: 4.4775in
.. |image7.png| image:: media/image7.png
   :width: 2.39879in
   :height: 2.88051in
.. |image8.png| image:: media/image8.png
   :width: 7.5in
   :height: 4.47917in
.. |image9.png| image:: media/image9.png
   :width: 7.5in
   :height: 4.47917in
.. |image10.png| image:: media/image10.png
   :width: 7.5in
   :height: 3.89006in
.. |image11.png| image:: media/image11.png
   :width: 7.5in
   :height: 4.47917in
.. |image12.png| image:: media/image12.png
   :width: 7.5in
   :height: 4.47396in
.. |image13.png| image:: media/image13.png
   :width: 7.5in
   :height: 4.47917in
.. |image14.png| image:: media/image14.png
   :width: 7.5in
   :height: 4.54167in
.. |image15.png| image:: media/image15.png
   :width: 7.5in
   :height: 4.47917in
.. |image16.png| image:: media/image16.png
   :width: 7.5in
   :height: 4.47917in
.. |image17.png| image:: media/image17.png
   :width: 7.5in
   :height: 4.47917in
.. |image18.png| image:: media/image18.png
   :width: 7.5in
   :height: 4.47917in
.. |image19.png| image:: media/image19.png
   :width: 7.5in
   :height: 3.19271in
.. |image20.png| image:: media/image20.png
   :width: 7.5in
   :height: 3.74479in
.. |image21.png| image:: media/image21.png
   :width: 7.5in
   :height: 2.85417in
.. |image22.png| image:: media/image22.png
   :width: 7.5in
   :height: 3.51563in
.. |image23.png| image:: media/image23.png
   :width: 7.5in
   :height: 3.46314in
.. |image24.png| image:: media/image24.png
   :width: 7.5in
   :height: 3.48958in
.. |image25.png| image:: media/image25.png
   :width: 7.5in
   :height: 4.47396in
.. |image26.png| image:: media/image26.png
   :width: 2.75in
   :height: 6.40278in
.. |image27.png| image:: media/image27.png
   :width: 7.5in
   :height: 4.55208in
.. |image28.png| image:: media/image28.png
   :width: 7.5in
   :height: 10in
.. |image29.png| image:: media/image29.png
   :width: 7.5in
   :height: 6.98222in
.. |image30.png| image:: media/image30.png
   :width: 7.5in
   :height: 4.76136in
.. |image31.png| image:: media/image31.png
   :width: 7.5in
   :height: 3.45313in
.. |image32.png| image:: media/image32.png
   :width: 7.5in
   :height: 3.51563in
.. |image33.png| image:: media/image33.png
   :width: 7.5in
   :height: 4.49479in
.. |image34.png| image:: media/image34.png
   :width: 7.5in
   :height: 4.37598in
.. |image35.png| image:: media/image35.png
   :width: 7.5in
   :height: 3.49479in
.. |image36.png| image:: media/image36.png
   :width: 7.5in
   :height: 3.46875in
.. |image37.png| image:: media/image37.png
   :width: 7.5in
   :height: 4.47396in
.. |image38.png| image:: media/image38.png
   :width: 7.5in
   :height: 2.99202in
.. |image39.png| image:: media/image39.png
   :width: 7.5in
   :height: 3.50243in
.. |image40.png| image:: media/image40.png
   :width: 7.5in
   :height: 3.59375in
.. |image41.png| image:: media/image41.png
   :width: 7.5in
   :height: 1.45278in

