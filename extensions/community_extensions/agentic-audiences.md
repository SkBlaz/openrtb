# Agentic Audiences in OpenRTB

**Sponsors**: LiveRamp

## Overview

Agentic Audiences (formerly the User Context Protocol/UCP) is an open standard that defines how intelligent agents in advertising exchange signals—identity, contextual, and reinforcement information—that represent a consumer's real-time intent and response to advertising. Agentic Audiences has been added to IAB Tech Lab's open-source agentic initiative and is maintained in the [IABTechLab/agentic-audiences](https://github.com/IABTechLab/agentic-audiences) repository.

Rather than exchanging raw data points or text descriptions, Agentic Audiences leverages **embeddings**—compact, learned vector representations that efficiently encode complex signals in a privacy-preserving, interoperable format. This enables the sub-100ms response times required for real-time bidding.

## Request Change

This community extension defines how Agentic Audiences embeddings are conveyed in OpenRTB bid requests. The extension uses the existing `Data` and `Segment` objects in `BidRequest.user.data`. Each data provider supplies one or more segment entries, where each entry is a vector embedding with metadata describing its type and model.

## Specification

### Object: `BidRequest.user.data`

Per the OpenRTB 2.x API, the `Data` object array in `user.data` allows additional data about the user to be specified. For Agentic Audiences, each provider contributes one `Data` object with:

- **name**: Provider identifier in snake_case (e.g., `live_ramp`, `optable`). Used to identify the source of the embedding data.
- **segment**: Array of Agentic Audiences segment entries (see below).

### Object: `Data.segment` (Agentic Audiences Segment Extension)

When conveying Agentic Audiences embeddings, each element in the `segment` array is an object with the following attributes:

<table>
  <thead>
    <tr>
      <td><strong>Attribute</strong></td>
      <td><strong>Type</strong></td>
      <td><strong>Description</strong></td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>ver</code></td>
      <td>string</td>
      <td>Specification version for embedding schema compatibility (e.g., "1.0").</td>
    </tr>
    <tr>
      <td><code>vector</code></td>
      <td>number array</td>
      <td>Vector embedding as a float array. Typically 256–1024 dimensions.</td>
    </tr>
    <tr>
      <td><code>model</code></td>
      <td>string</td>
      <td>Model identifier that produced the embedding (e.g., "sbert-mini-ctx-001", "optable-embed-v1").</td>
    </tr>
    <tr>
      <td><code>dimension</code></td>
      <td>number</td>
      <td>Vector dimension (length of the <code>vector</code> array).</td>
    </tr>
    <tr>
      <td><code>type</code></td>
      <td>number array</td>
      <td>Embedding type(s): 1 = identity, 2 = contextual, 3 = reinforcement. An entry may encode multiple signal types.</td>
    </tr>
  </tbody>
</table>

### List: Embedding Type Values

<table>
  <thead>
    <tr>
      <th>Value</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>Identity – Who the user is (hashed identifiers, segments, behavioral history)</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Contextual – What the user is doing right now (page content, time of day, device, engagement)</td>
    </tr>
    <tr>
      <td>3</td>
      <td>Reinforcement – How the user responds to advertising (impressions, clicks, conversions, engagement)</td>
    </tr>
  </tbody>
</table>

### Known Providers

When integrating with client-side or server-side sources, the following provider names (snake_case) are commonly used:

| Provider | Data object <code>name</code> | Notes |
| -------- | ---------------------------- | ----- |
| LiveRamp | <code>live_ramp</code> | Default storage key: <code>_lr_agentic_audience_</code> |
| Optable | <code>optable</code> | Default storage key: <code>_optable_agentic_audience_</code> |

Additional providers may use their own identifier in snake_case. Implementations should pass the `name` value unchanged so downstream systems can identify the embedding source.

## Example Bid Request

### Single provider (LiveRamp only)

```json
{
  "id": "req-12345",
  "imp": [{ "id": "1", "banner": { "w": 300, "h": 250 } }],
  "user": {
    "data": [
      {
        "name": "live_ramp",
        "segment": [
          {
            "ver": "1.0",
            "vector": [0.1, -0.2, 0.3],
            "model": "sbert-mini-ctx-001",
            "dimension": 3,
            "type": [1, 2]
          }
        ]
      }
    ]
  }
}
```

### Multiple providers (LiveRamp and Optable)

```json
{
  "id": "req-12345",
  "imp": [{ "id": "1", "banner": { "w": 300, "h": 250 } }],
  "user": {
    "data": [
      {
        "name": "live_ramp",
        "segment": [
          {
            "ver": "1.0",
            "vector": [0.1, -0.2, 0.3],
            "model": "sbert-mini-ctx-001",
            "dimension": 3,
            "type": [1]
          }
        ]
      },
      {
        "name": "optable",
        "segment": [
          {
            "ver": "1.0",
            "vector": [0.5, 0.6, -0.1],
            "model": "optable-embed-v1",
            "dimension": 3,
            "type": [2]
          }
        ]
      }
    ]
  }
}
```

*Note: Embedding vectors in examples are truncated for illustration; actual vectors are typically 256–1024 dimensions.*

## Storage (Client-Side)

When Agentic Audiences data is sourced from browser storage (localStorage or cookie), the stored value **must be base64-encoded** JSON. The decoded structure must include an `entries` array, where each entry has: `ver`, `vector`, `model`, `dimension`, and `type`. The wire format sent in the bid request uses the decoded `entries` as the `segment` array for that provider's `Data` object.

## Implementation Notes

- **Vector dimensions**: Embedding vectors are typically 256–1024 dimensions. Implementers should agree on dimension and vector-space alignment when interoperating across providers.
- **Privacy**: Embeddings encode semantic meaning without exposing raw user data. Implementers must ensure appropriate consent and data handling policies are followed.
- **Model interoperability**: The `model` field enables downstream systems to select compatible embeddings. Similarity computations are meaningful only within the same model/vector space.
- **Related implementations**: [Prebid.js Agentic Audience Adapter](https://github.com/prebid/Prebid.js/pull/14626) (RTD module <code>agenticAudienceAdapter</code>) reads from browser storage and injects Agentic Audiences data into the OpenRTB bid request per this specification.

## References

- [Agentic Audiences | IAB Tech Lab](https://iabtechlab.com/standards/agentic-audiences/)
- [Agentic Audiences GitHub Repository](https://github.com/IABTechLab/agentic-audiences)
- [Prebid.js Agentic Audience Adapter (PR #14626)](https://github.com/prebid/Prebid.js/pull/14626)
