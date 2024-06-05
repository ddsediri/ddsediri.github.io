---
layout: project
title: StraightPCF&#58; Straight Point Cloud Filtering
description: A new method that models noisy patches as intermediate states in a filtering process that moves points along straight flows. Accepted to CVPR, 2024
img: assets/img/projects/straightpcf/network.png
importance: 1
category: [Point Cloud Filtering]
---

<!-- <div class="authors-image">
    {% include figure.html path="assets/img/projects/iterativepfn/profiles.png" title="People" %}
</div>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        <p class="authors">
            Dasith de Silva Edirimuni<sup>1</sup>, Xuequan Lu<sup>1</sup>, Zhiwen Shao<sup>2</sup>, Gang Li<sup>1</sup>, Antonio Robles-Kelly<sup>1,4</sup>, Ying He<sup>3</sup>
        </p>
    </div>
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        <p class="affiliations">
            <sup>1</sup>School of Information Technology, Deakin University, <sup>2</sup>School of Computer Science and Technology, China University of Mining and Technology, <sup>3</sup>School of Computer Science and Engineering, Nanyang Technological University, <sup>4</sup>Defense Science and Technology Group, Australia
        </p>
    </div>
</div>

<pre><a href="https://openaccess.thecvf.com/content/CVPR2023/papers/de_Silva_Edirimuni_IterativePFN_True_Iterative_Point_Cloud_Filtering_CVPR_2023_paper.pdf" target="_blank">[Paper]</a>    <a href="https://github.com/ddsediri/IterativePFN" target="_blank">[Code]</a>   <a href='{{ "assets/bibtex/iterativepfn.bib" | relative_url }}' target="_blank">[BibTeX]</a>
</pre>

<p style="text-align: justify; text-justify: inter-word;"><strong>ABSTRACT</strong>: The quality of point clouds is often limited by noise introduced during their capture process. Consequently, a fundamental 3D vision task is the removal of noise, known as point cloud filtering or denoising. State-of-the-art learning based methods focus on training neural networks to infer filtered displacements and directly shift noisy points onto the underlying clean surfaces. In high noise conditions, they iterate the filtering process. However, this iterative filtering is only done at test time and is less effective at ensuring points converge quickly onto the clean surfaces. We propose IterativePFN (iterative point cloud filtering network), which consists of multiple IterationModules that model the true iterative filtering process internally, within a single network. We train our IterativePFN network using a novel loss function that utilizes an adaptive ground truth target at each iteration to capture the relationship between intermediate filtering results during training. This ensures that the filtered results converge faster to the clean surfaces. Our method is able to obtain better performance compared to state-of-the-art methods. The source code can be found at <a href="https://github.com/ddsediri/IterativePFN">https://github.com/ddsediri/IterativePFN</a>.
</p>

## Method

<p style="text-align: justify; text-justify: inter-word;">We present IterativePFN, a novel architecture geared towards filtering point clouds. We design IterationModules, that consume directed graphs of point cloud patches to produce rich node (point) features that are then used to generate filtered displacements. These displacements regress noisy points back towards the underlying clean surface. Unlike current methods that only apply iterative filtering at test times, our method extends iterative filtering to a true train + test time solution where the relationships between consecutive, intermediate displacements are learned by the network.  
</p>

### IterativePFN pipeline
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/projects/iterativepfn/network.png" title="IterativePFN architecture" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    IterativePFN network architecture.</div>

<p style="text-align: justify; text-justify: inter-word;">To train our network, we employ a novel adaptive ground truth loss function to supervise training.
</p>

### Adaptive ground truth loss function

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/projects/iterativepfn/Adaptive_GT_0.png" title="Adaptive ground truth" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/projects/iterativepfn/Adaptive_GT_1.png" title="Adaptive ground truth" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

<div class="caption">
    Left: Current methods attempt to directly regress noisy points back to underlying clean surfaces. However, for high noise point clouds, they must iteratively apply filtering. These methods do not consider the relationships between inferred displacements across iterations is not considered. Right: Our novel adaptive ground truth loss function gradually filters points and facilitates joint training of IterationModules. This allows the network to learn relationships between intermediate filtered displacements. 
</div>

<p style="text-align: justify; text-justify: inter-word;">The adaptive ground truth loss function that supervises training is constructed as follows:
</p>

Our adaptive ground truth loss, per point, per IterationModule, can be expressed as:

$$
\newcommand{\norm}[1]{\left\|#1\right\|} 
\begin{align}
L^{(\tau)}_i(\mathcal{Y}^{(\tau)}) =~&\norm{\pmb{d}^{(\tau)}_i - \big[NN(\pmb{x}^{(\tau-1)}_i, \mathcal{Y}^{(\tau)})-\pmb{x}^{(\tau-1)}_i\big]}^2_2,  \notag
\end{align}
$$

Gaussian weights based on position from patch center

$$
\begin{align}
    w_i = \frac{\exp\left(-\norm{\pmb{x}_i-\pmb{x}_r}^2_2/r^2_s\right)}{\sum_i \exp\left(-\norm{\pmb{x}_i-\pmb{x}_r}^2_2/r^2_s\right)}, \notag
\end{align}
$$

Single IterationModule loss $$\leftrightarrow$$ weighted average across points

$$
\begin{align}
    L^{(\tau)} = \sum_i w_i L^{(\tau)}_i,  \notag
\end{align}
$$

Sum loss contributions across all ItMs $$\rightarrow$$ allows joint training

$$
\begin{align}
    \mathcal{L}_a = \sum^{T}_{\tau=1} L^{(\tau)}.  \notag
\end{align}
$$

## Results
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/projects/iterativepfn/results.gif" title="Results on PUNet dataset" class="img-fluid rounded z-depth-1" %}
    </div>
</div> -->