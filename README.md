## Neural_Network_for_Satellite_Data_Classification
![](https://miro.medium.com/max/2363/1*j4TxWuLiRL-nmjQ89HZMBw.gif)
Deep Learning has taken over the majority of fields in solving complex problems, and the geospatial field is no exception. The title of the article interests you and hence, I hope that you are familiar with satellite datasets; for now, Landsat 5 TM. Little knowledge of how Machine Learning (ML) algorithms work, will help you grasp this hands-on tutorial quickly. For those who are unfamiliar with ML concept, in a nutshell, it is establishing the relationship between a few characteristics (features or Xs) of an entity with its other property (value or label or Y) — we provide plenty of examples (labelled data) to the model so that it learns from it and then predicts values/ labels for the new data (unlabelled data). That is enough of theory brush-up for machine learning!
#The general problem with satellite data:
Two or more feature classes (e.g. built-up/ barren/ quarry) in the satellite data can have similar spectral values, which has made the classification a challenging task in the past couple of decades. The conventional supervised and unsupervised methods fail to be the perfect classifier due to the aforementioned issue, although they robustly perform the classification. But, there are always related issues. Let us understand this with the example below:
![](https://miro.medium.com/max/875/1*YmkGeHsbwKLg64xuOPjT1g.jpeg)
In the above figure, if you were to use a vertical line as a classifier and move it only along the x-axis in such a way that it classifies all the images to its right as houses, the answer might not be straight forward. This is because the distribution of data is in such a way that it is impossible to separate them with just one vertical line. However, this doesn’t mean that the houses can’t be classified at all!
![](https://miro.medium.com/max/875/1*w5z0YYvpEBXINh4VUobxOg.gif)
Let us say you use the red line, as shown in the figure above, to separate the two features. In this instance, the majority of the houses were identified by the classifier but, a house was still left out, and a tree got misclassified as a house. To make sure that not even a single house is left behind, you might use the blue line. In that case, the classifier will cover all the house; this is called a high recall. However, not all the classified images are truly houses, this is called a low precision. Similarly, if we use the green line, all the images classified as houses are houses; therefore, the classifier possesses high precision. The recall will be lesser in this case because three houses were still left out. In the majority of cases, this trade-off between precision and recall holds.
The house and tree problem demonstrated above is analogous to the built-up, quarry and barren land case. The classification priorities for satellite data can vary with the purpose. For example, if you want to make sure that all the built-up cells are classified as built-up, leaving none behind, and you care less about pixels of other classes with similar signatures being classified as built-up, then a model with a high recall is required. On the contrary, if the priority is to classify pure built-up pixels only without including any of the other class pixels, and you are okay to let go of mixed built-up pixels, then a high precision classifier is required. A generic model will use the red line in the case of the house and the tree to maintain the balance between precision and recall.
Data used in the current scope
Here, we will treat six bands (band 2 — band 7) of Landsat 5 TM as features and try to predict the binary built-up class. A multispectral Landsat 5 data acquired in the year 2011 for Bangalore and its corresponding binary built-up layer will be used for training and testing. Finally, another multispectral Landsat 5 data acquired in the year 2011 for Hyderabad will be used for new predictions.
#Since we are using labelled data to train the model, this is a supervised ML approach.
![](https://miro.medium.com/max/875/1*aGi6o1UHH43-PecK6NJjEg.png)
Multispectral training data and its corresponding binary built-up layer
We will be using Google’s Tensorflow library in Python to build a Neural Network (NN). The following other libraries will be required, please make sure you install them in advance:
pyrsgis — to read and write GeoTIFF
scikit-learn — for data pre-processing and accuracy checks
numpy — for basic array operations
Without further delay, let us get started with coding.
Place all the three files in a directory — assign the path and input file names in the script, and read the GeoTIFF files.

The raster module of the pyrsgis package reads the GeoTIFF’s geolocation information and the digital number (DN) values as a NumPy array separately. For details on this, please refer to the pyrsgis page.
Let us print the size of the data that we have read.

Output:
Bangalore multispectral image shape: 6, 2054, 2044
Bangalore binary built-up image shape: 2054, 2044
Hyderabad multispectral image shape: 6, 1318, 1056

As evident from the output, the number of rows and columns in the Bangalore images is the same, and the number of layers in the multispectral images are the same. The model will learn to decide whether a pixel is built-up or not based on the respective DN values across all the bands, and therefore, both the multispectral images should have the same number of features (bands) stacked in the same order.
We will now change the shape of the arrays to a two-dimensional array, which is expected by the majority of ML algorithms, where each row represents a pixel. The convert module of the pyrsgis package will do that for us.
![](https://miro.medium.com/max/875/1*DP57Bv0fS9oIWP2Bdvm4-A.jpeg)
Schemata of restructuring of data

Output:
Bangalore multispectral image shape: 4198376, 6
Bangalore binary built-up image shape: 4198376
Hyderabad multispectral image shape: 1391808, 6

In the seventh line of the code snippet above, we extract all the pixels with the value one. This is a fail-safe to avoid issues due to NoData pixels that often has extreme high and low values.
Now, we will split the data for training and validation. This is done to make sure that the model has not seen the test data and it performs equally well on new data. Otherwise, the model will overfit and perform well only on training data.

Output:
(2519025, 6)
(2519025,)
(1679351, 6)
(1679351,)

The test_size (0.4) in the code snippet above signifies that the training-testing proportion is 60/40.
Many ML algorithms including NNs expect normalised data. This means that the histogram is stretched and scaled between a certain range (here, 0 to 1). We will normalise our features to suffice this requirement. Normalisation can be achieved by subtracting the minimum value and dividing by range. Since the Landsat data is an 8-bit data, the minimum and maximum values are 0 and 255 (2⁸ = 256 values).
Note that it is always a good practice to calculate the minimum and maximum values from the data for normalisation. To avoid complexity, we will stick to the default rage of the 8-bit data here.
Another additional pre-processing step is to reshape the features from two-dimensions to three-dimensions, such that each row represents an individual pixel.
![](https://miro.medium.com/max/875/1*V_UeS8G733mL9TSInMavKA.jpeg)

Output:
(2519025, 1, 6) (1679351, 1, 6) (1391808, 1, 6)

Now that everything is in place, let us build the model using keras. To start with, we will use the sequential model, to add the layers one after the other. There is one input layer with the number of nodes equal to nBands. One hidden layer with 14 nodes and ‘relu’ as the activation function is used. The final layer contains two nodes for the binary built-up class with ‘softmax’ activation function, which is suitable for categorical output. You can find more activation functions here.

Image for post
Neural Network architecture
As mentioned in line 10, we compile the model with ‘adam’ optimiser. (There are several others that you can check.) The loss type that we will be using, for now, is the categorical-sparse-crossentropy. You can check details here. The metric for model performance evaluation is ‘accuracy’.
Finally, we run the model on xTrain and yTrain with two epochs (or iterations). Fitting the model will take some time depending on your data size and computational power. The following can be seen after the model compilation:
![](https://miro.medium.com/max/875/1*f-xOVpBNuRQPIc95whMucQ.jpeg)
Let us predict the values for the test data that we have kept separately, and perform various accuracy checks.

The softmax function generates separate columns for each class type probability values. We extract only for class one (built-up), as mentioned in the sixth line in the code snippet above. The models for geospatial-related analysis become tricky to evaluate because unlike other general ML problems, it would not be fair to rely on a generalised summed up error; the spatial location is the key to the winning model. Therefore, the confusion matrix, precision and recall can reflect a clearer picture of how well the model performs.

![](https://miro.medium.com/max/875/1*itdRMrnWwuwIOvDWrLbVUA.jpeg)

Confusion matrix, precision and recall as displayed in the terminal
As seen in the confusion matrix above, there are thousands of built-up pixels classified as non-built-up and vice versa, but the proportion to the total data size is less. The precision and recall as obtained on the test data are more than 0.8.
You can always spend some time and perform a few iterations to find the optimum number of hidden layers, the number of nodes in each hidden layer, and the number of epochs to get accuracy. Some commonly used remote sensing indices such as the NDBI or NDWI can also be used as features, as and when required. Once the desired accuracy is reached, use the model to predict for the new data and export the GeoTIFF. A similar model with minor tweaks can be applied for similar applications.

Note that we are exporting the GeoTIFF with the predicted probability values, and not its thresholded binary version. We can always threshold the float type layer in a GIS environment later, as shown in the image below.
![](https://miro.medium.com/max/875/1*Rg1Viw8hYkjmBlL04mTGwA.gif)
Hyderabad built-up layer as predicted by the model using the multispectral data
The accuracy of the model has been evaluated already with precision and recall — you can also do the traditional checks (e.g. kappa coefficient) on the new predicted raster. Apart from the aforementioned challenges of satellite data classification, other intuitive limitations include the inability of the model to predict on data acquired in different seasons and over different regions, due to variation in the spectral signatures.
The model that we used in the present article is a very basic architecture of the NN, some of the complex models including Convolution Neural Networks (CNN) have been proven by researchers to produce better results. To get started with CNN for satellite data classification, you can check out this post “Is CNN equally shiny on mid-resolution satellite data?”. The major advantage of such classification techniques is the scalability once the model is trained.
