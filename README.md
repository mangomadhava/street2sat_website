# Street2Sat

 Street2Sat is a new framework for obtaining large data sets of geo-referenced crop type labels obtained from vehicle mounted cameras that can be extended to other applications.

Paper accepted to ICML 2021 Tackling Climate Change Using AI Workshop. 🎉 
Link coming soon! 

## Setting up the environment
1. Clone the repository
2. Set up and activate a Python virtual environment 
    ```bash
    python3 -m venv venv 
    source venv/bin/activate
    ```
3. Install the dependencies
    ```bash
    pip install -r requirements.txt
    ```
## Usage
#### Notebooks
[Multiple_image_pipeline.ipynb](noteboooks/Multiple_image_pipeline.ipynb) - demonstrates how predictions are generated.

[db_to_shapefile.ipynb](noteboooks/db_to_shapefile.ipynb) demonstrates access to the FireStore database and generation of dataset (geodataframe, shapefile).

#### Google Cloud Platform 
Ensure the [Google Cloud CLI](https://cloud.google.com/sdk/docs/install) is installed.

**Initial Setup (only done once)**
```bash
gsutil mb gs://street2sat-uploaded
```
**Deploying resources (done on every code update)**
```bash
sh deploy.sh
```

## Background
<p></p>
<p>
Ground-truth labels on crop type and other variables are critically needed to develop machine learning methods that use satellite observations to combat climate change and food insecurity. These labels difficult and costly to obtain over large areas, particularly in Sub-Saharan Africa where they are most scarce.  Street2Sat is a new framework for obtaining large data sets of geo-referenced crop type labels obtained from vehicle mounted cameras that can be extended to other applications.
</p>

The Street2Sat pipeline has 6 steps:
<li>Data Collection drives</li>
<li>Image Preprocessing</li>
<li>Crop Type Prediction</li>
<li>Distance Estimation</li>
<li>GPS Coordinate Correction</li>
<li>Quality Assesement and Control</li>

<p></p>
<h5 id="Data Collection">Data Collection</h5>
Data was collected in Western Kenya using car mounted GoPro Cameras. This study will be extended to 5 more countries in the near future.



<h5 id="Image Processing">Image Processing</h5>

An iterated approach on <a href="https://en.wikipedia.org/wiki/Otsu%27s_method">Otsu's Method</a> for thresholding was used to straighten images before they were used to train the Yolo object detection. Otsu's method was run on the whole image, halves of the image, thirds of the image, and fourths of the image, and all the predicted rotation angles were averaged. In addition, the image was blurred using a gaussian filter before running on Otsu's method in order to reduce incorrect rotations resulting from small objects.



<h5 id="pred">Crop Type Prediction</h5>

The <a href="https://github.com/ultralytics/yolov5">Yolo Object Detection</a> framework is used to train and predict where crops were located in the image. The images were labeled for crop type by Nasa Harvest teams with agricultural expert guidance. The trained Yolo model is hosted on Heroku using Flask and MongoDB. The crop type prediction can be run on the web app.



<h5 id="pred">Distance Estimation</h5>

The heights of the predicted bounding boxes are used to predict the distance from the camera to the crop. We calculated the distance to each bounding box and then calculated the average depth for all of the boxes of the same crop type class to get a single depth for the field. We used the following equation to predict distance:

<img src="https://render.githubusercontent.com/render/math?math=d = \frac{l_{focal} * h_{crop} * h_{image}}{h_{bbox} * h_{sensor}}">


The focal length, *l<sub>focal</sub>*, was obtained from the EXIF image metadata; the crop height, *h<sub>crop</sub>*, is from known heights of crops; *h<sub>bbox</sub>* is the height of the bounding box in pixels; *h<sub>image</sub>* is the height of the image in pixels; and the GoPro sensor height *h<sub>sensor</sub>* is 4.55 mm.

<h5 id="gps">GPS Coordinate Estimation</h5>

After obtaining the average distance to the crop in the image, the bearing of the camera had to be calculated. To find the bearing, a velocity vector is calculated between the current image and the closest other image in time that is uploaded. We assume that the camera is pointed 90 degrees orthogonal to the drive direction and relocate the point the average distance in that direction.


<h5 id="gps">Quality Assesement and Control</h5>
Errors could occur in this pipeline due to a variety of factors such as poor lighting, mixed crop fields, occlusions, and object detection errors. In future work, we plan to apply techniques for quality assessment and control (QA/QC) to identify points with possible errors and correct them, e.g., out-of-distribution detection to find outliers located on roads or other objects.
