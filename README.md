# LiveFreeOrDICOM
DICOM to SQL DVH Database

This code is intended for Radiation Oncology departments to build a SQL database of DVH's from DICOM files (Plan, Structure, Dose).
This is a work in progress.  This file will eventually contain instructions for an end-user.


### SQL Database format for this project
This code is being built assuming the database is MySQL.  There will be a master table called 'PatientPlans'
in the database 'DVH'.  This table contains the following data:

Field | Type
----- | ----
PatientUID | bigint(12) unsigned zerofill
MRN | bigint(12) unsigned zerofill
PlanID | tinyint(4) unsigned zerofill
Birthdate | date
Age | tinyint(3) unsigned
Sex | char(1)
PlanDate | date
RadOnc | varchar(3)
TxSite | varchar(100)
RxDose | float
Fractions | tinyint(3) unsigned
Modality | varchar(20)
MUs | int(6) unsigned
ROITableUID | varchar(19)

PatientUID is generated based on existing data in the SQL database.  PlanID is is based on the plans currently associated with
the PatientUID (e.g., no other plans with PatientUID then PlanID = 0001). ROITableUID is  'ROI' + PatientUID + PlanID (e.g., PatientUID =
000111222333, PlanID = 0005, then ROITableUID = ROI0001112223330005.

DICOM files do not explicitly contain prescriptions or treatment sites.  This code requires that the plan name follow a specific format.
The plan name populated in the treatment planning system should be '[Tx Site] [fractions] x [Dose]Gy'.  For example, an a treatment of
50Gy in 5 fractions to the right lung would have a plan name of 'Lung R 5 x 10Gy'.  The code will parse the fractional dose by detecting
the number between the final space and prior to 'Gy'.  The number of fractions will be parsed by detecting the integer prior to ' x ' and
after the prior space.  The remainder will be the treatment site.  Please see section about treatment site requirements (although, they
are soft requirements... the code will work but your SQL datebase will not be particularly useful).

All other data in the above table is populated from DICOM RT Plan file using pydicom's dicom.read_file function to parse the data.

Each ROITableUID is the table name for associated DVH data.  ROI tables follow this format:

Field | Type
----- | ----
Name | varchar(20) 
Type | varchar(20) 
Volume | double      
MinDose | double      
MeanDose | double      
MaxDose | double      
DoseBinSize | float       
VolumeString | mediumtext  
VolumeUnits | varchar(20) 


All data is populated from a combination of DICOM RT strucutre and dose files using dicompyler code.  VolumeString is a comma separated
value string, when parsed generates a vector for the DVH y-values.  The x-values are to be generated as a vector of equal length to the
y-axis with equally spaced values based on the DoseBinSize (e.g., VolumeString = '100,100,90,50,20,0' and DoseBinSize = 1.0 then
x-axis vector would equal [0.5 1.5 2.5 3.5 4.5 5.5]).

### Code organization
*DICOM_to_Python*  
This code contains functions that read dicom files and generate python objects containing the data required for input into the
SQL database.  There is no explicit user input.  All data is pulled from DICOM files.  The few values that DICOM files to not explicitly
record are entered in the Plan Name assuming the format '[Tx Site] [fractions] x [Dose]Gy.

*SQL_Tools*  
This code handles all communication with SQL with MySQL Connector.  No DICOM files are used in this code and require the python objects
generated by DICOM_to_Python functions.

*DICOM_to_SQL*  
This has the simple objective of writing to SQL Database wiht only the DICOM file paths as the input.
Currently this code is the least developed as it depends heavily on the previous two codes.

### To Do List
* Write DICOM_to_Python functions to
  * calculate total MU of a plan from DICOM RT Plan

* Write SQL_Tools fuctions to
  * determine Patient_UID for current DICOM import
  * determine PlanID for current DICOM import
  * determine ROI_UID for current DICOM import
  * delete an ROI Table
  * remove a plan row from PatientPlans master table

* Write DICOM pre-import validation function
  * Ensure proper data is populated in DICOM files prior to import
  * Input will be a folder, function will find file path for all needed DICOM files

* Adjust Connect_to_SQL() from SQL_Tools to read a config file for SQL server login information


### References
Built on these Python libraries

pydicom  
https://github.com/darcymason/pydicom

dicompyler  
https://github.com/bastula/dicompyler

