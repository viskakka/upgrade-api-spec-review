# GET_cxue-platform-mgr_api_v1_upgrades_import

**Recommendations:** 4
**Generated:** 2025-12-09 23:25:05

[← Back to Overview](README.md)

---

## Legend

| Indicator | Priority | Severity | Meaning |
|-----------|----------|----------|---------|
| 🔴 | P0, P1 | Critical, High | Stop-ship requirement or must-have for API quality |
| 🟠 | P2 | Medium | Should-have best practice for consistency |
| 🟡 | P3 | Low | Nice-to-have enhancement or partial compliance |
| 🟢 | - | - | Pass - meets requirements |

---

## Table of Contents

- [P1] 🔴 [Issue 1: SEMANTIC.NAMING.FIELD.03-02 - Timestamp Field Naming Inconsistency](#issue-1-semanticnamingfield03-02---timestamp-field-naming-inconsistency)
- [P1] 🔴 [Issue 2: SEMANTIC.NAMING.FIELD.03-05 - Duration Field Missing Explicit Unit](#issue-2-semanticnamingfield03-05---duration-field-missing-explicit-unit)
- [P0] 🟡 [Issue 3: SEMANTIC.NAMING.FIELD.01-10 - Insufficient Field Descriptions](#issue-3-semanticnamingfield01-10---insufficient-field-descriptions)
- [P1] 🟡 [Issue 4: SEMANTIC.NAMING.FIELD.04-12 - Missing Unit Annotations on Quantity Fields](#issue-4-semanticnamingfield04-12---missing-unit-annotations-on-quantity-fields)

---

## 🔴 Issue 1: SEMANTIC.NAMING.FIELD.03-02 - Timestamp Field Naming Inconsistency

**Severity:** HIGH  
**Status:** FAIL  
**Category:** `paths./cxue-platform-mgr/api/v1/upgrades/import.get.responses.200.content.application/json.schema.properties - multiple timestamp fields (createdAt, updatedAt, startTime, endTime)`

### Overview
| | |
|---|---|
| **Priority Reasoning** | MUST - Timestamp field naming consistency is foundational to API quality and developer experience. |
| **Issue** | Timestamp fields use inconsistent naming patterns (startTime/endTime vs. {qualifier}At convention) |
| **Rule** | SEMANTIC.NAMING.FIELD.03-02 - https://developer.cisco.com/api-guidelines/semantic-naming-field-03/#timestamp-fields-date--time |
| **Breaking Change** | true |
| **Location** | StageInfo schema properties in components.schemas and paths./cxue-platform-mgr/api/v1/upgrades/import.get.responses.200 |

### Analysis

#### Issue Reasoning
The StageInfo schema (used in completedStages and currentStage arrays) contains timestamp fields 'startTime' and 'endTime' that violate the {qualifier}At naming convention specified in SEMANTIC.NAMING.FIELD.03-02. Per the rule, timestamp fields representing when events occurred (stage started/completed) should follow the {qualifier}At pattern (e.g., startedAt, completedAt). The rule explicitly states this pattern applies to "action timestamps" and similar operational timestamps. Additionally, the 'elapsedTime' field lacks explicit unit specification; per SEMANTIC.NAMING.FIELD.03-05, numeric durations should include units (elapsedSeconds).

#### Issue Impact
Developers consuming this API will encounter inconsistent timestamp naming conventions within a single API response object. The BundleStatus schema correctly uses 'createdAt' and 'updatedAt', but the nested StageInfo objects use 'startTime' and 'endTime', creating a confusing and non-uniform pattern. This reduces API usability and violates Cisco API naming standards.

#### Breaking Change Reasoning
💥 Renaming fields from 'startTime'/'endTime' to 'startedAt'/'endedAt' is a breaking change. Existing API clients have hardcoded field references in JSON deserialization logic (e.g., payload.startTime, payload.endTime). When fields are renamed, clients attempting to access the old field names will receive undefined/null values or throw exceptions. Client code must be updated to reference the new field names, requiring version bumps or dual-field support during deprecation periods.

### Implementation

#### Current Implementation
```json
"startTime": {
  "anyOf": [
    {"type": "string"},
    {"type": "null"}
  ],
  "format": "date-time",
  "title": "Starttime",
  "description": "ISO timestamp when stage started"
},
"endTime": {
  "anyOf": [
    {"type": "string"},
    {"type": "null"}
  ],
  "format": "date-time",
  "title": "Endtime",
  "description": "ISO timestamp when stage ended"
},
"elapsedTime": {
  "anyOf": [
    {"type": "integer"},
    {"type": "null"}
  ],
  "title": "Elapsedtime",
  "description": "Elapsed time in seconds"
}
```

#### Recommended Implementation
```json
"startedAt": {
  "anyOf": [
    {"type": "string"},
    {"type": "null"}
  ],
  "format": "date-time",
  "title": "StartedAt",
  "description": "ISO timestamp when stage started"
},
"completedAt": {
  "anyOf": [
    {"type": "string"},
    {"type": "null"}
  ],
  "format": "date-time",
  "title": "CompletedAt",
  "description": "ISO timestamp when stage ended"
},
"elapsedSeconds": {
  "anyOf": [
    {"type": "integer"},
    {"type": "null"}
  ],
  "title": "ElapsedSeconds",
  "description": "Elapsed time in seconds",
  "x-unit": "s"
}
```

---

## 🔴 Issue 2: SEMANTIC.NAMING.FIELD.03-05 - Duration Field Missing Explicit Unit

**Severity:** HIGH  
**Status:** FAIL  
**Category:** `components.schemas.StageInfo.properties.elapsedTime (used in completedStages and currentStage arrays)`

### Overview
| | |
|---|---|
| **Priority Reasoning** | MUST - Duration field naming with explicit units is a key convention for clarity. |
| **Issue** | Duration field 'elapsedTime' lacks explicit unit specification in the field name |
| **Rule** | SEMANTIC.NAMING.FIELD.03-05 - https://developer.cisco.com/api-guidelines/semantic-naming-field-03/#duration-fields |
| **Breaking Change** | true |
| **Location** | components.schemas.StageInfo.properties.elapsedTime |

### Analysis

#### Issue Reasoning
The StageInfo schema contains a field 'elapsedTime' that violates SEMANTIC.NAMING.FIELD.03-05. The rule explicitly requires: 'APIs using numerical values for durations must include the unit explicitly in the field name using the {qualifier}{Unit} pattern.' Examples provided include 'timeoutSeconds', 'retentionHours', 'maxSessionDurationMinutes'. The field 'elapsedTime' uses generic 'Time' as the unit indicator rather than the specific 'Seconds' unit. While the description states 'Elapsed time in seconds', the field name should communicate this directly per the rule. The correct naming would be 'elapsedSeconds'.

#### Issue Impact
API consumers reading the field name 'elapsedTime' cannot immediately determine the time unit without consulting documentation. This creates potential for misinterpretation (is it milliseconds? seconds? minutes?) and violates the principle of explicit API contracts. Developers might incorrectly assume the value represents milliseconds (a common assumption) and calculate incorrect durations, leading to bugs in client code.

#### Breaking Change Reasoning
💥 Renaming 'elapsedTime' to 'elapsedSeconds' is a breaking change. Existing API clients have JSON deserialization logic that references the field by name (e.g., `stage.elapsedTime`, `response['elapsedTime']`). When the field name changes, these references fail, resulting in undefined values or property access errors. Client code must be updated to reference the new field name, requiring version updates and client-side deployment.

### Implementation

#### Current Implementation
```json
"elapsedTime": {
  "anyOf": [
    {"type": "integer"},
    {"type": "null"}
  ],
  "title": "Elapsedtime",
  "description": "Elapsed time in seconds"
}
```

#### Recommended Implementation
```json
"elapsedSeconds": {
  "anyOf": [
    {"type": "integer"},
    {"type": "null"}
  ],
  "title": "ElapsedSeconds",
  "description": "Elapsed time in seconds"
}
```

---

## 🟡 Issue 3: SEMANTIC.NAMING.FIELD.01-10 - Insufficient Field Descriptions

**Severity:** CRITICAL  
**Status:** PARTIAL  
**Category:** `components.schemas.BundleStatus.properties and components.schemas.StageInfo.properties - multiple fields`

### Overview
| | |
|---|---|
| **Priority Reasoning** | MUST - Field descriptions are critical for API usability and reduce ambiguity in implementation. |
| **Issue** | Multiple fields have minimal or unclear descriptions that do not sufficiently explain purpose, values, or usage context |
| **Rule** | SEMANTIC.NAMING.FIELD.01-10 - https://developer.cisco.com/api-guidelines/semantic-naming-field-01/#field-description-guidelines |
| **Breaking Change** | false |
| **Location** | components.schemas.BundleStatus.properties and components.schemas.StageInfo.properties |

### Analysis

#### Issue Reasoning
Multiple fields in the API specification have descriptions that are too brief or vague per SEMANTIC.NAMING.FIELD.01-10 requirements. Examples: (1) bundleId description 'Unique identifier for the bundle' is self-evident and lacks context about its format/lifecycle persistence; (2) fileSource description 'Source of the file (remote_host:path or local upload)' mentions the format but doesn't explain what remote_host:path specifically means or how it differs from local upload; (3) status field description 'Current bundle status' doesn't enumerate the valid status values beyond the enum constraint; (4) failed field description 'Whether the stage failed' doesn't clarify when this flag is set relative to the status field or the stage lifecycle. Per the rule, descriptions must be 'clear and concise' and must 'resolve ambiguity' - current descriptions fail this test.

#### Issue Impact
Developers using this API will struggle to understand field semantics and may misinterpret data values. For example, a developer might not understand that fileSource format is 'ip:path' or might confuse the boolean 'failed' field with the 'status' field's failure indication. This creates integration bugs and increases support burden.

#### Breaking Change Reasoning
🛡️ Adding or enhancing field descriptions in the OpenAPI specification is non-breaking. Field descriptions are documentation metadata only - they do not affect the JSON serialization, HTTP behavior, or data contract. Existing API clients continue to function identically. This change only improves developer understanding without modifying any structural or behavioral aspects of the API.

### Implementation

#### Current Implementation
```json
"bundleId": {
  "type": "string",
  "format": "uuid",
  "title": "Bundleid",
  "description": "Unique identifier for the bundle"
},
"fileName": {
  "anyOf": [{"type": "string"}, {"type": "null"}],
  "title": "Filename",
  "description": "Name of the upgrade bundle file"
},
"fileSource": {
  "anyOf": [{"type": "string"}, {"type": "null"}],
  "title": "Filesource",
  "description": "Source of the file (remote_host:path or local upload)"
},
"status": {
  "anyOf": [{
    "type": "string",
    "enum": ["inProgress", "completed", "failed"],
    "title": "StageStatus",
    "description": "Enum for bundle processing stage statuses."
  }, {"type": "null"}],
  "description": "Current bundle status"
},
"failed": {
  "type": "boolean",
  "title": "Failed",
  "description": "Whether the stage failed",
  "default": false
}
```

#### Recommended Implementation
```json
"bundleId": {
  "type": "string",
  "format": "uuid",
  "title": "Bundleid",
  "description": "Unique identifier (UUID v4) for the upgrade bundle. This identifier persists across the entire bundle lifecycle and is used to correlate import progress with backend processing."
},
"fileName": {
  "anyOf": [{"type": "string"}, {"type": "null"}],
  "title": "Filename",
  "description": "Name of the upgrade bundle file as provided during upload. May include file extension (e.g., bundle-v2.0.zstd). Null if bundle import has not yet retrieved file metadata."
},
"fileSource": {
  "anyOf": [{"type": "string"}, {"type": "null"}],
  "title": "Filesource",
  "description": "Origin of the upgrade bundle file. Format: 'remote_host:path' for remote sources (e.g., '192.168.1.1:/upgrades/bundle.zstd') or 'local upload' for files uploaded directly via multipart form data. Null if source not yet determined."
},
"status": {
  "anyOf": [{
    "type": "string",
    "enum": ["inProgress", "completed", "failed"],
    "title": "StageStatus",
    "description": "Overall bundle processing status: 'inProgress' indicates active import, 'completed' indicates all stages finished successfully, 'failed' indicates an error occurred during processing."
  }, {"type": "null"}],
  "description": "Current bundle import status. Transitions from inProgress to either completed or failed."
},
"failed": {
  "type": "boolean",
  "title": "Failed",
  "description": "Indicates whether the stage encountered an error. Set to true when a stage completes with status='failed', false for successful or in-progress stages. Used for quick error detection without parsing status enum.",
  "default": false
}
```

---

## 🟡 Issue 4: SEMANTIC.NAMING.FIELD.04-12 - Missing Unit Annotations on Quantity Fields

**Severity:** HIGH  
**Status:** PARTIAL  
**Category:** `components.schemas.StageInfo.properties.elapsedTime and components.schemas.BundleStatus.properties.fileSize`

### Overview
| | |
|---|---|
| **Priority Reasoning** | MUST - Unit annotations on quantity fields are mandatory per explicit requirement language. |
| **Issue** | Numeric quantity fields (elapsedTime, fileSize) lack schema-level unit annotations |
| **Rule** | SEMANTIC.NAMING.FIELD.04-12 - https://developer.cisco.com/api-guidelines/semantic-naming-field-04/#unit-annotation-requirements |
| **Breaking Change** | false |
| **Location** | components.schemas.StageInfo.properties.elapsedTime and components.schemas.BundleStatus.properties.fileSize |

### Analysis

#### Issue Reasoning
The API specification includes numeric quantity fields that lack mandatory schema-level unit annotations as required by SEMANTIC.NAMING.FIELD.04-12. The rule states: 'All quantity fields in API specifications must include explicit unit information through schema-level unit annotations.' Current fields: (1) 'elapsedTime' is documented as 'Elapsed time in seconds' but contains no x-unit annotation; (2) 'fileSize' is documented as 'Size of the file in bytes' but contains no x-unit annotation. Units are described in the description text but not in the schema layer where tooling and code generators expect them per the rule requirement.

#### Issue Impact
Without schema-level unit annotations, API tooling cannot reliably parse quantity semantics. SDK generators cannot automatically add unit conversion helpers, API documentation generators cannot display units consistently, and clients must rely on description text rather than structured data. This violates the principle of explicit schema semantics and reduces API quality for automation and tooling.

#### Breaking Change Reasoning
🛡️ Adding schema-level unit annotations (x-unit OpenAPI extension metadata) is non-breaking. The x-unit field is purely metadata that extends the OpenAPI schema without affecting JSON structure, field types, or serialization. Existing API clients parsing responses continue to work identically - they simply ignore the x-unit extension if not understood. This is a documentation/metadata enhancement only.

### Implementation

#### Current Implementation
```json
"elapsedTime": {
  "anyOf": [
    {"type": "integer"},
    {"type": "null"}
  ],
  "title": "Elapsedtime",
  "description": "Elapsed time in seconds"
},
"fileSize": {
  "anyOf": [
    {"type": "integer"},
    {"type": "null"}
  ],
  "title": "Filesize",
  "description": "Size of the file in bytes"
}
```

#### Recommended Implementation
```json
"elapsedTime": {
  "anyOf": [
    {"type": "integer"},
    {"type": "null"}
  ],
  "title": "Elapsedtime",
  "description": "Elapsed time in seconds",
  "x-unit": "s"
},
"fileSize": {
  "anyOf": [
    {"type": "integer"},
    {"type": "null"}
  ],
  "title": "Filesize",
  "description": "Size of the file in bytes",
  "x-unit": "byte"
}
```

---
