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

This section defines the 'upload' fdo serviceinfo module (fsim). An fsim is a set of key-value pairs; they define the 
onboarding operations that can be performed on a given FDO device. Fsim key-value pairs are exchanged between the device and it’s owning Device Management Service. It is up to the owning management service and the
device to interpret the key-value pairs in accordance with the fsim specification.

## fdo.upload fsim definition
The upload module provides the functionality to transfer a binary file from the FDO Device to the FDO Owner. There is no provision for textual conversion (e.g., UTF-8 to UTF-16).  Any format conversion should be done either in the sender, before the file is sent (e.g., by using fdo.command), or in the receiver.  

FDO.upload uses length-prefix mode, where the size of the file is sent first, using the `upload.length` message.  Then the file data is sent.  

At least one `upload.data` message must be sent, where a zero length file `upload.data` message with a zero byte bstr in it.

Although the FSIM's communicate over a reliable channel, transfering a file from one file system to another involves more mechanism than just this channel.  Therefore, it is recommended that the file contents be verified after the file transmission.  The Owner may request a SHA-384 hash of the file be returned after the file is transfered.  It is up to the Owner to determine if the hash matches or not.

The following table describes key-value pairs for the upload fsim.


| Direction | Key Name                      | Value                      | Meaning   |
|:----------|:------------------------------|:---------------------------|:----------|
| o <-> d   | `fdo.upload.active` | `bool` | Instructs the device to activate or deactivate the module  | 
| o --> d   | `fdo.upload.name` | `tstr` | Specifies the *relative or* full path name of the file on the device to be retrieved.   |
| o --> d   | `fdo.upload.need-sha` | `bool` | Indicates whether a sha-384 message is to be transmitted by the Device for subsequent file transfers.  If no `need-sha` message is sent, then the `sha-384` message is never sent. |
| d --> o   | `fdo.upload.length` | `uint` | Specifies the expected length of the file in bytes.  This message must be sent before the first `data` message | 
| d --> o   | `fdo.upload.data` | `bstr` | Writes a block of data up to 1014 bytes in size to the end of file  | 
| d --> o   | `fdo.upload.sha-384` | `bytes[48]` | Contains the sha-384 of the file that has just been completely transfered. |

* Geof: what happens to relative pathnames?
* Geof: is there a concept of currnet directory?
* Geof: what about a device for Windows?
* Geof: Adding length prefix and streaming modes.
* Geof: Specified sha-384 as a state (send or do not send) -- maybe download should follow suite?

The following table describes the expected message flow for the upload fsim:

| Device sends  | Owner sends | Meaning   |
|:----------------------|:----------------------------------|:------------------------|
| -  | `[fdo.upload.active, True]` | Owner instructs device to activate the upload fsim  | 
| - | `[fdo.upload.need-sha, True ]` |  (Optional) Request Sha-384 for the next file | 
| - | `[fdo.upload.name, "foo"]` |  Owner instructs device to send file "foo" | 
| `[fdo.upload.active, True]` | - | Device confirms the module is available | 
| `[fdo.upload.length, 700]` |  Device instructs owner to expect 700 total bytes | 
| `[fdo.upload.data,  (bstr)590200...]` | - |  Device sends the first 512 bytes to the file | 
| `[fdo.upload.data, 188]` | - |  Device sends the final 188 bytes to the file | 
| `[fdo.upload.sha-384, (bstr)af3799... ]` |  SHA-384 for the file (if requested above) | 

