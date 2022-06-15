# Lab Setup Guide

## #1 Basic Lab Setup 


### Setup virtual machines manually

使用Vmware架設以下三部機器，並將機器放置同一內網

| Hostname        | OS          | User  | Password |	Specs
| ------------- |-------------| -----|-----|-----|
|AD-DC|	Windows Server 2019	|Administrator|	P@ssw0rd123!!!|	At least 2 Cores, 4 GB RAM
|AD-00	|Windows Server 2019|	Administrator|	P@ssw0rd123!| At least 2 Cores, 4 GB RAM
|AD-01|	Windows Server 2019|	Administrator|	P@ssw0rd123! |At least 2 Cores, 4 GB RAM


每台機器先執行以下步驟

- 將 DNS server的IP指向 ADSEC-DC
- 關閉防火牆 (以管理員模式執行powershell)
   - `Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False`
- 反安裝Windows Defender
   - `Uninstall-WindowsFeature -Name Windows-Defender`

#### Promote domain controller

- 將以下腳本下載至AD-DC [domain-setup-scripts](../lab-setup/domain-setup-scripts) 
- 在AD-DC上使用管理員權限執行以下powershell腳本 [create-domain.ps1](../lab-setup/domain-setup-scripts/create-domain.ps1)，該機器會變成contoso.com的網域控制站 
- 重新啟動AD-DC

- 重新啟動後，確認 **DNS Manager** 和 **Active Directory Users and Computers** 已安裝


#### Join member server

將 ADS-00 and AD-01加入Domain


## #2 Prepare domain

將 [domainprepare.ps1](../lab-setup/domain-setup-scripts/domainprepare.ps1) 和 [domainprepare.ps1](../lab-setup/domain-setup-scripts/domainprepare.ps1)下載到AD-DC上相同資料夾，並使用管理員權限執行

```powershell
cd <whatever-directory-you-choose>
cat -raw .\domainprepare.ps1 | iex
```

本腳本會創建網域內的使用者、群組以及組織(OU)

## #3 Prepare member server

- 將Domain user `john` 加入AD-00的本機管理員群組 
- 將Domain user `blee` 加入AD-01的本機管理員群組 

```powershell
Add-LocalGroupMember -Group "Administrators" -Member "domain\user or group"
```

| Display Name        | samAccountName | Password |	
| ------------- |-------------| -----|
|John Doe|john|P@ssw0rd|
|Bruce Lee|blee|TekkenIsAwesome!|

## #4 Prepare attacker vm (AD-00)
 
 本次實作，我們假設網域使用者 `john`以及所使用的機器 `AD-00` 已經被完全滲透，因此我們可在該機器上安裝內網攻擊
所需要的攻擊程式

1.	下載 [attacker-tools zip files](../exercises/attacker-tools) 並解壓縮到 `ADSEC-00`上的`C:\attacker-tools`   資料夾(密碼 "infected")
2.	使用管理權限執行 `install-choco-and-dependencies.ps1` ，該腳本會安裝 python, neo4j 和 Firefox

   ```powershell
   cat -raw .\install-choco-and-dependencies.ps1 | iex
   ```

3. 設定neo4j
   - After successful installation, open Firefox and navigate to the neo4j web interface at [http://127.0.0.1:7474/browser/](http://127.0.0.1:7474/browser/). 
   - Log in with the default credentials (neo4j/neo4j) and choose a new password for the neo4j user. This step is mandatory, otherwise we can't use Bloodhound later on so make sure that you remember the password.

4. Set up impacket (we'll need this later for NTLM relay). Open an administrative shell and run
   ```
   pip install pyreadline
   pip install impacket
   ```
5. You may also want to install a text editor of your choice.