# Reference: Stripe Webhook Patterns

> Supplementary detail for payments-patterns.md. Load only when implementing Stripe webhooks.

## Webhook Events to Handle

| Event | When | What to do |
|-------|------|------------|
| `checkout.session.completed` | User completes payment | Activate subscription, update user plan |
| `customer.subscription.updated` | Plan changed (upgrade/downgrade) | Update plan in DB, adjust feature access |
| `customer.subscription.deleted` | Subscription cancelled | Downgrade to free, remove paid features |
| `invoice.payment_failed` | Payment failed | Notify user, start grace period, begin dunning |
| `invoice.payment_succeeded` | Recurring payment processed | Update `current_period_end`, send receipt |
| `customer.subscription.trial_will_end` | Trial ending in 3 days | Send reminder email |

## Webhook Handler Pattern (Laravel)

```php
// app/Http/Controllers/StripeWebhookController.php
public function handle(Request $request)
{
    $payload = $request->getContent();
    $sig = $request->header('Stripe-Signature');

    try {
        $event = \Stripe\Webhook::constructEvent($payload, $sig, config('services.stripe.webhook_secret'));
    } catch (\Exception $e) {
        return response('Invalid signature', 400);
    }

    match ($event->type) {
        'checkout.session.completed' => $this->handleCheckoutCompleted($event->data->object),
        'customer.subscription.updated' => $this->handleSubscriptionUpdated($event->data->object),
        'customer.subscription.deleted' => $this->handleSubscriptionDeleted($event->data->object),
        'invoice.payment_failed' => $this->handlePaymentFailed($event->data->object),
        default => null,
    };

    return response('OK', 200);
}
```

## Webhook Handler Pattern (Node.js / Hono)

```typescript
app.post('/api/webhooks/stripe', async (c) => {
  const sig = c.req.header('stripe-signature')
  const body = await c.req.text()

  const event = stripe.webhooks.constructEvent(body, sig, WEBHOOK_SECRET)

  switch (event.type) {
    case 'checkout.session.completed':
      await activateSubscription(event.data.object)
      break
    case 'customer.subscription.deleted':
      await downgradeToFree(event.data.object)
      break
    case 'invoice.payment_failed':
      await handleFailedPayment(event.data.object)
      break
  }

  return c.text('OK', 200)
})
```

## Dunning Sequence (Failed Payments)

```
Day 0: Payment fails → send email "Payment failed, update your card"
Day 3: Retry payment → if fails, send "Your account will be downgraded in 4 days"
Day 7: Retry payment → if fails, send "Last chance — update payment method"
Day 10: Final retry → if fails, cancel subscription → send "Your account has been downgraded"
```

## Idempotency

Always check if the event was already processed:

```
1. Store event ID in a processed_events table
2. Before processing, check if event.id exists
3. If exists → return 200 (already handled)
4. If not → process + insert event.id
```
