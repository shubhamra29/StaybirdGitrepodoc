# TB Suite - Technical Architecture

## 🏗️ System Architecture Overview

The TB Suite follows a modular, microservices-inspired architecture while leveraging the monolithic benefits of Frappe Framework.

```mermaid
graph TB
    subgraph "Client Layer"
        WEB[Web Browser]
        MOBILE[Mobile Apps]
        KIOSK[Kiosk/Tablets]
        API[API Clients]
    end
    
    subgraph "TB Suite Core"
        CORE[tb_hotel_core]
        MGMT[tb_hotel_management]
        PMS[tb_hotel_pms]
    end
    
    subgraph "TB Restaurant"
        POS[tb_restaurant_pos]
        REST_MGMT[tb_restaurant_management]
    end
    
    subgraph "TB Travel"
        TRAVEL[tb_travel_management]
        GDS[tb_gds_integrations]
    end
    
    subgraph "TB Integrations"
        CHANNEL[tb_channel_bridge]
        BILLING[tb_billing]
        CRM[tb_crm_reservations]
    end
    
    subgraph "External Systems"
        STAAH[STAAH/OTAs]
        AMADEUS[Amadeus GDS]
        PAYMENT[Payment Gateways]
        COMM[SMS/WhatsApp]
    end
    
    subgraph "Data Layer"
        DB[(MariaDB)]
        REDIS[(Redis)]
        FILES[File Storage]
    end
    
    WEB --> CORE
    MOBILE --> POS
    KIOSK --> PMS
    API --> TRAVEL
    
    PMS --> CORE
    POS --> CORE
    TRAVEL --> GDS
    
    CHANNEL --> STAAH
    GDS --> AMADEUS
    BILLING --> PAYMENT
    CRM --> COMM
    
    CORE --> DB
    CORE --> REDIS
    CORE --> FILES
```

## 📦 Module Architecture

### Core Modules Hierarchy

```mermaid
graph LR
    subgraph "Foundation"
        FRAPPE[Frappe Framework]
        ERPNEXT[ERPNext]
    end
    
    subgraph "TB Core"
        CORE[tb_hotel_core]
        MGMT[tb_hotel_management]
    end
    
    subgraph "Operations"
        PMS[tb_hotel_pms]
        POS[tb_restaurant_pos]
        REST[tb_restaurant_management]
    end
    
    subgraph "Extensions"
        TRAVEL[tb_travel_management]
        GDS[tb_gds_integrations]
        CHANNEL[tb_channel_bridge]
    end
    
    FRAPPE --> CORE
    ERPNEXT --> CORE
    CORE --> MGMT
    CORE --> PMS
    CORE --> POS
    POS --> REST
    TRAVEL --> GDS
    PMS --> CHANNEL
```

### Module Dependencies

| Module | Depends On | Required For |
|--------|------------|--------------|
| **tb_hotel_core** | frappe, erpnext | All TB modules |
| **tb_hotel_management** | tb_hotel_core | Multi-property operations |
| **tb_hotel_pms** | tb_hotel_core, tb_hotel_management | Front desk operations |
| **tb_restaurant_pos** | tb_hotel_core | Restaurant operations |
| **tb_restaurant_management** | tb_restaurant_pos | F&B management |
| **tb_travel_management** | tb_hotel_core | Travel bookings |
| **tb_gds_integrations** | tb_travel_management | GDS connectivity |
| **tb_channel_bridge** | tb_hotel_pms | OTA integration |
| **tb_billing** | tb_hotel_core | Invoicing |
| **tb_crm_reservations** | tb_hotel_core | CRM features |

## 🔄 Data Flow Architecture

### PMS Operational Flow

```mermaid
sequenceDiagram
    participant Guest
    participant FrontDesk
    participant PMS
    participant Core
    participant Channel
    participant Payment
    
    Guest->>FrontDesk: Walk-in/Online Booking
    FrontDesk->>PMS: Create Reservation
    PMS->>Core: Check Availability
    Core-->>PMS: Room Available
    PMS->>Channel: Update OTA Inventory
    PMS->>Payment: Process Advance
    Payment-->>PMS: Payment Confirmed
    PMS-->>FrontDesk: Reservation Confirmed
    FrontDesk-->>Guest: Confirmation
    
    Note over Guest,Payment: Check-in Process
    Guest->>FrontDesk: Arrive for Check-in
    FrontDesk->>PMS: Check-in Guest
    PMS->>Core: Update Room Status
    PMS-->>FrontDesk: Room Key Generated
    FrontDesk-->>Guest: Hand Over Key
```

### Restaurant POS Flow

```mermaid
sequenceDiagram
    participant Customer
    participant Waiter
    participant POS
    participant Kitchen
    participant Billing
    participant PMS
    
    Customer->>Waiter: Place Order
    Waiter->>POS: Enter Order
    POS->>Kitchen: Send KOT
    Kitchen-->>POS: Confirm KOT
    POS-->>Waiter: Order Confirmed
    
    Note over Kitchen: Prepare Food
    Kitchen->>POS: Mark Ready
    POS->>Waiter: Notify Ready
    Waiter->>Customer: Serve Food
    
    Customer->>Waiter: Request Bill
    Waiter->>POS: Generate Bill
    POS->>Billing: Calculate Total
    
    alt Room Guest
        POS->>PMS: Post to Room
        PMS-->>POS: Charge Posted
    else Direct Payment
        POS->>Billing: Process Payment
        Billing-->>POS: Payment Complete
    end
    
    POS-->>Waiter: Bill Settled
    Waiter-->>Customer: Thank You
```

### Travel Booking with GDS

```mermaid
sequenceDiagram
    participant User
    participant Travel
    participant GDS
    participant Amadeus
    participant Payment
    participant Supplier
    
    User->>Travel: Search Flights
    Travel->>GDS: Query Availability
    GDS->>Amadeus: Search Request
    Amadeus-->>GDS: Flight Options
    GDS-->>Travel: Formatted Results
    Travel-->>User: Display Options
    
    User->>Travel: Select Flight
    Travel->>GDS: Create PNR
    GDS->>Amadeus: Book Segments
    Amadeus-->>GDS: PNR Created
    
    Travel->>Payment: Process Payment
    Payment-->>Travel: Payment Success
    
    Travel->>GDS: Confirm Booking
    GDS->>Amadeus: Issue Ticket
    Amadeus-->>GDS: E-Ticket
    GDS-->>Travel: Ticket Details
    Travel-->>User: Booking Confirmed
    
    Travel->>Supplier: Notify Booking
```

## 💾 Data Architecture

### Core Entity Model

```mermaid
erDiagram
    Company ||--o{ Property : owns
    Property ||--o{ Building : contains
    Building ||--o{ Floor : has
    Floor ||--o{ Room : contains
    
    Property ||--o{ RoomType : defines
    RoomType ||--o{ Room : categorizes
    RoomType ||--o{ RatePlan : has
    RatePlan ||--o{ SeasonalRate : contains
    
    Guest ||--o{ Reservation : makes
    Reservation ||--o{ Stay : generates
    Stay ||--o{ Room : occupies
    Stay ||--o{ Folio : has
    Folio ||--o{ FolioCharge : contains
    Folio ||--o{ Payment : receives
    
    Property ||--o{ Outlet : has
    Outlet ||--o{ POSProfile : uses
    POSProfile ||--o{ MenuItem : offers
    MenuItem ||--o{ Recipe : follows
    Recipe ||--o{ Ingredient : requires
    
    Company ||--o{ CorporateAccount : manages
    CorporateAccount ||--o{ CreditProfile : has
    CreditProfile ||--o{ Reservation : books
```

### Multi-Company Architecture

```mermaid
graph TB
    subgraph "Holding Company"
        HC[TechBird Hospitality Group]
    end
    
    subgraph "Operating Companies"
        C1[StayBird Hotels Pvt Ltd]
        C2[TranqVillas Resorts Ltd]
        C3[StarBird Travel Services]
    end
    
    subgraph "Properties"
        P1[StayBird Mumbai]
        P2[StayBird Delhi]
        P3[TranqVillas Goa]
        P4[TranqVillas Kerala]
    end
    
    subgraph "Shared Services"
        SS1[Central Reservations]
        SS2[Revenue Management]
        SS3[Procurement]
        SS4[Finance & Accounting]
    end
    
    HC --> C1
    HC --> C2
    HC --> C3
    
    C1 --> P1
    C1 --> P2
    C2 --> P3
    C2 --> P4
    
    C1 --> SS1
    C2 --> SS1
    C3 --> SS1
    
    SS1 --> SS2
    SS1 --> SS3
    SS1 --> SS4
```

## 🔌 Integration Architecture

### Provider Interface Pattern

All external integrations follow a standardized provider interface pattern:

```python
# Base Provider Interface
class BaseProvider:
    def authenticate(self) -> bool:
        """Authenticate with provider"""
        pass
    
    def validate_config(self) -> bool:
        """Validate configuration"""
        pass
    
    def handle_webhook(self, payload: dict) -> dict:
        """Process webhook from provider"""
        pass
    
    def get_status(self) -> dict:
        """Get provider connection status"""
        pass

# Channel Manager Provider
class ChannelProvider(BaseProvider):
    def sync_inventory(self, property_id: str, date_range: tuple) -> bool:
        """Push inventory to channels"""
        pass
    
    def fetch_bookings(self, last_sync: datetime) -> list:
        """Pull new bookings"""
        pass
    
    def update_rates(self, rate_updates: list) -> bool:
        """Update rate plans"""
        pass

# GDS Provider
class GDSProvider(BaseProvider):
    def search_availability(self, criteria: dict) -> list:
        """Search for availability"""
        pass
    
    def create_booking(self, booking_data: dict) -> str:
        """Create PNR/booking"""
        pass
    
    def issue_ticket(self, pnr: str) -> dict:
        """Issue ticket/voucher"""
        pass
```

### Integration Points

```mermaid
graph LR
    subgraph "TB Suite"
        CORE[TB Core]
        INT[Integration Layer]
    end
    
    subgraph "Channel Partners"
        STAAH[STAAH]
        BOOKING[Booking.com]
        EXPEDIA[Expedia]
        MMT[MakeMyTrip]
    end
    
    subgraph "GDS Systems"
        AMADEUS[Amadeus]
        SABRE[Sabre]
        GALILEO[Galileo]
    end
    
    subgraph "Payment"
        RAZORPAY[Razorpay]
        STRIPE[Stripe]
        PAYU[PayU]
    end
    
    subgraph "Communication"
        TWILIO[Twilio]
        EXOTEL[Exotel]
        WHATSAPP[WhatsApp Business]
    end
    
    INT --> STAAH
    INT --> AMADEUS
    INT --> RAZORPAY
    INT --> EXOTEL
    
    STAAH --> BOOKING
    STAAH --> EXPEDIA
    STAAH --> MMT
```

## 🔐 Security Architecture

### Authentication & Authorization

```mermaid
graph TB
    subgraph "Authentication"
        LOGIN[Login]
        MFA[2FA/MFA]
        SSO[Single Sign-On]
        API_KEY[API Keys]
    end
    
    subgraph "Authorization"
        RBAC[Role-Based Access]
        PROPERTY[Property-Level]
        COMPANY[Company-Level]
        MODULE[Module-Level]
    end
    
    subgraph "Roles"
        ADMIN[System Admin]
        MANAGER[Property Manager]
        FRONTDESK[Front Desk]
        HOUSEKEEPING[Housekeeping]
        FNB[F&B Staff]
        ACCOUNTS[Accounts]
    end
    
    LOGIN --> MFA
    MFA --> RBAC
    SSO --> RBAC
    API_KEY --> RBAC
    
    RBAC --> PROPERTY
    RBAC --> COMPANY
    RBAC --> MODULE
    
    PROPERTY --> MANAGER
    PROPERTY --> FRONTDESK
    MODULE --> FNB
    COMPANY --> ADMIN
```

### Data Security Layers

1. **Application Level**
   - Input validation
   - SQL injection prevention
   - XSS protection
   - CSRF tokens

2. **Transport Level**
   - TLS 1.3 encryption
   - Certificate pinning
   - API rate limiting
   - DDoS protection

3. **Data Level**
   - Encryption at rest
   - Field-level encryption for PII
   - Audit logging
   - Data masking

4. **Compliance**
   - GDPR compliance
   - PCI DSS for payments
   - Data retention policies
   - Right to forget implementation

## 🚀 Deployment Architecture

### Production Deployment

```mermaid
graph TB
    subgraph "Load Balancer"
        LB[Nginx/HAProxy]
    end
    
    subgraph "Application Tier"
        APP1[App Server 1]
        APP2[App Server 2]
        APP3[App Server 3]
    end
    
    subgraph "Worker Tier"
        W1[Worker 1]
        W2[Worker 2]
        W3[Worker 3]
    end
    
    subgraph "Data Tier"
        DB_MASTER[(DB Master)]
        DB_SLAVE1[(DB Slave 1)]
        DB_SLAVE2[(DB Slave 2)]
        REDIS_MASTER[(Redis Master)]
        REDIS_SLAVE[(Redis Slave)]
    end
    
    subgraph "Storage"
        S3[Object Storage]
        CDN[CDN]
    end
    
    LB --> APP1
    LB --> APP2
    LB --> APP3
    
    APP1 --> DB_MASTER
    APP2 --> DB_SLAVE1
    APP3 --> DB_SLAVE2
    
    APP1 --> REDIS_MASTER
    APP2 --> REDIS_MASTER
    APP3 --> REDIS_MASTER
    
    W1 --> DB_MASTER
    W2 --> REDIS_MASTER
    W3 --> S3
    
    S3 --> CDN
```

### Container Architecture

```yaml
# docker-compose.yml
version: '3.8'

services:
  tb-app:
    image: techbirdit/tb-suite:latest
    environment:
      - SITE_NAME=hotel.local
      - DB_HOST=mariadb
      - REDIS_CACHE=redis://redis:6379/0
    volumes:
      - sites:/home/frappe/frappe-bench/sites
      - apps:/home/frappe/frappe-bench/apps
    
  tb-worker:
    image: techbirdit/tb-suite:latest
    command: bench worker
    environment:
      - SITE_NAME=hotel.local
    
  tb-scheduler:
    image: techbirdit/tb-suite:latest
    command: bench schedule
    
  mariadb:
    image: mariadb:10.6
    environment:
      - MYSQL_ROOT_PASSWORD=admin
      - MYSQL_DATABASE=tb_suite
    
  redis:
    image: redis:7-alpine
    
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
```

## 📊 Performance Architecture

### Caching Strategy

```mermaid
graph LR
    subgraph "Cache Layers"
        L1[Browser Cache]
        L2[CDN Cache]
        L3[Nginx Cache]
        L4[Redis Cache]
        L5[Database Cache]
    end
    
    subgraph "Cache Types"
        STATIC[Static Assets]
        API[API Responses]
        SESSION[User Sessions]
        QUERY[Query Results]
        COMPUTED[Computed Values]
    end
    
    L1 --> STATIC
    L2 --> STATIC
    L3 --> API
    L4 --> SESSION
    L4 --> QUERY
    L4 --> COMPUTED
    L5 --> QUERY
```

### Optimization Techniques

1. **Database Optimization**
   - Proper indexing
   - Query optimization
   - Connection pooling
   - Read replicas

2. **Application Optimization**
   - Lazy loading
   - Pagination
   - Batch processing
   - Async operations

3. **Frontend Optimization**
   - Code splitting
   - Tree shaking
   - Image optimization
   - Progressive web app

4. **Infrastructure Optimization**
   - Auto-scaling
   - Load balancing
   - CDN distribution
   - Edge computing

## 🔄 Scalability Patterns

### Horizontal Scaling

- Add more application servers
- Distribute load across servers
- Scale workers independently
- Use read replicas for reports

### Vertical Scaling

- Increase server resources
- Optimize database performance
- Upgrade Redis capacity
- Enhanced caching

### Microservices Evolution

Future architecture can evolve to:
- Separate PMS service
- Independent POS service
- Standalone travel service
- Dedicated integration service

---

*TB Suite Architecture - Built for scale, designed for flexibility*
