# pbxCanvas_MouseDown Method - Workflow Analysis

## Overview
The `pbxCanvas_MouseDown` method handles mouse click events on the canvas. When a user clicks on a node, it highlights the clicked node and its neighbors within transmission range, providing visual feedback for network topology exploration.

## Method Signature
```csharp
private void pbxCanvas_MouseDown(object sender, MouseEventArgs e)
```

## Complete Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│                    START: Mouse Click Event                     │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ STEP 1: Extract Mouse Position                                  │
│   - mousePos = new Point(e.X, e.Y)                              │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ STEP 2: Identify Clicked Node                                   │
│   - no = GetNodeNumber(mousePos)                                │
│   - Returns node index or -1 if no node clicked                 │
│   - Uses refnod or quicknod based on ref_quick flag             │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ STEP 3: Highlight Clicked Node                                  │
│   - HighlightNode(no, Color.Red)                                │
│   - Draws red circle around clicked node                        │
│   - pbxCanvas.Refresh()                                         │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
                    ┌─────────┴─────────┐
                    │                   │
               no >= 0?             no < 0
                    │                   │
                    ▼                   ▼
        ┌───────────────────┐  ┌──────────────┐
        │  Continue to      │  │   END: No    │
        │  Neighbor Logic   │  │   Action     │
        └───────────────────┘  └──────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────────────────┐
│ STEP 4: Determine Neighbor Source Array                         │
│                                                                 │
│   ┌─────────────────────────────────────────────────────┐       │
│   │ IF ref_quick == 1 (Quicknod plot displayed)         │       │
│   │   ┌─────────────────────────────────────────────┐   │       │
│   │   │ IF chkUseQuickNod.Checked == true           │   │       │
│   │   │   → neighborSourceArray = quicknod          │   │       │
│   │   │   (Use quicknod neighbors in quicknod plot) │   │       │
│   │   │ ELSE                                        │   │       │
│   │   │   → neighborSourceArray = refnod            │   │       │
│   │   │   (Use refnod neighbors in quicknod plot)   │   │       │
│   │   └─────────────────────────────────────────────┘   │       │
│   │ ELSE (ref_quick == 0, Refnod plot displayed)        │       │
│   │   → neighborSourceArray = refnod                    │       │
│   │   (Always use refnod neighbors)                     │       │
│   └─────────────────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ STEP 5: Determine Current Node Array                            │
│                                                                 │
│   ┌─────────────────────────────────────────────────────┐       │
│   │ IF ref_quick == 1 AND quicknod exists               │       │
│   │   → currentNodeArray = quicknod                     │       │
│   │ ELSE IF ref_quick == 0 AND refnod exists            │       │
│   │   → currentNodeArray = refnod                       │       │
│   └─────────────────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
        ┌─────────────────────┴────────────────────────────┐
        │                                                  │
   Valid Arrays?                                      Invalid Arrays
        │                                                  │
        ▼                                                  ▼
┌─────────────────────────────┐                ┌──────────────────┐
│ Continue to Neighbor        │                │   END: Skip      │
│ Discovery                   │                │   Neighbor Logic │
└─────────────────────────────┘                └──────────────────┘
        │
        ▼
┌─────────────────────────────────────────────────────────────────┐
│ STEP 6: Calculate Transmission Distance                         │
│   - CalculateTxDistance()                                       │
│   - Updates txDistance based on numudTxRange percentage         │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ STEP 7: Find Neighbors Within Transmission Range                │
│                                                                 │
│   FOR each node j in neighborSourceArray:                       │
│     IF j != no (skip self):                                     │
│       - Calculate distance:                                     │
│        dx = neighborSourceArray[no].X - neighborSourceArray[j].X│
│        dy = neighborSourceArray[no].Y - neighborSourceArray[j].Y│
│         distance = √(dx² + dy²)                                 │
│       - IF distance <= txDistance:                              │
│           → Add j to neighbors list                             │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ STEP 8: Sort Neighbors by Distance                              │
│   - neighbors.Sort() using distance comparison                  │
│   - Ascending order (closest first)                             │
│   - Recalculates distances for sorting                          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ STEP 9: Highlight Neighbors                                     │
│   - neighborCount = Min(neighbors.Count, txRange)               │
│   - FOR k = 0 to neighborCount - 1:                             │
│       neighborId = neighbors[k]                                 │
│       HighlightNode(neighborId, Color.Brown)                    │
│   - Draws brown circles around neighbor nodes                   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ STEP 10: Refresh Canvas                                         │
│   - pbxCanvas.Refresh()                                         │
│   - Updates display with all highlights                         │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
                            END
```

## Key Components

### 1. Node Identification (`GetNodeNumber`)
- Converts mouse coordinates to node index
- Checks if click is within node's circular boundary (radius = nodSize/2)
- Returns -1 if no node was clicked

### 2. Array Selection Logic

#### Neighbor Source Array (`neighborSourceArray`)
Determines which node positions to use for neighbor distance calculations:
- **Case 1**: `ref_quick == 1` (Quicknod displayed)
  - **Sub-case 1a**: `chkUseQuickNod.Checked == true`
    - Uses `quicknod` positions
    - Shows neighbors based on quicknod's current positions
  - **Sub-case 1b**: `chkUseQuickNod.Checked == false`
    - Uses `refnod` positions
    - Shows neighbors based on reference topology (localization starting point)
- **Case 2**: `ref_quick == 0` (Refnod displayed)
  - Always uses `refnod` positions

#### Current Node Array (`currentNodeArray`)
Determines which array contains the clicked node's position:
- If `ref_quick == 1` → `quicknod`
- If `ref_quick == 0` → `refnod`

**Important Note**: The clicked node position comes from `currentNodeArray`, but neighbor calculations use `neighborSourceArray`. This allows visualizing neighbors from one topology while clicking nodes in another.

### 3. Neighbor Discovery Algorithm
1. **Distance Calculation**: Euclidean distance between clicked node and all other nodes
2. **Range Filtering**: Only nodes within `txDistance` are considered neighbors
3. **Sorting**: Neighbors sorted by distance (ascending)
4. **Limiting**: Only top `txRange` neighbors are highlighted

### 4. Visual Feedback
- **Red Circle**: Highlights the clicked node
- **Brown Circles**: Highlight neighbors within transmission range (up to `txRange`)

## Use Cases

### Use Case 1: Exploring Reference Topology
- **State**: `ref_quick == 0` (Refnod plot displayed)
- **Behavior**: Click any node → See its neighbors from reference topology
- **neighborSourceArray**: `refnod`
- **currentNodeArray**: `refnod`

### Use Case 2: Quicknod Plot - Using Quicknod Neighbors
- **State**: `ref_quick == 1`, `chkUseQuickNod.Checked == true`
- **Behavior**: Click quicknod node → See neighbors based on quicknod's current positions
- **neighborSourceArray**: `quicknod`
- **currentNodeArray**: `quicknod`

### Use Case 3: Quicknod Plot - Using Refnod Neighbors (Localization Mode)
- **State**: `ref_quick == 1`, `chkUseQuickNod.Checked == false`
- **Behavior**: Click quicknod node → See neighbors based on reference topology
- **Purpose**: Compare quicknod positions with reference topology
- **neighborSourceArray**: `refnod`
- **currentNodeArray**: `quicknod`

## Dependencies

### Methods Called
- `GetNodeNumber(Point)`: Identifies clicked node
- `HighlightNode(int, Color)`: Draws highlight circle
- `CalculateTxDistance()`: Updates transmission distance
- `pbxCanvas.Refresh()`: Updates canvas display

### Variables Used
- `refnod`: Reference node positions
- `quicknod`: Quick test node positions
- `ref_quick`: Display mode flag (0=refnod, 1=quicknod)
- `chkUseQuickNod`: Checkbox control
- `txDistance`: Transmission range in pixels
- `txRange`: Maximum number of neighbors to display
- `nodSize`: Node circle diameter

## Potential Issues & Observations

1. **Inefficient Distance Calculation**: Neighbors list is sorted by recalculating distances, even though distances were already calculated in the discovery loop.

2. **Unused Variable**: `clickedNodePos` is assigned but never used. The code uses `neighborSourceArray[no]` directly for distance calculations.

3. **Double Distance Calculation**: Distances are calculated twice:
   - Once in the neighbor discovery loop (lines 776-778)
   - Again in the sort comparison (lines 790-795)

4. **Edge Case**: If `neighborSourceArray[no]` and `currentNodeArray[no]` have different positions (when ref_quick==1 and chkUseQuickNod is false), the distance calculation uses `neighborSourceArray[no]` position, not the clicked node's visual position.

## Recommendations for Optimization

1. **Store distances during discovery**: Instead of recalculating in sort, store distance with neighbor ID.
2. **Remove unused variable**: Delete `clickedNodePos` if not needed for future enhancements.
3. **Consider using distance matrix**: If `dstmtrxval` and `dstmtrxid` are populated, use them instead of recalculating distances.

