---
title: Renewing a RDS/RDG SSL certificate with Godaddy SSL
date: 2021-08-20 10:45
categories: [Sysadmin]
author: Bwilliamson
tags: [powershell, sysadmin, RDS, RDG, SSL, windows server]
---

This is a quick walkthrough on how to do this. This was written for Windows server 2012R2, but should still be applicable to 2016 and 2019 as well. This is specifically for an RDG (Remote Desktop Gateway) server. Your on-prem *RDS* server may or may not contain this role, so keep that in mind.

As of this writing the way to get your cert, assuming you're using Godaddy, is to go here:  

https://certs.godaddy.com/cert  

- Click on the cert you want to download
- Check the expiration date and be sure it's valid
- Select **IIS** as the download type
- Unzip the files to somewhere the server can reach them
- On the RDS/RDG server:
  - Open `mmc.exe`
  - File->Add/Remove snapin->certificates->Computer Account->Local Computer->Finish->OK
  - Expand the Certificates node in the left navigation panel
  - Right click on 'Personal'->All Tasks->Import
    - ![mmc import](/assets/img/post images/sysadmin/RDS SSL certificates/01.png){: .normal }
  - Next->Browse...
  - Find the crt file, Select it, click **Open**, then click Next-> Select **Place all certificates in the following store->Certificate Store:** should read  **Personal**, if not then click Browse... and select Personal. Next -> Finish.
    - ![Cert import wizard](/assets/img/post images/sysadmin/RDS SSL certificates/02.png){: .normal }
  - **IMPORTANT**: If you do not see the little key symbol over the cert when you import it- you need to provide a key for that cert.
    - In most cases you need to provide a key
    - ![Certificates mmc](/assets/img/post images/sysadmin/RDS SSL certificates/03.png){: .normal }
  - To install a key for the imported cert- Open the cert file->Details tab->copy the 'Serial Number' value.
    - ![Cert serial number](/assets/img/post images/sysadmin/RDS SSL certificates/04.png){: .normal }
  - Open admin cmd and type: `certutil -repairstore my “serialnumber”`
  - Where serialnumber is your serial number. So it would look like this: `certutil -repairstore my “xxxxxxxxxxxxxxxx”` (No Spaces)
  - You'll see a bunch of output about the cert, with '**Encryption test passed CertUtil: -repairstore command completed successfully.**' at the bottom if successful.
    - ![Powershell to fix key](/assets/img/post images/sysadmin/RDS SSL certificates/05.png){: .normal }
  - Refresh the MMC, and make sure the little key symbol is on the cert file.
  - For good process- check the 'friendly name' column in the mmc. If that's empty, right click on the cert file->Properties. Enter a friendly name. 'yourFQDN' should be fine. e.g. rds.yourdomain.com.
  - Now we import the cert to RDG:
  - Open Remote Desktop Gateway Manager
    - On Windows Server 2012 R2: Start->Administrative Tools->Remote Desktop Services->Remote Desktop Gateway Manager
      - ![Remote Desktop Services Control Panel](/assets/img/post images/sysadmin/RDS SSL certificates/06.png){: .normal }
  - In the RDGM console tree, right click the local RDG Server, and click Properties
    - ![RDGM tree properties](/assets/img/post images/sysadmin/RDS SSL certificates/07.png){: .normal }
  
  - In the properties, go to the SSL Certificate tab
    - Click "Select an existing certificate from the RD Gateway {Name} Certificates (Local Computer)/Personal store
      - ![SSL certificates tab](/assets/img/post images/sysadmin/RDS SSL certificates/08.png){: .normal }
  - Click "Import Certificate"
  - Select the cert you want to use, and click Import.
  - Click OK to close the Properties window  

Use an SSL checker to verify that you're all set-  https://ssltools.godaddy.com/views/certChecker











