<p align="center">
<img src="images/option5.png" alt="ALPACA logo" width='500' height='200' >
</p>

# Automated landmarking through pointcloud alignment and correspondence analysis (ALPACA) 

`ALPACA` provides fast landmark transfer from one or more 3D models and thier associated landmark sets to target 3D model(s) through pointcloud alignment and deformable mesh registration. Unlike the Slicermorph's semi-landmark methods, it does not require presence of fixed landmarks. Optimal set of parameters that gives the best correspondence can be investigated (and outcome can be visualized) in single alignment mode, and then applied to a number of 3D models in batch mode. Invoked first time, `ALPACA` needs your permission to download `open3D` library. Depending on the internet speed, download may take some time but it is a one-time event.

## Module Overview
Open the ALPACA module. 

:pencil2:  If this is the first time you are opening `ALPACA`, it will ask you if you are ok with installing `open3d`. If you are using a Windows machine, the installation process can take a few minutes. 

Otherwise, you should observe the following screen:

<img src="images/1.png">

A closer examination of the module's main menu reveals that there are three main tabs in `ALPACA` : `Single Alignment`, `Batch processing`, and `Advanced Settings`.

* __Single Alignment Options__

  * __Source Model__: Under the `Source Model`, the user is expected to select the `*.ply` mesh file to be used as a template. Note that for single aligment, files must first be loaded into Slicer using the `Data` tab, shown later.
  
  * __Source Landmark Set__: Under the `Source Landmark Set`, the user is expected to select the `*.fcsv` file containing the landmarks to be transferred to the target mesh.
  
  * __Target Model__: Under the `Target Model`, the user is expected to select the `*.ply` mesh file to be used as a target (i.e., the specimen we are interested in predicting landmark positions for).
  

<br />
<p align="center">
<img src="images/2.png">
</p>
<br />

* __Advanced Settings Options__ 

  * __Point Density Adjustment__: Here the user can control the voxel size to be used in subsampling the source mesh. The ideal value here will vary according to the size of the specimen. In general, the goal should be to find a voxel size that results in a total of `5000 - 6000` points per mesh. This is essential for optimal performance and will be discussed further below. Single alignment will report the number of points after subsampling.
  
  * __Acceleration__: The user can check this box to use the optional accelerated deformable registration algorithm available [here](https://github.com/ohirose/bcpd). Windows users must then provide the path to the `win` folder containing the `bcpd.exe` file. Mac or Linux users must compile bcpd in their systems according to the instructions given at the repository. Acceleration will then be applied, cutting runtime by a factor of ~5.

  * __Deformable registration Parameters__: This is generally not recommended for novice users, but hyperparameter tuning can significantly improve the end result. In general, the `Deformable registration` parameters are the most likely ones to improve the quality of the registration. Parameter `Alpha` is a regularization parameter that tends to affect the length of the deformation vectors. Lower values of `Alpha` lead to larger overall deformations, and vice versa. Parameter `Beta`, on the other hand, is a regularization parameter that tends to affect the degree of motion coherence of neighboring points. Large values of `Beta` will lead to greater motion coherence among neighboring points, and vice versa.


<br />
<p align="center">
<img src="images/3.png">
</p>
<br />

## Single Alignment

Now that we are acquainted with the overall layout of the module, let's start by doing an alignment between two example meshes. Download the ALPACA sample data set from the [tutorial sample data folder](../samples/) and switch to the ALPACA module.

We will start by importing the meshes we will be aligning using Slicer's data loading capabilities. Please click the `Data load` button, then navigate to the tutorial data folder, and load the following two meshes:


* `A_J_.ply`: A/J inbred mice are widely used to model cancer and for carcinogen testing given their high susceptibility to carcinogen-induced tumors.
* `NOD_SHILTJ_.ply`: NOD/ShiLtJ inbred mice are widely used as a polygenic model for autoimmune type 1 diabetes.

<img src="images/4.png">


 * If everything worked properly, you should observe something that looks like this:

<img src="images/5.png">


* As can be seen above, the two meshes lie in arbitrary positions in 3D space. Contrary to other approaches present in the literature, `ALPACA` can deal with arbitrary starting points. 

* But let's see `ALPACA` in action. Select those two meshes under the `Single Alignment` tab. 

* We will also load the `*.fcsv` file containing the landmarks that we wish to transfer from the source to the target meshes. Finally, let's select a subsampling voxel size of `0.5mm`.

<img src="images/6.png">

* After selecting all three inputs, we can go ahead and press `Run subsampling`. 

* As seen below, `ALPACA` will print the number of points sampled in each mesh. It is essential to aim for `5000-6000` points per mesh, so the user has the option of pressing the `Run subsampling` as much as needed. A good follow-up exercise to this tutorial is to vary the number of sampled points per mesh and evaluating the impact it has on the performance of the method. 

* The `Run subsampling` button will also load a visual representation of the `Target` pointcloud into the 3D scene (in blue).

<img src="images/7.png">

* Once we are satisfied with the number of sampled points, we can proceed with `rigid registration` steps of the pipeline by using the `Run rigid alignment` button. 

* This step will produce an output corresponding to the visual representation of the alignment between the `Source`(red) and `Target` (blue) pointclouds in the 3D scene. Please feel free to rotate those pointclouds in 3D space to make sure the alignment occured correctly.

<img src="images/8.png">

* Depending on the complexity of the structure of interest, it may be hard to tell if the pointclouds are properly aligned. For that reason, `ALPACA` offers the users the option of displaying the rigid aligned meshes. Please press `Diplay rigid aligned meshes`. You should observe something as seen below. Again, feel free to rotate the 3D surfaces to make sure they are properly aligned. 

* In our example case (mice), you will notice that even though we get a proper alignment between the two strains, the `A_J` mice have a downward curved face when compared to the `NOD` mice. Note how the nasal bones are distant from each other. For that reason, simply transferring the landmarks after the rigid registration step is unlikely to produce good results.

<img src="images/9.png">

* To further improve the quality of the alignment, we need to be able to deform the `Source` mesh to match the `Target` one. We can obtain the deformable alignment by pressing `Run CPD non-rigid registration`. Note that the non-rigid step is by far the longest (time) step of the pipeline. In modern laptops, this should take around 2 or 3 minutes. If the `Acceleration` box was checked earlier, it should take around 30 seconds.

* You should get as an output the final landmark predictions in the form of `Fiducial` points. In the `Single Alignment` tab, the output fiducials are not saved into a file. In part, this is because the `Single Alignment` tab's main role is to find the best combination of parameters necessary to transfer landmarks between specimens. These parameters can then be transferred to the `Batch processing` tab to process an entire specimen folder. 

<img src="images/10.png">

* Note that, as a final step the `Single Alignment` pipeline, the user has the option of visualizing the registered `Source` model (i.e., deformed `Source` mesh). Simply press `Show registered source model` to obtain the deformed mesh (green). Note how the alignment of the nasal bone is much better than prior to the deformed step. The same is true for posterior part of the zygomatic arch.

<img src="images/11.png">

* The changes can be more easily visualized by navigating back the `Data` module and hiding all outputs with the exception of the `Target` (blue) and `Warped Source Model` (green). 

<img src="images/12.png">

## Batch processing 

* As mentioned in the prior section, the main purpose of the `Single Aligmment` tab is to find the best combination of hyperparameters for the task, so that they can be applied to a large array of specimens. In `ALPACA`, the parameters used in `Single Alignment` tab get transferred to the `Batch processing` tab once that tab gets selected. 

* Note that the `Batch processing` tab has much of the same elements as the `Single Alignment` one. Here you can choose to use multiple source models in the `Method` dropdown. This will compare each target model with several different sources to improve accuracy. ALPACA will then ask for the paths of folders containing the source model and landmark files. Ensure that mesh and corresponding landmark files have matching filenames. ALPACA will take the median of all outputs for a given target.

* You must also select a `Target output landmark directory`. Before an output directory gets selected, the `Run-autolandmarking` button cannot be pressed.

<img src="images/13.png">

* So, let's select an output folder! You should note that once the folder is selected, the `Run-autolandmarking` button becomes active. Once pressed, the analysis will proceed by iterating through all `*.ply` files contained in the `Target mesh directory`. Depending on the number of specimens, this may take a considerable amount of time. In most modern laptops, an average of `2-4min` per specimen should be expected, or `30-50s` with acceleration. An output `*.fcsv` file will be produced for each specimen using the same name as the original `*.ply` files.


* The `ALPACA` module can be used synergistically with other `SlicerMorph` modules. Users can use not only manually annotated landmarks in the `ALPACA` pipeline, but also include semi-landmarks that were sampled on the 3D surface using the `Spherical Sampling` lab. The performance of the method can be explored using the GPA module to analyze the transferred landmarks.

