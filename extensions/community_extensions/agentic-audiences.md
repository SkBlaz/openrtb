# Agentic Audiences in OpenRTB

**Sponsors**: LiveRamp, Raptive

## Overview

Agentic Audiences (formerly the User Context Protocol/UCP) is an open standard that defines how intelligent agents in advertising exchange signals—identity, contextual, and reinforcement information—that represent a consumer's real-time intent and response to advertising. Agentic Audiences has been added to IAB Tech Lab's open-source agentic initiative and is maintained in the [IABTechLab/agentic-audiences](https://github.com/IABTechLab/agentic-audiences) repository.

Rather than exchanging raw data points or text descriptions, Agentic Audiences leverages **embeddings**—compact, learned vector representations that efficiently encode complex signals in a privacy-preserving, interoperable format. This enables the sub-100ms response times required for real-time bidding.

## Extension Scope

Embeddings are conveyed in `BidRequest.user.data` using the existing `Data` and `Segment` objects. Each data provider contributes one `Data` object whose `segment` array contains one or more embedding entries. Each segment is a standard OpenRTB Segment with `id` and `name` for identification. The `Segment.ext` object carries the embedding and metadata that downstream systems need to interpret, filter, and use the vector—for example, for similarity matching or model-specific processing.

## Specification

### Object: `BidRequest.user.data`

Per the OpenRTB 2.x API, the `Data` object array in `user.data` allows additional data about the user to be specified. For Agentic Audiences, each provider contributes one `Data` object with:

- **name**: Provider identifier (e.g., `liveramp`). Used to identify the source of the embedding data.
- **segment**: Array of Agentic Audiences segment entries (see below).

### Object: `Data.segment` (Agentic Audiences Segment Extension)

When conveying Agentic Audiences embeddings, each element in the `segment` array is a standard OpenRTB Segment object. The segment uses `id` and `name` for identification, with Agentic Audiences–specific attributes in `ext`:

| Attribute | Type | Description |
| :-- | :-- | :-- |
| `id` | string | ID of the data segment (standard Segment field). |
| `name` | string | Descriptive name for the segment (standard Segment field). |
| `ext` | object | Extension object containing Agentic Audiences attributes (see below). |

### Object: `Segment.ext` (Agentic Audiences)

The `Segment.ext` object extends the standard Segment with Agentic Audiences attributes. It carries the embedding vector and metadata so that buyers can interpret the segment (e.g., by model or signal type), validate compatibility, and use the vector for scoring or similarity operations.

| Attribute | Type | Description |
| :-- | :-- | :-- |
| `ver` | string | Specification version for embedding schema compatibility (e.g., "1.0.0"). |
| `vector` | number array | Vector embedding as a float array. Typically 256–1024 dimensions. |
| `model` | string | Model identifier that produced the embedding (e.g., "sbert-mini-ctx-001"). |
| `dimension` | number | Vector dimension (length of the `vector` array). |
| `type` | number array | Embedding type(s): 1 = identity, 2 = contextual, 3 = reinforcement. An entry may encode multiple signal types. |

### List: Embedding Type Values

| Value | Description |
| :-- | :-- |
| 1 | Identity – Who the user is (hashed identifiers, segments, behavioral history) |
| 2 | Contextual – What the user is doing right now (page content, time of day, device, engagement) |
| 3 | Reinforcement – How the user responds to advertising (impressions, clicks, conversions, engagement) |

## Example Bid Request

### Single provider

```json
{
  "user": {
    "data": [
      {
        "name": "data-provider",
        "segment": [
          {
            "id": "some-id",
            "name": "descriptive-name",
            "ext": {
              "ver": "1.0.0",
              "vector": [0.1, -0.2, 0.3],
              "model": "sbert-mini-ctx-001",
              "dimension": 3,
              "type": [1, 2]
            }
          }
        ]
      }
    ]
  }
}
```

### Multiple providers

```json
{
  "user": {
    "data": [
      {
        "name": "provider-a",
        "segment": [
          {
            "id": "seg-1",
            "name": "identity-contextual",
            "ext": {
              "ver": "1.0.0",
              "vector": [0.1, -0.2, 0.3],
              "model": "sbert-mini-ctx-001",
              "dimension": 3,
              "type": [1, 2]
            }
          }
        ]
      },
      {
        "name": "provider-b",
        "segment": [
          {
            "id": "seg-2",
            "name": "contextual",
            "ext": {
              "ver": "1.0.0",
              "vector": [0.5, 0.6, -0.1],
              "model": "contextual-model-v1",
              "dimension": 3,
              "type": [2]
            }
          }
        ]
      }
    ]
  }
}
```

*Note: Embedding vectors in examples are truncated for illustration; actual vectors are typically 256–1024 dimensions.*

## Implementation Notes

- **Vector dimensions**: Embedding vectors are typically 256–1024 dimensions. Implementers should agree on dimension and vector-space alignment when interoperating across providers.
- **Privacy**: Embeddings encode semantic meaning without exposing raw user data. Implementers must ensure appropriate consent and data handling policies are followed.
- **Model interoperability**: The `model` field enables downstream systems to select compatible embeddings. Similarity computations are meaningful only within the same model/vector space.

## References

- [Agentic Audiences | IAB Tech Lab](https://iabtechlab.com/standards/agentic-audiences/)
- [Agentic Audiences GitHub Repository](https://github.com/IABTechLab/agentic-audiences)
