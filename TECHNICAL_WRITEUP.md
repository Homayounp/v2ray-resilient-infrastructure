Executive Summary
This document details the architecture, deployment, and operational challenges of building a V2Ray-based circumvention infrastructure serving 10-25 concurrent users across multiple ISPs in a high-censorship environment. The system evolved over 2+ years (2022-present) through iterative testing under active Deep Packet Inspection (DPI) and protocol-based blocking.

Key Technical Challenges Addressed:

Multi-ISP compatibility (mobile carriers vs. residential fiber)
SNI-based blocking and TLS fingerprinting
IP lifecycle management under traffic analysis
Protocol resilience during censorship escalation waves
1. Technical Architecture
1.1 Core Infrastructure Stack
┌─────────────────────────────────────────┐

│ CloudFlare DNS Layer │

│ (subdomain-based configuration) │

└──────────────┬──────────────────────────┘

│

┌──────────────▼──────────────────────────┐

│ SSL/TLS Termination (Let’s Encrypt) │

└──────────────┬──────────────────────────┘

│

┌──────────────▼──────────────────────────┐

│ 3x-ui Management Panel │

│ (github.com/MHSanaei/3x-ui) │

└──────────────┬──────────────────────────┘

│

┌──────────────▼──────────────────────────┐

│ V2Ray Core (Xray-core) │

│ Protocols: Reality, WebSocket, TCP │

└──────────────┬──────────────────────────┘

│

┌──────────────▼──────────────────────────┐

│ WARP Integration (Google/Spotify) │

└──────────────┬──────────────────────────┘

│

┌──────────────▼──────────────────────────┐

│ Hetzner VPS (Ubuntu 22.04) │

│ Google BBR (tested) │

└─────────────────────────────────────────┘

VPS Selection Criteria:

Provider: Hetzner (primary), Azure (initial testing)
OS: Ubuntu 22.04 LTS
IP Validation: Pre-deployment accessibility testing via SSH
Rotation Capability: Flexible IP allocation for proactive rotation
2. Deployment Process
2.1 Initial VPS Setup (Days 1-2)
Learning Curve:

The first deployment involved self-teaching basic VPS administration:

bash
# Essential commands learned during initial setup
apt update && apt upgrade -y
ufw allow 22/tcp
ufw allow 443/tcp
ufw allow [custom-ports]/tcp
ufw enable
Critical Discovery: Clean IP validation must occur before installing any circumvention software to avoid deploying on pre-blocked infrastructure.

2.2 V2Ray Installation & Configuration
Management Panel Deployment:

Used 3x-ui panel for multi-user configuration management
Simplified protocol testing and port allocation
Enabled non-technical user onboarding via QR codes/subscription links
SSL/TLS Setup:

bash
# Let's Encrypt certificate provisioning
certbot --nginx -d example.com -d sub.example.com
CloudFlare Integration:

DNS proxy enabled for subdomain strategy
Example: config.example.com instead of direct IP exposure
Mitigates basic IP-based blocking
2.3 Protocol Configuration Strategy
Reality Protocol (Primary - Post-2023)
Why Reality Won:

Survived major censorship escalation wave
TLS fingerprint mimicry proved most resilient against DPI
Outperformed WebSocket/TCP under active probing
SNI Target Selection Process:

Requirement: TLS 1.3 support + non-blocked domain
Validation Tool: qTLS Chrome Extension (link)
Working Targets: GitHub subdomains, Yahoo subdomains
Selection Method: Trial-and-error testing across multiple ISPs
Critical Lesson: Single SNI does not guarantee multi-ISP compatibility. Different carriers require different SNI targets.

Multi-Protocol Backup Strategy
Configuration Matrix:

Protocol	Use Case	ISP Compatibility
Reality	Primary (post-censorship wave)	High (with SNI tuning)
WebSocket	Fallback for specific ISPs	Medium
TCP	Legacy support	Low (deprecated)
3. Operational Challenges
3.1 Multi-ISP Compatibility Hell
The Problem:

Iranian ISPs (mobile carriers vs. residential fiber) implement different DPI strategies and blocking patterns.

Hardest ISP: MCI (Mobile Communications Company of Iran)

Solution Architecture:

Per-ISP Configuration Sets: Created different configs for different networks
Port Diversity: Forced different ports per user to bypass port-based blocks
SNI Variation: Mandatory SNI diversity to ensure universal connectivity
Example Scenario:

User A (Irancell mobile): Port 443, GitHub SNI
User B (MCI): Port 8443, Yahoo SNI
User C (Home fiber): Port 2053, GitHub subdomain SNI
3.2 IP Lifecycle Management
Blocking Pattern Observed:

Traffic Threshold: ~2TB usage over ~6 months triggers increased scrutiny
Strategy: Proactive rotation before blocks occur (vs. reactive rotation)
Rotation Process:

Deploy new VPS with clean IP
Migrate users gradually (test connectivity per ISP)
Decommission old VPS after full migration
Hetzner Advantage: No IP allocation limits (flexible scaling)
3.3 Security Trade-off: Iran Routing Rules
Implementation:

Integrated chocolate4u/Iran-v2ray-rules (link)
Purpose: Block VPN access to Iranian domestic websites
Rationale:

Prevents VPS IP from being logged on Iranian servers
Reduces risk of traffic correlation attacks
Split-tunneling: Iranian sites → direct connection, foreign sites → VPN
Configuration Example:

json
{
  "routing": {
    "domainStrategy": "IPIfNonMatch",
    "rules": [
      {
        "type": "field",
        "ip": ["geoip:ir"],
        "outboundTag": "block"
      }
    ]
  }
}
3.4 Performance Optimization (Google BBR)
Initial Hypothesis: Google BBR would improve throughput under high-latency conditions.

Reality: Performance decreased in most test cases.

Lesson: Always benchmark before deploying optimizations. “Best practices” may not apply in censored network conditions where packet loss patterns differ from standard internet behavior.

4. User Support & Distribution
4.1 Configuration Distribution
Methods Tested:

QR Codes: Fastest onboarding for mobile users
Subscription Links: Auto-update capability (critical during blocks)
Manual Config Files: Fallback for advanced users
Most Effective: Subscription links via 3x-ui panel (users auto-receive config updates during IP rotations)

4.2 Common User Issues
Top Support Request: “Why doesn’t it work on [specific ISP]?”

Root Cause: ISP-specific blocking patterns require ISP-specific configs

Support Workflow:

Identify user’s ISP
Provide ISP-tested config
If fails → test new port/SNI combination
Document working combination for future users on same ISP
5. Technical Lessons Learned
5.1 SNI Selection is Critical
Bad Approach: Use single popular SNI (e.g., cloudflare.com)

Good Approach:

Maintain pool of verified TLS 1.3 SNIs
Test across all target ISPs
Rotate SNIs during censorship escalation
Validation Tools:

qTLS extension for TLS version check
Manual testing across ISP test accounts
5.2 Port Diversity > Port Standardization
Standard Advice: Use port 443 (looks like HTTPS)

Reality in Censored Networks: Port 443 gets saturated with blocks first

Better Strategy: Distribute users across non-standard ports (8443, 2053, custom ports)

5.3 Proactive > Reactive IP Rotation
Reactive Strategy: Wait for IP block → scramble to deploy new VPS → users experience downtime

Proactive Strategy: Rotate at 80% of observed block threshold (~5 months, ~1.6TB) → zero downtime migrations

5.4 Protocol Evolution Under Pressure
Timeline:

2022: WebSocket/TCP sufficient
2023: Major censorship wave → Reality protocol adoption
2024: Reality remains most resilient
Key Insight: Protocol selection is not static. Censorship technology evolves → circumvention protocols must evolve faster.

6. System Metrics
Scale:

Concurrent Users: 10 (typical), 25 (peak)
Uptime: ~95% (excluding planned migrations)
IP Lifespan: ~6 months at 2TB usage
Support Time: ~2-3 hours/week (mostly ISP-specific troubleshooting)
Cost Analysis:

VPS: ~€5-10/month (Hetzner)
Domain/SSL: Minimal (Let’s Encrypt free)
Time Investment: ~20 hours initial setup, ~10 hours/month maintenance
7. Future Improvements
Identified Gaps:

Automated ISP Detection: Client-side script to auto-select optimal config based on detected ISP
Traffic Obfuscation: Explore additional obfuscation layers beyond Reality
Monitoring/Alerting: Automated IP health checks and proactive block detection
Load Balancing: Multi-VPS deployment for redundancy
Currently Exploring:

paqet protocol testing (next-generation circumvention)
8. Conclusion
Building resilient circumvention infrastructure requires:

Deep ISP-specific knowledge (no universal solutions)
Proactive lifecycle management (reactive = downtime)
Protocol diversity (single protocol = single point of failure)
Operational security (Iran routing rules, IP rotation)
Key Takeaway: Textbook deployments fail in censored environments. Real-world resilience comes from iterative testing, ISP-specific tuning, and operational discipline.

References
3x-ui Panel: https://github.com/MHSanaei/3x-ui
Iran V2Ray Rules: https://github.com/chocolate4u/Iran-v2ray-rules
qTLS Extension: https://chromewebstore.google.com/detail/qtls/mlhndmhjkoeggdhifdlglkldgppjifpf
