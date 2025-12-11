The term "Version 1," "Version 2," "Version 3," and "Version 4" refers to the **Certificate Template Schema Version**—essentially, the generation of the certificate template and which features it supports.
Here is a breakdown of the differences, focusing on the jump from Version 1 to Version 2, as that was the most significant shift in terms of configurability and security features.

---

## 🏛️ Certificate Template Version Differences

The different versions are tied to the Windows Server operating system where they were introduced, as they depend on new features in the Certificate Authority (CA) and Active Directory.

### 1. Version 1 Templates (Introduced with Windows 2000)

Version 1 templates are the **original, immutable templates**. They are highly restrictive in their configuration.

|**Feature**|**Details**|
|---|---|
|**Origin**|Windows 2000 Server|
|**Editability**|**Cannot be edited or duplicated.**|
|**Modification**|The only property you can change is the **Security Permissions** (who can read, enroll, etc.).|
|**Autoenrollment**|Limited support, mainly for computer certificates via Group Policy.|
|**Customization**|Very little control over certificate properties (validity, key usage, etc.).|
|**Use Case**|They are primarily the default templates (like `DomainController`, `Administrator`) that you will see in an old or newly installed AD CS environment.|

### 2. Version 2 Templates (Introduced with Windows Server 2003)

Version 2 templates were the game-changer, allowing administrators to create **custom, flexible templates**. This is the version most commonly used for creating custom templates.

|**Feature**|**Details**|
|---|---|
|**Origin**|Windows Server 2003|
|**Editability**|**Fully editable and duplicable.** This is the first version that allows you to create and configure a custom template.|
|**Autoenrollment**|**Full support for autoenrollment** for both user and computer certificates via Group Policy.|
|**Customization**|Allows configuration of:|
||* **Key Usage** and **Extended Key Usage (EKU)**|
||* **Validity and Renewal Periods**|
||* **Minimum Key Size**|
||* **Issuance Requirements** (e.g., manager approval, policy)|
|**Relevance to ESC1**|The **ESC1** vulnerability (Enrollee Supplies Subject) is only possible on Version 2 (or higher) templates because it requires setting the `CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT` flag, which is a template feature introduced in Version 2.|

### 3. Version 3 and Version 4 Templates (Introduced with later Windows Server versions)

These versions build upon Version 2 by introducing features related to modern cryptography and hardware security.

|**Version**|**Origin**|**Key Features Added**|
|---|---|---|
|**Version 3**|Windows Server 2008 / 2008 R2|Added support for **Suite B Cryptographic Algorithms** (e.g., specific ECC curves), making it compatible with more advanced/strict cryptographic requirements.|
|**Version 4**|Windows Server 2012 / 2012 R2|Added support for **Key Attestation** (proving a private key is protected by a hardware Trusted Platform Module - TPM) and **Key-Based Renewal**.|

---

## 💡 Summary for ESC Attacks

From a security and attack perspective, the key takeaway is:

- **Version 1 templates** are generally safe from **template misconfiguration** attacks (like ESC1) because they cannot be edited. If they are exploitable, it's typically due to having overly permissive _default_ permissions (like `Enroll` for everyone).
- **Version 2 templates** (and higher) are the source of most AD CS misconfiguration vulnerabilities (ESC1, ESC2, ESC3, etc.) because administrators were given the ability to customize them, leading to errors in setting flags (like **Enrollee Supplies Subject**) or permissions.
When an attacker is looking for an ESC1 vulnerability, they are specifically looking for a **Version 2 (or higher) template** that has the dangerous combination of flags and permissions.
