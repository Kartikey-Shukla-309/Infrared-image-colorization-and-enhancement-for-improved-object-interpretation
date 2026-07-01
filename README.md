# Infrared-image-colorization-and-enhancement-for-improved-object-interpretation


### **ISRO Bharatiya Antriksh Hackathon 2026 — Problem Statement #10**

---

##  Project Overview & Context
Satellite-based Thermal Infrared (TIR) remote sensing is indispensable for global thermodynamic observation, tracking urban heat islands, water stress, and geothermal anomalies[cite: 2]. However, hardware constraints impose severe trade-offs between a sensor's signal-to-noise ratio and its spatial resolution[cite: 2]. 

This project delivers an end-to-end deep learning and sensor-physics pipeline developed for **Problem Statement #10 of the ISRO Bharatiya Antriksh Hackathon 2026**[cite: 2]. The framework mitigates the spatial constraints of Landsat 9's Thermal Infrared Sensor-2 (TIRS-2) Band 10 data through a dual-stage approach[cite: 2]:
1. **Physics-Guided Spatial Super-Resolution:** Upscaling degraded 200m thermal imagery to an enhanced 100m native baseline using a Swin Transformer backbone governed by strict thermodynamic invariants[cite: 2].
2. **Cross-Modal Latent-Semantic Translation:** Converting the super-resolved 100m single-channel thermal heatmaps into intuitive, high-fidelity 3-channel synthetic true-color (RGB) optical representations to accelerate downstream analysis[cite: 1].

---

##  Repository Folder Structure
The repository is systematically organized into separate operational modules to ensure seamless reproducibility:

---

##  Technical Architecture & Pipeline Phases

###  Phase 1: Physics-Guided TIR Spatial Super-Resolution (200m → 100m)
Standard computer vision models optimize solely for high perceptual quality, frequently hallucinating unphysical thermal artifacts that violate fundamental thermodynamic properties[cite: 2]. Our solution prevents this by embedding sensor radiometry directly into a deep transformer architecture[cite: 2].

* **Radiometric Calibration & Feature Layer:** Instead of feeding raw uncalibrated digital numbers (DN), a dedicated layer converts inputs into physical invariants—Top-of-Atmosphere (TOA) spectral radiance and absolute at-sensor Brightness Temperature ($T_B$)[cite: 2]. Concurrently, high-resolution visible Red and Near-Infrared (VNIR) channels are extracted to compute Fractional Vegetation Cover (FVC) and isolate accurate Land Surface Emissivity ($\epsilon$)[cite: 2].
* **Hybrid Thermal Denoising:** Before neural feature extraction, a fast Non-Local Means (NLM) / BM3D block eliminates microbolometer striping noise while anchoring sharp spatial thermal edges[cite: 2].
* **SwinIR Transformer Backbone:** Features project into a deep extraction sequence composed of six Residual Swin Transformer Blocks (RSTB) leveraging Windowed and Shifted-Window Multi-Head Self-Attention ($W-MSA$ / $SW-MSA$) to map long-range spatial context across terrain interfaces[cite: 2].
* **Flux Conservation Layer (The Physical Anchor):** To maintain strict physical consistency, the model includes a customized loss layer[cite: 2]. Passing the sub-pixel predictions through an inverse Planck formulation guarantees that the total integrated radiant flux of the four constituent 100m sub-pixels matches the energy signature of their 200m parent pixel[cite: 2].

###  Phase 2: Multimodal Latent-Semantic Generative Translation (TIR → RGB)
Translating thermal radiance to visible reflectance is a highly ill-posed, underdetermined optimization dilemma, as vastly different materials (e.g., cold water bodies and high-rise structural shadows) often share identical brightness temperatures[cite: 1].

* **Multi-Scale Pix2PixHD Core:** We utilize a coarse-to-fine multi-scale generative framework[cite: 1]. A Global Generator ($G_1$) extracts massive thermodynamic land trends at $128 \times 128$ resolution, while a Local Enhancer ($G_2$) preserves localized structural borders at $256 \times 256$ resolution[cite: 1].
* **Integrated Auxiliary Segmentation Head (The Novel Feature):** To suppress unphysical color bleeding without requiring hard-to-acquire external maps, the latent feature matrix splits into two parallel processing paths: a Generation Head and a **Self-Supervised Auxiliary Segmentation Head**[cite: 1]. The auxiliary branch simultaneously classifies pixels into four base categories: Water, Vegetation, Urban, and Barren[cite: 1]. This forces the latent embeddings to bind temperature variations directly to semantic compositions[cite: 1].
* **Multi-Scale Discriminators:** Dual patch-based discriminators ($D_1$, $D_2$) evaluate outputs at varying spatial frequencies, stabilizing global color transitions alongside micro-structural contours[cite: 1].

---

##  Uniqueness & Core Scientific Novelty

>  **What makes this framework stand out?**
> Most traditional approaches use generic computer vision models that merely look visually convincing but lack radiometric validity. This architecture enforces physics at every milestone:

* **Energy Conservation Invariant:** Incorporates an explicit constraint derived from Planck's Law to prevent mathematical hallucinations and guarantee absolute radiometric integrity[cite: 2].
* **Emissivity-Guided Structural Anchors:** Leverages land surface material compositions ($\epsilon$) as invariant spatial boundaries, which locks down fine structures (like shorelines or infrastructure footprints) and completely suppresses unphysical edge bleeding[cite: 1, 2].
* **Implicit Semantic Inversion:** Bypasses the need for expensive label masks by mapping underlying land-cover probability layouts using an internal self-supervised branch initialized through automated unsupervised clustering over visible target pairs[cite: 1].

---

##  Dataset Parameters & Convergence Scaling Notice

| Tensor Component | Layout Specification | Underlying Physical Property |
| :--- | :--- | :--- |
| **Input Feature Block ($\Phi$)** | $3 \times \text{Height} \times \text{Width}$ | Brightness Temp ($T_B$), Emissivity ($\epsilon$), Spatial Gradient Magnitude ($|\nabla T_B|$)[cite: 2] |
| **Semantic Classes ($c$)** | 4 Target Dimensions | Water ($c=0$), Vegetation ($c=1$), Urban ($c=2$), Barren ($c=3$)[cite: 1] |

###  Training Profile Note
Due to sandbox environment and compute runtime limitations within the hackathon framework, the pipeline was trained using a tight patch assembly of **340 high-resolution images for training** and **60 images for testing**[cite: 1]. 

* **Scalability:** While current performance indicators demonstrate sharp structural delineations, our ablation models show that **Peak Signal-to-Noise Ratio (PSNR) and Structural Similarity Index Measures (SSIM) correlate directly with training dataset volume**[cite: 1, 2]. 
* Expanding this dataset beyond the initial test boundaries eliminates residual illumination variations and scales up validation metrics predictably across seasonal distributions[cite: 1].

---

##  Future Work Roadmap
* **Full PDE-Constrained PINN Layer:** Migrating from empirical loss approximations to embedding partial differential equations for transient atmospheric thermal diffusion directly into the neural layers[cite: 2].
* **Diffusion-Guided Refinement Modules:** Deploying conditional Denoising Diffusion Probabilistic Models (DDPM) that utilize the flux conservation gradient vectors as reverse-process guidance steps[cite: 2].
* **Vision-Language Prior Injection:** Integrating cross-attention transformer layers to inject regional text metadata (such as local seasonal conditions, geography, and weather logs) to guide ambient color distribution modeling[cite: 1].
* **Multi-Temporal Sequence Modeling:** Introducing recurrent cross-attention networks to analyze temporal thermal inertia patterns across diurnal heating and cooling curves[cite: 1, 2].

---

##  Key Research References & Bibliography

1. **SwinIR Backbone:** Liang, J., Cao, J., Sun, G., Zhang, K., Van Gool, L., & Timofte, R. (2021). *SwinIR: Image Restoration Using Swin Transformer*. IEEE International Conference on Computer Vision Workshops (ICCVW).  
🔗 [https://arxiv.org/abs/2108.10257](https://arxiv.org/abs/2108.10257)
2. **Pix2PixHD Core Framework:** Wang, T. C., Mikołajczyk, A., Zhu, J. Y., Tao, A., Kautz, J., & Catanzaro, B. (2018). *High-Resolution Image Synthesis and Semantic Manipulation with Conditional GANs*. IEEE Conference on Computer Vision and Pattern Recognition (CVPR).  
🔗 [https://arxiv.org/abs/1711.11585](https://arxiv.org/abs/1711.11585)
3. **Physics-Guided Infrared Super-Resolution:** *Thermal-Physics Guided Infrared Image Super-Resolution with Dynamic High-Frequency Amplification*. (2026). ResearchGate Publication Technical Notes.  
🔗 [https://www.researchgate.net/publication/402631438_Thermal-Physics_Guided_Infrared_Image_Super-Resolution_with_Dynamic_High-Frequency_Amplification](https://www.researchgate.net/publication/402631438_Thermal-Physics_Guided_Infrared_Image_Super-Resolution_with_Dynamic_High-Frequency_Amplification)
4. **Conditional Adversarial Foundations:** Isola, P., Zhu, J. Y., Zhou, T., & Efros, A. A. (2017). *Image-to-Image Translation with Conditional Adversarial Networks*. IEEE Conference on Computer Vision and Pattern Recognition (CVPR).  
🔗 [https://arxiv.org/abs/1611.07004](https://arxiv.org/abs/1611.07004)
5. **Emissivity-Guided Texture Preservation:** *Unsupervised Super-Resolution for UAV Thermal Imagery via Diffusion Models with Emissivity-Guided Texture Transfer*. (2026). MDPI Remote Sensing Journal.  
🔗 [https://www.mdpi.com/2072-4292/18/5/815](https://www.mdpi.com/2072-4292/18/5/815)
