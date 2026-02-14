# scanoss.json Schema

The `scanoss.json` file is used to configure SCANOSS scanning behavior, define policies, and manage Software Bill of Materials (BOM). It is typically placed at the root of a project directory and is automatically detected by SCANOSS tools.

## Table of Contents

- [Overview](#overview)
- [Top-Level Structure](#top-level-structure)
- [Settings](#settings)
  - [BOM Section](#bom-section)
  - [Skip Sizes](#skip-sizes)
- [BOM Operations](#bom-operations)
  - [Include](#include)
  - [Remove](#remove)
  - [Replace](#replace)
  - [Ignore](#ignore)
- [Component Identification](#component-identification)
  - [By Path](#by-path)
  - [By PURL](#by-purl)
- [Policy Settings](#policy-settings)
  - [Copyleft](#copyleft)
  - [Undeclared Components](#undeclared-components)
- [Complete Example](#complete-example)

## Overview

The `scanoss.json` file provides a way to:

- Define which components to include, remove, replace, or ignore in scan results
- Configure file scanning behavior (e.g., skip files by size or extension)
- Set policies for license compliance (e.g., copyleft detection)
- Manage undeclared component policies
- Fine-tune scanning settings

## Top-Level Structure

The top-level structure of the `scanoss.json` file:

```json
{
  "settings": {},
  "bom": {}
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `settings` | object | No | General scanning settings and policies |
| `bom` | object | No | BOM management operations (include, remove, replace, ignore) |

## Settings

The `settings` section contains scanning configuration and policy definitions.

```json
{
  "settings": {
    "bom": {},
    "skip": {},
    "policy": {}
  }
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `bom` | object | No | BOM-related settings |
| `skip` | object | No | File/directory skip settings |
| `policy` | object | No | Policy configuration |

### BOM Section

The `settings.bom` section configures BOM-related settings:

```json
{
  "settings": {
    "bom": {
      "include": [],
      "remove": [],
      "replace": [],
      "ignore": []
    }
  }
}
```

> **Note:** The `settings.bom` section follows the same structure as the top-level `bom` section. See [BOM Operations](#bom-operations) for details on each sub-section.

### Skip Sizes

The `settings.skip` section allows configuring which files to skip during scanning based on size or extensions:

```json
{
  "settings": {
    "skip": {
      "sizes": {
        "min": 0,
        "max": 0
      },
      "extensions": [],
      "patterns": []
    }
  }
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `skip.sizes` | object | No | File size limits for scanning |
| `skip.sizes.min` | integer | No | Minimum file size in bytes. Files smaller than this are skipped |
| `skip.sizes.max` | integer | No | Maximum file size in bytes. Files larger than this are skipped |
| `skip.extensions` | array | No | List of file extensions to skip (e.g., `[".png", ".jpg"]`) |
| `skip.patterns` | array | No | List of glob patterns for files/directories to skip |

**Example:**

```json
{
  "settings": {
    "skip": {
      "sizes": {
        "min": 256,
        "max": 10485760
      },
      "extensions": [".png", ".jpg", ".gif", ".pdf"],
      "patterns": ["test/**", "docs/**"]
    }
  }
}
```

## BOM Operations

The `bom` section (both at the top level and within `settings.bom`) supports four operations for managing how components appear in scan results: **include**, **remove**, **replace**, and **ignore**.

### Include

The `include` section explicitly declares components that should always appear in the BOM, regardless of whether they are detected during scanning:

```json
{
  "bom": {
    "include": [
      {
        "path": "src/lib/crypto",
        "purl": "pkg:github/openssl/openssl"
      }
    ]
  }
}
```

Each entry in the `include` array is an object with:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `path` | string | No | File or directory path relative to the project root |
| `purl` | string | Yes | Package URL (PURL) identifying the component |

### Remove

The `remove` section specifies components that should be removed from scan results. This is useful when a scanner incorrectly identifies a component:

```json
{
  "bom": {
    "remove": [
      {
        "path": "src/utils/helper.c",
        "purl": "pkg:github/someorg/wrongmatch"
      }
    ]
  }
}
```

Each entry in the `remove` array is an object with:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `path` | string | No | File or directory path relative to the project root |
| `purl` | string | Yes | Package URL (PURL) of the component to remove |

### Replace

The `replace` section maps detected components to different components. This is useful when the scanner identifies the correct code but attributes it to the wrong component:

```json
{
  "bom": {
    "replace": [
      {
        "path": "src/lib/json",
        "purl": "pkg:github/wrong/component",
        "replace_with": "pkg:github/correct/component"
      }
    ]
  }
}
```

Each entry in the `replace` array is an object with:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `path` | string | No | File or directory path relative to the project root |
| `purl` | string | Yes | Package URL (PURL) of the component to replace |
| `replace_with` | string | Yes | Package URL (PURL) of the replacement component |

### Ignore

The `ignore` section specifies files or directories that should be completely excluded from scanning:

```json
{
  "bom": {
    "ignore": [
      {
        "path": "test/"
      },
      {
        "path": "docs/"
      }
    ]
  }
}
```

Each entry in the `ignore` array is an object with:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `path` | string | Yes | File or directory path relative to the project root to ignore |

## Component Identification

Components in the `scanoss.json` file can be identified by path, by PURL (Package URL), or by a combination of both.

### By Path

When a `path` is specified, the operation applies to scan results associated with that specific file or directory:

```json
{
  "path": "src/lib/crypto/aes.c"
}
```

Paths are relative to the project root directory where the `scanoss.json` file is located.

### By PURL

A [PURL (Package URL)](https://github.com/package-url/purl-spec) uniquely identifies a software package. PURLs follow the format:

```
pkg:<type>/<namespace>/<name>@<version>?<qualifiers>#<subpath>
```

**Examples:**

| PURL | Description |
|------|-------------|
| `pkg:github/scanoss/scanner` | A GitHub-hosted component |
| `pkg:npm/express` | An npm package |
| `pkg:pypi/requests` | A PyPI package |
| `pkg:maven/org.apache/commons-lang3@3.12.0` | A Maven package with version |
| `pkg:gem/rails@7.0.0` | A RubyGems package with version |

## Policy Settings

The `settings.policy` section defines compliance and governance policies for scanning results.

```json
{
  "settings": {
    "policy": {
      "copyleft": {},
      "undeclared": {}
    }
  }
}
```

### Copyleft

The `copyleft` policy allows you to flag or control behavior when copyleft-licensed components are detected:

```json
{
  "settings": {
    "policy": {
      "copyleft": {
        "exclude": [
          "GPL-2.0-only",
          "GPL-3.0-only"
        ],
        "include": [
          "MIT",
          "Apache-2.0"
        ]
      }
    }
  }
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `copyleft.exclude` | array | No | List of SPDX license identifiers to exclude from copyleft checks |
| `copyleft.include` | array | No | List of SPDX license identifiers to explicitly include in copyleft checks |

### Undeclared Components

The `undeclared` policy flags components detected by the scanner that are not explicitly declared in the project's dependency manifests:

```json
{
  "settings": {
    "policy": {
      "undeclared": {
        "exclude": [
          "pkg:github/scanoss/scanner"
        ]
      }
    }
  }
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `undeclared.exclude` | array | No | List of PURLs to exclude from undeclared component checks |

## Complete Example

Below is a comprehensive example of a `scanoss.json` file that demonstrates all available configuration options:

```json
{
  "settings": {
    "skip": {
      "sizes": {
        "min": 256,
        "max": 10485760
      },
      "extensions": [".png", ".jpg", ".gif", ".ico", ".pdf", ".svg"],
      "patterns": ["test/**", "docs/**", "*.min.js"]
    },
    "policy": {
      "copyleft": {
        "exclude": [
          "LGPL-2.1-only",
          "LGPL-3.0-only"
        ]
      },
      "undeclared": {
        "exclude": [
          "pkg:github/myorg/internal-lib"
        ]
      }
    }
  },
  "bom": {
    "include": [
      {
        "path": "src/lib/crypto",
        "purl": "pkg:github/openssl/openssl@3.0.0"
      }
    ],
    "remove": [
      {
        "path": "src/utils/helper.c",
        "purl": "pkg:github/someorg/false-positive"
      }
    ],
    "replace": [
      {
        "path": "src/lib/json",
        "purl": "pkg:github/wrong/attribution",
        "replace_with": "pkg:github/correct/component@1.0.0"
      }
    ],
    "ignore": [
      {
        "path": "vendor/"
      },
      {
        "path": "third_party/"
      },
      {
        "path": "node_modules/"
      }
    ]
  }
}
```

## File Location

The `scanoss.json` file should be placed at the root of your project directory. SCANOSS tools automatically detect and use this file when running scans from the project directory.

```
my-project/
├── scanoss.json
├── src/
│   └── ...
├── package.json
└── ...
```

## References

- [SCANOSS Python SDK](https://github.com/scanoss/scanoss.py) — Python SDK and CLI tool that uses `scanoss.json`
- [PURL Specification](https://github.com/package-url/purl-spec) — Package URL specification used for component identification
- [SPDX License List](https://spdx.org/licenses/) — License identifiers used in policy configuration
