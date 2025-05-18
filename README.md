# SPS30_I2C MicroPython Driver

A MicroPython driver for the **Sensirion SPS30** particulate matter sensor over I2C.

This driver provides an object-oriented interface to initialize the sensor, start/stop measurements, read PM data, access status flags, and perform housekeeping tasks such as fan cleaning or sleep/wakeup control.

A manual for the SPS30 can be found on Sensirion's website: [Dataasheet SPS30](https://sensirion.com/media/documents/8600FF88/64A3B8D6/Sensirion_PM_Sensors_Datasheet_SPS30.pdf)


## 📦 Features

- Supports **IEEE754 float** and **16-bit unsigned integer** measurement modes
- Handles **CRC8 integrity checks** on sensor data
- Reads:
  - PM1.0 / PM2.5 / PM4.0 / PM10 concentrations, typical particle size
  - Number concentration per size bin
  - Typical particle size
- Access to:
  - Sensor status register
  - Auto-cleaning interval
  - Firmware version
- Implements:
  - Manual fan cleaning trigger
  - Sleep and wake control


## 🛠️ Requirements

- MicroPython-compatible board (e.g. Raspberry Pi Pico, ESP32)
- `machine.I2C` or `SoftI2C` object
- Sensirion SPS30 sensor module (I2C variant)


## 🧰 Class Overview

### `SPS30_I2C(i2c, datatype="UI16")`

Creates a new SPS30 driver instance.

#### Args:
- `i2c` (I2C): An initialized MicroPython I2C object.
- `datatype` (str): `'UI16'` (default) or `'FP32'` to select the data format.

#### Attributes:
- `dict_values`: Dictionary of last-read measurement values (see blow).
- `dict_register`: Status flags like `"Fan error"` and `"Laser error"`.
- `dict_fimware_verion`: Major and minor firmware verision.


## 🔧 Example Usage

```python
'''
Test and example of the SPS30 micropython driver using the I2C protocol.
'''

from machine import Pin, SoftI2C
import time
from u_sps30 import SPS30_I2C

# Iniaitialize the I2C port
i2c = SoftI2C(scl=Pin(5), sda=Pin(4), freq=10000)

# Create an instance of the SDS driver.

test = SPS30_I2C(i2c, datatype="FP32") # Standard datatype is UInt
test.wakeup()

# Give information of the SPS30 device
print("*** DEVICE INFORMATION ***")
print(test.read_status_register())
print("SPS30 firmware verison:", test.read_firmware_version())
print("Intervall self cleaning:", test.read_auto_cleaning_interval())
print("Product type:", test.read_product_type())
print("Serial Number: ", test.read_serial_number())

# Doing a measurement
print("*** MEASURMENT ***")
test.start_measurement()
time.sleep(0.3)
print(f"Data ready flag:{test.read_data_ready_flag()}")
data = test.read_data()
print("Recorded Data:", data)
test.stop_measurement()
time.sleep(0.5)
print(f"Data ready flag:{test.read_data_ready_flag()}")

# Testing  Sleep mode (reducing energy consumption)
print("*** TESTING SLEEP MODE ***")
test.sleep()
time.sleep(5)
test.wakeup()
time.sleep(2)
test.start_measurement()
print(test.read_data())

# Testing fan vleaning and put device to sleep to conserve energy
print("*** TESTING FAN/FINISHING UP (SLEEP) ***")
test.start_fan_cleaning() # Runs for 10 s
time.sleep(10)
# # Put the SPS30 into sleep to save energy
test.sleep()
```
### Typical output:
```
 SPS30 waking up
*** DEVICE INFORMATION ***
Reading SPS30 register data. Warning: Formally works, but I'm not able to debug.
{'Fan error': False, 'Fan speed error': False, 'Laser error': False}
SPS30 reading firmware version
SPS30 firmware verison: {'major': 2, 'minor': 3}
Intervall self cleaning: 604800
SPS30 reading product type
Product type: 00080000
SPS30 reading serial number
b'55tD0NED\x869B\xafEB E0\xba3F\x85042\x00\x00\x81\x00\x00\x81\x00\x00\x81\x00\x00\x81\x00\x00\x81\x00\x00\x81\x00\x00\x81\x00\x00\x81'
[53, 53, 68, 48, 69, 68, 57, 66, 69, 66, 69, 48, 51, 70, 48, 52, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
Serial Number:  55D0ED9BEBE03F04
*** MEASURMENT ***
Measurement started
SPS30 reading data ready flag
Data ready flag:True
Recorded Data: {'pc_pm10.0': 32.93801, 'mc_pm2.5': 4.361726, 'typical_size': 0.3564478, 'mc_pm4.0': 4.361728, 'mc_pm10.0': 4.361725, 'pc_pm0.5': 28.18589, 'mc_pm1.0': 4.124712, 'pc_pm1.0': 32.75014, 'pc_pm2.5': 32.91869, 'pc_pm4.0': 32.93015}
Measurement stopped
SPS30 reading data ready flag
Data ready flag:False
*** TESTING SLEEP MODE ***
Setting SPS30 into sleep mode.
SPS30 waking up
Measurement started
{'pc_pm10.0': 32.93801, 'mc_pm2.5': 4.361726, 'typical_size': 0.3564478, 'mc_pm4.0': 4.361728, 'mc_pm10.0': 4.361725, 'pc_pm0.5': 28.18589, 'mc_pm1.0': 4.124712, 'pc_pm1.0': 32.75014, 'pc_pm2.5': 32.91869, 'pc_pm4.0': 32.93015}
*** TESTING FAN/FINISHING UP (SLEEP) ***
SPS30 sending self cleaning command
Setting SPS30 into sleep mode.
```

## 📋 Available Methods

| Method                         | Description                                                  |
|--------------------------------|--------------------------------------------------------------|
| `start_measurement()`         | Begin continuous measurement                                 |
| `stop_measurement()`          | Stop measurement and halt sensor data collection            |
| `read_data()`                 | Return measurement data (dict of `float` or `int`)         |
| `read_data_ready_flag()`      | Returns `True` if new data is available                      |
| `read_status_register()`      | Return a `dict` with laser/fan error flags                   |
| `read_product_type()`         | Returns the product type info string (number)                 |
| `clear_status_register()`     | Clears all status flags                                      |
| `read_firmware_version()`     | Returns firmware version (`int`)                             |
| `read_serial_nmber()`         | Returns a string containing the serial number                |
| `read_auto_cleaning_interval()` | Returns cleaning interval in seconds (`int`)              |
| `start_fan_cleaning()`        | Triggers fan cleaning cycle                                  |
| `sleep()` / `wakeup()`        | Put the sensor to sleep or resume operation                  |


## 📖 Supported Measurements

The sensor returns the following values in order:

1. – mass concentration PM1.0 [μg/m³]  
2. – mass concentration PM2.5 [μg/m³]  
3. – mass concentration PM4.0 [μg/m³]  
4. – mass concentration PM10 [μg/m³]  
5. – Particle count >0.5μm [/cm³]  
6. – Particle count >1.0μm [/cm³]  
7. – Particle count >2.5μm [/cm³]  
8. – Particle count >4.0μm [/cm³]  
9. – Particle count >10.0μm [/cm³]  
10. – Typical particle size, depends on the selected datatype FP32: [μm], UInt: [nm]

### Measurement values are returned as a dict:

```python
dict_values = {"mc_pm1.0"  : None,
                    "mc_pm2.5"  : None,
                    "mc_pm4.0"  : None,
                    "mc_pm10.0" : None,
                    "pc_pm0.5"  : None,
                    "pc_pm1.0"  : None,
                    "pc_pm2.5"  : None,
                    "pc_pm4.0"  : None,
                    "pc_pm10.0" : None,
                    "typical_size": None}
```


## 📝 License

MIT License. Based on hardware protocols by Sensirion.  
Partial command references adapted from [@dvsu's sps30.py](https://github.com/dvsu/sps30).



## 📎 Comments and TODO's

- Add support for passive read mode (not just continuous), this seems to implemented in newer versions/firmware of the SPS30. Might be good for energy sensitive implementations.
- Implement `read_serial_number()` and `read_product_type()`
- The driver is not async-friendly.