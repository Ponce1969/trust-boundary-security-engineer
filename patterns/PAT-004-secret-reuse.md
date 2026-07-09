---
id: PAT-004
title: Cryptographic Secret Reuse
signals:
  - cryptography
  - encryption
  - key-derivation
execution_model:
  - Sequential
  - Concurrent
  - Distributed
  - Mixed
confidence: established
evidence_count: 3
---

# PAT-004: Cryptographic Secret Reuse

## Reasoning Hypothesis (SPEC-005)
Reusing the same cryptographic key, password, or secret for different security bounds (e.g., using the cookie encryption key for JWT signing or DB encryption) compromises defense-in-depth. A vulnerability or compromise in one layer (like a JWT leak) automatically compromises the other layers, amplifying the blast radius of any security incident.

## Vulnerable Code (Before - Go)
```go
func initSecurity() {
    // Reusing the same master secret for cookies and JWT signing
    sharedSecret := os.Getenv("MASTER_SECRET") 
    
    cookieStore := sessions.NewCookieStore([]byte(sharedSecret))
    jwtSigner := jwt.NewSigner(jwt.SigningMethodHS256, []byte(sharedSecret))
}
```

## Corrected Code (After - Go)
```go
func initSecurity() {
    masterSecret := os.Getenv("MASTER_SECRET")
    
    // Derive unique keys for each specific security purpose using HKDF
    cookieKey := deriveKey(masterSecret, "CookieEncryption")
    jwtKey := deriveKey(masterSecret, "JwtSigning")

    cookieStore := sessions.NewCookieStore(cookieKey)
    jwtSigner := jwt.NewSigner(jwt.SigningMethodHS256, jwtKey)
}

func deriveKey(masterSecret, info string) []byte {
    hkdf := hkdf.New(sha256.New, []byte(masterSecret), nil, []byte(info))
    key := make([]byte, 32)
    io.ReadFull(hkdf, key)
    return key
}
```

## Sufficient Control
Cryptographic keys must have strict domain isolation. Use dedicated environmental variables for each key, or dynamically derive distinct domain-specific keys from a single master key using a secure Key Derivation Function (KDF) like HKDF or PBKDF2.
