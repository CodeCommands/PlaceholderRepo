I see the issue—the mobile view or specific Markdown editors can sometimes break the rendering of code blocks or lists if the spacing isn't exactly right. I have adjusted the formatting below to be as clean and "bulletproof" as possible for all viewers.
Here is the complete, corrected memorandum in Markdown:
TECHNICAL MEMORANDUM
TO: Information System Security Officer (ISSO) / Network Engineering
FROM: [Your Name/Role]
DATE: February 24, 2026
SUBJECT: Secure Connectivity Architecture for Copado (Salesforce Gov Cloud) to On-Prem Jira
CLASSIFICATION: Internal / Sensitive / High-Availability
1. Executive Summary
This memorandum outlines the technical solution for establishing a secure, bidirectional data synchronization between our Salesforce Government Cloud instance (utilized by Copado) and our On-Premise Jira server. To maintain compliance and protect internal assets, we are implementing a Zero-Exposure Reverse Proxy Architecture utilizing Mutual TLS (mTLS).
2. System Architecture
The integration relies on a "Double-Lock" validation process. Traffic is only permitted if it meets two concurrent criteria:
 * Network-Level: The request originates from a pre-approved Salesforce Government Cloud IP Range.
 * Identity-Level: The request presents a unique Digital Client Certificate (mTLS) generated within our specific Salesforce Organization.
3. Network-Level Security (The First Lock)
The perimeter firewall must be configured to allow inbound traffic on Port 443 strictly from the following IP ranges. All other traffic should be dropped by default.
A. Salesforce Government Cloud Core Ranges
 * 13.108.0.0/14
 * 128.17.0.0/16
 * 128.245.0.0/16
B. Gov Cloud Plus (Instance-Specific) Ranges
If our instance is on the newer Gov Cloud Plus infrastructure, the following regional ranges are required:
 * 18.252.0.0/15
 * 18.254.0.0/15
C. Copado Backend Static IPs (CopaCLOUD)
To ensure all Copado-specific management features function (e.g., automated sync triggers), the following static IPs must also be whitelisted:
 * 34.73.251.138
 * 35.243.255.155
 * 104.196.211.229
4. Identity-Level Security (The Second Lock)
We will implement Mutual TLS (mTLS). This ensures that even if an IP is spoofed, the request will be rejected unless it possesses the private key stored in our Salesforce instance.
A. NGINX Configuration (Example)
The Reverse Proxy in the DMZ should be configured as follows:
server {
    listen 443 ssl;
    server_name [Jira_Proxy_URL];

    # Proxy Server Certificate (Issued by Internal CA)
    ssl_certificate /etc/nginx/ssl/proxy_identity.crt;
    ssl_certificate_key /etc/nginx/ssl/proxy_identity.key;

    # mTLS: Client Certificate Verification
    # This is the public key exported from our Salesforce Org
    ssl_client_certificate /etc/nginx/ssl/salesforce_gov_org.crt;
    ssl_verify_client on;
    ssl_verify_depth 2;

    # Protocol Hardening
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    location /rest/ {
        # Only allow if client certificate is valid
        if ($ssl_client_verify != SUCCESS) { return 403; }

        proxy_pass http://[Internal_Jira_IP]:8080;
        proxy_set_header Host $host;
    }
}

5. Application-Level Authentication
 * Service Account: A dedicated "Copado_Integration" account will be used.
 * Permissions: The account will be granted "Edit Issues" and "Browse Projects" only for applicable projects.
 * Validation: Salesforce Named Credentials will be used to store these credentials encrypted within the platform.
6. Implementation Checklist
 * [ ] Create Self-Signed or CA-Signed Certificate in Salesforce.
 * [ ] Export Public Key (.crt) and provide to Security Team.
 * [ ] Whitelist Salesforce/Copado IPs at the Firewall.
 * [ ] Configure Proxy to enforce ssl_verify_client.
 * [ ] Update Jira Base URL to reflect the new Proxy URL.
Would you like me to draft a step-by-step validation guide to ensure the mTLS handshake is working correctly once IT finishes the setup?
