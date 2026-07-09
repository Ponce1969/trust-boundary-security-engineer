---
id: PAT-003
title: Background Task Lifetime Leak
signals:
  - async
  - background-worker
  - task-scheduling
execution_model:
  - Concurrent
confidence: validated
evidence_count: 2
---

# PAT-003: Background Task Lifetime Leak

## Reasoning Hypothesis (SPEC-005)
A background task executed asynchronously (e.g., fire-and-forget) capturing a reference to a request-scoped context or resource (like an HTTP Context or a DB Connection) will attempt to use it after the request has finished. This causes concurrency exceptions, access violations, or data leakage if the thread pool reassigns that context to another user.

## Vulnerable Code (Before - Node.js Express)
```javascript
app.post('/checkout', (req, res) => {
    const order = req.body;
    res.status(202).send('Order processing started');

    // Asynchronous background task captures HTTP request context (req.user)
    setTimeout(async () => {
        // req.user might be mutated or cleaned up after response is sent, 
        // leading to null pointer errors or processing the order for the wrong user
        await processOrder(order, req.user.id); 
    }, 1000);
});
```

## Corrected Code (After - Node.js Express)
```javascript
app.post('/checkout', (req, res) => {
    const order = req.body;
    // Extract and copy the exact static values needed before response completes
    const userIdCopy = req.user.id; 
    
    res.status(202).send('Order processing started');

    setTimeout(async () => {
        // Background task only depends on isolated copy of data
        await processOrder(order, userIdCopy); 
    }, 1000);
});
```

## Sufficient Control
Ensure background tasks only capture immutable, cloned value types rather than references to request-scoped container resources or request context objects.
