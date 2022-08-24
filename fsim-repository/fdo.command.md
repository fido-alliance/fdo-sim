This section defines the 'command' fdo serviceinfo module (fsim). An fsim is a set of key-value pairs; they define the
onboarding operations that can be performed on a given FDO device. Fsim key-value pairs are exchanged between the device and it’s owning Device Management Service. It is up to the owning management service and the
device to interpret the key-value pairs in accordance with the fsim specification.

## fdo.command fsim definition
The command module provides the functionality to execute arbitrary commands on the device.
The following table describes key-value pairs for the command fsim.


Device to owner
['command:command,'sh']
['command:args, String array]
['command:stdout',bstr]  can be streamed
['command:stderr',bstr]  can be streamed
['command:exit_code,int]


| fdo.command.*                  | Value                      | Meaning   |
|:------------------------------ |:----------------------------------|:------------------------|
| active | bool | Instructs the device to activate or deactivate the module  | 
| command| tstr | Specifies the command to execute (e.g. sh , bash, or cmd )  | 
| args | bstr | bstr encoded cbor array of arguments   | 
| may_fail | bool | If true, TO2 will continue if the command fails otherwise TO2 will fail with message 255 if the command fails  | 
| return_stdout | bool | If true the device should return the standard out stream to the owner  | 
| return_stderr | bool | If true, the device should return the standard err stream to the owner  | 
| sig | int | Sends a signal to the running process  | 
| stdout | bstr | A block of data containing standard out | 
| stderr | bstr | A block of data containing standard error | 
| exitcode | int | A block of data containing standard error | 

The following table describes the expected message flow for the command fsim

| Device sends (json-style)      | Owner sends                     | Meaning   |
|:------------------------------ |:---------------------------------- |:------------------------ |
| `['fdo.command:active', True]` | - | Module is active |
| `['fdo.command:command', 'sh']` | - | Indicates shell to use |
| `['fdo.command:args', bstr .cbor ['sfdisk', '-al', '/dev/sda']]` | - | request (short) list of file systems |
| `['fdo.command:may_fail', True]` | - | failure of command does not cause TO2 to fail |
| `['fdo.command:return_stdout, True]` | - | send stdout in reverse message |
| `['fdo.command:return_stderr, False]` | - | do not send stderr (swallow it) |

| `['fdo.command:

## Example

```
[command:active,True]
['fdo.command:command,'sh']
['fdo.command:args, bstr .cbor ['sfdisk', '-ql', '/dev/sda']]
['fdo.command:'may_fail',True]
['fdo.command:'return_stdout' True]
['fdo.command':return_stderr',True]
['fdo.command:execute, cbor null]
['fdo.command:sig,int]
]
```



