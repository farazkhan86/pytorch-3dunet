# pytorch-3dunet

PyTorch implementation of 3D U-Net based on:

[3D U-Net: Learning Dense Volumetric Segmentation from Sparse Annotation](https://arxiv.org/abs/1606.06650)
Özgün Çiçek, Ahmed Abdulkadir, Soeren S. Lienkamp, Thomas Brox, Olaf Ronneberger

## Prerequisites
- Linux
- NVIDIA GPU
- CUDA CuDNN

## Getting Started

### Dependencies
- pytorch (0.4.1+)
- torchvision (0.2.1+)
- tensorboardx (1.4+)
- h5py
- pytest

Setup a new conda environment with the required dependencies via:
```
conda create -n 3dunet pytorch torchvision tensorboardx h5py pytest -c conda-forge
``` 
Activate newly created conda environment via:
```
source activate 3dunet
```

## Train
```
usage: train.py [-h] --checkpoint-dir CHECKPOINT_DIR --in-channels IN_CHANNELS
                --out-channels OUT_CHANNELS [--interpolate]
                [--layer-order LAYER_ORDER] [--loss LOSS] [--epochs EPOCHS]
                [--iters ITERS] [--patience PATIENCE]
                [--learning-rate LEARNING_RATE] [--weight-decay WEIGHT_DECAY]
                [--validate-after-iters VALIDATE_AFTER_ITERS]
                [--log-after-iters LOG_AFTER_ITERS] [--resume RESUME]

UNet3D training

optional arguments:
  -h, --help            show this help message and exit
  --checkpoint-dir CHECKPOINT_DIR
                        checkpoint directory
  --in-channels IN_CHANNELS
                        number of input channels
  --out-channels OUT_CHANNELS
                        number of output channels
  --interpolate         use F.interpolate instead of ConvTranspose3d
  --layer-order LAYER_ORDER
                        Conv layer ordering, e.g. 'brc' ->
                        BatchNorm3d+ReLU+Conv3D
  --loss LOSS           Which loss function to use. Possible values: [bce, ce,
                        dice]. Where bce - BinaryCrossEntropy (binary
                        classification only), ce - CrossEntropy (multi-class
                        classification), dice - DiceLoss (binary
                        classification only)
  --epochs EPOCHS       max number of epochs (default: 500)
  --iters ITERS         max number of iterations (default: 1e5)
  --patience PATIENCE   number of validation steps with no improvement after
                        which the training will be stopped (default: 20)
  --learning-rate LEARNING_RATE
                        initial learning rate (default: 0.0002)
  --weight-decay WEIGHT_DECAY
                        weight decay (default: 0.0001)
  --validate-after-iters VALIDATE_AFTER_ITERS
                        how many iterations between validations (default: 100)
  --log-after-iters LOG_AFTER_ITERS
                        how many iterations between tensorboard logging
                        (default: 100)
  --resume RESUME       path to latest checkpoint (default: none); if provided
                        the training will be resumed from that checkpoint
```


E.g. fit to randomly generated 3D volume and random segmentation mask (see [train.py](train.py)):
```
python train.py --checkpoint-dir ~/3dunet --in-channels 1 --out-channels 2 --layer-order brc --validate-after-iters 10 --log-after-iters 10 --epoch 50 --learning-rate 0.0001 --weight-decay 0.0005 --interpolate       
```
In order to resume training from the last checkpoint:
```
python train.py --resume ~/3dunet/last_checkpoint.pytorch --in-channels 1 --out-channels 2 --layer-order brc --validate-after-iters 10 --log-after-iters 10 --epoch 50 --learning-rate 0.0001 --weight-decay 0.0005 --interpolate        
```
In order to train on your own data just replace the `_get_loaders` implementation in [train.py](train.py) by returning your own 'train' and 'valid' loaders.

Monitor progress with Tensorboard `tensorboard --logdir ~/3dunet/logs/ --port 8666` (you need `tensorboard` installed in your conda env).
![3dunet-training](https://user-images.githubusercontent.com/706781/45916217-9626d580-be62-11e8-95c3-508e2719c915.png)


## Test
```
usage: predict.py [-h] --model-path MODEL_PATH --in-channels IN_CHANNELS
                  --out-channels OUT_CHANNELS [--interpolate] [--layer-order]

3D U-Net predictions

optional arguments:
  -h, --help            show this help message and exit
  --model-path MODEL_PATH
                        path to the model
  --in-channels IN_CHANNELS
                        number of input channels
  --out-channels OUT_CHANNELS
                        number of output channels
  --interpolate         use F.interpolate instead of ConvTranspose3d
  --layer-order LAYER_ORDER
                        Conv layer ordering, e.g. 'brc' ->
                        BatchNorm3d+ReLU+Conv3D

```

Test on randomly generated 3D volume (just for demonstration purposes). See [predict.py](predict.py) for more info.
```
python predict.py --model-path ~/3dunet/best_checkpoint.pytorch --in-channels 1 --out-channels 2 --interpolate --layer-order brc
```
Prediction masks will be saved to `~/3dunet/probabilities.h5`.

Replace the `_get_dataset` implementation in [predict.py](predict.py) to test the trained model on you own data.
