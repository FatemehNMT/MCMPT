**Following my previous dissertation — Multi-Person Localization and Tracking Using Dynamic Bayesian Networks and Hierarchical Data Association — which was primarily based on statistical modeling and MATLAB implementations, I decided to extend my research direction toward deep-learning-based and more creative approaches using Python.**

**Building upon the novelties introduced in my earlier work, I have now expanded the framework to both Single-Camera and Multi-Camera Multi-Person Tracking scenarios.**
This includes rethinking the data association pipeline, improving robustness under occlusion, and introducing new ideas that generalize across camera views.

**Numerical results and video demonstrations are currently being generated and will be added soon.**
Below, I have listed the main novelties and methodological contributions of this line of research.
The core scripts, refined codebase, and final outputs will be uploaded shortly.


**Single-Camera Multi-Person Localization & Tracking - Key Contributions**

*   **Frame-wise Affinity Matrix** Construction between incoming detections and existing trajectories.

*   **Scene-aware Detection Evaluation** using a novel scale calibration factor.

*   **Hierarchical Data Association Pipeline** combining coarse-to-fine matching.

*   **Graph-based Clustering via Connected Components** for association refinement.

  In this work, the affinity matrix is analyzed using graph-based methods. Each element with value **1** is considered as a node in a graph. An edge is created between two nodes if they share the same row or the same column. This construction naturally transforms the problem into finding **connected components** of the graph. Each connected component corresponds to a cluster of positions in the matrix that are related through rows and/or columns. This approach allows us to go beyond simple row-wise or column-wise matches and instead capture larger groups of interdependent elements in a unified, systematic way.
In short, the use of graph theory (connected components) provides a clear and elegant solution for grouping and analyzing the structure of the affinity matrix.

*   **Unified Entry-Exit Modeling** for robust handling of targets appearing or disappearing.

The primary contribution of this work lies in the **automatic discovery of entry and exit regions** in multi-object tracking (MOT) scenarios. Unlike existing approaches that either ignore entry/exit modeling or rely on manually annotated regions of interest, our method introduces a **data-driven and adaptive framework** with the following innovations:

1. **Unified Entry/Exit Modeling.** Instead of treating entries and exits separately, we aggregate both trajectory start and end points into a single set. This unification reflects the natural duality of scene boundaries, which may function simultaneously as entry and exit zones depending on perspective.

2. **Clustering-Based Zone Discovery.** We employ density-based clustering (e.g., DBSCAN/MeanShift) to identify high-density zones of trajectory endpoints. Clustering not only reveals the dominant entry/exit regions but also eliminates isolated noisy tracks through outlier rejection.

3. **Adaptive Margin Estimation.** We propose a novel mechanism to derive **dynamic margins** around the scene boundaries. For each boundary (top, bottom, left, right), margins are defined based on the farthest endpoint of the nearest cluster. This ensures margins are aligned with real observed motion, instead of being arbitrarily chosen.

4. **Threshold-Guided Robustness.** To prevent cluster elongation or outliers from inflating margins, side-specific thresholds are applied. These thresholds can incorporate prior knowledge of scene geometry (e.g., narrower upper entrances, wider lateral exits), combining data-driven discovery with human interpretability.

5. **Visualization and Interpretability.** Discovered regions are visualized both through convex hulls of endpoint clusters and heatmaps of spatial density. This dual visualization provides clear interpretability for both qualitative analysis and system debugging.

**Impact.** This framework enhances MOT by improving the handling of trajectories at scene boundaries, reducing identity switches and track fragmentation. More importantly, the proposed method lays the groundwork for **multi-camera MOT**, where accurate modeling of entry/exit zones is crucial for cross-camera handover. By providing an unsupervised yet adaptive mechanism for entry/exit discovery, this work contributes a novel perspective to MOT literature.

In single-camera multi-object tracking, entry and exit region extraction has proven useful to detect when targets appear or leave the scene. Extending this approach to multi-camera setups, we applied the same methodology independently to each camera in the WILDTRACK dataset. However, due to the nearly simultaneous coverage of all cameras, entry and exit points are widely distributed along all image edges in each view. Consequently, individual camera-based regions lose discriminative power when used for multi-camera data association. While the method effectively identifies local clusters and margins for single-camera scenarios, the overlap across views and projection onto a common ground plane reduces their direct utility for predicting target appearance or disappearance. Nonetheless, this analysis provides valuable insights into camera coverage patterns and informs the design of trajectory-based data association strategies in multi-camera tracking systems.

While entry and exit region extraction has proven useful in single-camera multi-object tracking scenarios—helping to identify when targets appear or leave the scene—our experiments on the WILDTRACK dataset show that, for multi-camera setups with nearly simultaneous camera views, these regions are widely distributed across all edges of each camera view. Consequently, their direct utility for decision-making in multi-camera data association is limited. Each camera captures targets from different angles at the same time, and the projection of these points onto a common ground plane dilutes the spatial significance of any single entry/exit region. Therefore, trajectory-based methods and motion modeling across cameras remain more reliable for predicting target appearance and disappearance in multi-camera tracking. Highlighting this limitation provides insight into the challenges of extending single-camera heuristics to multi-camera systems.

**Multi-Camera Multi-Person Localization & Tracking - Key Contributions**

*   **Projection-Based Cross-Camera Consistency** on the shared ground plane.

*   **Synchronized Multi-Camera Warping** for geometric agreement checking.

*   **Cross-View Geometric Gating** to remove implausible match candidates.

*   **Two-Path Threshold Updating Strategy** for adaptive T_temp and T_geom.

*   **Hybrid Update Mechanism combining** temporal & geometric constraints.

*   **Camera-Aware Spatial DBSCAN** for clustering detections on ground plane.

*   **Global Fusion Graph Construction** linking detections across all cameras.

*   **Component-Based Visualization & Box Coloring** for interpretability.

*   **Retention of Small Clusters/Subclusters** to avoid over-filtering.

*   **Removal of YOLO-Only Artifact Regions** (non-existent objects).

*   **Global ID Initialization from First Frame** based on multi-camera agreement.

*   **Structured Storage of Ground/Camera Features** for scalable processing.

*   **Inter-Camera Consistency Verification** for identity stability.

*   **Independent Kalman Filters per Camera and Ground Plane**, supporting fusion.

*   **Optional Multi-Camera Appearance Fusion** (feature-based refinement).

*   **Temporal Smoothing Across Cameras** for cross-view trajectory stability.

*   **Unified Spatio-Temporal-Appearance Scoring Function.**

*   **General Multi-Camera Data Association Framework** for each new frame.
