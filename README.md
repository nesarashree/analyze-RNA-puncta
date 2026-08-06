# An open-source image analysis pipeline for single-cell mRNA puncta quantification and colocalization
This repository provides a reproducible, batch-analysis pipeline for quantifying mRNA puncta in confocal microscopy images, designed for experiments with excitatory (VGLUT) and inhibitory (GAD) neuron cell-type masks. The pipeline enables accurate detection, segmentation, and per-cell analysis of mRNA puncta, facilitating high-throughput analysis of RNAscope datasets.
<p align="center">
  <img src="images-for-README/analysispipeline.png" width="700">
</p>

# Abstract: Society for Neuroscience Conference 2026

A Computational Pipeline For Analyzing In-Situ Hybridization Images to Detect Cell-Type Specific Changes In Alzheimerʼs Model Mice
**Authors:** N. SHREE, M. S. MENDES, J. TOMORSKY, J. HUANG, C. J. SHATZ; Departments of Biol. & Neurobio., Stanford Univ., Stanford, CA

Alzheimer's disease (AD) is a complex neurodegenerative disease that manifests in the aging adult brain. Yet growing transcriptomic evidence suggests that neuron-intrinsic molecular disruptions emerge far earlier, before cognitive decline and before conventional tools can reliably detect them. Understanding these earliest changes requires cell-type resolution that bulk sequencing cannot provide, such as RNA scope in situ hybridization. Existing analysis tools for single-cell mRNA quantification are often inaccessible or inflexible. We developed a trainable, open-source pipeline to detect and quantify neuronal changes. We hypothesized that synaptic changes in APP/PS1 mice at postnatal day 30 (P30) are accompanied by cell-type-specific transcriptional alterations, detectable prior to overt neuropathology. To test this, RNAscope fluorescence in situ hybridization was performed on layer 2/3 visual cortex brain sections from APP/PS1 and wild-type Ribotag mice (n=3-4 per genotype). Five candidate genes identified in neuronal Ribotag experiments were significantly upregulated at P30 in APP/PS1 mice. Their expression was quantified using ImageJ's open-access WEKA machine learning classifier trained on a three-class scheme (puncta, background, & cellular noise). A custom MATLAB application performed batch colocalization with VGLUT-positive excitatory and GAD-positive inhibitory neuron masks. Validation against ground truth annotations revealed no significant diﬀerence in puncta counts (paired t-test, p>0.05), with classifier precision of 0.95 and recall of 0.84. Critically, the pipeline processes a full batch (6-10 images) with colocalization metrics in minutes, compared to the hours of manual annotation completed for validation. Using our custom computational method, we discovered cell-type-specific dysregulation in APP/PS1 mice. Inhba was bidirectionally regulated, elevated in excitatory but decreased in inhibitory neurons. Penk and Pde10a were selectively upregulated in excitatory neurons; Acvr1c was exclusively upregulated in inhibitory neurons. These findings reveal cell-type-specific transcriptional disruption preceding AD pathology and establish a validated open-source tool for single-cell mRNA quantification in various disease models.

## WEKA Trainable Classifier Integration (FIJI)
The FIJI Trainable Weka Segmentation (TWS) plugin integrates the image-processing framework of FIJI with the machine learning algorithms of WEKA to enable supervised and unsupervised image segmentation. Using a limited set of user-provided pixel annotations, TWS extracts multiscale image features and trains a classifier that can be interactively refined to segment complex biological images.
<p align="center">
  <img src="images-for-README/classifieroutput.png" width="1000">
</p>

The classifier model in this repository ( [Google Drive download link](https://drive.google.com/file/d/1DyxPmiG2cWH34HeAA1bCvjoVOSbJI5gX/view?usp=sharing) ) was trained on ~15 INHBA mRNA puncta images with 3 training (or classification) labels: puncta, background, and noise.

<p align="center">
  <img src="images-for-README/train.png" width="600">
</p>

**Post-processing of WEKA classification results (batch)**

Apply *watershed* to segment clumps of detected puncta (in FIJI: Process -> Binary -> Watershed, or use a .macro automation for a folder of images). Before applying the watershed, ensure the output is binarized (in FIJI: Image -> Threshold -> B&W). 
<p align="center">
  <img src="images-for-README/watershed.png" width="900">
</p>

## Custom MATLAB GUI for quantification & colocalization metrics
* Creates count masks for puncta quantification and exports numeric data (CSV), including puncta area and total count per image.
<p align="center">
  <img src="images-for-README/countmask.png" width="400">
</p>

* Loads folders of excitatory (VGLUT) and inhibitory (GAD) masks to compute colocalization metrics per cell, including mean puncta per cell, # non-coloc puncta, density, etc.
<p align="center">
  <img src="images-for-README/GUI.png" width="1000">
</p>

* Designed for **batch analysis**! Process large datasets of images reproducibly and efficiently using the "batch count" / "batch coloc" buttons. Numeric info is exported to CSV for each image in the dataset.
<p align="center">
  <img src="images-for-README/csv.png" width="700">
</p>

## EXAMPLE WORKFLOW
**1. Adjust original mRNA images in FIJI**
The purpose of this step is to auto-adjust brightness/contrast of vGLUT / Gad mRNA original readouts from RNAscope so that they can be run through the Weka trainable classifier model.
* Open FIJI
* Process -> Batch -> Macro
* Run the following macro on input folder of original mRNA image data to auto-adjust brightness: 
<p align="center">
  <img src="images-for-README/adjustmacro.png" width="400">
</p>

**2. Classify puncta with trainable classifier (WEKA)**
Download example model (trained on RNAscope images for INHBA) here: [Google Drive download link](https://drive.google.com/file/d/1DyxPmiG2cWH34HeAA1bCvjoVOSbJI5gX/view?usp=sharing), or train your own in FIJI -> Plugins -> Trainable WEKA Segmentation
* Once WEKA app is loaded, click "Load Classifier" -> download and choose the pretrained model above
* "Apply Classifier" -> select only 8-10 raw images at a time (based on computer memory! FIJI will crash!)
* Popup will ask if you want results stored locally instead of opened in Fiji. When prompted, click yes and create a results folder for that batch for FIJI to store the classification results in.
* Popup will ask if you want to generate a “Probability Map,” click yes
* Let it run (approx 2 mins)! When it is finished (see log), ensure probability maps saved to results folder
* Repeat in batches until all images have been classified
RESULT: Probability maps of detected puncta in the original mRNA images!

<p align="center">
  <img src="images-for-README/classified.png" width="600">
</p>

**3. Threshold & segment classification results**
The purpose of this step is to turn the 3-layer probability map (one layer per training class: noise, background, and puncta) into a binary B&W image that we can watershed to segment clumped puncta. Essentially, “flattening” the model’s output for further analysis in MATLAB.
* Open FIJI
* Process -> Batch -> Macro
* To generate the macro, I applied the following to the classified images generated by Weka:
Image -> Adjust -> Threshold (dark background, B&W) -> Apply -> Convert to Mask
Process -> Binary -> Watershed
Save image
* Emulate these steps to write / record the macro and batch process classifier result images, then click “Process”
NOTE: save images to a new output folder path, choose input folder as the classification output images

**4. Quantify / colocalize puncta in MATLAB GUI**
* Download latest GUI from this repo and run in MATLAB
* Click “Load Puncta,” open folder of generated “ready-to-count” puncta masks
* Click “Load Cells,” open folder of vGLUT / Gad cell masks
* Set min px = 30 (filter outlier specks)
* Click “Count All Puncta” for one image OR “Batch Count” for all at once
* Click “Batch Coloc” to colocalize all at once
* Export CSV and graph results!

<p align="center">
  <img src="images-for-README/workflow.png" width="1000">
</p>
