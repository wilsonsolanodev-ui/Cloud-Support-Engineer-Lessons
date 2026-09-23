# Cloud Support Ticket #4092 - Resolution Runbook

## Incident Details
* **Ticket ID:** #4092
* **Severity:** Severity 2 (Major / Partial Service Interruption)
* **Impact:** Users experiencing `ERR_NAME_NOT_RESOLVED` when attempting to reach `app.example-tech.com`.
* **Category:** Network / DNS Infrastructure


BREAK-FIX TROUBLESHOOTING SCENARIO
Incident Ticket #4092: "My Domain is Down After AWS CloudFront Migration"
Client Escalation:

"We pointed our domain app.example-tech.com to our new AWS CloudFront distribution earlier today. Internal staff are reporting that the website won't load at all, throwing ERR_NAME_NOT_RESOLVED in their browser. However, our external consultant says it works fine for them. Please fix ASAP!"

The 3-Step Diagnostic Investigation
As a Cloud Support Engineer, here is how you systematically isolate this issue using your CLI tools:

Diagnostic Command 1: Test Local Name Resolution
Bash
nslookup app.example-tech.com
What to look for in the output: Check if nslookup returns *** UnKnown can't find app.example-tech.com: Server failure or Non-existent domain (NXDOMAIN). This confirms that the user's default local DNS server cannot find a valid record.

Diagnostic Command 2: Query Authoritative/Public DNS Directly
Bash
dig @8.8.8.8 app.example-tech.com CNAME +short
What to look for in the output: If querying Google's DNS (8.8.8.8) successfully returns the CloudFront endpoint (e.g., d111111abcdef8.cloudfront.net), but querying the domain directly without @8.8.8.8 fails, the problem is cached stale DNS records or local ISP propogation delays.

If 8.8.8.8 also returns nothing, check the raw response status header: look for status: SERVFAIL or status: NXDOMAIN.

Diagnostic Command 3: Bypass DNS and Force a Direct Connection (curl --resolve)
Bash
curl -v -I --resolve app.example-tech.com:443:d111111abcdef8.cloudfront.net https://app.example-tech.com
What to look for in the output: The --resolve flag forces curl to map app.example-tech.com:443 directly to the target CloudFront IP/endpoint, bypassing DNS lookups entirely.

If this returns HTTP/1.1 200 OK, it proves 100% that the application, web server, and CloudFront distribution are fully operational, isolating the root cause strictly to the DNS propagation/TTL configuration layer.

Root Cause Analysis (RCA) Summary
Root Cause: The client's previous DNS record had a high TTL (Time-To-Live) (e.g., 86,400 seconds / 24 hours). Local ISP DNS resolvers were serving the old/cached record or failing until the cache expired.

Resolution: Wait for TTL expiration or flush local DNS cache (ipconfig /flushdns on Windows). Recommend lowering TTL to 300 seconds prior to future migrations.