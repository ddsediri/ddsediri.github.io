---
layout: project
title: StraightPCF&#58; Straight Point Cloud Filtering
description: A new method that models noisy patches as intermediate states in a filtering process that moves points along straight flows. Accepted to CVPR, 2024
img: assets/img/projects/straightpcf/network.png
importance: 1
category: [Point Cloud Filtering]
---
<div class="row justify-content-sm-center">
  <div class="col-sm-10 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/straightpcf_teaser.png" title="StraightPCF Filtering Trajectories" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  Filtered trajectories for the Icosahedron shape at 50K resolution and noise scale σ = 3%. StraightPCF filters points along much straighter paths compared to ScoreDenoise.
</div>

## Abstract

Point cloud filtering is a fundamental 3D vision task, which aims to remove noise while recovering the underlying clean surfaces. State-of-the-art methods remove noise by moving noisy points along stochastic trajectories to the clean surfaces, often requiring regularization within the training objective and/or post-processing to ensure fidelity.

We introduce **StraightPCF**, a new deep learning based method for point cloud filtering that works by moving noisy points along **straight paths**, reducing discretization errors while ensuring faster convergence to the clean surfaces. We model noisy patches as intermediate states between high-noise patch variants and their clean counterparts, and design a **VelocityModule** to infer a constant flow velocity from the former to the latter. This constant flow leads to straight filtering trajectories. In addition, we introduce a **DistanceModule** that scales the straight trajectory using an estimated distance scalar to attain convergence near the clean surface.

Our network is lightweight with only ~530K parameters — just 17% of IterativePFN (a recent state-of-the-art point cloud filtering network). Extensive experiments on both synthetic and real-world data show our method achieves state-of-the-art results and produces well-distributed filtered points without requiring any regularization.

---

## Method

<div class="row">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/straightpcf_architecture.png" title="StraightPCF Network Architecture" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  The StraightPCF network. A coupled VelocityModule stack infers a constant flow velocity for patch states, while the DistanceModule infers a distance scalar that scales the velocity to ensure filtered points converge to the surface.
</div>

### Filtering via Straight Flows

Prior methods (e.g., ScoreDenoise, IterativePFN) treat filtering as a reverse Markov process, resulting in stochastic filtering trajectories that are sensitive to step sizes and prone to clustering artifacts. We reformulate filtering as an **optimal transport problem**: we move noisy input patches from a high-noise distribution $$\pi_0$$ to the clean surface distribution $$\pi_1$$ along the shortest (straight) path.

Noisy patches $$\mathbf{X}_t$$ are modelled as intermediate states of a linear interpolation between a high-noise variant $$\mathbf{X}_0$$ and the clean patch $$\mathbf{X}_1$$:

$$\mathbf{X}_t = (1-t)\mathbf{X}_0 + t\mathbf{X}_1$$

The flow is governed by an ODE $$d\mathbf{X}_t = \mathbf{v}(\mathbf{X}_t)\,dt$$, where the velocity field $$\mathbf{v}$$ is approximated by our **VelocityModule** — a DGCNN-based graph neural network.

### Straighter Flows via VelocityModule Coupling

To further improve path straightness, we couple $$K$$ VelocityModules together. Each module handles a segment of the trajectory from $$\mathbf{X}_t$$ to $$\mathbf{X}_1$$. Empirically, $$K=2$$ provides the best balance between accuracy and efficiency. The coupled modules are fine-tuned with an objective that simultaneously encourages constant velocity inference and minimises deviation from the linear interpolation path.

### DistanceModule

Constant flows may cause filtered points to overshoot the clean surface. The **DistanceModule** addresses this by estimating a distance scalar corresponding to the relative distance of the noisy patch from the clean surface. This scalar is used to scale the flow velocity, ensuring convergence near the surface. Crucially, StraightPCF is the **first method to decompose point cloud filtering into a dual objective**: inferring a vector field of flow velocities and a distance scalar.

---

## Results

### Quantitative Results on Synthetic Data

StraightPCF achieves state-of-the-art results on the **PUNet** and **PCNet** benchmarks across all noise scales (1%, 2%, 3%) and resolutions (10K sparse, 50K dense), as measured by Chamfer Distance (CD) and Point-to-Mesh distance (P2M).

On the PCNet dataset at 10K resolution and σ = 3% noise (unseen during training), our method achieves:
- **17.1% reduction** in CD error vs. the previous best
- **24.1% reduction** in P2M error vs. the previous best

<div class="row">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/straightpcf_synthetic.png" title="Visual Filtering Results" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  Visual filtering results for 50K resolution shapes (σ = 2%). StraightPCF recovers complex geometric details and produces well-distributed points, avoiding the clustering holes exhibited by DeepPSR and IterativePFN.
</div>

### Quantitative Results on Real-World Scanned Data

On the **Kinect v1** dataset (71 point clouds), StraightPCF achieves a 1.82% reduction in CD error compared to IterativePFN, while maintaining better point distributions without clustering artifacts.

<div class="row">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/straightpcf_ruemadame.png" title="Paris-Rue-Madame Results" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  Visual filtering results on two scenes from the Paris-Rue-Madame dataset. StraightPCF recovers the underlying clean surface with better point distributions, while methods like PDFlow and IterativePFN leave noisy artefacts or cause clustering along scan lines.
</div>

### Ablation Study

| Variant | CD (1%) | P2M (1%) | CD (2%) | P2M (2%) | CD (3%) | P2M (3%) |
|---|---|---|---|---|---|---|
| V1: VM w/o DM | 2.16 | 0.42 | 3.06 | 0.92 | 3.73 | 1.42 |
| V2: VM w/ DM | 2.00 | 0.32 | 3.06 | 0.96 | 3.81 | 1.55 |
| V3: Large VM | 2.17 | 0.41 | 2.99 | 0.84 | 3.54 | 1.29 |
| V4: CVM w/o DM | 1.97 | 0.32 | 3.01 | 0.92 | 3.71 | 1.45 |
| **V5: CVM w/ DM (Ours)** | **1.87** | **0.24** | **2.64** | **0.60** | **3.29** | **1.13** |

The ablation confirms that both VelocityModule coupling and the DistanceModule are critical to the full method's performance. Values ×10⁴ at 10K resolution on PUNet.

---

## BibTeX

```bibtex
@InProceedings{deSilvaEdirimuni_2024_CVPR,
  author    = {de Silva Edirimuni, Dasith and Lu, Xuequan and Li, Gang and Wei, Lei and Robles-Kelly, Antonio and Li, Hongdong},
  title     = {StraightPCF: Straight Point Cloud Filtering},
  booktitle = {Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR)},
  month     = {June},
  year      = {2024},
  pages     = {20721--20730}
}
```

---

## Links

<div class="row mt-3">
  <div class="col-sm-auto">
    <a href="https://openaccess.thecvf.com/content/CVPR2024/papers/de_Silva_Edirimuni_StraightPCF_Straight_Point_Cloud_Filtering_CVPR_2024_paper.pdf" class="btn btn-sm z-depth-0" role="button" target="_blank" style="font-size:12px;">
      <i class="fas fa-file-pdf"></i> PDF
    </a>
  </div>
  <div class="col-sm-auto">
    <a href="https://github.com/ddsediri/StraightPCF" class="btn btn-sm z-depth-0" role="button" target="_blank" style="font-size:12px;">
      <i class="fab fa-github"></i> Code
    </a>
  </div>
  <div class="col-sm-auto">
    <a href="https://openaccess.thecvf.com/content/CVPR2024/html/de_Silva_Edirimuni_StraightPCF_Straight_Point_Cloud_Filtering_CVPR_2024_paper.html" class="btn btn-sm z-depth-0" role="button" target="_blank" style="font-size:12px;">
      <i class="ai ai-cvf"></i> CVF
    </a>
  </div>
</div>
