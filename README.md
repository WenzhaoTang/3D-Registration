# Praktikum on 3D Vision - 3D Registration


## Install requirements

Download surfemb:

```shell
$ git clone https://github.com/WenzhaoTang/3D-Registration.git
$ cd surfemb
```

[Install conda](https://docs.conda.io/projects/conda/en/latest/user-guide/install/index.html)
, create a new environment, *surfbase*, and activate it:

```shell
$ conda create --name surfbase python=3.8
$ pip install -r requirements.txt
$ conda activate surfbase
```

## Prepare Datasets
<p align="center">
<img src="/src/texture1.png" alt="Texture 1" width="150" /> <img src="/src/texture2.png" alt="Texture 2" width="150" />
<img src="/src/texture3.png" alt="Texture 3" width="150" /> <img src="/src/texture4.png" alt="Texture 4" width="150" /> <img src="/src/texture5.png" alt="Texture 15" width="150" /> 
</p>
Our datasets can be downloaded through the following link in accordance with the BOP's format: [Niubility](https://bop.felk.cvut.cz/datasets/).

Extract the datasets under ```data/bop``` (or make a symbolic link).

The following images display the rendered objects with applied and selected patterns:
<img src="/src/p1.png" width="250" /> <img src="/src/p2.png" width="250" /> <img src="/src/p3.png" width="250" />

## Training
To observe differences, train a model using varying numbers of epochs.
Configure the following settings in the training script:
```shell
wandb.log({'epoch': num})
```

The value of ```num``` can be selected from the following options: ```5```, ```10```, or ```20```.

| number of epochs | convergent speed | perceptibility of differences |
|------------------|------------------|-------------------------------|
| 20 epochs        | convergence      | imperceptible                 |
| 10 epochs        | near convergence | barely noticeable             |
| 5 epochs         | no convergence   | obvious                       |

The following figure illustrates this concept:

<img src="/src/5_10_20.png" width="600" />

```shell
$ python -m surfemb.scripts.train [dataset] --gpus [gpu ids]
```

For example, to train a model on *T-LESS-Nonetextured* on *cuda:0*

```shell
$ python -m surfemb.scripts.train tless --gpus 0
```

## Inference data

We use the detections from [CosyPose's](https://github.com/ylabbe/cosypose) MaskRCNN models, and sample surface points
evenly for inference.  
For ease of use, this data can be downloaded and extracted as follows:

```shell
$ wget https://github.com/rasmushaugaard/surfemb/releases/download/v0.0.1/inference_data.zip
$ unzip inference_data.zip
```

**OR**

<details>
<summary>Extract detections and sample surface points</summary>

### Surface samples

First, flip the normals of ITODD object 18, which is inside out. 

Then remove invisible parts of the objects

```shell
$ python -m surfemb.scripts.misc.surface_samples_remesh_visible [dataset] 
```

sample points evenly from the mesh surface

```shell
$ python -m surfemb.scripts.misc.surface_samples_sample_even [dataset] 
```

and recover the normals for the sampled points.

```shell
$ python -m surfemb.scripts.misc.surface_samples_recover_normals [dataset] 
```

### Detection results

Download CosyPose in the same directory as SurfEmb was downloaded in, install CosyPose and follow their guide to
download their BOP-trained detection results. Then:

```shell
$ python -m surfemb.scripts.misc.load_detection_results [dataset]
```

</details>

## Inference inspection

To see pose estimation examples on the training images run

```shell
$ python -m surfemb.scripts.infer_debug [model_path] --device [device]
```
Here is an example of inference on a training image:

*[device]* could for example be *cuda:0* or *cpu*. 

Add ```--real``` to use the test images with simulated crops based on the ground truth poses, or further
add ```--detections``` to use the CosyPose detections.

## Inference for BOP evaluation

Inference is run on the (real) test images with CosyPose detections:

```shell
$ python -m surfemb.scripts.infer [model_path] --device [device]
```

Pose estimation results are saved to ```data/results```.  
To obtain results with depth (requires running normal inference first), run

```shell
$ python -m surfemb.scripts.infer_refine_depth [model_path] --device [device]
```

The results can be formatted for BOP evaluation using

```shell
$ python -m surfemb.scripts.misc.format_results_for_eval [poses_path]
```

Either upload the formatted results to the BOP Challenge website or evaluate using
the [BOP toolkit](https://github.com/thodan/bop_toolkit).

## Extra

Custom dataset:
Format the dataset as a BOP dataset and put it in *data/bop*.# 3D-Registration
# 3D-Registration
