---
title: AUDIO PROCESSING IN 8 DAYS
filename: audio_processing.md
--- 

Overall goal : To be part of People + AI community, mainly as a developer

Goal : To learn audio processing required to contribute to Sunva codebase

## Day 0
Research on material. Downloaded pdf from Datacamp Spoken Language Processing in Python - https://projector-video-pdf-converter.datacamp.com/17718/chapter3.pdf

## Day 1 
#### Duration : 1 hour

#### Approach and Learnings
* Finished first 5 pages of pdf - Spoken Language Processing in Python by Datacamp
* Learnt about core audio concepts covered in first 5 pages like channel, sampling, quantization, sample rate, sample size, bit width
* **Channel** - passageway or location through which sound comes or goes. A single microphone can produce a single channel of audio, a speaker can receive a single channel of audio (refer 1)
* **Sample rate** - how many times audio is measured per second - refer 3
* **Sample width (aka bit depth or sample size)** - Related to quantization. Sample width determines how many digital steps are available to represent the analogue signal level (voltage) at each sample (slice of time). For example, using 4 bits, the analog signal can be quantized to any of the 2^4 = 16 levels
* Bit width - number of binary digits, called bits, necessary to represent an unsigned integer as a binary number
* An audio signal is just amplitude versus time. The frequency information is just inherently available when we measure amplitude against time. For example, a high frequency wave will have its amplitude go up and down much faster compared to a low frequency wave, so just by measuring amplitude against time, we are indirectly measuring the frequency
* Mono vs stereo - Mono uses one sound channel whereas stereo uses 2 sound channels - left and right
* Doubts : What is the y axis of an audio signal? - it is amplitude - refer 4

#### Resources
1. https://www.wildlifeacoustics.com/resources/faqs/what-is-an-audio-channel
2. https://audioaudit.io/articles/podcast/sample-width
3. https://superuser.com/questions/388382/what-does-the-sample-rate-and-sample-size-of-audio-means/388395#388395
4. https://medium.com/analytics-vidhya/audio-data-processing-feature-extraction-science-concepts-behind-them-be97fbd587d8
5. https://www.reddit.com/r/DSP/comments/ccenx1/im_a_complete_beginner_could_someone_layout_a/


## Day 2
#### Duration : 1 hour

#### Approach and Learnings

- Downloaded sample dataset from kaggle : https://www.kaggle.com/datasets/crischir/sample-wav-audio-files
- Ran code from first 7 pages

```
from pydub import AudioSegment
from pydub.playback import play
from pydub.effects import normalize
from speech_recognition import Recognizer
import numpy as np

wav_file = AudioSegment.from_file(file=r"data\BAK.wav")

print(wav_file.channels) # 2 which means stereo
print(wav_file.frame_rate) # 44100 which means 44100 samples per second
print(wav_file.sample_width) #2 which means 2 bytes = 16 bits = 65536 levels
print(wav_file.max) # 14108 (relation between this and 65536)
print(len(wav_file)) # 8334 


print(np.array(wav_file.get_array_of_samples()).shape) #(735076,)
# which implies (735076/2) / 44100 = 8.33 secs is audio duration
# divide by 2 because it is stereo, so 2 samples for each sampling instance


quiet_wav_file = wav_file - 60
loud_wav_file = wav_file + 10

play(loud_wav_file)

# recognizer = Recognizer()
# recognizer.recognize_google(quiet_wav_file)
# normalized_wav_file = normalize(loud_wav_file)
# loud_wav_file_channels = loud_wav_file.split_to_mono()


```

- recognize_google not working
* Unable to install librosa using poetry. After some research found out that it was due to numpy version in pyproject.toml being incompatible with the librosa version. So removed numpy from dependencies in pyproject.toml, and then installed librosa and numpy using poetry add
* Doubts :  When to normalize an audio track?

#### Resources
1. https://realpython.com/python-speech-recognition/ (for recognizer.recognize_google)
2. https://www.youtube.com/watch?v=RR7fCquUDAM (audio normalization effect)
3. https://www.reddit.com/r/audioengineering/comments/o9my78/when_should_you_normalize_an_audio_track/
4. https://github.com/numba/numba/issues/8304
5. https://stackoverflow.com/questions/35735497/how-to-create-a-pydub-audiosegment-using-an-numpy-array

## Day 3
#### Duration : 2 hour

#### Approach and Learnings

* Array length = sample rate * duration, for eg. 44100 * 1 sec = 44000. Explanation : An audio signal is digitized by measuring its amplitude at certain instants. A sample is the value of the magnitude at that instant.If sample rate is 48kHz, it means 48000 times per second
* A sample is a snapshot of the amplitude of some audio signal at a particular moment.
* A frame is the group of all samples that happened at that moment (e.g. in stereo, there are 2 samples in the frame)
* Learnt Numpy functions like hstack, vstack, reshape, iinfo, finfo

```
import numpy as np

a = np.array([1,2,3])
b = np.array([4,5,6])

np.hstack((a,b)) # stack the sequence of input arrays horizontally
# [1,2,3,4,5,6]

np.vstack((a,b)) 
# [[1 2 3], [4 5 6]]

a.reshape(-1,1)
# [[1],[2],[3]] # only 1 column, as many rows based on the dimension of the array

a.reshape(1,-1)
# [[1,2,3]]

a.reshape(-1)
# [1,2,3]

# shows machine limits for integer types.
np.iinfo(np.int16).max

```

* Downloaded AUdacity from Microsoft store
* Question : When to convert stereo (2 channels) to mono (1 channel)? Why convert audio file to numpy array?(to use scipy filters on audio) How is length of audio file numpy array calculated? DIff b/w frame rate and sample rate? How is stereo sound stored in an array? WHat is pulse code modulation? What does length of wav file signify?

#### Resources
 1. https://www.sonible.com/blog/stereo-to-mono/
 2. https://github.com/RCJacH/monolizer
 3. https://stackoverflow.com/questions/38015319/how-to-create-a-numpy-array-from-a-pydub-audiosegment 
 4. https://stackoverflow.com/questions/5120555/how-can-i-convert-a-wav-from-stereo-to-mono-in-python
 5. https://github.com/jiaaro/pydub/blob/master/API.markdown#audiosegmentget_array_of_samples
 6. https://stackoverflow.com/questions/10357992/how-to-generate-audio-from-a-numpy-array
 7. https://www.reddit.com/r/audioengineering/comments/1ds5c4q/what_exactly_do_you_mean_by_a_sample_sample_rate/ 
 8. https://stackoverflow.com/questions/25941986/how-can-an-audio-wave-be-represented-in-a-long-array-of-floats 
 9. https://github.com/henryxrl/Listening-to-Sound-of-Silence-for-Speech-Denoising 
 10. https://www2.cs.uic.edu/~i101/SoundFiles/ 
 11. https://stackoverflow.com/questions/18691084/what-does-1-mean-in-numpy-reshape

## Day 4
#### Duration : 1 hour


#### Approach and Learnings
* **Short Time Fourier Transform** - normal fourier transform does not contain time information i.e. it shows frequency in the entire signal across all time and does not tell when a particular frequncy occurs.  This is a challenge. To solve this, we have short time fourier transform i.e. stft, where signal is broken down into blocks of smaller duration (say 5 seconds), and FFT (fast fourier transform) is done on each block. This is then laid on a spectogram, with time on x axis, frequency on y axis and color indicating the magnitude of frequency (refer 1)

* In STFT time and frequency resolution are inversely proportional. If we increase the window size/frame size, the frequency resolution increases because for the same frequency range (0 to fs/2), we get more frequency bins i.e. smaller frequency bins, so we can pin point more accurately which frequency the sound belongs to (e.g 0 to 5hz bin and 5 to 10hz bin instead of just a 0 to 10hz bin). But we lose out on time resolution as we have taken a larger time window, so it is harder to pin point when exactly that frequency occured. Similarly, when we reduce the window size/frame size, the time resolution increases, but the frequency resolution decreases. 

#### Resources
1. https://www.youtube.com/watch?v=-Yxj3yfvY-4

## Day 5 and 6
#### Duration : 4.5 hours


#### Approach and Learnings

* Sampling frequncy (fs) : Number of sample of taken per second from a continuous signal
* FFT length/FFT size (N) : The number of points in FFT output array
* Width of each frequency bin is fs/N. The first bin in the FFT is DC (0 Hz), the second bin is fs / N. The next bin is 2 * Fs / N. To express this in general terms, the nth bin is n * Fs / N (refer 13 for clear explanation)
* Since the second half of the FFT contains no additional useful information, most FFT (like Librosa fft) return only N/2 + 1 data points (rewrite this using 21)
* Number of samples vs FFT length - The length of the FFT merely interpolates the spectral frequency curve represented by the number of samples. But why are hey equal?
* Given a fixed length recording, you can easily change your DFT bin frequency spacing by performing arbitrary length FFTs on the given data. But this will not change your spectral resolution, which is limited by the initial data length. Roughly speaking, the longer your total observation interval is, the better (finer) your spectral resolution will be (If you want any kind of spectral resolution at a given frequency then 1/T should be at least 10 times less than that frequency. e.g. if you care about 0.5 Hz you need to take at least 40 seconds of data)
* Below are code and learnings for basic fft using numpy

```
# sampling rate
fs = 2000
# sampling interval
ts = 1.0/fs
# create array starting 0, ending just before 1, interval bw 2 values is ts
t = np.arange(0, 1, ts) #ends at 1 - 1/2000 = 0.9995

# input of sin function is in radians not degrees, 1 radian = 57.29 degrees, 2*pi radian = 360 degree
# freq and t together determine the value, since freq is fixed, different values of t give different values
# When t = 1/freq = 1, it finishes 1 cycle, all values of t in between are different points in that cycle
freq = 1
x = 3*np.sin(2*np.pi*freq*t)

# when t= 1/freq = 0.25, it finishes 1 cycle, all values of t in between (0.0005 to 0.2495) are different points in that cycle
freq = 4
x += np.sin(2*np.pi*freq*t)

freq = 7   
x += 0.5* np.sin(2*np.pi*freq*t)

plt.figure(figsize = (8, 6))
plt.plot(t, x, 'r')
plt.ylabel('Amplitude')

plt.show()


from numpy.fft import fft, ifft, rfft
# Number of points in frequency domain is 2000, which is equal to number of points in time domain
X = fft(x)
# number of points in frequency domain, which is also 
N = len(X)
n = np.arange(N)
# fs/N = 2000/2000 = 1, is the frequency bin size, first bin is 0 Hz, second bin is from 0Hz to 1Hz
freq = n * (fs / N) 
# fft gives full whereas "rfft" (real FFT), you will get back the first half and the Nyquist bin for even N.
# Hence length of rfft is only 1001, the first 1000 points plus 1
Xr = rfft(x)
nr = np.arange(len(Xr))
# We have to add 2 in denominator as rfft only goes from 0 to fs/2 (fs/2 to fs is same as -fs/2 to 
# 0, which for real valued signal is same as 0 to fs/2)
freqr = nr * (fs / (2 *len(Xr))) 


plt.figure(figsize = (12, 6))
plt.subplot(121)
plt.plot(freq, np.abs(X))
plt.xlabel('Freq (Hz)')
plt.ylabel('FFT Amplitude |X(freq)|')


plt.subplot(122)
plt.figure(figsize = (12, 6))
plt.plot(freqr, np.abs(Xr))
plt.xlabel('Freq (Hz)')
plt.ylabel('FFT Amplitude |X(freq)|')
plt.tight_layout()

plt.show()

```

* Below are code and learnings for basic sfft using librosa

```
import numpy as np
import librosa

N = 48000
x = 10*np.ones(N) #  np.ones((N,1)) will not work

S = np.abs(librosa.stft(x))
print(S.shape) # (1025, 94)
# S is basically time frequency domain 
# n_fft = length of windowed signal after zero padding = fft length = 2048 (default value)
# hop_length = number of samples bw two successive windows in time domain = 512 (default value is win_length // 4)
# Number of rows in STFT matrix = number of frequncy bins = (n_fft/2) + 1 = 2048/2 + 1 = 1025
# Number of columns in STFT matrix = number of frames = N / hop_length = 48000 / 512 = 93.75

fig, ax = plt.subplots()
img = librosa.display.specshow(librosa.amplitude_to_db(S,
                                                       ref=np.max),
                               y_axis='log', x_axis='time', ax=ax)
ax.set_title('Power spectrogram')
fig.colorbar(img, ax=ax, format="%+2.0f dB")

## refer 2 for similar examples
```

* Doubts
1. Does frequency resolution depend on duration of signal?
2. What if number of samples in time domain is not equal to FFT length?
3. What is the difference between frequency resolution and frequncy bin width?
4. What exactly is hop length in a STFT?
5. If an audio is 10 seconds long, and FFT size is 1024, and we want to perform FFT on the entire signal, how do we do it, since number of samples in time domain is much greater than fft size? (answer is using overlap add or overlap save, refer 16 and 17)

#### Resources
1. https://github.com/TUIlmenauAMS/MRSP_Tutorials/tree/master/seminars (YT video - https://www.youtube.com/watch?v=7hEyXME4oMc)
2. https://nbviewer.org/github/TUIlmenauAMS/MRSP_Tutorials/blob/master/seminars/mrsp_support_02.ipynb
3. https://stackoverflow.com/questions/31399903/pydub-accessing-the-sampling-ratehz-and-the-audio-signal-from-an-mp3-file
4. https://stackoverflow.com/questions/63350459/getting-the-frequencies-associated-with-stft-in-librosa
5. https://www.youtube.com/watch?v=O0Y8FChBaFU - How to Compute FFT and Plot Frequency Spectrum in Python using Numpy and Matplotlib
6. https://www.nickwritesablog.com/introduction-to-oversampling-for-alias-reduction/ (skip for now)
7. https://www.reddit.com/r/DSP/comments/8oxxp9/fft_length_vs_sample_length/
8. https://dsp.stackexchange.com/questions/75579/why-does-fft-size-equals-the-numbers-of-samples-in-the-time-domain
9. https://dsp.stackexchange.com/questions/48216/understanding-fft-fft-size-and-bins
10. https://www.reddit.com/r/DSP/comments/wwhou8/how_to_determine_fft_sample_frequency_and_the/
11. https://dsp.stackexchange.com/questions/55226/what-is-frequency-resolution
12. https://dsp.stackexchange.com/questions/59922/is-frequency-resolution-bin-size-inverse-always-directly-proportional-to-sam
13. https://stackoverflow.com/questions/4364823/how-do-i-obtain-the-frequencies-of-each-value-in-an-fft
14. https://dsp.stackexchange.com/questions/79292/hop-size-in-stft
15. https://docs.scipy.org/doc/scipy/reference/generated/scipy.signal.stft.html
16. https://www.reddit.com/r/DSP/comments/142cjii/how_to_process_longer_signals_than_fft_size_or/
17. https://dsp.stackexchange.com/questions/81310/intuitive-understanding-of-how-overlap-save-works-in-contrast-to-overlap-add
18.https://dsp.stackexchange.com/questions/48085/why-is-the-size-of-the-output-of-the-fft-of-a-signal-is-same-as-the-size-of-the
19.https://pythonnumericalmethods.studentorg.berkeley.edu/notebooks/chapter24.04-FFT-in-Python.html
20. https://numpy.org/doc/stable/reference/generated/numpy.fft.rfft.html
21. https://www.youtube.com/watch?v=-Yxj3yfvY-4


## Day 7
#### Duration : 1.5 hour

#### Approach and Learnings

* To get an array of samples from audio data in pydub, we can use get_array_of_samples function, and convert that to numpy array using np.array

```
from pydub import AudioSegment
import numpy as np
import librosa

file = r'speech_with_silence.wav'
audio_segment = AudioSegment.from_file(file, format="wav")
audio_data = np.array(audio_segment.get_array_of_samples())
audio_data = audio_data.reshape((-1, audio_segment.channels))
audio_data = audio_data.mean(axis=1, dtype=np.int16)
audio_data = audio_data / np.iinfo(audio_data.dtype).max
S = np.abs(librosa.stft(audio_data))

```

* Pyaudio : Python library to play and record audio. Basically provides python binding for PortAudio a cross platform audio i/o  library

* Code to convert speech to text using Groq ASR/STT api 


```
from groq import Groq

client = Groq(api_key=groq_api_key)

def transcribe_audio(audio_file_path):
    try:
        # b stands for binary file, rb means read binary
        # file parameter of Groq client needs filename and file content as input
        # since its a binary file, output of file.read() will be a series of binary characters like
        # \x95\xff\x96\00 
        with open(audio_file_path, "rb") as file:
            transcription = client.audio.transcriptions.create(
                file=(audio_file_path, file.read()),
                model="whisper-large-v3",
                response_format="text",
                language="en",
                prompt="",
            )
            return transcription 
        
    except Exception as e:
        print(str(e))
        return None

# wav file with my voice recorded using Audacity
transcription = transcribe_audio('data/speech_with_silence.wav')
print(transcription)

```

* Doubt 
1. How to read the following binary sequence - \xeb\xffG\x00F\x00z\x00 ?

#### Resources
1. https://betterexplained.com/articles/an-interactive-guide-to-the-fourier-transform/
2. https://sites.northwestern.edu/elannesscohn/2019/07/30/developing-an-intuition-for-fourier-transforms/
3. https://stackoverflow.com/questions/38015319/how-to-create-a-numpy-array-from-a-pydub-audiosegment
4. https://console.groq.com/docs/speech-text
5. https://stackoverflow.com/questions/50644407/understanding-characters-in-a-binary-file
6. https://stackoverflow.com/questions/15746954/what-is-the-difference-between-rb-and-rb-modes-in-file-objects
7. https://github.com/KennyVaneetvelde/groq_whisperer/blob/main/main.py


## Day 8
#### Duration : 1.5 hour

#### Approach and Learnings

* In numpy, when we do mean across axis 0, it implies result will be projected onto/compressed along x axis i.e. final dimension will be number of columns. 

```
import numpy as np
my_array = np.array([[1,2,3],[4,5,6]])
print(np.mean(my_array, axis=0)) #array([2.5, 3.5, 4.5]) 
print(np.mean(my_array, axis=1)) #array([2., 5.])

```

* np.abs(S[f,t]) gives magnitude of frequncy bin f at time t. So averaging that across all bins for a given t gives magnitude of the frame at time t. This can be implemented using np.mean as shown below 


```
from pydub import AudioSegment
import numpy as np
import librosa

file = r'speech_with_silence.wav'
audio_segment = AudioSegment.from_file(file, format="wav")
audio_data = np.array(audio_segment.get_array_of_samples())
audio_data = audio_data.reshape((-1, audio_segment.channels))
audio_data = audio_data.mean(axis=1, dtype=np.int16)
audio_data = audio_data / np.iinfo(audio_data.dtype).max #normalize audio
S = np.abs(librosa.stft(audio_data))

frame_energies = np.mean(S, axis=0)
low_energy_frames = np.sum(np.mean(S, axis=0) < 0.02) # get number of frames with magnitude less than 0.02

# get percentage of frames with low energy out of all the frames 
# if this percentage is very high say greater than 75%, then we can say that it is a silent sound
proportion_low_energy = low_energy_frames / len(frame_energies) 


## np.sum(np.mean(S[:, 50:100], axis=0) > 0.02) # check energies of smaller chunks of frames 


```

* Silence detection: if majority of frames in say a 1 sec audio segment have low energy, then we can consider that audio segment as silent (useful when we get audio over websockets)

#### Resources


For full documentation visit [mkdocs.org](https://www.mkdocs.org).

To run docs, within virtual environment use : mkdocs serve 



