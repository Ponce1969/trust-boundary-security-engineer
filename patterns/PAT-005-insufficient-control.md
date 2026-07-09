---
id: PAT-005
title: Control Present but Insufficient
signals:
  - rate-limiting
  - middleware
execution_model:
  - Concurrent
confidence: validated
evidence_count: 2
---

# PAT-005: Control Present but Insufficient (Rate Limiting)

## Reasoning Hypothesis (SPEC-005)
A security control is present but insufficient if its implementation details can be circumvented or bypassed under typical concurrent execution patterns. For instance, a rate limiter middleware that relies entirely on client-provided headers (such as `X-Forwarded-For`) without upstream proxy validation allows attackers to bypass the control by sending spoofed headers.

## Vulnerable Code (Before - Node.js Express)
```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100, // Limit each IP to 100 requests per window
    // Vulnerable: trusts client-controlled header without validation
    keyGenerator: (req) => req.headers['x-forwarded-for'] || req.ip 
});

app.use(limiter);
```

## Corrected Code (After - Node.js Express)
```javascript
const rateLimit = require('express-rate-limit');

// Express must be configured to trust the specific upstream reverse proxy (e.g. Cloudflare, Nginx)
app.set('trust proxy', '127.0.0.1'); // Trust localhost or CIDR range of your proxy only

const limiter = rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 100,
    // Safely reads the validated IP resolved by Express
    keyGenerator: (req) => req.ip 
});

app.use(limiter);
```

## Sufficient Control
A protection control is only sufficient if it is anchored to trusted, immutable inputs. When reading proxy headers, validation must be anchored to trusted network boundaries (e.g., configuring `trust proxy` to target specific trusted upstream server IP/CIDR subnets).
