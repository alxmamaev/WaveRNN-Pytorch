# WaveRNN-Pytorch
This repository is a fork of Fatcord's [Alternative](https://github.com/fatchord/WaveRNN) WaveRNN implementation. 
The original model has been significantly simplified to allow real-time synthesis of high fidelity speech. This repository
also contains a C++ library that can be used for real-time speech synthesis on a single CPU core. 

WaveRNN-Pytorch is a **vocoder** - it converts from speech features (i.e. mel spectrograms) to speech sound. On can build a 
complete text-to-speech pipeline by using, for example, Tacotron-2 to turn text into speech features, and then use this 
vocoder to produce a sound file.


# Highlights
* 10 bit quantized wav modeling for higher quality
* Weight pruning for reducing model complexity
* Fast, CPU only, C++ inference library running faster than real time on modern cpu.
* Compressed pruned weight format to make weight files small
* Python bindings for the C++ library
* Can be used with a Tacotron-2 implementation for TTS.

# Planned
* Real time inference on modern ARM processors (e.g. inference on smartphone for high quality TTS) 

# Audio Samples
* See Wiki

# Pretrained Checkpoints
* See "model_outputs" directory

# Requirements
### Training: 
* Python 3
* CUDA >=8.0
* PyTorch >= v1.0
* Python requirements:
>pip install -r requirements.txt
* sudo aptitude install libsoundtouch-dev

### C++ library
* cmake, gcc, etc
* Eigen3 development files
> apt-get install libeigen3-dev
* pybind11 https://github.com/pybind/pybind11

# Installation
Ensure above requirements are met.

```
git clone https://github.com/geneing/WaveRNN-Pytorch.git
cd WaveRNN-Pytorch
pip3 install -r requirements.txt
```

# Build C++ library
```
cd library
mkdir build
cd build
cmake ../src
make
cp WaveRNNVocoder*.so python_install_directory
```

# Usage
## 1. Adjusting Hyperparameters
Before running scripts, one can adjust hyperparameters in `hparams.py`.

Some hyperparameters that you might want to adjust:
* `input_type` (best performing ones are currently `bits` and `raw`, see `hparams.py` for more details)
* `batch_size` - depends on your GPU memory. For 8GB memory, you should use batch_size=64
* `save_every_step` (checkpoint saving frequency)
* `evaluate_every_step` (evaluation frequency)
* `seq_len_factor` (sequence length of training audio, the longer the more GPU it takes)

## 2. Preprocessing

#### Using TTS preprocessing
If you are planning to use this vocoder together with a TTS network (e.g. Tacotron-2) you should train on exactly the same data. 
Each implementation of TTS network uses slightly different definition of "mel-spectrogram". I recommend using TTS preoprocessing.

This code has been tested with [Tacotron-2](https://github.com/Rayhane-mamah/Tacotron-2) and M-AILABS dataset. Example:
```
cd Tacotron-2
python3 preprocess.py --dataset='M-AILABS' --language='en_US' --voice='female' --reader='mary_ann' --merge_books=True --output training_data
```

#### Using WaveRNN-Pytorch preprocessing
If you are using vocoder as standalone library you can use native preprocessing.
This function processes raw wav files into corresponding mel-spectrogram and wav files according to the audio processing hyperparameters.

Example usage:
```
python3 preprocess.py --output-dir training_data /path/to/my/wav/files 
```
This will process all the `.wav` files in the folder `/path/to/my/wav/files` and save them in the default local directory called `data_dir`.

## 3. Training
Start training process. checkpoints are by default stored in the local directory `checkpoints`.
The script will automatically save a checkpoint when terminated by `crtl + c`.


Example 1: starting a new model for training from Tacotron-2 data
```
python3 train.py --dataset Tacotron training_data
```
`training_data` is the directory containing the processed files.

Example 2: starting a new model for training
```
python3 train.py --dataset Audiobooks training_data
```

Example 3: Restoring training from checkpoint
```
python3 train.py training_data --checkpoint=checkpoints/checkpoint0010000.pth
```
Evaluation `.wav` files and plots are saved in `checkpoints/eval`.

## 4. Converting model for C++ library

First you need to train the model for at least (hparams.start_prune+hparams.prune_steps) steps 
to ensure that the model is properly pruned. 

In order to use C++ library you need to convert the trained network to compressed model format.

```
python3 convert_model.py --output-dir model_outputs checkpoints/checkpoint_step000400000.pth
```

Example 1: Use python3 interface to the C++ library
```
import WaveRNNVocoder
import numpy as np

vocoder=WaveRNNVocoder.Vocoder()
vocoder.loadWeights('model_outputs/model.bin')

mel = np.load(fname) #make sure that mel.shape[0] == hparams.num_mels
wav = vocoder.melToWav(mel)
``` 







