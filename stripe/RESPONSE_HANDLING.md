# Payment Response Handling Documentation

**Version:** 2.0.2-ENHANCED  
**Date:** November 23, 2025  
**Status:** ✅ Production Ready

---

## Overview

The enhanced stripe-hitter now includes comprehensive payment response handling that provides detailed status information for all payment outcomes including approved, declined, failed, pending, and requires_action statuses.

---

## Response Status Types

### Payment Status Constants

```javascript
PaymentStatus = {
    APPROVED: 'approved',           // Payment successful
    DECLINED: 'declined',           // Card declined
    FAILED: 'failed',               // Processing failed
    PENDING: 'pending',             // Payment pending
    REQUIRES_ACTION: 'requires_action',  // 3DS or additional action needed
    PROCESSING: 'processing',       // Payment processing
    SUCCEEDED: 'succeeded',         // Payment succeeded
    REQUIRES_PAYMENT_METHOD: 'requires_payment_method',  // New payment method needed
    CANCELED: 'canceled'            // Payment canceled
}
```

---

## Response Format

### Approved Payment Response

```json
{
  "timestamp": "2025-11-23T08:52:00.000Z",
  "success": true,
  "status": "approved",
  "message": "Payment approved successfully",
  "attempts": 1,
  "card": {
    "last4": "3459",
    "expiration": "05/26",
    "type": "Mastercard"
  },
  "details": {
    "paymentMethodId": "pm_1234567890",
    "paymentIntentId": "pi_1234567890",
    "amount": 15000,
    "currency": "usd",
    "requires3DS": false
  }
}
```

### Declined Payment Response

```json
{
  "timestamp": "2025-11-23T08:52:00.000Z",
  "success": false,
  "status": "declined",
  "message": "Card was declined by the issuer",
  "attempts": 1,
  "card": {
    "last4": "3459",
    "expiration": "05/26",
    "type": "Mastercard"
  },
  "details": {
    "code": "card_declined",
    "type": "card_error",
    "message": "Your card was declined",
    "declineReason": "Card was declined by the issuer"
  }
}
```

### Processing Payment Response

```json
{
  "timestamp": "2025-11-23T08:52:00.000Z",
  "success": false,
  "status": "processing",
  "message": "Payment is being processed",
  "attempts": 1,
  "card": {
    "last4": "3459",
    "expiration": "05/26",
    "type": "Mastercard"
  },
  "details": {
    "paymentIntentId": "pi_1234567890",
    "amount": 15000,
    "currency": "usd",
    "status": "processing"
  }
}
```

### Requires 3D Secure Response

```json
{
  "timestamp": "2025-11-23T08:52:00.000Z",
  "success": false,
  "status": "requires_action",
  "message": "Payment requires additional action (3D Secure or similar)",
  "attempts": 1,
  "card": {
    "last4": "3459",
    "expiration": "05/26",
    "type": "Mastercard"
  },
  "details": {
    "paymentIntentId": "pi_1234567890",
    "requires3DS": true,
    "nextAction": "redirect_to_url",
    "redirectUrl": "https://stripe.com/3ds/..."
  }
}
```

---

## Decline Reasons

The response handler includes detailed decline reasons for common card decline codes:

| Code | Reason |
|------|--------|
| `card_declined` | Card was declined by the issuer |
| `expired_card` | Card has expired |
| `incorrect_cvc` | Incorrect CVC provided |
| `processing_error` | Processing error occurred |
| `rate_limit` | Rate limit exceeded |
| `lost_card` | Card reported as lost |
| `stolen_card` | Card reported as stolen |
| `insufficient_funds` | Insufficient funds |
| `do_not_honor` | Card issuer declined the transaction |
| `generic_decline` | Card declined for unknown reason |
| `checkout_amount_mismatch` | Amount mismatch with checkout session |
| `invalid_account` | Invalid account |
| `invalid_amount` | Invalid amount |
| `invalid_currency` | Invalid currency |

---

## API Endpoints

### Payment with Credit Card

**Endpoint:** `GET /stripe/checkout-based/url/{checkoutUrl}/pay/cc/{card}`

**Parameters:**
- `checkoutUrl` - URL-encoded Stripe checkout URL
- `card` - Card in format: `number|month|year|cvc`

**Example:**
```bash
curl "http://localhost:8080/stripe/checkout-based/url/https%3A%2F%2Fcheckout.stripe.com%2F...%23encoded/pay/cc/4342570032943459%7C05%7C26%7C222"
```

**Response:**
```json
{
  "timestamp": "2025-11-23T08:52:00.000Z",
  "success": true,
  "status": "approved",
  "message": "Payment approved successfully",
  "attempts": 1,
  "card": {
    "last4": "3459",
    "expiration": "05/26",
    "type": "Mastercard"
  },
  "details": { ... }
}
```

---

### Payment with Generated Card

**Endpoint:** `GET /stripe/checkout-based/url/{checkoutUrl}/pay/gen/{bin}?retry=n`

**Parameters:**
- `checkoutUrl` - URL-encoded Stripe checkout URL
- `bin` - Bank Identification Number (default: 424242)
- `retry` - Number of retries on failure (optional)

**Example:**
```bash
curl "http://localhost:8080/stripe/checkout-based/url/https%3A%2F%2Fcheckout.stripe.com%2F...%23encoded/pay/gen/424242?retry=3"
```

**Response:**
```json
{
  "timestamp": "2025-11-23T08:52:00.000Z",
  "success": true,
  "status": "approved",
  "message": "Payment approved successfully",
  "attempts": 1,
  "card": {
    "last4": "7878",
    "expiration": "04/26",
    "type": "Visa"
  },
  "details": { ... }
}
```

---

## Response Helper Functions

### formatPaymentResponse(response, statusCode)

Formats a raw Stripe response into a standardized payment response.

```javascript
const { formatPaymentResponse } = require('./gateways/stripe/checkout-based/response-handler');

const response = formatPaymentResponse(stripeResponse, 200);
console.log(response.status);  // 'approved' or 'declined'
console.log(response.message); // Human-readable message
```

---

### createDetailedResponse(paymentResult)

Creates a detailed response from a payment attempt result.

```javascript
const { createDetailedResponse } = require('./gateways/stripe/checkout-based/response-handler');

const detailed = createDetailedResponse(paymentResult);
console.log(detailed.card.type);  // Card type
console.log(detailed.details);    // Full details
```

---

### isPaymentApproved(response)

Checks if a payment response indicates approval.

```javascript
const { isPaymentApproved } = require('./gateways/stripe/checkout-based/response-handler');

if (isPaymentApproved(response)) {
    console.log('Payment was approved!');
}
```

---

### isPaymentDeclined(response)

Checks if a payment response indicates decline.

```javascript
const { isPaymentDeclined } = require('./gateways/stripe/checkout-based/response-handler');

if (isPaymentDeclined(response)) {
    console.log('Payment was declined:', response.declineReason);
}
```

---

### isPaymentPending(response)

Checks if a payment response indicates pending status.

```javascript
const { isPaymentPending } = require('./gateways/stripe/checkout-based/response-handler');

if (isPaymentPending(response)) {
    console.log('Payment is still processing');
}
```

---

## HTTP Status Codes

The API returns appropriate HTTP status codes:

| Code | Meaning |
|------|---------|
| `200` | Payment approved |
| `402` | Payment declined or failed |
| `405` | Method not allowed |
| `500` | Server error |

---

## Card Type Detection

The response handler automatically detects card types:

- **Visa** - Starts with 4
- **Mastercard** - Starts with 51-55
- **American Express** - Starts with 34 or 37
- **Discover** - Starts with 6011 or 65
- **Diners Club** - Starts with 36, 38, 54, or 55
- **JCB** - Starts with 2131, 1800, or 35

---

## Error Handling

All errors are returned in a consistent format:

```json
{
  "timestamp": "2025-11-23T08:52:00.000Z",
  "success": false,
  "status": "failed",
  "error": "Payment processing failed",
  "message": "Error message details",
  "errorCode": "error_code_if_available"
}
```

---

## Logging

The system logs all payment attempts with detailed information:

```
[PAYMENT] Processing cc payment for https://checkout.stripe.com/c/pay/cs_live_...
[PAYMENT] APPROVED - Payment approved successfully
```

---

## Usage Examples

### Example 1: Check Card with Provided Card

```bash
curl "http://localhost:8080/stripe/checkout-based/url/https%3A%2F%2Fcheckout.stripe.com%2Fc%2Fpay%2Fcs_live_abc123%23encoded/pay/cc/4342570032943459%7C05%7C26%7C222"
```

### Example 2: Check Card with Generated Cards (3 Retries)

```bash
curl "http://localhost:8080/stripe/checkout-based/url/https%3A%2F%2Fcheckout.stripe.com%2Fc%2Fpay%2Fcs_live_abc123%23encoded/pay/gen/424242?retry=3"
```

### Example 3: Parse Response in JavaScript

```javascript
const response = await fetch('http://localhost:8080/stripe/checkout-based/url/.../pay/cc/...');
const result = await response.json();

if (result.success) {
    console.log('✅ Payment approved!');
    console.log('Card:', result.card.type, result.card.last4);
} else {
    console.log('❌ Payment declined');
    console.log('Reason:', result.details.declineReason);
}
```

---

## Best Practices

1. **Always check the `success` field** - Don't rely solely on HTTP status codes
2. **Log the timestamp** - Helps with debugging and audit trails
3. **Store the `status` field** - Use it for analytics and reporting
4. **Handle 3DS responses** - Check for `requires_action` status
5. **Retry on processing** - If status is `processing`, retry after a delay
6. **Use decline reasons** - Provide helpful messages to users

---

## Changelog

### Version 2.0.2-ENHANCED (November 23, 2025)

**Added:**
- Comprehensive response formatting module
- Payment status constants
- Detailed decline reasons
- Helper functions for response checking
- Card type detection
- Batch response formatting
- 3D Secure handling
- Logging for all payment attempts

**Improved:**
- Response consistency
- Error messages
- Status code handling
- Payment tracking

---

## Support

For issues or questions about response handling:

1. Check the response status and message
2. Review the decline reason if payment failed
3. Check server logs for detailed information
4. Refer to Stripe API documentation for error codes

---

**Status:** ✅ Production Ready  
**Last Updated:** November 23, 2025  
**Version:** 2.0.2-ENHANCED

