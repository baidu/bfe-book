# HTTPS Optimization

## Background And Necessity of HTTPS

With the rapid development of the Internet and the rapid growth of the user scale, the Internet has been faced with serious security threats and data privacy risks for a long time. The illegal industry targeting HTTP traffic, while making illegal profits, has a serious impact on the Internet user experience and security privacy, and has also brought huge losses to the reputation and interests of Internet service providers. Typical problems with using the HTTP protocol include:

- **Content tampering**: The content of the page visited by the user is maliciously tampered with, for example, the link of the search results page is maliciously modified, the installation package of the download page is maliciously replaced, and the browse page is implanted with a large number of advertisements

- **Privacy disclosure**: user's network activity is sniffed, personal data is leaked and used, and user is interfered by spam advertising or fraudulent phone calls

- **Traffic hijacking**: user access is hijacked to phishing websites, user account information is stolen under guidance, etc

Compared with HTTP protocol, HTTPS uses TLS protocol for transmission at the bottom layer, providing integrity, privacy and identity authentication mechanism, which can ensure the access security of Internet user traffic. Many large Internet companies at home and abroad have fully supported HTTPS. Browser manufacturers, mobile application stores and other ecological manufacturers are also accelerating the promotion of HTTPS. For example:

- The Google Chrome browser adds a "unsafe" prompt in front of the HTTP domain name input box. Google is pushing Chrome browsers to use HTTPS as the default setting (not HTTP). After the user directly enters the domain name, Chrome will first try to access it using HTTPS protocol.

- Apple iOS10 ATS (App Transport Security) policy requires that all apps on the iOS App Store must support HTTPS.

## Challenges of HTTPS

For a medium and large website, full use of HTTPS not only needs to complete the HTTPS transformation of website pages, but also faces the following important problems:

- **Access speed problem**: HTTPS generally introduces 1 to 2 additional RTTs compared with HTTP. Of course, this does not include the delay of users accessing HTTP first and then being redirected to HTTPS in some cases, and the delay introduced by HTTPS certificate status check. In the mobile network environment, the round-trip delay is often greater, and will also bring more obvious impact.

- **Performance and cost issues**: The cryptographic computation introduced by protocol handshake and data encryption transmission brings performance overhead that cannot be ignored. In particular, asymmetric cryptography computation during TLS handshake is the main source of performance overhead. Take Nginx as an example. In the case of short connection and full handshake, the throughput of HTTP is up to 10 times higher than that of HTTPS.

- **Security issues**: To properly deploy and ensure HTTPS security, you need to master certain knowledge and best practices in the security field. Operation and maintenance personnel are often prone to leave security risks during HTTPS deployment, and cannot meet the security compliance standards required by business operations.

- **Availability problem**: The HTTPS ecosystem is more complex, and third-party CA has become a new dependency on website stability. At the same time, due to the diversity of clients and the compatibility and defects of the end protocol implementation, service access exceptions may also be caused.

## Optimization Mechanism in HTTPS

Common HTTPS optimization methods are briefly described below.

### Access Delay Optimization

The total delay introduced by HTTPS depends on the round-trip time and the number of round-trip interactions. From the perspective of user access process, HTTPS interaction delay includes not only the delay of establishing a secure session, but also the delay of redirecting from HTTP to HTTPS and the delay of HTTPS certificate status check. Accordingly, we can optimize access delay through the following methods.

| Optimization Method               | Detailed Mechanism                                           |
| --------------------------------- | ------------------------------------------------------------ |
| Reduce round-trip delay           | Complete TLS handshake at the edge node.                     |
| Reduce the number of interactions | The number of interactions could be reduced by using TLS session multiplexing. TLS 1.2 requires 2 RTTs (including TCP handshake), TLS 1.3 only requires 1 RTT (including TCP handshake), and QUIC only requires 0 RTT. In addition, you can optimize the delay of redirecting from HTTP to HTTPS through HSTS or caching redirect; Certificate status check delay could be optimized  through OCSP Stapling. |
| Hide interaction delay            | The influence of connection delay cloud be shielded by heuristic pre-establishment of connection. |

### Performance Optimization of Asymmetric Cryptography

In the communication process of HTTPS server, asymmetric cryptography computation is the main source of performance overhead. The further improvement of the key length of asymmetric cryptography algorithms will also aggravate this problem. For example, since 2010, mainstream CAs have stopped issuing insecure 1024 bits length RSA certificates and used 2048 bits length RSA certificates. There are many ways to optimize the performance of asymmetric cryptography computing.

| Optimization Method                                          | Detailed Mechanism                                           |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Reduce the number of asymmetric cryptography calculations    | Increase the connection reuse rate and session reuse rate to reduce the number of TLS complete handshakes and asymmetric cryptography  calculations |
| Optimize performance of asymmetric cryptography calculations | Improve the performance of asymmetric cryptography algorithms through hardware accelerator cards or CPUs that support cryptographic computing instructions |
| Prefer higher performance algorithms                         | Adaptive priority is given to ECC certificates rather than RSA certificates (for scenarios without hardware acceleration) |

### Safety Assessment And Patrol Inspection

Security risks of HTTPS come from CA infrastructure, protocol and algorithm vulnerabilities, HTTPS deployment configuration, HTTP applications, etc. Some public services support automatic evaluation of the security of sites applying HTTPS and prompt potential security vulnerabilities.

In addition, service providers applying HTTPS need to develop and apply HTTPS deployment best practices and specifications. For example, ["SSL and TLS Deployment Best Practices"](https://github.com/ssllabs/research/wiki/SSL-and-TLS-Deployment-Best-Practices) written by SSLLabs,  ["Securing Web Transactions: TLS Server Certificate Management"](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.1800-16.pdf) written by NIST, etc.

At the same time, it is necessary to establish a monitoring mechanism for HTTPS. Including HTTPS certificate monitoring, HTTPS mixed content monitoring, security cookie monitoring, HTTPS security vulnerability scanning, etc.

### Response to Stability Risk

HTTPS services could be built with low risk of change, fast stop loss, and more stable and reliable through Canary Release mechanism and redundancy mechanism. See the following for details of relevant mechanisms (controlling the redirect from HTTP to HTTPS with Canary Release, and the update of HTTPS certificates with Canary Release).


## Relevant Enhancement Mechanism of BFE

In order to better adapt to the needs of large sites, BFE has made some targeted improvements compared with common open source solutions. The following is a brief introduction of some differentiation mechanisms.


### Distributed TLS Session Cache

BFE supports storing TLS session state in a distributed cache cluster. After the client connects to any instance in the BFE cluster, the session reuse handshake can be completed based on the session ID, thus improving the TLS session reuse rate in the cluster deployment mode.

### TLS Session State Format

In the distributed TLS session cache above, the stored session state can be in two formats: original format and OpenSSL format. If you are a new user of BFE, you can give priority to the original format, which is more compact and can reduce the storage cost of distributed cache; If you need to replace the OpenSSL-based reverse proxy, you can use the OpenSSL format, so that the two programs can read the session state saved by each other, and achieve smooth migration without affecting the session reuse handshake.

### Session Ticket Key Update

If the Session Ticket key is leaked, forward security cannot be guaranteed. For the connection based on Session Ticket session reuse, if a malicious attacker records the traffic in advance and then uses the leaked Session Ticket key, it can successfully calculate the session key and decrypt and restore the plaintext information.

To avoid the above problems, the Session Ticket key file should be updated regularly. BFE supports hot loading and updating the Session Ticket key, and does not require process restart to cause long connection interruption.

### Adaptive Fault Tolerance of OCSP Staple

The OCSP Staple file has a certain period of validity. If the server accidentally sends an expired OCSP Staple file during the handshake, the handshake may fail. In a large-scale distributed cluster environment, it is not uncommon that OCSP Staple is not updated successfully or missed due to process or mechanism reasons.

BFE supports adaptive processing of OCSP Staple files that are about to expire. When OCSP Staple is about to expire, BFE will automatically downgrade and stop sending OCSP Staple files to avoid the impact of potential handshake failure.


### Optimization of Asymmetric Cryptography Computation

Asymmetric key algorithm is generally used for identity authentication and key exchange during TLS handshake. At present, the mainstream asymmetric algorithms include RSA and ECC. Because of the compatibility of CA root certificates, RSA certificates are still the most widely used certificates at present. At present, the recommended key length of RSA algorithm is 2048 bits. Compared with the equivalent security strength of 256-bit ECC algorithm, the performance cost of RSA algorithm is far greater than ECC.

If all user traffic comes from its own mobile terminal, or the handshake message meets specific characteristics, the performance overhead of asymmetric cryptography calculation can be reduced by selecting ECC certificate and private key.

If a large amount of user traffic comes from low-end clients, RSA certificates and private keys are required to ensure compatibility. By using a hardware accelerator card that supports RSA algorithm, the performance overhead introduced by RSA computing can be significantly reduced.

There are two types of schemes based on hardware acceleration:

- **Same machine mode**: deploy CPU supporting RSA acceleration or deploy RSA hardware acceleration card on the same machine

- **Remote mode**: Remote access to asymmetric cryptography computing services with RSA hardware acceleration 

The advantages of "remote mode" over the "same machine mode" are:

- It is not necessary to fully upgrade the existing forwarding machine, which can reduce the upgrade cycle and cost

- It can flexibly adapt the computing requirements of BFE forwarding machine to avoid the waste of hardware accelerator card resources (note: for scenarios based on hardware accelerator card)

- It can provide centralized, hardware-based security protection mechanism for keys (note: for scenarios based on CPU special instructions)

The availability of the remote hardware acceleration service is difficult to reach 100%. In order to avoid introducing reliance on the remote hardware acceleration service to reduce the overall availability of BFE, when there is an occasional exception in accessing the remote hardware acceleration card, the impact could be eliminated by automatically degrading to using local computing.


### Multiple Certificates Selection

In a multi-tenant environment, different certificates need to be used for different tenants. The same tenant may also use multiple certificates for business reasons (such as brand considerations).

In actual deployment, BFE can provide different VIPs for different tenants, and different certificates can be configured for different VIPs.

### Memory Security Issues

Google engineers found that 70% of all serious security vulnerabilities in Chrome code base are memory management security vulnerabilities. Microsoft engineers also claimed that about 70% of Microsoft's security updates in the past 12 years were aimed at solving memory security vulnerabilities. OpenSSL famous [Heartbleed Bug](https://heartbleed.com/) known as one of the most serious security vulnerabilities in the history of the Internet, sensitive information in memory is leaked due to out-of-bounds access to memory information, affecting a large number of commonly used websites and services.

Benefiting from the built-in memory security feature of Go language, BFE can avoid the security problems caused by common C language buffer overflow memory problems.


### Security Grade of TLS

The correct configuration of various TLS/SSL parameters (protocol version, encryption suite) of the service requires the operation and maintenance personnel to have a deeper understanding of TLS/SSL security. In order to reduce the security risk of deployment caused by administrator misconfiguration, BFE provides four security grades for TLS, as follows. The TLS protocol version and encryption suite supported by BFE are different under different security grades.

| Security Grade | Description                               |
| -------------- | ----------------------------------------- |
| A+             | Highest security and lowest compatibility |
| A              | High security and moderate compatibility  |
| B              | Moderate security and high compatibility  |
| C              | Lowest security and highest compatibility |

Accordingly, different security grades have different security and compatibility, and are applicable to different business scenarios. For example, security grade A+ only supports TLS 1.2 and above and encryption suites with higher security strength. Grade A+ is applicable to business scenarios requiring PCI DSS level security compliance, such as financial payment business.

For the detailed definition of protocols and encryption suites of each security grade, see the description in "[Configure HTTPS Service](../../operation/configuration/https. md)".

### Secure Storage of Keys

The most secure solution is to create and save private keys inside hardware using HSM(Hardware Security Module). In this scheme, the calculation operation related to the private key is directly completed by the HSM hardware. The private key can never be separated from HSM, nor can it be extracted physically.

If the hardware is not available, the key can be encrypted and replaced regularly to control the risk and impact of key leakage to a certain extent. At present, CA manufacturers have begun to shorten the validity period of issuing certificates and will no longer issue certificates with a validity period of 2 years or more.

### TLS/SSL JA3 Fingerprint

BFE supports the calculation of TLS/SSL client fingerprints based on the characteristics in the ClientHello message. BFE uses JA3 algorithm, which can easily and efficiently identify client programs, and work with the business layer for anti-crawling or anti-cheating.

In BFE, you can use mod_header module to carry the JA3 fingerprint in the request header to the downstream. For specific usage, please refer to the relevant description of the [mod_header module](https://github.com/bfenetworks/bfe/blob/develop/docs/en_us/modules/mod_header/mod_header.md) in the user manual of BFE open source project.

### Traffic Visibility with TLS/SSL 

HTTPS brings challenges to bypass traffic attack detection and complex network problem diagnosis. The bypass attack detection system will not work effectively because it cannot handle the ciphertext traffic. In addition, R&D, operation and maintenance personnel sometimes rely on decryption and analysis of encrypted network packet capturing files. BFE can cooperate with the bypass attack detection system, and realize the analysis of attack characteristics at the transport layer, security layer, application layer, etc. through the secure shared session key support. At the same time, BFE can optionally write the TLS session master key to the key.log log file for wireshark software to analyze and encrypt network messages.


### Redirect From HTTP To HTTPS with Canary Release

Due to the complexity of HTTPS transformation, the migration from HTTP to HTTPS requires a flexible Canary Release mechanism to control the redirect. BFE supports the implementation of fine-grained policies, such as distinguishing domain names, clients, user regions, etc., and setting Canary Release redirect policies.

### Update Certificate with Canary Release

Certificate update is a high-risk operation. The certificate chain configuration error, the changes in sending the certificate by the intermediate CA in the certificate chain, the lack of necessary extensions in the certificate, and the non-standard format of the certificate may trigger compatibility problems, and lead to some user access exceptions after the certificate is updated. Reliable control of update risk and rapid detection of potential problems are indispensable for ordinary operation and maintenance personnel.

BFE supports updating certificates by user sampling with Canary Release, and supports real-time statistics of the change of handshake success rate between new and old certificates. It can quickly intercept exceptions that are difficult to find through the fluctuation of total traffic in the small traffic stage in Canary Release.

### Mutual Backup Certificates from Multi-CA

CA has become a new source of HTTPS website stability risk. The selection of CA is a key issue. In addition to the certificate cost of the CA, it is also necessary to consider the root certificate compatibility of the CA, OCSP service stability, security compliance history, and whether it is the core main business.

With the large-scale popularization of HTTPS, major failures of HTTPS caused by mainstream CAs are not uncommon in recent years. For example, in 2016, GlobalSign affected many well-known large websites due to OCSP service failure. In 2018, due to the security violation of Verisign, the world's largest CA, browsers such as Chrome/Firefox/Safari stopped trusting the certificates issued by Symantec.

HTTPS sites can achieve redundancy and mutual backup by issuing certificates from multiple CAs if conditions permit. When a CA's certificate access connectivity is abnormal, it can quickly switch and migrate to other CA certificates to stop loss.
