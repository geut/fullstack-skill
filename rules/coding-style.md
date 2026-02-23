---
title: Coding Style
impact: LOW-MEDIUM
tags: [code, clarity, maintenance, style]
---
# Frontend Coding Style

This is an on-purpose "general" rule. Write code that is easily readable for humans. 

## Why

- Code is read more than written. 
- Readable code helps to maintain it. 
- This is helpful in case the humans around the project need to inspect the codebase at any point. 

## Avoid Unnecessary Complex Components

Specially useful when refactoring code. Check for convoluted statements that can be abstracted into smaller components.

```tsx
// Bad: complex component with lots of conditions
function PaymentStatus({ status = 'approved' }: { status: 'approved' | 'pending' | 'rejected' }) {
  return (
    <div className="col-span-2 flex flex-col gap-2">
      <h4>Status</h4>
      <div className={cn('flex items-center gap-4 rounded-md p-2', status === 'approved' ? 'bg-green-500/30' : status === 'pending' ? 'bg-muted' : 'bg-red-500/30')}>
        {status === 'approved' && <CheckCircleIcon className="size-6" strokeWidth={2} />}
        {status === 'pending' && <ClockIcon className="size-6" strokeWidth={2} />}
        {status === 'rejected' && <XCircleIcon className="size-6" strokeWidth={2} />}
        {' '}
        <p className="text-lg">{status === 'approved' ? 'Payment approved' : status === 'pending' ? 'Pending' : 'Payment declined'}</p>
      </div>
      <span>{status === 'approved' ? 'You can share your gift card now.' : status === 'pending' ? 'Once payment is confirmed you will be able to share your gift card.' : 'Payment declined. Please, try again later.'}</span>
    </div>
  )
}

// Good: encapsulated logic into a reusable component makes it easier to read and to express the intention
function PaymentStatus({ status = 'approved' }: { status: 'approved' | 'pending' | 'rejected' }) {
  return (
    <div className="col-span-2 flex flex-col gap-2">
      <h4>Status</h4>
      <StatusFeedback currentStatus={status} />
      <StatusInfo currentStatus={status} />
    </div>
  )
}
```

You can read more about idiomatic react here: https://react.dev/reference/rules


## Explicit Over Implicit

Again, a useful rule when refactoring code. Variables named with clear purpose helps debugging.

```js
// Bad: complex long line 
const carryForwardOptions = getCarryForwardResultsForSourceQuestion(pageData.value?.carryForwardRules[0]?.sourceSurveyQuestionId, pageData.value?.carryForwardRules[0]?.carryForwardUnselected) ?? []

// Good: use explicit variables to indicate meaning
const cfQuestionId = pageData.value?.carryForwardRules[0]?.sourceSurveyQuestionId
const cfUnselectedQuestions = pageData.value?.carryForwardRules[0]?.carryForwardUnselected
const carryForwardOptions = getCarryForwardResultsForSourceQuestion(cfQuestionId, cfUnselectedQuestions) ?? []
```

## Rules

1. Prefer small, single-purpose components over large conditional blocks.
2. Extract repeated status logic into components like StatusFeedback and StatusInfo.
3. Use explicit variable names for nested optional values instead of long inline expressions.
4. Break long call chains into named variables to clarify intent and ease debugging.