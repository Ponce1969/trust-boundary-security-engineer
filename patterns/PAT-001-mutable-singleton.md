---
id: PAT-001
title: Mutable Singleton
signals:
  - dependency-injection
  - singleton
execution_model:
  - Concurrent
confidence: established
evidence_count: 5
---

# PAT-001: Mutable Singleton

## Reasoning Hypothesis (SPEC-005)
When multiple concurrent requests are processed inside a single process, a service registered as a Singleton is shared across all execution threads. Storing request-scoped or user-specific data in mutable instance fields of a Singleton crosses a temporal trust boundary, causing state leakage or cross-request pollution.

## Vulnerable Code (Before - C#)
```csharp
public class UserContextHolder : IUserContextHolder
{
    // Mutable instance field in a Singleton service
    private User _currentUser; 

    public void SetUser(User user)
    {
        _currentUser = user;
    }

    public User GetUser() => _currentUser;
}
```

## Corrected Code (After - C#)
```csharp
public class UserContextHolder : IUserContextHolder
{
    // State is scoped to the execution thread using AsyncLocal
    private readonly AsyncLocal<User> _currentUser = new AsyncLocal<User>();

    public void SetUser(User user)
    {
        _currentUser.Value = user;
    }

    public User GetUser() => _currentUser.Value;
}
```

## Sufficient Control
State inside a Singleton must either be immutable (read-only after construction) or scoped to the execution context flow (e.g., using `AsyncLocal` or request-scoped parameter passing).
