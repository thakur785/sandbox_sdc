# Practical Exercises for BICEPS Learning

## Exercise 1: MDIB Structure Analysis

### Objective
Learn to parse and understand MDIB structure by analyzing real device configurations.

### Task
Given the MDIB example from `MDIB_Examples.md`, complete the following:

1. **Extract Device Information**
   - Device manufacturer, model, serial number
   - List all VMDs and their purposes
   - Identify all metrics and their types

2. **Map Containment Hierarchy**
   Create a tree structure showing:
   ```
   MDS: Patient Monitor
   ├── SystemContext
   │   ├── PatientContext → Patient: John Doe
   │   └── LocationContext → ICU Bed 5
   ├── VMD: Vital Signs
   │   ├── ECG Channel
   │   │   ├── Heart Rate (72 bpm)
   │   │   └── ECG Waveform
   │   ├── NIBP Channel
   │   │   ├── Systolic BP (120 mmHg)
   │   │   ├── Diastolic BP (80 mmHg)
   │   │   └── Mean BP (93 mmHg)
   │   └── SpO2 Channel
   │       └── SpO2 (98%)
   └── AlertSystem
       ├── HR High Alert (inactive)
       └── HR Low Alert (inactive)
   ```

3. **Analyze Data Quality**
   - Check all metric validity indicators
   - Identify measurement timestamps
   - Verify physiological ranges

### Expected Output
```json
{
  "device": {
    "manufacturer": "ACME Medical",
    "model": "VitalWatch Pro",
    "serial": "VW123456789",
    "mdibVersion": "100"
  },
  "patient": {
    "id": "PATIENT12345",
    "name": "John Doe",
    "gender": "M",
    "birthdate": "1980-05-15"
  },
  "location": {
    "poc": "ICU",
    "room": "Room 205",
    "bed": "Bed 5"
  },
  "metrics": [
    {
      "handle": "heart_rate",
      "type": "Heart rate",
      "value": 72,
      "unit": "bpm",
      "timestamp": "2024-01-15T10:30:00.000Z",
      "quality": "Valid"
    }
    // ... other metrics
  ]
}
```

---

## Exercise 2: Creating Episodic Reports

### Objective
Practice generating state update messages for various scenarios.

### Scenario A: Vital Signs Change
Patient's heart rate increases from 72 to 95 bpm due to physical activity.

**Your Task**: Create an EpisodicMetricReport message.

**Template**:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<msg:EpisodicMetricReport 
    MdibVersion="[NEXT_VERSION]" 
    SequenceId="urn:uuid:12345678-1234-1234-1234-123456789abc" 
    InstanceId="monitor001">
    <msg:ReportPart>
        <!-- YOUR IMPLEMENTATION HERE -->
    </msg:ReportPart>
</msg:EpisodicMetricReport>
```

### Scenario B: Alert Activation
Heart rate exceeds 100 bpm threshold, triggering high heart rate alert.

**Your Task**: Create both:
1. EpisodicMetricReport for the new HR value
2. EpisodicAlertReport for the alert activation

### Scenario C: Patient Context Change
New patient "Jane Smith" (ID: PATIENT67890) is assigned to the bed.

**Your Task**: Create an EpisodicContextReport message.

---

## Exercise 3: Subscription Management

### Objective
Implement a complete subscription lifecycle.

### Part A: Subscribe to Heart Rate Updates
Create a subscription request that:
- Subscribes to heart rate metric changes only
- Sets expiration to 1 hour
- Specifies your consumer endpoint

### Part B: Handle Subscription Response
Process the subscription response and extract:
- Subscription manager endpoint
- Subscription ID
- Actual expiration time

### Part C: Renew Subscription
Before expiration, send a renewal request to extend for another hour.

### Part D: Process Notifications
Handle incoming notifications and:
- Validate message order (MdibVersion)
- Extract metric values
- Update local state cache

---

## Exercise 4: Network Discovery Simulation

### Objective
Simulate the complete device discovery process.

### Your Task
Implement both provider and consumer roles:

### Provider Side:
1. Send Hello message when device starts
2. Listen for Probe messages
3. Respond with ProbeMatch if criteria match
4. Handle GetMetadata requests

### Consumer Side:
1. Send Probe to discover ICU devices
2. Process ProbeMatch responses
3. Request metadata from discovered devices
4. Filter devices by capabilities

### Sample Criteria:
```xml
<!-- Look for devices in ICU with vital signs capability -->
<wsd:Types>dpws:Device</wsd:Types>
<wsd:Scopes>ldap:///ou=ICU,dc=hospital,dc=com</wsd:Scopes>
```

---

## Exercise 5: Error Handling and Edge Cases

### Objective
Handle real-world communication issues.

### Scenario A: Network Interruption
Subscription is lost due to network issues.

**Your Task**:
1. Detect the subscription failure
2. Implement automatic reconnection
3. Re-establish subscription
4. Handle any missed updates

### Scenario B: Invalid Message Received
Receive a malformed EpisodicMetricReport.

**Your Task**:
1. Validate message structure
2. Log the error appropriately
3. Continue processing other valid messages
4. Potentially request MDIB refresh

### Scenario C: Version Skew
Receive a report with unexpected MdibVersion.

**Your Task**:
1. Detect version inconsistency
2. Request current MDIB state
3. Synchronize local cache
4. Resume normal processing

---

## Exercise 6: Context-Aware Behavior

### Objective
Implement intelligent device behavior based on context.

### Scenario: Smart Alert Management
Your monitoring system should:

1. **Patient-Specific Thresholds**
   - Pediatric patients: Different HR ranges
   - Geriatric patients: Adjusted BP thresholds
   - Cardiac patients: More sensitive monitoring

2. **Location-Aware Settings**
   - ICU: Maximum sensitivity
   - Recovery ward: Reduced alert frequency
   - OR: Procedure-specific parameters

3. **Workflow Integration**
   - Pre-op: Basic monitoring
   - Intra-op: Continuous waveforms
   - Post-op: Recovery parameters

### Your Implementation:
```python
def adjust_parameters(patient_context, location_context, workflow_context):
    """
    Adjust monitoring parameters based on current context
    """
    # Your implementation here
    pass

def evaluate_alert_thresholds(metric_value, patient_age, location):
    """
    Determine if alert should be triggered based on context
    """
    # Your implementation here
    pass
```

---

## Exercise 7: Waveform Data Processing

### Objective
Handle real-time waveform data streaming.

### Setup
Create a RealTimeSampleArrayMetric for ECG data:
- Sample rate: 250 Hz (4ms intervals)
- Resolution: 0.001 mV
- Buffer size: 1000 samples (4 seconds)

### Your Tasks:

1. **Generate Waveform Stream**
   Create WaveformStream messages with simulated ECG data:
   ```xml
   <msg:WaveformStream MdibVersion="110" SequenceId="...">
       <msg:ReportPart>
           <pm:RealTimeSampleArrayMetricState Handle="ecg_wave_state">
               <pm:MetricValue DeterminationTime="2024-01-15T10:30:00.000Z">
                   <pm:Samples>0.5 0.7 0.9 0.8 0.6 ...</pm:Samples>
               </pm:MetricValue>
           </pm:RealTimeSampleArrayMetricState>
       </msg:ReportPart>
   </msg:WaveformStream>
   ```

2. **Process Streaming Data**
   - Buffer incoming samples
   - Detect R-peaks in ECG
   - Calculate heart rate from intervals
   - Trigger updates when rate changes

3. **Handle Data Loss**
   - Detect missing waveform packets
   - Implement gap detection
   - Mark data quality appropriately

---

## Exercise 8: Gateway Implementation

### Objective
Create a simple gateway between BICEPS and HL7 v2.

### Requirements
Convert BICEPS vital signs data to HL7 PCD-01 messages:

```
MSH|^~\&|BICEPS_GW|ICU|HIS|HOSPITAL|20240115103000||ORU^R01^ORU_R01|123456|P|2.6|||AL|NE|USA
PID|1||PATIENT12345^^^HOSPITAL^MR||DOE^JOHN^||19800515|M
PV1|1|I|ICU^205^5
OBR|1|||69650^Patient monitor^MDC
OBX|1|NM|147842^Heart rate^MDC||72|{/min}|60-100|N|||F
OBX|2|NM|150017^Systolic BP^MDC||120|{mmHg}|90-140|N|||F
OBX|3|NM|150018^Diastolic BP^MDC||80|{mmHg}|60-90|N|||F
```

### Your Tasks:
1. Parse BICEPS MDIB
2. Extract patient demographics
3. Map metrics to HL7 OBX segments
4. Generate compliant HL7 messages
5. Handle updates via subscriptions

---

## Exercise 9: Performance Testing

### Objective
Test system performance under realistic loads.

### Test Scenarios:

1. **High-Frequency Updates**
   - 100 metrics updating every second
   - Measure message processing latency
   - Monitor memory usage

2. **Multiple Consumers**
   - 10 consumers subscribing to same provider
   - Verify all receive identical updates
   - Test subscription scalability

3. **Large MDIB Retrieval**
   - Device with 1000+ metrics
   - Measure GetMdib response time
   - Test XML parsing performance

4. **Network Stress**
   - Simulated packet loss
   - High latency connections
   - Connection drops and recovery

---

## Exercise 10: Security Implementation

### Objective
Implement security best practices.

### Requirements:
1. **TLS Configuration**
   - Enable TLS 1.3 for all communications
   - Validate certificates properly
   - Handle certificate updates

2. **Authentication**
   - Implement device authentication
   - Use proper credentials management
   - Handle authentication failures

3. **Message Integrity**
   - Validate message signatures
   - Detect replay attacks
   - Ensure message ordering

### Sample Security Policy:
```xml
<SecurityPolicy>
    <RequiredTLSVersion>1.3</RequiredTLSVersion>
    <CertificateValidation>strict</CertificateValidation>
    <MessageSigning>required</MessageSigning>
    <MaxMessageAge>PT30S</MaxMessageAge>
</SecurityPolicy>
```

---

## Evaluation Criteria

For each exercise, you'll be evaluated on:

1. **Correctness**: Does your solution work as specified?
2. **Standards Compliance**: Does it follow BICEPS specifications?
3. **Error Handling**: How well does it handle edge cases?
4. **Performance**: Is it efficient and scalable?
5. **Security**: Are security best practices followed?
6. **Documentation**: Is the code well-documented?

## Bonus Challenges

1. **Multi-Device Coordination**: Implement device ensemble behavior
2. **AI Integration**: Add intelligent monitoring algorithms
3. **Mobile Interface**: Create a tablet app for clinicians
4. **Database Integration**: Store historical data efficiently
5. **Cloud Connectivity**: Enable remote monitoring capabilities

These exercises progress from basic understanding to advanced implementation, giving you practical experience with all aspects of the BICEPS standard.
