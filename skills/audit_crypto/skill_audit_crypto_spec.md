# Skill: audit_crypto

**Version:** 1.0.0  
**Last Updated:** January 18, 2026  
**Maintained By:** Security Team

---

## Purpose

Scans configuration files and source code for cryptographic usage patterns, identifies security weaknesses, validates compliance with modern cryptographic standards (NIST, FIPS 140-2, OWASP), and provides actionable remediation recommendations.

---

## Inputs

### Required Inputs

1. **Target Paths**
   - File paths to scan
   - Directory to recursively scan
   - List of specific files

2. **Language/Framework**
   - Python (cryptography, PyCryptodome, hashlib)
   - Java (javax.crypto, Bouncy Castle)
   - Node.js (crypto, node-forge)
   - Go (crypto/*)
   - C/C++ (OpenSSL, libsodium)
   - Auto-detect if not specified

3. **Compliance Baseline**
   - `nist`: NIST SP 800-175B standards
   - `fips`: FIPS 140-2 requirements
   - `owasp`: OWASP cryptographic guidelines
   - `pci`: PCI-DSS requirements
   - `custom`: User-defined rules

### Optional Inputs

- **Exclusions**: Paths or patterns to skip (e.g., test files, vendor code)
- **Depth**: How deeply to scan (surface-level vs. deep analysis)
- **Output Format**: Text, JSON, SARIF (for CI/CD integration)
- **Severity Threshold**: Minimum severity to report (CRITICAL, HIGH, MEDIUM, LOW)

### Input Example

```json
{
  "paths": ["./src", "./config"],
  "language": "python",
  "baseline": "nist",
  "exclusions": ["./src/tests/*", "./vendor/*"],
  "depth": "deep",
  "output_format": "markdown",
  "severity_threshold": "MEDIUM"
}
```

---

## Outputs

### 1. Cryptographic Inventory

Table of all cryptographic operations found:

| Location | Algorithm | Key Size | Mode | Usage | Status |
|----------|-----------|----------|------|-------|--------|
| auth.py:42 | AES | 256 | GCM | Encryption | ✓ Secure |
| util.py:15 | MD5 | N/A | N/A | Hashing | ✗ Weak |
| config.py:8 | RSA | 2048 | OAEP | Key Exchange | ✓ Secure |

### 2. Findings Report

Categorized by severity:

```markdown
## CRITICAL Findings (3)

### CWE-327: Use of Broken or Risky Cryptographic Algorithm
**File**: `auth/password.py`, Line 23  
**Code**: `hashlib.md5(password.encode()).hexdigest()`  
**Issue**: MD5 is cryptographically broken and unsuitable for password hashing  
**Impact**: Passwords can be recovered via rainbow tables or collision attacks  
**Recommendation**: Use bcrypt, argon2, or PBKDF2 with high iteration count  
**CWE**: CWE-327  
**CVSS**: 9.1 (Critical)

## HIGH Findings (5)
...

## MEDIUM Findings (7)
...

## LOW Findings (2)
...
```

### 3. Remediation Recommendations

Specific, actionable guidance:

```python
# BEFORE (Insecure)
import hashlib
password_hash = hashlib.md5(password.encode()).hexdigest()

# AFTER (Secure)
import bcrypt
password_hash = bcrypt.hashpw(password.encode(), bcrypt.gensalt(rounds=12))
```

### 4. Compliance Gap Analysis

```markdown
## NIST SP 800-175B Compliance

✗ FAIL: MD5 and SHA-1 usage detected (deprecated since 2015)
✗ FAIL: RSA key size < 2048 bits found in 2 locations
✓ PASS: AES usage is compliant (256-bit keys with GCM mode)
✗ FAIL: Hardcoded cryptographic keys detected
✓ PASS: TLS 1.2+ enforced for network communications

Overall Compliance: 60% (3/5 checks passed)
```

---

## Tool Usage

- **read_file**: Read source code and configuration files
- **web_search**: Look up CVE details, algorithm safety status, current best practices
- **write_file**: Save audit report and findings

---

## Algorithm

### Step 1: Scan Files for Crypto Library Imports

```python
crypto_libraries = {
    'python': ['cryptography', 'PyCryptodome', 'hashlib', 'hmac', 'secrets'],
    'javascript': ['crypto', 'node-forge', 'crypto-js', 'bcrypt'],
    'java': ['javax.crypto', 'java.security', 'org.bouncycastle'],
    'go': ['crypto/', 'golang.org/x/crypto'],
    'c': ['openssl/', 'libsodium', 'mbedtls']
}

imports = scan_for_imports(files, crypto_libraries[language])
```

### Step 2: Extract Algorithm Usage via Regex and AST Parsing

```python
patterns = {
    'hashing': r'(md5|sha1|sha-1|MD5|SHA1)',
    'encryption': r'(des|3des|rc4|blowfish|DES|RC4)',
    'key_generation': r'(Random\(\)|Math\.random|rand\(\))',
    'modes': r'(ECB|CBC|CFB|OFB|GCM|CTR)',
    'key_sizes': r'(128|192|256|512|1024|2048|4096)'
}

# Use AST parsing for accurate detection
for file in files:
    ast_tree = parse_ast(file)
    for node in ast_tree.walk():
        if isinstance(node, CryptoCall):
            extract_algorithm_details(node)
```

### Step 3: Check Against Known-Weak Algorithms

```python
weak_algorithms = {
    'hashing': {
        'MD2': {'status': 'broken', 'since': 2004, 'severity': 'CRITICAL'},
        'MD4': {'status': 'broken', 'since': 1995, 'severity': 'CRITICAL'},
        'MD5': {'status': 'broken', 'since': 2005, 'severity': 'CRITICAL'},
        'SHA1': {'status': 'deprecated', 'since': 2017, 'severity': 'HIGH'},
    },
    'encryption': {
        'DES': {'status': 'broken', 'since': 1999, 'severity': 'CRITICAL'},
        '3DES': {'status': 'deprecated', 'since': 2023, 'severity': 'HIGH'},
        'RC4': {'status': 'broken', 'since': 2015, 'severity': 'CRITICAL'},
        'Blowfish': {'status': 'weak', 'since': 2016, 'severity': 'MEDIUM'},
    }
}

for usage in crypto_usages:
    if usage.algorithm in weak_algorithms[usage.category]:
        create_finding(usage, weak_algorithms[usage.category][usage.algorithm])
```

### Step 4: Verify Key Sizes

```python
minimum_key_sizes = {
    'RSA': 2048,  # NIST recommends 3072+ for new systems
    'DSA': 2048,
    'DH': 2048,
    'ECDSA': 256,
    'ECDH': 256,
    'AES': 128,  # 256 recommended for sensitive data
}

for usage in crypto_usages:
    if usage.algorithm in minimum_key_sizes:
        if usage.key_size < minimum_key_sizes[usage.algorithm]:
            create_finding(
                usage,
                severity='HIGH',
                message=f'{usage.algorithm} key size {usage.key_size} is below minimum {minimum_key_sizes[usage.algorithm]}'
            )
```

### Step 5: Check for Hardcoded Keys, IVs, and Salts

```python
secret_patterns = [
    r'key\s*=\s*["\']([A-Za-z0-9+/=]{16,})["\']',
    r'password\s*=\s*["\'](.+?)["\']',
    r'secret\s*=\s*["\'](.+?)["\']',
    r'iv\s*=\s*["\']([A-Fa-f0-9]{16,})["\']',
    r'salt\s*=\s*["\'](.+?)["\']',
    r'["\']([A-Za-z0-9]{32,})["\']',  # Looks like an API key or secret
]

for file in files:
    content = read_file(file)
    for pattern in secret_patterns:
        matches = re.findall(pattern, content)
        if matches:
            create_finding(
                file=file,
                severity='CRITICAL',
                cwe='CWE-798',
                message='Hardcoded cryptographic secret detected'
            )
```

### Step 6: Validate Modes of Operation

```python
insecure_modes = {
    'ECB': {
        'severity': 'HIGH',
        'reason': 'ECB mode does not provide semantic security (identical plaintexts produce identical ciphertexts)',
        'recommendation': 'Use GCM, CBC+HMAC, or CTR mode'
    },
    'CBC_without_HMAC': {
        'severity': 'MEDIUM',
        'reason': 'CBC mode without authentication is vulnerable to padding oracle attacks',
        'recommendation': 'Use authenticated encryption (GCM) or add HMAC'
    }
}

for usage in crypto_usages:
    if usage.mode in insecure_modes:
        create_finding(usage, insecure_modes[usage.mode])
```

### Step 7: Check Random Number Generation Sources

```python
weak_prng = {
    'python': ['random.random', 'random.randint'],  # Use secrets module instead
    'javascript': ['Math.random'],  # Use crypto.randomBytes
    'java': ['java.util.Random'],  # Use SecureRandom
    'c': ['rand()', 'srand()'],  # Use /dev/urandom or RAND_bytes
}

for usage in prng_usages:
    if usage.function in weak_prng[language]:
        create_finding(
            usage,
            severity='HIGH',
            cwe='CWE-338',
            message='Weak PRNG unsuitable for cryptographic operations'
        )
```

### Step 8: Generate Findings with CWE References

```python
finding = {
    'file': file_path,
    'line': line_number,
    'code_snippet': extract_code_context(file_path, line_number, context=3),
    'algorithm': algorithm_name,
    'severity': severity_level,
    'cwe': cwe_id,
    'cvss': calculate_cvss_score(severity, exploitability, impact),
    'description': detailed_description,
    'impact': potential_impact,
    'recommendation': specific_remediation,
    'references': [relevant_links]
}
```

### Step 9: Recommend Modern Alternatives

```python
recommendations = {
    'MD5': {
        'for_hashing': 'Use SHA-256 or SHA-3',
        'for_passwords': 'Use bcrypt, argon2, or PBKDF2',
        'for_integrity': 'Use HMAC-SHA256'
    },
    'SHA1': {
        'for_hashing': 'Use SHA-256 or SHA-3',
        'for_passwords': 'Use bcrypt, argon2, or PBKDF2',
        'for_signatures': 'Use RSA-PSS with SHA-256'
    },
    'DES': {
        'for_encryption': 'Use AES-256-GCM'
    },
    'ECB': {
        'for_encryption': 'Use GCM mode (provides authentication) or CBC with HMAC'
    }
}
```

---

## Failure Modes & Mitigations

### Failure Mode 1: False Positives

**Symptom:** Flagging test code, comments, or legitimate weak crypto usage

**Impact:** Noise in findings, reduced trust in tool

**Mitigation:**
1. Context analysis:
   - Check if code is in test directory
   - Look for test framework imports
   - Check for comments indicating intentional weak crypto (e.g., compatibility)
2. Severity reduction for test code (HIGH → LOW)
3. Allow user-defined exclusions via `.cryptoaudit-ignore` file
4. Smart filtering:
   ```python
   if is_test_file(file_path) or is_example_code(file_path):
       finding.severity = reduce_severity(finding.severity)
       finding.note = "Found in test/example code"
   ```

### Failure Mode 2: Obfuscated or Dynamically Generated Code

**Symptom:** Cannot detect crypto usage when algorithm is constructed at runtime

**Impact:** False negatives, missed vulnerabilities

**Mitigation:**
1. Flag for manual review any dynamic constructions:
   ```python
   algorithm = eval(user_input)  # Flag this pattern
   cipher = getattr(crypto_module, algo_name)  # Flag dynamic lookup
   ```
2. Warn user about limitations
3. Recommend static analysis complement (manual code review)
4. Pattern detection for common obfuscation:
   ```python
   obfuscation_patterns = [
       r'eval\(',
       r'exec\(',
       r'getattr\(',
       r'__import__\(',
   ]
   ```

### Failure Mode 3: Unknown or Proprietary Algorithms

**Symptom:** Encounter algorithm not in known database

**Impact:** Cannot assess security of unfamiliar algorithm

**Mitigation:**
1. Use web_search to look up algorithm:
   ```python
   if algorithm not in known_algorithms:
       search_result = web_search(f"{algorithm} cryptographic algorithm security")
       if search_result:
           assess_based_on_search(search_result)
       else:
           flag_unknown_algorithm(algorithm)
   ```
2. Flag as "UNKNOWN - REQUIRES MANUAL REVIEW"
3. Recommend consulting cryptography expert
4. Assume conservative stance (flag as potential risk)

### Failure Mode 4: Context-Dependent Security

**Symptom:** Algorithm may be acceptable in some contexts but not others (e.g., MD5 for checksums vs. passwords)

**Impact:** Either too many false positives or false negatives

**Mitigation:**
1. Context detection:
   ```python
   if algorithm == 'MD5':
       context = detect_usage_context(code)
       if context == 'checksum':
           severity = 'LOW'  # Acceptable for non-security use
           note = "MD5 acceptable for checksums, but not for security"
       elif context == 'password':
           severity = 'CRITICAL'  # Unacceptable
       else:
           severity = 'MEDIUM'  # Uncertain context
   ```
2. Provide nuanced recommendations based on context
3. Flag ambiguous cases for user review

### Failure Mode 5: Library Version Differences

**Symptom:** Different library versions have different defaults or available algorithms

**Impact:** Recommendations may not apply to user's specific version

**Mitigation:**
1. Detect library version from imports or package files:
   ```python
   version = detect_library_version('cryptography', requirements_file)
   if version < '3.0':
       warn_user("Outdated cryptography library detected")
   ```
2. Version-specific recommendations
3. Recommend upgrading to latest stable version
4. Cross-reference CVEs for specific library versions

---

## Test Cases

### Test Case 1: Critical - Weak Password Hashing

**Input Code (Python):**
```python
# auth/password.py
import hashlib

def hash_password(password):
    return hashlib.md5(password.encode()).hexdigest()

def verify_password(password, hash):
    return hash_password(password) == hash
```

**Expected Output:**
```markdown
## CRITICAL Finding: Weak Password Hashing Algorithm

**Location**: `auth/password.py`, Lines 4-5  
**CWE**: CWE-327 (Use of a Broken or Risky Cryptographic Algorithm)  
**CVSS Score**: 9.1 (Critical)  

### Issue
MD5 is cryptographically broken and completely unsuitable for password hashing:
- Vulnerable to rainbow table attacks
- Fast computation enables brute force (billions of hashes/second on GPUs)
- Collision attacks demonstrated since 2005
- No computational cost to deter attackers

### Impact
- Attacker gaining database access can recover ALL user passwords
- Average password can be cracked in seconds to hours
- Compliance failure (violates PCI-DSS, NIST, OWASP guidelines)

### Recommendation

Replace with bcrypt, argon2, or PBKDF2:

```python
# SECURE IMPLEMENTATION
import bcrypt

def hash_password(password):
    # bcrypt automatically handles salting
    return bcrypt.hashpw(password.encode(), bcrypt.gensalt(rounds=12))

def verify_password(password, password_hash):
    return bcrypt.checkpw(password.encode(), password_hash)
```

**Why this is better:**
- Bcrypt is designed for password hashing (intentionally slow)
- Automatically handles salting (prevents rainbow tables)
- Configurable work factor (rounds=12 provides good security/performance balance)
- Industry standard, widely audited

### References
- OWASP Password Storage Cheat Sheet
- NIST SP 800-63B (Digital Identity Guidelines)
- CWE-327: https://cwe.mitre.org/data/definitions/327.html
```

### Test Case 2: High - Hardcoded Encryption Key

**Input Code (Java):**
```java
// config/SecurityConfig.java
public class SecurityConfig {
    private static final String AES_KEY = "1234567890abcdef1234567890abcdef";
    
    public static String encrypt(String data) throws Exception {
        SecretKeySpec key = new SecretKeySpec(AES_KEY.getBytes(), "AES");
        Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
        cipher.init(Cipher.ENCRYPT_MODE, key);
        return Base64.getEncoder().encodeToString(cipher.doFinal(data.getBytes()));
    }
}
```

**Expected Output:**
```markdown
## CRITICAL Finding: Hardcoded Cryptographic Key

**Location**: `config/SecurityConfig.java`, Line 3  
**CWE**: CWE-798 (Use of Hard-coded Credentials)  
**CVSS Score**: 9.8 (Critical)  

### Issue
Cryptographic key is hardcoded in source code:
- Key is visible to anyone with code access (developers, code repos, CI/CD)
- Key cannot be rotated without code changes
- Same key used across all environments (dev, staging, prod)
- If code is decompiled or leaked, ALL encrypted data is compromised

### Impact
- Complete compromise of encrypted data confidentiality
- Cannot rotate key without application redeployment
- Compliance violations (PCI-DSS 3.6, NIST SP 800-57)
- Insider threat (any developer can decrypt data)

---

## HIGH Finding: Insecure Cipher Mode (ECB)

**Location**: `config/SecurityConfig.java`, Line 7  
**CWE**: CWE-327 (Use of a Broken or Risky Cryptographic Algorithm)  
**CVSS Score**: 7.5 (High)  

### Issue
ECB (Electronic Codebook) mode does not provide semantic security:
- Identical plaintext blocks produce identical ciphertext blocks
- Reveals patterns in encrypted data
- Vulnerable to known-plaintext attacks

### Recommendation

```java
// SECURE IMPLEMENTATION
import javax.crypto.*;
import javax.crypto.spec.*;
import java.security.SecureRandom;

public class SecurityConfig {
    
    // Load key from secure key management system
    public static SecretKey loadKey() throws Exception {
        // Option 1: Environment variable
        String keyString = System.getenv("ENCRYPTION_KEY");
        
        // Option 2: AWS KMS, Azure Key Vault, Google Cloud KMS
        // KeyManagementServiceClient kms = new KeyManagementServiceClient();
        // return kms.getKey("my-encryption-key");
        
        // Option 3: Vault, HashiCorp Vault, etc.
        // VaultClient vault = new VaultClient();
        // return vault.readSecret("secret/encryption-key");
        
        if (keyString == null) {
            throw new IllegalStateException("ENCRYPTION_KEY not set");
        }
        
        byte[] keyBytes = Base64.getDecoder().decode(keyString);
        return new SecretKeySpec(keyBytes, "AES");
    }
    
    public static String encrypt(String data) throws Exception {
        SecretKey key = loadKey();
        
        // Use GCM mode for authenticated encryption
        Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
        
        // Generate random IV for each encryption
        byte[] iv = new byte[12];  // GCM standard IV size
        SecureRandom random = new SecureRandom();
        random.nextBytes(iv);
        GCMParameterSpec spec = new GCMParameterSpec(128, iv);
        
        cipher.init(Cipher.ENCRYPT_MODE, key, spec);
        
        byte[] encrypted = cipher.doFinal(data.getBytes(StandardCharsets.UTF_8));
        
        // Prepend IV to ciphertext (IV doesn't need to be secret)
        byte[] combined = new byte[iv.length + encrypted.length];
        System.arraycopy(iv, 0, combined, 0, iv.length);
        System.arraycopy(encrypted, 0, combined, iv.length, encrypted.length);
        
        return Base64.getEncoder().encodeToString(combined);
    }
    
    public static String decrypt(String encryptedData) throws Exception {
        SecretKey key = loadKey();
        
        byte[] combined = Base64.getDecoder().decode(encryptedData);
        
        // Extract IV (first 12 bytes)
        byte[] iv = new byte[12];
        System.arraycopy(combined, 0, iv, 0, 12);
        
        // Extract ciphertext
        byte[] encrypted = new byte[combined.length - 12];
        System.arraycopy(combined, 12, encrypted, 0, encrypted.length);
        
        Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
        GCMParameterSpec spec = new GCMParameterSpec(128, iv);
        cipher.init(Cipher.DECRYPT_MODE, key, spec);
        
        byte[] decrypted = cipher.doFinal(encrypted);
        return new String(decrypted, StandardCharsets.UTF_8);
    }
}
```

**Key Improvements:**
1. ✓ Key loaded from secure external source (KMS recommended)
2. ✓ GCM mode provides both encryption and authentication
3. ✓ Random IV generated for each encryption operation
4. ✓ IV prepended to ciphertext (standard practice)
5. ✓ Proper exception handling
6. ✓ Key rotation possible without code changes

### Key Management Options
1. **AWS KMS** - Managed key service with audit logs
2. **Azure Key Vault** - Microsoft's key management solution
3. **Google Cloud KMS** - Google's key management service
4. **HashiCorp Vault** - Open-source secrets management
5. **Environment Variables** - Minimum viable solution (better than hardcoding)

### References
- NIST SP 800-38D (GCM Mode)
- OWASP Cryptographic Storage Cheat Sheet
- CWE-798: https://cwe.mitre.org/data/definitions/798.html
```

---

## Performance Characteristics

- **Scan Speed**: ~1000 lines of code per second
- **Memory Usage**: ~100MB base + ~1MB per 1000 lines scanned
- **Accuracy**: 
  - False Positive Rate: ~5-10% (mostly test code)
  - False Negative Rate: ~15-20% (obfuscated code, dynamic construction)
- **Scalability**: Can scan codebases up to 1M lines

---

## Usage Examples

### Example 1: CLI Usage

```bash
claude skill/audit_crypto/execute \
  --path ./src \
  --language python \
  --baseline nist \
  --exclude "./tests/*" \
  --output crypto_audit_report.md
```

### Example 2: CI/CD Integration

```yaml
# .github/workflows/security-scan.yml
name: Crypto Audit

on: [pull_request]

jobs:
  crypto-audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run Crypto Audit
        run: |
          claude skill/audit_crypto/execute \
            --path ./src \
            --baseline nist \
            --format sarif \
            --output crypto-audit.sarif \
            --severity-threshold HIGH
      
      - name: Upload Results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: crypto-audit.sarif
```

### Example 3: Programmatic Usage

```python
from skills import audit_crypto

result = audit_crypto.execute(
    paths=['./src', './config'],
    language='python',
    baseline='nist',
    exclusions=['./tests/*']
)

print(f"Found {len(result.critical_findings)} critical issues")
print(f"Compliance score: {result.compliance_score}%")

for finding in result.critical_findings:
    print(f"  - {finding.file}:{finding.line} - {finding.message}")
```

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-01-18 | Initial release |

---

**End of Skill Specification**