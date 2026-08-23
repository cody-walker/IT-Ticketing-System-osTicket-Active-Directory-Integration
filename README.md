# IT Ticketing System: osTicket + Active Directory Integration

## Description

This project extends my [Active Directory Home Lab](https://github.com/cody-walker/ActiveDirectoryLab) by adding a self-hosted help desk ticketing system and demonstrating a realistic end-to-end Tier-1 workflow: a user submits a ticket, an agent triages and works it, and — where the request requires it — the agent performs the actual fix in Active Directory on the domain controller from that same lab.

I deployed osTicket, an open-source support ticketing platform, on a new Ubuntu Server VM added to the existing isolated lab network (`192.168.50.0/24`), configured a full LAMP stack (Linux, Apache, MySQL, PHP) to run it, and built out departments and help topics that mirror the Sales/IT/HR organizational structure from the AD lab. I then submitted five realistic sample tickets — using the same test user accounts (Test User1, Test User2) from the original AD lab rather than fictional new identities, so the two projects reference the same real environment — and worked each one as the agent, performing genuine AD actions (account unlock, offboarding, OU transfer) where called for, and documenting each resolution the way a Tier-1 technician would.

Along the way, I caught and corrected several real configuration issues: a PHP extension no longer available in PHP 8.5 (chose to skip it and document why rather than force a workaround), a department-routing default that would have misrouted general IT tickets into HR, and — while working the department transfer ticket — a stale Sales-Team security group membership left over from the original AD lab's OU move, which I found and cleaned up as part of the ticket resolution.

## Languages and Utilities Used

- VirtualBox (hypervisor)
- Ubuntu Server 26.04 LTS
- Apache (web server)
- MySQL (database server)
- PHP 8.5
- osTicket v1.18.4 (open-source helpdesk ticketing system)
- PowerShell (SSH client, used to administer the Ubuntu server remotely)
- Active Directory Users and Computers (ADUC), reused from the AD Home Lab

## Environments Used

- Ubuntu Server 26.04 LTS — osTicket host (`osticket-01`, `192.168.50.20`)
- Windows Server 2022 Standard Evaluation (Desktop Experience) — Domain Controller (`192.168.50.10`), reused from the AD Home Lab
- Windows 11 Enterprise Evaluation — Domain-joined client, reused from the AD Home Lab

---

## Program Walk-through

### Part 1: Standing up the osTicket server

#### 1. Adding a third VM to the existing lab network
**Screenshot: `Screenshot_1.png`**

![Network interfaces detected](screenshots/Screenshot_1.png)
Created a new Ubuntu Server 26.04 LTS VM (`osticket-01`) in VirtualBox, joined to the same Host-only Network (`192.168.50.0/24`) used by the Domain Controller and client from the AD Home Lab, plus a second NAT adapter for internet access during setup. The installer detected both interfaces; the Host-only adapter was subsequently configured with a static IP (`192.168.50.20`) and the Domain Controller (`192.168.50.10`) set as its DNS server.

#### 2. Installing Apache
**Screenshot: `Screenshot_3.png`**

![Apache install](screenshots/Screenshot_3.png)
Installed Apache via `apt`.

#### 3. Verifying Apache is running
**Screenshot: `Screenshot_4.png`**

![Apache status active](screenshots/Screenshot_4.png)
Confirmed Apache was active and running via `systemctl status apache2`.

#### 4. Confirming Apache is reachable across the lab network
**Screenshot: `Screenshot_5.png`**

![Apache default page](screenshots/Screenshot_5.png)
Loaded the server's IP from a browser on the host machine and confirmed the default Apache2 Ubuntu page, proving the web server was reachable across the Host-only network, not just locally.

#### 5. Verifying PHP and its MySQL connectivity
**Screenshot: `Screenshot_6.png`**

![phpinfo page](screenshots/Screenshot_6.png)
Installed PHP 8.5 and the extensions osTicket requires, then confirmed Apache was correctly executing PHP (rather than serving it as plain text) with a `phpinfo()` test page — also confirming PHP's MySQL extensions (mysqli, pdo_mysql, mysqlnd) were loaded.

### Part 2: Installing and configuring osTicket

#### 6. Downloading osTicket and creating its database
Downloaded osTicket v1.18.4 from the official GitHub release, deployed it to Apache's web root with correct file ownership (`www-data`), and created a dedicated MySQL database and scoped database user for osTicket — rather than using the MySQL root account, following the principle of least privilege.

#### 7. Running the web installer — prerequisites check
**Screenshot: `Screenshot_7.png`**

![Prerequisites before fix](screenshots/Screenshot_7.png)
Launched osTicket's browser-based setup wizard. The prerequisites check flagged both the PHP IMAP and APCu extensions as unavailable.

#### 8. Resolving what could be resolved
**Screenshot: `Screenshot_8.png`**

![Prerequisites after fix](screenshots/Screenshot_8.png)
Installed the APCu extension via the standard package manager. The IMAP extension remained unavailable — a known consequence of PHP 8.4+ removing IMAP from PHP core and moving it to PECL, requiring a compile-from-source install for a feature (email-to-ticket piping) not in scope for this project. Documented the decision and proceeded rather than chasing an unnecessary dependency.

#### 9. Completing installation
**Screenshots: `Screenshot_9.png`, `Screenshot_11.png`**

![Install form](screenshots/Screenshot_9.png)
![Installation complete](screenshots/Screenshot_11.png)
Filled out the installer with the helpdesk name, admin account, and the MySQL database connection details created earlier. Successfully completed installation. Afterward, removed the `/setup` installer directory and locked down the configuration file's permissions (`0644`) per osTicket's own post-install security recommendations.

### Part 3: Mirroring the AD lab's organizational structure

#### 10. Logging into the Staff Control Panel
**Screenshot: `Screenshot_12.png`**

![First SCP login](screenshots/Screenshot_12.png)
Logged into osTicket's Staff Control Panel (SCP) — the agent-facing side of the application — for the first time.

#### 11. Configuring departments
**Screenshot: `Screenshot_13.png`**

![Departments configured](screenshots/Screenshot_13.png)
Renamed osTicket's default departments to mirror the Sales/IT/HR organizational structure from the Active Directory Home Lab, and set IT as the default department (correcting an initial default that would have routed uncategorized tickets to HR instead).

#### 12. Configuring help topics
**Screenshot: `Screenshot_14.png`**

![Help topics configured](screenshots/Screenshot_14.png)
Mapped each help topic (the category an end user picks when submitting a ticket) to the correct department, ensuring general IT issues, HR requests, and Sales requests each route to the right team.

---

### Part 4: Working realistic Tier-1 tickets

Five tickets were submitted through the end-user portal, using the same Test User1 and Test User2 accounts from the Active Directory Home Lab rather than newly invented identities — keeping this project grounded in the same lab environment rather than a disconnected demo.

#### 13. Ticket queue overview
**Screenshot: `Screenshot_15.png`**

![Ticket queue](screenshots/Screenshot_15.png)
All five tickets submitted and correctly routed across Sales, IT, and HR departments, confirming the department/help-topic configuration worked as intended.

#### 14. Ticket 1 — Account lockout (High priority)
**Screenshots: `Screenshot_16.png`, `Screenshot_17.png`, `Screenshot_18.png`**

![Ticket opened](screenshots/Screenshot_16.png)
![ADUC unlock action](screenshots/Screenshot_17.png)
![Ticket resolved](screenshots/Screenshot_18.png)
Test User1 reported being locked out of their account. Verified and cleared the lockout in Active Directory via ADUC, then documented the resolution in the ticket and advised the user on avoiding repeat lockouts.

#### 15. Ticket 3 — Employee offboarding (High priority)
**Screenshots: `Screenshot_19.png`, `Screenshot_20.png`, `Screenshot_21.png`, `Screenshot_22.png`, `Screenshot_24.png`, `Screenshot_37.png`, `Screenshot_35.png`, `Screenshot_26.png`**

![Ticket opened, priority escalated](screenshots/Screenshot_19.png)
HR submitted an offboarding request for Test User2. I triaged the queue by priority rather than working tickets top-to-bottom, and escalated this one from Normal to High given the same-day termination.

![Account disabled](screenshots/Screenshot_20.png)
Rather than a single "disable and done" action, I ran a fuller offboarding checklist in Active Directory:

![Sales-Team membership found](screenshots/Screenshot_21.png)
- Reviewed group memberships and found the account still in the `Sales-Team` security group

![Group membership removed](screenshots/Screenshot_22.png)
- Removed the group membership

![Password reset to unrecorded value](screenshots/Screenshot_24.png)
- Reset the password to an unrecorded random value, as a defense-in-depth measure against accidental re-enablement

![Audit note added](screenshots/Screenshot_37.png)
- Added an audit note to the account's Description field documenting the disable date and reason

![Moved to Disabled Users OU](screenshots/Screenshot_35.png)
- Moved the account into a dedicated Disabled Users OU, created for this purpose

![Ticket closed with full resolution notes](screenshots/Screenshot_26.png)
Documented the full checklist in the ticket resolution notes and closed the ticket.

#### 16. Ticket 2 — Password reset request (Normal priority, closed without action)
**Screenshot: `Screenshot_27.png`**

![Ticket closed as no longer applicable](screenshots/Screenshot_27.png)
Test User2 requested a password reset. Before the reset was performed, the same user was offboarded via the ticket above. Rather than resetting the password on an account about to be disabled, I closed this ticket as no-longer-applicable and documented why — reflecting a judgment call a real Tier-1 tech has to make when overlapping requests come in.

#### 17. Ticket 4 — Department transfer (Normal priority)
**Screenshots: `Screenshot_28.png`, `Screenshot_38.png`, `Screenshot_39.png`, `Screenshot_40.png`, `Screenshot_32.png`**

![Ticket opened](screenshots/Screenshot_28.png)
HR submitted a request to move Test User1 from IT to HR.

![HR OU showing Test User1 after the move](screenshots/Screenshot_38.png)
Moved the account's OU in ADUC from IT to HR.

![Sales-Team membership still present](screenshots/Screenshot_39.png)
![Sales-Team membership removed](screenshots/Screenshot_40.png)
While reviewing the account's group memberships, discovered it was still a member of the `Sales-Team` security group — a leftover from an OU move performed in the original Active Directory Home Lab that had never been cleaned up, since OU membership and security group membership are tracked independently in Active Directory. Removed the stale group membership as part of resolving this ticket.

![Ticket closed with resolution notes](screenshots/Screenshot_32.png)
Documented both the OU move and the stale group discovery in the ticket resolution notes.

#### 18. Ticket 5 — Printer connectivity issue (Normal priority, no AD action required)
**Screenshots: `Screenshot_33.png`, `Screenshot_34.png`**

![Ticket opened](screenshots/Screenshot_33.png)
![Ticket resolved with troubleshooting notes](screenshots/Screenshot_34.png)
Test User1 reported being unable to print. Documented a standard Tier-1 hardware/connectivity troubleshooting process — confirming scope, verifying printer connectivity and drivers, and clearing a stuck print spooler job — to demonstrate that not every ticket in a real queue involves Active Directory.

---

## Notes

- All lab data (IP ranges, domain name, usernames, ticket content) is fictional and scoped to an isolated VirtualBox network — nothing here touches a real production environment.
- This project intentionally reuses the same Active Directory Home Lab environment and test user accounts rather than standing up a disconnected demo, to reflect how a ticketing system and its underlying directory service actually work together in a real IT environment.
- A production offboarding process would also typically include revoking mailbox/email access, disabling VPN or remote access credentials, removing SSO-based application access, and retrieving company equipment — none of which apply to an isolated AD-only lab with no Exchange, VPN, or SaaS infrastructure, but worth naming explicitly here.
