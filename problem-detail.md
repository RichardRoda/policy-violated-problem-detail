# 1. Introduction
This document specifies a Policy Violation problem detail per [RFC 9457 section 4.2 registered problem type](https://www.rfc-editor.org/rfc/rfc9457.html#registry) to be used with a 403 Forbidden  response code.  Certain AI systems can exhibit behaviors such as giving hallucinated, incorrect, and differing answers for a given input.  Such systems may need a remediation before requests are accepted by them.  This problem detail is designed to be used in any situation when requests should be rejected due to a policy violation until it has been signaled that the requester has been remediated or a specified amount of time has elapsed.

# 2. Policy Violation Problem Detail
Type URI: https://iana.org/assignments/http-problem-types#policy-violation  
Title: Policy Violation  
Recommended HTTP status code: 403  
Reference: This RFC

The following extension members are defined per [RFC 9457 section 3.2 extension members](https://www.rfc-editor.org/rfc/rfc9457.html#name-extension-members).  At least one of these MUST be present.

* remediationUri - A server specific URI that a requester uses to signal that the state resulting in the policy violation has been remediated.  This is a JSON string.
* retryAfter - The number of seconds that the requester SHOULD wait to try again unless the remediationUri or scope are present and successfully used.  The `Retry-After` header SHOULD also be present in the response if this member is present.  This is a JSON number.
* scope - An [RFC 7521 authorization scope](https://www.rfc-editor.org/rfc/rfc7521#section-3.3) required to assert remediation.  The `WWW-Authenticate: Bearer error="insufficient_scope", scope=<required scope>` header SHOULD be included per [RFC 6750 6.2.3 insufficent_scope error value](https://www.rfc-editor.org/rfc/rfc6750.html#section-6.2.3).  If remediationUri is specified, it SHOULD be used with a token bearing the specified authorization scope to assert remediation.  If remediationUri is not present, using the token with a subsequent request asserts remediation.  This is a JSON string.

The server SHOULD continue to reject requests unless the conditions described by the extension members present in the response are satisfied.  The server MAY allow requests if both a remediationUri and scope are specified, but the requester resumed requests with the token bearing the specified scope without using the remediationUri.

### Example:
```json
HTTP/1.1 403 Forbidden
Content-Type: application/problem+json
Content-Language: en
Retry-After: 3600
WWW-Authenticate: Bearer error="insufficient_scope", scope=policyRemediated
{
	"type": "https://iana.org/assignments/http-problem-types#policy-violation",
	"title": "Policy Violation",
	"detail": "erratic requests",
	"remediationUri": "https://example.com/policy-violation/remediation/AF4E234534DE",
	"retryAfter": 3600,
	"scope": "policyRemediated"
}
```
The above example shows a response that uses all of the extension members.  In this example, after an hour elapses, the server will accept requests.  Otherwise, the remedationUri must be accessed using a JWT authorization token with the specified scope.  Note that use of the retryAfter member and header will allow generic requesters that do not process this problem detail to eventually recover and resume requests.

A status code other than 403 MAY be used if it more accurately describes the policy violation.  For example, if the primary reason a policy is violated is because of excessive requests, a 429 error could be used.  Such a status code may improve interoperability with a generic requester. 

# 3. Security Considerations
The security considerations in [RFC 9457 section 5 Security Considerations](https://www.rfc-editor.org/rfc/rfc9457.html#name-security-considerations) apply to this.  Specifically, care must be taken that if present any information contained in the response to the "instance" URI does not leak any information that could be leveraged in an attack against the server.

# 4. IANA Considerations
Iana is asked to add the following to the "HTTP Problem Types" registry:
* Type URI: https://iana.org/assignments/http-problem-types#policy-violation
* Title: Policy Violation
* Recommended HTTP status code: 403
* Reference: This RFC
