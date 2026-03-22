# Agentic Audiences in OpenRTB

**Sponsors**: LiveRamp, IAB Tech Lab

## Version History

| Date | Version | Comments |
| :-- | :-- | :-- |
| March 2026 | 1.0 | Initial community extension for OpenRTB 2.x |

## Overview

Agentic Audiences (formerly the User Context Protocol/UCP) is an open standard that defines how intelligent agents in advertising exchange signals—identity, contextual, and reinforcement information—that represent a consumer's real-time intent and response to advertising. Thanks to a [donation from LiveRamp](https://liveramp.com/blog/accelerating-ai-with-standards-governance/), Agentic Audiences has been added to IAB Tech Lab's open-source agentic initiative and is maintained in the [IABTechLab/agentic-audiences](https://github.com/IABTechLab/agentic-audiences) repository.

Rather than exchanging raw data points or text descriptions, Agentic Audiences leverages **embeddings**—compact, learned vector representations of 256–1024 dimensions that efficiently encode complex signals in a privacy-preserving, interoperable format. This approach enables the sub-100ms response times required for real-time bidding while supporting transfer learning across systems.

## Background

As the industry transitions into the agentic web, where autonomous buyer, seller, and measurement agents powered by AI/ML models act on behalf of users and organizations, advertising decisions increasingly rely on these models to process billions of signals per second. Agentic Audiences defines a protocol for agents to exchange embeddings that encode:

- **Identity signals** (type 1): Who the user is (hashed identifiers, segments, behavioral history)
- **Contextual signals** (type 2): What the user is doing right now (page content, time of day, device, engagement patterns)
- **Reinforcement signals** (type 3): How users respond to advertising (impressions, clicks, conversions, engagement metrics)

Agentic Audiences is part of IAB Tech Lab's broader [Agentic Advertising Management Protocols (AAMP)](https://iabtechlab.com/standards/aamp-agentic-advertising-management-protocols/) initiative and complements the [Agentic Real Time Framework (ARTF)](https://iabtechlab.com/agentic-rtb-framework-specification-v1-0/).

## Goal

The goal of this community extension is to define how Agentic Audiences embeddings are conveyed in OpenRTB bid requests via the standard `user.data` structure. Each configured provider (e.g., identity platform, data provider) supplies its own Data object with `segment` entries containing vector embeddings and metadata. This enables interoperability between systems that produce and consume embeddings while maintaining privacy and performance requirements.

## Specification

### Object: `BidRequest.user.data`

This extension uses the standard OpenRTB `user.data` array. Each Agentic Audiences provider supplies one Data object. The `name` field identifies the provider (e.g., `live_ramp`, `optable`). The `segment` array contains Agentic Audiences embedding entries.

### Object: Agentic Audiences Segment

When a Data object contains Agentic Audiences embeddings, each element in its `segment` array has the following structure:

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
      <td>float array</td>
      <td>Vector embedding. Typically 256–1024 dimensions.</td>
    </tr>
    <tr>
      <td><code>model</code></td>
      <td>string</td>
      <td>Model identifier that produced the embedding (e.g., "sbert-mini-ctx-001", "optable-embed-v1").</td>
    </tr>
    <tr>
      <td><code>dimension</code></td>
      <td>number</td>
      <td>Vector dimension (length of the vector array).</td>
    </tr>
    <tr>
      <td><code>type</code></td>
      <td>number array</td>
      <td>Embedding type(s): 1 = identity, 2 = contextual, 3 = reinforcement. A single vector may represent multiple types.</td>
    </tr>
  </tbody>
</table>

#### Embedding Type Enumeration

| Value | Type | Description |
| :-- | :-- | :-- |
| 1 | identity | Who the user is |
| 2 | contextual | What the user is doing right now |
| 3 | reinforcement | How the user responds to advertising |

### Provider Identification

The Data object's `name` field identifies the provider in snake_case (e.g., `liveRamp` → `live_ramp`, `optable` → `optable`). Common providers include:

| Provider | Example name value |
| :-- | :-- |
| LiveRamp | live_ramp |
| Optable | optable |

## Example Bid Requests

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
            "vector": [0.1, -0.2, 0.3, 0.4, -0.5],
            "model": "sbert-mini-ctx-001",
            "dimension": 5,
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

## Implementation Notes

- **Vector dimensions**: Embedding vectors are typically 256–1024 dimensions. Implementers should agree on dimension and vector-space alignment when interoperating across providers.
- **Model identifier**: The `model` field enables buyers to understand which embedding space a vector belongs to. Similarity computations are meaningful only within compatible vector spaces.
- **Type array**: A segment entry may have `type: [1, 2]` when a single vector encodes both identity and contextual signals.
- **Privacy**: Embeddings encode semantic meaning without exposing raw user data. Implementers must ensure appropriate consent and data handling policies are followed.
- **Performance**: Compact vector representation supports sub-100ms inference, critical for real-time bidding.
- **Storage**: Publishers and identity providers may store Agentic Audiences data in browser storage (localStorage or cookie) as base64-encoded JSON containing an `entries` array. Each entry must include ver, vector, model, dimension, and type.

## References

- [Agentic Audiences | IAB Tech Lab](https://iabtechlab.com/standards/agentic-audiences/)
- [Agentic Audiences GitHub Repository](https://github.com/IABTechLab/agentic-audiences)
- [Prebid.js Agentic Audience Adapter (PR #14626)](https://github.com/prebid/Prebid.js/pull/14626)
- [Agentic Real Time Framework Specification v1.0](https://iabtechlab.com/agentic-rtb-framework-specification-v1-0/)
