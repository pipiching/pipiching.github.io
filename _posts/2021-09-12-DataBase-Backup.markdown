---
layout: post
title:  "Backup DataBase"
date:   2021-09-12 00:00:30 +0800
#categories: 
---

### Backup DataBase

Just backup

```sql
BACKUP DATABASE [dbName] TO DISK = N'target path'
```


### Restore DataBase

```sql
RESTORE DATABASE [dbName] FROM DISK = N'target path'
```

#### Rename DataBase

View the contents of the backup file

```sql
RESTORE FILELISTONLY FROM DISK = N'target path'
```

copy the logical names of the .mdf & .ldf

```sql
RESTORE DATABASE [dbName] FROM DISK = N'target path'
WITH
	MOVE '[mdfName]' TO 'target path',
	MOVE '[ldfName]' TO 'target path'
```