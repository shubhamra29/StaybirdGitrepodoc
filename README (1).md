# TechBird TB Suite 🏨🤖

An AI-enhanced, modular hospitality and travel platform built on Frappe Framework with integrated MCP (Model Context Protocol) servers for intelligent operations.

## 🌟 AI-Enhanced Features

- **🤖 AI-Powered Operations** - Sequential thinking and intelligent decision-making
- **🏨 Multi-Property Management** - Centralized control with AI insights
- **📊 Smart Analytics** - Dynamic charts and predictive analytics
- **✈️ Intelligent Travel Planning** - AI-assisted trip planning with weather/events
- **💰 Revenue Optimization** - AI-driven pricing strategies
- **🍽️ Smart Restaurant POS** - Intelligent order management and cost optimization
- **🏠 Automated Housekeeping** - AI-optimized room status and task management
- **📈 Predictive Inventory** - Smart stock management with demand forecasting
- **🔗 Channel Intelligence** - AI-enhanced OTA management
- **👤 Personalized Guest Experience** - AI-driven guest preferences and service

## 🚀 Quick Start (Enhanced with MCP)

```bash
# Clone the TB Suite
git clone https://github.com/ekanshjain/tb_suite.git
cd tb_suite

# Run enhanced setup with MCP integration
chmod +x scripts/setup_tb_suite_enhanced.sh
./scripts/setup_tb_suite_enhanced.sh

# Start TB Suite with AI capabilities
./start_tb_suite.sh

# Access the AI-enhanced platform
# URL: http://localhost:8001
# Username: Administrator  
# Password: [Your admin password]
```

For detailed setup instructions, see [Quick Setup Guide](docs/QUICK_SETUP_GUIDE.md)

## 📚 Documentation

- [TB Suite Overview](docs/README.md) - Complete platform overview
- [Quick Setup Guide](docs/QUICK_SETUP_GUIDE.md) - Get started in 5 minutes
- [Technical Architecture](docs/ARCHITECTURE.md) - AI-enhanced system design
- [MCP Integration Guide](docs/MCP_INTEGRATION.md) - AI server integration
- [Modules Overview](docs/MODULES_OVERVIEW.md) - Detailed module descriptions
- [Onboarding Wizard](docs/ONBOARDING_WIZARD.md) - Step-by-step setup
- [Development Guidelines](docs/DEV_GUIDE.md) - Coding standards and best practices
- [API Reference](docs/API_REFERENCE.md) - Complete API documentation
- [Deployment Guide](docs/DEPLOYMENT.md) - Production deployment
- [Development Roadmap](docs/ROADMAP.md) - Timeline and milestones

## 🏗️ Enhanced Project Structure

```
tb_suite/
├── apps/                    # TB Suite Applications
│   ├── frappe/             # Core framework
│   ├── erpnext/            # ERP system
│   ├── tb_hotel_core/      # Core hotel entities
│   ├── tb_hotel_management/# Multi-property management
│   ├── tb_hotel_pms/       # Property management system
│   ├── tb_restaurant_pos/  # Restaurant POS
│   ├── tb_restaurant_management/
│   ├── tb_travel_management/
│   ├── tb_gds_integrations/
│   ├── tb_channel_bridge/
│   ├── tb_billing/
│   ├── tb_crm_reservations/
│   └── tb_mcp_bridge/      # 🤖 AI/MCP integration layer
├── docs/                    # Comprehensive documentation
├── scripts/                 # Setup and utility scripts
├── mcp_config/             # 🤖 MCP server configurations
├── .env                    # Environment variables
└── start_tb_suite.sh       # 🚀 Enhanced startup script
```

## 🔧 Enhanced Technology Stack

- **Backend:** Python 3.10+, Frappe Framework v15
- **Package Manager:** UV (Astral-sh) - 10-100x faster than pip
- **Database:** MariaDB 10.6+
- **Cache:** Redis 7.x
- **Frontend:** Vue.js 3.x, TailwindCSS, Frappe-UI
- **AI Integration:** MCP Servers (Sequential Thinking, Travel Assistant)
- **Web Server:** Nginx
- **Containerization:** Docker & Kubernetes ready

## 📦 TB Suite Modules

| Module | Description | Repository | AI Features |
|--------|-------------|------------|-------------|
| `tb_hotel_core` | Core entities and shared modules | [GitHub](https://github.com/ekanshjain/tb_hotel_core) | 🤖 Smart configurations |
| `tb_hotel_management` | Multi-property management | [GitHub](https://github.com/ekanshjain/tb_hotel_management) | 📊 Predictive analytics |
| `tb_hotel_pms` | Property Management System | [GitHub](https://github.com/ekanshjain/tb_hotel_pms) | 🧠 Intelligent operations |
| `tb_restaurant_pos` | Restaurant Point of Sale | [GitHub](https://github.com/ekanshjain/tb_restaurant_pos) | 💡 Smart recommendations |
| `tb_restaurant_management` | F&B back-office management | [GitHub](https://github.com/ekanshjain/tb_restaurant_management) | 📈 Cost optimization |
| `tb_travel_management` | Travel booking engine | [GitHub](https://github.com/ekanshjain/tb_travel_management) | ✈️ AI trip planning |
| `tb_gds_integrations` | GDS connectivity (Amadeus) | [GitHub](https://github.com/ekanshjain/tb_gds_integrations) | 🌐 Smart booking |
| `tb_channel_bridge` | OTA/Channel manager | [GitHub](https://github.com/ekanshjain/tb_channel_bridge) | 🔗 Auto optimization |
| `tb_billing` | Advanced billing system | [GitHub](https://github.com/ekanshjain/tb_billing) | 💰 Smart invoicing |
| `tb_crm_reservations` | CRM & guest management | [GitHub](https://github.com/ekanshjain/tb_crm_reservations) | 👤 Personalization |
| `tb_mcp_bridge` | AI/MCP integration layer | [GitHub](https://github.com/ekanshjain/tb_mcp_bridge) | 🤖 AI orchestration |

## 🛠️ Development

### Prerequisites

- Python 3.10+
- Node.js 18+
- MariaDB 10.6+ or MySQL 8.0+
- Redis
- Git

### Setup Development Environment

```bash
# Clone all repositories
git clone https://github.com/ekanshjain/staybird-bench.git
cd staybird-bench

# Install Frappe bench
pip3 install frappe-bench

# Get apps
bench get-app https://github.com/ekanshjain/staybird-core.git
bench get-app https://github.com/ekanshjain/staybird-pms.git
# ... (other apps)

# Create site
bench new-site staybird.local
bench --site staybird.local install-app erpnext
bench --site staybird.local install-app staybird_core
# ... (install other apps)

# Start development server
bench start
```

### Running Tests

```bash
# Run all tests
bench --site staybird.local run-tests

# Run specific app tests
bench --site staybird.local run-tests --app staybird_pms

# Run with coverage
bench --site staybird.local run-tests --coverage
```

## 🤝 Contributing

We welcome contributions! Please see our [Development Guidelines](docs/DEVELOPMENT_GUIDELINES.md) for coding standards and submission process.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📄 License

This project is proprietary software. All rights reserved by StayBird Group.

## 📞 Support

- **GitHub Issues:** [Create an issue](https://github.com/ekanshjain/tb_suite/issues)
- **Email:** mailme@ekanshjain.com
- **Documentation:** [View Docs](https://github.com/ekanshjain/tb_suite/tree/main/docs)

## 👨‍💻 Author

**Ekansh Jain**
- GitHub: [@ekanshjain](https://github.com/ekanshjain)
- Email: mailme@ekanshjain.com

## 🙏 Acknowledgments

- Frappe Framework team for the excellent foundation
- ERPNext community for the robust ERP system
- All contributors and testers

---

*Built with ❤️ for the hospitality industry*
