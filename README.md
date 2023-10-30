# Vue Checkout

Solution for Vue 3 checkout component

## Requirements

### Features
- Delivery options
- Dynamic delivery price
- Payment options
- Discount (coupons)
- Flexible splitting to steps (load step from [URL parameter](https://vueuse.org/core/useUrlSearchParams/?foo=bar&vueuse=awesome))
- Persistent cart state

### UI Elements
- Product list (quantity, remove, link to product)
- Product summary (price)
- Coupon code
- Form (billing address, shipping address, validation, consent)
- Loader

### Focus points
Persistent cart state
Mapping cart + user details to internal data structure
Optimization for change
Correct prices (rounding)

## Composition

### Composable APIs

#### useCoupon
Exported vars:
- Ideally none

#### usePrice
Issues: 
- There are multiple in-between steps of calculating price. Which should be exposed? (use case: table of products not including delivery, payment, dynamic delivery based on order price)

> Calculating price on FE can be eliminated by waiting for sync after each quantity change
> This introduces UI delay but provides single source of truth for price calculations

##### Variant A (Collalloc)
Exporting a computed `dict` of productId mapped to the following prices:
- unit
- noVat (unit * quantity)
- vat (unit * quantity * vat)
Usage: `priceItems.itemId.vat`

##### Variant B
Exporting a function
- getProductPrice(item, variant)
Usage: `getProductPrice(item, 'vat')`

Discount calculation is implemented by checking for the `discount` in the **shared data store** and evaluating the case before generating prices (discount can be per-item).

Exported vars:
- priceItems
- priceTotal
- priceTotalVat
- priceDelivery
- pricePayment
- priceDiscount (show how much user saved)?

#### useProducts
Will enable **changing quantity** by simply `v-model="item.quantity"`
Price is shown by passing the item.id into the `usePrice` API to show computed price e.g. in summary or cart table.

Exported vars:
- items (including quantity, modifiable)

#### useLoader?
Handles UI updates

##### Variant A
Provide a composite of all the API call loaders - `store` variable holds an array of loaders for different API calls and one central loader is shown when appropriate based on this.

##### Variant B
Provide **loaders for specific UI elements** being changed instead (next to/instead of UI action elements) - upload coupon button; remove item icon...

#### useToast
Provides option to display any messages by calling a method. Closing UI options - automatic `timeout` or user action `dismiss`.
Must allow **stacking** messages.

When using `useAxios`, the error is a computed variable, but it also exposes `onSuccess` and `onError` methods (see [implementation](https://github.com/vueuse/vueuse/blob/main/packages/integrations/useAxios/index.ts#L82)).

Usage:
```
onError: (e) => {
	show(e.message)
}
```

Implementation:
Option to use library
https://github.com/Maronato/vue-toastification#features
Or write ourselves
https://derrickotte.medium.com/how-to-create-a-toasts-component-in-vue-3-with-vuex-1d8134541133

### Library composables

#### [useStorage](https://vueuse.org/core/useStorage/)
Enables sharing reactive state across composable functions.
Usage (within any scope):
```
const store = useStorage('vue-store', { items: [], user: 'John Doe' })
```

#### [useAxios](https://vueuse.org/integrations/useAxios/#useaxios)
Enables dynamic sync with BE (used for HT Configurator)

Should be attached to the global `store` variable

Issues:
- Which events should trigger sync? any update to store structure?
- There are different updates to store which need different UI behaviour (continuous sync vs coupon)

Usage: `execute()` allows to call a predefined method at any point. This should be initiated by handler functions or events.
Can be mapped to custom functions: `executeSync()`, `executeCoupon()`

API calls are **attached to the respective custom composables**:
`useCoupon` has access to `executeCoupon()` which handles coupon submission and assigns to the `coupon` property of the `store` variable.
Alternatively `executeCoupon()` returns the complete `store` variable in current state.

#### [useUrlSearchParams](https://vueuse.org/core/useUrlSearchParams/?foo=bar&vueuse=awesome)
Load state based on URL params (current step, product variant, etc.)

### Libraries

#### [FomKit](https://formkit.com/)
Possibly a good option for structuring form data.

⊕ Unification of inputs
⊕ Easy mapping of inputs to data structure
⊕ No v-model
⊕ Out of the box [validation](https://formkit.com/essentials/validation)
⊕ Out of the box [i18n](https://formkit.com/essentials/internationalization)
⊖ A bit annoying to style ([need to create a theme](https://formkit.com/essentials/styling#tailwind-css))

Alternative: [useAsyncValidator](https://vueuse.org/integrations/useAsyncValidator/#useasyncvalidator) - see HT Configurator

### [Cypress](https://www.cypress.io/)
Testing framework (E2E + Component)
