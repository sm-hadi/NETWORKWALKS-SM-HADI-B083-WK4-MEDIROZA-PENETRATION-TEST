# Mediroza General Hospital

## Penetration Testing & Security Assessment Report

**Prepared By:** Syed Muhammad Hadi
**Assessment Type:** Black-Box Penetration Testing
**Target:** https://medirozahospital.com
**Assessment Duration:** 5 Days
**Project Status:** Completed

---

# 1. Executive Summary

This report presents the findings of a five-day black-box penetration testing assessment conducted against the Mediroza General Hospital web application.

The assessment focused on evaluating the application's authentication mechanisms, input validation, protection of sensitive documents, exposure of confidential information, and overall security posture.

The assessment identified several significant security weaknesses, including:

* Authentication bypass through SQL injection.
* Unauthorized access to patient laboratory reports.
* Weak password protection on sensitive PDF documents.
* Exposure of an internal database backup containing staff information.
* Disclosure of internal system information through document metadata.

The identified vulnerabilities could allow an unauthorized attacker to gain access to protected medical information and sensitive organizational data.

---

# 2. Assessment Scope

The assessment covered the following areas:

* Patient portal authentication.
* Web application input validation.
* Access controls.
* Patient laboratory reports.
* PDF security mechanisms.
* Document metadata.
* Exposed database backup files.
* Sensitive staff information.

The assessment was performed within an authorized and controlled testing environment.

---

# 3. Assessment Objectives

The primary objectives were to:

1. Identify vulnerabilities in the patient authentication system.
2. Determine whether authentication could be bypassed.
3. Assess the application's protection of patient documents.
4. Evaluate the strength of PDF password protection.
5. Identify sensitive information exposed through files and metadata.
6. Assess the potential impact of exposed database backups.
7. Provide practical remediation recommendations.

---

# 4. Key Findings

| ID | Finding                                          | Severity     |
| -- | ------------------------------------------------ | ------------ |
| M1 | Authentication Bypass via SQL Injection          | **Critical** |
| M2 | Weak Password Protection on Patient Reports      | **High**     |
| M3 | Database Backup and Staff Information Exposure   | **Critical** |
| M4 | Information Disclosure Through Document Metadata | **Medium**   |

---

# 5. Finding M1 — Authentication Bypass via SQL Injection

**Severity:** Critical
**Affected Endpoint:** `/patient/login.php`
**Affected Parameter:** Username

## Description

The patient login functionality was found to be vulnerable to SQL injection.

User-supplied input was incorporated directly into a database query without adequate input validation or parameterized queries. This allowed the authentication logic to be manipulated by submitting specially crafted input.

During testing, the following test payload successfully bypassed the authentication mechanism:

```text
admin' -- 
```

The injected SQL comment caused the remainder of the authentication query to be ignored, allowing access without providing a valid password.

## Impact

Successful exploitation could allow an unauthenticated attacker to:

* Bypass the patient login mechanism.
* Access restricted patient functionality.
* Retrieve protected laboratory reports.
* Potentially access additional patient information depending on application permissions.

## Recommendation

The application should:

* Replace dynamically constructed SQL queries with prepared statements.
* Use parameterized database queries.
* Implement strict server-side input validation.
* Apply least-privilege permissions to the application database account.
* Implement security testing for all authentication-related database queries.

---

# 6. Finding M2 — Weak Password Protection on Patient Reports

**Severity:** High
**Affected Assets:** Patient laboratory PDF reports

## Description

Three patient laboratory reports were obtained during the assessment. Each report was protected with a PDF password; however, the passwords were sufficiently weak to be recovered through offline password testing.

The recovered passwords were common or easily guessable values.

## Affected Reports

| Report                 | Patient        | Password   | Finding                                 |
| ---------------------- | -------------- | ---------- | --------------------------------------- |
| `patient_report_1.pdf` | Sipho Dlamini  | `123456`   | Elevated white cell count               |
| `patient_report_2.pdf` | Priya Reddy    | `password` | Elevated lipid profile                  |
| `patient_report_3.pdf` | Emily Thompson | `!@#$%^&`  | Low haemoglobin, ferritin and vitamin D |

## Impact

Weak document passwords significantly reduce the effectiveness of PDF encryption.

An attacker who obtains a protected report could potentially recover its password offline and access confidential medical information without interacting with the hospital's systems.

## Recommendation

The organization should:

* Use randomly generated, high-entropy passwords for sensitive documents.
* Avoid dictionary words, common passwords, and predictable patterns.
* Use modern encryption mechanisms where supported.
* Establish a secure process for distributing document passwords.
* Consider secure authenticated portals instead of password-protected email attachments or publicly accessible files.

---

# 7. Finding M3 — Database Backup and Staff Information Exposure

**Severity:** Critical
**Affected Asset:** `mediroza_db_backup_2019.sql`
**Database:** `mediroza_hr`
**Table:** `staff`

## Description

An internal SQL database backup was identified during the assessment.

The backup contained sensitive information relating to hospital personnel, including:

* Employee names.
* Job titles.
* Departments.
* Email addresses.
* Telephone numbers.
* National identification numbers.
* Salary information.

The database contained records for approximately 30 staff members.

## Sample Exposure

| Employee                | Position                 | Department        | Monthly Salary |
| ----------------------- | ------------------------ | ----------------- | -------------- |
| Dr. Rajesh Naidoo       | Chief Pathologist        | Diagnostics Lab   | R 138,000      |
| Sarah Botha             | Chief Financial Officer  | Finance           | R 152,000      |
| Dr. Johan van der Merwe | Medical Director         | Management        | R 160,000      |
| Dr. Anita Naicker       | Consultant Cardiologist  | Cardiology        | R 132,000      |
| Dr. Ahmed Kara          | Consultant Physician     | Internal Medicine | R 128,000      |
| Jameel Malik            | IT Systems Administrator | IT                | R 58,000       |

## Impact

Exposure of the database backup could result in:

* Employee privacy violations.
* Identity theft risks.
* Financial and payroll information disclosure.
* Targeted phishing attacks.
* Social engineering attacks.
* Further compromise of internal systems.

## Recommendation

Database backups should:

* Never be stored within publicly accessible web directories.
* Be stored in dedicated, access-controlled backup infrastructure.
* Be encrypted both at rest and during transfer.
* Have access restricted according to the principle of least privilege.
* Be regularly audited for unauthorized exposure.
* Be removed from production web servers.
* Have retention policies and secure deletion procedures.

---

# 8. Finding M4 — Information Disclosure Through Document Metadata

**Severity:** Medium

## Description

Metadata contained within one of the patient PDF documents disclosed an internal username associated with the hospital's IT environment.

The metadata identified the account:

```text
j.malik
```

This information was associated with an internal IT administrator.

## Impact

Although metadata exposure alone does not provide direct system access, internal usernames can assist attackers during reconnaissance and social engineering activities.

When combined with other vulnerabilities, such information may contribute to a larger attack chain.

## Recommendation

The organization should:

* Remove unnecessary metadata from externally distributed documents.
* Configure document-generation systems to strip author and application information.
* Review documents before external publication.
* Avoid exposing internal usernames, hostnames, software versions, or directory information.

---

# 9. Overall Risk Assessment

The assessment identified vulnerabilities ranging from Medium to Critical severity.

The most significant risks were the SQL injection vulnerability and exposure of the internal database backup. Together, these issues demonstrate weaknesses in both application-level security and sensitive data protection.

The SQL injection vulnerability could allow unauthorized access to protected functionality, while the exposed database backup could disclose substantial amounts of confidential employee information.

Weak PDF passwords further increase the risk associated with the exposure of patient reports.

---

# 10. Remediation Roadmap

## Priority 1 — Immediate

### Fix SQL Injection

* Implement prepared statements throughout the application.
* Review all database queries for similar vulnerabilities.
* Validate and sanitize user input.
* Perform security testing after remediation.

### Remove Exposed Database Backups

* Remove SQL backups from publicly accessible locations.
* Move backups to secured storage.
* Restrict access using authentication and authorization controls.
* Encrypt backup files.

## Priority 2 — High

### Strengthen Document Security

* Replace weak passwords with strong randomly generated credentials.
* Use modern encryption mechanisms.
* Implement secure document-sharing mechanisms.
* Review all previously generated sensitive documents.

## Priority 3 — Medium

### Reduce Information Disclosure

* Strip unnecessary PDF metadata.
* Remove internal usernames and system information.
* Review document-generation configurations.
* Establish a document security review process.

---

# 11. Recommended Security Controls

The following controls should be implemented as part of the hospital's broader security program:

* Secure software development practices.
* Regular vulnerability assessments and penetration testing.
* Web application security testing.
* Database access controls.
* Encrypted backup storage.
* Strong authentication mechanisms.
* Least-privilege access controls.
* Secure document handling procedures.
* Centralized security logging and monitoring.
* Regular security awareness training.
* Incident response procedures.

---

# 12. Conclusion

The penetration testing assessment identified several security weaknesses affecting authentication, sensitive document protection, database security, and information disclosure.

The SQL injection vulnerability represents a critical application security issue because it can allow authentication bypass and unauthorized access to protected functionality.

The exposed database backup presents another critical risk due to the amount of sensitive employee information contained within it. Weak PDF passwords further increase the likelihood of unauthorized access to confidential patient reports.

Addressing these vulnerabilities should begin with securing the authentication mechanism and removing exposed database backups, followed by strengthening document protection and reducing unnecessary information disclosure.

All identified vulnerabilities should be retested after remediation to verify that the security issues have been effectively resolved.

---

# 13. Disclaimer

This assessment was conducted in a controlled and authorized environment for educational and security assessment purposes. The findings documented in this report represent observations made during the defined assessment period and scope.

**Prepared By:** Syed Muhammad Hadi
