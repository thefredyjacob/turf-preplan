### Full-Stack Football Turf Booking Application Design

#### 1\. Database Schema (MongoDB with Mongoose)


// User.js
const userSchema \= new Schema({
  phoneNumber: { type: String, required: true, unique: true },
  reliabilityScore: { type: Number, default: 100, min: 0, max: 100 },
  bookingHistory: \[{ type: Schema.Types.ObjectId, ref: 'Booking' }\],
  createdAt: { type: Date, default: Date.now }
});

// Turf.js
const turfSchema \= new Schema({
  name: { type: String, required: true },
  location: { type: String, required: true },
  slots: \[{
    date: Date,
    startTime: { type: Date, required: true },
    endTime: { type: Date, required: true },
    isAvailable: Boolean,
    price: Number
  }\],
  adminId: { type: Schema.Types.ObjectId, ref: 'Admin' }
});

// Booking.js
const bookingSchema \= new Schema({
  userId: { type: Schema.Types.ObjectId, ref: 'User', required: true },
  turfId: { type: Schema.Types.ObjectId, ref: 'Turf', required: true },
  slots: \[{ type: Schema.Types.ObjectId, ref: 'Slot' }\],
  status: { 
    type: String, 
    enum: \['pending', 'confirmed', 'cancelled', 'completed', 'no-show'\], 
    default: 'pending' 
  },
  qrCode: { type: String, required: true },
  uniqueCode: { type: String, unique: true },
  createdAt: { type: Date, default: Date.now },
  paymentStatus: { type: Boolean, default: false }
});

// Admin.js
const adminSchema \= new Schema({
  userId: { type: Schema.Types.ObjectId, ref: 'User' },
  managedTurfs: \[{ type: Schema.Types.ObjectId, ref: 'Turf' }\],
  notifications: \[{
    type: { type: String, enum: \['new-booking', 'cancellation', 'no-show'\] },
    bookingId: { type: Schema.Types.ObjectId, ref: 'Booking' },
    timestamp: Date
  }\]
});

#### 2\. API Endpoints Documentation

**Authentication**

*   `POST /auth/send-otp`  
    Request: `{ phoneNumber: string }`  
    Response: `{ success: boolean }`
    
*   `POST /auth/verify-otp`  
    Request: `{ phoneNumber: string, otp: string }`  
    Response: `{ token: JWT, userId: string }`
    

**Turf & Availability**

*   `GET /turf` (List all turfs)  
    Response: `[{ id: string, name: string, location: string }]`
    
*   `GET /turf/:id/availability?date=YYYY-MM-DD`  
    Response: `{ slots: [{ id: string, startTime: ISO, endTime: ISO, isAvailable: bool, price: number }] }`
    

**Booking Management**

*   `POST /booking`  
    Request: `{ userId: string, turfId: string, slotIds: [string] }`  
    Response: `{ bookingId: string, qrCode: base64, uniqueCode: string }`
    
*   `PATCH /booking/:id/cancel`  
    Response: `{ success: boolean }`
    
*   `GET /user/:id/bookings`  
    Response: `[{ id: string, turfName: string, date: ISO, slots: [], status: string, qrCode: string }]`
    

**Admin Endpoints**

*   `PATCH /admin/booking/:id/confirm`  
    Response: `{ success: boolean }`
    
*   `GET /admin/bookings?status=pending`  
    Response: `[{ bookingId: string, userPhone: string, turf: string, slots: [] }]`
    
*   `GET /admin/reliability-scores`  
    Response: `[{ userId: string, phoneNumber: string, score: number }]`
    

**QR Code**

*   `GET /booking/:code/verify` (QR validation)  
    Response: `{ valid: boolean, bookingDetails: object }`
    

#### 3\. State Management Diagram


#### 4\. Wireframes Description

**1\. QR Landing Page**

*   Large QR code graphic
    
*   "Scan to book turf" tagline
    
*   App download links (App Store/Play Store)
    

**2\. Phone Login Screen**

*   Phone number input field
    
*   "Send OTP" button
    
*   Country code selector
    

**3\. Slot Availability Grid**

*   Calendar date picker
    
*   15x2 grid (30-min slots from 8AM-11PM)
    
*   Color coding: Green (available), Gray (booked)
    
*   Slot selection counter
    

**4\. Booking Confirmation**

*   Turf name, date/time summary
    
*   QR code with share button
    
*   "Add to Calendar" option
    
*   Cancellation policy notice
    

**5\. Booking History**

*   Filterable list (Upcoming/Past)
    
*   Status badges: Confirmed/Cancelled/No-show
    
*   QR code regeneration button
    
*   Cancellation cut-off timer
    

**6\. Admin Dashboard**

*   Real-time booking notifications
    
*   Turf management panel
    
*   Reliability score heatmap
    
*   Quick-action buttons (Confirm/Cancel)
    

#### 5\. Error Handling Strategy

**Error Types:**

*   Validation Errors (400): Zod schema violations
    
*   Authentication Errors (401): Invalid OTP/token
    
*   Authorization Errors (403): Role-based access violations
    
*   Conflict Errors (409): Slot already booked
    
*   Business Rule Errors (422): Cancellation cutoff violation
    

**Response Structure:**

json

Copy

Download

{
  "error": {
    "code": "CANCELLATION\_CUTOFF",
    "message": "Cancellations require 1-hour advance notice",
    "details": {
      "minCancellationTime": "60 minutes"
    }
  }
}

#### 6\. Security Plan

**Authentication:**

*   JWT with 30-minute expiry
    
*   Refresh token rotation
    
*   OTP rate limiting (3 attempts/hour)
    

**Authorization:**

*   RBAC Middleware:
    
    javascript
    
    Copy
    
    Download
    
    const requireRole \= (role) \=> (req, res, next) \=> {
      if (req.user.role !== role) return res.status(403).json(...);
      next();
    };
    

**Data Protection:**

*   Field-level encryption for phone numbers
    
*   Mongoose schema validation for all models
    
*   Helmet.js for HTTP header security
    

**QR Security:**

*   HMAC-signed booking codes
    
*   Short-lived verification tokens (5-min validity)
    
*   Rate-limited validation endpoint
    

### Implementation Roadmap

1.  **Phase 1: Core Functionality**
    
    *   Authentication service
        
    *   Turf/slot management
        
    *   Booking engine with QR generation
        
    *   Real-time updates (Socket.IO)
        
2.  **Phase 2: Admin Features**
    
    *   Approval workflow
        
    *   Reliability scoring system
        
    *   Notification service
        
3.  **Phase 3: Add-ons**
    
    *   Mock payment integration
        
    *   Weather API integration
        
    *   Dynamic pricing engine
        

**Testing Strategy:**

*   Jest unit tests for business rules (cancellation cutoff, slot conflicts)
    
*   Supertest API integration tests
    
*   Cypress E2E test scenarios:
    
    javascript
    
    Copy
    
    Download
    
    it('Completes booking flow', () \=> {
      cy.scanQR();
      cy.loginWithOTP('+1234567890');
      cy.selectSlot('08:00-08:30');
      cy.confirmBooking();
      cy.get('.qr-code').should('be.visible');
    });
    

**Deployment:**

*   Docker containerization
    
*   AWS ECS Fargate deployment
    
*   GitHub Actions CI/CD pipeline
    
*   CloudWatch monitoring
    

This design provides a scalable foundation with clear extension points for add-on features while ensuring robust security and real-time functionality critical for turf booking operations.
