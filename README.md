# Nuclei segmentation
This repository contains deep learning method for nuclei segmentation task.

## 1. Dataset
I used MoNuSeg dataset, which is one of the biggest public datasets for the nuclei segmentation task and contains 44 H&E stained tissue images from 9 various organs with the resolution of 1000 * 1000.

<img width="731" alt="image" src="https://user-images.githubusercontent.com/91489940/208888673-cda0414c-2423-4264-96a2-03207621772b.png">

## 2. Method
I wanted to use a U-Net like architecture but nuclei segmentation is an instance segmentation task, so more information should be extracted from the tissue images in order to segment individual nuclei.
One of the classical ways to segment individual nuclei is to use distance transform of the nuclei binary mask and extract local maxima coordinates followed by watershed algorithm to implement region growing and specify the "dams" between overlapping nuclei, so i decided to infer nuclei markers by training a model as well as their binary masks. This way there is no need to manually set a threshold for minimum distance of the distance map or kernel size for smoothing the distance map because otherwise the image will be over-segmented. 

<img width="492" alt="image" src="https://user-images.githubusercontent.com/91489940/208902034-9874520e-fbfe-4c60-b31e-d2fb6f66f897.png">

### 2.1 Preprocessing
Preprocessing contained of 5 stages:
* Stain normalization: which is common technique in analysis of H&E stained images. Although in deep learning a model expected to learn diffrent color distributions but in nuclei segmentation task due to the limited availibility of supervised datasets it is a necessary step. for selecting reference image, i used mahbod et al. method which was to convert all the training images to grayscale and use the image in which nuclei and background histograms are most differnet.

<img width="618" alt="image" src="https://user-images.githubusercontent.com/91489940/208901677-15f3cad7-dad1-40e8-b96a-a934e6e8c602.png">

* Data augmentation: i used following transforms from alubumentations library. 

<img width="486" alt="image" src="https://user-images.githubusercontent.com/91489940/208902628-1ec8c5ac-1b0b-4af2-803b-da50cb8d9219.png">

* Tissue images resized to 1024 * 1024
* Pixel values normalized between [0, 1]
* Modifying provided ground truth: to infer nuclei markers i moidified the ground truth. an example of this procedure is shown in figure below.

<img width="736" alt="image" src="https://user-images.githubusercontent.com/91489940/208904347-05f6f021-a252-41bf-925e-06a1aea4a061.png">
