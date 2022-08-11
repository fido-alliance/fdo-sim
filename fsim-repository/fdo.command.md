This section defines the 'command' fdo serviceinfo module (fsim). An fsim is a set of key-value pairs; they define the
onboarding operations that can be performed on a given FDO device. Fsim key-value pairs are exchanged between the device and it’s owning Device Management Service. It is up to the owning management service and the
device to interpret the key-value pairs in accordance with the fsim specification.

## fdo.command fsim definition
The command module provides the functionality to execute arbitrary commands on the device.
The following table describes key-value pairs for the command fsim.


[command:active,True]
['command:command,'sh'
['command:args, String array
['command:'may_fail',Bool
['command:'return_stdout' Bool
['command':return_stderr',Bool
['command:execute, cbor null
['command:sig,int]
]
Device to owner
['command:command,'sh'
['command:args, String array
['command:stdout',bstr]  can be streamed
['command:stderr',bstr]  can be streamed
['command:exit_code,int]


| Key Name                      | Value                      | Meaning   |
| ------------------------------:|:----------------------------------:|:------------------------:|
| fdo.command.active | bool | Instructs the device to activate or deactivate the module  | 
| fdo.command.command| tstr | Specifies the command to execute (e.g. sh , bash, or cmd )  | 
| fdo.command.args | bstr | bstr encoded cbor array of arguments   | 
| fdo.command.may_fail | bool | If true, TO2 will continue if the command fails otherwise TO2 will fail with message 255 if the command fails  | 
| fdo.command.return_stdout | bool | If true the device should return the standard out stream to the owner  | 
| fdo.command.return_stderr | bool | If true, the device should return the standard err stream to the owner  | 
| fdo.command.sig | int | Sends a signal to the running process  | 
| fdo.command.stdout | bstr | A block of data containing standard out | 
| fdo.command.stderr | bstr | A block of data containing standard error | 
| fdo.command.exitcode | int | A block of data containing standard error | 

The following table describes the expected message flow for the command fsim

| Device sends                    | Owner sends                     | Meaning   |
| ------------------------------:|:----------------------------------:|:------------------------:|
| x | x | x |






