# Lip to Speech Synthesis with Visual-Context-Attentional-GAN

This repository contains the PyTorch implementation of the following paper:
> **Lip to Speech Synthesis with Visual-Context-Attentional-GAN**<br>
> Minsu Kim, Joanna Hong, and Yong Man Ro<br>
> Paper: https://proceedings.neurips.cc/paper/2021/file/16437d40c29a1a7b1e78143c9c38f289-Paper.pdf<br>

<div align="center"><img width="90%" src="img/Img.png?raw=true" /></div>

## Preparation

### Requirements
- python 3.7
- pytorch 1.6 ~ 1.8
- torchvision
- torchaudio
- ffmpeg
- av
- tensorboard
- pillow
- librosa
- pystoi
- pesq
- scipy

### Datasets
GRID dataset (video normal) can be downloaded from the below link.
- http://spandh.dcs.shef.ac.uk/gridcorpus/

#### Preprocessing
After download the dataset, preprocess the dataset with the following scripts.<br>
It supposes the data directory is constructed as
```
Data_dir
├── subject
|   ├── video
|   |   └── xxx.mpg
```

1. Extract frames
`Extract_frames.py` extract images and audio from the video. <br>
```shell
python Extract_frames.py --Grid_dir "Data dir of GRID_corpus" --Out_dir "Output dir of images and audio of GRID_corpus"
```

## Training the Model
`main.py` saves the weights in `--checkpoint_dir` and shows the training logs in `./runs`.

To train the model, run following command:
```shell
# Distributed training example for LRW
python -m torch.distributed.launch --nproc_per_node='number of gpus' main.py \
--lrw 'enter_data_path' \
--checkpoint_dir 'enter_the_path_for_save' \
--batch_size 80 --epochs 200 \
--mode train --radius 16 --n_slot 88 \
--augmentations True --distributed True --dataparallel False\
--gpu 0,1...
```

```shell
# Data Parallel training example for LRW
python main.py \
--lrw 'enter_data_path' \
--checkpoint_dir 'enter_the_path_for_save' \
--batch_size 320 --epochs 200 \
--mode train --radius 16 --n_slot 88 \
--augmentations True --distributed False --dataparallel True \
--gpu 0,1...
```

Descriptions of training parameters are as follows:
- `--lrw`: training dataset location (lrw)
- `--checkpoint_dir`: directory for saving checkpoints
- `--batch_size`: batch size  `--epochs`: number of epochs  `--mode`: train / val / test
- `--augmentations`: whether performing augmentation  `--distributed`: Use DataDistributedParallel  `--dataparallel`: Use DataParallel
- `--gpu`: gpu for using `--lr`: learning rate `--n_slot`: memory slot size `--radius`: scaling factor for addressing score
- Refer to `train.py` for the other training parameters

## Testing the Model
To test the model, run following command:
```shell
# Testing example for LRW
python main.py \
--lrw 'enter_data_path' \
--checkpoint 'enter_the_checkpoint_path' \
--batch_size 80 \
--mode test --radius 16 --n_slot 88 \
--test_aug True --distributed False --dataparallel False \
--gpu 0
```
Descriptions of training parameters are as follows:
- `--lrw`: training dataset location (lrw)
- `--checkpoint`: the checkpoint file
- `--batch_size`: batch size  `--mode`: train / val / test
- `--test_aug`: whether performing test time augmentation  `--distributed`: Use DataDistributedParallel  `--dataparallel`: Use DataParallel
- `--gpu`: gpu for using `--lr`: learning rate `--n_slot`: memory slot size `--radius`: scaling factor for addressing score
- Refer to `test.py` for the other testing parameters

## Pretrained Models
You can download the pretrained models. <br>
Put the ckpt in './data/'

**Bi-GRU Backend**
- https://drive.google.com/file/d/1wkgkRWxu7JM0uaNHmcyCpvVz9OFar8Do/view?usp=sharing <br>

To test the pretrained model, run following command:
```shell
# Testing example for LRW
python main.py \
--lrw 'enter_data_path' \
--checkpoint ./data/GRU_Back_Ckpt.ckpt \
--batch_size 80 --backend GRU\
--mode test --radius 16 --n_slot 88 \
--test_aug True --distributed False --dataparallel False \
--gpu 0
```

**MS-TCN Backend**
- https://drive.google.com/file/d/1uHZbmk9fgMqYVfvaoMUe-9XlGvQnXEcS/view?usp=sharing

To test the pretrained model, run following command:
```shell
# Testing example for LRW
python main.py \
--lrw 'enter_data_path' \
--checkpoint ./data/MSTCN_Back_Ckpt.ckpt \
--batch_size 80 --backend MSTCN\
--mode test --radius 16 --n_slot 168 \
--test_aug True --distributed False --dataparallel False \
--gpu 0
```

|       Architecture      |   Acc.   |
|:-----------------------:|:--------:|
|Resnet18 + MS-TCN + Multi-modal Mem   |   85.864    |
|Resnet18 + Bi-GRU + Multi-modal Mem   |   85.408    |

## AVSR
You can also use the pre-trained model to perform Audio Visual Speech Recognition (AVSR), since it is trained with both audio and video inputs. <br>
In order to use AVSR, just use ''tr_fusion'' (refer to the train code) for prediction.

## Citation
If you find this work useful in your research, please cite the paper:
```
@inproceedings{kim2021multimodalmem,
  title={Multi-Modality Associative Bridging Through Memory: Speech Sound Recollected From Face Video},
  author={Kim, Minsu and Hong, Joanna and Park, Se Jin and Ro, Yong Man},
  booktitle={Proceedings of the IEEE/CVF International Conference on Computer Vision},
  pages={296--306},
  year={2021}
}
```
