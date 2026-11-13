# Uploads

Use when an endpoint accepts files or issues direct-upload permissions.

- Enforce per-file and total bytes while streaming, plus file-count and time limits. Treat `Content-Length` as a hint, not the enforcement boundary. Bound decompressed output when extraction is supported.
- Allow only required formats. Validate actual content with appropriate parsers; filenames, MIME headers, and magic bytes alone cannot establish safety. Apply malware scanning or sanitization where the content's use warrants it; do not send private files to public scanners without authorization.
- Generate storage identifiers server-side. Keep client filenames as sanitized metadata, prevent path traversal/overwrites, and store files outside executable/public paths. Authorize upload, replacement, finalization, and retrieval independently. These controls follow [OWASP upload guidance](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html).
- Scope direct-upload grants to the intended object, operation, and expiry. At finalization, verify ownership, actual stored size/content, and required checks before attaching or publishing the object. Prevent later overwrite of a validated object through an outstanding grant.
- Preserve incomplete or quarantined state until checks finish; report it truthfully. Clean up rejected files, interrupted multipart uploads, and abandoned objects using existing lifecycle mechanisms. Return file metadata through the shared JSON envelope; keep download bytes separate.

**Verify:** a stream exceeding its declared size, disguised content, another user's object at finalization, repeated finalization, and interrupted-upload cleanup.
