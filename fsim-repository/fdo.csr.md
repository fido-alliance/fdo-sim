This section defines the 'csr' (certificate signing request) fdo serviceinfo module (fsim) for the purpose of certificate enrollment. An fsim is a set of key-value pairs; they define the onboarding operations that can be performed on a given FDO device. Fsim key-value pairs are exchanged between the device and it’s owning Device Management Service. It is up to the owning management service and the device to interpret the key-value pairs in accordance with the fsim specification.

## fdo.csr fsim definition

The csr module provides the functionality to issue a certificate signing request from the FDO Device to a Certification Authority (CA) or a Registration Authority (RA) via the FDO Owner. It supports a subset of the functionality defined in RFC 7030, i.e. the full CMC functionality is not supported. The benefit of re-using RFC 7030 functionality is the ability to integrate with existing certificate enrollment infrastructure.

This fsim functionality supports:
- Distribution of CA certificates
- Enrollments of clients
- Re-enrollment of clients
- Server-side key generation
- CSR attributes

While the first three features are mandatory-to-implement in this fsim, the latter two (server-side key generation and CSR attributes) are optional.

FSIM's communicate over a reliable channel that experiences communication security with confidentiality, integrity and replay protection. Certificate enrollment messages benefit from the security protection offered by the underlying channel but may require additional protection, depending on the use case. 

The following table describes key-value pairs for the csr fsim.

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
| o --> d   | `fdo.csr.error` | `uint`  | Error response |

## fdo.csr.cacerts-req and fdo.csr.cacerts-res

A device requests CA certificates by issuing the fdo.csr.cacerts-req message. The indicated value informs the owning management service in what format the CA certificates have to be returned. 

A successful response is conveyed in the fdo.csr.cacerts-res message. The format mandatory-to-implement is 'application/pkcs7-mime; smime-type=certs-only' with value 281. The optional format is 'application/pkix-cert' with value 287.

## fdo.csr.simpleenroll-req and fdo.csr.simpleenroll-res

A device uses a Simple PKI Request, as specified in CMC (RFC 5272, Section 3.1 (i.e., a PKCS #10 Certification Request [RFC2986]). The payload in the fdo.csr.simpleenroll-req message is encoded as a 'application/pkcs10' payload. 
The Certification Signing Request (CSR) signature provides proof-of-possession of the client-possessed private key.

A successful response is carried in a fdo.csr.simpleenroll-res message, which carries the certificate encoded as 'application/pkix-cert'.


## fdo.csr.simplereenroll-req and fdo.csr.simplereenroll-res

A device uses a Simple PKI Request, as specified in CMC (RFC 5272, Section 3.1 (i.e., a PKCS #10 Certification Request [RFC2986]) to request a certificate. The payload in the fdo.csr.simplereenroll-req message is encoded as a 'application/pkcs10' payload. The Certification Signing Request (CSR) signature provides proof-of-possession of the client-possessed private key.

A successful response is carried in a fdo.csr.simplereenroll-res message, which carries the certificate encoded as 'application/pkix-cert'.

## fdo.csr.serverkeygen-req and fdo.csr.serverkeygen-res

A device requests server-side key generation by issuing a fdo.csr.serverkeygen-req to the owning management service. 
The certificate request is the same format as for the fdo.csr.simpleenroll-req and fdo.csr.simplereenroll-req messages with the encoding.

The owning management service and consequently the certificate enrollment servers should treat the CSR as it would any enroll or re-enroll CSR; the only distinction is that the public key values and signature in the CSR must be ignored  These are included in the request only to allow re-use of existing codebases for generating and parsing such requests.
   
A successful response is returned in the fdo.csr.serverkeygen-res in form of a multipart-core MIME payload containing a PKCS#7-encoded certificate plus a PKCS#8-encoded unencrypted private key.

## fdo.csr.csrattrs-req and fdo.csr.csrattrs-res

A device requests CSR attributes from the owning management service and consequently from the certificate enrollment servers with a fdo.csr.csrattrs-req message. The value carried in the fdo.csr.csrattrs-req is ignored by the owning management service. 

A successful response informs the device about the fields to include in a CSR. This Certificate Signing Request (CSR) attribute messages is encoded in application/csrattrs format, as defined in Section 4.5.2 of RFC 7030.

## Error handling

TBD: This section describes the possible error cases. 

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


