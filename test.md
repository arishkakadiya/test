Here is an expanded version of the **README.md** file, including more detailed explanations, examples, and instructions:

```markdown
# Rideau Canal Skateway Monitoring System

## Scenario Description

The Rideau Canal Skateway in Ottawa is a globally recognized attraction and UNESCO World Heritage Site. It attracts millions of visitors annually, but its ice conditions and weather factors must be constantly monitored to ensure skater safety. 

The National Capital Commission (NCC) requires a **real-time monitoring system** to:
1. Simulate IoT sensors to monitor ice conditions and weather factors at three key locations (Dow's Lake, Fifth Avenue, and the NAC).
2. Process incoming sensor data to detect unsafe conditions in real time.
3. Store aggregated results in Azure Blob Storage for analysis and decision-making.

This project leverages **Azure IoT Hub**, **Azure Stream Analytics**, and **Azure Blob Storage** to build a scalable and reliable monitoring solution.

---

## System Architecture

The Rideau Canal Skateway Monitoring System is structured as follows:

1. **IoT Sensor Simulation**:
   - Sensors at three locations simulate real-time data every 10 seconds, including:
     - Ice Thickness (cm)
     - Surface Temperature (°C)
     - Snow Accumulation (cm)
     - External Temperature (°C)
   - JSON payloads are sent to **Azure IoT Hub**.

2. **Azure IoT Hub**:
   - Acts as the central message broker, receiving data from simulated IoT sensors and forwarding it to **Azure Stream Analytics**.

3. **Azure Stream Analytics**:
   - Processes sensor data in real time and performs the following:
     - Aggregates data over 5-minute tumbling windows for each location.
     - Calculates average ice thickness and maximum snow accumulation.
   - Outputs processed data to **Azure Blob Storage**.

4. **Azure Blob Storage**:
   - Stores aggregated data in a structured format for further analysis and historical recordkeeping.

### System Diagram

![System Architecture](screenshots/system-architecture-diagram.png)

---

## Implementation Details

### 1. IoT Sensor Simulation

The **IoT Sensor Simulation** generates JSON payloads every 10 seconds. Each payload includes the following attributes:

| Attribute          | Description                                  |
|--------------------|----------------------------------------------|
| `location`         | Location of the sensor (e.g., Dow's Lake).  |
| `iceThickness`     | Thickness of the ice in centimeters.        |
| `surfaceTemperature` | Temperature of the ice surface in Celsius. |
| `snowAccumulation` | Snow accumulation on the ice in centimeters.|
| `externalTemperature` | Ambient temperature in Celsius.           |
| `timestamp`        | UTC timestamp of the data.                  |

#### Example JSON Payload

```json
{
  "location": "Dow's Lake",
  "iceThickness": 27,
  "surfaceTemperature": -1,
  "snowAccumulation": 8,
  "externalTemperature": -4,
  "timestamp": "2024-11-23T12:00:00Z"
}
```

The simulation script is located in the `sensor-simulation/` directory. It uses the **Azure IoT Device SDK** to push messages to Azure IoT Hub.

#### Running the Simulation

1. Install dependencies:
   ```bash
   pip install -r sensor-simulation/requirements.txt
   ```
2. Run the simulation:
   ```bash
   python sensor-simulation/sensor_simulation.py
   ```
3. Verify messages are sent to Azure IoT Hub.

---

### 2. Azure IoT Hub Configuration

To set up Azure IoT Hub:
1. Create an Azure IoT Hub in the Azure Portal.
2. Add three devices for each sensor location:
   - `DowsLakeSensor`
   - `FifthAvenueSensor`
   - `NACSensor`
3. Configure message routing to **Azure Stream Analytics**:
   - Set the endpoint to forward messages to the Stream Analytics job.

#### Key Configuration Details
- **Message Format**: JSON
- **Message Routing**: Default routing to Stream Analytics.

---

### 3. Azure Stream Analytics Job

#### Job Configuration
- **Input**: Azure IoT Hub
- **Output**: Azure Blob Storage

#### SQL Query

The following query processes incoming sensor data:

```sql
SELECT
    System.Timestamp AS WindowEndTime,
    location,
    AVG(iceThickness) AS AvgIceThickness,
    MAX(snowAccumulation) AS MaxSnowAccumulation
FROM
    IoTHubInput
GROUP BY
    TumblingWindow(Duration(minute, 5)), location
```

#### Output Data Example
```json
{
  "WindowEndTime": "2024-11-23T12:05:00Z",
  "location": "Dow's Lake",
  "AvgIceThickness": 26.5,
  "MaxSnowAccumulation": 10
}
```

---

### 4. Azure Blob Storage

The processed data is stored in **Azure Blob Storage** for further analysis. 

#### Storage Structure
The data is organized by location and timestamp:

```
blob-storage/
  Dow's_Lake/
    2024-11-23_1200.json
  Fifth_Avenue/
    2024-11-23_1200.json
  NAC/
    2024-11-23_1200.json
```

#### File Format
The processed data is stored in **JSON format** for compatibility with analytics tools.

---

## Results

### Key Findings
- Aggregated ice thickness and snow accumulation data are available for each location in 5-minute intervals.
- Example output files can be found in the `screenshots/` directory.

---

## Usage Instructions

### Step-by-Step Guide

1. **Run the IoT Sensor Simulation**:
   - Navigate to `sensor-simulation/` and run the simulation script.

2. **Set Up Azure Services**:
   - Follow the `Azure-Setup.md` guide for configuring IoT Hub, Stream Analytics, and Blob Storage.

3. **Access Processed Data**:
   - Log in to the Azure Portal.
   - Navigate to the Blob Storage account and locate the data under the appropriate location and timestamp.

---

## Reflection

### Challenges Faced

1. **Data Synchronization**:
   - Ensuring consistent timestamps across simulated sensors was complex.
   - Solution: Used Python’s `time.sleep` to maintain timing intervals.

2. **IoT Hub Message Routing**:
   - Initial routing misconfigurations caused delays in data delivery.
   - Solution: Updated IoT Hub endpoints and verified routing rules.

3. **Query Optimization**:
   - Ensuring efficient query execution in Azure Stream Analytics required iterative testing.
   - Solution: Simplified SQL queries and used proper indexing.

### Lessons Learned
- Real-time systems require careful synchronization between simulation and processing components.
- Azure provides robust tools for IoT and real-time data streaming but requires thorough configuration.

---

## Repository Structure

```
Rideau-Canal-Monitoring/
├── README.md
├── sensor-simulation/
│   ├── sensor_simulation.py
│   ├── requirements.txt
├── screenshots/
│   ├── system-architecture-diagram.png
│   ├── iot-hub-configuration.png
│   ├── stream-analytics-settings.png
│   ├── blob-storage-output.png
├── Azure-Setup.md
└── output-data/
    ├── DowsLake/
    ├── FifthAvenue/
    └── NAC/
```

---

## Authors

- **Team Member 1**
- **Team Member 2**
- **Team Member 3**

---

## License

This project is licensed under the MIT License.
```

This expanded **README.md** includes more detailed instructions, configurations, challenges, and examples to make it comprehensive and user-friendly.
