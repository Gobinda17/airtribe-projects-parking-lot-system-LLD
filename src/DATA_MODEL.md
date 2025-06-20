## 🧱 Data Model

### MongoDB Collections Schema (Pseudocode)

---

### 1. `parking_spots`
```js
{
  _id: ObjectId,
  floor_number: Number,
  spot_number: String,
  size: String, // 'MOTORCYCLE', 'CAR', 'BUS'
  is_occupied: Boolean
}
```

---

### 2. `vehicles`
```js
{
  _id: ObjectId,
  vehicle_number: String, // Unique
  type: String // 'MOTORCYCLE', 'CAR', 'BUS'
}
```

---

### 3. `parking_sessions`
```js
{
  _id: ObjectId,
  vehicle_number: String, // FK to vehicles.vehicle_number
  spot_id: ObjectId,      // FK to parking_spots._id
  entry_time: Date,
  exit_time: Date,        // Nullable until checkout
  fee: Number             // Calculated on exit
}
```

