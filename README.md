## *Spherical Kernel for Efficient Graph Convolution on 3D Point Clouds*
Created by Huan Lei, Naveed Akhtar and Ajmal Mian

![alt text](https://github.com/hlei-ziyan/SPH3D-GCN/blob/master/image/intro_arch.png)

### Introduction
[This work](https://arxiv.org/abs/1909.09287) is a **significant** extension of [our original work](http://openaccess.thecvf.com/content_CVPR_2019/html/Lei_Octree_Guided_CNN_With_Spherical_Kernels_for_3D_Point_Clouds_CVPR_2019_paper.html) presented in IEEE CVPR2019, and is accepted to TPAMI in March 2020.

We propose a spherical kernel for efficient graph convolution of 3D point clouds. 
Our metric-based kernels systematically quantize the local 3D space 
to identify distinctive geometric relationships in the data. Similar to the regular grid CNN kernels, the spherical kernel maintains translation-invariance and asymmetry properties, where the former guarantees weight sharing among similar local structures in the  data and the latter facilitates fine geometric learning. 
The proposed kernel is applied to graph neural networks without edge-dependent filter generation, making it computationally attractive for large point clouds. 
In our graph networks, each vertex is associated with a single point location and edges connect the neighborhood points within a defined range. The graph gets coarsened in the network with farthest point sampling. 
Analogous to the standard CNNs, we define pooling and unpooling operations for our network. 
We demonstrate the effectiveness of the proposed spherical kernel with graph neural networks for point cloud classification and semantic segmentation using ModelNet, **ShapeNet, RueMonge2014, ScanNet and S3DIS datasets**.

In this repository, we release the code and trained models for classification and segmentation.

### Citation
If you find our work useful in your research, please consider citing:

```
@article{lei2020spherical,  
  title={Spherical Kernel for Efficient Graph Convolution on 3D Point Clouds},  
  author={Lei, Huan and Akhtar, Naveed and Mian, Ajmal},  
  journal={IEEE Transactions on Pattern Analysis and Machine Intelligence},  
  year={2020}  
}
```
```
@article{lei2019octree,  
  title={Octree guided CNN with Spherical Kernels for 3D Point Clouds},  
  author={Lei, Huan and Akhtar, Naveed and Mian, Ajmal},  
  journal={IEEE Conference on Computer Vision and Pattern Recognition},  
  year={2019}  
}  
```
### License
Our code is released under MIT License (see LICENSE file for details).

### Installation
Install [Tensorflow](https://www.tensorflow.org/install). The code was tested with Python 3.5, Tensorflow 1.12.0, Cuda 9.0 and Cudnn 7.1.4 on Ubuntu 16.04. The used GPU is NVIDIA Titan XP.   
**Note: while implementing the new tensorlfow operators, we assumed that the GPU supports a block of 1024 threads. 
  
Please compile the cuda-based operations in tf-ops folder using the command
```
(sudo) ./compile.sh
```



### 鬼知道我搞了这么久有多难受！！！
 anoconda3+python3.5+tensorflow==1.2.0+cuda==9.0+cudnn==7.1.4
 1、安装conda
 2、在conda中创建python3.5环境
 3、安装tensorflow、cuda、cudnn
 ### 4、conda多cuda设置
 参考1：https://blog.csdn.net/hizengbiao/article/details/88625044?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.edu_weight&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.edu_weight
 参考2：https://blog.csdn.net/qq_27825451/article/details/89082978
 查看当前cuda版本 nvcc -V
 查看当前cudnn版本（需要根据cuda安装位置确定，我这里cuda就在个人账号下的cuda/cuda-9.0）cat cuda/cuda-9.0/include/cudnn.h | grep CUDNN_MAJOR -A 2
 ### ./compile.sh文件在conda中的使用问题
 chmod +x compile.sh
 修改 compile.sh
 ```
cd tf_ops/convolution
sudo ./tf_conv3d_compile.sh
cd ../nnquery
sudo ./tf_nnquery_compile.sh
cd ../sampling
sudo ./tf_sample_compile.sh
cd ../pooling
sudo ./tf_pool3d_compile.sh
cd ../unpooling
sudo ./tf_unpool3d_compile.sh
cd ../buildkernel
sudo ./tf_buildkernel_compile.sh
 ```
chmod +x convolution/tf_conv3d_compile.sh nnquery/tf_nnquery_compile.sh sampling/tf_sample_compile.sh pooling/tf_pool3d_compile.sh unpooling/tf_unpool3d_compile.sh buildkernel/t
修改上面的xxx.sh(和shell解释器有关，个人感觉原因是你用了#!/usr/bin/env bash，conda安装的环境不能载入到PATH中，可以在.sh文件中用echo $PATH验证,
```
#!/usr/bin/env bash
source /home/gahho/.bashrc
source /home/gahho/anaconda3/bin/activate sph3d_try1
TF_CFLAGS=( $(python -c 'import tensorflow as tf; print(" ".join(tf.sysconfig.get_compile_flags()))') )
TF_LFLAGS=( $(python -c 'import tensorflow as tf; print(" ".join(tf.sysconfig.get_link_flags()))') )

nvcc -std=c++11 -c -o tf_conv3d_gpu.cu.o tf_conv3d_gpu.cu \
  ${TF_CFLAGS[@]} -D GOOGLE_CUDA=1 -x cu -Xcompiler -fPIC

g++ -std=c++11 -shared -o tf_conv3d_so.so tf_conv3d.cpp \
  tf_conv3d_gpu.cu.o ${TF_CFLAGS[@]} -fPIC -I/home/gahho/cuda/cuda-9.0/include -lcudart -L/home/gahho/cuda/cuda-9.0/lib64 ${TF_LFLAGS[@]}
``` 


### Data Preparation
You may need to install [Matlab](https://au.mathworks.com/products/matlab.html). It is required to preprocess the datasets, such as the grid-based downsampling.  
We preprocess each segmentation dataset using the corresponding function under the folder ***preprocessing***:
```
preprocessing/shapenet_removeSingularPoints.m
preprocessing/ruemonge2014_prepare_data.m.m
preprocessing/scannet_prepare_data.m
preprocessing/s3dis_prepare_data.m
```
And then transform the \*.txt files to tfrecord format for fast data feeding in Tensorflow:
```
cd io
python make_tfrecord_modelnet.py 
python make_tfrecord_shapenet.py  
python make_tfrecord_ruemonge2014.py   
python make_tfrecord_scannet.py  
python make_tfrecord_s3dis.py    
python make_tfrecord_s3dis_no_split.py 
```
### Usage  
All of the trained models and our results on ShapeNet and S3DIS can be downloaded from [this link](https://drive.google.com/open?id=1-085Tp4RI3eNbZSlOUo7T_F2qcjB8JeE).
- **ModelNet**
  * To train a model to classify the 40 object classes:
    ```
    cd modelnet40_cls 
    python train_modelnet.py  
    ```
  * To test the classification results:
    ```
    python evaluate_modelnet.py --num_votes=12  
    ```

- **ShapeNet**   
  * To train a model to segment parts of the ***Table*** Category:
    ```
    cd shapenet_seg   
    python train_shapenet.py --shape_name=Table 
    ```
  * To test the segmentation performance of the trained model:
    ```
    python evaluate_modelnet.py  --shape_name=Table  --model_name=xxxx    
    ```

- **RueMonge2014**   
  * train 
    ```
    cd ruemonge2014_seg    
    python train_ruemonge2014.py  
    ```
  * test 
    ```
    python evaluate_ruemonge2014.py  --model_name=xxxx    
    ```

- **ScanNet V2**   
  Download the [ScanNet dataset](https://github.com/ScanNet/ScanNet).
  * train 
    ```  
    cd scannet_seg  
    python train_scannet.py  
    ```
  * test
    ```
    python evaluate_scannet_with_overlap.py  --model_name=xxxx    
    python scannet_block2index_with_overlap.py    
    ```
- **S3DIS**    
  Download the [S3DIS dataset](http://buildingparser.stanford.edu/dataset.html).
  * train  
    ```   
    cd s3dis_seg  
    python train_s3dis.py    
    ```
  * test   
    ```
    python evaluate_s3dis_with_overlap.py --model_name=xxxx    
    python s3dis_block2index_with_overlap.py
    ```
### Merging
The datasets are trained and tested with split blocks. We merge them back into complete scenes using functions under the folder ***post-merging*** in Matlab.
```
post-merging/scannet_merge.m
post-merging/s3dis_merge.m
```
