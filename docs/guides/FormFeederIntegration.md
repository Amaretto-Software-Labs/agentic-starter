# FormFeeder Integration Guide

## Overview
FormFeeder is a privacy-first form handler that forwards submissions straight to your destinations (email, Slack, webhook) without persisting sensitive data. Base URL: `https://api.formfeeder.io`.

## 1. Create & Store Form IDs
- **Dashboard:** Use https://panel.formfeeder.io for production forms (analytics, multi-connectors, file uploads).
- **Private forms:** Generate from the homepage or via API:
  ```bash
  curl -X POST "https://api.formfeeder.io/v1/forms/from-email" \
    -H "Content-Type: application/json" \
    -d '{
      "email": "[email protected]",
      "allowedDomain": "https://www.example.com"
    }'
  ```
  Response includes `formId` and the normalized `allowedDomain`.

## 2. Hook Up Forms
- HTML endpoints: `https://api.formfeeder.io/v1/form/<FORM_ID>`
- Supported content types:
  - `application/x-www-form-urlencoded` - default HTML forms
  - `multipart/form-data` - required for uploads (`enctype="multipart/form-data"`)
  - `application/json` - fetch/axios style submissions
- Basic example:
  ```html
  <form action="https://api.formfeeder.io/v1/form/abcd1234" method="POST">
    <input type="text" name="name" required />
    <input type="email" name="email" required />
    <button type="submit">Send</button>
  </form>
  ```

## 3. Domain Allow Lists
Submissions must include an `Origin` header that matches the form's allow list or the API returns `401`.
- Add domains in dashboard **Security -> Allowed Domains** or via API. Examples: `http://localhost:3000`, `https://staging.example.com`, `https://www.example.com`.
- Normalization removes trailing slashes, keeps ports, and enforces protocol.
- Audit production forms regularly and keep environment-specific forms (`dev`, `staging`, `prod`).

## 4. Rate & File Limits
- Defaults: 10 submissions per minute per IP per form, 100 requests/min global per IP.
- File uploads: 5 files max, 5 MB each, 10 MB total (configurable). Supported types include images, PDFs, DOCX, XLSX, ZIP. Configure stricter accept lists client-side.
- When limits exceed, responses include `Retry-After` or error payload (`429` for rate, `400/413` for files).

## 5. Connectors
Define connectors in the dashboard or via JSON:
```json
{
  "connectors": [
    {
      "type": "MailJet",
      "config": {
        "to": "[email protected]",
        "subject": "New Contact: {{name}}",
        "replyToField": "email"
      }
    },
    {
      "type": "Slack",
      "config": {
        "webhookUrl": "https://hooks.slack.com/services/...",
        "showFileMetadata": true
      }
    },
    {
      "type": "Webhook",
      "config": {
        "url": "https://api.example.com/forms",
        "headers": { "Authorization": "Bearer token" }
      }
    }
  ]
}
```
- MailJet attachments respect FormFeeder file caps; customize subjects with `{{field}}` tokens.
- Slack connectors send rich messages (files remain hosted by FormFeeder for 24h in privacy mode).
- Webhooks receive JSON payloads with temporary file URLs; verify signatures using the `x-formfeeder-signature` header (HMAC SHA-256).

## 6. Privacy Mode & Security
- Private forms default to privacy mode: data is processed in-memory and wiped after connectors run; file artifacts are temporary.
- Ensure HTTPS endpoints, validate file types client-side, and handle rate-limit errors gracefully.
- Monitor connector logs for errors and retry conditions (max 3 attempts with exponential backoff).

## 7. Testing Checklist
- Submit from every environment domain to confirm allow-list coverage.
- Run Playwright smoke tests to verify marketing flows.
- Use the dashboard "Test Connectors" tool or `curl` to validate configuration.
- Track rate-limit headers (`X-RateLimit-*`) during load testing.

## 8. Bot Protection & UX Handling
- **Honeypot field:** Add a hidden input (e.g. `name="companyWebsite"`) and reject submissions where it is filled. Keep it visually hidden with CSS rather than `display:none` to avoid bots skipping it.
- **Timestamp check:** Capture form render time in a hidden field and reject submissions completed in less than a few hundred milliseconds.
- **Rate limiting:** FormFeeder already enforces per-IP limits; combine with client-side throttling to disable the submit button until the request resolves.
- **reCAPTCHA/hCaptcha (optional):** Use a client-side challenge then include the token as an extra field; validate server-side via a webhook if higher assurance is needed.

### JavaScript submission pattern
```tsx
function ContactForm() {
  const [status, setStatus] = useState<'idle' | 'loading' | 'success' | 'error'>('idle');
  const [message, setMessage] = useState<string>('');

  const handleSubmit = async (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();
    setStatus('loading');
    setMessage('');

    const form = event.currentTarget;
    const formData = new FormData(form);

    try {
      const response = await fetch(`https://api.formfeeder.io/v1/form/${import.meta.env.VITE_CONTACT_FORM_ID}`, {
        method: 'POST',
        body: formData,
        headers: {
          // surface the origin explicitly for local dev if needed
          // 'X-Forwarded-Origin': window.location.origin,
        },
      });

      if (response.ok) {
        setStatus('success');
        setMessage('Thanks! We'll be in touch soon.');
        form.reset();
        form.querySelectorAll('[data-honeypot]').forEach((el) => ((el as HTMLInputElement).value = ''));
      } else {
        const data = await response.json().catch(() => ({}));
        const retryAfter = data?.retryAfter ? ` Please retry in ${data.retryAfter} seconds.` : '';
        setStatus('error');
        setMessage(data?.message || `Submission failed (${response.status}).${retryAfter}`);
      }
    } catch (error) {
      console.error(error);
      setStatus('error');
      setMessage('Network error. Please check your connection and try again.');
    }
  };

  return (
    <form onSubmit={handleSubmit} aria-busy={status === 'loading'}>
      <div hidden aria-hidden data-honeypot>
        <label htmlFor="companyWebsite">Company Website</label>
        <input id="companyWebsite" name="companyWebsite" autoComplete="off" tabIndex={-1} />
      </div>

      {/* form fields... */}

      <button type="submit" disabled={status === 'loading'}>
        {status === 'loading' ? 'Sending...' : 'Submit'}
      </button>

      {status === 'success' && <p role="status" className="text-success">{message}</p>}
      {status === 'error' && <p role="alert" className="text-error">{message}</p>}
    </form>
  );
}
```
- Always `preventDefault` to avoid double submits, disable the button when loading, and reset success/error indicators on resubmission.
- Parse non-200 responses and surface FormFeeder messages (`RATE_LIMIT_EXCEEDED`, file validation). Include retry hints when `Retry-After` is present.
- Log unexpected errors to your observability stack; consider using `fetch` timeouts or AbortController to avoid hanging requests.
