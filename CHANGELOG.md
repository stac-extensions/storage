# Changelog
All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

### Changed

### Deprecated

### Removed

### Fixed

## [v2.0.0] - 2021-06-23

### Added

- `storage:schemes`, `storage:refs` and Storage Scheme Object
- Support the storage extension in Links
- Support for the Alternate Assets Extension
- Support for other storage providers, including custom S3 hosts

### Changed

- The storage providers are grouped in `storage:schemes` and located in the Item Properties, Collections or Catalog metadata
- Assets and Links reference the storage schemes by key in `storage:refs` 

### Removed

- `storage:platform`, `storage:region`, `storage:requester_pays` and `storage:tier`

## [v1.0.0] - 2021-06-23

Initial release

[Unreleased]: <https://github.com/stac-extensions/storage/compare/v2.0.0...HEAD>
[v2.0.0]: <https://github.com/stac-extensions/storage/compare/v1.0.0...v2.0.0>
[v1.0.0]: <https://github.com/stac-extensions/storage/tree/v1.0.0>
