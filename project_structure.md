# B2B Marketplace Project Structure

This document outlines the complete directory structure and key files for the B2B marketplace implementation.

## Root Directory Structure

```
b2b-marketplace/
├── app/                          # Main application package
├── migrations/                    # Database migration files
├── tests/                        # Test suite
├── config/                       # Configuration files
├── static/                       # Static assets (CSS, JS, images)
├── templates/                    # Jinja2 templates
├── requirements/                 # Python dependencies
├── docker/                       # Docker configuration
├── scripts/                      # Utility scripts
├── docs/                         # Documentation
├── .env.example                  # Environment variables template
├── .gitignore                    # Git ignore file
├── Dockerfile                    # Docker image definition
├── docker-compose.yml            # Docker compose configuration
├── gunicorn.conf.py              # Gunicorn configuration
├── run.py                        # Application entry point
└── README.md                     # Project documentation
```

## Application Package Structure (app/)

```
app/
├── __init__.py                   # Flask application factory
├── models/                       # Database models
│   ├── __init__.py
│   ├── user.py                   # User model
│   ├── company.py                # CompanyProfile model
│   ├── category.py               # Category model
│   ├── product.py                # Product and ProductImage models
│   ├── inquiry.py                # Inquiry and InquiryResponse models
│   └── saved.py                  # RecentlyViewed and SavedProduct models
├── routes/                       # Application routes
│   ├── __init__.py
│   ├── auth.py                   # Authentication routes
│   ├── main.py                   # Main application routes
│   ├── dashboard.py              # User dashboard routes
│   ├── admin.py                  # Admin panel routes
│   └── api.py                    # API routes for HTMX interactions
├── forms/                        # WTForms classes
│   ├── __init__.py
│   ├── auth.py                   # Authentication forms
│   ├── product.py                # Product forms
│   ├── inquiry.py                # Inquiry forms
│   ├── profile.py                # Profile and company forms
│   └── admin.py                  # Admin forms
├── services/                     # Business logic services
│   ├── __init__.py
│   ├── auth_service.py           # Authentication service
│   ├── product_service.py        # Product management service
│   ├── inquiry_service.py        # Inquiry handling service
│   ├── email_service.py          # Email notification service
│   ├── image_service.py          # Image upload service
│   └── search_service.py         # Search functionality service
├── utils/                        # Utility functions
│   ├── __init__.py
│   ├── decorators.py             # Custom decorators
│   ├── helpers.py                # Helper functions
│   ├── validators.py             # Custom validators
│   └── filters.py                # Jinja2 custom filters
├── static/                       # Static assets
│   ├── css/
│   │   ├── main.css              # Main stylesheet
│   │   ├── components.css        # Component-specific styles
│   │   └── admin.css             # Admin panel styles
│   ├── js/
│   │   ├── main.js               # Main JavaScript file
│   │   ├── htmx.js               # HTMX configuration
│   │   ├── alpine.js             # Alpine.js components
│   │   └── admin.js              # Admin panel JavaScript
│   └── images/
│       ├── logo.png
│       ├── placeholder.jpg
│       └── icons/
└── templates/                    # Jinja2 templates
    ├── base.html                 # Base template with common elements
    ├── auth/                     # Authentication templates
    │   ├── login.html
    │   ├── register_buyer.html
    │   ├── register_supplier.html
    │   ├── forgot_password.html
    │   └── reset_password.html
    ├── main/                     # Main application templates
    │   ├── homepage.html
    │   ├── search_results.html
    │   ├── product_detail.html
    │   ├── category_page.html
    │   └── supplier_profile.html
    ├── dashboard/                # Dashboard templates
    │   ├── dashboard_base.html
    │   ├── profile.html
    │   ├── company_profile.html
    │   ├── products/
    │   │   ├── list.html
    │   │   ├── form.html
    │   │   └── delete_confirm.html
    │   ├── inquiries/
    │   │   ├── sent.html
    │   │   ├── received.html
    │   │   └── detail.html
    │   └── saved_products.html
    ├── components/               # Reusable components
    │   ├── navbar.html
    │   ├── footer.html
    │   ├── search_bar.html
    │   ├── product_card.html
    │   ├── inquiry_modal.html
    │   ├── pagination.html
    │   ├── filters.html
    │   └── messages.html
    └── admin/                    # Admin panel templates
        ├── dashboard.html
        ├── users.html
        ├── suppliers.html
        ├── categories.html
        ├── products.html
        └── inquiries.html
```

## Configuration Structure (config/)

```
config/
├── __init__.py
├── base.py                       # Base configuration
├── development.py                # Development environment config
├── testing.py                    # Testing environment config
├── production.py                 # Production environment config
└── deployment.py                 # Deployment-specific config
```

## Test Structure (tests/)

```
tests/
├── __init__.py
├── conftest.py                   # Pytest configuration and fixtures
├── test_auth.py                  # Authentication tests
├── test_products.py              # Product functionality tests
├── test_inquiries.py             # Inquiry system tests
├── test_search.py                # Search functionality tests
├── test_admin.py                 # Admin panel tests
├── test_services/                # Service layer tests
│   ├── __init__.py
│   ├── test_auth_service.py
│   ├── test_product_service.py
│   └── test_email_service.py
└── test_utils/                   # Utility function tests
    ├── __init__.py
    ├── test_helpers.py
    └── test_validators.py
```

## Docker Configuration (docker/)

```
docker/
├── app/
│   └── Dockerfile                # Application Dockerfile
├── nginx/
│   ├── Dockerfile                # Nginx Dockerfile
│   └── nginx.conf                # Nginx configuration
└── postgres/
    └── init.sql                  # PostgreSQL initialization script
```

## Scripts Directory (scripts/)

```
scripts/
├── init_db.py                    # Database initialization script
├── create_admin.py               # Create admin user script
├── seed_data.py                  # Seed initial data
├── backup_db.py                  # Database backup script
└── deploy.sh                     # Deployment script
```

## Documentation (docs/)

```
docs/
├── api.md                        # API documentation
├── deployment.md                 # Deployment guide
├── development.md                # Development setup guide
├── user_guide.md                 # User documentation
└── admin_guide.md                # Admin documentation
```

## Requirements Directory (requirements/)

```
requirements/
├── base.txt                      # Base dependencies
├── development.txt               # Development dependencies
├── production.txt                # Production dependencies
└── testing.txt                   # Testing dependencies
```

## Key Files Content Overview

### run.py
```python
from app import create_app
import os

app = create_app(os.getenv('FLASK_ENV') or 'development')

if __name__ == '__main__':
    app.run()
```

### .env.example
```
FLASK_APP=run.py
FLASK_ENV=development
SECRET_KEY=your-secret-key-here
DATABASE_URL=sqlite:///b2b_marketplace.db
MAIL_SERVER=smtp.gmail.com
MAIL_PORT=587
MAIL_USE_TLS=True
MAIL_USERNAME=your-email@gmail.com
MAIL_PASSWORD=your-app-password
CLOUDINARY_CLOUD_NAME=your-cloud-name
CLOUDINARY_API_KEY=your-api-key
CLOUDINARY_API_SECRET=your-api-secret
REDIS_URL=redis://localhost:6379/0
```

### .gitignore
```
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
env/
venv/
ENV/
env.bak/
venv.bak/
instance/
.webassets-cache
.coverage
htmlcov/
.tox/
.nox/
.coverage
.coverage.*
.cache
nosetests.xml
coverage.xml
*.cover
*.py,cover
.hypothesis/
.pytest_cache/
*.log
.env
.DS_Store
uploads/
migrations/versions/
```

### Dockerfile
```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements/production.txt .
RUN pip install -r production.txt

COPY . .

EXPOSE 5000

CMD ["gunicorn", "--config", "gunicorn.conf.py", "run:app"]
```

### docker-compose.yml
```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: docker/app/Dockerfile
    ports:
      - "5000:5000"
    environment:
      - FLASK_ENV=production
      - DATABASE_URL=postgresql://user:password@db:5432/b2b_marketplace
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      - db
      - redis
    volumes:
      - ./uploads:/app/uploads

  db:
    image: postgres:13
    environment:
      - POSTGRES_DB=b2b_marketplace
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:6-alpine
    ports:
      - "6379:6379"

  nginx:
    build:
      context: .
      dockerfile: docker/nginx/Dockerfile
    ports:
      - "80:80"
    depends_on:
      - app
    volumes:
      - ./static:/app/static

volumes:
  postgres_data:
```

### gunicorn.conf.py
```python
bind = "0.0.0.0:5000"
workers = 4
worker_class = "sync"
worker_connections = 1000
timeout = 30
keepalive = 2
max_requests = 1000
max_requests_jitter = 100
preload_app = True
```

## Implementation Order

1. **First Week**: Create basic project structure, set up Flask application factory, implement basic models and authentication
2. **Second Week**: Build main templates, implement product management, create basic search functionality
3. **Third Week**: Add HTMX interactions, implement inquiry system, create user dashboards
4. **Fourth Week**: Build admin panel, add premium features, implement mobile number masking
5. **Fifth Week**: Optimize performance, add SEO features, prepare for deployment

This structure provides a solid foundation for building a scalable B2B marketplace while maintaining clean separation of concerns and following Flask best practices.