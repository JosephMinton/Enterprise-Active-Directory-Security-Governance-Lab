# [Enterprise Active Directory Security Governance Lab 🖥](https://github.com/JosephMinton/Enterprise-Active-Directory-Security-Governance-Lab/tree/main)
<h1>Active Directory Administration Lab 🏢</h1>

<h2>PowerShell Automation, Domain Services, and Workstation Authentication</h2>

<h2>Description</h2>
<p>Active Directory provides centralized identity and access management for Windows environments. This lab explored how account provisioning, network configuration, and domain authentication work together in a small 
virtualized Windows domain. PowerShell was used to automate bulk user creation, while Windows Server and client side tools were used to validate directory objects, DHCP addressing, name resolution, and domain sign-in.</p>
<br />
<img src="https://i.imgur.com/4y01I53.png"/>
<p>The focus was not only on configuring the environment, but also on verifying that each component behaved as expected before relying on it in the next stage.</p>

<h2>Objective</h2>

<ul>
<li><strong>Automate Active Directory user provisioning with PowerShell</li>
<li><strong>Organize test accounts in a dedicated Organizational Unit (OU)</li>
<li><strong>Validate client IP addressing, gateway configuration, and DNS resolution</li>
<li><strong>Connect a Windows workstation to an Active Directory domain</li>
<li><strong>Verify domain user authentication from the client workstation</li>
<li><strong>Identify security and reliability considerations for administrative automation</li>
</ul>

<h2>Technologies Used 🧪</h2>

<table>
  <tr>
    <td><b>VMware</b></td>
    <td><b>Windows Server 2022</b></td>
  </tr>
  <tr>
    <td><b>Windows 11 VM</b></td>
    <td><b>Active Directory Domain Services</b></td>
  </tr>
  <tr>
    <td><b>PowerShell ISE</b></td>
    <td><b>Active Directory Users and Computers</b></td>
  </tr>
</table>

[Environment Setup Details](https://github.com/JosephMinton/Enterprise-Active-Directory-Security-Governance-Lab/blob/main/Network%20Setup.md)





<h2>1. Lab Environment and Network Architecture</h2>
<p>The environment consisted of a Windows Server domain controller 2022 and a Windows 11 workstation named <code>client1</code>. The client was connected to a 
VMware Internal Network, creating a private virtual network for testing domain services. The domain used in the lab was <code>mydomain.com</code></p>
</br>
<p>The internal DHCP scope was configured for <code>172.16.0.100–172.16.0.200</code>, with <code>172.16.0.1</code> as the expected default gateway. The server also 
provided routing through the lab's NAT/RRAS configuration, allowing the isolated client network to reach external resources.</p>

Key configuration values

<table>
  <tr>
    <td><b>Setting</b></td>
    <td><b>Lab value</b></td>
  </tr>
  <tr>
    <td><b>Domain</b></td>
    <td><b>mydomain.com</b></td>
  </tr>
  <tr>
    <td><b>Client hostname</b></td>
    <td><b>client1</b></td>
  </tr>
    <tr>
    <td><b>Client network</b></td>
    <td><b>VMware Workstation Internal Network</b></td>
  </tr>
    <tr>
    <td><b>DHCP scope</b></td>
    <td><b>172.16.0.100–172.16.0.200</b></td>
  </tr>
    <tr>
    <td><b>Default gateway</b></td>
    <td><b>172.16.0.1</b></td>
  </tr>
</table>

> *These values described are only meant to be used only for my home lab enivornment.*



<p><h2>2. Automated Active Directory User Provisioning</h2>

<p>PowerShell was used to provision multiple test accounts from a text based list rather than creating each account individually through ADUC. The script created a 
  dedicated <code>_USERS</code> Organizational Unit, read names from <code>names.txt</code>, generated usernames using the first initial and surname, and created accounts with <code>New-ADUser</code>.</p>

<p>This exercise demonstrated how structured input and repeatable scripting can reduce repetitive administrative work and help standardize account creation.</p>
Commands and implementation
<p>The script was executed from its working directory on the domain controller. Its key operations included creating the OU and calling the Active Directory user management cmdlets.</p>

```powershell
New-ADOrganizationalUnit -Name _USERS -ProtectedFromAccidentalDeletion $false
```

<img src="https://i.imgur.com/YxkweX8.png"/>

The full script handled reading the source file, deriving usernames, preparing account attributes, and creating users. The directory was then inspected in Active Directory Users and Computers to verify that the <code>_USERS</code> OU and its accounts were present.

After navigate to the script folder: 

```powershell
cd C:\users\a-jminton\desktop\AD_PS-master
```

For command line verification, the following query was used to to execute the loop, creating the _USERS OU and populating user accounts:

```powershell
C:\Users\a-jminton\Desktop\AD_PS-master\1_CREATE_USERS.ps1
```

<img src="https://i.imgur.com/le2zVm1.png"/>
<img src="https://i.imgur.com/E0ZLUOd.png"/>

> *__Administrative considerations__*
>
> *In a production environment, bulk provisioning should also account for duplicate usernames, invalid input, failures partway through a batch, 
  logging, credential handling, and least privilege permissions. Broad execution policy changes and shared default passwords is only done to simplify my home lab enviornment*



<h2>3. Client Network Configuration and Service Validation</h2>

<p>During client network validation, the Windows workstation received a valid DHCP lease but could not access external networks. Investigation revealed that the client had no default gateway configured.</p>

<h3>DHCP Default Gateway Troubleshooting</h3>

<p><strong>Problem Identification:</strong> The initial <code>ipconfig</code> output showed an IPv4 address of <code>172.16.0.100</code> and subnet mask of <code>255.255.255.0</code>, but the Default Gateway field was blank. Without a default gateway, the client could not route traffic to destinations outside its local subnet.</p>

<p><strong>Initial Configuration</strong></p>

<img src="https://i.imgur.com/JGcaDVE.png" alt="Client ipconfig output showing a valid IPv4 address but no default gateway" width="700">

<p><strong>Root Cause:</strong> DHCP Scope Option 003 (Router) had not been configured, so the DHCP server was not supplying the client with a default gateway.</p>

<p><strong>Remediation:</strong> On the DHCP server, I configured Option 003 (Router) with the internal gateway address <code>172.16.0.1</code>.</p>

<p><strong>DHCP Scope Configuration</strong></p>

<img src="https://i.imgur.com/LSD3YR1.png" alt="Client showing the corrected default gateway and connectivity test results" width="700">

<p><strong>Verification:</strong> A subsequent <code>ipconfig</code> check confirmed that the client now had <code>172.16.0.1</code> configured as its default gateway.</p>

<p><strong>Corrected Client Configuration</strong></p>

<img src="https://i.imgur.com/7ORONIt.png" alt="Client ipconfig output confirming the default gateway is now 172.16.0.1" width="700">

> *__Technical Significance__*
>
> *DHCP provides clients with network configuration, DNS resolves hostnames, and routing forwards traffic between networks. A valid IP address alone does not guarantee connectivity. This troubleshooting exercise demonstrated how to identify a missing DHCP scope option, apply the correction, and verify the resulting client configuration.*





<h2>4. Domain Join and User Authentication</h2>
<p>After validating the client's network configuration, <code>client1</code> was joined to <code>mydomain.com</code> and restarted. The domain join established the workstation's 
membership in the Windows domain and created or updated its computer account in Active Directory. The computer account was checked in ADUC, and the client's DHCP lease was reviewed in DHCP Manager. A generated domain account was then used to sign in to the workstation.</p>

<img src="https://i.imgur.com/6d46IWC.png"/>
<img src="https://i.imgur.com/qDdILig.png"/>
</br>
<h3>Authentication verification</h3>
<p>The following command was run in the client's Command Prompt:</p>

```powershell
whoami
```
<p>The expected format was:</p>

```powershell
mydomain\jminton
```

<img src="https://i.imgur.com/QxfVAkG.png"/>

> *This check confirms the identity context of the current Windows session. It is a useful final verification that the session is using the domain account rather than a local workstation account.*



<h1>Key Takeaways</h1>
<ul>
<li><strong>PowerShell automation:</strong> Used PowerShell to automate bulk Active Directory user provisioning from a structured input file, reducing reliance on repetitive manual account creation</li>
<li><strong>Directory organization:</strong> Created a dedicated _USERS Organizational Unit (OU) to organize test accounts and provide a foundation for delegated administration and Group Policy management</li>
<li><strong>Network Troubleshooting:</strong> Identified a missing DHCP default gateway, configured DHCP Scope Option 003 (Router), and verified that the client received the expected gateway address</li>
<li><strong>Domain Services and Authentication:</strong> Joined a Windows workstation to mydomain.com and used the domain account context to verify authentication from the client</li>
<li><strong>Layered Validation:</strong> Used PowerShell, Active Directory Users and Computers, DHCP Manager, and Windows command line tools to inspect configuration and validate lab components at different stages</li>
<li><strong>Security Administration:</strong> Identified production considerations for account provisioning automation, including input validation, duplicate account handling, error logging, credential security, and least privilege permissions</li>
</ul>
