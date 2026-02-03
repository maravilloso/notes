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

- Check whether a remote machine port is reachable & open to connect:
```shell
  Test-NetConnection -ComputerName subdn058.prod.axponet.ch -Port 5672
  # Shorter version:
  tnc subdn058.prod.axponet.ch -p 5672

  # Sample output
ComputerName     : subdn058.prod.axponet.ch
RemoteAddress    : 10.54.191.95
RemotePort       : 5672
InterfaceAlias   : Ethernet
SourceAddress    : 10.54.187.125
TcpTestSucceeded : True

```

- Find your public IP address
```shell
  (Invoke-RestMethod ipinfo.io/json).ip
```
