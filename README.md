# Welcome! We're [BigCommerce](https://www.bigcommerce.com).

Day in, day out, we're creating the world's leading midmarket commerce platform. _Core to that is a great set of APIs, which developers like you can use to build amazing commerce experiences._ 

Currently, this repo contains Swagger files and documentation for our v3 catalog API, and a Request for Comment (RFC) on our upcoming Cart API. As new APIs are developed and discussed, you'll see their specs appear in this repo first.

## Public API Roadmap

We're in this together. Help us prioritize which APIs we build next: 
https://trello.com/b/1Od4oCsl/bigcommerce-api-roadmap

## v3 Catalog API

The new version of our catalog API is optimized for efficiency, while retaining backward compatibility with v2. The v3 API surfaces all the power of BigCommerce's market-leading catalog functionality, with a simplified variant and modifier model that makes for a much better integration experience. 

A few of the key features:

- Ability to create products, and all their variants, with one API call.
- Ability to fetch a list of all variants, separate from their parent product.
  - Simple products automatically create base variants.
- Clear delineation of options tied to SKUs, versus those that adjust only base products.
  - Options are now tied to variants, with adjustments tied to variant properties.
  - Modifiers are used when price, weight, and image adjustments don't require new variants.
- Rules are not required when using options or modifiers.
  - Modifiers have easy-to-understand adjusters instead.

View the v3 Catalog API overview [here](./docs/v3-catalog.md).

View the Swagger for the v3 Catalog API [here](https://raw.githubusercontent.com/bigcommerce/api/master/swagger/v3-catalog.yaml).

View the documentation generated from the Swagger file [here](http://editor.swagger.io/#/?import=https://raw.githubusercontent.com/bigcommerce/api/master/swagger/v3-catalog.yaml).

## Request for Comments on proposed BigCommerce Cart API

Welcome to the BigCommerce Cart API RFC!

In this document, we have opened up our proposed Cart API schema to solicit the developer community's feedback. Our goal is to ensure you, as a BigCommerce partner or developer, have the tools you need to build robust tools and cohesive integrations that benefit our merchants (your clients) and push our platform and ecosystem forward.

Building a world-class API is an iterative task. Our first release of this Cart API will be read-only, allowing you to access the contents and details of a BigCommerce cart on the storefront (via JavaScript), and to also access them via our backend HTTP REST API. Future releases will enable writable access to parts of the cart, unblocking many more possibilities and use cases.

Once we have gathered feedback, we will take time to digest all of the community's feedback and improve the API. The end result will be published on [BigEng.io](http://bigeng.io) once we have finalized the API specifications. This will detail the feedback that has been incorporated (or not), with the context for those changes.

Our initial specification takes into consideration use cases such as:

- Retrieve an abandoned cart.
- Reconstruct a cart from a URL.
- Support multiple shipping destinations for a single cart quote.
- Collect cart analytics, and build actionable reports.
- [your cool idea here].

This should help empower email remarketing, cart upsells, cross-browser carts for the same customer, and more.

### Introduction to Swagger API Documentation

[Swagger](http://swagger.io/) provides a set of rules for a format describing REST APIs. This format is both machine-readable and human-readable. Descriptions of each field are included inline, to help disambiguate terms – we might call things by different names internally than what you're used to.

If you are looking for something more readable, we suggest trying out the [Swagger browser editor](https://editor.swagger.io/#/?import=https://raw.githubusercontent.com/bigcommerce/api/master/swagger/checkout-draft.yaml).

### How to Provide Feedback

To provide your feedback on how we might improve this API, please submit comments on the latest pull request. Commenting directly on lines ([instructions here](https://developer.github.com/guides/working-with-comments/#pull-request-comments-on-a-line)) will help target your feedback to particular areas. We prefer this direct commenting in order to streamline conversation and centralize the discussion.

