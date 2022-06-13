

# Cisco-ASA-Configurator
GUI Tool for automated Cisco ASA configuration command generation. Receives a standard formatted input for whitelisted traffic flows, correlates the IPv4 addresses to their respective internal or external Cisco ASA/Context/ACL, then generates commands to configure those flows.

## Features

 - Supports CIDR notation (192.168.1.0/24)
 - Supports Range Shorthand (192.168.1.1-20)
 - Supports Range Expanded for addresses & ports (192.168.1.1-192.168.1.20 || 8443-8450)
 - Supports object-groups of: From_Address, To_Address, To_Port
 - Automatic protocol name translation of any ports/protocols (SSH -> 22, HTTPS-> 443, TEREDO-> 3544)
 - Commands include CLI navigation (ssh, conf t, changeto context)
 - Optional remark field
 - Rich debug logs

## User interface

Placeholder text on top text area indicates supported input formatting. Although certain fields will be matched, they may not be fully supported; i.e. 'Remove' rule action doesn't currently generate removal commands.
![Blank user interface, rendered with Viola in notebook](https://user-images.githubusercontent.com/96323936/173407881-fb3cf786-6658-4426-acce-6ada35ddf480.PNG)Generated commands and DataFrame table displayed below, including example groupings and comma-separated ports.
![Generated dataframe and Cisco ASA commands after specifying file locations and optional remark line](https://user-images.githubusercontent.com/96323936/173407884-8fa2d72c-2a9d-4407-aca0-5a31a82bbcc5.png)Support for any valid CIDR notation input and optional REMARK line for in-ACL comments.
![enter image description here](https://user-images.githubusercontent.com/96323936/173407888-2942b9bf-c68d-47cd-a071-c13bad862676.PNG)

## Script Assumptions
***TAKE NOTE! SCRIPT ASSUMES:***

 - ACL names end in either "SVI" if external interface or "VLAN" if internal interface.
 - Object group "settings.json" are formatted as `<basename><number>` and increments that number to avoid overlapping object groups without having prior knowledge of existing object groups.
- Your ACL exists within a virtual context, within a Cisco ASA.
- From port is matched but unused -- assumed ephemeral port.
- Script doesn't automatically generate return traffic flow commands for UDP.

## Setup Instructions

 **1. Update "settings.json" to include your object group naming and numbering convention per your preferences**

    {"object_network_index": 1, 
    "object_network_prefix": "<YOUR_NETWORK_OBJECT_GROUP>",
    "object_service_index": 5,
    "object_service_prefix": "<YOUR_SERVICE_OBJECT_GROUP>"}

 **2. Update "correlations.json" to include your Cisco ASA management addresses, context names and ACL names**

    {
    "10.10.10.1": {
        "<YOUR_CONTEXT_1>": {
            "acl-25-VLAN": "25.25.25.0/24",
            "acl-25-SVI": "25.25.25.0/24",
			"acl-50-VLAN": "50.50.50.0/24",
			"acl-50-SVI": "50.50.50.0/24"
        }
    },
    "20.20.20.1": {
        "<YOUR_CONTEXT_2>": {
			"acl-75-VLAN": "75.75.75.0/24",
            "acl-75-SVI": "75.75.75.0/24",
            "acl-100-VLAN": "100.100.100.0/24",
            "acl-100-SVI": "100.100.100.0/24"
        }
    }
    }

## Notes

 - "raw_input" field automatically translates port names to port numbers (i.e. ssh -> 22, https -> 443) to allow for more flexible inputs.
 - Duplicate rule requests / line entries are automatically removed but will still display in the DataFrame table but not in command output.
 - If a rule's field (column) fails to produce a full match, the generated dataframe output will indicate "None" for that field of the rule but may still generate correct commands. Validity indicator will indicate an issue and the debug log  will outline exact cause of the issue.

## Debug

 - If debugging is enabled, a _debug.log file will be created and will produce formatted information about the processes the script is performing. This can help find errors in flow determination, correlation, command generation, etc.
 
 EXAMPLE INPUT:
 

    add testing 50.50.50.200  any testing 15.15.165.201 443 TCP
    add testing 50.50.50.200  any testing 15.15.165.202 443 TCP
    add testing 50.50.50.200  any testing 15.15.165.203 443 TCP
    add testing 50.50.50.100  any testing 15.15.165.101 22 TCP
    add testing 50.50.50.100  any testing 15.15.165.102 22 TCP
    add testing 50.50.50.100  any testing 15.15.165.103 22 TCP

GENERATED DEBUG:

    2022-06-13 13:40:47,603 PROGRAM_INITIALIZED####################################################################################################
    2022-06-13 13:40:49,791 ---------------
    2022-06-13 13:40:49,791 INFO:__main__:generate_button_onclick:$ generate button clicked
    2022-06-13 13:40:49,791 ---------------
    2022-06-13 13:40:49,792 INFO:__main__:generate_commands:$    begin line 1 : add testing 50.50.50.200  any testing 15.15.165.201 443 TCP
    2022-06-13 13:40:49,793 INFO:Rule:determine_fields:$         determined : 'add','testing','50.50.50.200','any','testing','15.15.165.201','443','TCP'
    2022-06-13 13:40:49,794 INFO:Correlation:correlate_address:$ correlated : 50.50.50.200 in 50.50.50.0/24 at 10.10.10.15 > CONTEXT1 > acl-50-VLAN
    2022-06-13 13:40:49,796 ---------------
    2022-06-13 13:40:49,797 INFO:__main__:generate_commands:$    begin line 2 : add testing 50.50.50.200  any testing 15.15.165.202 443 TCP
    2022-06-13 13:40:49,797 INFO:Rule:determine_fields:$         determined : 'add','testing','50.50.50.200','any','testing','15.15.165.202','443','TCP'
    2022-06-13 13:40:49,797 INFO:Correlation:correlate_address:$ correlated : 50.50.50.200 in 50.50.50.0/24 at 10.10.10.15 > CONTEXT1 > acl-50-VLAN
    2022-06-13 13:40:49,799 ---------------
    2022-06-13 13:40:49,799 INFO:__main__:generate_commands:$    begin line 3 : add testing 50.50.50.200  any testing 15.15.165.203 443 TCP
    2022-06-13 13:40:49,799 INFO:Rule:determine_fields:$         determined : 'add','testing','50.50.50.200','any','testing','15.15.165.203','443','TCP'
    2022-06-13 13:40:49,799 INFO:Correlation:correlate_address:$ correlated : 50.50.50.200 in 50.50.50.0/24 at 10.10.10.15 > CONTEXT1 > acl-50-VLAN
    2022-06-13 13:40:49,800 ---------------
    2022-06-13 13:40:49,800 INFO:__main__:generate_commands:$    begin line 4 : add testing 50.50.50.100  any testing 15.15.165.101 22 TCP
    2022-06-13 13:40:49,801 INFO:Rule:determine_fields:$         determined : 'add','testing','50.50.50.100','any','testing','15.15.165.101','22','TCP'
    2022-06-13 13:40:49,801 INFO:Correlation:correlate_address:$ correlated : 50.50.50.100 in 50.50.50.0/24 at 10.10.10.15 > CONTEXT1 > acl-50-VLAN
    2022-06-13 13:40:49,802 ---------------
    2022-06-13 13:40:49,802 INFO:__main__:generate_commands:$    begin line 5 : add testing 50.50.50.100  any testing 15.15.165.102 22 TCP
    2022-06-13 13:40:49,802 INFO:Rule:determine_fields:$         determined : 'add','testing','50.50.50.100','any','testing','15.15.165.102','22','TCP'
    2022-06-13 13:40:49,803 INFO:Correlation:correlate_address:$ correlated : 50.50.50.100 in 50.50.50.0/24 at 10.10.10.15 > CONTEXT1 > acl-50-VLAN
    2022-06-13 13:40:49,804 ---------------
    2022-06-13 13:40:49,804 INFO:__main__:generate_commands:$    begin line 6 : add testing 50.50.50.100  any testing 15.15.165.103 22 TCP
    2022-06-13 13:40:49,804 INFO:Rule:determine_fields:$         determined : 'add','testing','50.50.50.100','any','testing','15.15.165.103','22','TCP'
    2022-06-13 13:40:49,805 INFO:Correlation:correlate_address:$ correlated : 50.50.50.100 in 50.50.50.0/24 at 10.10.10.15 > CONTEXT1 > acl-50-VLAN
    2022-06-13 13:40:49,816 ---------------
    2022-06-13 13:40:49,816 INFO:__main__:generate_commands:$    determinations and correlations completed -- begin groupings
    2022-06-13 13:40:49,816 ---------------
    2022-06-13 13:40:49,821 INFO:Grouper:group_to_address:$      object-groupings of To_Address discovered
    2022-06-13 13:40:49,824 ---------------------------------------------TO ADDRESS GROUPING / BY ACL---------------------------------
      Action From_Environment  From_Address From_Port To_Environment     To_Address To_Port Protocol  ASA_Address ASA_Context      ASA_ACL
    0    add          testing  50.50.50.200       any        testing  15.15.165.201     443      TCP  10.10.10.15    CONTEXT1  acl-50-VLAN
    1    add          testing  50.50.50.200       any        testing  15.15.165.202     443      TCP  10.10.10.15    CONTEXT1  acl-50-VLAN
    2    add          testing  50.50.50.200       any        testing  15.15.165.203     443      TCP  10.10.10.15    CONTEXT1  acl-50-VLAN
    3    add          testing  50.50.50.100       any        testing  15.15.165.101      22      TCP  10.10.10.15    CONTEXT1  acl-50-VLAN
    4    add          testing  50.50.50.100       any        testing  15.15.165.102      22      TCP  10.10.10.15    CONTEXT1  acl-50-VLAN
    5    add          testing  50.50.50.100       any        testing  15.15.165.103      22      TCP  10.10.10.15    CONTEXT1  acl-50-VLAN
    2022-06-13 13:40:49,827 ---------------------------------------------TO ADDRESS GROUPING / BY GROUPING---------------------------------
      Action From_Environment  From_Address From_Port To_Environment     To_Address To_Port Protocol  ASA_Address ASA_Context      ASA_ACL
    0    add          testing  50.50.50.200       any        testing  15.15.165.201     443      TCP  10.10.10.15    CONTEXT1  acl-50-VLAN
    1    add          testing  50.50.50.200       any        testing  15.15.165.202     443      TCP  10.10.10.15    CONTEXT1  acl-50-VLAN
    2    add          testing  50.50.50.200       any        testing  15.15.165.203     443      TCP  10.10.10.15    CONTEXT1  acl-50-VLAN
    2022-06-13 13:40:49,829 ---------------------------------------------TO ADDRESS GROUPING / BY GROUPING---------------------------------
      Action From_Environment  From_Address From_Port To_Environment     To_Address To_Port Protocol  ASA_Address ASA_Context      ASA_ACL
    3    add          testing  50.50.50.100       any        testing  15.15.165.101      22      TCP  10.10.10.15    CONTEXT1  acl-50-VLAN
    4    add          testing  50.50.50.100       any        testing  15.15.165.102      22      TCP  10.10.10.15    CONTEXT1  acl-50-VLAN
    5    add          testing  50.50.50.100       any        testing  15.15.165.103      22      TCP  10.10.10.15    CONTEXT1  acl-50-VLAN
    2022-06-13 13:40:49,830 ---------------
    2022-06-13 13:40:49,830 INFO:__main__:generate_commands:$    object groupings completed -- begin generating commands
    2022-06-13 13:40:49,830 ---------------
    2022-06-13 13:40:49,831 INFO:Commands:increment_settings:$ incremented settings : object_network_index in settings.json
    2022-06-13 13:40:49,835 INFO:Commands:generate_to_address:$ generated 9 To_Address grouped commands : 
    ..............................changeto context CONTEXT1
    ..............................conf t
    ..............................object-group network network_object_252
    ..............................network-object host 15.15.165.201
    ..............................network-object host 15.15.165.202
    ..............................network-object host 15.15.165.203
    ..............................exit
    ..............................access-list acl-50-VLAN line 1 extended permit TCP host 50.50.50.200 object-group network_object_252 eq 443
    ..............................
    
    2022-06-13 13:40:49,836 INFO:Commands:increment_settings:$ incremented settings : object_network_index in settings.json
    2022-06-13 13:40:49,840 INFO:Commands:generate_to_address:$ generated 9 To_Address grouped commands : 
    ..............................changeto context CONTEXT1
    ..............................conf t
    ..............................object-group network network_object_253
    ..............................network-object host 15.15.165.101
    ..............................network-object host 15.15.165.102
    ..............................network-object host 15.15.165.103
    ..............................exit
    ..............................access-list acl-50-VLAN line 1 extended permit TCP host 50.50.50.100 object-group network_object_253 eq 22
    ..............................


