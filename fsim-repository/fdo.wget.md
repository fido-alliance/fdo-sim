This section defines the 'wget' fdo serviceinfo module (fsim). An fsim is a set of key-value pairs; they define the 
onboarding operations that can be performed on a given FDO device. Fsim key-value pairs are exchanged between the device and it’s owning Device Management Service. It is up to the owning management service and the
device to interpret the key-value pairs in accordance with the fsim specification.

## fdo.wget fsim definition
The wget module provides the functionality to transfer a binary file from the network to the FDO Device using a URL.  This is different from fdo.download and fdo.upload, which transmit file contents in-band in the FDO encrypted tunnel.  The wget module requests that the FDO Device access the network independently, in parallel with FDO, and transfer the file over another connection.

In most situations, wget is superior for transfering bulk data from the network to the FDO Device.  This is true because the FDO encrypted channel is not optimized for transfer speed, and also because the file may be accessed directly from its location on the connected network, without being transfered to the FDO Owner first.  However, the FDO encrypted channel provides security guarantees that may be hard to replicate with WGET:

* The FDO channel extends the authentication of the FDO session
* The FDO channel encrypts data being transfered to ensure cryptographic privacy
* The FDO channel verifies cryptographic integrity of transfered data

The name of the module is intentionally similar to that of a Linux utility.  However, the module may be implemented by the device in any manner, and does not have to support all the functionality of the Linux utility namesake.  An implementation of this module MUST support the HTTP mode of transfer.  

More complex FDO devices MAY support encrypted transfers over the WGET module using "https://" URLs.  However, the implementor is warned that the security of such a transfer is dependent on whether the Device is able to verify the integrity of server certificate.  This may require that certificate databases within the Device be updated **before** this module is invoked.  For example, these databases might be downloaded using the `fdo.download` FSIM and installed using the `fdo.command` FSIM.

This module is particularly useful for transfering bulk data that: is not secret; is pre-encrypted with long-lived certificates to verify them, such as Linux software update modules.  

The following table describes key-value pairs for the wget fsim.


| Direction | Key Name                      | Value                      | Meaning   |
|:----------|:------------------------------|:---------------------------|:----------|
| o <-> d   | `fdo.wget.active` | `bool` | Instructs the device to activate or deactivate the module  | 
| o --> d   | `fdo.wget.sha-384` | `bytes[48]` | (Optional) Contains the sha-384 of the file that will be retrieved. |
| o --> d   | `fdo.wget.name` | `tstr` | Indicates the target filename where the file will be stored |
| o --> d   | `fdo.wget.url` | `tstr` | Indicates the URL to be retrieved   |
| d --> o   | `fdo.wget.error` | `tstr` | Indicates failure with error message |
| d --> o   | `fdo.wget.done` | `int` | Indicates the length of the file successfully retrieved.  If the `sha-384` message has been received, the receiver computes the sha-384 of the file downloaded before it sends this message and sends an error message if the transfer failed. |

* Geof: what happens to relative pathnames?
* Geof: is there a concept of currnet directory?
* Geof: what about a device for Windows?
* Geof: Adding length prefix and streaming modes.

The following table describes the expected message flow for the wget fsim:

| Device sends  | Owner sends | Meaning   |
|:----------------------|:----------------------------------|:------------------------|
| -  | `[fdo.wget.active, True]` | Owner instructs device to activate the wget fsim  | 
| -  | `[fdo.upload.sha-384, (bstr)af3799... ]` |  (Optional) SHA-384 for the next file | 
| - | `[fdo.upload.name, "foo"]` |  Store next file as "foo" | 
| - | `[fdo.upload.wget, (tstr)"http://mysite/python3.deb" ]` |  Request to immediately transfer URL | 
| `[fdo.wget.active, True]` | - | Device confirms the module is available | 
| `[fdo.wget.done, 700]` |  - | Device instructs owner that 700 bytes were transferred | 
| `[fdo.wget.sha-384, (bstr)af3799... ]` |  - | SHA-384 for the file (if requested above) |
| `[fdo.wget.error, (tstr)"500 no such file"]` | - | error message |
