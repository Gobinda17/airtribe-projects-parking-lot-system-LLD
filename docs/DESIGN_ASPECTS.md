## 🤩 Design Aspects for Smart Parking Lot System

---

### 📃 1. Data Model (MongoDB Schema)

We will use a **NoSQL database (MongoDB)** with three core collections:

#### `parking_spots`
```js
{
  _id: ObjectId,
  floor_number: Number,
  spot_number: String,
  size: String, // 'MOTORCYCLE', 'CAR', 'BUS'
  is_occupied: Boolean
}
```

#### `vehicles`
```js
{
  _id: ObjectId,
  vehicle_number: String, // Unique
  type: String // 'MOTORCYCLE', 'CAR', 'BUS'
}
```

#### `parking_sessions`
```js
{
  _id: ObjectId,
  vehicle_number: String, // Refers to vehicles.vehicle_number
  spot_id: ObjectId, // Refers to parking_spots._id
  entry_time: Date,
  exit_time: Date, // Nullable until checkout
  fee: Number // Calculated on exit
}
```

---

### 🧠 2. Spot Allocation Algorithm

We use a **Best Fit First** approach:

#### Logic:
- A vehicle should occupy the **smallest available** spot that fits it.
- Prefer **lower floors** and **earlier spot numbers** to improve user experience.

#### Code Snippet:
```js
function allocateSpot(vehicleType, parkingSpots) {
  return parkingSpots
    .filter(spot => !spot.is_occupied && canFit(vehicleType, spot.size))
    .sort((a, b) => {
      if (a.size !== b.size) return sizeRank(a.size) - sizeRank(b.size);
      if (a.floor_number !== b.floor_number) return a.floor_number - b.floor_number;
      return a.spot_number.localeCompare(b.spot_number);
    })[0] || null;
}

function canFit(vehicleType, spotSize) {
  const rank = sizeRank;
  return rank(spotSize) >= rank(vehicleType);
}

function sizeRank(type) {
  return { MOTORCYCLE: 1, CAR: 2, BUS: 3 }[type];
}
```

---

### 💰 3. Fee Calculation Logic

Fees depend on **vehicle type** and **total time** in hours.

#### Example Rates:
| Vehicle Type | Rate per Hour |
| ------------ | ------------- |
| MOTORCYCLE   | ₹10           |
| CAR          | ₹20           |
| BUS          | ₹30           |

#### Function:
```js
function calculateFee(vehicleType, entryTime, exitTime) {
  const rateMap = { MOTORCYCLE: 10, CAR: 20, BUS: 30 };
  const durationMs = exitTime - entryTime;
  const hours = Math.ceil(durationMs / (1000 * 60 * 60));
  return hours * rateMap[vehicleType];
}
```

---

### 🟢 Check-In and Check-Out Pseudocode

#### Check-In:
```js
function checkIn(vehicle, parkingSpots, activeSessions) {
  const spot = allocateSpot(vehicle.type, parkingSpots);
  if (!spot) return { success: false, message: "No spot available" };

  spot.is_occupied = true;

  const session = {
    _id: generateUUID(),
    vehicle_number: vehicle.number,
    spot_id: spot._id,
    entry_time: new Date(),
    exit_time: null,
    fee: null
  };

  activeSessions.push(session);

  return {
    success: true,
    message: "Vehicle checked in",
    spot: {
      floor: spot.floor_number,
      number: spot.spot_number
    },
    session_id: session._id
  };
}
```

#### Check-Out:
```js
function checkOut(vehicleNumber, activeSessions, parkingSpots) {
  const session = activeSessions.find(s => s.vehicle_number === vehicleNumber && !s.exit_time);
  if (!session) return { success: false, message: "Session not found" };

  session.exit_time = new Date();
  session.fee = calculateFee(getVehicleType(vehicleNumber), session.entry_time, session.exit_time);

  const spot = parkingSpots.find(s => s._id === session.spot_id);
  if (spot) spot.is_occupied = false;

  return {
    success: true,
    message: "Checked out successfully",
    duration_mins: Math.ceil((session.exit_time - session.entry_time) / (1000 * 60)),
    fee: session.fee
  };
}
```

---

### 🧵 4. Concurrency Handling

Multiple cars can check-in/out simultaneously. To ensure consistency:

#### a. Document-Level Locking in MongoDB
- Use `findOneAndUpdate()` with filters that ensure only one spot is allocated.
- Example: mark the spot as occupied only if `is_occupied: false`

#### b. Optimistic Updates
- MongoDB supports atomic updates; use versioning if needed via middleware.

#### c. Optional Queue-Based Control
- Use a message queue like **Kafka** or **RabbitMQ** to process high-volume traffic during peak hours.

#### d. Atomic Transactions (MongoDB 4.x+)
- Use session-based transactions if working with multiple documents in replica set or sharded clusters.

---

### ✅ Summary Table

| Design Aspect        | Solution Summary                                  |
| -------------------- | ------------------------------------------------- |
| Data Model           | MongoDB collections for flexibility and scale     |
| Spot Allocation      | Best-fit by size, floor, and spot order           |
| Fee Calculation      | Rate × hours rounded up, per vehicle type         |
| Concurrency Handling | findOneAndUpdate filters, atomic ops, or queues   |

