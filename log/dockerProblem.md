# DockerProblem

## Virtual Machine Platform is not enabled. Enable it using the following PowerShell script (in an administrative PowerShell) and then restart your computer before using Docker Desktop:

Enable-WindowsOptiona

```
dism.exe /Online /Enable-Feature:Microsoft-Hyper-V /All
```

