# AM-treetool
A shell script tool to export, import, list, describe, and prune Forgerock Identity Cloud and ForgeRock Access Management journeys/trees.

## Description:
A shell script to export authentication journeys/trees from any realm to standard output or a file and import into any realm from standard input or a file. The tool includes scripts referenced by scripted decision nodes and when used with Identity Cloud or ForgeOps, the tool also includes Email Templates referenced by the Email Template or Email Suspend nodes. Requires curl, jq, and uuidgen to be installed and available.


## Usage: 
    % ./amtree.sh ( -e | -E | -i | -I | -l | -d | -P ) [options]

    Export, import, list, describe, and prune authentication journeys/trees in the
    ForgeRock Identity Platform. The utility supports Identity Cloud, ForgeOps
    (CDM/CDK) deployments, and classic deployments.

    Actions/tasks (must specify only one):
    -e         Export an authentication journey/tree.
    -E         Export all the journeys/trees in a realm.
    -S         Export all the journeys/trees in a realm as separate files of the
               format <journey/tree name>.json.
    -s         Import all the journeys/trees in the current directory (*.json).
    -i         Import an authentication journey/tree.
    -I         Import all the journeys/trees in a realm.
    -d         If -h is supplied, describe the journey/tree indicated by -t, or
               all journeys/trees in the realm if no -t is supplied, otherwise
               describe the journey/tree export file indicated by -f.
    -D         If -h is supplied, describe all the journeys/trees in the realm,
               otherwise describe *.json files in the current directory.
    -l         List all the journeys/trees in a realm.
    -P         Prune orphaned configuration artifacts left behind after deleting
               authentication trees. You will be prompted before any irreversible
               operations are performed.
    -z         Login, print versions and tokens, then exit.

    Options:
    -h url     Access Management base URL, e.g.:
               https://openam-volker-dev.forgeblocks.com/am
    -u user    Username to login with. Must be an admin user with appropriate
               rights to manage authentication journeys/trees. For Identity Cloud
               use a tenant admin account if possible.
    -p passwd  Password.
    -r realm   Realm. If not specified, the root realm '/' is assumed. Specify
               realm as '/parent/child'. Note the leading '/'!
    -f file    If supplied, export/list to and import from <file> instead of
               stdout and stdin. For -S, use as file prefix
    -t tree    Specify the name of an authentication journey/tree. Mandatory in
               combination with the following actions: -i, -e, -d.
    -o version Override version. Notation: "X.Y.Z" e.g. "7.1.0"
               Override detected version with any version. This is helpful in
               order to check if journeys in one environment would be compatible
               running in another environment (e.g. in preparation of migrating
               from on-prem to ForgeRock Identity Cloud. Only impacts these
               actions: -d, -l.
    -m type    Override auto-detected deployment type. Valid values for type:
               Classic  - A classic Access Management-only deployment with custom
                          layout and configuration.
               Cloud    - A ForgeRock Identity Cloud environment.
               ForgeOps - A ForgeOps CDK or CDM deployment.
               The detected or provided deployment type controls certain behavior
               like obtaining an Identity Management admin token or not and whether
               to export/import referenced email templates or how to walk through
               the tenant admin login flow of Identity Cloud and skip MFA.

    Run ./amtree.sh without any parameters to display this usage information.

## Examples:
1) Export a journey/tree called "Login" from the root realm to a file:  
    % ./amtree.sh -h https://openam.example.com/openam -u amadmin -p password -e -t Login -f Login.json
  
2) Import a journey/tree into a sub-realm from a file and rename it to "LoginTree":  
    % ./amtree.sh -h https://openam.example.com/openam -u amadmin -p password -i -t LoginTree -f Login.json -r /parent/child
  
3) Export all the journeys/trees from the root realm to a file:  
    % ./amtree.sh -h https://openam.example.com/openam -u amadmin -p password -E -f trees.json
  
4) Export all the journeys/trees from the root realm to separate files in the current directory. 
    % ./amtree.sh -h https://openam.example.com/openam -u amadmin -p password -S  
  
5) Import all the journeys/trees from a file into a sub-realm:  
    % ./amtree.sh -h https://openam.example.com/openam -u amadmin -p password -I -f trees.json -r /parent/child
  
6) Import all the trees(*.json) from the currrent directory into a sub-realm:  
    % ./amtree.sh -h https://openam.example.com/openam -u amadmin -p password -s -r /parent/child  
  
7) Clone a journey/tree "Login" to "ClonedLogin":  
    % ./amtree.sh -h https://openam.example.com/openam -u amadmin -p password -e -t Login | ./amtree.sh -h https://openam.example.com/openam -u amadmin -p password -i ClonedLogin  
  
8) Copy a journey/tree "Login" to "ClonedLogin" in another ForgeRock Identity Platform instance:  
    % ./amtree.sh -h https://openam.example.com/openam -u amadmin -p password -e -t Login | ./amtree.sh -h https://another.domain.org/openam -u amadmin -p password -i ClonedLogin  
  
9) Copy all the journeys/trees from one realm in one ForgeRock Identity Platform instance to another realm in another instance:  
    % ./amtree.sh -h https://openam.example.com/openam -u amadmin -p password -E -r /internal | ./amtree.sh -h https://another.domain.org/openam -u amadmin -p password -I -r /external  
  
10) Pruning:  
    % ./amtree.sh -P -h https://openam.example.com/openam -u amadmin -p password  
    % ./amtree.sh -P -h https://openam.example.com/openam -r /parent/child -u amadmin -p password  
  
   Sample output during pruning:  
       
    Analyzing authentication nodes configuration artifacts...  
       
    Total:    74  
    Orphaned: 37  
     
    Do you want to prune (permanently delete) all the orphaned node instances? (N/y): y  
    Pruning.....................................  
    Done.
  
11) List all the journeys/trees from the root realm to a file or the console:  
    % ./amtree.sh -h https://openam.example.com/openam -u amadmin -p password -l -f trees.txt  
    % ./amtree.sh -h https://openam.example.com/openam -u amadmin -p password -l
  
12) Describe one specific journey/tree export file or all .json files in the current directory:
    If no file name is supplied, describe all json files in the current directory (from -S)
    % ./amtree.sh -d -f tree1.json  
    % ./amtree.sh -D
  
13) Describe one specific journey or all in the realm:
    If no file name is supplied, describe all json files in the current directory (from -S)
    % ./amtree.sh -h https://openam.example.com/openam -u amadmin -p password -d -t tree1
    % ./amtree.sh -h https://openam.example.com/openam -u amadmin -p password -D

## Limitations:
This tool can't export passwords (including API secrets, etc), so these need to be manually added back to an imported tree or alternatively, export the source tree to a file, edit the file to add the missing fields before importing. Any dependencies than scripts and email templates needed for a journey/tree must also exist prior to import, for example inner-trees and custom nodes. Currently, scripts are NOT given a new UUID on import; an option to allow re-UUID-ing scripts might be added in the future.
