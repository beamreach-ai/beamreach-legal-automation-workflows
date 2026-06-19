# Icons & Branding

Place your firm's branding assets in this folder.

## Required files

| File | Used in | Notes |
|------|---------|-------|
| `logo.png` | Confirmation email header | Recommended size: 300×80 px |
| `logo.svg` | Workflow diagrams | Vector preferred |
| `favicon.ico` | Any web forms you link in emails | 32×32 |

## Using your logo in the email node

In the **Send Confirmation Email** node, update the HTML body to reference your logo:

```html
<img src="https://yourfirm.com/logo.png" alt="Your Firm" width="200" />
```

Or if you host assets in n8n's static files folder, use a relative path.

## Default assets

The default `clio.svg` icon lives in the `n8n-nodes-clio` package (`nodes/Clio/clio.svg`).
Replace it with Clio's official branding if you have a partner license.
