---
layout: default
---

# Getting Started

---

To display offers on your webpage you will need:

<dl>
    <dt>Your Extend Store ID</dt>
    <dd>
        This can be found through the <a href="https://merchants.helloextend.com">Extend Merchant Portal</a> by clicking on the <i>"Settings"</i> navigation tab (<a href="https://dev.merchants.helloextend.com/dashboard/settings">direct link</a>) under <i>"Production Credentials"</i>
    <img src="https://helloextend-static-assets.s3.amazonaws.com/Credentials.png"/>
    </dd>
    <dt>A product referenceId</dt>
    <dd>This is the unique product identifier that you had specified in your product catalog upload.  We will use this identifier to know which warranty offer to display.  Generally this is the product SKU or similar, however the choice of <b>product reference ID</b> is up to a merchant to decide.  If you are having trouble finding out what your <b>product referenceId</b> is, please file a support request through the <a href="https://merchants.helloextend.com">Extend Merchant Portal</a>.
    </dd>
</dl>

<br />

# Installation

---

Add the following to your page

```html
<script src="https://sdk.helloextend.com/extend-sdk-client/v1/extend-sdk-client.min.js"></script>
<script>
  Extend.config({ storeId: "<YOUR_EXTEND_STORE_ID>" });
</script>
```

For more options that can be passed to the `Extend.config()` function, see the [API Reference](#api-reference)

<br />

# Displaying Product Offers and Cart Offers

---

Place an html element anywhere you would like to see the product or cart page offer with a css selector that you can reference.

```html
<div id="extend-offer"></div>
```

This element will be used as the container for the displayed warranty offer. If you have specific width or spacing requirements, you can add them directly to this element using css or inline styles.

<br />

# Initialization

---

```javascript
/* PRODUCT PAGE OFFER */
Extend.buttons.render('#extend-offer', {
  referenceId: '<PRODUCT_REFERENCE_ID>',
})

/* CART PAGE OFFER */
Extend.buttons.renderSimpleOffer('#extend-offer', {
  referenceId: '<PRODUCT_REFERENCE_ID>',
  onAddToCart({ plan, product, quantity }) {
    if (plan && product) {
      // a user has selected a plan.  Add warranty to their cart.
    }
})
```

<div class="info-container">
  <strong>Note:</strong> To query DOM elements, we use the native <b>document.querySelector</b>. For this reason, we recommend using element ids instead of classes for selector references.
</div>

## Accessing the component instance

It's possible to have multiple offers displayed on the same page and each might have their own product referenceId, so each rendered component acts independently of each other. The component instance will be tied to the element/selector passed in during the call to `Extend.buttons.render` and will be used for API calls later in this guide.

**To retrieve the component instance:**

```javascript
const component = Extend.buttons.instance("#extend-offer");
```

## Handling product selection changes

If your store has multiple product variants for a single product page (for example, if you have a product that allows a customer to select a color or size option) you'll need to pass the new **product reference id** to the SDK when the change occurs. This prevents a customer from accidentally purchasing the wrong warranty for an incorrect product.

**Example:**

```javascript
// calls are done through the component instance
const component = Extend.buttons.instance("#extend-offer");

component.setActiveProduct("<DIFFERENT_PRODUCT_REFERENCE_ID>");
```

<br />

# Adding a warranty to the cart

---

Typically you would add the a warranty selection to the cart when they
click on "Add to Cart". You can retrieve the users warranty
selection by calling `component.getPlanSelection();` and retrieve the
currently selected product by calling `component.getActiveProduct();`.

**Example**

```javascript
const component = Extend.buttons.instance("#extend-offer");
const plan = component.getPlanSelection();
const product = component.getActiveProduct();
```

If a user has selected a warranty option, `getPlanSelection` will return
a `Plan` object, otherwise it will return `null`. See the
[API Reference](#api-reference) for details on what is included with the
`plan` object.

Here is an example of how this might look in a basic store.

```javascript
$("#add-to-cart").on("click", function (e) {
  e.preventDefault();

  /** get the component instance rendered previously */
  const component = Extend.buttons.instance("#extend-offer");

  /** get the users plan selection */
  const plan = component.getPlanSelection();
  const product = component.getActiveProduct();

  if (plan) {
    /**
     * If you are using an ecommerce addon (e.g. Shopify) this is where you
     * would use the respective add-to-cart helper function.
     *
     * For custom integrations, use the plan data to determine which warranty
     * sku to add to the cart and at what price.
     */
    // add plan to cart, then handle form submission
  } else {
    // handle form submission
  }
});
```

<br />

# Displaying a Modal Offer

---

A modal offer can be triggered from anywhere on a page and provides the user with another opportunity to protect their product with a warranty.

<p align="center"><img src="https://helloextend-static-assets.s3.amazonaws.com/ExtendModalOffer.png" /></p>

## Opening the modal offer

```javascript
Extend.modal.open({
  referenceId: "<PRODUCT_REFERENCE_ID>",
  /**
   * This callback will be triggered both when a user declines the offer and if
   * they choose a plan.  If a plan is chosen, it will be passed into the
   * callback function, otherwise it will be undefined.
   */
  onClose: function (plan, product) {
    if (plan && product) {
      // a user has selected a plan.  Add it to their cart.
    }
  },
});
```

<br />

# Piecing it all together

---

Combining all of these tools you should now be able to create a complete
end-to-end warranty offer solution on your online store.

**Full Product Page Example**

```html
<html>
  <head></head>
</html>
<form id="product-form" action="/cart/add">
  <div id="extend-offer"></div>
  <button id="add-to-cart" type="submit">Add To Cart</button>
</form>
<script></script>
```

```javascript
/** configure */
Extend.config({ storeId: "<EXTEND_STORE_ID>" });

/** initialize offer */
Extend.buttons.render("#extend-offer", {
  referenceId: "<PRODUCT_REFERENCE_ID>",
});

/** bind to add-to-cart click */
$("#add-to-cart").on("click", function (event) {
  event.preventDefault();

  /** get the component instance rendered previously */
  const component = Extend.buttons.instance("#extend-offer");

  /** get the users plan selection */
  const plan = component.getPlanSelection();

  if (plan) {
    /**
     * Add the warranty to the cart using an AJAX call and then submit the form.
     * Replace this section with your own store specific cart functionality.
     */
    YourAjaxLib.addPlanToCart(plan, function () {
      $("#product-form").submit();
    });
  } else {
    Extend.modal.open({
      referenceId: "<PRODUCT_REFERENCE_ID>",
      onClose: function (plan, product) {
        if (plan && product) {
          YourAjaxLib.addPlanToCart(plan, product, quantity, function () {
            $("#product-form").submit();
          });
        } else {
          $("#product-form").submit();
        }
      },
    });
  }
});
```

<br />

**Full Cart Page Example**

```html
<html>
  <head></head>
</html>
<div id="cart-item">
  <h2>Product Title</h2>
  <div id="extend-offer"></div>
</div>
<script></script>
```

```javascript
/** configure */
Extend.config({ storeId: '<EXTEND_STORE_ID>' });

/** get quantity from store page if possible */
const quantity = document.querySelector('.storeQuantity');

/** initialize offer */
Extend.buttons.renderSimpleOffer('#extend-offer', {
  referenceId: '<PRODUCT_REFERENCE_ID>'
  onAddToCart:
    function({ plan, product }) {
        if (plan && product) {
          // if you can get quantity from store, pass it here.
          // otherwise, quantity defaults to 1
          YourAjaxLib.addPlanToCart(plan, product, quantity)
        }
    },
});
```

# API Reference

---

### Extend.config(config: object)

```typescript
Extend.config({
  /**
   * Extend store ID
   *
   * @required
   */
  storeId: string,
  /**
   * A list of product reference IDs for the current page.  This will preload
   * the offers for each product to improve performance when switching
   * referenceIds.
   *
   * @optional
   */
  referenceIds: Array<string>,
  /**
   * Optional theme configuration.  A theme passed in as an argument will
   * override any theme retrieved from the Extend Merchant Portal configuration
   * and will fallback to a default theme if neither are present.
   *
   * @optional
   */
  theme: {
    primaryColor: string
  },
  /**
   * If you are using our demo environment, set this to "demo".
   *
   * @optional
   * @deprecated Soon demo environment will be removed.  Do not use if you can help it.
   */
   environment: "demo" | "production"
})
```

### Extend.buttons.render(selector: string, options: object)

```typescript
Extend.buttons.render("#offer-container", {
  /** @required */
  referenceId: string,
  /** @optional */
  theme: {
    primaryColor: string,
  },
});
```

### Extend.buttons.renderSimpleOffer(selector: string, options: object)

```typescript
Extend.buttons.renderSimpleOffer('#offer-container', {
  /** @required */
  referenceId: string,
  onAddToCart?({ plan: PlanSelection, product: ActiveProduct, quantity?: number }): void | Promise<void>,
  /** @optional */
  theme: {
    primaryColor: string
  }
})
```

### Extend.buttons.instance(selector: string): ButtonsComponent

```typescript
const component = Extend.buttons.instance("#offer-container");
```

### ButtonsComponent#getActiveProduct(): ActiveProduct | null

```typescript
// usage
const product = component.getActiveProduct();

// product object structure
interface ActiveProduct {
  /**
   * The referenceId of the active product
   * @example "sku-12345"
   */
  id: string;
  /**
   * The name of the active product
   * @example "XBox One X"
   */
  name: string;
}
```

### ButtonsComponent#setActiveProduct(referenceId: string | number)

```typescript
component.setActiveProduct("item-12345");
```

### ButtonsComponent#getPlanSelection(): PlanSelection | null

```typescript
// usage
const plan = component.getPlanSelection();

// plan object structure
interface PlanSelection {
  /**
   * The unique plan identifier for this plan
   * @example "10001-misc-elec-adh-replace-1y"
   */
  planId: string;
  /**
   * The price of the warranty plan in cents (e.g. 10000 for $100.00)
   * @example 10000
   */
  price: number;
  /**
   * The coverage term length in months
   * @example 36
   */
  term: number;
}
```

### Extend.modal.open(options: object)

```typescript
Extend.modal.open({
  /** @required */
  referenceId: string,
  /** @optional */
  theme: {
    primaryColor: string,
  },
  /**
   * If a user made a warranty selection, the "plan" and "product" will be
   * passed to this callback function.  If the user declines the offer, "plan"
   * and "product" will be undefined.  See above for data structure for
   * "PlanSelection" and "ActiveProduct" arguments.
   *
   * This callback function is where you should handle adding the warranty to
   * the users cart, submitting your product form, etc.
   */
  onClose(plan?: PlanSelection, product?: ActiveProduct) {},
});
```
