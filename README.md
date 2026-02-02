# TechBird TB Suite - Enterprise Hospitality & Travel Platform

## 🏨 Overview

**TechBird TB Suite** is a comprehensive, modular, and industry-ready platform for hospitality and travel businesses. Built on Frappe Framework v15 and ERPNext, it provides enterprise-grade solutions for hotels, restaurants, and travel management.

**Publisher:** TechBirdIt.in  
**License:** Proprietary  
**Framework:** Frappe v15 + ERPNext v15  

## 🎯 Key Features

- **Multi-Property Management** - Centralized control across multiple properties and companies
- **Modular Architecture** - Enable only what you need: PMS, Restaurant, Travel, or all
- **Industry-Ready** - Generic, configurable for any hospitality business
- **Integrated Ecosystem** - Seamless integration with OTAs, GDS, payment gateways
- **Enterprise Scale** - Built for single boutique hotels to large chains

## 📦 TB Suite Modules

| Module | Description | Status |
|--------|-------------|--------|
| **tb_hotel_core** | Properties, Rooms, Rate Plans, Banquets, Licenses | 🟢 Core |
| **tb_hotel_management** | Multi-property control, analytics, HO services | 🟢 Core |
| **tb_hotel_pms** | Front desk operations, check-in/out, housekeeping | 🟢 Core |
| **tb_restaurant_pos** | Front-of-house POS, KOT, billing, payments | 🟡 Essential |
| **tb_restaurant_management** | Back-of-house, menus, recipes, inventory | 🟡 Essential |
| **tb_travel_management** | Flights, hotels, activities booking | 🔵 Optional |
| **tb_gds_integrations** | Amadeus & GDS provider interfaces | 🔵 Optional |
| **tb_channel_bridge** | OTA/STAAH integration | 🔵 Optional |
| **tb_billing** | Generic invoicing and payments | ⚪ Scaffold |
| **tb_crm_reservations** | CRM + guest/corporate accounts | ⚪ Scaffold |

## 🚀 Quick Start

```bash
# Clone TB Suite
git clone https://github.com/ekanshjain/tb_suite.git
cd tb_suite

# Initialize bench
bench init tb-bench --frappe-branch version-15
cd tb-bench

# Get TB Suite apps
bench get-app ../tb_suite/apps/tb_hotel_core
bench get-app ../tb_suite/apps/tb_hotel_pms
bench get-app ../tb_suite/apps/tb_restaurant_pos
# ... add modules as needed

# Create site
bench new-site hotel.local
bench --site hotel.local install-app tb_hotel_core
bench --site hotel.local install-app tb_hotel_pms

# Start
bench start
```

Access at: http://localhost:8001

## 📚 Documentation

- **[Architecture](ARCHITECTURE.md)** - System design and module breakdown
- **[Onboarding Wizard](ONBOARDING_WIZARD.md)** - Multi-step setup guide
- **[Modules Overview](MODULES_OVERVIEW.md)** - Detailed module descriptions
- **[Integrations](INTEGRATIONS.md)** - External system integration guides
- **[API Reference](API_REFERENCE.md)** - Complete API documentation
- **[Development Guide](DEV_GUIDE.md)** - Development standards and practices
- **[Deployment](DEPLOYMENT.md)** - Production deployment guide
- **[UI/UX Guide](UI_UX_GUIDE.md)** - Design system and components
- **[Roadmap](ROADMAP.md)** - Development milestones and timeline

## 🏗️ Repository Structure

```
tb_suite/
├── apps/                    # TB Suite Applications
│   ├── tb_hotel_core/      # Core hotel entities
│   ├── tb_hotel_management/# Multi-property management
│   ├── tb_hotel_pms/       # Property management system
│   ├── tb_restaurant_pos/  # Restaurant POS
│   ├── tb_restaurant_management/
│   ├── tb_travel_management/
│   ├── tb_gds_integrations/
│   ├── tb_channel_bridge/
│   ├── tb_billing/
│   └── tb_crm_reservations/
├── docs/                    # Documentation
├── scripts/                 # Setup and utility scripts
└── tests/                   # Test suites
```

## 🔧 System Requirements

**Minimum Requirements:**
- Python 3.10+
- Node.js 18+
- MariaDB 10.6+ / MySQL 8.0+
- Redis 7.0+
- 4GB RAM
- 20GB Storage

**Production Requirements:**
- 8+ CPU cores
- 16GB+ RAM
- 100GB+ SSD
- Ubuntu 22.04 LTS

## 🌟 Reference Implementations

The TB Suite powers these production deployments:
- **StayBird Hotels** - Multi-property hotel chain
- **TranqVillas** - Boutique resort management
- **StarBird Travel** - Corporate travel platform

## 🤝 Support & Contact

**Publisher:** TechBirdIt.in  
**Email:** support@techbirdit.in  
**GitHub:** https://github.com/ekanshjain/tb_suite  
**Documentation:** https://docs.techbirdit.in  

## 📄 License

Proprietary software by TechBirdIt.in. All rights reserved.

---

*TB Suite - Enterprise-ready hospitality & travel platform*
