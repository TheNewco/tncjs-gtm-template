# TNCiD - The Newco GTM Template

The official Google Tag Manager template for installing the **TNCiD** tracking script on your website.

---

## Requirements

- A Google Tag Manager **web container**
- A **Provider ID** provided by The Newco (UUID format)
- A Consent Management Platform (CMP) — recommended for GDPR compliance

---

## Installation

1. Download `template.tpl` from this repository
2. In GTM, go to **Templates** > **Tag Templates** > **New**
3. Click the menu icon (⋮) > **Import**
4. Select the downloaded `template.tpl` file
5. Click **Save**

The template **TNCiD - The Newco** is now available in your tag library.

---

## Basic Setup

1. Create a new tag: **Tags** > **New** > select **TNCiD - The Newco**
2. Enter your **Provider ID** (provided by The Newco)
3. Set **Event Name** to `Standard` > `pageview`
4. Add a firing trigger based on user consent:
   - If your CMP supports **TCF2**: use vendor consent for **The Newco S.r.l. (IAB code 750)**
   - Otherwise: fire the tag only when consent for profiling cookies is granted
5. Save and publish

---

## Documentation

Full implementation guide (e-commerce events, customer data, custom variables, GA4 integration):

[docs/TheNewco-TagManager-Template.md](docs/TheNewco-TagManager-Template.md)

---

## Support

For Provider ID requests or technical support, contact [The Newco](https://www.thenewco.tech/).
