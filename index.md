---
layout: default
---

<h1 id="intro">Welcome to the <code style="font-size: 1.3rem;">ExtendAnalytics</code> API reference!</h1>

This reference outlines all of the different functions available to you on the `ExtendAnalytics` object. Each section includes a description of what the function does, when it should be invoked, and what parameters to provide it. This reference should be used in conjunction with the <a href="https://www.google.com">Extend Analytics SDK Integration Guide.</a>

<h4 id="table-of-contents">Table of Contents:</h4>

- <ins>[ExtendAnalytics.config](#config)</ins>
- <ins>[ExtendAnalytics.setSelectedPlan](#setSelectedPlan)</ins>
- <ins>[ExtendAnalytics.trackOfferViewed](#trackOfferViewed)</ins>
- <ins>[ExtendAnalytics.trackOfferSold](#trackOfferSold)</ins>
- <ins>[ExtendAnalytics.trackOfferAddedToCart](#trackOfferAddedToCart)</ins>
- <ins>[Shared Interface and Type Glossary](#shared-interface-glossary)</ins>

<h2 id="config" class="section-function">ExtendAnalytics.config</h2>

---

`ExtendAnalytics.config` takes in a storeId and initializes the Extend Analytics SDK. This will ensure all `ExtendAnalytics` functions will be tracked with your specific storeId. This function should be called upon page load or on startup of your application.

```javascript
ExtendAnalytics.config({ storeId: "<YOUR_EXTEND_STORE_ID>" });
```

<h3 class="interface">Interface</h3>

```typescript
interface Config {
  storeId: string;
}
```

<h3>Attributes</h3>

| Attribute                    | Data type | Description                    |
| :--------------------------- | :-------- | :----------------------------- |
| storeId <br/> _**required**_ | string    | Your assigned Extend `storeId` |

<h2 id="setSelectedPlan" class="section-function">ExtendAnalytics.setSelectedPlan</h2>

---

`ExtendAnalytics.setSelectedPlan` takes in your products `productId`, the Extend warranty `planId`, and a `OfferType`. This function should be invoked each time a customer clicks on a Extend button offer. This will track what Extend warranty offer was selected by the user. This also allows our to `ExtendAnalytics.trackOfferSold` function to grab the users selected Extend offer automatically.

```javascript
ExtendAnalytics.setSelectedPlan({
  offerType,
  productId: "<PRODUCT_REFERENCE_ID>",
  planId: "<EXTEND_WARRANTY_ID>",
});
```

<h3 class="interface">Interface</h3>

```typescript
interface SetSelectedPlan {
  productId: string;
  offerType: OfferType;
  planId: string;
}
```

<h3>Attributes</h3>

| Attribute                      | Data type | Description                                                                                            |
| :----------------------------- | :-------- | :----------------------------------------------------------------------------------------------------- |
| productId <br/> _**required**_ | string    | the ID of the product for warranty being rendered                                                      |
| offerType <br/> _**required**_ | object    | object containing the offer area and component data (see <a href="#glossary-offer-type">OfferType<a/>) |
| planId <br/> _**required**_    | string    | the ID of the warranty plan being offered                                                              |

<h2 id="trackOfferViewed" class="section-function">ExtendAnalytics.trackOfferViewed</h2>

---

`ExtendAnalytics.trackOfferViewed` takes in your products `productId` and a `OfferType`. This function should be called whenever an offer is rendered on the screen to a user and passed the appropriated arguments. This function is used to track when a user views a Extend warranty offer.

```javascript
ExtendAnalytics.trackOfferViewed({
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
| offerType <br/> _**required**_ | OfferType | see OfferType                                                   |

<h2 id="trackOfferSold" class="section-function">ExtendAnalytics.trackOfferSold</h2>

---

`ExtendAnalytics.trackOfferSold` takes in your products `productId` and the quantity of the purchased product `productQuantity`. This function should be called whenever a product is checked out from your cart page and has an associated Extend warranty in the cart. This will help track the product along with the Extend offer sold for your store.

```javascript
ExtendAnalytics.trackOfferSold({
  productId: "<PRODUCT_REFERENCE_ID>",
  productQuantity: "<PRODUCT_QUANTITY>",
});
```

<h3 class="interface">Interface</h3>

```typescript
interface TrackOfferSold {
  productId: string;
  productQuantity: number;
}
```

<h3>Attributes</h3>

| Attribute                      | Data type | Description                                                           |
| :----------------------------- | :-------- | :-------------------------------------------------------------------- |
| productId <br/> _**required**_ | string    | the ID of the product associated with the warranty that was purchased |
| productQuantity _**reuired**_  | number    | the quantity of the associated product that was purchased             |

<h2 id="trackOfferAddedToCart" class="section-function">ExtendAnalytics.trackOfferAddedToCart</h2>

---

`ExtendAnalytics.trackOfferAddedToCart` takes in your products `productId` and the `productQuantity` of the purchased product. This function is used to track when a customer has added a Extend warranty to the cart. Invoke this function when a customer adds a product to the cart with an associated Extend warranty

```javascript
ExtendAnalytics.trackOfferAddedToCart({
  productId: "<PRODUCT_REFERENCE_ID>",
  productQuantity: "<PRODUCT_QUANTITY>",
});
```

<h3 class="interface">Interface</h3>

```typescript
interface TrackOfferAddToCart {
  productId: string;
  productQuantity: number;
}
```

<h3>Attributes</h3>

| Attribute                      | Data type | Description                                                                   |
| :----------------------------- | :-------- | :---------------------------------------------------------------------------- |
| productId <br/> _**required**_ | string    | the ID of the product associated with the warranty that was added to the cart |
| productQuantity _**required**_ | number    | the quantity of the associated product that was added to the cart             |

<h2 id="shared-interface-glossary" class="section-function">Shared Interface and Type Glossary</h2>

<h3><code id="glossary-offer-type" class="glossary-interface" style="font-size:1.3rem;">OfferType</code></h3>

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
| area <br/> _**required**_      | OfferTypeArea      | predfined string indicating where the offer was rendered   |
| component <br/> _**required**_ | OfferTypeComponent | predefined string indicating which offer type was rendered |

<br/>
<h3><code id="glossary-offer-type-area" class="glossary-interface" style="font-size:1.3rem;">OfferTypeArea</code></h3>

<h3 class="interface">Type</h3>

```typescript
type OfferTypeArea =
  | "product_page"
  | "product_modal"
  | "collection"
  | "cart_page"
  | "cart_page_load"
  | "app"
  | "post_purchase_modal";
```

<h3><code id="glossary-offer-type-component" class="glossary-interface" style="font-size:1.3rem;">OfferTypeComponent</code></h3>

<h3 class="interface">Type</h3>

```typescript
type OfferTypeComponent = "buttons" | "modal" | "app";
```
