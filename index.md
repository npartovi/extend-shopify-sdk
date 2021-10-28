---
layout: default
---

## Getting Started

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

## Installation

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

## Displaying Product Offers and Cart Offers

---

Place an html element anywhere you would like to see the product or cart page offer with a css selector that you can reference.

```html
<div id="extend-offer"></div>
```

This element will be used as the container for the displayed warranty offer. If you have specific width or spacing requirements, you can add them directly to this element using css or inline styles.

<br />

### Initialization

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

> Note: To query DOM elements, we use the native <b>document.querySelector</b>. For this reason, we recommend using element ids instead of classes for selector references.

### Accessing the component instance

It's possible to have multiple offers displayed on the same page and each might have their own product referenceId, so each rendered component acts independently of each other. The component instance will be tied to the element/selector passed in during the call to `Extend.buttons.render` and will be used for API calls later in this guide.

**To retrieve the component instance:**

```javascript
const component = Extend.buttons.instance("#extend-offer");
```

### Handling product selection changes

If your store has multiple product variants for a single product page (for example, if you have a product that allows a customer to select a color or size option) you'll need to pass the new **product reference id** to the SDK when the change occurs. This prevents a customer from accidentally purchasing the wrong warranty for an incorrect product.

**Example:**

```javascript
// calls are done through the component instance
const component = Extend.buttons.instance("#extend-offer");

component.setActiveProduct("<DIFFERENT_PRODUCT_REFERENCE_ID>");
```

<br />

## Adding a warranty to the cart

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

## Displaying a Modal Offer

---

A modal offer can be triggered from anywhere on a page and provides the user with another opportunity to protect their product with a warranty.

<p align="center"><img src="https://helloextend-static-assets.s3.amazonaws.com/ExtendModalOffer.png" /></p>

### Opening the modal offer

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

## Piecing it all together

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

<br />

## API Reference

---

#### Extend.config(config: object)

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

#### Extend.buttons.render(selector: string, options: object)

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

#### Extend.buttons.renderSimpleOffer(selector: string, options: object)

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

#### Extend.buttons.instance(selector: string): ButtonsComponent

```typescript
const component = Extend.buttons.instance("#offer-container");
```

#### ButtonsComponent#getActiveProduct(): ActiveProduct | null

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

#### ButtonsComponent#setActiveProduct(referenceId: string | number)

```typescript
component.setActiveProduct("item-12345");
```

#### ButtonsComponent#getPlanSelection(): PlanSelection | null

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

#### Extend.modal.open(options: object)

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

<h2 id='debug-mode'>Debug Mode</h2>

The Extend SDK has a debug feature that is useful for the testing the integration of the SDK in your store. This will help you determine if you are correctly integrating with the methods in the SDK.

Debug mode can be activated in the SDK by passing in `debug: true` to the [`Extend.config`](#config) method. When debug mode is activated, all methods will log an error to the browser's console if the methods are configured incorrectly, such as missing required arguments, incorrect quantity values, etc.

Please be sure to disable debug mode in production to prevent unwanted logs in the browser

<br>
<br>
<h1 id="intro">Integrating with Extend analytics</h1>

Extend analytics is used to capture the purchasing behavior of your customers, in order to provide you metrics and analytics about the performance of your Extend warranties and related products in your store.

This document outlines all of the different methods available to you on the `Extend` object. Each section includes a description of what the method does, when it should be invoked, and what parameters to provide it, as well as best practices for implementing into your website.

- API Reference
  - [Extend.trackOfferViewed](#track-offer-viewed)
  - [Extend.trackProductAddedToCart](#track-product-added-to-cart)
  - [Extend.trackOfferAddedToCart](#track-offer-added-to-cart)
  - [Extend.trackOfferRemovedFromCart](#track-offer-removed-from-cart)
  - [Extend.trackOfferUpdated](#track-offer-updated)
  - [Extend.trackProductRemovedFromCart](#track-product-removed-from-cart)
  - [Extend.trackProductUpdated](#track-product-updated)
  - [Extend.trackCartCheckout](#track-cart-checkout)
  - [Extend.trackLinkClicked](#track-link-clicked)
  - [Shared Interface and Type Glossary](#shared-interface-glossary)

<h1>API Reference</h1>

<h2 id="track-offer-viewed" class="section-function">Extend.trackOfferViewed</h2>

---

`Extend.trackOfferViewed` takes in your product's `productId` and an `OfferType`. This method should be called whenever an Extend warranty offer is rendered on the screen to a user.

<b>NOTE:</b> trackOfferViewed will be automatically called in the SDK whenever the offer buttons are rendered or whenever the Extend modal is opened. You will not need to worry about calling this functions yourself.

```javascript
Extend.trackOfferViewed({
  productId: "<PRODUCT_REFERENCE_ID>",
  offerType,
});
```

<h3 class="interface">Interface</h3>

```typescript
interface TrackOfferViewed {
  productId: string;
  offerType: OfferType;
}
```

<h3>Attributes</h3>

| Attribute                      | Data type | Description                                                     |
| :----------------------------- | :-------- | :-------------------------------------------------------------- |
| productId <br/> _**required**_ | string    | the ID of the product associated with the warranty being viewed |
| offerType <br/> _**required**_ | OfferType | see [OfferType](#glossary-offer-type)                           |

<h2 id="track-product-added-to-cart" class="section-function">Extend.trackProductAddedToCart</h2>

---

`Extend.trackProductAddedToCart` takes in your product's `productId` and the number of units the customer wishes to add to the cart, as `productQuantity`. This method should be called when a user adds a product to the cart.

```javascript
Extend.trackProductAddedToCart({
  productid: "<PRODUCT_REFERENCE_ID>",
  productQuantity: "<PRODUCT_QUANTITY>",
});
```

<h3 class="interface">Interface</h3>

```typescript
interface TrackProductAdded {
  productId: string;
  productQuantity: number;
}
```

<h3>Attributes</h3>

| Attribute                            | Data type | Description                                                                                                                                                                                                        |
| :----------------------------------- | :-------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| productId <br/> _**required**_       | string    | The ID of the product in your store. Should match the `referenceId` used in the [Create Product](https://developers.extend.com/2021-01-01#tag/Products/paths/~1stores~1{storeId}~1products/post) request to Extend |
| productQuantity <br/> _**required**_ | number    | The initial amount of units the customer added to the cart                                                                                                                                                         |

<h2 id="track-offer-added-to-cart" class="section-function">Extend.trackOfferAddedToCart</h2>

---

`Extend.trackOfferAddedToCart` takes in your product's `productId` and the `productQuantity`, along with the Extend warranty's `planId` and `offerType`. This method is used to track when a customer has added an Extend warranty to the cart. Invoke this method when a customer adds an Extend warranty to the cart for a specific product.

```javascript
Extend.trackOfferAddedToCart({
  productId: "<PRODUCT_REFERENCE_ID>",
  productQuantity: "<PRODUCT_QUANTITY>",
  planId: "<EXTEND_WARRANTY_PLAN_ID>",
  offerType: `OfferType`,
});
```

<h3 class="interface">Interface</h3>

```typescript
interface TrackOfferAddedToCart {
  productId: string;
  productQuantity: number;
  planId: string;
  offerType: OfferType;
}
```

<h3>Attributes</h3>

| Attribute                      | Data type | Description                                                                                                                                                                                                                                                                 |
| :----------------------------- | :-------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| productId <br/> _**required**_ | string    | The ID of the product associated with the warranty that was added to the cart. The product ID should match the `referenceId` used in the [Create Product](https://developers.extend.com/2021-01-01#tag/Products/paths/~1stores~1{storeId}~1products/post) request to Extend |
| productQuantity _**required**_ | number    | The quantity of the associated product that was added to the cart                                                                                                                                                                                                           |
| planId <br/> _**required**_    | string    | The `id` property on the Extend warranty product returned from the [Get Offers](https://developers.extend.com/2021-01-01#operation/getOffer) request                                                                                                                        |
| offerType <br/> _**required**_ | OfferType | Information regarding where in the website the user selected the warranty offer. See [OfferType](#glossary-offer-type)                                                                                                                                                      |

<h2 id="track-offer-removed-from-cart" class="section-function">Extend.trackOfferRemovedFromCart</h2>

---

`Extend.trackOfferRemovedFromCart` takes the `planId` of the Extend warranty, and the associated `productId` of the product the warranty would have covered. This method is used to track when a customer has removed an Extend warranty from the cart. Invoke this method when a customer removes an Extend warranty from the cart for a specific product.

```javascript
Extend.trackOfferRemovedFromCart({
  productId: "<PRODUCT_REFERENCE_ID>",
  planId: "<EXTEND_WARRANTY_PLAN_ID>",
});
```

<h3 class="interface">Interface</h3>

```typescript
interface TrackOfferRemovedFromCart {
  productId: string;
  planId: string;
}
```

<h3>Attributes</h3>

| Attribute                      | Data type | Description                                                                                                                                                                                                                                                                 |
| :----------------------------- | :-------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| productId <br/> _**required**_ | string    | The ID of the product associated with the warranty that was added to the cart. The product ID should match the `referenceId` used in the [Create Product](https://developers.extend.com/2021-01-01#tag/Products/paths/~1stores~1{storeId}~1products/post) request to Extend |
| planId <br/> _**required**_    | string    | The `id` property on the Extend warranty product returned from the [Get Offers](https://developers.extend.com/2021-01-01#operation/getOffer) request                                                                                                                        |

<h2 id="track-offer-updated" class="section-function">Extend.trackOfferUpdated</h2>

---

`Extend.trackOfferUpdated` takes the `planId` of the Extend warranty, and the associated `productId` of the product the warranty covers, as well as an update object containing the set of updates to apply to the warranty offer. This method is used to track when a customer increments or decrements the quantity of a warranty that has already been added to the cart. If the quantity of the warranty is updated to 0, then a [`Extend.trackOfferRemoved`](#track-offer-removed) event is called, and further updates to this planId/productId will result in a no-op until it is re-added via [`Extend.trackOfferAddedToCart`](#track-offer-added-to-cart).

```javascript
Extend.trackOfferUpdated({
  productId: "<PRODUCT_REFERENCE_ID>",
  planId: "<EXTEND_WARRANTY_PLAN_ID>",
  updates: {
    warrantyQuantity: "<NEW_WARRANTY_QUANTITY>",
    productQuantity: "<NEW_PRODUCT_QUANTITY>",
  },
});
```

<h3 class="interface">Interface</h3>

```typescript
interface TrackOfferUpdatedEvent {
  productId: string;
  planId: string;
  updates: {
    warrantyQuantity?: number;
    productQuantity?: number;
  };
}
```

<h3>Attributes</h3>

| Attribute                      | Data type | Description                                                                                                                                                                                                                                                                 |
| :----------------------------- | :-------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| productId <br/> _**required**_ | string    | The ID of the product associated with the warranty that was added to the cart. The product ID should match the `referenceId` used in the [Create Product](https://developers.extend.com/2021-01-01#tag/Products/paths/~1stores~1{storeId}~1products/post) request to Extend |
| planId <br/> _**required**_    | string    | The `id` property on the Extend warranty product returned from the [Get Offers](https://developers.extend.com/2021-01-01#operation/getOffer) request                                                                                                                        |
| updates <br/> _**required**_   | Object    | The set of updates to apply to the offer. Currently, either `warrantyQuantity` or `productQuantity` are values available to be updated.                                                                                                                                     |

<h2 id="track-product-removed-from-cart" class="section-function">Extend.trackProductRemovedFromCart</h2>

---

`Extend.trackProductRemovedFromCart` takes the `productId` of the product being removed from the cart. This method is used to track when a customer removes a product from the cart that does not have a warranty offer associated with it.

```javascript
Extend.trackProductRemovedFromCart({
  productId: "<PRODUCT_REFERENCE_ID>",
});
```

<h3 class="interface">Interface</h3>

```typescript
interface TrackProductRemovedEvent {
  productId: string;
}
```

<h3>Attributes</h3>

| Attribute                      | Data type | Description                                                                                                                                                                                                                                                                 |
| :----------------------------- | :-------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| productId <br/> _**required**_ | string    | The ID of the product associated with the warranty that was added to the cart. The product ID should match the `referenceId` used in the [Create Product](https://developers.extend.com/2021-01-01#tag/Products/paths/~1stores~1{storeId}~1products/post) request to Extend |

<h2 id="track-product-updated" class="section-function">Extend.trackProductUpdated</h2>

---

`Extend.trackProductUpdated` takes the `productId` of the product being updated, as well as an `updates` object containing the set of updates to apply to the product. This method is used to track when a customer increments or decrements the quantity of a product that has already been added to the cart. The product being updated must **not** be associated with a warranty offer. For updating products associated with a warranty offer, see [`Extend.trackOfferUpdated`](#track-offer-updated)

If the `productQuantity` passed into this method is 0, then [`Extend.trackProductRemovedFromCart`](#track-product-removed-from-cart) is implicitly called. The product will no longer be considered tracked. and must be re-added using [`Extend.trackProductAddedToCart`](#track-product-added-to-cart).

```javascript
Extend.trackProductUpdated({
  productId: "<PRODUCT_REFERENCE_ID>",
  updates: {
    productQuantity: "<NEW_PRODUCT_QUANTITY>",
  },
});
```

<h3 class="interface">Interface</h3>

```typescript
interface TrackProductUpdatedEvent {
  productId: string;
  updates: {
    productQuantity: number;
  };
}
```

<h3>Attributes</h3>

| Attribute                      | Data type | Description                                                                                                                                                                                                                                                                 |
| :----------------------------- | :-------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| productId <br/> _**required**_ | string    | The ID of the product associated with the warranty that was added to the cart. The product ID should match the `referenceId` used in the [Create Product](https://developers.extend.com/2021-01-01#tag/Products/paths/~1stores~1{storeId}~1products/post) request to Extend |
| updates <br/> _**required**_   | Object    | The set of updates to apply to the product. Currently, updating the `productQuantity` is the only available option.                                                                                                                                                         |

<h2 id="track-cart-checkout" class="section-function">Extend.trackCartCheckout</h2>

---

`Extend.trackCartCheckout` captures the resulting set of products and warranties that were tracked in a user's session (via `trackOfferAddedToCart`, `trackProductAddedToCart`, etc.). This method should be called whenever a customer completes a purchase. This will help track the products and Extend warranty offers sold for your store, and optionally, the total cart revenue of the items at checkout.

**Note**: The Analytics SDK leverages the tracked products and items in localStorage to determine what values were purchased at checkout. These values are removed from localStorages once this method is called after the user checks out.

```javascript
Extend.trackCartCheckout({
  cartTotal: "<CART_TOTAL>",
});
```

<h3>Attributes</h3>

| Attribute                  | Data type | Description                                                                                       |
| :------------------------- | :-------- | :------------------------------------------------------------------------------------------------ |
| cartTotal <br/> _optional_ | number    | The total amount of revenue (implicitly represented as USD), of the cart at the time of checkout. |

<h2 id="track-link-clicked" class="section-function">Extend.trackLinkClicked</h2>

---

`Extend.trackLinkClicked` captures customer interactions with "Learn More" and "?" buttons and "See Plan Details" links. This method is used to record details when a customer opens the learn-more modal or navigates to Extend's plan details page.

<b>NOTE:</b> trackLinkClicked will automatically be called whenever the Extend offer buttons "learn more" link is clicked or whenever the "plan-details" link is clicked on the learn more modal.

```javascript
Extend.trackLinkClicked({
  linkEvent: "LinkEvent",
  productId: "<PRODUCT_REFERENCE_ID>",
  linkType: "LinkType",
});
```

<h3 class="interface">Interface</h3>

```typescript
interface TrackLinkClicked {
  linkEvent: LinkEvent;
  productId: string;
  linkType: LinkType;
}
```

<h3>Attributes</h3>

| Attribute                      | Data type | Description                                                                                                                                                                                                                                                                 |
| :----------------------------- | :-------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| linkEvent <br/> _**required**_ | LinkEvent | Information about what link was clicked. See [LinkEvent](#glossary-link-event)                                                                                                                                                                                              |
| productId _**required**_       | string    | The ID of the product associated with the warranty that was added to the cart. The product ID should match the `referenceId` used in the [Create Product](https://developers.extend.com/2021-01-01#tag/Products/paths/~1stores~1{storeId}~1products/post) request to Extend |
| linkType <br/> _**required**_  | LinkType  | Information about where on the website a link was clicked. See [LinkType](#glossary-link-type)                                                                                                                                                                              |

<h2 id="shared-interface-glossary" class="section-function">Shared Interface and Type Glossary</h2>

<h2>OfferType</h2>

<h3 class="interface">Interface</h3>

```typescript
interface OfferType {
  area: OfferTypeArea;
  component: OfferTypeComponent;
}
```

<h3>Attributes</h3>

| Attribute                      | Data type          | Description                                                |
| :----------------------------- | :----------------- | :--------------------------------------------------------- |
| area <br/> _**required**_      | OfferTypeArea      | Predefined string indicating where the offer was rendered  |
| component <br/> _**required**_ | OfferTypeComponent | Predefined string indicating which offer type was rendered |

<h2>OfferTypeArea</h2>

<h3 class="interface">Type</h3>

```typescript
type OfferTypeArea =
  | "product_page"
  | "product_modal"
  | "collection"
  | "cart_page"
  | "post_purchase_modal";
```

<h2>OfferTypeComponent</h2>

<h3 class="interface">Type</h3>

```typescript
type OfferTypeComponent = "buttons" | "modal";
```

<h2>LinkEvent</h2>

<h3 class="interface">Type</h3>

```typescript
type LinkEvent = "learn_more_clicked" | "plan_details_clicked";
```

<h2>LinkType</h2>

<h3 class="interface">Interface</h3>

```typescript
interface LinkType {
  area: LinkTypeArea;
  component: LinkTypeComponent;
}
```

<h3>Attributes</h3>

| Attribute                      | Data type         | Description                                              |
| :----------------------------- | :---------------- | :------------------------------------------------------- |
| area <br/> _**required**_      | LinkTypeArea      | Predefined string indicating where the link was clicked  |
| component <br/> _**required**_ | LinkTypeComponent | Predefined string indicating which link type was clicked |

<h2>LinkTypeArea</h2>

<h3 class="interface">Type</h3>

```typescript
type LinkTypeArea =
  | "product_page"
  | "cart_page"
  | "learn_more_modal"
  | "offer_modal"
  | "simple_offer_modal"
  | "post_purchase_modal";
```

<h2>LinkTypeComponent</h2>

<h3 class="interface">Type</h3>

```typescript
type LinkTypeComponent =
  | "learn_more_info_icon"
  | "learn_more_info_link"
  | "see_plan_details_link";
```
