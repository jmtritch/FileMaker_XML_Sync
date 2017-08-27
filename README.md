# FileMaker XML Sync
The included Filemaker database syncs between a master database and deployed offline copies.  It uses a suite of custom functions, scripts and a few tricks within layouts to achieve syncing without the need of plugins.  My office works with rural communities where there is limited or no internet access, and this system was developed to sync between iPads collecting data in the field and the master system in our office.  This project builds on [fileMakerhacks XML Wrap and Unwrap Technique](https://filemakerhacks.com/2014/11/16/the-xml-wrap-and-unwrap-technique/) to include not only adding new records, but also updating existing records and deleting records.

There are some proprietary systems that are available for purchase, but none of them fit with our particular use case.  So I developed our own syncing system that I believe is fairly easy to adapt to any existing FileMaker system.

## Example XML_Sync.fmp12
Before going through the full syncing process, I let's review the example system that demonstrates how the syncing process works.  This sample is simply a list of books.  It contains an Authors table and a Books table.  The Authors tables has a one-to-many relationship to the Book table.  i.e. One author can write multiple books. (I know that technically a single book can be written by two individuals, but for simplicity let's assume the former.)

![screenshot](http://i.imgur.com/7yAI6mM.png)

## The Syncing Process

Please note that although I am using the word sync, the syncing is really only one way process.  Each deployed system exports all updates using an XML file, and the master system imports that XML file, incorporating the updates.

Any updates within the master database are not synced with the deployed copy(s).  Instead, after the master system incorporates updates from the deployed systems, it creates a copy of itself, which must then replace the deployed systems.  The process:

1. The master database creates a copy of itself
1. The copy is deployed onto remote machines(s) (E.g. computer, iPad, iPhone, etc.)
1. The master system and remote system(s) are updated independently
1. The remote system(s) export updates, including new records, updated records, and deleted records, using an XML file
1. The master system imports the XML file and incorporates these changes
1. The master system creates a copy of itself
1. The copy is redeployed onto remote machine(s) to replace the existing copies
1. The master system and remote system(s) are updated independently
1. The looping continues...

## Incorporating the Syncing Process Into Your Projects

It is fairly simple to incorporate the syncing process into your projects, but you will need a copy of FileMaker Pro Advanced since you will need to copy some custom functions.

To setup syncing in your system you will need to do the following, and it's best to follow the order:

1. From the XML_Sync.fmp12 file, copy the following tables and associated layouts to your database:
   * SyncLog - Log of syncs (Imports, Exports, Deployments)
   * SyncDetailedLog - Individual actions within a sync
   * DeleteLog - Log of deleted records
1. Copy all of the custom functions from XML_Sync.fmp12
   * AsciiToBase64 - Converts ASCII text to Base64 text
      * AssiiToBin - Converts ASCII text to binary text
      * BinToBase64 - Converts binary text to Base64 text
   * Base64ToAscii - Converts Base64 text to ASCII text
      * Base64ToBin - Converts Base64 text to binary text
      * BinToAscii - Converts binary text to ASCII text
         * ByteToInt - Converts binary byte (Set of eight 1s and 0s) to integer
   * ExtractXML - Extracts data between XML tags
   * PassXML - Creates XML tag with corresponding data
1. Copy all of the scripts in the __Sync__ folder from XML_Sync.fmp12
   * __Import__ folder
      * Import XML - Imports updates from the selected XML encoded data file
         * _Base64 To ASCII_ - Converts a Base64 text to ASCII text
         * _Import XML Records for Single Table_ - Imports XML encoded record updates into a single table
            * _Import XML Single XML Record_ - Imports a single XML encoded record update
         * _Delete Records Listed in Delete Log_ - Self explanatory
   * __Export__ folder
      * _Export XML_ - Exports XML formatted new, updated and deleted records into a file\
         * _ASCII To Base64_ - Converts ASCII text to Base64 text
   * __Deploy__ folder
      * _Deploy Database_ - Creates a copy of the database using the current date for deploying on remote computers/iPads
   * __SyncLog__ folder
      * _Sort Sync Log_ - Sorts the DetailedSyncLog layout from newest sync to oldest sync
      * _Delete Sync Log_ - Deletes the SyncLog when the Delete button is clicked in the DetailedSyncLog layout
      * _Log Sync Action_ - Logs a sync action to the DetailedSyncLog table
   * _Delete and Log Record_ - Delete a record and logs it to the DeleteLog
1. Add a field named _id_ of type _Text_ to __all__ tables
   * Set this field value automatically using Auto-Enter --> Calculation --> `Get ( UUID )` function
   * Ensure that you set a value of the _id_ field for all existing records
1. Add a field named _modTS_ of type _Timestamp_ to __all__ tables
   * Set automatically using the Auto-Enter --> Modification --> "Timestamp (Date and Time)"
   * Ensure that you set a value for all existing records
1. All tables need a layout with the same name as the table name.
1. All table layouts also need to include the _id_ field
   * The _id_ field in the layout must have Find Mode __checked__ and Browse Mode __unchecked__ within Inspector --> Data --> Field Entry.  This ensures that the _Quick Find_ script steps work when importing and exporting, while also preventing the user from changing the _id_.
1. Only Delete records using the _Delete and Log Record_ script
   * There needs to be a log of deleted records in order for the sync process to delete records on the master system that were deleted on the remote system(s).

## How to Sync
This section will detail how to sync between deployed copies and the master database.  To understand the process, we will start the journey on a deployed copy.  We will:
1. Export the updates to XML, Import the 

### Import XML

More coming...

### Deploy

More coming...
