
This is the code repository implementing the paper "Predicting Animation Skeletons for 3D Articulated Models via Volumetric Nets".

#### Dependecy and Setup

The project is developed on Ubuntu 16.04 with cuda9.2 + cudnn7.5. 
We suggest to use conda virtual environment, which can be set up as following: 

```
conda create -n AnimSkelVolNet python=3.6
. activate AnimSkelVolNet
pip install numpy, scipy, future, tensorboard, h5py, open3d, tqdm, opencv-python
pip install torch==1.2.0 torchvision==0.4.0 -f https://download.pytorch.org/whl/torch_stable.html
```

#### Data

Our dataset ModelResource has 3,193 models. 
We split it into 80% for training (2,554 models), 10%
for validation (319 models), and 10% for testing. 
All models in fbx format can be downloaded[here](https://umass.box.com/s/pgm8i7of0w1om4anzl9dxignepcaanym).

To use this dataset in this project, we need some pre-process, 
including calculating curvature and shape diameter, 
converting models into SDF voxels, calculating feature size as control parameter. 
Most of these works are done in C-plus-plus. Anyone who is interested 
in that part can easily inplement them with the help of[trimesh](https://gfx.cs.princeton.edu/proj/trimesh2/) 
and[Thea](https://github.com/sidch/Thea). 
We put the data after pre-processing[here](https://umass.box.com/s/o7dki17i431vd3xlvjz32aneq2t20o0o). 
The folder includes:

* obj: all meshes in obj format. We tried to fix them by[meshfix](https://github.com/MarcoAttene/MeshFix-V2.1)
* skel: we save skeleton information in txt files. Each row contains information for a joint, 
which are hierarchical level, joint name, joint position (x, y, z), parent joint name.
* curvature: curvature in voxel grids. Only surface voxels have value, otherwise 0. 
We use curvature along two directions, so the size is (2x88x88x88).
* sd: shape diameter in voxel grids. Only surface voxels have value, otherwise 0.
* vox_82: voxelized model with a resolution of (82x82x82). 
It is padded in the code to get a size of (88x88x88).
* feature size (fs): Our feature size file contains one bone sample per row. 
Each row are (x, y, z) coordinates of a bone sample, followed by the feature size. 
Feature size is calculated by shooting rays on a plane perpendicular to the bone. 
For each bone sample, its "feature size" is the median distance to all nearest hits 
of the rays from it. The file use "new bone" to seperate samples from different bones.

To create the data used directly by the code, see and run our script:

`python gen_dataset.py`. 

Remember to change the root_folder to the directory you uncompress the pre-processed data.

#### Inference
To run forward inference only, you can download a trained model from[here](https://umass.box.com/s/v01quhbc6zaqxtwlr6yk9por1m9cozr7). 
Then you put it into REPO_PATH/checkpoints/volNet/, and run the following command:

`python run_trainval.py -e --resume 'checkpoints/volNet/trained_model.pth.tar' --arch 'v2v_hg' --train-batch 4 --test-batch 4 --output_dir volNet --data_path 'DATA_PATH/model-resource-volumetric.h5' --json_file 'DATA_PATH/model-resource-volumetric.json' --input_feature curvature sd vertex_kde --num_stack 4`

This will output the predicted joint&bone heatmaps, as well as the binary input voxels, 
into a folder called 'results/OUTPUT_DIR'.

To generate the skeleton, you need to run our script:

`python mst_generate.py`

Remember to modify the result folder name and output folder name.

#### Training
To train a model by yourself, run the following command

`python run_trainval.py --arch 'v2v_hg' --data_path 'YOUR_PATH/model-resource-volumetric.h5' --json_file 'YOUR_PATH/model-resource-volumetric.json' --checkpoint 'checkpoints/volNet' --logdir 'logs/volNet' --lr 1e-4 --train-batch 4 --test-batch 4 --input_feature curvature sd vertex_kde --num_stack 4 --epochs 50`
