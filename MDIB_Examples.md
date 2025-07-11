# MDIB Examples and Practical Exercises

## Example 1: Simple Patient Monitor MDIB

This example shows a basic patient monitor with vital signs capabilities.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<msg:GetMdibResponse 
    xmlns:msg="http://standards.ieee.org/downloads/11073/11073-10207-2017/message"
    xmlns:pm="http://standards.ieee.org/downloads/11073/11073-10207-2017/participant"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    MdibVersion="100" 
    SequenceId="urn:uuid:12345678-1234-1234-1234-123456789abc"
    InstanceId="monitor001">
    
    <msg:Mdib MdibVersion="100" SequenceId="urn:uuid:12345678-1234-1234-1234-123456789abc">
        
        <!-- DESCRIPTIVE PART: Defines device capabilities -->
        <pm:MdDescription DescriptionVersion="1">
            <pm:Mds Handle="mds0" SafetyClassification="MedA">
                <pm:Type Code="69650">
                    <pm:ConceptDescription>Patient monitor</pm:ConceptDescription>
                </pm:Type>
                <pm:MetaData>
                    <pm:Manufacturer>ACME Medical</pm:Manufacturer>
                    <pm:ModelName>VitalWatch Pro</pm:ModelName>
                    <pm:ModelNumber>VW-3000</pm:ModelNumber>
                    <pm:SerialNumber>VW123456789</pm:SerialNumber>
                </pm:MetaData>
                
                <!-- Patient Context -->
                <pm:SystemContext Handle="system_context">
                    <pm:PatientContext Handle="patient_context"/>
                    <pm:LocationContext Handle="location_context"/>
                </pm:SystemContext>
                
                <!-- Virtual Medical Device for Vital Signs -->
                <pm:Vmd Handle="vital_signs_vmd" SafetyClassification="MedA">
                    <pm:Type Code="69476">
                        <pm:ConceptDescription>Vital signs</pm:ConceptDescription>
                    </pm:Type>
                    
                    <!-- ECG Channel -->
                    <pm:Channel Handle="ecg_channel">
                        <pm:Type Code="2112">
                            <pm:ConceptDescription>ECG</pm:ConceptDescription>
                        </pm:Type>
                        
                        <!-- Heart Rate Metric -->
                        <pm:Metric xsi:type="pm:NumericMetricDescriptor" 
                                   Handle="heart_rate" 
                                   SafetyClassification="MedA"
                                   MetricCategory="Msrmt"
                                   MetricAvailability="Intr"
                                   Resolution="1.0">
                            <pm:Type Code="147842">
                                <pm:ConceptDescription>Heart rate</pm:ConceptDescription>
                            </pm:Type>
                            <pm:Unit Code="264864">
                                <pm:ConceptDescription>beats per minute</pm:ConceptDescription>
                            </pm:Unit>
                            <pm:DeterminationPeriod>PT3S</pm:DeterminationPeriod>
                        </pm:Metric>
                        
                        <!-- ECG Waveform -->
                        <pm:Metric xsi:type="pm:RealTimeSampleArrayMetricDescriptor" 
                                   Handle="ecg_wave" 
                                   SafetyClassification="MedA"
                                   MetricCategory="Msrmt"
                                   MetricAvailability="Cont"
                                   Resolution="0.001"
                                   SamplePeriod="PT0.004S">
                            <pm:Type Code="131328">
                                <pm:ConceptDescription>ECG waveform</pm:ConceptDescription>
                            </pm:Type>
                            <pm:Unit Code="266016">
                                <pm:ConceptDescription>millivolt</pm:ConceptDescription>
                            </pm:Unit>
                        </pm:Metric>
                    </pm:Channel>
                    
                    <!-- Blood Pressure Channel -->
                    <pm:Channel Handle="nibp_channel">
                        <pm:Type Code="150020">
                            <pm:ConceptDescription>Noninvasive blood pressure</pm:ConceptDescription>
                        </pm:Type>
                        
                        <!-- Systolic BP -->
                        <pm:Metric xsi:type="pm:NumericMetricDescriptor" 
                                   Handle="bp_systolic" 
                                   SafetyClassification="MedA"
                                   MetricCategory="Msrmt"
                                   MetricAvailability="Intr"
                                   Resolution="1.0">
                            <pm:Type Code="150017">
                                <pm:ConceptDescription>Systolic blood pressure</pm:ConceptDescription>
                            </pm:Type>
                            <pm:Unit Code="266016">
                                <pm:ConceptDescription>mmHg</pm:ConceptDescription>
                            </pm:Unit>
                            <!-- Relation to other BP components -->
                            <pm:Relation Kind="SST" Entries="bp_diastolic bp_mean">
                                <pm:Code Code="150020">
                                    <pm:ConceptDescription>Noninvasive blood pressure</pm:ConceptDescription>
                                </pm:Code>
                            </pm:Relation>
                        </pm:Metric>
                        
                        <!-- Diastolic BP -->
                        <pm:Metric xsi:type="pm:NumericMetricDescriptor" 
                                   Handle="bp_diastolic" 
                                   SafetyClassification="MedA"
                                   MetricCategory="Msrmt"
                                   MetricAvailability="Intr"
                                   Resolution="1.0">
                            <pm:Type Code="150018">
                                <pm:ConceptDescription>Diastolic blood pressure</pm:ConceptDescription>
                            </pm:Type>
                            <pm:Unit Code="266016">
                                <pm:ConceptDescription>mmHg</pm:ConceptDescription>
                            </pm:Unit>
                            <pm:Relation Kind="SST" Entries="bp_systolic bp_mean">
                                <pm:Code Code="150020">
                                    <pm:ConceptDescription>Noninvasive blood pressure</pm:ConceptDescription>
                                </pm:Code>
                            </pm:Relation>
                        </pm:Metric>
                        
                        <!-- Mean BP -->
                        <pm:Metric xsi:type="pm:NumericMetricDescriptor" 
                                   Handle="bp_mean" 
                                   SafetyClassification="MedA"
                                   MetricCategory="Msrmt"
                                   MetricAvailability="Intr"
                                   Resolution="1.0">
                            <pm:Type Code="150023">
                                <pm:ConceptDescription>Mean blood pressure</pm:ConceptDescription>
                            </pm:Type>
                            <pm:Unit Code="266016">
                                <pm:ConceptDescription>mmHg</pm:ConceptDescription>
                            </pm:Unit>
                            <pm:Relation Kind="SST" Entries="bp_systolic bp_diastolic">
                                <pm:Code Code="150020">
                                    <pm:ConceptDescription>Noninvasive blood pressure</pm:ConceptDescription>
                                </pm:Code>
                            </pm:Relation>
                        </pm:Metric>
                    </pm:Channel>
                    
                    <!-- SpO2 Channel -->
                    <pm:Channel Handle="spo2_channel">
                        <pm:Type Code="150456">
                            <pm:ConceptDescription>Pulse oximetry</pm:ConceptDescription>
                        </pm:Type>
                        
                        <!-- SpO2 Metric -->
                        <pm:Metric xsi:type="pm:NumericMetricDescriptor" 
                                   Handle="spo2" 
                                   SafetyClassification="MedA"
                                   MetricCategory="Msrmt"
                                   MetricAvailability="Intr"
                                   Resolution="1.0">
                            <pm:Type Code="150456">
                                <pm:ConceptDescription>Oxygen saturation</pm:ConceptDescription>
                            </pm:Type>
                            <pm:Unit Code="262688">
                                <pm:ConceptDescription>percent</pm:ConceptDescription>
                            </pm:Unit>
                            <pm:DeterminationPeriod>PT1S</pm:DeterminationPeriod>
                        </pm:Metric>
                    </pm:Channel>
                </pm:Vmd>
                
                <!-- Alert System -->
                <pm:AlertSystem Handle="alert_system" SafetyClassification="MedA">
                    <pm:Type Code="69680">
                        <pm:ConceptDescription>Alarm system</pm:ConceptDescription>
                    </pm:Type>
                    
                    <!-- Heart Rate High Alert -->
                    <pm:AlertCondition Handle="hr_high_alert" 
                                      Kind="Tec" 
                                      Priority="Hi" 
                                      DefaultConditionGenerationDelay="PT5S">
                        <pm:Type Code="196616">
                            <pm:ConceptDescription>Heart rate high</pm:ConceptDescription>
                        </pm:Type>
                        <pm:Source>heart_rate</pm:Source>
                    </pm:AlertCondition>
                    
                    <!-- Heart Rate Low Alert -->
                    <pm:AlertCondition Handle="hr_low_alert" 
                                      Kind="Tec" 
                                      Priority="Hi" 
                                      DefaultConditionGenerationDelay="PT5S">
                        <pm:Type Code="196632">
                            <pm:ConceptDescription>Heart rate low</pm:ConceptDescription>
                        </pm:Type>
                        <pm:Source>heart_rate</pm:Source>
                    </pm:AlertCondition>
                </pm:AlertSystem>
            </pm:Mds>
        </pm:MdDescription>
        
        <!-- STATE PART: Current values and states -->
        <pm:MdState StateVersion="100">
            <!-- MDS State -->
            <pm:State xsi:type="pm:MdsState" 
                      Handle="mds0_state" 
                      DescriptorHandle="mds0" 
                      StateVersion="1">
                <pm:ActivationState>On</pm:ActivationState>
                <pm:OperatingHours>12345</pm:OperatingHours>
            </pm:State>
            
            <!-- Patient Context State -->
            <pm:State xsi:type="pm:PatientContextState" 
                      Handle="patient_context_state" 
                      DescriptorHandle="patient_context" 
                      ContextAssociation="Assoc" 
                      StateVersion="1">
                <pm:Identification Root="1.2.3.4.5" Extension="PATIENT12345"/>
                <pm:CoreData>
                    <pm:Givenname>John</pm:Givenname>
                    <pm:Familyname>Doe</pm:Familyname>
                    <pm:Sex>M</pm:Sex>
                    <pm:Birthdate>1980-05-15</pm:Birthdate>
                    <pm:Height MeasuredValue="175" Unit="cm"/>
                    <pm:Weight MeasuredValue="70" Unit="kg"/>
                </pm:CoreData>
            </pm:State>
            
            <!-- Location Context State -->
            <pm:State xsi:type="pm:LocationContextState" 
                      Handle="location_context_state" 
                      DescriptorHandle="location_context" 
                      ContextAssociation="Assoc" 
                      StateVersion="1">
                <pm:Identification Root="1.2.3.4.6" Extension="ICU_BED_5"/>
                <pm:LocationDetail>
                    <pm:PoC>ICU</pm:PoC>
                    <pm:Room>Room 205</pm:Room>
                    <pm:Bed>Bed 5</pm:Bed>
                </pm:LocationDetail>
            </pm:State>
            
            <!-- Heart Rate State -->
            <pm:State xsi:type="pm:NumericMetricState" 
                      Handle="heart_rate_state" 
                      DescriptorHandle="heart_rate" 
                      StateVersion="95" 
                      ActivationState="On">
                <pm:MetricValue Value="72" 
                               DeterminationTime="2024-01-15T10:30:00.000Z">
                    <pm:MetricQuality Validity="Vld" Qi="1.0"/>
                </pm:MetricValue>
                <pm:PhysiologicalRange Lower="60" Upper="100"/>
            </pm:State>
            
            <!-- Blood Pressure States -->
            <pm:State xsi:type="pm:NumericMetricState" 
                      Handle="bp_systolic_state" 
                      DescriptorHandle="bp_systolic" 
                      StateVersion="45" 
                      ActivationState="On">
                <pm:MetricValue Value="120" 
                               DeterminationTime="2024-01-15T10:28:30.000Z">
                    <pm:MetricQuality Validity="Vld" Qi="1.0"/>
                </pm:MetricValue>
                <pm:PhysiologicalRange Lower="90" Upper="140"/>
            </pm:State>
            
            <pm:State xsi:type="pm:NumericMetricState" 
                      Handle="bp_diastolic_state" 
                      DescriptorHandle="bp_diastolic" 
                      StateVersion="45" 
                      ActivationState="On">
                <pm:MetricValue Value="80" 
                               DeterminationTime="2024-01-15T10:28:30.000Z">
                    <pm:MetricQuality Validity="Vld" Qi="1.0"/>
                </pm:MetricValue>
                <pm:PhysiologicalRange Lower="60" Upper="90"/>
            </pm:State>
            
            <pm:State xsi:type="pm:NumericMetricState" 
                      Handle="bp_mean_state" 
                      DescriptorHandle="bp_mean" 
                      StateVersion="45" 
                      ActivationState="On">
                <pm:MetricValue Value="93" 
                               DeterminationTime="2024-01-15T10:28:30.000Z">
                    <pm:MetricQuality Validity="Vld" Qi="1.0"/>
                </pm:MetricValue>
                <pm:PhysiologicalRange Lower="70" Upper="105"/>
            </pm:State>
            
            <!-- SpO2 State -->
            <pm:State xsi:type="pm:NumericMetricState" 
                      Handle="spo2_state" 
                      DescriptorHandle="spo2" 
                      StateVersion="98" 
                      ActivationState="On">
                <pm:MetricValue Value="98" 
                               DeterminationTime="2024-01-15T10:30:00.000Z">
                    <pm:MetricQuality Validity="Vld" Qi="1.0"/>
                </pm:MetricValue>
                <pm:PhysiologicalRange Lower="95" Upper="100"/>
            </pm:State>
            
            <!-- Alert States -->
            <pm:State xsi:type="pm:AlertConditionState" 
                      Handle="hr_high_alert_state" 
                      DescriptorHandle="hr_high_alert" 
                      StateVersion="1" 
                      ActivationState="On">
                <pm:ActualConditionGenerationDelay>PT5S</pm:ActualConditionGenerationDelay>
                <pm:ActualPriority>Hi</pm:ActualPriority>
                <pm:Presence>false</pm:Presence>
                <pm:DeterminationTime>2024-01-15T10:30:00.000Z</pm:DeterminationTime>
            </pm:State>
            
            <pm:State xsi:type="pm:AlertConditionState" 
                      Handle="hr_low_alert_state" 
                      DescriptorHandle="hr_low_alert" 
                      StateVersion="1" 
                      ActivationState="On">
                <pm:ActualConditionGenerationDelay>PT5S</pm:ActualConditionGenerationDelay>
                <pm:ActualPriority>Hi</pm:ActualPriority>
                <pm:Presence>false</pm:Presence>
                <pm:DeterminationTime>2024-01-15T10:30:00.000Z</pm:DeterminationTime>
            </pm:State>
        </pm:MdState>
    </msg:Mdib>
</msg:GetMdibResponse>
```

## Key Points in This Example:

### 1. Device Hierarchy
- **MDS**: The main device (patient monitor)
- **VMD**: Virtual device for vital signs
- **Channels**: Grouped by clinical function (ECG, NIBP, SpO2)
- **Metrics**: Individual measurements

### 2. Compound Metrics
- Blood pressure components (systolic, diastolic, mean) are related through `pm:Relation` elements
- Each component references the others in the compound metric

### 3. Context Integration
- Patient information is bound to the device
- Location context shows ICU bed assignment
- Context association state indicates active binding

### 4. Safety and Quality
- Safety classification on critical components
- Metric quality indicators on measurements
- Physiological ranges for clinical validation

### 5. Real-Time Capabilities
- Different determination periods for different metrics
- Waveform descriptor for ECG signal
- Alert conditions for clinical alarms

## Practice Exercises:

### Exercise A: Parse the MDIB
1. Extract all metric handles and their current values
2. Identify the containment relationships
3. List all context information

### Exercise B: Simulate Updates
1. Create an EpisodicMetricReport with new heart rate value
2. Trigger an alert condition when HR > 100
3. Update patient context with new weight

### Exercise C: Add New Capabilities
1. Add a respiratory rate metric
2. Include a temperature measurement
3. Create alert conditions for new parameters

This example demonstrates a realistic patient monitoring scenario that you might encounter in an ICU setting.
