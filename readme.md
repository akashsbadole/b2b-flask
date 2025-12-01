Here’s a **clear, step-by-step instruction guide** (no code) to build a B2B marketplace like IndiaMART or TradeIndia using **Flask + HTMX + Alpine.js + Tailwind CSS**.

### Phase 1: Project Setup
1. Create a new project folder and virtual environment.
2. Install required packages:
   - Flask, Flask-Login, Flask-SQLAlchemy, Flask-Migrate, Flask-WTF
   - email-validator, python-dotenv
3. Set up Flask application factory pattern in `app/__init__.py`.
4. Configure SQLite for development, PostgreSQL for production.
5. Initialize Flask-Migrate and create first migration.

### Phase 2: Database Models
Create these core models:
- **User** (with role: buyer/supplier, company name, mobile, GST, etc.)
- **Category** (hierarchical if needed later)
- **Product** (title, description, price range, images, MOQ, category, supplier)
- **Inquiry/Lead** (buyer details, message, product reference, timestamp)
- **CompanyProfile** (linked to User – address, year established, verification status)

### Phase 3: Authentication
1. Implement registration (separate forms for buyers & suppliers).
2. Implement login/logout.
3. Use Flask-Login for session management.
4. Protect supplier-only routes with `@login_required` and role check.

### Phase 4: Core Pages & Features (Server-Rendered First)
Build these pages using Jinja templates:
1. **Homepage**
   - Big search bar
   - Popular categories grid
   - Featured/premium suppliers section
2. **Search Results Page**
   - Real-time search using HTMX (trigger on keyup with 500ms delay)
   - Filter by category, location, price range (all via GET params + HTMX)
   - Infinite scroll (load more when last card is visible)
3. **Product Detail Page**
   - Product images, description, specifications
   - Supplier info (company name, location, verified badge)
   - "Send Inquiry" button → opens modal via HTMX
4. **Supplier Dashboard**
   - Add/edit products
   - View received inquiries
   - Update company profile
   - Upload multiple product images

### Phase 5: Make It Dynamic with HTMX + Alpine.js
Use HTMX for:
- Live search without page reload
- Load more products on scroll
- Open "Send Inquiry" modal without JS
- Form submissions with instant feedback (success message inside modal)
- Star/like/save product (with login check)
- Filter sidebar (update results instantly)

Use Alpine.js for:
- Mobile menu toggle
- Modal open/close
- Tabs (e.g., product description/specifications)
- Dropdowns, accordions
- Form validation feedback (lightweight)

### Phase 6: Must-Have B2B Features
Implement these IndiaMART-style features:
1. **Send Inquiry / Get Quotes**
   - Floating or inline inquiry button
   - Pre-filled product name in form
   - After submission → show success + supplier gets email/SMS
2. **Verified Supplier Badge**
   - Admin approves → show shield icon
3. **Premium Listings**
   - Paid suppliers appear at top with golden border
4. **Mobile Number Masking**
   - Show only last 4 digits until inquiry is sent
5. **Call Now Button** (mobile only)
6. **Related Products** section
7. **Recently Viewed Products**

### Phase 7: SEO & Performance
- All pages server-rendered → great for Google
- Use proper meta titles/descriptions (dynamic per product/category)
- Add sitemap.xml and robots.txt
- Use lazy loading for images
- Add structured data (JSON-LD) for products

### Phase 8: Admin Panel (Simple)
- Use Flask-Admin or build minimal custom panel
- Approve suppliers
- Manage categories
- View all inquiries

### Phase 9: Deployment-Ready Steps
1. Switch to PostgreSQL + Redis (for background jobs)
2. Add Flask-Mail or Celery + SMTP for inquiry emails
3. Use Cloudinary or S3 for image uploads
4. Add rate limiting (Flask-Limiter)
5. Deploy with Gunicorn + Nginx + Docker (or Render/ Railway/ Fly.io)

### Phase 10: Future Upgrades (Monetization)
- Membership plans (Free, Silver, Gold)
- Featured listing packages
- Banner ads
- RFQ (Request for Quotation) posting system
- Mobile app (later with TWA or Flutter)

### Why This Stack Wins for B2B Marketplace
- Extremely fast development
- Excellent SEO (server-rendered HTML)
- Low hosting cost
- No heavy frontend framework
- Great mobile experience with minimal JS
- Easy to maintain and scale

Follow these instructions in order, and in **2–4 weeks** (solo) or **1–2 weeks** (small team), you’ll have a fully working, professional-looking B2B platform ready for real suppliers and buyers.

Let me know which part you want detailed next (e.g., “explain inquiry system flow” or “how to do infinite scroll with HTMX”) — I’ll give you exact steps without code.