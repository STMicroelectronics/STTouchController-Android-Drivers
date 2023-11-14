#  FTS Capacitive touch screen controller (FingerTipS) Guide

## Introduction

The driver was developed and tested in Linux environment for Android 5.0.2 on Qualcomm Platform Architecture and supports I2C or SPI connection.

The driver is made up by several files. Each file regroup functions and logics regarding a particular aspect

The main files are:
• fts.c/.h (mandatory) = file which implement the main OS related functions of a common driver such as driver mounting, probe, interrupt handler, exposition of file nodes, suspend and resume etc.

• fts_proc.c = file which expose a file node called driver_test in the path
/proc/fts/driver_test

• fts_fw.h (optional) = file which contains the definitions of two variables:
o an array of bytes which is the binary representation of the FW file that driver should flash in the IC
o a variable which indicate the dimension of the array.

• fts_limits.h (optional) = file which contains the definitions of two variables:
o an array of bytes which is the binary representation of the limits file used during the mass production test and which contains thresholds to validate the good initialization of the IC
o a variable which indicate the dimension of the array.

• fts_lib (mandatory) = this folder contains a collection of files which together constitute an API for executing any kind of operation (from low level to high level) on the IC.

## Driver source file prepare

1. Copy the content of the driver source code in:
```
/kernel/drivers/input/touchscreen
```

2. Modify the make file inside the previous path in order to build the Fingertip device driver as follow:
```
obj-y += STTouchControllerG2/
```
In this way both the primary file and the library files will be compiled with the entire image

## Add device declaration in the board devicetree

Please add  FTS Capacitive touch screen controller (FingerTipS) device declaration info in the board devicetree, you can refer the Appendix st-fts-i2c-dtsi and st-fts-spi-dtsi to see how to set device properties.

## Appendix

### **st-fts-i2c-dtsi**

Devicetree binding for  FTS (FingerTipS) touch driver

Add the device tree node in the building environment using the following reference 
```dts

&i2c_6 {
    status = "ok";
    st_fts@49 {
        compatible = "st,fts";
        reg = <0x49>; //SAD
        interrupt-parent = <&msm_gpio>;
        interrupts = <49 0x00>;
        vdd-supply = <&pm8994_l18>; //1.8V power regulator
        avdd-supply = <&pm8994_l9>; //3.3V power regulator
        pinctrl-names = "pmx_ts_active", "pmx_ts_suspend";
        pinctrl-0 = <&ts_active>;
        pinctrl-1 = <&ts_suspend>;
        st,irq-gpio = <&msm_gpio 49 0x00>; //interrupt gpio
        //st,reset-gpio = <&msm_gpio 50 0x00>; //specify the gpio which control the reset pin of the IC (if available, not mandatory)
        st,regulator_dvdd = "vdd";
        st,regulator_avdd = "avdd";
        };
    };
```

### **st-fts-spi-dtsi**

Add the device tree node in the building environment using the following reference 
```dts

&spi_0 {
    status = "ok";
    st_fts@49 {
        compatible = "st,fts";
        reg = <0x49>; //SAD
        interrupt-parent = <&msm_gpio>;
        interrupts = <49 0x00>;
        vdd-supply = <&pm8994_l18>; //1.8V power regulator
        avdd-supply = <&pm8994_l9>; //3.3V power regulator
        pinctrl-names = "pmx_ts_active", "pmx_ts_suspend";
        pinctrl-0 = <&ts_active>;
        pinctrl-1 = <&ts_suspend>;
        st,irq-gpio = <&msm_gpio 49 0x00>; //interrupt gpio
        //st,reset-gpio = <&msm_gpio 50 0x00>; //specify the gpio which control the reset pin of the IC (if available, not mandatory)
        st,regulator_dvdd = "vdd";
        st,regulator_avdd = "avdd";
        };
    };
```

The device is attached to an I2C master (in the example is the i2c_6) as an I2C slave with address 0x49. 

It is mandatory to assign 1.8V and 3.3V power regulators and a interrupt gpio (in this example the gpio number is 49) which usually is kept HIGH and it will be moved LOW only when an interrupt is requested. It is optional to specify a reset gpio for performing the system reset of the chip. 

In case no reset gpio is available the IC will be still able to perform system reset using a hardware command sent with an I2C transaction.

It is important remember that the information contained in the device tree node will be used by the pase_dt function inside fts.c, therefore if some change is made to some labels, apply the same change also to the function.