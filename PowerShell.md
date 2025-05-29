PowerShell
=====

- Batch take ownership of a folder and all its files and subfolders
```shell
  takeown /d S /a /r /f D:\JUEGOS
```

- Reset permissions of a folder or a bunch of files
```shell
  icacls D:\JUEGOS /reset /T /C
```
