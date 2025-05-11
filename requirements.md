# 📄 Backend Feature Requirement Specifications

## ✅ Overview

This document outlines the **technical and functional requirements** for the following core backend features in the Airbnb Clone project:

1. User Authentication
2. Property Management
3. Booking System

Each feature includes API specifications, validation rules, and performance expectations.

---

## 1. 🔐 User Authentication

### 📌 Functional Requirements

* Allow users to register as either guests or hosts.
* Secure login system using JWT.
* Support password hashing, email validation, and OAuth (Google).

### 🔧 API Endpoints

| Method | Endpoint             | Description                        | Auth Required |
| ------ | -------------------- | ---------------------------------- | ------------- |
| POST   | `/api/auth/register` | Register a new user                | ❌             |
| POST   | `/api/auth/login`    | Authenticate user and return token | ❌             |
| GET    | `/api/user/profile`  | Retrieve user profile              | ✅             |
| PUT    | `/api/user/profile`  | Update user profile info           | ✅             |

### 🧾 Input Specifications

#### `/api/auth/register`

```json
{
  "email": "user@example.com",
  "password": "SecurePass123!",
  "role": "guest" // or "host"
}
```

* Email must be unique and valid.
* Password must be 8+ characters, 1 special char, 1 number.

#### `/api/auth/login`

```json
{
  "email": "user@example.com",
  "password": "SecurePass123!"
}
```

### ✅ Output Example

```json
{
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "user": {
    "id": 1,
    "email": "user@example.com",
    "role": "host"
  }
}
```

### ⚙️ Performance

* Response time under 200ms for login/register.
* Throttle failed login attempts (max 5/min).

---

## 2. 🏠 Property Management

### 📌 Functional Requirements

* Hosts can create, update, delete, and retrieve property listings.
* Properties include metadata: title, description, price, availability, and amenities.
* Images should be stored in file system/cloud.

### 🔧 API Endpoints

| Method | Endpoint              | Description               | Auth Required |
| ------ | --------------------- | ------------------------- | ------------- |
| POST   | `/api/properties`     | Create a new property     | ✅ (host)      |
| GET    | `/api/properties`     | List all properties       | ❌             |
| GET    | `/api/properties/:id` | Retrieve property details | ❌             |
| PUT    | `/api/properties/:id` | Update a property         | ✅ (host)      |
| DELETE | `/api/properties/:id` | Delete a property         | ✅ (host)      |

### 🧾 Input Specifications

#### `/api/properties`

```json
{
  "title": "Cozy Apartment in Rabat",
  "description": "2 bedrooms, city view, fast Wi-Fi.",
  "price_per_night": 50,
  "location": "Rabat, Morocco",
  "amenities": ["Wi-Fi", "Balcony", "Washer"],
  "images": ["image1.jpg", "image2.jpg"]
}
```

### ✅ Output Example

```json
{
  "id": 12,
  "host_id": 4,
  "title": "Cozy Apartment in Rabat",
  "price_per_night": 50,
  "created_at": "2025-05-11"
}
```

### ⚙️ Performance

* Property listing load time under 300ms.
* Images are compressed and stored efficiently.

---

## 3. 📅 Booking System

### 📌 Functional Requirements

* Guests can book available properties for a date range.
* Bookings must validate availability and prevent overlaps.
* Support booking cancellation and status tracking.

### 🔧 API Endpoints

| Method | Endpoint            | Description        | Auth Required |
| ------ | ------------------- | ------------------ | ------------- |
| POST   | `/api/bookings`     | Create a booking   | ✅ (guest)     |
| GET    | `/api/bookings`     | List user bookings | ✅             |
| PUT    | `/api/bookings/:id` | Cancel booking     | ✅             |

### 🧾 Input Specifications

#### `/api/bookings`

```json
{
  "property_id": 12,
  "check_in": "2025-06-01",
  "check_out": "2025-06-05"
}
```

* `check_out` must be after `check_in`.
* Validate property availability for dates.

### ✅ Output Example

```json
{
  "booking_id": 44,
  "status": "confirmed",
  "total_price": 200,
  "guest_id": 3,
  "property_id": 12
}
```

### ⚙️ Performance

* Prevent race conditions using transaction locks.
* Check availability within 100ms for responsive UX.

---

## 4. 💳 Payments

### 📌 Functional Requirements

* Process payments securely for confirmed bookings.
* Integrate with a third-party gateway (e.g., Stripe).
* Support refunding and payment confirmation.
* Associate transactions with users and bookings.

### 🔧 API Endpoints

| Method | Endpoint                   | Description                    | Auth Required |
| ------ | -------------------------- | ------------------------------ | ------------- |
| POST   | `/api/payments/initiate`   | Start payment process          | ✅ (guest)     |
| GET    | `/api/payments/status/:id` | Check payment status           | ✅             |
| POST   | `/api/payments/refund/:id` | Refund a payment (if eligible) | ✅             |

### 🧾 Input Specifications

#### `/api/payments/initiate`

```json
{
  "booking_id": 44,
  "payment_method": "credit_card",
  "amount": 200
}
```

### ✅ Output Example

```json
{
  "payment_id": 101,
  "status": "pending",
  "payment_url": "https://checkout.stripe.com/pay/cs_test..."
}
```

### ⚙️ Performance

* Payment gateway response within 2 seconds.
* Retry logic for failed or pending payments.
* Webhook support for asynchronous status updates.

---

## 5. 🌟 Reviews

### 📌 Functional Requirements

* Guests can leave reviews after completed stays.
* Ratings (1–5) and optional text feedback.
* Hosts can view reviews per property.

### 🔧 API Endpoints

| Method | Endpoint                      | Description          | Auth Required |
| ------ | ----------------------------- | -------------------- | ------------- |
| POST   | `/api/reviews`                | Submit a review      | ✅ (guest)     |
| GET    | `/api/properties/:id/reviews` | Get property reviews | ❌             |

### 🧾 Input Specifications

#### `/api/reviews`

```json
{
  "property_id": 12,
  "rating": 5,
  "comment": "Great experience!"
}
```

* Rating must be integer between 1 and 5.
* One review per booking allowed.

### ✅ Output Example

```json
{
  "review_id": 66,
  "property_id": 12,
  "rating": 5,
  "comment": "Great experience!",
  "user": "Yassine A.",
  "created_at": "2025-06-08"
}
```

### ⚙️ Performance

* Indexed reviews by property for quick access.
* Average rating cached and updated on write.

---

## 6. 🛠️ Admin Dashboard

### 📌 Functional Requirements

* Admin can view user accounts, properties, bookings.
* Admin can deactivate users or remove listings.
* Generate reports (active users, revenue, etc.).

### 🔧 API Endpoints

| Method | Endpoint                    | Description               | Auth Required |
| ------ | --------------------------- | ------------------------- | ------------- |
| GET    | `/api/admin/users`          | List all users            | ✅ (admin)     |
| GET    | `/api/admin/properties`     | List all properties       | ✅ (admin)     |
| DELETE | `/api/admin/users/:id`      | Deactivate user account   | ✅ (admin)     |
| DELETE | `/api/admin/properties/:id` | Remove a property listing | ✅ (admin)     |

### ✅ Output Example

```json
{
  "users": [
    {
      "id": 3,
      "email": "guest@mail.com",
      "status": "active",
      "role": "guest"
    },
    {
      "id": 1,
      "email": "host@mail.com",
      "status": "banned",
      "role": "host"
    }
  ]
}
```

### ⚙️ Performance

* Pagination for datasets larger than 100 entries.
* Admin-only access enforced with RBAC (Role-Based Access Control).
* Audit logs recorded for all admin actions.

---

## 📌 Final Notes

This document now includes complete specifications for:

* User Authentication
* Property Management
* Booking System
* Payments
* Reviews
* Admin Dashboard

Each component is designed for **modularity, scalability, and API-first development**, following RESTful principles and ensuring secure, performant service delivery.
