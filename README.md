# Instant-angelo: Build high-fidelity Digital Twin within 20 Minutes!
![](assets/demo.gif)
## Introduction
Neuralangelo facilitates high-fidelity 3D surface reconstruction from RGB video captures. It enables the creation of digital replicas of both small-scale objects and large real-world scenes, derived from common mobile devices. These digital replicas, or 'twins', are represented with an exceptional level of three-dimensional geo-detail.

Nevertheless, substantial room for improvement exists. At present, the official and reimplemented Neuralangelo implementation requires 40 hours and 40 GB on an A100 for real world scene reconstructions. An expedited variant in instant-nsr has been developed, but the results have been subpar due to parameter limitations.

To fill this gap in high-speed, high-fidelity reconstruction, our objective is to engineer an advanced iteration of Neuralangelo. This refined model will focus on high-fidelity neural surface reconstruction, streamlining the process to achieve results within an unprecedented 20 minute timeline while maintaining the highest standard of quality. 

We provide [Quick Lookup](https://github.com/hugoycj/Instant-angelo_vis) examples of project outcomes. These examples can serve as a reference to help determine if this project is suitable for your use case scenario.

## Installation
```
pip install torch torchvision
pip install git+https://github.com/NVlabs/tiny-cuda-nn/#subdirectory=bindings/torch
pip install -r requirements.txt
```
For COLMAP, alternative installation options are also available on the [COLMAP website](https://colmap.github.io/)

## Data preparation
To extract COLMAP data from custom images, you must first have COLMAP installed (you can find installation instructions [here]). Afterwards, place your images in the images/ folder. The structure of your data should be as follows:
```
-data_001
    -images
-data_002
    -images
-data_003
    -images
```
Each separate data folder houses its own images folder.

## Start Reconstruction!
### Run Smooth Reconstruction Mode

---

The smooth reconstruction mode is well-suited for the following cases:
* When reconstructing a smooth object that does not have a high level of detail. The smooth mode works best for objects that have relatively simple, flowing surfaces without a lot of intricate features.
<img src="assets/general_smooth.png">
* When you want a higher-fidelity substitute for instant-nsr that takes a similar amount of time (within 20 minute) to generate but with fewer holes in the resulting model. 
<img src="assets/nsr2angelo.png">

---

Information you need to know before you start:
* The smooth reconstruction mode's reliance on curvature loss can over-smooth geometry, failing to capture flat surface structures and subtle variations on flatter regions of the original object.
<img src="assets/over-smooth.png">
* This mode also rely on sparse points generated by colmap for geometry guidence in the early stage of training. SFM sometimes will generate noisy point cloud due to repeated texture, noisy pose or inaccurate point matches. 
<img src="assets/nosiy_sfm.png">

---

Now it is time to start by running:
```
bash run_neuralangelo-colmap_sparse.sh $YOUR_DATA_DIR
```
The results will be saved under `logs` directory.

### Run Detail Reconstruction Mode without extra preprossing

---

The detail reconstruction mode without additional preprocessing is optimal for scenarios with:
* Image data captured under varying conditions over time or with inconsistent exposure levels. 
* High resolution image sources of 2K or 4K dimensions.
* Your images' resolution are 2K or 4K
* Reconstructing objects or scenes comprised of glossy, reflective materials. 
* Subjects containing large textureless or untextured surface regions.
  
---

Information you need to know before you start:
* The detail reconstruction mode requires 2-3 times longer to complete compared to the smooth mode, owing to the use of a larger final hash grid resolution and more training steps.
* For image inputs below 1K resolution, the detail mode may yield marginal improvements over other modes. Images under 1K likely do not provide sufficient information to take full advantage of the capabilities of detail reconstruction.
  
---

Now it is time to start by running:
```
bash run_neuralangelo-colmap_sparse-50k.sh  $YOUR_DATA_DIR
```

### Run Detail Reconstruction Mode with MVS prior
---
Generating high-fidelity surface reconstructions with only RGB inputs in 20,000 steps (around 20 minutes) is challenging, especially for sparse in-the-wild captures where occlusion and limited views make surface reconstruction an underconstrained problem. This can lead to optimization instability and difficulty converging. Introducing lidar, ToF depth, or predicted depth can help stabilize optimization and accelerate training. However, directly regularizing rendered depth is suboptimal due to bias introduced by density2sdf. Moreover, ensuring consistent depth across views is difficult, especially with lower-quality ToF sensors or predicted depth. We propose directly regularizing the SDF field using MVS point clouds and normals to alleviate the bias

Importantly, in real-world scenarios like oblique photography and virtual tours, dense point clouds are already intermediate outputs. This allows directly utilizing the existing point clouds for regularization without extra computation. In such use cases, the point cloud prior comes for free as part of the capture process. 

---

Information you need to know before you start:
* An aligned dense point cloud with normal is necessary, you could specify the relative path at `dataset.dense_pcd_path` in the config file
* The point cloud could be generated from various methods, either from traditional MVS like colmap or OpenMVS, or learning-based MVS method. You could even generate the point cloud using commercial photogrammetry software like metashape and DJI.
  
---

Now it is time to start by running:
```
bash run_neuralangelo-colmap_dense.sh  $YOUR_DATA_DIR
```

## Frequently asked questions (FAQ)
1. **Q:** CUDA out of memory. 
   **A:** Instant-angelo requires at least 10GB GPU memory. If you run out of memory,  consider decreasing `model.num_samples_per_ray` from 1024 to 512
2. **Q:** What's the License for this repo?
   **A:** This repository is built on top of instant-nsr-pl and is licensed under the MIT License. The materials, code, and assets in this repository can be used for commercial purposes without explicit permission, in accordance with the terms of the MIT License. Users are free to use, modify, and distribute this content, even for commercial applications. However, appropriate attribution to the original instant-nsr-pl authors and this repository is requested. Please refer to the LICENSE file for full terms and conditions.
3. **Q:** The reconstruction of my custom dataset is bad.
   **A:** This repository is under active development and its robustness across diverse real-world data is still unproven. Users may encounter issues when applying the method to new datasets. Please open an issue for any problems or contact the author directly at chongjieye@link.cuhk.edu.cn. 
4. **Q:** This project fails to run on Windows
   **A:** This project has not been tested on Windows and the scripts may have compatibility issues. For the best experience at this stage of development, we recommend running experiments on a Linux system. We apologize that Windows support cannot be guaranteed currently. Please feel free to open an issue detailing any problems encountered when attempting to run on Windows. Community feedback will help improve cross-platform compatibility going forward.
5. **Q:** Generate dense prior with Vis-MVSNet is slow
  **A:** Currently, preprocessing takes around 10~15 minutes for 300 frames, but there is still remains much room to improve efficiency by replacing Vis-MVSNet with state-of-the-art methods like MVSFormer or SimpleRecon. Moreover, preprocessing time could be substantially reduced by leveraging quantization and TensorRT. Overall, MVSNet allows generating the necessary point cloud prior an order of magnitude faster than traditional MVS approaches. 
6. **Q:** The poor reliabity of the sparse and dense prior introduce in this repo
   **A:** Currently, this method relies on point clouds which limits robustness for textureless surfaces or glossy materials. An updated point cloud-free version that directly regularizes surfaces using image matching and warping is under development for release in late November 2023. This will improve robustness in challenging cases. Please open an issue for any specific point cloud-related difficulties encountered currently.

## Related project:
- [instant-nsr-pl](https://github.com/bennyguo/instant-nsr-pl): Great Instant-NSR implementation in PyTorch-Lightning! 
- [neuralangelo](https://github.com/NVlabs/neuralangelo): Official implementation of *Neuralangelo: High-Fidelity Neural Surface Reconstruction*
- [sdfstudio](https://github.com/autonomousvision/sdfstudio): Unified Framework for SDF-based Neural Reconstruction, easy to development
- [torch-bakedsdf](https://github.com/hugoycj/torch-bakedsdf): Unofficial pytorch implementation of *BakedSDF:Meshing Neural SDFs for Real-Time View Synthesis*

## Acknocklement
* Thanks to bennyguo for his excellent pipeline [instant-nsr-pl](https://github.com/bennyguo/instant-nsr-pl)
* Thanks to RaduAlexandru for his implementation of improved curvature loss in [permuto_sdf](https://github.com/RaduAlexandru/permuto_sdf)
* Thanks for Zesong Yang and Chris for providing valuable insights and feedback that assisted development
