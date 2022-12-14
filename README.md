# KaggleRSNAMamograph
A repo for my attempt to win the RSNA Kaggle mammogram screening challenge


The RSNA Screening Mammography Breast Cancer Detection <a href = "https://www.kaggle.com/competitions/rsna-breast-cancer-detection/">RSNA Screening Mammography Breast Cancer Detection</a> is a Kaggle challenge sponsored by the Radiology Society of North America. In a nutshell, breast cancer is one of the most prevalent cancers among females. They are so common that women are  advised to perform routine screening using mammographs. Mammographs are X-Ray images (2D images) taken from the breast that can show some of structures, and anomalies (if any) inside the them. However, to be able to look at and interpret one, you need an experienced radiologist which can be really really expensive. Given that all women above some age limit are advised to perform screening one a year, it would be really beneficial if we could have a machine learning model that can take the mammography and provide a risk assessment of how likely it is for the person to have cancer.

In this challenge we have about 54000 mamograms from ... patients. Since mamograms are taken from each breast individaully, and from multiple angels, each patient will have an average of 4 images.(There are a lot of missing images though). These mamograms are stored in the dicom format. For all these images, we know if a biopsy was performed and what the outcome was (cancer / no-cancer) for each breast. The final aim for the competition is to get the highest probabilistic F1 score for predicting the chance of having cancer, given the imagings in the test data. In the training data, each row is an image and contains the following information: patient id, image id, age, breast side, imaging view, cancer or no cancer, biopsy or not, and some other information. (More information available on the competition webpage).

<h2> Methods: </h2>

The first step in developing this pipeline is to perform some preprocessing on the dicom files to make them easier to use in the future. Each dicom file contains some meta-data and an image that can have formats like jpeg, png,... . In addition, these images are quite large (more than 5000*5000) that adds up to a total of 300 Gb of data. My preprocessing includes extracting the image from each dicom file using the pydicom package to a numpy package. Since about half of the imagings have a format called jpeg2000, we need additional packages to preprocess those. These imagings are particularly time-consuming to process and might take 1-2 seconds each. After extracting the images, using the cv2 package I resized them into [512,512] arrays and normalized them to the [0-255] range. Then, I stored these arrays as png files for future use. (Total size of 3.2 Gbs)

I also created a list of final transformations to perform on the images before feeding them into the model. Since most of the models I am planning to apply use input size of [224, 224], I resize the images to this size, I will also need to increase the number of channels to meet model input requirements. To do this, for now I just concatenate three of the same image. (For now, I process each image individually, and not at a patient level). I also perform random horizontal flip and normalize the image based on resnet model recommendations. For the validation and test set, I only resize and normalize.

Next step is dividing the dataset into train and validation sets (Final train set is not accessible). Since the dataset is very imbalanced (only 1054 images end up to be cancer, belonging to 464 patients.) For initial training I randomly downsample on the cancer-free patients to twice the number of patients with cancer. Note that I also have to divide the dataset on a patient level rather than imaging level to prevent any dataleak between the two sets. Accounting for the number of patients with cancer, I randomly divide the data into train and validation sets with a 0.8/0.2 ratio. For now, I only use the image as input and whether there was cancer or not as output as a binary classification task.

I used and compared the performance of the following models: resnet18, resnext101 and ViT32. For this transfer learning task, I took each model and removed the final fully connected layer to replace it with another fully connected layer that will have only one output. I used the pretrained weights for each model. I trained each model for 20 epochs and with a momentum of 0.9. I also used dropout and weight decay for all models. I used SGD and reduced the learning rate by multiplying it by 0.8 on each epoch.

<h2> Evaluation </h2>

For my evaluation, I used the prbabilistic F1 score (Same as the criteria in the competition) and accuracy. Since I myself balaned the dataset accuracy is also a relevant outcome measure. I will also provide train, test learning curves for each of the models.

<h2> Results</h2>

<h2> Future Directions </h2>


