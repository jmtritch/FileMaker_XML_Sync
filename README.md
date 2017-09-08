# FileMaker XML Sync
The included Filemaker database syncs between a master database and deployed offline copies.  It uses a suite of custom functions, scripts and a few tricks within layouts to achieve syncing without the need of plugins.  My office works with rural communities where there is limited or no internet access, and this system was developed to sync between iPads collecting data in the field and the master database in our office.  This project builds on [fileMakerhacks XML Wrap and Unwrap Technique](https://filemakerhacks.com/2014/11/16/the-xml-wrap-and-unwrap-technique/) to include not only inserting new records, but also updating existing records and deleting records.

There are some proprietary systems that are available for purchase, but none of them fit with our particular use case.  So I developed our own syncing system that I believe is fairly easy to adapt to any existing FileMaker database.

## Example XML_Sync.fmp12
The example database, [XML_Sync.fmp12](https://github.com/jmtritch/FileMaker_XML_Sync/blob/master/XML_Sync.fmp12), demonstrates how the syncing process works using a simple database of Authors and Books.  The __Authors__ table has a one-to-many relationship to the __Books__ table.  i.e. One author can write multiple books.
### The Layouts
There are 5 layouts in the example:
1. Books - Lists the Books
   * Opened with the __Books__ button in the top menu
       ![Books](http://i.imgur.com/7yAI6mM.png)
       * __New Author__ button opens a popup window to insert new author
       * __New Book__ button inserts a new book
       * __Delete__ button deletes the book and logs it in the __DeleteLog__ table
       
1. Authors - Lists the Authors
   * Opened with the __Authors__ button in the top menu
       ![Authors](http://i.imgur.com/haY2Yvx.png)
       * __New Author__ button inserts a new author
       * __Delete__ button deletes the author (and the author's books) and logs it in the __DeleteLog__ table
       
1. SyncLog - Lists the sync sessions
   * No associated button - can only be opened from the Layout dropdown menu
       ![Sync Log](http://imgur.com/RFAJ8ua.png)
       * __Details__ button opens the SyncLogDetails layout and shows the details of the sync
       * __Delete__ button deletes the sync session (and its sync details) and logs it in the __DeleteLog__ table
       
1. SyncLogDetailed - Lists the detailed sync steps and groups them by sync session
   * Opened with the __Sync__ button in the top menu
       ![Detailed Sync Log](http://imgur.com/ZTrroVx.png)
       * __Export__ button exports records to an XML file, which can be imported into the master database
       * __Import__ button imports records from an XML file, which should be used with the master database
       * __Deploy__ button exports the entire database to be deployed on remote system(s).  This button should be used with the master database
       * __Sync Properties__ button opens a popup window to edit the properties of the sync process
       * __View XML__ button opens a popup window to view the XML data stored in the XML File
       * __Delete__ button deletes the sync session and associated sync details and logs it in the __DeleteLog__ table
       
1. DeleteLog - Lists the deleted records
   * Opened with the __Delete Log__ button in the top menu
       ![Delete Log](http://imgur.com/kz5JGrL.png)
       * __Clear__ deletes all records in the __DeleteLog__ table

1. ImportXml - Table to temporarily store the contents of the imported XML file for syncing
   * No associated button - can only be opened from the Layout dropdown menu
       ![Delete Log](https://imgur.com/cy7peCa.png)


## The Syncing Process

Please note that although I am using the word sync, the syncing is really only one way process.  Each deployed database exports all updates using an XML file, and the master database imports that XML file, incorporating the updates.

Updates performed directly on the master database should not be synced with the deployed copy(s).  Instead, after the master database incorporates updates from the deployed databases, it creates a copy of itself, which replaces the deployed databases.  The process:

1. The master database creates a copy of itself
1. The copy is deployed onto remote machine(s) (E.g. computer, iPad, iPhone, etc.)
1. The master database and remote database(s) are updated independently
1. The remote database(s) export updates, including new records, updated records, and deleted records, using an XML file
1. The master database imports the XML file and incorporates these changes
1. The master database creates a copy of itself
1. The copy is redeployed onto remote machine(s) to replace the existing copies
1. The master database and remote database(s) are updated independently
1. The looping continues...

## How to Sync
This section will detail how to sync between deployed copies and the master database.  There are three parts of the syncing process: Export, Import, and Deploy

### Export XML
Clicking the __Export__ button in the Sync Layout starts the Export XML script. It creates a new Sync Log.  It then searches through all tables and records in the database (Excluding those listed in the Global IGNORETABLES field) and selects any records with a modified timestamp (_modTS_ field) that is newer than the Global LASTSYNC field in the SyncLog table.  This includes records that have been updated, inserted, or deleted (as listed in the DeleteLog table).  When it finds a record to be exported it wraps it in XML using the following format:

```
<table_TableName>
<record>
<fieldName1>Field Value 1</fieldName1>
...
<fieldNameN>Field Value N</fieldNameN>
<containerfieldName><filename>nameOfFile.ext</filename><base64>
...
</base64></containerfieldName>
</record>
<record>
...
</record>
</table_TableName>
...
<table_TableNameN>
...
</table_TableNameN>
```
As it moves through the database wrapping records in XML, the SyncLog stores the number of records updated, inserted or ignored for each table.  When it finishes it stores the XML data into the _xmlData_ field of the SyncLog.  It then writes the XML data to a text file.  To write the XML data to a file the database uses the `Export Field Contents[]` script step and then imports the newly created XML file into the _xmlFile_ container field of the SyncLog.  The user can then export this XML file and send it to the master database using email, a web upload, or whatever method you choose.

Throughout the process the Sync Log shows the user the steps as they happen in real time.

### Import XML
Clicking the __Import__ button in the Sync Layout starts the Import XML script. The script first creates a full backup of the database, and then creates a new Sync Log.  It opens a dialog box for the user to select the XML file to import and reads the XML data from the selected file.  To read the data from the imported XML file the database uses a few tricks.  The file is exported using the `Export Field Contents[]` script step.  Then the contents of the XML file are imported into the __ImportXml__ table line by line using the `Import Records[]` script step.  These records are then combined back into a single field/variable.

Once the XML data is extracted, it checks that it is formatted correctly.  After passing the checks, it clears the delete log.  Now the data is imported.

The script traverses through the XML data extracting all records for each table, i.e. anything between `<table_TableName>` and `</table_TableName>`.  It then extracts the field data for each record, i.e. anything between `<record>` and `</record>`.  Finally, it extracts the field data for the record.

It uses the _id_ field data of the record to search through the master database for a corresponding record.  If there is no existing record in the master database, then it inserts that data as a new record.  If it finds a corresponding record, it checks the modified timestamp, _modTS_, of both the XML record and the master database record.  If the XML record is newer, then it updates the existing record with the XML data.  If the XML record is older, then it ignores the XML data.

After importing all the XML records, it then deletes the corresponding records listed in the DeleteLog.  Throughout the process the Sync Log shows the user the steps as they happen in real time.

### Deploy
Clicking the __Deploy__ button in the Sync Layout starts the Deploy Database script.  The script first creates a full backup of the database, and then creates a new Sync Log.  It clears the delete log, and resets the LASTSYNC global field in the SyncLog table to the current timestamp.  It then saves a compacted copy of the database to the same folder of the master database using the name `DatabaseFileName_Deployed_YY-MM-DD.fmp12`.  Throughout the process the Sync Log shows the user the steps as they happen in real time.

### Sync Properties Popup
The __Sync Properties__ popup is used to set various properties of the sync process.
![Sync Properties](http://imgur.com/0YFePTO.png)
* The _Ignore Tables_ field holds table names that should be ignored during the syncing process.  Each table name must be written on its own line.
* The _Last Sync_ field stores the timestamp of the last sync.  This timestamp is used to know which records to include during the Export XML process.
* The _Include Global Fields_ is a checkbox the toggles whether the Import/Exprot XML processes should include global fields.

## Incorporating the Syncing Process Into Your Projects
It is fairly simple to incorporate the syncing process into your projects, but you will need a copy of FileMaker Pro Advanced since you will need to copy some custom functions.

To setup syncing in your database you will need to do the following, and it's best to follow the order:

1. From the XML_Sync.fmp12 file, copy the following tables and associated layouts to your database:
   * SyncLog - Log of syncs (Imports, Exports, Deployments)
   * SyncDetailedLog - Individual actions within a sync
   * DeleteLog - Log of deleted records
1. Copy both of the custom functions from XML_Sync.fmp12
   * ExtractXML - Extracts content between XML tags
   * PassXML - Creates XML tag with content
1. Copy all of the scripts in the __Sync__ folder from XML_Sync.fmp12
   * __Import__ folder
      * Import XML - Imports updates from the selected XML encoded data file
         * _Import XML Records for Single Table_ - Imports XML encoded record updates into a single table
            * _Import XML Single XML Record_ - Imports a single XML encoded record update
         * _Delete Records Listed in Delete Log_ - Self explanatory
   * __Export__ folder
      * _Export XML_ - Exports XML formatted new, updated and deleted records into an XML file
   * __Deploy__ folder
      * _Deploy Database_ - Creates a copy of the database using the current date for deploying on remote computers/iPads
   * __SyncLog__ folder
      * _Sort Sync Log_ - Sorts the DetailedSyncLog layout from newest sync to oldest sync
      * _Delete Sync Log_ - Deletes the SyncLog when the Delete button is clicked in the DetailedSyncLog layout
      * _Log Sync Action_ - Logs a sync action to the DetailedSyncLog table
   * _Delete and Log Record_ - Delete a record and logs it to the DeleteLog
1. Add a field named _id_ of type _Text_ to __all__ of your existing tables
   * Set this field value automatically using Auto-Enter --> Calculation --> `Get ( UUID )` function
   * Ensure that you set a value of the _id_ field for all existing records
1. Add a field named _modTS_ of type _Timestamp_ to __all__ of your existing tables
   * Set automatically using the Auto-Enter --> Modification --> "Timestamp (Date and Time)"
   * Ensure that you set a value for all existing records
1. All tables need a layout with the same name as the table name.
1. All table layouts also need to include the _id_ field
   * The _id_ field in the layout must have Find Mode __checked__ and Browse Mode __unchecked__ within Inspector --> Data --> Field Entry.  This ensures that the _Quick Find_ script steps work when importing and exporting, while also preventing the user from changing the _id_.
1. Only Delete records using the _Delete and Log Record_ script
   * There needs to be a log of deleted records in order for the sync process to delete records in the master database that were deleted on the deployed database(s).

