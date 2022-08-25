This section defines the 'download' fdo serviceinfo module (fsim). An fsim is a set of key-value pairs; they define the 
onboarding operations that can be performed on a given FDO device. Fsim key-value pairs are exchanged between the device and it’s owning Device Management Service. It is up to the owning management service and the
device to interpret the key-value pairs in accordance with the fsim specification.

## fdo.download fsim definition
The download module provides the functionality to download a file from an owner to a device. 
The following table describes key-value pairs for the download fsim.


| Direction | Key Name                      | Value                      | Meaning   |
|:----------|:------------------------------|:---------------------------|:----------|
| o --> d   | fdo.download.active | bool | Instructs the device to activate or deactivate the module  | 
| o --> d   | fdo.download.name| tstr | Specifies the name or full path of the file on the device.  If just the file name is specified ... what happens?   | 
| o --> d   | fdo.download.length | int | Specifies the expected length of the file in bytes   | 
| o --> d   | fdo.download.data | bstr | Writes a block of data up to 1014 bytes in size to the end of file  | 

* Geof: what happens to relative pathnames?
* Geof: is there a concept of currnet directory?
* Geof: what about a device for Windows?

The following table describes the expected message flow for the download fsim:

| Device sends  | Owner sends | Meaning   |
|:----------------------|:----------------------------------|:------------------------|
| -  | [fdo.download.active, True] | Owner instructs device to activate the download fsim  | 
|  [fdo.download.active, True] | - | Device confirms the module is available | 
| - | [fdo.download.name, "foo"]|  Owner instructs device to create file "foo" | 
| - | [fdo.download.length, 700]|  Owner instructs device to expect 700 total bytes | 
| - | [fdo.download.data,  bstr[590200..] ]|  Owner sends the first 512 bytes to the file | 
| - | [fdo.download.data, 188]|  Owner sends the final 188 bytes to the file | 
|  [fdo.download.complete, 0]| |  The device acknowledges that download is complete and sends optional hash| 





### download:active Message


### download:name Message


### download:length Message

### download:data Message

### complete:data Message

