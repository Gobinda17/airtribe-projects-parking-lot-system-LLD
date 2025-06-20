## 🔄 Concurrency Handling (Pseudocode & Strategy)

---

### Goal:
Ensure system consistency when multiple vehicles enter or exit at the same time.

---

### Techniques:

#### 1. Document-Level Locking
Use MongoDB's atomic operations:
```js
// Attempt to allocate a spot
const allocatedSpot = db.parking_spots.findOneAndUpdate(
  { size: { $gte: vehicle.size }, is_occupied: false },
  { $set: { is_occupied: true } },
  { sort: { size: 1, floor_number: 1, spot_number: 1 }, returnDocument: "after" }
);
```

- Ensures only one thread allocates a given spot.
- Use inside a transaction if sessions span multiple collections.

---

#### 2. Retry Logic (Optimistic Locking Style)
If allocation fails:
```js
if (!allocatedSpot) {
  retryAllocate(vehicle, parkingSpots);
}
```

---

#### 3. Queued Check-In (Optional)
Use a queue system (like Kafka or RabbitMQ) for:
- High-volume entry/exit events
- FIFO processing to avoid conflict

---

#### 4. Transactions in MongoDB (Replica Set or Sharded)
```js
const session = await client.startSession();
session.startTransaction();
try {
  // allocate spot + insert parking session
  await spotsCollection.updateOne(..., { session });
  await sessionsCollection.insertOne(..., { session });
  await session.commitTransaction();
} catch (err) {
  await session.abortTransaction();
}
```

---

### Summary:
- Use `findOneAndUpdate` with proper filters
- Add retry for robustness
- For heavy load, use message queues
- Use transactions for operations spanning collections

