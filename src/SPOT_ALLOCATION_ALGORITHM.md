## 🚗 Spot Allocation Algorithm (Pseudocode)

---

### Goal:
Efficiently assign the best available parking spot to a vehicle based on size and availability.

---

### Pseudocode in JavaScript:
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
  return sizeRank(spotSize) >= sizeRank(vehicleType);
}

function sizeRank(type) {
  return { MOTORCYCLE: 1, CAR: 2, BUS: 3 }[type];
}
```

---

### Strategy:
- Select smallest suitable spot (best-fit)
- Prefer lowest floor and earliest spot number
- Mark the spot as `is_occupied = true` once assigned

