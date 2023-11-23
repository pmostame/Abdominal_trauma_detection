# Abdominal_trauma_detection
Here I designed a multi-model deep learning architecture to detect presence of trauma in multiple abdominal organs.


**Introduction**:
The goal of this project is to predict the medical condition of SEVERAL distinct abdominal organs using only a single slice of the CT scan of the patient. Although the test data is 2D images, the labeled input data of subjects includes all slices from a 3D scan. Minority of the subjects also have segmentation masks available that indicates the position of each organ in their scans. The output labels of the model should be probability of injury for each organ. Note that some of the organs have different levels of injury (healthy/low/high).

**Approach**: 
My approach to predict the output labels is to serialize two machine learning models as follows: 1) A deep neural network with UNet architecture that learns segmentations from the existing subjects, and expands it across all subjects.
2) Once segmentations of all organs of all subjects are extracted from the UNET, raw scans masked with segmentation probabilities (i.e. UNet output) are fed into separate CNN architectures per organ which outputs the probability of injury in that particular organ.

**UNet**:
Lr scheduler with cyclic triangular pattern that decays over time to avoid plateaus and speed up training.
In case of inferior performance, UNet can be leverage self-supervised learning. In other words, At every epoch, most confident predictions are concatenated with the original train set. This process goes on until all the unlabeled data that were assigned to train set are labeled. Finally, the model is tested on the rest of originally-labeled data for final evaluation. UPDATE: NO NEED FOR THIS!
Focal Tversky loss function was used for UNET. However, alpha and beta was set to 0.5 which makes it identical to Dice measure.
Since number of positive samples in each class (liver vs kidney slices) are largely different, we need to assure slices that lack an organ do not contribute to the loss function. However, Background is included in the loss function, which helps the model to learn avoiding false positives.

**CNN**:
Segmentations are multiplied to real images per organ across slices. Pixels are counted across slices and the slice with highest pixel is identified. 6 slices around that peak slice is selected to input the CNN and other slices (possibly noisy) are ignored.
Separate models are trained for each organ.
BCElosswithlogits is used for organs with only 2 levels of output (healthy/injured)
CrossEntropy is used for organs with more than 2 levels of output (healthy/low/high)

**Augmentations**:
CNN fails to learn without data augmentation since the ratio of class sizes are huge. A separate notebook exists that takes the CT images that were masked with segmentations per organ as input, then creates multiple augmentations of that masked image. This only applies to patients. Finally, number of samples across classes are balances in training the CNNs.

**Data Inspection**:
Implementing these pipelines are impossible without having a deep understanding of the data in hand. I ran a comprehensive exploratory data analysis on this data and the DICOM headers to this goal. 
