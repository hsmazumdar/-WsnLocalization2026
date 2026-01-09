# SIGMAPS-CG Algorithm Analysis & Implementation Guide

## Algorithm Overview

**SIGMAPS-CG** (Self-Only Relative Localization with Scale Clamp and Convergence Freeze) is a distributed, anchor-free localization algorithm that recovers relative node topology using RSS-based neighbor observations and peer broadcasts.

---

## Key Features

### 1. **Self-Only Updates**
- Each node updates ONLY `allnods[i][i]` (self-position)
- Other positions updated passively via broadcasts

### 2. **Scale Clamping**
- Periodic normalization prevents map collapse/expansion
- Maintains target map span (W_target × H_target)

### 3. **Convergence Freeze**
- Tracks cumulative displacement over sliding window
- Freezes updates when motion falls below threshold

### 4. **RSS-Weighted Centroid**
- Neighbors pull node toward weighted centroid
- Stronger RSS = higher weight = stronger pull

---

## Pros & Cons Analysis

### ✅ **PROS**

#### 1. **Robustness & Stability**
- **Scale clamping** prevents drift/collapse issues common in consensus algorithms
- **Convergence freeze** avoids unnecessary computation after convergence
- **Self-only updates** eliminate conflicting position modifications

#### 2. **Distributed & Scalable**
- No central coordinator required
- Each node operates independently
- Asynchronous operation supported

#### 3. **Anchor-Free**
- No GPS, beacons, or site survey needed
- Works in unknown environments
- Suitable for ad-hoc deployments

#### 4. **Routing-Focused**
- Recovers topological structure sufficient for routing
- Doesn't require metric precision
- Fast convergence for directional awareness

#### 5. **Practical Implementation**
- Clear, step-by-step algorithm
- Well-defined parameters
- Handles real-world issues (scale drift, convergence detection)

#### 6. **Theoretical Foundation**
- Based on consensus dynamics, not optimization
- Clear distinction from gradient descent
- Mathematically sound approach

---

### ❌ **CONS**

#### 1. **Parameter Sensitivity**
- **K (normalization interval)**: Too frequent = jittery, too rare = drift
- **alpha (learning rate)**: Too high = oscillation, too low = slow convergence
- **eps (convergence threshold)**: Too strict = never freezes, too loose = premature freeze
- **thr_low/thr_high**: Scale clamp thresholds need tuning

#### 2. **Broadcast Overhead**
- Each node broadcasts every tick
- Network traffic: O(N²) messages per tick
- May need message filtering/throttling

#### 3. **Initialization Dependency**
- Random initialization may lead to local minima
- No guarantee of global convergence
- May require multiple restarts

#### 4. **Scale Normalization Complexity**
- Periodic normalization adds computational overhead
- Bounding box calculation: O(N) per normalization
- May cause temporary map distortion during normalization

#### 5. **Convergence Window Management**
- Sliding window (L) requires memory management
- Queue operations add overhead
- Window size needs tuning

#### 6. **RSS Weight Function Design**
- Choice of w(rss) affects convergence speed
- Monotonic requirement limits flexibility
- May need domain-specific tuning

#### 7. **Asynchronous Challenges**
- Stale position updates from neighbors
- Clock synchronization not addressed
- Message ordering issues

#### 8. **No Guaranteed Convergence**
- No proof of convergence to correct topology
- May converge to incorrect local minimum
- Depends on network connectivity

---

## Implementation Challenges & Solutions

### Challenge 1: Scale Clamping Timing

**Problem**: Normalization every K ticks may cause jitter if K is too small, or drift if too large.

**Solution**:
```csharp
// Adaptive K based on displacement rate
if (displacementRate > threshold) {
    K = K_min;  // More frequent normalization
} else {
    K = K_max;  // Less frequent normalization
}
```

### Challenge 2: Convergence Detection

**Problem**: Sliding window management and threshold tuning.

**Solution**:
```csharp
// Use circular buffer for displacement window
private Queue<double> displacementWindow = new Queue<double>();
private double windowSum = 0;
private const int L = 50; // Window length

private void UpdateDisplacementWindow(double delta) {
    if (displacementWindow.Count >= L) {
        windowSum -= displacementWindow.Dequeue();
    }
    displacementWindow.Enqueue(delta);
    windowSum += delta;
    
    if (windowSum < eps && displacementWindow.Count == L) {
        stable = true;
    }
}
```

### Challenge 3: Broadcast Message Design

**Problem**: Message format and frequency.

**Solution**:
```csharp
// Compact message structure
public struct MapBroadcast {
    public int NodeId;
    public PointF SelfPosition;
    public float? ScaleFactor;      // Optional
    public bool Stable;             // Optional
    public long Timestamp;          // For staleness detection
}

// Throttle broadcasts if stable
if (!stable || (tickCount % broadcastInterval == 0)) {
    BroadcastMap();
}
```

### Challenge 4: RSS Weight Function

**Problem**: Choosing appropriate weight function.

**Solution**:
```csharp
// Option 1: Linear (simple)
private double CalculateWeight(double rss) {
    double rss_min = -100.0; // Minimum RSS threshold
    return Math.Max(0, rss - rss_min);
}

// Option 2: Exponential (more aggressive)
private double CalculateWeight(double rss) {
    double beta = 0.1; // Scaling factor
    double rss_offset = -80.0; // Shift to make positive
    return Math.Exp(beta * (rss - rss_offset));
}

// Option 3: Using kernel confidence (current implementation)
private double CalculateWeight(double rss) {
    // Use existing kernelTable confidence weight
    return kernelTable[i][k].ConfidenceWeight;
}
```

### Challenge 5: Bounding Box Calculation

**Problem**: Outliers can skew normalization.

**Solution**:
```csharp
// Use percentile-based bounds (5th-95th percentile)
private void CalculateBounds(Point[] map, out double xmin, out double xmax, 
                             out double ymin, out double ymax) {
    var xCoords = map.Select(p => (double)p.X).OrderBy(x => x).ToArray();
    var yCoords = map.Select(p => (double)p.Y).OrderBy(y => y).ToArray();
    
    int n = map.Length;
    int lowIdx = (int)(n * 0.05);  // 5th percentile
    int highIdx = (int)(n * 0.95); // 95th percentile
    
    xmin = xCoords[lowIdx];
    xmax = xCoords[highIdx];
    ymin = yCoords[lowIdx];
    ymax = yCoords[highIdx];
}
```

---

## Implementation Checklist

### Core Algorithm Steps

- [ ] **Step 0**: Initialization
  - [ ] Random map initialization
  - [ ] Displacement accumulator
  - [ ] Stable flag

- [ ] **Step 1**: RSS measurement
  - [x] Already implemented in `tmrPacketTx_Tick()`

- [ ] **Step 2**: Passive map refresh
  - [ ] Message receiver handler
  - [ ] Update `allnods[i][j]` from broadcasts

- [ ] **Step 3**: Convergence check
  - [ ] Skip motion if stable

- [ ] **Step 4**: RSS-weighted centroid
  - [x] Partially implemented in `PerformSelfCorrection()`
  - [ ] Need to use RSS directly (not kernel confidence)

- [ ] **Step 5**: Self-position update
  - [x] Implemented in `PerformSelfCorrection()`
  - [ ] Need to track displacement delta

- [ ] **Step 6**: Convergence tracking
  - [ ] Sliding window implementation
  - [ ] Cumulative displacement sum
  - [ ] Freeze when below threshold

- [ ] **Step 7**: Scale normalization
  - [ ] Periodic check (t mod K)
  - [ ] Bounding box calculation
  - [ ] Scale factor computation
  - [ ] Apply normalization to all positions

- [ ] **Step 8**: Broadcast
  - [ ] Message creation
  - [ ] Send to neighbors

### Data Structures Needed

```csharp
// Additional state variables
private bool stable = false;
private double displacementSum = 0;
private Queue<double> displacementWindow;
private int tickCount = 0;
private const int K = 10; // Normalization interval
private const double eps = 0.1; // Convergence threshold
private const int L = 50; // Window length
private const double thr_low = 0.90;
private const double thr_high = 1.10;
private const double W_target = 1.0;
private const double H_target = 1.0;
```

---

## Comparison with Current Implementation

| Feature | Current Implementation | SIGMAPS-CG |
|---------|----------------------|------------|
| **Self-updates** | ✅ Yes | ✅ Yes |
| **Kernel statistics** | ✅ EWMA tracking | ❌ Direct RSS weights |
| **Scale control** | ❌ None | ✅ Periodic normalization |
| **Convergence detection** | ❌ None | ✅ Displacement tracking |
| **Map broadcasting** | ❌ Not implemented | ✅ Required |
| **Passive updates** | ❌ Not implemented | ✅ Required |
| **Freeze mechanism** | ❌ None | ✅ Stable flag |

---

## Recommended Implementation Strategy

### Phase 1: Core Algorithm (Minimal)
1. Implement RSS-weighted centroid (replace kernel confidence)
2. Add displacement tracking
3. Implement convergence freeze
4. Test basic convergence

### Phase 2: Scale Control
1. Add bounding box calculation
2. Implement periodic normalization
3. Test scale stability

### Phase 3: Map Broadcasting
1. Design message format
2. Implement broadcast mechanism
3. Add passive map updates
4. Test distributed convergence

### Phase 4: Optimization
1. Add message throttling
2. Implement stale message filtering
3. Optimize normalization frequency
4. Fine-tune parameters

---

## Parameter Tuning Guidelines

### Initial Values (Start Here)

```csharp
alpha = 0.1;        // Conservative learning rate
K = 10;             // Normalize every 10 ticks
eps = 0.1;          // Convergence threshold (pixels)
L = 50;             // 50-tick window
thr_low = 0.90;     // 10% shrink tolerance
thr_high = 1.10;    // 10% expand tolerance
W_target = 500.0;   // Match canvas width
H_target = 400.0;   // Match canvas height
```

### Tuning Strategy

1. **Start conservative**: Low alpha, high K, loose eps
2. **Monitor convergence**: Track displacement over time
3. **Adjust incrementally**: Change one parameter at a time
4. **Test edge cases**: Sparse networks, high noise, etc.

---

## Critical Implementation Notes

### 1. **Coordinate System Consistency**
- Normalization must preserve relative positions
- Use isotropic scaling (same factor for x and y)
- Maintain aspect ratio if possible

### 2. **Message Ordering**
- Timestamp all broadcasts
- Ignore stale messages (older than threshold)
- Handle out-of-order delivery

### 3. **Numerical Stability**
- Check for division by zero (S > 0)
- Handle empty neighbor lists
- Use epsilon comparisons for floating point

### 4. **Performance Optimization**
- Cache bounding box calculations
- Batch normalization operations
- Limit broadcast frequency when stable

---

## Expected Behavior

### Convergence Timeline
- **Ticks 0-50**: Rapid initial organization
- **Ticks 50-200**: Fine-tuning and stabilization
- **Ticks 200+**: Convergence freeze (if stable)

### Success Criteria
- All nodes' maps converge to topologically consistent state
- Scale remains stable (within thresholds)
- Displacement sum falls below eps
- Maps match refnod up to rotation/translation/scale

---

## Risk Mitigation

### Risk 1: Premature Convergence Freeze
**Mitigation**: Use longer window (L) and stricter threshold (eps)

### Risk 2: Scale Drift
**Mitigation**: More frequent normalization (lower K)

### Risk 3: Oscillation
**Mitigation**: Lower alpha, add momentum damping

### Risk 4: Broadcast Overhead
**Mitigation**: Throttle broadcasts, use compression

---

## Conclusion

SIGMAPS-CG is a **well-designed, practical algorithm** with clear advantages:
- Addresses real-world issues (scale drift, convergence)
- Self-stabilizing mechanism
- Suitable for routing applications

**Main challenges**:
- Parameter tuning
- Broadcast overhead
- Implementation complexity

**Recommendation**: Implement in phases, starting with core algorithm, then adding scale control and broadcasting.
