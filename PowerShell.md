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

- Get brief information about my main graphics card
```shell
PS H:\> Get-WmiObject Win32_VideoController | Select-Object Name, AdapterCompatibility, VideoProcessor | ConvertTo-Json
{
    "Name":  "Intel(R) Iris(R) Xe Graphics",
    "AdapterCompatibility":  "Intel Corporation",
    "VideoProcessor":  "Intel(R) Iris(R) Xe Graphics Family"
}
```
