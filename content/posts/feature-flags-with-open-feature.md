---
title: "Feature Flags With Open Feature"
date: 2025-06-16T20:59:22-05:00
draft: false
---

# Feature Flags: The Clean Architecture Way

*How OpenFeature enables incremental development while keeping your architecture clean and vendor-agnostic*

---

## The Problem with Traditional Feature Flag Implementations

Picture this: You're working on a critical feature for your application. The business is breathing down your neck, demanding frequent updates on progress. The feature is complex, will take weeks to complete, and you know that merging half-finished code into the main branch is a recipe for disaster.

What do you do? You create a feature branch, of course. But now you're living in isolation, watching as the main branch evolves without you, knowing that each passing day makes your eventual merge more painful.

This is where feature flags come to the rescue. But here's where most teams make a crucial mistake: they tightly couple their application code to a specific feature flag provider. They embed vendor-specific SDKs throughout their codebase, creating what I call "vendor lock-in debt."

## The Dependency Inversion Principle Applied to Feature Flags

Uncle Bob taught us that high-level modules should not depend on low-level modules. Both should depend on abstractions. This principle applies just as much to feature flag systems as it does to databases, web frameworks, or any other external dependency.

Consider this typical, problematic approach:

```go
// BAD: Tightly coupled to LaunchDarkly
import "github.com/launchdarkly/go-server-sdk/v6"

func ProcessPayment(userID string, amount float64) error {
    // Direct dependency on LaunchDarkly client
    client := ldclient.Get()
    useNewPaymentFlow := client.BoolVariation("new-payment-flow", 
        ldcontext.NewBuilder(userID).Build(), false)
    
    if useNewPaymentFlow {
        return processPaymentV2(userID, amount)
    }
    return processPaymentV1(userID, amount)
}
```

What's wrong with this code? Everything. It violates the Dependency Inversion Principle, the Open-Closed Principle, and makes testing unnecessarily difficult. Worse yet, it locks you into LaunchDarkly forever.

## Enter OpenFeature: The SOLID Way

OpenFeature provides us with exactly what we need: a vendor-neutral abstraction that allows us to depend on interfaces, not implementations. Let's see how to do it right:

```go
// GOOD: Depending on abstractions
type FeatureFlags interface {
    GetBoolFlag(ctx context.Context, flagKey string, defaultValue bool) bool
    GetStringFlag(ctx context.Context, flagKey string, defaultValue string) string
    GetIntFlag(ctx context.Context, flagKey string, defaultValue int) int
}

type PaymentProcessor struct {
    flags FeatureFlags
}

func NewPaymentProcessor(flags FeatureFlags) *PaymentProcessor {
    return &PaymentProcessor{flags: flags}
}

func (p *PaymentProcessor) ProcessPayment(ctx context.Context, userID string, amount float64) error {
    useNewFlow := p.flags.GetBoolFlag(ctx, "new-payment-flow", false)
    
    if useNewFlow {
        return p.processPaymentV2(ctx, userID, amount)
    }
    return p.processPaymentV1(ctx, userID, amount)
}
```

Now our `PaymentProcessor` depends on an abstraction, not a concrete implementation. We can inject any feature flag provider that implements our interface.

## Implementing the OpenFeature Adapter

Here's how we implement our abstraction using OpenFeature:

```go
package featureflags

import (
    "context"
    "github.com/open-feature/go-sdk/openfeature"
)

type OpenFeatureAdapter struct {
    client *openfeature.Client
}

func NewOpenFeatureAdapter() *OpenFeatureAdapter {
    client := openfeature.NewClient("vacation-planner")
    return &OpenFeatureAdapter{client: client}
}

func (a *OpenFeatureAdapter) GetBoolFlag(ctx context.Context, flagKey string, defaultValue bool) bool {
    result, _ := a.client.BooleanValue(ctx, flagKey, defaultValue, openfeature.EvaluationContext{})
    return result
}

func (a *OpenFeatureAdapter) GetStringFlag(ctx context.Context, flagKey string, defaultValue string) string {
    result, _ := a.client.StringValue(ctx, flagKey, defaultValue, openfeature.EvaluationContext{})
    return result
}

func (a *OpenFeatureAdapter) GetIntFlag(ctx context.Context, flagKey string, defaultValue int) int {
    result, _ := a.client.IntValue(ctx, flagKey, int64(defaultValue), openfeature.EvaluationContext{})
    return int(result)
}
```

## The Power of Provider Flexibility

The beauty of this approach becomes evident when you need to switch providers. Let's say you start with a simple in-memory provider for local development:

```go
package main

import (
    "context"
    "github.com/open-feature/go-sdk/openfeature"
    "github.com/open-feature/go-sdk/openfeature/memprovider"
)

func main() {
    // Local development: use in-memory provider
    flags := map[string]interface{}{
        "new-payment-flow": true,
        "enable-analytics": false,
        "api-timeout": 5000,
    }
    
    provider := memprovider.NewInMemoryProvider(flags)
    openfeature.SetProvider(provider)
    
    // Your application code remains unchanged
    flagAdapter := featureflags.NewOpenFeatureAdapter()
    paymentProcessor := NewPaymentProcessor(flagAdapter)
    
    // Use your payment processor...
}
```

Later, when you're ready for production, you simply swap the provider:

```go
func main() {
    // Production: use LaunchDarkly provider
    provider := launchdarkly.NewProvider("your-sdk-key")
    openfeature.SetProvider(provider)
    
    // Same application code, different provider
    flagAdapter := featureflags.NewOpenFeatureAdapter()
    paymentProcessor := NewPaymentProcessor(flagAdapter)
    
    // Your business logic is completely unaffected
}
```

## Incremental Development: The Right Way

Feature flags enable a development workflow that would make any craftsman proud. Instead of long-lived branches that create integration nightmares, you can develop incrementally:

```go
type VacationService struct {
    flags FeatureFlags
    oldBookingEngine BookingEngine
    newBookingEngine BookingEngine
}

func (v *VacationService) BookVacation(ctx context.Context, request BookingRequest) (*Booking, error) {
    useNewEngine := v.flags.GetBoolFlag(ctx, "new-booking-engine", false)
    
    if useNewEngine {
        // New implementation being developed
        return v.newBookingEngine.Book(ctx, request)
    }
    
    // Stable, existing implementation
    return v.oldBookingEngine.Book(ctx, request)
}
```

This allows you to:

1. **Deploy incomplete features safely** - The flag starts as `false`, so only you see the new code
2. **Test in production** - Enable the flag for your test accounts only
3. **Gradual rollout** - Enable for 1% of users, then 10%, then 100%
4. **Instant rollback** - If something goes wrong, flip the flag back to `false`

## Context-Aware Feature Flags

OpenFeature's evaluation context allows for sophisticated targeting:

```go
func (v *VacationService) BookVacation(ctx context.Context, user User, request BookingRequest) (*Booking, error) {
    evaluationCtx := openfeature.NewEvaluationContext(
        user.ID,
        map[string]interface{}{
            "email": user.Email,
            "tier": user.Tier,
            "country": user.Country,
        },
    )
    
    useNewEngine := v.flags.GetBoolFlagWithContext(ctx, "new-booking-engine", false, evaluationCtx)
    
    if useNewEngine {
        return v.newBookingEngine.Book(ctx, request)
    }
    return v.oldBookingEngine.Book(ctx, request)
}
```

Now you can enable features based on user attributes, geographic location, subscription tiers, or any other criteria that makes business sense.

## Testing: The Clean Architecture Advantage

Because we've inverted our dependencies, testing becomes trivial:

```go
type MockFeatureFlags struct {
    flags map[string]interface{}
}

func (m *MockFeatureFlags) GetBoolFlag(ctx context.Context, key string, defaultValue bool) bool {
    if val, exists := m.flags[key]; exists {
        return val.(bool)
    }
    return defaultValue
}

func TestPaymentProcessor_NewFlow(t *testing.T) {
    // Arrange
    mockFlags := &MockFeatureFlags{
        flags: map[string]interface{}{
            "new-payment-flow": true,
        },
    }
    processor := NewPaymentProcessor(mockFlags)
    
    // Act
    err := processor.ProcessPayment(context.Background(), "user123", 100.0)
    
    // Assert
    assert.NoError(t, err)
    // Verify new flow was used...
}
```

No external dependencies, no network calls, no complex mocking frameworks. Just pure, fast unit tests.

## The Architectural Benefits

This approach delivers several key benefits:

### 1. **Vendor Independence**
Your business logic never touches vendor-specific code. You can switch from LaunchDarkly to Split to Flagsmith without changing a single line of business logic.

### 2. **Testability**
Every component that uses feature flags can be easily unit tested with mock implementations.

### 3. **Single Responsibility**
Your feature flag logic is separated from your business logic. Each has one reason to change.

### 4. **Extensibility**
Need to add logging to all flag evaluations? Implement a decorator around your adapter. Want to cache flag values? Another decorator. The Open-Closed Principle in action.

```go
type LoggingFeatureFlags struct {
    wrapped FeatureFlags
    logger  Logger
}

func (l *LoggingFeatureFlags) GetBoolFlag(ctx context.Context, key string, defaultValue bool) bool {
    result := l.wrapped.GetBoolFlag(ctx, key, defaultValue)
    l.logger.Info("Feature flag evaluated", 
        "key", key, 
        "result", result, 
        "default", defaultValue)
    return result
}
```

## Conclusion: Craftsmanship in Action

Feature flags, when implemented correctly, are more than just a deployment tool—they're an enabler of true incremental development and continuous delivery. By applying the SOLID principles and depending on abstractions rather than implementations, we create systems that are:

- **Flexible** - Easy to change providers or add new capabilities
- **Testable** - Every component can be isolated and tested
- **Maintainable** - Clear separation of concerns and responsibilities
- **Robust** - No single point of failure or vendor lock-in

OpenFeature provides us with the abstraction we need to build these systems correctly. It allows us to focus on what matters most: delivering value to our users through clean, well-crafted code.

Remember: The goal is not just to make our code work today, but to make it work well tomorrow, next month, and next year. That's the mark of true craftsmanship.

As Uncle Bob reminds us: "The only way to go fast is to go well." Feature flags, implemented with clean architecture principles, help us do exactly that.

