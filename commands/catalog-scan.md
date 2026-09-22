---
description: What a Shopify store sells, by collection and by price band
---

Scan a Shopify storefront.

Ask me for the store URL if I have not given it. The domain is enough.

Then:

1. Call `hasdata_shopify_collections_getCollections` with that `url`. Report the collections with their `products_count`, so I see the shape of the catalogue before anything is pulled. A count of zero is a staged or seasonal collection, not a failure.
2. Check that the response carries products or collections rather than an `error` string. A URL that is not a classic Shopify storefront still answers 200 with `status` reading `ok`, so retry a store you know is Shopify on its `myshopify.com` address before telling me it has nothing.
3. Call `hasdata_shopify_products_getProducts` with the `url`, the collection's `handle` as `collection`, and `limit` at 250. Page with `page` from 1 until a page comes back short, and say how many pages you pulled. There is no pagination block to tell you when to stop.
4. Parse every `price` and `compare_at_price` to a number before you rank or average anything. They arrive as strings, and sorting them as text is wrong in a way that still looks like an answer.
5. For each product report `title`, `vendor`, `product_type`, the tags, and a price range taken across `variants` rather than a single figure. A product with one variant has a single price, and saying which case it is matters.
6. Summarise the band, naming the cheapest and dearest variant in the collection, the median, and which `product_type` values dominate. Flag the products whose variants all report `available` as false, because those are sold out rather than cheap.
7. Strip the HTML out of `body_html` before quoting any description.

These are the store's published fields in Shopify's own snake_case. Quote them as they are and do not rename them in the output.
