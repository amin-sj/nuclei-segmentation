# Nuclei segmentation
This repository contains deep learning method for nuclei segmentation task.

## 1. Dataset
I used [MoNuSeg](https://ieeexplore.ieee.org/document/8880654) dataset, which is one of the biggest public datasets for the nuclei segmentation task and contains 44 H&E stained tissue images from 9 various organs with the resolution of 1000 * 1000.

<img width="731" alt="image" src="https://user-images.githubusercontent.com/91489940/208888673-cda0414c-2423-4264-96a2-03207621772b.png">

## 2. Method
I wanted to use a U-Net like architecture but nuclei segmentation is an instance segmentation task, so more information should be extracted from the tissue images in order to segment individual nuclei.
One of the classical ways to segment individual nuclei is to use distance transform of the nuclei binary mask and extract local maxima coordinates followed by watershed algorithm to perform region growing and specify the "dams" between overlapping nuclei, so i decided to infer nuclei markers by training a model as well as their binary masks. This way there is no need to manually set a threshold for minimum distance of the distance map or kernel size for smoothing the distance map because otherwise the image will be over-segmented. 

<img width="492" alt="image" src="https://user-images.githubusercontent.com/91489940/208902034-9874520e-fbfe-4c60-b31e-d2fb6f66f897.png">

<img width="661" alt="image" src="https://user-images.githubusercontent.com/91489940/208912438-c0ebfce1-2fe2-46c5-a286-8f28e98bd6d0.png">

### 2.1 Preprocessing
Preprocessing contained of 5 stages:
* Stain normalization: which is common technique in analysis of H&E stained images. Although in deep learning a model expected to learn diffrent color distributions but in nuclei segmentation task due to the limited availibility of supervised datasets it is a necessary step. for selecting reference image, i used [mahbod et al.](https://www.researchgate.net/publication/334185970_A_Two-Stage_U-Net_Algorithm_for_Segmentation_of_Nuclei_in_HE-Stained_Tissues?enrichId=rgreq-9852310c5ddf99bcfd9c12154b524a03-XXX&enrichSource=Y292ZXJQYWdlOzMzNDE4NTk3MDtBUzo3NzgzNTQ2MTc3NTc2OTlAMTU2MjU4NTYxMDU0Ng%3D%3D&el=1_x_3&_esc=publicationCoverPdf) method which was to convert all the training images to grayscale and use the image in which nuclei and background histograms are most differnet.

<img width="618" alt="image" src="https://user-images.githubusercontent.com/91489940/208901677-15f3cad7-dad1-40e8-b96a-a934e6e8c602.png">

* Data augmentation: i used following transforms from [alubumentations](https://github.com/albumentations-team/albumentations) library. 

<img width="486" alt="image" src="https://user-images.githubusercontent.com/91489940/208902628-1ec8c5ac-1b0b-4af2-803b-da50cb8d9219.png">

* Tissue images resized to 1024 * 1024
* Pixel values normalized between [0, 1]
* Modifying provided ground truth: to infer nuclei markers i moidified the provided ground truth. an example of this procedure is shown in figure below.

<img width="736" alt="image" src="https://user-images.githubusercontent.com/91489940/208904347-05f6f021-a252-41bf-925e-06a1aea4a061.png">

### 2.2 Model

<img width="549" alt="image" src="https://user-images.githubusercontent.com/91489940/208905672-f6d4e284-b696-4f93-8ed0-70ce5439c9fa.png">

I used the above model for training which is a [Double U-Net](https://www.researchgate.net/publication/344051841_DoubleU-Net_A_Deep_Convolutional_Neural_Network_for_Medical_Image_Segmentation) proposed by Jha et al. added by another U-Net for segmenting nuclei markers, the added U-Net structure is exactly like the second network of Double U-Net for more information please refer to the original paper. Here, we have 2 outputs, output 1 will give us the binary mask and output 2 will predict the inner part of nuclei, nuclei boundaries and backgroud as modified masks.

### 2.3 Postprocessing
Preprocessing contained of 4 stages:
* Extracting nuclei markers: after getting output 2, to have better masks for nuceli markers, nuceli boundaries subtracted from inner nuclei prediction and morphological erosion applied to obtain better distinction between markers.
* watershed algorithm: after obtaining nuclei markers watershed algorithm used to perform region growing and segmenting overlapping nuclei by using nuclei binary masks gradient and nuclei markers as seed points. 
* remove very small objects
* fill holes in detected objects
