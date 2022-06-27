# Exercise 1 - Reconnaissance

## Tools

使用attacker-tools2底下的腳本PowerView

PowerView: [https://github.com/PowerShellMafia/PowerSploit/blob/dev/Recon/PowerView.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/dev/Recon/PowerView.ps1)

## Exercise

```powershell
cd C:\users\john\Desktop\ADSec\exercises\attacker-tools\attacker-tools-2\
cat -raw ".\PowerView.ps1" | iex
```

取得Domain基本資訊以及網域控制站資訊

```powershell
Get-Domain
Get-DomainController
```

列舉網域內所有的電腦
```powershell
Get-DomainComputer
```

列舉網域內電腦的computer account和DNS名稱以及什麼時候創建
```powershell
Get-DomainComputer | select samaccountname,dnshostname,whencreated | Format-Table
```

列舉所有Domain user
```powershell
Get-DomainUser
```

列舉有Domain admin權限的Domain user還有詳細資訊
```powershell
Get-DomainUser | ? {$_.memberof -like "*Domain Admins*"}
```

列舉有Domain admin權限的Domain user的名稱
```powershell
Get-DomainUser | ? {$_.memberof -like "*Domain Admins*"} | select samaccountname
```

取得非預設的群組資訊
```powershell
Get-DomainGroup | ? { $_.distinguishedname -notlike "*CN=Users*" -and $_.distinguishedname -notlike "*CN=Builtin*"} | select samaccountname,description
```