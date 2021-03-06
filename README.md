# tiger-costume Voice Conversion project

A lightweight tool for performing voice conversion written in Python
with Pytroch and Numpy. It is mainly composed
of two parts. A voice alignment algorithm and a regression model.

[Voice Conversion](#voice-conversion)\
[Alignment](#alignment)\
[Model Training and MLPG](#model-training-and-mlpg)\
[Running the voice conversion](#running-the-voice-conversion)\
[Other resources](#other-resources)


## Voice Conversion

Voice conversion is the task of modifying the speech signal of one speaker
(source speaker) so that it sounds as if it had been pronounced by a different speaker (target speaker).

<img src="images/overview_schematic.png" width="320" height="260" />

We use a parametric method for voice generation. Raw waveforms are converted to
fundamental frequency (f0), spectral envelope coefficients (SP) and band aperiodicity coefficients (AP) for processing
using the WORLD codec. They are then converted to log_f0 and Mel Generalized Cepstral Coefficients
using SPTK. In order to regenerate synthetic waveforms, these conversions
are inverted. Note that this is a lossy process.
A 25ms window is used for utterance alignment. A 5ms window is used for waveform generation.

<img src="images/waveforms.png" width="900" height="200" />

In order to build a training set, we captured ten phrases with about seven words each.
They were repeated ten times by two speakers using the same microphone. Then we separated
 those phrases into words using a silence detector. We align these utterances, frame
by frame, using a modified version of the Dynamic Time Warping (DTW) algorithm.
We use the aligned frames to train a regression model which takes the source speaker's
audio data as input and outputs the converted frame parameters such that the
reconstructed waveform sounds matches that of the target speaker.
In this case we do regression frame by frame with a feed forward network.
However, we use contextual information by including delta features
(parameter time derivatives) and using Maximum Likelihood Parameter Generation MLPG.

For a more in depth overview see the project <a href="slides.pdf" download>slides.pdf</a>

## Alignment
Feature extraction and training corpus generation chain:

<img src="images/generate_train_data.png" width="500" height="250" />

A plot of the DTW cost matrices of a word, along with the optimal path for alignment.

<img src="images/dtw_plots.png" width="500" height="250" />

Two utterances of same word from different speakers, before and after alignment.

<img src="images/fo_alignment.png" width="700" height="400" />

We do this in the [Dataset_Analysis.ipynb](https://github.com/JavierAntoran/tiger-costume/blob/master/Notebooks/Dataset%20Analysis.ipynb) Notebook.

Note that you will have to run the notebook yourself as plotly plots are not displayed automatically.
## Model Training and MLPG

This is shown in the [Run_Models.ipynb](https://github.com/JavierAntoran/tiger-costume/blob/master/Notebooks/Run_Models.ipynb) Notebook.

Note that you will have to run the notebook yourself as Plotly plots are not displayed automatically.

We use a 3 layered Fully connected neural net that parametrises a diagonal
covariance Normal distribution (mean and std for each output) over the regression targets. The inputs/targets are
Generalized Cepstral Coefficients and log f0 with their first and second order
time gradients. We use the outputted covariance matrix as the uncertainty parameter
for MLPG. Both inputs and targets are normalized to have 0 mean and unit variance.

In the following image we show the conversion of a word's fundamental frequency. Although
the predicted values are temporarily aligned with the source values, they take the shape
of the target waveform. Additionally, we can see how MLPG uses slope information in order
to produce a smooth output.\
<img src="images/mlpg_f0.png" width="500" height="300" />

## Running the voice conversion
This script was developed to run on a low resource device like a raspberry pi 3. All dependencies must be satisfied.

```bash
python hobbes.py [-dn  DATASET_NAME -ts_num SAMPLE_NUM] [--harvest]

# Use an audio sample with stonemask algorithm
# Output conversion will be otorrino_10_dio.wav
python hobbes.py -dn otorrino -ts_num 10 

# Capture audio from microphone at 16 KHz for 5 seconds and use harvest algorithm
# Output conversion will be acq_harvest.wav
python hobbes.py --harvest
```


## Other resources

This project uses the WORLD vocoder http://www.kki.yamanashi.ac.jp/~mmorise/world/english/
Implemented through pyWORLD: https://github.com/JeremyCCHsu/Python-Wrapper-for-World-Vocoder

It also uses the Speech Signal Processing Toolkit, SPTK http://sp-tk.sourceforge.net implemented through pySPTK https://github.com/r9y9/pysptk

An assortment of voice generation / conversion publications can be found
in the [papers](https://github.com/JavierAntoran/tiger-costume/tree/master/papers) folder.

**Other VC repos:**
* Merlin voice conversion toolkit (uses neural networks) https://github.com/CSTR-Edinburgh/merlin
* Code for recent GMM based VC publication https://github.com/k2kobayashi/sprocket
* GMM, MLPG based VC tutorial by r9y9 (**very good**) https://r9y9.github.io/nnmnkwii/v0.0.1/nnmnkwii_gallery/notebooks/vc/01-GMM%20voice%20conversion%20(en).html
* GAN based VC https://github.com/r9y9/gantts
* Deep VC without parallel utterances, using a phoneme classifier https://github.com/andabi/deep-voice-conversion


