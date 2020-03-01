## About

This script is not supported by Checkmarx and edge cases are not tested, use it as your own.

This query enables Checkmarx to scan for common keys and tokens often found in GIT repositories.  It is based off regex's created by [GitLeaks](https://github.com/zricethezav/gitleaks) and [TruffleHog](https://github.com/dxa4481/truffleHog).

## Import the Query directly into Checkmarx

The file [gitleaks_query.xml](gitleaks_query.xml) can directly be imported into Checkmarx.  Please follow these [instructions](https://checkmarx.atlassian.net/wiki/spaces/KC/pages/91029540/Query+Viewer+v8.4.1+to+v8.9.0) on how to import queries.

Note: [gitleaks_query.xml](gitleaks_query.xml) will only import git leaks for the language Python.  If you wish to add another language, edit the xml file and make two changes:

1. Rename the value for LanguageName
2. Rename the value for Language

To retrieve your language to id mapping, please execute the following SQL statement:

```javascript
/****** Script for SelectTopNRows command from SSMS  ******/
SELECT TOP (1000) [Id]
      ,[Name]
  FROM [CxDB].[Config].[CxProgramLanguages]
```

An example of the programming language chart will be (but might differ for each customer):

| Id     | Name       |
| ------ | ---------- |
| 0      | Unknown    |
| 1      | CSharp     |
| 2      | Java       |
| 4      | CPP        |
| 8      | JavaScript |
| 16     | Apex       |
| 32     | VbNet      |
| 64     | VbScript   |
| 128    | ASP        |
| 256    | VB6        |
| 512    | PHP        |
| 1024   | Ruby       |
| 2048   | Perl       |
| 4096   | Objc       |
| 8192   | PLSQL      |
| 16384  | Python     |
| 32768  | Groovy     |
| 65536  | Scala      |
| 131072 | Go         |
| 262144 | Typescript |
| 524288 | Kotlin     |

## Scan configuration files

If you have configuration files that Checkmarx does not scan for (i.e. YAML or JSON), you need to add the extension to the Checkmarx Database.

Execute the SQL script in file [add_config_files.sql](add_config_files.sql) in the CxDB.  This script will add a stored procedure to the CxDB.  Then you must execute the stored procedure.  Examples:

```javascript
1. exec AddFileExtToLanguage 'Java', 'properties', 'JAVA_XML_EXTENSIONS'
2. exec AddFileExtToLanguage 'Java', 'yaml', 'JAVA_XML_EXTENSIONS'
3. exec AddFileExtToLanguage 'Java', 'yml', 'JAVA_XML_EXTENSIONS'
4. exec AddFileExtToLanguage 'Java', 'json', 'JAVA_XML_EXTENSIONS'
5. exec AddFileExtToLanguage 'JavaScript', 'properties', 'JS_XML_EXTENSIONS'
6. exec AddFileExtToLanguage 'JavaScript', 'yaml', 'JS_XML_EXTENSIONS'
7. exec AddFileExtToLanguage 'JavaScript', 'yml', 'JS_XML_EXTENSIONS'
8. exec AddFileExtToLanguage 'Python', 'properties', 'PYTHON_CORE_EXTENSIONS'
9. exec AddFileExtToLanguage 'Python', 'yaml', 'PYTHON_CORE_EXTENSIONS'
10. exec AddFileExtToLanguage 'Python', 'yml', 'PYTHON_CORE_EXTENSIONS'
```

These commands will add ".properties", ".yml", ".yaml", and ".json" files to the scan so the queries can execute Regex searches on the contents.

Note: Restart all IIS servers.


## Make Changes to the Query using CxAudit

1. In CxAudit, make a group under Corp for each language for which you want to support.  This was tested in Python, but may apply to other languages.  Example: Corp/GitLeaks

2. Under the created group, make a query with a name that corresponds to the name of each .cxq file included with this repo ([CxGitLeaks.cxq](CxAudit/Corp/Python/GitLeaks/CxGitLeaks.cxq) and [CxTruffleHog.cxq](CxAudit/Corp/Python/GitLeaks/CxTruffleHog.cxq)).  Paste the contents of each file into the corresponding query you created.

3. Set the properties on each query as follows:
   
CxTruffleHog:
* Set severity to the level desired
* Choose categories as needed (likely fits in Sensitive Data Exposure categories)
* CWE ID 615
* Executable should be checked

CxGitLeaks:
* Set severity to the level desired
* Choose categories as needed (likely fits in Sensitive Data Exposure categories)
* CWE ID 615
* Executable should be checked

4. Save the queries and exit CxAudit.

## Modify scanning presets on projects

The final is step is enabling the query in the preset you are scanning with.  For more directions, please refer to the [Preset Manager Documentation](https://checkmarx.atlassian.net/wiki/spaces/KC/pages/49250315/Preset+Manager).  It is necessary to check the boxes under Corp/GitLeaks that contains the queries CxGitLeaks and CxTruffleHog.

