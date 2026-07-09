---
id: PAT-002
title: Request Lifetime Mismatch
signals:
  - dependency-injection
  - transient
  - scoped
  - singleton
execution_model:
  - Concurrent
confidence: established
evidence_count: 4
---

# PAT-002: Request Lifetime Mismatch

## Reasoning Hypothesis (SPEC-005)
A long-lived service (Singleton) capturing a short-lived service (Scoped or Transient) at instantiation promotes the short-lived service to a Singleton lifecycle. This bypasses the lifecycle controls, resulting in shared state leaks or stale database connections across concurrent request contexts.

## Vulnerable Code (Before - Java Spring)
```java
@Component
@Scope("singleton")
public class SecurityAuditLogger {
    // Singleton captures Scoped dependency once at application startup
    private final UserSession userSession; 

    public SecurityAuditLogger(UserSession userSession) {
        this.userSession = userSession;
    }

    public void logAction(String action) {
        // userSession will always point to the session of the first user who hit the logger
        System.out.println("User: " + userSession.getUserId() + " performed: " + action);
    }
}
```

## Corrected Code (After - Java Spring)
```java
@Component
@Scope("singleton")
public class SecurityAuditLogger {
    // Inject Provider or ObjectProvider to resolve dependency dynamically per invocation
    private final Provider<UserSession> userSessionProvider; 

    public SecurityAuditLogger(Provider<UserSession> userSessionProvider) {
        this.userSessionProvider = userSessionProvider;
    }

    public void logAction(String action) {
        UserSession currentSession = userSessionProvider.get();
        System.out.println("User: " + currentSession.getUserId() + " performed: " + action);
    }
}
```

## Sufficient Control
Long-lived classes must never capture short-lived dependencies directly. They must resolve them dynamically using Factories, Providers, or by passing the scoped dependency directly as a method parameter.
