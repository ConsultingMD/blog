---
layout: post
title: "Secure Engineering for healthcare"
date: 2015-02-02 12:21:12 -0800
comments: true
categories:
author: Bashir Eghbali <bashir@grandroundshealth.com>
---

Innovating in the healthcare space has its own unique challenges. On one hand you have regulatory requirements and customer expectations that you strive to exceed and on the other hand you want to deliver a delightful experience to consumers and patients that solves a real problem and is engaging. In this article, I'll highlight some best engineering practices for developing secure applications in healthcare.
The main topic of engineering in health care due to requirements of HIPAA and sensitivity of Personal Health Information (PHI) is security. Here are some measure to take:

### Data Security ###
* **Encrypted Data Store:** Use an encrypted store that ensures all data is encrypted at rest. If you use AWS look into: http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.Encryption.html
* **Row-Level Encryption:** Use a different salt (random, high entropy) for encrypting PHI
* **Separate Concerns:** If possible store multiple PHI and Personally Identifiable Information (PII) in different data stores or tables
* **Deidentification:** Generate UIDs to use for referencing PHI and PII records that is deidentified. Optimally there is no foreign key to the PHI tables and the UID is computed with a pepper so even if the tables are compromised, the data cannot be linked as it was peppered for that session/application.

### Infrastructure Security ###
* **Use VPCs:** setup your production and staging environments inside Virtual Private Clouds and limit ssh access only through your office IP. on AWS: http://aws.amazon.com/vpc/
* **Inclusive, fine-grain permissions:** Do not give users poweruser permissions, start by giving them fine-grain permissions and add as necessary. You can use policy groups to give similarly fine-grain permissions to many users with similar roles
* **Service Oriented Architecture:** separate access and administration of PHI information into separate services on top of which you build REST applications that access those resources. This allows you to journal access, host those services internally in secure zone without public http access(port 80)
* **Secure application credentials:** This includes your secrets for communicating with other services, credentials for db or other resource accesses, environment variables. Mount encrypted volumes that have these secrets that require a passkey at deployment time to decrypt
* **Apply security patches** to production machines promptly: As security issues are identified and patched, make sure all your production machines are patched promptly.
* **Automate deployment/provisioning:** Avoid manual errors whenever possible. Use automated deployment and provisioning tools (puppet, chef) to roll out new code and services.

### Secure Access ###
* **Audit every access** read/write to PHI records.
* **Strong Authentication:** In a service oriented architecture, you can use oauth to authenticate access to internal services shielding the PHI. Enforce secure passwords for all users of the public systems. Force admins and anyone with expanded permissions to change passwords regularly and with higher entropy
* **Use VLANs:** Limit access to all internal services (access to PHI) to your office.
* **Use Two factor authentication:** For access and administration of these internal services where PHI is stored, use two factor auth.
* **Designate a Security Officer:** That reviews wireless access logs, audit records and login reports to ensure compliance and detect anomolies
* **Secure Network:** allow access to your wireless and wired network only to registered MAC addresses on official computers. Better yet, implement 802.1X authentication so only registered users can access wired or wireless networks
* Show PHI info only on a **need-to-know basis**. PHI should only be available where and during the time that it is needed. Remove permissions for access to a user that no longer 'needs the phi'. You can also use two-factor encryption to finely manage who can see what data at what time and expire keys for users that no longer have permission.

### Secure Development ###
* **Instill secure development methodology:** Get every engineer thinking about 'what can go wrong?', 'worst case scenario' and what the common sources or data security breaches are
* **Security minded code reviews:** Every commit should be code reviewed by someone else. Pay close attention to all queries and ensure arguments are sanitized, ensure form submissions are guarded and cannot change any data beyond what is intended.
* **Use Secure Components:** Know what you are putting in your soup. Do not use modules, gems, javascript libraries that you(and bunch of other people) have not reviewed. Try sourcing everything from trusted servers

### Healthy Paranoia ###
* **Be paranoid:** when designing, reviewing and deploying, have a healthy dose of paranoia.
* **Rinse, repeat wiser:** Security lapses, bugs, attacks happen, report them promptly, take action immediately, communicate them transparently and revise your processes to ensure they don't happen again
* **Insert your own PHI/PII info** in the system. It helps guide your compass when making security decisions.

Happy Innovating