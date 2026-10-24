# [Microsoft 365 Enterprise Simulation ☁️](https://github.com/JosephMinton/Microsoft-365-Enterprise/blob/main/README.md)
<h1>Active Directory Administration Lab 🏢</h1>

PowerShell Automation, Domain Services, and Workstation Authentication
<h2>Description</h2>
<p>Active Directory provides centralized identity and access management for Windows environments. This lab explored how account provisioning, network configuration, and domain authentication work together in a small 
virtualized Windows domain. PowerShell was used to automate bulk user creation, while Windows Server and client-side tools were used to validate directory objects, DHCP addressing, name resolution, and domain sign-in.</p>
<br />
<p>The focus was not only on configuring the environment, but also on verifying that each component behaved as expected before relying on it in the next stage.</p>

<h2>Objective</h2>

<ul>
<li><strong>Automate Active Directory user provisioning with PowerShell</li>
<li><strong>Organize test accounts in a dedicated Organizational Unit (OU)</li>
<li><strong>Validate client IP addressing, gateway configuration, and DNS resolution</li>
<li><strong>Connect a Windows workstation to an Active Directory domain</li>
<li><strong>Verify domain-user authentication from the client workstation</li>
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

<h2>1. Lab Environment and Network Architecture</h2>
<p>The environment consisted of a Windows Server domain controller and a Windows 10 Pro workstation named `client1`. The client was connected to a 
VirtualBox Internal Network, creating a private virtual network for testing domain services. The domain used in the lab was `mydomain.com`</p>
</br>
<p>The internal DHCP scope was configured for `172.16.0.100–172.16.0.200`, with `172.16.0.1` as the expected default gateway. The server also 
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
    <td><b>VirtualBox Internal Network</b></td>
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

These values describe the training environment and are not intended as production network recommendations.

<p>Installation was verified using 'Get-InstalledModule', which confirmed Microsoft Graph SDK version 2.37.0 was successfully installed from PSGallery.</p>


<h2>2. Automated Active Directory User Provisioning</h2>

<p>PowerShell was used to provision multiple test accounts from a text-based list rather than creating each account individually through ADUC. The script created a 
  dedicated _users Organizational Unit, read names from names.txt, generated usernames using the first initial and surname, and created accounts with New-ADUser.</p>

<p>This exercise demonstrated how structured input and repeatable scripting can reduce repetitive administrative work and help standardize account creation.</p>
Commands and implementation
<p>The script was executed from its working directory on the domain controller. Its key operations included creating the OU and calling the Active Directory user-management cmdlets.</p>

```powershell
New-ADOrganizationalUnit -Name "_users"
```

The full script handled reading the source file, deriving usernames, preparing account attributes, and creating users. The directory was then inspected in Active Directory Users and Computers to verify that the _users OU and its accounts were present.

For command-line verification, the following query can be used when the domain and OU distinguished name match the lab:

```powershell
Get-ADUser -Filter * `
  -SearchBase "OU=_users,DC=mydomain,DC=com" |
  Select-Object Name, SamAccountName, Enabled"
```

Administrative considerations

<p>The lab used a simplified account-creation workflow for learning purposes. In a production environment, bulk provisioning should also account for duplicate usernames, invalid input, failures partway through a batch, 
  logging, credential handling, and least-privilege permissions. Broad execution-policy changes and shared default passwords should not be treated as production defaults.</p>

<p><strong>Evidence:</strong> Insert a screenshot of the relevant script logic, the execution result, and the _users OU in ADUC. Captions should describe what each image verifies rather than simply stating that a screenshot was taken. </p>

<h2>3. Client Network Configuration and Service Validation</h2>
<p>Before joining the workstation to the domain, the client's network configuration was checked to confirm that it received an address from the expected DHCP scope and had the correct gateway. 
Connectivity and name-resolution tests were then used to assess whether the client could reach domain and external resources.</p>
The primary validation command was:

```powershell
ipconfig /all
```

Connectivity and name-resolution checks included:

```powershell
ping mydomain.com
ping google.com
```

<p>The client was expected to receive an address in the 172.16.0.100–172.16.0.200 range and use 172.16.0.1 as its default gateway. 
If a valid DHCP lease was not present, the lease could be renewed with ipconfig /renew.</p>

Technical significance

<p>DHCP, DNS, and routing serve different purposes. DHCP supplies client network settings; DNS resolves names to records; routing determines how traffic reaches other networks. A valid IP address alone does 
not prove that DNS or routing works, and a failed ping does not necessarily mean a service is unavailable because ICMP may be blocked.</p>

<p>For Active Directory domain discovery, the client must be able to query the appropriate domain DNS service and resolve the records required to locate a domain controller.</p>

<p><strong>Evidence:</strong> Insert the VirtualBox adapter configuration and client network-test output. Record the actual IP address, gateway, DNS server, and test outcomes visible in the lab rather than assuming every check succeeded.</p>


<h2>4. Domain Join and User Authentication</h2>
<p>After validating the client's network configuration, client1 was joined to mydomain.com and restarted. The domain join established the workstation's 
membership in the Windows domain and created or updated its computer account in Active Directory.</p>

<p>The computer account was checked in ADUC, and the client's DHCP lease was reviewed in DHCP Manager. A generated domain account was then used to sign in to the workstation.</p>

Authentication verification
 <p>The following command was run in the client's Command Prompt:</p>

```powershell
whoami
```
<p>The expected format was:</p>

```powershell
mydomain\username
```

<p>This check confirms the identity context of the current Windows session. It is a useful final verification that the session is using the domain account rather than a local workstation account.</p>

<p><strong>Evidence:</strong> Insert the domain-join confirmation, the computer account and DHCP lease where applicable, and the whoami output from the signed-in session. 
Use captions to identify the specific configuration or result each image supports.</p>

<h1>Key Takeaways</h1>
<ul>
<li><strong>PowerShell automation:</strong> Scripted provisioning can make repetitive account creation more consistent than individual manual operations. The scale and time savings should be reported only when measured or verified.</li>
<li><strong>Directory organization:</strong> Organizational Units provide a way to group and manage directory objects and can be used as targets for delegated administration and Group Policy.</li>
<li><strong>Service dependencies:</strong> Successful domain joining depends on more than an IP address. Correct DNS configuration, network connectivity, and domain-controller discovery are also essential.</li>
<li><strong>Layered troubleshooting:</strong> Checking IP configuration, DNS resolution, connectivity, and domain authentication as separate layers helps narrow down where a failure occurs.</li>
<li><strong>Centralized identity:</strong> Domain users can authenticate to appropriately configured domain-joined workstations, subject to account status, policy, permissions, and service availability.</li>
<li><strong>Executed a complete offboarding workflow: License removal, account deletion, and recycle bin verification entirely from the command line interface</li>
<li><strong>Security-conscious administration:</strong> Reliable automation should validate input, handle errors, avoid reusable shared passwords, and use only the permissions required for the task.</li>
</ul>
