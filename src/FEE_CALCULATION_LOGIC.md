## 💰 Parking Fee Calculation Logic (Pseudocode)

---

### Goal:

Calculate fees based on vehicle type and duration of stay.

---

### Rate Chart:

| Vehicle Type | Rate/Hour |
| ------------ | --------- |
| MOTORCYCLE   | ₹10       |
| CAR          | ₹20       |
| BUS          | ₹30       |

---

### Pseudocode in JavaScript:

```js
function calculateFee(vehicleType, entryTime, exitTime) {
  const rateMap = { MOTORCYCLE: 10, CAR: 20, BUS: 30 };
  const durationMs = exitTime - entryTime;
  const hours = Math.ceil(durationMs / (1000 * 60 * 60));
  return hours * rateMap[vehicleType];
}
```

---

### Notes:

- Duration is rounded **up** to the nearest hour.
- `entryTime` and `exitTime` should be valid `Date` objects.

