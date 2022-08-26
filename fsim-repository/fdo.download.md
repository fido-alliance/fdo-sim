This section defines the 'download' fdo serviceinfo module (fsim). An fsim is a set of key-value pairs; they define the 
onboarding operations that can be performed on a given FDO device. Fsim key-value pairs are exchanged between the device and it’s owning Device Management Service. It is up to the owning management service and the
device to interpret the key-value pairs in accordance with the fsim specification.

## fdo.download fsim definition
The download module provides the functionality to download a binary file from an owner to a device. There is no provision for textual conversion (e.g., UTF-8 to UTF-16).  Any format conversion should be done either in the sender, before the file is sent, or in the receiver.  

There are two modes of file transfer.  In length-prefix mode, the size of the file is sent first, using the `download.length` message.  Then the file data is sent.  When the file contents have been sent using one or more `download.data` messages, the receiver returns the length of file received in `download.done`.  

In streaming mode, the `download.length` message is not used.  Data for the file is sent.  When all the bytes have been transmitted a final `download.data` message of zero bytes indicates the end of file. 

In both cases, a zero length file requires a `download.data` message with a zero byte bstr in it.

Although the FSIM's communicate over a reliable channel, transfering a file from one file system to another involves more mechanism than just this channel.  Therefore, it is recommended that the file contents be verified after the file transmission.  This may be accomplished by transmitting a SHA-384 hash of the file before the first `data` message.  If the SHA-384 hash is received, the receiver MUST compute a SHA-384 hash of its own and compare the two hashes. If the hashes fail to match, the final `download.done` message MUST return a length of -1, regardless of the length of the file received.

The following table describes key-value pairs for the download fsim.


| Direction | Key Name                      | Value                      | Meaning   |
|:----------|:------------------------------|:---------------------------|:----------|
| o <-> d   | `fdo.download.active` | `bool` | Instructs the device to activate or deactivate the module  | 
| o --> d   | `fdo.download.name` | `tstr` | Specifies the *relative or* full path name of the file on the device.   | 
| o --> d   | `fdo.download.length` | `uint` | Specifies the expected length of the file in bytes.  If this message is provided, the transfer is in length-prefix mode.  Otherwise, the transfer is in streaming mode. | 
| o --> d   | `fdo.download.sha-384` | `bstr[48]` | Sends SHA-384 hash for file.  This forces the receiver to compute the SHA-384 of the received file and fail the transfer if the hashes do not match. |
| o --> d   | `fdo.download.data` | `bstr` | Writes a block of data up to 1014 bytes in size to the end of file  | 
| d --> o   | `fdo.download.done` | `int` | Indicates that the download has completed, returns the length of the target file.  Value of -1 indicates the sha-384 check failed, or other file write error |

* Geof: what happens to relative pathnames?   Relative to the client.  Assume this is a workspace area, should be cleared before the protocol.
* Geof: is there a concept of currnet directory?
* Geof: what about a device for Windows?  that's why you want to use relative pathnames
* Geof: Adding length prefix and streaming modes.

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
| - | `[fdo.download.data, 0]` |  Owner sends the final 188 bytes to the file | 
| `[fdo.download.active, True]` | - | Device confirms the module is available | 
| `[fdo.download.done, 700]`    | - |  The device acknowledges that download is complete | 






### download:active Message


### download:name Message


### download:length Message

### download:data Message

### complete:data Message

