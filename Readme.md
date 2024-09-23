# Public Key Infrastructure
PKI is a framework of hardware, software, policies, and standards used to manage digital certificates and public-private key pairs. Its purpose is to ensure secure electronic communication and authentication over networks, particularly the internet.
## Key Components of PKI:
- **Public and Private Keys:** PKI is based on asymmetric cryptography, where a public key encrypts data, and a corresponding private key decrypts it.

- **Public Key:** Shared with others, used to encrypt data.

- **Private Key:** Kept secret, used to decrypt data encrypted by the public key.

- **Digital Certificates:** These are electronic documents that link the public key to the identity of the certificate holder (such as a website or an individual). Certificates are issued by trusted entities called Certificate Authorities (CAs).

- **Certificate Authority (CA):** A trusted organization responsible for issuing and verifying digital certificates. CAs ensure that a public key belongs to the entity to whom the certificate is issued.

- **Registration Authority (RA):** An intermediary between the user and the CA. The RA verifies the user’s identity before forwarding a certificate request to the CA.

- **Certificate Revocation List (CRL):** A list of digital certificates that have been revoked before their expiration date, often due to compromise or misuse.

- **Trust Model:** PKI relies on a hierarchy of trust, where root CAs issue certificates to intermediate CAs, which in turn issue certificates to end-users (such as websites). These chains of trust are used to authenticate users or systems.

PKI allows for encrypted communication over the internet via the SSL/TLS protocol, ensuring that even if the data is intercepted, it cannot be decrypted without the private key.

## General Concept of Certificates

- **Key Pair Generation:**
When you generate a private key, it forms the foundation of your PKI setup.
The private key is stored securely on your server and is never shared.
The public key, which is mathematically linked to the private key, is embedded in the certificate.

- **Certificate Creation:**
A certificate signing request (CSR) is created using the private key. The CSR includes information about the website or server, including the public key, and is then used to generate the self-signed certificate.
A Certificate Authority (CA) normally verifies this information before signing the certificate, but in the case of a self-signed certificate, the website (you) is acting as both the issuer (CA) and the recipient.

- **Digital Signature:**
When you create a self-signed certificate, you sign the CSR with your private key, simulating the role of a CA. This signature helps verify the integrity of the certificate.
The self-signed certificate binds the public key to the identity (domain) mentioned in the certificate.

- **Establishing Trust:**
In a traditional PKI system, certificates are signed by a CA that browsers trust by default. When a browser encounters a certificate signed by a known CA, it trusts that certificate.
However, a self-signed certificate doesn’t come from a trusted CA, which is why browsers display a warning about trust issues when visiting sites with self-signed certificates. This warning occurs because there's no third-party validation of the certificate's authenticity.

## Easy-RSA
Easy-RSA is a simple command-line tool that helps users set up and manage their own Public Key Infrastructure (PKI), primarily for VPNs and internal networks. It uses OpenSSL under the hood but simplifies the process of managing certificate authorities (CAs), generating certificates, and signing requests.

**Key Features of Easy-RSA:**

- **Simplified PKI management:** Easy-RSA provides scripts to create and manage a CA, generate certificates and private keys for servers and clients, and manage certificate revocation lists (CRLs). You can use it to create your own CA, which can then sign other certificates.

- **Automation:** It automates many of the steps that would otherwise require multiple OpenSSL commands.
