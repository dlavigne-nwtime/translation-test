---
title: "Autokey Protocol"
type: archives
toc_hide: true
---

![gif](/archives/pic/alautun4a.gif)

[Maya glyph _alautun_](/reflib/maya)

* * *

#### Table of Contents

*  [Briefing Slides](/reflib/proto/#briefing-slides)
*  [Related Pages](/reflib/proto/#related-pages)
*  [Introduction](/reflib/proto/#introduction)
*  [Certificate Trails](/reflib/proto/#certificate-trails)
*  [Secure Groups](/reflib/proto/#secure-groups)
*  [Identity Schemes](/reflib/proto/#identity-schemes)
*  [Selected Publications](/reflib/proto/#selected-publications)

* * *

#### Briefing Slides

*   NTP Security Model [PostScript](/reflib/brief/autokey/autokey.ps) | [PowerPoint](/reflib/brief/autokey/autokey.ppt) | [PDF](/reflib/brief/autokey/autokey.pdf)
*   NTP Security Algorithms [PostScript](/reflib/brief/secalgor/secalgor.ps) | [PowerPoint](/reflib/brief/secalgor/secalgor.ppt) | [PDF](/reflib/brief/secalgor/secalgor.pdf)
*   NTP Security Protocol [PostScript](/reflib/brief/secproto/secproto.ps) | [PowerPoint](/reflib/brief/secproto/secproto.ppt) | [PDF](/reflib/brief/secproto/secproto.pdf)

* * *

#### Related Pages

*   [Autonomous Configuration](autocfg.html)
*   [Autonomous Authentication](autokey.html)
*   [Autokey Identity Schemes](ident.html)

* * *

#### Introduction

The Autokey protocol is based on the public key infrastructure (PKI) algorithms of the OpenSSL library, which includes an assortment of message digest, digital signature and encryption schemes. As in NTPv3, NTPv4 supports symmetric key cryptography using keyed MD5 message digests to detect message modification and sequence numbers (actually timestamps) to avoid replay. In addition, NTPv4 supports timestamped digital signatures and X.509 certificates to verify the source as per common industry practices. It also suppoorts several optional identity schemes based on cryptographic challenge-response algorithms.

What makes Autokey special is the way in which these algorithms are used to deflect intruder attacks while maintaining the integrity and accuracy of the time synchronization function. The detailed design is complicated by the need to provisionally authenticate under conditions when reliable time values have not yet been acquired. Only when the server identities have been confirmed, signatures verified and accurate time values obtained does the Autokey protocol declare success.

The NTP message format has been augmented to include one or more extension fields between the original NTP header and the message authenticator code (MAC). The Autokey protocol exchanges cryptographic values in a manner designed to resist clogging and replay attacks. It uses timestamped digital signatures to sign a session key and then a pseudo-random sequence to bind each session key to the preceding one and eventually to the signature. In this way the expensive signature computations are greatly reduced and removed from the critical code path for constructing accurate time values.

Each session key is hashed from the IPv4 or IPv6 source and destination addresses and key identifier, which are public values, and a cookie which can be a public value or hashed from a private value depending on the mode. The pseudo-random sequence is generated by repeated hashes of these values and saved in a key list. The server uses the key list in reverse order, so as a practical matter the next session key cannot be predicted from the previous one, but the client can verify it using the same hash as the server.

There are three Autokey protocol variants or dances in NTP, one for client/server mode, another for broadcast/multicast mode and a third for symmetric active/passive mode. The [Association Management](ntp/html/assoc.html) program documentation page provides additional details. For instance, in client/server mode the server keeps no state for each client, but uses a fast algorithm and a private value to regenerate the cookie upon arrival of a client message. A client sends its designated public key to the server, which generates the cookie and sends it to the client encrypted with this key. The client decrypts the cookie using its private key and generates the key list. Session keys from this list are used to generate message authentication codes (MAC) which are checked by the server for the request and by the client for the response. Operational details of this and the remaining modes are given in the Internet Draft cited at the end of this page.

* * *

#### Certificate Trails

A timestamped digital signature scheme provides secure server authentication, but it does not provide protection against masquerade, unless the server identity is verified by other means. The PKI security model assumes each client is able to verify the certificate trail to a trusted certificate authority (TA), where each ascendent server must prove identity to the immediately descendant client by independent means, such as a credit card number or PIN. While Autokey supports this model by default, in a hierarchical ad-hoc network, especially with server discovery schemes like Manycast, proving identity at each rest stop on the trail must be an intrinsic capability of Autokey itself.

Our model is that every member of a closed group, such as might be operated by a timestamping service, be in possession of a secret group key. This could take the form of a private certificate or one or another identification schemes described in the literature and below. Certificate trails and identification schemes are at the heart of the NTP security model preventing masquerade and middleman attacks. The Autokey protocol operates to hike the trails and run the identity schemes.

A NTP secure group consists of a number of hosts dynamically assembled as a forest with roots the trusted hosts (TH) at the lowest stratum of the group. The TH do not have to be, but ofter are, primary (stratum 1) servers. A TA, not necessarily a group host, generates private and public identiity values and deploys selected values to the group members using secure means.

![gif](/archives/pic/identa.gif)

In the above figure the Alice group consists of TH Alice, which is also the TA, and Carol. Dependent servers Brenda and Denise have configured Alice and Carol, respectively, as their time sources. Stratum 3 server Eileen has configured both Brenda and Denise as her time sources. The certificates are identified by the subject and signed by the issuer. Note that the group key has previously been generated by Alice and deployed by secure means to all group members.

The steps in hiking the certificate trails and verifying identity are as follows. Note the step number in the description matches the step number in the figure.

1.  At startup each host loads its self-signed certificate from a local file. By convention the lowest stratum server certificates are marked trusted in a X.509 extension field. As Alice and Carol have trusted certificates, they need do nothing further to validate the time. It could be that the TH depend on servers in other groups; this scenario is discussed later.
2.  Brenda, Denise and Eileen start with the Autokey Parameter Exchange, which establishes the server name, signature scheme and identity scheme for each configured server. They continue with the Certificate Exchange, which loads server certificates recursively until a self-signed trusted certificate is found. Brenda and Denise immediately find self-signed trusted certificates for Alice, but Eileen will loop because neither Brenda nor Denise have their own certificates signed by either Alice or Carol.
3.  Brenda and Denise continue with the Identity Exchange, which uses one of the identity schemes described below to verify each has the group key previously deployed by Alice. If this succeeds, each continues in step 4.
4.  Brenda and Denise present their certificates to Alice for signature. If this succeeds, either or both Brenda and Denise can now provide these signed certificates to Eileen, which may be looping in step 2\. When Eileen receives them, she can now follow the trail in either Brenda or Denise to the trusted certificates for Alice and Carol. Once this is done, Eileen can execute the Identity Exchange and Signature Exchange just as Brenda and Denise.

* * *

#### Secure Groups

The NTP security model is based on multiple overlapping security compartments or groups. The example above illustrates how groups can be used to construct closed compartments, depending on how the identity credentials are deployed. The rules can be summarized:

1.  Each host holds a private group key generated by a trusted authority (TA).
2.  A host is trusted if it operates at the lowest stratum in the group and has a trusted, self-signed certificate.
3.  A host uses the identity scheme to prove to another host it has the same group key.
4.  A client verifies group membership if the server has the same key and has an unbroken certificate trail to a trusted host.

Each compartment is assigned a group key by the TA, which is then deployed to all group members by secure means. For retrieval purposes the name of the group key file is the name of a TH.

For various reasons it may be convenient for a server to hold keys for more than one group. The figure below shows three secure groups Alice, Helen and Carol.

![gif](/archives/pic/identc.gif)

Hosts A, B, C and D belong to the Alice group, hosts R, S to the Helen group and hosts X, Y and Z to the Carol group. Hosts A, B, R and X hold trusted certificates, while the remaining hosts hold standard certificates. Hosts A, B, C, D and X hold the group key for Alice; hosts R, S and X hold the group key for Helen; hosts X, Y and Z hold the group key for Carol.

The name of the group key file in Carol is X, while the name of that file in Helen is R, since there is no ambiguity in server selection. Alice is a special case, as the name of the group key depends on which server is chosen by the selection algorithm. By convention, both A and B generate individual group keys and distribute them to all group hosts by secure means. Then, it doesn't matter whether the certificate trail lands on A or B. Note that only X needs the group keys for Alice and Helen; Carol and her dependents need only her group key.

In most identity schemes there are two kinds of group keys, server and client. The intent of the design is to provide security separation, so that servers cannot masquerade as TAs and clients cannot masquerade as servers. Assume for example that Alice and Helen belong to national standards laboratories and their group keys are used to confirm identity between members of each group. Carol is a prominent corporation receiving standards products via broadcast satellite and requiring cryptographic authentication.

Perhaps under contract, trusted host X belonging to the Carol group rents client keys for both Alice and Helen and has server keys for Carol. Hosts Y and Z have only client keys for Carol. The Autokey protocol operates as previously described for each group separately while preserving security separation. Host X can prove identity in Carol to clients Y and Z, but cannot prove to anybody that she can sign certificates for either Alice or Helen.

Ordinarily, it would not be desirable to reveal the group key in server keys and forbidden to reveal it in client keys. This can be avoided using the MV identity scheme described later. It allows the same broadcast transmission to be authenticated by more than one key, one used internally by the laboratories (Alice and/or Helen) and the other handed out to clients like Carol. In the MV scheme these keys can be separately activated upon subscription and deactivated if the subscriber fails to pay the bill.

![gif](/archives/pic/identb.gif)

The figure above shows operational details where more than one group is involved, in this case Carol and Alice. As in the previous example, Brenda has configured Alice as her time source and Denise has configured Carol as her time source. Alice and Carol have server keys; Brenda and Denise have server and client keys only for their respective groups. Eileen has client keys for both Alice and Carol. The protocol operates as previously described to verify Alice to Brenda and Carol to Denise.

The interesting case is Eileen, who may verify identity either via Brenda or Denise or both. To do that she uses the client keys of both Alice and Carol. But, Eileen doesn't know which of the two keys to use until hiking the cetificate trail to find the trusted certificate of either Alice or Carol and then loading the associated local key. This scenario can of course become even more complex as the number of servers and depth of the tree increase. The bottom line is that every host must have the client keys for all the lowest-stratum trusted hosts it is ever likely to find.

* * *

#### Identity Schemes

While the identity scheme described in RFC-2875 is based on a ubiquitous Diffie-Hellman infrastructure, it is expensive to generate and use when compared to others described here. There are five schemes now implemented in Autokey to prove identity: (1) private certificates (PC), (2) trusted certificates (TC), (3) a modified Schnorr algorithm (IFF aka Identify Friendly or Foe), (4) a modified Guillou-Quisquater algorithm (GQ), and (5) a modified Mu-Varadharajan algorithm (MV). The [Identity Schemes](ident.html) page contains a detailed description of each scheme.

When multiple identity schemes are supported in the Autokey protocol, the first message exchange determines which one is used, called the _cryptotype_. The client request message contains bits corresponding to which schemes it has available. The server response message contains bits corresponding to which schemes it has available. Both server and client match the received bits with their own and select a common scheme.

Following the principle that time is a public value, a server responds to any client packet that matches its cryptotype capabilities. Thus, a server receiving an unauthenticated packet will respond with an unauthenticated packet, while the same server receiving a packet of a cryptotype it supports will respond with packets of that cryptotype. However, unconfigured broadcast or manycast client associations or symmetric passive associations will not be mobilized unless the server supports a cryptotype compatible with the first packet received. By default, unauthenticated associations will not be mobilized unless overridden in a decidedly dangerous way.

Some examples may help to reduce confusion. Client Alice has no specific cryptotype selected. Server Bob has both a symmetric key file and minimal Autokey files. Alice's unauthenticated messages arrive at Bob, who replies with unauthenticated messages. Cathy has a copy of Bob's symmetric key file and has selected key ID 4 in messages to Bob. Bob verifies the message with his key ID 4\. If it's the same key and the message is verified, Bob sends Cathy a reply authenticated with that key. If verification fails, Bob sends Cathy a thing called a crypto-NAK, which tells her something broke. She can see the debris using the <tt>ntpq</tt> program.

Denise has rolled her own host key and certificate. She also uses one of the identity schemes as Bob. She sends the first Autokey message to Bob and they both dance the protocol authentication and identity steps. If all comes out okay, Denise and Bob continue as described above.

It should be clear from the above that Bob can support all the girls at the same time, as long as he has compatible authentication and identity credentials. Now, Bob can act just like the girls in his own choice of servers; he can run multiple configured associations with multiple different servers (or the same server, although that might not be useful). But, wise security policy might preclude some cryptotype combinations; for instance, running an identity scheme with one server and no authentication with another might not be wise.

* * *

#### Selected Publications

1.  Adams, C., S. Farrell. Internet X.509 public key infrastructure certificate management protocols. Network Working Group Request for Comments RFC-2510, Entrust Technologies, March 1999, 30 pp.
2.  Guillou, L.C., and J.-J. Quisquatar. A "paradoxical" identity-based signature scheme resulting from zero-knowledge. _Proc. CRYPTO 88 Advanced in Cryptology_, Springer-Verlag, 1990, 216-231.
3.  Housley, R., et al. Internet X.509 public key infrastructure certificate and certificate revocation list (CRL) profile. Network Working Group Request for Comments RFC-3280, RSA Laboratories, April 2002, 129 pp.
4.  Mills, D.L. The Autokey security architecture, protocol and algorithms. Electrical and Computer Engineering Technical Report 06-1-1, University of Delaware, January 2006, 59 pp, [PDF](/reflib/reports/stime1/stime.pdf).
5.  Mu, Y., and V. Varadharajan. Robust and secure broadcasting. Proc. INDOCRYPT 2001, LNCS 2247, Springer Verlag, 2001, 223-231.
6.  Prafullchandra, H., and J. Schaad. Diffie-Hellman proof-of-possession algorithms. Network Working Group Request for Comments RFC-2875, Critical Path, Inc., July 2000, 23 pp.
7.  Schnorr, C.P. Efficient signature generation for smart cards. _J. Cryptology 4, 3_ (1991), 161-174.
					