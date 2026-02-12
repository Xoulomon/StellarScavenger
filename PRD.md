# Product Requirements Document (PRD)
## Scavngr Stellar - Waste Pickup Platform

**Version:** 1.0  
**Date:** February 12, 2026  
**Status:** Draft

---

## 1. Product Overview

### 1.1 Vision
A blockchain-based platform connecting recyclers and collectors for efficient waste pickup with transparent, instant payments.

### 1.2 Problem Statement
- Recyclers lack convenient waste disposal options
- Collectors struggle to find waste sources efficiently
- Payment coordination is manual and unreliable
- No transparency in waste tracking

### 1.3 Solution
Two-sided marketplace on Stellar blockchain with GPS tracking, pickup scheduling, and direct payment system.

---

## 2. User Roles

### 2.1 Recycler
**Definition:** Individual or business with recyclable waste

**Goals:**
- Dispose of waste conveniently
- Earn money from recyclables
- Know pickup schedule in advance
- Receive instant payment

**Pain Points:**
- Uncertain pickup times
- Payment delays
- No visibility into collector availability

### 2.2 Collector
**Definition:** Individual or business collecting recyclable waste

**Goals:**
- Find waste sources efficiently
- Optimize collection routes
- Flexible payment options
- Build reputation

**Pain Points:**
- Finding waste locations
- Route planning inefficiency
- Payment coordination

---

## 3. Core Features

### 3.1 Waste Registration (Recycler)

**Description:** Recyclers register waste items with details and location.

**Requirements:**
- FR-1.1: Capture GPS coordinates automatically
- FR-1.2: Select waste type (Paper, Plastic, PET Plastic, Metal, Glass)
- FR-1.3: Enter weight in kg
- FR-1.4: Add optional notes
- FR-1.5: Generate unique waste ID
- FR-1.6: Store on blockchain

**Acceptance Criteria:**
- GPS accuracy within 10 meters
- Weight validation (0.1kg - 1000kg)
- Waste appears on collector map within 5 seconds

---

### 3.2 Waste Discovery (Collector)

**Description:** Collectors view available waste on interactive map.

**Requirements:**
- FR-2.1: Display waste as map markers
- FR-2.2: Filter by waste type
- FR-2.3: Filter by distance (radius)
- FR-2.4: Show waste details on marker click
- FR-2.5: Display distance from collector
- FR-2.6: Real-time updates

**Acceptance Criteria:**
- Map loads within 2 seconds
- Markers update in real-time
- Distance calculation accurate
- Filter results instant

---

### 3.3 Pickup Scheduling (Collector)

**Description:** Collectors schedule pickup dates for waste items.

**Requirements:**
- FR-3.1: Select waste from map
- FR-3.2: Choose pickup date/time
- FR-3.3: Add pickup notes
- FR-3.4: Send notification to recycler
- FR-3.5: Update waste status to SCHEDULED
- FR-3.6: Add to collector's calendar

**Acceptance Criteria:**
- Date must be future date
- Recycler receives notification within 10 seconds
- Calendar updates immediately
- Cannot double-book same time slot

---

### 3.4 Pickup Notifications (Both)

**Description:** Real-time notifications for pickup events.

**Requirements:**
- FR-4.1: Pickup scheduled notification (Recycler)
- FR-4.2: Pickup reminder 24h before (Both)
- FR-4.3: Payment received notification (Recycler)
- FR-4.4: Pickup confirmed notification (Both)
- FR-4.5: Pickup cancelled notification (Both)

**Acceptance Criteria:**
- Notifications delivered within 10 seconds
- Push notifications on mobile
- In-app notifications
- Email notifications (optional)

---

### 3.5 Payment System (Collector)

**Description:** Collectors pay recyclers directly upon waste collection.

**Requirements:**
- FR-5.1: Support SCV Token payment
- FR-5.2: Support USDC payment
- FR-5.3: Display suggested amount based on weight
- FR-5.4: Show collector's token balance
- FR-5.5: Validate sufficient balance
- FR-5.6: Execute payment transaction
- FR-5.7: Update waste status to PAID
- FR-5.8: Record payment on blockchain

**Acceptance Criteria:**
- Payment completes within 5 seconds
- Transaction recorded on blockchain
- Recycler receives payment instantly
- Payment cannot be reversed
- Gas fees displayed before confirmation

---

### 3.6 Pickup Confirmation (Both)

**Description:** Both parties confirm pickup completion.

**Requirements:**
- FR-6.1: Collector confirms after payment
- FR-6.2: Recycler confirms after receiving payment
- FR-6.3: Status updates to COMPLETED after both confirmations
- FR-6.4: Record completion timestamp
- FR-6.5: Update participant statistics

**Acceptance Criteria:**
- Requires both confirmations
- Cannot confirm before payment
- Completion timestamp accurate
- Statistics update immediately

---

## 4. Technical Requirements

### 4.1 Smart Contract (Soroban)

**Requirements:**
- TR-1.1: Deploy on Stellar testnet/mainnet
- TR-1.2: Support participant registration
- TR-1.3: Store waste data on-chain
- TR-1.4: Handle pickup scheduling
- TR-1.5: Process payments (SCV/USDC)
- TR-1.6: Emit events for all actions
- TR-1.7: Access control (role-based)
- TR-1.8: Reentrancy protection

**Performance:**
- Transaction confirmation: < 5 seconds
- Gas cost: < $0.001 per transaction
- Contract size: < 100KB

---

### 4.2 Frontend Application

**Requirements:**
- TR-2.1: React 18 + TypeScript
- TR-2.2: Responsive design (mobile-first)
- TR-2.3: PWA support
- TR-2.4: Wallet integration (Freighter)
- TR-2.5: Google Maps integration
- TR-2.6: Real-time updates
- TR-2.7: Offline support (basic)

**Performance:**
- Initial load: < 3 seconds
- Map render: < 2 seconds
- Transaction signing: < 1 second
- 90+ Lighthouse score

---

### 4.3 Backend Services

**Requirements:**
- TR-3.1: Firebase Authentication
- TR-3.2: Firestore database
- TR-3.3: Push notification service
- TR-3.4: Geocoding API
- TR-3.5: RESTful API endpoints

**Performance:**
- API response: < 500ms
- 99.9% uptime
- Auto-scaling

---

## 5. User Flows

### 5.1 Recycler Flow

```
1. Connect Wallet
   ↓
2. Register as Recycler
   ↓
3. Register Waste
   - Select type
   - Enter weight
   - GPS auto-captured
   - Submit
   ↓
4. Wait for Collector
   - View on dashboard
   - Status: PENDING
   ↓
5. Receive Notification
   - "Pickup scheduled for [date]"
   - View collector details
   ↓
6. Pickup Day
   - Receive payment notification
   - Verify amount
   ↓
7. Confirm Pickup
   - Click confirm button
   - Status: COMPLETED
   ↓
8. View Earnings
   - Check wallet balance
   - View transaction history
```

---

### 5.2 Collector Flow

```
1. Connect Wallet
   ↓
2. Register as Collector
   ↓
3. Fund Wallet
   - Add SCV or USDC
   ↓
4. View Waste Map
   - See available waste
   - Filter by type/distance
   ↓
5. Select Waste
   - Click marker
   - View details
   ↓
6. Schedule Pickup
   - Choose date/time
   - Add notes
   - Submit
   ↓
7. Pickup Day
   - Navigate to location
   - Collect waste
   ↓
8. Pay Recycler
   - Select token (SCV/USDC)
   - Enter amount
   - Confirm payment
   ↓
9. Confirm Pickup
   - Click confirm button
   - Status: COMPLETED
   ↓
10. View Statistics
    - Total collected
    - Earnings potential
```

---

## 6. Data Models

### 6.1 Participant
```
- address: Address (PK)
- role: RECYCLER | COLLECTOR
- name: String
- phone: String
- address: String
- latitude: i128
- longitude: i128
- is_active: bool
- total_waste_handled: u128
- total_tokens_earned: u128
- rating: u32
- created_at: u64
```

### 6.2 Waste
```
- waste_id: u128 (PK)
- waste_type: PAPER | PLASTIC | PETPLASTIC | METAL | GLASS
- weight: u128
- recycler: Address (FK)
- latitude: i128
- longitude: i128
- registered_timestamp: u64
- pickup_status: PENDING | SCHEDULED | PAID | COMPLETED | CANCELLED
- collector: Option<Address> (FK)
- pickup_date: Option<u64>
- payment_amount: u128
- payment_token: Address
- payment_timestamp: Option<u64>
- recycler_confirmed: bool
- collector_confirmed: bool
- completed_timestamp: Option<u64>
- notes: String
```

### 6.3 PickupSchedule
```
- pickup_id: u128 (PK)
- waste_id: u128 (FK)
- recycler: Address (FK)
- collector: Address (FK)
- scheduled_date: u64
- latitude: i128
- longitude: i128
- status: PickupStatus
- recycler_confirmed: bool
- collector_confirmed: bool
- created_timestamp: u64
- notes: String
```

---

## 7. API Specifications

### 7.1 Smart Contract API

#### register_participant
```rust
fn register_participant(
    env: Env,
    role: ParticipantRole,
    name: String,
    phone: String,
    address: String,
    latitude: i128,
    longitude: i128
)
```

#### register_waste
```rust
fn register_waste(
    env: Env,
    waste_type: WasteType,
    weight: u128,
    latitude: i128,
    longitude: i128,
    notes: String
) -> u128
```

#### schedule_pickup
```rust
fn schedule_pickup(
    env: Env,
    waste_id: u128,
    pickup_date: u64,
    notes: String
)
```

#### pay_for_waste
```rust
fn pay_for_waste(
    env: Env,
    waste_id: u128,
    payment_token: Address,
    amount: u128
)
```

#### confirm_pickup
```rust
fn confirm_pickup(
    env: Env,
    pickup_id: u128
)
```

---

### 7.2 REST API

#### GET /api/waste/nearby
```json
Query: { lat, long, radius }
Response: {
  "waste": [
    {
      "waste_id": 1,
      "type": "PLASTIC",
      "weight": 5000,
      "location": { "lat": 40.7128, "lng": -74.0060 },
      "distance": 2.5
    }
  ]
}
```

#### POST /api/notifications/send
```json
Body: {
  "userId": "address",
  "title": "Pickup Scheduled",
  "body": "Your waste will be collected on Feb 15",
  "data": { "wasteId": 1, "pickupDate": "2026-02-15T10:00:00Z" }
}
```

---

## 8. Security Requirements

### 8.1 Smart Contract Security
- SEC-1.1: Role-based access control
- SEC-1.2: Reentrancy protection
- SEC-1.3: Input validation
- SEC-1.4: Integer overflow protection
- SEC-1.5: Payment verification

### 8.2 Data Security
- SEC-2.1: Location data encrypted in transit
- SEC-2.2: Private keys never stored
- SEC-2.3: HTTPS only
- SEC-2.4: API rate limiting
- SEC-2.5: CORS configuration

### 8.3 Privacy
- SEC-3.1: Location visible only to assigned collector
- SEC-3.2: Personal data encrypted
- SEC-3.3: GDPR compliance
- SEC-3.4: User data deletion option

---

## 9. Performance Requirements

### 9.1 Response Times
- Map load: < 2 seconds
- Transaction confirmation: < 5 seconds
- API response: < 500ms
- Notification delivery: < 10 seconds

### 9.2 Scalability
- Support 10,000+ concurrent users
- Handle 1,000+ transactions per day
- Store 100,000+ waste records

### 9.3 Availability
- 99.9% uptime
- Graceful degradation
- Offline mode for basic features

---

## 10. Success Metrics

### 10.1 User Metrics
- Monthly Active Users (MAU)
- User retention rate (30-day)
- Average waste per recycler
- Average pickups per collector

### 10.2 Transaction Metrics
- Total waste collected (kg)
- Total transactions
- Average transaction value
- Payment success rate

### 10.3 Platform Metrics
- Average time to pickup
- Pickup completion rate
- User satisfaction rating
- Platform fee revenue

**Targets (Year 1):**
- 5,000 registered users
- 10,000 kg waste collected
- 95% pickup completion rate
- 4.5+ average rating

---

## 11. Pricing Model

### 11.1 Suggested Rates (per kg)
| Waste Type | SCV Token | USDC |
|-----------|-----------|------|
| Paper | 5-10 | $0.50-1.00 |
| Plastic | 10-15 | $1.00-1.50 |
| PET Plastic | 15-20 | $1.50-2.00 |
| Metal | 20-30 | $2.00-3.00 |
| Glass | 8-12 | $0.80-1.20 |

### 11.2 Platform Revenue
- Optional platform fee: 2-5% of transaction
- Premium features (analytics, bulk operations)
- Enterprise licenses

---

## 12. Development Phases

### Phase 1: MVP (8 weeks)
- ✅ Smart contract development
- ✅ Basic frontend (registration, waste listing)
- ✅ Wallet integration
- ✅ Map view
- ✅ Payment system

### Phase 2: Core Features (6 weeks)
- ✅ Pickup scheduling
- ✅ Notifications
- ✅ Calendar integration
- ✅ Route optimization
- ✅ Rating system

### Phase 3: Enhancement (4 weeks)
- ✅ Mobile app (PWA)
- ✅ Advanced analytics
- ✅ Multi-language support
- ✅ Dispute resolution

### Phase 4: Scale (Ongoing)
- ✅ Performance optimization
- ✅ Marketing & growth
- ✅ Partnership integrations
- ✅ Additional features

---

## 13. Risks & Mitigation

### 13.1 Technical Risks
| Risk | Impact | Mitigation |
|------|--------|------------|
| Smart contract bugs | High | Audit, testing, bug bounty |
| Stellar network downtime | Medium | Fallback mechanisms, status page |
| GPS inaccuracy | Medium | Validation, manual correction |
| Payment failures | High | Retry logic, error handling |

### 13.2 Business Risks
| Risk | Impact | Mitigation |
|------|--------|------------|
| Low user adoption | High | Marketing, incentives, partnerships |
| Price volatility (SCV) | Medium | USDC option, stablecoin default |
| Regulatory issues | Medium | Legal compliance, KYC if needed |
| Competition | Medium | Unique features, better UX |

---

## 14. Dependencies

### 14.1 External Services
- Stellar Network (blockchain)
- Google Maps API (mapping)
- Firebase (auth, database, notifications)
- Freighter Wallet (wallet integration)

### 14.2 Third-Party Libraries
- Stellar SDK
- React
- Tailwind CSS
- Soroban CLI

---

## 15. Compliance

### 15.1 Legal
- Terms of Service
- Privacy Policy
- Cookie Policy
- GDPR compliance

### 15.2 Blockchain
- Smart contract audit
- Security best practices
- Transparent operations

---

## 16. Future Enhancements

- AI-powered route optimization
- Carbon credit tokenization
- IoT device integration (smart bins)
- Marketplace for recycled materials
- Corporate sustainability programs
- Gamification & rewards
- Social features (leaderboards)
- Multi-chain support

---

## 17. Appendix

### 17.1 Glossary
- **SCV**: Scavenger Token (platform native token)
- **USDC**: USD Coin (stablecoin)
- **Soroban**: Stellar smart contract platform
- **Stroops**: Smallest unit of Stellar tokens (1e-7)
- **PWA**: Progressive Web App

### 17.2 References
- Stellar Documentation: https://developers.stellar.org/
- Soroban Docs: https://soroban.stellar.org/
- Freighter Wallet: https://www.freighter.app/

---

**Document Owner:** Product Team  
**Last Updated:** February 12, 2026  
**Next Review:** March 12, 2026
