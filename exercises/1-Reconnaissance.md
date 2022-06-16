# Exercise 1 - Reconnaissance

## Tools

使用attacker-tool2底下的腳本PowerView

PowerView: [https://github.com/PowerShellMafia/PowerSploit/blob/dev/Recon/PowerView.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/dev/Recon/PowerView.ps1)

## Exercise


```
cd C:\attacker-tools
cat -raw ".\PowerView.ps1" | iex
```

取得Domain基本資訊以及網域控制站資訊

```
Get-Domain
Get-DomainController
```

列舉網域內所有的電腦
```
Get-DomainComputer
```

列舉網域內電腦的computer account和DNS名稱以及什麼時候創建
```
Get-DomainComputer | select samaccountname,dnshostname,whencreated | Format-Table
```

列舉所有Domain user
```
Get-DomainUser
```

列舉有Domain admin權限的Domain user還有詳細資訊
```
Get-DomainUser | ? {$_.memberof -like "*Domain Admins*"}
```

列舉有Domain admin權限的Domain user的名稱
```
Get-DomainUser | ? {$_.memberof -like "*Domain Admins*"} | select samaccountname
```

取得非預設的群組資訊
```
Get-DomainGroup | ? { $_.distinguishedname -notlike "*CN=Users*" -and $_.distinguishedname -notlike "*CN=Builtin*"} | select samaccountname,description
```

## Questions

- How many computers are in the domain and what OS are they running on?
- How many user objects are in the domain? Write a powershell query to list all user in table form showing only the attributes samaccountname, displayname, description and last password change.
- How many group objects are in the domain and who is a member of which group? Write a powershell query that stores the result in a csv file.
- Think of simple ways to identify custom admin groups and get a list of only (!) those custom groups. Format the output as a table
- Who is a member of the custom admin group you found and when was his password last set?
- Think of simple ways to identify service accounts in the domain? Write a powershell query that lists all service accounts based on the pattern you came up with.