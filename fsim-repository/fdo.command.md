This section defines the 'command' fdo serviceinfo module (fsim). An fsim is a set of key-value pairs; they define the
onboarding operations that can be performed on a given FDO device. Fsim key-value pairs are exchanged between the device and it’s owning Device Management Service. It is up to the owning management service and the
device to interpret the key-value pairs in accordance with the fsim specification.

## fdo.command fsim definition
The command module provides the functionality to execute arbitrary commands on the device.
The following table describes key-value pairs for the command fsim.




| Direction | fdo.command.*                  | Value                             | Meaning                 |
|:----------|:-------------------------------|:----------------------------------|:------------------------|
| o --> d   | active | bool | Instructs the device to activate or deactivate the module  | 
| o --> d   | command| tstr | Specifies the command processor to execute (e.g. sh , bash, or cmd )  | 
| o --> d   | args | bstr | bstr encoded cbor array of arguments   | 
| o --> d   | may_fail | bool | If true, TO2 will continue if the command fails otherwise TO2 will fail with message 255 if the command fails  | 
| o --> d   | return_stdout | bool | If true the device should return the standard out stream to the owner  | 
| o --> d   | return_stderr | bool | If true, the device should return the standard err stream to the owner  | 
| o --> d   | sig | int | Sends a signal to the running process.  Terminates all running commands.  | 
| d --> o   | stdout | bstr | A block of data containing standard out | 
| d --> o   | stderr | bstr | A block of data containing standard error | 
| d --> o   | exitcode | int | A block of data containing standard error | 

* Geof: why is command:command different from args?  Perhaps this is an optional way to specify the shell to invoke?
* Geof: When does the command actually run?  When fdo.command.args is processed?
* Geof: can you run a second command?  Perhaps buffered (see below)?  Perhaps after exitcode is received?
* Geof: what happens if you send several commands in a buffer and then later you send a "sig"?

The following table describes the expected message flow for the command fsim (long entries reference to below the table):

| Device sends (json-style)      | Owner sends                     | Meaning   |
|:------------------------------ |:---------------------------------- |:------------------------ |
| - | `['fdo.command:active', True]` | Module is active |
| - | `['fdo.command:command', 'sh']` | Indicates shell to use |
| - | `['fdo.command:return_stdout, True]` | send stdout in reverse message |
| - | `['fdo.command:return_stderr, False]` | do not send stderr (swallow it) |
| - | `['fdo.command:may_fail', True]` | failure of command does not cause TO2 to fail |
| - | `['fdo.command:args', bstr .cbor *1]` | request (short) list of file systems.  Invokes command. |
| - | `['fdo.commmand:return_stderr, True]` | enable stderr after command 1 (for some reason) |
| - | `['fdo.command:args', bstr .cbor *2]` | Invokes second command. |
| [`'fdo.command:stdout', *3]` | - | stdout from invoked command 1 |
| [`'fdo.command:stdout', *4]` | - | more stdout from invoked command 1 |
| [`'fdo.command:exitcode', 0]` | - | exit code indicates command 1 is complete |
| [`'fdo.command:stderr', *5]` | - | stderr from invoked command 2 |
| [`'fdo.command:exitcode', 1]` | - | error exit code from command 2, which is complete

* ***1** is: `['sfdisk', '-al', '/dev/sda']`
* ***2** is: `['sfdisk', '-al', '/dev/sdb']`
* ***3** is: `'Device Start End Sectors Size Type /dev/sda1 2048 1050623 1048576 512M EFI System'`
* ***4** is: `'/dev/sda2  1050624 1000214527 999163904 476.4G Linux filesystem'`
* ***5** is: `'sfdisk: /dev/sdb: No such file or directory'`

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

Device to owner
['command:command,'sh']
['command:args, String array]
['command:stdout',bstr]  can be streamed
['command:stderr',bstr]  can be streamed
['command:exit_code,int]


