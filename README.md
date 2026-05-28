# Azure-Security-Posture-Audit

# <ins>**Introduction**</ins>

In this project I will deploy an insecure; VM, storage account, and IAM roles. I will then demonstrate correcting those vulnerabilities after Defender has flagged them. This lab will utilise tools like; Azure Portal which will be how I provision and manage resources, Azure Defender to help flag misconfigurations, Azure RBAC (as part of IAM) for applying privilege levels, and more. Image 1 below first shows all resources deployed within the resource group I created (Posture_audit_rg), in their original, vulnerable state.

_Image 1:_
<img width="1488" height="728" alt="Screenshot 2026-05-28 103154" src="https://github.com/user-attachments/assets/88029036-4160-4918-9a5b-2a492460dc98" />

# <ins>**The VM**</ins>

Taking a closer look at the VM in Image set 2, we can see it is deployed with a public IP of 20.91.239.125, paired with a RDP (port 3389) configuration set to open. This misconfiguration for the remote desktop protocol port leaves it vulnerable to brute force attacks, especially since port 3389 is the default for RDP, which means automated bots along with human attackers will specifically look for port 3389's with weak security. Since they have access to it over the internet, they may use it as an attack vector.

_Image Set 2:_
<img width="1672" height="466" alt="Screenshot 2026-05-28 103312" src="https://github.com/user-attachments/assets/6eb033aa-5f8d-48f9-88a9-081d955fdda2" />
<img width="1434" height="430" alt="image" src="https://github.com/user-attachments/assets/cbe6b782-8b97-4b68-a4f3-8dc71c8f98d6" />

**Update & EDR:**

Defender has also flagged misconfigurations, suggesting an update to windows server as well as EDR configuration issues. Keeping any device up-to date is one of the most effective ways to reduce vulnerabilities. This will remove bugs and known vulnerabilities, preventing attackers from exploiting them. In this case there are 58 of them. 

The EDR issue is important, it is saying microsoft defender for endpoint (MDE) is not running or is not configured correctly for this VM. A missing EDR (endpoint detection response) leaves the resource vulnerable to attack. There is no active prevention for cyberattacks as the EDR isn't continuously detecting, analysing, and then preventing attacks in real time like it would normally do. All of this is visible in Image set 3.

_Image Set 3:_
<img width="1699" height="548" alt="image" src="https://github.com/user-attachments/assets/f2a60b73-0ce1-4c58-b888-3ffc9088e34a" />
<img width="1481" height="825" alt="Screenshot 2026-05-28 150632" src="https://github.com/user-attachments/assets/d89e19f1-5c7c-4721-a98a-187139969582" />
<img width="1381" height="674" alt="image" src="https://github.com/user-attachments/assets/c160ccfa-5d4a-4429-a7a4-d4724be755ef" />

**Fixing the RDP misconfiguration issue:**

This is a simple fix, I simply navigated to 'Network settings' in the VM, and under 'Inbound port rules' selected the RDP port rule and changed 'Source' from 'Any' to 'My Ip address'. This removed the warning mark as now the VM isn't accessible to the entire internet. It is only accessible from my IP. Refer to Image 4.

_Image 4:_
<img width="1430" height="417" alt="image" src="https://github.com/user-attachments/assets/31c7c5a6-dce1-4980-8a7b-61894b9d6c52" />

**Fixing the EDR issue:**

This fix can not be done using the free plan. Microsoft defender for endpoint (plan 2) would be needed for this. Any organization which encounters this issue should upgrade to Microsoft defender for endpoint (plan 2) and configure it to protect the VM. It is important to note plan 1 would not be sufficient since that is not a full EDR.

**Updating Windows Server 2022:**

The Update for the VM included improved intelligence for Microsoft Defender Antivirus. To install this I selected 'Updates' from the side panel of the VM. I Prompted azure to check for updates using the 'check for updates' button (which caused the missing update to appear), after that I selected 'One-time update' which initiated the installation. Importantly I needed to select the right VM as well as properties like installation window and reboot options.

<img width="1478" height="630" alt="image" src="https://github.com/user-attachments/assets/c60f43d2-303f-4846-a6f1-f1c5cf9afaf8" />


# <ins>**Identity & Access Management Configurations**</ins>

Image 5 displays overprivilege. Owner is the most permissive role in azure which means it grants the user full control/power over everything, and allows them to grant permissions to others. This should not be granted to a regular user at the resource group level as it violates the concept of least privilege. Insider threats become a risk, accidental damage is possible, audits may fail compliance checks and if the account was to be compromised it would give the attacker significantly more control over the environment. 

_Image 5:_
<img width="1326" height="562" alt="image" src="https://github.com/user-attachments/assets/e6770d8c-6c5c-4da1-8ae9-e94cb2d97620" />


**Fixing the overprivilege issue:**

To fix this issue, I changed the role of the user in the IAM settings of the resource group. I deleted their owner role, added a new role assignment for the same user, and gave them the reader role instead. This now means they can only view the resource group contents, they can not modify it or escalate the privilege of others. Refer to Image 6.

_Image 6:_   
<img width="932" height="512" alt="image" src="https://github.com/user-attachments/assets/13877015-4a6c-42b7-9f3a-2837e7195a8e" />

# <ins>**The Storage Account**</ins>

The storage account in Image set 7 has been configured to allow public/anonymous access. This means anything on there will be viewable by the public which could put the organization out of compliance. Operations and trade secrets released to the public may also incur economic damage to the organization through loss of trust or competitor action. Access 'Enabled to all networks' and 'Allow blob anonymous access' are the misconfigurations that must be fixed. The former of the two will allow anyone to reach the storage account, the latter configuration means no authentication is needed, anyone that reaches the storage account will have access.

_Image Set 7:_
<img width="1678" height="749" alt="image" src="https://github.com/user-attachments/assets/5fc2202e-7547-40e9-adb1-5a50e7a4505e" />
<img width="1045" height="304" alt="image" src="https://github.com/user-attachments/assets/dc89466c-060e-483c-99e4-159c698a5df9" />


**Fixing the Storage Account:**

A simple fix, to stop the storage account being accessible to the internet, I went to the 'Networking' tab, clicked 'Manage' under Public network access, and changed the configuration to 'disabled'. Disabling anonymous access was completed by going to the 'Configurations' tab, and once again checking 'disabled' for 'Allow Blob anonymous access'.

_Image Set 8:_
<img width="1420" height="375" alt="image" src="https://github.com/user-attachments/assets/f4afc913-507d-491c-9ab0-bf15b5ea535d" />
<img width="1703" height="423" alt="image" src="https://github.com/user-attachments/assets/6dfdaefa-d817-44d6-b561-ca23b26e6eef" />

# <ins>**Conclusion**</ins>
This concludes the Audit Project, all vulnerabilities have been corrected. Attack surface and compliance risks have been minimized by reducing network exposure to the internet, applying patches through an update, and enforcing least privilege in the IAM settings. In a real environment, the EDR issue would need to be resolved by implementing MDE and making sure to configure it to send alerts from detection of malicious activity, which would require the company to spend money on a plan which allows use of the tool. All resources; the VM, storage account, and the IAM roles are now working as expected thanks to recommendations from Defender, along with manual analysis. 



