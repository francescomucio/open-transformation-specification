# The Open Transformation Specification

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

The Open Transformation Specification (OTS) is a community-driven open specification that defines a standard, programming language-agnostic interface description for data transformation pipelines and workflows, including transformations, data quality tests, and user-defined functions (UDFs).

This specification enables **interoperability** between tools and platforms, allowing data transformations to be shared, understood, and executed across different systems without vendor lock-in. By providing a common standard, OTS shifts the data transformation ecosystem from isolated, proprietary tools to an **open core** where tools can seamlessly work together around a shared specification.

This specification allows both humans and computers to discover and understand the capabilities of data transformations without requiring access to source code, additional documentation, or inspection of execution logs. When properly defined via OTS, a consumer can understand and interact with data transformation pipelines with a minimal amount of implementation logic.

## Versions

This repository contains the Markdown sources for all published Open Transformation Specification versions. For release notes and release candidate versions, refer to the [releases page](https://github.com/francescomucio/open-transformation-specification/releases).

- **Current Version**: [v0.2.1](versions/v0.2.1/transformation-specification.md)
- **Previous Versions**: 
  - [v0.2.0](versions/v0.2.0/transformation-specification.md)
  - [v0.1.0](versions/v0.1.0/transformation-specification.md)

## Interoperability and Ecosystem

The Open Transformation Specification enables true **interoperability** in the data transformation space. By defining a common standard, OTS allows:

- **Cross-tool compatibility**: Transformations defined in one tool can be consumed and executed by any OTS-compliant tool
- **Vendor independence**: Avoid lock-in to proprietary formats and tools
- **Ecosystem growth**: An open core standard enables a thriving ecosystem of compatible tools, libraries, and services
- **Seamless integration**: Tools can work together, sharing transformations, tests, and functions without conversion or manual intervention

## Tools and Libraries

The OTS ecosystem is growing with tools and libraries that implement the specification. These tools demonstrate the **interoperability** benefits of the standard:

- **[Tee for Transform](https://github.com/francescomucio/tee-for-transform)** - A Python framework for managing SQL data transformations with support for multiple database backends. Fully OTS-compliant with import/export capabilities.

*More tools and libraries will be documented as they become available. If you're building an OTS-compliant tool, we'd love to feature it here!*

## Participation

The current process for developing the Open Transformation Specification is described in the [Contributing Guidelines](CONTRIBUTING.md).



## Licensing

See: [License](LICENSE) (Apache-2.0)
