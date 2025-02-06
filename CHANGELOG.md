## [1.3.0] - 2025-02-06
- Updated Trustification image to 1.3.0

### Minor Changes
- TC-1886 Sbom license scanner backend
- TC-2117 Update Filters from 2023,2022 to 2024, 2023

### Security Fixes

### Bugfixes
- TC-1710 Inconsistent visuals when a CVE has multiple CVSS scores
- TC-1978 DA Report throws JsValue error for ubi sboms
- TC-2103 Mismatching count of CVE between Dashboard and CVE panel or SBOM Vulnerabilities
- TC-2118 CycloneDX SBOM Package with License Expression
- TC-2150 License ID with Spaces not captured properly
- TC-2160 Missing License Reference and Incorrect License ID on the License Export
- TC-2161 Alternate Package Reference column loaded Empty for CycloneDX SBOM
- TC-2182 There are large loading times in the search page and in Home page
- TC-2187 CycloneDX SBOM Package with multiple licenses
- TC-2212 License Export name column loaded with Package name
- TC-2213 CycloneDX SBOM Level license not captured on the Report

## [1.2.2] - 2025-01-21

### Release Summary
- Updated Trustification and Guac images to 1.2.2

### Minor Changes

### Security Fixes
- TC-2075 CVE-2024-45337: golang.org/x/crypto v0.26.0
- TC-2076 CVE-2024-45338: golang.org/x/net v0.28.0
- TC-2097 CVE-2025-21613/CVE-2025-21614 github.com/go-git/go-git/v5 v5.12.0

### Bugfixes
- TC-1817 Bombastic-collector doesn't handle special characters in sbom ID
- TC-2022 Collector-osv fails to ingest vulnerabilities with CVSS_V4 severity
- TC-2089 TPA version on Spog UI is not updated
