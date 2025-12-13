# **Strategies and KeyContributions**

**Following my previous dissertation — Multi-Person Localization and Tracking Using Dynamic Bayesian Networks and Hierarchical Data Association — which was primarily based on statistical modeling and MATLAB implementations, I decided to extend my research direction toward deep-learning-based and more creative approaches using Python.**

**Building upon the novelties introduced in my earlier work, I have now expanded the framework to both Single-Camera and Multi-Camera Multi-Person Tracking scenarios.**
This includes rethinking the data association pipeline, improving robustness under occlusion, and introducing new ideas that generalize across camera views.

**Numerical results and video demonstrations are currently being generated and will be added soon.**
This document summarizes the conceptual contributions and design choices of an ongoing research project.


## **Single-Camera Multi-Person Localization & Tracking - Key Strategies and Contributions**

*   **Frame-wise Affinity Matrix** Construction between incoming detections and existing trajectories.
  
Following each detection stage, an affinity matrix is constructed between existing trajectories (rows) and current detections (columns). Each trajectory has a predicted region of possible appearance; if a detection falls inside such a region, the corresponding entry becomes 1, otherwise 0.
Ideally, each column contains exactly one non-zero row; however, real-world tracking yields several ambiguous scenarios:

1. Zero row → lost trajectory / FN
2. Zero column → new entry or FP
3. Multiple matches → ambiguity resolved using appearance similarity and distance metrics
4. Unequal row/column counts → matrix augmentation required

*   **Applying Deep re-identification**

*   **Scene-aware Detection Evaluation** using a novel scale calibration factor.

False positives frequently occur in regions outside feasible human-presence areas or with implausible scales. We exploit camera geometry and scene depth to eliminate such detections.
A scale factor is derived from scene perspective, converting bounding-box heights into metric proxies for pedestrian height. It is used to:
1. reject detections with implausible bounding-box sizes,
2. restrict predicted areas for next-frame appearance.
Scale factors vary across datasets and can be estimated experimentally or learned via supervised analysis.

*   **Hierarchical Data Association Pipeline** combining coarse-to-fine matching.

  A novel strategy based on an affinity matrix that links current detections to predicted target states, combining other elements such as our short-term memory, similarities and the scene information.

*   **Graph-based Clustering via Connected Components** for association refinement.

In this work, the affinity matrix is analyzed using graph-based methods. Each element with value **1** is considered as a node in a graph. An edge is created between two nodes if they share the same row or the same column. This construction naturally transforms the problem into finding **connected components** of the graph. Each connected component corresponds to a cluster of positions in the matrix that are related through rows and/or columns. This approach allows us to go beyond simple row-wise or column-wise matches and instead capture larger groups of interdependent elements in a unified, systematic way.
In short, the use of graph theory (connected components) provides a clear and elegant solution for grouping and analyzing the structure of the affinity matrix.

*   **Entry-Exit Modeling** for robust handling of targets appearing or disappearing.

The primary contribution of this work lies in the **automatic discovery of entry and exit regions** in multi-object tracking (MOT) scenarios. Unlike existing approaches that either ignore entry/exit modeling or rely on manually annotated regions of interest, our method introduces a **data-driven and adaptive framework** with the following innovations:

1. **Unified Entry/Exit Modeling.** Instead of treating entries and exits separately, we aggregate both trajectory start and end points into a single set. This unification reflects the natural duality of scene boundaries, which may function simultaneously as entry and exit zones depending on perspective.

2. **Clustering-Based Zone Discovery.** We employ density-based clustering (e.g., DBSCAN/MeanShift) to identify high-density zones of trajectory endpoints. Clustering not only reveals the dominant entry/exit regions but also eliminates isolated noisy tracks through outlier rejection.

3. **Adaptive Margin Estimation.** We propose a novel mechanism to derive **dynamic margins** around the scene boundaries. For each boundary (top, bottom, left, right), margins are defined based on the farthest endpoint of the nearest cluster. This ensures margins are aligned with real observed motion, instead of being arbitrarily chosen.

4. **Threshold-Guided Robustness.** To prevent cluster elongation or outliers from inflating margins, side-specific thresholds are applied. These thresholds can incorporate prior knowledge of scene geometry (e.g., narrower upper entrances, wider lateral exits), combining data-driven discovery with human interpretability.

5. **Visualization and Interpretability.** Discovered regions are visualized both through convex hulls of endpoint clusters and heatmaps of spatial density. This dual visualization provides clear interpretability for both qualitative analysis and system debugging.

**Impact.** While entry and exit region extraction has proven useful in single-camera multi-object tracking scenarios—helping to identify when targets appear or leave the scene—our experiments on the WILDTRACK dataset show that, for multi-camera setups with nearly simultaneous camera views, these regions are widely distributed across all edges of each camera view. Consequently, their direct utility for decision-making in multi-camera data association is limited. Each camera captures targets from different angles at the same time, and the projection of these points onto a common ground plane dilutes the spatial significance of any single entry/exit region. Therefore, trajectory-based methods and motion modeling across cameras remain more reliable for predicting target appearance and disappearance in multi-camera tracking. Highlighting this limitation provides insight into the challenges of extending single-camera heuristics to multi-camera systems.

## **Multi-Camera Multi-Person Localization & Tracking - Key Strategies and Contributions**

*   **Projection-Based Cross-Camera Consistency** on the shared ground plane (Applying the position on the ground, instead of pixels, to compare detection boxes.).

To enable cross-camera spatial consistency, we established a unified ground-plane coordinate system.
Using each camera’s extrinsic calibration matrix, 2D image coordinates were back-projected to 3D rays, which were then intersected with the estimated ground plane.
This produced per-camera ground projections in the global metric space.
The mean ground position across all cameras was computed to define a shared spatial reference, and the per-camera deviations were analyzed to evaluate geometric consistency.
The resulting ground-plane alignment allowed direct geometric association of objects across cameras.

*   **Adaptive Homography Refinement for Ground Stability**
  
Although the homography matrices are initially computed from camera calibration parameters, minor temporal drift and misalignment can still emerge during long-term operation due to imperfect detections, synchronization offsets, or local depth inconsistencies.
To mitigate these effects, we implement a dynamic self-calibration mechanism that continuously monitors the mean reprojection error across frames. When systematic drift is detected, recent multi-view correspondences are aggregated to re-estimate refined homographies using a robust least-squares approach.
The refined transformations are fused with the existing ones via an exponential moving average, ensuring smooth temporal adaptation without abrupt geometric changes. This adaptive refinement maintains a stable and coherent ground-plane alignment, thereby improving temporal and geometric consistency across cameras.

*   **Multi-Camera Synchronized Projection** Comparison of frames projected onto the Earth at the same time.

*   **Two-Path Updating Strategy** on the shared ground plane for adaptive T_temp and T_geom.
  
  T_temp (Temporal Geometric Update)
  
  T_geom (Pure Geometric Threshold)

  Estimate these two terms independently and aggregate the results.

*   **Hybrid Update Mechanism** Selecting ID by combining temporal and geometric information and Building a score by weighting between the two.
     
*   **Camera-Aware Spatial DBSCAN** for clustering detections on ground plane (not Pixels).

    Running DBSCAN with thresholds dependent on camera distance (i.e. the ground distance is not the same for the near and far cameras).

    Used for: Removing FPs, Getting clean components, Threshold tuning and Visual analysis.

    In other words, we propose a geometry-aware adaptive distance threshold for cross-camera detection association. Instead of using a single fixed tolerance for all camera pairs, our method computes a per-pair geometric threshold from camera layout (distance between image centers projected to the common ground plane) and refines it with empirical statistics collected from annotated cross-view matches. The final threshold for a camera pair ( 𝐴 , 𝐵 ) (A,B) is given by the maximum of a geometry-driven baseline and an empirical percentile (e.g., 90th) of observed inter-camera projection errors. This hybrid strategy (1) accounts for camera topology and (2) absorbs real-world projection noise, improving robustness of cross-camera association in crowded and calibrated scenes.

*   **Component-Based Visualization & Box Coloring** for interpretability.

    Show color-coded boxes by component
  
    Helps analyze thresholds and DBSCAN behavior

*   **Retention of Small Clusters/Subclusters** to avoid over-filtering.

    More focus on Subclusters and their analyze to avoid getting real people lost in the crowd, considering inter-cluster distances.

*   **Camera Selection in workin with WILDTRACK-Seven-Camera-HD-Dataset**
    Camera Selection Principles and Criteria (What is Important and Why):
    1. Ground Coverage
    2. Overlap
    3. Baseline / Viewpoint Diversity
    4. Calibration Quality & Reprojection Error
    5. Effect of Occlusion and Crowding
    6. Temporal Distribution / Δt and Connectivity
    7. Image Clarity / FPS / Lighting
    8. Height and Distance Diversity
  
*   **Global Fusion Graph Construction** linking detections across all cameras.

*   **Removal of YOLO-Only Artifact Regions** (non-existent objects).

    Remove detections that are not in the main pipeline

    Prevent FP from propagating to the tracking chain

*   **Global ID Initialization from First Frame** based on multi-camera agreement.

    Work on global ground and then compare all cameras and Appearances, and finally extra unassigned ones.

*   **Structured Storage of Ground/Camera Features** for scalable processing.

*   **Inter-Camera Consistency Verification** for identity stability.

*   **Independent Kalman Filters per Camera and Ground Plane**, supporting fusion.

*   **Temporal Smoothing Across Cameras** for cross-view trajectory stability.
 
    Use short-term cross-camera history to prevent ID switches when a person leaves one camera's view and enters another.
    For this dataset, Δt_appear is useful; Since people are often seen in multiple cameras at the same time, appearance-to-appearance helps with matching speed and accuracy.

*   **Unified Spatio-Temporal-Appearance Scoring Function** for updates and best match selection.

    Design a complete Scoring Function including: geometry, temporal continuity, appearance embedding or color feature (later)

*   **General Multi-Camera Data Association Framework** for each new frame.
