# Module 3
### Mix n Match!

**<ins>Lecture Notes:</ins>**

[Lecture 3 Notes](https://drive.google.com/file/d/1KA0IPuKRptXwOLX6v_vErKq9SacYe6o9/view?usp=sharing)

**<ins>Motivation:</ins>**

We’ve thus far covered ways of amplifying signals and generating frequencies - but is there a way to generate frequencies given other frequencies? From last time, we saw that using feedback, we can create systems whose output oscillates at a designed frequency. One common use of generating known, stable frequencies is in use of the Local Oscillator or LO of a mixer, whose operation is covered in the Lecture 3 notes. 

This is also why we wanted to ensure that our signal generated by our Colpitts Oscillator had a high enough power level - our LO needs to be large enough in power that it can sustain a voltage drop of 0.6 to 0.7 V - enough to maintain forward operation of the diodes in the diode ring mixer. 

 

**<ins>Prerequisites:</ins>**

Lecture 1, 2 and 3 alongside assignment 1 and 2. 

Class A amplifiers/Common-emitter and common-base

LTspice

**<ins>Skills Learned:</ins>**

Mixer Operation

LO power and general Mixer Parameters of Interest

Filters

**<ins>Parts List:</ins>**

LTspice

**<ins>Setup:</ins>**

Thus far, we’ve designed a few important system components, including amplifiers and oscillators. But we have yet to combine these components together in a RF system. Although we still don’t have all the building blocks yet for a full wireless transmitter or receiver, we do have the tools to design an incredibly important function used in almost every RF system: frequency conversion.

In this assignment, your mission, should you choose to accept it, is to design a few diode ring mixers that can perform frequency upconversion from 1MHz to 27MHz. We will be using these devices to convert the ~1MHz output from the microcontroller DAC to a frequency we can transmit at. As discussed before, we will be doing this in two steps - from 1MHz to 5Mhz and from 5MHz to 27MHz.


## Part 1: Diode Ring Mixer

First, we’ll create a diode ring mixer and use it as an upconverter. Based on the schematic shown in the lecture, create a mixer simulation in LTSPICE. Note that the schematic assumes the input to the mixer is the RF signal. That is true for a downconverter. Here, we will be making an upconverter where the IF port is the input. The circuit is exactly the same, only what was previously called the RF port is now the IF port and vice versa (i.e., you will input your IF signal at what we were calling the RF port and measure the RF output at what had been labeled the IF port).

This is important, so please be sure you understand it - the output of our mixer is at the ‘bottom’ of the circuit diagram (at the port labeled IF) and the inputs are for the LO frequency and the input signal (at the port labeled RF). These are not interchangeable, and depending on whether you want to do upconversion or downconversion using the LO frequency, you will need to treat different ports as the input and output ports.



* For the diodes, we’ll use the MMBD101 Schottky diode ([datasheet](https://www.onsemi.com/pub/Collateral/MBD101-D.PDF)). This diode is designed for UHF circuits (ultra high frequency, 300MHz-3GHz) and easily does the trick for our application.
    * In theory, any type of diode could be used for this application. However, there are multiple reasons why Schottky diodes are used. First, Schottky diodes have a lower forward voltage than standard PN junction diodes, meaning that the mixer requires less LO power to function. Second, Schottky diodes tend to switch on and off faster than PN junction diodes and have lower junction capacitance (higher capacitance effectively increases the conversion loss of the mixer). 
    * Use the following SPICE model (the same way you used the MMBTH10 npn transistor model in the previous assignments) 

        .model MMBD101 d
        Is=1e-7 Rs=2.51244 N=1.71158 Tt=0
        Cjo=9.20838e-13 Vj=1.5 M=0.2
        Fc=0.5 Bv=1000 Ibv=0.0001
        Kf=0 Af=1 Xti=4 Eg=0.63401


             

* For the LO, use a 4MHz voltage source with a 50Ω source resistance. Give the LO a 0.5V amplitude (we’ll vary this later). 
* For the IF input, use a 1MHz voltage source with a 50Ω source resistance and 1mV amplitude (we’ll vary this later too).
* Use a 50Ω resistor as the load at the RF port. The signal at the RF port should consist of the sum and difference frequencies: 5MHz and 3MHz. We will filter out the 3MHz later.
* To determine the inductance of your transformer coils, try to make their impedance about 10x your load resistance - this is a good rule of thumb for designing components when you want reactive elements to dominate the response (eg - decoupling capacitors should have at least ~10x smaller impedance than what they are bypassing). Also make sure the turns ratio for each transformer is 1:1. 
    * Remember to stick to reasonable inductor values. For transformers, reasonable inductances range from nanohenries to ~15uH.

To make sure things are working properly, look at the following:



* If you plot the FFT of the RF resistor voltage, you should see the sum and difference frequencies, plus a bunch of harmonics.
* The voltage across the RF voltage source should be roughly 1mV amplitude plus some fuzz (the fuzz is due to the IF and LO signals leaking into the RF port). The IF voltage source is 1mV amplitude with a 50Ω source resistance, and the voltage across the voltage source is 1mV amplitude -- Can you guess what the resistance is looking into the IF & RF ports? (Hint: this is a transformer…)

**<ins>Checkpoint 1: The Mixer</ins>**



* Verify that the RF output contains the sum and difference frequencies by using the FFT method introduced in Assignment 2. Save a screenshot of the spectrum.
* Verify that there is essentially zero leakage from the LO port to the RF port, and the IF port to the RF port. You will need to perform the FFT of the voltage at the port in question and check the magnitude of the test port’s frequency - so you might put a cursor at 1MHz, on the FFT of your RF output to see the IF to RF leakage. Similarly, for the LO leakage, you would put the cursor at 4Mhz on the FFT of the RF output FFT.
* Determine the conversion loss of the mixer. Conversion loss is defined here as the ratio of output power at the RF port at 5MHz to the input power at the IF port at 1MHz. This calculation should be performed in decibels (Conversion Loss (dB) = 20log10(V_RF) - 20log10(V_IF) )
    * You can determine RF and IF voltage in dB easily by looking at the FFT of each signal. By default, the FFT will be plotted in dB scale. The difference between the two in dB is the conversion loss. Note: make sure to only look at the frequency of interest (5MHz) at the RF port.
    * If this was a downconversion mixer, the conversion loss would be defined as the ratio between the IF port power and the RF port power (i.e., your input would be at an intermediate frequency and your output would be at the radio frequency).
    * Fun fact: the smallest possible loss for a diode ring mixer is 3.9dB. That means your answer should be greater than 3.9dB.
* Try changing one of the LO transformer secondary winding inductances by 1%. What happens to the LO isolation?
* Try changing one of the RF transformer secondary winding inductances by 1%. What happens to the IF isolation? Now try changing the inductance by 50%. What happens to the IF isolation?
* Try varying the LO amplitude to 0.2V and 1V. What happens to the conversion loss?
    * It turns out that an LO amplitude of roughly 0.7V (or 7dBm, which means a voltage source of 1.4V) is close to optimal for diode ring mixers. Many commercial ring mixers will specify a LO drive of 7dBm.

Now that you designed one mixer to upconvert from 1MHz to 5MHz, you can design the other three as well - one to upconvert from 5MHz to 27MHz and two to downconvert, from 27MHz to 5MHz and from 5MHz to 1MHz. Do not try to connect the output of one mixer to the input of another - with so many unfiltered harmonics you will get near gibberish at the output. To design the other mixers, you will need to create new voltage sources at the appropriate frequencies (you can use the same amplitudes as above). The schematics will likely be the same, with some changes to the values of inductors used in the transformers. You should repeat the measurements for leakage and conversion loss.


## Part 2: Driving LO Port with an Oscillator

Next, we’ll see how we can use an oscillator to drive the LO port of the mixer. Pull out the oscillators you designed from module 2!

We will need to make a small modification to your oscillator. In module 2, the required output amplitude requirement was 1V. For this application, we’ll need to drop that a little to make our lives easier. Increase the emitter resistor of your oscillator until the output amplitude is roughly 0.5V (getting within 10% is fine).

Next, you will need to design a buffer between the oscillator and the mixer. Recall the discussion during the lecture of the weird impedance that the mixer will load your oscillator with if you connect them directly. Recall that the common collector amplifier acts as a good buffer, so let’s design one. Here’s some recommended specifications for this amplifier, though you may choose to deviate from this design:

Set DC emitter voltage to 2.5V

Go for a gain as close to 1 as possible

Set the collector current to roughly 2mA

Because the LO port of the mixer is highly nonlinear (input impedance goes from a high impedance to a low impedance during a full period due to diode switching), we need to ensure the buffer can supply the current needed by the mixer. This 2mA figure is chosen based on the suggested oscillator amplitude. Should we have had a higher amplitude, the buffer would need to supply even more current to the mixer.

When choosing your bias resistors, remember that the input impedance for this buffer will be loading your oscillator in parallel with the emitter resistor. Design Z_in for this buffer appropriately, so that your oscillator still functions as it needs to.

**<ins>Checkpoint 2: The Mixer + Oscillator</ins>**

With the buffer designed, hook up your oscillator, buffer and mixer together. The IF should still be driven by a perfect voltage source with a 50Ω internal resistance. With this configuration, find the conversion loss, LO isolation, and IF isolation. 

Again, make sure you can repeat this for all four mixers - with the two oscillators used for upconversion or downconversion. You should be able to use the same buffer for all schematics. When down converting, drive the RF port with a perfect voltage source with a 50Ω internal resistance, as you did for checkpoint 1.

By now you should have four functional ring mixers, being driven by Colpitts oscillators at the LO port, that make the appropriate frequency conversions (i.e, we will look at input and output FFTs).


## Part 3: Filters

Filters can be tedious to design by hand, especially to certain specifications. Thus, we are going to use an online tool to make them. In this project, we will apply filters mainly at 1 MHz, 5MHz and 27MHz, right after the Mixer Modules, as mixing introduces harmonics and other frequencies. For this part, use the tool below to create bandpass filters at these two frequencies, and put them into LTSpice. Feel free to play around with the Filter Type, Order, and Topology. Note that the higher order, the better the results, but also the more parts you will need to solder. Also, the tighter the passband, the better. However, the higher the frequency, the larger the passband will typically be. 

[RF Tools | LC Filter Design Tool (rf-tools.com)](https://rf-tools.com/lc-filter/)

**Specifications: **

**<ins>Filter 1:</ins>**

Passband must include 1 MHz

Lower Cutoff: at least 0.9 MHz

Upper Cutoff: at most 1.1 MHz

Input Impedance: 50Ω

Output Impedance: 50Ω

Component Values:Standard

Tolerance: 5%

**<ins>Filter 2:</ins>**

Passband must include 5MHz

Lower Cutoff: at least 4.7MHz

Upper Cutoff: at most 5.3 MHz

Input Impedance: 50Ω

Output Impedance: 50Ω

Component Values:Standard

Tolerance: 5%

**<ins>Filter 3:</ins>**

Passband must include 27MHz

Lower Cutoff: at least 26MHz

Upper Cutoff: at most 28MHz

Input Impedance: 50Ω

Output Impedance: 50Ω

Component Values:Standard

Tolerance: 5%

**<ins>Checkpoint 3: Implementing the filters</ins>**

With the filters designed, implement the filters into your mixer/oscillator module as the load. Include a 50Ω load at the end of the filter.

Apply this for the 1 - 5 MHz and 22 - 27 MHz up conversion (2 total). Verify using the FFT that:

The 1 - 5 MHz up conversion has a peak at 5 MHz

The 5 - 27 MHz up conversion has a peak at 27 MHz

The 27 - 5 MHz down conversion has a peak at 5 MHz

The 5 - 1 MHz down conversion has a peak at 1 MHz