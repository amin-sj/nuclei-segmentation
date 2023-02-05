# Nuclei segmentation
This repository contains deep learning method for nuclei segmentation task.

## 1. Dataset
We used [MoNuSeg](https://monuseg.grand-challenge.org/Data/) dataset, which is one of the biggest public datasets for the nuclei segmentation task and contains 44 H&E stained tissue images from 9 various organs with the resolution of 1000 * 1000.

<img width="731" alt="image" src="https://user-images.githubusercontent.com/91489940/208888673-cda0414c-2423-4264-96a2-03207621772b.png">

## 2. Method
We wanted to use a U-Net like architecture but nuclei segmentation is an instance segmentation task, so more information should be extracted from the tissue images in order to segment individual nuclei.
One of the classical ways to segment individual nuclei is to use distance transform of the nuclei binary mask and extract local maxima coordinates followed by watershed algorithm to perform region growing and specify the "dams" between overlapping nuclei, so we decided to infer nuclei markers by training a model as well as their binary masks. This way there is no need to manually set a threshold for minimum distance of the distance map or kernel size for smoothing the distance map because otherwise the image will be over-segmented. 

<img width="492" alt="image" src="https://user-images.githubusercontent.com/91489940/208902034-9874520e-fbfe-4c60-b31e-d2fb6f66f897.png">

<img width="661" alt="image" src="https://user-images.githubusercontent.com/91489940/208912438-c0ebfce1-2fe2-46c5-a286-8f28e98bd6d0.png">

### 2.1 Preprocessing
Preprocessing contained of 5 stages:
* Stain normalization: which is common technique in analysis of H&E stained images. Although in deep learning a model expected to learn diffrent color distributions but in nuclei segmentation task due to the limited availibility of supervised datasets it is a necessary step. for selecting reference image, we used [mahbod et al.](https://www.researchgate.net/publication/334185970_A_Two-Stage_U-Net_Algorithm_for_Segmentation_of_Nuclei_in_HE-Stained_Tissues?enrichId=rgreq-9852310c5ddf99bcfd9c12154b524a03-XXX&enrichSource=Y292ZXJQYWdlOzMzNDE4NTk3MDtBUzo3NzgzNTQ2MTc3NTc2OTlAMTU2MjU4NTYxMDU0Ng%3D%3D&el=1_x_3&_esc=publicationCoverPdf) method which was to convert all the training images to grayscale and use the image in which nuclei and background histograms are most differnet.

<img width="618" alt="image" src="https://user-images.githubusercontent.com/91489940/208901677-15f3cad7-dad1-40e8-b96a-a934e6e8c602.png">

* Data augmentation: we used following transforms from [alubumentations](https://github.com/albumentations-team/albumentations) library. 

<img width="486" alt="image" src="https://user-images.githubusercontent.com/91489940/208902628-1ec8c5ac-1b0b-4af2-803b-da50cb8d9219.png">

* Tissue images resized to 1024 * 1024
* Pixel values normalized between [0, 1]
* Modifying provided ground truth: to infer nuclei markers we moidified the provided ground truth. an example of this procedure is shown in figure below.

![image](https://user-images.githubusercontent.com/91489940/216271274-dedb9349-6f85-4449-9c8f-5da4cfc03fd8.png)

### 2.2 Model

We used [U-Net](https://arxiv.org/abs/1505.04597) and its more recent variants like [U-Net ++](https://link.springer.com/chapter/10.1007/978-3-030-00889-5_1), [U-Net 3+](https://ieeexplore.ieee.org/abstract/document/9053405) and [Double U-Net](https://www.researchgate.net/publication/344051841_DoubleU-Net_A_Deep_Convolutional_Neural_Network_for_Medical_Image_Segmentation) to compare their performances. For backbone of the U-Net, U-Net ++ and U-Net 3+ VGG-19 was used because of its similar architecture to U-Net, also another decoder added to these models to extract ternary masks (modified masks) as well as the binary masks simultaneously. Double U-Net model was not moidified as it has 2 outputs, output1 used to extract binary masks and output2 used to extract ternary masks.

<!-- #### U-Net with 2 decoders and VGG-19 backbone (Trainable params: 33,667,340)
![U-Net_DD](https://user-images.githubusercontent.com/91489940/216282352-df065408-41d7-45cb-8eef-e98806058c9b.png)

#### U-Net ++ with 2 decoders and VGG-19 backbone (Trainable params: 25,485,520)
![U-Net++_DD](https://user-images.githubusercontent.com/91489940/216287069-f320c31d-1926-4e65-9d73-b3a5aa681249.png)

#### U-Net 3+ with 2 decoders and VGG-19 backbone (Trainable params: 23,193,172)
![U-Net3+_DD](https://user-images.githubusercontent.com/91489940/216282754-0fbb55cd-badb-4ff3-92f0-b2c199f90c56.png)

#### Double U-Net model (Trainable params: 34,676,517)
![Double U-Net](https://user-images.githubusercontent.com/91489940/216285719-4b4362c1-f843-48a2-8475-130ca63f0aaa.png) -->

### 2.3 Postprocessing
Preprocessing contained of 4 stages:
* Extracting nuclei markers: after getting output 2, to have better masks for nuceli markers, nuceli boundaries subtracted from inner nuclei prediction and morphological erosion applied to obtain better distinction between markers.
* watershed algorithm: after obtaining nuclei markers watershed algorithm used to perform region growing and segmenting overlapping nuclei by using nuclei binary masks gradient and nuclei markers as seed points. 
* remove very small objects
* fill holes in detected objects

## 3. Results
For evaluation we performed 5-fold cross-validation and we used 3 metrics of AJI, PQ and DICE where AJI and PQ are sensitive to both semantic and instance segmentation performance and DICE is sensitive to semantic segmentation performance. Also we included the results of U-Net without adding the second decoder to measure the impact of adding another decoder and extracting nuclei markers information as well as binary masks for nuclei segmentation.

<img width="525" alt="image" src="https://user-images.githubusercontent.com/91489940/216344382-36c3e755-e078-4455-a84d-4bc76353f85f.png">

After performing cross-validation, models trained on whole training set to get the final results of the models on monuseg test set.

<img width="347" alt="image" src="https://user-images.githubusercontent.com/91489940/216349402-0a7c18d2-e8dd-43b0-ab8c-abfac581e2bb.png">

From the above tables it is evident that nuclei marker extraction improves the model's ability to separate nuclei (as evidenced by improved PQ and AJI scores) while not having a significant impact on overall semantic segmentation performance (expressed by the average DICE score). Among the other 4 models used in the research, Double U-Net has performed better in terms of AJI and PQ scores, but no significant change is seen in the cross-validation scores.

#### Qalitative results on different organs of monuseg test dataset
![image](https://user-images.githubusercontent.com/91489940/216361294-07f15dc8-c321-48ac-bc8b-633c5a587f69.png)
