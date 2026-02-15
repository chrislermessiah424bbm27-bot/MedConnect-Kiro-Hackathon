# MedConnect - System Design Document

## 1. Document Information
**Project Name:** MedConnect  
**Version:** 1.0  
**Date:** February 15, 2026  
**Technology Stack:** Python (Flask), SQLite, HTML/CSS/JavaScript

## 2. System Architecture Overview

### 2.1 High-Level Architecture
MedConnect follows a three-tier architecture pattern:

```
┌─────────────────────────────────────────────────────────────┐
│                    PRESENTATION LAYER                        │
│  (Web Browser - HTML/CSS/JavaScript + Bootstrap)            │
└─────────────────────────────────────────────────────────────┘
                            ↕ HTTP/HTTPS
┌─────────────────────────────────────────────────────────────┐
│                    APPLICATION LAYER                         │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   Patient    │  │    Doctor    │  │  Pharmacist  │     │
│  │   Module     │  │    Module    │  │    Module    │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │    Admin     │  │  Emergency   │  │     Auth     │     │
│  │   Module     │  │    Module    │  │    Module    │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│                                                              │
│              Flask Application (Python)                      │
│              - Routing & Controllers                         │
│              - Business Logic                                │
│              - Authentication & Authorization                │
└─────────────────────────────────────────────────────────────┘
                            ↕ SQLAlchemy ORM
┌─────────────────────────────────────────────────────────────┐
│                      DATA LAYER                              │
│                   SQLite Database                            │
│  (Unified Centralized Database)                             │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 Architectural Patterns
- **MVC Pattern:** Model-View-Controller separation
- **Blueprint Pattern:** Modular Flask blueprints for each module
- **Repository Pattern:** Data access abstraction through SQLAlchemy
- **Singleton Pattern:** Database connection management
- **Decorator Pattern:** Authentication and authorization decorators

### 2.3 Technology Stack Components

**Backend:**
- Flask (Web Framework)
- SQLAlchemy (ORM)
- Flask-Login (Session Management)
- Flask-WTF (Form Handling & CSRF Protection)
- Werkzeug (Password Hashing)

**Frontend:**
- HTML5/CSS3
- JavaScript (ES6+)
- Bootstrap 5 (Responsive UI)
- Jinja2 (Template Engine)

**Database:**
- SQLite (Development & Small-Scale Deployment)
- Migration Path: PostgreSQL/MySQL (Production Scale)

## 3. Module-Wise Design

### 3.1 Authentication Module

**Purpose:** Centralized authentication and authorization

**Components:**
- User registration handler
- Login/logout controller
- Session manager
- Password reset handler
- Role-based access control (RBAC) decorator

**Key Functions:**
```python
register_user(username, email, password, role)
authenticate_user(username, password)
check_permission(user, required_role)
reset_password(email)
logout_user()
```

**Security Features:**
- Password hashing with bcrypt
- Session token generation
- CSRF token validation
- Role verification middleware

### 3.2 Patient Module

**Purpose:** Patient-facing functionality

**Components:**
- Patient registration controller
- Profile management
- Appointment booking interface
- Medical record viewer
- Dashboard

**Key Functions:**
```python
create_patient_profile(user_id, demographics)
view_medical_history(patient_id)
book_appointment(patient_id, doctor_id, date, time)
view_appointments(patient_id)
cancel_appointment(appointment_id)
```

**Data Flow:**
1. Patient logs in → Session created
2. Patient views available doctors → Query doctors table
3. Patient books appointment → Validate availability → Create appointment record
4. Patient views medical records → Fetch records with doctor info

### 3.3 Doctor Module

**Purpose:** Doctor-facing functionality for patient care

**Components:**
- Doctor dashboard
- Appointment manager
- Medical record creator/editor
- Patient search interface
- Availability scheduler

**Key Functions:**
```python
view_appointments(doctor_id, date)
create_medical_record(patient_id, diagnosis, treatment, prescription)
update_medical_record(record_id, updates)
view_patient_history(patient_id)
set_availability(doctor_id, schedule)
search_patients(criteria)
```

**Data Flow:**
1. Doctor logs in → Load dashboard with today's appointments
2. Doctor selects patient → Load complete medical history
3. Doctor creates record → Validate data → Save to database → Update patient timeline
4. Doctor prescribes medicine → Check inventory → Create prescription record


### 3.4 Pharmacist Module

**Purpose:** Medicine inventory and prescription management

**Components:**
- Medicine inventory manager
- Stock level monitor
- Expiry alert system
- Prescription processor
- Inventory reports generator

**Key Functions:**
```python
add_medicine(name, type, dosage, quantity, expiry_date, batch)
update_stock(medicine_id, quantity_change)
check_expiry_alerts()
process_prescription(prescription_id)
generate_inventory_report(date_range)
set_stock_threshold(medicine_id, threshold)
search_medicine(criteria)
```

**Alert System Logic:**
```python
# Daily scheduled task
def check_medicine_expiry():
    today = datetime.now()
    medicines = get_all_medicines()
    
    for medicine in medicines:
        days_to_expiry = (medicine.expiry_date - today).days
        
        if days_to_expiry in [30, 15, 7, 3, 1]:
            create_alert(medicine, days_to_expiry)
            notify_pharmacist(medicine)
        
        if medicine.quantity < medicine.threshold:
            create_low_stock_alert(medicine)
```

**Data Flow:**
1. Pharmacist adds medicine → Validate data → Store in inventory
2. System runs daily check → Identify expiring medicines → Generate alerts
3. Stock falls below threshold → Trigger low stock alert
4. Prescription processed → Update stock levels → Log transaction


### 3.5 Administrator Module

**Purpose:** System administration and resource management

**Components:**
- User management interface
- Resource allocation manager
- System configuration panel
- Pandemic mode controller
- Analytics dashboard
- Ventilator tracking system

**Key Functions:**
```python
create_user(username, email, role)
deactivate_user(user_id)
manage_resources(resource_type, action, quantity)
activate_pandemic_mode()
deactivate_pandemic_mode()
track_ventilator_movement(ventilator_id, from_location, to_location)
generate_system_reports(report_type, parameters)
view_audit_logs(filters)
```

**Resource Management Logic:**
```python
def allocate_resource(resource_type, quantity, priority=None):
    resource = get_resource(resource_type)
    
    if pandemic_mode_active():
        if priority == 'HIGH':
            reserve_resource(resource, quantity)
        else:
            check_availability_after_reservation(resource, quantity)
    
    if resource.available_count >= quantity:
        resource.available_count -= quantity
        log_allocation(resource, quantity)
        return True
    return False
```

**Ventilator Tracking:**
```python
def move_ventilator(ventilator_id, to_location, moved_by):
    ventilator = get_ventilator(ventilator_id)
    
    movement_log = {
        'ventilator_id': ventilator_id,
        'from_location': ventilator.current_location,
        'to_location': to_location,
        'moved_by': moved_by,
        'timestamp': datetime.now()
    }
    
    ventilator.current_location = to_location
    save_movement_log(movement_log)
    update_resource_availability(ventilator.current_location)
```


### 3.6 Emergency Module

**Purpose:** Emergency coordination and alert management

**Components:**
- Ambulance alert broadcaster
- Internal emergency alert system
- Hospital response manager
- Emergency resource allocator
- Alert status tracker

**Key Functions:**
```python
broadcast_ambulance_alert(patient_condition, required_resources, location)
receive_hospital_response(hospital_id, available_resources)
create_internal_emergency(emergency_type, location, severity)
notify_departments(emergency_id, department_list)
allocate_emergency_resources(emergency_id, resources)
update_emergency_status(emergency_id, status)
```

**Ambulance Alert Workflow:**
```python
def broadcast_ambulance_alert(alert_data):
    # Create alert record
    alert = create_alert_record(alert_data)
    
    # Get all hospitals in network
    hospitals = get_nearby_hospitals(alert_data['location'], radius=50)
    
    # Broadcast to all hospitals
    responses = []
    for hospital in hospitals:
        response = send_alert_to_hospital(hospital, alert)
        responses.append(response)
    
    # Collect real-time availability
    for response in responses:
        if response.can_accommodate:
            update_alert_with_hospital_info(alert, response)
    
    # Notify ambulance of available hospitals
    return get_available_hospitals(alert.id)
```

**Internal Emergency Workflow:**
```python
def trigger_internal_emergency(emergency_data):
    # Create emergency record
    emergency = create_emergency_record(emergency_data)
    
    # Determine affected departments
    departments = determine_departments(emergency.type, emergency.location)
    
    # Send notifications
    for dept in departments:
        send_notification(dept, emergency)
        alert_on_duty_staff(dept, emergency)
    
    # Allocate resources
    if emergency.severity == 'CRITICAL':
        reserve_emergency_resources(emergency)
    
    # Start tracking
    create_response_tracker(emergency)
```


## 4. Data Flow Diagrams

### 4.1 Patient Appointment Booking Flow

```
Patient → Login → Dashboard → View Doctors
                                    ↓
                            Select Doctor & Date
                                    ↓
                        Check Doctor Availability
                                    ↓
                    [Available?] ─No→ Show Alternative Slots
                         │
                        Yes
                         ↓
                    Create Appointment
                         ↓
                    Update Doctor Schedule
                         ↓
                Send Confirmation (Email/SMS)
                         ↓
                    Update Patient Dashboard
```

### 4.2 Medical Record Creation Flow

```
Doctor → Login → View Appointments → Select Patient
                                          ↓
                                Load Patient History
                                          ↓
                                Create New Record
                                          ↓
                        Enter: Diagnosis, Symptoms, Treatment
                                          ↓
                                Add Prescriptions
                                          ↓
                            Check Medicine Availability
                                          ↓
                                Validate Data
                                          ↓
                            Save to Database
                                          ↓
                        Update Patient Timeline
                                          ↓
                            Log Audit Entry
```

### 4.3 Emergency Alert Flow

```
Ambulance → Create Alert → Specify Patient Condition
                                    ↓
                        Specify Required Resources
                                    ↓
                    Broadcast to Hospital Network
                                    ↓
            ┌───────────────┬───────────────┬───────────────┐
            ↓               ↓               ↓               ↓
        Hospital A      Hospital B      Hospital C      Hospital D
            ↓               ↓               ↓               ↓
    Check Resources  Check Resources  Check Resources  Check Resources
            ↓               ↓               ↓               ↓
    Send Response    Send Response    Send Response    Send Response
            └───────────────┴───────────────┴───────────────┘
                                    ↓
                        Aggregate Responses
                                    ↓
                    Display Available Hospitals
                                    ↓
                    Ambulance Selects Hospital
                                    ↓
                        Update Alert Status
                                    ↓
                    Reserve Resources at Hospital
```


### 4.4 Pandemic Mode Activation Flow

```
Administrator → Activate Pandemic Mode
                        ↓
            Set System-Wide Flag
                        ↓
        Update Resource Allocation Rules
                        ↓
    ┌───────────────────┴───────────────────┐
    ↓                                       ↓
Priority Bed Allocation              Enhanced Monitoring
    ↓                                       ↓
Reserve Critical Resources          Real-time Dashboards
    ↓                                       ↓
Modify Appointment Rules            Alert Thresholds
    ↓                                       ↓
Notify All Departments              Capacity Forecasting
```

## 5. Database Design

### 5.1 Entity Relationship Diagram

```
┌─────────────┐         ┌─────────────┐         ┌─────────────┐
│    Users    │         │  Patients   │         │   Doctors   │
├─────────────┤         ├─────────────┤         ├─────────────┤
│ id (PK)     │────1:1──│ id (PK)     │         │ id (PK)     │
│ username    │         │ user_id(FK) │         │ user_id(FK) │
│ email       │         │ name        │         │ name        │
│ password    │         │ dob         │         │ specializ.  │
│ role        │         │ blood_group │         │ qualific.   │
│ is_active   │         │ contact     │         │ contact     │
│ created_at  │         │ address     │         │ schedule    │
└─────────────┘         └─────────────┘         └─────────────┘
                              │                        │
                              │                        │
                              └────────┬───────────────┘
                                       │
                                       ↓
                            ┌─────────────────┐
                            │  Appointments   │
                            ├─────────────────┤
                            │ id (PK)         │
                            │ patient_id (FK) │
                            │ doctor_id (FK)  │
                            │ appt_date       │
                            │ time_slot       │
                            │ status          │
                            │ notes           │
                            └─────────────────┘
                                       │
                                       │
                            ┌──────────┴──────────┐
                            ↓                     ↓
                  ┌──────────────────┐  ┌──────────────────┐
                  │ Medical_Records  │  │  Prescriptions   │
                  ├──────────────────┤  ├──────────────────┤
                  │ id (PK)          │  │ id (PK)          │
                  │ patient_id (FK)  │  │ record_id (FK)   │
                  │ doctor_id (FK)   │  │ medicine_id (FK) │
                  │ visit_date       │  │ dosage           │
                  │ diagnosis        │  │ duration         │
                  │ symptoms         │  │ instructions     │
                  │ treatment        │  └──────────────────┘
                  └──────────────────┘


┌─────────────────┐         ┌─────────────────┐
│    Medicines    │         │    Resources    │
├─────────────────┤         ├─────────────────┤
│ id (PK)         │         │ id (PK)         │
│ name            │         │ resource_type   │
│ type            │         │ total_count     │
│ dosage          │         │ available_count │
│ quantity        │         │ location        │
│ expiry_date     │         │ last_updated    │
│ batch_number    │         │ reserved_count  │
│ threshold_level │         └─────────────────┘
└─────────────────┘

┌──────────────────┐        ┌──────────────────┐
│   Ventilators    │        │ Emergency_Alerts │
├──────────────────┤        ├──────────────────┤
│ id (PK)          │        │ id (PK)          │
│ ventilator_id    │        │ alert_type       │
│ current_location │        │ severity         │
│ status           │        │ description      │
│ patient_id (FK)  │        │ patient_info     │
│ last_moved       │        │ required_res     │
└──────────────────┘        │ status           │
                            │ created_at       │
┌──────────────────┐        │ resolved_at      │
│ Ventilator_Moves │        └──────────────────┘
├──────────────────┤
│ id (PK)          │        ┌──────────────────┐
│ ventilator_id(FK)│        │   Audit_Logs     │
│ from_location    │        ├──────────────────┤
│ to_location      │        │ id (PK)          │
│ moved_by (FK)    │        │ user_id (FK)     │
│ timestamp        │        │ action           │
│ reason           │        │ table_name       │
└──────────────────┘        │ record_id        │
                            │ timestamp        │
                            │ ip_address       │
                            │ details          │
                            └──────────────────┘
```

### 5.2 Key Database Tables

**Users Table:**
```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    role VARCHAR(20) NOT NULL,
    is_active BOOLEAN DEFAULT 1,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_login TIMESTAMP
);
```

**Patients Table:**
```sql
CREATE TABLE patients (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER UNIQUE NOT NULL,
    name VARCHAR(100) NOT NULL,
    date_of_birth DATE NOT NULL,
    gender VARCHAR(10),
    blood_group VARCHAR(5),
    contact VARCHAR(15),
    address TEXT,
    emergency_contact VARCHAR(15),
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```


**Doctors Table:**
```sql
CREATE TABLE doctors (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER UNIQUE NOT NULL,
    name VARCHAR(100) NOT NULL,
    specialization VARCHAR(100),
    qualification VARCHAR(200),
    contact VARCHAR(15),
    availability_schedule TEXT,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

**Appointments Table:**
```sql
CREATE TABLE appointments (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    patient_id INTEGER NOT NULL,
    doctor_id INTEGER NOT NULL,
    appointment_date DATE NOT NULL,
    time_slot VARCHAR(20) NOT NULL,
    status VARCHAR(20) DEFAULT 'scheduled',
    notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (patient_id) REFERENCES patients(id),
    FOREIGN KEY (doctor_id) REFERENCES doctors(id)
);
```

**Medical_Records Table:**
```sql
CREATE TABLE medical_records (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    patient_id INTEGER NOT NULL,
    doctor_id INTEGER NOT NULL,
    visit_date DATE NOT NULL,
    diagnosis TEXT,
    symptoms TEXT,
    treatment TEXT,
    vital_signs TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP,
    FOREIGN KEY (patient_id) REFERENCES patients(id),
    FOREIGN KEY (doctor_id) REFERENCES doctors(id)
);
```

**Medicines Table:**
```sql
CREATE TABLE medicines (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name VARCHAR(100) NOT NULL,
    type VARCHAR(50),
    dosage VARCHAR(50),
    quantity INTEGER NOT NULL,
    expiry_date DATE NOT NULL,
    batch_number VARCHAR(50),
    threshold_level INTEGER DEFAULT 10,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP
);
```

**Resources Table:**
```sql
CREATE TABLE resources (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    resource_type VARCHAR(50) NOT NULL,
    total_count INTEGER NOT NULL,
    available_count INTEGER NOT NULL,
    reserved_count INTEGER DEFAULT 0,
    location VARCHAR(100),
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```


**Ventilators Table:**
```sql
CREATE TABLE ventilators (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    ventilator_id VARCHAR(50) UNIQUE NOT NULL,
    current_location VARCHAR(100),
    status VARCHAR(20) DEFAULT 'available',
    patient_id INTEGER,
    last_moved TIMESTAMP,
    FOREIGN KEY (patient_id) REFERENCES patients(id)
);
```

**Emergency_Alerts Table:**
```sql
CREATE TABLE emergency_alerts (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    alert_type VARCHAR(50) NOT NULL,
    severity VARCHAR(20) NOT NULL,
    description TEXT,
    patient_info TEXT,
    required_resources TEXT,
    location VARCHAR(200),
    status VARCHAR(20) DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    resolved_at TIMESTAMP,
    created_by INTEGER,
    FOREIGN KEY (created_by) REFERENCES users(id)
);
```

**Audit_Logs Table:**
```sql
CREATE TABLE audit_logs (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER NOT NULL,
    action VARCHAR(100) NOT NULL,
    table_name VARCHAR(50),
    record_id INTEGER,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    ip_address VARCHAR(45),
    details TEXT,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

### 5.3 Database Indexes

```sql
-- Performance optimization indexes
CREATE INDEX idx_appointments_patient ON appointments(patient_id);
CREATE INDEX idx_appointments_doctor ON appointments(doctor_id);
CREATE INDEX idx_appointments_date ON appointments(appointment_date);
CREATE INDEX idx_medical_records_patient ON medical_records(patient_id);
CREATE INDEX idx_medical_records_date ON medical_records(visit_date);
CREATE INDEX idx_medicines_expiry ON medicines(expiry_date);
CREATE INDEX idx_emergency_alerts_status ON emergency_alerts(status);
CREATE INDEX idx_audit_logs_user ON audit_logs(user_id);
CREATE INDEX idx_audit_logs_timestamp ON audit_logs(timestamp);
```


## 6. Security and Authentication Design

### 6.1 Authentication Flow

```
User → Enter Credentials → Validate Format
                                  ↓
                        Query User from Database
                                  ↓
                    [User Exists?] ─No→ Return Error
                         │
                        Yes
                         ↓
                Verify Password Hash
                         ↓
            [Password Correct?] ─No→ Return Error
                         │
                        Yes
                         ↓
                Check Account Status
                         ↓
            [Account Active?] ─No→ Return Error
                         │
                        Yes
                         ↓
                Create Session Token
                         ↓
                Store Session in DB
                         ↓
                Set Secure Cookie
                         ↓
                Log Audit Entry
                         ↓
                Redirect to Dashboard
```

### 6.2 Authorization Mechanism

**Role-Based Access Control (RBAC):**

```python
# Decorator for route protection
def role_required(allowed_roles):
    def decorator(f):
        @wraps(f)
        def decorated_function(*args, **kwargs):
            if not current_user.is_authenticated:
                return redirect(url_for('auth.login'))
            
            if current_user.role not in allowed_roles:
                abort(403)  # Forbidden
            
            return f(*args, **kwargs)
        return decorated_function
    return decorator

# Usage
@app.route('/admin/dashboard')
@role_required(['admin'])
def admin_dashboard():
    return render_template('admin_dashboard.html')
```

**Access Control Matrix:**

| Resource              | Patient | Doctor | Pharmacist | Admin |
|-----------------------|---------|--------|------------|-------|
| Own Profile           | RW      | RW     | RW         | RW    |
| Other Profiles        | -       | R      | -          | RW    |
| Appointments          | RW      | RW     | -          | R     |
| Medical Records       | R       | RW     | -          | R     |
| Medicines             | R       | R      | RW         | RW    |
| Resources             | R       | R      | R          | RW    |
| Emergency Alerts      | -       | R      | R          | RW    |
| User Management       | -       | -      | -          | RW    |
| System Config         | -       | -      | -          | RW    |
| Audit Logs            | -       | -      | -          | R     |

(R = Read, W = Write, - = No Access)


### 6.3 Security Measures

**Password Security:**
```python
from werkzeug.security import generate_password_hash, check_password_hash

# Password hashing
def hash_password(password):
    return generate_password_hash(password, method='pbkdf2:sha256', salt_length=16)

# Password verification
def verify_password(password_hash, password):
    return check_password_hash(password_hash, password)

# Password policy
def validate_password(password):
    if len(password) < 8:
        return False, "Password must be at least 8 characters"
    if not any(c.isupper() for c in password):
        return False, "Password must contain uppercase letter"
    if not any(c.isdigit() for c in password):
        return False, "Password must contain a number"
    return True, "Valid"
```

**Session Security:**
```python
# Flask session configuration
app.config['SESSION_COOKIE_SECURE'] = True  # HTTPS only
app.config['SESSION_COOKIE_HTTPONLY'] = True  # No JavaScript access
app.config['SESSION_COOKIE_SAMESITE'] = 'Lax'  # CSRF protection
app.config['PERMANENT_SESSION_LIFETIME'] = timedelta(minutes=30)
```

**CSRF Protection:**
```python
from flask_wtf.csrf import CSRFProtect

csrf = CSRFProtect(app)

# All POST requests require CSRF token
# Token automatically added to forms via Flask-WTF
```

**SQL Injection Prevention:**
```python
# Use parameterized queries via SQLAlchemy
# BAD: f"SELECT * FROM users WHERE username = '{username}'"
# GOOD:
user = User.query.filter_by(username=username).first()
```

**XSS Prevention:**
```python
# Jinja2 auto-escapes by default
# {{ user_input }} is automatically escaped
# Use |safe filter only for trusted content
```

**Input Validation:**
```python
from flask_wtf import FlaskForm
from wtforms import StringField, PasswordField
from wtforms.validators import DataRequired, Email, Length

class LoginForm(FlaskForm):
    email = StringField('Email', validators=[DataRequired(), Email()])
    password = PasswordField('Password', validators=[DataRequired(), Length(min=8)])
```


### 6.4 Audit Logging

```python
def log_audit(user_id, action, table_name=None, record_id=None, details=None):
    """Log all sensitive operations for compliance"""
    audit_log = AuditLog(
        user_id=user_id,
        action=action,
        table_name=table_name,
        record_id=record_id,
        ip_address=request.remote_addr,
        details=details,
        timestamp=datetime.now()
    )
    db.session.add(audit_log)
    db.session.commit()

# Usage examples
log_audit(current_user.id, 'VIEW_MEDICAL_RECORD', 'medical_records', record_id)
log_audit(current_user.id, 'UPDATE_MEDICINE_STOCK', 'medicines', medicine_id)
log_audit(current_user.id, 'CREATE_EMERGENCY_ALERT', 'emergency_alerts', alert_id)
```

## 7. Emergency Alert Workflow

### 7.1 Ambulance-to-Hospital Alert System

**Detailed Workflow:**

```
Step 1: Alert Creation
├─ Ambulance staff creates emergency alert
├─ Input: Patient condition, required resources, location
└─ System validates input and creates alert record

Step 2: Hospital Network Query
├─ System identifies hospitals within radius
├─ Filters based on specialization if needed
└─ Prepares broadcast message

Step 3: Broadcast Alert
├─ Send alert to all identified hospitals simultaneously
├─ Include: Patient info, required resources, urgency level
└─ Set response timeout (e.g., 60 seconds)

Step 4: Hospital Response Collection
├─ Each hospital checks resource availability
│   ├─ Beds available?
│   ├─ Required equipment available?
│   ├─ Specialist doctors on duty?
│   └─ ICU/Emergency capacity?
├─ Hospital sends response with availability status
└─ System aggregates all responses

Step 5: Display Results
├─ Show ambulance staff list of available hospitals
├─ Display: Hospital name, distance, available resources
├─ Sort by: Distance, resource match, response time
└─ Allow ambulance to select destination

Step 6: Resource Reservation
├─ Ambulance selects hospital
├─ System reserves required resources
├─ Notify selected hospital of incoming patient
├─ Update alert status to 'en_route'
└─ Provide ETA to hospital
```


**Implementation Code:**

```python
class EmergencyAlertSystem:
    
    def broadcast_ambulance_alert(self, alert_data):
        """Broadcast emergency alert to hospital network"""
        
        # Step 1: Create alert
        alert = EmergencyAlert(
            alert_type='ambulance',
            severity=alert_data['severity'],
            patient_info=alert_data['patient_info'],
            required_resources=json.dumps(alert_data['resources']),
            location=alert_data['location'],
            status='broadcasting'
        )
        db.session.add(alert)
        db.session.commit()
        
        # Step 2: Get nearby hospitals
        hospitals = self.get_nearby_hospitals(
            alert_data['location'], 
            radius=50  # km
        )
        
        # Step 3: Broadcast to hospitals
        responses = []
        for hospital in hospitals:
            response = self.send_alert_to_hospital(hospital, alert)
            responses.append(response)
        
        # Step 4: Collect responses (with timeout)
        available_hospitals = []
        for response in responses:
            if response.can_accommodate:
                hospital_info = {
                    'hospital_id': response.hospital_id,
                    'name': response.hospital_name,
                    'distance': response.distance,
                    'available_resources': response.resources,
                    'response_time': response.timestamp
                }
                available_hospitals.append(hospital_info)
        
        # Step 5: Return sorted results
        available_hospitals.sort(key=lambda x: x['distance'])
        return {
            'alert_id': alert.id,
            'available_hospitals': available_hospitals
        }
    
    def reserve_resources_for_ambulance(self, alert_id, hospital_id):
        """Reserve resources at selected hospital"""
        
        alert = EmergencyAlert.query.get(alert_id)
        required_resources = json.loads(alert.required_resources)
        
        # Reserve each resource
        for resource_type, quantity in required_resources.items():
            resource = Resource.query.filter_by(
                resource_type=resource_type,
                hospital_id=hospital_id
            ).first()
            
            if resource.available_count >= quantity:
                resource.reserved_count += quantity
                resource.available_count -= quantity
            else:
                raise InsufficientResourcesError()
        
        # Update alert status
        alert.status = 'en_route'
        alert.selected_hospital_id = hospital_id
        db.session.commit()
        
        # Notify hospital
        self.notify_hospital_of_incoming_patient(hospital_id, alert)
```


### 7.2 Internal Hospital Emergency Alert

**Workflow:**

```
Step 1: Emergency Detection
├─ Staff member identifies emergency (accident, code blue, etc.)
└─ Accesses emergency alert interface

Step 2: Alert Creation
├─ Select emergency type (fire, accident, cardiac arrest, etc.)
├─ Specify location within hospital
├─ Set severity level (low, medium, high, critical)
└─ Add description and special instructions

Step 3: Department Notification
├─ System determines affected departments based on emergency type
├─ Emergency Department
├─ ICU
├─ Relevant specialists
├─ Security
└─ Administration

Step 4: Alert Broadcast
├─ Send notifications to all relevant departments
├─ Display on department dashboards
├─ Send SMS/email to on-duty staff
└─ Activate visual/audio alerts if configured

Step 5: Resource Mobilization
├─ Check required resources for emergency type
├─ Allocate resources automatically
├─ Track resource movement
└─ Update availability in real-time

Step 6: Response Tracking
├─ Track which departments acknowledged alert
├─ Monitor response times
├─ Update emergency status
└─ Log all actions taken

Step 7: Resolution
├─ Mark emergency as resolved
├─ Release reserved resources
├─ Generate incident report
└─ Archive for analysis
```

**Implementation:**

```python
def create_internal_emergency(emergency_data):
    """Create and broadcast internal emergency alert"""
    
    # Create emergency record
    emergency = EmergencyAlert(
        alert_type='internal',
        severity=emergency_data['severity'],
        description=emergency_data['description'],
        location=emergency_data['location'],
        status='active'
    )
    db.session.add(emergency)
    db.session.commit()
    
    # Determine departments to notify
    departments = get_departments_for_emergency(emergency.alert_type)
    
    # Broadcast to departments
    for dept in departments:
        notification = Notification(
            department_id=dept.id,
            emergency_id=emergency.id,
            message=f"Emergency Alert: {emergency.description}",
            priority='high',
            status='sent'
        )
        db.session.add(notification)
        
        # Send real-time notification
        send_realtime_notification(dept, emergency)
    
    # Allocate resources if critical
    if emergency.severity == 'critical':
        allocate_emergency_resources(emergency)
    
    db.session.commit()
    
    # Log audit entry
    log_audit(current_user.id, 'CREATE_INTERNAL_EMERGENCY', 
              'emergency_alerts', emergency.id)
    
    return emergency.id
```


## 8. Scalability Considerations

### 8.1 Current Architecture Limitations

**SQLite Limitations:**
- Single-file database
- Limited concurrent write operations
- No built-in replication
- Maximum database size: ~281 TB (theoretical, practical limits lower)
- Not suitable for high-concurrency production environments

**Single-Server Deployment:**
- Single point of failure
- Limited horizontal scaling
- Resource constraints on single machine

### 8.2 Scalability Strategies

**Phase 1: Database Migration (Immediate Production Need)**

```
SQLite → PostgreSQL/MySQL

Benefits:
├─ True concurrent connections
├─ Better transaction handling
├─ Advanced indexing capabilities
├─ Replication support
├─ Connection pooling
└─ Better performance at scale

Migration Steps:
1. Export SQLite data
2. Set up PostgreSQL/MySQL
3. Update SQLAlchemy connection string
4. Import data
5. Test thoroughly
6. Deploy
```

**Phase 2: Application Scaling**

```
Load Balancing Architecture:

                    ┌─────────────┐
                    │ Load Balancer│
                    │   (Nginx)    │
                    └──────┬───────┘
                           │
        ┏━━━━━━━━━━━━━━━━━━┻━━━━━━━━━━━━━━━━━━┓
        ↓                  ↓                   ↓
┌───────────────┐  ┌───────────────┐  ┌───────────────┐
│ Flask App     │  │ Flask App     │  │ Flask App     │
│ Instance 1    │  │ Instance 2    │  │ Instance 3    │
└───────┬───────┘  └───────┬───────┘  └───────┬───────┘
        └──────────────────┴───────────────────┘
                           ↓
                ┌─────────────────────┐
                │  PostgreSQL Master  │
                └──────────┬──────────┘
                           │
                ┌──────────┴──────────┐
                ↓                     ↓
        ┌──────────────┐      ┌──────────────┐
        │ Read Replica │      │ Read Replica │
        └──────────────┘      └──────────────┘
```

**Phase 3: Caching Layer**

```python
# Redis caching for frequently accessed data
from flask_caching import Cache

cache = Cache(app, config={
    'CACHE_TYPE': 'redis',
    'CACHE_REDIS_URL': 'redis://localhost:6379/0'
})

@cache.cached(timeout=300, key_prefix='doctor_availability')
def get_doctor_availability(doctor_id):
    return Doctor.query.get(doctor_id).availability_schedule

@cache.cached(timeout=60, key_prefix='resource_availability')
def get_resource_availability():
    return Resource.query.all()
```


**Phase 4: Microservices Architecture (Long-term)**

```
Monolithic Flask App → Microservices

Service Decomposition:
├─ Authentication Service
├─ Patient Management Service
├─ Appointment Service
├─ Medical Records Service
├─ Inventory Service
├─ Emergency Alert Service
└─ Resource Management Service

Benefits:
├─ Independent scaling
├─ Technology flexibility
├─ Fault isolation
├─ Easier maintenance
└─ Team autonomy

Communication:
├─ REST APIs
├─ Message Queue (RabbitMQ/Kafka)
└─ Service mesh (Istio)
```

### 8.3 Performance Optimization

**Database Optimization:**

```python
# Eager loading to prevent N+1 queries
appointments = Appointment.query\
    .options(joinedload(Appointment.patient))\
    .options(joinedload(Appointment.doctor))\
    .filter_by(doctor_id=doctor_id)\
    .all()

# Pagination for large result sets
def get_paginated_records(page=1, per_page=20):
    return MedicalRecord.query\
        .order_by(MedicalRecord.visit_date.desc())\
        .paginate(page=page, per_page=per_page)

# Database connection pooling
app.config['SQLALCHEMY_POOL_SIZE'] = 10
app.config['SQLALCHEMY_POOL_RECYCLE'] = 3600
app.config['SQLALCHEMY_POOL_TIMEOUT'] = 30
```

**Query Optimization:**

```sql
-- Use covering indexes
CREATE INDEX idx_appointments_lookup 
ON appointments(doctor_id, appointment_date, status);

-- Partition large tables
CREATE TABLE medical_records_2026 
PARTITION OF medical_records 
FOR VALUES FROM ('2026-01-01') TO ('2027-01-01');

-- Materialized views for complex queries
CREATE MATERIALIZED VIEW daily_resource_summary AS
SELECT 
    resource_type,
    SUM(available_count) as total_available,
    AVG(available_count) as avg_available,
    DATE(last_updated) as date
FROM resources
GROUP BY resource_type, DATE(last_updated);
```

**Frontend Optimization:**

```javascript
// Lazy loading for large lists
// Debouncing for search inputs
// Client-side caching
// Minimize API calls
// Use pagination
```


### 8.4 High Availability Design

**Database Replication:**

```
Master-Slave Replication:

┌─────────────────┐
│  Master DB      │ ← All Writes
│  (Primary)      │
└────────┬────────┘
         │ Replication
    ┌────┴────┐
    ↓         ↓
┌────────┐ ┌────────┐
│Slave 1 │ │Slave 2 │ ← Read Operations
└────────┘ └────────┘

Benefits:
- Read scaling
- Backup redundancy
- Failover capability
```

**Application Redundancy:**

```
Active-Active Configuration:

┌──────────────┐
│ Health Check │
│  Monitor     │
└──────┬───────┘
       │
       ↓
┌──────────────┐     ┌──────────────┐
│  App Server  │ ←→  │  App Server  │
│  Instance 1  │     │  Instance 2  │
│  (Active)    │     │  (Active)    │
└──────────────┘     └──────────────┘

- Both serve traffic
- Automatic failover
- Session replication
```

**Backup Strategy:**

```
Backup Schedule:
├─ Full backup: Daily at 2 AM
├─ Incremental backup: Every 6 hours
├─ Transaction logs: Continuous
└─ Retention: 30 days

Backup Locations:
├─ Local storage (immediate recovery)
├─ Remote storage (disaster recovery)
└─ Cloud storage (long-term archival)
```

## 9. Future Enhancement Considerations

### 9.1 Real-Time Communication

**WebSocket Integration:**

```python
from flask_socketio import SocketIO, emit

socketio = SocketIO(app)

# Real-time resource updates
@socketio.on('subscribe_resources')
def handle_resource_subscription():
    # Client subscribes to resource updates
    join_room('resource_updates')

def broadcast_resource_update(resource):
    socketio.emit('resource_updated', {
        'resource_type': resource.resource_type,
        'available_count': resource.available_count
    }, room='resource_updates')

# Real-time emergency alerts
@socketio.on('subscribe_emergency')
def handle_emergency_subscription(department_id):
    join_room(f'emergency_{department_id}')

def broadcast_emergency_alert(alert, departments):
    for dept in departments:
        socketio.emit('emergency_alert', {
            'alert_id': alert.id,
            'severity': alert.severity,
            'description': alert.description
        }, room=f'emergency_{dept.id}')
```


### 9.2 Mobile Application Architecture

```
Mobile App Architecture:

┌─────────────────────────────────────┐
│     Mobile Applications             │
│  ┌──────────┐    ┌──────────┐      │
│  │   iOS    │    │ Android  │      │
│  │  (Swift) │    │ (Kotlin) │      │
│  └──────────┘    └──────────┘      │
└─────────────┬───────────────────────┘
              │ REST API / GraphQL
              ↓
┌─────────────────────────────────────┐
│      API Gateway                    │
│  - Authentication                   │
│  - Rate limiting                    │
│  - Request routing                  │
└─────────────┬───────────────────────┘
              ↓
┌─────────────────────────────────────┐
│   Backend Services (Flask)          │
└─────────────────────────────────────┘

Features:
├─ Push notifications for appointments
├─ Offline mode with sync
├─ Biometric authentication
├─ QR code for patient check-in
└─ Telemedicine video calls
```

### 9.3 AI/ML Integration

**Predictive Analytics:**

```python
# Resource demand prediction
class ResourcePredictor:
    def predict_bed_demand(self, date, historical_data):
        """Predict bed demand using time series analysis"""
        # Use Prophet, ARIMA, or LSTM
        model = load_trained_model('bed_demand_model')
        prediction = model.predict(date)
        return prediction
    
    def predict_medicine_consumption(self, medicine_id):
        """Predict when medicine stock will run out"""
        historical_usage = get_medicine_usage_history(medicine_id)
        current_stock = get_current_stock(medicine_id)
        
        avg_daily_usage = calculate_average_usage(historical_usage)
        days_remaining = current_stock / avg_daily_usage
        
        return {
            'estimated_depletion_date': date.today() + timedelta(days=days_remaining),
            'recommended_reorder_date': date.today() + timedelta(days=days_remaining - 7)
        }

# Pandemic outbreak prediction
def predict_pandemic_risk(region_data):
    """Analyze trends to predict pandemic risk"""
    # Analyze: case trends, hospital admissions, resource usage
    risk_score = ml_model.predict(region_data)
    
    if risk_score > 0.7:
        recommend_pandemic_mode_activation()
    
    return risk_score
```

**Intelligent Appointment Scheduling:**

```python
def suggest_optimal_appointment_slot(patient_id, doctor_id, preferred_date):
    """AI-powered appointment suggestion"""
    
    # Consider factors:
    # - Doctor availability
    # - Patient history (no-show rate)
    # - Travel time for patient
    # - Doctor's workload distribution
    # - Emergency buffer slots
    
    available_slots = get_available_slots(doctor_id, preferred_date)
    patient_profile = get_patient_profile(patient_id)
    
    scored_slots = []
    for slot in available_slots:
        score = calculate_slot_score(slot, patient_profile, doctor_id)
        scored_slots.append((slot, score))
    
    # Return top 3 recommendations
    return sorted(scored_slots, key=lambda x: x[1], reverse=True)[:3]
```


### 9.4 Integration with External Systems

**Health Information Exchange (HIE):**

```python
class HIEIntegration:
    """Integration with national health information exchange"""
    
    def fetch_patient_records_from_network(self, patient_id, consent_token):
        """Retrieve patient records from other hospitals"""
        
        # FHIR (Fast Healthcare Interoperability Resources) standard
        fhir_client = FHIRClient(base_url=HIE_ENDPOINT)
        
        patient_bundle = fhir_client.search('Patient', {
            'identifier': patient_id,
            'consent': consent_token
        })
        
        # Parse and integrate records
        external_records = parse_fhir_bundle(patient_bundle)
        merge_with_local_records(patient_id, external_records)
        
        return external_records
    
    def share_patient_records(self, patient_id, target_hospital, consent):
        """Share patient records with another hospital"""
        
        if not verify_patient_consent(patient_id, consent):
            raise ConsentError("Patient consent not provided")
        
        records = get_patient_records(patient_id)
        fhir_bundle = convert_to_fhir_format(records)
        
        # Send to HIE
        response = fhir_client.post(target_hospital, fhir_bundle)
        log_data_sharing(patient_id, target_hospital, response)
```

**Laboratory Information System (LIS):**

```python
class LISIntegration:
    """Integration with laboratory systems"""
    
    def order_lab_test(self, patient_id, test_type, doctor_id):
        """Send lab test order to LIS"""
        
        order = {
            'patient_id': patient_id,
            'test_type': test_type,
            'ordered_by': doctor_id,
            'priority': 'routine',
            'timestamp': datetime.now()
        }
        
        # Send to LIS via HL7 message
        hl7_message = create_hl7_order(order)
        response = send_to_lis(hl7_message)
        
        return response.order_id
    
    def receive_lab_results(self, hl7_message):
        """Receive lab results from LIS"""
        
        results = parse_hl7_results(hl7_message)
        
        # Store in medical records
        record = MedicalRecord.query.filter_by(
            patient_id=results['patient_id']
        ).order_by(MedicalRecord.visit_date.desc()).first()
        
        record.lab_results = json.dumps(results)
        db.session.commit()
        
        # Notify doctor
        notify_doctor(record.doctor_id, results)
```

**Insurance/Billing Integration:**

```python
class BillingIntegration:
    """Integration with insurance and billing systems"""
    
    def verify_insurance(self, patient_id, insurance_info):
        """Verify patient insurance coverage"""
        
        response = insurance_api.verify_coverage(
            member_id=insurance_info['member_id'],
            provider_id=insurance_info['provider_id']
        )
        
        return {
            'is_active': response.active,
            'coverage_details': response.coverage,
            'copay_amount': response.copay
        }
    
    def submit_claim(self, appointment_id):
        """Submit insurance claim"""
        
        appointment = Appointment.query.get(appointment_id)
        medical_record = get_medical_record_for_appointment(appointment_id)
        
        claim = {
            'patient_id': appointment.patient_id,
            'provider_id': appointment.doctor_id,
            'diagnosis_codes': extract_icd_codes(medical_record),
            'procedure_codes': extract_cpt_codes(medical_record),
            'service_date': appointment.appointment_date,
            'charges': calculate_charges(medical_record)
        }
        
        # Submit via EDI 837
        claim_id = submit_edi_claim(claim)
        return claim_id
```


### 9.5 IoT Device Integration

**Smart Hospital Infrastructure:**

```python
class IoTDeviceManager:
    """Manage IoT devices in hospital"""
    
    def monitor_smart_beds(self):
        """Monitor smart bed sensors"""
        
        # Receive data from bed sensors
        bed_data = mqtt_client.subscribe('hospital/beds/+/status')
        
        for bed in bed_data:
            if bed['occupied'] and bed['patient_id']:
                # Monitor vital signs
                vitals = {
                    'heart_rate': bed['heart_rate'],
                    'blood_pressure': bed['blood_pressure'],
                    'temperature': bed['temperature'],
                    'oxygen_saturation': bed['spo2']
                }
                
                # Check for anomalies
                if detect_anomaly(vitals):
                    create_alert(bed['patient_id'], vitals)
                
                # Update patient record
                update_vital_signs(bed['patient_id'], vitals)
    
    def track_medical_equipment(self):
        """Track location of medical equipment via RFID/BLE"""
        
        equipment_locations = get_rfid_data()
        
        for equipment in equipment_locations:
            update_equipment_location(
                equipment_id=equipment['id'],
                location=equipment['location'],
                timestamp=equipment['timestamp']
            )
            
            # Alert if equipment moved to unauthorized area
            if not is_authorized_location(equipment):
                create_security_alert(equipment)
    
    def monitor_environmental_conditions(self):
        """Monitor temperature, humidity in critical areas"""
        
        sensors = ['icu', 'operation_theater', 'pharmacy', 'blood_bank']
        
        for sensor_location in sensors:
            data = get_sensor_data(sensor_location)
            
            if not within_acceptable_range(data, sensor_location):
                create_environmental_alert(sensor_location, data)
```

### 9.6 Blockchain for Medical Records

**Immutable Medical Record Ledger:**

```python
class BlockchainMedicalRecords:
    """Blockchain-based medical record management"""
    
    def create_record_hash(self, medical_record):
        """Create cryptographic hash of medical record"""
        
        record_data = {
            'patient_id': medical_record.patient_id,
            'doctor_id': medical_record.doctor_id,
            'visit_date': str(medical_record.visit_date),
            'diagnosis': medical_record.diagnosis,
            'treatment': medical_record.treatment
        }
        
        # Create hash
        record_hash = hashlib.sha256(
            json.dumps(record_data, sort_keys=True).encode()
        ).hexdigest()
        
        return record_hash
    
    def store_on_blockchain(self, record_hash, metadata):
        """Store record hash on blockchain"""
        
        transaction = {
            'record_hash': record_hash,
            'timestamp': datetime.now().isoformat(),
            'hospital_id': metadata['hospital_id'],
            'record_type': metadata['record_type']
        }
        
        # Submit to blockchain network
        tx_id = blockchain_client.submit_transaction(transaction)
        
        return tx_id
    
    def verify_record_integrity(self, medical_record, blockchain_hash):
        """Verify record hasn't been tampered with"""
        
        current_hash = self.create_record_hash(medical_record)
        
        # Retrieve hash from blockchain
        blockchain_record = blockchain_client.get_transaction(blockchain_hash)
        
        return current_hash == blockchain_record['record_hash']
```


## 10. Deployment Architecture

### 10.1 Development Environment

```
Developer Machine:
├─ Python 3.8+ with virtual environment
├─ SQLite database (local)
├─ Flask development server
├─ Git for version control
└─ IDE (VS Code, PyCharm)

Development Workflow:
1. Clone repository
2. Create virtual environment: python -m venv venv
3. Install dependencies: pip install -r requirements.txt
4. Initialize database: flask db init && flask db migrate
5. Run development server: flask run
6. Access at: http://localhost:5000
```

### 10.2 Production Deployment

**Option 1: Traditional Server Deployment**

```
Production Stack:

┌─────────────────────────────────────┐
│  Nginx (Reverse Proxy + SSL)       │
│  - Port 80/443                      │
│  - SSL termination                  │
│  - Static file serving              │
└─────────────┬───────────────────────┘
              ↓
┌─────────────────────────────────────┐
│  Gunicorn (WSGI Server)             │
│  - Multiple worker processes        │
│  - Process management               │
└─────────────┬───────────────────────┘
              ↓
┌─────────────────────────────────────┐
│  Flask Application                  │
└─────────────┬───────────────────────┘
              ↓
┌─────────────────────────────────────┐
│  PostgreSQL Database                │
└─────────────────────────────────────┘

Configuration:
# gunicorn_config.py
workers = 4
worker_class = 'sync'
bind = '127.0.0.1:8000'
timeout = 120
keepalive = 5

# nginx.conf
server {
    listen 80;
    server_name medconnect.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name medconnect.example.com;
    
    ssl_certificate /etc/ssl/certs/medconnect.crt;
    ssl_certificate_key /etc/ssl/private/medconnect.key;
    
    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    
    location /static {
        alias /var/www/medconnect/static;
    }
}
```

**Option 2: Docker Deployment**

```dockerfile
# Dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["gunicorn", "--config", "gunicorn_config.py", "app:app"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "5000:5000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/medconnect
      - FLASK_ENV=production
    depends_on:
      - db
      - redis
    volumes:
      - ./logs:/app/logs
  
  db:
    image: postgres:13
    environment:
      - POSTGRES_DB=medconnect
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
    volumes:
      - postgres_data:/var/lib/postgresql/data
  
  redis:
    image: redis:6-alpine
    ports:
      - "6379:6379"
  
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/ssl
    depends_on:
      - web

volumes:
  postgres_data:
```


**Option 3: Cloud Deployment (AWS Example)**

```
AWS Architecture:

┌─────────────────────────────────────┐
│  Route 53 (DNS)                     │
└─────────────┬───────────────────────┘
              ↓
┌─────────────────────────────────────┐
│  CloudFront (CDN)                   │
└─────────────┬───────────────────────┘
              ↓
┌─────────────────────────────────────┐
│  Application Load Balancer          │
└─────────────┬───────────────────────┘
              ↓
    ┌─────────┴─────────┐
    ↓                   ↓
┌─────────┐         ┌─────────┐
│  EC2    │         │  EC2    │
│Instance1│         │Instance2│
└─────────┘         └─────────┘
              ↓
┌─────────────────────────────────────┐
│  RDS PostgreSQL (Multi-AZ)          │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│  ElastiCache Redis                  │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│  S3 (Static files, backups)         │
└─────────────────────────────────────┘
```

### 10.3 Monitoring and Logging

**Application Monitoring:**

```python
# Logging configuration
import logging
from logging.handlers import RotatingFileHandler

# Configure logging
handler = RotatingFileHandler(
    'logs/medconnect.log',
    maxBytes=10485760,  # 10MB
    backupCount=10
)

formatter = logging.Formatter(
    '%(asctime)s %(levelname)s: %(message)s [in %(pathname)s:%(lineno)d]'
)
handler.setFormatter(formatter)

app.logger.addHandler(handler)
app.logger.setLevel(logging.INFO)

# Usage
app.logger.info('User login successful', extra={'user_id': user.id})
app.logger.error('Database connection failed', exc_info=True)
```

**Performance Monitoring:**

```python
from flask import request
import time

@app.before_request
def before_request():
    request.start_time = time.time()

@app.after_request
def after_request(response):
    if hasattr(request, 'start_time'):
        elapsed = time.time() - request.start_time
        
        # Log slow requests
        if elapsed > 1.0:
            app.logger.warning(
                f'Slow request: {request.path} took {elapsed:.2f}s'
            )
        
        # Add response time header
        response.headers['X-Response-Time'] = f'{elapsed:.3f}s'
    
    return response
```

**Health Check Endpoint:**

```python
@app.route('/health')
def health_check():
    """System health check endpoint"""
    
    health_status = {
        'status': 'healthy',
        'timestamp': datetime.now().isoformat(),
        'checks': {}
    }
    
    # Database check
    try:
        db.session.execute('SELECT 1')
        health_status['checks']['database'] = 'ok'
    except Exception as e:
        health_status['checks']['database'] = 'error'
        health_status['status'] = 'unhealthy'
    
    # Redis check (if using)
    try:
        cache.get('health_check')
        health_status['checks']['cache'] = 'ok'
    except:
        health_status['checks']['cache'] = 'error'
    
    # Disk space check
    disk_usage = get_disk_usage()
    if disk_usage > 90:
        health_status['checks']['disk'] = 'warning'
    else:
        health_status['checks']['disk'] = 'ok'
    
    status_code = 200 if health_status['status'] == 'healthy' else 503
    return jsonify(health_status), status_code
```


## 11. Testing Strategy

### 11.1 Unit Testing

```python
import unittest
from app import create_app, db
from app.models import User, Patient, Appointment

class TestPatientModule(unittest.TestCase):
    
    def setUp(self):
        """Set up test environment"""
        self.app = create_app('testing')
        self.client = self.app.test_client()
        self.app_context = self.app.app_context()
        self.app_context.push()
        db.create_all()
    
    def tearDown(self):
        """Clean up after tests"""
        db.session.remove()
        db.drop_all()
        self.app_context.pop()
    
    def test_patient_registration(self):
        """Test patient registration"""
        response = self.client.post('/register', data={
            'username': 'testpatient',
            'email': 'patient@test.com',
            'password': 'Test123!',
            'role': 'patient'
        })
        self.assertEqual(response.status_code, 201)
        
        user = User.query.filter_by(username='testpatient').first()
        self.assertIsNotNone(user)
        self.assertEqual(user.role, 'patient')
    
    def test_appointment_booking(self):
        """Test appointment booking"""
        # Create test patient and doctor
        patient = create_test_patient()
        doctor = create_test_doctor()
        
        response = self.client.post('/appointments/book', data={
            'patient_id': patient.id,
            'doctor_id': doctor.id,
            'date': '2026-03-01',
            'time_slot': '10:00'
        })
        
        self.assertEqual(response.status_code, 201)
        appointment = Appointment.query.first()
        self.assertIsNotNone(appointment)
```

### 11.2 Integration Testing

```python
class TestEmergencyAlertSystem(unittest.TestCase):
    
    def test_ambulance_alert_workflow(self):
        """Test complete ambulance alert workflow"""
        
        # Step 1: Create alert
        alert_data = {
            'severity': 'critical',
            'patient_info': 'Male, 45, chest pain',
            'required_resources': {'beds': 1, 'ventilator': 1},
            'location': 'Downtown'
        }
        
        response = self.client.post('/emergency/ambulance-alert', 
                                   json=alert_data)
        self.assertEqual(response.status_code, 201)
        alert_id = response.json['alert_id']
        
        # Step 2: Verify hospitals received alert
        hospitals = response.json['available_hospitals']
        self.assertGreater(len(hospitals), 0)
        
        # Step 3: Select hospital and reserve resources
        hospital_id = hospitals[0]['hospital_id']
        response = self.client.post(f'/emergency/select-hospital', json={
            'alert_id': alert_id,
            'hospital_id': hospital_id
        })
        
        self.assertEqual(response.status_code, 200)
        
        # Step 4: Verify resources were reserved
        resource = Resource.query.filter_by(
            resource_type='beds',
            hospital_id=hospital_id
        ).first()
        self.assertEqual(resource.reserved_count, 1)
```

### 11.3 Performance Testing

```python
from locust import HttpUser, task, between

class MedConnectUser(HttpUser):
    wait_time = between(1, 3)
    
    def on_start(self):
        """Login before starting tasks"""
        self.client.post('/login', {
            'username': 'testuser',
            'password': 'password'
        })
    
    @task(3)
    def view_dashboard(self):
        """View dashboard - most common action"""
        self.client.get('/dashboard')
    
    @task(2)
    def view_appointments(self):
        """View appointments"""
        self.client.get('/appointments')
    
    @task(1)
    def book_appointment(self):
        """Book appointment - less frequent"""
        self.client.post('/appointments/book', {
            'doctor_id': 1,
            'date': '2026-03-15',
            'time_slot': '14:00'
        })

# Run: locust -f performance_test.py --host=http://localhost:5000
```


## 12. Project Structure

```
medconnect/
├── app/
│   ├── __init__.py              # Application factory
│   ├── models.py                # Database models
│   ├── config.py                # Configuration settings
│   │
│   ├── auth/                    # Authentication module
│   │   ├── __init__.py
│   │   ├── routes.py
│   │   ├── forms.py
│   │   └── utils.py
│   │
│   ├── patient/                 # Patient module
│   │   ├── __init__.py
│   │   ├── routes.py
│   │   ├── forms.py
│   │   └── utils.py
│   │
│   ├── doctor/                  # Doctor module
│   │   ├── __init__.py
│   │   ├── routes.py
│   │   ├── forms.py
│   │   └── utils.py
│   │
│   ├── pharmacist/              # Pharmacist module
│   │   ├── __init__.py
│   │   ├── routes.py
│   │   ├── forms.py
│   │   └── utils.py
│   │
│   ├── admin/                   # Admin module
│   │   ├── __init__.py
│   │   ├── routes.py
│   │   ├── forms.py
│   │   └── utils.py
│   │
│   ├── emergency/               # Emergency module
│   │   ├── __init__.py
│   │   ├── routes.py
│   │   ├── forms.py
│   │   └── utils.py
│   │
│   ├── templates/               # HTML templates
│   │   ├── base.html
│   │   ├── auth/
│   │   ├── patient/
│   │   ├── doctor/
│   │   ├── pharmacist/
│   │   ├── admin/
│   │   └── emergency/
│   │
│   └── static/                  # Static files
│       ├── css/
│       ├── js/
│       └── images/
│
├── migrations/                  # Database migrations
├── tests/                       # Test files
│   ├── test_auth.py
│   ├── test_patient.py
│   ├── test_doctor.py
│   ├── test_pharmacist.py
│   ├── test_admin.py
│   └── test_emergency.py
│
├── logs/                        # Application logs
├── instance/                    # Instance-specific files
│   └── medconnect.db           # SQLite database
│
├── requirements.txt             # Python dependencies
├── config.py                    # Configuration
├── run.py                       # Application entry point
├── gunicorn_config.py          # Gunicorn configuration
├── docker-compose.yml          # Docker configuration
├── Dockerfile                  # Docker image
├── .env                        # Environment variables
├── .gitignore                  # Git ignore file
└── README.md                   # Project documentation
```

## 13. API Design

### 13.1 RESTful API Endpoints

**Authentication:**
```
POST   /api/auth/register       - Register new user
POST   /api/auth/login          - User login
POST   /api/auth/logout         - User logout
POST   /api/auth/reset-password - Password reset
```

**Patient:**
```
GET    /api/patients            - List all patients (admin/doctor)
GET    /api/patients/:id        - Get patient details
PUT    /api/patients/:id        - Update patient profile
GET    /api/patients/:id/records - Get medical records
GET    /api/patients/:id/appointments - Get appointments
```

**Doctor:**
```
GET    /api/doctors             - List all doctors
GET    /api/doctors/:id         - Get doctor details
PUT    /api/doctors/:id/availability - Update availability
GET    /api/doctors/:id/appointments - Get appointments
```

**Appointments:**
```
GET    /api/appointments        - List appointments
POST   /api/appointments        - Book appointment
GET    /api/appointments/:id    - Get appointment details
PUT    /api/appointments/:id    - Update appointment
DELETE /api/appointments/:id    - Cancel appointment
```

**Medical Records:**
```
GET    /api/records             - List medical records
POST   /api/records             - Create medical record
GET    /api/records/:id         - Get record details
PUT    /api/records/:id         - Update medical record
```

**Medicines:**
```
GET    /api/medicines           - List medicines
POST   /api/medicines           - Add medicine
GET    /api/medicines/:id       - Get medicine details
PUT    /api/medicines/:id       - Update medicine
DELETE /api/medicines/:id       - Remove medicine
GET    /api/medicines/expiring  - Get expiring medicines
```

**Resources:**
```
GET    /api/resources           - Get all resources
GET    /api/resources/:type     - Get specific resource type
PUT    /api/resources/:id       - Update resource
GET    /api/resources/availability - Real-time availability
```

**Emergency:**
```
POST   /api/emergency/ambulance-alert - Create ambulance alert
POST   /api/emergency/internal-alert  - Create internal alert
GET    /api/emergency/alerts          - List active alerts
PUT    /api/emergency/alerts/:id      - Update alert status
POST   /api/emergency/respond         - Respond to alert
```

**Admin:**
```
GET    /api/admin/users         - List all users
POST   /api/admin/users         - Create user
PUT    /api/admin/users/:id     - Update user
DELETE /api/admin/users/:id     - Deactivate user
POST   /api/admin/pandemic-mode - Toggle pandemic mode
GET    /api/admin/reports       - Generate reports
GET    /api/admin/audit-logs    - View audit logs
```


### 13.2 API Response Format

**Success Response:**
```json
{
  "status": "success",
  "data": {
    "id": 123,
    "name": "John Doe",
    "email": "john@example.com"
  },
  "message": "Operation completed successfully",
  "timestamp": "2026-02-15T10:30:00Z"
}
```

**Error Response:**
```json
{
  "status": "error",
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": {
      "email": "Invalid email format",
      "password": "Password too short"
    }
  },
  "timestamp": "2026-02-15T10:30:00Z"
}
```

**Pagination Response:**
```json
{
  "status": "success",
  "data": [...],
  "pagination": {
    "page": 1,
    "per_page": 20,
    "total_pages": 5,
    "total_items": 95
  }
}
```

## 14. Security Checklist

### 14.1 Implementation Checklist

- [ ] Password hashing with bcrypt/pbkdf2
- [ ] HTTPS/TLS encryption in production
- [ ] CSRF protection on all forms
- [ ] SQL injection prevention (parameterized queries)
- [ ] XSS protection (input sanitization, output escaping)
- [ ] Session timeout (30 minutes)
- [ ] Secure cookie flags (HttpOnly, Secure, SameSite)
- [ ] Rate limiting on authentication endpoints
- [ ] Input validation on all user inputs
- [ ] Role-based access control enforcement
- [ ] Audit logging for sensitive operations
- [ ] Regular security updates and patches
- [ ] Database backup encryption
- [ ] API authentication tokens
- [ ] File upload validation and sanitization
- [ ] Error messages don't leak sensitive information
- [ ] Security headers (CSP, X-Frame-Options, etc.)

### 14.2 Compliance Requirements

**HIPAA Considerations:**
- [ ] Patient data encryption at rest and in transit
- [ ] Access controls and authentication
- [ ] Audit trails for all PHI access
- [ ] Data backup and disaster recovery
- [ ] Business associate agreements
- [ ] Employee training on data privacy
- [ ] Incident response plan
- [ ] Regular security risk assessments

## 15. Conclusion

This system design document provides a comprehensive blueprint for implementing MedConnect, a healthcare management and emergency coordination system. The design emphasizes:

- **Modularity:** Clear separation of concerns with distinct modules for each user role
- **Security:** Multiple layers of security including authentication, authorization, encryption, and audit logging
- **Scalability:** Architecture designed to scale from SQLite to enterprise-grade databases and distributed systems
- **Emergency Response:** Robust emergency alert systems for both ambulance coordination and internal hospital emergencies
- **Resource Management:** Real-time tracking and allocation of critical healthcare resources
- **Future-Ready:** Extensible design supporting integration with external systems, IoT devices, AI/ML, and mobile applications

The system follows industry best practices for healthcare software development while maintaining flexibility for future enhancements and regulatory compliance requirements.

---

**Document Version:** 1.0  
**Last Updated:** February 15, 2026  
**Next Review Date:** May 15, 2026
