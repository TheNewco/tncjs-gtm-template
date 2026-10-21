# THE NEWCO GTM TEMPLATE

The **TNCiD Custom Template** is the official GTM template for installing the The Newco script and its components.

---

## PERMISSION

The template requires the following 3 permissions to operate correctly:

- **Accesses global variables** — reads and writes to the `dataLayer` and the `__tnc` global object
- **Injects scripts** — loads the The Newco tracking script from `*.tncid.app`
- **Logs to console** — outputs debug information in GTM preview mode

![Tag permissions](images/002_permissions.png)

---

## 1. CONFIGURATION SETTINGS (BASIC)

To activate the script, it is necessary to insert the **PROVIDER ID** provided by The Newco. Along with the provider ID, you will need to set up the template to send a standard `pageview` event. For basic implementations, no additional parameters or configurations are required.

![Tag configuration](images/001_tag_configuration.png)

It is possible to activate the TNCiD tag on the native GTM activator **"All Pages."**

![Basic implementation with All Pages trigger](images/003_main_snippet_basic.png)

The tag should be activated only if the user gives consent to profiling cookies (or similar technologies). Typically, various Consent Management Platforms (CMPs) provide indications in the data layer regarding the type of consent provided.

If your CMP does not support [TCF2](https://iabeurope.eu/transparency-consent-framework), it is advisable to activate the tag only if consent is given for profiling cookies.

If, on the other hand, your CMP supports TCF2, then you can associate the activation of the tag with the expressed consent for the vendor **The Newco S.r.l. (IAB code 750)**.

---

## 2. CONFIGURATION SETTINGS ECOMMERCE (Advanced)

If your website is an e-commerce site or a more complex site, you will definitely need to send multiple events beyond the simple initial pageview, such as the checkout or the purchase of a specific product.

Here is a list of dedicated standard events:

| Event Name     | Description                          |
|----------------|--------------------------------------|
| `pageview`     | Generic pageview                     |
| `add_to_cart`  | Product added to cart                |
| `view_cart`    | Pageview cart                        |
| `checkout`     | Pageview checkout                    |
| `view_product` | Pageview product page                |
| `purchase`     | Pageview post purchase               |
| `lead`         | Thank you page after form submission |

Here are some recommendations for implementation. Firstly, you should insert the `pageview` event on all pages of the website, just as with the basic implementation. After inserting the pageview event, you can add other events that capture specific moments or actions in the user's conversion funnel. For example, you could insert the `view_product` event when the user views a product page on your e-commerce site.

---

### 2.1. EXCEPTION PAGES

There are certain pages, such as Product Detail Pages (PDP), where two requests to The Newco servers will inevitably be triggered — one set in the `pageview` event and another in the `view_product` event. When both the `pageview` event and a specific standard event are triggered on the same page, we refer to them as **"exception pages."**

A good practice is to avoid triggering the generic pageview tag on these exception pages and to keep only the tag with the dedicated event active.

Each TNCiD tag firing sends its own self-contained hit, so firing both tags on the same page is technically safe: no event is lost. However, the dedicated events (`view_product`, `view_cart`, `checkout`, `purchase`, `lead`) already represent a view of that page, so keeping the generic `pageview` as well would:

- count the same page view twice;
- send two requests to The Newco servers, repeating data collection and ID synchronization;
- push the `tncid-retrieved` event (if enabled) twice, firing any tag attached to it twice as well (e.g. GA4).

Here are some exception pages:

- **PDP** (pageview event + view_product event)
- **Cart** (pageview event + view_cart event)
- **Checkout** (pageview event + checkout event)
- **Purchase** (pageview event + purchase event)
- **Lead**, if tracked with TY trigger (pageview event + lead event)

To prevent the double firing of a request to The Newco servers, it is advisable to identify these exception pages using variables, which can be JavaScript-based or related to the URL of the pages if identifiable. Usually, there is a variable in the data layer or the URL that allows the identification of the page category.

For example, on the Product Detail Page (PDP), you might have a `pageType` variable in the data layer.

![DataLayer pageType variable example](images/004_dataLayer_push_example.png)

At this point, you can create an exception trigger for the tag with the `pageview` event to prevent it from being activated on exception pages.

Here's how you can create an exception trigger based on the `pageType` variable being equal to `"product"`:

1. Open your Google Tag Manager (GTM) and go to the **"Triggers"** section.
2. Click on **"New"** to create a new trigger.
3. Select **"Exception Trigger."**
4. Assign a meaningful name to your trigger, for example, `"Exception - Product Pages."`
5. In the **"Conditions"** section, select `"Page Type"` (make sure this variable is correctly populated in your data layer).
6. Set the condition so that `pageType` is equal to `"product."`
7. Save the trigger.

![Exception trigger for product pages](images/005_pageview_exception.png)

> **Note:** the exception trigger is evaluated on the same event that fires the pageview tag (with the **"All Pages"** trigger, the `gtm.js` event). The `pageType` variable must therefore be pushed to the data layer **before** the GTM snippet: if it is pushed later, the condition is not yet true when the pageview tag fires and the exception has no effect. In that case, use a condition based on the page URL, or fire the pageview tag on a later event (e.g. **DOM Ready**).

By associating this exception trigger with your tag containing the `pageview` event, you will prevent the tag from firing on pages where the dedicated `view_product` event is expected. This way, you will avoid the double firing of requests to The Newco servers on these specific pages.

For convenience, you could already list all the exception pages and include them in your exception trigger, identifying them with a variable — in this case, `"dlv PageType."`

![Exception trigger with RegEx for all exception pages](images/006_pageview_exception_with_regexp.png)

At this point, you can insert the exception trigger created for the TNCiD pageview tag and freely add tags for other dedicated events.

![Pageview tag with exception trigger applied](images/007_add_exception_as_trigger.png)

> **Naming convention:** name the exception trigger `Exception TNCiD pageview tag` so it is easy to identify in the GTM interface.

---

### 2.2. CUSTOMER DATA

Customer data typically refers to attributes associated with the user. They can be sent along with any event and are appropriately handled directly by The Newco.

| Field | Description |
| --- | --- |
| **Email** | User's email. Can be sent in full or already hashed in SHA256 format. |
| **CRM User ID** | User ID identifiable in a CRM system. |
| **External User ID** | Any identifier of the user. |
| **Phone Without Country Prefix** | User's phone number without the country prefix, e.g., `3333333333` |
| **Phone With Country Prefix** | User's phone number with the country prefix, e.g., `+393333333333` |

![Customer Data Properties configuration](images/008_customer_data.png)

---

### 2.3. CUSTOM VARIABLE

Custom variables are key-value-type triplets that can be sent along with events. They can be simple strings or have other types of formats.

Here is an example of a custom variable. Let's say we want to send an `add_to_cart` event when the user adds a specific product to the cart, and we want to identify details about the product sent. In this configuration, you can insert the details of the product as `items` and associate their value with the variable `{{dlv items}}`.

![Custom variable example for add_to_cart](images/009_custom_variables.png)

You could send product information through a custom variable of type **'JSON or Array'**. If you have GA4 datalayer configuration enabled, you can use the same `items` variable.

Here's what the `items` variable must include. Typically, the `items` variable is an array of objects. Each object must contain:

- `item_id` (string)
- `item_name` (string)
- `price` (float)
- `quantity` (integer, float)
- `item_category` (string)
- `item_category2` (string) — *OPTIONAL*
- `item_category3` (string) — *OPTIONAL*
- `currency` (string)
- `item_brand` (string)

Example of the `items` variable:

```javascript
items: [
  {
    item_id: "SKU_12345",
    item_name: "Stan and Friends Tee",
    item_brand: "Google",
    item_category: "Apparel",
    item_category2: "Adult",
    item_category3: "Shirts",
    price: 10.01,
    quantity: 3,
    currency: "EUR"
  },
  {
    item_id: "SKU_12346",
    item_name: "Google Grey Women's Tee",
    item_brand: "Google",
    item_category: "Apparel",
    item_category2: "Adult",
    item_category3: "Shirts",
    price: 21.01,
    quantity: 2,
    currency: "EUR"
  }
]
```

When you send the `purchase` event, you also need to add some custom variables:

| Variable           | Type   |
|--------------------|--------|
| `transaction_id`   | string |
| `value`            | float  |
| `currency`         | string |
| `payment_method`   | string |

![Purchase event with all custom variables](images/010_purchase_event_example.png)

In some cases, you can send custom events that are not in the list. For example, you can send a `view_category` event when a user lands on a category page. In this case, it is not necessary to send all the `items` information — you can send only the `item_category` as a string custom variable.

![Custom event view_category example](images/011_view_category_example.png)

---

### 2.4. PUSH DATALAYER EVENT

It is possible to trigger a custom event in the data layer with the **TNCiD**. Enable the **"Enable Push dataLayer Event"** option in the template and set the event name to `tncid-retrieved` (default value).

![Enable Push dataLayer Event option](images/012_push_data_layer_event.png)

When the TNCiD tag fires, it will push the following event to the DataLayer:

```javascript
dataLayer.push({
  event: "tncid-retrieved",
  tncid: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  gtm.uniqueEventId: 232
})
```

![DataLayer push result in GTM preview](images/013_push_data_layer_event_result.png)

Thanks to this push, you can create a **DataLayer Variable** in GTM to capture the `tncid` value:

- **Variable Type:** Data Layer Variable
- **Data Layer Variable Name:** `tncid`
- **Data Layer Version:** Version 2

![GTM DataLayer Variable configuration](images/014_define_gtm_variable.png)

Here is an example of integration between The Newco and GA4, allowing the use of TNCiD as a `user_property` for cross-device user recognition and the construction of advanced attribution models on BigQuery.

**Step 1 — Create a Custom Event trigger:**

- **Trigger name:** `TNCiDRetrieved`
- **Trigger Type:** Custom Event
- **Event name:** `tncid-retrieved`

**Step 2 — Create a GA4 Event tag:**

- **Tag name:** `The Newco - tncid_get`
- **Measurement ID:** `G-XXXXXXXX`
- **Event Name:** `tncid_get`
- **User Properties:**

| Property Name | Value            |
|---------------|------------------|
| `tncid`       | `{{dlv tncid}}`  |

- **Firing Trigger:** `TNCiDRetrieved`

![GA4 integration with TNCiD as user property](images/015_GA4_integration.png)

Every time the TNCiD tag fires, the `tncid-retrieved` DataLayer event is triggered, which in turn fires the GA4 tag and sends `tncid` as a user property for cross-device user recognition.

---

### 2.5. WINDOW CUSTOM EVENT TRIGGER

This template features a function to trigger a custom event in the window, providing a useful way for event dispatching.

For instance, if you activate the `tncid-retrieved` event in the template, you can then call a listener in the code that waits for the The Newco event and performs specific actions. The TNCiD is available in `event.detail.tncid`.

Here's an example of how you might include the `tncid-retrieved` custom event trigger in the template and then execute code on the website to, for instance, display an alert or run custom code:

```javascript
window.addEventListener('tncid-retrieved', event => {
  console.log('TNCiD', event.detail.tncid);
  window.alert('Welcome back!');
});
```
