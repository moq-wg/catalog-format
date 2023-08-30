---
title: "Common Catalog Format for moq-transport"
category: info

docname: draft-wilaw-moq-catalogformat-latest
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
  github: "wilaw/catalog-format"
  latest: "https://wilaw.github.io/catalog-format/draft-wilaw-moq-catalogformat.html"

author:
 -
    fullname: Will Law
    organization: Akamai
    email: wilaw@akamai.com

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

informative:


--- abstract

This specification defines an Common Catalog specification for streaming formats implementing the MOQ Transport Protocol [MOQTransport]. Media over QUIC Transport (MOQT) defines a publish/subscribe based unified media delivery protocol for delivering media for streaming and interactive applications over QUIC. The catalog describes the content made available by a publisher, including information necessary for subscribers to select, subscribe and initialize tracks.


--- middle

# Introduction

MOQT [MOQTransport] defines a transport protocol that utilizes the QUIC network protocol [QUIC] and WebTransport[WebTrans] to move objects between publishers, subscribers and intermediaries. Tracks are identified using a tuple of the Track Namespace and the Track Name. A MOQT Catalog is a specialized track which captures details of all the tracks output by a publisher, including the identities, media profiles, initialization data and inter-track relationships. The mapping of media characteristics of objects with the tracks, as well as relative prioritization of those objects, are captured in separate MoQ Streaming Format specifications. This specification defines a JSON encoded catalog.


# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 [RFC2119].


# Catalog {#catalog}

A Catalog is a MOQT Object that provides information about tracks from a given publisher. A Catalog is used by publishers for advertising their output and for subscribers in consuming that output. The payload of the Catalog object is opaque to Relays and can be end-to-end encrypted. The Catalog provides the names and namespaces of the tracks being produced, along with the relationship between tracks, properties of the tracks that consumers may use for selection and any relevant initialization data.

## Catalog Fields

A catalog is a JSON [JSON] document, comprised of a series of mandatory and optional fields. At a minimum, a catalog MUST provide all mandatory fields. A producer MAY add additional fields to the ones described in this draft. Custom field names MUST NOT collide with field names described in this draft. To prevent custom field name collisions with future versions, custom field names SHOULD be prefixed using reverse domain name notation e.g "com.example-size". The order of field names within the JSON document is not important. Any track field declared at the root level is inherited by all tracks. Any track field declared within a track overwrites any inherited value.

A parser MUST ignore fields it does not understand.

Table 1 provides an overview of all fields defined by this document.

| Field                   |  Name  | Required |  Location |  JSON type |           Definition       |
|:========================|:=======|:=========|:==========|:===========|:===========================|
| Streaming format        | f      |  yes     |   R       |  Number    | {{streamingformat}}        |
| Streaming format version| v      |  yes     |   R       |  String    | {{streamingformatversion}} |
| Tracks                  | tracks |  yes     |   R       |  Array     | {{tracks}}                 |
| Parent sequence number  | psn    |  opt     |   R       |  Array     | {{parentsequencenumber}}   |
| Track namespace         | ns     |  yes     |   RT      |  String    | {{tracknamespace}}         |
| Track name              | n      |  yes     |   RT      |  String    | {{trackname}}              |
| Packaging               | p      |  yes     |   RT      |  String    | {{packaging}}              |
| Track operation         | op     |  yes     |   RT      |  Number    | {{trackoperations}}        |
| Track label             | lb     |  opt     |   RT      |  String    | {{tracklabel}}             |
| Render group            | gr     |  opt     |   RT      |  Number    | {{rendergroup}}            |
| Alternate group         | alt    |  opt     |   RT      |  Number    | {{altgroup}}              |
| Dependencies            | alt    |  opt     |   RT      |  Array     | {{dependencies}}          |
| Initialization data     | ind    |  opt     |   RT      |  String    | {{initdata}}               |
| Initialization track    | init   |  opt     |   RT      |  String    | {{inittrack}}              |
| Temporal ID             | tid    |  opt     |   RT      |  Number    | {{temporalid}}             |
| Spatial ID              | sid    |  opt     |   RT      |  Number    | {{spatialid}}              |
| Selection parameters    | sp     |  opt     |   RT      |  Object    | {{selectionparameters}}    |
| Codec                   | c      |  opt     |   S       |  String    | {{codec}}                  |
| Mime type               | mt     |  opt     |   S       |  String    | {{mimetype}}               |
| Framerate               | fr     |  opt     |   S       |  Number    | {{framerate}}              |
| Bitrate                 | br     |  opt     |   S       |  Number    | {{bitrate}}                |
| Width                   | wd     |  opt     |   S       |  Number    | {{width}}                  |
| Height                  | ht     |  opt     |   S       |  Number    | {{height}}                 |
| Audio sample rate       | sr     |  opt     |   S       |  Number    | {{audiosamplerate}}        |
| Channel configuration   | cc     |  opt     |   S       |  String    | {{channelconfiguration}}   |
| Display width           | dw     |  opt     |   S       |  Number    | {{displaywidth}}           |
| Display height          | dh     |  opt     |   S       |  Number    | {{displayheight}}          |
| Language                | la     |  opt     |   S       |  String    | {{language}}               |


Required: 'yes' indicates a mandatory field, 'opt' indicates an optional field

Location: 'R' - the field is located in the root of the JSON object, 'RT' - the field may be located in either the root or a track object, "S" - the field is located in the Selection Properties object.

### Streaming format {#streamingformat}
A number indicating the streaming format type. Every MoQ Streaming Format normatively referencing this catalog format MUST register itself in the "MoQ Streaming Format Type" table. See {{iana}} for additional details.

### Streaming format version {#streamingformatversion}
A string indicating the version of the streaming format to which this catalog applies. The structure of the version string is defined by the streaming format.

### Tracks {#tracks}
An array of track objects {{trackobject}}.

### Tracks object {#trackobject}
A track object is a collection of fields whose location is specified as 'RT' in Table 1.

### Parent sequence number {#parentsequencenumber}
A number specifying the moq-transport object number from which this catalog represents a delta update. See {{deltaupdate}} for additional details. Absence of this parent sequence number indicates that this catalog is independent and completely describes the content available in the broadcast.

### Track namespace {#tracknamespace}
The name space under which the track name is defined. See section 2.3 of {{MoQTransport}}. The track namespace is required to be specified for each track object. If the track namespace is declared in the root of the JSON document, then its value is inherited by all tracks and it does not need to be re-declared within each track object. A namespace declared in a track object overwrites any inherited name space.

### Track name {#trackname}
A string defining the name of the track. See section 2.3 of {{MoQTransport}}

### Packaging {#packaging}
A string defining the type of payload encapsulation. Allowed values are strings as defined in Table 2.

Table 2: Allowed packaging values

| Name            |   Value   |      Draft       |
|:================|:==========|:=================|
| CMAF            | "cmaf"    | See RFC XXXX     |
| LOC             | "loc"     | See RFC XXXX     |



### Track operations {#trackoperations}

Each track description can specify an optional operation value that identifies
the catalog producer's intent. Track operation is a enumeration of values
as defined below.

* Add: Indicates the track is added to the catalog and the consumers of the
 catalog may subscribe to the track.

* Delete: Indicates that media producer is no longer producing media on the
associated track.

A catalog update in which all previously added tracks are deleted SHOULD be interpreted by a subscriber to indicate that the publisher has terminated the broadcast.

Table 3 defines the numerical values for the track operations.

Table 3: Allowed track operations

| Name            | Value | Default value  |
|:================|:======|:===============|
| Add             | 1     |    yes         |
| Delete          | 0     |                |

The default track operation is 'Add'. This value does not need to be declared in the track object.


### Track label {#tracklabel}
A string defining a human-readable label for the track. Examples might be "Overhead camera view" or "Deutscher Kommentar". Note that the {{JSON}} spec requires UTF-8 support by decoders.

### Render group {#rendergroup}
An integer specifying a group of tracks which are designed to be rendered together. Tracks with the same group number SHOULD be rendered simultaneously, are usually time-aligned and are designed to accompany one another. A common example would be tying together audio and video tracks.

### Alternate group {#altgroup}
An integer specifying a group of tracks which are alternate versions of one-another. Alternate tracks represent the same media content, but differ in their selection properties. Alternate tracks SHOULD have matching framerate {{framerate}} and media time sequences. A subscriber SHOULD only subscribe to one track from a set of tracks specifying the same alternate group number. A common example would be a set video tracks of the same content offered in alternate bitrates.

### Dependencies {#dependencies}
Certain tracks may depend on other tracks for decoding. Dependencies holds an array of track names {{trackname}} on which the current track is dependent. Since only the track name is signaled, the namespace of the dependencies is assumed to match that of the track declaring the dependencies.

### Initialization data {#initdata}
A string holding Base64 [BASE64] encoded initialization data for the track.

### Initialization track {#inittrack}
A string specifying the track name of another track which holds initialization data for the current track. Initialization tracks MUST NOT be added to the tracks array {{tracks}}. They are referenced only via the initialization track field of the track which they initialize.

### Selection parameters {#selectionparameters}
An object holding a series of name/value pairs which a subscriber can use to select tracks for subscription. If present, the selection parameters object MUST NOT be empty. Any selection parameters declared at the root level are inherited by all tracks. A selection parameters object may exist at both the root and track level. Any declaration of a selection parameter at the track level overrides the inherited root value.

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

### Audio sample rate {#audiosamplerate}
The number of audio frame samples per second. This property SHOULD only accompany audio codecs.

### Width {#width}
A number expressing the encoded width of the track content in pixels.

### Height {#height}
A number expressing the encoded height of the video frames in pixels.

### Channel configuration {#channelconfiguration}
A string specifying the audio channel configuration. This property SHOULD only accompany audio codecs. A string is used in order to provide the flexibility to describe complex channel configurations for multi-channel and Next Generation Audio schemas.

### Display width {#displaywidth}
A number expressing the intended display width of the track content in pixels.

### Display height {#displayheight}
A number expressing the intended display height of the track content in pixels.

### Language {#language}
A string defining the dominant language of the track. The string MUST be one of the standard Tags for Identifying Languages as defined by [LANG].

### Temporal ID {#temporalid}
A number identifying the temporal layer/sub-layer encoding of the track, starting with 0 for the base layer, and increasing with higher temporal fidelity.

### Spatial ID {#spatialid}
A number identifying the spatial layer encoding of the track, starting with 0 for the base layer, and increasing with higher fidelity.

### Time aligned  {#timealigned}
An array of track names intended for synchronized playout. An example would be audio and video media synced for playout in a conference setting.

## Catalog Delta Updates {#deltaupdate}
A catalog might contain incremental changes. This is a useful property if many tracks may be initially declared but then there are small changes to a subset of tracks. The producer can issue a delta update to describe these small changes. Changes are described incrementally, meaning that a delta-update can itself depend on a previous delta update.

The following rules MUST be followed in processing delta updates:

* If a catalog is received without the parent sequence number field {{parentsequencenumber}} defined, then it is an independent catalog and no delta update processing is required.
* If a catalog is received with a parent sequence number field present, then the content of the catalog MUST be parsed as if the catalog contents had been added to the contents received on the referenced moq-transport object. Newer field definitions overwrite older field definitions.
* Track namespaces may not be changed across delta updates.
* Contents of the track selection properties object may not be varied across updates. To adjust a track selection property, the track must first be removed and then added with the new selection properties and a different name.
* Track names may not be changed across delta updates. To change a track name, remove the track and then add a new track with the new name and matching properties.


## Catalog Examples

The following section provides non-normative JSON examples of various catalogs compliant with this draft.


### Time-aligned Audio/Video Tracks with single quality

This example shows catalog for a media producer capable of sending LOC packaged, time-aligned audio and video tracks.

~~~json
{
  "f": 1,
  "v": "0.2",
  "ns": "conference.example.com/conference123/alice",
  "p": "loc",
  "gr": 1,
  "tracks": [
    {
      "n": "video",
      "sp":{"c":"av01.0.08M.10.0.110.09","wd":1920,"ht":1080,"fr":30,"br":1500000}
    },
    {
      "n": "audio",
      "sp":{"c":"opus","sr":48000,"cc":"2","br":32000}
    }
   ]
}

~~~


### Simulcast video tracks - 3 alternate qualities along with audio


This example shows catalog for a media producer capable
of sending 3 time-aligned video tracks for high definition, low definition and
medium definition video qualities, along with an audio track.


~~~json
{
  "f": 1,
  "v": "0.2",
  "ns": "conference.example.com/conference123/alice",
  "sp": {"c":"av01"},
  "gr": 1,
  "tracks":[
    {
      "n": "hd",
      "sp": {"wd":1920,"ht":1080,"br":5000000,"fr":30},
      "alt":1
    },
    {
      "n": "md",
      "sp": {"wd":720,"ht":640,"br":3000000,"fr":30},
      "alt":1
    },
    {
      "n": "sd",
      "sp": {"wd":192,"ht":144,"br":500000,"fr":30},
      "alt":1
    },
    {
      "n": "audio",
      "sp":{"c":"opus","sr":48000,"cc":"2","br":32000},
    }
   ]
}
~~~

### Delta update adding a track

This example shows catalog for the media producer adding a slide track to an established video conference

~~~json
{
  "psn":0,
  "tracks": [
    {
      "n": "slides",
      "sp":{"c":"av01.0.08M.10.0.110.09","wd":1920,"ht":1080,"fr":15,"br":750000},
      "gr":1
    }
   ]
}

~~~

### Delta update removing a track

This example shows delat catalog update for a media producer removing a slide track from an established video conference

~~~json
{
  "psn":1,
  "tracks": [
    {
      "n": "slides",
      "op": 0
    }
   ]
}

~~~

### Delta update removing all tracks and terminating the broadcast

This example shows a delta catalog update for a media producer removing all tracks and terminating the broadcast.

~~~json
{
  "psn":2,
  "op": 0,
  "tracks": [{"n": "audio"},{"n": "video"},{"n": "slides"}]
}

~~~

### CMAF Tracks with multiple qualities of audio and video

This example shows catalog for a sports broadcast sending time-aligned audio and video tracks using CMAF packaging. Init segments are delivered as separate tracks.

~~~json
{
  "f": 1,
  "v": "0.2",
  "ns": "sports.example.com/games/08-08-23/12345",
  "p": "cmaf",
  "gr":1,
  "tracks": [
    {
      "n": "video_4k",
      "sp":{"c":"avc1.640033","mt":"video/mp4","wd":3840,"ht":2160,"fr":30,"br":14931538},
      "init":"init_video_4k",
      "alt": 1
    },
    {
      "n": "video_1080",
      "sp":{"c":"avc1.640028","mt":"video/mp4","wd":1920,"ht":1080,"fr":30,"br":9914554},
      "init":"init_video_1080",
      "alt": 1
    },
    {
      "n": "video_720",
      "sp":{"c":"avc1.64001f","mt":"video/mp4","wd":1280,"ht":720,"fr":30,"br":4952892},
      "init":"init_video_720",
      "alt": 1
    },
    {
      "n": "audio_aac",
      "sp":{"c":"mp4a.40.5","mt":"audio/mp4","sr":48000,"cc":"2","br":67071},
      "init":"init_audio_aac",
      "alt": 2
    },
    {
      "n": "audio_ec3",
      "sp":{"c":"ec-3","mt":"audio/mp4","sr":48000,"cc":"F801","br":256000},
      "init":"init_audio_ec3",
      "alt": 2
    }
   ]
}
~~~

### Mixed format example - CMAF and LOC packaging in the same catalog

This example shows catalog describing a broadcast with CMAF packaged video and LOC packaged audio.

~~~json
{
  "f": 1,
  "v": "0.2",
  "ns": "output.example.com/event/12345",
  "gr":1
  "tracks": [
    {
      "n": "video0",
      "sp":{"c":"avc1.64001f","mt":"video/mp4","wd":1280,"ht":720,"fr":30,"br":4952892},
      "init":"init_video_720",
      "p":"loc",
    },
    {
      "n": "audio",
      "sp":{"c":"opus","sr":48000,"cc":"2","br":32000},
      "p": "loc",
    }
   ]
}
~~~


### CMAF Tracks with inband init segments

This example shows catalog for a sports broadcast sending time-aligned audio and video tracks using CMAF packaging. Init segments are delivered as inband data.

~~~json
{
  "f": 1,
  "v": "0.2",
  "ns": "sports.example.com/games/08-08-23/12345",
  "p": "cmaf",
  "gr":1,
  "tracks": [
    {
      "n": "video_1080",
      "sp":{"c":"avc1.640028","mt":"video/mp4","wd":1920,"ht":1080,"fr":30,"br":9914554},
"ind":"AAAAGGZ0eXBpc282AAAAAWlzbzZkYXNoAAAARWZyZWVJc29NZWRpYSBGaWxlIFByb2R1Y2VkIHdpdGggR1BBQyAxLjAuMS1yZXYwLWdkODUzOGU4YS1tYXN0ZXIAAAADLW1vb3YAAABsbXZoZAAAAADfdnly33Z5cgABX5AAAAAAAAEAAAEAAAAAAAAAAAAAAAABAAAAAAAAAAAAAAAAAAAAAQAAAAAAAAAAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAIAAAA4bXZleAAAABBtZWhkAAAAAAPwOXgAAAAgdHJleAAAAAAAAAABAAAAAQAAA+gAAAAAAAEAAAAAAkd0cmFrAAAAXHRraGQAAAAB33Y1w992eXIAAAABAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAEAAAAAAAAAAAAAAAAAAAABAAAAAAAAAAAAAAAAAABAAAAABQAAAAIWAAAAAAAkZWR0cwAAABxlbHN0AAAAAAAAAAEAAAAAAAAD6AABAAAAAAG/bWRpYQAAACBtZGhkAAAAAN92NcPfdnlyAABdwAAAAAAVxwAAAAAAQGhkbHIAAAAAAAAAAHZpZGUAAAAAAAAAAAAAAAAfTWFpbmNvbmNlcHQgVmlkZW8gTWVkaWEgSGFuZGxlcgAAAVdtaW5mAAAAFHZtaGQAAAABAAAAAAAAAAAAAAAkZGluZgAAABxkcmVmAAAAAAAAAAEAAAAMdXJsIAAAAAEAAADkc3RibAAAAJhzdHNkAAAAAAAAAAEAAACIYXZjMQAAAAAAAAABAAAAAAAAAAAAAAAAAAAAAAUAAhYASAAAAEgAAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABj//wAAADJhdmNDAU1AH//hABpnTUAfllKAoAi/NNQYGBkAAAMAAQAAAwAwhAEABWjpCTUgAAAAEHN0dHMAAAAAAAAAAAAAABBzdHNjAAAAAAAAAAAAAAAUc3RzegAAAAAAAAAAAAAAAAAAABBzdGNvAAAAAAAAAAAAAAAzaGRscgAAAAAAAAAAYWxpcwAAAAAAAAAAAAAAAEFsaWFzIERhdGEgSGFuZGxlcgAAAAA6dWR0YQAAABepVElNAAsAADAwOjAwOjAwOjAwAAAADqlUU0MAAgAAMjQAAAANqVRTWgABAAAx"
    },
    {
      "n": "audio_aac",
      "sp":{"c":"mp4a.40.5","mt":"audio/mp4","sr":48000,"cc":"2","br":67071},
"ind":"AAAAGGZ0eXBpc282AAAAAWlzbzZkYXNoAAAARWZyZWVJc29NZWRpYSBGaWxlIFByb2R1Y2VkIHdpdGggR1BBQyAxLjAuMS1yZXYwLWdkODUzOGU4YS1tYXN0ZXIAAAACzG1vb3YAAABsbXZoZAAAAADfdnly33Z5cgABX5AAAAAAAAEAAAEAAAAAAAAAAAAAAAABAAAAAAAAAAAAAAAAAAAAAQAAAAAAAAAAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAMAAAA4bXZleAAAABBtZWhkAAAAAAPwSAAAAAAgdHJleAAAAAAAAAACAAAAAQAABAAAAAAAAgAAAAAAAeZ0cmFrAAAAXHRraGQAAAAB33Y1w992eXIAAAACAAAAAAAAAAAAAAAAAAAAAAAAAAABAAAAAAEAAAAAAAAAAAAAAAAAAAABAAAAAAAAAAAAAAAAAABAAAAAAAAAAAAAAAAAAAGCbWRpYQAAACBtZGhkAAAAAN92NcPfdnlyAAC7gAAAAAAVxwAAAAAARGhkbHIAAAAAAAAAAHNvdW4AAAAAAAAAAAAAAAAjTWFpbmNvbmNlcHQgTVA0IFNvdW5kIE1lZGlhIEhhbmRsZXIAAAEWbWluZgAAABBzbWhkAAAAAAAAAAAAAAAkZGluZgAAABxkcmVmAAAAAAAAAAEAAAAMdXJsIAAAAAEAAACnc3RibAAAAFtzdHNkAAAAAAAAAAEAAABLbXA0YQAAAAAAAAABAAAAAAAAAAAAAgAQAAAAALuAAAAAAAAnZXNkcwAAAAADGQAAAAQRQBUABgAACBXTAATXtwUCEZAGAQIAAAAQc3R0cwAAAAAAAAAAAAAAEHN0c2MAAAAAAAAAAAAAABRzdHN6AAAAAAAAAAAAAAAAAAAAEHN0Y28AAAAAAAAAAAAAADNoZGxyAAAAAAAAAABhbGlzAAAAAAAAAAAAAAAAQWxpYXMgRGF0YSBIYW5kbGVyAAAAADp1ZHRhAAAAF6lUSU0ACwAAMDA6MDA6MDA6MDAAAAAOqVRTQwACAAAyNAAAAA2pVFNaAAEAADE="
    }
   ]
}
~~~


# Security Considerations

The catalog contents MAY be encrypted. The mechanism of encryption and the signalling of the keys are left to the Streaming Format referencing this catalog format.

# IANA Considerations {#iana}

This section details how the MoQ Streaming Format Type can be registered. The type registry can be updated by incrementally expanding the type space, i.e., by allocating and reserving new type identifiers. As per [RFC8126], this section details the creation of the "MoQ Streaming Format Type" registry.

## MoQ Streaming Format Type Registry

This document creates a new registry, "MoQ Streaming Format Type". The registry policy is "RFC Required". The Type value is 2 octets. The range is 0x0000-0xFFFF.The initial entry in the registry is:

         +--------+-------------+----------------------------------+
         | Type   |     Name    |            RFC                   |
         +--------+-------------+----------------------------------+
         | 0x0000 |   Reserved  |                                  |
         +--------+-------------+----------------------------------+

Every MoQ streaming format draft normatively referencing this catalog format MUST register itself a unique type identifier.

# Acknowledgments
{:numbered="false"}
Suhas Nandakumar & Mo Zanaty - this [draft](https://github.com/suhasHere/moq-drafts/tree/master/moq-catalog).
The IETF MoQ mailing lists and discussion groups.
