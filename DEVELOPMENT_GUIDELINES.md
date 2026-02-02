# StayBird Platform - Development Guidelines

## 📝 Code Style Guide

### Python Code Standards

#### PEP 8 Compliance
```python
# ✅ Good
def calculate_room_rate(base_rate, season_multiplier, discount=0):
    """
    Calculate the final room rate after applying seasonal rates and discounts.
    
    Args:
        base_rate (float): Base room rate
        season_multiplier (float): Seasonal adjustment factor
        discount (float): Discount percentage (0-100)
    
    Returns:
        float: Final calculated rate
    """
    if not 0 <= discount <= 100:
        raise ValueError("Discount must be between 0 and 100")
    
    seasonal_rate = base_rate * season_multiplier
    final_rate = seasonal_rate * (1 - discount / 100)
    
    return round(final_rate, 2)


# ❌ Bad
def calc_rate(r,s,d=0):
    return r*s*(1-d/100)
```

#### Naming Conventions
```python
# Classes: PascalCase
class ReservationManager:
    pass

# Functions/Variables: snake_case
def get_available_rooms():
    room_count = 10
    return room_count

# Constants: UPPER_SNAKE_CASE
MAX_OCCUPANCY = 4
DEFAULT_CHECK_IN_TIME = "14:00"

# Private methods: Leading underscore
def _validate_input():
    pass
```

### JavaScript/Vue.js Standards

#### Vue 3 Composition API
```javascript
// ✅ Good - Composition API with TypeScript
<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import type { Reservation } from '@/types'

// Props definition
interface Props {
  propertyId: string
  dateRange?: DateRange
}

const props = defineProps<Props>()

// Reactive state
const reservations = ref<Reservation[]>([])
const loading = ref(false)
const error = ref<string | null>(null)

// Computed properties
const occupancyRate = computed(() => {
  if (!reservations.value.length) return 0
  return (reservations.value.length / totalRooms.value) * 100
})

// Methods
async function fetchReservations() {
  loading.value = true
  try {
    const response = await api.get(`/reservations/${props.propertyId}`)
    reservations.value = response.data
  } catch (err) {
    error.value = err.message
  } finally {
    loading.value = false
  }
}

// Lifecycle
onMounted(() => {
  fetchReservations()
})
</script>
```

### SQL Standards

```sql
-- ✅ Good
SELECT 
    r.reservation_id,
    r.check_in_date,
    r.check_out_date,
    g.first_name,
    g.last_name,
    rm.room_number,
    rt.type_name
FROM 
    tabReservation r
    INNER JOIN tabGuest g ON r.guest_id = g.name
    INNER JOIN tabRoom rm ON r.room_id = rm.name
    INNER JOIN tabRoomType rt ON rm.room_type = rt.name
WHERE 
    r.status = 'Confirmed'
    AND r.check_in_date >= CURDATE()
ORDER BY 
    r.check_in_date ASC;

-- ❌ Bad
select * from tabReservation where status='Confirmed'
```

---

## 🏗️ Project Structure

### Frappe App Structure
```
staybird_pms/
├── staybird_pms/
│   ├── __init__.py
│   ├── hooks.py                 # App configuration
│   ├── modules.txt              # Module listing
│   ├── patches.txt              # Patch listing
│   ├── api/                     # REST API endpoints
│   │   ├── __init__.py
│   │   ├── v1/
│   │   │   ├── reservation.py
│   │   │   └── availability.py
│   │   └── v2/
│   ├── config/                  # App configuration
│   │   └── desktop.py           # Desktop icons
│   ├── doctype/                 # Business entities
│   │   ├── reservation/
│   │   │   ├── reservation.json # DocType definition
│   │   │   ├── reservation.py   # Business logic
│   │   │   ├── reservation.js   # Client-side logic
│   │   │   ├── reservation_list.js
│   │   │   └── test_reservation.py
│   │   └── room/
│   ├── page/                    # Single page apps
│   │   └── room_chart/
│   │       ├── room_chart.json
│   │       ├── room_chart.py
│   │       ├── room_chart.js
│   │       └── room_chart.html
│   ├── report/                  # Reports
│   │   └── occupancy_report/
│   │       ├── occupancy_report.json
│   │       ├── occupancy_report.py
│   │       └── occupancy_report.js
│   ├── print_format/            # Print templates
│   ├── workflow/                # Workflows
│   ├── public/                  # Static assets
│   │   ├── js/
│   │   ├── css/
│   │   └── images/
│   └── templates/               # Jinja2 templates
│       ├── includes/
│       └── pages/
├── requirements.txt             # Python dependencies
├── setup.py                     # Package setup
├── LICENSE
└── README.md
```

---

## 🔄 Git Workflow

### Branch Strategy
```bash
main                 # Production-ready code
├── develop          # Integration branch
│   ├── feature/*    # New features
│   ├── bugfix/*     # Bug fixes
│   ├── hotfix/*     # Emergency fixes
│   └── release/*    # Release preparation

# Set up Git with your credentials
git config --global user.name "ekanshjain"
git config --global user.email "mailme@ekanshjain.com"
```

### Commit Message Convention
```bash
# Format: <type>(<scope>): <subject>

# Types:
feat: New feature
fix: Bug fix
docs: Documentation
style: Code style (formatting, missing semicolons, etc)
refactor: Code refactoring
perf: Performance improvements
test: Testing
chore: Maintenance tasks
build: Build system changes
ci: CI/CD changes

# Examples:
git commit -m "feat(reservation): add bulk check-in functionality"
git commit -m "fix(payment): resolve decimal rounding issue"
git commit -m "docs(api): update reservation endpoint documentation"
git commit -m "refactor(room): optimize availability calculation"
git commit -m "test(folio): add unit tests for charge calculations"
```

### Pull Request Template
```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manual testing completed

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Comments added for complex logic
- [ ] Documentation updated
- [ ] No console.log or print statements
- [ ] No hardcoded values

## Author
- **GitHub:** @ekanshjain
- **Email:** mailme@ekanshjain.com
```

---

## 🧪 Testing Standards

### Unit Testing
```python
# test_reservation.py
import unittest
from unittest.mock import Mock, patch
import frappe
from frappe.tests.utils import FrappeTestCase

class TestReservation(FrappeTestCase):
    def setUp(self):
        """Set up test fixtures"""
        self.guest = create_test_guest()
        self.room = create_test_room()
        
    def tearDown(self):
        """Clean up after tests"""
        frappe.db.rollback()
        
    def test_reservation_creation(self):
        """Test successful reservation creation"""
        reservation = frappe.get_doc({
            "doctype": "Reservation",
            "guest": self.guest.name,
            "room": self.room.name,
            "check_in_date": "2025-09-10",
            "check_out_date": "2025-09-15",
            "status": "Confirmed"
        })
        reservation.insert()
        
        self.assertEqual(reservation.status, "Confirmed")
        self.assertEqual(reservation.nights, 5)
        
    def test_double_booking_prevention(self):
        """Test that double booking is prevented"""
        # Create first reservation
        create_reservation(self.room, "2025-09-10", "2025-09-15")
        
        # Attempt overlapping reservation
        with self.assertRaises(frappe.ValidationError):
            create_reservation(self.room, "2025-09-12", "2025-09-17")
    
    @patch('staybird_pms.api.send_confirmation_email')
    def test_confirmation_email(self, mock_email):
        """Test confirmation email is sent"""
        reservation = create_reservation(self.room, "2025-09-10", "2025-09-15")
        reservation.send_confirmation()
        
        mock_email.assert_called_once_with(
            reservation.guest_email,
            reservation.name
        )
```

### Integration Testing
```python
# test_integration_staah.py
class TestSTAAHIntegration(FrappeTestCase):
    @responses.activate
    def test_availability_sync(self):
        """Test availability sync with STAAH"""
        # Mock STAAH API response
        responses.add(
            responses.POST,
            'https://api.staah.com/v1/availability',
            json={'status': 'success'},
            status=200
        )
        
        # Trigger sync
        sync_availability('PROP001', '2025-09-01', '2025-09-30')
        
        # Verify API was called
        self.assertEqual(len(responses.calls), 1)
        self.assertIn('PROP001', responses.calls[0].request.body)
```

### E2E Testing (Cypress)
```javascript
// reservation.spec.js
describe('Reservation Flow', () => {
  beforeEach(() => {
    cy.login('front_desk@example.com', 'password')
    cy.visit('/app/reservation')
  })
  
  it('should create a new reservation', () => {
    cy.get('[data-cy=new-reservation]').click()
    
    // Fill guest details
    cy.get('[data-cy=guest-search]').type('John Doe')
    cy.get('[data-cy=guest-result]').first().click()
    
    // Select dates
    cy.get('[data-cy=check-in-date]').type('2025-09-10')
    cy.get('[data-cy=check-out-date]').type('2025-09-15')
    
    // Select room
    cy.get('[data-cy=room-select]').select('101')
    
    // Submit
    cy.get('[data-cy=submit-reservation]').click()
    
    // Verify success
    cy.get('[data-cy=success-message]')
      .should('be.visible')
      .and('contain', 'Reservation created successfully')
  })
})
```

---

## 📦 DocType Development

### DocType Best Practices
```python
# reservation.py
class Reservation(Document):
    def validate(self):
        """Validation before saving"""
        self.validate_dates()
        self.validate_room_availability()
        self.calculate_nights()
        self.calculate_total_amount()
    
    def before_insert(self):
        """Logic before first save"""
        self.set_reservation_number()
        self.set_default_status()
    
    def after_insert(self):
        """Logic after first save"""
        self.update_room_status()
        self.send_confirmation_email()
    
    def on_update(self):
        """Logic on every update"""
        self.sync_with_channel_manager()
        self.update_availability_calendar()
    
    def on_cancel(self):
        """Logic on cancellation"""
        self.release_room()
        self.process_refund()
        self.notify_housekeeping()
    
    def on_trash(self):
        """Prevent deletion of confirmed reservations"""
        if self.status == "Confirmed":
            frappe.throw(_("Cannot delete confirmed reservations"))
    
    @frappe.whitelist()
    def check_in(self):
        """Custom method for check-in process"""
        if self.status != "Confirmed":
            frappe.throw(_("Only confirmed reservations can be checked in"))
        
        self.status = "In House"
        self.actual_check_in_time = now_datetime()
        self.save()
        
        return {"status": "success", "message": _("Guest checked in successfully")}
```

### Field Naming Conventions
```json
{
  "fields": [
    {
      "fieldname": "guest_name",        // snake_case
      "fieldtype": "Link",
      "label": "Guest Name",             // Title Case
      "options": "Guest",
      "reqd": 1
    },
    {
      "fieldname": "check_in_date",     // Descriptive names
      "fieldtype": "Date",
      "label": "Check-in Date"
    },
    {
      "fieldname": "is_vip",            // Boolean prefix: is_, has_, can_
      "fieldtype": "Check",
      "label": "VIP Guest"
    },
    {
      "fieldname": "total_amount",      // Calculated fields: total_, calculated_
      "fieldtype": "Currency",
      "label": "Total Amount",
      "read_only": 1
    }
  ]
}
```

---

## 🔌 API Development

### RESTful API Standards
```python
# api/v1/reservation.py
import frappe
from frappe import _
from frappe.utils import validate_email_address

@frappe.whitelist(allow_guest=False, methods=['GET'])
def get_reservations(property_id=None, status=None, page=1, page_size=20):
    """
    Get list of reservations with pagination
    
    Args:
        property_id: Filter by property
        status: Filter by reservation status
        page: Page number (default: 1)
        page_size: Items per page (default: 20)
    
    Returns:
        dict: Paginated reservation list
    """
    filters = {}
    if property_id:
        filters['property'] = property_id
    if status:
        filters['status'] = status
    
    start = (int(page) - 1) * int(page_size)
    
    reservations = frappe.get_list(
        'Reservation',
        filters=filters,
        fields=['name', 'guest_name', 'check_in_date', 'check_out_date', 'status'],
        order_by='check_in_date desc',
        start=start,
        page_length=page_size
    )
    
    total_count = frappe.db.count('Reservation', filters)
    
    return {
        'data': reservations,
        'pagination': {
            'page': page,
            'page_size': page_size,
            'total_count': total_count,
            'total_pages': (total_count + page_size - 1) // page_size
        }
    }

@frappe.whitelist(allow_guest=False, methods=['POST'])
def create_reservation(**kwargs):
    """Create a new reservation"""
    # Validate required fields
    required_fields = ['guest', 'room', 'check_in_date', 'check_out_date']
    for field in required_fields:
        if not kwargs.get(field):
            frappe.throw(_(f"{field} is required"), frappe.MandatoryError)
    
    # Create reservation document
    reservation = frappe.get_doc({
        'doctype': 'Reservation',
        **kwargs
    })
    
    reservation.insert()
    frappe.db.commit()
    
    return {
        'status': 'success',
        'data': reservation.as_dict(),
        'message': _('Reservation created successfully')
    }
```

### Error Handling
```python
# utils/error_handler.py
import frappe
from functools import wraps

def handle_api_errors(func):
    """Decorator for consistent API error handling"""
    @wraps(func)
    def wrapper(*args, **kwargs):
        try:
            return func(*args, **kwargs)
        except frappe.PermissionError as e:
            frappe.response['http_status_code'] = 403
            return {
                'status': 'error',
                'error_code': 'PERMISSION_DENIED',
                'message': str(e)
            }
        except frappe.DoesNotExistError as e:
            frappe.response['http_status_code'] = 404
            return {
                'status': 'error',
                'error_code': 'NOT_FOUND',
                'message': str(e)
            }
        except frappe.ValidationError as e:
            frappe.response['http_status_code'] = 400
            return {
                'status': 'error',
                'error_code': 'VALIDATION_ERROR',
                'message': str(e)
            }
        except Exception as e:
            frappe.log_error(frappe.get_traceback(), 'API Error')
            frappe.response['http_status_code'] = 500
            return {
                'status': 'error',
                'error_code': 'INTERNAL_ERROR',
                'message': 'An unexpected error occurred'
            }
    
    return wrapper
```

---

## 🔐 Security Guidelines

### Input Validation
```python
# Always validate and sanitize input
def validate_reservation_input(data):
    # Validate dates
    if data.get('check_in_date') >= data.get('check_out_date'):
        frappe.throw(_("Check-out date must be after check-in date"))
    
    # Validate email
    if data.get('guest_email'):
        validate_email_address(data.get('guest_email'), throw=True)
    
    # Sanitize HTML input
    if data.get('special_requests'):
        data['special_requests'] = frappe.utils.strip_html(data['special_requests'])
    
    # Validate against injection
    if '<script>' in str(data.get('notes', '')).lower():
        frappe.throw(_("Invalid input detected"))
    
    return data
```

### SQL Injection Prevention
```python
# ✅ Good - Using parameterized queries
frappe.db.sql("""
    SELECT name, room_number 
    FROM tabRoom 
    WHERE property = %s AND status = %s
""", (property_id, 'Available'))

# ❌ Bad - String concatenation
frappe.db.sql(f"""
    SELECT name, room_number 
    FROM tabRoom 
    WHERE property = '{property_id}'
""")
```

---

## 📊 Performance Guidelines

### Database Queries
```python
# ✅ Good - Optimized query with specific fields
rooms = frappe.get_list('Room',
    filters={'property': property_id, 'status': 'Available'},
    fields=['name', 'room_number', 'room_type'],
    limit=100
)

# ❌ Bad - Fetching all fields
rooms = frappe.get_all('Room',
    filters={'property': property_id}
)
```

### Caching
```python
# Use Redis caching for expensive operations
@frappe.cache.memoize(ttl=300)  # Cache for 5 minutes
def get_room_availability(property_id, date):
    return frappe.db.sql("""
        SELECT COUNT(*) as available_rooms
        FROM tabRoom r
        WHERE r.property = %s
        AND r.name NOT IN (
            SELECT room FROM tabReservation
            WHERE %s BETWEEN check_in_date AND check_out_date
            AND status != 'Cancelled'
        )
    """, (property_id, date))[0][0]
```

### Background Jobs
```python
# Use background jobs for time-consuming tasks
@frappe.task(queue='long')
def generate_monthly_report(property_id, month, year):
    """Generate comprehensive monthly report"""
    # Long-running report generation
    pass

# Trigger the job
frappe.enqueue(
    generate_monthly_report,
    property_id='PROP001',
    month=9,
    year=2025,
    queue='long',
    timeout=600
)
```

---

## 📝 Documentation Standards

### Code Documentation
```python
def calculate_dynamic_pricing(
    base_rate: float,
    occupancy_rate: float,
    day_of_week: int,
    is_holiday: bool = False,
    competitor_rates: List[float] = None
) -> Dict[str, Any]:
    """
    Calculate dynamic room pricing based on multiple factors.
    
    This function implements a revenue management algorithm that adjusts
    room rates based on current occupancy, day of week, holidays, and
    competitor pricing.
    
    Args:
        base_rate: The standard room rate
        occupancy_rate: Current occupancy percentage (0-100)
        day_of_week: Day of week (0=Monday, 6=Sunday)
        is_holiday: Whether the date is a holiday
        competitor_rates: List of competitor rates for similar rooms
    
    Returns:
        Dict containing:
            - final_rate: The calculated room rate
            - adjustment_factor: The total adjustment applied
            - breakdown: Detailed breakdown of adjustments
    
    Raises:
        ValueError: If occupancy_rate is not between 0 and 100
        TypeError: If base_rate is not numeric
    
    Example:
        >>> calculate_dynamic_pricing(
        ...     base_rate=100.0,
        ...     occupancy_rate=75.0,
        ...     day_of_week=5,  # Saturday
        ...     is_holiday=False
        ... )
        {'final_rate': 125.0, 'adjustment_factor': 1.25, ...}
    """
    # Implementation here
    pass
```

---

## 🚀 Deployment Checklist

### Pre-deployment
- [ ] All tests passing
- [ ] Code review completed
- [ ] Documentation updated
- [ ] Database migrations prepared
- [ ] Environment variables configured
- [ ] Security scan completed
- [ ] Performance testing done
- [ ] Backup strategy verified

### Deployment Steps
```bash
# 1. Backup current state
bench backup

# 2. Pull latest code
git pull origin main

# 3. Install/update dependencies
bench setup requirements

# 4. Run migrations
bench migrate

# 5. Build assets
bench build

# 6. Clear cache
bench clear-cache

# 7. Restart services
bench restart

# 8. Verify deployment
bench doctor
```

### Post-deployment
- [ ] Smoke tests completed
- [ ] Monitoring alerts configured
- [ ] Performance metrics normal
- [ ] Error rates acceptable
- [ ] User acceptance verified
- [ ] Rollback plan ready

---

*Last Updated: September 2025*  
*Maintained by: Ekansh Jain ([@ekanshjain](https://github.com/ekanshjain))*  
*Contact: mailme@ekanshjain.com*
