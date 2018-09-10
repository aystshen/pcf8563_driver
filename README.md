# Philips pcf8563 Driver
This is the Linux driver for the Philips PCF8563 RTC.  

This program is free software; you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation; either version 2 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be a reference to you, when you are integrating the Philips's pcf8563 IC into your system, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU General Public License for more details.

## Usage
1. Copy drivers/rtc/rtc-pcf8563.c to kernel/drivers/rtc/ directory.
2. Modify the kernel/drivers/rtc/Kconfig as follows
```
config RTC_DRV_PCF8563
	tristate "Philips PCF8563/Epson RTC8564"
	help
	  If you say yes here you get support for the
	  Philips PCF8563 RTC chip. The Epson RTC8564
	  should work as well.

	  This driver can also be built as a module. If so, the module
	  will be called rtc-pcf8563.
```
3. Modify the kernel/drivers/rtc/Makefile as follows
```
obj-$(CONFIG_RTC_DRV_PCF8563)	+= rtc-pcf8563.o
```
4. Modify the dts as follows
```
&i2c2 {
	status = "okay";

	rtc@51 {
		compatible = "nxp,pcf8563";
		reg = <0x51>;
		irq-gpios = <&gpio1 GPIO_A3 GPIO_ACTIVE_LOW>;
	};
};
```

## Notes:
1. The alarm clock only supports day, hour, minute, and does not support seconds.
2. The second alarm is not supported, so you need to add the following code, otherwise the alarm function will be abnormal. 
```
static int pcf8563_rtc_set_alarm(struct device *dev, struct rtc_wkalrm *tm)
{
    struct i2c_client *client = to_i2c_client(dev);
    unsigned char buf[4];
    int err;

    /* The alarm has no seconds, round up to nearest minute */
    if(tm->time.tm_sec) {
        unsigned long time;
        rtc_tm_to_time(&tm->time, &time);

        time += 60 - tm->time.tm_sec;
        rtc_time_to_tm(time, &tm->time);
    }
...
```

## Developed By
* ayst.shen@foxmail.com

## License
```
An I2C driver for the Philips PCF8563 RTC
Copyright 2005-06 Tower Technologies

Author: Alessandro Zummo <a.zummo@towertech.it>
Maintainers: http://www.nslu2-linux.org/

based on the other drivers in this same directory.

http://www.semiconductors.philips.com/acrobat/datasheets/PCF8563-04.pdf

This program is free software; you can redistribute it and/or modify
it under the terms of the GNU General Public License version 2 as
published by the Free Software Foundation.
```
