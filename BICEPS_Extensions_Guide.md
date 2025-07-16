# BICEPS Extensions Guide

## Overview

BICEPS extensions provide a standardized way to add information to the BICEPS Participant Model that doesn't fit into the predefined structure. The SDPi specification defines several key extensions that enhance interoperability.

## Extension Namespace

All SDPi extensions use the XML namespace:
```xml
xmlns:sdpi="urn:oid:1.3.6.1.4.1.19376.1.6.2.10.1.1.1"
```

## Extension Principles

### Priority Order for Adding Information:
1. **Use existing BICEPS model** (if adequate)
2. **Use standardized coded values** (IEEE 11073-10101)
3. **Use standardized extensions** (SDPi extensions)
4. **Use private coded values** (last resort)

### Extension Constraints:
- **No QName types**: Extensions cannot use XML Schema QName types due to interoperability issues
- **Must be well-formed**: Extensions must follow XML schema validation
- **Backward compatibility**: Extensions should not break existing implementations

---

## 1. Coded Attributes Extension

### Purpose
Allows addition of IEEE 11073-10101 attributes that don't exist in the standard BICEPS model.

### Use Case
Adding equipment labels, firmware versions, or other metadata not covered by standard BICEPS elements.

### Example - Equipment Label (Soft ID):
```xml
<pm:Mds Handle="mds0">
    <ext:Extension>
        <sdpi:CodedAttributes>
            <sdpi:CodedStringAttribute>
                <sdpi:MdcAttribute Code="67886" SymbolicCodeName="MDC_ATTR_ID_SOFT"/>
                <sdpi:Value>PatMon03</sdpi:Value>
            </sdpi:CodedStringAttribute>
        </sdpi:CodedAttributes>
    </ext:Extension>
    <!-- Regular MDS content -->
</pm:Mds>
```

### Available Attribute Types:
- **CodedStringAttribute**: String-based attributes
- **CodedIntegerAttribute**: Integer-based attributes  
- **CodedDecimalAttribute**: Decimal-based attributes

### Common MDC Codes for Production Specifications:
| Code | Description | Example Use |
|------|-------------|-------------|
| 67886 (MDC_ATTR_ID_SOFT) | Equipment label/friendly name | "PatMon03", "ICU_Vent_42" |
| 531970 (MDC_ID_MODEL_MANUFACTURER) | Manufacturer name | "ACME Medical Systems" |
| 531969 (MDC_ID_MODEL_NUMBER) | Model number | "VW-3000-Pro" |
| 531977 (MDC_ID_PROD_SPEC_FW) | Firmware version | "v2.1.3-build.456" |
| 531972 (MDC_ID_PROD_SPEC_SERIAL) | Serial number | "VW123456789" |

---

## 2. Gender Extension

### Purpose
Distinguishes between biological sex and administrative gender, following modern healthcare standards.

### Background
BICEPS only provides `pm:Sex` for biological sex. Modern healthcare requires both biological sex and administrative gender information.

### Example:
```xml
<pm:State xsi:type="pm:PatientContextState" 
          Handle="patient_context_state" 
          ContextAssociation="Assoc">
    <pm:Identification Root="hospital.oid" Extension="PATIENT123"/>
    <pm:CoreData>
        <ext:Extension>
            <sdpi:Gender>Other</sdpi:Gender>
        </ext:Extension>
        <pm:Givenname>Alex</pm:Givenname>
        <pm:Familyname>Smith</pm:Familyname>
        <pm:Sex>M</pm:Sex> <!-- Biological sex -->
        <pm:Birthdate>1990-03-15</pm:Birthdate>
    </pm:CoreData>
</pm:State>
```

### Valid Gender Values:
- **Male**: Administrative gender is male
- **Female**: Administrative gender is female
- **Other**: Administrative gender is other/non-binary
- **Unknown**: Administrative gender is unknown

### Requirement:
If administrative gender is available, it **shall** be provided using this extension.

---

## 3. Equipment Identifier Extension

### Purpose
Provides stable, globally unique identification for medical devices across reinitializations.

### Why Needed
- UDIs aren't unique across jurisdictions
- Handles are not stable across device restarts
- Need persistent device identification for tracking

### Example:
```xml
<pm:Mds Handle="mds0">
    <ext:Extension>
        <sdpi:EquipmentIdentifier>urn:uuid:9c057bb4-8d83-4fc1-9ad1-832ad543e2b2</sdpi:EquipmentIdentifier>
    </ext:Extension>
    <!-- Regular MDS content -->
</pm:Mds>

<pm:Vmd Handle="vmd0">
    <ext:Extension>
        <sdpi:EquipmentIdentifier>urn:uuid:84051cdb-5353-47af-a916-b1f007e08ed8</sdpi:EquipmentIdentifier>
    </ext:Extension>
    <!-- Regular VMD content -->
</pm:Vmd>
```

### Requirements:
- **Mandatory for MDS**: Every MDS must have an equipment identifier
- **Recommended for VMDs**: Especially for removable subsystems
- **URI format**: Must use UUID or OID in URI format
- **Stable**: Must remain constant across device reinitializations
- **Globally unique**: Must be unique across all devices

### Implementation Notes:
- Can be generated from UDI if globally unique
- Can use UUIDv5 with manufacturer namespace + serial number
- Must perform case-sensitive string comparison for equality

---

## 4. Timestamp Versioning Extension

### Purpose
Handles time adjustments in medical devices while maintaining data integrity.

### Problem Addressed
When device clocks are adjusted (NTP sync, daylight saving time), timestamps may become unreliable. This extension helps track epoch changes.

### Example:
```xml
<pm:Clock Handle="clk">
    <ext:Extension>
        <sdpi:EpochSupport Version="1"/>
    </ext:Extension>
</pm:Clock>
```

### In Clock State:
```xml
<pm:State xsi:type="pm:ClockState" 
          Handle="clk_state" 
          DescriptorHandle="clk" 
          ActivationState="StndBy">
    <pm:DateAndTime>1733324400000</pm:DateAndTime>
    <pm:LastSet>1733317200000</pm:LastSet>
    <ext:Extension>
        <sdpi:EpochInfo>
            <sdpi:Epoch Version="5"/>
            <sdpi:EpochOffsets>
                <sdpi:EpochOffset FromVersion="3" ToVersion="4" Offset="PT300S"/>
                <sdpi:EpochOffset FromVersion="4" ToVersion="5" Offset="PT-600S"/>
            </sdpi:EpochOffsets>
        </sdpi:EpochInfo>
    </ext:Extension>
</pm:State>
```

### Versioned Timestamps:
```xml
<pm:MetricValue Value="72" DeterminationTime="1733320800000">
    <ext:Extension>
        <sdpi:Timestamp Version="5"/>
    </ext:Extension>
    <pm:MetricQuality Validity="Vld" Qi="1.0"/>
</pm:MetricValue>
```

---

## 5. Compound Metric Modeling

### Purpose
Models compound metrics (like blood pressure with systolic/diastolic/mean components) using relations.

### Implementation
Uses existing BICEPS `pm:Relation` elements but with specific conventions:

```xml
<!-- Systolic Blood Pressure -->
<pm:Metric xsi:type="pm:NumericMetricDescriptor" Handle="bp_sys">
    <pm:Type Code="150017">
        <pm:ConceptDescription>Systolic Blood Pressure</pm:ConceptDescription>
    </pm:Type>
    <pm:Unit Code="266016">
        <pm:ConceptDescription>mmHg</pm:ConceptDescription>
    </pm:Unit>
    <pm:Relation Kind="SST" Entries="bp_dia bp_mean">
        <pm:Code Code="150020">
            <pm:ConceptDescription>Noninvasive blood pressure</pm:ConceptDescription>
        </pm:Code>
    </pm:Relation>
</pm:Metric>

<!-- Diastolic Blood Pressure -->
<pm:Metric xsi:type="pm:NumericMetricDescriptor" Handle="bp_dia">
    <pm:Type Code="150018">
        <pm:ConceptDescription>Diastolic Blood Pressure</pm:ConceptDescription>
    </pm:Type>
    <pm:Unit Code="266016">
        <pm:ConceptDescription>mmHg</pm:ConceptDescription>
    </pm:Unit>
    <pm:Relation Kind="SST" Entries="bp_sys bp_mean">
        <pm:Code Code="150020">
            <pm:ConceptDescription>Noninvasive blood pressure</pm:ConceptDescription>
        </pm:Code>
    </pm:Relation>
</pm:Metric>
```

### Requirements:
- **Relation Kind**: Must be "SST" (Spatial-Spatial-Temporal)
- **Same Code**: All components must use the same compound metric code
- **Cross-references**: Each component lists all other components in Entries

---

## 6. Localization Text Catalog

### Purpose
Provides unique identification for localized text catalogs to enable sharing across devices.

### Implementation
Uses production specification with specific MDC code:

```xml
<pm:Mds Handle="mds0">
    <pm:ProductionSpecification>
        <pm:SpecType Code="68008" SymbolicCodeName="MDC_ATTR_VMS_MDS_TEXT_CAT"/>
        <pm:ProductionSpec>urn:oid:1.3.6.1.4.1.1234.4.1.2.1.2.2</pm:ProductionSpec>
    </pm:ProductionSpecification>
</pm:Mds>
```

### Best Practices:
- **Unique per catalog**: Each text catalog version gets unique OID
- **Version tracking**: Include version in OID structure
- **Update on changes**: Change OID when text catalog changes

---

## Extension XML Schemas

### Schema Location
Extensions are defined in separate XML Schema files:

```xml
<!-- Schema imports -->
<xsd:import namespace="http://standards.ieee.org/downloads/11073/11073-10207-2017/extension"/>
<xsd:import namespace="urn:oid:1.3.6.1.4.1.19376.1.6.2.10.1.1.1"/>
```

### Schema Structure Example:
```xml
<xsd:element name="CodedAttributes" type="sdpi:CodedAttributesType"/>
<xsd:complexType name="CodedAttributesType">
    <xsd:sequence>
        <xsd:element ref="sdpi:CodedStringAttribute" minOccurs="0" maxOccurs="unbounded"/>
        <xsd:element ref="sdpi:CodedIntegerAttribute" minOccurs="0" maxOccurs="unbounded"/>
        <xsd:element ref="sdpi:CodedDecimalAttribute" minOccurs="0" maxOccurs="unbounded"/>
    </xsd:sequence>
    <xsd:attribute ref="ext:MustUnderstand" use="optional"/>
</xsd:complexType>
```

---

## Implementation Guidelines

### 1. When to Use Extensions
- **Standard BICEPS insufficient**: Information cannot be expressed in standard model
- **Standardized extension exists**: Use SDPi extensions before creating custom ones
- **Interoperability required**: Custom extensions may break interoperability

### 2. Extension Best Practices
- **Follow schema**: Always validate against extension schemas
- **Document usage**: Clearly document any custom extensions
- **Version compatibility**: Consider impact on different BICEPS versions
- **Testing**: Test with multiple implementations

### 3. Custom Extensions
If you must create custom extensions:
- Use your own XML namespace
- Provide complete schema documentation
- Consider proposing for standardization
- Mark with `ext:MustUnderstand` if critical

### Example Custom Extension:
```xml
<pm:Mds Handle="mds0">
    <ext:Extension>
        <custom:MyCustomExtension xmlns:custom="http://mycompany.com/biceps/extensions">
            <custom:SomeValue>Custom Data</custom:SomeValue>
        </custom:MyCustomExtension>
    </ext:Extension>
</pm:Mds>
```

---

## Summary

BICEPS extensions provide a powerful mechanism for enhancing the standard model while maintaining interoperability. The SDPi specification defines several key extensions that address real-world implementation needs:

1. **Coded Attributes**: Add IEEE 11073-10101 attributes
2. **Gender Extension**: Distinguish biological sex from administrative gender
3. **Equipment Identifier**: Provide stable device identification
4. **Timestamp Versioning**: Handle time adjustments gracefully
5. **Compound Metrics**: Model complex measurements
6. **Text Catalogs**: Enable localization sharing

By following these extension patterns and guidelines, you can extend BICEPS functionality while maintaining standards compliance and interoperability.
