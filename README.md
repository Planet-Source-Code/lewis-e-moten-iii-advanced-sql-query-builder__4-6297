<div align="center">

## Advanced SQL Query Builder


</div>

### Description

This code lets visitors to your site perform complex queries. Users may choose if specific words (or phrases) must or must not match - or if they are optional (default).
 
### More Info
 
asFieldsAry - An array of field names in database to search.

asKeywords - The actual query that the user types to query the database.

Keyword Search Parameters

To find fields that may have a word in them

OR roger

| roger

roger

To find fields that must match a word

AND roger

+ roger

& roger

To find fields that must not match a word

NOT roger

- roger

Also use phrases

+"rogers dog" -cat

+(rogers dog)

It is assumed that the user knows how to build an array of field names and understand that syntax of SQL queries along with how to connect to databases. This procedure has only been tested with SQL Servers and Access databases.

Returns just the SQL arguments within a group that are to be places after the WHERE Clause.

This function uses the RegExp object that was introduced in vbScript 5.0 that came out with Internet Explorer 5.0. The vbScript can be installed without installing Internet Explorer by going to the subdirectory "Scripting" on the microsoft site.


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[Lewis E\. Moten III](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/lewis-e-moten-iii.md)
**Level**          |Intermediate
**User Rating**    |5.0 (45 globes from 9 users)
**Compatibility**  |ASP \(Active Server Pages\)
**Category**       |[Databases](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/databases__4-5.md)
**World**          |[ASP / VbScript](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/asp-vbscript.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/lewis-e-moten-iii-advanced-sql-query-builder__4-6297/archive/master.zip)

### API Declarations

Copyright (C) 1999, Lewis Moten. All rights reserved. Any modifications must be sent to me so that I may review and add them to the procedure as I feel fit.


### Source Code

```
Function BuildQuery(ByRef asFieldAry, ByVal asKeyWords)
	Dim loRegExp			' Regular Expression Object (requires vbScript 5.0)
	Dim loRequiredWords		' Words that MUST match within a search
	Dim loUnwantedWords		' Words that MUST NOT match within a search
	Dim loOptionalWords		' Words that AT LEAST ONE must match
	Dim lsSQL				' Arguments of SQL query that is returned (WHERE __Arguments___)
	Dim lnIndex				' Index of an array
	Dim lsKeyword			' Keyword or Phrase being worked with
	' An error may occur within your script
	' Even if you do not call this function
	' If you do not have vbScript 5.0 installed on your server
	' because of the next line.
	' Create regular Expression
	Set loRegExp = New RegExp
	' Match more then once
	loRegExp.Global = True
	' Every letter is created equal (uppercase-lowercase = same)
	loRegExp.IgnoreCase = True
	' pull out keywords and phrases that MUST match within a search
	loRegExp.Pattern = "((AND|[+&])\s*[\(\[\{""].*[\)\]\}""])|((AND\s|[+&])\s*\b[-\w']+\b)"
	Set loRequiredWords = loRegExp.Execute(asKeywords)
	asKeywords = loRegExp.Replace(asKeywords, "")
	' pull out keywords and phrases that MUST NOT match within a search
	loRegExp.Pattern = "(((NOT|[-])\s*)?[\(\[\{""].*[\)\]\}""])|(((NOT\s+|[-])\s*)\b[-\w']+\b)"
	Set loUnwantedWords = loRegExp.Execute(asKeywords)
	asKeywords = loRegExp.Replace(asKeywords, "")
	' pull out keywords and phrases that must have AT LEAST ONE match within a search
	loRegExp.Pattern = "(((OR|[|])\s*)?[\(\[\{""].*[\)\]\}""])|(((OR\s+|[|])\s*)?\b[-\w']+\b)"
	Set loOptionalWords = loRegExp.Execute(asKeywords)
	asKeywords = loRegExp.Replace(asKeywords, "")
	' If at least 1 required word was found
	If Not loRequiredWords.Count = 0 Then
		' REQUIRED
		' Open a new group
		lsSQL = lsSQL & "("
		' loop through each keyword/phrase
		For lnIndex = 0 To loRequiredWords.Count - 1
			' Pull the keyword out
			lsKeyword = loRequiredWords.Item(lnIndex).Value
			' Strip boolean language
			loRegExp.Pattern = "^(AND|[+&])\s*"
			lsKeyword = loRegExp.Replace(lsKeyword, "")
			loRegExp.Pattern = "[()""\[\]{}]"
			lsKeyword = loRegExp.Replace(lsKeyword, "")
			' Double Quote Keyword
			lsKeyword = Replace(lsKeyword, "'", "''")
			' If we are not working with the first keyword
			If Not lnIndex = 0 Then
				' append logic before the keyword
				lsSQL = lsSQL & " AND "
		 	End If ' Not lnIndex = 0
		 	' Append SQL to search for the keyword within all searchable fields
			lsSQL = lsSQL & "(" & Join(asFieldAry, " LIKE '%" & lsKeyword & "%' OR ") & " LIKE '%" & lsKeyword & "%')"
		Next ' lnIndex
		' Close the group
		lsSQL = lsSQL & ")"
	End If ' Not loRequiredWords.Count = 0
	' If at least 1 optional word was found
	If Not loOptionalWords.Count = 0 Then
		' OPTIONAL
		' If the SQL query is not yet defined
		If lsSQL = "" Then
			' Open a new group
			lsSQL = "("
		' Else SQL query has content
		Else
			' Append logic before the group
			lsSQL = lsSQL & " AND ("
		End If ' lsSQL = ""
		' loop through each keyword/phrase
		For lnIndex = 0 To loOptionalWords.Count - 1
			' Pull the keyword out
			lsKeyword = loOptionalWords.Item(lnIndex).Value
			' Strip Boolean Language
			loRegExp.Pattern = "^(OR|[|])\s*"
			lsKeyword = loRegExp.Replace(lsKeyword, "")
			loRegExp.Pattern = "[()""\[\]{}]"
			lsKeyword = loRegExp.Replace(lsKeyword, "")
			' Double Quote the keyword
			lsKeyword = Replace(lsKeyword, "'", "''")
			' If we are not working with the first keyword
			If Not lnIndex = 0 Then
				' Append Logic before the keyword search
				lsSQL = lsSQL & " OR "
			End If ' Not lnIndex = 0
			' Append SQL to search for the keyword within all searchable fields
			lsSQL = lsSQL & "(" & Join(asFieldAry, " LIKE '%" & lsKeyword & "%' OR ") & " LIKE '%" & lsKeyword & "%')"
		Next ' lnIndex
		' Close the group
		lsSQL = lsSQL & ")"
	End If ' Not loOptionalWords.Count = 0
	' If at least 1 Unwanted word was found
	If Not loUnwantedWords.Count = 0 Then
		' UNWANTED
		' If the SQL query is not yet defined
		If lsSQL = "" Then
			' Open a new group
			lsSQL = "("
		' Else SQL query has content
		Else
			' Append logic before the group
			lsSQL = lsSQL & " AND NOT ("
		End If ' lsSQL = ""
		' loop through each keyword/phrase
		For lnIndex = 0 To loUnwantedWords.Count - 1
			' Pull the keyword out
			lsKeyword = loUnWantedWords.Item(lnIndex).Value
			' Strip Boolean Language
			loRegExp.Pattern = "^(NOT|[-])\s*"
			lsKeyword = loRegExp.Replace(lsKeyword, "")
			loRegExp.Pattern = "[()""\[\]{}]"
			lsKeyword = loRegExp.Replace(lsKeyword, "")
			' Double Quote the keyword
			lsKeyword = Replace(lsKeyword, "'", "''")
			' If we are not working with the first keyword
			If Not lnIndex = 0 Then
				' Append Logic before the keyword search
				lsSQL = lsSQL & " OR "
			End If ' Not lnIndex = 0
			' Append SQL to search for the keyword within all searchable fields
			lsSQL = lsSQL & "(" & Join(asFieldAry, " LIKE '%" & lsKeyword & "%' OR ") & " LIKE '%" & lsKeyword & "%')"
		Next ' lnIndex
		' Close the group
		lsSQL = lsSQL & ")"
	End If ' Not loUnwantedWords.Count = 0
	' If arguments were created
	If Not lsSQL = "" Then
		' Encapsilate Arguments as a group
		' in case other aguments are to be appended
		lsSQL = "(" & lsSQL & ")"
	End If ' Not lsSQL = ""
	' Return the results
	BuildQuery = lsSQL
End Function ' BuildQuery
```

