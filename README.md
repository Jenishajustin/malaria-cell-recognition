# EXP-04 Deep Neural Network for Malaria Infected Cell Recognition

## AIM

To develop a deep neural network for Malaria infected cell recognition and to analyze the performance.

## Problem Statement and Dataset
To develop a deep neural network to accurately identify malaria-infected cells in microscopic blood images. This automated system should achieve high performance in diagnosis, improve treatment decisions, and potentially be deployed in resource-limited settings.Your task would be to optimize the model, possibly by tuning hyperparameters, trying different architectures, or using techniques like transfer learning to improve classification accuracy.
<br>
<img src="https://github.com/Jenishajustin/malaria-cell-recognition/assets/119405070/57a7bb06-410e-49c7-8d26-ffcd4d3bf00c" height=250 width=300>

## Neural Network Model
![Screenshot 2024-05-16 175534](https://github.com/Jenishajustin/malaria-cell-recognition/assets/119405070/4fd6c95f-3080-41d6-a736-cd91d3b08563)



## DESIGN STEPS

1. We begin by importing the necessary Python libraries, including TensorFlow for deep learning, data preprocessing tools, and visualization libraries.
2. To leverage the power of GPU acceleration, we configure TensorFlow to allow GPU processing, which can significantly speed up model training.
3. We load the dataset, consisting of cell images, and check their dimensions. Understanding the image dimensions is crucial for setting up the neural network architecture.
4. We create an image generator that performs data augmentation, including rotation, shifting, rescaling, and flipping. Data augmentation enhances the model's ability to generalize and recognize malaria-infected cells in various orientations and conditions.
5. We design a convolutional neural network (CNN) architecture consisting of convolutional layers, max-pooling layers, and fully connected layers. The model is compiled with appropriate loss and optimization functions.
6. We split the dataset into training and testing sets, and then train the CNN model using the training data. The model learns to differentiate between parasitized and uninfected cells during this phase.
7. We visualize the training and validation loss to monitor the model's learning progress and detect potential overfitting or underfitting.
8. We evaluate the trained model's performance using the testing data, generating a classification report and confusion matrix to assess accuracy and potential misclassifications.
9. We demonstrate the model's practical use by randomly selecting and testing a new cell image for classification.



## PROGRAM

### Name: J.JENISHA

### Register Number: 212222230056
```python
import os
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
from matplotlib.image import imread
from tensorflow.keras.preprocessing.image import ImageDataGenerator
from tensorflow import keras
from tensorflow.keras import layers
from tensorflow.keras import utils
from tensorflow.keras import models
from sklearn.metrics import classification_report,confusion_matrix
import tensorflow as tf
# to share the GPU resources for multiple sessions
from tensorflow.compat.v1.keras.backend import set_session
config = tf.compat.v1.ConfigProto()
config.gpu_options.allow_growth = True # dynamically grow the memory used on the GPU
config.log_device_placement = True # to log device placement (on which device the operation ran)
sess = tf.compat.v1.Session(config=config)
set_session(sess)

%matplotlib inline

my_data_dir = './dataset/cell_images'
os.listdir(my_data_dir)
test_path = my_data_dir+'/test/'
train_path = my_data_dir+'/train/'
os.listdir(train_path)
len(os.listdir(train_path+'/uninfected/'))
len(os.listdir(train_path+'/parasitized/'))
os.listdir(train_path+'/parasitized')[0]
para_img= imread(train_path+
                 '/parasitized/'+
                 os.listdir(train_path+'/parasitized')[0])
plt.imshow(para_img)

# Checking the image dimensions
dim1 = []
dim2 = []
for image_filename in os.listdir(test_path+'/uninfected'):
    img = imread(test_path+'/uninfected'+'/'+image_filename)
    d1,d2,colors = img.shape
    dim1.append(d1)
    dim2.append(d2)
sns.jointplot(x=dim1,y=dim2)
image_shape = (130,130,3)
help(ImageDataGenerator)
image_gen = ImageDataGenerator(rotation_range=20, # rotate the image 20 degrees
                               width_shift_range=0.10, # Shift the pic width by a max of 5%
                               height_shift_range=0.10, # Shift the pic height by a max of 5%
                               rescale=1/255, # Rescale the image by normalzing it.
                               shear_range=0.1, # Shear means cutting away part of the image (max 10%)
                               zoom_range=0.1, # Zoom in by 10% max
                               horizontal_flip=True, # Allo horizontal flipping
                               fill_mode='nearest' # Fill in missing pixels with the nearest filled value
                              )
image_gen.flow_from_directory(train_path)
image_gen.flow_from_directory(test_path)

model = models.Sequential()
model.add(layers.Conv2D(filters=64, kernel_size=(3,3),input_shape=image_shape, activation='relu',))
model.add(layers.MaxPooling2D(pool_size=(2, 2)))
model.add(layers.Conv2D(filters=32, kernel_size=(3,3), activation='relu',))
model.add(layers.MaxPooling2D(pool_size=(2, 2)))
model.add(layers.Flatten())
model.add(layers.Dense(128))
model.add(layers.Activation('relu'))
model.add(layers.Dropout(0.5))
model.add(layers.Dense(1))
model.add(layers.Activation('sigmoid'))

model.compile(loss='binary_crossentropy',
              optimizer='adam',
              metrics=['accuracy'])
model.summary()
batch_size = 16
help(image_gen.flow_from_directory)
train_image_gen = image_gen.flow_from_directory(train_path,
                                               target_size=image_shape[:2],
                                                color_mode='rgb',
                                               batch_size=batch_size,
                                               class_mode='binary')
train_image_gen.batch_size
len(train_image_gen.classes)
train_image_gen.total_batches_seen
test_image_gen = image_gen.flow_from_directory(test_path,
                                               target_size=image_shape[:2],
                                               color_mode='rgb',
                                               batch_size=batch_size,
                                               class_mode='binary',shuffle=False)
train_image_gen.class_indices
results = model.fit(train_image_gen,epochs=2,
                              validation_data=test_image_gen
                             )
model.save('cell_model.h5')
losses = pd.DataFrame(model.history.history)
losses[['loss','val_loss']].plot()
model.metrics_names
model.evaluate(test_image_gen)
pred_probabilities = model.predict(test_image_gen)
test_image_gen.classes
predictions = pred_probabilities > 0.5
print(classification_report(test_image_gen.classes,predictions))
confusion_matrix(test_image_gen.classes,predictions)

```
## OUTPUT

### Training Loss, Validation Loss Vs Iteration Plot
<img src="https://github.com/Jenishajustin/malaria-cell-recognition/assets/119405070/2163fb6d-6cf8-4a26-ba7d-63b923a231e1" height=400 width=500>


### Classification Report
<img src="https://github.com/Jenishajustin/malaria-cell-recognition/assets/119405070/b4bff1f7-af28-420c-9363-f7dedc890bd7" height=200 width=300>


### Confusion Matrix
<img src="https://github.com/Jenishajustin/malaria-cell-recognition/assets/119405070/6da30630-82df-46e5-9657-3658e4f71c81">


### New Sample Data Prediction
<img src="https://github.com/Jenishajustin/malaria-cell-recognition/assets/119405070/27ceaad5-ae38-41cd-bfdd-d9378f961f77" height=400 width=400>


## RESULT
Thus, a deep neural network for Malaria infected cell recognition is developed and the performance is analyzed.
