# BICEPS Extensions Implementation Examples

## Complete Example: ICU Monitor with Extensions

This example shows how to implement various BICEPS extensions in a realistic ICU monitoring scenario.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<msg:GetMdibResponse 
    xmlns:msg="http://standards.ieee.org/downloads/11073/11073-10207-2017/message"
    xmlns:pm="http://standards.ieee.org/downloads/11073/11073-10207-2017/participant"
    xmlns:ext="http://standards.ieee.org/downloads/11073/11073-10207-2017/extension"
    xmlns:sdpi="urn:oid:1.3.6.1.4.1.19376.1.6.2.10.1.1.1"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    MdibVersion="200" 
    SequenceId="urn:uuid:device-sequence-12345"
    InstanceId="icu-monitor-001">
    
    <msg:Mdib MdibVersion="200" SequenceId="urn:uuid:device-sequence-12345">
        
        <!-- DESCRIPTIVE PART with Extensions -->
        <pm:MdDescription DescriptionVersion="5">
            <pm:Mds Handle="mds0" SafetyClassification="MedA">
                
                <!-- 1. EQUIPMENT IDENTIFIER EXTENSION (Mandatory) -->
                <ext:Extension>
                    <sdpi:EquipmentIdentifier>urn:uuid:9c057bb4-8d83-4fc1-9ad1-832ad543e2b2</sdpi:EquipmentIdentifier>
                </ext:Extension>
                
                <!-- 2. CODED ATTRIBUTES EXTENSION -->
                <ext:Extension>
                    <sdpi:CodedAttributes>
                        <!-- Equipment friendly name -->
                        <sdpi:CodedStringAttribute>
                            <sdpi:MdcAttribute Code="67886" SymbolicCodeName="MDC_ATTR_ID_SOFT"/>
                            <sdpi:Value>ICU-Monitor-Bed-5</sdpi:Value>
                        </sdpi:CodedStringAttribute>
                        
                        <!-- Firmware version -->
                        <sdpi:CodedStringAttribute>
                            <sdpi:MdcAttribute Code="531977" SymbolicCodeName="MDC_ID_PROD_SPEC_FW"/>
                            <sdpi:Value>v3.2.1-build.789</sdpi:Value>
                        </sdpi:CodedStringAttribute>
                        
                        <!-- Hardware revision -->
                        <sdpi:CodedStringAttribute>
                            <sdpi:MdcAttribute Code="531974" SymbolicCodeName="MDC_ID_PROD_SPEC_HW"/>
                            <sdpi:Value>hw-rev-2.1</sdpi:Value>
                        </sdpi:CodedStringAttribute>
                        
                        <!-- Installation date -->
                        <sdpi:CodedStringAttribute>
                            <sdpi:MdcAttribute Code="531971" SymbolicCodeName="MDC_ID_PROD_SPEC_UNSPECIFIED"/>
                            <sdpi:Value>2024-01-15</sdpi:Value>
                        </sdpi:CodedStringAttribute>
                    </sdpi:CodedAttributes>
                </ext:Extension>
                
                <pm:Type Code="69650">
                    <pm:ConceptDescription>Patient monitor</pm:ConceptDescription>
                </pm:Type>
                
                <pm:MetaData>
                    <pm:Manufacturer>ACME Medical Systems</pm:Manufacturer>
                    <pm:ModelName>VitalWatch Pro Plus</pm:ModelName>
                    <pm:ModelNumber>VWP-4000</pm:ModelNumber>
                    <pm:SerialNumber>VWP987654321</pm:SerialNumber>
                </pm:MetaData>
                
                <!-- 3. LOCALIZATION TEXT CATALOG -->
                <pm:ProductionSpecification>
                    <pm:SpecType Code="68008" SymbolicCodeName="MDC_ATTR_VMS_MDS_TEXT_CAT"/>
                    <pm:ProductionSpec>urn:oid:1.3.6.1.4.1.1234.4.1.2.1.3.1</pm:ProductionSpec>
                </pm:ProductionSpecification>
                
                <!-- 4. TIMESTAMP VERSIONING SUPPORT -->
                <pm:Clock Handle="system_clock">
                    <ext:Extension>
                        <sdpi:EpochSupport Version="1"/>
                    </ext:Extension>
                    <pm:Type Code="69968">
                        <pm:ConceptDescription>System clock</pm:ConceptDescription>
                    </pm:Type>
                    <pm:TimeProtocol>NTP</pm:TimeProtocol>
                </pm:Clock>
                
                <!-- System Context -->
                <pm:SystemContext Handle="system_context">
                    <pm:PatientContext Handle="patient_context"/>
                    <pm:LocationContext Handle="location_context"/>
                </pm:SystemContext>
                
                <!-- VMD with Equipment Identifier -->
                <pm:Vmd Handle="vital_signs_vmd" SafetyClassification="MedA">
                    <ext:Extension>
                        <sdpi:EquipmentIdentifier>urn:uuid:84051cdb-5353-47af-a916-b1f007e08ed8</sdpi:EquipmentIdentifier>
                    </ext:Extension>
                    
                    <pm:Type Code="69476">
                        <pm:ConceptDescription>Vital signs</pm:ConceptDescription>
                    </pm:Type>
                    
                    <!-- Blood Pressure Channel with Compound Metrics -->
                    <pm:Channel Handle="nibp_channel">
                        <pm:Type Code="150020">
                            <pm:ConceptDescription>Noninvasive blood pressure</pm:ConceptDescription>
                        </pm:Type>
                        
                        <!-- 5. COMPOUND METRIC MODELING -->
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
                </pm:Vmd>
            </pm:Mds>
        </pm:MdDescription>
        
        <!-- STATE PART with Extensions -->
        <pm:MdState StateVersion="200">
            
            <!-- 6. PATIENT CONTEXT with GENDER EXTENSION -->
            <pm:State xsi:type="pm:PatientContextState" 
                      Handle="patient_context_state" 
                      DescriptorHandle="patient_context" 
                      ContextAssociation="Assoc" 
                      StateVersion="12">
                <pm:Identification Root="1.2.3.4.5.hospital" Extension="PATIENT98765"/>
                <pm:CoreData>
                    <!-- Gender Extension -->
                    <ext:Extension>
                        <sdpi:Gender>Female</sdpi:Gender>
                    </ext:Extension>
                    
                    <pm:Givenname>Sarah</pm:Givenname>
                    <pm:Middlename>Jane</pm:Middlename>
                    <pm:Familyname>Johnson</pm:Familyname>
                    <pm:Sex>F</pm:Sex> <!-- Biological sex -->
                    <pm:Birthdate>1975-08-22</pm:Birthdate>
                    <pm:Height MeasuredValue="165" Unit="cm"/>
                    <pm:Weight MeasuredValue="68" Unit="kg"/>
                </pm:CoreData>
            </pm:State>
            
            <!-- 7. CLOCK STATE with TIMESTAMP VERSIONING -->
            <pm:State xsi:type="pm:ClockState" 
                      Handle="system_clock_state" 
                      DescriptorHandle="system_clock" 
                      StateVersion="8"
                      ActivationState="On">
                <pm:DateAndTime>1733324400000</pm:DateAndTime>
                <pm:LastSet>1733317200000</pm:LastSet>
                <pm:TimeZone>+0100</pm:TimeZone>
                
                <!-- Timestamp versioning extension -->
                <ext:Extension>
                    <sdpi:EpochInfo>
                        <sdpi:Epoch Version="3"/>
                        <sdpi:EpochOffsets>
                            <sdpi:EpochOffset FromVersion="1" ToVersion="2" Offset="PT-3600S"/>
                            <sdpi:EpochOffset FromVersion="2" ToVersion="3" Offset="PT300S"/>
                        </sdpi:EpochOffsets>
                    </sdpi:EpochInfo>
                </ext:Extension>
            </pm:State>
            
            <!-- 8. METRIC STATES with VERSIONED TIMESTAMPS -->
            <pm:State xsi:type="pm:NumericMetricState" 
                      Handle="bp_systolic_state" 
                      DescriptorHandle="bp_systolic" 
                      StateVersion="87" 
                      ActivationState="On">
                <pm:MetricValue Value="135" DeterminationTime="1733324350000">
                    <!-- Timestamp version extension -->
                    <ext:Extension>
                        <sdpi:Timestamp Version="3" Clock="system_clock"/>
                    </ext:Extension>
                    <pm:MetricQuality Validity="Vld" Qi="0.95"/>
                </pm:MetricValue>
                <pm:PhysiologicalRange Lower="90" Upper="140"/>
            </pm:State>
            
            <pm:State xsi:type="pm:NumericMetricState" 
                      Handle="bp_diastolic_state" 
                      DescriptorHandle="bp_diastolic" 
                      StateVersion="87" 
                      ActivationState="On">
                <pm:MetricValue Value="85" DeterminationTime="1733324350000">
                    <ext:Extension>
                        <sdpi:Timestamp Version="3" Clock="system_clock"/>
                    </ext:Extension>
                    <pm:MetricQuality Validity="Vld" Qi="0.95"/>
                </pm:MetricValue>
                <pm:PhysiologicalRange Lower="60" Upper="90"/>
            </pm:State>
            
            <pm:State xsi:type="pm:NumericMetricState" 
                      Handle="bp_mean_state" 
                      DescriptorHandle="bp_mean" 
                      StateVersion="87" 
                      ActivationState="On">
                <pm:MetricValue Value="102" DeterminationTime="1733324350000">
                    <ext:Extension>
                        <sdpi:Timestamp Version="3" Clock="system_clock"/>
                    </ext:Extension>
                    <pm:MetricQuality Validity="Vld" Qi="0.95"/>
                </pm:MetricValue>
                <pm:PhysiologicalRange Lower="70" Upper="105"/>
            </pm:State>
        </pm:MdState>
    </msg:Mdib>
</msg:GetMdibResponse>
```

## Extension Implementation Examples

### 1. Adding Custom Equipment Attributes

```xml
<!-- Example: Adding maintenance information -->
<pm:Mds Handle="mds0">
    <ext:Extension>
        <sdpi:CodedAttributes>
            <!-- Last calibration date -->
            <sdpi:CodedStringAttribute>
                <sdpi:MdcAttribute Code="531971" SymbolicCodeName="MDC_ID_PROD_SPEC_UNSPECIFIED"/>
                <sdpi:Value>2024-07-15T09:00:00Z</sdpi:Value>
            </sdpi:CodedStringAttribute>
            
            <!-- Service technician -->
            <sdpi:CodedStringAttribute>
                <sdpi:MdcAttribute Code="531971" SymbolicCodeName="MDC_ID_PROD_SPEC_UNSPECIFIED"/>
                <sdpi:Value>Tech-ID-12345</sdpi:Value>
            </sdpi:CodedStringAttribute>
            
            <!-- Operating hours -->
            <sdpi:CodedIntegerAttribute>
                <sdpi:MdcAttribute Code="531971" SymbolicCodeName="MDC_ID_PROD_SPEC_UNSPECIFIED"/>
                <sdpi:Value>15420</sdpi:Value>
            </sdpi:CodedIntegerAttribute>
        </sdpi:CodedAttributes>
    </ext:Extension>
</pm:Mds>
```

### 2. Gender Extension in Different Scenarios

```xml
<!-- Scenario 1: Transgender patient -->
<pm:CoreData>
    <ext:Extension>
        <sdpi:Gender>Female</sdpi:Gender>
    </ext:Extension>
    <pm:Givenname>Jessica</pm:Givenname>
    <pm:Familyname>Smith</pm:Familyname>
    <pm:Sex>M</pm:Sex> <!-- Biological sex assigned at birth -->
    <pm:Birthdate>1990-03-15</pm:Birthdate>
</pm:CoreData>

<!-- Scenario 2: Non-binary patient -->
<pm:CoreData>
    <ext:Extension>
        <sdpi:Gender>Other</sdpi:Gender>
    </ext:Extension>
    <pm:Givenname>Alex</pm:Givenname>
    <pm:Familyname>Johnson</pm:Familyname>
    <pm:Sex>F</pm:Sex> <!-- Biological sex -->
    <pm:Birthdate>1985-11-22</pm:Birthdate>
</pm:CoreData>

<!-- Scenario 3: Unknown gender -->
<pm:CoreData>
    <ext:Extension>
        <sdpi:Gender>Unknown</sdpi:Gender>
    </ext:Extension>
    <pm:Givenname>John</pm:Givenname>
    <pm:Familyname>Doe</pm:Familyname>
    <pm:Sex>M</pm:Sex>
    <pm:Birthdate>1970-01-01</pm:Birthdate>
</pm:CoreData>
```

### 3. Handling Time Zone Changes

```xml
<!-- Before daylight saving time change -->
<pm:State xsi:type="pm:ClockState" Handle="clock_state" StateVersion="45">
    <pm:DateAndTime>1733324400000</pm:DateAndTime>
    <pm:TimeZone>+0100</pm:TimeZone>
    <ext:Extension>
        <sdpi:EpochInfo>
            <sdpi:Epoch Version="5"/>
        </sdpi:EpochInfo>
    </ext:Extension>
</pm:State>

<!-- After daylight saving time change -->
<pm:State xsi:type="pm:ClockState" Handle="clock_state" StateVersion="46">
    <pm:DateAndTime>1733328000000</pm:DateAndTime>
    <pm:TimeZone>+0200</pm:TimeZone>
    <ext:Extension>
        <sdpi:EpochInfo>
            <sdpi:Epoch Version="6"/>
            <sdpi:EpochOffsets>
                <sdpi:EpochOffset FromVersion="5" ToVersion="6" Offset="PT3600S"/>
            </sdpi:EpochOffsets>
        </sdpi:EpochInfo>
    </ext:Extension>
</pm:State>
```

### 4. Multi-Component Vital Signs

```xml
<!-- Respiratory Rate with multiple components -->
<pm:Channel Handle="resp_channel">
    <!-- Respiratory Rate -->
    <pm:Metric xsi:type="pm:NumericMetricDescriptor" Handle="resp_rate">
        <pm:Type Code="151562">
            <pm:ConceptDescription>Respiratory rate</pm:ConceptDescription>
        </pm:Type>
        <pm:Unit Code="264928">
            <pm:ConceptDescription>breaths per minute</pm:ConceptDescription>
        </pm:Unit>
        <pm:Relation Kind="SST" Entries="resp_tidal_volume resp_minute_volume">
            <pm:Code Code="151562">
                <pm:ConceptDescription>Respiration</pm:ConceptDescription>
            </pm:Code>
        </pm:Relation>
    </pm:Metric>
    
    <!-- Tidal Volume -->
    <pm:Metric xsi:type="pm:NumericMetricDescriptor" Handle="resp_tidal_volume">
        <pm:Type Code="151708">
            <pm:ConceptDescription>Tidal volume</pm:ConceptDescription>
        </pm:Type>
        <pm:Unit Code="263872">
            <pm:ConceptDescription>milliliter</pm:ConceptDescription>
        </pm:Unit>
        <pm:Relation Kind="SST" Entries="resp_rate resp_minute_volume">
            <pm:Code Code="151562">
                <pm:ConceptDescription>Respiration</pm:ConceptDescription>
            </pm:Code>
        </pm:Relation>
    </pm:Metric>
    
    <!-- Minute Volume -->
    <pm:Metric xsi:type="pm:NumericMetricDescriptor" Handle="resp_minute_volume">
        <pm:Type Code="151980">
            <pm:ConceptDescription>Minute volume</pm:ConceptDescription>
        </pm:Type>
        <pm:Unit Code="263936">
            <pm:ConceptDescription>liter per minute</pm:ConceptDescription>
        </pm:Unit>
        <pm:Relation Kind="SST" Entries="resp_rate resp_tidal_volume">
            <pm:Code Code="151562">
                <pm:ConceptDescription>Respiration</pm:ConceptDescription>
            </pm:Code>
        </pm:Relation>
    </pm:Metric>
</pm:Channel>
```

### 5. Custom Extension Example

```xml
<!-- Custom extension for hospital-specific data -->
<pm:Mds Handle="mds0">
    <ext:Extension>
        <!-- Standard SDPi extension -->
        <sdpi:EquipmentIdentifier>urn:uuid:12345678-1234-1234-1234-123456789abc</sdpi:EquipmentIdentifier>
        
        <!-- Custom hospital extension -->
        <hospital:HospitalExtension 
            xmlns:hospital="http://myhospital.com/biceps/extensions" 
            ext:MustUnderstand="false">
            <hospital:AssetTag>ASSET-2024-001</hospital:AssetTag>
            <hospital:Department>ICU-CARDIAC</hospital:Department>
            <hospital:MaintenanceContract>CONTRACT-789</hospital:MaintenanceContract>
            <hospital:CostCenter>CC-12345</hospital:CostCenter>
        </hospital:HospitalExtension>
    </ext:Extension>
</pm:Mds>
```

## Validation and Testing

### 1. Schema Validation
```python
def validate_extensions(mdib_xml):
    """Validate MDIB with extensions against schemas"""
    import xmlschema
    
    # Load BICEPS schemas
    biceps_schema = xmlschema.XMLSchema('BICEPS_ParticipantModel.xsd')
    
    # Load SDPi extension schemas
    sdpi_schema = xmlschema.XMLSchema('SDPi_Extensions.xsd')
    
    # Validate
    try:
        biceps_schema.validate(mdib_xml)
        print("BICEPS validation passed")
    except xmlschema.XMLSchemaException as e:
        print(f"BICEPS validation failed: {e}")
    
    try:
        sdpi_schema.validate(mdib_xml)
        print("SDPi extension validation passed")
    except xmlschema.XMLSchemaException as e:
        print(f"SDPi extension validation failed: {e}")
```

### 2. Extension Processing
```python
def process_extensions(mdib_element):
    """Process extensions from MDIB element"""
    extensions = {}
    
    # Find extension elements
    ext_elements = mdib_element.findall('.//ext:Extension', namespaces={
        'ext': 'http://standards.ieee.org/downloads/11073/11073-10207-2017/extension',
        'sdpi': 'urn:oid:1.3.6.1.4.1.19376.1.6.2.10.1.1.1'
    })
    
    for ext_elem in ext_elements:
        # Process Equipment Identifier
        eq_id = ext_elem.find('sdpi:EquipmentIdentifier', namespaces)
        if eq_id is not None:
            extensions['equipment_id'] = eq_id.text
        
        # Process Coded Attributes
        coded_attrs = ext_elem.find('sdpi:CodedAttributes', namespaces)
        if coded_attrs is not None:
            extensions['coded_attributes'] = []
            for attr in coded_attrs.findall('sdpi:CodedStringAttribute', namespaces):
                code = attr.find('sdpi:MdcAttribute', namespaces).get('Code')
                value = attr.find('sdpi:Value', namespaces).text
                extensions['coded_attributes'].append({'code': code, 'value': value})
        
        # Process Gender
        gender = ext_elem.find('sdpi:Gender', namespaces)
        if gender is not None:
            extensions['gender'] = gender.text
    
    return extensions
```

This comprehensive example demonstrates how to implement all the major BICEPS extensions in a realistic medical device scenario, providing both the XML structure and implementation guidance.
