# Implementation Plan - Scavngr Stellar with Buy Listings

**Version:** 1.0  
**Date:** February 12, 2026  
**Timeline:** 8 weeks (2 months)

📅 **[View Detailed Project Timeline](./PROJECT_TIMELINE.md)** - Week-by-week breakdown

---

## Overview

Implement Stellar/Soroban smart contract enabling:
- Recyclers to register waste (supply)
- Collectors to post buy listings (demand)
- Automatic matching between supply and demand
- Direct payment in SCV/USDC

---

## Phase 1: Smart Contract Core (Week 1-2)

**Timeline:** Feb 12-25, 2026  
**Focus:** Foundation + Waste Management

### 1.1 Data Structures

```rust
// contracts/waste-pickup/src/types.rs

pub struct Waste {
    waste_id: u128,
    waste_type: WasteType,
    weight: u128,              // grams
    recycler: Address,
    latitude: i128,            // * 1_000_000
    longitude: i128,           // * 1_000_000
    status: WasteStatus,
    created_at: u64,
    collector: Option<Address>,
    buy_listing_id: Option<u128>,
}

pub struct BuyListing {
    listing_id: u128,
    collector: Address,
    waste_type: WasteType,
    max_weight: u128,          // grams - max they can buy
    price_per_kg: u128,        // stroops (1 XLM = 10^7 stroops)
    payment_token: Address,    // SCV or USDC
    latitude: i128,
    longitude: i128,
    max_distance_km: u32,      // pickup radius
    status: ListingStatus,
    created_at: u64,
    expires_at: u64,
}

pub struct Match {
    match_id: u128,
    waste_id: u128,
    listing_id: u128,
    recycler: Address,
    collector: Address,
    weight: u128,
    total_price: u128,
    status: MatchStatus,
    created_at: u64,
    pickup_date: Option<u64>,
}

pub enum WasteType {
    PAPER,
    PLASTIC,
    PETPLASTIC,
    METAL,
    GLASS,
}

pub enum WasteStatus {
    AVAILABLE,
    MATCHED,
    SCHEDULED,
    COMPLETED,
    CANCELLED,
}

pub enum ListingStatus {
    ACTIVE,
    PAUSED,
    FILLED,
    EXPIRED,
    CANCELLED,
}

pub enum MatchStatus {
    PENDING,
    ACCEPTED,
    SCHEDULED,
    PAID,
    COMPLETED,
    CANCELLED,
}
```

### 1.2 Core Functions

```rust
// contracts/waste-pickup/src/lib.rs

// Participant Management
fn register_participant(
    env: Env,
    role: ParticipantRole,
    name: String,
    phone: String,
    address: String,
    latitude: i128,
    longitude: i128,
)

// Waste Management (Recyclers)
fn register_waste(
    env: Env,
    waste_type: WasteType,
    weight: u128,
    latitude: i128,
    longitude: i128,
    notes: String,
) -> u128  // returns waste_id

fn get_recycler_wastes(env: Env, recycler: Address) -> Vec<Waste>

fn cancel_waste(env: Env, waste_id: u128)

// Buy Listings (Collectors)
fn create_buy_listing(
    env: Env,
    waste_type: WasteType,
    max_weight: u128,
    price_per_kg: u128,
    payment_token: Address,
    latitude: i128,
    longitude: i128,
    max_distance_km: u32,
    expires_in_days: u32,
) -> u128  // returns listing_id

fn update_buy_listing(
    env: Env,
    listing_id: u128,
    price_per_kg: Option<u128>,
    max_weight: Option<u128>,
    max_distance_km: Option<u32>,
)

fn pause_buy_listing(env: Env, listing_id: u128)

fn resume_buy_listing(env: Env, listing_id: u128)

fn cancel_buy_listing(env: Env, listing_id: u128)

fn get_collector_listings(env: Env, collector: Address) -> Vec<BuyListing>

fn get_active_listings(
    env: Env,
    waste_type: Option<WasteType>,
    latitude: Option<i128>,
    longitude: Option<i128>,
    radius_km: Option<u32>,
) -> Vec<BuyListing>

// Matching System
fn find_matches_for_waste(env: Env, waste_id: u128) -> Vec<BuyListing>

fn find_matches_for_listing(env: Env, listing_id: u128) -> Vec<Waste>

fn create_match(
    env: Env,
    waste_id: u128,
    listing_id: u128,
) -> u128  // returns match_id

fn accept_match(env: Env, match_id: u128)

fn reject_match(env: Env, match_id: u128)

// Pickup & Payment
fn schedule_pickup(
    env: Env,
    match_id: u128,
    pickup_date: u64,
    notes: String,
)

fn pay_for_waste(
    env: Env,
    match_id: u128,
)

fn confirm_pickup(env: Env, match_id: u128)

fn get_collector_matches(env: Env, collector: Address) -> Vec<Match>

fn get_recycler_matches(env: Env, recycler: Address) -> Vec<Match>
```

### 1.3 Matching Logic

```rust
// contracts/waste-pickup/src/matching.rs

fn calculate_distance(lat1: i128, lon1: i128, lat2: i128, lon2: i128) -> u32 {
    // Haversine formula implementation
    // Returns distance in km
}

fn is_match_valid(waste: &Waste, listing: &BuyListing) -> bool {
    // Check waste type matches
    if waste.waste_type != listing.waste_type {
        return false;
    }
    
    // Check weight within listing capacity
    if waste.weight > listing.max_weight {
        return false;
    }
    
    // Check distance
    let distance = calculate_distance(
        waste.latitude, waste.longitude,
        listing.latitude, listing.longitude
    );
    if distance > listing.max_distance_km {
        return false;
    }
    
    // Check listing not expired
    if listing.expires_at < env.ledger().timestamp() {
        return false;
    }
    
    // Check statuses
    waste.status == WasteStatus::AVAILABLE && 
    listing.status == ListingStatus::ACTIVE
}

fn auto_match_waste(env: &Env, waste_id: u128) -> Option<u128> {
    // Find best matching listing (highest price, closest distance)
    // Create match automatically
    // Return match_id
}
```

---

## Phase 2: Frontend Integration (Week 3)

**Timeline:** Feb 26 - Mar 4, 2026  
**Focus:** Buy Listings UI

### 2.1 Collector Dashboard Updates

```typescript
// frontend/client/pages/CollectorDashboard.tsx

// Add Buy Listings Tab
<Tabs>
  <Tab label="Available Waste">
    <WasteMap />
  </Tab>
  <Tab label="My Buy Listings">
    <BuyListingsManager />
  </Tab>
  <Tab label="Matches">
    <MatchesList />
  </Tab>
</Tabs>
```

### 2.2 New Components

```typescript
// frontend/client/components/CreateBuyListingModal.tsx
interface CreateBuyListingModalProps {
  onClose: () => void;
  onSuccess: (listingId: string) => void;
}

// Fields:
// - Waste Type (dropdown)
// - Max Weight (kg)
// - Price per kg (SCV/USDC)
// - Payment Token (SCV/USDC toggle)
// - Pickup Radius (km slider)
// - Expiration (days)

// frontend/client/components/BuyListingCard.tsx
// Display listing details, edit/pause/cancel actions

// frontend/client/components/MatchCard.tsx
// Display match details, accept/reject/schedule actions

// frontend/client/components/WasteMatchesModal.tsx
// Show available buy listings for recycler's waste
```

### 2.3 Hooks

```typescript
// frontend/client/hooks/useBuyListings.ts
export function useBuyListings(collectorAddress?: string) {
  const [listings, setListings] = useState<BuyListing[]>([]);
  
  const createListing = async (data: CreateListingData) => {
    // Call contract
  };
  
  const updateListing = async (listingId: string, updates: Partial<BuyListing>) => {
    // Call contract
  };
  
  const pauseListing = async (listingId: string) => {
    // Call contract
  };
  
  return { listings, createListing, updateListing, pauseListing };
}

// frontend/client/hooks/useMatches.ts
export function useMatches(userAddress: string, role: 'recycler' | 'collector') {
  const [matches, setMatches] = useState<Match[]>([]);
  
  const acceptMatch = async (matchId: string) => {};
  const rejectMatch = async (matchId: string) => {};
  const schedulePickup = async (matchId: string, date: Date) => {};
  
  return { matches, acceptMatch, rejectMatch, schedulePickup };
}
```

---

## Phase 3: Matching & Notifications (Week 4)

**Timeline:** Mar 5-11, 2026  
**Focus:** Auto-matching system

### 3.1 Auto-Matching Service

```typescript
// frontend/server/services/matchingService.ts

class MatchingService {
  async findMatchesForWaste(wasteId: string): Promise<BuyListing[]> {
    // Query contract for compatible listings
    // Sort by price (highest first) and distance (closest first)
  }
  
  async findMatchesForListing(listingId: string): Promise<Waste[]> {
    // Query contract for compatible waste
    // Sort by distance and creation date
  }
  
  async notifyMatches(matchId: string) {
    // Send notifications to both parties
  }
}
```

### 3.2 Notification System

```typescript
// frontend/server/services/notificationService.ts

interface Notification {
  type: 'MATCH_FOUND' | 'MATCH_ACCEPTED' | 'PICKUP_SCHEDULED' | 'PAYMENT_RECEIVED';
  userId: string;
  data: any;
}

// Push notifications
// Email notifications
// In-app notifications
```

---

## Phase 4: Payment & Completion (Week 5-6)

**Timeline:** Mar 12-25, 2026  
**Focus:** Maps, Scheduling, Payment

### 4.1 Payment Flow

```rust
// contracts/waste-pickup/src/payment.rs

fn pay_for_waste(env: Env, match_id: u128) {
    let match_data = get_match(&env, match_id);
    let listing = get_listing(&env, match_data.listing_id);
    let collector = env.invoker();
    
    // Verify collector is match participant
    assert_eq!(collector, match_data.collector);
    
    // Verify match status
    assert_eq!(match_data.status, MatchStatus::SCHEDULED);
    
    // Transfer tokens from collector to recycler
    let token = token::Client::new(&env, &listing.payment_token);
    token.transfer(
        &collector,
        &match_data.recycler,
        &(match_data.total_price as i128)
    );
    
    // Update match status
    update_match_status(&env, match_id, MatchStatus::PAID);
    
    // Emit event
    env.events().publish(("payment", match_id), match_data.total_price);
}
```

### 4.2 Completion Flow

```typescript
// Both parties confirm pickup
// Status updates to COMPLETED
// Listing capacity decreases
// Stats update
```

---

## Phase 5: Testing & Deployment (Week 7-8)

**Timeline:** Mar 26 - Apr 12, 2026  
**Focus:** Polish, Testing, Launch

### 5.1 Contract Tests

```rust
// contracts/waste-pickup/tests/test.rs

#[test]
fn test_create_buy_listing() {
    // Test listing creation
}

#[test]
fn test_matching_logic() {
    // Test waste-listing matching
}

#[test]
fn test_payment_flow() {
    // Test payment execution
}

#[test]
fn test_distance_calculation() {
    // Test GPS distance calculation
}
```

### 5.2 Integration Tests

```typescript
// frontend/tests/integration/buyListings.test.ts

describe('Buy Listings Flow', () => {
  it('should create buy listing', async () => {});
  it('should match waste with listing', async () => {});
  it('should complete payment', async () => {});
});
```

### 5.3 Deployment

```bash
# Deploy to Stellar Testnet
soroban contract deploy \
  --wasm target/wasm32-unknown-unknown/release/waste_pickup.wasm \
  --source alice \
  --network testnet

# Deploy SCV Token
soroban contract deploy \
  --wasm scv_token.wasm \
  --source alice \
  --network testnet

# Initialize contracts
soroban contract invoke \
  --id $CONTRACT_ID \
  --source alice \
  --network testnet \
  -- initialize \
  --scv_token $SCV_TOKEN_ID
```

---

## Key Features Summary

### For Collectors:
✅ Post buy listings with price and pickup radius  
✅ Auto-match with available waste  
✅ View matched waste on map  
✅ Schedule pickups  
✅ Pay directly in SCV/USDC  

### For Recyclers:
✅ Register waste with GPS  
✅ See matching buy listings  
✅ Accept/reject matches  
✅ Receive instant payment  
✅ Confirm pickup completion  

### Matching Algorithm:
- Waste type compatibility
- Weight capacity check
- Distance validation (GPS-based)
- Price optimization (best offer)
- Expiration handling

---

## Technical Stack

**Smart Contract:**
- Rust + Soroban SDK
- Stellar Network
- Token standards (SEP-41)

**Frontend:**
- React 18 + TypeScript
- Stellar SDK
- Freighter Wallet
- Google Maps API

**Backend:**
- Node.js + Express
- Firebase (Auth/Notifications)
- Cron jobs (matching service)

---

## Database Schema (Off-chain cache)

```typescript
// Firebase collections for fast queries

// users/{userId}
{
  address: string,
  role: 'recycler' | 'collector',
  name: string,
  phone: string,
  location: { lat: number, lng: number },
  fcmToken: string,
}

// wastes/{wasteId}
{
  wasteId: string,
  recycler: string,
  wasteType: string,
  weight: number,
  location: { lat: number, lng: number },
  status: string,
  createdAt: timestamp,
}

// buyListings/{listingId}
{
  listingId: string,
  collector: string,
  wasteType: string,
  maxWeight: number,
  pricePerKg: number,
  location: { lat: number, lng: number },
  maxDistance: number,
  status: string,
  expiresAt: timestamp,
}

// matches/{matchId}
{
  matchId: string,
  wasteId: string,
  listingId: string,
  recycler: string,
  collector: string,
  status: string,
  pickupDate: timestamp,
  totalPrice: number,
}
```

---

## API Endpoints

```typescript
// Backend REST API for off-chain queries

GET    /api/wastes?type=PLASTIC&lat=6.5&lng=3.4&radius=10
GET    /api/buy-listings?type=PLASTIC&lat=6.5&lng=3.4
GET    /api/matches/:userId
POST   /api/notifications/send
GET    /api/stats/collector/:address
GET    /api/stats/recycler/:address
```

---

## Security Considerations

1. **Access Control:**
   - Only listing owner can update/cancel
   - Only match participants can accept/pay
   - Role-based function restrictions

2. **Payment Security:**
   - Verify token balance before payment
   - Atomic token transfers
   - Prevent double-payment

3. **Data Validation:**
   - GPS coordinate bounds checking
   - Weight limits (0.1kg - 10,000kg)
   - Price limits (prevent overflow)
   - Distance calculation accuracy

4. **Privacy:**
   - Exact GPS only visible to matched parties
   - Phone numbers encrypted
   - Transaction history private

---

## Performance Optimizations

1. **Caching:**
   - Cache active listings in Firebase
   - Cache waste locations for map
   - Invalidate on status changes

2. **Indexing:**
   - Geohash for location queries
   - Composite indexes for filtering
   - Pagination for large result sets

3. **Batch Operations:**
   - Batch notification sends
   - Batch match calculations
   - Lazy load map markers

---

## Monitoring & Analytics

```typescript
// Track key metrics

- Total waste registered
- Total buy listings created
- Match success rate
- Average time to match
- Average payment amount
- Active users (recyclers/collectors)
- Geographic distribution
- Waste type distribution
```

---

## Future Enhancements

**Phase 2 Features:**
- Bulk buy listings (multiple waste types)
- Recurring listings (weekly/monthly)
- Reputation system (ratings)
- Route optimization for collectors
- Price negotiation
- Escrow payments
- Multi-collector coordination
- Waste quality verification (photos)

---

## Timeline Summary

| Week | Phase | Deliverables |
|------|-------|-------------|
| 1 | Foundation | Project setup, wallet connection |
| 2 | Waste Management | Recyclers can register waste |
| 3 | Buy Listings | Collectors can create listings |
| 4 | Matching | Auto-matching working |
| 5 | Maps & Scheduling | Interactive map, pickup scheduling |
| 6 | Payment | End-to-end payment flow |
| 7 | Polish & Testing | Production-ready app |
| 8 | Launch | Public launch on mainnet |

**Total:** 8 weeks (2 months) to production launch

📅 **[View Detailed Week-by-Week Plan](./PROJECT_TIMELINE.md)**

---

## Success Metrics

- ✅ Collectors can create buy listings
- ✅ Recyclers see matching listings
- ✅ Auto-matching works within 5 seconds
- ✅ Payment completes in <30 seconds
- ✅ 90%+ match acceptance rate
- ✅ <5% cancellation rate

---

## Next Steps

1. **Week 1:** Set up Soroban project structure
2. **Day 1-2:** Implement data structures
3. **Day 3-5:** Implement core functions
4. **Day 6-7:** Implement matching logic
5. **Week 2:** Testing and refinement

**Ready to start implementation?**
