# Title: Integration Adapter Architecture and Design Description

# Project\Product: Annalise Enterprise

# Document Number: OPT-SW-027

# Version: 3

**DO NOT EDIT THIS PAGE OR THE HEADERS.**

**To update this page press Ctrl+A then F9 to update the document
fields. (this red section will not appear when printed)**

**If this is the OPT-XXX-###– copy this document and rename it to match
the document you are updating.**

**New Title**

**Project Name**

**OPT-XXX-###**

**0**

# Table of Contents

[1 Purpose [4](#purpose)](#purpose)

[2 Scope [4](#scope)](#scope)

[3 References [4](#references)](#references)

[4 Definitions [4](#definitions)](#definitions)

[5 Architecture and Design
[5](#architecture-and-design)](#architecture-and-design)

[5.1 Requirements [5](#requirements)](#requirements)

[5.2 Constraints [5](#constraints)](#constraints)

[5.3 Overview [6](#overview)](#overview)

[5.3.1 The CXR workflow [6](#the-cxr-workflow)](#the-cxr-workflow)

[5.3.2 The CTB workflow [6](#the-ctb-workflow)](#the-ctb-workflow)

[5.3.3 The Triage workflow
[6](#the-triage-workflow)](#the-triage-workflow)

[5.3.4 Configurations [7](#configurations)](#configurations)

[5.4 Architecture [8](#architecture)](#architecture)

[5.5 Tech Stack [10](#tech-stack)](#tech-stack)

[5.5.1 Programming Language
[10](#programming-language)](#programming-language)

[5.5.2 Message Broker [10](#message-broker)](#message-broker)

[5.5.3 Data Storage [10](#data-storage)](#data-storage)

[5.5.4 File Storage [10](#file-storage)](#file-storage)

[5.5.5 Package and Deployment
[10](#package-and-deployment)](#package-and-deployment)

[5.6 Detailed Design [11](#detailed-design)](#detailed-design)

[5.6.1 Micro Services [11](#micro-services)](#micro-services)

[5.6.2 Message Broker [11](#message-broker-1)](#message-broker-1)

[5.6.3 Data Storage [14](#data-storage-1)](#data-storage-1)

[5.6.4 File Storage [14](#file-storage-1)](#file-storage-1)

[5.6.5 Orchestration [14](#orchestration)](#orchestration)

[5.6.6 Installation [14](#installation)](#installation)

[5.6.7 Data Security and Integrity
[15](#data-security-and-integrity)](#data-security-and-integrity)

[6 Secrets [15](#secrets)](#secrets)

[6.1.1 Logs [15](#logs)](#logs)

[6.1.2 Metrics [15](#metrics)](#metrics)

[6.1.3 Components [15](#components)](#components)

[6.1.4 Services [17](#services)](#services)

**Revision History**

<table>
<colgroup>
<col style="width: 11%" />
<col style="width: 21%" />
<col style="width: 39%" />
<col style="width: 28%" />
</colgroup>
<thead>
<tr class="header">
<th><blockquote>
<p><strong>Version</strong> </p>
</blockquote></th>
<th><blockquote>
<p><strong>Date</strong> </p>
</blockquote></th>
<th><blockquote>
<p><strong>Description of change</strong> </p>
</blockquote></th>
<th><blockquote>
<p><strong>Author \ Updated by</strong> </p>
</blockquote></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><blockquote>
<p>0</p>
</blockquote></td>
<td><blockquote>
<p>May-2021</p>
</blockquote></td>
<td><blockquote>
<p>Initial Version</p>
</blockquote></td>
<td><blockquote>
<p>Hao Dong</p>
</blockquote></td>
</tr>
<tr class="even">
<td>1</td>
<td>Sept-2021</td>
<td>Added section for JSON Payload Triage feature</td>
<td>Arthur Oviedo</td>
</tr>
<tr class="odd">
<td>2</td>
<td>Mar-2022</td>
<td><p>Refactored document to add the workflow/pipeline concept</p>
<p>Added details of the CTB pipeline</p></td>
<td>Hao Dong</td>
</tr>
<tr class="even">
<td>3</td>
<td>Jul-2022</td>
<td>Updated sections for Triage workflow and pipeline</td>
<td>Arthur Oviedo</td>
</tr>
</tbody>
</table>

#  

# Purpose

The purpose of this document is to capture and define the key
architecture design elements of the Annalise Enterprise Integration
Adapter. As a reference document, it addresses the key design decisions
and captures the detailed design.

# Scope

The architecture design description in this document is constrained to
the following scope:

- The Integration Adapter components of the Annalise Enterprise product.

- Infrastructure for the Integration Adapter.

# References

Input Documents

1.  OPT-SYS-038 - Annalise Enterprise Integration Adapter Software
    Requirements Specification

2.  OPT-SW-024 - Annalise Integration Adapter API Specification

3.  OPT-SW-007 – DICOM and HL7 Specification

# Definitions

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 74%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>Term/Acronym</strong></th>
<th><strong>Definition</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Integration Adapter</td>
<td><p>Integration Adapter, Integration Services, Integration Layer or
Loki is used interchangeably in Optimus project.</p>
<p>Integration Adapter is responsible for acquiring data from the Image
Manager (for example a PACS), and Order Manager (for example RIS)
systems and sending the data to the Backend Service, also for accepting
result data from Backend Services and sending data into PACS or RIS
systems</p></td>
</tr>
<tr class="even">
<td>Optimus / Annalise Enterprise</td>
<td>Optimus is the project name for the <a
href="http://annalise.ai/">aAnnalise.ai</a> product that uses machine
learning to assist users in the clinical assessment of chest X-ray,
non-contrast CT brain images. Annalise Enterprise is the product
name</td>
</tr>
<tr class="odd">
<td>Optimus Backend / Annalise Enterprise Backend / Backend Services /
the Backend</td>
<td>Optimus Backend, Annalise Enterprise Backend, Backend Services or
the Backend is used interchangeably in Optimus project refers to the
backend system to accept and save DICOM images and meta data, do AI
processing, push result back to the Integration Adapter, support
pushing, querying or retrieving the results by Integration Adapter or
other sub-system of the Annalise Enterprise</td>
</tr>
<tr class="even">
<td>CXR</td>
<td>Chest X-Rays</td>
</tr>
<tr class="odd">
<td>CTB</td>
<td>CT Brain</td>
</tr>
<tr class="even">
<td>Triage</td>
<td>The function of updating priority of order in RIS system based on
the AI findings predicted by Annalise Enterprise</td>
</tr>
<tr class="odd">
<td>Attributes</td>
<td>Attributes is a trained AI model to predict whether a CTB series is
a primary series or not. This is based a score predicted by the
model</td>
</tr>
<tr class="even">
<td>Registration</td>
<td>Registration is a service to register primary CTB series. Register a
series is the process to normalise the images of a series. A registered
archive including the normalised pixel data of all images, and meta data
will be generated if a primary series is registered successfully</td>
</tr>
<tr class="odd">
<td>DICOM</td>
<td>Digital Imaging and Communications in Medicine. DICOM is the
international standard to transmit, store, retrieve, print, process, and
display medical imaging information</td>
</tr>
<tr class="even">
<td>SOP</td>
<td>A Service-Object Pair (SOP) Class is defined by the union of an
Information Object Definition (IOD) and a DICOM Message Service Elements
(DIMSE)</td>
</tr>
<tr class="odd">
<td>IOD</td>
<td>An Information Object Definition (IOD) is an object-oriented
abstract data model used to specify information about Real-World
Objects. An IOD provides communicating Application Entities with a
common view of the information to be exchanged.</td>
</tr>
<tr class="even">
<td>DIMSE</td>
<td>The particular Application Service Element defined in the DICOM
Standard</td>
</tr>
<tr class="odd">
<td>HL7</td>
<td>Health Level 7. HL7 refers to a set of international standards for
transfer of clinical and administrative data between software
applications used by various healthcare providers</td>
</tr>
<tr class="even">
<td>Image Manager / PACS</td>
<td><p>The system that is responsible for acquiring and archiving
digital images from image acquisition hardware. For example, PACS
(Picture Archiving and Communication System).</p>
<p>Image Manager and PACS terminology is used interchangeably throughout
this project</p></td>
</tr>
<tr class="odd">
<td>Order Manager / RIS</td>
<td><p>The system responsible for managing the patient workflow of
procedure bookings and fulfilment of bookings. For example, RIS
(Radiology Information System).</p>
<p>Order Manager and RIS terminology is used interchangeably throughout
this project</p></td>
</tr>
<tr class="even">
<td>Study Complete</td>
<td>This is the order status in HL7 message that indicates that the
radiographer has QA and completed the study scan and is ready for
radiologist to view in work list</td>
</tr>
<tr class="odd">
<td>On-prem tools</td>
<td>On-prem tools is a toolset including provisioner, unlocker and
imagetool which are used to generate packages and install them on
customised OVA Linux images for the Integration Adapter and the on-prem
Backend</td>
</tr>
<tr class="even">
<td>Provisioner</td>
<td>Provisioner is tool to install the packages</td>
</tr>
<tr class="odd">
<td>OVA</td>
<td>Open Virtual Appliance, a virtual OS image format. Customised OVA
images are needed to include base services needed for the Integration
Adapter and the on-prem Backend</td>
</tr>
<tr class="even">
<td>TLS</td>
<td>Transport Layer Security, or TLS, is a widely adopted security
protocol designed to facilitate privacy and data security for
communications over the Internet</td>
</tr>
</tbody>
</table>

# Architecture and Design

This section describes the functionality, architecture, and design of
the Integration Adapter as well as the constraints on the design.

## Requirements

See reference OPT-SYS-038 - Annalise Enterprise Integration Adapter
Software Requirements Specification for input requirements.

## Constraints 

The Integration Adapter will be accepting images and data from the
customers' existing PACS and RIS systems. Therefore, the external
interface might need to be compliant to those systems standard
Application Programming Interfaces (APIs) such as DICOM and HL7 or for
some cases, their propriety APIs. The integration cannot cause a
disruption to customers' production services and needs to be configured
to operate within the customers' data centre.

## Overview

Based on the data the Integration Adapter is handling and the direction
the data is flowing, the functions of the Integration Adapter can be
categorised into workflows or pipelines stated below in section 5.3.1 to
5.3.3.

### The CXR workflow

Chest X-Ray images are pushed to the Integration Adapter from PACS. The
images then are uploaded to Annalise Enterprise Backend.

To trigger the AI processing at the Backend:

- HL7 messages can be accepted from RIS to indicate studies are
  completed. Based on information from the HL7 messages, Integration
  Adapter send trigger messages to the Backend.

- Or alternatively, Integration Adapter use a time-based trigger to send
  triggers to the Backend after no new images are coming for the studies
  for some time.

### The CTB workflow

CT brain images are pushed to the Integration Adapter from PACS. The
images are then being processed series by series along the pipeline:

- A time-based trigger is used to check when no new images are coming
  for each series for some time. It will trigger the series to be
  processed.

- Filters will be applied to check for patient age, contrast agent,
  number of images, slices thickness etc. based on rules.

- Series passed the filters are sent to an Attributes AI model to
  predict whether the series are primary series.

- Primary series are then sent to a Registration services to register.

- All results of each series are sent to the Backend in the end. The
  result can be:

  - The series did not pass filters

  - The series predicted not as primary series

  - The series failed to be predicted by the Attributes model

  - The score from Attributes model is too low

  - The series failed to be registered

  - The series is registered

### The Triage workflow

Once the AI processing is finished for a study, the backend will notify
the Integration Adapter the processing result. This includes all
findings and related information. The Integration Adapter will calculate
the priority of the order based the findings and their priorities. The
Integration Adapter can support two different approaches to send the
priority messages to RIS systems:

- The Integration Adapter supports to send HL7 messages either directly
  to RIS systems or to an integration engine. Patient data can be
  included in these HL7 messages, on top of the priority information.

- The Integration Adapter supports to send the messages to an
  integration engine as Open API calls. The integration engine can then
  send message to the RIS system either using standard APIs like HL7 or
  proprietary APIs of the RIS system. With this approach, additional
  information includes patient data, all related findings, highest
  priority finding etc can be sent to the integration engine.

### Configurations

The Integration Adapter allows for regional differences in
configuration, depending on the geographic location of the product
deployment. Note that not all configurations are regional related, some
are based on organisational preferences. The following items can be
configured in the Integration Adapter:

- Enabling or disabling CXR and CTB modules. This is achieved by
  enabling or disabling CXR, CTB SOP Classes in the Integration Adapter.

  - CXR

    - Computed Radiography Image Storage SOP Class (UID
      1.2.840.10008.5.1.4.1.1.1)

    - Digital X-Ray Image Storage For Presentation SOP Class (UID
      1.2.840.10008.5.1.4.1.1.1.1)

  - CTB

    - CT Image Storage SOP Class (UID 1.2.840.10008.5.1.4.1.1.2)

    - Enhanced CT Image Storage (UID 1.2.840.10008.5.1.4.1.1.2.1)

    - Legacy Converted Enhanced CT Image Storage (UID
      1.2.840.10008.5.1.4.1.1.2.2)

- Enabling or disabling worklist triage feature.

- Worklist triage injection can be either HL7 based using the RIS
  injector or based on a JSON external API interface. See reference
  OPT-SW-007 for details of the HL7 message and see reference OPT-SW-024
  for details of the JSON message.

- Enabling or disabling the following patient data to be included in the
  injection payload. Note that by default this is disabled.

  - Patient Id

  - Patient Name

  - Patient DOB

  - Patient Sex

- Enabling or disabling the following transfer syntaxes for SOP Classes:

| **Image format**                                   | **Transfer Syntax UID** | **Module** |
|----------------------------------------------------|-------------------------|------------|
| JPEG 2000 image (lossless)                         | 1.2.840.10008.1.2.4.90  | CTB/CXR    |
| JPEG 2000 image (lossy)                            | 1.2.840.10008.1.2.4.91  | CXR        |
| Raw uncompressed image (Implicit VR Endian)        | 1.2.840.10008.1.2       | CTB/CXR    |
| Raw uncompressed image (Explicit VR Little Endian) | 1.2.840.10008.1.2.1     | CTB/CXR    |
| Raw uncompressed image (Explicit VR Big Endian)    | 1.2.840.10008.1.2.2     | CTB/CXR    |

- Configuration of triggering for CXR. Either time based or RIS HL7
  based. For RIS, the message is based on what the organisation has
  defined in the HL7 order status. e.g. image verified

- Configuration of the following CTB study filtering:

  - Patient Age check threshold. This is the minimum age of the patient
    at the time of the study that the Integration Adapter will allow the
    study to be processed. The information is obtained from DICOM tags.

  - Minimum Number of Images threshold. This is the minimum number of
    images for the CTB series that the Integration Adapter will allow
    the series to be processed.

  - Maximum Slice Thickness threshold. This is the maximum slice
    thickness for the CTB series that the Integration Adapter will allow
    the series to be processed.

## Architecture

<img src="./media/image2.png"
style="width:6.66027in;height:3.52439in" />

Figure 1 - The CXR and Triage Pipeline

The Figure 1 above is a high-level depiction of services within the
Integration Adapter for CXR and Triage pipeline.

<img src="./media/image3.png" style="width:6.26806in;height:2.54861in"
alt="Diagram Description automatically generated" />

Figure 2 - The CTB pipeline

The Figure 2 above is a high-level depiction of services within the
Integration Adapter for CTB pipeline.

The arrows in the above diagrams indicate the flow of data. For detailed
diagrams, refer to the Detailed Design section (section 5.6).

A component based, message/data driven, containerised micro-service
architecture was selected as the design choice for Integration Adapter.

<img src="./media/image4.png"
style="width:6.26033in;height:3.15625in" />

Figure 3 - Component Bases, Message/Data Driven, Containerised
Micro-Services

Integration Adapter will be deployed into customer network to
communicate with customer systems like RIS and PACS systems using HL7,
DICOM or customised private APIs. Varying customers will have different
RIS/PACS systems. The integration Adapter was designed to be highly
flexible and configurable to adapt to different systems and different
integration pipelines.

The aim of this components based, message/data driven, containerised
micro-service architecture is to suit this flexibility requirement.
While each component has a single and dedicated function, multiple
components are assembled to form micro-services as containers which can
run independently and not rely on other micro-services to complete its
function. This way all the functions of the whole Integration Adapter is
highly decoupled as much as possible.

The advantage of components based micro-service design is that the
components can be assembled to form different micro-services. For
example, QueueProducer and CStoreServer are assembled as DICOM Receiver,
while QueueProducer and Hl7Server are assembled as HL7 Receiver.

<img src="./media/image5.png" style="width:6.2632in;height:2.45988in"
alt="_scroll_external/attachments/high-residence-drawio-f3735448023f6dab8b85de38108682b19ad4281c6c8b42d18dd1581750b2b275.png" />

Figure 4 - Component Based Micro-Services, CXR DICOM Accept Workflow and
HL7 Messages Based Trigger Workflow

The benefit of message/data driven micro-service design is to provide a
separation to the micro-services and add resilience to the entire
system. Since most of the micro-services do not talk to each other
directly and do not have any states, each micro-service can be
independently replaced or scaled as needed. This will mitigate single
points of failure since if any individual service fails, it can be
restarted automatically, without affecting any other micro-services.

As containerised micro-services, the services can be easily developed,
tested, and deployed in different environments without the need of any
changes to the code itself. The services can easily run on any
environments like laptop/desktop, or servers in data centres or in the
cloud.

## Tech Stack

### Programming Language

The components and micro-services of Integration Adapter are written in
Python 3 for many of the advantages of Python like fast development,
mature libs for all the functions needed like image processing, HL7 and
DICOM communication, AMQP message protocol, HTTP/HTTPS, Web Socket
support etc.

### Message Broker

Since Integration Adapter was design as a group of message/data driven
microservices, a message broker system is needed as a communication
channel between most of the micro-services.

The message broker system selected to use in the integration Adapter is
RabbitMQ.

RabbitMQ was chosen as the message broker because it is lightweight,
open source, high performance and with rich features. It is one of the
standards go-to message brokers available.

### Data Storage

Some of workflows of Integration Adaptor need data be saved in database
short-termly as cache or context. For example, the CxrTrigger service
needs the context of Study-Series-Images from the messages.

The data storage system selected to use in the integration Adapter is
PostgreSQL.

PostgreSQL is a powerful, open-source object-relational database system
with over 30 years of active development that has earned it a strong
reputation for reliability, feature robustness, and performance.

It’s the same database system used in Optimus Backend Services also.

### File Storage

Some of workflows of Integration Adaptor need source DICOM files or
result files generated be saved in file storage short-termly so they can
be accessed by other services. For example, the CTB pipeline need the
source DICOM files be saved so that the Attributes service and
Registration service can access all images of a series. Also, the
registered archives generated by Registration need be saved before been
sent to the Backend.

The file storage system selected to use in the integration Adapter is
MinIO.

MinIO offers high-performance, S3 compatible object storage. Native to
Kubernetes, MinIO is the only object storage suite available on every
public cloud, every Kubernetes distribution, the private cloud and the
edge.

### Package and Deployment

All micro-services are containerized and built as Docker images. K3s as
light-weight Kubernetes was chosen as the orchestration engine for
Integration Adapter. The reason for Docker is that it is now almost the
standard for containers. And the reason for K3s is that we can share the
same technology with Backend Services and its rich features comes with
Kubernetes.

## Detailed Design

### Micro Services

All services of Integration Adapter are designed as message/data driven
micro-services. All services are driven either by messages from customer
systems, the Backend, queue systems, or data in the database system.
Messages/data are processed by the services and then either be published
to queue system, data system, the Backend or customer systems.

All services of Integration Adapter are transparent and stateless. That
is, they do not hold any state related to any resources. This means that
services crash, restart, or be replaced, updated, or removed without
affecting the entire system and without causing any data loss.

### Message Broker

Because of the nature of role Integration Adapter is playing in the
Optimus system - as a sub-system to collect data from customer systems
and send to Optimus backend, or accepting data from Optimus backend and
send into customer systems, it needs to talk in different protocols at
the two sides and the load from one side and the response from the other
side might up and down and not match, so it is important to have a
message broker system sits in the middle to act as a cache or buffer.
The message broker is also acting as indirect communication mechanism
for the micro-services. This separates the micro-services so that they
do not rely on each other and make them all stateless.

RabbitMQ is the selected message broker of Integration Adapter.

Although all the micro-services are stateless, the message broker does
store state because at any given time it stores the messages that are
yet to be processed. It also needs store the dead lettered messages. It
is important to make sure the messages in the message broker are
protected.

#### Retry and Dead Letter

A continuously retry mechanism was designed to handle short term Backend
or customer systems unreachable or unavailable scenario. When
Integration Adapter cannot reach the Backend or customer systems, or
when the Backend is in maintenance mode, it can requeue the failed
messages at the head of queues and retry them continuously. The messages
will only be removed until the Backend or customer systems are reachable
or TTL of the messages or length of the queue reached. This can avoid
data loss when the Backend is taken down for upgrading.

A 1+n retry mechanism was designed to handle one-off or brief time
errors from the Backend or customer systems. For the Backend, a list of
error status codes is configured in Integration Adapter so when the
Backend returns these error codes, the 1+n retry mechanism will be
triggered. Here 1+n means one immediate retry and number of delayed
retries. While immediate retry will happen immediately, delayed retries
will happen after some delayed time. The delayed time for each delayed
retry will be i\*t where i is the sequence of the delayed retry for
example 1, 2, 3, t is the base delay time in seconds for example 10. The
default value for n is 3 and for t is 10. This means the first delayed
retry will delay 10 secs, the second will be 20 seconds and the third
will be 30 seconds (If no other messages ahead) etc. The immediate retry
was designed to recover from one-off issues for example a network
package lost. The delayed retries was designed to recover from brief
time issues for example backend is too busy for few seconds.

Below is a diagram explaining the 1+n retry mechanism (using CXR DICOM
workflow as example):

<img src="./media/image6.png" style="width:6.2632in;height:3.48792in"
alt="_scroll_external/attachments/1-plus-n-retry-mechanism-drawio-6d003cb092c4beb798145549181bdcd06340b88d60b95efd7ecba67ac7bc1d69.png" />

Figure 5 - The 1+n Retry Mechanism

#### Below table lists the retry behaviour for each queue.

<table>
<colgroup>
<col style="width: 9%" />
<col style="width: 13%" />
<col style="width: 22%" />
<col style="width: 18%" />
<col style="width: 18%" />
<col style="width: 16%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>Pipeline</strong></th>
<th><strong>Queue</strong></th>
<th><strong>Consumer Service</strong></th>
<th><strong>Continuous Retry</strong></th>
<th><strong>1+n Retry</strong></th>
<th><strong>No Retry</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td rowspan="2">CXR</td>
<td>study</td>
<td>DicomTransmitter</td>
<td>Cannot reach backend or backend returns 418</td>
<td><p>Backend returns 1+N retry status codes</p>
<p>Or internal errors during handling of messages</p></td>
<td>Backend returns other HTTP status codes from 400 to 600</td>
</tr>
<tr class="even">
<td>study-completion</td>
<td>StudyCompletion</td>
<td>Cannot reach backend or backend returns 418</td>
<td><p>Backend returns 1+N retry status codes</p>
<p>Or internal errors during handling of messages</p></td>
<td>Backend returns other HTTP status codes from 400 to 600</td>
</tr>
<tr class="odd">
<td rowspan="5">CTB</td>
<td>study-ctb</td>
<td>CtbDicomProcessor</td>
<td>N/A</td>
<td>Internal errors during handling of messages</td>
<td>N/A</td>
</tr>
<tr class="even">
<td>ctb-needs-attributes</td>
<td>Attributes</td>
<td>N/A</td>
<td>N/A</td>
<td>N/A</td>
</tr>
<tr class="odd">
<td>ctb-check-registration</td>
<td>CtbRegistrationTrigger</td>
<td>N/A</td>
<td>Internal errors during handling of messages</td>
<td>N/A</td>
</tr>
<tr class="even">
<td>ctb-needs-registration</td>
<td>Registration</td>
<td>N/A</td>
<td>N/A</td>
<td>N/A</td>
</tr>
<tr class="odd">
<td>ctb-needs-transmit</td>
<td>CtbTransmitter</td>
<td>Cannot reach backend or backend returns 418</td>
<td><p>Backend returns 1+N retry status code</p>
<p>Or internal errors during handling of messages</p></td>
<td>Backend returns other HTTP status codes from 400 to 600</td>
</tr>
<tr class="even">
<td>Triage</td>
<td>triage</td>
<td>TriageProcessor</td>
<td>Cannot reach RIS/Integration Engine or Integration Engine returns
418</td>
<td><p>Integration Engine returns 1+N retry status codes</p>
<p>Or internal errors during handling of messages</p></td>
<td><p>Integration Engine returns other HTTP status codes from 400 to
600</p>
<p>Integration Engine returns AR or AE as ACK code for HL7</p>
<p>Failed to send triage metrics to backend for whatever
reasons</p></td>
</tr>
</tbody>
</table>

Below Table list the HTTP response status codes for 1+N Retry.

| **HTTP Status Code** | **Meaning**           |
|----------------------|-----------------------|
| 429                  | Too many requests     |
| 500                  | Internal server error |
| 502                  | Bad gateway           |
| 503                  | Service unavailable   |
| 504                  | Gateway timeout       |

#### Data Persistence

All exchanges and queues are declared as durable so they will survive
from message broker restart. All messages in the queues are published as
persistence so the messages will be persisted on disk and also can
survive from message broker restart. Dead letter queues (queue names
with the suffix .error) are declared as lazy so that messages won’t be
kept in memory. TTL and max length were setup for transaction queues
(queue names without the suffix .error) to limit the messages persisted.
TTL and max length were also setup at dead letter queues to limit the
dead letter messages persisted.

The default TTL for transaction queues is 48 hours. The default max
length for transaction queues is 6000. Queues for different pipeline
might have different TTL and length as needed by the pipeline. Details
listed in below table.

The default TTL for error queues is 24 hours. The default max length for
transaction queues is 1000.

Below table lists all transaction queues in the system and their TTL and
length settings.

<table style="width:100%;">
<colgroup>
<col style="width: 10%" />
<col style="width: 22%" />
<col style="width: 22%" />
<col style="width: 22%" />
<col style="width: 7%" />
<col style="width: 13%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>Pipeline</strong></th>
<th><strong>Queue</strong></th>
<th><strong>Publisher Service</strong></th>
<th><strong>Consumer Service</strong></th>
<th><strong>TTL</strong></th>
<th><strong>Length</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td rowspan="2">CXR</td>
<td>study</td>
<td>DicomReceiver</td>
<td>DicomTransmitter</td>
<td>48</td>
<td>500,000</td>
</tr>
<tr class="even">
<td>study-completion</td>
<td><p>CxrTrigger</p>
<p>Hl7Receiver</p></td>
<td>StudyCompletion</td>
<td>48</td>
<td>6000</td>
</tr>
<tr class="odd">
<td rowspan="5">CTB</td>
<td>study-ctb</td>
<td>DIcomReceiver</td>
<td>CtbDicomProcessor</td>
<td>48</td>
<td>500,000</td>
</tr>
<tr class="even">
<td>ctb-needs-attributes</td>
<td>CtbAttributesTrigger</td>
<td>Attributes</td>
<td>48</td>
<td>18000</td>
</tr>
<tr class="odd">
<td>ctb-check-registration</td>
<td>Attributes</td>
<td>CtbRegistrationTrigger</td>
<td>48</td>
<td>18000</td>
</tr>
<tr class="even">
<td>ctb-needs-registration</td>
<td>CtbRegistrationTrigger</td>
<td>Registration</td>
<td>48</td>
<td>18000</td>
</tr>
<tr class="odd">
<td>ctb-needs-transmit</td>
<td><p>CtbAttributesTrigger</p>
<p>CtbRegistrationTrigger</p>
<p>Registration</p></td>
<td>CtbTransmitter</td>
<td>48</td>
<td>18000</td>
</tr>
<tr class="even">
<td>Triage</td>
<td>triage</td>
<td>NotificationReceiver</td>
<td>TriageProcessor</td>
<td>12</td>
<td>6000</td>
</tr>
</tbody>
</table>

### Data Storage

The PostgreSQL database is acting as short-term cache to provide context
for some of the workflows or services. Data is saved in
Study-\>Series-\>Images model similar as the data schema in DICOM. Data
will be deleted from the database after some time like one hour which is
configurable.

Below is a diagram explaining the basic database schema:

<img src="./media/image7.png" style="width:6.26806in;height:2.99861in"
alt="Diagram Description automatically generated" />

Figure 6 - Data relationship

The studies table saves all study level data and patient data.

The series table saves all series level data.

The series_triggers table is used to track trigger details each time a
CTB series got triggered.

The images table saves all image level data.

### File Storage

The MinIO file system is acting as short-term cache to provide file
storages for some of the workflows or services. Files are saved in
buckets and referenced by database records so that they are indexed.
Files will be deleted together with reference database records after
some time like one hour which is configurable. There is also an expiry
policy set up for each bucket so that orphaned files will be removed by
Minio.

### Orchestration

Orchestration engine is the infrastructure to manage the containers and
provide basic services needed by the containers like monitoring, health
checking, scaling up and down, secret management, load balancer etc. The
orchestration engine for Integration Adapter is K3s as a Kubernetes
cluster running with Docker.

### Installation

Integration Adapter needs to be installed with the provisioner tool from
the on-prem tools set onto systems bootstrapped from the customised OVA
Linux OS image. All configuration items are exposed as provisioner
prompts which can be set when install.

### Data Security and Integrity

Data security and integrity are maintained during each step of the
transmission. Communication between the Integration Adapter and the
Backend API is made via HTTPS or Web Socket with TLS. All HTTPS
interaction with the Backed API must pass authentication through the
process of request signing. Web Socket connection with TLS is upgraded
from HTTPS request with request signing. This make sure the
communication is encrypted, authorised, and signed.

# Secrets

Secrets are credentials like username/password, client id/secret etc.
which used to securely communicate with services like
RabbitMQ/PostgreSQL, customer systems or backend systems. Secrets are
securely saved in Hashicorp Vault as a sidecar container running
together with Integration Adapter services.

### Logs

Detailed logs will be outputted at both component and service level. Log
output level can be configured, also logger can be configured to log to
files or console. Log aggregation/rotation were setup so that the log
files will not grow forever and occupy all the disk space on the host
system.

### Metrics

Datadog was picked as the metrics collecting and analysing platform for
Optimus. For Integration Adapter, metrics are defined at both component
and service level. Datadog agent is running as a sidecar container
besides all Integration Adapter micro-service containers. Metrics were
sent by the services to the agent and then forwarded to Datadog in the
cloud. Datadog agent is also fully integrated with the hosting system
and the orchestration engine Kubernetes/Docker. So, metrics of the
hosting OS, orchestration engine, all containers are also automatically
collected. Datadog agent is also fully integrated with RabbitMQ.

With a centralised metrics collecting and analysing platform like
Datadog, all instances of Integration Adapter running at customer sites
can be remotely monitored in a central place.

### Components

Components are the corner stones of Integration Adapter. They are the
basic parts of Integration Adapter. Each component has a solo function
and are assembled with other components to form services.

#### CStoreServer

CStoreServer is a DICOM C-ECHO and C-STORE SCP (Sevice Class Provider,
or server). It is the component to accept DICOM associations from PACS
and receive DICOM images from PACS. It will publish the accepted DICOM
dataset out so that other components can consume.

CStoreServer is implemented based on the python pydicom and pynetdicom
libs. CStoreServer is implemented as multiple threaded, which means it
is able to accept multiple associations simultaneously and accept DICOM
data from multiple peers at the same time.

Supported Presentation Contexts:

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>SOP Classes</strong></th>
<th><strong>Transfer Syntaxes</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>1.2.840.10008.1.1 (Verification SOP Class)</td>
<td><p>1.2.840.10008.1.2 (Implicit VR Endian)</p>
<p>1.2.840.10008.1.2.1 (Explicit VR Little Endian)</p>
<p>1.2.840.10008.1.2.2 (Explicit VR Big Endian)</p></td>
</tr>
<tr class="even">
<td><p>1.2.840.10008.5.1.4.1.1.1 (Computed Radiography Image
Storage)</p>
<p>1.2.840.10008.5.1.4.1.1.1.1 (Digital X-Ray Image Storage – for
Presentation)</p></td>
<td><p>1.2.840.10008.1.2.4.90 (JPEG 2000 Image Compression - Lossless
Only)</p>
<p>1.2.840.10008.1.2 (Implicit VR Endian)</p>
<p>1.2.840.10008.1.2.1 (Explicit VR Little Endian)</p>
<p>1.2.840.10008.1.2.2 (Explicit VR Big Endian)</p>
<p>1.2.840.10008.1.2.4.91 (JPEG 2000 Image Compression)</p></td>
</tr>
<tr class="odd">
<td><p>1.2.840.10008.5.1.4.1.1.2 (CT Image Storage)</p>
<p>1.2.840.10008.5.1.4.1.1.2.1 (Enhanced CT Image Storage)</p>
<p>1.2.840.10008.5.1.4.1.1.2.2 (Legacy Converted Enhanced CT Image
Storage)</p></td>
<td><p>1.2.840.10008.1.2.4.90 (JPEG 2000 Image Compression - Lossless
Only)</p>
<p>1.2.840.10008.1.2 (Implicit VR Endian)</p>
<p>1.2.840.10008.1.2.1 (Explicit VR Little Endian)</p>
<p>1.2.840.10008.1.2.2 (Explicit VR Big Endian)</p></td>
</tr>
</tbody>
</table>

#### Hl7Server

Hl7Server is a HL7 V2 MLLP server. It is the component to accept MLLP
socket connection from RIS/PACS and receive HL7 messages from RIS/PACS.
It will publish the parsed HL7 ORM^O01 (General order message) out so
that other components can consume.

Hl7Server is implemented based on the python hl7apy lib. Hl7Server is
implemented as multiple threaded, which means it is able to accept
multiple MLLP socket connections simultaneously and accept HL7 messages
from multiple peers at the same time.

Supported HL7 V2 messages: ORM^O01 (General Order Message)

#### QueueProducer

QueueProducer is an AMQP (Advanced Message Queuing Protocol) client
which publish messages to message brokers which support AMQP like
RabbitMQ.

Connection to queue will only be setup when a new message arrives, and
the connection will then be kept unless it is lost because of connection
issues. Recovering from lost connection to message broker is built into
QueueProducer so that even if a temporary connection lost happened, it
can automatically recover.

A one-off republish mechanism is built into QueueProducer which means if
issue happens while publishing a message, it will try to republish the
message immediately, but only try once. If issue continually happen for
that message, it will be dropped.

#### QueueConsumer

QueueConsumer is an AMQP client which consumes messages from message
brokers which supports AMQP like RabbitMQ.

Connection to queue will be setup immediately once QueueConsumer created
to start consuming messages. Recovering from lost connection to message
broker is built into QueueConsumer so that even if a temporary
connection lost happened, it can automatically recover.

The continuously retry mechanism and 1+n retry mechanism are built into
QueueConsumer.

#### ImageProcessor (also known as DicomFileProcessor)

The duty of ImageProcessor is to parse the tags in DICOM datasets and
convert them as JSON datasets. This includes normal DICOM tags, or
headers, and the pixel data tag, or the actual image.

For pixel data processing, ImageProcessor support Transfer Syntax
1.2.840.10008.1.2.4.90 (JPEG2000 lossless), 1.2.840.10008.1.2.4.91 (JPEG
2000 Image Compression) and uncompressed.

#### WebTransport (also known as HttpClient)

The function of WebTransport is to act as a HTTP/HTTPS client to make
HTTP/HTTPS request.

The connection between WebTransport and Backend is through HTTPS and
authorised using request signing.

#### WebSocketClient

The function of WebSocketClient is to act as a Web Socket client to
securely connect to Optimus Backend Services and accept data from it.

The Web Socket connection is upgraded from an HTTPS request authorised
by request signing and all communications are using TLS.

#### Repository (also known as DatabaseRepository or DatabaseClient)

The function of Repository is to act as database clients to connect to
database and manipulate data in database like create, read, update, or
delete data from PostgreSQL database.

#### Scheduler

Scheduler is acting like a timer to schedule callbacks at configured
time.

#### Hl7Client

Hl7Client is a HL7 V2 MLLP client. It is the component to setup MLLP
socket connection to RIS/PACS and send HL7 messages to RIS/PACS.

Hl7Client is implemented based on the python hl7apy lib.

Supported HL7 V2 messages: ORM^O01 (General Order Message)

#### ApiServer (Gunicorn)

The RisInjector service is an API server with Python Gunicorn as
framework.

#### S3Client

S3Client is a client for S3 API compatible file storage services like S3
or MinIO. It supports basic S3 operations like create buckets, save
files, get files, delete files etc.

#### VaultClient (also known as SecretService)

VaultClient is a client for Vault to retrieve secrets saved in Vault.

### Services

#### CXR pipeline

Refer figure 1 for details about how data flows and services interact.

For the CXR pipeline, DICOM files are accepted by DicomReceiver and then
sent to the Backend by DicomTransmitter service. Images will also be
indexed into database which will be used by time-based trigger or other
pipelines like Triage.

For each CXR study, a separate trigger message from the Integration
Adapter is needed to trigger the AI predication at the Backend. HL7
message-based trigger works by accepting study completion HL7 messages
from RIS then forwarding to StudyCompletion to send as trigger to the
Backend. While time-based trigger works by checking the data indexed in
database for any study that is ready to trigger then sending to
StudyCompletion to send as trigger to the Backend. Only one trigger
mechanism should be used, this can be configured when install the
Integration Adapter.

##### DicomReceiver (Shared with CTB pipeline)

DicomReceiver is assembled with CStoreServer and QueueProducer to accept
DICOM data from PACS and publish to queue.

It supports publish accepted DICOM images into different queues based on
mappings between SOP classes and queue names. This way it distributes
CXR images and CTB images to two different pipelines.

##### DicomTransmitter

DicomTransmitter is assembled with QueueConsumer, ImageProcessor,
Repository and WebTransport to consume DICOM dataset from queue, parse
as JSON, save the metadata into database and send to Backend Services.
Pixel data is converted to JPEG 2000 lossless if it is in other formats
using ImageProcessor.

##### Hl7Receiver

Hl7Receiver is assembled with Hl7Server and QueueProducer to accept HL7
V2 ORM^O01 messages from RIS and publish study completion message to
queue if the messages satisfy configured conditions.

##### StudyCompletion Transmitter

StudyCompletion Transmitter is assembled with QueueConsumer and
WebTransport to consume study completion messages from queue and send to
Optimus Backend Services to trigger CXR predications.

##### CxrTrigger

CxrTrigger is assembled with Scheduler, Repository and QueueProducer to
schedule a job to periodically check from database for any CXR studies
that are not processed yet and did not receive new images for a
configured time, then send study completion message for them into queue
of StudyCompletion.

#### Triage Pipeline

Refer figure 1 for details about how data flows and services interact.

Once AI predication is finished, a notification message is pushed from
the Backend to the Integration Adapter. The type of this message is
checked and if it equals to TRIAGE then the message is sent to the
Triage queue.

The notification message includes very basic information about the
study, and the Triage processor uses the prediction_id field to retrieve
form the Backend a full Triage message with all findings and their
priorities based on the definition from the customer RIS system. This
full triage message is processed then sent to either RisInjector or an
integration engine to send to customer RIS system using HL7 or
customised APIs of the RIS systems.

##### NotificationReceiver

NotificationReceiver is assembled with WebSocketClient and QueueProducer
to receive AI processing notifications from Backend Services, check the
notification type and send to corresponding queue.

##### TriageProcessor

TriageProcessor is assembled with QueueConsumer, Reposity and HttpClient
to consume notification message from the queue, fetch the full triage
message, calculate priorities from the findings and then sends the
result to either RisInjector or an external integration engine. The
followed path depends on the URL this service is configured with.
Optionally, patient data can be loaded from database and send as part of
the result.

##### RisInjector

RisInjector is acting as an API server and assembled with HttpClient and
Hl7Client. Its function is to accept calls from ResultProcessor and send
HL7 messages to RIS/PACS to update priority of orders. Optionally it can
check the current status and priority of the order if RIS is Visage RIS.

#### CTB Pipeline

Refer figure 2 for details about how data flows and services interact.

For the CTB pipeline, DICOM files are accepted by DicomReceiver and then
processed by CtbDicomProcessor to be saved into file storage and indexed
into database. The CtbAttributesTrigger service checks the database and
looks for any series that are not processed yet and did not receive new
images for a configured time. Filters will then be applied to the series
picked up and series passed filters will then be sent to CtbAttributes
to predict whether they are primary series or not. Primary series will
then be sent to CtbRegistration to be registered. At the end of the
pipeline, CtbTransmitter will send the process result of each series to
the Backend.

##### DicomReceiver (Shared with CXR pipeline)

Refer [DicomReceiver](#dicomreceiver-shared-with-ctb-pipeline) of CXR
pipeline in section 6.1.4.1.1

##### CtbDicomProcessor

CtbDicomProcessor is assembled with QueueConsumer, ImageProcessor,
Repository and S3Client to consume DICOM images from queue, parse
headers from DICOM images, save DICOM files into MinIO and index them
into database.

##### CtbAttributesTrigger

CtbAttributesTrigger is assembled with Scheduler, Repository and
QueueProducer to schedule a job to periodically check from database for
any CTB series that are not processed yet and did not receive new images
for a configured time. For the series picked up, it will then apply the
filters to them. Series passed all filters will be sent to
CtbAttributes, otherwise they will be sent to CtbTransmitter with filter
error codes.

Below filters are applied in order:

1.  Minimum patient age in years. Patient must be equal to or older than
    configured patient age in years.

2.  Minimum and maximum number of images. Number of images must be
    inside the range. The range is inclusive.

3.  Whether contrast agent is present. Must not have contrast agent.

4.  Maximum slice thickness. Most common slice thickness of the series
    must equal to or smaller than configured slice thickness.

##### CtbAttributes

CtbAttributes is assembled with QueueConsumer, QueueProducer, S3Client
and an Attributes model. It will run the model against the series passed
in to predict whether they are primary CTB series or not. Results are
passed to CtbRegistrationTrigger for processing.

##### CtbRegistrationTrigger

CtbRegistrationTrigger is assembled with QueueConsumer, QueueProducer
and Repository. It accepts the result from CtbAttributes, check whether
the series are predicted as primary or not. It will also check whether
the study already have other primary series with higher attributes score
or not. Primary series passed check will be sent to CtbRegistration to
register, otherwise they will be sent to CtbTransmitter with attributes
error codes.

##### CtbRegistration

CtbRegistration is assembled with QueueConsumer, QueueProducer, S3Client
and a Registration algorism (which includes an AI model as one of its
steps). It will check the series passed to it and register. A registered
achieve will be generated for a series if registered successfully,
otherwise a registration error code indict what is wrong. The results
will be passed to CtbTransmitter.

##### CtbTransmitter

CtbTransmitter is assembled with QueueConsumer, Repository, S3Client and
HttpClient. It is the last service along the CTB pipeline to send result
of a series to the Backend. The result for a series includes meta data
from study, series and image level, any error from filters, attributes
or registration, registered archive it no errors and other additional
data like filter meta etc.

#### DataCleaner

DataCleaner is assembled with Scheduler, Repository and S3Client.
DataCleaner is the cleaner for the Integration Adapter to remove old
data it saved so that it won’t use up all disk space allocated to it.

DataCleaner will schedule a job to periodically check any studies that
are saved in database longer than a configured value. Any data satisfies
the conditions will be removed from database; files saved in file
storage will be removed together with the database records.
