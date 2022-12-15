# KaggleRSNAMamograph
A repo for my attempt to win the RSNA Kaggle mammogram screening challenge.

https://user-images.githubusercontent.com/101527689/207956984-25113ad4-af2d-4054-9406-b4d2175dbd42.mp4"
 
The RSNA Screening Mammography Breast Cancer Detection <a href = "https://www.kaggle.com/competitions/rsna-breast-cancer-detection/">RSNA Screening Mammography Breast Cancer Detection</a> is a Kaggle challenge sponsored by the Radiology Society of North America. In a nutshell, breast cancer is one of the most prevalent cancers among females. They are so common that women are advised to perform routine screening using mammography. Mammographs are X-Ray images (2D images) taken from the breast that can show some of the structures, and anomalies (if any) inside them. However, to be able to look at and interpret one, you need an experienced radiologist which can be expensive. Given that all women above some age limit are advised to perform screening once a year, it would be beneficial if we could have a machine learning model that can take the mammography and provide a risk assessment of how likely it is for the person to have cancer.

In this challenge, we have about 54000 mammograms from about 11000 patients. Since mammograms are taken from each breast individually, and with multiple views, each patient will have an average of 4 images. (There are a lot of missing images though). These mammograms are stored in the Dicom format. For all these images, we know if a biopsy was performed and what the outcome was (cancer / no-cancer) for each breast. The final aim of the competition is to get the highest probabilistic F1 score for predicting the chance of having cancer, given the imagings in the test data. In the training data, each row is an image and contains the following information: patient id, image id, age, breast side, imaging view, cancer or no cancer, biopsy or not, and some other information. (More information is available on the competition webpage).

<h2> Methods: </h2>

The first step in developing this pipeline is to perform some preprocessing on the Dicom files to make them easier to use in the future. Each Dicom file contains some meta-data and an image that can have formats like jpeg, png,... . In addition, these images are quite large (more than 5000*5000) which adds up to a total of 300 Gb of data. My preprocessing includes extracting the image from each Dicom file using the pydicom package to a NumPy package. Since about half of the imagings have a format called jpeg2000, we need additional packages to preprocess those. These imagings are particularly time-consuming to process and might take 1-2 seconds each. After extracting the images, using the cv2 package I resized them into [512,512] arrays and normalized them to the [0-255] range. Then, I stored these arrays as png files for future use. (Total size of 3.2 Gbs) This step took a whole two days.

I also created a list of final transformations to perform on the images before feeding them into the model. Since most of the models I am planning to apply use an input size of [224, 224], I resize the images to this size, I will also need to increase the number of channels to meet model input requirements. To do this, for now, I just concatenate three of the same image. (For now, I process each image individually, and not at a patient level). I also perform random horizontal flip and normalize the image based on resnet model recommendations. For the validation and test set, I only resize and normalize.

The next step is dividing the dataset into train and validation sets (The final train set is not accessible). Since the dataset is very imbalanced (only 1054 images end up being cancer, belonging to 464 patients.) For initial training, I randomly downsample the cancer-free patients to twice the number of patients with cancer. Note that I also have to divide the dataset on a patient level rather than an imaging level to prevent any data leak between the two sets. Accounting for the number of patients with cancer, I randomly divide the data into train and validation sets with a 0.8/0.2 ratio. For now, I only use the image as input and whether there was cancer or not as output as a binary classification task.

I used and compared the performance of the following models: resnet18 and resnext101. For this transfer learning task, I took each model and removed the final fully connected layer to replace it with another fully connected layer that will have only one output. I used the pre-trained weights for each model. I trained each model for 20 epochs with a momentum of 0.9. I also used dropout and weight decay for all models. I used SGD and reduced the learning rate by multiplying it by 0.8 on each epoch.

<h2> Evaluation </h2>

For my evaluation, I used the probabilistic F1 score (Same as the criteria in the competition) and accuracy. Since I balanced the dataset accuracy is also a relevant outcome measure. I will also provide train, and test learning curves for each of the models.


<h2> Results</h2>

For the Resnet18 model, after training for 20 epochs here are the learning curves:

![ResNet_1  Train loss_ResNet_1](https://user-images.githubusercontent.com/101527689/207950457-52f9cf3b-3f2e-4ba5-ad52-c28c6bbae093.png)
![ResNet_1 Net  Test loss_ResNet_1](https://user-images.githubusercontent.com/101527689/207950479-be86f83f-8813-47aa-a027-a34d306456a6.png)
![ResNet_1 Test accuracy_ResNet_1](https://user-images.githubusercontent.com/101527689/207950500-e806b15a-e7c8-407b-92d9-f3f64e959d00.png)
![ResNet_1 Probabilistic F1s_ResNet_1](https://user-images.githubusercontent.com/101527689/207950561-65f9e2f7-0129-472c-98ca-4a9c1f761b56.png)
![confusion_matrix_ResNet_6](https://user-images.githubusercontent.com/101527689/207948878-65f98fc9-0921-445d-9274-6ece202d4641.png)

Here on this model, we reached a pF1 score of 0.36 and an accuracy of 0.57 on the test set. (The current leading team has a pF1 score of 0.55).

And here are the plots for the resNext101 model:

![ResNext_4  Train loss_ResNext_4](https://user-images.githubusercontent.com/101527689/207948236-5fff66ba-3d3b-4de6-8c50-fcd12ca9da20.png)

![ResNext_4  Train Accuracy_ResNext_4](https://user-images.githubusercontent.com/101527689/207948204-abf28e90-20df-4470-bf2a-b7ba81d148da.png)

![ResNext_4 Test accuracy_ResNext_4](https://user-images.githubusercontent.com/101527689/207948287-8ce7d359-4a70-4175-8dfb-395a5cd4d984.png)

![ResNext_4 Probabilistic F1s_ResNext_4](https://user-images.githubusercontent.com/101527689/207948315-52833c7e-8a36-458c-b8bf-eb2f99ebbbb9.png)

![confusion_matrix_ResNext_5](https://user-images.githubusercontent.com/101527689/207950620-d01bef17-6d27-4ebc-90e8-204eaf713ca6.png)



<h2> Future Directions </h2>

For this project, I quickly ran out of GPU training time in Kaggle for this week. I understand that my models need more hyperparameter tuning and more advanced data preprocessing. I am planning to perform the following steps to improve this model:

1. Currently, I analyze each mammogram separately. It would be ideal if I could pass all the images from the same breast at once in different channels. However, due to a problem with a lot of missing images I have to perform some cleaning first. After cleaning I am going to put two images with different views of the same breast in the first two channels and put a similar image from the breast of the other side (Usually only one breast is involved so this would provide a good point of comparison) on the third channel. But first, I have to come up with a plan to deal with missing imagings.

2. In this project, I passed the whole mammogram. However, this can be inefficient because sometimes the breast only takes a proportion of the whole image. Someone has already created a labeled dataset for object detection at Kaggle. I am going to use that dataset to train my own object detector and only pass that to the model to improve efficacy.

3. The dataset includes some additional information that could be helpful. For example, we know which mammograms were selected for biopsy (So they were more likely to have cancer), and also for some, we have risk scoring (BIRADS). If I add that information as an outcome for my training, I believe the performance of the model will increase. especially since the dataset is imbalanced.


