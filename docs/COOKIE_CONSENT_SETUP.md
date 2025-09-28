# Cookie Consent Setup Guide for Astro

This guide walks through implementing GDPR-compliant cookie consent on an Astro static website, including Microsoft Clarity analytics integration.

## Table of Contents
1. [Overview](#overview)
2. [Implementation Steps](#implementation-steps)
3. [Microsoft Clarity Integration](#microsoft-clarity-integration)
4. [Cookie Policy Page](#cookie-policy-page)
5. [Footer Cookie Settings Button](#footer-cookie-settings-button)
6. [Testing](#testing)
7. [Troubleshooting](#troubleshooting)

## Overview

### What This Setup Provides
- GDPR-compliant cookie consent banner
- Cookie preference management modal
- Microsoft Clarity analytics with consent mode
- Cookie Policy page
- Footer button to manage preferences anytime
- Auto-clearing of tracking cookies when consent is revoked

### Technologies Used
- **vanilla-cookieconsent v3** - Cookie consent management library
- **Microsoft Clarity** - Analytics with session recordings
- **Astro** - Static site generator

## Implementation Steps

### Step 1: Add Cookie Consent to Your Layout

In your main layout file (e.g., `src/layouts/Layout.astro`), add the following in the `<head>` section:

```html
<!-- Cookie Consent -->
<link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/orestbida/cookieconsent@3.0.1/dist/cookieconsent.css">
<script defer src="https://cdn.jsdelivr.net/gh/orestbida/cookieconsent@3.0.1/dist/cookieconsent.umd.js"></script>
<script is:inline>
  window.addEventListener('DOMContentLoaded', function() {
    function initCookieConsent() {
      if (typeof CookieConsent === 'undefined') {
        setTimeout(initCookieConsent, 100);
        return;
      }
      
      window.CookieConsent = CookieConsent;
      
      window.CookieConsent.run({
        categories: {
          necessary: {
            enabled: true,
            readOnly: true
          },
          analytics: {
            enabled: false,
            autoClear: {
              cookies: [
                {
                  name: /^_clck/,
                },
                {
                  name: /^_clsk/,
                },
                {
                  name: 'CLID',
                }
              ]
            }
          },
          functionality: {
            enabled: false
          }
        },
        
        language: {
          default: 'en',
          translations: {
            en: {
              consentModal: {
                title: 'We use cookies',
                description: 'We use cookies to enhance your browsing experience and analyze our traffic. By clicking "Accept All", you consent to our use of cookies.',
                acceptAllBtn: 'Accept All',
                acceptNecessaryBtn: 'Reject All',
                showPreferencesBtn: 'Manage Preferences',
                closeIconLabel: 'Close',
                footer: '<a href="/privacy">Privacy Policy</a> | <a href="/cookies">Cookie Policy</a>'
              },
              preferencesModal: {
                title: 'Cookie Preferences',
                acceptAllBtn: 'Accept All',
                acceptNecessaryBtn: 'Accept Necessary Only',
                savePreferencesBtn: 'Save Preferences',
                closeIconLabel: 'Close',
                sections: [
                  {
                    title: 'Cookie Usage',
                    description: 'We use cookies to ensure the basic functionalities of the website and to enhance your online experience. You can choose for each category to opt-in/out whenever you want.'
                  },
                  {
                    title: 'Strictly Necessary Cookies',
                    description: 'These cookies are essential for the proper functioning of the website. Without these cookies, the website would not work properly.',
                    linkedCategory: 'necessary'
                  },
                  {
                    title: 'Analytics Cookies',
                    description: 'These cookies allow us to measure traffic and analyze your behavior to improve our service. We use Microsoft Clarity for session recordings and heatmaps to understand how you interact with our website.',
                    linkedCategory: 'analytics'
                  },
                  {
                    title: 'Functionality Cookies',
                    description: 'These cookies enable the website to provide enhanced functionality and personalization based on your interaction with the website.',
                    linkedCategory: 'functionality'
                  },
                  {
                    title: 'More Information',
                    description: 'For any queries in relation to our policy on cookies and your choices, please <a href="/contact">contact us</a> or view our <a href="/privacy">Privacy Policy</a> and <a href="/cookies">Cookie Policy</a>.'
                  }
                ]
              }
            }
          }
        },
        
        guiOptions: {
          consentModal: {
            layout: 'box',
            position: 'bottom right',
            flipButtons: false,
            equalWeightButtons: true
          },
          preferencesModal: {
            layout: 'box',
            position: 'right',
            flipButtons: false,
            equalWeightButtons: true
          }
        },
        
        onFirstConsent: function({cookie}) {},
        onConsent: function({cookie}) {},
        onChange: function({changedCategories, changedServices}) {},
        onModalReady: function({modalName}) {}
      });
    }
    
    initCookieConsent();
  });
</script>
```

### Step 2: Configuration Options Explained

#### Categories
- **necessary**: Always enabled, cannot be disabled by users
- **analytics**: For tracking cookies (Microsoft Clarity)
- **functionality**: For preference cookies (optional)

#### Auto-Clear Configuration
The `autoClear` option automatically removes cookies when consent is revoked:
```javascript
autoClear: {
  cookies: [
    {
      name: /^_clck/,  // Regex pattern for Clarity cookies
    },
    {
      name: /^_clsk/,
    },
    {
      name: 'CLID',    // Exact cookie name
    }
  ]
}
```

#### GUI Options
- **layout**: 'box' (rectangular) or 'cloud' (rounded)
- **position**: 'bottom right', 'bottom left', 'top right', 'top left', 'bottom center', 'top center'
- **flipButtons**: Swap accept/reject button positions
- **equalWeightButtons**: Make buttons the same size

## Microsoft Clarity Integration

Add this code AFTER the Cookie Consent script in your layout:

```html
<!-- Microsoft Clarity Analytics with Consent API v2 -->
<script type="text/javascript" is:inline>
  (function(c,l,a,r,i,t,y){
    c[a]=c[a]||function(){(c[a].q=c[a].q||[]).push(arguments)};
    t=l.createElement(r);t.async=1;t.src="https://www.clarity.ms/tag/"+i;
    y=l.getElementsByTagName(r)[0];y.parentNode.insertBefore(t,y);
  })(window, document, "clarity", "script", "YOUR_CLARITY_PROJECT_ID");
  
  window.clarity('consentv2', {
    ad_storage: "denied",
    analytics_storage: "denied"
  });
  
  window.addEventListener('cc:onConsent', function() {
    if (typeof window.CookieConsent !== 'undefined') {
      const userPreferences = window.CookieConsent.getUserPreferences();
      
      if (userPreferences && userPreferences.acceptedCategories && userPreferences.acceptedCategories.includes('analytics')) {
        window.clarity('consentv2', {
          ad_storage: "denied",
          analytics_storage: "granted"
        });
      }
    }
  });
  
  window.addEventListener('cc:onChange', function() {
    if (typeof window.CookieConsent !== 'undefined') {
      const userPreferences = window.CookieConsent.getUserPreferences();
      
      if (userPreferences && userPreferences.acceptedCategories && userPreferences.acceptedCategories.includes('analytics')) {
        window.clarity('consentv2', {
          ad_storage: "denied",
          analytics_storage: "granted"
        });
      } else {
        window.clarity('consentv2', {
          ad_storage: "denied",
          analytics_storage: "denied"
        });
      }
    }
  });
</script>
```

### Getting Your Clarity Project ID
1. Sign up at [clarity.microsoft.com](https://clarity.microsoft.com)
2. Create a new project
3. Copy your Project ID from the setup instructions
4. Replace `YOUR_CLARITY_PROJECT_ID` in the script above

### How Clarity Consent Works
- **Initial state**: Both `ad_storage` and `analytics_storage` are "denied"
- **User accepts analytics**: `analytics_storage` becomes "granted"
- **User revokes consent**: `analytics_storage` returns to "denied"
- **Note**: We keep `ad_storage` as "denied" since we don't use advertising

## Cookie Policy Page

Create a Cookie Policy page at `src/pages/cookies.astro`:

```astro
---
import Layout from '../layouts/Layout.astro';

const title = 'Cookie Policy';
const description = 'How we use cookies on our website';
---

<Layout title={title} description={description}>
  <main class="min-h-screen bg-white">
    <section class="py-16">
      <div class="max-w-4xl mx-auto px-8">
        <h1 class="text-4xl font-bold mb-8">Cookie Policy</h1>
        
        <h2 class="text-2xl font-semibold mt-8 mb-4">What cookies do we use?</h2>
        
        <h3 class="text-xl font-semibold mt-6 mb-3">Essential Cookies</h3>
        <p class="mb-4">These cookies are necessary for the website to function properly.</p>
        <ul class="list-disc ml-6 mb-4">
          <li><strong>cc_cookie</strong>: Stores your cookie consent preferences (6 months retention)</li>
        </ul>
        
        <h3 class="text-xl font-semibold mt-6 mb-3">Analytics Cookies</h3>
        <p class="mb-4">We use Microsoft Clarity to understand how visitors interact with our website.</p>
        <ul class="list-disc ml-6 mb-4">
          <li><strong>_clck</strong>: Links multiple page views by the same user (1 year retention)</li>
          <li><strong>_clsk</strong>: Connects multiple page views in a single session (1 day retention)</li>
          <li><strong>CLID</strong>: Identifies first-time Clarity saw this user (1 year retention)</li>
        </ul>
        
        <h3 class="text-xl font-semibold mt-6 mb-3">What we DON'T use</h3>
        <ul class="list-disc ml-6 mb-4">
          <li>No marketing or advertising cookies</li>
          <li>No social media tracking pixels</li>
          <li>No third-party advertising networks</li>
        </ul>
        
        <h2 class="text-2xl font-semibold mt-8 mb-4">Managing Your Preferences</h2>
        <p class="mb-4">You can manage your cookie preferences at any time by:</p>
        <ul class="list-disc ml-6 mb-4">
          <li>Clicking the "Cookie Settings" button in the footer</li>
          <li>Clearing your browser cookies and refreshing the page</li>
        </ul>
      </div>
    </section>
  </main>
</Layout>
```

## Footer Cookie Settings Button

Add this button to your footer component to allow users to manage preferences anytime:

### For React Component
```jsx
<button
  type="button"
  data-cc="show-preferencesModal"
  className="text-gray-400 hover:text-white text-sm transition-colors"
>
  Cookie Settings
</button>
```

### For Plain HTML
```html
<button type="button" data-cc="show-preferencesModal">
  Cookie Settings
</button>
```

The `data-cc="show-preferencesModal"` attribute is automatically recognized by vanilla-cookieconsent.

## Testing

### 1. Check Cookie Banner Appears
- Open your site in an incognito/private window
- You should see the cookie consent banner on first visit

### 2. Test Accept/Reject Flow
- Click "Reject All" - only essential cookies should be set
- Click "Accept All" - analytics cookies should be allowed
- Check browser DevTools > Application > Cookies

### 3. Test Preferences Modal
- Click "Manage Preferences" on the banner
- Or click "Cookie Settings" in the footer
- Toggle categories and save

### 4. Verify Clarity Integration
- Accept analytics cookies
- Open browser console
- Look for Clarity script loading
- Check Microsoft Clarity dashboard for data

### 5. Check Cookie Auto-Clear
- Accept all cookies
- Then open preferences and reject analytics
- Clarity cookies should be automatically removed

## Troubleshooting

### Cookie Banner Not Showing
- Check browser console for errors
- Ensure scripts are loading in correct order
- Clear all cookies and try again

### CookieConsent is undefined
- The library might not be loaded yet
- The initialization script waits for it with `setTimeout`
- Check network tab to ensure CDN resources load

### Clarity Not Tracking
- Verify analytics cookies are accepted
- Check Clarity project ID is correct
- Look for `clarity('consentv2', ...)` calls in console
- Allow 2-3 hours for data to appear in Clarity dashboard

### Cookie Settings Button Not Working
- Ensure button has `data-cc="show-preferencesModal"` attribute
- Check that CookieConsent library is loaded
- Test in browser console: `window.CookieConsent.showPreferences()`

### Static Build Issues
- Add `is:inline` directive to inline scripts in Astro
- Don't use `is:inline` for external script src references
- Use `defer` attribute for external scripts

## Additional Customization

### Change Cookie Banner Position
```javascript
guiOptions: {
  consentModal: {
    position: 'bottom left'  // or 'top center', etc.
  }
}
```

### Add More Cookie Categories
```javascript
categories: {
  marketing: {
    enabled: false,
    autoClear: {
      cookies: [
        {name: '_ga'},
        {name: '_gid'}
      ]
    }
  }
}
```

### Customize Translations
Add more languages or modify existing text:
```javascript
language: {
  default: 'en',
  autoDetect: 'browser',
  translations: {
    en: { /* ... */ },
    es: { /* Spanish translations */ }
  }
}
```

### Style Customization
Override CSS variables in your global styles:
```css
:root {
  --cc-bg: #ffffff;
  --cc-text: #000000;
  --cc-btn-primary-bg: #007bff;
  --cc-btn-primary-text: #ffffff;
  --cc-btn-primary-hover-bg: #0056b3;
}
```

## Legal Considerations

1. **Privacy Policy**: Update your privacy policy to mention cookie usage
2. **Cookie Policy**: Create a detailed cookie policy page
3. **Consent Records**: The implementation stores consent proof in cookies
4. **Regular Updates**: Keep your cookie list updated as you add services
5. **Testing**: Regularly test that consent management works properly

## Resources

- [vanilla-cookieconsent Documentation](https://cookieconsent.orestbida.com/)
- [Microsoft Clarity Consent API](https://learn.microsoft.com/en-us/clarity/setup-and-installation/clarity-consent-api-v2)
- [GDPR Cookie Requirements](https://gdpr.eu/cookies/)
- [Astro Documentation](https://docs.astro.build/)

---

*Last updated: January 2025*