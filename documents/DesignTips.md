# WSN Localization Visualizer - Design Tips and Recommendations

## 1. Architecture and Structure

### Core Principles
- **Use TabControl for multiple views** - Organize different visualization modes and features
- **Separate data models from visualization** - Maintain clean separation of concerns
- **Implement the localization algorithm as a separate module** - Enable reuse and testing

---

## 2. Proposed Tab Structure

### 2.1 Overview/Dashboard
- Network status summary
- Quick start controls
- Key metrics at a glance
- Real-time convergence indicator

### 2.2 Real vs Relative Maps
- **Split view**: Real (left) | Relative (right)
- **Overlay mode**: Toggle to show both on one canvas
- **Color coding**:
  - Real nodes: Green circles
  - Relative nodes: Blue circles
  - Error vectors: Red lines connecting real→relative
  - Neighbor links: Gray lines (optional)

### 2.3 Convergence Analysis
- **Real-time charts**:
  - TCE (Topological Consistency Error) vs iterations
  - Angular error vs time
  - Average node displacement
- **Statistics panel**: Current values, convergence rate
- **Export data button**

### 2.4 Routing Demonstration
- Select source/destination nodes
- Animate packet forwarding using direction vectors
- Show routing path (real vs estimated)
- Success rate statistics

### 2.5 Settings/Parameters
- **Network**: # nodes, deployment area, noise level
- **Algorithm**: Smoothing factors, SIGMAPS interval, kernel parameters
- **Visualization**: Colors, node size, animation speed
- **Save/load configurations**

### 2.6 Help/Documentation
- Navigation tree/index
- Context-sensitive help
- Algorithm explanation
- Keyboard shortcuts
- Paper reference links

---

## 3. Color Scheme

| Element | Color | Hex Code |
|---------|-------|----------|
| Real positions | Green | `#00FF00` or `#32CD32` |
| Relative positions | Blue | `#4169E1` or `#1E90FF` |
| Error vectors | Red/Orange | `#FF0000` or `#FF8C00` |
| Convergence lines | Yellow | `#FFD700` |
| Neighbor links | Light gray | `#D3D3D3` |
| Selected nodes | Magenta | `#FF00FF` |
| Background | Dark gray/Black | `#2F2F2F` or `#000000` |

---

## 4. Visualization Features

### 4.1 Dual-Map View
- Synchronized pan/zoom
- Alignment indicators
- Toggle grid/coordinates

### 4.2 Convergence Animation
- Step-by-step progression
- Speed control (1x, 2x, 5x, 10x)
- Pause at key moments
- Export frames for video

### 4.3 Metrics Display
- Live updating charts (use Chart control or ZedGraph)
- Numerical displays with color coding
- Progress bars for convergence

---

## 5. Implementation Suggestions

### 5.1 Data Models

```csharp
class Node {
    PointF RealPosition;      // Ground truth
    PointF RelativePosition;   // Estimated
    Dictionary<int, SignalKernel> Kernels;
    int NodeId;
}

class SignalKernel {
    double RSSMean;
    double RSSVariance;
    double Confidence;
}

class NetworkState {
    List<Node> Nodes;
    int Iteration;
    double TCE;
    double AngularError;
    DateTime StartTime;
}
```

### 5.2 Algorithm Implementation
- **Separate class**: `LocalizationEngine`
- **Methods**: `Initialize()`, `SelfCorrection()`, `SIGMAPS()`, `UpdateMetrics()`
- **Thread-safe** for real-time updates

### 5.3 UI Components
- **TabControl** for main structure
- **SplitContainer** for side-by-side views
- **Chart control** (System.Windows.Forms.DataVisualization)
- **Timer** for animation
- **StatusBar** for metrics

### 5.4 Help System
- **RichTextBox** with formatted text
- **TreeView** for navigation
- **Search functionality**
- **Print/export help**

---

## 6. Additional Features

### 6.1 Export Capabilities
- Save screenshots
- Export convergence data (CSV)
- Generate report (PDF/HTML)
- Video export (frame sequence)

### 6.2 Interactive Features
- Click nodes to see details
- Hover tooltips with node info
- Drag to reposition (for testing)
- Zoom with mouse wheel

### 6.3 Performance
- Efficient rendering (double buffering)
- Background calculation thread
- Progress indicators for long operations

---

## 7. Review Considerations

For paper reviewers, ensure:
- ✅ Clear visual distinction between real and relative
- ✅ Smooth, professional animations
- ✅ Clear metrics and convergence evidence
- ✅ Easy navigation and intuitive controls
- ✅ Comprehensive help for reviewers
- ✅ Export options for documentation

---

## 8. Implementation Priority

Suggested implementation order:
1. Create the TabControl structure
2. Implement the data models
3. Build the dual-map visualization
4. Add the convergence analysis charts
5. Create the help system

---

## 9. Migration Continuity: Current → SIGMAPS-CG

### 9.1 Current Implementation State

**What's Already Implemented:**
- ✅ `refnod[100]`: Reference node positions
- ✅ `allnods[100][100]`: Node local maps (each node's estimate of all positions)
- ✅ `rssmtrxval[][]` and `rssmtrxid[][]`: RSS matrices sorted by distance
- ✅ `PopulateRssMatrix()`: RSS calculation from refnod
- ✅ `kernelTable[][]`: Kernel statistics (RssMean, RssVariance, ConfidenceWeight)
- ✅ `UpdateKernel()`: EWMA-based kernel updates
- ✅ `PerformSelfCorrection()`: Self-position updates (partial implementation)
- ✅ Timer-based simulation (`tmrPacketTx_Tick`)

**Current Algorithm:**
- Uses kernel confidence weights (derived from RSS mean/variance)
- Self-correction moves node toward confidence-weighted centroid
- No scale control
- No convergence detection
- No map broadcasting

---

### 9.2 SIGMAPS-CG Migration Path

#### Phase 1: Core Algorithm Enhancement (Keep Existing, Add New)

**Keep:**
- ✅ `rssmtrxval[][]` and `rssmtrxid[][]` structure
- ✅ `PopulateRssMatrix()` method
- ✅ Self-correction concept in `PerformSelfCorrection()`
- ✅ Timer-based iteration

**Modify:**
- 🔄 Replace kernel confidence weights with direct RSS weights
- 🔄 Add displacement tracking to self-correction
- 🔄 Add convergence freeze mechanism

**Add:**
- ➕ Displacement accumulator and sliding window
- ➕ Stable flag
- ➕ RSS weight function (direct, not kernel-based)

**Code Changes:**
```csharp
// ADD: New state variables
private bool stable = false;
private double displacementSum = 0;
private Queue<double> displacementWindow = new Queue<double>();
private int tickCount = 0;

// MODIFY: PerformSelfCorrection to use direct RSS weights
private double CalculateRSSWeight(double rss) {
    // Direct RSS weight (monotonic)
    double rss_min = -100.0;
    return Math.Max(0, rss - rss_min);
}

// MODIFY: Track displacement in self-correction
private void PerformSelfCorrection(int nodeSetIdx, int nodeId) {
    // ... existing code ...
    double delta = Math.Sqrt(dx*dx + dy*dy); // Calculate displacement
    UpdateDisplacementWindow(delta); // NEW: Track displacement
    if (displacementSum < eps) stable = true; // NEW: Check convergence
}
```

---

#### Phase 2: Scale Control (New Feature)

**Add:**
- ➕ Periodic normalization (every K ticks)
- ➕ Bounding box calculation
- ➕ Scale factor computation
- ➕ Normalization application

**Code Structure:**
```csharp
// ADD: Normalization parameters
private const int K = 10; // Normalization interval
private const double thr_low = 0.90;
private const double thr_high = 1.10;
private const double W_target = 500.0; // Match canvas
private const double H_target = 400.0; // Match canvas

// ADD: Normalization method
private void NormalizeMap(int nodeSetIdx) {
    if (tickCount % K != 0 || stable) return;
    
    // Calculate bounds (use percentile-based for robustness)
    CalculateBounds(allnods[nodeSetIdx], out xmin, out xmax, out ymin, out ymax);
    
    double W = xmax - xmin;
    double H = ymax - ymin;
    
    if (W < thr_low*W_target || W > thr_high*W_target ||
        H < thr_low*H_target || H > thr_high*H_target) {
        // Apply normalization
        double sW = W_target / Math.Max(W, 0.001);
        double sH = H_target / Math.Max(H, 0.001);
        double s = Math.Min(sW, sH); // Isotropic scaling
        
        for (int j = 0; j < allnods[nodeSetIdx].Length; j++) {
            allnods[nodeSetIdx][j].X = (int)((allnods[nodeSetIdx][j].X - xmin) * s);
            allnods[nodeSetIdx][j].Y = (int)((allnods[nodeSetIdx][j].Y - ymin) * s);
        }
    }
}
```

---

#### Phase 3: Map Broadcasting (New Feature)

**Add:**
- ➕ Broadcast message structure
- ➕ Message sending mechanism
- ➕ Message receiving handler
- ➕ Passive map updates

**Code Structure:**
```csharp
// ADD: Broadcast message structure
public struct MapBroadcast {
    public int NodeId;
    public PointF SelfPosition;
    public float? ScaleFactor;
    public bool Stable;
    public long Timestamp;
}

// ADD: Broadcast method
private void BroadcastMap(int nodeId) {
    if (stable && tickCount % broadcastInterval != 0) return; // Throttle when stable
    
    MapBroadcast msg = new MapBroadcast {
        NodeId = nodeId,
        SelfPosition = new PointF(allnods[nodeId][nodeId].X, allnods[nodeId][nodeId].Y),
        Stable = stable,
        Timestamp = DateTime.Now.Ticks
    };
    
    // Send to all neighbors (simulated)
    for (int k = 0; k < rssmtrxid[nodeId].Length; k++) {
        int neighborId = rssmtrxid[nodeId][k];
        ReceiveMapBroadcast(neighborId, msg); // Simulated reception
    }
}

// ADD: Receive and update passive maps
private void ReceiveMapBroadcast(int receiverId, MapBroadcast msg) {
    // Update receiver's map with sender's self-position
    if (msg.Timestamp > lastUpdateTime[receiverId][msg.NodeId]) {
        allnods[receiverId][msg.NodeId] = new Point(
            (int)msg.SelfPosition.X, 
            (int)msg.SelfPosition.Y
        );
        lastUpdateTime[receiverId][msg.NodeId] = msg.Timestamp;
    }
}
```

---

### 9.3 Migration Checklist

#### Step 1: Prepare Data Structures
- [ ] Add `stable` flag per node
- [ ] Add `displacementWindow` queue per node
- [ ] Add `tickCount` counter
- [ ] Add `lastUpdateTime[][]` for message staleness

#### Step 2: Modify Self-Correction
- [ ] Replace kernel confidence with direct RSS weight
- [ ] Add displacement calculation
- [ ] Add displacement window update
- [ ] Add convergence check (stable flag)

#### Step 3: Add Scale Control
- [ ] Implement `CalculateBounds()` with percentile filtering
- [ ] Implement `NormalizeMap()` method
- [ ] Add periodic check (tickCount % K)
- [ ] Test scale stability

#### Step 4: Add Broadcasting
- [ ] Design message structure
- [ ] Implement `BroadcastMap()` method
- [ ] Implement `ReceiveMapBroadcast()` handler
- [ ] Add passive map updates
- [ ] Test distributed convergence

#### Step 5: Integration & Testing
- [ ] Integrate all components
- [ ] Test convergence behavior
- [ ] Tune parameters (alpha, K, eps, L)
- [ ] Verify scale stability
- [ ] Test with different network sizes

---

### 9.4 Compatibility Notes

**Backward Compatibility:**
- Existing `rssmtrxval[][]` and `rssmtrxid[][]` remain unchanged
- `PopulateRssMatrix()` continues to work as-is
- Timer mechanism (`tmrPacketTx_Tick`) remains the same

**Breaking Changes:**
- `PerformSelfCorrection()` signature may need update (add return value for displacement)
- `kernelTable` may become optional (if using direct RSS weights)
- New dependencies: Queue for displacement window

**Data Migration:**
- No data format changes required
- Existing node positions (`allnods`) remain compatible
- RSS matrices remain compatible

---

### 9.5 Parameter Migration Guide

| Current Parameter | SIGMAPS-CG Equivalent | Notes |
|------------------|----------------------|-------|
| `alpha` (EWMA) | `alpha` (self-move gain) | Same concept, different usage |
| `beta` (EWMA) | N/A | Not used in SIGMAPS-CG |
| `epsilon` (kernel) | `eps` (convergence) | Different purpose |
| N/A | `K` (normalization interval) | New parameter |
| N/A | `L` (window length) | New parameter |
| N/A | `thr_low`, `thr_high` | New parameters |
| N/A | `W_target`, `H_target` | New parameters |

**Recommended Initial Values:**
```csharp
// Keep existing
alpha = 0.1;  // Same learning rate concept

// Add new
K = 10;              // Normalize every 10 ticks
eps = 0.1;           // Convergence threshold (pixels)
L = 50;              // 50-tick window
thr_low = 0.90;      // 10% shrink tolerance
thr_high = 1.10;     // 10% expand tolerance
W_target = pbxCanvas.Width;   // Match canvas
H_target = pbxCanvas.Height;  // Match canvas
```

---

### 9.6 Testing Strategy

**Unit Tests:**
- Test RSS weight function (monotonicity)
- Test displacement window (correct sum)
- Test normalization (scale preservation)
- Test convergence detection (stable flag)

**Integration Tests:**
- Test full algorithm cycle
- Test scale stability over time
- Test convergence with different initializations
- Test broadcast message handling

**Performance Tests:**
- Measure computation time per tick
- Measure memory usage (displacement windows)
- Measure broadcast overhead
- Test with large networks (100+ nodes)

---

### 9.7 Rollback Plan

If migration issues occur:

1. **Keep kernel-based weights**: Can revert to `kernelTable` confidence weights
2. **Disable scale control**: Set K = very large number (effectively disabled)
3. **Disable broadcasting**: Comment out broadcast calls
4. **Disable convergence freeze**: Set eps = 0 (never freezes)

**Fallback Configuration:**
```csharp
// Minimal SIGMAPS-CG (core only)
bool useKernelWeights = true;  // Fallback to kernel
int K = int.MaxValue;          // Disable normalization
double eps = 0.0;              // Disable freeze
bool enableBroadcast = false;  // Disable broadcasting
```

---

### 9.8 Documentation Updates Required

- [ ] Update algorithm description in code comments
- [ ] Update `PerformSelfCorrection()` documentation
- [ ] Document new parameters
- [ ] Update design document references
- [ ] Add migration notes to README

---

### 9.9 Key Differences Summary

| Aspect | Current | SIGMAPS-CG |
|--------|---------|------------|
| **Weight Source** | Kernel confidence (RssMean/Variance) | Direct RSS weight function |
| **Scale Control** | None | Periodic normalization |
| **Convergence** | None | Displacement tracking + freeze |
| **Map Updates** | Self-only | Self + passive (from broadcasts) |
| **Broadcasting** | Not implemented | Required feature |
| **Normalization** | None | Every K ticks |

---

### 9.10 Migration Timeline Estimate

- **Phase 1 (Core)**: 2-3 days
- **Phase 2 (Scale)**: 1-2 days
- **Phase 3 (Broadcast)**: 2-3 days
- **Testing & Tuning**: 2-3 days
- **Total**: ~1-2 weeks

---

## 10. Implementation Priority (Updated)

Suggested implementation order with migration:
1. ✅ **Current**: Basic kernel-based self-correction (DONE)
2. 🔄 **Phase 1**: Enhance self-correction (RSS weights, displacement tracking)
3. ➕ **Phase 2**: Add scale control (normalization)
4. ➕ **Phase 3**: Add map broadcasting
5. ➕ **Phase 4**: Advanced visualization (TabControl, dual maps)
6. ➕ **Phase 5**: Convergence analysis and metrics