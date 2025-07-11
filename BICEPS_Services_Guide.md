# BICEPS Services and Transactions Guide

## Overview of BICEPS Services

BICEPS defines several core services that enable medical device communication:

1. **GET SERVICE** (Mandatory) - Retrieve device information
2. **SET SERVICE** - Control device parameters  
3. **DESCRIPTION EVENT SERVICE** - Subscribe to descriptor changes
4. **STATE EVENT SERVICE** - Subscribe to state changes
5. **CONTEXT SERVICE** - Manage patient/location context
6. **WAVEFORM SERVICE** - Handle real-time waveform data

---

## 1. GET SERVICE - Retrieving Device Information

### Transaction: GetMdib [DEV-30]

This is the fundamental transaction to retrieve the complete MDIB from a provider.

#### Request Message:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<s12:Envelope
    xmlns:s12="http://www.w3.org/2003/05/soap-envelope"
    xmlns:msg="http://standards.ieee.org/downloads/11073/11073-10207-2017/message"
    xmlns:wsa="http://www.w3.org/2005/08/addressing">
    <s12:Header>
        <wsa:Action>http://standards.ieee.org/downloads/11073/11073-20701-2018/GetService/GetMdib</wsa:Action>
        <wsa:MessageID>urn:uuid:12345678-90ab-cdef-1234-567890abcdef</wsa:MessageID>
        <wsa:To>http://device.hospital.com:8080/GetService</wsa:To>
    </s12:Header>
    <s12:Body>
        <msg:GetMdib/>
    </s12:Body>
</s12:Envelope>
```

#### Response Message Structure:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<s12:Envelope
    xmlns:s12="http://www.w3.org/2003/05/soap-envelope"
    xmlns:msg="http://standards.ieee.org/downloads/11073/11073-10207-2017/message"
    xmlns:pm="http://standards.ieee.org/downloads/11073/11073-10207-2017/participant"
    xmlns:wsa="http://www.w3.org/2005/08/addressing">
    <s12:Header>
        <wsa:Action>http://standards.ieee.org/downloads/11073/11073-20701-2018/GetService/GetMdibResponse</wsa:Action>
        <wsa:MessageID>urn:uuid:87654321-ba09-fedc-4321-ba0987654321</wsa:MessageID>
        <wsa:RelatesTo>urn:uuid:12345678-90ab-cdef-1234-567890abcdef</wsa:RelatesTo>
    </s12:Header>
    <s12:Body>
        <msg:GetMdibResponse MdibVersion="156" SequenceId="urn:uuid:device-seq-123" InstanceId="device001">
            <msg:Mdib MdibVersion="156" SequenceId="urn:uuid:device-seq-123" InstanceId="device001">
                <pm:MdDescription DescriptionVersion="10">
                    <!-- Device descriptors -->
                </pm:MdDescription>
                <pm:MdState StateVersion="156">
                    <!-- Current device states -->
                </pm:MdState>
            </msg:Mdib>
        </msg:GetMdibResponse>
    </s12:Body>
</s12:Envelope>
```

---

## 2. Discover BICEPS Services [DEV-25]

Before retrieving the MDIB, you need to discover what services a device offers.

### GetMetadata Transaction

#### Request:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<s12:Envelope
    xmlns:s12="http://www.w3.org/2003/05/soap-envelope"
    xmlns:wsa="http://www.w3.org/2005/08/addressing">
    <s12:Header>
        <wsa:Action>http://schemas.xmlsoap.org/ws/2004/09/transfer/Get</wsa:Action>
        <wsa:MessageID>urn:uuid:metadata-request-123</wsa:MessageID>
        <wsa:To>http://device.hospital.com:8080/</wsa:To>
    </s12:Header>
    <s12:Body/>
</s12:Envelope>
```

#### Response:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<s12:Envelope
    xmlns:s12="http://www.w3.org/2003/05/soap-envelope"
    xmlns:wsa="http://www.w3.org/2005/08/addressing"
    xmlns:wsm="http://schemas.xmlsoap.org/ws/2004/09/mex"
    xmlns:dpws="http://docs.oasis-open.org/ws-dd/ns/dpws/2009/01">
    <s12:Header>
        <wsa:Action>http://schemas.xmlsoap.org/ws/2004/09/transfer/GetResponse</wsa:Action>
        <wsa:MessageID>urn:uuid:metadata-response-456</wsa:MessageID>
        <wsa:RelatesTo>urn:uuid:metadata-request-123</wsa:RelatesTo>
    </s12:Header>
    <s12:Body>
        <wsm:Metadata>
            <!-- Device Model Information -->
            <wsm:MetadataSection Dialect="http://docs.oasis-open.org/ws-dd/ns/dpws/2009/01/ThisModel">
                <dpws:ThisModel>
                    <dpws:Manufacturer>ACME Medical</dpws:Manufacturer>
                    <dpws:ModelName>VitalWatch Pro</dpws:ModelName>
                    <dpws:ModelNumber>VW-3000</dpws:ModelNumber>
                    <dpws:ModelUrl>http://acme-medical.com/vitalwatch</dpws:ModelUrl>
                </dpws:ThisModel>
            </wsm:MetadataSection>
            
            <!-- Device Instance Information -->
            <wsm:MetadataSection Dialect="http://docs.oasis-open.org/ws-dd/ns/dpws/2009/01/ThisDevice">
                <dpws:ThisDevice>
                    <dpws:FriendlyName>ICU Monitor Bed 5</dpws:FriendlyName>
                    <dpws:FirmwareVersion>2.1.3</dpws:FirmwareVersion>
                    <dpws:SerialNumber>VW123456789</dpws:SerialNumber>
                </dpws:ThisDevice>
            </wsm:MetadataSection>
            
            <!-- Available Services -->
            <wsm:MetadataSection Dialect="http://docs.oasis-open.org/ws-dd/ns/dpws/2009/01/Relationship">
                <dpws:Relationship Type="http://docs.oasis-open.org/ws-dd/ns/dpws/2009/01/host">
                    <dpws:Host>
                        <wsa:EndpointReference>
                            <wsa:Address>http://device.hospital.com:8080/</wsa:Address>
                        </wsa:EndpointReference>
                        <dpws:Types>device:GetService device:StateEventService</dpws:Types>
                    </dpws:Host>
                    <dpws:Hosted>
                        <wsa:EndpointReference>
                            <wsa:Address>http://device.hospital.com:8080/GetService</wsa:Address>
                        </wsa:EndpointReference>
                        <dpws:Types>{http://standards.ieee.org/downloads/11073/11073-20701-2018}GetService</dpws:Types>
                    </dpws:Hosted>
                    <dpws:Hosted>
                        <wsa:EndpointReference>
                            <wsa:Address>http://device.hospital.com:8080/StateEventService</wsa:Address>
                        </wsa:EndpointReference>
                        <dpws:Types>{http://standards.ieee.org/downloads/11073/11073-20701-2018}StateEventService</dpws:Types>
                    </dpws:Hosted>
                </dpws:Relationship>
            </wsm:MetadataSection>
        </wsm:Metadata>
    </s12:Body>
</s12:Envelope>
```

---

## 3. Manage BICEPS Subscription [DEV-27]

To receive real-time updates, you need to establish subscriptions.

### Subscribe Request:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<s12:Envelope
    xmlns:s12="http://www.w3.org/2003/05/soap-envelope"
    xmlns:wsa="http://www.w3.org/2005/08/addressing"
    xmlns:wse="http://schemas.xmlsoap.org/ws/2004/08/eventing">
    <s12:Header>
        <wsa:Action>http://schemas.xmlsoap.org/ws/2004/08/eventing/Subscribe</wsa:Action>
        <wsa:MessageID>urn:uuid:subscribe-request-789</wsa:MessageID>
        <wsa:To>http://device.hospital.com:8080/StateEventService</wsa:To>
    </s12:Header>
    <s12:Body>
        <wse:Subscribe>
            <wse:EndTo>
                <wsa:Address>http://consumer.hospital.com:9090/notifications</wsa:Address>
            </wse:EndTo>
            <wse:Delivery Mode="http://schemas.xmlsoap.org/ws/2004/08/eventing/DeliveryModes/Push">
                <wse:NotifyTo>
                    <wsa:Address>http://consumer.hospital.com:9090/notifications</wsa:Address>
                </wse:NotifyTo>
            </wse:Delivery>
            <wse:Expires>PT3600S</wse:Expires>
            <wse:Filter Dialect="http://www.w3.org/TR/1999/REC-xpath-19991116/">
                //pm:MetricState[@DescriptorHandle='heart_rate']
            </wse:Filter>
        </wse:Subscribe>
    </s12:Body>
</s12:Envelope>
```

### Subscribe Response:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<s12:Envelope
    xmlns:s12="http://www.w3.org/2003/05/soap-envelope"
    xmlns:wsa="http://www.w3.org/2005/08/addressing"
    xmlns:wse="http://schemas.xmlsoap.org/ws/2004/08/eventing">
    <s12:Header>
        <wsa:Action>http://schemas.xmlsoap.org/ws/2004/08/eventing/SubscribeResponse</wsa:Action>
        <wsa:MessageID>urn:uuid:subscribe-response-abc</wsa:MessageID>
        <wsa:RelatesTo>urn:uuid:subscribe-request-789</wsa:RelatesTo>
    </s12:Header>
    <s12:Body>
        <wse:SubscribeResponse>
            <wse:SubscriptionManager>
                <wsa:Address>http://device.hospital.com:8080/subscriptions/sub123</wsa:Address>
                <wsa:ReferenceParameters>
                    <sub:SubscriptionId>sub123</sub:SubscriptionId>
                </wsa:ReferenceParameters>
            </wse:SubscriptionManager>
            <wse:Expires>2024-01-15T14:30:00Z</wse:Expires>
        </wse:SubscribeResponse>
    </s12:Body>
</s12:Envelope>
```

---

## 4. Episodic Reports - Receiving Updates

### EpisodicMetricReport:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<s12:Envelope
    xmlns:s12="http://www.w3.org/2003/05/soap-envelope"
    xmlns:wsa="http://www.w3.org/2005/08/addressing"
    xmlns:msg="http://standards.ieee.org/downloads/11073/11073-10207-2017/message"
    xmlns:pm="http://standards.ieee.org/downloads/11073/11073-10207-2017/participant"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <s12:Header>
        <wsa:Action>http://standards.ieee.org/downloads/11073/11073-20701-2018/StateEventService/EpisodicMetricReport</wsa:Action>
        <wsa:MessageID>urn:uuid:notification-def456</wsa:MessageID>
        <wsa:To>http://consumer.hospital.com:9090/notifications</wsa:To>
    </s12:Header>
    <s12:Body>
        <msg:EpisodicMetricReport MdibVersion="157" SequenceId="urn:uuid:device-seq-123" InstanceId="device001">
            <msg:ReportPart>
                <pm:MetricState xsi:type="pm:NumericMetricState" 
                               Handle="heart_rate_state" 
                               DescriptorHandle="heart_rate" 
                               StateVersion="96" 
                               ActivationState="On">
                    <pm:MetricValue Value="75" 
                                   DeterminationTime="2024-01-15T10:31:00.000Z">
                        <pm:MetricQuality Validity="Vld" Qi="1.0"/>
                    </pm:MetricValue>
                    <pm:PhysiologicalRange Lower="60" Upper="100"/>
                </pm:MetricState>
            </msg:ReportPart>
        </msg:EpisodicMetricReport>
    </s12:Body>
</s12:Envelope>
```

### EpisodicAlertReport (Alert Triggered):
```xml
<?xml version="1.0" encoding="UTF-8"?>
<s12:Envelope
    xmlns:s12="http://www.w3.org/2003/05/soap-envelope"
    xmlns:wsa="http://www.w3.org/2005/08/addressing"
    xmlns:msg="http://standards.ieee.org/downloads/11073/11073-10207-2017/message"
    xmlns:pm="http://standards.ieee.org/downloads/11073/11073-10207-2017/participant"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <s12:Header>
        <wsa:Action>http://standards.ieee.org/downloads/11073/11073-20701-2018/StateEventService/EpisodicAlertReport</wsa:Action>
        <wsa:MessageID>urn:uuid:alert-notification-123</wsa:MessageID>
        <wsa:To>http://consumer.hospital.com:9090/notifications</wsa:To>
    </s12:Header>
    <s12:Body>
        <msg:EpisodicAlertReport MdibVersion="158" SequenceId="urn:uuid:device-seq-123" InstanceId="device001">
            <msg:ReportPart>
                <pm:AlertState xsi:type="pm:AlertConditionState" 
                              Handle="hr_high_alert_state" 
                              DescriptorHandle="hr_high_alert" 
                              StateVersion="2" 
                              ActivationState="On">
                    <pm:ActualConditionGenerationDelay>PT5S</pm:ActualConditionGenerationDelay>
                    <pm:ActualPriority>Hi</pm:ActualPriority>
                    <pm:Presence>true</pm:Presence>
                    <pm:DeterminationTime>2024-01-15T10:32:00.000Z</pm:DeterminationTime>
                </pm:AlertState>
            </msg:ReportPart>
        </msg:EpisodicAlertReport>
    </s12:Body>
</s12:Envelope>
```

---

## 5. Network Discovery Process

### Step 1: Hello Message (Device Announces Presence)
```xml
<s12:Envelope
    xmlns:s12="http://www.w3.org/2003/05/soap-envelope"
    xmlns:wsa="http://www.w3.org/2005/08/addressing"
    xmlns:wsd="http://docs.oasis-open.org/ws-dd/ns/discovery/2009/01">
    <s12:Header>
        <wsa:Action>http://docs.oasis-open.org/ws-dd/ns/discovery/2009/01/Hello</wsa:Action>
        <wsa:MessageID>urn:uuid:hello-message-123</wsa:MessageID>
        <wsa:To>urn:docs-oasis-open-org:ws-dd:ns:discovery:2009:01</wsa:To>
    </s12:Header>
    <s12:Body>
        <wsd:Hello>
            <wsa:EndpointReference>
                <wsa:Address>http://device.hospital.com:8080/</wsa:Address>
            </wsa:EndpointReference>
            <wsd:Types>dpws:Device</wsd:Types>
            <wsd:Scopes>ldap:///ou=ICU,dc=hospital,dc=com</wsd:Scopes>
            <wsd:XAddrs>http://device.hospital.com:8080/</wsd:XAddrs>
            <wsd:MetadataVersion>1</wsd:MetadataVersion>
        </wsd:Hello>
    </s12:Body>
</s12:Envelope>
```

### Step 2: Probe Message (Consumer Searches for Devices)
```xml
<s12:Envelope
    xmlns:s12="http://www.w3.org/2003/05/soap-envelope"
    xmlns:wsa="http://www.w3.org/2005/08/addressing"
    xmlns:wsd="http://docs.oasis-open.org/ws-dd/ns/discovery/2009/01">
    <s12:Header>
        <wsa:Action>http://docs.oasis-open.org/ws-dd/ns/discovery/2009/01/Probe</wsa:Action>
        <wsa:MessageID>urn:uuid:probe-message-456</wsa:MessageID>
        <wsa:To>urn:docs-oasis-open-org:ws-dd:ns:discovery:2009:01</wsa:To>
    </s12:Header>
    <s12:Body>
        <wsd:Probe>
            <wsd:Types>dpws:Device</wsd:Types>
            <wsd:Scopes>ldap:///ou=ICU,dc=hospital,dc=com</wsd:Scopes>
        </wsd:Probe>
    </s12:Body>
</s12:Envelope>
```

### Step 3: ProbeMatch Response
```xml
<s12:Envelope
    xmlns:s12="http://www.w3.org/2003/05/soap-envelope"
    xmlns:wsa="http://www.w3.org/2005/08/addressing"
    xmlns:wsd="http://docs.oasis-open.org/ws-dd/ns/discovery/2009/01">
    <s12:Header>
        <wsa:Action>http://docs.oasis-open.org/ws-dd/ns/discovery/2009:01/ProbeMatches</wsa:Action>
        <wsa:MessageID>urn:uuid:probematch-message-789</wsa:MessageID>
        <wsa:RelatesTo>urn:uuid:probe-message-456</wsa:RelatesTo>
        <wsa:To>http://consumer.hospital.com:9090/</wsa:To>
    </s12:Header>
    <s12:Body>
        <wsd:ProbeMatches>
            <wsd:ProbeMatch>
                <wsa:EndpointReference>
                    <wsa:Address>http://device.hospital.com:8080/</wsa:Address>
                </wsa:EndpointReference>
                <wsd:Types>dpws:Device</wsd:Types>
                <wsd:Scopes>ldap:///ou=ICU,dc=hospital,dc=com</wsd:Scopes>
                <wsd:XAddrs>http://device.hospital.com:8080/</wsd:XAddrs>
                <wsd:MetadataVersion>1</wsd:MetadataVersion>
            </wsd:ProbeMatch>
        </wsd:ProbeMatches>
    </s12:Body>
</s12:Envelope>
```

---

## Typical Communication Flow

```
1. Device boots up → Sends Hello message
2. Consumer discovers devices → Sends Probe message
3. Device responds → Sends ProbeMatch message
4. Consumer requests metadata → Sends GetMetadata message
5. Device provides service info → Sends GetMetadataResponse
6. Consumer retrieves MDIB → Sends GetMdib message
7. Device provides current state → Sends GetMdibResponse
8. Consumer subscribes for updates → Sends Subscribe message
9. Device confirms subscription → Sends SubscribeResponse
10. Device sends real-time updates → Sends EpisodicReports
```

## Key Implementation Notes:

1. **Message Ordering**: Reports must be sent in ascending MdibVersion order
2. **Versioning**: Each state change increments the StateVersion
3. **Quality Indicators**: Always include validity and quality information
4. **Error Handling**: Implement proper fault responses for all services
5. **Security**: Use TLS for all communications in production environments

This guide provides the foundation for implementing BICEPS service communication in your SDC system.
