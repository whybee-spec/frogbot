

[comment]: <> (FrogbotReviewComment)

<div align='center'>

[![🚨 Frogbot scanned this pull request and found the below:](https://raw.githubusercontent.com/jfrog/frogbot/master/resources/v2/vulnerabilitiesBannerPR.png)](https://docs.jfrog-applications.jfrog.io/jfrog-applications/frogbot)

</div>



## 📗 Scan Summary
- Frogbot scanned for vulnerabilities and found 2 issues

| Scan Category                | Status                  | Security Issues                  |
| --------------------- | :-----------------------------------: | ----------------------------------- |
| **Software Composition Analysis** | ✅ Done | <details><summary><b>2 Issues Found</b></summary><img src="https://raw.githubusercontent.com/jfrog/frogbot/master/resources/v2/smallHigh.svg" alt=""/> 2 High<br></details> |
| **Contextual Analysis** | ✅ Done | - |
| **Static Application Security Testing (SAST)** | ✅ Done | Not Found |
| **Secrets** | ✅ Done | - |
| **Infrastructure as Code (IaC)** | ✅ Done | Not Found |

### 📦 Vulnerable Dependencies

<div align='center'>

| Severity                | ID                  | Contextual Analysis                  | Direct Dependencies                  | Impacted Dependency                  | Fixed Versions                  |
| :---------------------: | :-----------------------------------: | :-----------------------------------: | :-----------------------------------: | :-----------------------------------: | :-----------------------------------: |
| ![high (not applicable)](https://raw.githubusercontent.com/jfrog/frogbot/master/resources/v2/notApplicableHigh.png)<br>    High | CVE-2022-3517 | Not Applicable | minimatch:3.0.4 | minimatch 3.0.4 | [3.0.5] |
| ![high](https://raw.githubusercontent.com/jfrog/frogbot/master/resources/v2/applicableHighSeverity.png)<br>    High | CVE-2022-29217 | Not Covered | pyjwt:1.7.1 | pyjwt 1.7.1 | [2.4.0] |

</div>


### 🔖 Details


<details><summary><b>[ CVE-2022-3517 ] minimatch 3.0.4</b></summary>

### Vulnerability Details
|                 |                   |
| --------------------- | :-----------------------------------: |
| **Contextual Analysis:** | Not Applicable |
| **Direct Dependencies:** | minimatch:3.0.4 |
| **Impacted Dependency:** | minimatch:3.0.4 |
| **Fixed Versions:** | [3.0.5] |
| **CVSS V3:** | 7.5 |

A vulnerability was found in the minimatch package. This flaw allows a Regular Expression Denial of Service (ReDoS) when calling the braceExpand function with specific arguments, resulting in a Denial of Service.<br></details>

<details><summary><b>[ CVE-2022-29217 ] pyjwt 1.7.1</b></summary>

### Vulnerability Details
|                 |                   |
| --------------------- | :-----------------------------------: |
| **Jfrog Research Severity:** | <img src="https://raw.githubusercontent.com/jfrog/frogbot/master/resources/v2/smallMedium.svg" alt=""/> Medium |
| **Contextual Analysis:** | Not Covered |
| **Direct Dependencies:** | pyjwt:1.7.1 |
| **Impacted Dependency:** | pyjwt:1.7.1 |
| **Fixed Versions:** | [2.4.0] |
| **CVSS V3:** | 7.5 |

Algorithm confusion in PyJWT leads to authentication bypass.

### 🔬 JFrog Research Details

**Description:**
[PyJWT](https://pypi.org/project/PyJWT) is a Python implementation of the RFC 7519 standard (JSON Web Tokens). [JSON Web Tokens](https://jwt.io/) are an open, industry standard method for representing claims securely between two parties. A JWT comes with an inline signature that is meant to be verified by the receiving application. JWT supports multiple standard algorithms, and the algorithm itself is **specified in the JWT token itself**.

The PyJWT library uses the signature-verification algorithm that is specified in the JWT token (that is completely attacker-controlled), however - it requires the validating application to pass an `algorithms` kwarg that specifies the expected algorithms in order to avoid key confusion. Unfortunately -  a non-default value `algorithms=jwt.algorithms.get_default_algorithms()` exists that allows all algorithms.
The PyJWT library also tries to mitigate key confusions in this case, by making sure that public keys are not used as an HMAC secret. For example, HMAC secrets that begin with `-----BEGIN PUBLIC KEY-----` are rejected when encoding a JWT.

It has been discovered that due to missing key-type checks, in cases where -
1. The vulnerable application expects to receive a JWT signed with an Elliptic-Curve key (one of the algorithms `ES256`, `ES384`, `ES512`, `EdDSA`)
2. The vulnerable application decodes the JWT token using the non-default kwarg `algorithms=jwt.algorithms.get_default_algorithms()` (or alternatively, `algorithms` contain both an HMAC-based algorithm and an EC-based algorithm)

An attacker can create an HMAC-signed (ex. `HS256`) JWT token, using the (well-known!) EC public key as the HMAC key. The validating application will accept this JWT token as a valid token.

For example, an application might have planned to validate an `EdDSA`-signed token that was generated as follows -
```python
# Making a good jwt token that should work by signing it with the private key
encoded_good = jwt.encode({"test": 1234}, priv_key_bytes, algorithm="EdDSA")
```
An attacker in posession of the public key can generate an `HMAC`-signed token to confuse PyJWT - 
```python
# Using HMAC with the public key to trick the receiver to think that the public key is a HMAC secret
encoded_bad = jwt.encode({"test": 1234}, pub_key_bytes, algorithm="HS256")
```

The following vulnerable `decode` call will accept BOTH of the above tokens as valid - 
```
decoded = jwt.decode(encoded_good, pub_key_bytes, 
algorithms=jwt.algorithms.get_default_algorithms())
```

**Remediation:**
##### Development mitigations

Use a specific algorithm instead of `jwt.algorithms.get_default_algorithms`.
For example, replace the following call - 
`jwt.decode(encoded_jwt, pub_key_bytes, algorithms=jwt.algorithms.get_default_algorithms())`
With -
`jwt.decode(encoded_jwt, pub_key_bytes, algorithms=["ES256"])`
<br></details>


---
<div align='center'>

[🐸 JFrog Frogbot](https://docs.jfrog-applications.jfrog.io/jfrog-applications/frogbot)

</div>
