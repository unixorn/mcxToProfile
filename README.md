# mcxToProfile

## Overview

mcxToProfile is a simple command-line utility to create "Custom Settings" Configuration Profiles without the need for the Profile Manager service in Lion Server. It can take input from property list files on disk or directly from a Directory Services node (Local MCX or Open Directory).

Administrators who would like to move from MCX-based management to Profiles may find this tool useful to speed up the process of migrating and testing. Currently it only supports the "Custom Settings" type, as this seems to be the functional equivalent of key-value domain management in Workgroup Manager.

mcxToProfile should function on OS X 10.5 or greater, though it's been mostly only tested on Lion. It also makes use of Greg Neagle's FoundationPlist library from the Munki project, which provides native plist support via the PyObjC bridge framework.

## Example usage

Here's an example:

`./mcxToProfile.py --plist /path/to/a/plist --identifier MyApplicationPrefs`

This will create a .mobileconfig file in the same directory, which is equivalent to a "Custom Settings" profile configured in the Profile Manager web application included in Lion Server.

Here's another example, which will import an already-configured MCX preference defined in an available Directory Services computer object:

`./mcxToProfile.py --dsobject /LDAPv3/od.my.org/ComputerGroups/StandardPreferences --identifier MyBasePreferences`

The `--dsobject` option should work with objects defined in either a LocalMCX or standard Open Directory node on another server. This hasn't been tested with MCX attributes in OpenLDAP or Active Directory.

## Plist input options:

### Once/Often/Always management

One downside to the Profile Manager web GUI is that it does not provide a mechanism to choose the "management frequency" of an MCX-set preference, such as "Once" or "Often". It was discovered very recently (again, by Greg Neagle) that it's possible to regain that functionality, by slightly altering the contents of the .mobileconfig file to match the MCX XML as when created in Workgroup Manager:

- for "Often" behaviour, use the 'Set-Once' key instead of 'Forced' for a domain
- for "Once" behaviour, do this and set an mcx_data_timestamp alongside the mcx_preference_settings which is an NSDate

mcxToProfile provides the same functionality:

`./mcxToProfile.py --plist /path/to/a/plist --identifier MyApplicationPrefs --manage Often`

When using the `--dsobject` option, the `--manage` option isn't used, as this information is already defined in the object.

### Domains

Plist files used for application preferences are typically named by a reverse-domain format, and end in '.plist'. Currently, mcxToProfile will assume that the name portion of the plist file _is_ the domain to be used by MCX. In other words, application preferences won't function if you use something like `--plist my.orgs.office.2011.prefs.plist`, because it will assemble the profile to use the domain 'my.orgs.office.2011.prefs'. If you have collections of default preferences you would like to manage for various applications and system settings, it's best to store these settings in the properly-named plist files.

### ByHost preferences

A plist that contains one of the following patterns in its filename will automatically be configured as a ByHost preference:

- com.org.app.ByHost.plist (the literal string '.ByHost')
- com.org.app.001122aabbcc.plist (a 12-hex-digit MAC address)
- com.org.app.01234567-89AB-CDEF-0123-456789ABCDEF.plist (a hardware UUID)


## Payload Identifiers

The only option required besides `--plist` or `--dsobject` is an identifier. The identifier is crucial: it is what is defined in the toplevel payload's PayloadIdentifier key, and is what would be used to identify the profile to remove using `profiles -R -p [identifier]`. Also, if you attempt to install a profile with the same identifier, it will update the existing profile instead of installing another profile. The newer version could have completely different payloads and a different toplevel PayloadUUID, but it will still replace it.

Specify an identifier using either the `--identifier` or `--identifier-from-profile` options.

As far as I can tell, two profiles with unique toplevel PayloadIdentifiers but matching toplevel PayloadUUIDs will both install successfully. mcxToProfile will always generate unique UUIDs for the toplevel and nested payloads, but because the PayloadIdentifier has to be paid attention to, it's required to manually specify it.


## Other functionality

- Multiple plists can be defined (with `--plist` or `-p`) and they will be combined as individual payloads within the Configuration Profile
- A profile can be made "Always removable" using `--removal-allowed` or `-r` (default is "Never removable")
- An organization name for the profile can be specified using `--organization` or `-g`
- A specific output filename for the .mobileconfig file can be specified using `--output` or `-o`

## To-do

- error handling
- add status output and a verbose mode
- append '.mobileconfig' to filename if not already specified
- potentially 'convert' known existing preference types (loginwindow, dock, etc.) into their native payload types, rather than as Custom Payloads

## Note

I really do not have much experience with Configuration Profiles, and this is a rough first pass at a generic tool that I knew would help me understand better how Configuration Profiles actually work. There may well be fundamental design changes in how this tool should work to make it useful for others, so I welcome anyone's feedback/suggestions/pull requests.

Special thanks to Greg Neagle for some very useful intial feedback, and for adding the --dsimport functionality.