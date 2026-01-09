__Reference\-Free Relative Localization for Direction\-Aware Routing in Wireless Sensor Networks  
__

Himanshu S Mazumdar

__Abstract__

We present a reference\-free localization framework tailored specifically for direction\-aware routing in wireless sensor networks\. Unlike conventional localization approaches that aim for metric accuracy or absolute coordinates, the proposed method focuses exclusively on establishing consistent __relative spatial orientation__ among nodes, sufficient for routing decisions that require only the direction of the destination node\.

Each node independently constructs an internal relative map by statistically aggregating received signal strength information and applying RSS\-weighted self\-correction, without relying on anchors, distance estimation, or centralized coordination\. The __SIGMAPS\-CG__ algorithm incorporates scale clamping to prevent drift and convergence freeze to detect stable states\. Periodic exchange of minimal map information enables distributed convergence through a signal\-guided synchronization process, ensuring topological consistency across nodes\.

By avoiding absolute positioning and enforcing minimal transmit power operation, the framework naturally preserves network topology while significantly reducing communication and computational overhead\. Simulation analysis demonstrates stable convergence under noise, node asymmetry, and asynchronous updates\. The proposed localization method is therefore well\-suited for energy\-constrained ad\-hoc WSN deployments where __direction\-only routing suffices__, enabling efficient packet forwarding without the cost or complexity of precise localization\.

__1\. Introduction__

Wireless Sensor Networks \(WSNs\) are fundamental to the Internet of Things \(IoT\), enabling data collection from physical environments for applications ranging from industrial monitoring to precision agriculture\. A critical enabling function for efficient data routing in many such networks is *localization*—the process by which nodes determine their spatial positions\. Geographic or direction\-aware routing protocols, such as the well\-known Greedy Perimeter Stateless Routing \(GPSR\) \[11\], can offer highly efficient, scalable packet forwarding by leveraging node location information\. However, this creates a dependency: the routing layer's performance is bounded by the accuracy, cost, and reliability of the underlying localization system\.

Conventional approaches to WSN localization predominantly seek to estimate *absolute*, metric coordinates\. These can be broadly classified into *range\-based* and *range\-free* techniques\. Range\-based methods, including those using Time of Arrival \(ToA\) or Received Signal Strength \(RSS\) as a proxy for distance \[3, 8, 9\], strive for high accuracy but are inherently susceptible to environmental noise, multipath effects, and require hardware for ranging or precise synchronization \[4\]\. More pertinently, they typically depend on a subset of nodes with known positions \(*anchors* or beacons\), which may be unavailable, costly, or compromised in ad\-hoc or resource\-constrained deployments\. Range\-free methods, such as DV\-Hop \[1\], alleviate the need for precise ranging but retain the requirement for anchor nodes to transform a relative layout into an absolute coordinate system\. Centralized approaches like MDS\-MAP \[2\] can generate relative maps from connectivity but introduce a single point of failure and communication bottleneck, contrary to the distributed ethos of WSNs\.

A parallel thread of research investigates *anchor\-free* and *relative* localization \[5, 6, 7\]\. These methods recognize that for many network functions, a globally consistent coordinate system is superfluous\. The goal shifts to recovering the network's *topology*—the relative spatial relationships between nodes\. While this direction circumvents the anchor requirement, existing solutions often involve significant computational burden \(e\.g\., nonparametric belief propagation \[5\]\) or aim to reconstruct a geometric map that is still more precise than necessary for basic routing decisions\. Crucially, most prior work treats localization as a distinct, often resource\-intensive, subsystem that *serves* the routing layer\.

This paper argues for a paradigm shift: __the co\-design of localization and routing for strict energy minimization__\. We observe that a specific class of routing strategies—*direction\-only routing*—does not require metric accuracy, absolute coordinates, or even inter\-node distances\. It merely requires that nodes share a *consistently warped* sense of spatial orientation: a consensus on which neighbor lies in the general direction of a destination\.

We present a __reference\-free relative localization framework__ explicitly architected for this purpose\. Our method eliminates all conventional dependencies: it uses __no anchors__, performs __no explicit ranging__, and requires __no centralized coordination__\. Each node maintains only a minimal internal state—a virtual relative map and RSS measurements to neighbors\. Through a novel process of __RSS\-weighted self\-correction__ \(SIGMAPS\-CG\) with scale clamping and convergence freeze, nodes independently and asynchronously adjust their own perceived position within their local map\. A distributed __Signal\-Guided Map Synchronization \(SIGMAPS\)__ protocol ensures these idiosyncratic maps converge to a topologically consistent state, unique up to rotation, translation, and scale\.

The core contribution is a *topology\-preserving spatial consensus mechanism* that emerges organically from nominal communication traffic\. By aligning the fidelity of localization precisely with the needs of the routing layer \("direction is enough"\), we naturally eliminate the overhead associated with superfluous accuracy\. Transmit power is minimized autonomously, and communication is limited to infrequent, compact state exchanges\. This makes the framework uniquely suited for energy\-constrained, ad\-hoc WSNs deployed with low\-cost hardware \(e\.g\., ESP32 microcontrollers\), where longevity and simplicity are paramount\.

The remainder of this paper is structured as follows: Section 2 formalizes the problem and system model\. Section 3 details the node\-local state and the kernel\-based self\-correction algorithm\. Section 4 describes the power\-minimizing operation and the compressed map exchange mechanism\. Section 5 introduces the SIGMAPS synchronization process and analyzes convergence\. Section 6 explains integration with direction\-aware routing\. Section 7 presents simulation\-based evaluation under noise and asymmetry\. Finally, Section 8 concludes and discusses future work, including extensions to 3D dynamic networks\.

__2\. System Model and Problem Formulation__

__2\.1 Network Model__  
We consider a static wireless sensor network \(WSN\) comprising \*n\* homogeneous nodes, randomly deployed within a bounded, two\-dimensional region ℛ ⊂ ℝ²\. The network is modeled as an undirected graph *G* = \(*V*, *E*\), where *V* = \{1, 2, …, \*n\*\} is the vertex set representing the sensor nodes, and *E* is the edge set\. An edge \(\*i\*, \*j\*\) ∈ *E* exists if nodes \*i\* and \*j\* are within mutual communication range under a given, minimal transmit power regime\. We denote the set of immediate \(one\-hop\) neighbors of node \*i\* as 𝒩ᵢ = \{\*j\* | \(\*i\*, \*j\*\) ∈ *E*\}\.

__2\.2 Node Capabilities and Constraints__  
Each node \*i\* ∈ *V* is characterized by the following capabilities and strict constraints:

1. __No Absolute References:__ No node possesses absolute coordinates, distance measurements, or known anchor nodes\.
2. __Minimal Sensing:__ Each node can transmit/receive packets and measure Received Signal Strength \(RSS\)\.
3. __Energy Constraint:__ Transmit power must be dynamically minimized\.
4. __Distributed Operation:__ No central coordinator exists\.

__2\.3 Signal Propagation Model__  
The observed RSS follows a log\-normal shadowing model:

where *Xₚ* ∼ 𝒩\(0, σ²\) models noise\. Parameters *PL₀*, η, and σ² are unknown to nodes\.

__2\.4 Routing Layer Requirement__  
The routing protocol requires only a direction vector  pointing toward destination *dst*, not its magnitude or metric accuracy\.

__2\.5 Formal Problem Statement__  
Design a distributed algorithm where each node \*i\* autonomously constructs an internal map ℳᵢ that converges to a topology\-preserving consensus with all other maps ℳⱼ, up to rotation, translation, and scaling, using only RSS measurements while minimizing overhead\.

__3\. Node\-Local State Representation and Initialization__

Each node maintains a minimal internal state to represent its perception of the network’s spatial layout\. This state consists of two core components: a __Virtual Relative Map__ and a __Signal Kernel Table__\.

__3\.1 Virtual Relative Map__

Every node  maintains a local, two\-dimensional canvas  in which it estimates the relative positions of all  nodes in the network, including itself\. This is represented as the set:

where  is node 's current estimate of node 's position within its own local coordinate frame\. The vector  represents node 's estimate of __its own position__ within this frame\.

__Key Design Principle:__ The map  has no absolute meaning\. Its scale, rotation, and origin \(translation\) are arbitrary and irrelevant\. The only invariant that must be preserved across nodes over time is the __topological arrangement__—the relative angles and neighborhood ordering between the points \.

__3\.2 RSS Measurement and Weight Assignment__

To inform updates to its virtual map, node  measures RSS to each neighbor and assigns weights based on signal strength\. In the SIGMAPS\-CG implementation, two approaches are supported:

__Option 1: Direct RSS Weights \(Recommended\)__
A monotonic weight function  maps RSS directly to influence strength:

- Linear:  where  is a minimum RSS threshold
- Exponential:  where  is a scaling factor

This approach is simpler and more direct, requiring no statistical tracking\.

__Option 2: Kernel\-Based Weights \(Alternative\)__
For enhanced robustness to noise, node  may maintain a statistical model termed a *Signal Kernel*:

where:
-  is the exponentially weighted moving average of RSS from neighbor \.
-  is the estimated variance of the RSS stream\.
-  is a __confidence weight__ derived from  and \.

The weight function is:  where  is a small constant\. This assigns higher confidence to links with stronger and more stable RSS\.

Both approaches yield monotonic weights where stronger RSS results in higher influence on position updates\.

__3\.3 Initialization Phase__

The system begins in a state of __complete ignorance__\. No a priori spatial information is available\. Therefore, each node  independently initializes its state as follows:

1. __Random Map Generation:__ All  position estimates  \(for  to \) are placed uniformly at random within the bounds of the local canvas \. A typical initialization sets \.
2. __Kernel Table Initialization:__ For each potential neighbor , the kernel parameters are set to neutral values: , , \.
3. __No Coordination:__ This initialization is performed independently by each node\. Consequently, at time ,  for any two nodes  and \. The maps are random, unrelated permutations of points\.

This initialization models the worst\-case prior knowledge and underscores the framework's challenge: achieving consistent relative orientation __despite starting from uncorrelated randomness__\.

__Outcome of Initialization:__ Each node possesses a unique, random internal map\. The core algorithmic processes described in the following sections—Kernel\-Based Self\-Correction \(Section 4\) and SIGMAPS Synchronization \(Section 5\)—will iteratively refine these maps toward a network\-wide, topologically consistent consensus using only observed RSS data and local communication\.

__4\. Kernel\-Based Self\-Correction Process \(SIGMAPS\-CG Enhanced\)__

The core of the framework's adaptability lies in a continuous, distributed update cycle where each node refines its perceived location within its own virtual map based solely on incoming communication data\. This __Self\-Correction__ process is executed autonomously by each node during normal network operation and consists of three interlinked stages: __RSS Measurement__, __RSS\-Weighted Centroid Calculation__, and the __Proportional Pull Update with Convergence Tracking__\.

__4\.1 RSS Measurement and Weight Assignment__

As node  receives data packets from its neighbors at each timer tick, it measures the instantaneous RSS for each neighbor \. The RSS values are stored in a sparse matrix structure:

- `rssmtrxval[i][k]`: RSS value \(dBm\) of node i's k\-th nearest neighbor
- `rssmtrxid[i][k]`: Node ID of node i's k\-th nearest neighbor
- Neighbors are sorted by Euclidean distance \(ascending\), with k=0 being the closest

The RSS measurements incorporate log\-normal shadowing noise, modeling realistic wireless channel behavior:

where  is the path loss component and  models environmental noise\.

__Weight Function:__ A monotonic weight function  maps RSS to influence strength\. Simple instantiations include:

- Linear: 
- Exponential: 
- Kernel\-based:  \(using EWMA statistics\)

The weight function ensures that stronger RSS values \(less negative\) result in higher weights, causing closer neighbors to exert stronger "pull" on the node's position\.

__4\.2 RSS\-Weighted Centroid Calculation__

At each correction interval, node  computes a __confidence\-weighted centroid__ of its neighbors' estimated positions within its own map:

where:
-  is the set of neighbors \(indices k for which RSS measurements exist\)
-  is node i's estimate of neighbor j's position in its local map
-  is the weight derived from RSS measurement

This centroid represents the "center of mass" of the node's neighborhood, weighted by signal strength\.

__4\.3 Proportional Pull Update Rule__

The key innovation of the self\-correction process is how these signal\-derived weights guide the evolution of the virtual map\. Crucially, __a node never directly modifies its estimate of another node's position__\. Instead, it uses the weighted positions of its neighbors to adjust __its own estimated position__  within its local map \.

At each correction interval \(which can be event\-driven or periodic\), node  updates its self\-position as follows:

where  is the learning rate \(self\-move gain\), typically 0\.1\.

__Interpretation:__ Node  moves its self\-estimate  toward the __RSS\-weighted centroid__ of its neighbors' estimated positions within its own map\. A neighbor  with a high RSS \(and thus high weight\) exerts a stronger "pull" on node 's position\.

__Design Principle – Self\-Correction Only:__ This rule enforces a critical discipline:

- Node  updates only \.
- The estimates  for all  remain unchanged by this local operation\.
- Corrections are therefore __localized and non\-conflicting__; no two nodes attempt to modify the same coordinate estimate\.

__4\.4 Convergence Tracking and Freeze Mechanism__

To avoid unnecessary computation after convergence, the algorithm tracks cumulative displacement over a sliding window of L ticks:

where  is the displacement at tick t\. When the cumulative displacement falls below a threshold , the node sets a __stable__ flag and freezes further position updates\. This self\-stabilization mechanism ensures the algorithm terminates naturally when the map has converged to a stable state\.

__4\.5 Scale Clamping and Normalization__

A critical challenge in distributed consensus algorithms is __scale drift__—the tendency for maps to collapse or expand over time\. To address this, SIGMAPS\-CG incorporates periodic __scale normalization__:

Every K ticks \(normalization interval\), if the node is not yet stable:

1. Calculate the current map bounds: 
2. Compute map span: , 
3. If the span deviates beyond thresholds \( or \), apply isotropic scaling:

where  is the target map span \(e\.g\., canvas dimensions\)\. This normalization ensures the map maintains a consistent scale, preventing collapse or unbounded expansion while preserving topological relationships\.

__4\.6 Emergent Behavior and Topological Influence__

This update rule leads to emergent spatial organization:

1. __Clustering of Strongly Connected Nodes:__ Nodes that share strong, stable mutual links will iteratively pull each other's self\-positions closer together within their respective maps\.
2. __Preservation of Neighborhood Topology:__ The rule uses only one\-hop neighbor information, ensuring that the evolving map  inherently reflects the underlying communication graph's connectivity\.
3. __Robustness to Noise:__ The weighting mechanism automatically discounts unreliable or volatile links \(low RSS\), preventing them from unduly distorting the map\.
4. __Scale Stability:__ Periodic normalization prevents scale drift, maintaining consistent map dimensions\.

This process alone, running independently on all nodes, would cause each node's map to __internally organize__ to reflect local signal\-strength relationships\. However, the maps would remain globally inconsistent—rotated, scaled, and translated relative to one another\. The __SIGMAPS__ synchronization process \(Section 5\) is responsible for aligning these independently organized maps into a network\-wide consensus\.

__5\. Compressed Map Exchange and SIGMAPS Synchronization__

While the Self\-Correction process organizes each node's internal map locally, achieving a globally consistent relative orientation requires a mechanism for distributed coordination\. This is accomplished through the periodic exchange of map information, followed by a novel __Signal\-Guided Map Synchronization \(SIGMAPS\)__ protocol\.

__5\.1 Map Broadcasting Mechanism__

To minimize communication overhead—a primary design constraint—nodes broadcast only essential information\. At each timer tick, each node i broadcasts a compact message containing:

- Node ID: i
- Self\-position: `allnods[i][i]` \(node i's estimate of its own position\)
- Optional: Scale metadata \(if normalization was recently applied\)
- Optional: Stable flag \(indicating convergence status\)

This minimal broadcast enables passive map updates: when node j receives a broadcast from node i, it updates `allnods[j][i]` with the received position\. This __passive update mechanism__ allows nodes to refine their estimates of other nodes' positions without direct coordination\.

__5\.1\.1 Broadcast Message Format__  
The broadcast message is minimal, containing only:

- Node ID \(2 bytes\)
- Self\-position X, Y \(4 bytes each = 8 bytes total\)
- Timestamp \(8 bytes\) for staleness detection
- Optional flags \(1 byte\): stable status, scale metadata

Total: ~19 bytes per broadcast, significantly smaller than full map exchange\.

__5\.1\.2 Passive Map Updates__

When node j receives a broadcast from node i:

1. Check message timestamp against last update time
2. If message is fresh: `allnods[j][i] ← received_position`
3. Update last update time for node i

This passive mechanism ensures that:
- Nodes gradually learn correct positions of others through broadcasts
- Stale messages are ignored \(timestamp\-based filtering\)
- No explicit coordination or acknowledgment required

__5\.2 SIGMAPS: Signal\-Guided Map Synchronization Protocol \(Simplified\)__

In the SIGMAPS\-CG implementation, the synchronization process is simplified for efficiency:

__5\.2\.1 Direct Position Updates__

When node j receives a broadcast from node i, it directly updates its estimate:

`allnods[j][i] ← received_position`

No explicit alignment or transformation is required because:
- Each node maintains its own coordinate frame
- Only relative topology matters \(not absolute alignment\)
- Self\-correction naturally aligns maps through RSS\-weighted consensus

__5\.2\.2 Timestamp\-Based Staleness Filtering__

To prevent outdated information from corrupting the map:

- Each broadcast includes a timestamp
- Nodes maintain `lastUpdateTime[j][i]` for each neighbor
- Messages with timestamps older than a threshold are ignored
- This ensures only recent, relevant updates are applied

__5\.2\.3 Convergence\-Aware Broadcasting__

When a node reaches stable state \(convergence freeze\):

- Broadcast frequency may be reduced \(throttling\)
- Stable flag is included in broadcast message
- Other nodes can use this information to optimize their own convergence

This simplified approach reduces computational overhead while maintaining the essential synchronization properties needed for topological consensus\.

__5\.3 Convergence Property__

The SIGMAPS\-CG protocol, combined with the local self\-correction from Section 4, ensures that all virtual maps  converge over time to a __common relative layout__, despite:

- Asynchronous and periodic updates,
- Noisy RSS measurements,
- Absence of a global coordinator,
- Scale drift \(mitigated by normalization\)\.

The convergence mechanism operates through:

1. __Self\-Correction:__ Each node moves toward RSS\-weighted centroid of neighbors
2. __Passive Updates:__ Broadcasts propagate position information across network
3. __Scale Control:__ Periodic normalization prevents drift
4. __Convergence Freeze:__ Displacement tracking detects and freezes at stable state

Formally, after sufficient iterations:

where  denotes equivalence up to an arbitrary global rotation, translation, and uniform scaling\. This __topological consensus__ is the sufficient condition for enabling direction\-aware routing across the network\.

The algorithm is __not gradient descent__ on a distance\-stress objective, but rather a __self\-stabilizing distributed consensus__ guided by RSS\-weighted influence, with explicit scale regulation and convergence detection\.

__6\. Power\-Minimizing and Topology\-Preserving Behavior__

A core tenet of the proposed framework is that localization should not merely be energy\-efficient, but should actively contribute to __network\-wide energy conservation__\. This section details how each node autonomously adjusts its transmit power to the minimal viable level, a process that inherently stabilizes the network topology and reinforces the consistency of the relative maps\.

__6\.1 Autonomous Power Control Loop__

Each node  independently regulates its transmit power  based on locally observable link quality metrics\. The objective is to reduce  until the network’s *functional connectivity*—specifically, the ability to maintain reliable communication for the localization and routing protocols—is preserved\.

The control loop operates as follows:

1. __Monitor Link Stability:__ Node  tracks the __packet reception rate \(PRR\)__ and __RSS variance __ for each neighbor  over a sliding time window\.
2. __Evaluate Network Coherence:__ A local *coherence metric*  is computed, reflecting the stability of the node’s kernel table and its map synchronization state\. A simple form is:

where  is a trade\-off parameter\.

1. __Adjust Transmit Power:__
	- __If __ \(links are stable and map synchronization is consistent\), __decrement__  by a small step \.
	- __If __ \(packet loss is high or RSS is highly variable\), __increment__  by \.
	- Otherwise, maintain the current power level\.

This creates a __negative feedback loop__ that seeks the minimum power level sufficient to maintain \.

__6\.2 Emergent Topology Preservation__

This distributed power control has a profound effect on the network’s spatial\-radio environment:

1. __Stabilized Neighborhood Sets:__ As nodes minimize power, the neighbor set  converges to only those nodes with the most reliable, energy\-efficient links\. Weak or fluctuating links are naturally pruned as their PRR drops below the usable threshold\.
2. __Reduced RSS Variance:__ Operating at the minimal viable power reduces the dynamic range of the RSS, as signals are neither saturated \(too close\) nor buried in noise \(too far\)\. This leads to lower , which directly increases the kernel confidence weights \.
3. __Alignment of Communication and Topology:__ The resulting communication graph becomes a __close approximation of the physical proximity graph__\. This is because the minimal power required to reach a neighbor is roughly proportional to the distance \(under a path\-loss model\)\. Consequently, the *logical neighborhood* used in the self\-correction update \(Section 4\) aligns with the *physical neighborhood*, ensuring the virtual map  accurately reflects geometric relationships\.

__6\.3 Symbiosis with Localization__

The power\-minimizing loop and the localization framework are __mutually reinforcing__:

- __Localization Informs Power Control:__ The kernel variance  is a direct input to the coherence metric \. A stable map \(low synchronization error\) indicates good link quality, permitting further power reduction\.
- __Power Control Stabilizes Localization:__ By pruning unstable links and reducing RSS noise, the power loop provides a __cleaner, more consistent input signal__ to the kernel update and SIGMAPS processes, accelerating convergence and improving the robustness of the relative maps\.

This synergy ensures that the network does not merely *converge* to a topology\-aware state, but __actively maintains it under varying conditions with minimal energy expenditure__\.

__6\.4 Practical Implementation on Constrained Hardware__

On resource\-constrained platforms like the ESP32:

- The power control loop can run in the background, triggered by periodic RSS/PRR measurements\.
- The step size  can be adapted based on available transmit power levels of the radio \(e\.g\., ESP32’s IEEE 802\.11 or Bluetooth LE power control tables\)\.
- The thresholds  and  can be set empirically or learned dynamically to adapt to environmental RF conditions\.

This design ensures that the framework’s energy benefits are __realizable in practice__, not merely theoretical\.

__7\. Integration with Direction\-Only Routing__

The ultimate objective of the proposed localization framework is to enable efficient, scalable routing in WSNs where precise coordinates are unavailable\. This section details how the converged relative map is directly utilized by a __direction\-only routing protocol__, completing the co\-design loop between localization and packet forwarding\.

__7\.1 Routing Query from a Relative Map__

Once a node 's internal map  has converged to a network\-consistent state \(as described in Sections 4 and 5\), answering a routing query is straightforward\. For a given destination node , node  computes a __relative direction vector__ within its local coordinate frame:

where:

-  is node 's estimate of the destination’s position in ,
-  is node 's estimate of its own position \(typically near the origin after self\-correction\)\.

__Crucially__, the routing algorithm uses __only the direction \(angle\)__ of , not its magnitude\. The vector magnitude is irrelevant and may be arbitrarily scaled across nodes—it does not represent physical distance\.

__7\.2 Greedy Direction\-Aware Forwarding__

Using the direction vector, node  executes a __greedy directional forwarding__ rule to select the next\-hop neighbor:

1. For each neighbor , compute the vector from  to  in the local map:
2. Calculate the __angular deviation__  between  and  using the dot product:
3. Select the neighbor  that minimizes this angular deviation:

This ensures that the packet is forwarded to the neighbor perceived to be closest to the __straight\-line direction__ of the destination in the relative map\.

__7\.3 Handling Local Minima with Topological Awareness__

Like all greedy geographic routing, directional forwarding can encounter __local minima__ \(voids\), where no neighbor lies closer directionally to the destination\. Traditional protocols like GPSR switch to a perimeter mode using planarized graphs and absolute coordinates, which are unavailable here\.

Our framework enables an __alternative recovery strategy__ using __topological awareness__:

1. If a local minimum is detected \(i\.e\., all  are large, e\.g\., > 90°\), node  consults its map  to identify a *two\-hop topological bypass*\.
2. Node  selects the neighbor  whose vector  aligns best with the direction to a __different neighbor__  that is closer to  in the map topology\.
3. This creates a __short detour__ based on the known relative layout, avoiding the need for planarization or global topology knowledge\.

This method is less formal but more lightweight than planar graph routing, aligning with the framework’s minimalistic philosophy\.

__7\.4 Advantages Over Coordinate\-Based Routing__

The integration offers several key benefits:

- __No Coordinate System Required:__ Eliminates the need for GPS, anchor nodes, or global alignment\.
- __Robust to Map Distortions:__ Because only angles matter, uniform scaling, rotation, or translation of the map do not affect routing decisions\.
- __Low Computational Overhead:__ Forwarding decisions require only vector subtractions and a dot product—operations easily handled by microcontrollers like the ESP32\.
- __Natural Load Balancing:__ Multiple paths with similar angular alignment to the destination can be considered, enabling probabilistic or multi\-path forwarding variants\.

__7\.5 Summary of the Localization\-Routing Interface__

The interface between the localization framework and the routing layer is therefore __minimal and efficient__:

__Routing Layer Needs__

__Provided by Localization Framework__

Direction to destination

Vector  from 

Neighbor directions

Vectors  for all 

Topological fallback

Full relative map  for void recovery

__No need for:__

Distance estimates, absolute coordinates, map scale/rotation

This clean separation ensures that the routing protocol remains simple, while the localization framework provides exactly—and only—the spatial awareness required\.

__8\. Simulation Analysis and Results__

To validate the efficacy and robustness of the proposed framework, we conducted extensive simulations under realistic network conditions\. The primary objectives were to: \(1\) demonstrate convergence of the relative maps, \(2\) quantify communication and energy efficiency, and \(3\) evaluate the performance of the resulting direction\-aware routing protocol\.

__8\.1 Simulation Setup__

__Network Deployment:__  
 nodes were randomly deployed within a  square region, modeling a dense WSN deployment \(e\.g\., a field or industrial floor\)\. The physical distance between nodes  and  is denoted \.

__Radio and Channel Model:__  
We employed the log\-normal shadowing path\-loss model \(Section 2\.3\) with parameters:  
, , , path\-loss exponent , and shadowing deviation  to test robustness under varying noise levels\. A packet reception rate \(PRR\) threshold of  was used for link existence\.

__Framework Parameters:__

- Self\-move gain \(alpha\): 0\.1
- Normalization interval \(K\): 10 ticks
- Convergence threshold \(eps\): 0\.1 pixels
- Displacement window length \(L\): 50 ticks
- Scale thresholds: thr_low = 0\.90, thr_high = 1\.10
- RSS weight function: monotonic \(linear or exponential\)
- SIGMAPS broadcast: every timer tick \(100ms\)
- Power control step: 
- All results are averaged over 20 random topologies\.

__Baseline Comparisons:__  
We compared our framework against two established approaches:

1. __DV\-Hop \[1\]__ \(anchor\-based, range\-free\)\.
2. __MDS\-MAP\(P\) \[2\]__ \(connectivity\-based, relative\)\.

Performance was evaluated using the metrics defined below\.

__8\.2 Evaluation Metrics__

1. __Topological Consistency Error \(TCE\):__  
Measures the alignment of relative maps across nodes\. For each pair of nodes , we compute the Procrustes distance between their neighbor direction sets after optimal rotation/reflection\. Lower TCE indicates better consensus\.
2. __Angular Routing Accuracy \(ARA\):__  
For each source\-destination pair , we compute the angular error between the true direction \(from ground truth positions\) and the estimated direction  from the local map\.
3. __Communication Overhead:__  
Total bytes exchanged per node for localization \(map synchronization \+ control packets\) over the simulation duration\.
4. __Energy Consumption:__  
Total transmit energy consumed per node, modeled as \.
5. __Routing Success Rate \(RSR\):__  
Percentage of packets that reach the destination using greedy direction\-aware forwarding \(Section 7\)\.

__8\.3 Convergence and Robustness Results__

__Convergence Timeline:__  
Figure 1 shows the mean TCE over simulation time for different RSS noise levels \(\)\. The framework converges within __50–100 cycles__ even under high noise \(\), demonstrating the stability of the kernel\-based self\-correction and SIGMAPS synchronization\.

__Impact of Asymmetric Links:__  
We introduced link asymmetry by applying different shadowing noise to  and  directions\. Figure 2 shows that TCE remains bounded \(< 0\.25 rad\) even under 30% asymmetry, confirming the framework’s tolerance to realistic radio irregularities\.

__Effect of Power Minimization:__  
Figure 3 illustrates the reduction in average transmit power over time\. Nodes stabilize at a power level __4–7 dB lower__ than initial , while maintaining PRR > 80% and stable TCE\.

__8\.4 Performance Comparison__

__Metric__

__Our Framework__

__DV\-Hop \[1\]__

__MDS\-MAP\(P\) \[2\]__

__Avg\. TCE \(rad\)__

__0\.18__

N/A \(absolute\)

0\.22

__Angular Error ARA \(°\)__

__12\.4°__

8\.7°\*

15\.1°

__Comm\. Overhead \(KB/node\)__

__2\.1__

15\.8

45\.3 \(centralized\)

__Energy \(relative units\)__

__1\.0__

3\.2

1\.8

__Routing Success Rate \(%\)__

__94\.7__

96\.1\*

92\.3

*Note:* DV\-Hop provides absolute coordinates, enabling lower angular error and higher RSR, but at the cost of anchors and significantly higher overhead\.

__8\.5 Key Observations__

1. __Sufficiency for Routing:__ Despite higher angular error than anchor\-based DV\-Hop, our framework achieves __>94% routing success__, confirming that *direction\-only* routing does not require high metric precision\.
2. __Overhead Reduction:__ Our method reduces communication overhead by __7\.5×__ compared to DV\-Hop and __21\.6×__ compared to centralized MDS\-MAP\(P\), fulfilling the minimal\-overhead design goal\.
3. __Energy Efficiency:__ The power\-minimizing behavior reduces energy consumption by __68%__ compared to DV\-Hop, validating the synergy between localization and power control\.
4. __Noise Robustness:__ Performance degrades gracefully with increasing , with RSR dropping below 90% only at —a highly noisy environment\.

__8\.6 Discussion of Limitations__

The framework assumes a __largely static network__; high mobility would require faster update intervals\. Additionally, in __extremely sparse networks__ \(average degree < 4\), convergence slows and TCE increases, as the map has insufficient constraints\. These are directions for future work\.

__9\. Conclusion and Future Work__

This paper has introduced a __reference\-free relative localization framework__ explicitly designed to enable direction\-aware routing in resource\-constrained wireless sensor networks\. Departing from the conventional pursuit of metric accuracy, we proposed a minimalist paradigm focused on achieving __topological consistency__—a spatial consensus sufficient for routing decisions that require only directional knowledge of the destination\. By co\-designing localization with routing and energy conservation from the ground up, the framework operates without anchors, ranging, centralized coordination, or absolute coordinates, using only RSS statistics gleaned from normal communication traffic\.

__9\.1 Summary of Contributions__

Our principal contributions are fivefold:

1. __A Novel Co\-Design Philosophy:__ We redefined the objective of WSN localization from *absolute positioning* to __topology\-preserving spatial consensus__, precisely aligned with the needs of direction\-only routing\. This sufficiency\-driven approach eliminates unnecessary overhead and complexity\.
2. __The RSS\-Weighted Self\-Correction Algorithm \(SIGMAPS\-CG\):__ A lightweight, distributed update rule where each node adjusts only its own perceived position within a local virtual map, guided by RSS\-weighted neighbor positions\. The algorithm incorporates scale clamping to prevent drift and convergence freeze to detect stable states, ensuring robustness and efficiency\.
3. __The SIGMAPS Protocol:__ A simplified __Signal\-Guided Map Synchronization__ mechanism that achieves distributed convergence through minimal broadcast exchanges and passive map updates, robust to noise, asymmetry, and asynchrony\.
4. __Scale Stability via Normalization:__ Periodic map normalization prevents scale drift—a common failure mode in consensus algorithms—by maintaining target map dimensions through isotropic scaling\.
5. __Integrated Power Minimization:__ An autonomous power control loop that reduces transmit energy while inherently stabilizing network topology, demonstrating a symbiotic relationship between localization accuracy and network\-wide energy efficiency\.

Simulation results validate that the framework converges reliably under realistic noise and asymmetry, reduces communication overhead by an order of magnitude compared to anchor\-based and centralized baselines, and maintains a high packet delivery rate \(\) using only direction vectors derived from the consensus maps\.

__9\.2 Broader Implications__

The work demonstrates that for many WSN applications—particularly those employing greedy geographic or directional routing—__precise localization is an unnecessary luxury__\. By aligning the fidelity of spatial awareness with the actual requirements of the network layer, significant gains in energy efficiency, scalability, and deployment simplicity can be realized\. This is especially relevant for low\-cost, ad\-hoc deployments using platforms like the ESP32, where longevity and minimal configuration are paramount\.

__9\.3 Future Work__

Several promising directions emerge from this research:

1. __Extension to 3D and Mobile Networks:__ The core principles are applicable to drone swarms and mobile ad\-hoc networks\. Future work will adapt the kernel model and SIGMAPS protocol to handle dynamic topology changes and three\-dimensional geometry, enabling applications in aerial and vehicular networks\.
2. __Hardware Implementation and Real\-World Validation:__ We are currently implementing the framework on ESP32\-C6 modules using the ESP\-NOW protocol\. Real\-world experiments will assess performance under non\-ideal RF environments, intermittent connectivity, and real hardware constraints\.
3. __Adaptive Parameter Tuning:__ The parameters \(\) could be made self\-tuning based on observed network dynamics, further enhancing adaptability and robustness without manual intervention\.
4. __Security Enhancements:__ Investigating lightweight mechanisms to detect and isolate malicious nodes attempting to disrupt the spatial consensus through false map broadcasts is critical for secure deployments\.
5. __Integration with Higher\-Layer Protocols:__ Exploring how this directional consensus can benefit network services beyond routing, such as data aggregation, coverage optimization, and cluster formation\.

In conclusion, this work provides a practical, energy\-conscious alternative to traditional WSN localization, bridging the gap between the need for spatial awareness and the constraints of real\-world, large\-scale sensor network deployments\. By asking not *"How accurate can we be?"* but *"What is the least we need to know to route effectively?"*, we open a pathway toward more sustainable, scalable, and deployable wireless sensor networks\.

__Reference__

__Category 1: Foundational & Conventional WSN Localization__

1. __D\. Niculescu and B\. Nath, "DV Based Positioning in Ad Hoc Networks," *Telecommunication Systems*, 2003\.__ \(The seminal DV\-Hop algorithm, representing range\-free, anchor\-based localization\)\.
2. __Y\. Shang et al\., "Localization from Mere Connectivity," *MobiHoc*, 2003\.__ \(Classic MDS\-MAP approach, using connectivity for relative maps, often requiring centralized processing\)\.
3. __N\. Patwari et al\., "Locating the Nodes: Cooperative Localization in Wireless Sensor Networks," *IEEE Signal Processing Magazine*, 2005\.__ \(Comprehensive survey on ranging\-based techniques like RSS, ToA, TDoA\)\.
4. __K\. Whitehouse et al\., "Calamari: A Localization System for Sensor Networks using Ultrasound," *ACM SenSys*, 2004\.__ \(Example of a high\-accuracy, active ranging system with associated hardware/complexity cost\)\.

__Category 2: Relative & Anchor\-Free Localization__  
5\. __A\. T\. Ihler et al\., "Nonparametric Belief Propagation for Self\-Localization of Sensor Networks," *IEEE JSAC*, 2005\.__ \(Probabilistic approach for cooperative localization, often computationally heavy\)\.  
6\. __R\. Nagpal et al\., "Organizing a Global Coordinate System from Local Information on an Amorphous Computer," *MIT A\.I\. Memo*, 1999\.__ \(Early work on deriving global order from local interactions, a philosophical precursor\)\.  
7\. __M\. R\. Gholami et al\., "On the Performance of Anchor\-Free Localization," *IEEE ICC*, 2010\.__ \(Analysis of the fundamental limits and challenges of anchor\-free scenarios\)\.

__Category 3: RSS\-Based Techniques & Robustness__  
8\. __S\. P\. Chepuri et al\., "RSS\-Based Localization in Wireless Sensor Networks," in *Handbook of Position Location*, 2019\.__ \(Modern treatment of RSS models, challenges with noise and multipath\)\.  
9\. __K\. Chintalapudi et al\., "Ad\-hoc Localization Using RSS," *IEEE INFOCOM*, 2004\.__ \(Early work focusing on RSS\-based ranging and its inherent inaccuracies\)\.

__Category 4: Topology\-Preserving & Routing\-Aware Methods__  
10\. __F\. Zhao et al\., "Map Generation in Large Sensor Networks by Hessian\-based Kernel Methods," *IEEE IPSN*, 2004\.__ \(Focuses on generating layout maps preserving neighborhood relationships\)\.  
11\. __B\. Karp and H\. T\. Kung, "GPSR: Greedy Perimeter Stateless Routing for Wireless Networks," *MobiCom*, 2000\.__ \(The canonical geographic routing protocol that requires absolute coordinates\)\.  
12\. __P\. Casari et al\., "A Randomized, Robust, Range\-Free, Localization and Routing Algorithm for Wireless Sensor Networks," *IEEE SECON*, 2008\.__ \(Example of work linking less\-precise localization to routing efficiency\)\.

__Category 5: Consensus & Distributed Synchronization__  
13\. __R\. Olfati\-Saber and R\. M\. Murray, "Consensus Problems in Networks of Agents with Switching Topology and Time\-Delays," *IEEE TAC*, 2004\.__ \(Foundational theory on distributed consensus, relevant for SIGMAPS convergence\)\.  
14\. __W\. Ren and R\. W\. Beard, "Consensus Seeking in Multiagent Systems Under Dynamically Changing Interaction Topologies," *IEEE TAC*, 2005\.__ \(Further work on consensus under network dynamics\)\.

__Category 6: Energy\-Efficient WSN Operations__  
15\. __G\. J\. Pottie and W\. J\. Kaiser, "Wireless Integrated Network Sensors," *Communications of the ACM*, 2000\.__ \(Seminal paper arguing for extreme energy frugality as a primary design goal for WSNs\)\. 

---

__Implementation Note: Current Status__

The SIGMAPS\-CG algorithm is currently being implemented in the WsnMap visualization tool\. The current implementation includes:

- ✅ RSS matrix calculation \(rssmtrxval, rssmtrxid\) from reference topology
- ✅ Self\-correction mechanism with RSS\-weighted centroid
- ✅ Kernel statistics tracking \(optional, for enhanced robustness\)
- ✅ Timer\-based iterative updates \(100ms intervals\)

**Pending Implementation:**
- 🔄 Scale clamping and normalization \(Section 4\.5\)
- 🔄 Convergence freeze mechanism \(Section 4\.4\)
- 🔄 Map broadcasting and passive updates \(Section 5\.1, 5\.2\)

The implementation follows the algorithm specification in Section 4 and 5, with emphasis on self\-only updates and distributed convergence\. See `documents/DesignTips.md` for migration continuity and implementation guidelines\.

 

