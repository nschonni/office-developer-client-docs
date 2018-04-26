---
title: "Database.CreateQueryDef Method (DAO)"
  
  
manager: soliver
ms.date: 3/9/2015
ms.audience: Developer
 
  
localization_priority: Normal
ms.assetid: 75ee7cac-dcd0-b4c5-b55b-9cbaaae2cbf0
description: "Creates a new QueryDef object."
---

# Database.CreateQueryDef Method (DAO)

Creates a new **[QueryDef](querydef-object-dao.md)** object. 
  
## Syntax

 *expression*  . **CreateQueryDef**( ** *Name* **, ** *SQLText* ** ) 
  
 *expression*  A variable that represents a **Database** object. 
  
### Parameters

|**Name**|**Required/Optional**|**Data Type**|**Description**|
|:-----|:-----|:-----|:-----|
| _Name_ <br/> |Optional  <br/> |**Variant** <br/> |A **Variant** ( **String** subtype) that uniquely names the new **QueryDef**.  <br/> |
| _SQLText_ <br/> |Optional  <br/> |**Variant** <br/> |A **Variant** ( **String** subtype) that is an SQL statement defining the **QueryDef**. If you omit this argument, you can define the **QueryDef** by setting its **[SQL](querydef-sql-property-dao.md)** property before or after you append it to a collection.  <br/> |
   
### Return Value

QueryDef
  
## Remarks

In a Microsoft Access workspace, if you provide anything other than a zero-length string for the name when you create a **QueryDef**, the resulting **QueryDef** object is automatically appended to the **[QueryDefs](querydefs-collection-dao.md)** collection. 
  
If the object specified by  _name_ is already a member of the **QueryDefs** collection, a run-time error occurs. You can create a temporary **QueryDef** by using a zero-length string for the  _name_ argument when you execute the **CreateQueryDef** method. You can also accomplish this by setting the **[Name](querydef-name-property-dao.md)** property of a newly created **QueryDef** to a zero-length string (""). Temporary **QueryDef** objects are useful if you want to repeatedly use dynamic SQL statements without having to create any new permanent objects in the **QueryDefs** collection. You can't append a temporary **QueryDef** to any collection because a zero-length string isn't a valid name for a permanent **QueryDef** object. You can always set the **Name** and **SQL** properties of the newly created **QueryDef** object and subsequently append the **QueryDef** to the **QueryDefs** collection. 
  
To run the SQL statement in a **QueryDef** object, use the **[Execute](querydef-execute-method-dao.md)** or **[OpenRecordset](database-openrecordset-method-dao.md)** method. 
  
Using a **QueryDef** object is the preferred way to perform SQL pass-through queries with ODBC databases. 
  
To remove a **QueryDef** object from a **QueryDefs** collection in a Microsoft Access database engine database, use the **[Delete](querydefs-delete-method-dao.md)** method on the collection. 
  
## Example

This example uses the **CreateQueryDef** method to create and execute both a temporary and a permanent **QueryDef**. The **GetrstTemp** function is required for this procedure to run. 
  
```
Sub CreateQueryDefX() 
 
   Dim dbsNorthwind As Database 
   Dim qdfTemp As QueryDef 
   Dim qdfNew As QueryDef 
 
   Set dbsNorthwind = OpenDatabase("Northwind.mdb") 
 
   With dbsNorthwind 
      ' Create temporary QueryDef. 
      Set qdfTemp = .CreateQueryDef("", _ 
         "SELECT * FROM Employees") 
      ' Open Recordset and print report. 
      GetrstTemp qdfTemp 
      ' Create permanent QueryDef. 
      Set qdfNew = .CreateQueryDef("NewQueryDef", _ 
         "SELECT * FROM Categories") 
      ' Open Recordset and print report. 
      GetrstTemp qdfNew 
      ' Delete new QueryDef because this is a demonstration. 
      .QueryDefs.Delete qdfNew.Name 
      .Close 
   End With 
 
End Sub 
 
Function GetrstTemp(qdfTemp As QueryDef) 
 
   Dim rstTemp As Recordset 
 
   With qdfTemp 
      Debug.Print .Name 
      Debug.Print "  " &amp; .SQL 
      ' Open Recordset from QueryDef. 
      Set rstTemp = .OpenRecordset(dbOpenSnapshot) 
 
      With rstTemp 
         ' Populate Recordset and print number of records. 
         .MoveLast 
         Debug.Print "  Number of records = " &amp; _ 
            .RecordCount 
         Debug.Print 
         .Close 
      End With 
 
   End With 
 
End Function 

```

This example uses the **CreateQueryDef** and **OpenRecordset** methods and the **SQL** property to query the table of titles in the Microsoft SQL Server sample database Pubs and return the title and title identifier of the best-selling book. The example then queries the table of authors and instructs the user to send a bonus check to each author based on his or her royalty share (the total bonus is $1,000 and each author should receive a percentage of that amount). 
  
```
Sub ClientServerX2() 
 
   Dim dbsCurrent As Database 
   Dim qdfBestSellers As QueryDef 
   Dim qdfBonusEarners As QueryDef 
   Dim rstTopSeller As Recordset 
   Dim rstBonusRecipients As Recordset 
   Dim strAuthorList As String 
 
   ' Open a database from which QueryDef objects can be  
   ' created. 
   Set dbsCurrent = OpenDatabase("DB1.mdb") 
 
   ' Create a temporary QueryDef object to retrieve 
   ' data from a Microsoft SQL Server database. 
   Set qdfBestSellers = dbsCurrent.CreateQueryDef("") 
   With qdfBestSellers 
      ' Note: The DSN referenced below must be configured to  
      '       use Microsoft Windows NT Authentication Mode to  
      '       authorize user access to the Microsoft SQL Server. 
      .Connect = "ODBC;DATABASE=pubs;DSN=Publishers" 
      .SQL = "SELECT title, title_id FROM titles " &amp; _ 
         "ORDER BY ytd_sales DESC" 
      Set rstTopSeller = .OpenRecordset() 
      rstTopSeller.MoveFirst 
   End With 
 
   ' Create a temporary QueryDef to retrieve data from 
   ' a Microsoft SQL Server database based on the results from 
   ' the first query. 
   Set qdfBonusEarners = dbsCurrent.CreateQueryDef("") 
   With qdfBonusEarners 
      ' Note: The DSN referenced below must be configured to  
      '       use Microsoft Windows NT Authentication Mode to  
      '       authorize user access to the Microsoft SQL Server. 
      .Connect = "ODBC;DATABASE=pubs;DSN=Publishers" 
      .SQL = "SELECT * FROM titleauthor " &amp; _ 
         "WHERE title_id = '" &amp; _ 
         rstTopSeller!title_id &amp; "'" 
      Set rstBonusRecipients = .OpenRecordset() 
   End With 
 
   ' Build the output string. 
   With rstBonusRecipients 
      Do While Not .EOF 
         strAuthorList = strAuthorList &amp; "  " &amp; _ 
            !au_id &amp; ":  $" &amp; (10 * !royaltyper) &amp; vbCr 
         .MoveNext 
      Loop 
   End With 
 
   ' Display results. 
   MsgBox "Please send a check to the following " &amp; _ 
      "authors in the amounts shown:" &amp; vbCr &amp; _ 
      strAuthorList &amp; "for outstanding sales of " &amp; _ 
      rstTopSeller!Title &amp; "." 
 
   rstTopSeller.Close 
   dbsCurrent.Close 
 
End Sub 
 
```

The following example shows how to create a parameter query. A query named **myQuery** is created with two parameters, named  _Param1_ and  _Param2_. To do this, the **SQL** property of the query is set to a Structured Query Language (SQL) statement that defines the parameters. 
  
 **Sample code provided by:** The [Microsoft Access 2010 Programmer's Reference](http://www.wrox.com/WileyCDA/WroxTitle/Access-2010-Programmer-s-Reference.productCd-0470591668.mdl) | [About the Contributors](#AboutContributors)
  
```
Sub CreateQueryWithParameters()
    Dim dbs As DAO.Database
    Dim qdf As DAO.QueryDef
    Dim strSQL As String
    Set dbs = CurrentDb
    Set qdf = dbs.CreateQueryDef("myQuery")
    Application.RefreshDatabaseWindow
    strSQL = "PARAMETERS Param1 TEXT, Param2 INT; "
    strSQL = strSQL &amp; "SELECT * FROM [Table1] "
    strSQL = strSQL &amp; "WHERE [Field1] = [Param1] AND [Field2] = [Param2];"
    qdf.SQL = strSQL
    qdf.Close
    Set qdf = Nothing
    Set dbs = Nothing
End Sub
```

## About the Contributors
<a name="AboutContributors"> </a>

Wrox Press is driven by the Programmer to Programmer philosophy. Wrox books are written by programmers for programmers, and the Wrox brand means authoritative solutions to real-world programming problems. 
  
