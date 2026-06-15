# Multimodal Optical Feature Extraction with a Free-Space Photonic Extreme Learning Machine

Paper drive link -https://drive.google.com/file/d/1qBtAQ_IFrtfCs1o31qVBEVgVMR9AKuN6/view?usp=sharing  

NPZ files drive link  - https://drive.google.com/drive/folders/15WQ4clja7MEvxCUGoy_Oq0b7v5QBxdXE?usp=sharing    



## Repository Workflow  
The repository is organized into five major stages:  

1) Optical system setup and verification  
2) Training and testing (ridge regression readout)  
3) Saving feature datasets (.npz)  
4) Lambda optimization and model selection  
5) Kernel and feature-space analysis  

For detailed experimental methodology and hardware specifications, please refer to the paper.  

## Files and their functions  

### 1) optical_driver.py  
Responsible for:  
Camera initialization  
SLM communication  
Optical frame acquisition  
Feature extraction from captured images  
Hardware synchronization  

Run the optical verification routine first to ensure - Camera is detected, SLM is detected,  First-order diffraction is properly aligned
,Features are being extracted correctly

For hardware details and alignment guidelines, see the paper.

### 2) main.py   
Main experiment runner.  
Handles:  
Dataset loading  
Optical feature extraction  
Ridge regression training  
Cross-validation         
Testing  
Result storage

### 3) pelm_core.py  
Contains the core PELM implementation:   
Phase encoding    
Noise embedding  
Fourier embedding  
Feature normalization  
Ridge regression solver  
Evaluation functions  

### 4) Config.py
   
Update the configuration file before running experiments.  
Typical settings include:  
Dataset selection  - MNIST, FSDD, Mushroom, Abalone  
Embedding type  - Noise Embedding, Fourier Embedding  
Number of samples  
Cross-validation settings  
Ridge parameter sweep range  

### 5) kernel.py  
Maps the empirical optical kernel to several theoretical kernels:  
Angular RBF Kernel  
Phase Kernel  
Gaussian Kernel  
Arc-Cosine Kernel (K1)  
Arc-Cosine Kernel (K2)  
Produces:  
Kernel fit plots  
Pearson correlation  
RMSE statistics

### 6) analysis_2.py
Generates:  
CKA Analysis
Linear CKA   
RBF CKA  
Accuracy Comparisons   
Linear vs RBF kernel alignment  
Accuracy vs CKA  
,Kernel Heatmaps  
,Empirical kernel matrices  
,Ideal label kernels  
,Centered kernel products  

### 7) isometry.py  
Evaluates geometric preservation of the optical feature map.    
Generates:    
Distance preservation scatter plots   
Pearson correlation   
Spearman correlation     
Distance-separation statistics   

## Recommended execution order:  

1. Optical system setup   
2. Setup the SLM using configuration manager and follow all the details mentioned in the manual    
3. Add appropriate DLLs in the folders mentioned   
4. Make appropriate changes in config.py     
5. Run main.py
   → Generate optical features         
6. Save features as NPZ , can also compare with the NPZ files given
7. Run lambda_cv.py       
   → Find best λ     
8. Run kernel.py
   → Kernel characterization     
9. Run analysis_2.py    
   → CKA + kernel heatmaps      
10. Run isometry.py     
   → Distance preservation analysis      

