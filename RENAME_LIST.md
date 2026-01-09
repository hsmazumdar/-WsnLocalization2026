# Renaming List for IEEE Review

This document lists all suggested name changes for GUI elements, methods, and variables to replace "QUICK NOD" and related experimental naming with more intuitive, self-explanatory terminology suitable for academic presentation.

## GUI Elements (Buttons, Labels, Controls)

### Button Text Changes
| Current Display Text | Suggested Display Text | Control Name | Notes |
|---------------------|------------------------|--------------|-------|
| QUICK NOD | LOCALIZED | `btnQuickNode` | Short enough for button; refers to estimated/localized node positions |
| QUICK TEST | START | `btnQuickTest` | When running, changes to "STOP" - refers to localization algorithm |
| REF NOD | REFERENCE | `btnRefNode` | Short enough; refers to ground truth/reference node positions |
| FIT | NORMALIZE | `btnFIT` | More descriptive of what the operation does |
| Self Correct | Self-Correction | `chkSelfCorrect` | Slightly improved formatting |

### Control Name Changes (Internal)
| Current Control Name | Suggested Control Name | Description |
|---------------------|------------------------|-------------|
| `btnQuickTest` | `btnStartLocalization` | Button that starts/stops the localization algorithm |
| `btnQuickNode` | `btnLocalizedNodes` | Button to display estimated/localized node positions |
| `btnRefNode` | `btnReferenceNodes` | Button to display reference/ground truth node positions |
| `tmrQuickTest` | `tmrLocalization` | Timer that drives the iterative localization algorithm |

---

## Variables

### Private Field Variables
| Current Variable Name | Suggested Variable Name | Type | Description |
|----------------------|------------------------|------|-------------|
| `quicknod` | `localizedNodes` | `Point[]` | Array of estimated/localized node positions |
| `refnod` | `referenceNodes` | `Point[]` | Array of reference/ground truth node positions |
| `quicksno` | `localizedSequenceNumbers` | `int[]` | Sequence numbers for localized nodes |
| `refsno` | `referenceSequenceNumbers` | `int[]` | Sequence numbers for reference nodes |
| `quickrefsno` | `localizedRefMapping` | `int[]` | Mapping from localized sequence to reference sequence |
| `ref_quick` | `activeDisplayMode` | `int` | Flag indicating which node set is displayed (0=reference, 1=localized) |
| `isQuickNodDisplayed` | `isLocalizedNodesDisplayed` | `bool` | Flag indicating if localized nodes are currently displayed |
| `isQuickTestRunning` | `isLocalizationRunning` | `bool` | Flag indicating if localization algorithm is running |
| `quickTestCount` | `localizationIterationCount` | `int` | Counter for localization algorithm iterations |

---

## Methods

### Private Methods
| Current Method Name | Suggested Method Name | Description |
|-------------------|----------------------|-------------|
| `InitializeQuickNodes()` | `InitializeLocalizedNodes()` | Initializes both reference and localized node arrays |
| `NormalizeQuickNodes()` | `NormalizeLocalizedNodes()` | Normalizes localized node positions to 90% canvas occupancy |
| `btnQuickTest_Click()` | `btnStartLocalization_Click()` | Event handler for start/stop localization button |
| `btnQuickNode_Click()` | `btnLocalizedNodes_Click()` | Event handler for display localized nodes button |
| `btnRefNode_Click()` | `btnReferenceNodes_Click()` | Event handler for display reference nodes button |
| `tmrQuickTest_Tick()` | `tmrLocalization_Tick()` | Timer tick event handler for iterative localization updates |

---

## Summary by Category

### High Priority (GUI Display Text - User Visible)
1. **QUICK NOD** → **LOCALIZED** (Button text)
2. **QUICK TEST** → **START** (Button text, changes to "STOP" when running)
3. **REF NOD** → **REFERENCE** (Button text)
4. **FIT** → **NORMALIZE** (Button text)

### Medium Priority (Control Names - Internal but appears in code)
1. `btnQuickTest` → `btnStartLocalization`
2. `btnQuickNode` → `btnLocalizedNodes`
3. `btnRefNode` → `btnReferenceNodes`
4. `tmrQuickTest` → `tmrLocalization`

### High Priority (Variable Names - Core Logic)
1. `quicknod` → `localizedNodes`
2. `refnod` → `referenceNodes`
3. `quicksno` → `localizedSequenceNumbers`
4. `refsno` → `referenceSequenceNumbers`
5. `quickrefsno` → `localizedRefMapping`
6. `ref_quick` → `activeDisplayMode`

### Medium Priority (Variable Names - Supporting)
1. `isQuickNodDisplayed` → `isLocalizedNodesDisplayed`
2. `isQuickTestRunning` → `isLocalizationRunning`
3. `quickTestCount` → `localizationIterationCount`

### High Priority (Method Names)
1. `InitializeQuickNodes()` → `InitializeLocalizedNodes()`
2. `NormalizeQuickNodes()` → `NormalizeLocalizedNodes()`
3. `btnQuickTest_Click()` → `btnStartLocalization_Click()`
4. `btnQuickNode_Click()` → `btnLocalizedNodes_Click()`
5. `btnRefNode_Click()` → `btnReferenceNodes_Click()`
6. `tmrQuickTest_Tick()` → `tmrLocalization_Tick()`

---

## Terminology Rationale

### Why "Localized" instead of "Quick"?
- **"Localized"** is the standard academic term in WSN literature for nodes with estimated positions
- Clearly indicates the nodes are the result of a localization algorithm
- Contrasts meaningfully with "Reference" (ground truth) nodes

### Why "Reference" instead of "Ref"?
- More formal and explicit
- Standard terminology for ground truth data in research
- Self-explanatory to reviewers

### Why "START" instead of "QUICK TEST"?
- More intuitive button label
- Standard start/stop pattern (changes to "STOP" when running)
- The "QUICK TEST" terminology was experimental development jargon

### Why "NORMALIZE" instead of "FIT"?
- More descriptive of the actual operation (scaling and centering)
- Standard term in computer graphics and data processing

---

## Implementation Notes

1. **Button Text Length**: All suggested button texts are kept short (≤10 characters) to fit the 92-pixel button width
2. **Consistency**: All related names follow consistent patterns (e.g., `localized*` vs `reference*`)
3. **Academic Terminology**: Uses standard WSN localization terminology familiar to IEEE reviewers
4. **Self-Documentation**: Variable and method names clearly describe their purpose

---

## Files to Modify

### Source Code Files
1. `WsnMap\WsnMap.cs` - Main code file with variables, methods, and event handlers
2. `WsnMap\WsnMap.Designer.cs` - GUI control definitions and event handler wiring

### Documentation Files (Additional References Found)
1. `README.md` - Multiple references to "QUICK TEST", "QUICK NODE", "quicknod", "refnod", etc.
2. `documents\WsmLocalizationHsm.md` - Technical documentation with variable names
3. `documents\pbxCanvas_MouseDown_Workflow.md` - Workflow documentation
4. `documents\pbxCanvas_MouseDown_WorkflowHsm.md` - Workflow documentation
5. `documents\CodeAnalysisReport.md` - Code analysis references
6. `documents\LocalizationAlgorithm_Design.md` - Algorithm design documentation
7. `documents\SIGMAPS_CG_Analysis.md` - Analysis documentation

### Note on "Quick Start" Section
The "Quick Start" section header in `README.md` (line 35) should remain as-is, as it's a standard documentation term meaning "Getting Started Quickly" and is unrelated to the "QUICK NOD" terminology.

---

## Search & Replace Checklist

When implementing these changes, search for:
- `quick` (case-insensitive)
- `Quick` 
- `QUICK`
- `refnod`
- `ref_quick`
- `quicksno`
- `quickrefsno`
- `quicknod`
- All method names listed above
- All variable names listed above
- All control names listed above
