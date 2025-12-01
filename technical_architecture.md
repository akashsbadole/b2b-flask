# B2B Marketplace Technical Architecture Plan

## Overview
This document outlines the technical architecture for a B2B marketplace platform similar to IndiaMART or TradeIndia, built using Flask + HTMX + Alpine.js + Tailwind CSS.

## Technology Stack

### Backend
- **Framework**: Flask (Python)
- **Database**: SQLite (development), PostgreSQL (production)
- **ORM**: SQLAlchemy
- **Authentication**: Flask-Login
- **Form Handling**: Flask-WTF
- **Database Migrations**: Flask-Migrate
- **Email**: Flask-Mail
- **Rate Limiting**: Flask-Limiter
- **Background Jobs**: Celery with Redis
- **File Storage**: Cloudinary or AWS S3

### Frontend
- **Templating**: Jinja2
- **Styling**: Tailwind CSS
- **Interactivity**: HTMX + Alpine.js
- **Icons**: Heroicons or Font Awesome

### Deployment
- **WSGI Server**: Gunicorn
- **Web Server**: Nginx
- **Containerization**: Docker
- **Hosting**: Render/Railway/Fly.io or VPS

## Database Schema Design

### User Model
```python
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    email = db.Column(db.String(120), unique=True, nullable=False)
    password_hash = db.Column(db.String(128), nullable=False)
    role = db.Column(db.Enum('buyer', 'supplier', 'admin'), default='buyer', nullable=False)
    first_name = db.Column(db.String(50), nullable=False)
    last_name = db.Column(db.String(50), nullable=False)
    mobile = db.Column(db.String(20), nullable=False)
    company_name = db.Column(db.String(100))
    gst_number = db.Column(db.String(20))  # GST identification
    is_verified = db.Column(db.Boolean, default=False)
    is_premium = db.Column(db.Boolean, default=False)
    premium_expiry = db.Column(db.DateTime)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    last_login = db.Column(db.DateTime)
    
    # Relationships
    company_profile = db.relationship('CompanyProfile', backref='user', uselist=False)
    products = db.relationship('Product', backref='supplier', lazy='dynamic')
    inquiries_sent = db.relationship('Inquiry', foreign_keys='Inquiry.buyer_id', backref='buyer', lazy='dynamic')
    inquiries_received = db.relationship('Inquiry', foreign_keys='Inquiry.supplier_id', backref='supplier', lazy='dynamic')
```

### CompanyProfile Model
```python
class CompanyProfile(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
    company_name = db.Column(db.String(100), nullable=False)
    description = db.Column(db.Text)
    address = db.Column(db.String(255))
    city = db.Column(db.String(50))
    state = db.Column(db.String(50))
    postal_code = db.Column(db.String(20))
    country = db.Column(db.String(50), default='India')
    website = db.Column(db.String(100))
    year_established = db.Column(db.Integer)
    employee_count = db.Column(db.String(50))
    annual_turnover = db.Column(db.String(50))
    certifications = db.Column(db.Text)  # JSON string of certifications
    verification_status = db.Column(db.Enum('pending', 'verified', 'rejected'), default='pending')
    verification_documents = db.Column(db.Text)  # JSON string of document paths
    logo_url = db.Column(db.String(255))
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    updated_at = db.Column(db.DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
```

### Category Model
```python
class Category(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    slug = db.Column(db.String(100), unique=True, nullable=False)
    description = db.Column(db.Text)
    icon = db.Column(db.String(100))
    parent_id = db.Column(db.Integer, db.ForeignKey('category.id'))
    sort_order = db.Column(db.Integer, default=0)
    is_active = db.Column(db.Boolean, default=True)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    
    # Self-referential relationship for hierarchy
    parent = db.relationship('Category', remote_side=[id], backref='children')
    products = db.relationship('Product', backref='category', lazy='dynamic')
```

### Product Model
```python
class Product(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(200), nullable=False)
    slug = db.Column(db.String(200), unique=True, nullable=False)
    description = db.Column(db.Text, nullable=False)
    specifications = db.Column(db.Text)  # JSON string of specifications
    min_order_quantity = db.Column(db.Integer, default=1)
    price_min = db.Column(db.Float)
    price_max = db.Column(db.Float)
    price_unit = db.Column(db.String(20))  # per piece, kg, etc.
    supplier_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
    category_id = db.Column(db.Integer, db.ForeignKey('category.id'), nullable=False)
    is_featured = db.Column(db.Boolean, default=False)
    is_active = db.Column(db.Boolean, default=True)
    view_count = db.Column(db.Integer, default=0)
    inquiry_count = db.Column(db.Integer, default=0)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    updated_at = db.Column(db.DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
    
    # Relationships
    images = db.relationship('ProductImage', backref='product', lazy='dynamic', cascade='all, delete-orphan')
```

### ProductImage Model
```python
class ProductImage(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    product_id = db.Column(db.Integer, db.ForeignKey('product.id'), nullable=False)
    image_url = db.Column(db.String(255), nullable=False)
    alt_text = db.Column(db.String(200))
    sort_order = db.Column(db.Integer, default=0)
    is_primary = db.Column(db.Boolean, default=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
```

### Inquiry Model
```python
class Inquiry(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    buyer_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
    supplier_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
    product_id = db.Column(db.Integer, db.ForeignKey('product.id'))
    subject = db.Column(db.String(200), nullable=False)
    message = db.Column(db.Text, nullable=False)
    quantity = db.Column(db.Integer)
    contact_preference = db.Column(db.Enum('email', 'phone', 'both'), default='email')
    status = db.Column(db.Enum('new', 'responded', 'closed'), default='new')
    is_phone_revealed = db.Column(db.Boolean, default=False)  # For mobile number masking
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    updated_at = db.Column(db.DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
    
    # Relationships
    product = db.relationship('Product', backref='inquiries')
    responses = db.relationship('InquiryResponse', backref='inquiry', lazy='dynamic', cascade='all, delete-orphan')
```

### InquiryResponse Model
```python
class InquiryResponse(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    inquiry_id = db.Column(db.Integer, db.ForeignKey('inquiry.id'), nullable=False)
    sender_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
    message = db.Column(db.Text, nullable=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    
    # Relationships
    sender = db.relationship('User', foreign_keys=[sender_id])
```

### RecentlyViewed Model
```python
class RecentlyViewed(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
    product_id = db.Column(db.Integer, db.ForeignKey('product.id'), nullable=False)
    viewed_at = db.Column(db.DateTime, default=datetime.utcnow)
    
    # Relationships
    user = db.relationship('User', backref='recently_viewed')
    product = db.relationship('Product')
```

### SavedProduct Model
```python
class SavedProduct(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
    product_id = db.Column(db.Integer, db.ForeignKey('product.id'), nullable=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    
    # Relationships
    user = db.relationship('User', backref='saved_products')
    product = db.relationship('Product')
    
    # Ensure a user can't save the same product twice
    __table_args__ = (db.UniqueConstraint('user_id', 'product_id'),)
```

## API Endpoints and Routes Structure

### Authentication Routes
```
/auth/register/buyer - GET, POST (Buyer registration)
/auth/register/supplier - GET, POST (Supplier registration)
/auth/login - GET, POST
/auth/logout - POST
/auth/forgot-password - GET, POST
/auth/reset-password/<token> - GET, POST
```

### Main Application Routes
```
/ - GET (Homepage)
/search - GET (Search results page)
/products/<slug> - GET (Product detail page)
/category/<slug> - GET (Category page)
/suppliers/<username> - GET (Supplier profile page)
/inquiry/send/<product_id> - POST (Send inquiry via HTMX)
/inquiry/<id> - GET, POST (Inquiry detail and response)
```

### User Dashboard Routes
```
/dashboard - GET (User dashboard)
/dashboard/profile - GET, POST (Edit profile)
/dashboard/company - GET, POST (Company profile)
/dashboard/products - GET (List products)
/dashboard/products/new - GET, POST (Add product)
/dashboard/products/edit/<id> - GET, POST (Edit product)
/dashboard/products/delete/<id> - POST (Delete product)
/dashboard/inquiries/sent - GET (Sent inquiries)
/dashboard/inquiries/received - GET (Received inquiries)
/dashboard/inquiries/<id> - GET, POST (Inquiry detail and response)
/dashboard/saved-products - GET (Saved products)
```

### Admin Routes
```
/admin - GET (Admin dashboard)
/admin/users - GET, POST (User management)
/admin/suppliers/pending - GET, POST (Pending supplier verification)
/admin/categories - GET, POST (Category management)
/admin/products - GET (Product management)
/admin/inquiries - GET (Inquiry management)
```

### API Routes (for HTMX interactions)
```
/api/search/products - GET (Live search)
/api/search/suppliers - GET (Supplier search)
/api/products/load-more - GET (Infinite scroll)
/api/products/save/<id> - POST (Save/unsave product)
/api/products/view/<id> - POST (Track product view)
/api/inquiry/send - POST (Send inquiry)
/api/inquiry/respond/<id> - POST (Respond to inquiry)
```

## Frontend Component Organization

### Template Structure
```
templates/
├── base.html (Main layout with common elements)
├── auth/
│   ├── login.html
│   ├── register_buyer.html
│   └── register_supplier.html
├── main/
│   ├── homepage.html
│   ├── search_results.html
│   ├── product_detail.html
│   ├── category_page.html
│   └── supplier_profile.html
├── dashboard/
│   ├── dashboard_base.html
│   ├── profile.html
│   ├── company_profile.html
│   ├── products/
│   │   ├── list.html
│   │   ├── form.html
│   │   └── delete_confirm.html
│   └── inquiries/
│       ├── sent.html
│       ├── received.html
│       └── detail.html
├── components/
│   ├── navbar.html
│   ├── footer.html
│   ├── search_bar.html
│   ├── product_card.html
│   ├── inquiry_modal.html
│   ├── pagination.html
│   └── filters.html
└── admin/
    ├── dashboard.html
    ├── users.html
    ├── suppliers.html
    ├── categories.html
    └── inquiries.html
```

### HTMX Components
- Live search with debouncing
- Infinite scroll for product listings
- Modal dialogs for inquiries
- Dynamic filtering
- Form submissions with instant feedback
- Save/unsave products

### Alpine.js Components
- Mobile navigation menu
- Tab switching (product details)
- Accordion for specifications
- Image gallery
- Form validation feedback
- Dropdown menus

## Authentication and Authorization Flow

### Registration Process
1. User selects role (Buyer/Supplier)
2. Different registration forms based on role
3. Email verification required
4. Suppliers require additional business details
5. Admin approval for supplier verification

### Login Process
1. User enters email/password
2. Session created with Flask-Login
3. Role-based redirection
4. Remember me functionality

### Authorization Levels
1. **Guest**: Can browse products, view supplier profiles
2. **Buyer**: Can send inquiries, save products, view contact details after inquiry
3. **Supplier**: Can manage products, respond to inquiries, update company profile
4. **Verified Supplier**: All supplier features plus verification badge
5. **Premium Supplier**: All verified features plus premium listings
6. **Admin**: Full system access

### Security Measures
- Password hashing with bcrypt
- CSRF protection with Flask-WTF
- Rate limiting on sensitive endpoints
- Input validation and sanitization
- Secure session management
- SQL injection prevention with SQLAlchemy ORM

## Key Technical Decisions and Rationale

### 1. Flask + HTMX + Alpine.js Stack
**Decision**: Use server-side rendering with minimal JavaScript
**Rationale**: 
- Excellent SEO for product discovery
- Faster initial page loads
- Lower hosting costs
- Easier maintenance
- Great mobile experience without heavy frontend frameworks

### 2. SQLite for Development, PostgreSQL for Production
**Decision**: Start with SQLite, migrate to PostgreSQL
**Rationale**:
- SQLite simplifies development setup
- PostgreSQL handles production scale and concurrent users
- Easy migration with Flask-Migrate

### 3. Cloudinary/S3 for Image Storage
**Decision**: Use cloud storage instead of local file system
**Rationale**:
- Scalability for growing image collections
- CDN integration for faster delivery
- Automatic image optimization and resizing
- Backup and reliability

### 4. Celery with Redis for Background Jobs
**Decision**: Implement task queue for email notifications and other async tasks
**Rationale**:
- Better user experience (no waiting for email sending)
- Scalability for high-volume operations
- Retry mechanisms for failed tasks
- Scheduled tasks for maintenance

### 5. Role-Based Access Control
**Decision**: Implement granular permissions based on user roles
**Rationale**:
- Clear separation of buyer/supplier functionality
- Premium features for paying customers
- Admin control over platform
- Future extensibility for new roles

## Project Structure

```
b2b-marketplace/
├── app/
│   ├── __init__.py (Flask application factory)
│   ├── models/
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── product.py
│   │   ├── category.py
│   │   ├── inquiry.py
│   │   └── company.py
│   ├── routes/
│   │   ├── __init__.py
│   │   ├── auth.py
│   │   ├── main.py
│   │   ├── dashboard.py
│   │   ├── admin.py
│   │   └── api.py
│   ├── forms/
│   │   ├── __init__.py
│   │   ├── auth.py
│   │   ├── product.py
│   │   ├── inquiry.py
│   │   └── profile.py
│   ├── services/
│   │   ├── __init__.py
│   │   ├── auth_service.py
│   │   ├── product_service.py
│   │   ├── inquiry_service.py
│   │   ├── email_service.py
│   │   └── image_service.py
│   ├── utils/
│   │   ├── __init__.py
│   │   ├── decorators.py
│   │   ├── helpers.py
│   │   └── validators.py
│   ├── static/
│   │   ├── css/
│   │   ├── js/
│   │   └── images/
│   └── templates/
│       ├── base.html
│       ├── auth/
│       ├── main/
│       ├── dashboard/
│       ├── components/
│       └── admin/
├── migrations/
├── tests/
│   ├── __init__.py
│   ├── test_auth.py
│   ├── test_products.py
│   ├── test_inquiries.py
│   └── conftest.py
├── config/
│   ├── __init__.py
│   ├── development.py
│   ├── production.py
│   └── testing.py
├── requirements.txt
├── requirements-dev.txt
├── .env.example
├── .gitignore
├── Dockerfile
├── docker-compose.yml
├── gunicorn.conf.py
├── run.py
└── README.md
```

## Potential Challenges and Solutions

### 1. Mobile Number Masking
**Challenge**: Show only last 4 digits until inquiry is sent
**Solution**:
- Store full mobile number in database
- Create a template filter to mask numbers
- Reveal full number only after successful inquiry submission
- Track inquiry status to determine visibility

### 2. Supplier Verification
**Challenge**: Verify supplier authenticity to build trust
**Solution**:
- Document upload system (GST certificate, business registration)
- Admin approval workflow
- Verification badge system
- Multi-level verification (basic, premium, trusted)

### 3. Premium Listings
**Challenge**: Implement paid features without complex payment integration
**Solution**:
- Start with manual premium assignment by admin
- Later integrate payment gateway (Razorpay/PayU)
- Time-based premium subscriptions
- Different premium tiers with varying features

### 4. Multi-Image Uploads
**Challenge**: Handle multiple product images efficiently
**Solution**:
- Use JavaScript for client-side image preview
- Implement drag-and-drop interface
- Cloudinary for image storage and optimization
- Image validation (size, format, dimensions)

### 5. Search Performance
**Challenge**: Fast search across thousands of products
**Solution**:
- Database indexing on key fields
- Full-text search with PostgreSQL
- Implement caching for popular searches
- Consider Elasticsearch for advanced search later

### 6. Scalability
**Challenge**: Handle growing traffic and data
**Solution**:
- Database connection pooling
- Redis for caching and session storage
- CDN for static assets
- Load balancing with multiple app servers

### 7. Security
**Challenge**: Protect user data and prevent fraud
**Solution**:
- Input validation and sanitization
- Rate limiting on sensitive endpoints
- CSRF protection
- Regular security audits
- HTTPS enforcement

## Implementation Phases Priority

### Phase 1: Foundation (Week 1)
1. Project setup with Flask application factory
2. Database models and migrations
3. Basic authentication (register, login, logout)
4. Basic templates and styling with Tailwind
5. Homepage with search functionality

### Phase 2: Core Features (Week 2)
1. Product management (add, edit, delete)
2. Product detail pages
3. Basic search and filtering
4. Inquiry system (send and receive)
5. User dashboards

### Phase 3: Enhanced Features (Week 3)
1. HTMX integration for dynamic interactions
2. Alpine.js for UI components
3. Image upload and management
4. Supplier verification system
5. Mobile number masking

### Phase 4: Advanced Features (Week 4)
1. Premium listings
2. Advanced search with filters
3. Recently viewed and saved products
4. Admin panel
5. Email notifications

### Phase 5: Production Ready (Week 5)
1. Performance optimization
2. SEO implementation
3. Security hardening
4. Testing and bug fixes
5. Deployment preparation

This architecture provides a solid foundation for building a scalable B2B marketplace while keeping development time and complexity manageable.