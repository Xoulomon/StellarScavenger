# Scavngr Stellar ♻️

Blockchain-powered waste pickup platform connecting recyclers and collectors on Stellar network.

## Overview

**Two-Role System:**
- **Recyclers**: Register waste with GPS location, receive payment
- **Collectors**: Post buy listings (demand), view waste locations on map, schedule pickups, pay for waste

**Marketplace Model:**
- **Supply**: Recyclers register waste (what they have)
- **Demand**: Collectors post buy listings (what they want to buy)
- **Matching**: Auto-match waste with buy listings based on type, location, price

**Payment**: Collectors pay recyclers directly in SCV Token or USDC when collecting waste.

📋 **[View Implementation Plan](./IMPLEMENTATION_PLAN.md)** - Detailed technical roadmap

---

## Key Features

- 📍 GPS location tracking for waste
- 🗺️ Interactive map for collectors
- 💰 Buy listings - collectors post demand
- 🔗 Auto-matching waste with listings
- 📅 Pickup scheduling system
- 💸 Direct payment (SCV or USDC)
- 🔔 Real-time notifications
- 📱 Mobile-first PWA

---

## Workflow

```
Recycler                          Collector
   │                                 │
   ├─ Register waste (GPS) ─────────►│
   │                                 │
   │                                 ├─ Create buy listing
   │                                 │  (waste type, price, radius)
   │                                 │
   │◄──── Auto-match ────────────────┤
   │                                 │
   ├─ View matching listings         │
   │                                 │
   ├─ Accept match ─────────────────►│
   │                                 │
   │◄──── Schedule pickup ───────────┤
   │                                 │
   │  (Pickup day arrives)           │
   │                                 │
   │◄──── Pay (SCV/USDC) ────────────┤
   │                                 │
   ├─ Confirm pickup ────────────────►│
   │                                 │
   │◄──── Confirm pickup ────────────┤
   │                                 │
   ✓ Complete                        ✓
```

---

## Smart Contract Functions

### Participant Management
```rust
register_participant(role, name, phone, address, lat, long)
```

### Waste Management (Recyclers)
```rust
register_waste(waste_type, weight, lat, long, notes) → waste_id
get_recycler_wastes(address) → Vec<Waste>
cancel_waste(waste_id)
find_matches_for_waste(waste_id) → Vec<BuyListing>
```

### Buy Listings (Collectors)
```rust
create_buy_listing(waste_type, max_weight, price_per_kg, payment_token, lat, long, max_distance_km, expires_in_days) → listing_id
update_buy_listing(listing_id, price_per_kg, max_weight, max_distance_km)
pause_buy_listing(listing_id)
cancel_buy_listing(listing_id)
get_collector_listings(address) → Vec<BuyListing>
get_active_listings(waste_type, lat, long, radius_km) → Vec<BuyListing>
find_matches_for_listing(listing_id) → Vec<Waste>
```

### Matching System
```rust
create_match(waste_id, listing_id) → match_id
accept_match(match_id)
reject_match(match_id)
get_collector_matches(address) → Vec<Match>
get_recycler_matches(address) → Vec<Match>
```

### Pickup & Payment
```rust
schedule_pickup(match_id, pickup_date, notes)
pay_for_waste(match_id)
confirm_pickup(match_id)
```

---

## Data Models

### Waste
```rust
{
  waste_id: u128,
  waste_type: WasteType,        // PAPER, PLASTIC, METAL, GLASS, PETPLASTIC
  weight: u128,                 // grams
  recycler: Address,
  latitude: i128,               // * 1_000_000
  longitude: i128,              // * 1_000_000
  status: WasteStatus,          // AVAILABLE, MATCHED, SCHEDULED, COMPLETED
  created_at: u64,
  collector: Option<Address>,
  buy_listing_id: Option<u128>,
}
```

### BuyListing
```rust
{
  listing_id: u128,
  collector: Address,
  waste_type: WasteType,
  max_weight: u128,             // grams - max capacity
  price_per_kg: u128,           // stroops (1 XLM = 10^7 stroops)
  payment_token: Address,       // SCV or USDC contract address
  latitude: i128,               // collector's location
  longitude: i128,
  max_distance_km: u32,         // pickup radius
  status: ListingStatus,        // ACTIVE, PAUSED, FILLED, EXPIRED
  created_at: u64,
  expires_at: u64,
}
```

### Match
```rust
{
  match_id: u128,
  waste_id: u128,
  listing_id: u128,
  recycler: Address,
  collector: Address,
  weight: u128,
  total_price: u128,
  status: MatchStatus,          // PENDING, ACCEPTED, SCHEDULED, PAID, COMPLETED
  created_at: u64,
  pickup_date: Option<u64>,
}
```

---

## Installation

### Prerequisites
```bash
# Rust & Soroban
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
cargo install --locked soroban-cli

# Node.js 18+
# Freighter Wallet
```

### Deploy Contract
```bash
# Configure network
soroban network add testnet \
  --rpc-url https://soroban-testnet.stellar.org:443 \
  --network-passphrase "Test SDF Network ; September 2015"

# Generate & fund account
soroban keys generate alice --network testnet
soroban keys fund alice --network testnet

# Build & deploy
cd contracts/waste-pickup
soroban contract build
soroban contract deploy \
  --wasm target/wasm32-unknown-unknown/release/waste_pickup.wasm \
  --source alice \
  --network testnet
```

### Setup Frontend
```bash
cd frontend
npm install

# Configure .env
cat > .env << EOF
VITE_STELLAR_NETWORK=testnet
VITE_CONTRACT_ID=<your-contract-id>
VITE_SCV_TOKEN_ID=<scv-token-id>
VITE_USDC_TOKEN_ID=<usdc-token-id>
VITE_MAPS_API_KEY=<google-maps-key>
EOF

npm run dev
```

---

## Payment System

### Pricing (Suggested per kg)
| Waste Type | SCV Token | USDC |
|-----------|-----------|------|
| Paper | 5-10 | $0.50-1.00 |
| Plastic | 10-15 | $1.00-1.50 |
| PET Plastic | 15-20 | $1.50-2.00 |
| Metal | 20-30 | $2.00-3.00 |
| Glass | 8-12 | $0.80-1.20 |

### Payment Flow
1. Collector collects waste
2. Collector pays recycler (SCV or USDC)
3. Payment transfers instantly
4. Both confirm pickup
5. Status → COMPLETED

---

## Tech Stack

**Blockchain:**
- Stellar Network (Soroban)
- Rust smart contracts
- SCV Token / USDC

**Frontend:**
- React 18 + TypeScript
- Vite
- Tailwind CSS
- Google Maps API
- Freighter Wallet

**Backend:**
- Node.js + Express
- Firebase (Auth/DB)
- Push notifications

---

## Project Structure

```
stellar-port/
├── contracts/
│   └── waste-pickup/
│       └── src/
│           ├── lib.rs
│           ├── waste.rs
│           ├── pickup.rs
│           └── payment.rs
│
├── frontend/
│   ├── client/
│   │   ├── components/
│   │   │   ├── WasteMap.tsx
│   │   │   ├── PaymentModal.tsx
│   │   │   └── PickupCalendar.tsx
│   │   ├── pages/
│   │   │   ├── RecyclerDashboard.tsx
│   │   │   └── CollectorDashboard.tsx
│   │   └── hooks/
│   │       ├── useGeolocation.ts
│   │       └── usePickupSchedule.ts
│   └── server/
│
└── README.md
```

---

## API Examples

### Register Waste (Recycler)
```typescript
const { location } = useGeolocation();
const wasteId = await registerWaste(
  WasteType.PLASTIC,
  5000, // 5kg in grams
  location.latitude * 1_000_000,
  location.longitude * 1_000_000,
  "Clean plastic bottles"
);
```

### Create Buy Listing (Collector)
```typescript
const listingId = await createBuyListing(
  WasteType.PLASTIC,
  100_000, // 100kg max capacity
  50, // 50 stroops per kg
  SCV_TOKEN_ADDRESS,
  collectorLat * 1_000_000,
  collectorLong * 1_000_000,
  10, // 10km pickup radius
  30  // expires in 30 days
);
```

### Find Matches for Waste (Recycler)
```typescript
const matchingListings = await findMatchesForWaste(wasteId);
// Returns buy listings that match waste type, location, and capacity
```

### Find Matches for Listing (Collector)
```typescript
const matchingWaste = await findMatchesForListing(listingId);
// Returns available waste matching listing criteria
```

### Create Match
```typescript
const matchId = await createMatch(wasteId, listingId);
// Links waste with buy listing
```

### Accept Match (Recycler)
```typescript
await acceptMatch(matchId);
// Recycler confirms they'll sell to this collector
```

### Schedule Pickup (Collector)
```typescript
const pickupDate = new Date('2026-02-15T10:00:00');
await schedulePickup(
  matchId,
  Math.floor(pickupDate.getTime() / 1000),
  "Will arrive between 10-11 AM"
);
```

### Pay for Waste (Collector)
```typescript
await payForWaste(matchId);
// Transfers tokens based on match total_price
```

### Confirm Pickup (Both)
```typescript
await confirmPickup(matchId);
// Both parties confirm completion
```

---

## Security

- ✅ Location visible only to assigned collector
- ✅ Payment required before completion
- ✅ Both parties must confirm pickup
- ✅ Blockchain transaction proof
- ✅ Encrypted data transmission

---

## Deployment

### Testnet
```bash
soroban contract deploy --wasm waste_pickup.wasm --network testnet
```

### Mainnet
```bash
soroban contract deploy --wasm waste_pickup.wasm --network mainnet
```

### Frontend
```bash
npm run build
vercel --prod
```

---

## License

MIT License

---

## Support

- GitHub: [Issues](https://github.com/your-repo/issues)
- Email: support@scavngr.com
- Discord: [Community](https://discord.gg/scavngr)

---

**Scavngr Stellar** - Efficient waste pickup coordination on blockchain ♻️
