+++

date = "2019-01-04T00:00:00+00:00"
description = "Python code to simulate DSSS modulation and demodulation, with each step visualized."
title = "Direct Sequence Spread Spectrum Illustrated"

+++

**TL;DR**:
Python code to simulate [DSSS](https://en.wikipedia.org/wiki/Direct-sequence_spread_spectrum) modulation and demodulation, with each step plotted.
The [SciPy](https://www.scipy.org/) stack is used to handle signal processing and plotting.
DSSS is used in [CDMA](https://en.wikipedia.org/wiki/Code-division_multiple_access), [GNSS](https://en.wikipedia.org/wiki/Satellite_navigation) and [WiFi](https://en.wikipedia.org/wiki/Wi-Fi), among others.

<!--more-->

Part of my studies in the first half of 2017 involved GPS research, and while I never had to implement a GPS software-defined radio, I did interface with GPS hardware.
Knowing how GPS works in greater detail would have been very helpful, because I encountered a lot of GPS-related terminology and signal processing principles without understanding them.
The documentation I had access to assumed that the reader had more-than-general knowledge about GPS, which I was lacking.
Some things are easy to look up and understand -- Wikipedia and its references being a great resource -- but I struggled with some basic questions, such as
"How does a single bit of information get transferred from a GPS satellite to a GPS receiver, end to end?"
The answer is Direct Sequence Spread Spectrum.

## Disclaimer

> I hear and I forget.
> I see and I remember.
> I do and I understand.
> - [Chinese proverb or Confucius or Xunzi or a random person from the 1960s](https://web.archive.org/web/20181215210240/https://english.stackexchange.com/questions/226886/origin-of-i-hear-and-i-forget-i-see-and-i-remember-i-do-and-i-understand)

I do not have a strong background in signal processing, elecronics or telecommunication, so do take this post with a grain of salt when it comes to the terminology used and concepts explained.
This is a product of me trying to undersand DSSS, with a background primarily in computer engineering.
I tried to find plots showing what exactly happens with the elecromagnetic wave as it's being DSSS modulated and demodulated, but couldn't find plots that I was happy with and that could fill in all the gaps in the knowledge I had.
I didn't have the time to go through a course or a book, so I did what the quote says and wrote some Python to produce what I was looking for, without going too much into the theoretical background.

## Python setup

I used Python 3.5 installed through Ubuntu's repos.
As mentioned, SciPy stack was used for signal processing.

```python
import numpy as np
import scipy.signal as sp
import matplotlib.pyplot as plt
```

## The task

The task that the code will perform is transmitting the message "Test" at 500 Hz through a noisy channel using DSSS, and then reading the message on the receiving end.
The string will be encoded and sent as 7-bit ASCII with the least significant bit first.


```python
message_frequency = 500
message_code = np.array(tobits("Test"))
```

The `tobits` function does the message encoding into an array of ones and zeros.
It was taken from [John Gaines Jr.'s StackOverflow answer](https://web.archive.org/web/20190102120807/https://stackoverflow.com/questions/10237926/convert-string-to-list-of-bits-and-viceversa) and modified slightly.

```python
def tobits(s):
    result = []
    for c in s:
        bits = bin(ord(c))[2:]
        bits = '00000000'[len(bits):] + bits
        result.extend([int(b) for b in bits][1:])
    return result
```

## The transmitting end

My work was mostly concerned with the receiver side of the GPS stack and didn't need the details of how the signal was generated on a GPS satellite.
Therefore, I started with a sine carrier wave.
It is called a "carrier" wave because the transmitted information is piggybacking on it.
Piggybacking is achieved by modulating (also called multiplying) the carrier signal with the information carrying signal, which will be shown shortly.
The carrier wave usually has a frequency much higher that the signal that carries information and determines some features of the system, such as atmosphere penetration properties or antenna aperture size.
GPS uses a "base" frequency that many other frequencies in the system are derived from -- including the carrier wave frequency -- so this example uses the same at 10.23 MHz.

```python
amplitude = 1

phase = 0

base_frequency = 10.23e6
carrier_frequency_multiplier = 7  # GPS uses a higher multiplier, which makes the simulation much harder to run.
carrier_frequency = base_frequency * carrier_frequency_multiplier  # 71610000 Hz = 71610 kHz = 71.61 MHz

time_start = 0
time_end = 1 / message_frequency * message_code.size  # 0.056 s = 56 ms
time_duration = time_end - time_start  # 56 ms
samples_per_carrier_period = 5  # How many samples per carrier period will the simulation have, since we can't have a truly continuous signal.
dt = 1 / carrier_frequency / samples_per_carrier_period
time_steps = time_duration / dt
t = np.linspace(time_start, time_end, int(time_steps))

carrier_signal = amplitude * np.sin(2 * np.pi * carrier_frequency * t + phase)
```

A selected part of the carrier is shown in the following time plot.

![Carrier signal and its spectrum](https://luka.strizic.info/fig-dsss-illustrated/not-noisy-plots/1-Carrier-signal.png)

DSSS allows multiple parties to use the same medium simultaneously in time.
That is achieved by PRN coding (pseudo-random number coding), a medium access control method which assigns each party a known pseudo-random code to modulate the carrier signal with.
All the codes assigned seem like random binary sequences, hence "pseudo-random", but are actually carefully constructed sequences of bits such that that their correlation doesn't produce strong matches, unless correlating two same sequences.
That property is called being orthogonal to each other.
I have used real GPS PRN codes for my exercise, programmatically generated by Python code taken off [Jeremy Gold's Github repo](https://web.archive.org/web/20190102113509/https://github.com/jeremygold/gps-prn), where it is explained how they're generated.
The PRN codes are also called coarse/acquisition codes (C/A codes) in the GNSS context.

Jeremy Gold's code generates 1023-elment series of ones and zeros, but, to match the amplitude of the carrier and number of samples in my code, I've had to tile it, repeat it and turn each 0 into a -1.

```python
import PRN  # Jeremy Gold's PRN-generating library

satellite_vehicle_number = 10
coarse_acquisition_code_period = 1e-3

original_binarized_coarse_acquisition_code = np.array(PRN.PRN(satellite_vehicle_number))
coarse_acquisition_code = np.tile(original_binarized_coarse_acquisition_code, int(time_duration / coarse_acquisition_code_period))
coarse_acquisition_code = np.repeat(coarse_acquisition_code, t.size / coarse_acquisition_code.size)
coarse_acquisition_code_signal = binarize(coarse_acquisition_code, [-1, 1])
```

I've used the adjective "binarized" to represent a binary signal consisting of eiter 1 and 0 or 1 and -1, and the `binarize` function does that transformation.

```python
def binarize(xs, digits):
    return np.array([digits[0] if x <= 0 else digits[1] for x in xs])
```

Modulating an analog signal with a binary signal is essentially performing 180 degree phase shifts -- "flipping" the signal -- at times where the binary signal transitions from 0 to 1 and from 1 to 0.
Mathematically, a 0 could then be interpreted as a -1, so that the modulated signal is the result of multiplying the analog signal by the binary signal.
Therefore, modulating such signals is often called multiplying them.

The time scale in the following time plot is the same as in the previous plot.
Not all 1023 PRN bits are displayed.
The shown time period was selected to display a falling and a rising edge.

![The PRN (C/A) signal and its spectrum](https://luka.strizic.info/fig-dsss-illustrated/not-noisy-plots/2-Coarse-acquisition-code-signal.png)

If everything works correctly, the "Test" signal should be recoverable from and visible in the final signal at the receiver's end.
Again, the series of bits needs to be modified slighly, to get a signal with the proper number of samples that fit in with the rest.

```python
message_signal = binarize(np.repeat(message_code, t.size / message_code.size), [-1, 1])
```

Time scale in the following time plot is different from the last two and displays every bit.
It is visible that the message frequency is much lower than the PRN and the carrier frequency.

![The message signal and its spectrum](https://luka.strizic.info/fig-dsss-illustrated/not-noisy-plots/3-Message-signal.png)

## The DSSS magic

Now that all three elements of the outgoing signal are ready (the carrier wave, the PRN signal and the message signal), the first part of the DSSS magic happens -- multiplication of the signals.
The PRN signal and the message signal only contain values 1 or -1, so that during multiplication they either have no effect on the output, or cause a 180 degree phase shift, respecitevly.
An analoguous operation is a XOR (exclusive or) in digital logic with bits 0 and 1.
Just like with XORing, multiplying a signal two times with such a PRN signal or two times with such a message will give us the exact signal that we started with.
That fact is the basis on which the message is recovered on the reveiver's end, or to express it a bit more mathematically, assuming no noise:

```
OUTPUT = MESSAGE * PRN * CARRIER
MESSAGE = OUTPUT * PRN * CARRIER
```

From the previous formula it is clear that the PRN code and carrier wave properties must be known at both ends of the transceiving stack, which in reality is the case.

```python
output_signal = carrier_signal * message_signal * coarse_acquisition_code_signal
```

As the name of DSSS implies, some "spreading" occures -- the information is spread in spectrum and centered at the carrier frequency, visible in the following spectrum plot.
The following time plot has the same time scale as the first two plots, showing the two phase shifts from the PRN signal.
The spreading enhances noise resistance through the "despreading" gain, or the [process gain](https://en.wikipedia.org/wiki/Process_gain).

![The output signal and its spectrum](https://luka.strizic.info/fig-dsss-illustrated/not-noisy-plots/4-Output-signal.png)

## The receiving end

To simulate a receiving signal, noise is added to the output signal.
A [carrier-to-noise-density ratio](https://en.wikipedia.org/wiki/Carrier-to-noise-density_ratio) of 36.5 dB-Hz is used in these examples, so that the standard deviation of the noise signal is 40 times the amplitude of the output signal.

```python
noise_stddev = 40  # 40 is a CN0 of ~36.5; 27.2 is a CN0 of 40; 8.6 is a CN0 of 50
noise_offset = 0

input_signal = output_signal + np.random.normal(noise_offset, noise_stddev, output_signal.size)
```

On the receiving end, the incoming signal might look something like in the following plot.
It is easily seen that the useful signal is significantly less powerful than the noise.

![The input noisy signal and its spectrum](https://luka.strizic.info/fig-dsss-illustrated/noisy-plots/5-Input-signal.png)

It is hard to perform digital signal processing with signals on high frequencies that are usually used in GPS, due to high storage, power and processing requirements.
Filters also don't perform too well at those frequencies, therefore, another way to process them is to first bring them down to an [intermediate frequency](https://en.wikipedia.org/wiki/Intermediate_frequency), which is achieved by [mixing (or heterodyning)](https://en.wikipedia.org/wiki/Heterodyne) it with another signal.
Mixing signals of frequencies f1 and f2 creates two signals at the output, one at the frequency f1-f2 and the other at the frequency f1+f2.

The mixer signal that the input signal will be mixed with is derived from the reference frequency, which is usually sourced from a crystal oscillator in the receiver.
Mathematically, the mixing is performed by multiplying the signals.

```python
reference_frequency = 16.368e6

mixer_frequency_multiplier = int(carrier_frequency / reference_frequency)
mixer_frequency = reference_frequency * mixer_frequency_multiplier  # 65472000 Hz = 65472 kHz = 65.472 MHz

mixer_signal = amplitude * np.sin(2 * np.pi * mixer_frequency * t + phase)

mixed_signal = input_signal * mixer_signal
```

The high part and low part of the mixed signal are shown in the following time plot.
Additionally, the same mixed signal with no noise is shown afterwards.

![The mixed noisy signal and its spectrum](https://luka.strizic.info/fig-dsss-illustrated/noisy-plots/6-Mixed-input-signal.png)
![The mixed signal without noise and its spectrum](https://luka.strizic.info/fig-dsss-illustrated/not-noisy-plots/6-Mixed-input-signal.png)

After mixing, the signal contains the same information, but at a desired frequency.
The parts we don't care about are filtered out with a [band-pass filter](https://en.wikipedia.org/wiki/Band-pass_filter).

```python
intermediate_frequency = carrier_frequency - mixer_frequency
filter_bandwidth = 2.5e6
filter_lo_frequency = intermediate_frequency - filter_bandwidth / 2
filter_hi_frequency = intermediate_frequency + filter_bandwidth / 2
filter_order = 4

mixed_frequencies = np.fft.rfftfreq(mixed_signal.size, dt)
filter_nyquist_frequency = mixed_frequencies[-1]
z, p, k = sp.butter(filter_order, [filter_lo_frequency / filter_nyquist_frequency, filter_hi_frequency / filter_nyquist_frequency], btype='bandpass', output='zpk')
sos = sp.zpk2sos(z, p, k)

filtered_signal = sp.sosfiltfilt(sos, mixed_signal)
```

The filtered signal and its spectrum, both with and without noise, are shown in the following plots.

![The filtered noisy signal and its spectrum](https://luka.strizic.info/fig-dsss-illustrated/noisy-plots/7-Filtered-input-signal.png)
![The filtered signal without noise and its spectrum](https://luka.strizic.info/fig-dsss-illustrated/not-noisy-plots/7-Filtered-input-signal.png)

The signal will move from the analog domain to the digital domain by getting sampled at an appropriate frequency and quantized into a bit per sample.
A sampling frequency four times the highest filter frequency is selected, although something lower should suffice too, depending on how much noise is in the channel.

```python
sampling_frequency = filter_hi_frequency * 4

sampled_signal, sampled_t = sp.resample(filtered_signal, int(sampling_frequency * time_duration), t)
sampled_dt = time_duration / sampled_t.size
```

The sampled signal and its spectrum are shown in the following plots, with and without noise.

![The sampled noisy signal and its spectrum](https://luka.strizic.info/fig-dsss-illustrated/noisy-plots/8-Sampled-input-signal.png)
![The sampled signal without noise and its spectrum](https://luka.strizic.info/fig-dsss-illustrated/not-noisy-plots/8-Sampled-input-signal.png)

Before the signal is despread by multiplying the binarized sampled input signal with the targeted PRN, we'll generate a reference carrier signal on the recieving end, since we need to multiply by it too.
The reference carrier signal is a copy of what the carrier signal should look like within the binarized sampled input signal, so that we know exactly which part to take out before we're left with just the message signal plus the noise.
Therefore, to get the reference carrier, a regular carrier signal that we started with is taken through the same process of mixing, filtering and sampling.

```python
reference_carrier_signal = sp.resample(sp.sosfiltfilt(sos, carrier_signal * mixer_signal), int(sampling_frequency * time_duration))
```

The reference carrier signal and its spectrum are shown in the following plots.

![The refernece signal and its spectrum](https://luka.strizic.info/fig-dsss-illustrated/not-noisy-plots/9-Reference-carrier-signal.png)

Before multiplication/demodulation, all the signals are binarized so that basic math of -1 times -1 equals 1 applies.

```python
binarized_sampled_signal = binarize(sampled_signal, [-1, 1])

binarized_coarse_acquisition_code = np.tile(original_binarized_coarse_acquisition_code, int(time_duration / coarse_acquisition_code_period))
binarized_coarse_acquisition_code = np.repeat(binarized_coarse_acquisition_code, sampled_t.size / binarized_coarse_acquisition_code.size)
binarized_coarse_acquisition_code = sp.resample(binarized_coarse_acquisition_code, int(sampling_frequency * time_duration))
binarized_coarse_acquisition_code_signal = binarize(binarized_coarse_acquisition_code, [-1, 1])

binarized_reference_carrier_signal = binarize(reference_carrier_signal, [-1, 1])

binarized_no_carrier_signal = binarized_sampled_signal * binarized_reference_carrier_signal

demodulated_signal = binarized_no_carrier_signal * binarized_coarse_acquisition_code_signal
```

The demodulated signal is also noisy, as shown in the plots below, but by taking an average value across the period of one bit, the starting message can be successfully decoded.

```python
decoded_message_code = binarize(np.average(demodulated_signal.reshape((int(time_duration * message_frequency), -1)), 1), [0, 1])
```

![The demodulated noisy signal and the whole decoded message by averaging each bit's samples](https://luka.strizic.info/fig-dsss-illustrated/noisy-plots/10-Decoded-message.png)
![The demodulated signal without noise and the whole decoded message by averaging each bit's samples](https://luka.strizic.info/fig-dsss-illustrated/not-noisy-plots/10-Decoded-message.png)

The message itself is retrieved form the decoded signal with the `frombits` function, again inspired by [John Gaines Jr.'s StackOverflow answer](https://web.archive.org/web/20190102120807/https://stackoverflow.com/questions/10237926/convert-string-to-list-of-bits-and-viceversa).

```python
def frombits(bits):
    chars = []
    for b in range(int(len(bits) / 7)):
        byte = bits[b * 7:(b + 1) * 7]
        chars.append(chr(int(''.join([str(bit) for bit in byte]), 2)))
    return ''.join(chars)

decoded_message = frombits(decoded_message_code)
```

If an attempt was made to decode the message with a different PRN code, a wrong message would be read, instead of what was sent intially.

```python
satellite_vehicle_number_bad = 15

original_binarized_coarse_acquisition_code_bad = np.array(PRN.PRN(satellite_vehicle_number_bad))

binarized_coarse_acquisition_code_bad = np.tile(original_binarized_coarse_acquisition_code_bad, int(time_duration / coarse_acquisition_code_period))
binarized_coarse_acquisition_code_bad = np.repeat(binarized_coarse_acquisition_code_bad, sampled_t.size / binarized_coarse_acquisition_code_bad.size)
binarized_coarse_acquisition_code_bad = sp.resample(binarized_coarse_acquisition_code_bad, int(sampling_frequency * time_duration))
binarized_coarse_acquisition_code_signal_bad = binarize(binarized_coarse_acquisition_code_bad, [-1, 1])

demodulated_signal_bad = binarized_no_carrier_signal * binarized_coarse_acquisition_code_signal_bad

decoded_message_code_bad = binarize(np.average(demodulated_signal_bad.reshape((int(time_duration * message_frequency), -1)), 1), [0, 1])

decoded_message_bad = frombits(decoded_message_code_bad)
```

The following plots show the wrongly demodulated signal and the wrongly decoded message.

![The demodulated noisy signal and the whole decoded message by averaging each bit's samples](https://luka.strizic.info/fig-dsss-illustrated/noisy-plots/11-Decoded-bad-message.png)
![The demodulated signal without noise and the whole decoded message by averaging each bit's samples](https://luka.strizic.info/fig-dsss-illustrated/not-noisy-plots/11-Decoded-bad-message.png)

## Final notes

There it is, an illustration of DSSS!
Complete code, including plotting functions, can be found in [my GitHub repo](https://github.com/lstrz/dsss-illustrated).

Three booleans at the begging control whether the signals are plotted, saved and shown, respectively.
Plots are saved in .png and .svg formats.

```python
dofig = True
savefig = True
showfig = True
```

It takes my i7-8550U-based laptop 30 to 40 seconds to run the whole simulation and save the plots.
Running the `main.py` script without arguments should work fine, provided that required libraries and the correct Python version are installed.
