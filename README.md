# Shopify MCP Server

<!-- mcp-name: com.hasdata/shopify -->

A hosted Model Context Protocol (MCP) server that gives Claude, Cursor, Windsurf and any other MCP client two read-only Shopify tools. Pull the product catalogue of any public Shopify storefront by its URL, with variants, SKUs and prices, and list the collections that organise it, all as structured JSON, with no app to install and no merchant token.

It reads the catalogue a signed-out visitor can see, on any classic Shopify storefront, whether it sits on a `myshopify.com` address or a custom domain. A headless store answers on its `myshopify.com` domain.

**1,000 free credits every month, no card required**, which is 200 Shopify calls at the 5-credit rate.

```
https://mcp.hasdata.com/mcp?apis=shopify
```

[![Glama score](https://glama.ai/mcp/servers/HasData/shopify-mcp/badges/score.svg)](https://glama.ai/mcp/servers/HasData/shopify-mcp)
[![tool contract](https://github.com/HasData/shopify-mcp/actions/workflows/contract.yml/badge.svg)](https://github.com/HasData/shopify-mcp/actions/workflows/contract.yml)
[![MCP](https://img.shields.io/badge/MCP-remote%20%7C%20streamable%20HTTP-6366f1?style=flat-square)](https://mcp.hasdata.com/mcp?apis=shopify)
[![Tools](https://img.shields.io/badge/tools-2-10b981?style=flat-square)](#tools)
[![npm](https://img.shields.io/npm/v/@hasdata/shopify-mcp?style=flat-square&logo=npm&label=npm&color=cb3837)](https://www.npmjs.com/package/@hasdata/shopify-mcp)
[![PyPI](https://img.shields.io/pypi/v/hasdata-shopify-mcp?style=flat-square&logo=pypi&logoColor=white&label=PyPI&color=3775a9)](https://pypi.org/project/hasdata-shopify-mcp/)
[![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](LICENSE)

## Contents

- [What you need](#what-you-need)
- [Quick start](#quick-start)
- [Example prompts](#example-prompts)
- [Tools](#tools)
- [Errors and failure paths](#errors-and-failure-paths)
- [Pricing, free tier and limits](#pricing-free-tier-and-limits)
- [Tool selection](#tool-selection)
- [How it compares](#how-it-compares)
- [FAQ](#faq)
- [HasData links](#hasdata-links)
- [Development](#development)
- [Contributing](#contributing)
- [License](#license)

## What you need

An MCP client and a HasData API key from the [dashboard](https://app.hasdata.com/sign-up?utm_source=github&utm_medium=syndication&utm_campaign=shopify-mcp), free to create with no card, and the free tier covers about 200 calls a month at the 5-credit rate. This is a remote server, so the simplest path is a URL and an `x-api-key` header, with no container to run. A client that only speaks stdio reaches it through a thin launcher, published as `@hasdata/shopify-mcp` on npm and `hasdata-shopify-mcp` on PyPI, shown below.

## Quick start

The server URL is the same for every client. We run it hands-on in Claude Code and Claude Desktop. The other blocks follow each client's own documented format for a remote server.

| Field | Value |
| :--- | :--- |
| URL | `https://mcp.hasdata.com/mcp?apis=shopify` |
| Transport | HTTP, streamable |
| Auth header | `x-api-key: HASDATA_API_KEY` |

Clients with OAuth support can add the same URL as a connector and sign in without putting a key in a config file.

<details>
<summary><b>Claude Code</b></summary>

```bash
claude mcp add --transport http shopify "https://mcp.hasdata.com/mcp?apis=shopify" \
  --header "x-api-key: HASDATA_API_KEY"
```

</details>

<details>
<summary><b>Claude Desktop</b></summary>

Settings, then Connectors, then Add custom connector, then paste `https://mcp.hasdata.com/mcp?apis=shopify` and sign in.

For the config-file route, Claude Desktop loads only local (stdio) servers, so it reaches a remote server through a stdio launcher. The `@hasdata/shopify-mcp` package is that launcher, and it reads the key from the environment. Add this to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "shopify": {
      "command": "npx",
      "args": ["-y", "@hasdata/shopify-mcp"],
      "env": { "HASDATA_API_KEY": "YOUR_KEY" }
    }
  }
}
```

For Python instead of Node, swap the launcher for the PyPI package, which `uvx` runs without a manual install:

```json
{
  "mcpServers": {
    "shopify": {
      "command": "uvx",
      "args": ["hasdata-shopify-mcp"],
      "env": { "HASDATA_API_KEY": "YOUR_KEY" }
    }
  }
}
```

</details>

<details>
<summary><b>Cursor</b></summary>

`~/.cursor/mcp.json` for every project, or `.cursor/mcp.json` for one:

```json
{
  "mcpServers": {
    "shopify": {
      "url": "https://mcp.hasdata.com/mcp?apis=shopify",
      "headers": { "x-api-key": "HASDATA_API_KEY" }
    }
  }
}
```

</details>

<details>
<summary><b>Windsurf</b></summary>

`~/.codeium/windsurf/mcp_config.json`. Windsurf calls the field `serverUrl`, not `url`:

```json
{
  "mcpServers": {
    "shopify": {
      "serverUrl": "https://mcp.hasdata.com/mcp?apis=shopify",
      "headers": { "x-api-key": "HASDATA_API_KEY" }
    }
  }
}
```

</details>

<details>
<summary><b>VS Code</b></summary>

`.vscode/mcp.json` in the workspace:

```json
{
  "servers": {
    "shopify": {
      "type": "http",
      "url": "https://mcp.hasdata.com/mcp?apis=shopify",
      "headers": { "x-api-key": "HASDATA_API_KEY" }
    }
  }
}
```

</details>

## Example prompts

Each of these lands on one tool, or on two in sequence when the second needs a handle the first returns.

- List the collections on allbirds.com and tell me which ones hold the most products.
- Pull the first 250 products from this store and group them by `product_type`.
- Which variants in this store are out of stock right now?
- Find every product on this store that is discounted, comparing `price` against `compare_at_price`.
- Page through the shoes collection on this storefront and give me the price range per size.
- Compare the sock prices on these two Shopify stores.

A prompt that names a category rather than a handle takes two calls, one to list the collections and one to pull the products in the matching handle. The collection tool returns `handle`, and that value goes straight into the `collection` argument of the product tool.

## Tools

| Tool | What it returns |
| --- | --- |
| `hasdata_shopify_collections_getCollections` | Each collection's id, title, handle, body_html description, image, and timestamps. 5 credits a call |
| `hasdata_shopify_products_getProducts` | Product id, title, handle, vendor, product_type, tags, body_html, images, variants with prices/SKUs/inventory status, and timestamps. 5 credits a call |

Two tools, 5 credits per successful call. Both take a storefront URL and page through the results with `limit` and `page`, where `limit` accepts up to 250.

### Get Shopify store products

[`hasdata_shopify_products_getProducts`](https://docs.hasdata.com/apis/shopify/products?utm_source=github&utm_medium=syndication&utm_campaign=shopify-mcp)

A page of products from one storefront.

| Parameter | Type | Required | Notes |
| :--- | :--- | :--- | :--- |
| `url` | string | yes | The storefront, such as `https://www.allbirds.com` |
| `limit` | number | | Products per page, 1 to 250 |
| `page` | number | | Page number, starting at 1 |
| `collection` | string | | Restrict to one collection, by its handle |

Returns a `products` array. Each product carries `id`, `title`, `handle`, `body_html`, `vendor`, `product_type`, `tags`, `published_at`, `created_at`, `updated_at`, an `options` array naming the axes the variants vary on, an `images` array, and a `variants` array.

The price lives on the variant, never on the product. A variant carries `id`, `title`, `sku`, `price`, `compare_at_price`, `available`, `grams`, `position`, `requires_shipping`, `taxable`, the `option1` through `option3` values and its own timestamps.

```json
{
  "id": 6889962537040,
  "title": "Anytime Ankle Sock - Basin Blue",
  "handle": "anytime-ankle-sock-basin-blue",
  "vendor": "Allbirds",
  "product_type": "Socks",
  "updated_at": "2026-09-09T05:25:09-07:00",
  "options": [{ "name": "Size", "position": 1, "values": ["S (W5-7)", "M (W8-10 / M8)", "L (W11 / M9-12)", "XL (M13-14)"] }],
  "variants": [
    {
      "id": 40356485202000,
      "title": "S (W5-7)",
      "sku": "A10842U001",
      "price": "16.00",
      "compare_at_price": null,
      "available": false,
      "grams": 59,
      "position": 1
    }
  ],
  "images": [
    {
      "id": 36355227844688,
      "position": 1,
      "src": "https://cdn.shopify.com/s/files/1/1104/4168/files/A10842_S24Q1_Anytime_Ankle_Sock_Basin_Blue_A-1400x1400.png?v=1776183348",
      "width": 1400,
      "height": 1400
    }
  ]
}
```

### Get Shopify store collections

[`hasdata_shopify_collections_getCollections`](https://docs.hasdata.com/apis/shopify/collections?utm_source=github&utm_medium=syndication&utm_campaign=shopify-mcp)

The collections that organise one storefront.

| Parameter | Type | Required | Notes |
| :--- | :--- | :--- | :--- |
| `url` | string | yes | The storefront, such as `https://www.allbirds.com` |
| `limit` | number | | Collections per page, 1 to 250 |
| `page` | number | | Page number, starting at 1 |

Returns a `collections` array. Each entry carries `id`, `title`, `handle`, `description`, `image`, `products_count`, `published_at` and `updated_at`.

This is the merchandising taxonomy as the store publishes it, which makes it the cheapest way to see how a competitor groups a catalogue before pulling any products. The `handle` is the join key into the product tool.

```json
{
  "id": 135995326544,
  "title": "Accessories",
  "handle": "womens-accessories",
  "description": "You know what they say: It's all in the details. Customize your look with planet-friendly face masks, hats, and more. ",
  "published_at": "2019-08-05T14:01:17-07:00",
  "updated_at": "2026-07-08T13:39:17-07:00",
  "image": null,
  "products_count": 28
}
```

## Errors and failure paths

Plan for these rather than assuming a happy path.

**`price` and `compare_at_price` are strings, not numbers.** They arrive as `"16.00"`, exactly as the storefront publishes them. Convert before you compare or sum, because string ordering puts `"9.00"` above `"16.00"`.

**A product has no price of its own.** Anything about cost has to go through the `variants` array, and a product with a size or colour axis usually has several prices. Reading the first variant and calling it the price is the most common mistake here.

**`compare_at_price` is null when nothing is discounted**, so a discount check is a null test first and a comparison second.

**`available` is per variant and reflects the moment of the call.** A product is not out of stock, a variant is, and stock moves. Two calls minutes apart can disagree, which is the point when you are monitoring, and a trap when you are diffing catalogues.

**A collection can report `products_count` of zero.** Stores leave empty, staged and seasonal collections published, so an empty collection is normal rather than a failed call.

**Stores publish things that are not for sale.** Internal, retired and staging items sit in the public catalogue on plenty of stores, sometimes flagged in the title and sometimes not. Filter on what you need instead of trusting that every row is a live product.

**`tags` are whatever the merchant wrote.** Some stores use them as plain keywords, others push namespaced metafield strings into them. Treat the array as free text.

**`body_html` is HTML.** Strip it before you index or embed the description.

**`collection.image` is often null**, and so is `featured_image` on a variant. Fall back to the product `images` array.

**A URL that is not a classic Shopify storefront still answers 200 and still bills.** There is no `products` array in that response and an `error` string in its place, while `requestMetadata.status` stays `ok`. Test for the array before you read it, because the shape changes rather than the status.

**A headless Shopify store fails on its custom domain and works on its `myshopify.com` one.** Headless shops serve the storefront from their own front end, so the catalogue is not published under the public domain. When a store you know runs Shopify comes back with the `error` string, retry it as `https://<shop>.myshopify.com`.

Results that carry data also carry a `requestMetadata.id` worth quoting in support.

## Pricing, free tier and limits

Each Shopify tool costs **5 credits per successful call**. Response size does not change the price, so a 250-product page and a 3-product page cost the same, which makes the largest page the cheapest way to mirror a catalogue.

The free tier is **1,000 credits every month with no card**, which is 200 Shopify calls at the base rate. It renews with the billing cycle, so a low-volume agent runs on the free tier indefinitely.

Paid plans start at **$59 a month** for 200,000 credits, which is 40,000 calls. The unit price falls with volume, from **$1.48 per 1,000 calls** on the entry plan to **$0.60** on Basic and **$0.41** across the Growth tiers. Current figures live on the [pricing page](https://hasdata.com/prices?utm_source=github&utm_medium=syndication&utm_campaign=shopify-mcp).

Your plan also sets concurrency. The free tier allows 1 request at a time, Startup 5, Basic 15, and the Growth tiers run from 50 to 500. Retry on the 429 with a backoff in anything unattended, because an agent that fans out across stores will reach the ceiling before you do.

A request that comes back non-200 is not billed. A successful call that finds nothing is still a call.

## Tool selection

Start from what the prompt gives you. A question about the catalogue itself goes to the product tool. A question about how the store is organised, or a prompt that names a category by its shop-facing name, goes to the collection tool first.

Then think about page size. Both tools accept `limit` up to 250 and cost the same at any size, so a catalogue of 900 products is four calls, not ninety. Leaving `limit` at its default is the most expensive habit you can pick up here.

Filter server-side when you can. Passing `collection` to the product tool costs one call and returns the subset, where pulling the whole catalogue and filtering locally costs one call per page of everything you did not want.

## How it compares

Shopify's Admin API is the official route to a store's catalogue, and it answers a different question.

| | Shopify Admin API | This server |
| :--- | :--- | :--- |
| Which stores | The ones you own or were granted access to | Any public storefront |
| Setup | Create an app, request scopes, hold a token per store | One header |
| Credential per store | Yes | No |
| Inventory levels | Exact counts | An `available` flag per variant |
| Draft and hidden products | Returned | Not returned, they are not public |
| Orders and customers | Returned | Not returned |
| Cost | Free within rate limits | Paid past the free tier, 5 credits a call |

The row that decides it is which stores. The Admin API is built for a merchant working on their own shop, and it needs a token that only that merchant can issue, which rules it out for comparing yourself against ten competitors. When the store is yours, the Admin API is more complete and free, and you should use it.

## FAQ

### Does Shopify have its own MCP server?

Yes, and it does something else. Every eligible storefront exposes one at its own domain, and its tools are built for an agent that is shopping, such as searching the catalogue, building a cart and running a checkout. It is per-store, so an agent comparing thirty shops needs thirty connections. This server is for reading catalogues in bulk across arbitrary stores, so the two do not overlap. If your agent is buying from one shop, use Shopify's.

### What is a Shopify MCP server?

An MCP server exposes tools an AI client can call. This one turns the public catalogue of any Shopify storefront into JSON an agent can reason over, without a browser or a scraping library in your stack.

### Do I need a Shopify account, an app or a merchant token?

No. The only credential is your HasData key.

### Does it work on custom domains?

For a classic storefront, yes, and a store on its own domain is the same as one on `myshopify.com`. A headless store is the exception. Its front end is served by something other than Shopify, so the catalogue is not published under the custom domain and the call comes back empty. Retry those as `https://<shop>.myshopify.com`.

### How do I tell whether a site runs Shopify?

Call the product tool on it, then look at the shape rather than the status. A classic Shopify storefront answers with a `products` array. Anything else answers 200 with an `error` string and no array, and that response is billed like any other successful call.

### Can I get inventory counts?

No, only the `available` flag each variant publishes. Exact stock levels are not public, and they come from the Admin API on a store you control.

### How do I pull a whole catalogue?

Page with `limit` at 250 and step `page` until a page comes back short or empty. Cost scales with pages, not with products, so the largest page size is always the cheapest route.

### Can I use this together with other HasData APIs?

Yes. One key covers everything, and one endpoint serves them all through the `apis` parameter. Point a client at `?apis=shopify,amazon` to get both tool sets in one connection, or at [`mcp.hasdata.com/api/mcp`](https://docs.hasdata.com/mcp-server?utm_source=github&utm_medium=syndication&utm_campaign=shopify-mcp) for the full catalogue.

### Is HasData affiliated with Shopify?

No. HasData is an independent service and is not affiliated with, endorsed by, or sponsored by Shopify. Shopify is a trademark of its respective owner. The tools work with publicly available data only, and you are responsible for using the results in line with the terms of the stores you read and the law that applies to you.

### Compliance and personal data

A product catalogue is business data, and these tools return no customer, order or contact information. The `vendor` field can carry a sole trader's own name on a small store, which is the one place a person can appear. Storing a competitor's catalogue is a commercial decision rather than a privacy one, so read the terms of the store you are pulling from and check your own obligations.

## HasData links

- [Shopify Scraper API](https://hasdata.com/apis/shopify-api?utm_source=github&utm_medium=syndication&utm_campaign=shopify-mcp), the REST endpoints behind these tools
- [API documentation](https://docs.hasdata.com/apis/shopify/products?utm_source=github&utm_medium=syndication&utm_campaign=shopify-mcp)
- [MCP server documentation](https://docs.hasdata.com/mcp-server?utm_source=github&utm_medium=syndication&utm_campaign=shopify-mcp)
- [Pricing](https://hasdata.com/prices?utm_source=github&utm_medium=syndication&utm_campaign=shopify-mcp)
- [Dashboard](https://app.hasdata.com/sign-up?utm_source=github&utm_medium=syndication&utm_campaign=shopify-mcp)

Other HasData MCP servers: [Google Search](https://github.com/HasData/google-search-mcp), [Google Maps](https://github.com/HasData/google-maps-mcp), [Google Trends](https://github.com/HasData/google-trends-mcp), [Google Flights](https://github.com/HasData/google-flights-mcp), [DuckDuckGo](https://github.com/HasData/duckduckgo-mcp), [YouTube](https://github.com/HasData/youtube-mcp), [TikTok](https://github.com/HasData/tiktok-mcp), [Instagram](https://github.com/HasData/instagram-mcp), [Amazon](https://github.com/HasData/amazon-mcp), [Yelp](https://github.com/HasData/yelp-mcp), [Zillow](https://github.com/HasData/zillow-mcp), [Airbnb](https://github.com/HasData/airbnb-mcp), [Booking.com](https://github.com/HasData/booking-mcp), [Indeed](https://github.com/HasData/indeed-mcp).

## Development

The launcher is a thin stdio bridge to the remote server, so there is nothing to build.

```bash
npm install
HASDATA_API_KEY=your_key_here npm test
```

The tests in `test/` assert the tool contract, the part that can break without a commit here. They check that `?apis=shopify` returns the expected tool count, that no name changed, that every tool still declares its required parameter and carries a description, and that the key in use is actually accepted. That last check calls a tool for real and costs 5 credits, which is the price of a canary that can fail for the right reason.

The contract suite also runs weekly on a schedule, because the upstream tool list can change without anyone touching this repository.

## Contributing

A tool table, a response sample or a documented behaviour that does not match reality is worth an issue. There is a template for exactly that. Pull requests are welcome for the same, and for anything in the launcher.

## License

MIT, see [LICENSE](LICENSE).
