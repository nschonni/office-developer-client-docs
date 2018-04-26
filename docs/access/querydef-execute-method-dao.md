---
title: "QueryDef.Execute Method (DAO)"
 
 
manager: soliver
ms.date: 3/9/2015
ms.audience: Developer
ms.topic: reference
f1_keywords:
- dao360.chm1052884
  
localization_priority: Normal
ms.assetid: ad9e859e-c6fe-496c-a1f2-a000cf4bebcc
description: "Executes an SQL statement on the specified object."
---

# QueryDef.Execute Method (DAO)

Executes an SQL statement on the specified object.
  
## Syntax

 *expression*  . **Execute**( ** *Options* ** ) 
  
 *expression*  A variable that represents a **QueryDef** object. 
  
### Parameters

|**Name**|**Required/Optional**|**Data Type**|**Description**|
|:-----|:-----|:-----|:-----|
| _Options_ <br/> |Optional  <br/> |**Variant** <br/> ||
   
## Remarks

You can use the following **[RecordsetOptionEnum](recordsetoptionenum-enumeration-dao.md)** constants for  _Options_.
  
|**Constant**|**Description**|
|:-----|:-----|
|**dbDenyWrite** <br/> |Denies write permission to other users (Microsoft Access workspaces only).  <br/> |
|**dbInconsistent** <br/> |(Default) Executes inconsistent updates (Microsoft Access workspaces only).  <br/> |
|**dbConsistent** <br/> |Executes consistent updates (Microsoft Access workspaces only).  <br/> |
|**dbSQLPassThrough** <br/> |Executes an SQL pass-through query. Setting this option passes the SQL statement to an ODBC database for processing (Microsoft Access workspaces only).  <br/> |
|**dbFailOnError** <br/> |Rolls back updates if an error occurs (Microsoft Access workspaces only).  <br/> |
|**dbSeeChanges** <br/> |Generates a run-time error if another user is changing data you are editing (Microsoft Access workspaces only).  <br/> |
|**dbRunAsync** <br/> |Executes the query asynchronously (ODBCDirect Connection and **QueryDef** objects only).  <br/> |
|**dbExecDirect** <br/> |Executes the statement without first calling **SQLPrepare** ODBC API function (ODBCDirect Connection and **QueryDef** objects only).  <br/> |
   
> [!NOTE]
> ODBCDirect workspaces are not supported in Microsoft Access 2013. Use ADO if you want to access external data sources without using the Microsoft Access database engine. 
  
> [!NOTE]
> The constants **dbConsistent** and **dbInconsistent** are mutually exclusive. You can use one or the other, but not both in a given instance of **OpenRecordset**. Using both **dbConsistent** and **dbInconsistent** causes an error. 
  
Use the **[RecordsAffected](querydef-recordsaffected-property-dao.md)** property of the **[Connection](connection-object-dao.md)**, **[Database](database-object-dao.md)**, or **[QueryDef](querydef-object-dao.md)** object to determine the number of records affected by the most recent **[Execute](querydef-execute-method-dao.md)** method. For example, **RecordsAffected** contains the number of records deleted, updated, or inserted when executing an action query. When you use the **Execute** method to run a query, the **RecordsAffected** property of the **QueryDef** object is set to the number of records affected. 
  
In a Microsoft Access workspace, if you provide a syntactically correct SQL statement and have the appropriate permissions, the **Execute** method won't fail — even if not a single row can be modified or deleted. Therefore, always use the **dbFailOnError** option when using the **Execute** method to run an update or delete query. This option generates a run-time error and rolls back all successful changes if any of the records affected are locked and can't be updated or deleted. 
  
In earlier versions of the Microsoft Jet database engine, SQL statements were automatically embedded in implicit transactions. If part of a statement executed with **dbFailOnError** failed, the entire statement would be rolled back. To improve performance, these implicit transactions were removed starting with version 3.5. If you are updating older DAO code, be sure to consider using explicit transactions around **Execute** statements. 
  
For best performance in a Microsoft Access workspace, especially in a multiuser environment, nest the **Execute** method inside a transaction. Use the **[BeginTrans](workspace-begintrans-method-dao.md)** method on the current **[Workspace](workspace-object-dao.md)** object, then use the **Execute** method, and complete the transaction by using the **[CommitTrans](workspace-committrans-method-dao.md)** method on the **Workspace**. This saves changes on disk and frees any locks placed while the query is running. 
  
## Example

This example demonstrates the **Execute** method when run from both a **QueryDef** object and a **Database** object. The **ExecuteQueryDef** and **PrintOutput** procedures are required for this procedure to run. 
  
```
Sub ExecuteX() 
 
   Dim dbsNorthwind As Database 
   Dim strSQLChange As String 
   Dim strSQLRestore As String 
   Dim qdfChange As QueryDef 
   Dim rstEmployees As Recordset 
   Dim errLoop As Error 
 
   ' Define two SQL statements for action queries. 
   strSQLChange = "UPDATE Employees SET Country = " &amp; _ 
      "'United States' WHERE Country = 'USA'" 
   strSQLRestore = "UPDATE Employees SET Country = " &amp; _ 
      "'USA' WHERE Country = 'United States'" 
 
   Set dbsNorthwind = OpenDatabase("Northwind.mdb") 
   ' Create temporary QueryDef object. 
   Set qdfChange = dbsNorthwind.CreateQueryDef("", _ 
      strSQLChange) 
   Set rstEmployees = dbsNorthwind.OpenRecordset( _ 
      "SELECT LastName, Country FROM Employees", _ 
      dbOpenForwardOnly) 
 
   ' Print report of original data. 
   Debug.Print _ 
      "Data in Employees table before executing the query" 
   PrintOutput rstEmployees 
    
   ' Run temporary QueryDef. 
   ExecuteQueryDef qdfChange, rstEmployees 
    
   ' Print report of new data. 
   Debug.Print _ 
      "Data in Employees table after executing the query" 
   PrintOutput rstEmployees 
 
   ' Run action query to restore data. Trap for errors, 
   ' checking the Errors collection if necessary. 
   On Error GoTo Err_Execute 
   dbsNorthwind.Execute strSQLRestore, dbFailOnError 
   On Error GoTo 0 
 
   ' Retrieve the current data by requerying the recordset. 
   rstEmployees.Requery 
 
   ' Print report of restored data. 
   Debug.Print "Data after executing the query " &amp; _ 
      "to restore the original information" 
   PrintOutput rstEmployees 
 
   rstEmployees.Close 
    
   Exit Sub 
    
Err_Execute: 
 
   ' Notify user of any errors that result from 
   ' executing the query. 
   If DBEngine.Errors.Count > 0 Then 
      For Each errLoop In DBEngine.Errors 
         MsgBox "Error number: " &amp; errLoop.Number &amp; vbCr &amp; _ 
            errLoop.Description 
      Next errLoop 
   End If 
    
   Resume Next 
 
End Sub 
 
Sub ExecuteQueryDef(qdfTemp As QueryDef, _ 
   rstTemp As Recordset) 
 
   Dim errLoop As Error 
    
   ' Run the specified QueryDef object. Trap for errors, 
   ' checking the Errors collection if necessary. 
   On Error GoTo Err_Execute 
   qdfTemp.Execute dbFailOnError 
   On Error GoTo 0 
 
   ' Retrieve the current data by requerying the recordset. 
   rstTemp.Requery 
    
   Exit Sub 
 
Err_Execute: 
 
   ' Notify user of any errors that result from 
   ' executing the query. 
   If DBEngine.Errors.Count > 0 Then 
      For Each errLoop In DBEngine.Errors 
         MsgBox "Error number: " &amp; errLoop.Number &amp; vbCr &amp; _ 
            errLoop.Description 
      Next errLoop 
   End If 
    
   Resume Next 
 
End Sub 
 
Sub PrintOutput(rstTemp As Recordset) 
 
   ' Enumerate Recordset. 
   Do While Not rstTemp.EOF 
      Debug.Print "  " &amp; rstTemp!LastName &amp; _ 
         ", " &amp; rstTemp!Country 
      rstTemp.MoveNext 
   Loop 
 
End Sub 

```

The following example shows how to execute a parameter query. The **Parameters** collection is used to set the **Organization** parameter of the **myActionQuery** query before the query is executed. 
  
 **Sample code provided by:** The [Microsoft Access 2010 Programmer's Reference](http://www.wrox.com/WileyCDA/WroxTitle/Access-2010-Programmer-s-Reference.productCd-0470591668.mdl) | [About the Contributors](#AboutContributors)
  
```
Public Sub ExecParameterQuery()
    Dim dbs As DAO.Database
    Dim qdf As DAO.QueryDef
    Set dbs = CurrentDb
    Set qdf = dbs.QueryDefs("myActionQuery")
    'Set the value of the QueryDef's parameter
    qdf.Parameters("Organization").Value = "Microsoft"
    'Execute the query
    qdf.Execute dbFailOnError
    'Clean up
    qdf.Close
    Set qdf = Nothing
    Set dbs = Nothing
End Sub
```

## About the Contributors
<a name="AboutContributors"> </a>

Wrox Press is driven by the Programmer to Programmer philosophy. Wrox books are written by programmers for programmers, and the Wrox brand means authoritative solutions to real-world programming problems. 
  
