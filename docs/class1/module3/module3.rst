Proxying traditional DNS to DNS over TLS
----------------------------------------

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

|image17.png|

.. _viewing-statistics-1:

Viewing Statistics
~~~~~~~~~~~~~~~~~~

Back in the BIG-IP web UI, you will see that the VIP is receiving
connections:

|image18.png|

Issuing the same command with TCP will increment the counters on the
corresponding virtual server:

kdig +tcp @10.1.10.101 `www.f5.com <http://www.f5.com>`__

|image19.png|

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

|image20.png|

Stop your capture before moving on to the next section. This concludes
the DNS-to-DoT section.




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
