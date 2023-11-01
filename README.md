Generating a signal using DAC Polling Mode and DMA Mode. 

## R-2R DAC using STM32 Nucleo 
DAC peripherals available in STM32 microcontrollers are based on the common R-2R Ladder network.

STM32 microcontrollers usually provide only a DAC with one or two dedicated outputs, with the first one with two outputs and the other one with just one output.

DAC channels can be configured to work in **8/12-bit mode**, and the conversion of the two channels can be performed **independently** or **simultaneously**. 
>[!Note] The **simultaneous mode** is useful in those applications where two independent but synchronous signals must be generated (for example, in audio applications). 

Like the ADC peripheral, even the DAC **can be triggered by a dedicated timer**, in order to generate analog signals at a given frequency.


# 1. Polling Mode:
## Scope Output:
![alt text](https://github.com/haris-mujeeb/DAC-using-STM32-Nucleo-F410RB/blob/main/DAC%20Polling%20Mode%20-Scope%20Output.jpg)
*Note: In the code uploaded above, only sine waves are generated for both modes (no triangle wave is generated).

1. First, starting the peripheral by calling the function:
```C
HAL_DAC_Start(&hdac, DAC_CHANNEL_1);
```

2. Once the DAC channel is enabled, we can perform a conversion by calling the function:
```C
HAL_DAC_SetValue(&hdac, DAC_CHANNEL_1, DAC_ALIGN_12B_R, data);
```
>[!Note] 
>For, *12 bit* DAC, max. value = *4095*
>and *8 bit* DAC, max. value = *255*

> [!Note] 
> where the **Alignment** parameter can assume the value _DAC_ALIGN_8B_R_ to drive the DAC in 8-bit mode, _DAC_ALIGN_12B_L_ or _DAC_ALIGN_12B_R_ to drive the DAC in 12-bit mode passing the output value left- or right-aligned respectively.

### Example Code for [[STM32 Nucleo F410RB]]:
```C
int main(void)
{
	uint16_t valuesBuffer[NO_OF_SAMPLES], value;
	for (uint16_t i = 0; i < NO_OF_SAMPLES; i++) {
		value = (uint16_t) rint( (sinf( ((2*PI)/NO_OF_SAMPLES)*i)
																	 + 1 )*2048 );
		valuesBuffer[i] = value < 4096 ? value : 4095;
	}
...

... 
  HAL_DAC_Start(&hdac, DAC_CHANNEL_1);	// for Polling Mode
  while (1)
  {
// <-- using Poll Mode -->
	  for (uint16_t i = 0; i < NO_OF_SAMPLES; i++) {
		  HAL_DAC_SetValue(&hdac, DAC_CHANNEL_1, 
								  DAC_ALIGN_12B_R, valuesBuffer[i]);
		  HAL_Delay(1);
	  }
...
```
## 2. DMA Mode:
## Scope Output:
![alt text](https://github.com/haris-mujeeb/DAC-using-STM32-Nucleo-F410RB/blob/main/DAC%20DMA%20Mode%20-Scope%20Output.jpg)

- Enable the *Timer* that is *connected to DAC DMA channel*. You can find the specific channel in MCUs datasheet.

- Enable *DAC* and add DAC's *DMA request* from DMA setting.

- Enable _'Trigger Out Event'_ for DAC in DAC Parameters Settings.

- Also make sure to run Timer with *Trigger Output* set to *Update Event*.
```C
...
int main(void)
{
  /* USER CODE BEGIN 1 */
	// Defining an array with sine wave values.
	uint16_t valuesBuffer[NO_OF_SAMPLES], value;
	for (uint16_t i = 0; i < NO_OF_SAMPLES; i++) {
		value = (uint16_t) rint( (sinf( ((2*PI)/NO_OF_SAMPLES)*i)
																	 + 1 )*2048 );
		valuesBuffer[i] = value < 4096 ? value : 4095;
	}
...

...
  HAL_TIM_Base_Start(&htim6);	//
  HAL_DAC_Start_DMA(&hdac, DAC_CHANNEL_1,
	   (uint32_t*)valuesBuffer, NO_OF_SAMPLES, DAC_ALIGN_12B_R);
while(1){};
... 
```
