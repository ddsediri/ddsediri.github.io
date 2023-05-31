---
layout: page
title: IterativePFN&#58; True Iterative Point Cloud Filtering
description: Accepted to CVPR, 2023
img: assets/img/projects/iterativepfn/network.png
importance: 1
category: work
---

**ABSTRACT**: The quality of point clouds is often limited by noise introduced during their capture process. Consequently, a fundamental 3D vision task is the removal of noise, known as point cloud filtering or denoising. State-of-the-art learning based methods focus on training neural networks to infer filtered displacements and directly shift noisy points onto the underlying clean surfaces. In high noise conditions, they iterate the filtering process. However, this iterative filtering is only done at test time and is less effective at ensuring points converge quickly onto the clean surfaces. We propose IterativePFN (iterative point cloud filtering network), which consists of multiple IterationModules that model the true iterative filtering process internally, within a single network. We train our IterativePFN network using a novel loss function that utilizes an adaptive ground truth target at each iteration to capture the relationship between intermediate filtering results during training. This ensures that the filtered results converge faster to the clean surfaces. Our method is able to obtain better performance compared to state-of-the-art methods. The source code can be found at <https://github.com/ddsediri/IterativePFN>.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/projects/iterativepfn/network.png" title="IterativePFN architecture" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/projects/iterativepfn/Adaptive_GT_1.png" title="Adaptive ground truth" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Left: IterativePFN network architecture. Right: Illustration of novel adaptive ground truth loss function for gradually filtering points which also facilitates joint training of IterationModules and allows the network to learn relationships between intermediate filtered displacements. 
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/projects/iterativepfn/results.gif" title="Results on PUNet dataset" class="img-fluid rounded z-depth-1" %}
    </div>
</div>