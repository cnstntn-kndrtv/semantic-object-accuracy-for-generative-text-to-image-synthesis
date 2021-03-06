#   Semantic Object Accuracy for Generative Text-to-Image Synthesis
Code for our paper [Semantic Object Accuracy for Generative Text-to-Image Synthesis](https://arxiv.org/abs/1910.13321).

Summary in our [blog post](https://www.tobiashinz.com/2019/10/30/semantic-object-accuracy-for-generative-text-to-image-synthesis).

Contents:
* [Calculate SOA Scores](#calculate-soa-scores-semantic-object-accuracy)
* [Use Our Model (OP-GAN)](#use-our-model-op-gan)

## Calculate SOA Scores (Semantic Object Accuracy)

Semantic Object Accuracy (SOA) is a score we introduce to evaluate the quality of generative text-to-image models. For this, we provide captions from the MS-COCO data set from which the evaluated model should generate images. We then use a pre-trained object detector to check whether the generated images contain the object that was specified in the caption.

E.g. when an image is generated from the caption `a car is driving down the street` we check if the generated image actually contains a car. For more details check section 4 of our [paper](https://arxiv.org/abs/1910.13321)

How to calculate the SOA scores for a model:

1. Go to ``SOA``. The captions are in ``SOA/captions``
    1. each file is named ``label_XX_XX.pkl`` describing for which labels the captions in the file are
    2. load the file with pickle
        * ```python
             import pickle 
             with open(label_XX_XX.pkl, "rb") as f:
                 captions = pickle.load(f)
          ```
    3. each file is a list and each entry in the list is a dictionary containing information about the caption:
        * ```python
            [{'image_id': XX, 'id': XX, 'idx': [XX, XX], 'caption': u'XX'}, ...]
          ```
        * where ``'idx': [XX, XX]`` gives the indices for the validation captions in the commonly used captions file from [AttnGAN](https://github.com/taoxugit/AttnGAN)
2. Use your model to generate images from the specified captions

    1. each caption file contains the relevant captions for the given label
    2. create a new and empty folder 
    3. use each caption file to generate images for each caption and save the images in a folder within the previously created empty folder, i.e. for each of the labels (0-79) there should be a new folder in the previously created folder and the folder structure should look like this
        * images
            * label_00 -> folder contains images generated from captions for label 0
            * label_01 -> folder contains images generated from captions for label 1
            * ...
            * label_79 -> folder contains images generated from captions for label 79
    4. each new folder (that contains generated images) should contain the string "label_XX" somewhere in its name (make sure that integers are formated to two digits, e.g. "0", "02", ...) -> ideally give the folders the same name as the label files
    5. generate **three images for each caption** in each file
        * exception: for label "00" (person) randomly sample 30,000 captions and generate one image each for a total of 30,000 images
    6. in the end you should have 80 folders in the folder created in the step (2.ii), each folder should have the string "label_XX" in it for identification, and each folder should contain the generated images for this label

3. Once you have generated images for each label you can calculate the SOA scores:
    1. Install requirements from ``SOA/requirements.txt`` (we use Python 3.5.2)
    2. [download](https://www2.informatik.uni-hamburg.de/wtm/software/semantic-object-accuracy/yolov3.weights.tar.gz) the YOLOv3 weights file and save it as ``SOA/yolov3.weights``
    3. run ``python calculate_soa.py --images path/to/folder/created-in-step-2ii --output path/to/folder/where-results-are-saved --gpu 0``

4. If you also want to calculate IoU values check the detailed instructions [here](SOA/README.md)
5. Calculating the SOA scores takes about 30-45 minutes (tested with a NVIDIA GTX 1080TI) depending on your hardware (not including the time it takes to generate the images)
6. More detailed information (if needed) [here](SOA/README.md)

## Use Our Model (OP-GAN)
#### Dependencies
- python 3.5.2
- pytorch 1.1.0

Go to ``OP-GAN``.
Please add the project folder to PYTHONPATH and install the required dependencies:

```
pip install -r requirements.txt
```

#### Data
- MS-COCO:
    - [download](https://www2.informatik.uni-hamburg.de/wtm/software/semantic-object-accuracy/data.tar.gz) our preprocessed data (bounding boxes, bounding box labels, preprocessed captions), save to `data/` and extract
        - the preprocessed captions are obtained from and are the same as in the [AttnGAN implementation](https://github.com/taoxugit/AttnGAN)
        - the generateod bounding boxes for evaluating at test time were generated with code from the [Obj-GAN](https://github.com/jamesli1618/Obj-GAN)
    - obtain the train and validation images from the 2014 split [here](http://cocodataset.org/#download), extract and save them in `data/train/` and `data/test/`
    - download the pre-trained DAMSM for COCO model from [here](https://github.com/taoxugit/AttnGAN), put it into `models/` and extract

#### Training
- to start training run `sh train.sh gpu-ids` where you choose which gpus to train on
    - e.g. `sh train.sh 0,1,2,3`
- training parameters can be adapted via `code/cfg/dataset_train.yml`, if you train on more/fewer GPUs or have more VRAM adjust the batch sizes as needed
- make sure the DATA_DIR in the respective `code/cfg/cfg_file_train.yml` points to the correct path
- results are stored in `output/`

#### Evaluating
- update the eval cfg file in `code/cfg/dataset_eval.yml` and adapt the path of `NET_G` to point to the model you want to use (default path is to the pretrained model linked below)
- run `sh sample.sh gpu-ids` to generate images using the specified model
    - e.g. `sh sample.sh 0`

#### Pretrained Models
- OP-GAN: [download](https://www2.informatik.uni-hamburg.de/wtm/software/semantic-object-accuracy/op-gan.pth.tar.gz), save to `models` and extract


## Acknowledgement
- Code and preprocessed metadata for the experiments on MS-COCO are adapted from [AttnGAN](https://github.com/taoxugit/AttnGAN) and [AttnGAN+OP](https://github.com/tohinz/multiple-objects-gan).
- Code to generate bounding boxes for evaluation at test time is from the [Obj-GAN](https://github.com/jamesli1618/Obj-GAN) implementation.
- Code for using YOLOv3 is adapted from [here](https://pjreddie.com/darknet/), [here](https://github.com/eriklindernoren/PyTorch-YOLOv3), and [here](https://github.com/ayooshkathuria/pytorch-yolo-v3).

## Citing
If you find our model useful in your research please consider citing:

```
@article{hinz2019semantic,
title     = {Semantic Object Accuracy for Generative Text-to-Image Synthesis},
author    = {Tobias Hinz and Stefan Heinrich and Stefan Wermter},
journal   = {arXiv preprint arXiv:1910.13321},
year      = {2019},
}
```
