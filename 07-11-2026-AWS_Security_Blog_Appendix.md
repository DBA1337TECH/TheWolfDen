# AWS Security Guides - Appendix
## Technical Terms & Definitions

---

## Table of Contents
1. [AWS Services & Products](#aws-services--products)
2. [Network & VPN Technologies](#network--vpn-technologies)
3. [Security & Cryptography](#security--cryptography)
4. [Cloud Infrastructure Concepts](#cloud-infrastructure-concepts)
5. [Compliance & Standards](#compliance--standards)
6. [Monitoring, Logging & Detection](#monitoring-logging--detection)
7. [Access Control & Identity](#access-control--identity)
8. [Acronyms & Abbreviations](#acronyms--abbreviations)

---

## AWS Services & Products

### Access Analyzer
A security tool that helps you identify resources in your AWS account that are shared with external principals. It validates IAM policies to ensure they only grant intended access and highlights overly permissive policies.

### AWS CLI (Command Line Interface)
The official command-line tool for managing AWS services. It provides a unified interface to interact with AWS without using the web console, enabling automation and scripting of infrastructure management.

### AWS Config
A service that records and evaluates the configuration of your AWS resources against desired configurations. It tracks changes over time and can enforce compliance rules automatically through Config Rules.

### AWS Config Rules
Automated checks that evaluate whether AWS resources comply with desired configurations. Rules can be AWS-managed (pre-built) or custom (Lambda-based), running on-demand or continuously.

### AWS CloudFormation
Infrastructure-as-Code service that allows you to define and provision AWS infrastructure using JSON or YAML templates. Enables version control, reproducible deployments, and automated stack management.

### AWS CloudTrail
A logging service that records all API calls and activities in your AWS account. Provides audit trails for compliance, security monitoring, and troubleshooting across all AWS services.

### AWS CloudWatch
A monitoring and observability service that collects metrics, logs, and events from AWS resources. Enables dashboards, alarms, and automated responses based on defined thresholds.

### AWS Firewall Manager
Centralized security management service for deploying and managing AWS WAF rules, Shield protections, and security groups across AWS Organizations.

### AWS GuardDuty
Managed threat detection service using machine learning to identify malicious activity, unauthorized access attempts, and anomalous behavior in your AWS environment.

### AWS IAM (Identity and Access Management)
Service for managing users, roles, policies, and permissions in AWS. Implements least-privilege access control and enables multi-factor authentication.

### AWS Macie
Data discovery and protection service that identifies and classifies sensitive data (like PII) in S3 buckets and alerts on unintended data access.

### AWS Route 53
DNS (Domain Name System) service that routes end-user traffic to applications. Supports public zones (internet) and private zones (VPC-only), with health checking and failover.

### AWS S3 (Simple Storage Service)
Object storage service for storing and retrieving any amount of data. Provides durability, availability, and features like versioning, encryption, and access logging.

### AWS Security Hub
Centralized SIEM-like service that aggregates security findings from multiple AWS services and third-party tools, providing compliance status and security insights.

### AWS Shield
DDoS protection service. Shield Standard (free) provides automatic protection against common attacks; Shield Advanced ($3k/month) adds manual response assistance and cost protection.

### AWS Systems Manager
Service suite for managing AWS resources at scale, including patch management, document automation, session management, and parameter storage.

### AWS WAF (Web Application Firewall)
Service that protects web applications from common exploits by filtering traffic based on rules. Blocks SQL injection, cross-site scripting, and other Layer 7 attacks.

### Amazon EC2 (Elastic Compute Cloud)
Virtual server service providing on-demand computing capacity. Instances can be configured with different sizes, operating systems, and security settings.

### Amazon Inspector
Automated vulnerability scanning service that assesses EC2 instances and container images for security vulnerabilities and deviations from best practices.

### Amazon Macie
Data security service that uses machine learning to discover, classify, and protect sensitive data in S3 buckets.

### Amazon VPC (Virtual Private Cloud)
Isolated virtual network environment where you can launch AWS resources. Enables custom IP addressing, subnets, routing, and security configurations.

### Elastic IP
Static public IP address associated with your AWS account. Can be assigned to EC2 instances and remains associated even if the instance is stopped.

### Lambda
Serverless computing service that executes code in response to events without managing servers. Used for automation, data processing, and triggered responses.

### RDS (Relational Database Service)
Managed database service supporting PostgreSQL, MySQL, Oracle, SQL Server, and MariaDB. Handles backups, patches, and high availability.

### SNS (Simple Notification Service)
Message notification service enabling publish-subscribe messaging, email alerts, and SMS notifications.

### SQS (Simple Queue Service)
Managed message queue service for decoupling components and enabling asynchronous communication between services.

### VPC Peering
Direct network connection between two VPCs allowing resources to communicate as if they're in the same network, without exposing to public internet.

---

## Network & VPN Technologies

### AES (Advanced Encryption Standard)
Symmetric encryption algorithm using 128-bit, 192-bit, or 256-bit keys. AES-256 is cryptographically secure and widely used for data protection.

### BGP (Border Gateway Protocol)
Routing protocol used to exchange routing information between autonomous systems on the internet. Used in AWS for dynamic routing between on-premises and cloud networks.

### ChaCha20
Modern stream cipher designed as an alternative to AES. Offers good performance on systems without AES hardware acceleration.

### CIDR (Classless Inter-Domain Routing)
Notation for IP address ranges (e.g., 10.0.0.0/16). The prefix (e.g., /16) indicates the number of fixed bits in the network address.

### DNS (Domain Name System)
Protocol that translates human-readable domain names (example.com) into IP addresses. Uses hierarchical naming system and can be public or private.

### DNSSEC (DNS Security Extensions)
Protocol extensions that add cryptographic signing to DNS responses, preventing DNS spoofing and tampering attacks.

### DynamoDB
NoSQL database service offering key-value and document storage with automatic scaling. Supports encryption and fine-grained access control.

### Flow Logs
VPC feature that captures network traffic information (source IP, destination IP, ports, protocol) for troubleshooting and security analysis.

### GRE (Generic Routing Encapsulation)
Tunneling protocol that encapsulates packets for transmission across a network. Often used with IPsec for secure site-to-site VPN.

### HTTP (HyperText Transfer Protocol)
Application layer protocol for transferring web pages and data. Unencrypted version; HTTPS is the encrypted variant using TLS.

### HTTPS (HyperText Transfer Protocol Secure)
Secure version of HTTP that encrypts data in transit using TLS/SSL. Required for protecting sensitive information.

### IGW (Internet Gateway)
VPC component that connects your VPC to the internet and enables communication between resources and the public internet.

### IPsec (Internet Protocol Security)
Protocol suite for securing internet communication by authenticating and encrypting packets. Forms the basis of VPN tunnels.

### Mesh Network
Network topology where nodes can connect to multiple other nodes, creating redundant paths for communication. Enables load balancing and fault tolerance.

### NAT (Network Address Translation)
Process of mapping private IP addresses to public IP addresses for outbound internet access. NAT Gateway provides high-availability outbound access.

### NTP (Network Time Protocol)
Protocol for synchronizing clocks across network devices. Critical for VPN and security operations requiring timestamp accuracy.

### Onion Routing
Encryption technique with multiple layers, where each relay decrypts only its layer. Prevents intermediate nodes from seeing end-to-end traffic.

### OpenVPN
Open-source VPN protocol and software. Flexible and widely used; supports multiple authentication methods and encryption algorithms.

### Overlay Network
Virtual network built on top of physical network infrastructure. VPN tunnels are overlay networks that provide encryption and isolation.

### PKI (Public Key Infrastructure)
System for managing digital certificates and asymmetric encryption keys. Enables secure authentication and encryption.

### Route Table
Set of rules (routes) determining where network traffic is directed in a VPC. Each subnet has an associated route table.

### Security Group
Stateful firewall at the instance level controlling inbound and outbound traffic. Acts as the first line of defense for EC2 instances.

### SIP (Session Initiation Protocol)
Protocol for initiating, maintaining, and terminating multimedia sessions. Used in VoIP and video conferencing.

### TLS (Transport Layer Security)
Cryptographic protocol providing secure communication over networks. TLS 1.3 is the current standard with better performance and security.

### Transit Gateway
Central hub simplifying management of connections between VPCs, on-premises networks, and remote offices.

### UDP (User Datagram Protocol)
Connectionless transport layer protocol. Faster than TCP but without guaranteed delivery; used by VPN and real-time applications.

### VPN (Virtual Private Network)
Secure network connection that encrypts traffic and provides privacy. Creates encrypted tunnel between client and VPN server.

### WireGuard
Modern VPN protocol with minimal code (~4000 lines). Known for high performance, modern cryptography, and ease of configuration.

---

## Security & Cryptography

### Authentication
Process of verifying identity of a user or system. Methods include passwords, multi-factor authentication (MFA), certificates, and biometrics.

### Authorization
Process of determining what authenticated users are permitted to do. Typically implemented through IAM policies and access control lists.

### CAVP (Cryptographic Algorithm Validation Program)
NIST program that validates cryptographic algorithms meet specific standards, ensuring compliance and security.

### Certificate
Digital document binding a public key to an identity. Issued by Certificate Authorities and used for authentication and encryption.

### Certificate Authority (CA)
Trusted entity that issues digital certificates. Self-signed certificates can be used internally; public CAs for internet-facing services.

### Cipher
Algorithm for encryption/decryption. Stream ciphers (ChaCha20) and block ciphers (AES) are common types.

### Compliance
Adherence to security standards, regulations, and best practices. Examples: PCI-DSS, HIPAA, SOC 2, ISO 27001.

### CRL (Certificate Revocation List)
List of revoked digital certificates. Checked to verify certificates haven't been compromised before accepting them.

### Cryptography
Study and practice of secure communication using mathematical algorithms. Includes symmetric encryption, asymmetric encryption, and hashing.

### DDoS (Distributed Denial of Service)
Attack where multiple sources flood a target with traffic, making it unavailable. Can be volumetric (L3/L4) or application-layer (L7).

### Defense-in-Depth
Security strategy using multiple overlapping security controls. If one layer is breached, others prevent full system compromise.

### Encryption
Process of converting plaintext to ciphertext using an algorithm and key. Only authorized parties with the key can decrypt.

### Forward Secrecy
Property where session keys are not compromised even if long-term keys are stolen. Each session uses a unique key.

### HSTS (HTTP Strict Transport Security)
HTTP header telling browsers to always use HTTPS for a domain, preventing man-in-the-middle attacks.

### Key Derivation
Process of generating cryptographic keys from passwords or other sources. Should use salt and multiple iterations (PBKDF2, bcrypt, scrypt).

### Key Rotation
Regular replacement of cryptographic keys. Limits exposure if a key is compromised and is required by most compliance frameworks.

### KMS (Key Management Service)
AWS service for managing encryption keys. Integrates with AWS services and provides audit logging and access control.

### Man-in-the-Middle (MITM)
Attack where attacker intercepts communication between two parties. SSL pinning and HTTPS help prevent this.

### MFA (Multi-Factor Authentication)
Authentication requiring multiple verification methods (something you know, have, or are). Significantly improves account security.

### Nonce
"Number used once" - random value used in cryptographic protocols to prevent replay attacks.

### OWASP (Open Web Application Security Project)
Organization providing guidance on web application security. OWASP Top 10 lists most critical vulnerabilities.

### Perfect Forward Secrecy (PFS)
Cryptographic property ensuring session keys aren't compromised if long-term keys are revealed. Requires unique ephemeral keys per session.

### Pre-Shared Key (PSK)
Symmetric key shared beforehand between communicating parties. Used in some VPN configurations for additional security.

### Public Key Infrastructure (PKI)
System for managing keys and certificates. Enables secure communication without pre-sharing secrets.

### Rainbow Table
Precomputed table mapping password hashes to passwords. Thwarted by using random salt during hashing.

### Replay Attack
Attack where attacker captures and resends a valid message. Prevented by using nonces, timestamps, or sequence numbers.

### Risk Assessment
Process of identifying threats, vulnerabilities, and impacts. Informs security decisions and resource allocation.

### Salt
Random value added to password before hashing. Prevents identical passwords from producing identical hashes.

### Session Token
Unique identifier issued after authentication. Used to maintain authenticated state without requiring repeated authentication.

### SHA (Secure Hash Algorithm)
Cryptographic hash function producing fixed-length digest. SHA-256 and SHA-512 are commonly used; SHA-1 is deprecated.

### Signature
Cryptographic proof of authenticity and non-repudiation. Created using private key; verified using public key.

### SSRF (Server-Side Request Forgery)
Attack where attacker tricks server into making unauthorized requests. Can expose internal services or credentials.

### SSL (Secure Sockets Layer)
Predecessor to TLS. Largely deprecated; TLS is the modern standard for secure communication.

### SSH (Secure Shell)
Protocol for secure remote login and command execution. Uses public key authentication and provides encrypted terminal sessions.

### Threat Model
Framework for identifying potential attacks against a system. Helps prioritize security controls.

### Vulnerability
Weakness in system that can be exploited by attackers. Can be in code, configuration, or processes.

### Zero-Trust
Security model assuming no implicit trust. Every access request must be authenticated and authorized, even from "trusted" networks.

---

## Cloud Infrastructure Concepts

### Availability Zone (AZ)
Isolated data center within a region. AZs are connected by low-latency network; placing resources across AZs enables high availability.

### Backup
Copy of data and configuration used for recovery if primary data is lost or corrupted. Can be automatic (RDS) or manual (S3).

### Baseline
Known good configuration used for comparison. Security baselines define minimal security requirements.

### Capacity Planning
Process of estimating resource requirements for current and future load. Prevents performance issues and cost overruns.

### Configuration Drift
Unintended changes to resource configuration over time. Infrastructure-as-Code and Config Rules help prevent drift.

### Consolidation
Process of combining multiple services or instances to reduce complexity and cost. Can improve security through centralized management.

### Disaster Recovery (DR)
Process and procedures for recovering from disasters. Includes backup strategies and failover mechanisms.

### EBS (Elastic Block Store)
Block storage service providing persistent volumes for EC2 instances. Supports snapshots for backup and encryption.

### Failover
Automatic or manual process of switching to backup systems when primary fails. Critical for high availability.

### Geo-Redundancy
Data replication across geographically distant locations. Protects against regional outages.

### High Availability (HA)
System design ensuring availability despite component failures. Uses redundancy and failover mechanisms.

### Hybrid Cloud
Architecture combining on-premises infrastructure with cloud services. Requires secure network connections (VPN/ExpressRoute).

### Immutability
Property where data or configuration cannot be changed after creation. Important for audit logs and compliance.

### Infrastructure-as-Code (IaC)
Practice of defining infrastructure using code/configuration files. Enables version control, automation, and reproducibility.

### Instance Profile
Container for IAM role allowing EC2 instances to assume role and access AWS resources.

### Isolation
Separation of resources to prevent unauthorized access or interference. Network isolation (VPC) and resource isolation are key concepts.

### Load Balancing
Distribution of network traffic across multiple servers. Improves performance, availability, and fault tolerance.

### Microservices
Architecture pattern where application is decomposed into small, independently deployable services. Enables scaling and resilience.

### Monitoring
Continuous collection and analysis of metrics and logs. Enables detection of issues and performance optimization.

### Multi-Region
Architecture distributing resources across multiple AWS regions. Provides disaster recovery and low-latency access.

### MTBF (Mean Time Between Failures)
Average time between failures. Higher MTBF indicates more reliable systems.

### MTTR (Mean Time To Recovery)
Average time to restore system after failure. Lower MTTR indicates better reliability and disaster recovery.

### On-Demand
Purchasing model where you pay for resources as you use them. Provides flexibility without long-term commitments.

### Provisioning
Process of allocating and configuring resources. Can be manual or automated.

### Region
Geographic area containing multiple availability zones. AWS operates multiple regions worldwide.

### Replication
Automatic copying of data across multiple locations. Ensures data availability and durability.

### Scalability
Ability to handle increased load by adding resources. Horizontal scaling (more instances) vs. vertical scaling (larger instances).

### SLA (Service Level Agreement)
Contractual guarantee of service availability and performance. AWS publishes SLAs for its services.

### Snapshot
Point-in-time backup of a volume or database. Can be used to restore or clone resources.

### Stateless
Design where service doesn't store session state locally. Enables horizontal scaling since requests can go to any instance.

### Subnet
Subdivision of VPC with its own IP address range. Can be public (with internet access) or private.

### Tag
Key-value metadata attached to resources. Enables organization, cost tracking, and automation.

### Throughput
Rate of data transfer, measured in bits per second (Gbps) or operations per second. Important for performance analysis.

### TTL (Time To Live)
Duration a resource or cached value remains valid. Lower TTL enables faster updates; higher TTL improves performance.

### Vertical Scaling
Increasing capacity by using larger/more powerful instances. Contrast with horizontal scaling (more instances).

### Warm Start
Pre-initialized resources ready to handle requests. Important for Lambda and containerized services.

### Workload
Set of applications and resources supporting a business function. Can be monolithic or distributed.

---

## Compliance & Standards

### CIS (Center for Internet Security)
Organization providing consensus benchmarks for security configurations. CIS AWS Foundations Benchmark is commonly used.

### COBIT (Control Objectives for Information and Related Technologies)
Framework for IT governance and control. Provides structure for managing IT resources and processes.

### Compliance
Adherence to laws, regulations, and organizational policies. Security compliance includes data protection, access control, and audit logging.

### COSO (Committee of Sponsoring Organizations)
Framework for internal controls providing guidance on risk management and governance.

### CSA (Cloud Security Alliance)
Organization promoting security best practices in cloud computing. Provides Cloud Controls Matrix (CCM).

### Data Classification
Categorization of data based on sensitivity and impact. Common levels: Public, Internal, Confidential, Restricted.

### Data Residency
Requirement that data remain within specific geographic location. Driven by regulations or business requirements.

### DLP (Data Loss Prevention)
Tools and processes preventing sensitive data from leaving organization. Monitors and blocks unauthorized data transfers.

### FEDRAMP (Federal Risk and Authorization Management Program)
US government program providing standardized assessment and authorization of cloud services.

### GDPR (General Data Protection Regulation)
European regulation governing personal data protection. Requires consent, data rights, and privacy impact assessments.

### HIPAA (Health Insurance Portability and Accountability Act)
US regulation protecting health information privacy. Requires encryption, access controls, and audit logging.

### HITRUST
Certification framework combining HIPAA, HITECH, and NIST requirements. Commonly required for healthcare providers.

### Incident Response
Process for responding to security incidents. Includes detection, containment, eradication, and recovery.

### ISO 27001
International standard for information security management systems. Requires comprehensive security controls across organization.

### Least Privilege
Security principle where users/systems receive minimum permissions necessary. Reduces blast radius of compromised accounts.

### NIST (National Institute of Standards and Technology)
US government agency providing cybersecurity frameworks and guidelines. NIST Cybersecurity Framework is widely adopted.

### PCI-DSS (Payment Card Industry Data Security Standard)
Standard for securing payment card data. Requires encryption, access controls, and vulnerability scanning.

### Remediation
Process of fixing security issues or policy violations. Can be manual or automated.

### Right to be Forgotten
GDPR right allowing individuals to request deletion of personal data. Requires ability to delete data when requested.

### Risk
Combination of threat likelihood and impact. Risk management includes identification, assessment, and mitigation.

### RTO (Recovery Time Objective)
Maximum acceptable time to recover from disaster. Drives disaster recovery strategy.

### RPO (Recovery Point Objective)
Maximum acceptable data loss (time-wise). Drives backup frequency and replication strategy.

### SOC 2 (Service Organization Control)
Audit standard for service providers. SOC 2 Type II demonstrates controls over time; Type I is point-in-time.

### Threat
Potential cause of security incident. Can be malicious (attacker) or accidental (misconfiguration).

---

## Monitoring, Logging & Detection

### Anomaly Detection
Process of identifying unusual patterns in data. Machine learning helps identify threats humans might miss.

### Audit Log
Record of access and changes to systems and data. Critical for compliance and forensics.

### Baseline
Known good state used for comparison. Security baselines help detect deviations.

### Benchmark
Standard measurement for comparison. Security benchmarks (CIS) define best practices.

### CEF (Common Event Format)
Standardized log format for security events. Enables interoperability between security tools.

### CloudWatch Logs Insights
Service for querying CloudWatch Logs using SQL-like syntax. Enables log analysis and threat hunting.

### Correlation
Linking related events from multiple sources. Helps identify attack patterns.

### Dashboard
Visual representation of metrics and status. Enables quick assessment of system health and security.

### Detection
Process of identifying security incidents. Can be signature-based (known threats) or behavioral (anomalies).

### Event
Activity or change in system. Events are logged and can trigger automated responses.

### False Negative
Security issue that goes undetected. Reduces visibility and security posture.

### False Positive
Alert for non-issue activity. Increases alert fatigue and reduces effectiveness of monitoring.

### Fingerprinting
Identifying systems by unique characteristics. Used in reconnaissance and attack preparation.

### Forensics
Analysis of incident to understand what happened and how to prevent recurrence. Requires detailed logs and artifacts.

### HDFS (Hadoop Distributed File System)
Distributed file system for storing large files across multiple machines. Used for big data and log analysis.

### HUMINT (Human Intelligence)
Intelligence gathered from human sources. Complements technical monitoring.

### Hunting
Proactive search for threats in logs and data. More active than passive monitoring.

### IDS (Intrusion Detection System)
Tool that detects suspicious network activity. NIDS (network) and HIDS (host) are variations.

### Indicator of Compromise (IOC)
Evidence of potential security incident. Can be IP addresses, file hashes, or domain names.

### Integration
Connecting security tools and systems. Enables centralized monitoring and automated responses.

### JMESPath
JSON query language used by AWS CLI. Enables filtering and transforming JSON output.

### JSON (JavaScript Object Notation)
Data format using key-value pairs. Standard for APIs and configuration.

### Latency
Time delay in system response. Important for real-time monitoring and alerting.

### Logstash
Open-source tool for processing and forwarding logs. Often used with Elasticsearch and Kibana (ELK Stack).

### Machine Learning
Automated learning from data to identify patterns and make predictions. Useful for anomaly detection.

### Metric
Quantifiable measurement of system performance or activity. Collected over time for trend analysis.

### Mitre ATT&CK
Framework documenting adversary tactics and techniques. Provides common vocabulary for threats.

### Observability
Ability to understand system state from external outputs (logs, metrics, traces). Key for incident response.

### Parsing
Process of extracting structured data from logs. Enables filtering and analysis.

### PCAP (Packet Capture)
Network traffic capture file format. Used for network forensics and traffic analysis.

### Query
Request for specific data from logs or metrics. Examples: Athena SQL, CloudWatch Logs Insights.

### Sampling
Collecting subset of events to reduce volume. Can miss low-probability events.

### SIEM (Security Information and Event Management)
Centralized system collecting and analyzing security events. Provides dashboard, alerting, and reporting.

### Splunk
Commercial SIEM platform for log collection, analysis, and alerting. Integrates with AWS.

### Stack Trace
Sequence of function calls at time of error. Helps identify where issues occur.

### Telemetry
Automated transmission of diagnostic data. Used for monitoring and troubleshooting.

### Time Series
Sequence of data points ordered by time. Used for tracking metrics and trends.

### UEBA (User and Entity Behavior Analytics)
Technology detecting unusual user/system behavior using machine learning. Identifies compromised accounts.

### VPC Flow Logs
Logs of network traffic to/from ENIs in VPC. Useful for troubleshooting and security analysis.

---

## Access Control & Identity

### Account
AWS account representing isolated security boundary. Root account has full access; should be secured with MFA.

### API Key
Credentials for programmatic API access. Should be rotated regularly and not hardcoded in applications.

### AssumeRole
Process of obtaining temporary credentials for a role. Used for cross-account access and federation.

### Attribute-Based Access Control (ABAC)
Access control based on attributes/tags rather than just role/identity. Provides finer-grained control.

### Cross-Account Access
Allowing principals in one account to access resources in another account. Requires role assumption and trust relationship.

### Credential
Information proving identity (passwords, access keys, certificates, etc.). Should be protected and rotated.

### Directory
Centralized repository of user identities and attributes. AWS Directory Service provides this functionality.

### External ID
Additional security mechanism preventing confused deputy problem. Used when assuming cross-account roles.

### Federation
Delegating authentication to external identity provider. Enables SSO and reduces password management burden.

### IAM Policy
JSON document defining permissions. Can be attached to users, roles, or resources.

### IAM Principal
Entity that can make requests to AWS (user, role, service principal). Identified by ARN.

### IAM Role
Identity with specific permissions that can be assumed by principals. Used for delegation and temporary access.

### IAM User
Individual with long-term credentials (access key, console password). Recommended for specific cases only.

### Identity Provider (IdP)
External system providing authentication. Examples: Active Directory, Okta, Google, Azure AD.

### Inline Policy
Permission policy directly attached to user/role. Not reusable; discouraged in favor of managed policies.

### LDAP (Lightweight Directory Access Protocol)
Protocol for accessing directory services. Often used for centralized identity management.

### Managed Policy
Reusable policy with own ARN and version history. Can be AWS-managed (maintained by AWS) or customer-managed.

### MFA Device
Physical device (hardware key, token generator) or software (authenticator app) generating time-based one-time passwords.

### OAuth 2.0
Open authorization standard enabling delegated access without sharing credentials. Used by many SaaS platforms.

### OpenID Connect (OIDC)
Extension of OAuth 2.0 adding authentication layer. Enables federation with external identity providers.

### Permissions Boundary
Maximum permissions an IAM entity can have. Acts as safety net for delegation.

### Principal
Entity (user, role, service, AWS account) that can make AWS API calls. Identified by ARN.

### Resource-Based Policy
Policy attached directly to a resource (S3 bucket, KMS key) rather than to a principal. Defines who can access what.

### Role Assumption
Process of a principal acquiring temporary credentials for a role. Requires trust relationship and permissions.

### Root Account
AWS account with full access and no restrictions. Should only be used for account setup; daily work should use IAM users/roles.

### SAML (Security Assertion Markup Language)
XML-based standard for authentication and authorization. Enables federation with enterprise identity systems.

### Secret
Sensitive credential requiring protection. AWS Secrets Manager encrypts and manages secrets.

### Service Control Policy (SCP)
Policy limiting maximum permissions available to accounts in AWS Organization. Provides additional security control layer.

### Service Principal
AWS service that can assume IAM roles. Examples: EC2, Lambda, CloudFormation.

### Session Token
Temporary credential issued after authentication. Expires after configured duration.

### Sign-In Credentials
Credentials used for console access (username + password/MFA). Different from programmatic access keys.

### Single Sign-On (SSO)
Authentication system enabling users to log in once and access multiple systems. Improves user experience and security.

### STS (Security Token Service)
AWS service providing temporary credentials. Used for session tokens, role assumption, and federation.

### Trusted Account
Account authorized to assume a role in another account. Trust relationship is configured in the role.

### User Pool
Directory of users managed by AWS Cognito. Enables user authentication for applications.

---

## Acronyms & Abbreviations

| Acronym | Full Form |
|---------|-----------|
| ACL | Access Control List |
| ADFS | Active Directory Federation Services |
| API | Application Programming Interface |
| ARN | Amazon Resource Name |
| ASG | Auto Scaling Group |
| AWS | Amazon Web Services |
| AZ | Availability Zone |
| BYOK | Bring Your Own Key |
| CA | Certificate Authority |
| CAVP | Cryptographic Algorithm Validation Program |
| CDN | Content Delivery Network |
| CEF | Common Event Format |
| CIDR | Classless Inter-Domain Routing |
| CLI | Command Line Interface |
| CMK | Customer Master Key |
| CORS | Cross-Origin Resource Sharing |
| COBIT | Control Objectives for Information and Related Technologies |
| CSA | Cloud Security Alliance |
| CSP | Cloud Service Provider |
| CSPM | Cloud Security Posture Management |
| CRL | Certificate Revocation List |
| CTF | Capture The Flag |
| DDoS | Distributed Denial of Service |
| DLP | Data Loss Prevention |
| DNS | Domain Name System |
| DNSSEC | DNS Security Extensions |
| DoS | Denial of Service |
| DR | Disaster Recovery |
| DRT | DDoS Response Team |
| DV | Domain Validation |
| EAR | Encryption At Rest |
| EBS | Elastic Block Store |
| EC2 | Elastic Compute Cloud |
| ECR | Elastic Container Registry |
| ECS | Elastic Container Service |
| EDR | Endpoint Detection and Response |
| EKS | Elastic Kubernetes Service |
| ELB | Elastic Load Balancer |
| ELK | Elasticsearch, Logstash, Kibana |
| ENI | Elastic Network Interface |
| FEDRAMP | Federal Risk and Authorization Management Program |
| FTP | File Transfer Protocol |
| GDPR | General Data Protection Regulation |
| GRE | Generic Routing Encapsulation |
| HA | High Availability |
| HIPAA | Health Insurance Portability and Accountability Act |
| HITRUST | Health Information Trust Alliance |
| HMAC | Hash-based Message Authentication Code |
| HTTPS | HyperText Transfer Protocol Secure |
| HUMINT | Human Intelligence |
| IaaS | Infrastructure as a Service |
| IAM | Identity and Access Management |
| IAST | Interactive Application Security Testing |
| IdP | Identity Provider |
| IDS | Intrusion Detection System |
| IOC | Indicator of Compromise |
| IP | Internet Protocol |
| IPsec | Internet Protocol Security |
| ISO | International Organization for Standardization |
| JSON | JavaScript Object Notation |
| JWE | JSON Web Encryption |
| JWT | JSON Web Token |
| KMS | Key Management Service |
| LDAP | Lightweight Directory Access Protocol |
| L7 | Layer 7 (Application Layer) |
| ML | Machine Learning |
| MTBF | Mean Time Between Failures |
| MTTR | Mean Time To Recovery |
| NAT | Network Address Translation |
| NIST | National Institute of Standards and Technology |
| NTP | Network Time Protocol |
| OAuth | Open Authorization |
| OIDC | OpenID Connect |
| OWASP | Open Web Application Security Project |
| PaaS | Platform as a Service |
| PCI-DSS | Payment Card Industry Data Security Standard |
| PCAP | Packet Capture |
| PEM | Privacy Enhanced Mail |
| PFS | Perfect Forward Secrecy |
| PKI | Public Key Infrastructure |
| PRNG | Pseudorandom Number Generator |
| PSK | Pre-Shared Key |
| RDS | Relational Database Service |
| REST | Representational State Transfer |
| RFC | Request for Comments |
| RNG | Random Number Generator |
| RTO | Recovery Time Objective |
| RPO | Recovery Point Objective |
| RTC | Real-Time Communication |
| S3 | Simple Storage Service |
| SAML | Security Assertion Markup Language |
| SCA | Static Code Analysis |
| SAST | Static Application Security Testing |
| SCM | Source Code Management |
| SCP | Service Control Policy |
| SDLC | Software Development Lifecycle |
| SDN | Software-Defined Network |
| SG | Security Group |
| SHA | Secure Hash Algorithm |
| SIEM | Security Information and Event Management |
| SIRT | Security Incident Response Team |
| SLA | Service Level Agreement |
| SNS | Simple Notification Service |
| SOC | Security Operations Center |
| SOC 2 | Service Organization Control 2 |
| SOAP | Simple Object Access Protocol |
| SQL | Structured Query Language |
| SSRF | Server-Side Request Forgery |
| SSH | Secure Shell |
| SSO | Single Sign-On |
| SSE | Server-Side Encryption |
| STS | Security Token Service |
| SUSE | SUSE Linux Enterprise |
| TDE | Transparent Data Encryption |
| TFA | Two-Factor Authentication |
| TLS | Transport Layer Security |
| TTL | Time To Live |
| UEBA | User and Entity Behavior Analytics |
| VPC | Virtual Private Cloud |
| VPN | Virtual Private Network |
| WAF | Web Application Firewall |
| YAML | YAML Ain't Markup Language |
| XSS | Cross-Site Scripting |
| YARA | Yet Another Recursive Acronym |
| ZTA | Zero-Trust Architecture |

---

## Cross-Reference Index

### By Category

**AWS Security Services:**
- CloudTrail, GuardDuty, Security Hub, AWS Config, AWS WAF, AWS Shield, Macie, Inspector, Access Analyzer

**AWS Core Services:**
- EC2, S3, VPC, RDS, Lambda, IAM, KMS, Route 53, CloudFormation, CloudWatch

**Networking & VPN:**
- VPN, WireGuard, OpenVPN, IPsec, BGP, DNS, VPC Peering, Transit Gateway

**Encryption & Cryptography:**
- AES, ChaCha20, TLS, SSL, SSH, PKI, Certificate, Forward Secrecy, Nonce

**Compliance & Standards:**
- PCI-DSS, HIPAA, SOC 2, ISO 27001, GDPR, CIS, NIST, FedRAMP

**Access Control:**
- IAM, AssumeRole, MFA, SAML, OAuth 2.0, OIDC, Federation

**Monitoring & Logging:**
- CloudTrail, CloudWatch, VPC Flow Logs, SIEM, Splunk, Athena Queries

---

## Pronunciation Guide

| Term | Pronunciation |
|------|-----------------|
| CIDR | "CIDER" |
| AWS | "Uh-Double-You-Ess" or "AWS" |
| IAM | "Eye-Am" or "I-A-M" |
| S3 | "S-3" or "S-Three" |
| VPC | "V-P-C" |
| EC2 | "E-C-2" |
| KMS | "K-M-S" |
| DNS | "D-N-S" or "Domain Name System" |
| NTP | "N-T-P" |
| OAuth | "O-Auth" (not "Oh-Auth") |
| GDPR | "G-D-P-R" |
| OWASP | "OH-Wasp" |
| NIST | "NIST" |
| PCI-DSS | "P-C-I-D-S-S" |
| HIPAA | "HIP-uh" |
| SOC2 | "Sock-Two" |
| SAML | "SAM-el" |
| WireGuard | "Wire-Guard" |
| OpenVPN | "Open-V-P-N" |

---

## Common Abbreviations in Code

| Abbreviation | Meaning | Example |
|--------------|---------|---------|
| env | Environment | `AWS_ENV=production` |
| cfg | Configuration | `cloudtrail-cfg.json` |
| id | Identifier | `--instance-id i-1234567890abcdef0` |
| arn | Amazon Resource Name | `arn:aws:s3:::bucket-name` |
| az | Availability Zone | `us-east-1a` |
| sg | Security Group | `sg-12345678` |
| vpc | Virtual Private Cloud | `vpc-12345678` |
| subnet | Subnet | `subnet-12345678` |
| acl | Access Control List | `--bucket-acl private` |
| kms | Key Management Service | `--sse-kms-key-id alias/key-name` |
| psk | Pre-Shared Key | `wg set peer PubKey PresharedKey...` |
| ttl | Time To Live | `--ttl 300` |
| rto | Recovery Time Objective | `--rto 4h` |
| rpo | Recovery Point Objective | `--rpo 1h` |

---

## Related Documentation

For more information on specific terms, refer to:

- **AWS Documentation**: https://docs.aws.amazon.com/
- **OWASP Guide**: https://owasp.org/www-project-top-ten/
- **NIST Cybersecurity Framework**: https://www.nist.gov/cyberframework/
- **CIS Benchmarks**: https://www.cisecurity.org/cis-benchmarks/
- **RFC Documents**: https://www.ietf.org/rfc/

---

## Index by Guide

**Guide 0 - AWS CLI Fundamentals:**
- AWS CLI, Access Key, Profile, Region, Output Format, JQ, Environment Variables

**Guide 1 - Logging & Visibility:**
- CloudTrail, AWS Config, Config Rules, CloudWatch, Athena, VPC Flow Logs, Audit Log

**Guide 2 - Threat Detection & Response:**
- GuardDuty, Security Hub, Indicators of Compromise, Machine Learning, Anomaly Detection, EventBridge

**Guide 3 - DDoS & Web Protection:**
- AWS Shield, AWS WAF, DDoS, Layer 7, Rate Limiting, Geo-blocking, SSL/TLS

**Guide 4 - S3 Security:**
- S3, Bucket Policies, Encryption, Versioning, MFA Delete, Access Logging, CORS

**Advanced Guide - VPN & Network Architecture:**
- VPN, WireGuard, OpenVPN, Onion Routing, Mesh Network, IPsec, TLS, DNS, Route 53

---

## Quick Reference by Use Case

### Setting Up Security Monitoring
- CloudTrail, AWS Config, CloudWatch, GuardDuty, Security Hub, VPC Flow Logs

### Protecting Web Applications
- AWS WAF, AWS Shield, HTTPS, Security Groups, Rate Limiting

### Securing Data
- Encryption, KMS, S3 Versioning, Access Controls, Audit Logging

### Managing Access
- IAM, MFA, Roles, Policies, Federation, SSO

### Incident Response
- CloudTrail, Logs, Forensics, SIEM, EventBridge, Lambda Automation

### Compliance
- CIS Benchmarks, AWS Config Rules, Audit Logs, Security Hub, Conformance Packs

### Network Security
- VPC, Security Groups, Network ACLs, VPN, WireGuard, Flow Logs

---

## Related Terms by Topic

### Encryption Methods
- Symmetric: AES, ChaCha20
- Asymmetric: RSA, ECC
- Hashing: SHA-256, SHA-512
- Protocols: TLS, IPsec, SSH

### Authentication Methods
- Knowledge: Passwords
- Possession: Hardware keys, MFA apps
- Inherence: Biometrics
- Federation: SAML, OAuth 2.0, OIDC

### Threat Types
- Reconnaissance: Scanning, Fingerprinting
- Exploitation: Code injection, Privilege escalation
- Malware: Trojans, Ransomware, Crypto-miners
- Social Engineering: Phishing, Pretexting

### AWS Service Families
- **Compute**: EC2, Lambda, ECS, EKS
- **Storage**: S3, EBS, EFS, Glacier
- **Database**: RDS, DynamoDB, Aurora
- **Networking**: VPC, Route 53, CloudFront, VPN
- **Security**: IAM, KMS, Secrets Manager, GuardDuty

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-07-06 | Initial release with terms from Guides 0-4 and Advanced |

---

*This appendix is a living document. Terms and definitions may be updated as AWS services evolve and new technologies emerge.*

*Last Updated: 2026-07-06*
