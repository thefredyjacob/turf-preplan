### Full-Stack Football Turf Booking Application Design

#### 1\. Database Schema (MongoDB with Mongoose)

javascript

Copy

Download

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

Diagram

Code

Download

.kvfysmfp{overflow:hidden;touch-action:none}.ufhsfnkm{transform-origin: 0 0}

#mermaid-svg-42{font-family:"trebuchet ms",verdana,arial,sans-serif;font-size:16px;fill:#333;}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#mermaid-svg-42 .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#mermaid-svg-42 .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#mermaid-svg-42 .error-icon{fill:#552222;}#mermaid-svg-42 .error-text{fill:#552222;stroke:#552222;}#mermaid-svg-42 .edge-thickness-normal{stroke-width:1px;}#mermaid-svg-42 .edge-thickness-thick{stroke-width:3.5px;}#mermaid-svg-42 .edge-pattern-solid{stroke-dasharray:0;}#mermaid-svg-42 .edge-thickness-invisible{stroke-width:0;fill:none;}#mermaid-svg-42 .edge-pattern-dashed{stroke-dasharray:3;}#mermaid-svg-42 .edge-pattern-dotted{stroke-dasharray:2;}#mermaid-svg-42 .marker{fill:#333333;stroke:#333333;}#mermaid-svg-42 .marker.cross{stroke:#333333;}#mermaid-svg-42 svg{font-family:"trebuchet ms",verdana,arial,sans-serif;font-size:16px;}#mermaid-svg-42 p{margin:0;}#mermaid-svg-42 .label{font-family:"trebuchet ms",verdana,arial,sans-serif;color:#333;}#mermaid-svg-42 .cluster-label text{fill:#333;}#mermaid-svg-42 .cluster-label span{color:#333;}#mermaid-svg-42 .cluster-label span p{background-color:transparent;}#mermaid-svg-42 .label text,#mermaid-svg-42 span{fill:#333;color:#333;}#mermaid-svg-42 .node rect,#mermaid-svg-42 .node circle,#mermaid-svg-42 .node ellipse,#mermaid-svg-42 .node polygon,#mermaid-svg-42 .node path{fill:#ECECFF;stroke:#9370DB;stroke-width:1px;}#mermaid-svg-42 .rough-node .label text,#mermaid-svg-42 .node .label text,#mermaid-svg-42 .image-shape .label,#mermaid-svg-42 .icon-shape .label{text-anchor:middle;}#mermaid-svg-42 .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#mermaid-svg-42 .rough-node .label,#mermaid-svg-42 .node .label,#mermaid-svg-42 .image-shape .label,#mermaid-svg-42 .icon-shape .label{text-align:center;}#mermaid-svg-42 .node.clickable{cursor:pointer;}#mermaid-svg-42 .root .anchor path{fill:#333333!important;stroke-width:0;stroke:#333333;}#mermaid-svg-42 .arrowheadPath{fill:#333333;}#mermaid-svg-42 .edgePath .path{stroke:#333333;stroke-width:2.0px;}#mermaid-svg-42 .flowchart-link{stroke:#333333;fill:none;}#mermaid-svg-42 .edgeLabel{background-color:rgba(232,232,232, 0.8);text-align:center;}#mermaid-svg-42 .edgeLabel p{background-color:rgba(232,232,232, 0.8);}#mermaid-svg-42 .edgeLabel rect{opacity:0.5;background-color:rgba(232,232,232, 0.8);fill:rgba(232,232,232, 0.8);}#mermaid-svg-42 .labelBkg{background-color:rgba(232, 232, 232, 0.5);}#mermaid-svg-42 .cluster rect{fill:#ffffde;stroke:#aaaa33;stroke-width:1px;}#mermaid-svg-42 .cluster text{fill:#333;}#mermaid-svg-42 .cluster span{color:#333;}#mermaid-svg-42 div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:"trebuchet ms",verdana,arial,sans-serif;font-size:12px;background:hsl(80, 100%, 96.2745098039%);border:1px solid #aaaa33;border-radius:2px;pointer-events:none;z-index:100;}#mermaid-svg-42 .flowchartTitleText{text-anchor:middle;font-size:18px;fill:#333;}#mermaid-svg-42 rect.text{fill:none;stroke-width:0;}#mermaid-svg-42 .icon-shape,#mermaid-svg-42 .image-shape{background-color:rgba(232,232,232, 0.8);text-align:center;}#mermaid-svg-42 .icon-shape p,#mermaid-svg-42 .image-shape p{background-color:rgba(232,232,232, 0.8);padding:2px;}#mermaid-svg-42 .icon-shape rect,#mermaid-svg-42 .image-shape rect{opacity:0.5;background-color:rgba(232,232,232, 0.8);fill:rgba(232,232,232, 0.8);}#mermaid-svg-42 :root{--mermaid-font-family:"trebuchet ms",verdana,arial,sans-serif;}

Yes

No

Enabled

Disabled

Success

User Scans QR

Login with Phone/OTP

Multiple Turfs?

Turf Selection

Real-time Slot Availability

Slot Selection

Payment Add-on?

Mock Payment Flow

Create Booking

Generate QR Code

Share Booking

Admin Login

New Booking Alerts

Confirm/Reject Booking

Update Availability

User Notification

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
