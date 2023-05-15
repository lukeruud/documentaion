# Backend Model Documentaiton

This document details how the computer vision model backend works and how the training loop operates. 

## Human-in-the-Loop Modeling
This project utilizes human-in-the-loop modeling, which is a process of labeling images by both a human and a computer in the same model. The following steps outline how a generic human-in-the-loop model runs:

1) The training loop begins with a dataset of human-labeled images, which are used as a baseline reference for the model. 
2) The model auto-annotates images based on the human-labeled reference data.
3) A human corrects any errors made by the model.
4) The corrected images are fed back into the model to fine-tune it. 

## Our Model's Loop
The following instructions are a summary of how users interact with the model and how the model predicts labels: 
### Labeling: 
1) Mark the labels in GeoJSON format, which contains a list of polygons.
2) Convert the polygon boundary from geo-coordinates to corresponding pixel coordinates and create a label mask (e.g. (1024 x 1024 x 4) if there are 4 levels of damage) from the list of polygons.
3) Fine-tune or calibrate the model.

### Prediction:

1) Select an area. An image downloads with the area's Geo-Information.
2) Feed the image the model to obtain a localization mask. Note: The localization mask is binary with the same shape as the image.
3) Extract polygons from the binary mask using an island finding algorithm (depth-first-search).
4) Fetch the damage prediction for each polygon. The damage prediction is also a mask with the same shape as the image. Only pixels bounded by the polygons in Step 3 are extracted. Average or mode is used to assign each polygon a single damage level.
5) Map the polygons’ pixel coordinates to the original Geo-coordinates using the mapping matrix found in the original downloaded satellite imagery. Note: There is an offset.
6) Format the list of polygons and their corresponding damage level into the required GeoJSON format and send it back to the frontend.

## Training Model Pipeline
In order to run the model, data is sent through a training model pipeline. This pipeline consists of five steps:
1) Mask generation
2) Data spliting 
3) Training and fine-tuning 
4) Evaluation 
5) Building segmenetation inferencing

Each step's code can be found in the [Pipelines](https://github.com/UMassCDS/RedCross2022/tree/main/backend/ds4cg/pipelines) folder within the RedCross2022 repository. 

The scripts to run each of the steps are found in the [README.md](https://github.com/UMassCDS/RedCross2022/blob/main/README.md) file within the RedCross 2022 repository. 
## Mask Generation
In this project, masks are used to extract an object of interest from a labeled .json file and isolate it as a new .png file for futher analysis. The masks serve as truth values and are used to evalute the model after training. The GeoJSON format is a list of polygons, so this step is necessary to isolate the individual polygons into .png files. 

## Data Splitting
In this project, data is split into three sets: training, validation, and test.

Data uses:
- Training: This data is used to directly train the model. It normally consists of a high percentage of the total data entered into the model. 
- Validation: This data is used to test hyperparameters (values that control the learning process of the model) and prevent overfitting (when a model trains a data set so much that it learns irrellevant information from the data set). Validation data is used while the model trains and can show class imbalances in the data. 
- Test: This data is used to assess the performance of the trained model by introducing new data to the model. The test data is the only data reported by the model. 

## Training and Fine-Tuning

### Training vs. Fine-Tuning

When optimizing a machine learning model, the model is both trained and fine-tuned. Training is the process of feeding a large, unknown dataset to a model so that it can learn patterns, adjust hyperparameters, and improve prediction accuracy. Fine-tuning is the process of giving a trained model new data to improve its accuracy in a given task. In this project, fine-tuning improves the performance of the vision model in regions with unique landscapes or building styles.

### Training Model Basics 

This project uses PyTorch, a machine learning framework written in Python, to train the image-labeling model. The training code uses an optimizer (an algorithm that adjusts attributes within the neural network) and a loss function (a function that calculates the difference between the current model output and the expected model output). As the optimizer reduces the loss function, the model is trained to be more percise. 

## Evaluation

After training and fine-tuning the model, the model needs to be evaluated on its F1 (a machine learning metric that measures performance on classifcation tasks), precision, recall, and accuracy scores. These scores are determined by comparing truth values from the test data to the model output values. Based on these output values, the model can be further fine-tuned for accuracy if necessary. 

## Building Segementation Inferencing 
Finally, the trained model is used to produce a product: machine-labeled satellite images. The following steps summarize what the inferencing step does:
1) The input .png files are converted to a GeoTIFF image, which is a file fromat that contains spatial referencing infromation for satellite imagery. 
2) The polygons are then converted back into a GeoJSON format and a function filters the inputs to a specific geographic bounding box. The filtered polygons are returned alongside their damage levels. 
3) The model takes a filtered image or directory of images as an input, where it applies the `predict` function that outputs an estimated mask. 

After inferencing is complete, the returned images can be used to inform the RedCross on the building damage a particular area suffered or to continue to train and improve the model. 

