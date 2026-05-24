+++
title = 'General security concepts'
date = '2026-05-24T08:17:58-04:00'
weight = 10
draft = false
+++

## Compare security controls

### Control categories

A *security control category* describes the domain or layer of the organization in which a control operates. The four categories are technical, managerial, operational, and physical.

*Objective 1.1: Compare and contrast various types of security controls.*

#### Technical

*Technical controls* (also called *logical controls*) use technology to protect systems and data. They are implemented in hardware, software, or firmware.

Examples: firewalls, data encryption, access control lists, intrusion detection systems, multi-factor authentication, antivirus software.

#### Managerial

*Managerial controls* (also called *administrative controls*) address security through policies, procedures, and governance. They shape how people make decisions and behave within the organization.

Examples: code of conduct, performance reviews, risk assessments, security policies, background checks, acceptable use policies.

#### Operational

*Operational controls* are carried out by people rather than technology. They focus on day-to-day security processes and human-driven procedures.

Examples: incident response procedures, security awareness training, user access management, change management procedures, media disposal processes.

#### Physical

*Physical controls* protect the physical environment and restrict unauthorized access to facilities, equipment, and assets.

Examples: access control vestibules, biometric locks, security guards, fences, closed-circuit television (CCTV), mantraps, vehicle barriers, tamper-evident seals, panic buttons.

### Control types

A *security control type* describes the intended function or effect of a control. The six types are preventive, deterrent, detective, corrective, compensating, and directive.

#### Preventive

*Preventive controls* stop a security incident before it occurs. They block unauthorized actions or access.

Examples: firewalls, encryption, multi-factor authentication, access control lists, security awareness training.

#### Deterrent

*Deterrent controls* discourage potential attackers by making an attack less appealing or more risky. They do not technically prevent an attack but reduce the likelihood that one is attempted.

Examples: warning banners, visible security cameras, security guard presence, posted policies.

#### Detective

*Detective controls* identify and record security incidents after they occur or while they are in progress. They uncover issues and anomalies to initiate corrective actions.

Examples: intrusion detection systems (IDS), security information and event management (SIEM) systems, audit logs, video surveillance review.

#### Corrective

*Corrective controls* reduce the impact of an incident after it has occurred and help restore normal operations.

Examples: patch management, incident response plans, system backups, disaster recovery procedures.

#### Compensating

*Compensating controls* are alternative measures used when a primary control cannot be implemented. They provide an equivalent level of protection through a different means.

Examples: using network segmentation to isolate an unpatched legacy system when patching is not possible; using a secondary authentication method when the primary authentication method fails.

#### Directive

*Directive controls* specify required behavior through policy, regulation, or guidance. They direct people to act in a secure manner.

Examples: acceptable use policies, security training requirements, data classification policies, standard operating procedures (SOPs), regulatory compliance mandates such as HIPAA or PCI DSS.


## Fundamental security concepts

### CIA triad

The *CIA triad* is the foundational model for information security. It stands for Confidentiality, Integrity, and Availability.

#### Confidentiality

*Confidentiality* ensures that information is accessible only to those authorized to view it. It protects sensitive data from unauthorized disclosure. Types of information confidentiality protects include:

- *Personally identifiable information (PII):* names, addresses, Social Security numbers
- *Protected health information (PHI):* medical records, diagnoses
- *Financial data:* credit card numbers, bank account details
- *Intellectual property:* trade secrets, proprietary source code
- *Sensitive business data:* contracts, strategic plans

Examples of controls: data encryption, access controls, multi-factor authentication, data classification policies.

#### Integrity

*Integrity* ensures that information is accurate and has not been altered without authorization. It protects data from unauthorized modification.

Examples: hashing algorithms (SHA-256), digital signatures, file integrity monitoring, checksums.

#### Availability

*Availability* ensures that systems and data are accessible to authorized users when needed. It protects against disruptions to legitimate access.

Examples: redundant systems, load balancers, backups, disaster recovery plans, uninterruptible power supplies (UPS).

### Non-repudiation

*Non-repudiation* is the assurance that a party cannot deny having performed an action or sent a message. It provides proof of origin and delivery, and is critical for electronic transactions and communications such as contracts, financial transfers, and email exchanges.

#### Digital signatures

*Digital signatures* use asymmetric cryptography to bind a sender's identity to a message or document. The sender encrypts a hash of the message with their private key; the recipient decrypts it with the sender's public key to verify authenticity.

Examples: signing emails with S/MIME, signing code releases, signing legal documents electronically.

#### Audit trails

*Audit trails* are chronological records of system activity that document who performed an action and when. They provide evidence of actions taken and support forensic investigation.

Examples: system event logs, database transaction logs, authentication logs, file access logs.

#### Access controls

*Access controls* restrict what users can see and do within a system. They enforce non-repudiation by tying actions to verified identities. Access control consists of three steps: identification, authentication, and authorization.

Identification
: The process of claiming an identity. A user presents a unique identifier to a system. Examples: username, email address, employee ID, smart card. In Windows environments, the system assigns each account a *Security Identifier (SID)*—a unique value used internally to identify the entity. The system tracks permissions and access by SID, not by username, so the SID persists even if the username changes.

Authentication
: The process of proving that the claimed identity is genuine. The system verifies the identifier against stored credentials. Examples: password, PIN, fingerprint scan, one-time password (OTP), multi-factor authentication (MFA).

Authorization
: Determines what an authenticated user is permitted to do. The system grants or denies access to resources based on the user's permissions. Examples: role-based access control (RBAC), access control lists (ACLs), file permissions, least privilege policies.

### Authentication, Authorization, and Accounting

An *AAA server* is a network server that centralizes authentication, authorization, and accounting services. It controls access to network resources by verifying identities, enforcing permissions, and logging activity.

#### Authenticating people

The AAA server verifies user identities by checking credentials against a directory service. In Windows environments, it interfaces with a *domain controller (DC)*. A domain controller is a server running Windows Server that manages authentication and authorization for all users, computers, and resources in a Windows domain. It hosts Active Directory (AD), the directory service that stores account credentials, group policies, and organizational structure. When a user attempts to log in, the AAA server forwards the authentication request to the domain controller, which checks the submitted credentials against AD. If they match, the domain controller returns a success response and the AAA server grants access. If the domain controller is unavailable, domain authentication fails.

#### Authenticating systems

The AAA server authenticates devices using the *IEEE 802.1X* standard, a port-based Network Access Control (NAC) protocol. It uses the Extensible Authentication Protocol (EAP) to pass credentials between three entities:

- *Supplicant:* the client device requesting network access
- *Authenticator:* the network device (switch or wireless access point) that enforces access control
- *Authentication server:* the AAA server that validates credentials

When a device connects to a network port, the authenticator blocks all traffic except EAP messages until the AAA server authenticates the supplicant. Only then does the authenticator open the port and allow full network access.

#### Authorization models

After authenticating a user or device, the AAA server determines what actions they can perform on the network. Authorization policies can restrict:

- Which network segments a user can access
- What time of day access is permitted
- Which applications or services a user can use
- Which commands a user can execute on network devices

#### Accounting

The AAA server logs user activity for auditing, billing, and compliance. Details captured include:

- Login and logout times
- Session duration
- Volume of data transferred
- Resources accessed
- Commands executed

This data is stored in logs on the AAA server and can be forwarded to a security information and event management (SIEM) system for centralized analysis and alerting.

#### AAA protocols

*RADIUS*, *Diameter*, and *TACACS+* are the three primary AAA protocols. All three provide authentication, authorization, and accounting services, but they differ in architecture, transport, and use case.

##### RADIUS

*Remote Authentication Dial-In User Service (RADIUS)* is the most widely deployed AAA protocol for network access. RADIUS combines authentication and authorization into a single process and uses UDP for transport (port 1812 for authentication, port 1813 for accounting). It encrypts only the password field in the authentication request, leaving other attributes in plaintext. RADIUS is commonly used for Wi-Fi, VPN, and dial-up access.

##### Diameter

*Diameter* is the successor to RADIUS (the name is a mathematical reference: a diameter is larger than a radius). It uses TCP or SCTP for reliable transport, supports more robust error handling and failover, and encrypts the full payload rather than just the password. Diameter is used in carrier-grade and mobile networks, including 4G/LTE base stations and WiMAX access points. It can interoperate with RADIUS environments through gateway translation.

##### TACACS+

*Terminal Access Controller Access-Control System Plus (TACACS+)* was developed by Cisco and is widely supported across network vendors. Unlike RADIUS, TACACS+ separates authentication, authorization, and accounting into distinct processes, allowing more granular control over each. It uses TCP (port 49) for reliable transport and encrypts the entire communication payload. TACACS+ is the preferred protocol for managing network infrastructure devices such as routers and switches, where per-command authorization is required.

### Gap analysis

A *gap analysis* evaluates an organization's security practices against established standards, regulations, and industry best practices. It identifies where current controls fall short and guides improvements.

#### Assessment

The assessment examines the current state of the organization's security controls, policies, and procedures. It involves reviewing documentation, interviewing staff, and testing systems to build an accurate picture of what protections are in place and how they are implemented.

#### Benchmarking

Benchmarking compares the organization's current security posture against an established framework or standard such as NIST CSF, ISO 27001, or CIS Controls. The chosen benchmark defines what an acceptable security posture looks like and provides a measurable baseline for identifying gaps.

#### Identification

Using the results of the assessment and benchmarking, the organization identifies specific areas where its practices fall short of the target standard. Each identified gap represents a security risk, a compliance shortfall, or both.

#### Prioritization

Identified gaps are ranked by risk level, potential business impact, and remediation effort. Gaps with high exploitation likelihood or regulatory compliance deadlines are addressed first. Prioritization ensures limited resources are applied where they reduce the most risk.

#### Remediation strategy

The remediation strategy defines a plan to close prioritized gaps. It specifies the actions required, assigns ownership to responsible parties, sets target completion dates, and allocates budget and resources. Actions may include implementing new controls, updating policies, or delivering security awareness training.

### Zero trust

Zero trust is a security model built on the principle of "never trust, always verify." Rather than assuming that users and devices inside a network perimeter are safe, zero trust requires continuous validation of every access request regardless of where it originates. Every user, device, and connection must prove legitimacy before accessing any resource.

#### Data plane and control plane

Zero trust architecture separates network functions into two distinct planes: the *control plane* and the *data plane*. The term "plane" comes from networking architecture, where it describes a distinct functional layer of a system, similar to how a physical plane separates different levels of space. Each plane handles a different concern:

- *Data plane:* Ensures the efficient movement of information between endpoints and enforces access rules set by the control plane.
- *Control plane:* Manages the intelligence behind data routing, network health, and device coordination. It evaluates identity, applies policies, and determines whether to grant or deny access.

Separating these planes prevents vulnerabilities that arise from tight coupling. When both functions run in the same system, a compromise of the data-handling layer can expose the policy decision layer. Separation means the policy engine can be hardened and audited independently of data forwarding. An attacker who compromises a data path cannot automatically influence access decisions, and each plane can scale and be updated without affecting the other.

##### Centralizing control

Centralizing intelligence in the control plane gives the organization a single, authoritative source for access decisions. Policy changes propagate instantly to all enforcement points without requiring updates to individual network devices. Auditing and visibility are simplified because all decisions flow through one layer. Network devices in the data plane handle only forwarding, which reduces their complexity and attack surface. Enforcement points can scale independently of policy logic because they rely on the control plane for decisions rather than making them locally.

#### Data plane

The *data plane* is responsible for the actual movement and forwarding of data packets within a network. It handles tasks such as routing, switching, and packet forwarding based on rules and policies established by the control plane. The data plane ensures efficient and secure data transmission between devices and across networks.

##### Subject

A *subject* is any user, device, or application requesting access to a resource. In zero trust, every subject must be authenticated and authorized before access is granted.

Examples: an employee logging in from a laptop, a mobile device connecting to a corporate application, a service account making an API call.

##### System

A *system* refers to the network infrastructure that moves and manages data packets. Systems are the physical and virtual components that the data plane relies on to forward traffic.

Examples: routers, switches, firewalls, load balancers.

#### Trust zones

*Trust zones* are network segments assigned a level of trust that determines how traffic entering and leaving them is treated. They exist to enforce least privilege at the network level by limiting what can communicate with what. Each zone is protected by controls that inspect and filter traffic crossing its boundary.

##### Implicit trust zone

The *implicit trust zone* (also called the *trusted zone* or *secure zone*) is the area of the network where access has been explicitly granted and communication is actively occurring. In traditional architectures, the entire internal network functioned as an implicit trust zone: anything inside the perimeter was trusted by default. Zero trust eliminates this assumption, shrinking the implicit trust zone to the smallest possible segment between the policy enforcement point and the resource being accessed.

Examples: a specific application segment that a verified user has been granted access to.

##### Internal network zone

The *internal network zone* (also called the *local area network (LAN)*, *intranet zone*, or *corporate network zone*) is the organization's private network, sitting behind the corporate firewall. In traditional architectures, this zone was considered inherently trusted. In zero trust, it receives the same scrutiny as any other zone. Users and devices on the internal network must still authenticate and be authorized before accessing resources.

Examples: corporate LAN, internal servers, on-premises data centers.

##### DMZ

The *demilitarized zone (DMZ)* is a network segment that sits between the internal network and the external network. It hosts services that must be reachable from the internet while keeping those services isolated from the internal network. Also called a *perimeter network* or *screened subnet*.

Examples: public-facing web servers, email gateways, DNS servers, VPN concentrators.

##### External network zone

The *external network zone* (also called the *untrusted zone* or *internet zone*) is any network outside the organization's control. All traffic from it is subject to strict inspection and filtering.

Examples: the public internet, partner networks, cloud provider networks.

#### Control plane

The control plane is the decision-making layer of a zero trust architecture. It houses the policy engine and policy administrator, which evaluate trust signals and issue authorization decisions. The policy engine assesses each access request using identity, device health, threat intelligence, and organizational policy. It passes its decision to the policy administrator, which translates the decision into specific instructions for the policy enforcement point. The enforcement point then permits or blocks the connection. This flow ensures that no implicit trust is assumed and that every access decision is deliberate, policy-driven, and continuously re-evaluated.

The concept of an *implicit trust zone* (the assumption that anything inside the network is safe) is explicitly rejected in zero trust. Every request is treated as potentially hostile until the policy engine validates it.

##### Control plane

The *control plane* is the layer responsible for making and communicating access decisions. It does not carry user data. Instead, it evaluates trust signals and instructs enforcement points on how to handle each connection request.

##### Adaptive identity

*Adaptive identity* continuously evaluates user and device identity based on contextual signals such as location, device health, behavior patterns, and time of access. Rather than granting access based on a one-time authentication event, adaptive identity reassesses trust throughout the session and can trigger step-up authentication or revoke access if conditions change.

##### Threat scope reduction

*Threat scope reduction* limits the blast radius of a potential compromise by restricting what any single user, device, or application can access. Strategies include minimizing exposed services, reducing the attackable code base, and employing rigorous patch management. If an attacker gains access, they can reach only the resources explicitly permitted to that identity.

##### Policy-driven access control

*Policy-driven access control* bases access decisions on defined policies rather than network location. Policies define access rights, permissions, and responses to specific scenarios. No location receives automatic trust.

##### Policy administrator

The *policy administrator* executes decisions made by the policy engine to control access to the network. It translates trust decisions into actionable instructions such as issuing session tokens or configuring authentication credentials. It can also revoke access mid-session if the policy engine signals a change in trust level.

##### Policy engine

The *policy engine* is the core decision-maker of the control plane. It evaluates each access request by analyzing identity, device health, threat intelligence, and organizational policy. It produces a trust decision—allow, deny, or conditional—and passes that decision to the policy administrator.

> **Note:** The policy engine evaluates company policies alongside real-time threat intelligence data to make access decisions on a per-user basis. No user receives blanket access—every request is assessed individually against both internal policy and current threat context.

##### Policy enforcement point

The *policy enforcement point (PEP)* is the operational gateway that permits or blocks access to resources based on instructions from the policy administrator. Every access request passes through the PEP and is double-checked by the policy engine to ensure it complies with current policy before the connection is allowed.

### Physical security

*Physical security* is a framework that safeguards people, assets, and critical information by controlling access to facilities and detecting unauthorized activity. The following controls work together to create layered physical protection.

#### Bollards

*Bollards* are short, sturdy vertical posts installed around building perimeters and entry points to prevent vehicles from entering restricted areas. They protect facilities and pedestrians from vehicle-borne attacks and accidental collisions. Bollards are available in fixed, removable, and retractable configurations, allowing organizations to balance security with operational access needs.

#### Access control vestibule

An *access control vestibule* (also called a *mantrap* or *airlock*) is an enclosed entry chamber with two interlocking doors. The first door must close and lock before the second opens, ensuring only one person passes through at a time. This prevents tailgating (an unauthorized person following an authorized one through a secured door) and forces each individual to present credentials before entering the secured area.

#### Fencing

*Fencing* defines the physical perimeter of a secured area and deters or delays unauthorized entry. Height, material, and design affect its effectiveness. Features such as barbed wire, anti-climb coatings, and chain-link construction increase resistance to intrusion. Fencing also channels visitors toward controlled entry points where guards and access controls can verify identity.

#### Video surveillance

*Video surveillance* uses cameras to monitor and record activity in and around a facility. It serves as both a deterrent and a detective control. Modern systems include IP cameras, pan-tilt-zoom (PTZ) cameras, and analytics software for motion detection. Recorded footage supports incident investigation and provides evidence for forensic review.

#### Security guard

Security guards monitor access points, patrol facilities, and respond to incidents. Their visible presence deters criminal activity. Unlike automated controls, security guards can exercise judgment, identify suspicious behavior, and take immediate action during emergencies.

#### Access badge

An *access badge* is a credential such as a smart card, RFID card, or proximity card that grants access to secured areas. Each badge is tied to an individual identity, creating an audit trail of who accessed which areas and when. Badges that are lost or stolen can be quickly deactivated to revoke access.

#### Lighting

Adequate lighting deters criminal activity by eliminating concealment opportunities. Well-lit areas improve the effectiveness of video surveillance and make it easier for security guards to monitor the environment. Motion-activated lighting alerts staff to unexpected activity in normally unoccupied areas.

#### Visitor logs

*Visitor logs* record individuals who enter a facility. Each entry captures the visitor's name, purpose of visit, time in, time out, and the employee they are visiting. Visitor logs support accountability and provide evidence to support investigation after a security incident.

#### Sensor technologies

Sensor technologies detect unauthorized movement or presence within a secured area and trigger alarms or alerts when activity is detected.

##### Infrared

*Infrared sensors* detect heat signatures emitted by people and animals. Passive infrared (PIR) sensors monitor changes in infrared radiation caused by movement. They are commonly used in motion detectors and alarm systems and are effective for detecting human presence in indoor environments.

##### Pressure

*Pressure sensors* detect weight or force applied to a surface such as a floor mat or pressure plate. When a person steps on the monitored surface, the sensor triggers an alarm. They are typically installed under carpets or at entry points to detect unauthorized access.

##### Microwave

*Microwave sensors* emit microwave pulses and measure the reflected signal. Movement within the detection field alters the reflection pattern and triggers an alert. Microwave sensors can cover large areas and penetrate non-metallic materials, making them more difficult to defeat than infrared sensors.

##### Ultrasonic

*Ultrasonic sensors* emit high-frequency sound waves and detect changes in the reflected pattern caused by movement. They are effective for monitoring motion in enclosed spaces and are commonly used in interior intrusion detection systems.

### Deception and disruption technology

*Deception and disruption technology* is a modern defensive approach that actively deceives and disrupts potential threats using digital decoys. Unlike passive defenses that simply block attacks, this dynamic method disorients attackers, exposes their techniques, and generates actionable threat intelligence without exposing real systems or data.

#### Honeypot

A *honeypot* is a decoy system configured to resemble a legitimate target. It attracts attackers by appearing to contain valuable resources, then records their activity without putting real systems at risk. Security teams analyze the recorded behavior to identify attack techniques, tools, and objectives. Honeypots also serve as an early warning system: any interaction with a honeypot indicates malicious activity, since legitimate users have no reason to access it.

#### Honeynet

A *honeynet* is a network of interconnected honeypots that simulates an entire organizational network environment. It includes decoy servers, workstations, and network devices arranged to resemble a realistic infrastructure. Because the honeynet contains no legitimate traffic, any activity within it is inherently suspicious. Honeynets engage attackers for longer periods than a single honeypot and gather more detailed intelligence about attacker behavior, lateral movement techniques, and persistence methods.

#### Honeyfile

A *honeyfile* is a decoy file placed within a real file system to detect unauthorized access. The file is named and positioned to appear valuable (for example, "passwords.xlsx" or "payroll_2025.xlsx") but contains no real data. Any access to the honeyfile triggers an immediate alert, providing a clear indicator of compromise. Honeyfiles are effective for detecting both external attackers and malicious insiders.

#### Honeytoken

A *honeytoken* is a fake credential or piece of data embedded within real systems to detect theft and unauthorized use. When an attacker obtains and attempts to use the token, the system generates an alert revealing that credentials have been compromised. Honeytokens can take many forms, including fake API keys, fabricated database records, and planted username and password combinations in configuration files or code repositories.

#### Fake information

*Fake information* consists of deliberately false data seeded throughout systems to mislead attackers. If an attacker exfiltrates or acts on fake information, their actions become detectable and their intelligence is corrupted, reducing the effectiveness of further attacks. Fake information may include fabricated network topology maps, false user accounts, and fictitious customer records designed to appear genuine.

A *DNS sinkhole* is a form of fake information applied at the network level. When a device attempts to resolve a domain known to be malicious (such as a command-and-control server or phishing site), the DNS sinkhole returns a false IP address pointing to a controlled server instead of the real destination. This cuts off communication between infected devices and attacker infrastructure. The sinkhole server logs all requests, revealing which devices on the network are attempting to contact malicious domains—a clear indicator of compromise.