# LoyaltyLion in-cart rewards reference store

This repository contains a reference Shopify theme which has LoyaltyLion's in-cart rewards feature integrated. It can be used as a reference when adding this feature to your own Shopify theme.

To compare the changes we've made from the default Shopify theme, we have an open pull request which you can see here: https://github.com/loyaltylion-docs/shopify-incart-rewards-reference/pull/2

## Live reference store

https://loyaltylion-reference-9b3d.myshopify.com

* you'll need to be logged in and have at least one item in your cart to see the widget on the cart page
* upon signup you'll get lots of points to use

<img src='https://d.pr/i/IDU7y.png'>

## Adding in-cart rewards to your theme

### 1. Implement the changes shown the reference pull request

https://github.com/loyaltylion-docs/shopify-incart-rewards-reference/pull/2

* Hide reward products from collection views and search results
* Insert the "redemption widget" onto your cart page
* Use our reference CSS and customise it to match your theme's look and feel

### 2. Create your in-cart rewards

You can do this from your LoyaltyLion admin account on the "Create new reward" page. The easiest option is to duplicate an existing product to create a reward version of that product. These duplicated products have the title suffix "(Reward)" to differentiate them from regular products.

Keeping regular products separate from reward products keeps things organised, and allows you to manage their inventories separately.

When we create a duplicated product, we will:

* suffix the title with "(Reward)"
* tag the product with "reward" and "loyaltylion"
* set the price of the product's variants to a multiple of the original product's price
  * we do this because reward products are not intended to be purchased normally, only via the in-cart redemption mechanism

## The "redemption widget" in detail

The LoyaltyLion SDK will replace instances of `<div data-lion-seamless-product-reward-widget></div>` it finds on the page with a "carousel" style widget:

<img src='https://d.pr/i/w4e9yl.png' width='300'>

The buttons inside the widget will have the following classes:

* `loyaltylion-product-reward__button`
* `btn`

When a button is in a loading state we'll add to it the `data-lion-working` attribute. The CSS in our reference theme targets this attribute to display a loading spinner while the reward is being redeemed.

## Usage with Ajax carts

The reference implementation in this repository targets non-ajax carts. The redemption widget fully supports Ajax carts by firing events from the global `lion` object when we add or remove items from the cart.

```javascript
lion.on("cart.changed", function() {
  // fired any time we modify the cart. You should repaint the cart UI whenever
  // we emit this event
  yourCart.refresh();
});

// note: we always emit a `cart.changed` event at the same time as the other cart.*
// events

lion.on("cart.removedItems", function(data) {
  // fired whenever we remove items from the cart. We only ever remove reward items,
  // usually in response to paid items being removed, or the line items being adjusted
  // in such a way that there is no longer enough cart points to fund all the rewards
  // already in the cart

  // `data.items` will be an array of at least one shopify cart item objects, whose
  // shape is the same as is given via the Shopify Ajax API
  var items = data.items;
  console.log("loyaltylion removed cart items", items);
});

lion.on("cart.addedItem", function(data) {
  // fired whenever we add an item to the cart, which will always be a reward variant
  // we have created in response to user action

  // `data.item` is the line item object we added, with Shopify Ajax API line item shape
  var item = data.item;
  console.log("loyaltylion added a cart item", items);
});
```

You should also notify the `lion` object whenever your code makes a change to the cart that doesn't result in a full page reload. You can do this using the `lion.setCartState` method, which expects a full Shopify cart object as is returned from the `/cart.js` Ajax API call.

## Using in-cart rewards without the "redemption widget"

The redemption widget is not required to use in-cart rewards - it's just often a quick way to get started. It's also possible to render the "Buy with points" buttons by any other means.

To do this, add a link onto the page with the `data-lion-seamless-product-reward` attribute:

```html
<a href='#' data-lion-seamless-product-reward='{{ product.id }}' class='btn'></a>
```

The `product.id` should be the id of a product for which there is a corresponding reward in LoyaltyLion. For example, if you have a collection containing reward products, you could loop through and render a button for each one:

```html
{% for product in collection.rewards %}
  <h3>{{ product.name }}</h3>
  <a href='#' data-lion-seamless-product-reward='{{ product.id }}' class='btn'></a>
{% endfor %}
```

When the LoyaltyLion SDK loads, it'll look for these buttons, match them up with the corresponding reward and set the text of the button and add the redemption functionality on click.

You can find out more about using the standalone buttons in our docs here: https://loyaltylion.com/docs/seamless-product-rewards#advanced
