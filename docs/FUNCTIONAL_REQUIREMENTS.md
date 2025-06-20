## 📌 Functional Requirements for Smart Parking Lot System

---

### 1. 🅿️ Parking Spot Allocation

**Goal:** Automatically assign an available parking spot based on the vehicle's size.

**Details:**

- A vehicle can occupy a spot that is equal to or larger than its size.
- Preferential assignment to the smallest fitting spot, on the lowest floor, with the earliest available number.
- Supports vehicle types: `MOTORCYCLE`, `CAR`, `BUS`.

**Expected Behavior:**

- On vehicle entry, the system finds and reserves the best matching available spot.

---

### 2. ✅ Check-In and Check-Out

**Check-In:**

- Input: `vehicle_number`, `vehicle_type`
- Steps:
  - Register vehicle if new
  - Allocate suitable spot
  - Create parking session with entry timestamp
- Output: Assigned spot details and session ID

**Check-Out:**

- Input: `vehicle_number`
- Steps:
  - Find active session
  - Set exit timestamp
  - Calculate fee
  - Mark spot as available
- Output: Parking fee and duration

---

### 3. 💸 Parking Fee Calculation

**Goal:** Calculate parking fees based on vehicle type and duration.

**Rate Table:**

| Vehicle Type | Rate/Hour |
| ------------ | --------- |
| MOTORCYCLE   | ₹10       |
| CAR          | ₹20       |
| BUS          | ₹30       |

**Formula:**

- Total hours = `ceil(exit_time - entry_time)` in hours
- Fee = `rate[vehicle_type] * hours`

---

### 4. 📊 Real-Time Availability Update

**Goal:** Maintain and reflect accurate parking spot availability at all times.

**Mechanism:**

- On check-in: Mark spot as occupied
- On check-out: Mark spot as free
- Real-time counters per floor and vehicle size updated automatically

**Optional Endpoint:**

```json
GET /availability
{
  "floor_1": { "MOTORCYCLE": 4, "CAR": 10, "BUS": 2 },
  "floor_2": { "MOTORCYCLE": 3, "CAR": 5, "BUS": 1 }
}
```

---

