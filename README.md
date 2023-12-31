# AIPI_561_Individual_Project_2 :AutoML

With this project I explored the Azure's no code/zero code AutoML functionalities provided as part of the Azure Machine Learning Studio. I made use of and explored the different components such as Data Stores, Data Labelling service, Jobs, Automated ML, etc. but the one i found the most useful in the real sense of "no/zero code" is the pipeline tool called Designer.So while i used both the Azure CLI and the python SDK, for this specific project I will be detailing how I made use of the Desginer and created a robust training pipeline, deployed the ML model on an  endpoint and gathered the required  metrics.

## Data:

For this project i wanted to explore a computer vision usecase as i have already done a fair amount of work with simple tabular data and NLP tasks with  text data , in Azure's ML studio. Also, since the image data is a binary in nature and has in the past given me a lot issues with the format when i tried to use it directly for the AutoML functionalities, this project gave me a good incentive to sort that out and figure out what was going wrong.

So the data i chose was the publically available brain tumor dataset from Kaggle. This dataset (https://www.kaggle.com/datasets/navoneel/brain-mri-images-for-brain-tumor-detection) has a total of 253 images (not the most ideal number for fine-tuning but will do for a POC) with 98 negative for the brain tumor and the rest positive. So we are essentially training the model to recognize how an MRI being postitive for a tumor looks like. While ideally it would make sense to do a bit of data augmentation to increase the number of negative cases, this case is not too bad even if we consider accuracy as the metric because it isnt really a huge imbalance.

#### When feeding the data to AutoML, if you do it directly as a dataset, it will always show you that the dataset is not supported for your image files. So you need to place your data directly in the default blob storage as a Datastore instead. The good thing about this is that you dont really need to divide the data yourself into train and test folders or the 2 classes (if you have the label info, it is recommended that you divide the data into the 2 classes). You can directly feed the data to the Data Labeling tool to get the labeling done before beginning the training or pre-processing. This labeling task is done using both human labelers and ML assisted labeling, so you have an option to opt out of ML assisted labeling and what 3rd party vendor do you need from the Azure marketplace.These tweaks can be made when you are creating your Data labeling project.

### Samples:

## Designer:

After the data is loaded in the datastore, you can either create an automated ML job to train or test. I preferred using the Designer, which would let me implement different components to create a pipeline. Here i used a pretrained DenseNet model for the training. The designer has different components for different usecases. I picked the computer vision usecase and found components that facilitate image transformations (and other preprocessing utilities) along with a choice of the pretrained models available (DenseNet andd ResNet) for the image classification task.
Once you have the required components, based on the number and types if inputs you can just drag and drop those components and create flow charts using them to define your pipelines. The neat thing about this is that if you change your data at some point, or add more data, you dont really have to bother too much about making any changes to any scripts or in the pipeline. You can simply remove the reference to the previous data component and add the reference to the new one and resubmit the pipeline. 

#### The designer essentially runs the whole pipeline on some compute resource. This could either be a compute cluster or a compute instance. So it is imperative that these be created ahead of firing any jobs from the pipeline. For my experiments i created a dedicated compute instance for which i created startup and shutdown schedules based on when i want the training pipeline to rerun (right now it is on every 6 months cadence). For real time inferencing, if the compute instance is down you will not be able to get any inference as that happens using the CPU resources in the instance. For training purposes my compute instance has a GPU resource, which brings down the training time to about 5 mins for the current data size. Below is the flow chart design of my training pipeline.

<img width="1450" alt="Screen Shot 2023-07-03 at 3 22 53 PM" src="https://github.com/NehaBardeDUKE/AIPI_561_Individual_Project_2/assets/110474064/04818901-f096-443a-aa2a-94723ccc0939">

#### The last bloack i.e Evaluation Results gives the confusion matrix for the classification that was performed. Below is the confusion matrix for my dataset. Here yyou can see that the model does a decent job of identifying the negative images even though the training data there is less than that of the positive samples. The positive classification does take a significant hit. The recall is 0.87 which can be made better for the usecase with more training data.


<img width="426" alt="Screen Shot 2023-07-03 at 3 02 37 PM" src="https://github.com/NehaBardeDUKE/AIPI_561_Individual_Project_2/assets/110474064/25eab6dd-2193-43ee-9f01-c6d2796a5541">

## Endpoint Deployment:

Now that we have the model ready, we can "register" it in Azure. "Registering the model" is nothing but saving the model weights and the metadata so that this fine-tuned model can be used for future usecases to do real time or batch inferencing. Once the model is registered, it can be deployed (exposed as an endpoint or rest api for inference) with the click of a button , which makes everything extremely easy.

#### Once the pipeline run is complete and you register the model, note that this is run cannot be directly used for inferencing, as it takes only the dataset you fed to it earlier as an input. So you need to create a "Real-Time endpoint" and select the model which you want to use for inferencing. What this basically does is it creates another pipeline which takes the required model weights and has Web Input and Web Output components. These components can be tweaked to receive the data from you local machine or as raw data coming as a post request. Below is a snapshot of my Real-Time inferencing pipeline. Here there is an element of scoring, which is why the json fed to the scoring module needs to have the actual label of the image being sent to it. This is just for the POC and if a real user is to interact with this app, I will be removing the scoring model and just pass the prediction forward instead.

<img width="1321" alt="Screen Shot 2023-07-03 at 3 47 12 PM" src="https://github.com/NehaBardeDUKE/AIPI_561_Individual_Project_2/assets/110474064/48d4fc17-2309-4fdb-951c-7d217a76b485">

#### With creating the required rest api endpoint, Azure also provides a functionality to test if the endpoint is healthy and providing the output as expected. This test instance is just picked from the 10% of the test dataset that was split earlier during training phase. It also provides a script that acts like a blueprint of how to consume the api. Based on the usecase you can modify the script. For this project i have only modified it to read a file from my local machine and run the inference.
The python notebook attached in this project provides that implementation.

## Future Scope:

1. Deploy a Flask or gradio application which provides a UI for image upload
2. Create a pipeline without the scoring component for real time inferencing.
3. Increase the dataset and retrain for better fine tuning.

