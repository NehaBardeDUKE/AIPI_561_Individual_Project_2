# AIPI_561_Individual_Project_2 :AutoML

With this project I explored the Azure's no code/zero code AutoML functionalities provided as part of the Azure Machine Learning Studio. I made use of and explored the different components such as Data Stores, Data Labelling service, Jobs, Automated ML, etc. but the one i found the most useful in the real sense of "no/zero code" is the pipeline tool called Designer.So while i used both the Azure CLI and the python SDK, for this specific project I will be detailing how I made use of the Desginer and created a robust training pipeline, deployed the ML model on an  endpoint and gathered the required  metrics.

## Data

For this project i wanted to explore a computer vision usecase as i have already done a fair amount of work with simple tabular data and NLP tasks with  text data , in Azure's ML studio. Also, since the image data is a binary in nature and has in the past given me a lot issues with the format when i tried to use it directly for the AutoML functionalities, this project gave me a good incentive to sort that out and figure out what was going wrong.

So the data i chose was the publically available brain tumor dataset from Kaggle. This dataset (https://www.kaggle.com/datasets/navoneel/brain-mri-images-for-brain-tumor-detection) has a total of 253 images (not the most ideal number for fine-tuning but will do for a POC) with 98 negative for the brain tumor and the rest positive. So we are essentially training the model to recognize how an MRI being postitive for a tumor looks like. While ideally it would make sense to do a bit of data augmentation to increase the number of negative cases, this case is not too bad even if we consider accuracy as the metric because it isnt really a huge imbalance.

#### When feeding the data to AutoML, if you do it directly as a dataset, it will always show you that the dataset is not supported for your image files. So you need to place your data directly in the default blob storage as a Datastore instead. The good thing about this is that you dont really need to divide the data yourself into train and test folders or the 2 classes (if you have the label info, it is recommended that you divide the data into the 2 classes). You can directly feed the data to the Data Labeling tool to get the labeling done before beginning the training or pre-processing. This labeling task is done using both human labelers and ML assisted labeling, so you have an option to opt out of ML assisted labeling and what 3rd party vendor do you need from the Azure marketplace.These tweaks can be made when you are creating your Data labeling project.

### Samples:

## Designer

After the data is loaded in the datastore, you can either create an automated ML job to train or test. I preferred using the Designer, which would let me implement different components to create a pipeline. Here i used a pretrained DenseNet model for the training. The designer has different components like 

