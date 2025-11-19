# Workspace Booking & Pricing System  
Candidate: Venkata Satyasree Guttula  
Email: satyasreeg2002@gmail.com  

This repository contains both the frontend and backend implementations for the Workspace Booking & Pricing System assignment. The system allows users to book meeting rooms by the hour with conflict prevention, dynamic pricing, cancellation policy, and analytics. Both applications are deployed independently and work together through API communication.

Live Deployments  
Frontend (Netlify): https://thriving-stroopwafel-e7229d.netlify.app/  
Backend (Render): https://craft-backend-expp.onrender.com  

Repository Structure  
/frontend → React frontend  
/backend → Node.js + Express backend  
README.md → Setup instructions and project summary  
ARCHITECTURE.md → System design, rules, pricing logic, analytics explanation  

Project Overview  
The purpose of this mini project is to demonstrate full-stack skills including API design, routing, business rules implementation, pricing logic, booking validation, and frontend + backend integration. This project uses only an in-memory data store so no external database is required. The entire system uses Asia/Kolkata timezone.

How to Run the Backend  
cd backend  
npm install  
npm start  
Backend runs on http://localhost:5000  

How to Run the Frontend  
cd frontend  
npm install  
npm start  
Frontend runs on http://localhost:3000  
Set the backend API URL in a .env file using:  
REACT_APP_API_URL=http://localhost:5000  

API Summary  
1) Create Booking  
POST /api/bookings  
Request  
{  
  "roomId": "101",  
  "userName": "Priya",  
  "startTime": "2025-11-20T10:00:00.000+05:30",  
  "endTime": "2025-11-20T12:30:00.000+05:30"  
}  
Responses  
Success: booking details with calculated price  
Conflict: { "error": "Room already booked from 10:30 AM to 11:30 AM" }  
Validation: duration > 0 and ≤ 12 hours  

2) Cancel Booking  
POST /api/bookings/:id/cancel  
Allowed only if current time is more than 2 hours before startTime. Otherwise responds with 400 Bad Request. Cancelled bookings are excluded from analytics.  

3) Analytics  
GET /api/analytics?from=YYYY-MM-DD&to=YYYY-MM-DD  
Returns a list of rooms with total booked hours and total revenue. Only CONFIRMED bookings are included.  

Pricing Logic Summary  
Peak hours receive 1.5× base rate:  
• Weekdays Mon–Fri  
• 10:00–13:00 and 16:00–19:00  
Off-peak uses baseHourlyRate.  
Billing is calculated per minute (prorated) and rounded to nearest rupee.  

Cancellation Rules  
• Allowed only if current time ≤ startTime − 2 hours  
• Otherwise cancellation fails  

Environment Variables  
Backend uses PORT=5000  
Frontend uses REACT_APP_API_URL  

Why No Database  
The assignment allows in-memory data. This setup makes deployment easier on free tiers and keeps the focus on logic, not DB configuration. In a real system, Postgres or MongoDB would be used with transactions to prevent race conditions.

Known Limitations  
No persistent storage  
No authentication  
Limited error handling  

Future Improvements  
Add DB with transactions  
Add user login  
Add pagination  
Add email notifications  
Add unit tests  

