# Welcome to the Yeti Project

"One World, One Internet, One Namespace" is the essence for the success of today's Internet. The top level of the unique identifier system, the DNS root system, has been operational for 25+ years. It is a pivot to make the current Internet useful. So it is considered somewhat ossified for stability reasons. It is hard to test and implement new ideas evolving to a more advanced level to counter challenges like IPv6-only operation, DNSSEC key/algorithm rollover, scaling issues, etc. Because changes to the current production system are not risk-free, and a large-scale testbed is necessary, on which extensive tests and studies of the impact can be carried out. It is possible to pre-evaluate such changes in a laboratory test, however, the coverage is limited. In order to make the test more practical, it is also necessary to involve users’ environment which is highly diversified, to study the effect of the changes in question. 
 
As a result the proposal of Yeti Project is formed which is aiming to build such a live environment. The basic idea is to build a parallel experimental live DNS root system to provide useful technical output with the existing production systems. Possible research agenda will be explored on this testbed covering several aspects but not limited to:

* IPv6-only operation
* DNSSEC key rollover
* Renumbering issues
* Scalability issues
* Multiple zone file signers

In addition measurement study will be done to create useful analysis as output of Yeti Project.

Participants will be invited into Yeti Project which is hopefully helpful for their own interests. We would like to invite some interested parties in this community, like the research labs of universities and institutes. We also hope Yeti Project can gain the support from vendors, for example, the DNS software implementers, developers of CPE devices & IoT devices, middlebox developers who can test their product and connect their own testbed into Yeti testbed. It is expected that the activity of Yeti Project could have good input to the DNS related industry, which will finally make them ready for advanced DNS and Root services.  

*NOTE THAT THIS PROJECT NEVER INTENTS TO REPLACE CURRENT OPERATIONAL ROOT DNS SYSTEM.　REGULAR DNS SERVICES MUST NOT DEPEND ON THE TESTBED.*

# How to Join Yeti

Please contact <discuss@lists.yeti-dns.org> if you wish to ask questions and receive latest updates in the Yeti project.

Below are details about the different ways of joining the effort.

1. Joining Yeti as a Yeti Root Server Operator

    To run a Yeti root server, you need a server with good IPv6 Internet access, and a dedicated domain name of the root server which is configured as a slave to the Yeti distribution masters (DM). There are a few steps to join Yeti as a Yeti root server operator.

    **Step 1: Application**

    Please send a mail to <coordinators@lists.yeti-dns.org> with your basic contact information, short introduction or a short declaration to join Yeti Project as a volunteer authority server. Most importantly send a domain name and IPv6 address of the new root server which is to be added into the apex NS  RRset of our root zone. 

    Note that even though we publish strictly IANA information for TLD data and metadata, it's necessary for us to replace the apex NS RRset.  Once we verify that your server is operational we will add it to the apex NS RRset, and we will add you and your designated colleagues to the <operators@lists.yeti-dns.org> mailing list.

    **Step 2: Root server setup**

    The root server must provide DNS service only over IPv6. No A record and no answer when queried over IPv4. 

    ACLs are in place on some of the distribution masters so you need
    to request a hole for your server's IPv6 address (send an email to
    coordinators@lists.yeti-dns.org. Test with `dig
    @$DistributionMaster AXFR .` to see if you can do a zone
    transfer. You may have to add `-b $ServiceIPaddress` if your
    machine is multihomed.
    
    Configure the root server as a slave to the Yeti DM. You can add the following to the configuration file of your root server.

    BIND:

    ```
    masters yeti-dm {
        240c:f:1:22::7;            # bii
        2001:200:1d9::53;          # wide
        2001:4f8:3:1006::1:5;      # tisf
        };
    ```
    
    NSD:

    ```
    zone:
        name: "."
	# BII
        request-xfr: 240c:f:1:22::7 NOKEY
	# WIDE
        request-xfr: 2001:200:1d9::53 NOKEY
	# TISF
        request-xfr: 2001:4f8:3:1006::1:5 NOKEY
        allow-notify: 240c:f:1:22::7 NOKEY
        allow-notify: 2001:200:1d9::53 NOKEY
        allow-notify: 2001:4f8:3:1006::1:5 NOKEY
    ```

    Knot:
    
    ```
    remote:
      - id: yeti-dm-bii
        address: 240c:f:1:22::7
      - id: yeti-dm-wide
        address: 2001:200:1d9::53
      - id: yeti-dm-tisf
        address: 2001:4f8:3:1006::1:5
    acl:
      - id: yeti-notify-bii
        address: 240c:f:1:22::7
        action: notify
      - id: yeti-notify-wide
        address: 2001:200:1d9::53
        action: notify
      - id: yeti-notify-tisf
        address: 2001:4f8:3:1006::1:5
        action: notify
    zone:
      - domain: "."
        file: "root.zone"
        master: [ yeti-dm-bii, yeti-dm-wide, yeti-dm-tisf ]
        acl: [ yeti-notify-bii, yeti-notify-wide, yeti-notify-tisf ]
    ```

    Afterward, please send a mail to coordinators' mailing list to notify that it is done.

    **Step 3: Monitoring system setup**

    For the purpose of experiment and measurement study, we require each root server operator to capture DNS packets on DNS servers and save as pcap file, then send to our storage server. Regarding the data sharing issue, please turn to the [data sharing document](doc/DataSharingDeclaration.md) of YETI Project.

    Please read the following link how to setup and join the YETI monitoring system: https://github.com/BII-Lab/Yeti-Project/blob/master/script/monitor-external/README.txt 

    This script submits DNS packet via SSH. Note that it uses SSH public key authentication, so user should provide SSH public key via mail to the coordinators (note that currently DSA and RSA are OK, ECC will be supported later).

2. Joining Yeti as a Resolver Operator

    We encourage people running resolvers to join the project. These should be used for real-world queries, but for informed users in non-critical environments.

    To join the Yeti project as a resolver operator, you need to have a working DNS resolver with IPv6 support. You need to update your "hints" file to use the Yeti root servers instead of the IANA root servers. The current "hints" file can be found here: 

    https://raw.githubusercontent.com/BII-Lab/Yeti-Project/master/domain/named.cache  

    And the DNSSEC key is:

    https://raw.githubusercontent.com/BII-Lab/Yeti-Project/master/domain/KSK.pub 

    *Warning*: the DNSSEC key of the Yeti root (the KSK) changes *often* (typically every three months). You must therefore configure your resolver to use RFC 5011 automatic update *or* be ready to make many changes manually.
    
    In the purpose of some experiment, we need information and feedback from client side, so we encourage resolver operators to register their mail address for technical assistance, Yeti testbed changes or experiments coordination. If you setup your recursive server linked with Yeti root server, please contact <coordinators@lists.yeti-dns.org>.

    Configuration of the resolver:

    Unbound:

    ```
    server:
        root-hints: "yeti-hints"
        # Check the file is writable by Unbound
        auto-trust-anchor-file: autokey/yeti-key.key
    ```

    BIND:

        zone "." {
           type hint;
           file "/etc/bind/yeti-hints";
        };

        managed-keys {
           "." initial-key 257 3 8 "AwEAAdZZqL65TA/kHkLq1+ON5eQYm9PUBgV5UQbPcQtRAXbad1l6m6R0 iJIg46IiyFyUkEh+H7Z9/oPNnkM9zub2TjFiNVZUSnpyWtPqVD5nHrhU OdS3yW/AXpZuNJ3zX9XDXUpiEnfTPOMrUiZppP1fqx/jnAC9YDLs4K26 ocoDyQp+umu+eOrP/TOacRag+9r9NiQzsVuXHQnCwpPY4NwlA7QRaOOj BiI9tNEDD2khVE7Yy5c/sZYirlTOTEBbXkd9l9WVqRgEO+ikb8GMg7hg Oddvqj7ItBZvBUACQc3c0OqaLnEZx6CwIQpjxpPPYdyiEdKSwHGH3V3T fS+AEQlW8uk=";
        };

    In the BIND example, the text between quotes is the key, from https://raw.githubusercontent.com/BII-Lab/Yeti-Project/master/domain/KSK.pub

    Knot:

    ```
    -- -*- mode: lua -*-
    -- Knot uses a specific format for the hints so we cannot use the official hints file.
    modules = {
       'hints' -- Add other modules, if necessary
    }
    hints.root({                                   
          ['bii.dns-lab.net.'] = '240c:f:1:22::6',
          ['yeti-ns.tisf.net.'] = '2001:4f8:3:1006::1:4',
          ['yeti-ns.wide.ad.jp.'] = '2001:200:1d9::35',
          ['yeti-ns.as59715.net.'] = '2a02:cdc5:9715:0:185:5:203:53',
          ['dahu1.yeti.eu.org.'] = '2001:4b98:dc2:45:216:3eff:fe4b:8c5b',
          ['ns-yeti.bondis.org.'] = '2a02:2810:0:405::250',
          ['yeti-ns.ix.ru.'] = '2001:6d0:6d06::53',
          ['yeti.bofh.priv.at.'] = '2a01:4f8:161:6106:1::10',
          ['yeti.ipv6.ernet.in.'] = '2001:e30:1c1e:1::333',
          ['yeti.aquaray.com.'] = '2a02:ec0:200::1',
          ['dahu2.yeti.eu.org.'] = '2001:67c:217c:6::2',
          ['yeti-ns.switch.ch.'] = '2001:620:0:ff::29',
          ['yeti-ns.lab.nic.cl.'] = '2001:1398:1:21::8001',
          ['yeti-ns1.dns-lab.net.'] = '2001:da8:a3:a027::6',
          ['yeti-ns3.dns-lab.net.'] = '2400:a980:30ff::6',
          ['3f79bb7b435b05321651daefd374cd.yeti-dns.net.'] = '2401:c900:1401:3b:c::6',
          ['ca978112ca1bbdcafac231b39a23dc.yeti-dns.net.'] = '2c0f:f530::6',
          ['yeti1.ipv6.ernet.in.'] = '2001:e30:187d::333',
          ['xn--r2bi1c.xn--h2bv6c0a.xn--h2brj9c.'] = '2001:e30:1c1e:10::333',
          ['yeti.mind-dns.nl.'] = '2a02:990:100:b01::53:0',
	      ['yeti-ns.datev.net.'] = '2a00:e50:f15c:1000::1:53',
	      ['yeti.jhcloos.net.'] = '2001:19f0:5401:1c3::53'
    })
    
    trust_anchors.config('yeti-root.key')
    ```

    yeti-root.key is the official root key file, from https://raw.githubusercontent.com/BII-Lab/Yeti-Project/master/domain/KSK.pub

    TODO: The above should work with RFC 5011 but let's test

3. Joining Yeti as a Researcher

    Researchers are encouraged to join the Yeti discussion list:

    http://lists.yeti-dns.org/mailman/listinfo/discuss 

    Potential experiments or analysis can be discussed there.

    Confidential inquiries can be sent to <coordinators@lists.yeti-dns.org>.

## FAQ

Q：The requirement for the machine (apparently, a VPS could be enough?)

A: A VPS is OK. The experiments we expected so far (described in the webpage) are not strongly related to computing and mem capacity of server, but networking connectivity matters.
 
Q: How about its connectivity?

A: IPv6 networking environment is important to our experiment especially IPv6 DNS MTU problem study. We need native, non-tunneled IPv6 connectivity, either as a customer of a national or international backbone network, or as a multi-homed network who peers aggressively.
 
Q: Human resources commitment for root server operator. 

A: Well, this is a good question. Before we announced the project, three Initiators (WIDE, TISF and BII) had a basic consensus on the commitment to this scientific project. Because it is not for production network, so we do not expect any urgent configuration changes. However the server cannot be fully on auto-pilot, due to experiments that will affect the distribution master servers and may require changes to the authority servers. Therefore, we expect authority operators to offer 24-hours response time to questions and outage notifications, and 72-hours response time for planned configuration changes. We are non-profit, no exchange of money, only for public and technical interest. So we would like to invite qualified contributors who are willing to share the same interest and commitment with us.

Q: Is there Data usage policy in Yeti project?

A: Please turn to the [data sharing document](doc/DataSharingDeclaration.md) of Yeti DNS project. Basically, the Yeti project is not for production network and the Yeti data is for scientific usage which means the data with personal information or with privacy concern is not expected to join Yeti experiment. On the other hand, every participant who wants to get access to the data should make a public statement of using Yeti data to protect privacy, and do not publish the raw data.
