# Azure Cloud-based VM Project
#### - We will be building a Virtual Machine primarily using Microsoft Azure and Sentinel. 

#### - To enhance our testing, we will intentionally leave RDP (Remote Desktop Protocol) access open on port 3389 to observe any real-time malicious threats attempting to access our machine.
#
1. First we need to download Microsoft Azure and create an account. Once we have that done, we can create a VM by clicking create and naming our container "eren999VM_group" and the VM itself "eren999VM." Once the VM is created, deployed, and running, we can create a Log Analytics workspace via Microsoft Sentinel to monitor traffic.

![1](https://github.com/user-attachments/assets/59f0fcf1-1bed-4738-ad0d-ee5e6883d6ce)
![2](https://github.com/user-attachments/assets/c676ad9c-1853-43fd-89bf-73cab188965d)
#
2. Now that the VM is running and the Log Analytics workspace is ready, we need to add a data source (Windows Event Logs) to actually see the events and utilize the workspace effectively. This can be done by navigating to Sentinel>Data Connectors>Content Hub>Windows Security Events>Install to connect to our VM.

![3 1](https://github.com/user-attachments/assets/fd70044e-ba14-490c-a4e6-46f28ff8efee)
![3 2](https://github.com/user-attachments/assets/b7633be5-2c9d-4883-b76e-2ff6aeb0f9b2)
#
3. In the VM workspace, under the Network Settings tab, we can see that the RDP connection on port 3389 is open. Let’s go back and generate an alert for activities containing keywords such as “success,” “fail,” “user,” “admin,” and “program” to see if any connections were made.

![3](https://github.com/user-attachments/assets/8340dd06-27ff-4b23-a303-14d3cc79b1e5)
![4 1](https://github.com/user-attachments/assets/49bcf08f-3b16-4f39-882c-f028fdf2e28b)
![4](https://github.com/user-attachments/assets/07a9df7a-257b-4b92-bac4-4513fca1a02c)
#
4. It looks like there’s a significant hit on “program” and “user.” Based on suspicious account names correlated with Event ID 4625 (failed login attempts), we can see that brute force attempts were made to gain access, but all attempts failed.

![5 1](https://github.com/user-attachments/assets/058dfbde-bd0e-4fda-8eb8-151ba5e9a370)
![5](https://github.com/user-attachments/assets/3a85c03c-8fb8-4fd3-b0e4-2741a89c8cb3)
![6](https://github.com/user-attachments/assets/a839e618-e90e-4835-80b7-4e5ea4130e17)
#
5. We can investigate these IP addresses using VirusTotal, revealing that most of the traffic originates from Russia and Ukraine, with several IPs marked as malicious just a few days ago.

![7](https://github.com/user-attachments/assets/e8bc61a3-3db5-4304-9207-7fdfbeab3c32)
![8](https://github.com/user-attachments/assets/e8f4014f-3cf0-499b-8d10-034e38b129e9)
#
6. Although there were no successful logins, it’s important to set up alerts for Event ID 4624 (successful logins) and monitor the malicious IP addresses for potential future access.

![9](https://github.com/user-attachments/assets/977b20a6-8aac-4838-b33a-d1186371fa1f)
![10 1](https://github.com/user-attachments/assets/eb1ade08-50c4-49f8-9fcb-f9db4cb9aa02)
![10 2](https://github.com/user-attachments/assets/283e6521-dbb9-4d4c-af92-38a5b5de7f56)
#
7. We also received another alert with Event ID 4798 (local group access) and detected a potential malicious executable, WmiPrvSE[.]exe. After performing OSINT, I didn’t find any hits on Talos Intelligence or VirusTotal, but HybridAnalysis flagged it as suspicious.

![10](https://github.com/user-attachments/assets/ae71f345-d8e7-4f94-a315-158e3d87c259)
![11 1](https://github.com/user-attachments/assets/b33b1710-edbe-495c-ab25-d118babad0e7)
#
8. Let’s check how many times this executable was associated with the event and whether any child processes were opened. There were 44 instances, but no child processes were launched.

![11](https://github.com/user-attachments/assets/310d856e-baa2-4735-92a5-afd77ad88847)
#
9. HybridAnalysis confirms that this executable is malicious and should be further investigated, as it can adjust token privileges and hide processes by launching under different user credentials.

![12 1](https://github.com/user-attachments/assets/a6a28ddf-63e6-4816-81b0-135e12d27fe3)
#
10. Lastly, I wanted to see if this alert correlates with the earlier brute force attack, so I queried for any failed login events or associations between the malicious IPs and the executable.

![13](https://github.com/user-attachments/assets/236e4599-da7c-47c0-b1b9-b629bf71d902)
![14](https://github.com/user-attachments/assets/ab22eddd-4f89-4dda-99cc-e51d5a4a3604)

There are no results in the past 24 hours supporting a correlation which may suggest they are 2 different attacks, but it’s best to continue monitoring these threats, setting up more alerts, and blocking the malicious IPs’ inbound and outbound traffic via the Firewall/Network Settings tab.
