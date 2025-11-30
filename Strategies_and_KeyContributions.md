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

*   **Unified Entry-Exit Modeling** for robust handling of targets appearing or disappearing.

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
