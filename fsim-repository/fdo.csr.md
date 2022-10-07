This specification defines the 'CSR' (certificate signing request) FDO serviceinfo module (FSIM) for the purpose of certificate enrollment. An FSIM is a set of key-value pairs; they define the onboarding operations that can be performed on a given FDO device. FSIM key-value pairs are exchanged between the device and it's owning Device Management Service. It is up to the owning Device Management Service and the device to interpret the key-value pairs in accordance with the FSIM specification.

## fdo.csr FSIM Definition

The CSR module provides the functionality to issue a certificate signing request from the FDO Device to a Certification Authority (CA) or a Registration Authority (RA) via the owning Device Management Service. It supports a subset of the functionality defined in RFC 7030, i.e. the full Certificate Management over CMS (CMC) functionality is not supported. The benefit of re-using RFC 7030 is the ability to integrate with existing certificate enrollment infrastructure. Such integration may, for example, utilize the owning Device Management Service to relay communication between a CA back and the device. The communication from the owning Device Management Service to the CA may happen via the Enrollment over Secure Transport protocol (EST). Since this specification re-uses the standardized payloads, those can be re-used for the communication between the owning Device Management Service and the device.

The CSR FSIM supports the following functionality:
- Distribution of CA certificates
- Enrollments of clients
- Re-enrollment of clients
- Server-side key generation
- CSR attributes

While the first three features are mandatory-to-implement in this FSIM, the latter two (server-side key generation and CSR attributes) are optional.

FSIM's communicate over a reliable channel that experiences communication security with confidentiality, integrity and replay protection. Certificate enrollment messages benefit from the security protection offered by the underlying channel but may require additional protection, depending on the use case. 

The following table describes key-value pairs for the CSR FSIM.

| Direction | Key Name                      | Value                      | Meaning   |
|:----------|:------------------------------|:---------------------------|:----------|
| o <-> d   | `fdo.csr.active` | `bool` | Instructs the device to activate or deactivate the module  | 
| o <-- d   | `fdo.csr.cacerts-req` | `uint` | Request to obtain CA certificates |
| o --> d   | `fdo.csr.cacerts-res` | `tstr` | CA certificates |
| o <-- d   | `fdo.csr.simpleenroll-req` | `tstr` | Certificate enrollment request |
| o --> d   | `fdo.csr.simpleenroll-res` | `tstr` | Enrollments of clients |
| o <-- d   | `fdo.csr.simplereenroll-req` | `tstr` | Request to re-enroll a client |
| o --> d   | `fdo.csr.simplereenroll-res` | `tstr` | Re-enrollment response |
| o <-- d   | `fdo.csr.serverkeygen-req` | `tstr` | Request for server-side key generation |
| o --> d   | `fdo.csr.serverkeygen-res` | `tstr` | Certificate and private key |
| o <-- d   | `fdo.csr.csrattrs-req` | `uint` | Request for CSR attributes |
| o --> d   | `fdo.csr.csrattrs-res` | `tstr`  | CSR attributes |
| o --> d   | `fdo.csr.error` | `uint`  | Error Indication |

## fdo.csr.cacerts-req and fdo.csr.cacerts-res

A device requests CA certificates by issuing the fdo.csr.cacerts-req message. The indicated value informs the owning Device Management Service in what format the CA certificates have to be returned. 

A successful response is conveyed in the fdo.csr.cacerts-res message. The format mandatory-to-implement is 'application/pkcs7-mime; smime-type=certs-only' with value 281. The optional format is 'application/pkix-cert' with value 287.

## fdo.csr.simpleenroll-req and fdo.csr.simpleenroll-res

A device uses a Simple PKI Request, as specified in CMC (RFC 5272, Section 3.1 (i.e., a PKCS #10 Certification Request [RFC2986]). The payload in the fdo.csr.simpleenroll-req message is encoded as a 'application/pkcs10' payload. 
The Certification Signing Request (CSR) signature provides proof-of-possession of the client-possessed private key.

A successful response is carried in a fdo.csr.simpleenroll-res message, which carries the certificate encoded as 'application/pkix-cert'.

## fdo.csr.simplereenroll-req and fdo.csr.simplereenroll-res

A device uses a Simple PKI Request, as specified in CMC (RFC 5272, Section 3.1 (i.e., a PKCS #10 Certification Request [RFC2986]), to request a certificate. The payload in the fdo.csr.simplereenroll-req message is encoded as a 'application/pkcs10' payload. The Certificate Signing Request (CSR) signature provides proof-of-possession of the client-possessed private key.

A successful response is carried in a fdo.csr.simplereenroll-res message, which carries the certificate encoded as 'application/pkix-cert'.

## fdo.csr.serverkeygen-req and fdo.csr.serverkeygen-res

A device requests server-side key generation by issuing a fdo.csr.serverkeygen-req to the owning Device Management Service. 
The request uses the same format as the fdo.csr.simpleenroll-req and the fdo.csr.simplereenroll-req messages. The owning Device
Management Service and the certificate enrollment servers SHOULD treat the CSR as it would any enroll or re-enroll CSR, as 
described in RFC 7030. The only distinction is that the public key values and signature in the CSR MUST be ignored. These are
included in the request only to allow re-use of existing libraries for generating and parsing such requests.

If the device wants to receive an private key encrypted at the object layer in addition to that of the communication security
offered by FDO, then the client MUST include an attribute in the CSR indicating the encryption key to be used. Section 4.4.1.1 
and Section 4.4.1.2 of [RFC 7030] describe how to request symmetric and asymmetric key encryption of the private key. 

A successful response is returned in the fdo.csr.serverkeygen-res in form of a multipart/mixed MIME payload with boundary
set to "fdo" containing two parts: one part is the private key and the other part is the certificate.  

The certificate is an "application/pkcs7-mime" and exactly matches the certificate response to simpleenroll response message.

The format in which the private key part is returned is dependent on whether the private key is being returned with
additional encryption on top of that provided by FDO.

If additional encryption is not being employed, the private key data MUST be placed in an "application/pkcs8". 
An "application/pkcs8" part consists of the base64-encoded DER-encoded PrivateKeyInfo with a 
Content-Transfer-Encoding of "base64" [RFC2045].

If additional encryption is being employed, the private key is placed inside of a CMS SignedData. The text below describes
the possible variants and is an adaption of the description in Section 4.4.2 of [RFC7030]. The normative text herein 
intentionally aligns with RFC 7030 for code-reuse purposes.

The SignedData is signed by the party that generated the private key, which may or may not be the owning Device Management 
Service or the CA. The SignedData is further protected by placing it inside of a CMS EnvelopedData, as described in Section 4 of
[RFC5958]. The following list shows how the EncryptedData is used, depending on the type of protection key specified by the client.

- If the client specified a symmetric encryption key to protect the server-generated private key, the EnvelopedData content is 
encrypted using the secret key identified in the request.  The EnvelopedData RecipientInfo field MUST indicate the key-encryption
kekri key management technique.  The values are as follows: version is set to 4, key-encryption key identifier (kekid) is set
to the value of the DecryptKeyIdentifier from Section 4.4.1.1 of [RFC 7030]; keyEncryptionAlgorithm is set to one of the key wrap
algorithms that the client included in the SMIMECapabilities accompanying the request; and encryptedKey is the encrypted key.

- If the client specified an asymmetric encryption key suitable for key transport operations to protect the server-generated private
key, the EnvelopedData content is encrypted using a randomly generated symmetric encryption key.  The cryptographic strength of
the symmetric encryption key SHOULD be equivalent to the client-specified asymmetric key.  The EnvelopedData RecipientInfo field
MUST indicate the KeyTransRecipientInfo (ktri) key management technique.  In KeyTransRecipientInfo, the RecipientIdentifier
(rid) is either the subjectKeyIdentifier copied from the attribute defined in Section 4.4.1.2 of [RFC 7030] or the server determines
an associated issuerAndSerialNumber from the attribute; version is derived from the choice of rid [RFC5652], keyEncryptionAlgorithm 
is set to one of the key wrap algorithms that the client included in the SMIMECapabilities accompanying the request, and encryptedKey
is the encrypted key.

- If the client specified an asymmetric encryption key suitable for key agreement operations to protect the server-generated private
key, the EnvelopedData content is encrypted using a randomly generated symmetric encryption key.  The cryptographic strength of
the symmetric encryption key SHOULD be equivalent to the client-specified asymmetric key.  The EnvelopedData RecipientInfo field
MUST indicate the KeyAgreeRecipientInfo (kari) key management technique.  In the KeyAgreeRecipientInfo type, version,
originator, and user keying material (ukm) are as in [RFC5652], and keyEncryptionAlgorithm is set to one of the key wrap
algorithms that the client included in the SMIMECapabilities accompanying the request.  The recipient's key identifier is
either copied from the attribute defined in Section 4.4.1.2 of [RFC 7030] to subjectKeyIdentifier or the server determines an
IssuerAndSerialNumber that corresponds to the value provided in the attribute.

In all three additional encryption cases, the EnvelopedData is returned in the response as an "application/pkcs7-mime" part with an
smime-type parameter of "server-generated-key" and a Content-Transfer-Encoding of "base64".

An example of a successful response to a fdo.csr.serverkeygen-req might look like:

```
--fdo
Content-Type: application/pkcs8
Content-Transfer-Encoding: base64

MIIEvgIB...//Base64-encoded private key//..ATp4HiBmgQ
--fdo
Content-Type: application/pkcs7-mime; smime-type=certs-only
Content-Transfer-Encoding: base64

MIIDRQYJK..//Base64-encoded certificate//..dDoQAxAA==
--fdo--
```

## fdo.csr.csrattrs-req and fdo.csr.csrattrs-res

A device requests CSR attributes from the owning Device Management Service and consequently from the certificate enrollment servers with a fdo.csr.csrattrs-req message. The value carried in the fdo.csr.csrattrs-req message is ignored by the owning Device Management Service. 

A successful response informs the device about the fields to include in a CSR. This Certificate Signing Request (CSR) attribute messages is encoded in application/csrattrs format, as defined in Section 4.5.2 of RFC 7030.

## fdo.csr.error

The following table lists error codes returned by the fdo.csr.error message. 

| Error Number          | Description               | Sent in response to           |
|:----------------------|:--------------------------|:------------------------------|
| 1                     | Bad request.              | fdo.csr.simpleenroll-req      |
|                       |                           | fdo.csr.simplereenroll-req    |
|                       |                           | fdo.csr.serverkeygen-req      | 
|                       |                           | fdo.csr.csrattrs-req          | 
| 2                     | Unauthorized.             | fdo.csr.simpleenroll-req      |
|                       |                           | fdo.csr.simplereenroll-req    |
|                       |                           | fdo.csr.serverkeygen-req      | 
|                       |                           | fdo.csr.csrattrs-req          | 
| 3                     | Feature not supported.    | fdo.csr.csrattrs-req          |
|                       |                           | fdo.csr.serverkeygen-req      |
| 4                     | Rate exceeded. Try later. | fdo.csr.simpleenroll-req      |
|                       |                           | fdo.csr.simplereenroll-req    |
|                       |                           | fdo.csr.serverkeygen-req      | 
| 5                     | Unsupported format.       | fdo.csr.cacerts-req           |

An error of type 'unauthorized' is used when the request by the client cannot be processed by the Device Management Service, Certification Authority (CA) or Registration Authority (RA) due to insufficient permissions. The error of type 'bad request' is used when the request is malformed and parsing failed. 

## Example

The following table describes an example exchange for the csr fsim:

| Device sends  | Owner sends | Meaning   |
|:----------------------|:----------------------------------|:------------------------|
| `[fdo.csr.active, True]`  | - | Device instructs owner to activate the csr fsim  |
| `[fdo.csr.cacerts-req, 281]` | - | Request for CA certs |
| - | `[fdo.csr.cacerts-res, (tstr)abc...]` | CA cert response |
| `[fdo.csr.simpleenroll-req, (tstr)cde...]` | -  | Certificate enrollment request |
| - | `[fdo.csr.simpleenroll-res, (tstr)efa...]` | Certificate |
| `[fdo.csr.active, False]`  | - | Device instructs owner to deactivate the csr fsim  |


