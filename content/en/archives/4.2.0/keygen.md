---
title: "ntp-keygen - generate public and private keys"
type: archives
---

![gif](/archives/pic/alice23.gif)[from _Alice's Adventures in Wonderland_, Lewis Carroll](/reflib/pictures)

Alice holds the key.

Last update: 03:13 AM UTC Monday, October 13, 2003

* * *

#### Table of Contents

*   [Synopsis](/archives/4.2.0/keygen/#synopsis)
*   [Description](/archives/4.2.0/keygen/#description)
*   [Running the program](/archives/4.2.0/keygen/#running-the-program)
*   [Trusted Hosts and Groups](/archives/4.2.0/keygen/#trusted-hosts-and-groups)
*   [Identity Schemes](/archives/4.2.0/keygen/#identity-schemes)
*   [Command Line Options](/archives/4.2.0/keygen/#command-line-options)
*   [Random Seed File](/archives/4.2.0/keygen/#random-seed-file)
*   [Cryptographic Data Files](/archives/4.2.0/keygen/#cryptographic-data-files)
*   [Bugs](/archives/4.2.0/keygen/#bugs)

* * *

#### Synopsis

<tt>ntp-keygen [ -deGgHIMnPT ] [ -c [ RSA-MD2 | RSA-MD5 | RSA-SHA | RSA-SHA1 | RSA-MDC2 | RSA-RIPEMD160 | DSA-SHA | DSA-SHA1 ] ] [-i _name_ ] [ -p _password_ ] [ -S [ RSA | DSA ] ] [ -s _name_ ] [ -v _nkeys_ ]</tt>

* * *

#### Description

This program generates cryptographic data files used by the NTPv4 authentication and identification schemes. It generates MD5 key files used in symmetric key cryptography. In addition, if the OpenSSL software library has been installed, it generates keys, certificate and identity files used in public key cryptography. These files are used for cookie encryption, digital signature and challenge/response identification algorithms compatible with the Internet standard security infrastructure.

All files are in PEM-encoded printable ASCII format, so they can be embedded as MIME attachments in mail to other sites and certificate authorities. By default, files are not encrypted. The <tt>-p password</tt> option specifies the write password and <tt>-q password</tt> option the read password for previously encrypted files. The <tt>ntp-keygen</tt> program prompts for the password if it reads an encrypted file and the password is missing or incorrect. If an encrypted file is read successfully and no write password is specified, the read password is used as the write password by default.

The <tt>ntpd</tt> configuration command <tt>crypto pw _password_</tt> specifies the read password for previously encrypted files. The daemon expires on the spot if the password is missing or incorrect. For convenience, if a file has been previously encrypted, the default read password is the name of the host running the program. If the previous write password is specified as the host name, these files can be read by that host with no explicit password.

File names begin with the prefix <tt>ntpkey_</tt> and end with the postfix <tt>_hostname.filestamp</tt>, where <tt>hostname</tt> is the owner name, usually the string returned by the Unix <tt>gethostname()</tt> routine, and <tt>filestamp</tt> is the NTP seconds when the file was generated, in decimal digits. This both guarantees uniqueness and simplifies maintenance procedures, since all files can be quickly removed by a <tt>rm ntpkey*</tt> command or all files generated at a specific time can be removed by a <tt>rm *filestamp</tt> command. To further reduce the risk of misconfiguration, the first two lines of a file contain the file name and generation date and time as comments.

All files are installed by default in the keys directory <tt>/usr/local/etc</tt>, which is normally in a shared filesystem in NFS-mounted networks. The actual location of the keys directory and each file can be overridden by configuration commands, but this is not recommended. Normally, the files for each host are generated by that host and used only by that host, although exceptions exist as noted later on this page.

Normally, files containing private values, including the host key, sign key and identification parameters, are permitted root read/write-only; while others containing public values are permitted world readable. Alternatively, files containing private values can be encrypted and these files permitted world readable, which simplifies maintenance in shared file systems. Since uniqueness is insured by the hostname and file name extensions, the files for a NFS server and dependent clients can all be installed in the same shared directory.

The recommended practice is to keep the file name extensions when installing a file and to install a soft link from the generic names specified elsewhere on this page to the generated files. This allows new file generations to be activated simply by changing the link. If a link is present, <tt>ntpd</tt> follows it to the file name to extract the filestamp. If a link is not present, <tt>ntpd</tt> extracts the filestamp from the file itself. This allows clients to verify that the file and generation times are always current. The <tt>ntp-keygen</tt> program uses the same extension for all files generated at one time, so each generation is distinct and can be readily recognized in monitoring data.

#### Running the program

The safest way to run the <tt>ntp-keygen</tt> program is logged in directly as root. The recommended procedure is change to the keys directory, usually <tt>/ust/local/etc</tt>, then run the program. When run for the first time, or if all <tt>ntpkey</tt> files have been removed, the program generates a RSA host key file and matching RSA-MD5 certificate file, which is all that is necessary in many cases. The program also generates soft links from the generic names to the respective files. If run again, the program uses the same host key file, but generates a new certificate file and link.

The host key is used to encrypt the cookie when required and so must be RSA type. By default, the host key is also the sign key used to encrypt signatures. When necessary, a different sign key can be specified and this can be either RSA or DSA type. By default, the message digest type is MD5, but any combination of sign key type and message digest type supported by the OpenSSL library can be specified, including those using the MD2, MD5, SHA, SHA1, MDC2 and RIPE160 message digest algorithms. However, the scheme specified in the certificate must be compatible with the sign key. Certificates using any digest algorithm are compatible with RSA sign keys; however, only SHA and SHA1 certificates are compatible with DSA sign keys.

Private/public key files and certificates are compatible with other OpenSSL applications and very likely other libraries as well. Certificates or certificate requests derived from them should be compatible with extant industry practice, although some users might find the interpretation of X509v3 extension fields somewhat liberal. However, the identification parameter files, although encoded as the other files, are probably not compatible with anything other than Autokey.

Running the program as other than root and using the Unix <tt>su</tt> command to assume root may not work properly, since by default the OpenSSL library looks for the random seed file <tt>.rnd</tt> in the user home directory. However, there should be only one <tt>.rnd</tt>, most conveniently in the root directory, so it is convenient to define the <tt>$RANDFILE</tt> environment variable used by the OpenSSL library as the path to <tt>/.rnd</tt>.

Installing the keys as root might not work in NFS-mounted shared file systems, as NFS clients may not be able to write to the shared keys directory, even as root. In this case, NFS clients can specify the files in another directory such as <tt>/etc</tt> using the <tt>keysdir</tt> command. There is no need for one client to read the keys and certificates of other clients or servers, as these data are obtained automatically by the Autokey protocol.

Ordinarily, cryptographic files are generated by the host that uses them, but it is possible for a trusted agent (TA) to generate these files for other hosts; however, in such cases files should always be encrypted. The subject name and trusted name default to the hostname of the host generating the files, but can be changed by command line options. It is convenient to designate the owner name and trusted name as the subject and issuer fields, respectively, of the certificate. The owner name is also used for the host and sign key files, while the trusted name is used for the identity files.

#### Trusted Hosts and Groups

Each cryptographic configuration involves selection of a signature scheme and identification scheme, called a cryptotype, as explained in the [Authentication Options](/archives/4.2.0/authopt) page. The default cryptotype uses RSA encryption, MD5 message digest and TC identification. First, configure a NTP subnet including one or more low-stratum trusted hosts from which all other hosts derive synchronization directly or indirectly. Trusted hosts have trusted certificates; all other hosts have nontrusted certificates. These hosts will automatically and dynamically build authoritative certificate trails to one or more trusted hosts. A trusted group is the set of all hosts that have, directly or indirectly, a certificate trail ending at a trusted host. The trail is defined by static configuration file entries or dynamic means described on the [Automatic NTP Configuration Options](/archives/4.2.0/manyopt) page.

On each trusted host as root, change to the keys directory. To insure a fresh fileset, remove all <tt>ntpkey</tt> files. Then run <tt>ntp-keygen -T</tt> to generate keys and a trusted certificate. On all other hosts do the same, but leave off the <tt>-T</tt> flag to generate keys and nontrusted certificates. When complete, start the NTP daemons beginning at the lowest stratum and working up the tree. It may take some time for Autokey to instantiate the certificate trails throughout the subnet, but setting up the environment is completely automatic.

If it is necessary to use a different sign key or different digest/signature scheme than the default, run <tt>ntp-keygen</tt> with the <tt>-S</tt>_ <tt>type</tt>_ option, where _<tt>type</tt>_ is either <tt>RSA</tt> or <tt>DSA</tt>. The most often need to do this is when a DSA-signed certificate is used. If it is necessary to use a different certificate scheme than the default, run <tt>ntp-keygen</tt> with the <tt>-c _scheme_</tt> option and selected _<tt>scheme</tt>_ as needed. If <tt>ntp-keygen</tt> is run again without these options, it generates a new certificate using the same scheme and sign key.

After setting up the environment it is advisable to update certificates from time to time, if only to extend the validity interval. Simply run <tt>ntp-keygen</tt> with the same flags as before to generate new certificates using existing keys. However, if the host or sign key is changed, <tt>ntpd</tt> should be restarted. When ntpd is restarted, it loads any new files and restarts the protocol. Other dependent hosts will continue as usual until signatures are refreshed, at which time the protocol is restarted.

#### Identity Schemes

As mentioned on the Autonomous Authentication page, the default TC identity scheme is vulnerable to a middleman attack. However, there are more secure identity schemes available, including PC, IFF, GQ and MV described on the [Identification Schemes](/reflib/ident) page. These schemes are based on a TA, one or more trusted hosts and some number of nontrusted hosts. Trusted hosts prove identity using values provided by the TA, while the remaining hosts prove identity using values provided by a trusted host and certificate trails that end on that host. The name of a trusted host is also the name of its sugroup and also the subject and issuer name on its trusted certificate. The TA is not necessarily a trusted host in this sense, but often is.

In some schemes there are separate keys for servers and clients. A server can also be a client of another server, but a client can never be a server for another client. In general, trusted hosts and nontrusted hosts that operate as both server and client have parameter files that contain both server and client keys. Hosts that operate only as clients have key files that contain only client keys.

The PC scheme supports only one trusted host in the group. On trusted host _alice_ run <tt>ntp-keygen -P -p _password_</tt> to generate the host key file <tt>ntpkey_RSAkey__alice.filestamp_</tt> and trusted private certificate file <tt>ntpkey_RSA-MD5_cert__alice.filestamp_</tt>. Copy both files to all group hosts; they replace the files which would be generated in other schemes. On each host _bob_ install a soft link from the generic name <tt>ntpkey_host__bob_</tt> to the host key file and soft link <tt>ntpkey_cert__bob_</tt> to the private certificate file. Note the generic links are on _bob_, but point to files generated by trusted host _alice_. In this scheme it is not possible to refresh either the keys or certificates without copying them to all other hosts in the group.

For the IFF scheme proceed as in the TC scheme to generate keys and certificates for all group hosts, then for every trusted host in the group, generate the IFF parameter file. On trusted host _alice_ run <tt>ntp-keygen -T</tt> <tt>-I -p _password_</tt> to produce her parameter file <tt>ntpkey_IFFpar__alice.filestamp_</tt>, which includes both server and client keys. Copy this file to all group hosts that operate as both servers and clients and install a soft link from the generic <tt>ntpkey_iff__alice_</tt> to this file. If there are no hosts restricted to operate only as clients, there is nothing further to do. As the IFF scheme is independent of keys and certificates, these files can be refreshed as needed.

If a rogue client has the parameter file, it could masquerade as a legitimate server and present a middleman threat. To eliminate this threat, the client keys can be extracted from the parameter file and distributed to all restricted clients. After generating the parameter file, on _alice_ run <tt>ntp-keygen</tt> <tt>-e</tt> and pipe the output to a file or mail program. Copy or mail this file to all restricted clients. On these clients install a soft link from the generic <tt>ntpkey_iff__alice_</tt> to this file. To further protect the integrity of the keys, each file can be encrypted with a secret password.

For the GQ scheme proceed as in the TC scheme to generate keys and certificates for all group hosts, then for every trusted host in the group, generate the IFF parameter file. On trusted host _alice_ run <tt>ntp-keygen -T</tt> <tt>-G -p _password_</tt> to produce her parameter file <tt>ntpkey_GQpar__alice.filestamp_</tt>, which includes both server and client keys. Copy this file to all group hosts and install a soft link from the generic <tt>ntpkey_gq__alice_</tt> to this file. In addition, on each host _bob_ install a soft link from generic <tt>ntpkey_gq__bob_</tt> to this file. As the GQ scheme updates the GQ parameters file and certificate at the same time, keys and certificates can be regenerated as needed.

For the MV scheme, proceed as in the TC scheme to generate keys and certificates for all group hosts. For illustration assume _trish_ is the TA, _alice_ one of several trusted hosts and _bob_ one of her clients. On TA _trish_ run <tt>ntp-keygen</tt> <tt>-V _n_ -p _password_</tt>, where _n_ is the number of revokable keys (typically 5) to produce the parameter file <tt>ntpkeys_MVpar__trish.filestamp_ </tt>and client key files <tt>ntpkeys_MVkey_d___trish.filestamp_</tt> where _<tt>d</tt>_ is the key number (0 < _<tt>d</tt>_ < _n_). Copy the parameter file to _alice_ and install a soft link from the generic <tt>ntpkey_mv__alice_</tt> to this file. Copy one of the client key files to _alice_ for later distribution to her clients. It doesn't matter which client key file goes to _alice_, since they all work the same way. _Alice_ copies the client key file to all of her clients. On client _bob_ install a soft link from generic <tt>ntpkey_mvkey__bob_ </tt>to the client key file. As the MV scheme is independent of keys and certificates, these files can be refreshed as needed.

#### Command Line Options

<dt><tt>-c [ RSA-MD2 | RSA-MD5 | RSA-SHA | RSA-SHA1 | RSA-MDC2 | RSA-RIPEMD160 | DSA-SHA | DSA-SHA1 ]</tt></dt>

Select certificate message digest/signature encryption scheme. Note that RSA schemes must be used with a RSA sign key and DSA schemes must be used with a DSA sign key. The default without this option is <tt>RSA-MD5</tt>.

<dt><tt>-d</tt></dt>

Enable debugging. This option displays the cryptographic data produced in eye-friendly billboards.

<dt><tt>-e</tt></dt>

Write the IFF client keys to the standard output. This is intended for automatic key distribution by mail.

<dt><tt>-G</tt></dt>

Generate parameters and keys for the GQ identification scheme, obsoleting any that may exist.

<dt><tt>-g</tt></dt>

Generate keys for the GQ identification scheme using the existing GQ parameters. If the GQ parameters do not yet exist, create them first.

<dt><tt>-H</tt></dt>

Generate new host keys, obsoleting any that may exist.

<dt><tt>-I</tt></dt>

Generate parameters for the IFF identification scheme, obsoleting any that may exist.

<dt><tt>-i _name_</tt></dt>

Set the subject name to _name_. This is used as the subject field in certificates and in the file name for host and sign keys.

<dt><tt>-M</tt></dt>

Generate MD5 keys, obsoleting any that may exist.

<dt><tt>-P</tt></dt>

Generate a private certificate. By default, the program generates public certificates.

<dt><tt>-p _password_</tt></dt>

Encrypt generated files containing private data with <tt>_password_</tt> and the DES-CBC algorithm.

<dt><tt>-q</tt></dt>

Set the password for reading files to <tt>_password_</tt>.

<dt><tt>-S [ RSA | DSA ]</tt></dt>

Generate a new sign key of the designated type, obsoleting any that may exist. By default, the program uses the host key as the sign key.

<dt><tt>-s _name_</tt></dt>

Set the issuer name to _name_. This is used for the issuer field in certificates and in the file name for identity files.

<dt><tt>-T</tt></dt>

Generate a trusted certificate. By default, the program generates a non-trusted certificate.

<dt><tt>-V _nkeys_</tt></dt>

Generate parameters and keys for the Mu-Varadharajan (MV) identification scheme.

#### Random Seed File

All cryptographically sound key generation schemes must have means to randomize the entropy seed used to initialize the internal pseudo-random number generator used by the library routines. The OpenSSL library uses a designated random seed file for this purpose. The file must be available when starting the NTP daemon and <tt>ntp-keygen</tt> program. If a site supports OpenSSL or its companion OpenSSH, it is very likely that means to do this are already available.

It is important to understand that entropy must be evolved for each generation, for otherwise the random number sequence would be predictable. Various means dependent on external events, such as keystroke intervals, can be used to do this and some systems have built-in entropy sources. Suitable means are described in the OpenSSL software documentation, but are outside the scope of this page.

The entropy seed used by the OpenSSL library is contained in a file, usually called <tt>.rnd</tt>, which must be available when starting the NTP daemon or the <tt>ntp-keygen</tt> program. The NTP daemon will first look for the file using the path specified by the <tt>randfile</tt> subcommand of the <tt>crypto</tt> configuration command. If not specified in this way, or when starting the <tt>ntp-keygen</tt> program, the OpenSSL library will look for the file using the path specified by the <tt>RANDFILE</tt> environment variable in the user home directory, whether root or some other user. If the <tt>RANDFILE</tt> environment variable is not present, the library will look for the <tt>.rnd</tt> file in the user home directory. If the file is not available or cannot be written, the daemon exits with a message to the system log and the program exits with a suitable error message.

#### Cryptographic Data Files

All other file formats begin with two lines. The first contains the file name, including the generated host name and filestamp. The second contains the datestamp in conventional Unix <tt>date</tt> format. Lines beginning with <tt>#</tt> are considered comments and ignored by the _<tt>ntp-keygen</tt> _program and <tt>ntpd</tt> daemon. Cryptographic values are encoded first using ASN.1 rules, then encrypted if necessary, and finally written PEM-encoded printable ASCII format preceded and followed by MIME content identifier lines.

The format of the symmetric keys file is somewhat different than the other files in the interest of backward compatibility. Since DES-CBC is deprecated in NTPv4, the only key format of interest is MD5 alphanumeric strings. The keys are entered one per line in the format

`keyno type key`

where _<tt>keyno</tt>_ is a positive integer in the range 1-65,535, _<tt>type</tt>_ is the string <tt>MD5</tt> defining the key format and _<tt>key</tt>_ is the key itself, which is a printable ASCII string 16 characters or less in length. Each character is chosen from the 93 printable characters in the range 0x21 through 0x7f excluding space and the `#` character.

Note that the keys used by the <tt>ntpq</tt> and <tt>ntpdc</tt> programs are checked against passwords requested by the programs and entered by hand, so it is generally appropriate to specify these keys in human readable ASCII format.

The <tt>ntp-keygen</tt> program generates a MD5 symmetric keys file <tt>ntpkey_MD5key__hostname.filestamp_</tt>. Since the file contains private shared keys, it should be visible only to root and distributed by secure means to other subnet hosts. The NTP daemon loads the file <tt>ntp.keys</tt>, so <tt>ntp-keygen</tt> installs a soft link from this name to the generated file. Subsequently, similar soft links must be installed by manual or automated means on the other subnet hosts. While this file is not used with the Autokey Version 2 protocol, it is needed to authenticate some remote configuration commands used by the [<tt>ntpq</tt>](/archives/4.2.0/ntpq) and [<tt>ntpdc</tt>](/archives/4.2.0/ntpdc) utilities.

#### Bugs

It can take quite a while to generate some cryptographic values, from one to several minutes with modern architectures such as UltraSPARC and up to tens of minutes to an hour with older architectures such as SPARC IPC.