# WSN Relative Localization Algorithm - Design Specification

## Problem Statement

Design a distributed algorithm where each node independently corrects its own local map to converge toward the reference topology, using only RSS measurements and neighbor information.

---

## System Model

### Data Structures

1. **Reference Topology** (Ground Truth):
   - `refnod[100]`: Array of 100 Point positions representing the actual network topology

2. **Node Local Maps** (Per-Node Estimates):
   - `allnods[100][100]`: 2D array where `allnods[i][j]` = Node i's estimate of Node j's position
   - Initially: Each `allnods[i]` is a random permutation/incorrect map of `refnod`

3. **RSS Information** (Per-Node Neighbor Data):
   - `rssmtrxval[i][k]`: RSS value (dBm) of node i's k-th nearest neighbor
   - `rssmtrxid[i][k]`: Node ID of node i's k-th nearest neighbor
   - Sorted by distance (ascending): k=0 is closest, k=1 is second closest, etc.

4. **Kernel Statistics** (Per-Node Per-Neighbor):
   - `kernelTable[i][k]`: Contains {RssMean, RssVariance, ConfidenceWeight} for node i's k-th neighbor

---

## Algorithm: Kernel-Based Self-Correction with Map Broadcasting

### Phase 1: Initialization

**Step 1.1**: Generate reference topology
```
refnod[0..99] = PopulateNodes()  // Reference positions
```

**Step 1.2**: Initialize each node's local map
```
FOR i = 0 TO 99:
    allnods[i][0..99] = RandomPermutation(refnod)  // Each node starts with random/incorrect map
```

**Step 1.3**: Calculate RSS matrices from reference topology
```
PopulateRssMatrix():
    FOR each node i:
        Find all neighbors within txDistance
        Calculate RSS using log-normal shadowing model
        Sort neighbors by Euclidean distance (ascending)
        Store top txRange neighbors in:
            rssmtrxval[i][0..txRange-1]  // RSS values
            rssmtrxid[i][0..txRange-1]   // Neighbor IDs
```

**Step 1.4**: Initialize kernel tables
```
FOR i = 0 TO 99:
    FOR k = 0 TO rssmtrxval[i].Length-1:
        kernelTable[i][k].RssMean = rssmtrxval[i][k]
        kernelTable[i][k].RssVariance = sigma²
        kernelTable[i][k].ConfidenceWeight = CalculateConfidenceWeight(RssMean, RssVariance)
```

---

### Phase 2: Iterative Self-Correction Loop (Every Timer Tick)

**Step 2.1**: Simulate RSS measurements
```
FOR each node i = 0 TO 99:
    FOR each neighbor k = 0 TO rssmtrxval[i].Length-1:
        // Simulate receiving packet from neighbor
        baseRss = rssmtrxval[i][k]  // Reference RSS from refnod
        measurementNoise = NextGaussian(0.0, sigma_measurement)
        measuredRss = baseRss + measurementNoise
        
        // Update kernel statistics using EWMA
        UpdateKernel(i, k, measuredRss)
```

**Step 2.2**: Self-correction update (Proportional Pull Rule)
```
FOR each node i = 0 TO 99:
    // Node i adjusts ONLY its own position estimate in its map
    // It does NOT modify estimates of other nodes
    
    currentSelfPos = allnods[i][i]  // Node i's estimate of its own position
    
    // Calculate confidence-weighted centroid of neighbors
    totalWeight = 0
    weightedX = 0
    weightedY = 0
    
    FOR each neighbor k = 0 TO rssmtrxid[i].Length-1:
        neighborId = rssmtrxid[i][k]  // ID of k-th nearest neighbor
        neighborPos = allnods[i][neighborId]  // Node i's estimate of neighbor's position
        weight = kernelTable[i][k].ConfidenceWeight
        
        weightedX += weight * neighborPos.X
        weightedY += weight * neighborPos.Y
        totalWeight += weight
    
    // Move self-position toward weighted centroid
    IF totalWeight > 0:
        centroidX = weightedX / totalWeight
        centroidY = weightedY / totalWeight
        learningRate = 0.1
        
        newX = currentSelfPos.X + learningRate * (centroidX - currentSelfPos.X)
        newY = currentSelfPos.Y + learningRate * (centroidY - currentSelfPos.Y)
        
        allnods[i][i] = Point(newX, newY)  // Update ONLY self-position
```

**Step 2.3**: Map broadcasting (Optional - for SIGMAPS synchronization)
```
FOR each node i = 0 TO 99:
    // Compress and broadcast local map to neighbors
    compressedMap = CompressMap(allnods[i])
    BroadcastToNeighbors(i, compressedMap)
    
    // Receive and fuse neighbor maps (if implementing SIGMAPS)
    FOR each received map from neighbor j:
        alignedMap = AlignMap(allnods[i], receivedMap)
        FuseMap(allnods[i], alignedMap)  // Update estimates of other nodes
```

---

### Phase 3: Convergence Criteria

**Step 3.1**: Check convergence (optional monitoring)
```
FOR each node i = 0 TO 99:
    // Calculate error metric (e.g., Procrustes distance)
    error = CalculateTopologicalError(allnods[i], refnod)
    
    IF error < threshold:
        node i has converged
```

---

## Key Design Principles

### 1. **Self-Correction Only**
- Each node modifies ONLY `allnods[i][i]` (its own position estimate)
- Does NOT directly modify `allnods[i][j]` for j ≠ i
- Other nodes' positions in the map are updated only through:
  - Map broadcasting/fusion (SIGMAPS)
  - Indirect influence through self-correction

### 2. **RSS-Based Weighting**
- Confidence weights derived from RSS statistics:
  - Higher RSS mean → Higher confidence
  - Lower RSS variance → Higher confidence
  - Formula: `weight = RssMean / (RssVariance + epsilon)`

### 3. **Iterative Convergence**
- Each timer tick performs one update cycle
- Nodes gradually move toward confidence-weighted centroid
- Convergence achieved through repeated iterations

### 4. **Distributed Operation**
- Each node operates independently
- No central coordinator required
- Uses only local neighbor information

---

## Implementation Checklist

### Core Functions Required:

- [x] `PopulateRssMatrix()` - Calculate RSS from refnod
- [x] `InitializeKernelTables()` - Initialize kernel statistics
- [x] `UpdateKernel()` - Update RSS statistics using EWMA
- [x] `CalculateConfidenceWeight()` - Compute confidence from RSS stats
- [x] `PerformSelfCorrection()` - Self-position update rule
- [ ] `CompressMap()` - Compress local map for broadcasting
- [ ] `BroadcastToNeighbors()` - Send map to neighbors
- [ ] `AlignMap()` - Align received map with local coordinate frame
- [ ] `FuseMap()` - Fuse aligned map into local map
- [ ] `CalculateTopologicalError()` - Convergence metric

### Timer Tick Sequence:

```
Every 100ms:
    1. FOR each node i:
         a. Simulate RSS measurements for all neighbors
         b. Update kernel statistics (EWMA)
         c. Perform self-correction (update allnods[i][i])
         d. (Optional) Broadcast compressed map
         e. (Optional) Receive and fuse neighbor maps
```

---

## Expected Behavior

1. **Initial State**: Each `allnods[i]` is a random/incorrect map
2. **After N iterations**: Each `allnods[i]` gradually converges toward `refnod`
3. **Convergence**: All nodes' maps become topologically consistent with `refnod`
   - May differ by rotation, translation, and scale
   - But relative positions match

---

## Algorithm Pseudocode (Main Loop)

```
INITIALIZE:
    refnod = GenerateReferenceTopology()
    allnods = InitializeRandomMaps()
    rssmtrxval, rssmtrxid = CalculateRSSMatrix(refnod)
    kernelTable = InitializeKernels(rssmtrxval)

WHILE not converged:
    FOR each node i:
        // Step 1: Update RSS measurements
        FOR each neighbor k:
            measuredRss = SimulateRSSMeasurement(i, k)
            UpdateKernel(i, k, measuredRss)
        
        // Step 2: Self-correction
        centroid = CalculateWeightedCentroid(i)
        allnods[i][i] = MoveTowardCentroid(allnods[i][i], centroid, learningRate)
        
        // Step 3: (Optional) Map exchange
        BroadcastMap(i, CompressMap(allnods[i]))
        FOR each received map:
            FuseMap(i, receivedMap)
    
    CheckConvergence()
```

---

## Notes for Implementation

1. **Self-Position Update**: Only `allnods[i][i]` is modified in self-correction
2. **Neighbor Positions**: Updated indirectly through map fusion (if implemented)
3. **RSS Reference**: Always uses `rssmtrxval` calculated from `refnod` (ground truth)
4. **Convergence**: Achieved through iterative self-correction pulling nodes toward correct relative positions
5. **Topological Consistency**: Final maps match `refnod` up to rotation/translation/scale
