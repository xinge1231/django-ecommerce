# CODEBUDDY.md

This file provides guidance to CodeBuddy Code when working with code in this repository.

## Project Overview

This is a Django e-commerce website that allows users to browse products, manage a shopping cart, and process payments via Stripe. The project uses Django 2.2 with class-based views and django-allauth for authentication.

## Development Commands

### Setup
```bash
# Create virtual environment
virtualenv env

# Activate virtual environment (Windows)
env\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### Running the Server
```bash
# Run development server
python manage.py runserver
```

### Database
```bash
# Create migrations
python manage.py makemigrations

# Apply migrations
python manage.py migrate
```

### Testing
```bash
# Run all tests
python manage.py test

# Run tests for specific app
python manage.py test core
```

### Static Files
```bash
# Collect static files for production
python manage.py collectstatic
```

## Environment Configuration

The `.env` file contains all configuration. Required keys:
- `SECRET_KEY` - Django secret key
- `STRIPE_TEST_PUBLIC_KEY` / `STRIPE_TEST_SECRET_KEY` - Stripe test keys for development
- `STRIPE_LIVE_PUBLIC_KEY` / `STRIPE_LIVE_SECRET_KEY` - Stripe live keys for production
- `DB_NAME`, `DB_USER`, `DB_PASSWORD`, `DB_HOST` - PostgreSQL config for production

## Architecture

### Settings Structure
Settings are split into three files under `djecommerce/settings/`:
- `base.py` - Common settings (installed apps, middleware, templates)
- `development.py` - SQLite database, debug toolbar, Stripe test keys
- `production.py` - PostgreSQL, Stripe live keys, password validators

The active settings module is determined by `DJANGO_SETTINGS_MODULE` environment variable.

### Main App: `core`
All e-commerce logic resides in the `core` app:
- **Models** (`models.py`): `Item`, `Order`, `OrderItem`, `Address`, `Payment`, `Coupon`, `Refund`, `UserProfile`
- **Views** (`views.py`): Class-based views (`HomeView`, `CheckoutView`, `PaymentView`, `OrderSummaryView`, `ItemDetailView`) and function views for cart operations
- **Forms** (`forms.py`): `CheckoutForm`, `CouponForm`, `RefundForm`, `PaymentForm`
- **URLs** (`urls.py`): All routes use the `core:` namespace

### Order Flow
1. User adds items to cart (`add_to_cart`)
2. Cart displayed via `OrderSummaryView`
3. Checkout collects shipping/billing addresses
4. Payment processed via Stripe in `PaymentView`
5. Order marked as `ordered=True` with a reference code

### URL Namespacing
All core URLs use the `core:` namespace. Use `reverse("core:view-name")` for URL resolution.

### Template Structure
- `templates/base.html` - Base template with navbar and footer
- `templates/home.html` - Product listing with pagination
- `templates/product.html` - Single product detail view
- `templates/checkout.html` - Checkout form with address and payment options
- `templates/payment.html` - Stripe payment form
- `templates/order_summary.html` - Shopping cart view

### Static Files
- Development: `static_in_env/` (CSS, JS, fonts, images using MDB framework)
- Production: `static_root/` and `media_root/` for collected static and media files

### Authentication
Uses `django-allauth` with templates in `templates/account/` and `templates/socialaccount/`. Login redirect goes to home page.

## Key Patterns

### Cart Implementation
The cart uses the `Order` model with `ordered=False` to represent an active cart. `OrderItem` tracks quantity per product. Cart state persists in the database, not session.

### Price Calculation
`OrderItem.get_final_price()` handles discount logic - uses `discount_price` if available, otherwise regular `price`. `Order.get_total()` sums all items and applies coupon discount.

### Stripe Integration
Stripe customer IDs stored in `UserProfile.stripe_customer_id`. The `one_click_purchasing` flag enables saved card functionality. Payment errors are caught with specific exception handlers.
