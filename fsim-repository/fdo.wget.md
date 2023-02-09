Copyright 2023 FIDO Alliance

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

------------------

This section defines the 'wget' fdo serviceinfo module (fsim). An fsim is a set of key-value pairs; they define the 
onboarding operations that can be performed on a given FDO device. Fsim key-value pairs are exchanged between the device and it’s owning Device Management Service. It is up to the owning management service and the
device to interpret the key-value pairs in accordance with the fsim specification.

## fdo.wget fsim definition
The wget module provides the functionality to transfer a binary file, identified by a URL, to the FDO Device using the network(s) connected to the device.  This is different from fdo.download and fdo.upload, which transmit file contents in-band in the FDO encrypted tunnel.  The wget module requests that the FDO Device access the network independently, in parallel with FDO, and transfer the file over another connection.

If the target file already exists, it is replaced by the new file during wget.  The state of the target file of a wget is undefined until the wget completes.

In most situations, wget is superior for transfering bulk data from the network to the FDO Device.  This is true because the FDO encrypted channel is not optimized for transfer speed, and also because the file may be accessed directly from its location on the connected network, without being transfered to the FDO Owner first.  However, the FDO encrypted channel provides security guarantees that may be hard to replicate with WGET:

* The FDO channel extends the authentication of the FDO session
* The FDO channel encrypts data being transfered to ensure cryptographic privacy
* The FDO channel verifies cryptographic integrity of transfered data

The name of the module is intentionally similar to that of a Linux utility.  However, the module may be implemented by the device in any manner, and does not have to support all the functionality of the Linux utility namesake.  An implementation of this module MUST support the HTTP mode of transfer, and may support other modes of transfer using the familiar URL syntax.  Also, implementors are advised that this module is intended to be compatible between multiple devices, and is *not* intended to mirror all features of the Linux command; if this is desired, the user can invoke wget directly using the `fdo.command` module.  

More complex FDO devices MAY support encrypted transfers over the WGET module using "https://" URLs.  However, the implementor is warned that, even if TLS is used, the security of such a transfer is dependent on whether the Device is able to verify the integrity of the TLS server certificate.  This may require that certificate databases within the Device be updated **before** this module is invoked.  For example, these databases might be downloaded using the `fdo.download` FSIM and installed using the `fdo.command` FSIM.

This module is particularly useful for transfering bulk data that either:

* is not secret
* is pre-encrypted with long-lived certificates to verify them, such as Linux software update modules

The URL format identifies a name and path for the file, from which an be derived the default name for the downloaded file.  By default, this name is the last part of the URL's path (aka **pathinfo**).  For example: http://myhost.net/dir1/dir2/name.txt the default file name is "name.txt", relative to the current directory on the FDO Device.  An alternate path name may be specified using the `wget:name` message.

The following table describes key-value pairs for the wget fsim.


| Direction | Key Name                      | Value                      | Meaning   |
|:----------|:------------------------------|:---------------------------|:----------|
| o <-> d   | `fdo.wget.active` | `bool` | Instructs the device to activate or deactivate the module  | 
| o --> d   | `fdo.wget.sha-384` | `bytes[48]` | (Optional) Contains the sha-384 of the file that will be retrieved. |
| o --> d   | `fdo.wget.name` | `tstr` | Optional message, which overrides the target filename where the file will be stored.  This may be also be used to target a downloaded file into an absolute pathname, rather than the current directory. |
| o --> d   | `fdo.wget.url` | `tstr` | Indicates the URL to be retrieved   |
| d --> o   | `fdo.wget.error` | `tstr` | Indicates failure with error message |
| d --> o   | `fdo.wget.done` | `int` | Indicates the length of the file successfully retrieved.  If the `sha-384` message has been received, the receiver computes the sha-384 of the file downloaded before it sends this message and sends an error message if the transfer failed. |


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
