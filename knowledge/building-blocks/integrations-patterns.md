# Building Block: Integrations Patterns (Email, Notifications, File Storage)

Covers: transactional email, push notifications, in-app notifications, file uploads, image optimization, and CDN.

---

## EMAIL

### Decision Matrix

| Solution | Best For | Cost |
|----------|----------|------|
| **Resend** | Modern apps (best DX, React Email templates) | Free 3k/mo, $20/mo for 50k |
| **SendGrid** | High volume, marketing + transactional | Free 100/day, $19.95/mo for 50k |
| **AWS SES** | High volume, cost-sensitive | $0.10 per 1,000 emails |
| **Postmark** | Transactional only (best deliverability) | Free 100/mo, $15/mo for 10k |

**Recommendation:** Resend for most projects. AWS SES at scale.

### Resend Setup

```typescript
import { Resend } from 'resend'
const resend = new Resend(process.env.RESEND_API_KEY)

await resend.emails.send({
  from: 'App <hello@yourdomain.com>',
  to: userEmail,
  subject: 'Welcome!',
  react: WelcomeEmail({ name }),  // React Email component
})
```

### Common Transactional Emails

| Email | Trigger | Priority |
|-------|---------|----------|
| Welcome / Verify email | Signup | Must have |
| Password reset | Request | Must have |
| Invoice / Receipt | Payment | Must have (if payments) |
| Team invitation | Admin invites | Must have (if teams) |

### Rules
- **Send async** — never block API response waiting for email. Use queue or background job.
- **Verify domain** — set up SPF, DKIM, DMARC. Without these = spam folder.
- **Include unsubscribe link** — required by law for non-critical emails.

---

## PUSH NOTIFICATIONS

| Solution | Best For | Cost |
|----------|----------|------|
| **Expo Notifications** | Expo/React Native | Free |
| **Firebase FCM** | Any mobile, web push | Free |
| **OneSignal** | Advanced targeting, multi-channel | Free 10k subs, $9/mo+ |

### Expo Pattern
```typescript
const token = (await Notifications.getExpoPushTokenAsync()).data
// Save token to backend, then send:
await fetch('https://exp.host/--/api/v2/push/send', {
  method: 'POST',
  body: JSON.stringify({ to: token, title: 'New message', body: 'From Alice' }),
})
```

---

## IN-APP NOTIFICATIONS

### Database Schema
```sql
notifications (id UUID PK, user_id FK, type TEXT, title TEXT, body TEXT, link TEXT, read BOOLEAN DEFAULT false, created_at TIMESTAMPTZ)
CREATE INDEX idx_notifications_user_unread ON notifications(user_id, read) WHERE read = false;
```

### Pattern
Bell icon (unread count) → dropdown with notifications → click = mark read + navigate.

### When to Use Each Channel

| Event | In-App | Email | Push |
|-------|--------|-------|------|
| New comment | Yes | If not active | If not active |
| Team invitation | Yes | Always | No |
| Payment receipt | Yes | Always | No |
| Mentioned you | Yes | If not active | If not active |

**Rule:** In-app always. Email for records. Push only for time-sensitive + user inactive.

---

## FILE STORAGE

### Decision Matrix

| Solution | Best For | Cost |
|----------|----------|------|
| **Uploadthing** | Next.js apps (simplest DX) | Free 2GB, $10/mo for 100GB |
| **Supabase Storage** | Already using Supabase (RLS integration) | Free 1GB, included in plan |
| **Cloudinary** | Image/video heavy (transforms, CDN) | Free 25 credits/mo |
| **AWS S3** | Enterprise, full control | ~$0.023/GB/mo |
| **Vercel Blob** | Vercel projects (zero config) | Free 256MB, $0.15/GB/mo |

**Recommendation:** Uploadthing for Next.js. Supabase Storage if already using Supabase. Cloudinary for heavy image transforms.

### Upload Pattern (Presigned URLs)

```
Client requests upload URL from API
→ Server creates presigned URL
→ Client uploads directly to storage (not through your server)
→ Server saves file metadata in database
```

**Why:** Your server never handles file bytes = no memory issues, no timeouts.

### Image Optimization

| Context | Max Width | Format | Loading |
|---------|-----------|--------|---------|
| Hero/banner | 1200-1600px | AVIF/WebP | Priority (eager) |
| Card thumbnail | 400-600px | WebP | Lazy |
| Avatar | 128-256px | WebP | Lazy |

Use Next.js `<Image>` (auto-optimizes) or Cloudinary URL transforms.

### Access Control

| Pattern | When |
|---------|------|
| **Public** | Avatars, product images |
| **Signed URLs** | Private documents (time-limited, 15min-24hr) |
| **RLS (Supabase)** | Per-user files |

### CDN Rules
- Serve all files through CDN (all providers above include CDN)
- `Cache-Control: public, max-age=31536000, immutable` for content-addressed files
- Invalidate by changing URL, not purging cache

---

## OUTGOING WEBHOOKS (Your App → External)

For B2B/integrations: sign payloads (HMAC-SHA256), deliver via queue, retry with backoff (1min, 5min, 30min, 2hr, 24hr), log every attempt.

## Common Pitfalls

- **Don't send emails synchronously.** Queue or background job.
- **Don't skip domain verification.** Unverified = spam.
- **Don't push-notify everything.** Users disable aggressive notifications.
- **Don't upload through your server.** Presigned URLs.
- **Don't store files in the database.** Store the URL/key.
- **Don't skip file type validation.** Validate MIME on client AND server.
- **Don't allow unlimited file sizes.** Set max per upload type.
