# creamq-equalizer-plugin
simple equalizer audio plugin

# Parameters Code Breakdown
## Plugin Processor h

These lines of code are responsible for setting up the parameters and connecting them to the `AudioProcessorValueTreeState`.  
### Step 1: Defining Parameter Layout  
```cpp static juce::AudioProcessorValueTreeState::ParameterLayout createParameterLayout();```

This line declares a static function `createParameterLayout()`, which will be responsible for defining the layout of the plugin's parameters. The `juce::AudioProcessorValueTreeState::ParameterLayout` is a type representing the layout of parameters in the plugin.

### Step 2: Creating the AudioProcessorValueTreeState

```cpp
`juce::AudioProcessorValueTreeState apvts {*this, nullptr, "Parameters", createParameterLayout()};
```

In this line, I create an instance of `juce::AudioProcessorValueTreeState`, which is a class provided by JUCE to handle the management and communication of plugin parameters. Let's break down the constructor arguments:

1. `*this`: The first argument is a pointer to the audio processor class (The plugin) itself. This is passed so that the `AudioProcessorValueTreeState` can be connected to the plugin.
    
2. `nullptr`: The second argument is the undo manager. Since we're passing `nullptr`, the `AudioProcessorValueTreeState` won't support undo/redo functionality for parameter changes.
    
3. `"Parameters"`: The third argument is a string identifier representing the XML node name where parameter values will be stored in the plugin's state. This is useful for state persistence when saving and loading plugin projects.
    
4. `createParameterLayout()`: The fourth argument is the parameter layout created by calling the `createParameterLayout()` function. This layout defines the structure of the parameters, such as their names, types, range, and default values.


## Plugin Processor cpp

Here's a breakdown of what each part of the Parameter Layout does:

```cpp

juce::AudioProcessorValueTreeState::ParameterLayout    EqualizerAudioProcessor::createParameterLayout() { 
	// Create aninstance of ParameterLayout to store the parameters
	juce::AudioProcessorValueTreeState::ParameterLayout layout;
}
```

The function starts by creating an instance of `ParameterLayout` called `layout`, which will hold all the defined parameters for the equalizer.

```cpp
// Add "Low Cut Frequency" parameter

layout.add(std::make_unique<juce::AudioParameterFloat>("Low Cut Frequency",                                                             "Low Cut Frequency",                                                             juce::NormalisableRange<float>(20.f, 20000.f, 1.f, 1.f),20.f));
```

Next, I add the "Low Cut Frequency" parameter using the `layout.add()` function. This parameter is of type `juce::AudioParameterFloat`, representing a floating-point parameter. It has a range from 20 Hz to 20,000 Hz, with a step size of 1 Hz and a default value of 20 Hz.

```cpp
// Add "High Cut Frequency" parameter
layout.add(std::make_unique<juce::AudioParameterFloat>("High Cut Frequency",                                                             "High Cut Frequency",                                                             juce::NormalisableRange<float(20.f,20000.f,1.f,1.f),20000.f));
```

Similarly, it adds the "High Cut Frequency" parameter with the same configuration but a default value of 20,000 Hz.

```cpp
// Add "Peak Frequency" parameter    
layout.add(std::make_unique<juce::AudioParameterFloat>("Peak Frequency",                                                             "Peak Frequency",                                                             juce::NormalisableRange<float>(20.f, 20000.f, 1.f, 1.f), 750.f));
```


Next, I add the "Peak Frequency" parameter with a default value of 750 Hz.

```cpp
// Add "Peak Gain" parameter     
layout.add(std::make_unique<juce::AudioParameterFloat>("Peak Gain",                                                             "Peak Gain",                                                             juce::NormalisableRange<float>(-24.f, 24.f, 0.5f, 1.f), 0.0f));
```

The "Peak Gain" parameter is added, representing the gain of the peak filter. It has a range from -24 dB to 24 dB, with a step size of 0.5 dB, and a default value of 0 dB.

```cpp
// Add "Peak Quality" parameter     
layout.add(std::make_unique<juce::AudioParameterFloat>("Peak Quality",                                                             "Peak Quality",                                                          juce::NormalisableRange<float>(0.1f, 10.f, 0.05f, 1.f), 1.f));
```

The "Peak Quality" parameter is added, representing the quality (Q) factor of the peak filter. It has a range from 0.1 to 10, with a step size of 0.05, and a default value of 1.

```cpp

// Create a StringArray for the choices in "Low Cut Slope" and "High Cut Slope" parameters     
juce::StringArray stringArray;
for (int i = 0; i < 4; i++)
{
	juce::String str;
	str << (12 + i * 12);
	str << " db/Oct";
	stringArray.add(str);
}
```

A `StringArray` called `stringArray` is created to hold the choices for the "Low Cut Slope" and "High Cut Slope" parameters.

```cpp
// Add "Low Cut Slope" parameter   
layout.add(std::make_unique<juce::AudioParameterChoice>("Low Cut Slope", "Low Cut Slope", stringArray, 0));
```

The "Low Cut Slope" parameter is added as a `juce::AudioParameterChoice`, representing a choice-based parameter. It will have options like "12 db/Oct," "24 db/Oct," "36 db/Oct," and "48 db/Oct," with the default value set to the first option (index 0).

```cpp
// Add "High Cut Slope" parameter    
layout.add(std::make_unique<juce::AudioParameterChoice>("High Cut Slope", "High Cut Slope", stringArray, 0));
```

Similarly, the "High Cut Slope" parameter is added as a choice-based parameter, using the same `stringArray` and default value (index 0).

```cpp
// Return the parameter layout
return layout; }
```
Finally, the function returns the `ParameterLayout` `layout`, which now contains all the defined parameters. This layout will be used by the `AudioProcessorValueTreeState` to manage and control the parameters in the equalizer audio plugin.

# DSP Code Breakdown

## Plugin Processor h

Here i setup an audio Processing Chain with juces dsp module to make filters and process audio data.

In the Private area of the public class
```cpp
using Filter = juce::dsp::IIR::Filter<float>; using CutFilter = juce::dsp::ProcessorChain<Filter, Filter, Filter, Filter>; using MonoChain = juce::dsp::ProcessorChain<CutFilter, Filter, CutFilter>;  MonoChain leftChain, rightChain;
```

1. `using Filter = juce::dsp::IIR::Filter<float>;`: This line defines a type alias `Filter` for a single-order IIR filter for floating-point audio samples.
    
2. `using CutFilter = juce::dsp::ProcessorChain<Filter, Filter, Filter, Filter>;`: This line defines a type alias `CutFilter` for a processor chain that includes four instances of the `Filter` type. This represents a chain of four single-order IIR filters connected in series.
    
3. `using MonoChain = juce::dsp::ProcessorChain<CutFilter, Filter, CutFilter>;`: This line defines a type alias `MonoChain` for another processor chain. This chain includes three blocks of processors: `CutFilter`, `Filter`, and `CutFilter`. These represent a combination of two series of single-order IIR filters with a single-order IIR filter in between.
    
4. `MonoChain leftChain, rightChain;`: Two instances of the `MonoChain` are created: `leftChain` and `rightChain`. This suggests that this audio processing chain is meant to be applied separately to the left and right channels of the audio input.

## Plugin Processor.cpp

To summarize, this code sets up two separate audio processing chains (`leftChain` and `rightChain`) and processes mono audio data on each channel independently. The chains consist of single-order IIR filters and a combination of filters designed for audio equalization and processing. The `ProcessContextReplacing` objects are used to manage the audio data and apply the audio processing chains to the left and right channels, respectively. Finally, the chains are prepared for processing using the `ProcessSpec` object that defines the processing specifications.

```cpp
juce::dsp::AudioBlock<float> block(buffer); auto leftBlock = block.getSingleChannelBlock(0); auto rightBlock = block.getSingleChannelBlock(1);
```

1. `juce::dsp::AudioBlock<float> block(buffer);`: An `AudioBlock` is created, wrapping the audio buffer (`buffer`) containing the audio data to be processed.
    
2. `auto leftBlock = block.getSingleChannelBlock(0);`: An `AudioBlock` named `leftBlock` is extracted from the original `block`, representing the left channel audio data (channel index 0).
    
3. `auto rightBlock = block.getSingleChannelBlock(1);`: An `AudioBlock` named `rightBlock` is extracted from the original `block`, representing the right channel audio data (channel index 1).
    

```cpp
juce::dsp::ProcessContextReplacing<float> leftContext(leftBlock); juce::dsp::ProcessContextReplacing<float> rightContext(rightBlock);  leftChain.process(leftContext);
rightChain.process(rightContext);
```

1. `juce::dsp::ProcessContextReplacing<float> leftContext(leftBlock);`: A `ProcessContextReplacing` object is created for the `leftBlock`. This object holds information about the audio block to be processed, including read/write pointers to audio data.
    
2. `juce::dsp::ProcessContextReplacing<float> rightContext(rightBlock);`: A `ProcessContextReplacing` object is created for the `rightBlock`. Similar to the left context, it holds information about the audio block to be processed.
    
3. `leftChain.process(leftContext);`: The `leftChain` audio processing chain is applied to the left channel audio data. The `process()` function processes the audio data using the DSP operations defined in the `leftChain`.
    
4. `rightChain.process(rightContext);`: The `rightChain` audio processing chain is applied to the right channel audio data. The `process()` function processes the audio data using the DSP operations defined in the `rightChain`.
    

```cpp
juce::dsp::ProcessSpec spec;

spec.maximumBlockSize = samplesPerBlock;

spec.numChannels = 1;

spec.sampleRate = sampleRate;

leftChain.prepare(spec);
rightChain.prepare(spec);
```

1. `juce::dsp::ProcessSpec spec;`: A `ProcessSpec` object is created to specify the processing specifications, such as the maximum block size, the number of channels, and the sample rate.
    
2. `spec.maximumBlockSize = samplesPerBlock;`: The maximum block size for processing is set to `samplesPerBlock`. This value determines the number of audio samples that will be processed in each block.
    
3. `spec.numChannels = 1;`: The `numChannels` property is set to 1, indicating that the audio processing is performed on a mono audio stream.
    
4. `spec.sampleRate = sampleRate;`: The `sampleRate` property is set to the sample rate of the audio data to be processed.
    
5. `leftChain.prepare(spec);`: The `leftChain` audio processing chain is prepared with the specified `ProcessSpec`. This step initializes the processing chain with the provided parameters.
    
6. `rightChain.prepare(spec);`: The `rightChain` audio processing chain is prepared in a similar manner to the left channel.



# Peak Parameter Breakdown

## Plugin Processor . cpp

This block is used to setup and assign the chainSettigns Coefficients

```cpp
auto chainSettings = getChainSettings(apvts);

auto peakCoefficients = 
juce::dsp::IIR::Coefficients<float>::makePeakFilter(
sampleRate,
chainSettings.peakFrequency,
chainSettings.peakQuality, juce::Decibels::decibelsToGain(chainSettings.peakGainInDecibels)
);

*leftChain.get<ChainPositions::Peak>().coefficients = *peakCoefficients; *rightChain.get<ChainPositions::Peak>().coefficients = *peakCoefficients; }
```

Explanation:

1. `auto chainSettings = getChainSettings(apvts);`
    
    - This line calls the function `getChainSettings(apvts)` to retrieve the audio processing parameters from the `apvts` object and store them in a local variable named `chainSettings`.
2. `auto peakCoefficients = juce::dsp::IIR::Coefficients<float>::makePeakFilter(...)`
    
    - This line constructs a set of IIR (Infinite Impulse Response) filter coefficients for a peak filter.
    - It uses the `makePeakFilter` static function from `juce::dsp::IIR::Coefficients<float>` class to generate the coefficients.
    - The peak filter is defined by the `chainSettings.peakFrequency`, `chainSettings.peakQuality`, and `chainSettings.peakGainInDecibels` values.
3. `*leftChain.get<ChainPositions::Peak>().coefficients = *peakCoefficients;`
    
    - This line sets the coefficients of the peak filter in the left channel's processing chain.
    - It retrieves the pointer to the `Peak` node in the `leftChain` and assigns the coefficients from `peakCoefficients`.
4. `*rightChain.get<ChainPositions::Peak>().coefficients = *peakCoefficients;`
    
    - This line sets the coefficients of the peak filter in the right channel's processing chain.
    - It retrieves the pointer to the `Peak` node in the `rightChain` and assigns the coefficients from `peakCoefficients`.

###  setChainSettings

```cpp
ChainSettings setChainSettings(juce::AudioProcessorValueTreeState& apvts)
{
	ChainSettings settings;
	settings.lowCutFrequency =
	apvts.getRawParameterValue("Low CutFrequency")->load();
	
	settings.highCutFrequency =
	apvts.getRawParameterValue("High Cut Frequency")->load();
	
	settings.peakFrequency =
	apvts.getRawParameterValue("Peak Frequency")->load();
	
	settings.peakGainInDecibels =
	apvts.getRawParameterValue("Peak Gain")->load();
	
	settings.peakQuality = apvts.getRawParameterValue("Peak Quality")->load();
	
	settings.lowCutSlope =
	apvts.getRawParameterValue("Low Cut Slope")->load();
	
	settings.highCutSlope =
	apvts.getRawParameterValue("High Cut Slope")->load();
	
	return settings;
}
```

Explanation:

1. `ChainSettings setChainSettings(juce::AudioProcessorValueTreeState& apvts)`
    
    - This line defines the function `setChainSettings`.
    - It takes a reference to a `juce::AudioProcessorValueTreeState` object named `apvts` as an argument.
    - The return type of the function is `ChainSettings`, indicating that it will return an object of the `ChainSettings` struct or class.
2. `ChainSettings settings;`
    
    - This line declares a local variable `settings` of type `ChainSettings`.
    - `settings` will be used to store the audio processing parameter values retrieved from `apvts`.
3. `settings.lowCutFrequency = apvts.getRawParameterValue("Low Cut Frequency")->load();`
    
    - This line retrieves the value of the "Low Cut Frequency" parameter from the `apvts` object.
    - The value is accessed through a pointer, so `->load()` is used to dereference the pointer and retrieve the actual value.
    - The retrieved value is then assigned to the `lowCutFrequency` member of the `settings` object.
4. `settings.highCutFrequency = apvts.getRawParameterValue("High Cut Frequency")->load();`
    
    - This line is similar to the previous line but retrieves the value of the "High Cut Frequency" parameter and assigns it to the `highCutFrequency` member of the `settings` object.
5. `settings.peakFrequency = apvts.getRawParameterValue("Peak Frequency")->load();`
    
    - This line is similar to the previous lines but retrieves the value of the "Peak Frequency" parameter and assigns it to the `peakFrequency` member of the `settings` object.
6. `settings.peakGainInDecibels = apvts.getRawParameterValue("Peak Gain")->load();`
    
    - This line is similar to the previous lines but retrieves the value of the "Peak Gain" parameter and assigns it to the `peakGainInDecibels` member of the `settings` object.
7. `settings.peakQuality = apvts.getRawParameterValue("Peak Quality")->load();`
    
    - This line is similar to the previous lines but retrieves the value of the "Peak Quality" parameter and assigns it to the `peakQuality` member of the `settings` object.
8. `settings.lowCutSlope = apvts.getRawParameterValue("Low Cut Slope")->load();`
    
    - This line is similar to the previous lines but retrieves the value of the "Low Cut Slope" parameter and assigns it to the `lowCutSlope` member of the `settings` object.
9. `settings.highCutSlope = apvts.getRawParameterValue("High Cut Slope")->load();`
    
    - This line is similar to the previous lines but retrieves the value of the "High Cut Slope" parameter and assigns it to the `highCutSlope` member of the `settings` object.
10. `return settings;`
    
    - This line returns the `settings` object, which now contains all the retrieved audio processing parameter values.



## Plugin Processor . h

```cpp
struct ChainSettings
{    
	float peakFrequency{ 0 }, peakGainInDecibels{ 0 }, peakQuality{ 1.f };        float lowCutFrequency{ 0 }, highCutFrequency{ 0 };
	int lowCutSlope{ 0 }, highCutSlope{ 0 };
};
```

- `ChainSettings` is a custom C++ struct that defines a set of audio processing parameters.
- It contains the following members:
    - `peakFrequency`: A floating-point value representing the frequency of the peak filter.
    - `peakGainInDecibels`: A floating-point value representing the gain of the peak filter in decibels.
    - `peakQuality`: A floating-point value representing the quality factor of the peak filter.
    - `lowCutFrequency`: A floating-point value representing the frequency of the low-cut filter.
    - `highCutFrequency`: A floating-point value representing the frequency of the high-cut filter.
    - `lowCutSlope`: An integer value representing the slope of the low-cut filter.
    - `highCutSlope`: An integer value representing the slope of the high-cut filter.
- The struct members are initialized with default values in the curly braces (e.g., `peakFrequency{ 0 }`). If no explicit value is provided during the creation of a `ChainSettings` object, these default values will be used.

### Chain settings

```cpp
ChainSettings getChainSettings(juce::AudioProcessorValueTreeState& apvts);
```

- This is the function declaration for a function named `getChainSettings`.
- It takes a reference to a `juce::AudioProcessorValueTreeState` object named `apvts` as an argument.
- The return type of the function is `ChainSettings`, indicating that it will return an object of the `ChainSettings` struct.

### Chain Position Enum


```cpp
enum ChainPositions
{
	LowCut,
	Peak,
	HighCut
};
```

- This is an enumeration (enum) that defines a set of named constants for identifying different positions in an audio processing chain.
- The enum includes three constants: `LowCut`, `Peak`, and `HighCut`.
- Each constant is associated with an integer value by default (0 for the first constant, 1 for the second, and so on).
- The enum is used to differentiate between different stages of an audio processing chain. For example, it can be used to access specific elements in an array or container that represents the processing chain.








Dmup text


There are 2 ways to get parameter values from the apvts the first way iS to call get value and then get value but the issue with this approach is were given a normalized value and all functions that produce coeffs for our filter expect real world values not normalized values so im goin to use

GetRawParameterValues(stringref paramid)

always update parameters before the audio gets processed through it


# Low Cut Parameter Breakdown

## Auto Cut Coefficients and Low-Cut Filtering

this is responsible for automatically designing and applying low-cut filters to an audio signal. Let's break down the code and explain its functionality.

```cpp
auto cutCoefficients = juce::dsp::FilterDesign<float>::
designIIRHighpassHighOrderButterworthMethod(
chainSettings.lowCutFrequency,
getSampleRate(),
2 * (chainSettings.lowCutSlope + 1));
```

this calculates the coefficients for the low-cut filter using the JUCE DSP library's `designIIRHighpassHighOrderButterworthMethod`. It takes three arguments:

- `chainSettings.lowCutFrequency`: The cutoff frequency of the low-cut filter.
- `getSampleRate()`: A function that returns the sample rate of the audio signal.
- `2 * (chainSettings.lowCutSlope + 1)`: It calculates the filter order based on the selected slope. The higher the slope, the steeper the roll-off of the filter.

```cpp
auto& leftLowCut = leftChain.get<ChainPositions::LowCut>(); leftLowCut.setBypassed<0>(true); leftLowCut.setBypassed<1>(true); leftLowCut.setBypassed<2>(true); leftLowCut.setBypassed<3>(true);
```

Next, the code sets up the left channel's low-cut filter. The variable `leftLowCut` is a reference to the low-cut filter in the `leftChain`. It then sets all the stages of the filter to bypass mode (`setBypassed(true)`), meaning the filter won't have any effect initially.

```cpp
switch (chainSettings.lowCutSlope)
{ 
	case Slope_12:
	{
		*leftLowCut.get<0>().coefficients = *cutCoefficients[0];                leftLowCut.setBypassed<0>(false);
		break;
	}
	case Slope_24:
	{
		*leftLowCut.get<0>().coefficients = *cutCoefficients[0];
		leftLowCut.setBypassed<0>(false);
		*leftLowCut.get<1>().coefficients = *cutCoefficients[1];
		leftLowCut.setBypassed<1>(false);
		break;
	} // More cases for Slope_36 and Slope_48 follow... 
	
}
```

Here, a switch statement is used based on the `chainSettings.lowCutSlope`. This variable holds the selected slope for the low-cut filter, which can be one of the following: `Slope_12`, `Slope_24`, `Slope_36`, or `Slope_48`.

- For each case, the code selectively enables specific stages of the low-cut filter, starting from the first stage (`<0>`) and progressing to higher stages (`<1>`, `<2>`, etc.).
- The `*cutCoefficients[i]` refers to the calculated filter coefficients for the corresponding filter stage `i` from the `cutCoefficients` array.
- Setting `leftLowCut.setBypassed<stage>(false)` for a specific stage `stage` means the filter stage will be active (not bypassed) and will process the audio signal.

The same process is repeated for the right channel's low-cut filter using the `rightChain` and `rightLowCut` variables.

## Slope Enum

### Slope Enum

The `Slope` enum represents different slope options for the low-cut and high-cut filters. These slope options determine the steepness of the roll-off of the filters. The enum provides a set of named constants to choose from, making the code more readable and self-explanatory.

#### Enum Values:

- `Slope_12`: Represents a 12 dB/octave slope, which provides a gentle roll-off for the filter.
- `Slope_24`: Represents a 24 dB/octave slope, providing a steeper roll-off compared to `Slope_12`.
- `Slope_36`: Represents a 36 dB/octave slope, which further increases the roll-off steepness.
- `Slope_48`: Represents the steepest roll-off with a 48 dB/octave slope.

im able to select the desired slope option for both the low-cut and high-cut filters.

# DSP Refactoring

## Plugin Processor CPP
```cpp
#include "PluginProcessor.h" 
// ... 
void EqualizerAudioProcessor::updatePeakFilter(const ChainSettings& chainSettings)
{
	// Calculate peak filter coefficients based on the provided chain settings.
	auto peakCoefficients = juce::dsp::IIR::Coefficients<float>:
		makePeakFilter(getSampleRate(),
		chainSettings.peakFrequency, 
		chainSettings.peakQuality,
	juce::Decibels::decibelsToGain(chainSettings.peakGainInDecibels));
	
		updateCoefficients(leftChain.get<ChainPositions::Peak
		().coefficients, peakCoefficients);
		
		updateCoefficients(rightChain.get<ChainPositions::Peak>
		().coefficients, peakCoefficients); }
```

Explanation:

- The `updatePeakFilter` function is responsible for updating the peak filter coefficients for the equalizer.
- It takes a `ChainSettings` object as input, which contains the necessary parameters for calculating the peak filter coefficients: `peakFrequency`, `peakQuality`, and `peakGainInDecibels`.
- It calculates the peak filter coefficients using the `makePeakFilter` method of the `juce::dsp::IIR::Coefficients` class. This method creates coefficients for a peak filter with the specified parameters.
- The `peakCoefficients` variable holds the newly calculated peak filter coefficients.
- It updates the left and right channel's peak filter coefficients using the `updateCoefficients` function twice. This helper function updates the old coefficients with the newly calculated coefficients.

```cpp
void EqualizerAudioProcessor::updateCoefficients(Coefficients& old, const Coefficients& replacements)
{     // Copy the values of the newly calculated coefficients to the  memory location of the old coefficients.
	*old = *replacements;
}
```

Explanation:

- The `updateCoefficients` function is a helper function used to update filter coefficients.
- It takes two arguments: a reference to the old coefficients (`old`) and a constant reference to the new coefficients (`replacements`).
- It updates the filter coefficients by copying the values of the new coefficients to the memory location of the old coefficients.
- This function is used by the `updatePeakFilter`, `updateLowCutFilters`, and `updateHighCutFilters` functions to update their respective filter coefficients.

```cpp
void EqualizerAudioProcessor::updateLowCutFilters(const ChainSettings& chainSettings)
{     // Calculate low-cut filter coefficients based on the provided chain settings.
	auto lowCutCoefficients=juce::dsp::FilterDesign<float>::
	designIIRHighpassHighOrderButterworthMethod(chainSettings.lowCutFrequency, getSampleRate(),
	2 * (chainSettings.lowCutSlope + 1));
	
	auto& leftLowCut = leftChain.get<ChainPositions::LowCut>();     updateCutFilter(leftLowCut, lowCutCoefficients, chainSettings.lowCutSlope);
	
	auto& rightLowCut = rightChain.get<ChainPositions::LowCut>();     updateCutFilter(rightLowCut, lowCutCoefficients, chainSettings.lowCutSlope);
}
```

Explanation:

- The `updateLowCutFilters` function is responsible for updating the low-cut filter coefficients for the equalizer.
- It takes a `ChainSettings` object as input, which contains the necessary parameters for calculating the low-cut filter coefficients: `lowCutFrequency` and `lowCutSlope`.
- It calculates the low-cut filter coefficients using the `designIIRHighpassHighOrderButterworthMethod` method of the `juce::dsp::FilterDesign` class. This method creates coefficients for a high-order Butterworth highpass filter with the specified low-cut frequency and slope.
- The `lowCutCoefficients` variable holds the newly calculated low-cut filter coefficients.
- It updates the left and right channel's low-cut filter coefficients and slope using the `updateCutFilter` function twice. This helper function updates the old coefficients and slope with the newly calculated values.

```cpp
void EqualizerAudioProcessor::updateHighCutFilters(const ChainSettings& chainSettings)
{
	auto highCutCoefficients = juce::dsp::FilterDesign<float>::designIIRHighpassHighOrderButterworthMethod(chainSettings.highCutFrequency, getSampleRate(), 2 * (chainSettings.highCutSlope + 1));
	auto& leftHighCut = leftChain.get<ChainPositions::HighCut>();     updateCutFilter(leftHighCut, highCutCoefficients, chainSettings.highCutSlope);
	auto& rightHighCut = rightChain.get<ChainPositions::HighCut>();     updateCutFilter(rightHighCut, highCutCoefficients, chainSettings.highCutSlope);
}
```

Explanation:

- The `updateHighCutFilters` function is responsible for updating the high-cut filter coefficients for the equalizer.
- It takes a `ChainSettings` object as input, which contains the necessary parameters for calculating the high-cut filter coefficients: `highCutFrequency` and `highCutSlope`.
- It calculates the high-cut filter coefficients using the `designIIRHighpassHighOrderButterworthMethod` method of the `juce::dsp::FilterDesign` class. This method creates coefficients for a high-order Butterworth highpass filter with the specified high-cut frequency and slope.
- The `highCutCoefficients` variable holds the newly calculated high-cut filter coefficients.
- It updates the left and right channel's high-cut filter coefficients and slope using the `updateCutFilter` function twice. This helper function updates the old coefficients and slope with the newly calculated values.

```cpp
void EqualizerAudioProcessor::updateFilters()
{
		auto chainSettings = getChainSettings(apvts);
		updateLowCutFilters(chainSettings);
	    updatePeakFilter(chainSettings);
	    updateHighCutFilters(chainSettings);
}
```

Explanation:

- The `updateFilters` function is the main entry point for updating all the filters in the audio processor's equalizer.
- It starts by retrieving the current chain settings from the AudioProcessorValueTreeState (`apvts`) using the `getChainSettings` function.
- It then updates the low-cut filters, peak filter, and high-cut filters in sequence by calling the respective update functions (`updateLowCutFilters`, `updatePeakFilter`, and `updateHighCutFilters`) with the retrieved `chainSettings`.

These functions work together to update the filter coefficients and settings for the equalizer based on the given `ChainSettings`
