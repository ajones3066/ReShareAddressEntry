# Overview

This directory resolves an institution’s ISIL to a canonical organizational record and returns a structured profile of its capabilities, memberships, and trust relationships. Assertions are versioned and provenance-tagged so consumers can distinguish self-declared information from externally validated claims. Responses are tiered: a public summary provides high-level visibility, while authenticated partners can follow links to retrieve more detailed operational profiles.

The service is not a routing or fulfillment system. It functions as a discovery and trust layer that allows clients to understand institutional context—such as network participation or service platforms—without relying on proprietary registries. Time-bounded assertions support change tracking and synchronization, enabling interoperable service discovery while keeping operational control with the institution and making trust decisions explicit.

## How the directory works

The directory exposes information through three trust levels tied to URL paths. Public access (/org/{ISIL}) returns a general organizational summary intended for open discovery. The partner profile (/org/{ISIL}/profiles) returns additional service and participation details once the requester proves identity. Tiered profiles (/org/{ISIL}/profiles/{tier}) disclose progressively more specific stanzas according to the requester’s authorized trust level. Access to non-public tiers requires two elements: a directory-recognized key establishing the requester’s identity and a valid issuer-signed assertion indicating the tier they are entitled to. Together, these determine which portions of the record are disclosed while keeping operational data scoped to appropriate partners.

### Example interaction

#### Public Lookup 

Request

GET /org/US-NNNS
Accept: application/json

Response (200)

{
  "isil": "US-NNNS",
  "name": "Example University Library",
  "summary": {
    "services": ["interlibrary-loan", "shared-print"],
    "memberships": ["NetworkX"]
  },
  "links": {
    "profiles": "/org/US-NNNS/profiles"
  }
}

#### Partner Profile

The client signs the request (or uses mTLS). The directory verifies the key and resolves the requester’s org identity.

Request

GET /org/US-NNNS/profiles
Accept: application/json
Date: Tue, 17 Feb 2026 13:05:00 GMT
Signature: keyId="client-key-123",algorithm="rsa-pss-sha512",headers="(request-target) date",signature="BASE64..."

{
  "isil": "US-NNNS",
  "profile": "partner",
  "stanzas": {
    "serviceEndpoints": {
      "iso18626": "https://api.example.edu/ill"
    },
    "participation": {
      "networks": ["NetworkX"]
    }
  },
  "links": {
    "tiers": ["/org/US-NNNS/profiles/member"]
  }
}

#### Tiered Profile

The client includes:

A signed request (directory key)
An issuer-signed assertion (e.g., consortium membership token)

GET /org/US-NNNS/profiles/member
Accept: application/json
Date: Tue, 17 Feb 2026 13:06:10 GMT
Signature: keyId="client-key-123",algorithm="rsa-pss-sha512",headers="(request-target) date",signature="BASE64..."
X-Assertion: eyJhbGciOiJSUzI1NiIsImlzc3VlciI6Ik5ldHdvcmtYIiwi

Server validation steps

1. Verify request signature → identifies requester org
2. Validate assertion signature + expiry → confirms tier entitlement
3. Evaluate trust policy → determine allowed stanzas

Response (200)

{
  "isil": "US-NNNS",
  "profile": "member",
  "stanzas": {
    "fulfillmentOptions": {
      "loan": true,
      "copy": true
    },
    "retentionSignals": {
      "sharedPrintParticipant": true
    }
  }
}

#### Unauthorized tier attempt

Response (403)

Server validation steps

{
  "error": "insufficient_trust",
  "message": "Valid issuer assertion required for requested tier"
}
