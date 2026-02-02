# StayBird Platform - API Documentation

## 🌐 API Overview

The StayBird Platform provides a comprehensive REST API for all hospitality operations. All endpoints follow RESTful conventions and return JSON responses.

**Base URL:** `https://api.staybird.com/v1` (Production)  
**Development:** `http://localhost:8001/api/v1`  
**Authentication:** Bearer Token / API Key  
**Rate Limiting:** 100 requests per minute  

---

## 🔐 Authentication

### Obtaining Access Token

```http
POST /auth/login
Content-Type: application/json

{
  "username": "ekanshjain",
  "password": "your_password"
}
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expires_in": 28800,
  "user": {
    "username": "ekanshjain",
    "email": "mailme@ekanshjain.com",
    "full_name": "Ekansh Jain",
    "roles": ["Property Manager", "System Admin"]
  }
}
```

### Using the Token

Include the token in the Authorization header:
```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### API Key Authentication (for integrations)

```http
X-API-Key: your_api_key_here
```

---

## 📋 Reservation Management

### List Reservations

```http
GET /reservations?property=PROP001&status=confirmed&page=1&page_size=20
```

**Query Parameters:**
| Parameter | Type | Description | Required |
|-----------|------|-------------|----------|
| property | string | Property ID | No |
| status | string | confirmed, in_house, checked_out, cancelled | No |
| check_in_date | date | Filter by check-in date (YYYY-MM-DD) | No |
| check_out_date | date | Filter by check-out date | No |
| guest_name | string | Search by guest name | No |
| page | integer | Page number (default: 1) | No |
| page_size | integer | Items per page (default: 20, max: 100) | No |

**Response:**
```json
{
  "data": [
    {
      "reservation_id": "RES-2025-00001",
      "guest": {
        "name": "John Doe",
        "email": "john@example.com",
        "phone": "+1234567890"
      },
      "room": {
        "number": "101",
        "type": "Deluxe",
        "floor": 1
      },
      "check_in_date": "2025-09-10",
      "check_out_date": "2025-09-15",
      "nights": 5,
      "adults": 2,
      "children": 1,
      "status": "confirmed",
      "total_amount": 5000.00,
      "paid_amount": 2000.00,
      "balance": 3000.00,
      "created_at": "2025-09-01T10:30:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "page_size": 20,
    "total_count": 150,
    "total_pages": 8
  }
}
```

### Get Single Reservation

```http
GET /reservations/RES-2025-00001
```

**Response:**
```json
{
  "reservation_id": "RES-2025-00001",
  "property": "PROP001",
  "guest": {
    "guest_id": "GUEST-001",
    "name": "John Doe",
    "email": "john@example.com",
    "phone": "+1234567890",
    "address": "123 Main St, City, Country",
    "id_type": "Passport",
    "id_number": "A1234567"
  },
  "room": {
    "room_id": "ROOM-101",
    "number": "101",
    "type": "Deluxe",
    "floor": 1,
    "view": "Sea View",
    "amenities": ["WiFi", "AC", "Mini Bar", "Safe"]
  },
  "dates": {
    "check_in": "2025-09-10T14:00:00Z",
    "check_out": "2025-09-15T12:00:00Z",
    "nights": 5,
    "booking_date": "2025-09-01T10:30:00Z"
  },
  "occupancy": {
    "adults": 2,
    "children": 1,
    "infants": 0
  },
  "rates": {
    "room_rate": 900.00,
    "extra_bed": 100.00,
    "subtotal": 5000.00,
    "taxes": 600.00,
    "total": 5600.00
  },
  "payments": [
    {
      "payment_id": "PAY-001",
      "date": "2025-09-01T10:35:00Z",
      "amount": 2000.00,
      "method": "Credit Card",
      "reference": "TXN123456"
    }
  ],
  "special_requests": "Late check-out requested",
  "source": "Direct",
  "status": "confirmed"
}
```

### Create Reservation

```http
POST /reservations
Content-Type: application/json

{
  "property": "PROP001",
  "guest": {
    "guest_id": "GUEST-001",  // Or create new guest
    "name": "John Doe",
    "email": "john@example.com",
    "phone": "+1234567890"
  },
  "room_type": "Deluxe",
  "check_in_date": "2025-09-10",
  "check_out_date": "2025-09-15",
  "adults": 2,
  "children": 1,
  "rate_plan": "BAR",
  "special_requests": "Late check-out",
  "source": "Direct"
}
```

**Response:**
```json
{
  "status": "success",
  "message": "Reservation created successfully",
  "data": {
    "reservation_id": "RES-2025-00002",
    "confirmation_number": "CNF789012",
    "status": "confirmed"
  }
}
```

### Update Reservation

```http
PUT /reservations/RES-2025-00001
Content-Type: application/json

{
  "check_out_date": "2025-09-16",
  "special_requests": "Airport pickup required"
}
```

### Cancel Reservation

```http
POST /reservations/RES-2025-00001/cancel
Content-Type: application/json

{
  "reason": "Guest request",
  "cancellation_charges": 500.00,
  "refund_amount": 1500.00
}
```

---

## 🏨 Room Management

### Check Room Availability

```http
GET /rooms/availability?property=PROP001&check_in=2025-09-10&check_out=2025-09-15&room_type=Deluxe
```

**Response:**
```json
{
  "available_rooms": [
    {
      "room_id": "ROOM-101",
      "number": "101",
      "type": "Deluxe",
      "floor": 1,
      "view": "Sea View",
      "max_occupancy": 3,
      "rate": 900.00,
      "amenities": ["WiFi", "AC", "Mini Bar"]
    },
    {
      "room_id": "ROOM-102",
      "number": "102",
      "type": "Deluxe",
      "floor": 1,
      "view": "Garden View",
      "max_occupancy": 3,
      "rate": 850.00,
      "amenities": ["WiFi", "AC", "Mini Bar"]
    }
  ],
  "summary": {
    "total_available": 5,
    "room_types": {
      "Deluxe": 5,
      "Suite": 0,
      "Standard": 0
    }
  }
}
```

### Update Room Status

```http
PATCH /rooms/ROOM-101/status
Content-Type: application/json

{
  "status": "Under Maintenance",
  "reason": "AC repair",
  "expected_available": "2025-09-08"
}
```

---

## 👤 Guest Management

### Search Guests

```http
GET /guests?search=john&email=john@example.com
```

### Create Guest Profile

```http
POST /guests
Content-Type: application/json

{
  "first_name": "John",
  "last_name": "Doe",
  "email": "john@example.com",
  "phone": "+1234567890",
  "date_of_birth": "1990-01-15",
  "nationality": "USA",
  "id_type": "Passport",
  "id_number": "A1234567",
  "address": {
    "street": "123 Main St",
    "city": "New York",
    "state": "NY",
    "country": "USA",
    "postal_code": "10001"
  },
  "preferences": {
    "room_type": "Deluxe",
    "floor": "High",
    "smoking": false,
    "newspaper": true
  },
  "vip": false,
  "blacklisted": false
}
```

### Get Guest History

```http
GET /guests/GUEST-001/history
```

**Response:**
```json
{
  "guest_id": "GUEST-001",
  "total_stays": 5,
  "total_nights": 23,
  "total_revenue": 25000.00,
  "average_stay_length": 4.6,
  "last_stay": "2025-08-15",
  "loyalty_points": 2500,
  "tier": "Gold",
  "reservations": [
    {
      "reservation_id": "RES-2025-00001",
      "check_in": "2025-08-10",
      "check_out": "2025-08-15",
      "room": "101",
      "total": 4500.00,
      "status": "checked_out"
    }
  ]
}
```

---

## 💳 Payment Processing

### Process Payment

```http
POST /payments
Content-Type: application/json

{
  "reservation_id": "RES-2025-00001",
  "amount": 2000.00,
  "currency": "USD",
  "method": "credit_card",
  "card_details": {
    "number": "4111111111111111",
    "exp_month": "12",
    "exp_year": "2025",
    "cvv": "123",
    "name": "John Doe"
  }
}
```

**Response:**
```json
{
  "status": "success",
  "payment_id": "PAY-002",
  "transaction_id": "TXN789012",
  "amount": 2000.00,
  "currency": "USD",
  "timestamp": "2025-09-07T14:30:00Z",
  "receipt_url": "https://api.staybird.com/receipts/PAY-002"
}
```

### Refund Payment

```http
POST /payments/PAY-001/refund
Content-Type: application/json

{
  "amount": 500.00,
  "reason": "Partial refund for early checkout"
}
```

---

## 🍽️ POS Integration

### Create POS Order

```http
POST /pos/orders
Content-Type: application/json

{
  "outlet": "Restaurant",
  "table": "T-05",
  "items": [
    {
      "item_code": "ITEM-001",
      "name": "Caesar Salad",
      "quantity": 2,
      "rate": 15.00
    },
    {
      "item_code": "ITEM-002",
      "name": "Grilled Salmon",
      "quantity": 1,
      "rate": 35.00
    }
  ],
  "charge_to_room": true,
  "room_number": "101",
  "server": "John Smith"
}
```

### Post Charge to Room

```http
POST /folios/charges
Content-Type: application/json

{
  "reservation_id": "RES-2025-00001",
  "charge_type": "Restaurant",
  "description": "Dinner at Main Restaurant",
  "amount": 65.00,
  "reference": "POS-ORDER-123"
}
```

---

## 🏢 Channel Manager Integration

### Sync Availability

```http
POST /channels/sync/availability
Content-Type: application/json

{
  "property": "PROP001",
  "channel": "STAAH",
  "date_from": "2025-09-01",
  "date_to": "2025-09-30",
  "room_types": ["Deluxe", "Suite"]
}
```

### Fetch Channel Bookings

```http
GET /channels/bookings?channel=STAAH&from_date=2025-09-01&to_date=2025-09-30
```

### Update Channel Rates

```http
POST /channels/rates
Content-Type: application/json

{
  "property": "PROP001",
  "channel": "Booking.com",
  "room_type": "Deluxe",
  "rates": [
    {
      "date": "2025-09-10",
      "rate": 120.00,
      "availability": 5
    },
    {
      "date": "2025-09-11",
      "rate": 125.00,
      "availability": 3
    }
  ]
}
```

---

## 📊 Reports

### Occupancy Report

```http
GET /reports/occupancy?property=PROP001&month=9&year=2025
```

**Response:**
```json
{
  "property": "PROP001",
  "period": "2025-09",
  "occupancy_rate": 78.5,
  "total_rooms": 50,
  "room_nights_available": 1500,
  "room_nights_sold": 1177,
  "adr": 95.50,
  "revpar": 75.00,
  "daily_occupancy": [
    {
      "date": "2025-09-01",
      "occupied": 40,
      "available": 50,
      "occupancy_rate": 80.0
    }
  ]
}
```

### Revenue Report

```http
GET /reports/revenue?property=PROP001&from=2025-09-01&to=2025-09-30
```

---

## 🔔 Webhooks

### Available Events

- `reservation.created`
- `reservation.confirmed`
- `reservation.cancelled`
- `guest.checked_in`
- `guest.checked_out`
- `payment.received`
- `payment.refunded`

### Webhook Payload Example

```json
{
  "event": "reservation.created",
  "timestamp": "2025-09-07T14:30:00Z",
  "data": {
    "reservation_id": "RES-2025-00001",
    "property": "PROP001",
    "guest_name": "John Doe",
    "check_in_date": "2025-09-10",
    "check_out_date": "2025-09-15",
    "total_amount": 5000.00
  }
}
```

### Register Webhook

```http
POST /webhooks
Content-Type: application/json

{
  "url": "https://your-app.com/webhook",
  "events": ["reservation.created", "guest.checked_in"],
  "secret": "your_webhook_secret"
}
```

---

## 🔍 Error Handling

### Error Response Format

```json
{
  "status": "error",
  "error_code": "VALIDATION_ERROR",
  "message": "Check-out date must be after check-in date",
  "details": {
    "field": "check_out_date",
    "value": "2025-09-09",
    "constraint": "must be after 2025-09-10"
  },
  "timestamp": "2025-09-07T14:30:00Z",
  "request_id": "req_abc123"
}
```

### Common Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `UNAUTHORIZED` | 401 | Invalid or missing authentication |
| `FORBIDDEN` | 403 | Insufficient permissions |
| `NOT_FOUND` | 404 | Resource not found |
| `VALIDATION_ERROR` | 400 | Invalid input data |
| `DUPLICATE_ENTRY` | 409 | Resource already exists |
| `RATE_LIMIT_EXCEEDED` | 429 | Too many requests |
| `INTERNAL_ERROR` | 500 | Server error |

---

## 🧪 Testing

### Test Environment

**Base URL:** `https://sandbox.staybird.com/api/v1`  
**Test Credentials:**
- Username: `test_ekanshjain`
- Password: `test_password`
- API Key: `test_api_key_123`

### Sample cURL Commands

```bash
# Login
curl -X POST https://sandbox.staybird.com/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"test_ekanshjain","password":"test_password"}'

# Get Reservations
curl -X GET https://sandbox.staybird.com/api/v1/reservations \
  -H "Authorization: Bearer YOUR_TOKEN"

# Create Reservation
curl -X POST https://sandbox.staybird.com/api/v1/reservations \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "property": "PROP001",
    "guest": {"name": "Test Guest", "email": "test@example.com"},
    "room_type": "Deluxe",
    "check_in_date": "2025-09-10",
    "check_out_date": "2025-09-15"
  }'
```

---

## 📦 SDKs & Libraries

### Python SDK

```python
from staybird import StayBirdClient

client = StayBirdClient(
    api_key="your_api_key",
    environment="production"  # or "sandbox"
)

# Get reservations
reservations = client.reservations.list(
    property="PROP001",
    status="confirmed"
)

# Create reservation
reservation = client.reservations.create(
    property="PROP001",
    guest_name="John Doe",
    check_in="2025-09-10",
    check_out="2025-09-15"
)
```

### JavaScript/Node.js SDK

```javascript
const StayBird = require('@ekanshjain/staybird-sdk');

const client = new StayBird({
  apiKey: 'your_api_key',
  environment: 'production'
});

// Get reservations
const reservations = await client.reservations.list({
  property: 'PROP001',
  status: 'confirmed'
});

// Create reservation
const reservation = await client.reservations.create({
  property: 'PROP001',
  guest: { name: 'John Doe', email: 'john@example.com' },
  checkIn: '2025-09-10',
  checkOut: '2025-09-15'
});
```

---

## 📞 Support

**Developer Support:** mailme@ekanshjain.com  
**GitHub Issues:** https://github.com/ekanshjain/staybird-bench/issues  
**API Status:** https://status.staybird.com  

---

*API Version: 1.0.0*  
*Last Updated: September 2025*  
*Maintained by: Ekansh Jain ([@ekanshjain](https://github.com/ekanshjain))*
