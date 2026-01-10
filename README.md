# Sustainably Smart Facility using AWS Cloud

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![ESP32](https://img.shields.io/badge/Platform-ESP32-blue.svg)](https://www.espressif.com/en/products/socs/esp32)
[![AWS IoT](https://img.shields.io/badge/Cloud-AWS%20IoT-orange.svg)](https://aws.amazon.com/iot/)

## 📋 Overview

An IoT-based environmental monitoring and control system designed for indoor environments such as facilities, factories, or offices. The system monitors temperature, humidity, and gas levels in real-time to ensure environmental safety and comfort, with automated responses through fan and servo motor control.

### Key Features

- **Real-time Environmental Monitoring**
  - Temperature sensing via DHT11 sensor
  - Humidity monitoring
  - Gas level detection using MQ-2 sensor
  
- **Cloud Integration**
  - Secure MQTT connection to AWS IoT Core
  - Real-time data publishing every 30 seconds
  - Cloud-based data storage in DynamoDB
  
- **Automated Alert System**
  - Email notifications via Amazon SNS
  - Configurable thresholds (Temperature: 30°C, Gas: 1200, Humidity: 70%)
  
- **Remote Actuator Control**
  - Fan control (ON/OFF) via cloud commands
  - Servo motor control (0-180 degrees)

## 🏗️ System Architecture

```
┌─────────────┐        ┌──────────────┐        ┌─────────────┐
│   ESP32     │◄──────►│  AWS IoT     │◄──────►│   Lambda    │
│  + Sensors  │  MQTT  │    Core      │ Trigger │  Function   │
└─────────────┘        └──────────────┘        └─────────────┘
                              │                        │
                              │                        ▼
                              │                 ┌─────────────┐
                              │                 │  DynamoDB   │
                              │                 └─────────────┘
                              │                        │
                              │                        ▼
                              │                 ┌─────────────┐
                              └────────────────►│     SNS     │
                                                │   Alerts    │
                                                └─────────────┘
```

## 🛠️ Hardware Requirements

| Component | Description |
|-----------|-------------|
| ESP32 | Main microcontroller board |
| DHT11 | Temperature & Humidity sensor |
| MQ-2 | Gas sensor (Smoke, LPG, CO) |
| Fan Module | DC fan (controlled via relay/MOSFET) |
| Servo Motor | SG90 or similar (0-180 degrees) |
| Breadboard & Wires | For connections |
| Power Supply | 5V for ESP32 and peripherals |

### Pin Configuration

```cpp
DHT11 Sensor:    GPIO 4
MQ-2 Sensor:     GPIO 35 (ADC)
Fan Control:     GPIO 27
Servo Motor:     GPIO 25
```

## ☁️ AWS Services Configuration

### Services Used

1. **AWS IoT Core**
   - Device management and MQTT broker
   - Topic: `sensor/data` (publish)
   - Topic: `actuator/command` (subscribe)

2. **AWS Lambda**
   - Processes incoming sensor data
   - Checks thresholds and triggers alerts
   - Stores data in DynamoDB

3. **Amazon DynamoDB**
   - Table: `ProcessSensorData`
   - Stores sensor readings with timestamps

4. **Amazon SNS**
   - Topic: `SensorAlertTopic`
   - Sends email alerts on threshold violations

## 🚀 Getting Started

### Prerequisites

- Arduino IDE with ESP32 board support
- AWS Account with IoT Core enabled
- Python 3.x (for AWS Lambda deployment)

### Step 1: AWS IoT Setup

1. **Create an IoT Thing**
   ```bash
   # In AWS IoT Console
   - Navigate to Manage > Things > Create
   - Download certificates (certificate.pem.crt, private.pem.key, AmazonRootCA1.pem)
   ```

2. **Create IoT Policy**
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": ["iot:*"],
         "Resource": ["*"]
       }
     ]
   }
   ```

3. **Attach Policy to Certificate**

### Step 2: DynamoDB Setup

1. Create table `ProcessSensorData`
2. Primary key: `sensor_id` (String)
3. Sort key: `timestamp` (Number)

### Step 3: Lambda Function Setup

1. Create Lambda function with Python 3.x runtime
2. Copy code from `Lambda_Code.txt`
3. Update the following variables:
   - `table = 'ProcessSensorData'`
   - `sns_topic_arn = 'your-sns-topic-arn'`
4. Add environment variables for region and account ID
5. Attach IAM role with permissions:
   - DynamoDB PutItem
   - SNS Publish

### Step 4: SNS Setup

1. Create SNS Topic: `SensorAlertTopic`
2. Create email subscription
3. Confirm subscription via email

### Step 5: IoT Rule

Create IoT Rule to trigger Lambda:
```sql
SELECT * FROM 'sensor/data'
```
Action: Invoke Lambda function

### Step 6: ESP32 Configuration

1. Install required libraries in Arduino IDE:
   ```
   - WiFi (built-in)
   - PubSubClient
   - ArduinoJson
   - ESP32Servo
   - DHT sensor library
   ```

2. Create `secrets.h` file:
   ```cpp
   #define THINGNAME "YourThingName"
   
   const char WIFI_SSID[] = "YourWiFiSSID";
   const char WIFI_PASSWORD[] = "YourWiFiPassword";
   const char AWS_IOT_ENDPOINT[] = "your-endpoint.iot.region.amazonaws.com";
   
   // Amazon Root CA 1
   static const char AWS_CERT_CA1[] PROGMEM = R"EOF(
   -----BEGIN CERTIFICATE-----
   [Your Root CA Certificate]
   -----END CERTIFICATE-----
   )EOF";
   
   // Device Certificate
   static const char AWS_CERT_CRT[] PROGMEM = R"KEY(
   -----BEGIN CERTIFICATE-----
   [Your Device Certificate]
   -----END CERTIFICATE-----
   )KEY";
   
   // Device Private Key
   static const char AWS_CERT_PRIVATE[] PROGMEM = R"KEY(
   -----BEGIN RSA PRIVATE KEY-----
   [Your Private Key]
   -----END RSA PRIVATE KEY-----
   )KEY";
   ```

3. Upload `Esp32_Code.txt` to ESP32

## 📊 Data Format

### Published Data (sensor/data)
```json
{
  "temperature": 25.5,
  "humidity": 60.0,
  "gas": 850
}
```

### Subscribed Commands (actuator/command)
```json
{
  "fan": 1,
  "servo": 90
}
```

## 🔧 Threshold Configuration

Modify in Lambda function:
```python
TEMPERATURE_THRESHOLD = 30.0  # °C
GAS_VALUE_THRESHOLD = 1200     # Analog reading
HUMIDITY_THRESHOLD = 70.0      # %
```

## 📈 Monitoring

- **AWS IoT Console**: Monitor MQTT messages in real-time
- **CloudWatch Logs**: View Lambda execution logs
- **DynamoDB Console**: Query stored sensor data
- **Serial Monitor**: Debug ESP32 locally (115200 baud)

## 🐛 Troubleshooting

| Issue | Solution |
|-------|----------|
| ESP32 won't connect to WiFi | Check SSID/password in secrets.h |
| AWS IoT connection fails | Verify certificates and endpoint |
| No Lambda triggers | Check IoT Rule SQL and action |
| Sensor readings incorrect | Calibrate sensors, check pin connections |
| Fan/Servo not responding | Verify GPIO pins and power supply |

## 📁 Project Structure

```
IoT_Project/
├── README.md                 # This file
├── LICENSE                   # MIT License
├── .gitignore               # Git ignore rules
├── hardware/
│   ├── Esp32_Code.ino       # ESP32 firmware
│   └── secrets.h            # WiFi & AWS credentials (not in repo)
├── lambda/
│   └── Lambda_Function.py   # AWS Lambda code
└── docs/
    └── Cloud Report.pdf     # Detailed documentation
```

## 🔒 Security Notes

- **Never commit `secrets.h`** to version control
- Use AWS IAM least-privilege policies
- Rotate certificates periodically
- Enable AWS CloudTrail for audit logging

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 👤 Author

**Hussein Mohammed**
- GitHub: [@hussiny7](https://github.com/hussiny7)
- Email: hussin77.hm@gmail.com
- Affiliation: Master Student, University of Calabria, Italy

## 🙏 Acknowledgments

- University of Calabria
- AWS IoT documentation and community
- ESP32 and Arduino communities
- Open-source library contributors

## 📞 Support

For issues, questions, or contributions, please open an issue on GitHub or contact the author.

---

**Last Updated:** January 2026
