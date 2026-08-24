# Migrating eBay `UploadSiteHostedPictures` before September 30, 2026

eBay lists `UploadSiteHostedPictures` for decommission on **September 30,
2026**. Its official replacements are the Media API methods
`createImageFromFile` and `createImageFromUrl`.

- [eBay API deprecation status](https://developer.ebay.com/develop/get-started/api-deprecation-status)
- [UploadSiteHostedPictures reference](https://developer.ebay.com/devzone/xml/docs/Reference/ebay/UploadSiteHostedPictures.html)
- [Media API image guide](https://developer.ebay.com/api-docs/sell/static/inventory/managing-image-media.html)
- [Media API overview](https://developer.ebay.com/develop/api/sell/media_api)

## What actually changes

The migration is an upload/authentication/response-boundary change, not
necessarily a listing rewrite.

| Legacy Trading API boundary | Media API boundary |
| --- | --- |
| `POST /ws/api.dll` with Trading XML | `POST /commerce/media/v1_beta/image/create_image_from_file` or `.../create_image_from_url` |
| Auth'n'Auth or XML-embedded token | OAuth user bearer token with `sell.inventory` scope |
| File multipart plus Trading XML, or `ExternalPictureURL` in XML | Multipart `image` part for a file, or JSON `{"imageUrl":"..."}` for a URL |
| XML `SiteHostedPictureDetails.FullURL` | HTTP 201 JSON `imageUrl` plus the `Location` response header |

For file uploads, let the HTTP library generate the multipart
`Content-Type` boundary. Do not preserve a fixed multipart header that omits
the generated boundary.

## What may stay

If the repository's data flow only passes the returned hosted-image URL into
`PictureURL`, that downstream URL consumption can stay. Existing listing
construction, including an `AddFixedPriceItem` flow, should be changed only
when repository evidence shows that it depends on another legacy response
detail.

That is deliberately a narrow claim: preserving `PictureURL` consumption does
not prove that every listing field or authentication path is already
compatible.

## Representative public-code evidence

This [public Python example](https://gist.github.com/BabonNow/76366e2bb6ac5e90046731a12096eb99)
shows the migration shape the scanner is designed to recognize: Python
`requests` calls `UploadSiteHostedPictures`, then consumes
`SiteHostedPictureDetails.FullURL` for the listing flow. It is representative
technical evidence, not evidence of demand or an endorsement by its author.

## Repository-specific free scan

Use the self-service scanner:

**[Scan a public GitHub repository or ZIP](https://ebay-migration-rescue-test.megkeyvisual.chatgpt.site/)**

The free result shows:

- **WHAT BREAKS** — runtime legacy calls and their upload/auth/response risks;
- **WHAT CAN STAY** — only the downstream boundary supported by repository
  data-flow evidence;
- **SELECTED REPLACEMENT** — `createImageFromFile` or `createImageFromUrl`;
- **PATCH READY** or **SCAN ONLY** — before checkout is offered;
- a redacted preview of the repository-specific change when a deterministic
  patch is supported.

The free scan covers broader legacy integrations. The paid patch is currently
available only for two deterministic Python `requests` shapes:

1. a binary upload that parses `SiteHostedPictureDetails.FullURL`; or
2. an `ExternalPictureURL` upload that parses the same response field.

Unsupported languages, ambiguous response flows, and secret-bearing required
hunks remain **SCAN ONLY** and cannot normally enter checkout.

## What the $39 Patch Pack contains

For a supported repository, the one-time **USD 39** download contains:

- `migration.patch`;
- `scan-report.md`;
- `verification-checklist.md`;
- `auth-and-rollout.md`;
- `evidence.json`.

The generated patch removes the legacy upload boundary, selects the evidenced
Media API method, moves authentication to an OAuth bearer header, handles HTTP
201/JSON/`Location`, and retains proven-compatible `PictureURL` consumption.
It does not fabricate token-minting logic or embed a credential.

## Privacy and delivery boundary

- No account, sales call, subscription, private-repository access, GitHub App,
  automatic pull request, or repository write access.
- A ZIP is scanned in the browser. Source code and filenames are not sent to
  product analytics.
- The encrypted Patch Pack is available for download for up to 24 hours after
  checkout starts. Access expires after that; encrypted provider-side copies
  may persist longer due to hosting-provider retention and deletion timing.
- Review and test the patch in eBay Sandbox and through your existing release
  controls before production rollout.

This artifact documents one bounded migration path. It does not claim support
for every eBay SDK, language, wrapper, or authentication architecture.
