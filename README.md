# Nuclei segmentation
This repository contains a deep learning method for nuclei segmentation task.

## 1. Dataset
We used the [MoNuSeg](https://monuseg.grand-challenge.org/Data/) dataset, one of the biggest public datasets for the nuclei segmentation task, which contains 44 H&E stained tissue images from 9 organs with a resolution of 1000 * 1000.

<img width="731" alt="image" src="https://user-images.githubusercontent.com/91489940/208888673-cda0414c-2423-4264-96a2-03207621772b.png">

## 2. Method
We wanted to use a U-Net-like architecture, but nuclei segmentation is an instance segmentation task, so additional information should be generated from the tissue images to segment individual nuclei. One of the classical ways to segment individual nuclei is to use the distance transform of the nuclei binary mask and extract local maxima coordinates,  followed by a watershed algorithm to perform region growing and specify the "dams" between overlapping nuclei. We decided to directly predict individual nuclei markers by training a model in addition to binary mask prediction. This way, there is no need to manually set a threshold for a minimum distance of the distance map or kernel size for smoothing the distance map to obtain individual nuclei markers by extracting local maxima.

<img width="492" alt="image" src="https://user-images.githubusercontent.com/91489940/208902034-9874520e-fbfe-4c60-b31e-d2fb6f66f897.png">

<img width="495" alt="image" src="https://user-images.githubusercontent.com/91489940/220624962-07971354-8759-4c21-84ea-2bbfdf070d45.png">

### 2.1 Preprocessing
Preprocessing contained five stages:
* Stain normalization: This is a common technique in analyzing H&E stained tissue images. Although in deep learning, a model is expected to learn different color distributions, in the nuclei segmentation task, it is a necessary step due to the limited availability of supervised datasets. For selecting reference images, we used [mahbod et al.](https://www.researchgate.net/publication/334185970_A_Two-Stage_U-Net_Algorithm_for_Segmentation_of_Nuclei_in_HE-Stained_Tissues?enrichId=rgreq-9852310c5ddf99bcfd9c12154b524a03-XXX&enrichSource=Y292ZXJQYWdlOzMzNDE4NTk3MDtBUzo3NzgzNTQ2MTc3NTc2OTlAMTU2MjU4NTYxMDU0Ng%3D%3D&el=1_x_3&_esc=publicationCoverPdf) method to convert all the training images to grayscale and use the image in which nuclei and background histograms are most different. 

<img width="618" alt="image" src="https://user-images.githubusercontent.com/91489940/208901677-15f3cad7-dad1-40e8-b96a-a934e6e8c602.png">

* Data augmentation: we used the following transforms from the [alubumentations](https://github.com/albumentations-team/albumentations) library. 

<img width="486" alt="image" src="https://user-images.githubusercontent.com/91489940/208902628-1ec8c5ac-1b0b-4af2-803b-da50cb8d9219.png">

* Tissue images resized to 1024 * 1024
* Pixel values normalized between [0, 1]
* Modifying provided ground truth: We modified the provided ground truth to predict nuclei markers. An example of this procedure is shown in the figure below.

![image](https://user-images.githubusercontent.com/91489940/216271274-dedb9349-6f85-4449-9c8f-5da4cfc03fd8.png)

### 2.2 Model

We used [U-Net](https://arxiv.org/abs/1505.04597) and its more recent variants like [U-Net ++](https://link.springer.com/chapter/10.1007/978-3-030-00889-5_1), [U-Net 3+](https://ieeexplore.ieee.org/abstract/document/9053405), and [Double U-Net](https://www.researchgate.net/publication/344051841_DoubleU-Net_A_Deep_Convolutional_Neural_Network_for_Medical_Image_Segmentation) to compare their performances. For the backbone of the U-Net, U-Net ++, and U-Net 3+, we used VGG-19  because of its similar architecture to U-Net. Also, we added another decoder unit to these models to extract ternary masks (modified masks) and binary masks simultaneously. We didn't modify the Double U-Net model as it has two outputs, output 1 used to extract binary masks and output 2 used to extract ternary masks.

<!-- #### U-Net with 2 decoders and VGG-19 backbone (Trainable params: 33,667,340)
![U-Net_DD](https://user-images.githubusercontent.com/91489940/216282352-df065408-41d7-45cb-8eef-e98806058c9b.png)

#### U-Net ++ with 2 decoders and VGG-19 backbone (Trainable params: 25,485,520)
![U-Net++_DD](https://user-images.githubusercontent.com/91489940/216287069-f320c31d-1926-4e65-9d73-b3a5aa681249.png)

#### U-Net 3+ with 2 decoders and VGG-19 backbone (Trainable params: 23,193,172)
![U-Net3+_DD](https://user-images.githubusercontent.com/91489940/216282754-0fbb55cd-badb-4ff3-92f0-b2c199f90c56.png)

#### Double U-Net model (Trainable params: 34,676,517)
![Double U-Net](https://user-images.githubusercontent.com/91489940/216285719-4b4362c1-f843-48a2-8475-130ca63f0aaa.png) -->

### 2.3 Postprocessing
Postprocessing contained four stages:
* Extracting nuclei markers: after obtaining output 2, to have better masks for nuclei markers, nuclei boundary prediction was subtracted from inner nuclei prediction, and morphological erosion was applied to obtain better distinction between markers.
* Watershed algorithm: after obtaining nuclei markers watershed algorithm based on nuclei markers and nuclei binary masks was used to segment individual nuclei.
* Remove very small objects
* Fill holes in detected objects

## 3. Results
For evaluation, we performed 5-fold cross-validation. We used three metrics of AJI, PQ, and DICE, where AJI and PQ are sensitive to both semantic and instance segmentation performance, and DICE is sensitive to semantic segmentation performance. Also, we included the results of U-Net without adding the second decoder to measure the impact of adding another decoder unit and extracting nuclei markers information as well as binary masks for nuclei segmentation.

<img width="646" alt="image" src="https://user-images.githubusercontent.com/91489940/220625830-21f903d9-3380-4fe6-9d9d-05a996a7cccc.png">

After performing cross-validation, to evaluate the generalization ability of the models and compare our method with algorithms proposed in the monuseg2018 challenge, we train the models on the whole training set to get the final results of the models on the monuseg test set.

<img width="347" alt="image" src="https://user-images.githubusercontent.com/91489940/216349402-0a7c18d2-e8dd-43b0-ab8c-abfac581e2bb.png">

<img width="654" alt="image" src="https://user-images.githubusercontent.com/91489940/220626460-7cda36a9-ec81-4c5d-9e76-10c9bb13f213.png">

From the above tables, it is evident that nuclear marker extraction improves the model's ability to differentiate nuclei, as evidenced by improved PQ and AJI scores compared to the raw U-Net. However, it does not considerably impact the general semantic segmentation performance, as expressed by the mean DICE score. Among the other four models used in the research that employed the proposed method, Double U-Net exhibited the highest average scores of AJI and PQ. To further analyze the performance of these four models, a one-way ANOVA test was conducted, which did not reveal a significant statistical difference. Consequently, when using the pre-trained VGG-19 as an encoder for U-Net, U Net ++, U-Net 3+, and Double U-Net networks, these networks show similar performance on the MoNuSeg dataset.

#### Qualitative results on different organs of the monuseg test dataset
<img width="389" alt="image" src="https://user-images.githubusercontent.com/91489940/220628448-8afc53f0-57da-4509-84c0-11938b2a4ecd.png">
