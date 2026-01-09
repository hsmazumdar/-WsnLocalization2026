# WSN Auto-Mapping Application - Code Analysis Report

**Generated:** January 2026  
**Application:** WsnMap (WSN Auto-Maping)  
**Framework:** .NET Windows Forms (C#)

---

## Executive Summary

The **WsnMap** application is a visualization and testing tool designed to support the development and validation of a **Reference-Free Relative Localization Framework** for Wireless Sensor Networks (WSNs). The application provides a graphical interface for generating, displaying, and analyzing multiple node configurations and their associated sequence mappings, which are fundamental to the direction-aware routing algorithm described in the research paper.

---

## Primary Objective

The application serves as a **visualization and testing platform** for:

1. **Node Position Generation**: Creating multiple random spatial configurations of sensor nodes
2. **Sequence Mapping**: Generating randomized sequence numbers (SNo) for each node configuration
3. **Comparative Visualization**: Enabling side-by-side comparison of different node sets and their mappings
4. **Algorithm Development Support**: Facilitating the testing of localization and routing algorithms

---

## Core Functionality

### 1. **Node Generation System**

#### `PopulateNodes()` Method
- **Purpose**: Generates a random spatial distribution of sensor nodes
- **Algorithm**: 
  - Calculates optimal grid dimensions (approximately square grid)
  - Places nodes randomly within grid cells
  - Ensures nodes stay within canvas bounds with proper margins
- **Output**: Array of `Point` objects representing node positions
- **Randomization**: Uses `Random` instance seeded with current time

#### Key Features:
- Configurable number of nodes (default: 100)
- Grid-based placement with random jitter
- Boundary-aware positioning
- Canvas-adaptive sizing

### 2. **Sequence Number Generation**

#### `PopulateSNo()` Method
- **Purpose**: Creates a shuffled sequence of node indices (0 to numNodes-1)
- **Algorithm**:
  - Initializes array with sequential values 0, 1, 2, ..., numNodes-1
  - Performs extensive shuffling (100 × numNodes iterations)
  - Uses random swap operations for thorough randomization
- **Output**: Array of integers representing randomized node sequence
- **Use Case**: Provides randomized node ordering for testing routing algorithms

### 3. **Multi-Configuration Management**

#### Data Structures:
- **`refnod`**: Reference node positions (`Point[]`)
- **`allnods`**: Array of node position sets (`Point[][]`)
  - Each `allnods[i]` contains a complete set of node positions
  - Generated independently for diversity
- **`refsno`**: Reference sequence numbers (`int[]`)
- **`sno`**: Array of sequence number sets (`int[][]`)
  - Each `sno[i]` contains a shuffled sequence for the corresponding node set

#### `btnNodes_Click()` Workflow:
1. Generates reference node set (`refnod`)
2. Creates `numNodes` independent node configurations (`allnods`)
3. Generates corresponding sequence numbers for each configuration (`sno`)
4. Initializes reference sequence (`refsno`) as sequential (0, 1, 2, ...)
5. Sets maximum value for node selector control

### 4. **Visualization System**

#### `DrawNodes()` Method
- **Purpose**: Renders node positions on canvas with sequence number labels
- **Features**:
  - Draws nodes as green outlined circles
  - Displays sequence numbers (from `sno` array) as yellow text
  - Anti-aliased rendering for smooth graphics
  - Background color configurable (Black, DarkBlue, etc.)

#### Display Logic:
- Node positions determine circle placement
- Sequence numbers determine displayed labels
- This allows visualization of how node positions map to sequence indices

### 5. **Interactive Node Selection**

#### `numudEachNode_ValueChanged()` Method
- **Purpose**: Allows user to view different node configurations
- **Behavior**:
  - Value 0: Displays reference nodes (`refnod`) with reference sequence (`refsno`)
  - Value 1-N: Displays corresponding node set (`allnods[i-1]`) with its sequence (`sno[i-1]`)
- **Use Case**: Compare different random configurations and their sequence mappings

---

## Technical Architecture

### Application Structure
```
WsnMap (Windows Forms Application)
├── Main Form (WsnMap)
│   ├── Canvas Panel (pnlCanvas)
│   │   └── Picture Box (pbxCanvas) - Drawing surface
│   ├── Controls
│   │   ├── TextBox (txbxNodes) - Node count input
│   │   ├── Button (btnNodes) - Generate nodes
│   │   ├── NumericUpDown (numudEachNode) - Node set selector
│   │   └── Button (btnExit) - Close application
│   └── Event Handlers
│       ├── WsnMap_Shown - Initialization
│       ├── btnNodes_Click - Node generation
│       ├── numudEachNode_ValueChanged - View switching
│       └── numudNodeSize_ValueChanged - Size adjustment
└── Core Methods
    ├── PopulateNodes() - Position generation
    ├── PopulateSNo() - Sequence generation
    └── DrawNodes() - Visualization
```

### Data Flow
```
User Input (Node Count)
    ↓
PopulateNodes() → Point[] (Node Positions)
    ↓
btnNodes_Click()
    ├── Generate refnod (Reference)
    ├── Generate allnods[0..N-1] (Multiple sets)
    ├── Generate sno[0..N-1] (Sequences)
    └── Initialize refsno
    ↓
numudEachNode Selection
    ↓
DrawNodes(nodes, sequence, color)
    ↓
Visual Display (Canvas)
```

---

## Relationship to Research Framework

### Connection to Localization Algorithm

The application supports the **Reference-Free Relative Localization Framework** by:

1. **Initialization Testing**: 
   - Simulates the random map initialization phase (Section 3.3 of paper)
   - Each `allnods[i]` represents a different node's initial random map

2. **Sequence Mapping**:
   - The `sno` arrays represent randomized node identity mappings
   - Tests how different sequence orders affect routing decisions

3. **Multi-Perspective Visualization**:
   - `refnod` represents a reference/ground truth perspective
   - `allnods[i]` represent different nodes' local maps
   - Enables comparison of map consistency (topological alignment)

4. **Algorithm Validation**:
   - Visual verification of node distributions
   - Testing of sequence-based routing logic
   - Comparison of different random configurations

### Alignment with Paper Concepts

- **Virtual Relative Map**: Each `allnods[i]` represents a node's internal map `ℳᵢ`
- **Topological Consistency**: Visualization helps verify that different maps maintain relative relationships
- **Direction-Aware Routing**: Sequence numbers (`sno`) support testing of directional forwarding decisions
- **Random Initialization**: Multiple independent node sets test convergence from different starting states

---

## Key Design Patterns

### 1. **Separation of Concerns**
- Position generation (`PopulateNodes`) separate from visualization (`DrawNodes`)
- Sequence generation (`PopulateSNo`) independent of position generation

### 2. **Data-Driven Display**
- Display adapts to data arrays (nodes, sequences)
- No hardcoded node positions or counts

### 3. **Multi-Configuration Support**
- Single reference set + multiple test sets
- Enables comparative analysis

### 4. **Randomization Strategy**
- Extensive shuffling (100× iterations) ensures thorough randomization
- Independent random generation for each node set

---

## Current Limitations & Observations

### Identified Issues:

1. **Random Seed Problem** (Partially Addressed):
   - Uses `DateTime.Now.Millisecond` which may cause identical sequences if called rapidly
   - Should use a single `Random` instance or better seeding strategy

2. **Array Length Mismatch Risk**:
   - `DrawNodes()` uses `refnod.Length` instead of `nod.Length` parameter
   - Could cause index out of bounds if arrays differ

3. **Memory Efficiency**:
   - Stores all node sets in memory simultaneously
   - For large networks (1000+ nodes), this could be memory-intensive

4. **No Persistence**:
   - Generated configurations are not saved
   - Cannot reload or export node positions

### Strengths:

1. **Clean UI Design**: Simple, focused interface
2. **Flexible Configuration**: Easy to adjust node count
3. **Visual Feedback**: Immediate display of generated nodes
4. **Comparative Viewing**: Easy switching between configurations

---

## Potential Enhancements

### Recommended Improvements:

1. **Export Functionality**:
   - Save node positions to file (.nod format as per memory)
   - Export sequence mappings
   - Support for distance matrix export

2. **Distance Matrix Generation**:
   - Calculate and display inter-node distances
   - Support for routing algorithm testing

3. **Animation/Convergence Visualization**:
   - Show map synchronization process
   - Visualize SIGMAPS convergence

4. **Statistics Display**:
   - Node distribution statistics
   - Sequence mapping analysis
   - Topological consistency metrics

5. **RSS Simulation**:
   - Simulate signal strength between nodes
   - Visualize communication links
   - Test kernel-based self-correction

6. **Routing Visualization**:
   - Display routing paths
   - Show direction vectors
   - Test greedy forwarding

---

## Usage Workflow

### Typical User Workflow:

1. **Launch Application**: Form displays with default settings
2. **Set Node Count**: Enter desired number in `txbxNodes` (default: 100)
3. **Generate Nodes**: Click "NODES" button
   - Generates reference set + N additional sets
   - Creates corresponding sequence mappings
4. **View Configurations**: Use `numudEachNode` to switch between:
   - 0: Reference nodes with sequential sequence
   - 1-N: Different random node sets with shuffled sequences
5. **Adjust Display**: Change node size if needed
6. **Analyze**: Compare different configurations visually

---

## Conclusion

The **WsnMap** application is a well-structured visualization tool that supports the development and testing of the Reference-Free Relative Localization Framework. It provides essential functionality for:

- Generating multiple node configurations
- Creating randomized sequence mappings
- Visualizing spatial relationships
- Comparing different initialization states

The application aligns with the research objectives by enabling visual verification of the localization algorithm's behavior, particularly the initialization phase and the relationship between node positions and sequence mappings used in direction-aware routing.

**Status**: Functional core implementation with room for enhancement in persistence, analysis, and advanced visualization features.

---

## Code Statistics

- **Total Lines**: ~213
- **Methods**: 8 core methods
- **Data Structures**: 4 main arrays (refnod, allnods, refsno, sno)
- **UI Controls**: 7 (Panel, PictureBox, 2 Buttons, TextBox, NumericUpDown, 2 Labels)
- **Dependencies**: System.Drawing, System.Windows.Forms

---

*Report generated by automated code analysis*
