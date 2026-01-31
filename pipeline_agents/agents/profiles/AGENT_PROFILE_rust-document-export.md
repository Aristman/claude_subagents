# Agent Profile: Rust Document Export

## Profile Identity

- Profile ID: rust-document-export
- Domain: D-EX-001 Export
- Scope: Document generation (PDF, DOCX) using Rust
- Applies to agents:
    - Developer Agent
    - Test Engineer Agent
    - Code Reviewer Agent
    - Release / DevOps Agent

This profile is **mandatory** for any feature assigned to the `D-EX-001` domain.

Profiles are loaded from:
~/.claude/agents/profiles/AGENT_PROFILE_rust-document-export.md

---

## Supported Languages

The agent MUST use:
- Rust (stable channel, edition 2021)

The agent MUST NOT introduce other languages without architectural approval.

---

## Framework and Runtime Assumptions

The system is assumed to run:
- using lo-pdf or pdf-gen for PDF generation
- using docx-rs or rust-docx for DOCX generation
- targeting Windows 10/11, macOS 12+, Linux Ubuntu 22.04+
- supporting Russian language text

The agent MUST:
- follow Rust best practices and idioms
- handle file system operations correctly
- respect platform-specific file paths

---

## Architectural Patterns

The agent MUST follow these principles:
- Clear separation between:
    - document generation logic
    - layout/formatting logic
    - file system operations
- Format abstraction (support multiple export formats)
- Error handling for file operations

The agent MUST NOT:
- embed layout logic in file operations
- create tight coupling to specific formats
- ignore file system errors

---

## PDF Generation Rules

- PDFs MUST be generated using native Rust libraries
- PDFs MUST support Russian characters (UTF-8)
- PDFs MUST include embedded fonts for Russian
- PDFs MUST support images
- PDFs MUST follow magazine layout standards

The agent MUST:
- handle font embedding correctly
- support page breaks
- handle image inclusion
- validate PDF output

---

## DOCX Generation Rules

- DOCX files MUST be compatible with Microsoft Word
- DOCX files MUST support Russian characters
- DOCX files MUST include images
- DOCX files MUST preserve document structure

The agent MUST:
- validate DOCX output
- handle proper XML structure
- support document formatting

---

## Error Handling

- All export operations MUST return Result<T, E>
- File system errors MUST be handled gracefully
- Export errors MUST provide clear messages
- Partial exports MUST be cleaned up

The agent MUST:
- use thiserror for error definitions
- clean up temporary files
- provide context for export errors

---

## Testing Strategy (MANDATORY)

The agent MUST implement tests in the following order:

1. Unit tests
    - document building logic
    - formatting utilities
    - layout calculations

2. Integration tests
    - PDF generation
    - DOCX generation
    - file system operations

The agent MUST NOT:
- skip tests defined in the TDD roadmap
- rely only on manual file inspection

---

## Non-Functional Priorities

The agent MUST prioritize:
1. Correctness (valid output files)
2. Compatibility (Word, PDF readers)
3. Performance (reasonable export time)
4. File size (efficient compression)
5. Maintainability

The agent SHOULD:
- optimize for file size when appropriate
- provide progress feedback for long exports

---

## Performance Constraints

The agent MUST:
- handle large documents without excessive memory use
- stream content when possible
- avoid loading entire documents in memory

The agent SHOULD:
- profile export operations
- optimize for common document sizes

---

## Security Constraints

The agent MUST:
- validate file paths before writing
- sanitize file names
- handle file permissions correctly
- not expose system paths in errors

The agent MUST NOT:
- write files outside designated directories
- use predictable temporary file names
- expose absolute paths in error messages

---

## File System Operations

The agent MUST:
- use appropriate directories for exports
- handle file name conflicts
- clean up temporary files
- respect user-selected export locations

The agent SHOULD:
- provide meaningful default file names
- suggest safe file names

---

## Image Handling

The agent MUST:
- support image inclusion in documents
- handle different image formats (PNG, JPG)
- resize images when appropriate
- maintain image aspect ratios

The agent SHOULD:
- optimize images for document embedding
- handle missing images gracefully

---

## Russian Language Support

The agent MUST:
- support UTF-8 encoding
- embed appropriate fonts for PDF
- handle Cyrillic characters correctly
- support proper text direction

The agent SHOULD:
- test with various Russian text samples
- handle special characters correctly

---

## Document Structure

For CraftWriter articles, the agent MUST support:
- Title (heading)
- Lead paragraph
- Materials list
- Step-by-step instructions
- Tips section
- Safety section
- Image placement

---

## Folder Export

The agent MUST:
- create folder with article title
- export text as separate file
- copy images to folder
- maintain image references

---

## Forbidden Practices (Strict)

The agent MUST NOT:
- write files outside designated directories
- ignore file system errors
- create invalid documents
- "fix" unrelated code opportunistically
- hardcode file paths

---

## Deviation Policy

If deviation from this profile is unavoidable:
- the deviation MUST be explicitly documented
- the reason MUST be technical, not convenience-based
- the deviation will be reviewed and verified

Undocumented deviations are considered defects.

---

## Profile Authority

This profile:
- constrains implementation behavior
- overrides agent preferences
- is subordinate only to:
    - ARCHITECTURE_OVERVIEW.md
    - explicit feature roadmap instructions

Failure to comply with this profile is grounds for rejection
during review or verification.
