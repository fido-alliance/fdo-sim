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

This section defines the 'download' fdo serviceinfo module (fsim). An fsim is a set of key-value pairs; they define the 
onboarding operations that can be performed on a given FDO device. Fsim key-value pairs are exchanged between the device and it’s owning Device Management Service. It is up to the owning management service and the
device to interpret the key-value pairs in accordance with the fsim specification.

## fdo.download fsim definition
The download module provides the functionality to download a binary file from the FDO Owner to the FDO Device. There is no provision for format conversion (e.g., UTF-8 to UTF-16).  Any format conversion needed must be accomplished either in the sender, before the file is sent, or in the receiver.  In Linux-based Devices it is assumed that the target file contains the exact data transmitted, byte-for-byte.

There are two modes of file transfer.  In length-prefix mode, the size of the file is sent first, using the `download.length` message.  Then the file data is sent.  When the file contents have been sent using one or more `download.data` messages, the receiver returns the length of file received in `download.done`.  

In streaming mode, the `download.length` message is not used.  Data for the file is sent by the FDO Owner.  When all the bytes have been transmitted, the Owner sends a final `download.data` message of zero bytes to indicate the end of file transmission. 

In both cases, a zero length file requires a `download.data` message with a zero byte bstr in it.

Although the FSIM's communicate over a reliable channel, transfering a file from one file system to another involves more mechanism than just this channel.  Therefore, it is recommended that the file contents be verified after the file transmission.  This may be accomplished by transmitting a SHA-384 hash of the file before the first `data` message.  If the SHA-384 hash is received, the receiver MUST compute a SHA-384 hash of its own and compare the two hashes. If the hashes fail to match, the final `download.done` message MUST return a length of -1, regardless of the length of the file received.

Relative pathnames are permitted.  This may be useful for downloading short files, such as script files, to the device.  Downloading a large file requires knowledge of the storage architecture of the target; this might be obtained in advance for a particular model of device or might be queried using a script file.  

It may be assumed that relative pathnames always end up in the same place for a given device.  So it is possible to download a file and expect it to be in the same default directory as the command modules uses.  

The following table describes key-value pairs for the download fsim.


| Direction | Key Name                      | Value                      | Meaning   |
|:----------|:------------------------------|:---------------------------|:----------|
| o <-> d   | `fdo.download.active` | `bool` | Instructs the device to activate or deactivate the module  | 
| o --> d   | `fdo.download.name` | `tstr` | Specifies the *relative or* full path name of the file on the device.   | 
| o --> d   | `fdo.download.length` | `uint` | Specifies the expected length of the file in bytes.  If this message is provided, the transfer is in length-prefix mode.  Otherwise, the transfer is in streaming mode. | 
| o --> d   | `fdo.download.sha-384` | `bstr[48]` | Sends SHA-384 hash for file.  This forces the receiver to compute the SHA-384 of the received file and fail the transfer if the hashes do not match. |
| o --> d   | `fdo.download.data` | `bstr` | Writes a block of data up to 1014 bytes in size to the end of file  | 
| d --> o   | `fdo.download.done` | `int` | Indicates that the download has completed, returns the length of the target file.  Value of -1 indicates the sha-384 check failed, or other file write error |

The intended order of these commands is as follows:

* Active message is used only once per FDO session to activate the module (as for all FSIM's)
* .name
* .length
* .sha-384 if desired
* .download as many times as needed
* .done is the response

For relative pathnames, see the `fdo.command` module, which explains how the default directory is chosen.  

If a given pathname resolves to a device, behavior is device specific.  A drive letter on Windows is a part of the pathname, and may be used.  It might be necessary to use the `fdo.command` module to determine the correct drive letter to use.

The following table describes the expected message flow for the download fsim:

| Device sends  | Owner sends | Meaning   |
|:----------------------|:----------------------------------|:------------------------|
| -  | `[fdo.download.active, True]` | Owner instructs device to activate the download fsim  | 
| - | `[fdo.download.name, "foo"]` |  Owner instructs device to create file "foo" | 
| - | `[fdo.download.length, 700]` |  Owner instructs device to expect 700 total bytes | 
| - | `[fdo.download.sha-384, (bstr)ab34...]` |  (Optional) Sha-384 for the file | 
| - | `[fdo.download.data,  (bstr)590200...]` |  Owner sends the first 512 bytes to the file | 
| - | `[fdo.download.data, 188]` |  Owner sends the final 188 bytes to the file | 
|  `[fdo.download.active, True]` | - | Device confirms the module is available | 
|  `[fdo.download.done, 700]` | - |  The device acknowledges that download is complete | 

This example gives message flow for streaming:

| Device sends  | Owner sends | Meaning   |
|:----------------------|:----------------------------------|:------------------------|
| - | `[fdo.download.active, True]` | Owner instructs device to activate the download fsim  | 
| - | `[fdo.download.name, "foo"]`  |  Owner instructs device to create file "foo" | 
| - | `[fdo.download.data,  (bstr)590200...]` |  Owner sends the first 512 bytes to the file | 
| - | `[fdo.download.data, 188]` |  Owner sends the final 188 bytes to the file | 
| - | `[fdo.download.data, 0]` |  Owner indicates entire file has been sent | 
| `[fdo.download.active, True]` | - | Device confirms the module is available | 
| `[fdo.download.done, 700]`    | - |  The device acknowledges that download is complete | 






### download:active Message


### download:name Message


### download:length Message

### download:data Message

### complete:data Message

