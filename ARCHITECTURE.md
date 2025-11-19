# ARCHITECTURE.md – Workspace Booking & Pricing System

## Overview
This document explains the complete architecture, data flow, business rules, pricing logic, conflict detection, cancellation policy, analytics calculations, and scalability plan of the Workspace Booking & Pricing System. The application is fully timezone-aware (Asia/Kolkata) and uses an in-memory data store as allowed by the assignment instructions.

---

## 1. Data Models

### Room
Each room has basic metadata such as name, base rate, and capacity.
{
  "id": "101",
  "name": "Cabin 1",
  "baseHourlyRate": 300,
  "capacity": 6
}

### Booking
Every new booking is inserted with calculated price and status.
{
  "bookingId": "b123",
  "roomId": "101",
  "userName": "Priya",
  "startTime": "2025-11-20T10:00:00.000+05:30",
  "endTime": "2025-11-20T12:30:00.000+05:30",
  "totalPrice": 975,
  "status": "CONFIRMED"
}

Rooms are statically seeded (3–5 rooms).  
Bookings are stored in an in-memory array (no database as allowed in assignment).

---

## 2. Timezone Handling (Important)
All calculations, validations, comparisons use:
- Asia/Kolkata timezone  
- ISO timestamp format with +05:30 offset  
This avoids issues with peak-hour calculation, conflict detection, and analytics.

---

## 3. Booking Conflict Detection Logic

A new booking **conflicts** with an existing confirmed booking if:

existing.start < new.end  AND  new.start < existing.end

This is the standard interval-overlap rule.

### Example
Existing booking: 10:00–11:00  
New booking: 10:30–11:30 → **Conflict**

### Allowed Case
Existing: 10:00–11:00  
New: 11:00–12:00 → **Allowed** (end == start is not a conflict)

### Additional Validations
- startTime < endTime  
- Duration ≤ 12 hours  
- Room must exist  
- Timestamp must be valid ISO 8601  

---

## 4. Dynamic Pricing Logic (Core Requirement)

The price is calculated **per minute**, based on whether the minute falls in:
- **Peak hours** → 1.5 × baseHourlyRate  
- **Off-peak hours** → baseHourlyRate  

### Peak Hours (Mon–Fri)
- 10:00 AM – 1:00 PM  
- 4:00 PM – 7:00 PM  

### Billing Method
1. Convert booking into minute-wise slots  
2. For each minute:
   - Check if minute falls in peak range  
   - Apply correct rate  
3. Sum total  
4. Round to nearest rupee  

This method is accurate and avoids rounding errors.

---

## 5. Cancellation Logic

A booking can be cancelled only if:

currentTime ≤ startTime − 2 hours

Else:
- Return 400 Bad Request  
- Reason: "Cannot cancel less than 2 hours before booking start time"  

Cancelled bookings are marked:
status = "CANCELLED"  
and are **excluded from analytics**.

---

## 6. Analytics Design

### Endpoint
GET /api/analytics?from=YYYY-MM-DD&to=YYYY-MM-DD

### Rules
- Only CONFIRMED bookings count  
- Cancelled bookings ignored  
- Only bookings inside date range are counted  

### For each room:
- totalHours = sum of confirmed booking durations  
- totalRevenue = sum of price of confirmed bookings  

### Output Example
[
  {
    "roomId": "101",
    "roomName": "Cabin 1",
    "totalHours": 15.5,
    "totalRevenue": 5250
  }
]

---

## 7. Backend Architecture

### Layered Structure
- **Routes** → handles API inputs  
- **Services** → business logic (pricing, conflicts, analytics)  
- **Data store** → in-memory arrays  
- **Utils** → time & pricing helpers  

### Why in-memory?
The assignment clearly mentions DB is optional.  
Using in-memory helps:
- Deploy easily on Render free tier  
- No environment variable issues  
- Focus stays on logic (as assignment expects)

---

## 8. Frontend Architecture

### Tech stack:
- React  
- Axios for API calls  
- Components for Rooms List, Booking Form, Admin View, Analytics  

### Interaction Flow:
1. User selects room + times  
2. React sends request to backend  
3. Backend responds with:
   - confirmation  
   - price  
   - conflict error message  
4. Admin page shows all bookings  
5. Analytics page shows revenue summary  

---

## 9. Deployment Architecture

### Frontend
- Hosted on **Netlify**
- Connects to backend using environment variable:
REACT_APP_API_URL=https://craft-backend-expp.onrender.com

### Backend
- Hosted on **Render**
- Exposes public `/api` endpoints  

### Workflow
Frontend → makes HTTP requests → Backend  
Both deployed independently but integrated via API URL.

---

## 10. Edge Cases Handled

- StartTime equals EndTime → Rejected  
- Duration > 12 hours → Rejected  
- Invalid roomId → Rejected  
- Overlapping exact boundary end==start allowed  
- Wrong timestamp formats rejected  
- Cancel request after deadline rejected  
- Analytics without valid date range returns 400  

---

## 11. Scaling & Production Notes

To make this production-ready:

### Storage
- Replace in-memory with Postgres / MongoDB  
- Use DB-level row locks to avoid double booking  

### Performance
- Cache room data in Redis  
- Paginate bookings list  
- Add indexes on startTime, roomId  

### Observability
- Add logs for conflicts  
- Add monitoring + rate limits  

### Reliability
- Use queues for analytics  
- Use integer paise-based pricing instead of floating calculation  




