# V2Ray Resilient Infrastructure

Technical documentation and case study of building production V2Ray infrastructure in high-censorship environments.

## Overview

This repository documents the architecture, deployment process, and operational lessons learned from running a V2Ray-based circumvention system serving 10-25 concurrent users across multiple ISPs under active Deep Packet Inspection (DPI) and protocol-based blocking.

**Read the full technical writeup:** [TECHNICAL_WRITEUP.md](./TECHNICAL_WRITEUP.md)

## Key Technical Contributions

- Multi-ISP compatibility strategies (mobile carriers vs. residential fiber)
- SNI-based blocking circumvention using Reality protocol
- Proactive IP lifecycle management (rotation before blocks)
- Security trade-offs: Iran routing rules for operational security

## Technologies Used

- **V2Ray Core** (Xray-core)
- **Management:** 3x-ui panel
- **Protocols:** Reality (primary), WebSocket, TCP
- **Infrastructure:** Hetzner VPS, Ubuntu 22.04, CloudFlare DNS
- **Routing Rules:** chocolate4u/Iran-v2ray-rules
- **Extras:** WARP integration, Google BBR testing

## Project Timeline

- **Started:** 2022
- **Status:** Active production deployment
- **Scale:** 10-25 concurrent users
- **Uptime:** ~95% (excluding planned migrations)

## Documentation Structure

1. **Technical Architecture** - Infrastructure stack and design decisions
2. **Deployment Process** - Step-by-step setup and configuration
3. **Operational Challenges** - Multi-ISP compatibility, IP management, security
4. **Lessons Learned** - SNI selection, port diversity, protocol evolution
5. **Metrics** - Scale, uptime, cost analysis

## Related Projects

- [Byte-Sized-Security](https://github.com/Homayounp/Byte-Sized-Security) - Security CTF challenges (OverTheWire Bandit)

## License

MIT License - See LICENSE file for details
