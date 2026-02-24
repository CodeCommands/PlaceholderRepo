# TECHNICAL MEMORANDUM

**TO:** Information System Security Officer (ISSO) / Network Engineering  
**FROM:** [Your Name/Role]  
**DATE:** February 24, 2026  
**SUBJECT:** Secure Connectivity Architecture for Copado (Salesforce Gov Cloud) to On-Prem Jira  
**CLASSIFICATION:** Internal / Sensitive / High-Availability  

---

## 1. Executive Summary
This memorandum outlines the technical solution for establishing a secure, bidirectional data synchronization between our **Salesforce Government Cloud** instance (utilized by Copado) and our **On-Premise Jira** server. To maintain compliance and protect internal assets, we are implementing a **Zero-Exposure Reverse Proxy Architecture** utilizing Mutual TLS (mTLS).



## 2. System Architecture
The integration relies on a "Double-Lock" validation process. Traffic is only permitted if it meets two concurrent criteria:
1.  **Network-Level Lock:** The request originates from a pre-approved **Salesforce Government Cloud IP Range**.
2.  **Identity-Level Lock:** The request presents a unique **Digital Client Certificate** (mTLS) generated within our specific Salesforce Organization.

## 3. Network-Level Security (The First Lock)
The perimeter firewall must be configured to allow inbound traffic on **Port 443** strictly from the following IP ranges. All other traffic should be dropped by default.

### A. Salesforce Government Cloud Core Ranges
* `13.108.0.0/14`
* `128.17.0.0/16`
* `128.245.0.0/16`

### B. Gov Cloud Plus (Instance-Specific) Ranges
If our instance is on the newer Gov Cloud Plus infrastructure, the following regional ranges are required:
* `18.252.0.0/15`
* `18.254.0.0/15`

### C. Copado Backend Static IPs (CopaCLOUD)
To ensure all Copado-specific management features function (e.g., automated sync triggers), the following static IPs must also be whitelisted:
* `34.73.251.138`, `35.243.255.155`, `104.196.211.229`

## 4. Identity-Level Security (The Second Lock)
We will implement **Mutual TLS (mTLS)**. This ensures that even if an IP is spoofed, the request will be rejected unless it possesses the private key stored in our Salesforce instance.

* **Certificate Handshake:** The Reverse Proxy in the DMZ will act as the server and will be configured to **require** a client certificate from Salesforce.
* **Verification:** The Proxy will compare the incoming certificate against the Public Key provided by our team.
* **Protocol Hardening:** All connections must use a minimum of **TLS 1.2** with high-strength encryption ciphers; self-signed certificates for the proxy identity are prohibited.

## 5. Application-Level Authentication
* **Service Account:** A dedicated "Copado_Integration" account will be created in Jira.
* **Least Privilege:** The account will be granted only "Edit Issues" and "Browse Projects" permissions for necessary projects.
* **Named Credentials:** Salesforce will securely store these credentials and automatically append the necessary authentication headers to outbound calls.

## 6. Implementation Checklist
* [ ] **Salesforce:** Generate and Export the Public Key (.crt).
* [ ] **Firewall:** Whitelist the Salesforce and Copado IP ranges on Port 443.
* [ ] **Reverse Proxy:** Configure a listener to receive the Public Key and enforce Client Certificate Verification.
* [ ] **Jira:** Update the Base URL settings to reflect the new Proxy URL.
* [ ] **Validation:** Perform a test sync to confirm the end-to-end handshake.

---
