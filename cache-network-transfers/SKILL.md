---
name: cache-network-transfers
description: Cache every network transfer in a local temporary file and reuse that file for all further operations to reduce latency and avoid rate limits. Always apply this skill on every task, including downloads, searches, browsing, API calls, and remote tool operations.
---

# Cache network transfers

Apply this skill on every task. Before any network operation, check for an existing local copy. Save each new transfer to a local temporary file before processing its data. Perform subsequent inspection, searching, parsing, transformation, and extraction from that file.

## Fetch once, reuse locally

1. Create a task cache under the system temporary directory with `mktemp -d`, or reuse the task's existing cache directory. Keep its path available throughout the task and pass it to any collaborating agents.
2. Identify each request by its method, URL or remote resource, query parameters, request body, representation, and account context. Use a digest for filenames and a local manifest to map requests to files. Keep different pages, versions, accounts, and representations distinct. Do not put credentials in filenames or the manifest.
3. Check the manifest and any existing tool-managed download or local checkout before transferring. Reuse a complete matching artifact. A cached response satisfies repeated reads of the same request throughout the task.
4. On a cache miss, transfer directly into a temporary partial file. Record the source, retrieval time, status, and relevant version or response headers. Publish the completed cache entry only after the transfer succeeds, using an atomic rename. Never treat a partial download or error response as the requested artifact.
5. Read the completed file for every later operation. Use local tools such as `rg`, `jq`, archive readers, and parsers. Do not repeat the remote request to inspect another field, change output formatting, recover truncated terminal output, or feed another tool. Keep the full response on disk and limit only what is displayed.

Coordinate concurrent requests for the same cache entry with a lock or a single fetcher. Retain completed cache entries until the task and its dependent operations finish.

## Tools and transfer types

- For shell downloads and API clients, use output-file options or redirect the full response to disk. Do not repeatedly pipe the same remote resource into different commands.
- For browser, search, connector, and other remote tools, use their download or export option when available. Otherwise serialize the returned data into a temporary file immediately. Preserve source URLs and citation identifiers alongside the result. Reuse the saved response and any tool-provided local artifacts. If the tool cannot expose the full payload, save what it exposes and record that limitation; do not fetch the same resource again merely to obtain a cacheable copy.
- For paginated responses, cache each page separately and build combined results locally. Fetch only missing pages.
- For package managers, Git, and similar clients with native caches, reuse their downloaded objects and local artifacts. Inspect locally and enable offline operation when supported. Avoid duplicate downloads into a second cache solely to satisfy this instruction.
- For uploads or other remote mutations, use a fixed local payload file and cache the response or receipt. Reuse that receipt for later inspection. A cached response never authorizes repeating the mutation, and this skill does not authorize any network action by itself.

## Refresh and failure handling

Fetch again only when the cache is missing, incomplete, corrupt, represents a different request, or fails the task's actual freshness requirement. An explicit user request for fresh data also permits a refresh. Preserve the previous complete file until its replacement succeeds and record why the refresh was needed. Do not issue a remote validation request on every cache read.

Save failure details separately from successful artifacts. Respect server retry instructions and use bounded retries for transient failures. Inspect the saved error before retrying; do not repeatedly send an unchanged failing request. Resume an interrupted transfer only when the server supports it and the remote object still matches.

Use private temporary directories for authenticated or sensitive responses. Keep credentials out of logs and source metadata. When policy forbids writing a response to disk, use permitted tool-managed storage or in-memory reuse and state the caching limitation instead of repeating the transfer.
