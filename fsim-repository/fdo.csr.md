This specification defines the 'CSR' (certificate signing request) FDO serviceinfo module (FSIM) for the purpose of certificate enrollment. An FSIM is a set of key-value pairs; they define the onboarding operations that can be performed on a given FDO device. FSIM key-value pairs are exchanged between the device and it's owning Device Management Service. It is up to the owning Device Management Service and the device to interpret the key-value pairs in accordance with the FSIM specification.

## fdo.csr FSIM Definition

The CSR module provides the functionality to issue a certificate signing request from the FDO Device to a Certification Authority (CA) or a Registration Authority (RA) via the Device Management Service. It supports a subset of the functionality defined in RFC 7030 [RFC7030], i.e. the full Certificate Management over CMS (CMC) functionality is not supported. The benefit of re-using RFC 7030 is the ability to integrate with existing certificate enrollment infrastructure.

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
The request uses the same format as the fdo.csr.simpleenroll-req and the fdo.csr.simplereenroll-req messages.

The owning Device Management Service and the certificate enrollment servers SHOULD treat the CSR as it would any enroll or re-enroll CSR. The only distinction is that the public key values and signature in the CSR MUST be ignored. These are included in the request only to allow re-use of existing libraries for generating and parsing such requests.
   
A successful response is returned in the fdo.csr.serverkeygen-res in form of a multipart-core MIME payload containing a PKCS#7-encoded certificate plus a PKCS#8-encoded unencrypted private key.

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


## References

[RFC5272]  Schaad, J. and M. Myers, "Certificate Management over CMS (CMC)", RFC 5272, DOI 10.17487/RFC5272, June 2008, <https://www.rfc-editor.org/info/rfc5272>.

[RFC7030]  Pritikin, M., Ed., Yee, P., Ed., and D. Harkins, Ed., "Enrollment over Secure Transport", RFC 7030, DOI 10.17487/RFC7030, October 2013, <https://www.rfc-editor.org/info/rfc7030>.

[RFC2986]  Nystrom, M. and B. Kaliski, "PKCS #10: Certification Request Syntax Specification Version 1.7", RFC 2986, DOI 10.17487/RFC2986, November 2000, <https://www.rfc-editor.org/info/rfc2986>.

[RFC2046]  Freed, N. and N. Borenstein, "Multipurpose Internet Mail Extensions (MIME) Part Two: Media Types", RFC 2046, DOI 10.17487/RFC2046, November 1996,
<https://www.rfc-editor.org/info/rfc2046>.

[RFC2045]  Freed, N. and N. Borenstein, "Multipurpose Internet Mail Extensions (MIME) Part One: Format of Internet Message Bodies", RFC 2045, DOI 10.17487/RFC2045, November 1996, <https://www.rfc-editor.org/info/rfc2045>.

[RFC5751]  Ramsdell, B. and S. Turner, "Secure/Multipurpose Internet Mail Extensions (S/MIME) Version 3.2 Message Specification", RFC 5751, DOI 10.17487/RFC5751, January 2010, <https://www.rfc-editor.org/info/rfc5751>.

[RFC8551]  Schaad, J., Ramsdell, B., and S. Turner, "Secure/Multipurpose Internet Mail Extensions (S/MIME) Version 4.0 Message Specification", RFC 8551, DOI 10.17487/RFC8551, April 2019, <https://www.rfc-editor.org/info/rfc8551>.
			  
[RFC5967]  Turner, S., "The application/pkcs10 Media Type", RFC 5967, DOI 10.17487/RFC5967, August 2010, <https://www.rfc-editor.org/info/rfc5967>.
			  
[RFC2585]  Housley, R. and P. Hoffman, "Internet X.509 Public Key Infrastructure Operational Protocols: FTP and HTTP", RFC 2585, DOI 10.17487/RFC2585, May 1999, <https://www.rfc-editor.org/info/rfc2585>.
			  
