

### Tool Security Evaluation: Threat Identification

Transitioning back to pipeline management, the Discord bot acts as a bridge between a public-facing communication app and our proprietary source code repository. Because it transmits binary data and metadata over an external network, it introduces several critical attack vectors that required auditing.

* **Data Transmission & Exposure Risks:** The bot processes proprietary 3D models, textures, animations, and now raw audio stems. While it does not process sensitive *player* data, it handles confidential project data. If the transmission is intercepted, unreleased game assets could be leaked.
* **External Attackers & Compromise:** The highest severity risk is the compromise of the GitHub Personal Access Tokens (PATs). If an external attacker gains access to the `.env` file (e.g., through a compromise of the host machine), they would gain authorized write access to our GitHub repository. They could maliciously delete the codebase, inject compromised code, or hold the repository ransom.
* **Internal Team Misuse:** Without proper safeguards, an internal team member could intentionally or accidentally spam the `/upload` command with massive, irrelevant files, effectively launching a Denial of Service (DoS) attack on our repository's storage limits.
* **Misconfiguration:** Accidental hardcoding of API keys into a public branch or failing to restrict branch protection rules could lead to unintentional data leakage or unauthorized merges into the `main` production branch.

### Proposed Technical Mitigations

To secure the pipeline against these identified threats, several technical measures and protocols must be enforced before full studio deployment:

1. **Protocol and Encryption (Data in Transit):** All data transmitted between the Discord API, the host machine, and the GitHub API must be strictly routed over **HTTPS** utilizing TLS 1.2 or higher. While libraries like `discord.py` and `PyGithub` default to HTTPS, we must ensure no HTTP fallback is permitted to prevent Man-in-the-Middle (MitM) packet sniffing.
2. **Authentication and Access Control (RBAC):** We must implement Discord Role-Based Access Control (RBAC) *(Discord, s.d.)*. The `/upload` slash command should be restricted so it can only be executed by users holding the `@Artist`, `@Audio`, or `@Lead` roles. Furthermore, the GitHub Classic PATs should be deprecated and replaced with **Fine-Grained Access Tokens**, scoped explicitly with "Read/Write" access *only* to the `assets/dropoff` branch.
3. **Logging and Auditing:** A formalized logging system using Python's native `logging` module must be implemented. Every transaction should log the user's Discord ID, the timestamp, and the file hash. This creates a non-repudiable audit trail if an asset breaks the build.
4. **Compliance and Data Minimisation (GDPR):** In adherence to GDPR data minimisation principles *(ICO, s.d.)*, the bot must only process data strictly necessary for the transaction. It will temporarily cache the file in memory, push it, and immediately drop the payload. We will only log the user's randomized Discord ID for auditing, stripping any identifiable personal information (names, IP addresses) from the logs.

### Deployment Architecture: The Case for Cloud Networking

Currently, the bot operates as a localized script running on my personal terminal. While functional for prototyping, this introduces significant risk. A compromise of my personal workstation means a compromise of the pipeline.

**Does networking/cloud deployment add value?** Absolutely. Transitioning this tool from a local script to a **containerised deployment** (e.g., Docker) hosted on a dedicated authoritative service (like AWS or Heroku) dramatically reduces security risks *(Shostack, 2014)*. 
* **Trust & Isolation:** A containerized environment isolates the bot's execution from other localized software vulnerabilities. 
* **Centralised Validation:** A cloud host allows for centralized validation of API requests and guarantees 24/7 uptime for the art and audio teams, removing the dependency on my personal hardware being active. 

Ultimately, migrating to a secure, containerized client-server model adds immense value by standardizing the security environment and protecting the project's core repository from localized vulnerabilities.

---

# BIBLIOGRAPHY

*(In order they appear in the writeup)*

Discord (s.d.) *Command Permissions - Discord Developer Portal*. At: https://discord.com/developers/docs/interactions/application-commands#permissions (Accessed 17/04/2026).

Information Commissioner's Office (ICO) (s.d.) *Principle (c): Data minimisation*. At: https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/data-protection-principles/a-guide-to-the-data-protection-principles/the-principles/data-minimisation/ (Accessed 17/04/2026).

Shostack, A. (2014) *Threat Modeling: Designing for Security*. Indianapolis: Wiley.