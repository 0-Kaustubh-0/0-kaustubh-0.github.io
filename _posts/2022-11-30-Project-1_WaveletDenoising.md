---
title: A Comprehensive Guide to Wavelet Denoising in MATLAB and Python
date: 2022-11-30 -500
categories: [MATLAB, Python]
tags: [Wavelets, denoising, wavelet transform, thresholding, discrete wavelet transform]
published: true
---

<style>body {text-align: justify} </style>


# Introduction

Wavelet denoising arises from the [Wavelet Transform][1], specifically the [Discrete Wavelet Trasform (DWT)][1]. DWT can be utilized for:

* Filtering
* Frequency specific feature extraction
* Downsampling a time-series
* Obtaining high resolution temporo-spectral information on a signal
* and Denoising 

Wavelet denosiing is a very powerful technique for removing noise from signals and images while preserving most relevant features. It has applications in various fields such as image processing, signal processing, and data analysis. In this guide, we will explore how to perform wavelet denoising in  `MATLAB` and `Python`.


## Understanding Wavelet Denoising

Before diving into the implementation, let's briefly understand the concept of wavelet denoising. Wavelet denoising involves decomposing a signal or image into wavelet coefficients and then applying a thresholding operation to remove unwanted noise components. The key steps in the process are as follows:

* **Signal Decomposition:** Decompose the signal into wavelet coefficients using a chosen wavelet transformation. This transformation has options on the wavelet of choice and the level of decomposition
* **Thresholding:** The process of applying a threshold to the wavelet coefficients, where coefficients below a certain threshold are considered as noise and set to zero, while the rest are retained. 
* **Inverse Transformation:** Reconstruct the denoised signal from the modified wavelet coefficients.


### Signal decomposition

Single level decomposition can be utilized to decompose the data by **just one level** and throw out **detailed** and **approximate** coefficients, . While that is sufficient towards denoising the data, there is an advantage to the single-level DWT decomposition which can be utilized more precisely. Single-level DWT can also be utlized for denoising when utilized in a loop flow structure to expore different thresholding schemes, which can be explored in numerous ways, for ex:
    
* cDetailed_level_N can be removed/ thresholded
* cApprox_level_N can be given a special statistical threshold
* Reconstruct backwards using `idwt` *only upto a certain level* to obtained desired results 

> With Multi-level decomposition, the thresholding occurs **ONCE ALL the levels** have been decomposed   


### Thresholding

#### Value selection:

Statistical hard/ soft thresholding: Various peer-reveiewed methods have been developed to idenitfy custom threshold values based on the type of application. For example, these are some of the thresholding methodologies:

* modwtsqtwolog - [Maximal overlap DWT (pg 164)][3] + Universal threshold √2ln(length(x))
* sqtwolog - Universal threshold √2ln(length(x))
* rigsure - Principle of Stein's Unbiased Risk
* heursure - A heuristic variant of Stein's Unbiased Risk
* minimaxi - Minimax thresholding

You can also utilize your own custom thresholds, but then you will have to implement the multi-level decomposition yourself by the way of utilizing DWT *as also shown below* and thresholding whenever, as per your application. 

#### Threhsolding methodology: 

* Hard thresholding: Values below the threshold are hard-set to 0  
* Soft thresholding: Values beelow the threshold are subtracted or added from the coefficient given whether the threshold is lower/ higher, hence the transition is softer

An example for the methodology is shown on [stackExchange][4]. There are additional choices on threshold rescaling:

* one - Basic, no rescaling
* sln - Basic scaling with noise level from 1st level coefficients (use for DWT)
* mln - rescaling with level dependent estimation of noise (use if doing multi-level denoising)

Now, let's see how to perform wavelet denoising in MATLAB. Please ensure you have the [Wavelet Toolbox][5] package installed within your version of MATLAB. 


## MATLAB Implementation

```matlab
% Load the signal data into MATLAB and perform the Wavelet Decomposition
% Assuming the data is in an excel file and the time-series is saved in the second column
DAT = readtable('Data.xlsx');
signal = DAT(:,2);

% Choose primary decomposition metrics
level = 4; % define level
order = 4; % Which order wavelet from the wavelet family
wavelet = ["sym", "db", "haar"]; waveletChoice = 1;

%% Method 1: Using wavedec
% Perform wavelet decomposition x levels
[c, l] = wavedec(signal, level, wavelet);
% Extract the detailed coefficients for ALL levels at once!:
cApprox = appcoef(c,l,)
[cDetail1, cDetail2, cDetail3, cDetail4] = detcoef(c,l,[1,2,3,4]); 
% Perform thresholding and denoising
denoisedSignalM1 = wden('modwtsqtwolog', c, l, wavelet, level);
% Inverse Transformation reconstruct the denoised signal using the inverse wavelet transform
reconstructed_signal = waverec(denoisedSignalM1, l, wavelet);

%% Method 2: Using wden to denoised the signal directly
thresholdSelection = 'modwtsqtwolog';
thresholdType = 's'; % s: soft, h: hard
rescalingType = 'mln';
denoisedSignalM2 = wden(signal, thresholdSelection, thresholdType, rescalingType, level, [char(wavelet(waveletChoice) num2str(order))]);
```

The `denoisedSignalM2` is the directly denoised result, which I often-times utilize for a quick look at the signal to roughly estimate the noise that I'm dealing with. This one-line command in MATLAB quite is helpful to quickly determine noise and noise floor for a given signal.


## Python Implementation

Before you start, you need to install the PyWavelet library if you haven't already. You can do this using pip:

```terminal
pip install PyWavelets
```

Now moving to Python, we import the required libraries and setup the wavelet decomposition:

```python 
import pandas as pd
import pywt

# Load your signal or data into Python using your preferred method, such as NumPy or pandas, assuming it's an excel file with Time and Data columns
dataDF = pd.read_excel(r'Data.xlsx', sheet_name='allTimeSeriesData')

# Display all cokumns and make relevant assignments
print('These are the columns', dataDF.columns.ravel())
time = dataDF.['Time']
signal = dataDF.['Data']

# Wavelet Decomposition
# Select wavelet and also obtain approximate and detailed coefficients (this is LIMITED to 1 LEVEL ONLY):
useWavelet = pywt.Wavelet('sym4')
cApprox, cDetailed = pywt.dwt(signal, wavelet = useWavelet, mode = 'symmetric')

# Set a level for the direct wavelet decomposition:
useLevel = 4 # I usually select 4 levels to start with i.e. N/4 sample decomposition but that may change depending on the dataset of your choice
coeffs = pywt.wavedec(signal, useWavelet, level=useLevel)

# Thresholding and Denoising
denoised_coeffs = pywt.threshold(coeffs, threshold_value, mode='soft')

# Inverse Transformation
# Reconstruct the denoised signal from the modified coefficients:

# Reconstruct the denoised signal
denoised_signal = pywt.waverec(denoised_coeffs, wavelet)

```

## Conclusion

Wavelet denoising is a valuable tool for enhancing the quality of signals and images by removing unwanted noise while preserving important features. This guide demonstrates how to perform wavelet denoising in both MATLAB and Python using both directly available tools and . Depending on your preference and application, you can choose the platform that best suits your needs and utilize the appropriate functions and libraries for effective denoising.


[1]: https://en.wikipedia.org/wiki/Wavelet_transform
[2]: https://en.wikipedia.org/wiki/Discrete_wavelet_transform
[3]: https://www.cambridge.org/core/books/abs/wavelet-methods-for-time-series-analysis/maximal-overlap-discrete-wavelet-transform/2226423BCF1E061B98331E39D8149C45
[4]: https://dsp.stackexchange.com/questions/15464/wavelet-thresholding
[5]: https://www.mathworks.com/help/wavelet/index.html?s_tid=CRUX_lftnav