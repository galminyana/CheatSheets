## ClearPass CheatSheet
---

### CLI Commands
---

| Command | Description |
|-|-|
| `cluster list` | Show all cluster members info | 
| `system boot-image` | Show/make active unit images |

## Deal with boot images
```markup
[appadmin@CPPM]# system boot-image
Usage:
    system boot-image [-l] [-a <index>]
Where,
    -l             --  List boot images installed on the system
    -a <index>     --  Set active boot image version. Index is displayed
                       with [-l] option in front of an entry
[appadmin@CPPM]#
```