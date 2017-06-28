# FileMaker XML Sync
The attached Filemaker database is an example system that syncs between a master database and deployed offline copies.  It uses a suite of custom functions, scripts and a few tricks within layouts to achieve syncing without the need of plugins.  My office works with rural communities that have limited or no internet access, and this system was developed to sync between iPads collecting data in the field and the master system in our office.

There are some proprietary systems that are available for purchase, but none of them fit with our particular use case.  So I developed our own syncing system that I believe is fairly easy to adapt to any existing FileMaker system.

## The Syncing Process

Please note that although I am using the word sync, the syncing is really only one way process.  The deployed systems export all updates using an XML file, and the master system imports the XML file and incorporates the changes.

Any updates within the master system are not incorporated into the deployed system.  Instead, after the master system incorporates updates from the deployed systems, it creates a copy of itself, which must then replace the deployed systems.

1. The master system creates a copy of itself (Deploy)
1. The copy is deployed onto remote machine(s) (E.g. computer, iPad, iPhone, etc.)
1. The master system and remote system(s) are updated independently
1. The remote system(s) export updates, including new records, updated records, and deleted records, using an XML file
1. The master system imports the XML file and incorporates these changes
1. The master system creates a copy of itself
1. The copy is redeployed onto remote machine(s) to replace the existing copies
1. The master system and remote system(s) are updated independently
1. The looping continues...

## Incorporating the Syncing Process Into Your Projects

