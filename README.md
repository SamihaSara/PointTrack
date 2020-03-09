# PointTrack: Segment as Points for Efficient Online Multi-Object Tracking and Segmentation

This codebase implements the highly efficient framework for multi-object tracking and segmentation (MOTS) described in: 

[Segment as Points for Efficient Online Multi-Object Tracking and Segmentation](TBD)
Zhenbo Xu, Wei Zhang, Xiao Tan, Wei Yang*, Huan Huang, Shilei Wen, Errui Ding, Liusheng Huang,
Conference on Computer Vision and Pattern Recognition (ECCV) 2020


Our network architecture adopts [SpatialEmbedding](https://github.com/davyneven/SpatialEmbeddings) as the segmentation sub-network. 
The current ranking of PointTrack is available in [KITTI leader-board](http://www.cvlibs.net/datasets/kitti/eval_mots.php). Until now (03/09/2020), PointTrack still ranks first for both cars and pedestrians. The screenshot is saved as `The KITTI Vision Benchmark Suite - MOTS - 20200309.pdf` under this repo.
The detailed task description of MOTS is avaliable in [MOTS challenge](https://www.vision.rwth-aachen.de/page/mots).  


## License

This software is released under a creative commons license which allows for personal and research use only. For a commercial license please contact the authors. You can view a license summary [here](http://creativecommons.org/licenses/by-nc/4.0/).

## Getting started

This codebase showcases the proposed framework named PointTrack for MOTS using the KITTI MOTS dataset. 

### Prerequisites
Dependencies: 
- Pytorch 1.3.1 (and others), please set up an virtual env and run:
```
$ pip install -r requirements.txt
```
- Python 3.6 (or higher)
- [KITTI Images](http://www.cvlibs.net/download.php?file=data_tracking_image_2.zip) + [Annotations](https://www.vision.rwth-aachen.de/media/resource_files/instances.zip)

Note that the scripts for evaluation is included in this repo. After images and instances (annotations) are downloaded, put them under **kittiRoot** and change the path in **repoRoot**/config.py accordingly. 
The structure under **kittiRoot** should looks like:

```
kittiRoot
│   images -> training/image_02/ 
│   instances
│   │    0000
│   │    0001
│   │    ...
│   training
│   │   image_02
│   │   │    0000
│   │   │    0001
│   │   │    ...  
│   testing
│   │   image_02
│   │   │    0000
│   │   │    0001
│   │   │    ... 
```

## Pretrained weights
We provide fine-tuned models on KITTI MOTS (for cars):
- SpatialEmbedding for cars.
- PointTrack for cars.
You can download them via [Baidu Disk](https://pan.baidu.com/s/1Mk9JWNcM1W08EAjhyq0yLA) or [Google Drive](https://drive.google.com/open?id=14Hn4ZztfjGUYEjVd-9FRNB5a-CtBkPXc).


## Testing

You can download pretrained models from the above links. Save these weight file under **repoRoot**/pointTrack_weights.

1.To generate the instance segmentation results:

```
$ python -u test_mots_se.py car_test_se_to_save
```
The segmentation result will be saved according to the config file **repoRoot**/config_mots/car_test_se_to_save.py.

2.To test PointTrack on the instance segmentation results:
```
$ python -u test_tracking.py car_test_tracking_val
```

The pretrained model gets 85.51 sMOTSA for cars on the validation set. 


## Training of PointTrack
The training procedure of instance association is as follows.

1.To generate the segmentation result on the validation set as the instruction of the first step in Testing.

2.To generate the instance DB from videos:
```
$ python -u datasets/MOTSInstanceMaskPool.py
``` 

3.Change the line which loads weights to the default path as follows:
```
checkpoint_path='./pointTrack_weights/PointTrack.pthCar' --> checkpoint_path='./car_finetune_tracking/checkpoint.pth'
```

4.Afterwards start training:
```
$ python -u train_tracker_with_val.py car_finetune_tracking
``` 
The best tracker on the validation set will be saved under the folder specified in **repoRoot**/config_mots/car_finetune_tracking.py.


## Training of SpatialEmbedding

Note that the training of SpatialEmbedding needs KITTI object detection left color images as well as the KINS annotations.
Please download images from [KITTI dataset](http://www.cvlibs.net/download.php?file=data_object_image_2.zip), and unzip the zip file under **kittiRoot**.
Please download two KINS annotation json files from [instances_train.json,instances_val.json](https://github.com/qqlu/Amodal-Instance-Segmentation-through-KINS-Dataset), and put files under **kittiRoot**.

For the training of SpatialEmbedding, we follow the original training setting of [SpatialEmbedding](https://github.com/davyneven/SpatialEmbeddings). 
Different foreground weights are adopted for different classes (200 for cars and 50 for pedestrians). In this following, we take cars for example to explain training procedures. 

0.As there are many in-valid frames in MOTS that contain no cars, we only select these valid frames for training SpatialEmbedding.
 ```
$ python -u datasets/MOTSImageSelect.py
``` 

1.To parse KINS annotations, run:
```
$ python -u datasets/ParseKINSInstance.py
``` 
After this step, KINS annotations are saved under **kittiRoot**/training/KINS/ and **kittiRoot**/testing/KINS/.

2.To generate these crops do the following:
```
$ python -u utils/generate_crops.py
``` 
After this step, crops are saved under **kittiRoot**/crop_KINS. (roughly 92909 crops)

3.Afterwards start training on crops: 
```
$ python -u train_SE.py car_finetune_SE_crop
```

4.Afterwards start finetuning on KITTI MOTS with BN fixed:
```
$ python -u train_SE.py car_finetune_SE_mots
```


## Cite us
TBD


## Contact
If you find a problem in the code, please open an issue.

For general questions, please contact the corresponding author Wei Yang (qubit@ustc.edu.cn).








