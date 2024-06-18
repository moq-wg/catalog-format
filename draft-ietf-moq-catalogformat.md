---
title: "Common Catalog Format for moq-transport"
category: info

docname: draft-ietf-moq-catalogformat-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Applications and Real-Time"
workgroup: "Media Over QUIC"
keyword:
 - moq
 - warp
 - catalog
venue:
  group: "Media Over QUIC"
  type: "Working Group"
  mail: "moq@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/moq/"
  github: "moq-wg/catalog-format"
  latest: "https://moq-wg.github.io/catalog-format/draft-ietf-moq-catalogformat.html"

author:

 -
    fullname: Suhas Nandakumar
    organization: Cisco
    email: snandaku@cisco.com

 -
    fullname: Will Law
    organization: Akamai
    email: wilaw@akamai.com

 -
    fullname: Mo Zanaty
    organization: Cisco
    email: mzanaty@cisco.com

normative:
  MoQTransport: I-D.ietf-moq-transport
  Framemarking: I-D.ietf-avtext-framemarking
  WebCodecs:
    title: "WebCodecs"
    date: July 2023
    target: https://www.w3.org/TR/webcodecs/
  WEBCODECS-CODEC-REGISTRY:
    title: "WebCodecs Codec Registry"
    date: July 2023
    target: https://www.w3.org/TR/webcodecs-codec-registry/
  CMAF:
    title: "Information technology -- Multimedia application format (MPEG-A) -- Part 19: Common media application format (CMAF) for segmented media"
    date: 2020-03
  JSON: RFC8259
  BASE64: RFC4648
  LANG: RFC5646
  MIME: RFC6838
  JSON-PATCH: RFC6902
  RFC5226: RFC5226

informative:


--- abstract

This specification defines a Common Catalog specification for streaming formats
implementing the MOQ Transport Protocol [MOQTransport]. Media over QUIC
Transport (MOQT) defines a publish/subscribe based unified media delivery
protocol for delivering media for streaming and interactive applications over
QUIC. The catalog describes the content made available by a publisher, including
information necessary for subscribers to select, subscribe and initialize
tracks.

--- middle

# Introduction

MOQT [MOQTransport] defines a transport protocol that utilizes the QUIC network
protocol [QUIC] and WebTransport[WebTrans] to move objects between publishers,
subscribers and intermediaries. Tracks are identified using a tuple of the Track
Namespace and the Track Name. A MOQT Catalog is a specialized track which
captures details of all the tracks output by a publisher, including the
identities, media profiles, initialization data and inter-track relationships.
The mapping of media characteristics of objects with the tracks, as well as
relative prioritization of those objects, are captured in separate MoQ
Streaming Format specifications. This specification defines a JSON encoded
catalog.


# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in RFC 2119 [RFC2119].


# Catalog {#catalog}

A Catalog is a MOQT Object that provides information about tracks from a given
publisher. A Catalog is used by publishers for advertising their output and for
subscribers in consuming that output. The payload of the Catalog object is
opaque to Relays and can be end-to-end encrypted. The Catalog provides the names
and namespaces of the tracks being produced, along with the relationship between
tracks, properties of the tracks that consumers may use for selection and any
relevant initialization data.

A special case of the catalog exists which describes other catalogs instead of
tracks. A catalog might describe tracks, or catalogs, but never both at the same
time.



## Catalog Fields

A catalog is a JSON [JSON] document, comprised of a series of mandatory and
optional fields. At a minimum, a catalog MUST provide all mandatory fields and
one of either a 'tracks' field or a 'catalogs' field.  A producer MAY add
additional fields to the ones described in this draft. Custom field names MUST
NOT collide with field names described in this draft. To prevent custom field
name collisions with future versions, custom field names SHOULD be prefixed
using reverse domain name notation e.g "com.example-size". The order of field
names within the JSON document is not important. Any track or catalog field
declared at the root level is inherited by all tracks or catalogs. Any track or
catalog field declared within a track or catalog object overwrites any inherited
value.


A parser MUST ignore fields it does not understand.

Table 1 provides an overview of all fields defined by this document.

| Field                   |  Name                  |           Definition      |
|:========================|:=======================|:==========================|
| Catalog version         | version                | {{catalogversion}}        |
| Streaming format        | streamingFormat        | {{streamingformat}}       |
| Streaming format version| streamingFormatVersion | {{streamingformatversion}}|
| Supports delta updates  | supportsDeltaUpdates   | {{supportsdeltaupdates}}  |
| Common Track Fields     | commonTrackFields      | {{commontrackfields}}     |
| Tracks                  | tracks                 | {{tracks}}                |
| Catalogs                | catalogs               | {{catalogs}}              |
| Track namespace         | namespace              | {{tracknamespace}}        |
| Track name              | name                   | {{trackname}}             |
| Packaging               | packaging              | {{packaging}}             |
| Track operation         | operation              | {{trackoperations}}       |
| Track label             | label                  | {{tracklabel}}            |
| Render group            | renderGroup            | {{rendergroup}}           |
| Alternate group         | altGroup               | {{altgroup}}              |
| Initialization data     | initData               | {{initdata}}              |
| Initialization track    | initTrack              | {{inittrack}}             |
| Selection parameters    | selectionParams        | {{selectionparameters}}   |
| Dependencies            | depends                | {{dependencies}}          |
| Temporal ID             | temporalId             | {{temporalid}}            |
| Spatial ID              | spatialId              | {{spatialid}}             |
| Codec                   | codec                  | {{codec}}                 |
| Mime type               | mimeType               | {{mimetype}}              |
| Framerate               | framerate              | {{framerate}}             |
| Bitrate                 | bitrate                | {{bitrate}}               |
| Width                   | width                  | {{width}}                 |
| Height                  | height                 | {{height}}                |
| Audio sample rate       | samplerate             | {{audiosamplerate}}       |
| Channel configuration   | channelConfig          | {{channelconfiguration}}  |
| Display width           | displayWidth           | {{displaywidth}}          |
| Display height          | displayHeight          | {{displayheight}}         |
| Language                | lang                   | {{language}}              |


| Field                   |  Name                  | Required |  Location |  JSON type |           Definition       |
|:========================|:=======================|:=========|:==========|:===========|:===========================|
| Catalog version         | version                |  yes     |   R       |  String    | {{catalogversion}}         |
| Streaming format        | streamingFormat        |  yes     |   RC      |  String    | {{streamingformat}}        |
| Streaming format version| streamingFormatVersion |  yes     |   RC      |  String    | {{streamingformatversion}} |
| Supports delta updates  | supportsDeltaUpdates   |  opt     |   RC      |  Boolean   | {{supportsdeltaupdates}}   |
| Common Track Fields     | commonTrackFields      |  opt     |   R       |  Object    | {{commontrackfields}}      |
| Tracks                  | tracks                 |  opt     |   R       |  Array     | {{tracks}}                 |
| Catalogs                | catalogs               |  opt     |   R       |  Array     | {{catalogs}}               |
| Track namespace         | namespace              |  opt     |   TFC     |  String    | {{tracknamespace}}         |
| Track name              | name                   |  yes     |   TFC     |  String    | {{trackname}}              |
| Packaging               | packaging              |  yes     |   TF      |  String    | {{packaging}}              |
| Track operation         | operation              |  yes     |   TF      |  String    | {{trackoperations}}        |
| Track label             | label                  |  opt     |   TF      |  String    | {{tracklabel}}             |
| Render group            | renderGroup            |  opt     |   TF      |  Number    | {{rendergroup}}            |
| Alternate group         | altGroup               |  opt     |   TF      |  Number    | {{altgroup}}               |
| Initialization data     | initData               |  opt     |   TF      |  String    | {{initdata}}               |
| Initialization track    | initTrack              |  opt     |   TF      |  String    | {{inittrack}}              |
| Selection parameters    | selectionParams        |  opt     |   TF      |  Object    | {{selectionparameters}}    |
| Dependencies            | depends                |  opt     |   T       |  Array     | {{dependencies}}           |
| Temporal ID             | temporalId             |  opt     |   T       |  Number    | {{temporalid}}             |
| Spatial ID              | spatialId              |  opt     |   T       |  Number    | {{spatialid}}              |
| Codec                   | codec                  |  opt     |   S       |  String    | {{codec}}                  |
| Mime type               | mimeType               |  opt     |   S       |  String    | {{mimetype}}               |
| Framerate               | framerate              |  opt     |   S       |  Number    | {{framerate}}              |
| Bitrate                 | bitrate                |  opt     |   S       |  Number    | {{bitrate}}                |
| Width                   | width                  |  opt     |   S       |  Number    | {{width}}                  |
| Height                  | height                 |  opt     |   S       |  Number    | {{height}}                 |
| Audio sample rate       | samplerate             |  opt     |   S       |  Number    | {{audiosamplerate}}        |
| Channel configuration   | channelConfig          |  opt     |   S       |  String    | {{channelconfiguration}}   |
| Display width           | displayWidth           |  opt     |   S       |  Number    | {{displaywidth}}           |
| Display height          | displayHeight          |  opt     |   S       |  Number    | {{displayheight}}          |
| Language                | lang                   |  opt     |   S       |  String    | {{language}}               |

Table 2 defines the allowed locations for these fields within the document

| Location |                Allowed locations for the field                |
|:=========|:==============================================================|
| R        | The Root of the JSON object                                   |
| RC       | The Root or a Catalog object                                  |
| TFC      | A Track object, Common Track Fields object or Catalog object. |
| TF       | A Track object or a Common Track Fields object                |  
| T        | Track object                                                  |
| S        | Selection Parameters object.                                  |

## Catalog version {#catalogversion}
Location: R    Required: Yes    Json Type: String 
Versions of this catalog specification are defined using monotonically
increasing integers. There is no guarantee that future catalog versions are
backwards compatible and field definitions and interpretation may change between
versions. A subscriber MUST NOT attempt to parse a catalog version which it does
not understand.

This document defines version 1.

### Streaming format {#streamingformat}
Location: RC    Required: Yes    Json Type: String 
A number indicating the streaming format type. Every MoQ Streaming Format
normatively referencing this catalog format MUST register itself in the "MoQ
Streaming Format Type" table. See {{ianaconsiderations}} for additional details.

### Streaming format version {#streamingformatversion}
Location: RC    Required: Yes    Json Type: String 
A string indicating the version of the streaming format to which this catalog
applies. The structure of the version string is defined by the streaming format.

### Supports delta updates {#supportsdeltaupdates}
Location: RC    Required: Optional    Json Type: Boolean
A boolean that if true indicates that the publisher MAY issue incremental
(delta) updates - see {{patch}}. If false or absent, then the publisher
gaurantees that they will NOT issue any incremental updates and that any future
updates to the catalog will be independent. The default value is false. This
field MUST be present if its value is true, but may be omitted if the value is
false.

### Common track fields {#commontrackfields}
Location: R    Required: Optional    Json Type: Object
An object holding a collection of Track Fields (objects with a location of TF
or TFC in table 1) which are to be inherited by all tracks. A field defined at
the Track object level always supercedes any value inherited from the Common
Track Fields object.

### Tracks {#tracks}
Location: R    Required: Optional    Json Type: Array
An array of track objects {{trackobject}}. If the 'tracks' field is present
then the 'catalog' field MUST NOT be present.

### Catalogs {#catalogs}
Location: R    Required: Optional    Json Type: Array
An array of catalog objects {{catalogobject}}. If the 'catalogs' field is
present then the 'tracks' field MUST NOT be present. A catalog MUST NOT list
itself in the catalog array.

### Catalog object {#catalogobject}
A catalog object is a collection of fields whose location is specified as 'RC'
or 'TFC' in Table 2.

### Tracks object {#trackobject}
A track object is a collection of fields whose location is specified as 'TFC',
'TF' or 'T' in Table 2.

### Track namespace {#tracknamespace}
Location: TFC    Required: Optional    Json Type: String
The name space under which the track name is defined. See section 2.3 of
{{MoQTransport}}. The track namespace is optional. If it is not declared within
the Common Track Fields object or within a track, then each track MUST inherit
the namespace of the catalog track. A namespace declared in a track object or
catalog object overwrites any inherited name space.

### Track name {#trackname}
Location: TFC    Required: Yes   Json Type: String
A string defining the name of the track. See section 2.3 of {{MoQTransport}}.
Within the catalog, track names MUST be unique per namespace.

### Packaging {#packaging}
A string defining the type of payload encapsulation. Allowed values are strings
as defined in Table 3.

Table 3: Allowed packaging values

| Name            |   Value   |      Draft       |
|:================|:==========|:=================|
| CMAF            | "cmaf"    | See RFC XXXX     |
| LOC             | "loc"     | See RFC XXXX     |



### Track operations {#trackoperations}

Each track description can specify an optional operation value that identifies
the catalog producer's intent. Track operation is an enumeration of values
as defined below.

* Add: Indicates the track is added to the catalog and the consumers of the
 catalog may subscribe to the track.

* Delete: Indicates that media producer is no longer producing media on the
associated track. Subscribers SHOULD terminate their subscriptions.

A catalog update in which all previously added tracks are deleted SHOULD be
interpreted by a subscriber to indicate that the publisher has terminated the
broadcast.

Table 4 defines the numerical values for the track operations.

Table 4: Allowed track operations

| Name            | Value  | Default value  |
|:================|:=======|:===============|
| Add             | add    |    yes         |
| Delete          | delete |                |

The default track operation is 'add'. This value does not need to be declared
in the track object.


### Track label {#tracklabel}
A string defining a human-readable label for the track. Examples might be "Overhead camera view" or "Deutscher Kommentar". Note that the {{JSON}} spec requires UTF-8 support by decoders.

### Render group {#rendergroup}
An integer specifying a group of tracks which are designed to be rendered together. Tracks with the same group number SHOULD be rendered simultaneously, are usually time-aligned and are designed to accompany one another. A common example would be tying together audio and video tracks.

### Alternate group {#altgroup}
An integer specifying a group of tracks which are alternate versions of one-another. Alternate tracks represent the same media content, but differ in their selection properties. Alternate tracks SHOULD have matching framerate {{framerate}} and media time sequences. A subscriber typically subscribes to one track from a set of tracks specifying the same alternate group number. A common example would be a set video tracks of the same content offered in alternate bitrates.


### Initialization data {#initdata}
A string holding Base64 [BASE64] encoded initialization data for the track.

### Initialization track {#inittrack}
A string specifying the track name of another track which holds initialization data for the current track. Initialization tracks MUST NOT be added to the tracks array {{tracks}}. They are referenced only via the initialization track field of the track which they initialize.

### Selection parameters {#selectionparameters}
An object holding a series of name/value pairs which a subscriber can use to select tracks for subscription. If present, the selection parameters object MUST NOT be empty. Any selection parameters declared at the root level are inherited by all tracks. A selection parameters object may exist at both the root and track level. Any declaration of a selection parameter at the track level overrides the inherited root value.

### Dependencies {#dependencies}
Certain tracks may depend on other tracks for decoding. Dependencies holds an array of track names {{trackname}} on which the current track is dependent. Since only the track name is signaled, the namespace of the dependencies is assumed to match that of the track declaring the dependencies.

### Temporal ID {#temporalid}
A number identifying the temporal layer/sub-layer encoding of the track, starting with 0 for the base layer, and increasing with higher temporal fidelity.

### Spatial ID {#spatialid}
A number identifying the spatial layer encoding of the track, starting with 0 for the base layer, and increasing with higher fidelity.

### Codec {#codec}
A string defining the codec used to encode the track.
For LOC packaged content, the string codec registrations are defined in Sect 3 and Section 4 of {{WEBCODECS-CODEC-REGISTRY}}.
For CMAF packaged content, the string codec registrations are defined in XXX.

### Mimetype {#mimetype}
A string defining the mime type [MIME] of the track. This parameter is typically supplied with CMAF packaged content.

### Framerate {#framerate}
A number defining the framerate of the track, expressed as frames per second.

### Bitrate {#bitrate}
A number defining the bitrate of track, expressed in bits second.

### Width {#width}
A number expressing the encoded width of the track content in pixels.

### Height {#height}
A number expressing the encoded height of the video frames in pixels.

### Audio sample rate {#audiosamplerate}
The number of audio frame samples per second. This property SHOULD only accompany audio codecs.

### Channel configuration {#channelconfiguration}
A string specifying the audio channel configuration. This property SHOULD only accompany audio codecs. A string is used in order to provide the flexibility to describe complex channel configurations for multi-channel and Next Generation Audio schemas.

### Display width {#displaywidth}
A number expressing the intended display width of the track content in pixels.

### Display height {#displayheight}
A number expressing the intended display height of the track content in pixels.

### Language {#language}
A string defining the dominant language of the track. The string MUST be one of the standard Tags for Identifying Languages as defined by [LANG].


## Catalog Patch {#patch}
A catalog update might contain incremental changes. This is a useful property if many tracks may be initially declared but then there are small changes to a subset of tracks. The producer can issue a patch to describe these small changes. Changes are described incrementally, meaning that a patch can itself modify a prior patch. Patching leverages JSON PATCH [JSON-PATCH] to modify the catalog.   JSON Patch is a format for expressing a sequence of operations to apply to a target JSON document.

The following rules MUST be followed in processing patches:

* The target JSON to be modified is the JSON document described by the preceding [MOQTransport] Object in the Catalog track, post any patching that may have been applied to that Object.
* A Catalog Patch is identified by having a single array at the root level, holding a series of JSON objects, each object representing a single operation to be applied to the target JSON document.
* Operations are applied sequentially in the order they appear in the array.  Each operation in the sequence is applied to the target document; the resulting document becomes the target of the next operation.  Evaluation continues until all operations are successfully applied or until an error condition is encountered.
* Track namespaces and track names may not be changed across patch updates. To change either namespace or name, remove the track and then add a new track with matching properties and the new namespace and name.
* Contents of the track selection properties object may not be varied across updates. To adjust a track selection property, the track must first be removed and then added with the new selection properties and a different name.



## Catalog Examples

The following section provides non-normative JSON examples of various catalogs compliant with this draft.


### Time-aligned Audio/Video Tracks with single quality

This example shows catalog for a media producer capable of sending LOC packaged, time-aligned audio and video tracks.

~~~json
{
  "version": 1,
  "streamingFormat": 1,
  "streamingFormatVersion": "0.2",
  "commonTrackFields": {
     "namespace": "conference.example.com/conference123/alice",
     "packaging": "loc",
     "renderGroup": 1
  },
  "tracks": [
    {
      "name": "video",
      "selectionParams":{"codec":"av01.0.08M.10.0.110.09","width":1920,"height":1080,"framerate":30,"bitrate":1500000}
    },
    {
      "name": "audio",
      "selectionParams":{"codec":"opus","samplerate":48000,"channelConfig":"2","bitrate":32000}
    }
   ]
}

~~~



### Simulcast video tracks - 3 alternate qualities along with audio

This example shows catalog for a media producer capable
of sending 3 time-aligned video tracks for high definition, low definition and
medium definition video qualities, along with an audio track. In this example the namespace is absent, which infers that each track must inherit the namespace of the catalog.
Additionally this example shows the presence of the supportsDeltaUpdates flag.


~~~json
{
  "version": 1,
  "streamingFormat": 1,
  "streamingFormatVersion": "0.2",
  "supportsDeltaUpdates": true,
  "commonTrackFields": {
     "renderGroup": 1,
     "packaging": "loc"
  },
  "tracks":[
    {
      "name": "hd",
      "selectionParams": {"codec":"av01","width":1920,"height":1080,"bitrate":5000000,"framerate":30},
      "altGroup":1
    },
    {
      "name": "md",
      "selectionParams": {"codec":"av01","width":720,"height":640,"bitrate":3000000,"framerate":30},
      "altGroup":1
    },
    {
      "name": "sd",
      "selectionParams": {"codec":"av01","width":192,"height":144,"bitrate":500000,"framerate":30},
      "altGroup":1
    },
    {
      "name": "audio",
      "selectionParams":{"codec":"opus","samplerate":48000,"channelConfig":"2","bitrate":32000}
    }
   ]
}
~~~


### SVC video tracks with 2 spatial and 2 temporal qualities


This example shows catalog for a media producer capable
of sending scalable video codec with 2 spatial and 2 temporal
layers with a dependency relation as shown below:

~~~ascii-figure

                  +----------+
     +----------->|  S1T1    |
     |            | 1080p30  |
     |            +----------+
     |                  ^
     |                  |
+----------+            |
|  S1TO    |            |
| 1080p15  |            |
+----------+      +-----+----+
      ^           |  SOT1    |
      |           | 480p30   |
      |           +----------+
      |               ^
+----------+          |
|  SOTO     |         |
| 480p15    |---------+
+----------+
~~~



The corresponding catalog uses "depends" attribute to
express the track relationships.

~~~json
{
  "version": 1,
  "streamingFormat": 1,
  "streamingFormatVersion": "0.2",
  "supportsDeltaUpdates": true,
  "commonTrackFields": {
     "namespace": "conference.example.com/conference123/alice",
     "renderGroup": 1,
     "packaging": "loc"
  },
  "tracks":[
    {
      "name": "480p15",
      "selectionParams": {"codec":"av01.0.01M.10.0.110.09","width":640,"height":480,"bitrate":3000000,"framerate":15}
    },
    {
      "name": "480p30",
      "selectionParams": {"codec":"av01.0.04M.10.0.110.09","width":640,"height":480,"bitrate":3000000,"framerate":30},
      "depends": ["480p15"]
    },
    {
      "name": "1080p15",
      "selectionParams": {"codec":"av01.0.05M.10.0.110.09","width":1920,"height":1080,"bitrate":3000000,"framerate":15},
      "depends":["480p15"]
    },

    {
      "name": "1080p30",
      "selectionParams": {"codec":"av01.0.08M.10.0.110.09","width":1920,"height":1080,"bitrate":5000000,"framerate":30},
      "depends": ["480p30", "1080p15"]
    },
    {
      "name": "audio",
      "selectionParams":{"codec":"opus","samplerate":48000,"channelConfig":"2","bitrate":32000}
    }
   ]
}
~~~

### Patch update adding a track

This example shows catalog for the media producer adding a slide track to an established video conference.

~~~json
[
    {
        "op": "add",
        "path": "/tracks/-",
        "value": {
            "name": "slides",
            "selectionParams": {
                "codec": "av01.0.08M.10.0.110.09",
                "width": 1920,
                "height": 1080,
                "framerate": 15,
                "Bitrate": 750000
            },
            "renderGroup": 1
        }
    }
]


~~~

### Patch update removing a track

This example shows patch catalog update for a media producer removing the track from an established video conference.

~~~json
[
  { "op": "remove", "path": "/tracks/2"}
]
~~~

### Patch update removing all tracks and terminating the broadcast

This example shows a patch catalog update for a media producer removing all tracks and terminating the broadcast.

~~~json
[
  { "op": "remove", "path": "/tracks/2"},
  { "op": "remove", "path": "/tracks/1"},
  { "op": "remove", "path": "/tracks/0"},
]

~~~

### CMAF Tracks with multiple qualities of audio and video

This example shows catalog for a sports broadcast sending time-aligned audio and video tracks using CMAF packaging. Init segments are delivered as separate tracks.

~~~json
{
  "version": 1,
  "streamingFormat": 1,
  "streamingFormatVersion": "0.2",
  "supportsDeltaUpdates": true,
  "commonTrackFields": {
     "namespace": "sports.example.com/games/08-08-23/12345",
     "packaging": "cmaf",
     "renderGroup":1
  },
  "tracks": [
    {
      "name": "video_4k",
      "selectionParams":{"codec":"avc1.640033","mimeType":"video/mp4","width":3840,"height":2160,"framerate":30,"bitrate":14931538},
      "initTrack":"init_video_4k",
      "altGroup": 1
    },
    {
      "name": "video_1080",
      "selectionParams":{"codec":"avc1.640028","mimeType":"video/mp4","width":1920,"height":1080,"framerate":30,"bitrate":9914554},
      "initTrack":"init_video_1080",
      "altGroup": 1
    },
    {
      "name": "video_720",
      "selectionParams":{"codec":"avc1.64001f","mimeType":"video/mp4","width":1280,"height":720,"framerate":30,"bitrate":4952892},
      "initTrack":"init_video_720",
      "altGroup": 1
    },
    {
      "name": "audio_aac",
      "selectionParams":{"codec":"mp4a.40.5","mimeType":"audio/mp4","samplerate":48000,"channelConfig":"2","bitrate":67071},
      "initTrack":"init_audio_aac",
      "altGroup": 2
    },
    {
      "name": "audio_ec3",
      "selectionParms":{"codec":"ec-3","mimeType":"audio/mp4","samplerate":48000,"channelConfig":"F801","bitrate":256000},
      "initTrack":"init_audio_ec3",
      "altGroup": 2
    }
   ]
}
~~~

### Mixed format example - CMAF and LOC packaging in the same catalog

This example shows catalog describing a broadcast with CMAF packaged video and LOC packaged audio.

~~~json
{
  "version": 1,
  "streamingFormat": 1,
  "streamingFormatVersion": "0.2",
  "commonTrackFields": {
     "namespace": "output.example.com/event/12345",
     "renderGroup":1
  },
  "tracks": [
    {
      "name": "video0",
      "selectionParams":{"codec":"avc1.64001f","mimeType":"video/mp4","width":1280,"height":720,"framerate":30,"bitrate":4952892},
      "initTrack":"init_video_720",
      "packaging":"cmaf"
    },
    {
      "name": "audio",
      "selectionParams":{"codec":"opus","samplerate":48000,"channelConfig":"2","bitrate":32000},
      "packaging": "loc"
    }
   ]
}
~~~


### CMAF Tracks with inband init segments

This example shows catalog for a sports broadcast sending time-aligned audio and video tracks using CMAF packaging. Init segments are delivered as inband data. The data has been truncated for clarity.

~~~json
{
  "version": 1,
  "streamingFormat": 1,
  "streamingFormatVersion": "0.2",
  "commonTrackFields": {
     "namespace": "sports.example.com/games/08-08-23/12345",
     "packaging": "cmaf",
     "renderGroup":1
  },
  "tracks": [
    {
      "name": "video_1080",
      "selectionParams":{"codec":"avc1.640028","mimeType":"video/mp4","width":1920,"height":1080,"framerate":30,"bitrate":9914554},
      "initData":"AAAAGG...BAAAx"
    },
    {
      "name": "audio_aac",
      "selectionParams":{"codec":"mp4a.40.5","mimeType":"audio/mp4","samplerate":48000,"channelConfig":"2","bitrate":67071},
      "initData":"AAAAGG...EAADE="
    }
   ]
}
~~~

### Time-aligned Audio/Video Tracks with custom field values

This example shows catalog for a media producer capable of sending LOC packaged, time-aligned audio and video tracks along with custom fields in each track description.

~~~json
{
  "version": 1,
  "streamingFormat": 1,
  "streamingFormatVersion": "0.2",
  "commonTrackFields": {
     "namespace": "conference.example.com/conference123/alice",
     "packaging": "loc",
     "renderGroup": 1
  },
  "tracks": [
    {
      "name": "video",
      "selectionParams":{"codec":"av01.0.08M.10.0.110.09","width":1920,"height":1080,"framerate":30,"bitrate":1500000},
      "com.example-billing-code": 3201,
      "com.example-tier": "premium",
      "com.example-debug": "h349835bfkjfg82394d945034jsdfn349fns"
    },
    {
      "name": "audio",
      "selectionParams":{"codec":"opus","samplerate":48000,"channelConfig":"2","bitrate":32000}
    }
   ]
}

~~~

### A catalog referencing catalogs for two different formats

This example shows the catalog for a media producer that is outputting two streaming formats simultaneously under different namespaces. Note that each track name referenced points at another catalog object and that only the first catalog supports incremental delta updates.

~~~json
{
  "version": 1,
  "catalogs": [
    {
      "name": "catalog-for-format-one",
      "namespace": "sports.example.com/games/08-08-23/live",
      "streamingFormat":1,
      "streamingFormatVersion": "0.2",
      "supportsDeltaUpdates": true,
    },
    {
      "name": "catalog-for-format-five",
      "namespace": "chat.example.com/games/08-08-23/chat",
      "streamingFormat":5,
      "streamingFormatVersion": "1.6.2"
    }
  ]
}

~~~

# Security Considerations

The catalog contents MAY be encrypted. The mechanism of encryption and the signaling of the keys are left to the Streaming Format referencing this catalog format.

# IANA Considerations {#ianaconsiderations}

This section details how the MoQ Streaming Format Type and new fields can be registered for inclusion in a catalog.

## MoQ Streaming Format Type Registry

This document creates a new registry, "MoQ Streaming Format Type". This registry is managed by the IANA according to the RFC Required  policy of [RFC5226]. The Type value is 2 octets. The range is 0x0000-0xFFFF. The initial entry in the registry is:


| Type   |     Name    |                                RFC                                |
|:-------|:------------|:------------------------------------------------------------------|
| 0x0000 |   Reserved  | https://datatracker.ietf.org/doc/draft-wilaw-moq-catalogformat/   |

No RFC is provided for the initial entry as it is reserved for Every MoQ streaming format draft normatively referencing this catalog format MUST register itself a unique type identifier. The type registry can be updated by incrementally expanding by allocating and reserving new type identifiers.

## Common Catalog Field Registry

This document creates a new IANA registry for the Common Catalog fields.  The registry is called "MoQ Common Catalog Fields".  This registry is managed by the IANA according to the Specification Required policy of [RFC5226]. The initial entries in the registry are:

| Descriptive Name        |  Field Name            | Required |  Location |  JSON type |                         Specification                             |
|:========================|:=======================|:=========|:==========|:===========|:==================================================================|
| Catalog version         | version                |  yes     |   R       |  String    | https://datatracker.ietf.org/doc/draft-wilaw-moq-catalogformat/   |
| Streaming format        | streamingFormat        |  yes     |   RC      |  String    | https://datatracker.ietf.org/doc/draft-wilaw-moq-catalogformat/   |
| Streaming format version| streamingFormatVersion |  yes     |   RC      |  String    | https://datatracker.ietf.org/doc/draft-wilaw-moq-catalogformat/   |
| Tracks                  | tracks                 |  opt     |   R       |  Array     | https://datatracker.ietf.org/doc/draft-wilaw-moq-catalogformat/   |
| Catalogs                | catalogs               |  opt     |   R       |  Array     | https://datatracker.ietf.org/doc/draft-wilaw-moq-catalogformat/   |
| Track namespace         | namespace              |  opt     |   RTC     |  String    | https://datatracker.ietf.org/doc/draft-wilaw-moq-catalogformat/   |
| Track name              | name                   |  yes     |   TC      |  String    | https://datatracker.ietf.org/doc/draft-wilaw-moq-catalogformat/   |
| Packaging               | packaging              |  yes     |   RT      |  String    | https://datatracker.ietf.org/doc/draft-wilaw-moq-catalogformat/   |
| Track operation         | operation              |  yes     |   RT      |  String    | https://datatracker.ietf.org/doc/draft-wilaw-moq-catalogformat/   |
| Track label             | label                  |  opt     |   RT      |  String    | https://datatracker.ietf.org/doc/draft-wilaw-moq-catalogformat/   |
| Render group            | renderGroup            |  opt     |   RT      |  Number    | https://datatracker.ietf.org/doc/draft-wilaw-moq-catalogformat/   |
| Alternate group         | altGroup               |  opt     |   RT      |  Number    | https://datatracker.ietf.org/doc/draft-wilaw-moq-catalogformat/   |
| Initialization data     | initData               |  opt     |   RT      |  String    | https://datatracker.ietf.org/doc/draft-wilaw-moq-catalogformat/   |
| Initialization track    | initTrack              |  opt     |   RT      |  String    | https://datatracker.ietf.org/doc/draft-wilaw-moq-catalogformat/   |
| Selection parameters    | selectionParams        |  opt     |   RT      |  Object    | https://datatracker.ietf.org/doc/draft-wilaw-moq-catalogformat/   |
| Dependencies            | depends                |  opt     |   T       |  Array     | https://datatracker.ietf.org/doc/draft-wilaw-moq-catalogformat/   |
| Temporal ID             | temporalId             |  opt     |   T       |  Number    | https://datatracker.ietf.org/doc/draft-wilaw-moq-catalogformat/   |
| Spatial ID              | spatialId              |  opt     |   T       |  Number    | https://datatracker.ietf.org/doc/draft-wilaw-moq-catalogformat/   |
| Codec                   | codec                  |  opt     |   S       |  String    | https://datatracker.ietf.org/doc/draft-wilaw-moq-catalogformat/   |
| Mime type               | mimeType               |  opt     |   S       |  String    | https://datatracker.ietf.org/doc/draft-wilaw-moq-catalogformat/   |
| Framerate               | framerate              |  opt     |   S       |  Number    | https://datatracker.ietf.org/doc/draft-wilaw-moq-catalogformat/   |
| Bitrate                 | bitrate                |  opt     |   S       |  Number    | https://datatracker.ietf.org/doc/draft-wilaw-moq-catalogformat/   |
| Width                   | width                  |  opt     |   S       |  Number    | https://datatracker.ietf.org/doc/draft-wilaw-moq-catalogformat/   |
| Height                  | height                 |  opt     |   S       |  Number    | https://datatracker.ietf.org/doc/draft-wilaw-moq-catalogformat/   |
| Audio sample rate       | samplerate             |  opt     |   S       |  Number    | https://datatracker.ietf.org/doc/draft-wilaw-moq-catalogformat/   |
| Channel configuration   | channelConfig          |  opt     |   S       |  String    | https://datatracker.ietf.org/doc/draft-wilaw-moq-catalogformat/   |
| Display width           | displayWidth           |  opt     |   S       |  Number    | https://datatracker.ietf.org/doc/draft-wilaw-moq-catalogformat/   |
| Display height          | displayHeight          |  opt     |   S       |  Number    | https://datatracker.ietf.org/doc/draft-wilaw-moq-catalogformat/   |
| Language                | lang                   |  opt     |   S       |  String    | https://datatracker.ietf.org/doc/draft-wilaw-moq-catalogformat/   |

Any registration for a new Field name MUST provide the following information:

* Descriptive Name - a descriptive name for the field.
* Field Name - the JSON field name, as will be used inside the JSON catalog.
* Required - the string "yes" if the field is required in all catalogs and "opt" if it is not.
* Location - a string defining the permissible locations for the field within the catalog:
  * 'R' - the field is located in the Root of the JSON object.
  * 'RC' - the field may be located in either the Root or a Catalog object.
  * 'RTC' - the field may be located in either the Root, or a Track object or a Catalog object.
  * 'TC' - the field may be located in either a Track object or a Catalog object.
  * 'RT' - the field may be located in either the Root or a Track object.
  * 'T' - the field is located in a Track object.
  * 'S' - the field is located in the Selection Parameters object.
* JSON Type  - the JSON type of the field value, which must be one of String, Array, Number, Object or Boolean.
* Specification - a URL to the specification which defines the usage of the field within the catalog, per the Specification Required policy of [RFC5226].



# Acknowledgments
{:numbered="false"}
 - Suhas Nandakumar & Mo Zanaty - this [draft](https://github.com/suhasHere/moq-drafts/tree/master/moq-catalog).
 - The IETF MoQ mailing lists and discussion groups.
