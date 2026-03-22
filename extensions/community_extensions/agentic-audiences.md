# Agentic Audiences in OpenRTB

**Sponsors**: LiveRamp, Raptive

## Overview

Agentic Audiences (formerly the User Context Protocol/UCP) is an open standard that defines how intelligent agents in advertising exchange signals—identity, contextual, and reinforcement information—that represent a consumer's real-time intent and response to advertising. Agentic Audiences has been added to IAB Tech Lab's open-source agentic initiative and is maintained in the [IABTechLab/agentic-audiences](https://github.com/IABTechLab/agentic-audiences) repository.

Rather than exchanging raw data points or text descriptions, Agentic Audiences leverages **embeddings**—compact, learned vector representations that efficiently encode complex signals in a privacy-preserving, interoperable format. This enables the sub-100ms response times required for real-time bidding.

## Request Change

This community extension defines how Agentic Audiences embeddings are conveyed in OpenRTB bid requests. The extension uses the existing `Data` and `Segment` objects in `BidRequest.user.data`. Each data provider supplies one or more segment entries, where each entry is a vector embedding with metadata describing its type and model.

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

## Storage (Client-Side)

When Agentic Audiences data is sourced from browser storage (localStorage or cookie), the stored value **must be base64-encoded** JSON. The decoded structure must include an `entries` array. Each entry is a segment object with `id`, `name`, and `ext` containing `ver`, `vector`, `model`, `dimension`, and `type`. The wire format sent in the bid request uses these entries as the `segment` array for that provider's `Data` object.

## Implementation Notes

- **Vector dimensions**: Embedding vectors are typically 256–1024 dimensions. Implementers should agree on dimension and vector-space alignment when interoperating across providers.
- **Privacy**: Embeddings encode semantic meaning without exposing raw user data. Implementers must ensure appropriate consent and data handling policies are followed.
- **Model interoperability**: The `model` field enables downstream systems to select compatible embeddings. Similarity computations are meaningful only within the same model/vector space.

## References

- [Agentic Audiences | IAB Tech Lab](https://iabtechlab.com/standards/agentic-audiences/)
- [Agentic Audiences GitHub Repository](https://github.com/IABTechLab/agentic-audiences)
