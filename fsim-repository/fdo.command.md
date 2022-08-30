This section defines the 'command' fdo serviceinfo module (fsim). An fsim is a set of key-value pairs; they define the
onboarding operations that can be performed on a given FDO device. Fsim key-value pairs are exchanged between the device and it’s owning Device Management Service. It is up to the owning management service and the
device to interpret the key-value pairs in accordance with the fsim specification.

## fdo.command fsim definition
The command module provides the functionality to execute arbitrary commands on the device.
The following table describes key-value pairs for the command fsim.

The `fdo.command` module offers the ability to invoke a shell command on the FDO device during onboarding.  For this FSIM, only one command may be in progress at a time.  It is not recommended to buffer more than one command in a ServiceInfo message, because a buffer deadlock may result for some devices.  (The authors accept that a more complex FSIM may permit multiple buffered commands to execute simultaneously, and welcome submissions).

Commmands run to termination synchronously with the ServiceInfo processing.  I.e., no new ServiceInfo is processed between the time that command:execute is processed and when that command finishes with command:exitcode.  Only a single command may be buffered at a time.  This means that each command must be in its own ServiceInfo message.  The message command:sig applies to the currently executing command.

It is intended that this FSIM support multiple OS targets, including Linux and MS-Windows.  Small devices might implement shell-like functionality for specific commands, to limit the need for custom FSIM's.  

The messages: command, args, may_fail, return_stdout, return_stderr may appear in any order, so long as they appear before the execute commmand.

| Direction | fdo.command.*                  | Value                             | Meaning                 |
|:----------|:-------------------------------|:----------------------------------|:------------------------|
| o --> d   | active | bool | Instructs the device to activate or deactivate the module  | 
| o --> d   | command| tstr | Specifies the command processor to execute (e.g. sh , bash, or cmd )  | 
| o --> d   | args | bstr | bstr encoded cbor array of arguments   | 
| o --> d   | may_fail | bool | If false, Device will terminate TO2 on error (FDO message 255); otherwise device will send exitcode and continue to process ServiceInfo.  This permits the Owner side to either take recovery action or fail the connection (FDO message 255).  | 
| o --> d   | return_stdout | bool | If true the device should return the standard out stream to the owner  | 
| o --> d   | return_stderr | bool | If true, the device should return the standard err stream to the owner  | 
| o --> d   | execute | cbor Null | execute the command  | 
| o --> d   | sig | int | Sends a signal to the running process.  Terminates all running commands.  | 
| d --> o   | stdout | bstr | A block of data containing standard out | 
| d --> o   | stderr | bstr | A block of data containing standard error | 
| d --> o   | exitcode | int | A block of data containing standard error.  Follows *nix convention, where exitcode==0 is a normal return. | 

The following table describes the expected message flow for the command fsim (long entries reference to below the table):

| Device sends (json-style)      | Owner sends                     | Meaning   |
|:------------------------------ |:---------------------------------- |:------------------------ |
| - | `['fdo.command:active', True]` | Module is active |
| - | `['fdo.command:command', 'sh']` | Indicates shell to use |
| - | `['fdo.command:return_stdout, True]` | send stdout in reverse message |
| - | `['fdo.command:return_stderr, False]` | do not send stderr (swallow it) |
| - | `['fdo.command:may_fail', True]` | failure of command does not cause TO2 to fail |
| - | `['fdo.command:args', bstr .cbor *1]` | request (short) list of file systems.  Invokes command. |
| - | `['fdo.commmand:execute, bstr .cbor Null]` | Invoke command |
| [`'fdo.command:stdout', *3]` | - | stdout from invoked command 1 |
| [`'fdo.command:stdout', *4]` | - | more stdout from invoked command 1 |
| [`'fdo.command:exitcode', 0]` | - | exit code indicates command 1 is complete |

* ***1** is: `['sfdisk', '-al', '/dev/sda']`
* ***3** is: `'Device Start End Sectors Size Type /dev/sda1 2048 1050623 1048576 512M EFI System'`
* ***4** is: `'/dev/sda2  1050624 1000214527 999163904 476.4G Linux filesystem'`

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


