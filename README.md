# 3D Object Detection from Stereo Images

This repository contains a partial implementation of CVPR 2018 paper ['Multi-Level Fusion based 3D Object Detection from Monocular Images'](http://openaccess.thecvf.com/content_cvpr_2018/papers/Xu_Multi-Level_Fusion_Based_CVPR_2018_paper.pdf) by Xu, Chen et al. based on the awesome open source codebase [`maskrcnn-benchmark`](https://github.com/facebookresearch/maskrcnn-benchmark). It is worth noting that depth estimation module(monodepth) in the original paper is replaced with a more accurate module([PSMNet](https://github.com/JiaRenChang/PSMNet)) to generate disparity maps from stereo images.



## Highlights
 - [**`Box3dList`**](https://github.com/abbyxxn/maskrcnn-benchmark-3d/blob/master/maskrcnn_benchmark/structures/bounding_box_3d.py#L11) is a structure holds a set of 3D bounding boxes represented as a Nx7 'ry-lwhxyz' tensor in one image. It also contains a set of utility methods that allow to perform geometric transformations to the bounding boxes (such as scaling and flipping).
 - [**`box3d_head`**](https://github.com/abbyxxn/maskrcnn-benchmark-3d/tree/master/maskrcnn_benchmark/modeling/roi_heads/box3d_head) is a building block similar to `roi_heads/box_head` in 2d, implemented to regress size, dimension and rotation of 3D bounding box.
 - [**`orientation_coder`**](https://github.com/abbyxxn/maskrcnn-benchmark-3d/blob/master/maskrcnn_benchmark/modeling/orientation_coder.py) the implementation of Multibin in an independent module, separated into encode and decode like modeling/box-coder.
 - [**`kittiDataset`**](https://github.com/abbyxxn/maskrcnn-benchmark-3d/blob/master/maskrcnn_benchmark/data/datasets/kitti3d.py) a subclasses of torch.utils.data.Dataset, this module support for KITTI dataset and evaluate.
 - provide some useful functions convert KITTI annotation to json format, if you want to train 2d object detection on KITTI with original powerful annotation processing tools cocoapi/PythonAPI/pycocotools.
 - [**`Tensorboard`**](https://github.com/abbyxxn/maskrcnn-benchmark-3d/blob/master/maskrcnn_benchmark/utils/miscellaneous.py#L28) support for training visualization.

### Installation

For environment requirements and compilation, please refer to [maskrcnn-benchmark](https://github.com/facebookresearch/maskrcnn-benchmark).
You can check INSTALL.md for installation instructions instead.

Clone the repository

```
git clone https://github.com/abbyxxn/maskrcnn-benchmark-3d
```

## Perform training on KITTI dataset
To train on the Kitti Object Detection Dataset:
 - Download the KITTI object detection dataset, calib, label and place it in your home folder at ~/kitti/object
 - Follow chen, et al. [here](https://xiaozhichen.github.io/files/mv3d/imagesets.tar.gz) to split dataset into train, val and test sets.
 - It is worth noting that download color images of object data set instead of temporally preceding frames from KITTI wesite, because annotations of 3D object detector is just prepared for single frame image.
 - And make sure to put the files as the following structure:

```
kitti
    object
        testing
        training
            calib
            image_2
            label_2
            depth
        ImageSets
            train.txt
            val.txt
            trainval.txt
        annotations
            kitti_train_car_gt_roidb.pkl
            kitti_val_car_gt_roidb.pkl
```

Recommend to symlink the path to the kitti dataset to `datasets/` as follows

```bash
# symlink the kitti dataset
cd ~/github/maskrcnn-benchmark-3d
mkdir -p datasets/kitti
ln -s /path_to_kitti_dataset/object datasets/kitti/object

```

You can also configure your own paths to the datasets. For that, all you need to do is to modify `maskrcnn_benchmark/config/paths_catalog.py` to point to the location where your dataset is stored.
You can also create a new `paths_catalog.py` file which implements the same two classes, and pass it as a config argument `PATHS_CATALOG` during training.
### Single GPU training

Most of the configuration files that we provide assume that we are running on 8 GPUs.
In order to be able to run it on fewer GPUs, there are a few possibilities:

**1. Run the following without modifications**

```bash
python /path_to_maskrcnn_benchmark/tools/train_net.py --config-file "/path/to/config/file.yaml"
```


**2. Modify the cfg parameters**

Here is an example for Mask R-CNN R-50 FPN with the 1x schedule:
```bash
python tools/train_net.py --config-file "configs/e2e_mask_rcnn_R_50_FPN_1x.yaml" SOLVER.IMS_PER_BATCH 2 SOLVER.BASE_LR 0.0025 SOLVER.MAX_ITER 720000 SOLVER.STEPS "(480000, 640000)" TEST.IMS_PER_BATCH 1
```


### Multi-GPU training
We use internally `torch.distributed.launch` in order to launch multi-gpu training. This utility function from PyTorch spawns as many Python processes as the number of GPUs we want to use, and each Pythonprocess will only use a single GPU.
```bash
export NGPUS=8
python -m torch.distributed.launch --nproc_per_node=$NGPUS /path_to_maskrcnn_benchmark/tools/train_net.py --config-file "path/to/config/file.yaml"
```

## Authors

* **Xuan Xiong**
This project is based on [maskrcnn-benchmark](https://github.com/facebookresearch/maskrcnn-benchmark). Thanks Francisco Massa and his colleagues for their great work!


## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details.




