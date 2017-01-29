# v3 Catalog API Documentation

_Welcome! Please note that this API is in our partner-release stage, which means that in the short term, we'll be iterating on feedback from partners. Our goal is to make sure that all concerns are addressed, and that we reach our goal of creating the most powerful, easiest-to-use catalog API in ecommerce. Because of this iterative approach, please expect small changes and many additions to occur._

**Have suggestions, feedback or questions? Submit them as an issue here:** 
https://github.com/bigcommerce/api/issues

**Want to see what we have in development, and help direct our roadmap? View our public API roadmap here:** https://trello.com/b/1Od4oCsl/bigcommerce-api-roadmap

## Access and Authentication

All BigCommerce stores have access to the v3 Catalog API.

The base URI is: https://api.bigcommerce.com/stores/{store_hash}/v3/

To authenticate, you'll need to use an OAuth client ID and token, sent along with the following headers:
  
- Accept: application/json  
- X-Auth-Client: {client_id}  
- X-Auth-Token: {oauth_token}

The flow to register for a client ID and retrieve a token is the same as with the v2 API:
- To get your Client ID, you must complete [App Registration](https://developer.bigcommerce.com/api/registration).
- To get your OAuth token, you must complete [App Installation](https://developer.bigcommerce.com/api/callback).

On our short-term roadmap is the ability to more easily create OAuth credentials, within the control panel – similar to the way legacy v2 keys are created now. _Note that in the future, we'll be deprecating legacy keys from v2, and removing the ability to create v2 keys within the CP. So grokking our OAuth flow now is not a wasted effort!_

Existing v2 client ids and tokens will also work with the v3 API. So, if you've already integrated with v2 using our OAuth flow, you should be golden!

## What's New?

- Variants
  - Every purchasable entity in the catalog is now a variant. When you create a product without any options, we automatically create a variant for you. This enables enhanced flows around inventory management, such as the ability to solely use the variants endpoint to manage inventory levels.
- Options and Modifiers
  - There is now a clear separation of options that define variants versus those that are modifiers of a variant. This enables us to simplify the creation and management of variant prices and modifier adjusters. It removes the need to use complex rules, in all but the most extreme cases.
  - Options and modifiers can also be attached directly to the product, without the requirement to create an option set beforehand.
- Creating a Product with its Variants in One API Call
  - When creating an initial catalog, you can send the core product and variant data in the same request. This helps you create more performant, managable codebases. We'll handle the option and option value creation for you.
- Including Variants in Product GETs
  - Following our goal of streamlining catalog management, you can now request to `?include=variants` (along with other resources). This further eliminates API calls.
- Ready-made Catalog Tree
  - There's now an endpoint specifically for building out the catalog tree, which previously took considerable work to construct.  
- Full Access to Modifer Configuration values
  - Properties like number-only field limits, and product-list inventory adjustment settings, are now available via this API. This exposes more than 20 properties previously unavailable to our developers.

## What's Not Here?

If you're currently consuming our v2 API, you'll notice that some catalog endpoints and elements are missing from this version. Some of the omissions are intentional; we're iterating on others, making sure they're done right. 

- Intentional
  - Product -> Configurable Fields
      - Modifiers should now be used for any use case where you'd use configurable fields. You can attach modifiers to products as well and they cover a larger array of uses. 
  - Option Sets
      - In v3, you attach options directly to products. So option sets are not required, and v3 includes no endpoint to manage options sets. However, v3 will respect option sets that have been attached via v2 or the control panel.
- Iterating
  - Product -> Complex Rules
      - Keep in mind that the majority of rule use cases can already be solved through variant properties and modifier adjusters.
  - Product -> Reviews
  - Product -> Videos
  - Product -> Downloads
  - Product -> Google Search Mappings
      - Might instead bring fields onto variant entity as properties/metadata.
  - Product -> Open Graph and Accounting Fields
      - Might group into their own product objects, to keep resource clean.

You can see how we're planning to iterate by looking at the [public API roadmap](https://trello.com/b/1Od4oCsl/bigcommerce-api-roadmap). 

## v2 Catalog API and Control-Panel Interoperability

The v3 Catalog API is essentially our catalog's future state. This means that many concepts don't map visibly to their v2 and control-panel relatives.

The good news here is we've built this API with v2 interoperability in mind. So you should be able to use both APIs simultaneously as you (in an ideal scenario) fully transition all catalog management to v3. The key areas to be aware of are:    

- Option Sets
  - The Product resource in v3 has an `option_set_id` field that, if set, will prevent you from directly editing product options and modifiers. If you want to edit the option set, you will need to either use v2, or else set the `option_set_id` field to null. The latter will remove the option set and allow you to directly attach options and modifiers.
  - In our control panel's Add/Edit Product section, any products created by v3 will have not have an option set applied, but merchants can still edit the options. If the merchant edits/chooses an option set, any variants will be removed from the product.
- Product Rules
  - Any variant created in v3 with non-null core properties (price, weight, image, purchasablilty) will create a rule under the hood. The same goes for modifier adjusters. These will show in v2 as product rules, and any edits to them will be shared across API versions.

_We're already refreshing our control panel's Add/Edit Product workflow to align with the concepts in v3._

### Product POST with Variants

When you include variants in your Product POST, we'll automatically create all the options and option values for you. If you don't pass the price and weight with the variants, the product price and weight will be used for the variants on the storefront.

Here's a sample POST to https://api.bigcommerce.com/stores/{store-hash}/v3/catalog/products:
```javascript
{
    "name": "T-shirt",
    "type": "physical",
    "price": 10.25,
    "description": "<h4>Great T-shirt</h4>The best t-shirt ever.",
    "weight": 1.20,
    "categories": [
        18
    ],
    "variants": [
        {
            "sku": "SKU-R-SM",
            "option_values": [
                {
                    "option_display_name": "Color",
                    "label": "Red"
                },
                {
                    "option_display_name": "Size",
                    "label": "Small"
                }
            ]
        },
        {
            "sku": "SKU-B-SM",
            "option_values": [
                {
                    "option_display_name": "Color",
                    "label": "Blue"
                },
                {
                    "option_display_name": "Size",
                    "label": "Small"
                }
            ]
        },
        {
            "sku": "SKU-R-MD",
            "option_values": [
                {
                    "option_display_name": "Color",
                    "label": "Red"
                },
                {
                    "option_display_name": "Size",
                    "label": "Medium"
                }
            ]
        },
        {
            "sku": "SKU-B-MD",
            "option_values": [
                {
                    "option_display_name": "Color",
                    "label": "Blue"
                },
                {
                    "option_display_name": "Size",
                    "label": "Medium"
                }
            ]
        },
        {
            "sku": "SKU-R-LG",
            "price": 10.50,
            "weight": 1.25,
            "option_values": [
                {
                    "option_display_name": "Color",
                    "label": "Red"
                },
                {
                    "option_display_name": "Size",
                    "label": "Large"
                }
            ]
        },
        {
            "sku": "SKU-B-LG",
            "price": 10.50,
            "weight": 1.25,
            "option_values": [
                {
                    "option_display_name": "Color",
                    "label": "Blue"
                },
                {
                    "option_display_name": "Size",
                    "label": "Large"
                }
            ]
        }
    ]
}
```

When you create a product, we'll automatically return variants in the response:
```javascript
{
    "data": {
        "id": 114,
        "name": "T-shirt",
        "type": 1,
        "sku": "",
        "description": "<h4>Great T-shirt</h4>The best t-shirt ever.",
        "weight": 1.2,
        "width": 0,
        "depth": 0,
        "height": 0,
        "price": 10.25,
        "cost_price": 0,
        "retail_price": 0,
        "sale_price": 0,
        "tax_class_id": 0,
        "product_tax_code": "",
        "calculated_price": 10.25,
        "categories": [
            18
        ],
        "brand_id": 0,
        "option_set_id": null,
        "inventory_level": 0,
        "inventory_warning_level": 0,
        "inventory_tracking": 0,
        "fixed_cost_shipping_price": 0,
        "is_free_shipping": false,
        "is_visible": true,
        "is_featured": false,
        "warranty": "",
        "bin_picking_number": "",
        "layout_file": "",
        "upc": "",
        "search_keywords": "",
        "availability": "available",
        "availability_description": "",
        "gift_wrapping_options": 0,
        "sort_order": 0,
        "condition": "New",
        "is_condition_shown": true,
        "order_quantity_minimum": 0,
        "order_quantity_maximum": 0,
        "page_title": "",
        "meta_keywords": [],
        "meta_description": "",
        "date_created": "2016-07-03T00:39:00+00:00",
        "date_modified": "2016-07-03T00:39:00+00:00",
        "view_count": 0,
        "preorder_release_date": null,
        "preorder_message": "",
        "is_preorder_only": false,
        "is_price_hidden": false,
        "price_hidden_label": "",
        "custom_url": {
            "url": "/t-shirt/",
            "is_customized": false
        },
        "variants": [
            {
                "id": 78,
                "product_id": 114,
                "sku": "SKU-R-SM",
                "sku_id": 127,
                "price": null,
                "weight": null,
                "purchasing_disabled": false,
                "purchasing_disabled_message": "",
                "image_file": null,
                "cost_price": 0,
                "upc": "",
                "inventory_level": 0,
                "inventory_warning_level": 0,
                "bin_picking_number": "",
                "option_values": [
                    {
                        "id": 98,
                        "option_id": 113
                    },
                    {
                        "id": 99,
                        "option_id": 114
                    }
                ]
            },
            {
                "id": 79,
                "product_id": 114,
                "sku": "SKU-B-SM",
                "sku_id": 128,
                "price": null,
                "weight": null,
                "purchasing_disabled": false,
                "purchasing_disabled_message": "",
                "image_file": null,
                "cost_price": 0,
                "upc": "",
                "inventory_level": 0,
                "inventory_warning_level": 0,
                "bin_picking_number": "",
                "option_values": [
                    {
                        "id": 100,
                        "option_id": 113
                    },
                    {
                        "id": 99,
                        "option_id": 114
                    }
                ]
            },
            {
                "id": 80,
                "product_id": 114,
                "sku": "SKU-R-MD",
                "sku_id": 129,
                "price": null,
                "weight": null,
                "purchasing_disabled": false,
                "purchasing_disabled_message": "",
                "image_file": null,
                "cost_price": 0,
                "upc": "",
                "inventory_level": 0,
                "inventory_warning_level": 0,
                "bin_picking_number": "",
                "option_values": [
                    {
                        "id": 98,
                        "option_id": 113
                    },
                    {
                        "id": 101,
                        "option_id": 114
                    }
                ]
            },
            {
                "id": 81,
                "product_id": 114,
                "sku": "SKU-B-MD",
                "sku_id": 130,
                "price": null,
                "weight": null,
                "purchasing_disabled": false,
                "purchasing_disabled_message": "",
                "image_file": null,
                "cost_price": 0,
                "upc": "",
                "inventory_level": 0,
                "inventory_warning_level": 0,
                "bin_picking_number": "",
                "option_values": [
                    {
                        "id": 100,
                        "option_id": 113
                    },
                    {
                        "id": 101,
                        "option_id": 114
                    }
                ]
            },
            {
                "id": 82,
                "product_id": 114,
                "sku": "SKU-R-LG",
                "sku_id": 131,
                "price": 10.5,
                "weight": 1.25,
                "purchasing_disabled": false,
                "purchasing_disabled_message": "",
                "image_file": null,
                "cost_price": 0,
                "upc": "",
                "inventory_level": 0,
                "inventory_warning_level": 0,
                "bin_picking_number": "",
                "option_values": [
                    {
                        "id": 98,
                        "option_id": 113
                    },
                    {
                        "id": 102,
                        "option_id": 114
                    }
                ]
            },
            {
                "id": 83,
                "product_id": 114,
                "sku": "SKU-B-LG",
                "sku_id": 132,
                "price": 10.5,
                "weight": 1.25,
                "purchasing_disabled": false,
                "purchasing_disabled_message": "",
                "image_file": null,
                "cost_price": 0,
                "upc": "",
                "inventory_level": 0,
                "inventory_warning_level": 0,
                "bin_picking_number": "",
                "option_values": [
                    {
                        "id": 100,
                        "option_id": 113
                    },
                    {
                        "id": 102,
                        "option_id": 114
                    }
                ]
            }
        ],
        "images": [],
        "custom_fields": [],
        "bulk_pricing_rules": []
    },
    "meta": {}
}
```

## Expanding Product Sub-Resources on GET

You can include sub-resources on a product, as a comma-separated list, by using `include={sub-resources}` as a query string. Valid expansions currently include `variants`, `images`, `custom_fields`, and `bulk_pricing_rules`. For instance, if you wanted variants and custom fields to also return in the product response, you'd GET: 
https://api.bigcommerce.com/stores/{store-hash}/v3/catalog/products?include=variants,custom_fields

# v3 Catalog API Reference

Please view the documentation generated from the Swagger file [here](http://editor.swagger.io/#/?import=https://raw.githubusercontent.com/bigcommerce/api/master/swagger/v3-catalog.yaml).

## <a name="__Methods">Methods/Endpoints</a>

BasePath: `/stores/{{store_id}}/v3`

[ Jump to [Models](#__Models) ]

### Table of Contents

#### [Catalog](#Catalog)

*   [`get /catalog/summary`](#catalogSummaryGet)
*   [`post /catalog/brands`](#createBrand)
*   [`post /catalog/brands/{brand_id}/image`](#createBrandImage)
*   [`post /catalog/brands/{brand_id}/metafields`](#createBrandMetafield)
*   [`post /catalog/categories`](#createCategory)
*   [`post /catalog/categories/{category_id}/image`](#createCategoryImage)
*   [`post /catalog/categories/{category_id}/metafields`](#createCategoryMetafield)
*   [`post /catalog/products/{product_id}/complex-rules`](#createComplexRule)
*   [`post /catalog/products/{product_id}/modifiers`](#createModifier)
*   [`post /catalog/products/{product_id}/modifiers/{modifier_id}/values/{value_id}/image`](#createModifierImage)
*   [`post /catalog/products/{product_id}/options`](#createOption)
*   [`post /catalog/products`](#createProduct)
*   [`post /catalog/products/{product_id}/images`](#createProductImage)
*   [`post /catalog/products/{product_id}/metafields`](#createProductMetafield)
*   [`post /catalog/products/{product_id}/videos`](#createProductVideo)
*   [`post /catalog/products/{product_id}/variants`](#createVariant)
*   [`post /catalog/products/{product_id}/variants/{variant_id}/image`](#createVariantImage)
*   [`post /catalog/products/{product_id}/variants/{variant_id}/metafields`](#createVariantMetafield)
*   [`delete /catalog/brands/{brand_id}`](#deleteBrandById)
*   [`delete /catalog/brands/{brand_id}/image`](#deleteBrandImage)
*   [`delete /catalog/brands/{brand_id}/metafields/{metafield_id}`](#deleteBrandMetafieldById)
*   [`delete /catalog/brands`](#deleteBrands)
*   [`delete /catalog/categories`](#deleteCategories)
*   [`delete /catalog/categories/{category_id}`](#deleteCategoryById)
*   [`delete /catalog/categories/{category_id}/image`](#deleteCategoryImage)
*   [`delete /catalog/categories/{category_id}/metafields/{metafield_id}`](#deleteCategoryMetafieldById)
*   [`delete /catalog/products/{product_id}/complex-rules/{complex_rule_id}`](#deleteComplexRuleById)
*   [`delete /catalog/products/{product_id}/modifiers/{modifier_id}`](#deleteModifierById)
*   [`delete /catalog/products/{product_id}/modifiers/{modifier_id}/values/{value_id}/image`](#deleteModifierImage)
*   [`delete /catalog/products/{product_id}/options/{option_id}`](#deleteOptionById)
*   [`delete /catalog/products/{product_id}`](#deleteProductById)
*   [`delete /catalog/products/{product_id}/images/{image_id}`](#deleteProductImage)
*   [`delete /catalog/products/{product_id}/metafields/{metafield_id}`](#deleteProductMetafieldById)
*   [`delete /catalog/products/{product_id}/videos/{video_id}`](#deleteProductVideo)
*   [`delete /catalog/products`](#deleteProducts)
*   [`delete /catalog/products/{product_id}/variants/{variant_id}`](#deleteVariantById)
*   [`delete /catalog/products/{product_id}/variants/{variant_id}/metafields/{metafield_id}`](#deleteVariantMetafieldById)
*   [`get /catalog/brands/{brand_id}`](#getBrandById)
*   [`get /catalog/brands/{brand_id}/metafields/{metafield_id}`](#getBrandMetafieldByBrandId)
*   [`get /catalog/brands/{brand_id}/metafields`](#getBrandMetafieldsByBrandId)
*   [`get /catalog/brands`](#getBrands)
*   [`get /catalog/categories`](#getCategories)
*   [`get /catalog/categories/{category_id}`](#getCategoryById)
*   [`get /catalog/categories/{category_id}/metafields/{metafield_id}`](#getCategoryMetafieldByCategoryId)
*   [`get /catalog/categories/{category_id}/metafields`](#getCategoryMetafieldsByCategoryId)
*   [`get /catalog/categories/tree`](#getCategoryTree)
*   [`get /catalog/products/{product_id}/complex-rules/{complex_rule_id}`](#getComplexRuleById)
*   [`get /catalog/products/{product_id}/complex-rules`](#getComplexRules)
*   [`get /catalog/products/{product_id}/modifiers/{modifier_id}`](#getModifierById)
*   [`get /catalog/products/{product_id}/modifiers`](#getModifiers)
*   [`get /catalog/products/{product_id}/options/{option_id}`](#getOptionById)
*   [`get /catalog/products/{product_id}/options`](#getOptions)
*   [`get /catalog/products/{product_id}`](#getProductById)
*   [`get /catalog/products/{product_id}/images/{image_id}`](#getProductImageById)
*   [`get /catalog/products/{product_id}/images`](#getProductImages)
*   [`get /catalog/products/{product_id}/metafields/{metafield_id}`](#getProductMetafieldByProductId)
*   [`get /catalog/products/{product_id}/metafields`](#getProductMetafieldsByProductId)
*   [`get /catalog/products/{product_id}/videos/{video_id}`](#getProductVideoById)
*   [`get /catalog/products/{product_id}/videos`](#getProductVideos)
*   [`get /catalog/products`](#getProducts)
*   [`get /catalog/products/{product_id}/variants/{variant_id}`](#getVariantById)
*   [`get /catalog/products/{product_id}/variants/{variant_id}/metafields/{metafield_id}`](#getVariantMetafieldByProductIdAndVariantId)
*   [`get /catalog/products/{product_id}/variants/{variant_id}/metafields`](#getVariantMetafieldsByProductIdAndVariantId)
*   [`get /catalog/variants`](#getVariants)
*   [`get /catalog/products/{product_id}/variants`](#getVariantsByProductId)
*   [`put /catalog/brands/{brand_id}`](#updateBrand)
*   [`put /catalog/brands/{brand_id}/metafields/{metafield_id}`](#updateBrandMetafield)
*   [`put /catalog/categories/{category_id}`](#updateCategory)
*   [`put /catalog/categories/{category_id}/metafields/{metafield_id}`](#updateCategoryMetafield)
*   [`put /catalog/products/{product_id}/complex-rules/{complex_rule_id}`](#updateComplexRule)
*   [`put /catalog/products/{product_id}/modifiers/{modifier_id}`](#updateModifier)
*   [`put /catalog/products/{product_id}/options/{option_id}`](#updateOption)
*   [`put /catalog/products/{product_id}`](#updateProduct)
*   [`put /catalog/products/{product_id}/images/{image_id}`](#updateProductImage)
*   [`put /catalog/products/{product_id}/metafields/{metafield_id}`](#updateProductMetafield)
*   [`put /catalog/products/{product_id}/videos/{video_id}`](#updateProductVideo)
*   [`put /catalog/products/{product_id}/variants/{variant_id}`](#updateVariant)
*   [`put /catalog/products/{product_id}/variants/{variant_id}/metafields/{metafield_id}`](#updateVariantMetafield)

#### [Customers](#Customers)

*   [`post /customers/subscribers`](#createSubscriber)
*   [`delete /customers/subscribers/{subscriber_id}`](#deleteSubscriberById)
*   [`delete /customers/subscribers`](#deleteSubscribers)
*   [`get /customers/subscribers/{subscriber_id}`](#getSubscriberById)
*   [`get /customers/subscribers`](#getSubscribers)
*   [`put /customers/subscribers/{subscriber_id}`](#updateSubscriber)

## <a name="Catalog">Catalog</a>

## <div class="method"><a name="catalogSummaryGet"></a> get /catalog/summary

</div>

<div class="method-summary">(<span class="nickname">catalogSummaryGet</span>)</div>

<div class="method-notes">Returns a lightweight inventory summary from the BigCommerce Catalog.</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Return type

[CatalogSummaryResponse](#CatalogSummaryResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "inventory_value" : 1.3579000000000001069366817318950779736042022705078125,
        "inventory_count" : 123,
        "primary_category_id" : 123,
        "primary_category_name" : "aeiou"
      },
      "meta" : { }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

An array of catalog summary and metadata. [CatalogSummaryResponse](#CatalogSummaryResponse)</div>

* * *

## <div class="method"><a name="createBrand"></a> post /catalog/brands

</div>

<div class="method-summary">(<span class="nickname">createBrand</span>)</div>

<div class="method-notes">Creates a `Brand` object.</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Request body

<div class="field-items">

</p>
<div class="param">Brand [Brand](#Brand) (required)</div>

Body Parameter — A `Brand` object.

</div>

### Return type

[BrandResponse](#BrandResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "meta_description" : "aeiou",
        "page_title" : "aeiou",
        "image_url" : "aeiou",
        "name" : "aeiou",
        "id" : 123,
        "meta_keywords" : [ "aeiou" ],
        "search_keywords" : "aeiou"
      },
      "meta" : { }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A brand object [BrandResponse](#BrandResponse)

#### 409

Brand was in conflict with another brand. This is the result of duplicate unique fields such as name. [ErrorResponse](#ErrorResponse)

#### 422

Brand was not valid. This is the result of missing required fields, or of invalid data. See the response for more details. [ErrorResponse](#ErrorResponse)</div>

* * *

## <div class="method"><a name="createBrandImage"></a> post /catalog/brands/{brand_id}/image

</div>

<div class="method-summary">(<span class="nickname">createBrandImage</span>)</div>

<div class="method-notes">Creates an image on a `Brand`. Publicly accessible URLs and files (form post) are valid parameters.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">brand_id (required)</div>

Path Parameter — The ID of the `Brand` to which the image is being attached.

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `multipart/form-data`

### Form parameters

<div class="field-items">

</p>
<div class="param">image_file (required)</div>

Form Parameter — An image file. Supported MIME types include GIF, JPEG, and PNG.

</div>

### Return type

[ImageResponse](#ImageResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "image_url" : "aeiou"
      },
      "meta" : { }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A ResourceImage and metadata. [ImageResponse](#ImageResponse)

#### 404

The resource was not found. [NotFound](#NotFound)

#### 422

Image was not valid. This is the result of a missing `image_file` field or an incorrect file type. See the response for more details. [ErrorResponse](#ErrorResponse)</div>

* * *

## <div class="method"><a name="createBrandMetafield"></a> post /catalog/brands/{brand_id}/metafields

</div>

<div class="method-summary">(<span class="nickname">createBrandMetafield</span>)</div>

<div class="method-notes">Creates a product `Metafield`.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">brand_id (required)</div>

Path Parameter — The ID of the `Brand` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Request body

<div class="field-items">

</p>
<div class="param">Metafield [Metafield](#Metafield) (required)</div>

Body Parameter — A `Metafield` object.

</div>

### Query parameters

<div class="field-items">

</p>
<div class="param">key (optional)</div>

Query Parameter — Filter based on a metafield's key.

</p>
<div class="param">namespace (optional)</div>

Query Parameter — Filter based on a metafield's key.

</div>

### Return type

(MetafieldResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "updated_at" : "2000-01-23T04:56:07.000+00:00",
        "namespace" : "aeiou",
        "resource_type" : "aeiou",
        "description" : "aeiou",
        "resource_id" : 123,
        "created_at" : "2000-01-23T04:56:07.000+00:00",
        "id" : 123,
        "value" : "aeiou",
        "key" : "aeiou",
        "permission_set" : "aeiou"
      },
      "meta" : { }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A `Metafield` object. [MetafieldResponse](#MetafieldResponse)

#### 409

The `Metafield` was in conflict with another `Metafield`. This can be the result of duplicate unique key combination of the app's client id, namespace, key, resource_type, and resource_id. [ErrorResponse](#ErrorResponse)

#### 422

The `Metafield` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details. [ErrorResponse](#ErrorResponse)</div>

* * *

## <div class="method"><a name="createCategory"></a> post /catalog/categories

</div>

<div class="method-summary">(<span class="nickname">createCategory</span>)</div>

<div class="method-notes">Creates a `Category` in the BigCommerce Catalog.</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Request body

<div class="field-items">

</p>
<div class="param">category [Category](#Category) (required)</div>

Body Parameter — A BigCommerce `Category` object.

</div>

### Return type

[CategoryResponse](#CategoryResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "is_visible" : true,
        "page_title" : "aeiou",
        "image_url" : "aeiou",
        "description" : "aeiou",
        "meta_keywords" : [ "aeiou" ],
        "search_keywords" : "aeiou",
        "default_product_sort" : "aeiou",
        "meta_description" : "aeiou",
        "layout_file" : "aeiou",
        "parent_id" : 123,
        "name" : "aeiou",
        "id" : 123,
        "sort_order" : 123,
        "views" : 123,
        "custom_url" : {
          "is_customized" : true,
          "url" : "aeiou"
        }
      },
      "meta" : { }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A category object. [CategoryResponse](#CategoryResponse)

#### 409

The `Category` was in conflict with another category. This is the result of duplicate unique values, such as `name` or `custom_url`. [ErrorResponse](#ErrorResponse)

#### 422

The `Category` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details. [ErrorResponse](#ErrorResponse)</div>

* * *

## <div class="method"><a name="createCategoryImage"></a> post /catalog/categories/{category_id}/image

</div>

<div class="method-summary">(<span class="nickname">createCategoryImage</span>)</div>

<div class="method-notes">Creates an image on a category. Publicly accessible URLs and files (form post) are valid parameters.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">category_id (required)</div>

Path Parameter — The ID of the `Category` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `multipart/form-data`

### Form parameters

<div class="field-items">

</p>
<div class="param">image_file (required)</div>

Form Parameter — An image file. Supported MIME types include GIF, JPEG, and PNG.

</div>

### Return type

[ImageResponse](#ImageResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "image_url" : "aeiou"
      },
      "meta" : { }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A ResourceImage and metadata. [ImageResponse](#ImageResponse)

#### 404

The resource was not found. [NotFound](#NotFound)

#### 422

Image was not valid. This is the result of a missing `image_file` field or an incorrect file type. See the response for more details. [ErrorResponse](#ErrorResponse)</div>

* * *

## <div class="method"><a name="createCategoryMetafield"></a> post /catalog/categories/{category_id}/metafields

</div>

<div class="method-summary">(<span class="nickname">createCategoryMetafield</span>)</div>

<div class="method-notes">Creates a product `Metafield`.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">category_id (required)</div>

Path Parameter — The ID of the `Category` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Request body

<div class="field-items">

</p>
<div class="param">Metafield [Metafield](#Metafield) (required)</div>

Body Parameter — A `Metafield` object.

</div>

### Return type

[MetafieldResponse](#MetafieldResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "updated_at" : "2000-01-23T04:56:07.000+00:00",
        "namespace" : "aeiou",
        "resource_type" : "aeiou",
        "description" : "aeiou",
        "resource_id" : 123,
        "created_at" : "2000-01-23T04:56:07.000+00:00",
        "id" : 123,
        "value" : "aeiou",
        "key" : "aeiou",
        "permission_set" : "aeiou"
      },
      "meta" : { }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A `Metafield` object. [MetafieldResponse](#MetafieldResponse)

#### 409

The `Metafield` was in conflict with another `Metafield`. This can be the result of duplicate unique key combinations of the app's client id, namespace, key, resource_type, and resource_id. [ErrorResponse](#ErrorResponse)

#### 422

The `Metafield` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details. [ErrorResponse](#ErrorResponse)</div>

* * *

## <div class="method"><a name="createComplexRule"></a> post /catalog/products/{product_id}/complex-rules

</div>

<div class="method-summary">(<span class="nickname">createComplexRule</span>)</div>

<div class="method-notes">Creates a `ComplexRule`.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the `ComplexRule` belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Request body

<div class="field-items">

</p>
<div class="param">ComplexRule [ComplexRule](#ComplexRule) (required)</div>

Body Parameter — `ComplexRule` object

</div>

### Return type

[ComplexRuleResponse](#ComplexRuleResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "purchasing_hidden" : true,
        "stop" : true,
        "price_adjuster" : {
          "adjuster" : "aeiou",
          "adjuster_value" : 1.3579000000000001069366817318950779736042022705078125
        },
        "image_url" : "aeiou",
        "product_id" : 123,
        "weight_adjuster" : "",
        "id" : 123,
        "purchasing_disabled" : true,
        "purchasing_disabled_message" : "aeiou",
        "conditions" : [ {
          "rule_id" : 123,
          "variant_id" : 123,
          "combination_id" : 123,
          "modifier_id" : 123,
          "modifier_value_id" : 123,
          "id" : 123
        } ],
        "sort_order" : 123,
        "enabled" : true
      },
      "meta" : {
        "per_page" : 123,
        "total" : 123,
        "count" : 123,
        "links" : {
          "next" : "aeiou",
          "current" : "aeiou",
          "previous" : "aeiou"
        },
        "total_pages" : 123,
        "current_page" : 123
      }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A `ComplexRule` object [ComplexRuleResponse](#ComplexRuleResponse)

#### 409

The `ComplexRule` was in conflict with another `ComplexRule`. This is the result of duplicate conditions. [ErrorResponse](#ErrorResponse)

#### 422

The `ComplexRule` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details. [ErrorResponse](#ErrorResponse)</div>

* * *

## <div class="method"><a name="createModifier"></a> post /catalog/products/{product_id}/modifiers

</div>

<div class="method-summary">(<span class="nickname">createModifier</span>)</div>

<div class="method-notes">Creates a `Modifier`.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the `Modifier` belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Request body

<div class="field-items">

</p>
<div class="param">Modifier [Modifier](#Modifier) (required)</div>

Body Parameter — A `Modifier` object.

</div>

### Return type

[ModifierResponse](#ModifierResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "product_id" : 123,
        "option_values" : [ "" ],
        "name" : "aeiou",
        "id" : 123,
        "display_name" : "aeiou",
        "type" : "aeiou",
        "config" : {
          "text_max_length" : 123,
          "file_types_other" : [ "aeiou" ],
          "file_max_size" : 123,
          "file_types_supported" : [ "aeiou" ],
          "text_characters_limited" : true,
          "product_list_adjusts_inventory" : true,
          "number_limited" : true,
          "checked_by_default" : true,
          "date_latest_value" : "2000-01-23",
          "product_list_adjusts_pricing" : true,
          "default_value" : "aeiou",
          "date_limited" : true,
          "text_max_lines" : 123,
          "checkbox_label" : "aeiou",
          "text_min_length" : 123,
          "file_types_mode" : "aeiou",
          "text_lines_limited" : true,
          "number_highest_value" : 1.3579000000000001069366817318950779736042022705078125,
          "date_earliest_value" : "2000-01-23",
          "date_limit_mode" : "aeiou",
          "number_lowest_value" : 1.3579000000000001069366817318950779736042022705078125,
          "number_integers_only" : true,
          "product_list_shipping_calc" : "aeiou",
          "number_limit_mode" : "aeiou"
        },
        "required" : true
      },
      "meta" : {
        "per_page" : 123,
        "total" : 123,
        "count" : 123,
        "links" : {
          "next" : "aeiou",
          "current" : "aeiou",
          "previous" : "aeiou"
        },
        "total_pages" : 123,
        "current_page" : 123
      }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A modifier object. [ModifierResponse](#ModifierResponse)

#### 409

The `Modifier` was in conflict with another option. This is the result of duplicate unique fields, such as `name`. [ErrorResponse](#ErrorResponse)

#### 422

The `Modifier` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details. [ErrorResponse](#ErrorResponse)</div>

* * *

## <div class="method"><a name="createModifierImage"></a> post /catalog/products/{product_id}/modifiers/{modifier_id}/values/{value_id}/image

</div>

<div class="method-summary">(<span class="nickname">createModifierImage</span>)</div>

<div class="method-notes">Adds an image to a modifier value; the image will show on the storefront when the value is selected.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the `Modifier` belongs. format: int

</p>
<div class="param">modifier_id (required)</div>

Path Parameter — The ID of the `Modifier`.

</p>
<div class="param">value_id (required)</div>

Path Parameter — The ID of the `Modifier`.

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `multipart/form-data`

### Form parameters

<div class="field-items">

</p>
<div class="param">image_file (required)</div>

Form Parameter — An image file. Supported MIME types include GIF, JPEG, and PNG.

</div>

### Return type

[ImageResponse](#ImageResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "image_url" : "aeiou"
      },
      "meta" : { }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A ResourceImage and metadata. [ImageResponse](#ImageResponse)

#### 404

The resource was not found. [NotFound](#NotFound)

#### 422

Modifier image was not valid. This is the result of missing `image_file` fields, orof a non-URL value for the `image_file` field. See the response for more details. [ErrorResponse](#ErrorResponse)</div>

* * *

## <div class="method"><a name="createOption"></a> post /catalog/products/{product_id}/options

</div>

<div class="method-summary">(<span class="nickname">createOption</span>)</div>

<div class="method-notes">Creates an `Option`.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Request body

<div class="field-items">

</p>
<div class="param">Option [Option](#Option) (required)</div>

Body Parameter — An `Option` object.

</div>

### Return type

[OptionResponse](#OptionResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "product_id" : 123,
        "option_values" : [ {
          "id" : 123,
          "label" : "aeiou",
          "value_data" : "{}",
          "is_default" : true,
          "sort_order" : 123
        } ],
        "name" : "aeiou",
        "id" : 123,
        "display_name" : "aeiou",
        "type" : "aeiou",
        "config" : {
          "text_max_length" : 123,
          "file_types_other" : [ "aeiou" ],
          "file_max_size" : 123,
          "file_types_supported" : [ "aeiou" ],
          "text_characters_limited" : true,
          "product_list_adjusts_inventory" : true,
          "number_limited" : true,
          "checked_by_default" : true,
          "date_latest_value" : "2000-01-23",
          "product_list_adjusts_pricing" : true,
          "default_value" : "aeiou",
          "date_limited" : true,
          "text_max_lines" : 123,
          "checkbox_label" : "aeiou",
          "text_min_length" : 123,
          "file_types_mode" : "aeiou",
          "text_lines_limited" : true,
          "number_highest_value" : 1.3579000000000001069366817318950779736042022705078125,
          "date_earliest_value" : "2000-01-23",
          "date_limit_mode" : "aeiou",
          "number_lowest_value" : 1.3579000000000001069366817318950779736042022705078125,
          "number_integers_only" : true,
          "product_list_shipping_calc" : "aeiou",
          "number_limit_mode" : "aeiou"
        }
      },
      "meta" : {
        "per_page" : 123,
        "total" : 123,
        "count" : 123,
        "links" : {
          "next" : "aeiou",
          "current" : "aeiou",
          "previous" : "aeiou"
        },
        "total_pages" : 123,
        "current_page" : 123
      }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

An option object. [OptionResponse](#OptionResponse)

#### 409

Option was in conflict with another option. This is the result of duplicate unique fields, such as `name`. [ErrorResponse](#ErrorResponse)

#### 422

Option was not valid. This is the result of missing required fields, or of invalid data. See the response for more details. [ErrorResponse](#ErrorResponse)</div>

* * *

## <div class="method"><a name="createProduct"></a> post /catalog/products

</div>

<div class="method-summary">(<span class="nickname">createProduct</span>)</div>

<div class="method-notes">Creates a `Product` in the BigCommerce Catalog.</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Request body

<div class="field-items">

</p>
<div class="param">product [ProductPost](#ProductPost) (required)</div>

Body Parameter — A BigCommerce `Product` object.

</div>

### Return type

[ProductResponse](#ProductResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "calculated_price" : 1.3579000000000001069366817318950779736042022705078125,
        "gift_wrapping_options_list" : [ 123 ],
        "page_title" : "aeiou",
        "videos" : [ {
          "product_id" : 123,
          "length" : "aeiou",
          "description" : "aeiou",
          "id" : 123,
          "title" : "aeiou",
          "sort_order" : 123
        } ],
        "is_condition_shown" : true,
        "variants" : [ {
          "image_url" : "aeiou",
          "option_values" : [ {
            "option_display_name" : "aeiou",
            "option_id" : 123,
            "id" : 123,
            "label" : "aeiou"
          } ],
          "weight" : 1.3579000000000001069366817318950779736042022705078125,
          "upc" : "aeiou",
          "bin_picking_number" : "aeiou",
          "sku_id" : 123,
          "purchasing_disabled_message" : "aeiou",
          "inventory_level" : 123,
          "price" : 1.3579000000000001069366817318950779736042022705078125,
          "inventory_warning_level" : 123,
          "product_id" : 123,
          "id" : 123,
          "purchasing_disabled" : true,
          "sku" : "aeiou",
          "cost_price" : 1.3579000000000001069366817318950779736042022705078125
        } ],
        "type" : "aeiou",
        "retail_price" : 1.3579000000000001069366817318950779736042022705078125,
        "layout_file" : "aeiou",
        "price" : 1.3579000000000001069366817318950779736042022705078125,
        "inventory_warning_level" : 123,
        "warranty" : "aeiou",
        "is_free_shipping" : true,
        "id" : 123,
        "sku" : "aeiou",
        "height" : 1.3579000000000001069366817318950779736042022705078125,
        "custom_url" : {
          "is_customized" : true,
          "url" : "aeiou"
        },
        "images" : [ "" ],
        "custom_fields" : [ {
          "product_id" : 123,
          "name" : "aeiou",
          "id" : 123,
          "value" : "aeiou"
        } ],
        "weight" : 1.3579000000000001069366817318950779736042022705078125,
        "upc" : "aeiou",
        "brand_id" : 123,
        "meta_description" : "aeiou",
        "condition" : "aeiou",
        "inventory_level" : 123,
        "name" : "aeiou",
        "inventory_tracking" : "aeiou",
        "preorder_release_date" : "aeiou",
        "description" : "aeiou",
        "bin_picking_number" : "aeiou",
        "availability" : "aeiou",
        "search_keywords" : "aeiou",
        "meta_keywords" : [ "aeiou" ],
        "is_price_hidden" : true,
        "order_quantity_minimum" : 123,
        "availability_description" : "aeiou",
        "fixed_cost_shipping_price" : 123,
        "categories" : [ 123 ],
        "sort_order" : 123,
        "cost_price" : 1.3579000000000001069366817318950779736042022705078125,
        "order_quantity_maximum" : 123,
        "is_visible" : true,
        "is_preorder_only" : true,
        "date_created" : "aeiou",
        "preorder_message" : "aeiou",
        "tax_class_id" : 123,
        "bulk_pricing_rules" : [ {
          "amount" : 1.3579000000000001069366817318950779736042022705078125,
          "quantity_min" : 123,
          "quantity_max" : 123,
          "id" : 123,
          "type" : "aeiou"
        } ],
        "sale_price" : 1.3579000000000001069366817318950779736042022705078125,
        "product_tax_code" : "aeiou",
        "depth" : 1.3579000000000001069366817318950779736042022705078125,
        "date_modified" : "aeiou",
        "gift_wrapping_options_type" : "aeiou",
        "width" : 1.3579000000000001069366817318950779736042022705078125,
        "price_hidden_label" : "aeiou",
        "is_featured" : true,
        "view_count" : 123
      },
      "meta" : { }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A product [ProductResponse](#ProductResponse)

#### 409

`Product` was in conflict with another product. This is the result of duplicate unique values, such as name or SKU; a missing or invalid category id, brand id, or tax_class id; or a conflicting `bulk_pricing_rule`. [ErrorResponse](#ErrorResponse)

#### 422

`Product` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details. [ErrorResponse](#ErrorResponse)</div>

* * *

## <div class="method"><a name="createProductImage"></a> post /catalog/products/{product_id}/images

</div>

<div class="method-summary">(<span class="nickname">createProductImage</span>)</div>

<div class="method-notes">Creates an image on a product. Publically accessible URLs and files (form post) are valid parameters.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the image is being attached.

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Request body

<div class="field-items">

</p>
<div class="param">productImage [ProductImagePost](#ProductImagePost) (required)</div>

Body Parameter — A BigCommerce `ProductImage` object.

</div>

### Return type

[ProductImageResponse](#ProductImageResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : "",
      "meta" : { }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A product image. [ProductImageResponse](#ProductImageResponse)

#### 404

The product ID does not exist. [NotFound](#NotFound)</div>

* * *

## <div class="method"><a name="createProductMetafield"></a> post /catalog/products/{product_id}/metafields

</div>

<div class="method-summary">(<span class="nickname">createProductMetafield</span>)</div>

<div class="method-notes">Creates a product `Metafield`.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Request body

<div class="field-items">

</p>
<div class="param">Metafield [Metafield](#Metafield) (required)</div>

Body Parameter — `Metafield` object

</div>

### Return type

[MetafieldResponse](#MetafieldResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "updated_at" : "2000-01-23T04:56:07.000+00:00",
        "namespace" : "aeiou",
        "resource_type" : "aeiou",
        "description" : "aeiou",
        "resource_id" : 123,
        "created_at" : "2000-01-23T04:56:07.000+00:00",
        "id" : 123,
        "value" : "aeiou",
        "key" : "aeiou",
        "permission_set" : "aeiou"
      },
      "meta" : { }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A `Metafield` object [MetafieldResponse](#MetafieldResponse)

#### 409

The `Metafield` was in conflict with another `Metafield`. This can be the result of duplicate unique key combinations of the app's client id, namespace, key, resource_type, and resource_id. [ErrorResponse](#ErrorResponse)

#### 422

The `Metafield` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details. [ErrorResponse](#ErrorResponse)</div>

* * *

## <div class="method"><a name="createProductVideo"></a> post /catalog/products/{product_id}/videos

</div>

<div class="method-summary">(<span class="nickname">createProductVideo</span>)</div>

<div class="method-notes">Creates a video on a product, using a video ID from YouTube.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the video is being attached.

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Request body

<div class="field-items">

</p>
<div class="param">productVideo [ProductVideo](#ProductVideo) (required)</div>

Body Parameter — A BigCommerce `ProductVideo` object.

</div>

### Return type

[ProductVideoResponse](#ProductVideoResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "product_id" : 123,
        "length" : "aeiou",
        "description" : "aeiou",
        "id" : 123,
        "title" : "aeiou",
        "sort_order" : 123
      },
      "meta" : {
        "per_page" : 123,
        "total" : 123,
        "count" : 123,
        "links" : {
          "next" : "aeiou",
          "current" : "aeiou",
          "previous" : "aeiou"
        },
        "total_pages" : 123,
        "current_page" : 123
      }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A product video [ProductVideoResponse](#ProductVideoResponse)

#### 404

The resource was not found. [NotFound](#NotFound)</div>

* * *

## <div class="method"><a name="createVariant"></a> post /catalog/products/{product_id}/variants

</div>

<div class="method-summary">(<span class="nickname">createVariant</span>)</div>

<div class="method-notes">Creates a `Variant` object.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Request body

<div class="field-items">

</p>
<div class="param">Variant [Variant](#Variant) (required)</div>

Body Parameter — A `Variant` object.

</div>

### Return type

[VariantResponse](#VariantResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "image_url" : "aeiou",
        "option_values" : [ {
          "option_display_name" : "aeiou",
          "option_id" : 123,
          "id" : 123,
          "label" : "aeiou"
        } ],
        "weight" : 1.3579000000000001069366817318950779736042022705078125,
        "upc" : "aeiou",
        "bin_picking_number" : "aeiou",
        "sku_id" : 123,
        "purchasing_disabled_message" : "aeiou",
        "inventory_level" : 123,
        "price" : 1.3579000000000001069366817318950779736042022705078125,
        "inventory_warning_level" : 123,
        "product_id" : 123,
        "id" : 123,
        "purchasing_disabled" : true,
        "sku" : "aeiou",
        "cost_price" : 1.3579000000000001069366817318950779736042022705078125
      },
      "meta" : { }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A variant and metadata. [VariantResponse](#VariantResponse)

#### 404

The resource was not found. [NotFound](#NotFound)</div>

* * *

## <div class="method"><a name="createVariantImage"></a> post /catalog/products/{product_id}/variants/{variant_id}/image

</div>

<div class="method-summary">(<span class="nickname">createVariantImage</span>)</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the resource belongs. format: int

</p>
<div class="param">variant_id (required)</div>

Path Parameter — The ID of the `Variant`. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `multipart/form-data`

### Form parameters

<div class="field-items">

</p>
<div class="param">image_file (required)</div>

Form Parameter — An image file. Supported MIME types include GIF, JPEG, and PNG.

</div>

### Return type

[ImageResponse](#ImageResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "image_url" : "aeiou"
      },
      "meta" : { }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A ResourceImage and metadata. [ImageResponse](#ImageResponse)

#### 404

The resource was not found. [NotFound](#NotFound)

#### 422

Image was not valid. This is the result of a missing image_file field or an incorrect file type. See the response for more details. [ErrorResponse](#ErrorResponse)</div>

* * *

## <div class="method"><a name="createVariantMetafield"></a> post /catalog/products/{product_id}/variants/{variant_id}/metafields

</div>

<div class="method-summary">(<span class="nickname">createVariantMetafield</span>)</div>

<div class="method-notes">Creates a variant `Metafield`.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the resource belongs. format: int

</p>
<div class="param">variant_id (required)</div>

Path Parameter — The ID of the `Variant` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Request body

<div class="field-items">

</p>
<div class="param">Metafield [Metafield](#Metafield) (required)</div>

Body Parameter — A `Metafield` object.

</div>

### Return type

[MetafieldResponse](#MetafieldResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "updated_at" : "2000-01-23T04:56:07.000+00:00",
        "namespace" : "aeiou",
        "resource_type" : "aeiou",
        "description" : "aeiou",
        "resource_id" : 123,
        "created_at" : "2000-01-23T04:56:07.000+00:00",
        "id" : 123,
        "value" : "aeiou",
        "key" : "aeiou",
        "permission_set" : "aeiou"
      },
      "meta" : { }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A `Metafield` object. [MetafieldResponse](#MetafieldResponse)

#### 409

The `Metafield` was in conflict with another `Metafield`. This can be the result of duplicate unique-key combinations of the app's client id, namespace, key, resource_type, and resource_id. [ErrorResponse](#ErrorResponse)

#### 422

The `Metafield` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details. [ErrorResponse](#ErrorResponse)</div>

* * *

## <div class="method"><a name="deleteBrandById"></a> delete /catalog/brands/{brand_id}

</div>

<div class="method-summary">(<span class="nickname">deleteBrandById</span>)</div>

<div class="method-notes">Deletes a `Brand` from the BigCommerce Catalog.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">brand_id (required)</div>

Path Parameter — The ID of the `Brand` requested. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 204

An empty response.[](#)</div>

* * *

## <div class="method"><a name="deleteBrandImage"></a> delete /catalog/brands/{brand_id}/image

</div>

<div class="method-summary">(<span class="nickname">deleteBrandImage</span>)</div>

<div class="method-notes">Deletes a `Brand` image from the BigCommerce Catalog.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">brand_id (required)</div>

Path Parameter — The ID of the `Brand` to which the image is being attached.

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 204

Image cleared from the brand.[](#)</div>

* * *

## <div class="method"><a name="deleteBrandMetafieldById"></a> delete /catalog/brands/{brand_id}/metafields/{metafield_id}

</div>

<div class="method-summary">(<span class="nickname">deleteBrandMetafieldById</span>)</div>

<div class="method-notes">Delete a `Metafield`</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">metafield_id (required)</div>

Path Parameter — The ID of the `Metafield`. format: int

</p>
<div class="param">brand_id (required)</div>

Path Parameter — The ID of the `Brand` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 204

An empty response.[](#)</div>

* * *

## <div class="method"><a name="deleteBrands"></a> delete /catalog/brands

</div>

<div class="method-summary">(<span class="nickname">deleteBrands</span>)</div>

<div class="method-notes">Deletes one or more `Brand` objects from the BigCommerce Catalog.</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Query parameters

<div class="field-items">

</p>
<div class="param">name (optional)</div>

Query Parameter — Filter items by name.

</p>
<div class="param">page_title (optional)</div>

Query Parameter — Filter items by page_title.

</div>

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 204

An empty response.[](#)</div>

* * *

## <div class="method"><a name="deleteCategories"></a> delete /catalog/categories

</div>

<div class="method-summary">(<span class="nickname">deleteCategories</span>)</div>

<div class="method-notes">Deletes a product or products from the BigCommerce Catalog.</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Query parameters

<div class="field-items">

</p>
<div class="param">name (optional)</div>

Query Parameter — Filter items by name.

</p>
<div class="param">parent_id (optional)</div>

Query Parameter — Filter items by parent_id.

</p>
<div class="param">page_title (optional)</div>

Query Parameter — Filter items by page_title.

</p>
<div class="param">keyword (optional)</div>

Query Parameter — Filter items by keywords.

</p>
<div class="param">is_visible (optional)</div>

Query Parameter — Filter items by is_visible.

</div>

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 204

An empty response.[](#)</div>

* * *

## <div class="method"><a name="deleteCategoryById"></a> delete /catalog/categories/{category_id}

</div>

<div class="method-summary">(<span class="nickname">deleteCategoryById</span>)</div>

<div class="method-notes">Deletes one or more `Category` objects from the BigCommerce catalog.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">category_id (required)</div>

Path Parameter — The ID of the `Category` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 204

An empty response.[](#)</div>

* * *

## <div class="method"><a name="deleteCategoryImage"></a> delete /catalog/categories/{category_id}/image

</div>

<div class="method-summary">(<span class="nickname">deleteCategoryImage</span>)</div>

<div class="method-notes">Deletes a `Category` image from the BigCommerce Catalog.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">category_id (required)</div>

Path Parameter — The ID of the `Category` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 204

An empty response.[](#)</div>

* * *

## <div class="method"><a name="deleteCategoryMetafieldById"></a> delete /catalog/categories/{category_id}/metafields/{metafield_id}

</div>

<div class="method-summary">(<span class="nickname">deleteCategoryMetafieldById</span>)</div>

<div class="method-notes">Delete a `Metafield`</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">metafield_id (required)</div>

Path Parameter — The ID of the `Metafield`. format: int

</p>
<div class="param">category_id (required)</div>

Path Parameter — The ID of the `Category` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 204

An empty response.[](#)</div>

* * *

## <div class="method"><a name="deleteComplexRuleById"></a> delete /catalog/products/{product_id}/complex-rules/{complex_rule_id}

</div>

<div class="method-summary">(<span class="nickname">deleteComplexRuleById</span>)</div>

<div class="method-notes">Deletes a Product's `ComplexRule`, based on the `product_id` and `complex_rule_id`.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the `ComplexRule` belongs. format: int

</p>
<div class="param">complex_rule_id (required)</div>

Path Parameter — The ID of the `ComplexRule`.

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 204

An empty response.[](#)</div>

* * *

## <div class="method"><a name="deleteModifierById"></a> delete /catalog/products/{product_id}/modifiers/{modifier_id}

</div>

<div class="method-summary">(<span class="nickname">deleteModifierById</span>)</div>

<div class="method-notes">Delete a Product's `Modifier` based on the product_id and modifier_id.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the `Modifier` belongs. format: int

</p>
<div class="param">modifier_id (required)</div>

Path Parameter — The ID of the `Modifier`.

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 204

An empty response.[](#)</div>

* * *

## <div class="method"><a name="deleteModifierImage"></a> delete /catalog/products/{product_id}/modifiers/{modifier_id}/values/{value_id}/image

</div>

<div class="method-summary">(<span class="nickname">deleteModifierImage</span>)</div>

<div class="method-notes">Deletes the image assigned to show when the modifier value is selected.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the `Modifier` belongs. format: int

</p>
<div class="param">modifier_id (required)</div>

Path Parameter — The ID of the `Modifier`.

</p>
<div class="param">value_id (required)</div>

Path Parameter — The ID of the `Modifier`.

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 204

Image cleared for this modifier value.[](#)</div>

* * *

## <div class="method"><a name="deleteOptionById"></a> delete /catalog/products/{product_id}/options/{option_id}

</div>

<div class="method-summary">(<span class="nickname">deleteOptionById</span>)</div>

<div class="method-notes">Delete a Product's `Option`, based on the product_id and option_id.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the `Option` belongs. format: int

</p>
<div class="param">option_id (required)</div>

Path Parameter — The ID of the `Option`.

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 204

An empty response.[](#)</div>

* * *

## <div class="method"><a name="deleteProductById"></a> delete /catalog/products/{product_id}

</div>

<div class="method-summary">(<span class="nickname">deleteProductById</span>)</div>

<div class="method-notes">Deletes a `Product` object from the BigCommerce Catalog</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 204

An empty response.[](#)</div>

* * *

## <div class="method"><a name="deleteProductImage"></a> delete /catalog/products/{product_id}/images/{image_id}

</div>

<div class="method-summary">(<span class="nickname">deleteProductImage</span>)</div>

<div class="method-notes">Deletes a `ProductImage` in the BigCommerce Catalog.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the image is being attached.

</p>
<div class="param">image_id (required)</div>

Path Parameter — The ID of the `Image` that is being operated on.

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 204

An empty response.[](#)</div>

* * *

## <div class="method"><a name="deleteProductMetafieldById"></a> delete /catalog/products/{product_id}/metafields/{metafield_id}

</div>

<div class="method-summary">(<span class="nickname">deleteProductMetafieldById</span>)</div>

<div class="method-notes">Deletes a `Metafield`.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">metafield_id (required)</div>

Path Parameter — The ID of the `Metafield`. format: int

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 204

An empty response.[](#)</div>

* * *

## <div class="method"><a name="deleteProductVideo"></a> delete /catalog/products/{product_id}/videos/{video_id}

</div>

<div class="method-summary">(<span class="nickname">deleteProductVideo</span>)</div>

<div class="method-notes">Deletes a `ProductVideo` in the BigCommerce Catalog.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the video is being attached.

</p>
<div class="param">video_id (required)</div>

Path Parameter — The ID of the `Video` being operated on.

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 204

An empty response[](#)</div>

* * *

## <div class="method"><a name="deleteProducts"></a> delete /catalog/products

</div>

<div class="method-summary">(<span class="nickname">deleteProducts</span>)</div>

<div class="method-notes">Deletes one or more `Product` objects from the BigCommerce Catalog</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Query parameters

<div class="field-items">

</p>
<div class="param">name (optional)</div>

Query Parameter — Filter items by name.

</p>
<div class="param">sku (optional)</div>

Query Parameter — Filter items by sku.

</p>
<div class="param">price (optional)</div>

Query Parameter — Filter items by price.

</p>
<div class="param">weight (optional)</div>

Query Parameter — Filter items by weight.

</p>
<div class="param">condition (optional)</div>

Query Parameter — Filter items by condition.

</p>
<div class="param">brand_id (optional)</div>

Query Parameter — Filter items by brand_id.

</p>
<div class="param">date_modified (optional)</div>

Query Parameter — Filter items by date_modified. format: data-time

</p>
<div class="param">date_last_imported (optional)</div>

Query Parameter — Filter items by date_last_imported. format: data-time

</p>
<div class="param">is_visible (optional)</div>

Query Parameter — Filter items by is_visible.

</p>
<div class="param">is_featured (optional)</div>

Query Parameter — Filter items by is_featured.

</p>
<div class="param">inventory_level (optional)</div>

Query Parameter — Filter items by inventory_level.

</p>
<div class="param">total_sold (optional)</div>

Query Parameter — Filter items by total_sold.

</p>
<div class="param">type (optional)</div>

Query Parameter — Filter items by type: `physical` or `digital`.

</p>
<div class="param">categories (optional)</div>

Query Parameter — Filter items by categories.

</p>
<div class="param">keyword (optional)</div>

Query Parameter — Filter items by keywords found in the name, description, sku, keywords, or brand name.

</div>

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 204

An empty response.[](#)</div>

* * *

## <div class="method"><a name="deleteVariantById"></a> delete /catalog/products/{product_id}/variants/{variant_id}

</div>

<div class="method-summary">(<span class="nickname">deleteVariantById</span>)</div>

<div class="method-notes">Deletes a `Variant`.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the resource belongs. format: int

</p>
<div class="param">variant_id (required)</div>

Path Parameter — The ID of the `Variant` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 204

An empty response.[](#)</div>

* * *

## <div class="method"><a name="deleteVariantMetafieldById"></a> delete /catalog/products/{product_id}/variants/{variant_id}/metafields/{metafield_id}

</div>

<div class="method-summary">(<span class="nickname">deleteVariantMetafieldById</span>)</div>

<div class="method-notes">Delete a `Metafield`</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">metafield_id (required)</div>

Path Parameter — The ID of the `Metafield`. format: int

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the resource belongs. format: int

</p>
<div class="param">variant_id (required)</div>

Path Parameter — The ID of the `Variant` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 204

An empty response.[](#)</div>

* * *

## <div class="method"><a name="getBrandById"></a> get /catalog/brands/{brand_id}

</div>

<div class="method-summary">(<span class="nickname">getBrandById</span>)</div>

<div class="method-notes">Gets a `Brand` object.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">brand_id (required)</div>

Path Parameter — The ID of the `Brand` requested. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Return type

[BrandResponse](#BrandResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "meta_description" : "aeiou",
        "page_title" : "aeiou",
        "image_url" : "aeiou",
        "name" : "aeiou",
        "id" : 123,
        "meta_keywords" : [ "aeiou" ],
        "search_keywords" : "aeiou"
      },
      "meta" : { }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A `Brand` object. [BrandResponse](#BrandResponse)

#### 404

The resource was not found. [NotFound](#NotFound)</div>

* * *

## <div class="method"><a name="getBrandMetafieldByBrandId"></a> get /catalog/brands/{brand_id}/metafields/{metafield_id}

</div>

<div class="method-summary">(<span class="nickname">getBrandMetafieldByBrandId</span>)</div>

<div class="method-notes">Gets a `Metafield`, by `category_id`.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">metafield_id (required)</div>

Path Parameter — The ID of the `Metafield`. format: int

</p>
<div class="param">brand_id (required)</div>

Path Parameter — The ID of the `Brand` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Return type

[Metafield](#Metafield)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "updated_at" : "2000-01-23T04:56:07.000+00:00",
      "namespace" : "aeiou",
      "resource_type" : "aeiou",
      "description" : "aeiou",
      "resource_id" : 123,
      "created_at" : "2000-01-23T04:56:07.000+00:00",
      "id" : 123,
      "value" : "aeiou",
      "key" : "aeiou",
      "permission_set" : "aeiou"
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A `Metafield` object. [Metafield](#Metafield)

#### 404

The resource was not found. [NotFound](#NotFound)</div>

* * *

## <div class="method"><a name="getBrandMetafieldsByBrandId"></a> get /catalog/brands/{brand_id}/metafields

</div>

<div class="method-summary">(<span class="nickname">getBrandMetafieldsByBrandId</span>)</div>

<div class="method-notes">Gets a `Metafield` object list, by `category_id`.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">brand_id (required)</div>

Path Parameter — The ID of the `Brand` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Query parameters

<div class="field-items">

</p>
<div class="param">key (optional)</div>

Query Parameter — Filter based on a metafield's key.

</p>
<div class="param">namespace (optional)</div>

Query Parameter — Filter based on a metafield's key.

</p>
<div class="param">page (optional)</div>

Query Parameter — Control the page in a limited list of products.

</p>
<div class="param">limit (optional)</div>

Query Parameter — Control the items per page.

</p>
<div class="param">key (optional)</div>

Query Parameter — Filter based on a metafield's key.

</p>
<div class="param">namespace (optional)</div>

Query Parameter — Filter based on a metafield's key.

</div>

### Return type

[MetaFieldCollectionResponse](#MetaFieldCollectionResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : [ {
        "updated_at" : "2000-01-23T04:56:07.000+00:00",
        "namespace" : "aeiou",
        "resource_type" : "aeiou",
        "description" : "aeiou",
        "resource_id" : 123,
        "created_at" : "2000-01-23T04:56:07.000+00:00",
        "id" : 123,
        "value" : "aeiou",
        "key" : "aeiou",
        "permission_set" : "aeiou"
      } ],
      "meta" : {
        "per_page" : 123,
        "total" : 123,
        "count" : 123,
        "links" : {
          "next" : "aeiou",
          "current" : "aeiou",
          "previous" : "aeiou"
        },
        "total_pages" : 123,
        "current_page" : 123
      }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

An array of metafields and metadata. [MetaFieldCollectionResponse](#MetaFieldCollectionResponse)

#### 404

The resource was not found. [NotFound](#NotFound)</div>

* * *

## <div class="method"><a name="getBrands"></a> get /catalog/brands

</div>

<div class="method-summary">(<span class="nickname">getBrands</span>)</div>

<div class="method-notes">Gets `Brand` objects.</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Query parameters

<div class="field-items">

</p>
<div class="param">name (optional)</div>

Query Parameter — Filter items by name.

</p>
<div class="param">page_title (optional)</div>

Query Parameter — Filter items by page_title.

</p>
<div class="param">page (optional)</div>

Query Parameter — Control the page in a limited list of products.

</p>
<div class="param">limit (optional)</div>

Query Parameter — Control the items per page.

</div>

### Return type

[BrandCollectionResponse](#BrandCollectionResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : [ {
        "meta_description" : "aeiou",
        "page_title" : "aeiou",
        "image_url" : "aeiou",
        "name" : "aeiou",
        "id" : 123,
        "meta_keywords" : [ "aeiou" ],
        "search_keywords" : "aeiou"
      } ],
      "meta" : {
        "per_page" : 123,
        "total" : 123,
        "count" : 123,
        "links" : {
          "next" : "aeiou",
          "current" : "aeiou",
          "previous" : "aeiou"
        },
        "total_pages" : 123,
        "current_page" : 123
      }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

An array of brand objects and metadata. [BrandCollectionResponse](#BrandCollectionResponse)</div>

* * *

## <div class="method"><a name="getCategories"></a> get /catalog/categories

</div>

<div class="method-summary">(<span class="nickname">getCategories</span>)</div>

<div class="method-notes">Returns a paginated categories collection from the BigCommerce Catalog.</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Query parameters

<div class="field-items">

</p>
<div class="param">name (optional)</div>

Query Parameter — Filter items by name.

</p>
<div class="param">parent_id (optional)</div>

Query Parameter — Filter items by parent_id.

</p>
<div class="param">page_title (optional)</div>

Query Parameter — Filter items by page_title.

</p>
<div class="param">keyword (optional)</div>

Query Parameter — Filter items by keywords.

</p>
<div class="param">is_visible (optional)</div>

Query Parameter — Filter items by is_visible.

</p>
<div class="param">page (optional)</div>

Query Parameter — Control the page in a limited list of products.

</p>
<div class="param">limit (optional)</div>

Query Parameter — Control the items per page.

</div>

### Return type

[CategoryCollectionResponse](#CategoryCollectionResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : [ {
        "is_visible" : true,
        "page_title" : "aeiou",
        "image_url" : "aeiou",
        "description" : "aeiou",
        "meta_keywords" : [ "aeiou" ],
        "search_keywords" : "aeiou",
        "default_product_sort" : "aeiou",
        "meta_description" : "aeiou",
        "layout_file" : "aeiou",
        "parent_id" : 123,
        "name" : "aeiou",
        "id" : 123,
        "sort_order" : 123,
        "views" : 123,
        "custom_url" : {
          "is_customized" : true,
          "url" : "aeiou"
        }
      } ],
      "meta" : {
        "per_page" : 123,
        "total" : 123,
        "count" : 123,
        "links" : {
          "next" : "aeiou",
          "current" : "aeiou",
          "previous" : "aeiou"
        },
        "total_pages" : 123,
        "current_page" : 123
      }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

An array of category objects and metadata. [CategoryCollectionResponse](#CategoryCollectionResponse)</div>

* * *

## <div class="method"><a name="getCategoryById"></a> get /catalog/categories/{category_id}

</div>

<div class="method-summary">(<span class="nickname">getCategoryById</span>)</div>

<div class="method-notes">Returns a `Category` from the BigCommerce Catalog.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">category_id (required)</div>

Path Parameter — The ID of the `Category` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Return type

[CategoryResponse](#CategoryResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "is_visible" : true,
        "page_title" : "aeiou",
        "image_url" : "aeiou",
        "description" : "aeiou",
        "meta_keywords" : [ "aeiou" ],
        "search_keywords" : "aeiou",
        "default_product_sort" : "aeiou",
        "meta_description" : "aeiou",
        "layout_file" : "aeiou",
        "parent_id" : 123,
        "name" : "aeiou",
        "id" : 123,
        "sort_order" : 123,
        "views" : 123,
        "custom_url" : {
          "is_customized" : true,
          "url" : "aeiou"
        }
      },
      "meta" : { }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A category object. [CategoryResponse](#CategoryResponse)

#### 404

The resource was not found. [NotFound](#NotFound)</div>

* * *

## <div class="method"><a name="getCategoryMetafieldByCategoryId"></a> get /catalog/categories/{category_id}/metafields/{metafield_id}

</div>

<div class="method-summary">(<span class="nickname">getCategoryMetafieldByCategoryId</span>)</div>

<div class="method-notes">Gets a `Metafield` by category_id.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">metafield_id (required)</div>

Path Parameter — The ID of the `Metafield`. format: int

</p>
<div class="param">category_id (required)</div>

Path Parameter — The ID of the `Category` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Return type

[Metafield](#Metafield)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "updated_at" : "2000-01-23T04:56:07.000+00:00",
      "namespace" : "aeiou",
      "resource_type" : "aeiou",
      "description" : "aeiou",
      "resource_id" : 123,
      "created_at" : "2000-01-23T04:56:07.000+00:00",
      "id" : 123,
      "value" : "aeiou",
      "key" : "aeiou",
      "permission_set" : "aeiou"
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A metafield object, [Metafield](#Metafield)

#### 404

The resource was not found. [NotFound](#NotFound)</div>

* * *

## <div class="method"><a name="getCategoryMetafieldsByCategoryId"></a> get /catalog/categories/{category_id}/metafields

</div>

<div class="method-summary">(<span class="nickname">getCategoryMetafieldsByCategoryId</span>)</div>

<div class="method-notes">Gets a `Metafield` object list, by category_id.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">category_id (required)</div>

Path Parameter — The ID of the `Category` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Query parameters

<div class="field-items">

</p>
<div class="param">page (optional)</div>

Query Parameter — Control the page in a limited list of products.

</p>
<div class="param">limit (optional)</div>

Query Parameter — Control the items per page.

</p>
<div class="param">key (optional)</div>

Query Parameter — Filter based on a metafield's key.

</p>
<div class="param">namespace (optional)</div>

Query Parameter — Filter based on a metafield's key.

</div>

### Return type

[MetaFieldCollectionResponse](#MetaFieldCollectionResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : [ {
        "updated_at" : "2000-01-23T04:56:07.000+00:00",
        "namespace" : "aeiou",
        "resource_type" : "aeiou",
        "description" : "aeiou",
        "resource_id" : 123,
        "created_at" : "2000-01-23T04:56:07.000+00:00",
        "id" : 123,
        "value" : "aeiou",
        "key" : "aeiou",
        "permission_set" : "aeiou"
      } ],
      "meta" : {
        "per_page" : 123,
        "total" : 123,
        "count" : 123,
        "links" : {
          "next" : "aeiou",
          "current" : "aeiou",
          "previous" : "aeiou"
        },
        "total_pages" : 123,
        "current_page" : 123
      }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

An array of metafields and metadata. [MetaFieldCollectionResponse](#MetaFieldCollectionResponse)

#### 404

The resource was not found. [NotFound](#NotFound)</div>

* * *

## <div class="method"><a name="getCategoryTree"></a> get /catalog/categories/tree

</div>

<div class="method-summary">(<span class="nickname">getCategoryTree</span>)</div>

<div class="method-notes">Returns the categories tree, a nested lineage of the categories with parent->child relationship. The `Category` objects returned are simplified versions of the category objects returned in the rest of this API.</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Return type

[CategoryTreeCollectionResponse](#CategoryTreeCollectionResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : [ {
        "is_visible" : true,
        "children" : [ "" ],
        "parent_id" : 123,
        "name" : "aeiou",
        "id" : 123,
        "url" : "aeiou"
      } ],
      "meta" : {
        "per_page" : 123,
        "total" : 123,
        "count" : 123,
        "links" : {
          "next" : "aeiou",
          "current" : "aeiou",
          "previous" : "aeiou"
        },
        "total_pages" : 123,
        "current_page" : 123
      }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A array of nested category tree objects and metadata. [CategoryTreeCollectionResponse](#CategoryTreeCollectionResponse)</div>

* * *

## <div class="method"><a name="getComplexRuleById"></a> get /catalog/products/{product_id}/complex-rules/{complex_rule_id}

</div>

<div class="method-summary">(<span class="nickname">getComplexRuleById</span>)</div>

<div class="method-notes">Get a `ComplexRule` by product_id</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the `ComplexRule` belongs. format: int

</p>
<div class="param">complex_rule_id (required)</div>

Path Parameter — The ID of the `ComplexRule`.

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Return type

[ComplexRuleResponse](#ComplexRuleResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "purchasing_hidden" : true,
        "stop" : true,
        "price_adjuster" : {
          "adjuster" : "aeiou",
          "adjuster_value" : 1.3579000000000001069366817318950779736042022705078125
        },
        "image_url" : "aeiou",
        "product_id" : 123,
        "weight_adjuster" : "",
        "id" : 123,
        "purchasing_disabled" : true,
        "purchasing_disabled_message" : "aeiou",
        "conditions" : [ {
          "rule_id" : 123,
          "variant_id" : 123,
          "combination_id" : 123,
          "modifier_id" : 123,
          "modifier_value_id" : 123,
          "id" : 123
        } ],
        "sort_order" : 123,
        "enabled" : true
      },
      "meta" : {
        "per_page" : 123,
        "total" : 123,
        "count" : 123,
        "links" : {
          "next" : "aeiou",
          "current" : "aeiou",
          "previous" : "aeiou"
        },
        "total_pages" : 123,
        "current_page" : 123
      }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A modifier object [ComplexRuleResponse](#ComplexRuleResponse)

#### 404

The resource was not found. [NotFound](#NotFound)</div>

* * *

## <div class="method"><a name="getComplexRules"></a> get /catalog/products/{product_id}/complex-rules

</div>

<div class="method-summary">(<span class="nickname">getComplexRules</span>)</div>

<div class="method-notes">Get an array of `ComplexRule` objects.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the `ComplexRule` belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Return type

[ComplexRuleCollectionResponse](#ComplexRuleCollectionResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : [ {
        "purchasing_hidden" : true,
        "stop" : true,
        "price_adjuster" : {
          "adjuster" : "aeiou",
          "adjuster_value" : 1.3579000000000001069366817318950779736042022705078125
        },
        "image_url" : "aeiou",
        "product_id" : 123,
        "weight_adjuster" : "",
        "id" : 123,
        "purchasing_disabled" : true,
        "purchasing_disabled_message" : "aeiou",
        "conditions" : [ {
          "rule_id" : 123,
          "variant_id" : 123,
          "combination_id" : 123,
          "modifier_id" : 123,
          "modifier_value_id" : 123,
          "id" : 123
        } ],
        "sort_order" : 123,
        "enabled" : true
      } ],
      "meta" : {
        "per_page" : 123,
        "total" : 123,
        "count" : 123,
        "links" : {
          "next" : "aeiou",
          "current" : "aeiou",
          "previous" : "aeiou"
        },
        "total_pages" : 123,
        "current_page" : 123
      }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

An array of `ComplexRule` objects and metadata. [ComplexRuleCollectionResponse](#ComplexRuleCollectionResponse)</div>

* * *

## <div class="method"><a name="getModifierById"></a> get /catalog/products/{product_id}/modifiers/{modifier_id}

</div>

<div class="method-summary">(<span class="nickname">getModifierById</span>)</div>

<div class="method-notes">Get a `Modifier` by product_id and modifier_id</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the `Modifier` belongs. format: int

</p>
<div class="param">modifier_id (required)</div>

Path Parameter — The ID of the `Modifier`.

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Return type

[ModifierResponse](#ModifierResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "product_id" : 123,
        "option_values" : [ "" ],
        "name" : "aeiou",
        "id" : 123,
        "display_name" : "aeiou",
        "type" : "aeiou",
        "config" : {
          "text_max_length" : 123,
          "file_types_other" : [ "aeiou" ],
          "file_max_size" : 123,
          "file_types_supported" : [ "aeiou" ],
          "text_characters_limited" : true,
          "product_list_adjusts_inventory" : true,
          "number_limited" : true,
          "checked_by_default" : true,
          "date_latest_value" : "2000-01-23",
          "product_list_adjusts_pricing" : true,
          "default_value" : "aeiou",
          "date_limited" : true,
          "text_max_lines" : 123,
          "checkbox_label" : "aeiou",
          "text_min_length" : 123,
          "file_types_mode" : "aeiou",
          "text_lines_limited" : true,
          "number_highest_value" : 1.3579000000000001069366817318950779736042022705078125,
          "date_earliest_value" : "2000-01-23",
          "date_limit_mode" : "aeiou",
          "number_lowest_value" : 1.3579000000000001069366817318950779736042022705078125,
          "number_integers_only" : true,
          "product_list_shipping_calc" : "aeiou",
          "number_limit_mode" : "aeiou"
        },
        "required" : true
      },
      "meta" : {
        "per_page" : 123,
        "total" : 123,
        "count" : 123,
        "links" : {
          "next" : "aeiou",
          "current" : "aeiou",
          "previous" : "aeiou"
        },
        "total_pages" : 123,
        "current_page" : 123
      }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A `Modifier` object. [ModifierResponse](#ModifierResponse)

#### 404

The resource was not found. [NotFound](#NotFound)</div>

* * *

## <div class="method"><a name="getModifiers"></a> get /catalog/products/{product_id}/modifiers

</div>

<div class="method-summary">(<span class="nickname">getModifiers</span>)</div>

<div class="method-notes">Gets an array of `Modifier` objects.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the `Modifier` belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Return type

[ModifierCollectionResponse](#ModifierCollectionResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : [ {
        "product_id" : 123,
        "option_values" : [ "" ],
        "name" : "aeiou",
        "id" : 123,
        "display_name" : "aeiou",
        "type" : "aeiou",
        "config" : {
          "text_max_length" : 123,
          "file_types_other" : [ "aeiou" ],
          "file_max_size" : 123,
          "file_types_supported" : [ "aeiou" ],
          "text_characters_limited" : true,
          "product_list_adjusts_inventory" : true,
          "number_limited" : true,
          "checked_by_default" : true,
          "date_latest_value" : "2000-01-23",
          "product_list_adjusts_pricing" : true,
          "default_value" : "aeiou",
          "date_limited" : true,
          "text_max_lines" : 123,
          "checkbox_label" : "aeiou",
          "text_min_length" : 123,
          "file_types_mode" : "aeiou",
          "text_lines_limited" : true,
          "number_highest_value" : 1.3579000000000001069366817318950779736042022705078125,
          "date_earliest_value" : "2000-01-23",
          "date_limit_mode" : "aeiou",
          "number_lowest_value" : 1.3579000000000001069366817318950779736042022705078125,
          "number_integers_only" : true,
          "product_list_shipping_calc" : "aeiou",
          "number_limit_mode" : "aeiou"
        },
        "required" : true
      } ],
      "meta" : {
        "per_page" : 123,
        "total" : 123,
        "count" : 123,
        "links" : {
          "next" : "aeiou",
          "current" : "aeiou",
          "previous" : "aeiou"
        },
        "total_pages" : 123,
        "current_page" : 123
      }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

An array of modifiers and metadata. [ModifierCollectionResponse](#ModifierCollectionResponse)</div>

* * *

## <div class="method"><a name="getOptionById"></a> get /catalog/products/{product_id}/options/{option_id}

</div>

<div class="method-summary">(<span class="nickname">getOptionById</span>)</div>

<div class="method-notes">Gets `Option` object, by product id and option id.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the `Option` belongs. format: int

</p>
<div class="param">option_id (required)</div>

Path Parameter — The ID of the `Option`.

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Return type

[OptionResponse](#OptionResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "product_id" : 123,
        "option_values" : [ {
          "id" : 123,
          "label" : "aeiou",
          "value_data" : "{}",
          "is_default" : true,
          "sort_order" : 123
        } ],
        "name" : "aeiou",
        "id" : 123,
        "display_name" : "aeiou",
        "type" : "aeiou",
        "config" : {
          "text_max_length" : 123,
          "file_types_other" : [ "aeiou" ],
          "file_max_size" : 123,
          "file_types_supported" : [ "aeiou" ],
          "text_characters_limited" : true,
          "product_list_adjusts_inventory" : true,
          "number_limited" : true,
          "checked_by_default" : true,
          "date_latest_value" : "2000-01-23",
          "product_list_adjusts_pricing" : true,
          "default_value" : "aeiou",
          "date_limited" : true,
          "text_max_lines" : 123,
          "checkbox_label" : "aeiou",
          "text_min_length" : 123,
          "file_types_mode" : "aeiou",
          "text_lines_limited" : true,
          "number_highest_value" : 1.3579000000000001069366817318950779736042022705078125,
          "date_earliest_value" : "2000-01-23",
          "date_limit_mode" : "aeiou",
          "number_lowest_value" : 1.3579000000000001069366817318950779736042022705078125,
          "number_integers_only" : true,
          "product_list_shipping_calc" : "aeiou",
          "number_limit_mode" : "aeiou"
        }
      },
      "meta" : {
        "per_page" : 123,
        "total" : 123,
        "count" : 123,
        "links" : {
          "next" : "aeiou",
          "current" : "aeiou",
          "previous" : "aeiou"
        },
        "total_pages" : 123,
        "current_page" : 123
      }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

An `Option` object. [OptionResponse](#OptionResponse)

#### 404

The resource was not found. [NotFound](#NotFound)</div>

* * *

## <div class="method"><a name="getOptions"></a> get /catalog/products/{product_id}/options

</div>

<div class="method-summary">(<span class="nickname">getOptions</span>)</div>

<div class="method-notes">Gets an array of `Option` objects.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Return type

[OptionCollectionResponse](#OptionCollectionResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : [ {
        "product_id" : 123,
        "option_values" : [ {
          "id" : 123,
          "label" : "aeiou",
          "value_data" : "{}",
          "is_default" : true,
          "sort_order" : 123
        } ],
        "name" : "aeiou",
        "id" : 123,
        "display_name" : "aeiou",
        "type" : "aeiou",
        "config" : {
          "text_max_length" : 123,
          "file_types_other" : [ "aeiou" ],
          "file_max_size" : 123,
          "file_types_supported" : [ "aeiou" ],
          "text_characters_limited" : true,
          "product_list_adjusts_inventory" : true,
          "number_limited" : true,
          "checked_by_default" : true,
          "date_latest_value" : "2000-01-23",
          "product_list_adjusts_pricing" : true,
          "default_value" : "aeiou",
          "date_limited" : true,
          "text_max_lines" : 123,
          "checkbox_label" : "aeiou",
          "text_min_length" : 123,
          "file_types_mode" : "aeiou",
          "text_lines_limited" : true,
          "number_highest_value" : 1.3579000000000001069366817318950779736042022705078125,
          "date_earliest_value" : "2000-01-23",
          "date_limit_mode" : "aeiou",
          "number_lowest_value" : 1.3579000000000001069366817318950779736042022705078125,
          "number_integers_only" : true,
          "product_list_shipping_calc" : "aeiou",
          "number_limit_mode" : "aeiou"
        }
      } ],
      "meta" : {
        "per_page" : 123,
        "total" : 123,
        "count" : 123,
        "links" : {
          "next" : "aeiou",
          "current" : "aeiou",
          "previous" : "aeiou"
        },
        "total_pages" : 123,
        "current_page" : 123
      }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

An array of options and metadata. [OptionCollectionResponse](#OptionCollectionResponse)

#### 404

The resource was not found. [NotFound](#NotFound)</div>

* * *

## <div class="method"><a name="getProductById"></a> get /catalog/products/{product_id}

</div>

<div class="method-summary">(<span class="nickname">getProductById</span>)</div>

<div class="method-notes">Returns a `Product` from the BigCommerce Catalog.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Query parameters

<div class="field-items">

</p>
<div class="param">include (optional)</div>

Query Parameter — Include sub-resources on a product, with a comma-separated list. Valid expansions currently include `variants`, `images`, `custom_fields`, and `bulk_pricing_rules`.

</div>

### Return type

[ProductResponse](#ProductResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "calculated_price" : 1.3579000000000001069366817318950779736042022705078125,
        "gift_wrapping_options_list" : [ 123 ],
        "page_title" : "aeiou",
        "videos" : [ {
          "product_id" : 123,
          "length" : "aeiou",
          "description" : "aeiou",
          "id" : 123,
          "title" : "aeiou",
          "sort_order" : 123
        } ],
        "is_condition_shown" : true,
        "variants" : [ {
          "image_url" : "aeiou",
          "option_values" : [ {
            "option_display_name" : "aeiou",
            "option_id" : 123,
            "id" : 123,
            "label" : "aeiou"
          } ],
          "weight" : 1.3579000000000001069366817318950779736042022705078125,
          "upc" : "aeiou",
          "bin_picking_number" : "aeiou",
          "sku_id" : 123,
          "purchasing_disabled_message" : "aeiou",
          "inventory_level" : 123,
          "price" : 1.3579000000000001069366817318950779736042022705078125,
          "inventory_warning_level" : 123,
          "product_id" : 123,
          "id" : 123,
          "purchasing_disabled" : true,
          "sku" : "aeiou",
          "cost_price" : 1.3579000000000001069366817318950779736042022705078125
        } ],
        "type" : "aeiou",
        "retail_price" : 1.3579000000000001069366817318950779736042022705078125,
        "layout_file" : "aeiou",
        "price" : 1.3579000000000001069366817318950779736042022705078125,
        "inventory_warning_level" : 123,
        "warranty" : "aeiou",
        "is_free_shipping" : true,
        "id" : 123,
        "sku" : "aeiou",
        "height" : 1.3579000000000001069366817318950779736042022705078125,
        "custom_url" : {
          "is_customized" : true,
          "url" : "aeiou"
        },
        "images" : [ "" ],
        "custom_fields" : [ {
          "product_id" : 123,
          "name" : "aeiou",
          "id" : 123,
          "value" : "aeiou"
        } ],
        "weight" : 1.3579000000000001069366817318950779736042022705078125,
        "upc" : "aeiou",
        "brand_id" : 123,
        "meta_description" : "aeiou",
        "condition" : "aeiou",
        "inventory_level" : 123,
        "name" : "aeiou",
        "inventory_tracking" : "aeiou",
        "preorder_release_date" : "aeiou",
        "description" : "aeiou",
        "bin_picking_number" : "aeiou",
        "availability" : "aeiou",
        "search_keywords" : "aeiou",
        "meta_keywords" : [ "aeiou" ],
        "is_price_hidden" : true,
        "order_quantity_minimum" : 123,
        "availability_description" : "aeiou",
        "fixed_cost_shipping_price" : 123,
        "categories" : [ 123 ],
        "sort_order" : 123,
        "cost_price" : 1.3579000000000001069366817318950779736042022705078125,
        "order_quantity_maximum" : 123,
        "is_visible" : true,
        "is_preorder_only" : true,
        "date_created" : "aeiou",
        "preorder_message" : "aeiou",
        "tax_class_id" : 123,
        "bulk_pricing_rules" : [ {
          "amount" : 1.3579000000000001069366817318950779736042022705078125,
          "quantity_min" : 123,
          "quantity_max" : 123,
          "id" : 123,
          "type" : "aeiou"
        } ],
        "sale_price" : 1.3579000000000001069366817318950779736042022705078125,
        "product_tax_code" : "aeiou",
        "depth" : 1.3579000000000001069366817318950779736042022705078125,
        "date_modified" : "aeiou",
        "gift_wrapping_options_type" : "aeiou",
        "width" : 1.3579000000000001069366817318950779736042022705078125,
        "price_hidden_label" : "aeiou",
        "is_featured" : true,
        "view_count" : 123
      },
      "meta" : { }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A product. [ProductResponse](#ProductResponse)

#### 404

The resource was not found. [NotFound](#NotFound)</div>

* * *

## <div class="method"><a name="getProductImageById"></a> get /catalog/products/{product_id}/images/{image_id}

</div>

<div class="method-summary">(<span class="nickname">getProductImageById</span>)</div>

<div class="method-notes">Gets image on a product.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the image is being attached.

</p>
<div class="param">image_id (required)</div>

Path Parameter — The ID of the `Image` that is being operated on.

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Return type

[ProductImageResponse](#ProductImageResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : "",
      "meta" : { }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

An array of product images and metadata. [ProductImageResponse](#ProductImageResponse)

#### 404

The resource was not found. [NotFound](#NotFound)</div>

* * *

## <div class="method"><a name="getProductImages"></a> get /catalog/products/{product_id}/images

</div>

<div class="method-summary">(<span class="nickname">getProductImages</span>)</div>

<div class="method-notes">Gets all images on a product.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the image is being attached.

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Return type

[ProductImageCollectionResponse](#ProductImageCollectionResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : [ "" ],
      "meta" : {
        "per_page" : 123,
        "total" : 123,
        "count" : 123,
        "links" : {
          "next" : "aeiou",
          "current" : "aeiou",
          "previous" : "aeiou"
        },
        "total_pages" : 123,
        "current_page" : 123
      }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

List of product images and metadata. [ProductImageCollectionResponse](#ProductImageCollectionResponse)

#### 204

There are not any images on this product.[](#)

#### 404

The product ID does not exist. [NotFound](#NotFound)</div>

* * *

## <div class="method"><a name="getProductMetafieldByProductId"></a> get /catalog/products/{product_id}/metafields/{metafield_id}

</div>

<div class="method-summary">(<span class="nickname">getProductMetafieldByProductId</span>)</div>

<div class="method-notes">Gets a `Metafield`, by `product_id`.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">metafield_id (required)</div>

Path Parameter — The ID of the `Metafield`. format: int

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Return type

[Metafield](#Metafield)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "updated_at" : "2000-01-23T04:56:07.000+00:00",
      "namespace" : "aeiou",
      "resource_type" : "aeiou",
      "description" : "aeiou",
      "resource_id" : 123,
      "created_at" : "2000-01-23T04:56:07.000+00:00",
      "id" : 123,
      "value" : "aeiou",
      "key" : "aeiou",
      "permission_set" : "aeiou"
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A metafield object. [Metafield](#Metafield)

#### 404

The resource was not found. [NotFound](#NotFound)</div>

* * *

## <div class="method"><a name="getProductMetafieldsByProductId"></a> get /catalog/products/{product_id}/metafields

</div>

<div class="method-summary">(<span class="nickname">getProductMetafieldsByProductId</span>)</div>

<div class="method-notes">Gets a `Metafield` object list, by `product_id`.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Query parameters

<div class="field-items">

</p>
<div class="param">page (optional)</div>

Query Parameter — Control the page in a limited list of products.

</p>
<div class="param">limit (optional)</div>

Query Parameter — Control the items per page.

</p>
<div class="param">key (optional)</div>

Query Parameter — Filter based on a metafield's key.

</p>
<div class="param">namespace (optional)</div>

Query Parameter — Filter based on a metafield's key.

</div>

### Return type

[MetaFieldCollectionResponse](#MetaFieldCollectionResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : [ {
        "updated_at" : "2000-01-23T04:56:07.000+00:00",
        "namespace" : "aeiou",
        "resource_type" : "aeiou",
        "description" : "aeiou",
        "resource_id" : 123,
        "created_at" : "2000-01-23T04:56:07.000+00:00",
        "id" : 123,
        "value" : "aeiou",
        "key" : "aeiou",
        "permission_set" : "aeiou"
      } ],
      "meta" : {
        "per_page" : 123,
        "total" : 123,
        "count" : 123,
        "links" : {
          "next" : "aeiou",
          "current" : "aeiou",
          "previous" : "aeiou"
        },
        "total_pages" : 123,
        "current_page" : 123
      }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

An array of metafields and metadata. [MetaFieldCollectionResponse](#MetaFieldCollectionResponse)

#### 404

The resource was not found. [NotFound](#NotFound)</div>

* * *

## <div class="method"><a name="getProductVideoById"></a> get /catalog/products/{product_id}/videos/{video_id}

</div>

<div class="method-summary">(<span class="nickname">getProductVideoById</span>)</div>

<div class="method-notes">Gets video on a product.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the video is being attached.

</p>
<div class="param">video_id (required)</div>

Path Parameter — The ID of the `Video` being operated on.

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Return type

[ProductVideoResponse](#ProductVideoResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "product_id" : 123,
        "length" : "aeiou",
        "description" : "aeiou",
        "id" : 123,
        "title" : "aeiou",
        "sort_order" : 123
      },
      "meta" : {
        "per_page" : 123,
        "total" : 123,
        "count" : 123,
        "links" : {
          "next" : "aeiou",
          "current" : "aeiou",
          "previous" : "aeiou"
        },
        "total_pages" : 123,
        "current_page" : 123
      }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

An array of product videos and metadata. [ProductVideoResponse](#ProductVideoResponse)

#### 404

The resource was not found. [NotFound](#NotFound)</div>

* * *

## <div class="method"><a name="getProductVideos"></a> get /catalog/products/{product_id}/videos

</div>

<div class="method-summary">(<span class="nickname">getProductVideos</span>)</div>

<div class="method-notes">Gets all videos on a product.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the video is being attached.

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Return type

[ProductVideoCollectionResponse](#ProductVideoCollectionResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : [ {
        "product_id" : 123,
        "length" : "aeiou",
        "description" : "aeiou",
        "id" : 123,
        "title" : "aeiou",
        "sort_order" : 123
      } ],
      "meta" : {
        "per_page" : 123,
        "total" : 123,
        "count" : 123,
        "links" : {
          "next" : "aeiou",
          "current" : "aeiou",
          "previous" : "aeiou"
        },
        "total_pages" : 123,
        "current_page" : 123
      }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

List of product videos and metadata. [ProductVideoCollectionResponse](#ProductVideoCollectionResponse)</div>

* * *

## <div class="method"><a name="getProducts"></a> get /catalog/products

</div>

<div class="method-summary">(<span class="nickname">getProducts</span>)</div>

<div class="method-notes">Returns a paginated collection of `Products` objects from the BigCommerce Catalog.</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Query parameters

<div class="field-items">

</p>
<div class="param">id (optional)</div>

Query Parameter — Filter items by id.

</p>
<div class="param">name (optional)</div>

Query Parameter — Filter items by name.

</p>
<div class="param">sku (optional)</div>

Query Parameter — Filter items by sku.

</p>
<div class="param">upc (optional)</div>

Query Parameter — Filter items by upc.

</p>
<div class="param">price (optional)</div>

Query Parameter — Filter items by price.

</p>
<div class="param">weight (optional)</div>

Query Parameter — Filter items by weight.

</p>
<div class="param">condition (optional)</div>

Query Parameter — Filter items by condition.

</p>
<div class="param">brand_id (optional)</div>

Query Parameter — Filter items by brand_id.

</p>
<div class="param">date_modified (optional)</div>

Query Parameter — Filter items by date_modified. format: data-time

</p>
<div class="param">date_last_imported (optional)</div>

Query Parameter — Filter items by date_last_imported. format: data-time

</p>
<div class="param">is_visible (optional)</div>

Query Parameter — Filter items by is_visible.

</p>
<div class="param">is_featured (optional)</div>

Query Parameter — Filter items by is_featured.

</p>
<div class="param">is_free_shipping (optional)</div>

Query Parameter — Filter items by is_free_shipping.

</p>
<div class="param">inventory_level (optional)</div>

Query Parameter — Filter items by inventory_level.

</p>
<div class="param">inventory_low (optional)</div>

Query Parameter — Filter items by inventory_low; values: 1, 0.

</p>
<div class="param">out_of_stock (optional)</div>

Query Parameter — Filter items by out_of_stock. To enable the filter, pass `out_of_stock`=`1`.

</p>
<div class="param">total_sold (optional)</div>

Query Parameter — Filter items by total_sold.

</p>
<div class="param">type (optional)</div>

Query Parameter — Filter items by type: `physical` or `digital`.

</p>
<div class="param">categories (optional)</div>

Query Parameter — Filter items by categories.

</p>
<div class="param">keyword (optional)</div>

Query Parameter — Filter items by keywords found in the name, description, sku, keywords, or brand name.

</p>
<div class="param">keyword_context (optional)</div>

Query Parameter — Set context for a product search.

</p>
<div class="param">include (optional)</div>

Query Parameter — Include sub-resources on a product, with a comma-separated list. Valid expansions currently include `variants`, `images`, `custom_fields`, and `bulk_pricing_rules`.

</p>
<div class="param">availability (optional)</div>

Query Parameter — Filter items by availability. Values are: available, disabled, preorder.

</p>
<div class="param">page (optional)</div>

Query Parameter — Control the page in a limited list of products.

</p>
<div class="param">limit (optional)</div>

Query Parameter — Control the items per page.

</p>
<div class="param">direction (optional)</div>

Query Parameter — Sort direction. Values are: asc, desc.

</p>
<div class="param">sort (optional)</div>

Query Parameter — Field name to sort by. Values: id, name, sku, price, date_modified, date_last_imported, inventory_level, is_visible.

</div>

### Return type

[ProductCollectionResponse](#ProductCollectionResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : [ {
        "calculated_price" : 1.3579000000000001069366817318950779736042022705078125,
        "gift_wrapping_options_list" : [ 123 ],
        "page_title" : "aeiou",
        "videos" : [ {
          "product_id" : 123,
          "length" : "aeiou",
          "description" : "aeiou",
          "id" : 123,
          "title" : "aeiou",
          "sort_order" : 123
        } ],
        "is_condition_shown" : true,
        "variants" : [ {
          "image_url" : "aeiou",
          "option_values" : [ {
            "option_display_name" : "aeiou",
            "option_id" : 123,
            "id" : 123,
            "label" : "aeiou"
          } ],
          "weight" : 1.3579000000000001069366817318950779736042022705078125,
          "upc" : "aeiou",
          "bin_picking_number" : "aeiou",
          "sku_id" : 123,
          "purchasing_disabled_message" : "aeiou",
          "inventory_level" : 123,
          "price" : 1.3579000000000001069366817318950779736042022705078125,
          "inventory_warning_level" : 123,
          "product_id" : 123,
          "id" : 123,
          "purchasing_disabled" : true,
          "sku" : "aeiou",
          "cost_price" : 1.3579000000000001069366817318950779736042022705078125
        } ],
        "type" : "aeiou",
        "retail_price" : 1.3579000000000001069366817318950779736042022705078125,
        "layout_file" : "aeiou",
        "price" : 1.3579000000000001069366817318950779736042022705078125,
        "inventory_warning_level" : 123,
        "warranty" : "aeiou",
        "is_free_shipping" : true,
        "id" : 123,
        "sku" : "aeiou",
        "height" : 1.3579000000000001069366817318950779736042022705078125,
        "custom_url" : {
          "is_customized" : true,
          "url" : "aeiou"
        },
        "images" : [ "" ],
        "custom_fields" : [ {
          "product_id" : 123,
          "name" : "aeiou",
          "id" : 123,
          "value" : "aeiou"
        } ],
        "weight" : 1.3579000000000001069366817318950779736042022705078125,
        "upc" : "aeiou",
        "brand_id" : 123,
        "meta_description" : "aeiou",
        "condition" : "aeiou",
        "inventory_level" : 123,
        "name" : "aeiou",
        "inventory_tracking" : "aeiou",
        "preorder_release_date" : "aeiou",
        "description" : "aeiou",
        "bin_picking_number" : "aeiou",
        "availability" : "aeiou",
        "search_keywords" : "aeiou",
        "meta_keywords" : [ "aeiou" ],
        "is_price_hidden" : true,
        "order_quantity_minimum" : 123,
        "availability_description" : "aeiou",
        "fixed_cost_shipping_price" : 123,
        "categories" : [ 123 ],
        "sort_order" : 123,
        "cost_price" : 1.3579000000000001069366817318950779736042022705078125,
        "order_quantity_maximum" : 123,
        "is_visible" : true,
        "is_preorder_only" : true,
        "date_created" : "aeiou",
        "preorder_message" : "aeiou",
        "tax_class_id" : 123,
        "bulk_pricing_rules" : [ {
          "amount" : 1.3579000000000001069366817318950779736042022705078125,
          "quantity_min" : 123,
          "quantity_max" : 123,
          "id" : 123,
          "type" : "aeiou"
        } ],
        "sale_price" : 1.3579000000000001069366817318950779736042022705078125,
        "product_tax_code" : "aeiou",
        "depth" : 1.3579000000000001069366817318950779736042022705078125,
        "date_modified" : "aeiou",
        "gift_wrapping_options_type" : "aeiou",
        "width" : 1.3579000000000001069366817318950779736042022705078125,
        "price_hidden_label" : "aeiou",
        "is_featured" : true,
        "view_count" : 123
      } ],
      "meta" : {
        "per_page" : 123,
        "total" : 123,
        "count" : 123,
        "links" : {
          "next" : "aeiou",
          "current" : "aeiou",
          "previous" : "aeiou"
        },
        "total_pages" : 123,
        "current_page" : 123
      }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

An array of products and metadata. [ProductCollectionResponse](#ProductCollectionResponse)</div>

* * *

## <div class="method"><a name="getVariantById"></a> get /catalog/products/{product_id}/variants/{variant_id}

</div>

<div class="method-summary">(<span class="nickname">getVariantById</span>)</div>

<div class="method-notes">Gets a `Variant` object.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the resource belongs. format: int

</p>
<div class="param">variant_id (required)</div>

Path Parameter — The ID of the `Variant` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Return type

[VariantResponse](#VariantResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "image_url" : "aeiou",
        "option_values" : [ {
          "option_display_name" : "aeiou",
          "option_id" : 123,
          "id" : 123,
          "label" : "aeiou"
        } ],
        "weight" : 1.3579000000000001069366817318950779736042022705078125,
        "upc" : "aeiou",
        "bin_picking_number" : "aeiou",
        "sku_id" : 123,
        "purchasing_disabled_message" : "aeiou",
        "inventory_level" : 123,
        "price" : 1.3579000000000001069366817318950779736042022705078125,
        "inventory_warning_level" : 123,
        "product_id" : 123,
        "id" : 123,
        "purchasing_disabled" : true,
        "sku" : "aeiou",
        "cost_price" : 1.3579000000000001069366817318950779736042022705078125
      },
      "meta" : { }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A variant and metadata. [VariantResponse](#VariantResponse)

#### 404

The resource was not found. [NotFound](#NotFound)</div>

* * *

## <div class="method"><a name="getVariantMetafieldByProductIdAndVariantId"></a> get /catalog/products/{product_id}/variants/{variant_id}/metafields/{metafield_id}

</div>

<div class="method-summary">(<span class="nickname">getVariantMetafieldByProductIdAndVariantId</span>)</div>

<div class="method-notes">Gets a `Metafield`, by product_id and variant_id.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">metafield_id (required)</div>

Path Parameter — The ID of the `Metafield`. format: int

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the resource belongs. format: int

</p>
<div class="param">variant_id (required)</div>

Path Parameter — The ID of the `Variant` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Return type

[Metafield](#Metafield)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "updated_at" : "2000-01-23T04:56:07.000+00:00",
      "namespace" : "aeiou",
      "resource_type" : "aeiou",
      "description" : "aeiou",
      "resource_id" : 123,
      "created_at" : "2000-01-23T04:56:07.000+00:00",
      "id" : 123,
      "value" : "aeiou",
      "key" : "aeiou",
      "permission_set" : "aeiou"
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A metafield object. [Metafield](#Metafield)

#### 404

The resource was not found. [NotFound](#NotFound)</div>

* * *

## <div class="method"><a name="getVariantMetafieldsByProductIdAndVariantId"></a> get /catalog/products/{product_id}/variants/{variant_id}/metafields

</div>

<div class="method-summary">(<span class="nickname">getVariantMetafieldsByProductIdAndVariantId</span>)</div>

<div class="method-notes">Gets a `Metafield` object list, by product_id and variant_id.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the resource belongs. format: int

</p>
<div class="param">variant_id (required)</div>

Path Parameter — The ID of the `Variant` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Query parameters

<div class="field-items">

</p>
<div class="param">page (optional)</div>

Query Parameter — Control the page in a limited list of products.

</p>
<div class="param">limit (optional)</div>

Query Parameter — Control the items per page.

</p>
<div class="param">key (optional)</div>

Query Parameter — Filter based on a metafield's key.

</p>
<div class="param">namespace (optional)</div>

Query Parameter — Filter based on a metafield's key.

</div>

### Return type

[MetaFieldCollectionResponse](#MetaFieldCollectionResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : [ {
        "updated_at" : "2000-01-23T04:56:07.000+00:00",
        "namespace" : "aeiou",
        "resource_type" : "aeiou",
        "description" : "aeiou",
        "resource_id" : 123,
        "created_at" : "2000-01-23T04:56:07.000+00:00",
        "id" : 123,
        "value" : "aeiou",
        "key" : "aeiou",
        "permission_set" : "aeiou"
      } ],
      "meta" : {
        "per_page" : 123,
        "total" : 123,
        "count" : 123,
        "links" : {
          "next" : "aeiou",
          "current" : "aeiou",
          "previous" : "aeiou"
        },
        "total_pages" : 123,
        "current_page" : 123
      }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

An array of metafields and metadata. [MetaFieldCollectionResponse](#MetaFieldCollectionResponse)

#### 404

The resource was not found. [NotFound](#NotFound)</div>

* * *

## <div class="method"><a name="getVariants"></a> get /catalog/variants

</div>

<div class="method-summary">(<span class="nickname">getVariants</span>)</div>

<div class="method-notes">Returns a `Variant` object list from the BigCommerce Catalog.</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Query parameters

<div class="field-items">

</p>
<div class="param">id (optional)</div>

Query Parameter — Filter items by id.

</p>
<div class="param">sku (optional)</div>

Query Parameter — Filter items by sku.

</p>
<div class="param">page (optional)</div>

Query Parameter — Control the page in a limited list of products.

</p>
<div class="param">limit (optional)</div>

Query Parameter — Control the items per page.

</div>

### Return type

[VariantCollectionResponse](#VariantCollectionResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : [ {
        "image_url" : "aeiou",
        "option_values" : [ {
          "option_display_name" : "aeiou",
          "option_id" : 123,
          "id" : 123,
          "label" : "aeiou"
        } ],
        "weight" : 1.3579000000000001069366817318950779736042022705078125,
        "upc" : "aeiou",
        "bin_picking_number" : "aeiou",
        "sku_id" : 123,
        "purchasing_disabled_message" : "aeiou",
        "inventory_level" : 123,
        "price" : 1.3579000000000001069366817318950779736042022705078125,
        "inventory_warning_level" : 123,
        "product_id" : 123,
        "id" : 123,
        "purchasing_disabled" : true,
        "sku" : "aeiou",
        "cost_price" : 1.3579000000000001069366817318950779736042022705078125
      } ],
      "meta" : {
        "per_page" : 123,
        "total" : 123,
        "count" : 123,
        "links" : {
          "next" : "aeiou",
          "current" : "aeiou",
          "previous" : "aeiou"
        },
        "total_pages" : 123,
        "current_page" : 123
      }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

An array of variants and metadata. [VariantCollectionResponse](#VariantCollectionResponse)

#### 404

The resource was not found. [NotFound](#NotFound)</div>

* * *

## <div class="method"><a name="getVariantsByProductId"></a> get /catalog/products/{product_id}/variants

</div>

<div class="method-summary">(<span class="nickname">getVariantsByProductId</span>)</div>

<div class="method-notes">Returns a `Variant` object list from the BigCommerce Catalog.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Query parameters

<div class="field-items">

</p>
<div class="param">page (optional)</div>

Query Parameter — Control the page in a limited list of products.

</p>
<div class="param">limit (optional)</div>

Query Parameter — Control the items per page.

</div>

### Return type

[VariantCollectionResponse](#VariantCollectionResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : [ {
        "image_url" : "aeiou",
        "option_values" : [ {
          "option_display_name" : "aeiou",
          "option_id" : 123,
          "id" : 123,
          "label" : "aeiou"
        } ],
        "weight" : 1.3579000000000001069366817318950779736042022705078125,
        "upc" : "aeiou",
        "bin_picking_number" : "aeiou",
        "sku_id" : 123,
        "purchasing_disabled_message" : "aeiou",
        "inventory_level" : 123,
        "price" : 1.3579000000000001069366817318950779736042022705078125,
        "inventory_warning_level" : 123,
        "product_id" : 123,
        "id" : 123,
        "purchasing_disabled" : true,
        "sku" : "aeiou",
        "cost_price" : 1.3579000000000001069366817318950779736042022705078125
      } ],
      "meta" : {
        "per_page" : 123,
        "total" : 123,
        "count" : 123,
        "links" : {
          "next" : "aeiou",
          "current" : "aeiou",
          "previous" : "aeiou"
        },
        "total_pages" : 123,
        "current_page" : 123
      }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

An array of variants and metadata. [VariantCollectionResponse](#VariantCollectionResponse)

#### 404

The resource was not found. [NotFound](#NotFound)</div>

* * *

## <div class="method"><a name="updateBrand"></a> put /catalog/brands/{brand_id}

</div>

<div class="method-summary">(<span class="nickname">updateBrand</span>)</div>

<div class="method-notes">Updates a `Brand` in the BigCommerce Catalog.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">brand_id (required)</div>

Path Parameter — The ID of the `Brand` requested. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Request body

<div class="field-items">

</p>
<div class="param">brand [Brand](#Brand) (required)</div>

Body Parameter — Returns a `Brand` from the BigCommerce Catalog.

</div>

### Return type

[BrandResponse](#BrandResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "meta_description" : "aeiou",
        "page_title" : "aeiou",
        "image_url" : "aeiou",
        "name" : "aeiou",
        "id" : 123,
        "meta_keywords" : [ "aeiou" ],
        "search_keywords" : "aeiou"
      },
      "meta" : { }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A `Brand` object. [BrandResponse](#BrandResponse)

#### 404

The resource was not found. [NotFound](#NotFound)

#### 409

The `Brand` was in conflict with another product. This is the result of duplicate unique values, such as `name`. [ErrorResponse](#ErrorResponse)

#### 422

The `Brand` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details. [ErrorResponse](#ErrorResponse)</div>

* * *

## <div class="method"><a name="updateBrandMetafield"></a> put /catalog/brands/{brand_id}/metafields/{metafield_id}

</div>

<div class="method-summary">(<span class="nickname">updateBrandMetafield</span>)</div>

<div class="method-notes">Updates a `Metafield` object.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">metafield_id (required)</div>

Path Parameter — The ID of the `Metafield`. format: int

</p>
<div class="param">brand_id (required)</div>

Path Parameter — The ID of the `Brand` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Request body

<div class="field-items">

</p>
<div class="param">Metafield [Metafield](#Metafield) (required)</div>

Body Parameter — A `Metafield` object.

</div>

### Return type

[MetafieldResponse](#MetafieldResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "updated_at" : "2000-01-23T04:56:07.000+00:00",
        "namespace" : "aeiou",
        "resource_type" : "aeiou",
        "description" : "aeiou",
        "resource_id" : 123,
        "created_at" : "2000-01-23T04:56:07.000+00:00",
        "id" : 123,
        "value" : "aeiou",
        "key" : "aeiou",
        "permission_set" : "aeiou"
      },
      "meta" : { }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A metafield and metadata. [MetafieldResponse](#MetafieldResponse)

#### 404

The resource was not found. [NotFound](#NotFound)</div>

* * *

## <div class="method"><a name="updateCategory"></a> put /catalog/categories/{category_id}

</div>

<div class="method-summary">(<span class="nickname">updateCategory</span>)</div>

<div class="method-notes">Updates a `Category` in the BigCommerce Catalog.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">category_id (required)</div>

Path Parameter — The ID of the `Category` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Request body

<div class="field-items">

</p>
<div class="param">category [Category](#Category) (required)</div>

Body Parameter — A BigCommerce `Category` object.

</div>

### Return type

[CategoryResponse](#CategoryResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "is_visible" : true,
        "page_title" : "aeiou",
        "image_url" : "aeiou",
        "description" : "aeiou",
        "meta_keywords" : [ "aeiou" ],
        "search_keywords" : "aeiou",
        "default_product_sort" : "aeiou",
        "meta_description" : "aeiou",
        "layout_file" : "aeiou",
        "parent_id" : 123,
        "name" : "aeiou",
        "id" : 123,
        "sort_order" : 123,
        "views" : 123,
        "custom_url" : {
          "is_customized" : true,
          "url" : "aeiou"
        }
      },
      "meta" : { }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A category object. [CategoryResponse](#CategoryResponse)

#### 404

The resource was not found. [NotFound](#NotFound)

#### 409

The `Category` was in conflict with another category. This is the result of duplicate unique values, such as `name` or `custom_url`. [ErrorResponse](#ErrorResponse)

#### 422

The `Category` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details. [ErrorResponse](#ErrorResponse)</div>

* * *

## <div class="method"><a name="updateCategoryMetafield"></a> put /catalog/categories/{category_id}/metafields/{metafield_id}

</div>

<div class="method-summary">(<span class="nickname">updateCategoryMetafield</span>)</div>

<div class="method-notes">Updates a `Metafield` object.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">metafield_id (required)</div>

Path Parameter — The ID of the `Metafield`. format: int

</p>
<div class="param">category_id (required)</div>

Path Parameter — The ID of the `Category` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Request body

<div class="field-items">

</p>
<div class="param">Metafield [Metafield](#Metafield) (required)</div>

Body Parameter — A `Metafield` object.

</div>

### Return type

[MetafieldResponse](#MetafieldResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "updated_at" : "2000-01-23T04:56:07.000+00:00",
        "namespace" : "aeiou",
        "resource_type" : "aeiou",
        "description" : "aeiou",
        "resource_id" : 123,
        "created_at" : "2000-01-23T04:56:07.000+00:00",
        "id" : 123,
        "value" : "aeiou",
        "key" : "aeiou",
        "permission_set" : "aeiou"
      },
      "meta" : { }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A metafield and metadata. [MetafieldResponse](#MetafieldResponse)

#### 404

The resource was not found. [NotFound](#NotFound)</div>

* * *

## <div class="method"><a name="updateComplexRule"></a> put /catalog/products/{product_id}/complex-rules/{complex_rule_id}

</div>

<div class="method-summary">(<span class="nickname">updateComplexRule</span>)</div>

<div class="method-notes">Update an Product's `ComplexRule`, based on the `product_id` and `complex_rule_id`.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the `ComplexRule` belongs. format: int

</p>
<div class="param">complex_rule_id (required)</div>

Path Parameter — The ID of the `ComplexRule`.

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Return type

[ComplexRuleResponse](#ComplexRuleResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "purchasing_hidden" : true,
        "stop" : true,
        "price_adjuster" : {
          "adjuster" : "aeiou",
          "adjuster_value" : 1.3579000000000001069366817318950779736042022705078125
        },
        "image_url" : "aeiou",
        "product_id" : 123,
        "weight_adjuster" : "",
        "id" : 123,
        "purchasing_disabled" : true,
        "purchasing_disabled_message" : "aeiou",
        "conditions" : [ {
          "rule_id" : 123,
          "variant_id" : 123,
          "combination_id" : 123,
          "modifier_id" : 123,
          "modifier_value_id" : 123,
          "id" : 123
        } ],
        "sort_order" : 123,
        "enabled" : true
      },
      "meta" : {
        "per_page" : 123,
        "total" : 123,
        "count" : 123,
        "links" : {
          "next" : "aeiou",
          "current" : "aeiou",
          "previous" : "aeiou"
        },
        "total_pages" : 123,
        "current_page" : 123
      }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A `ComplexRule` object [ComplexRuleResponse](#ComplexRuleResponse)

#### 409

The `ComplexRule` was in conflict with another `ComplexRule`. This is the result of duplicate conditions. [ErrorResponse](#ErrorResponse)

#### 422

The `ComplexRule` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details. [ErrorResponse](#ErrorResponse)</div>

* * *

## <div class="method"><a name="updateModifier"></a> put /catalog/products/{product_id}/modifiers/{modifier_id}

</div>

<div class="method-summary">(<span class="nickname">updateModifier</span>)</div>

<div class="method-notes">Update an Product's `Modifier` based on the product_id and modifier_id.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the `Modifier` belongs. format: int

</p>
<div class="param">modifier_id (required)</div>

Path Parameter — The ID of the `Modifier`.

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Request body

<div class="field-items">

</p>
<div class="param">modifier [Modifier](#Modifier) (required)</div>

Body Parameter — A BigCommerce `Modifier` object.

</div>

### Return type

[ModifierResponse](#ModifierResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "product_id" : 123,
        "option_values" : [ "" ],
        "name" : "aeiou",
        "id" : 123,
        "display_name" : "aeiou",
        "type" : "aeiou",
        "config" : {
          "text_max_length" : 123,
          "file_types_other" : [ "aeiou" ],
          "file_max_size" : 123,
          "file_types_supported" : [ "aeiou" ],
          "text_characters_limited" : true,
          "product_list_adjusts_inventory" : true,
          "number_limited" : true,
          "checked_by_default" : true,
          "date_latest_value" : "2000-01-23",
          "product_list_adjusts_pricing" : true,
          "default_value" : "aeiou",
          "date_limited" : true,
          "text_max_lines" : 123,
          "checkbox_label" : "aeiou",
          "text_min_length" : 123,
          "file_types_mode" : "aeiou",
          "text_lines_limited" : true,
          "number_highest_value" : 1.3579000000000001069366817318950779736042022705078125,
          "date_earliest_value" : "2000-01-23",
          "date_limit_mode" : "aeiou",
          "number_lowest_value" : 1.3579000000000001069366817318950779736042022705078125,
          "number_integers_only" : true,
          "product_list_shipping_calc" : "aeiou",
          "number_limit_mode" : "aeiou"
        },
        "required" : true
      },
      "meta" : {
        "per_page" : 123,
        "total" : 123,
        "count" : 123,
        "links" : {
          "next" : "aeiou",
          "current" : "aeiou",
          "previous" : "aeiou"
        },
        "total_pages" : 123,
        "current_page" : 123
      }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A modifier object. [ModifierResponse](#ModifierResponse)

#### 409

The `Modifier` was in conflict with another modifier or option. This is the result of duplicate unique fields, such as `name`. [ErrorResponse](#ErrorResponse)

#### 422

The `Modifier` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details. [ErrorResponse](#ErrorResponse)</div>

* * *

## <div class="method"><a name="updateOption"></a> put /catalog/products/{product_id}/options/{option_id}

</div>

<div class="method-summary">(<span class="nickname">updateOption</span>)</div>

<div class="method-notes">Update a Product's `Option`, based on the product_id and option_id.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the `Option` belongs. format: int

</p>
<div class="param">option_id (required)</div>

Path Parameter — The ID of the `Option`.

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Request body

<div class="field-items">

</p>
<div class="param">option [Option](#Option) (required)</div>

Body Parameter — A BigCommerce `Option` object.

</div>

### Return type

[OptionResponse](#OptionResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "product_id" : 123,
        "option_values" : [ {
          "id" : 123,
          "label" : "aeiou",
          "value_data" : "{}",
          "is_default" : true,
          "sort_order" : 123
        } ],
        "name" : "aeiou",
        "id" : 123,
        "display_name" : "aeiou",
        "type" : "aeiou",
        "config" : {
          "text_max_length" : 123,
          "file_types_other" : [ "aeiou" ],
          "file_max_size" : 123,
          "file_types_supported" : [ "aeiou" ],
          "text_characters_limited" : true,
          "product_list_adjusts_inventory" : true,
          "number_limited" : true,
          "checked_by_default" : true,
          "date_latest_value" : "2000-01-23",
          "product_list_adjusts_pricing" : true,
          "default_value" : "aeiou",
          "date_limited" : true,
          "text_max_lines" : 123,
          "checkbox_label" : "aeiou",
          "text_min_length" : 123,
          "file_types_mode" : "aeiou",
          "text_lines_limited" : true,
          "number_highest_value" : 1.3579000000000001069366817318950779736042022705078125,
          "date_earliest_value" : "2000-01-23",
          "date_limit_mode" : "aeiou",
          "number_lowest_value" : 1.3579000000000001069366817318950779736042022705078125,
          "number_integers_only" : true,
          "product_list_shipping_calc" : "aeiou",
          "number_limit_mode" : "aeiou"
        }
      },
      "meta" : {
        "per_page" : 123,
        "total" : 123,
        "count" : 123,
        "links" : {
          "next" : "aeiou",
          "current" : "aeiou",
          "previous" : "aeiou"
        },
        "total_pages" : 123,
        "current_page" : 123
      }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

An `Option` object. [OptionResponse](#OptionResponse)

#### 409

The `Option` was in conflict with another option. This is the result of duplicate unique fields, such as `name`. [ErrorResponse](#ErrorResponse)

#### 422

The `Option` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details. [ErrorResponse](#ErrorResponse)</div>

* * *

## <div class="method"><a name="updateProduct"></a> put /catalog/products/{product_id}

</div>

<div class="method-summary">(<span class="nickname">updateProduct</span>)</div>

<div class="method-notes">Updates a `Product` in the BigCommerce Catalog.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Request body

<div class="field-items">

</p>
<div class="param">product [ProductPut](#ProductPut) (required)</div>

Body Parameter — A BigCommerce `Product` object.

</div>

### Return type

[ProductResponse](#ProductResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "calculated_price" : 1.3579000000000001069366817318950779736042022705078125,
        "gift_wrapping_options_list" : [ 123 ],
        "page_title" : "aeiou",
        "videos" : [ {
          "product_id" : 123,
          "length" : "aeiou",
          "description" : "aeiou",
          "id" : 123,
          "title" : "aeiou",
          "sort_order" : 123
        } ],
        "is_condition_shown" : true,
        "variants" : [ {
          "image_url" : "aeiou",
          "option_values" : [ {
            "option_display_name" : "aeiou",
            "option_id" : 123,
            "id" : 123,
            "label" : "aeiou"
          } ],
          "weight" : 1.3579000000000001069366817318950779736042022705078125,
          "upc" : "aeiou",
          "bin_picking_number" : "aeiou",
          "sku_id" : 123,
          "purchasing_disabled_message" : "aeiou",
          "inventory_level" : 123,
          "price" : 1.3579000000000001069366817318950779736042022705078125,
          "inventory_warning_level" : 123,
          "product_id" : 123,
          "id" : 123,
          "purchasing_disabled" : true,
          "sku" : "aeiou",
          "cost_price" : 1.3579000000000001069366817318950779736042022705078125
        } ],
        "type" : "aeiou",
        "retail_price" : 1.3579000000000001069366817318950779736042022705078125,
        "layout_file" : "aeiou",
        "price" : 1.3579000000000001069366817318950779736042022705078125,
        "inventory_warning_level" : 123,
        "warranty" : "aeiou",
        "is_free_shipping" : true,
        "id" : 123,
        "sku" : "aeiou",
        "height" : 1.3579000000000001069366817318950779736042022705078125,
        "custom_url" : {
          "is_customized" : true,
          "url" : "aeiou"
        },
        "images" : [ "" ],
        "custom_fields" : [ {
          "product_id" : 123,
          "name" : "aeiou",
          "id" : 123,
          "value" : "aeiou"
        } ],
        "weight" : 1.3579000000000001069366817318950779736042022705078125,
        "upc" : "aeiou",
        "brand_id" : 123,
        "meta_description" : "aeiou",
        "condition" : "aeiou",
        "inventory_level" : 123,
        "name" : "aeiou",
        "inventory_tracking" : "aeiou",
        "preorder_release_date" : "aeiou",
        "description" : "aeiou",
        "bin_picking_number" : "aeiou",
        "availability" : "aeiou",
        "search_keywords" : "aeiou",
        "meta_keywords" : [ "aeiou" ],
        "is_price_hidden" : true,
        "order_quantity_minimum" : 123,
        "availability_description" : "aeiou",
        "fixed_cost_shipping_price" : 123,
        "categories" : [ 123 ],
        "sort_order" : 123,
        "cost_price" : 1.3579000000000001069366817318950779736042022705078125,
        "order_quantity_maximum" : 123,
        "is_visible" : true,
        "is_preorder_only" : true,
        "date_created" : "aeiou",
        "preorder_message" : "aeiou",
        "tax_class_id" : 123,
        "bulk_pricing_rules" : [ {
          "amount" : 1.3579000000000001069366817318950779736042022705078125,
          "quantity_min" : 123,
          "quantity_max" : 123,
          "id" : 123,
          "type" : "aeiou"
        } ],
        "sale_price" : 1.3579000000000001069366817318950779736042022705078125,
        "product_tax_code" : "aeiou",
        "depth" : 1.3579000000000001069366817318950779736042022705078125,
        "date_modified" : "aeiou",
        "gift_wrapping_options_type" : "aeiou",
        "width" : 1.3579000000000001069366817318950779736042022705078125,
        "price_hidden_label" : "aeiou",
        "is_featured" : true,
        "view_count" : 123
      },
      "meta" : { }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A product. [ProductResponse](#ProductResponse)

#### 404

The resource was not found. [NotFound](#NotFound)

#### 409

`Product` was in conflict with another product. This is the result of duplicate unique values, such as name or SKU; a missing category, brand, or tax_class with which the product is being associated; or a conflicting bulk_pricing_rule. [ErrorResponse](#ErrorResponse)

#### 422

`Product` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details. [ErrorResponse](#ErrorResponse)</div>

* * *

## <div class="method"><a name="updateProductImage"></a> put /catalog/products/{product_id}/images/{image_id}

</div>

<div class="method-summary">(<span class="nickname">updateProductImage</span>)</div>

<div class="method-notes">Updates an image on a product. Publicly accessible URLs and files (form post) are valid parameters.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the image is being attached.

</p>
<div class="param">image_id (required)</div>

Path Parameter — The ID of the `Image` that is being operated on.

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Request body

<div class="field-items">

</p>
<div class="param">productImage [ProductImagePut](#ProductImagePut) (required)</div>

Body Parameter — A BigCommerce `ProductImage` object.

</div>

### Return type

[ProductImageResponse](#ProductImageResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : "",
      "meta" : { }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A product image. [ProductImageResponse](#ProductImageResponse)

#### 404

The resource was not found. [NotFound](#NotFound)</div>

* * *

## <div class="method"><a name="updateProductMetafield"></a> put /catalog/products/{product_id}/metafields/{metafield_id}

</div>

<div class="method-summary">(<span class="nickname">updateProductMetafield</span>)</div>

<div class="method-notes">Updates a `Metafield` object.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">metafield_id (required)</div>

Path Parameter — The ID of the `Metafield`. format: int

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Request body

<div class="field-items">

</p>
<div class="param">Metafield [Metafield](#Metafield) (required)</div>

Body Parameter — `Metafield` object

</div>

### Return type

[MetafieldResponse](#MetafieldResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "updated_at" : "2000-01-23T04:56:07.000+00:00",
        "namespace" : "aeiou",
        "resource_type" : "aeiou",
        "description" : "aeiou",
        "resource_id" : 123,
        "created_at" : "2000-01-23T04:56:07.000+00:00",
        "id" : 123,
        "value" : "aeiou",
        "key" : "aeiou",
        "permission_set" : "aeiou"
      },
      "meta" : { }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A metafield and metadata. [MetafieldResponse](#MetafieldResponse)

#### 404

The resource was not found. [NotFound](#NotFound)</div>

* * *

## <div class="method"><a name="updateProductVideo"></a> put /catalog/products/{product_id}/videos/{video_id}

</div>

<div class="method-summary">(<span class="nickname">updateProductVideo</span>)</div>

<div class="method-notes">Updates a video on a product.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the video is being attached.

</p>
<div class="param">video_id (required)</div>

Path Parameter — The ID of the `Video` being operated on.

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Request body

<div class="field-items">

</p>
<div class="param">productVideo [ProductVideo](#ProductVideo) (required)</div>

Body Parameter — A BigCommerce `ProductVideo` object.

</div>

### Return type

[ProductVideoResponse](#ProductVideoResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "product_id" : 123,
        "length" : "aeiou",
        "description" : "aeiou",
        "id" : 123,
        "title" : "aeiou",
        "sort_order" : 123
      },
      "meta" : {
        "per_page" : 123,
        "total" : 123,
        "count" : 123,
        "links" : {
          "next" : "aeiou",
          "current" : "aeiou",
          "previous" : "aeiou"
        },
        "total_pages" : 123,
        "current_page" : 123
      }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A product video [ProductVideoResponse](#ProductVideoResponse)

#### 404

The resource was not found. [NotFound](#NotFound)</div>

* * *

## <div class="method"><a name="updateVariant"></a> put /catalog/products/{product_id}/variants/{variant_id}

</div>

<div class="method-summary">(<span class="nickname">updateVariant</span>)</div>

<div class="method-notes">Updates a `Variant` object.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the resource belongs. format: int

</p>
<div class="param">variant_id (required)</div>

Path Parameter — The ID of the `Variant` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Request body

<div class="field-items">

</p>
<div class="param">Variant [Variant](#Variant) (required)</div>

Body Parameter — `Variant` object

</div>

### Return type

[VariantResponse](#VariantResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "image_url" : "aeiou",
        "option_values" : [ {
          "option_display_name" : "aeiou",
          "option_id" : 123,
          "id" : 123,
          "label" : "aeiou"
        } ],
        "weight" : 1.3579000000000001069366817318950779736042022705078125,
        "upc" : "aeiou",
        "bin_picking_number" : "aeiou",
        "sku_id" : 123,
        "purchasing_disabled_message" : "aeiou",
        "inventory_level" : 123,
        "price" : 1.3579000000000001069366817318950779736042022705078125,
        "inventory_warning_level" : 123,
        "product_id" : 123,
        "id" : 123,
        "purchasing_disabled" : true,
        "sku" : "aeiou",
        "cost_price" : 1.3579000000000001069366817318950779736042022705078125
      },
      "meta" : { }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A variant and metadata. [VariantResponse](#VariantResponse)

#### 404

The resource was not found. [NotFound](#NotFound)</div>

* * *

## <div class="method"><a name="updateVariantMetafield"></a> put /catalog/products/{product_id}/variants/{variant_id}/metafields/{metafield_id}

</div>

<div class="method-summary">(<span class="nickname">updateVariantMetafield</span>)</div>

<div class="method-notes">Updates a `Metafield` object.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">metafield_id (required)</div>

Path Parameter — The ID of the `Metafield`. format: int

</p>
<div class="param">product_id (required)</div>

Path Parameter — The ID of the `Product` to which the resource belongs. format: int

</p>
<div class="param">variant_id (required)</div>

Path Parameter — The ID of the `Variant` to which the resource belongs. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Request body

<div class="field-items">

</p>
<div class="param">Metafield [Metafield](#Metafield) (required)</div>

Body Parameter — A `Metafield` object.

</div>

### Return type

[MetafieldResponse](#MetafieldResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "updated_at" : "2000-01-23T04:56:07.000+00:00",
        "namespace" : "aeiou",
        "resource_type" : "aeiou",
        "description" : "aeiou",
        "resource_id" : 123,
        "created_at" : "2000-01-23T04:56:07.000+00:00",
        "id" : 123,
        "value" : "aeiou",
        "key" : "aeiou",
        "permission_set" : "aeiou"
      },
      "meta" : { }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A metafield and metadata. [MetafieldResponse](#MetafieldResponse)

#### 404

The resource was not found. [NotFound](#NotFound)</div>

* * *

## <a name="Customers">Customers</a>

## <div class="method"><a name="createSubscriber"></a> post /customers/subscribers

</div>

<div class="method-summary">(<span class="nickname">createSubscriber</span>)</div>

<div class="method-notes">Creates a `Subscriber` object.</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Request body

<div class="field-items">

</p>
<div class="param">subscriber [Subscriber](#Subscriber) (required)</div>

Body Parameter — `Subscriber` object

</div>

### Return type

[SubscriberResponse](#SubscriberResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "date_modified" : "aeiou",
        "date_created" : "aeiou",
        "last_name" : "aeiou",
        "id" : 123,
        "source" : "aeiou",
        "first_name" : "aeiou",
        "order_id" : 123,
        "email" : "aeiou"
      },
      "meta" : { }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A `Subscriber` object. [SubscriberResponse](#SubscriberResponse)

#### 409

The `Subscriber` was in conflict with another subscriber. This is the result of duplicate unique values, such as `email`. [ErrorResponse](#ErrorResponse)

#### 422

The `Subscriber` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details. [ErrorResponse](#ErrorResponse)</div>

* * *

## <div class="method"><a name="deleteSubscriberById"></a> delete /customers/subscribers/{subscriber_id}

</div>

<div class="method-summary">(<span class="nickname">deleteSubscriberById</span>)</div>

<div class="method-notes">Deletes a `Subscriber` object.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">subscriber_id (required)</div>

Path Parameter — The ID of the `Subscriber` requested. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 204

An empty response.[](#)</div>

* * *

## <div class="method"><a name="deleteSubscribers"></a> delete /customers/subscribers

</div>

<div class="method-summary">(<span class="nickname">deleteSubscribers</span>)</div>

<div class="method-notes">Deletes a Subscriber or Subscribers from BigCommerce Customers.</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Query parameters

<div class="field-items">

</p>
<div class="param">email (optional)</div>

Query Parameter — Filter items by email.

</p>
<div class="param">first_name (optional)</div>

Query Parameter — Filter items by first_name.

</p>
<div class="param">last_name (optional)</div>

Query Parameter — Filter items by last_name.

</p>
<div class="param">source (optional)</div>

Query Parameter — Filter items by source.

</p>
<div class="param">order_id (optional)</div>

Query Parameter — Filter items by order_id.

</p>
<div class="param">date_created (optional)</div>

Query Parameter — Filter items by date_created. format: data-time

</p>
<div class="param">date_modified (optional)</div>

Query Parameter — Filter items by date_modified. format: data-time

</div>

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 204

An empty response.[](#)</div>

* * *

## <div class="method"><a name="getSubscriberById"></a> get /customers/subscribers/{subscriber_id}

</div>

<div class="method-summary">(<span class="nickname">getSubscriberById</span>)</div>

<div class="method-notes">Gets `Subscriber` object.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">subscriber_id (required)</div>

Path Parameter — The ID of the `Subscriber` requested. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Return type

[SubscriberResponse](#SubscriberResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "date_modified" : "aeiou",
        "date_created" : "aeiou",
        "last_name" : "aeiou",
        "id" : 123,
        "source" : "aeiou",
        "first_name" : "aeiou",
        "order_id" : 123,
        "email" : "aeiou"
      },
      "meta" : { }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A `Subscriber` object. [SubscriberResponse](#SubscriberResponse)

#### 404

The resource was not found. [NotFound](#NotFound)</div>

* * *

## <div class="method"><a name="getSubscribers"></a> get /customers/subscribers

</div>

<div class="method-summary">(<span class="nickname">getSubscribers</span>)</div>

<div class="method-notes">Returns a paginated Subscribers collection.</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Query parameters

<div class="field-items">

</p>
<div class="param">email (optional)</div>

Query Parameter — Filter items by email.

</p>
<div class="param">first_name (optional)</div>

Query Parameter — Filter items by first_name.

</p>
<div class="param">last_name (optional)</div>

Query Parameter — Filter items by last_name.

</p>
<div class="param">source (optional)</div>

Query Parameter — Filter items by source.

</p>
<div class="param">order_id (optional)</div>

Query Parameter — Filter items by order_id.

</p>
<div class="param">date_created (optional)</div>

Query Parameter — Filter items by date_created. format: data-time

</p>
<div class="param">date_modified (optional)</div>

Query Parameter — Filter items by date_modified. format: data-time

</p>
<div class="param">page (optional)</div>

Query Parameter — Control the page in a limited list of products.

</p>
<div class="param">limit (optional)</div>

Query Parameter — Control the items per page.

</div>

### Return type

[SubscriberCollectionResponse](#SubscriberCollectionResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : [ {
        "date_modified" : "aeiou",
        "date_created" : "aeiou",
        "last_name" : "aeiou",
        "id" : 123,
        "source" : "aeiou",
        "first_name" : "aeiou",
        "order_id" : 123,
        "email" : "aeiou"
      } ],
      "meta" : {
        "per_page" : 123,
        "total" : 123,
        "count" : 123,
        "links" : {
          "next" : "aeiou",
          "current" : "aeiou",
          "previous" : "aeiou"
        },
        "total_pages" : 123,
        "current_page" : 123
      }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

An array of `Subscriber` objects and metadata. [SubscriberCollectionResponse](#SubscriberCollectionResponse)</div>

* * *

## <div class="method"><a name="updateSubscriber"></a> put /customers/subscribers/{subscriber_id}

</div>

<div class="method-summary">(<span class="nickname">updateSubscriber</span>)</div>

<div class="method-notes">Updates a `Subscriber` object.</div>

### Path parameters

<div class="field-items">

</p>
<div class="param">subscriber_id (required)</div>

Path Parameter — The ID of the `Subscriber` requested. format: int

</div>

### Consumes

This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:

*   `application/json`

### Request body

<div class="field-items">

</p>
<div class="param">subscriber [Subscriber](#Subscriber) (required)</div>

Body Parameter — Returns a `Subscriber` object.

</div>

### Return type

[SubscriberResponse](#SubscriberResponse)

### Example data

<div class="example-data-content-type">Content-Type: application/json</div>

    {
      "data" : {
        "date_modified" : "aeiou",
        "date_created" : "aeiou",
        "last_name" : "aeiou",
        "id" : 123,
        "source" : "aeiou",
        "first_name" : "aeiou",
        "order_id" : 123,
        "email" : "aeiou"
      },
      "meta" : { }
    }

### Produces

This API call produces the following media types according to the <span class="header">Accept</span> request header; the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.

*   `application/json`

### Responses

#### 200

A `Subscriber` object. [SubscriberResponse](#SubscriberResponse)

#### 404

The resource was not found. [NotFound](#NotFound)

#### 409

The `Subscriber` was in conflict with another subscriber. This is the result of duplicate unique values, such as `email`. [ErrorResponse](#ErrorResponse)

#### 422

The `Subscriber` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details. [ErrorResponse](#ErrorResponse)</div>

* * *

<div class="up">[Up](#__Models)</div>

## <a name="__Models">Models</a>

[ Jump to [Methods](#__Methods) ]

### Table of Contents

1.  [`Adjuster`](#Adjuster)
2.  [`BaseError`](#BaseError)
3.  [`Brand`](#Brand)
4.  [`BrandCollectionResponse`](#BrandCollectionResponse)
5.  [`BrandResponse`](#BrandResponse)
6.  [`BulkPricingRule`](#BulkPricingRule)
7.  [`CatalogSummary`](#CatalogSummary)
8.  [`CatalogSummaryResponse`](#CatalogSummaryResponse)
9.  [`Category`](#Category)
10.  [`CategoryCollectionResponse`](#CategoryCollectionResponse)
11.  [`CategoryNode`](#CategoryNode)
12.  [`CategoryResponse`](#CategoryResponse)
13.  [`CategoryTreeCollectionResponse`](#CategoryTreeCollectionResponse)
14.  [`CollectionMeta`](#CollectionMeta)
15.  [`CollectionMeta_links`](#CollectionMeta_links)
16.  [`ComplexRule`](#ComplexRule)
17.  [`ComplexRuleCollectionResponse`](#ComplexRuleCollectionResponse)
18.  [`ComplexRuleCondition`](#ComplexRuleCondition)
19.  [`ComplexRuleResponse`](#ComplexRuleResponse)
20.  [`CustomField`](#CustomField)
21.  [`CustomUrl`](#CustomUrl)
22.  [`DetailedErrors`](#DetailedErrors)
23.  [`ErrorResponse`](#ErrorResponse)
24.  [`ImageResponse`](#ImageResponse)
25.  [`Meta`](#Meta)
26.  [`MetaFieldCollectionResponse`](#MetaFieldCollectionResponse)
27.  [`Metafield`](#Metafield)
28.  [`MetafieldResponse`](#MetafieldResponse)
29.  [`Modifier`](#Modifier)
30.  [`ModifierCollectionResponse`](#ModifierCollectionResponse)
31.  [`ModifierResponse`](#ModifierResponse)
32.  [`ModifierValue`](#ModifierValue)
33.  [`ModifierValue_adjusters`](#ModifierValue_adjusters)
34.  [`ModifierValue_adjusters_purchasing_disabled`](#ModifierValue_adjusters_purchasing_disabled)
35.  [`NotFound`](#NotFound)
36.  [`Option`](#Option)
37.  [`OptionCollectionResponse`](#OptionCollectionResponse)
38.  [`OptionConfig`](#OptionConfig)
39.  [`OptionResponse`](#OptionResponse)
40.  [`OptionValue`](#OptionValue)
41.  [`OptionValueShort`](#OptionValueShort)
42.  [`OptionValueShortPost`](#OptionValueShortPost)
43.  [`Product`](#Product)
44.  [`ProductCollectionResponse`](#ProductCollectionResponse)
45.  [`ProductImage`](#ProductImage)
46.  [`ProductImageBase`](#ProductImageBase)
47.  [`ProductImageCollectionResponse`](#ProductImageCollectionResponse)
48.  [`ProductImagePost`](#ProductImagePost)
49.  [`ProductImagePut`](#ProductImagePut)
50.  [`ProductImageResponse`](#ProductImageResponse)
51.  [`ProductPost`](#ProductPost)
52.  [`ProductPut`](#ProductPut)
53.  [`ProductResponse`](#ProductResponse)
54.  [`ProductVideo`](#ProductVideo)
55.  [`ProductVideoCollectionResponse`](#ProductVideoCollectionResponse)
56.  [`ProductVideoResponse`](#ProductVideoResponse)
57.  [`ResourceImage`](#ResourceImage)
58.  [`Subscriber`](#Subscriber)
59.  [`SubscriberCollectionResponse`](#SubscriberCollectionResponse)
60.  [`SubscriberResponse`](#SubscriberResponse)
61.  [`Variant`](#Variant)
62.  [`VariantCollectionResponse`](#VariantCollectionResponse)
63.  [`VariantPost`](#VariantPost)
64.  [`VariantResponse`](#VariantResponse)


### <a name="Adjuster">Adjuster </a>

<div class="field-items">

</p>
<div class="param">adjuster (optional)</div>

String: The type of adjuster for either the price or the weight of the variant, when the modifier value is selected on the storefront.

</p>
<div class="param-enum-header">Enum:</div>

<div class="param-enum">relative</div>

<div class="param-enum">percentage</div>

</p>
<div class="param">adjuster_value (optional)</div>

(OptionConfig:) The numeric amount by which the adjuster will change either the price or the weight of the variant, when the modifier value is selected on the storefront.

</div>

</div>


### <a name="BaseError">BaseError </a>

<div class="model-description">Error payload for the BigCommerce API.</div>

<div class="field-items">

</p>
<div class="param">status (optional)</div>

Integer: The HTTP status code

</p>
<div class="param">title (optional)</div>

String: The error title describing the particular error.

</p>
<div class="param">type (optional)</div>

(String)

</p>
<div class="param">instance (optional)</div>

(String)

</div>

</div>


### <a name="Brand">Brand </a>

<div class="field-items">

</p>
<div class="param">id (optional)</div>

Integer: The unique numeric ID of the brand; increments sequentially.

</p>
<div class="param">name (optional)</div>

String: The name of the brand. Must be unique.

</p>
<div class="param">page_title (optional)</div>

String: The title shown in the browser while viewing the brand.

</p>
<div class="param">meta_keywords (optional)</div>

array[String]: Comma-separated list of meta keywords to include in the HTML.

</p>
<div class="param">meta_description (optional)</div>

String: A meta description to include.

</p>
<div class="param">search_keywords (optional)</div>

String: A comma-separated list of keywords that can be used to locate this brand.

</p>
<div class="param">image_url (optional)</div>

String: Image URL used for this category on the storefront. Images can be uploaded via form file post to `/brands/{brandId}/image`, or by providing a publicly accessible URL in this field.

</div>

</div>


### <a name="BrandCollectionResponse">BrandCollectionResponse </a>

<div class="model-description">Response payload for the Bigcommerce API.</div>

<div class="field-items">

</p>
<div class="param">data (optional)</div>

(array[Brand])

</p>
<div class="param">meta (optional)</div>

(CollectionMeta)

</div>

</div>


### <a name="BrandResponse">BrandResponse </a>

<div class="model-description">Response payload for the Bigcommerce API.</div>

<div class="field-items">

</p>
<div class="param">data (optional)</div>

(Brand)

</p>
<div class="param">meta (optional)</div>

(Meta)

</div>

</div>


### <a name="BulkPricingRule">BulkPricingRule </a>

<div class="model-description">Rules that offer price discounts based on quantity breaks.</div>

<div class="field-items">

</p>
<div class="param">id (optional)</div>

Integer: The ID of the bulk pricing rule.

</p>
<div class="param">quantity_min (optional)</div>

Integer: The minimum inclusive quantity of a product to satisfy this rule. Must be greater than or equal to zero.

</p>
<div class="param">quantity_max (optional)</div>

Integer: The maximum inclusive quantity of a product to satisfy this rule. Must be greater than the `quantity_min` value – unless this field has a value of 0 (zero), in which case there will be no maximum bound for this rule.

</p>
<div class="param">type (optional)</div>

String: The type of adjustment that is made. Values: `price` - the adjustment amount per product; `percent` - the adjustment as a percentage of the original price; `fixed` - the adjusted absolute price of the product.

</p>
<div class="param-enum-header">Enum:</div>

<div class="param-enum">price</div>

<div class="param-enum">percent</div>

<div class="param-enum">fixed</div>

</p>
<div class="param">amount (optional)</div>

Double: The value of the adjustment by the bulk pricing rule. format: double

</div>

</div>


### <a name="CatalogSummary">CatalogSummary </a>

<div class="model-description">A BigCommerce Catalog Summary object describes a lightweight summary of the catalog.</div>

<div class="field-items">

</p>
<div class="param">inventory_count (optional)</div>

Integer: A count of all inventory items in the catalog.

</p>
<div class="param">inventory_value (optional)</div>

Double: Total value of store's inventory. format: double

</p>
<div class="param">primary_category_id (optional)</div>

Integer: ID of the category containing the most products.

</p>
<div class="param">primary_category_name (optional)</div>

String: Name of the category containing the most products.

</div>

</div>


### <a name="CatalogSummaryResponse">CatalogSummaryResponse </a>

<div class="model-description">Response payload for the Bigcommerce API.</div>

<div class="field-items">

</p>
<div class="param">data (optional)</div>

(CatalogSummary)

</p>
<div class="param">meta (optional)</div>

(Meta)

</div>

</div>


### <a name="Category">Category </a>

<div class="model-description">A BigCommerce category object.</div>

<div class="field-items">

</p>
<div class="param">id (optional)</div>

Integer: The unique numeric ID of the category; increments sequentially.

</p>
<div class="param">parent_id (optional)</div>

Integer: The unique numeric ID of the category's parent. This field controls where the category sits in the tree of categories that organize the catalog.

</p>
<div class="param">name (optional)</div>

String: The name displayed for the category. Name is unique with respect to the category's siblings.

</p>
<div class="param">description (optional)</div>

String: The product description, which can include HTML formatting.

</p>
<div class="param">views (optional)</div>

Integer: Number of views the category has on the storefront.

</p>
<div class="param">sort_order (optional)</div>

Integer: Priority this category will be given when included in the menu and category pages. The lower the number, the closer to the top of the results the category will be.

</p>
<div class="param">page_title (optional)</div>

String: Custom title for the category page. If not defined, the category name will be used as the meta title.

</p>
<div class="param">meta_keywords (optional)</div>

array[String]: Custom meta keywords for the category page. If not defined, the store's default keywords will be used. Must post as an array like: ["awesome","sauce"].

</p>
<div class="param">meta_description (optional)</div>

String: Custom meta description for the category page. If not defined, the store's default meta description will be used.

</p>
<div class="param">layout_file (optional)</div>

String: The layout template file used to render this category.

</p>
<div class="param">image_url (optional)</div>

String: Image URL used for this category on the storefront. Images can be uploaded via form file post to `/categories/{categoryId}/image`, or by providing a publicly accessible URL in this field.

</p>
<div class="param">is_visible (optional)</div>

Boolean: Flag to determine whether the product should be displayed to customers browsing the store. If `true`, the category will be displayed. If `false`, the category will be hidden from view.

</p>
<div class="param">search_keywords (optional)</div>

String: A comma-separated list of keywords that can be used to locate the category when searching the store.

</p>
<div class="param">default_product_sort (optional)</div>

String: Determines how the products are sorted on category page load.

</p>
<div class="param-enum-header">Enum:</div>

<div class="param-enum">use_store_settings</div>

<div class="param-enum">featured</div>

<div class="param-enum">newest</div>

<div class="param-enum">best_selling</div>

<div class="param-enum">alpha_asc</div>

<div class="param-enum">alpha_desc</div>

<div class="param-enum">avg_customer_review</div>

<div class="param-enum">price_asc</div>

<div class="param-enum">price_desc</div>

</p>
<div class="param">custom_url (optional)</div>

(CustomUrl))

</div>

</div>


### <a name="CategoryCollectionResponse">CategoryCollectionResponse </a>

<div class="model-description">Response payload for the Bigcommerce API.</div>

<div class="field-items">

</p>
<div class="param">data (optional)</div>

(array[Category])

</p>
<div class="param">meta (optional)</div>

(CollectionMeta)

</div>

</div>


### <a name="CategoryNode">CategoryNode </a>

<div class="model-description">A BigCommerce category node object. Used to reflect parent <> child category relationships.</div>

<div class="field-items">

</p>
<div class="param">id (optional)</div>

Integer: The unique numeric ID of the category; increments sequentially.

</p>
<div class="param">parent_id (optional)</div>

Integer: The unique numeric ID of the category's parent. This field controls where the category sits in the tree of categories that organize the catalog.

</p>
<div class="param">name (optional)</div>

String: The name displayed for the category. Name is unique with respect to the category's siblings.

</p>
<div class="param">is_visible (optional)</div>

Boolean: Flag to determine whether the product should be displayed to customers browsing the store. If `true`, the category will be displayed. If `false`, the category will be hidden from view.

</p>
<div class="param">url (optional)</div>

String: The custom URL for the category on the storefront.

</p>
<div class="param">children (optional)</div>

array[CategoryNode]: The list of children of the category.

</div>

</div>


### <a name="CategoryResponse">CategoryResponse </a>

<div class="model-description">Response payload for the Bigcommerce API.</div>

<div class="field-items">

</p>
<div class="param">data (optional)</div>

(Category)

</p>
<div class="param">meta (optional)</div>

(Meta)

</div>

</div>


### <a name="CategoryTreeCollectionResponse">CategoryTreeCollectionResponse </a>

<div class="model-description">Response payload for the Bigcommerce API.</div>

<div class="field-items">

</p>
<div class="param">data (optional)</div>

(array[CategoryNode])

</p>
<div class="param">meta (optional)</div>

(CollectionMeta)

</div>

</div>


### <a name="CollectionMeta">CollectionMeta </a>

<div class="model-description">Data about the response, including pagination and collection totals.</div>

<div class="field-items">

</p>
<div class="param">total (optional)</div>

Integer: Total number of items return in the result set.

</p>
<div class="param">count (optional)</div>

Integer: Total number of items in the collection.

</p>
<div class="param">per_page (optional)</div>

Integer: The amount of items returned in the collection per page, controlled by the limit parameter.

</p>
<div class="param">current_page (optional)</div>

Integer: The page you are currently on within the collection.

</p>
<div class="param">total_pages (optional)</div>

Integer: The total number of pages in the collection.

</p>
<div class="param">links (optional)</div>

(CollectionMeta_links)

</div>

</div>


### <a name="CollectionMeta_links">CollectionMeta_links </a>

<div class="model-description">Pagination links for the previous and next parts of the whole collection.</div>

<div class="field-items">

</p>
<div class="param">previous (optional)</div>

String: Link to the previous page returned in the response.

</p>
<div class="param">current (optional)</div>

String: Link to the current page returned in the response.

</p>
<div class="param">next (optional)</div>

String: Link to the next page returned in the response.

</div>

</div>


### <a name="ComplexRule">ComplexRule </a>

<div class="model-description">Apply price, weight, image, or availabilty adjustments to product, based on a set of conditions. A complex rule's condition must either contain more than one modifier value, or else contain a modifier value and a variant id.</div>

<div class="field-items">

</p>
<div class="param">id (optional)</div>

Integer: The unique numeric ID of the rule; increments sequentially.

</p>
<div class="param">product_id (optional)</div>

Integer: The unique numeric ID of the product with which the rule is associated; increments sequentially.

</p>
<div class="param">sort_order (optional)</div>

Integer: Priority this rule will be given, when making adjustments to the product properties.

</p>
<div class="param">enabled (optional)</div>

Boolean: Flag for determining whether the rule is to be used when adjusting a product's price, weight, image, or availabilty.

</p>
<div class="param">stop (optional)</div>

Boolean: Flag for determining whether other rules should not be applied after this rule has been applied.

</p>
<div class="param">price_adjuster (optional)</div>

(Adjuster)

</p>
<div class="param">weight_adjuster (optional)</div>

(Adjuster)

</p>
<div class="param">purchasing_disabled (optional)</div>

Boolean: Flag for determining whether the rule should disable purchasing of a product when the conditions are applied.

</p>
<div class="param">purchasing_disabled_message (optional)</div>

String: Message displayed on the storefront when a rule disables the purchasing of a product.

</p>
<div class="param">purchasing_hidden (optional)</div>

Boolean: Flag for determining whether the rule should hide purchasing of a product when the conditions are applied.

</p>
<div class="param">image_url (optional)</div>

String: The URL for an image displayed on the storefront when the conditions are applied.

</p>
<div class="param">conditions (optional)</div>

(array[ComplexRuleCondition)

</div>

</div>


### <a name="ComplexRuleCollectionResponse">ComplexRuleCollectionResponse </a>

<div class="model-description">Response payload for the Bigcommerce API.</div>

<div class="field-items">

</p>
<div class="param">data (optional)</div>

(array[ComplexRule])

</p>
<div class="param">meta (optional)</div>

(CollectionMeta)

</div>

</div>


### <a name="ComplexRuleCondition">ComplexRuleCondition </a>

<div class="model-description">Complex rules may return with conditions that apply to one or more variants, or with a single modifier value (if the rules were created using the v2 API or the control panel). Complex rules created or updated in the v3 API must have conditions that either reference multiple `modifier_value_id`'s, or else reference a `modifier_value_id` and a `variant_id`.</div>

<div class="field-items">

</p>
<div class="param">id (optional)</div>

Integer: The unique numeric ID of the rule condition; increments sequentially.

</p>
<div class="param">rule_id (optional)</div>

Integer: The unique numeric ID of the rule with which the condition is associated.

</p>
<div class="param">modifier_id (optional)</div>

Integer: The unique numeric ID of the modifier with which the rule condition is associated.

</p>
<div class="param">modifier_value_id (optional)</div>

Integer: The unique numeric ID of the modifier value with which the rule condition is associated.

</p>
<div class="param">variant_id (optional)</div>

Integer: The unique numeric ID of the variant the rule condition is associated with.

</p>
<div class="param">combination_id (optional)</div>

Integer: (READ-ONLY:) The unique numeric ID of the SKU (v2 API), or Combination, with which the rule condition is associated. This is to maintain cross-compatibility between v2 and v3.

</div>

</div>


### <a name="ComplexRuleResponse">ComplexRuleResponse </a>

<div class="model-description">Response payload for the Bigcommerce API.</div>

<div class="field-items">

</p>
<div class="param">data (optional)</div>

(ComplexRule)

</p>
<div class="param">meta (optional)</div>

(CollectionMeta)

</div>

</div>


### <a name="CustomField">CustomField </a>

<div class="model-description">Gets custom fields associated with a product. These allow you to specify additional information that will appear on the product's page, such as a book's ISBN or a DVD's release date.</div>

<div class="field-items">

</p>
<div class="param">id (optional)</div>

Integer: The unique numeric ID of the custom field; increments sequentially.

</p>
<div class="param">name (optional)</div>

String: The name of the field, shown on the storefront, orders, etc.

</p>
<div class="param">value (optional)</div>

String: The values or text of the field, shown on the storefront, orders, etc.

</p>
<div class="param">product_id (optional)</div>

Integer: The unique numeric identifier for the product with which the field is associated.

</div>

</div>


### <a name="CustomUrl">CustomUrl </a>

<div class="model-description">The custom URL for the product on the storefront.</div>

<div class="field-items">

</p>
<div class="param">url (optional)</div>

String: Product URL on the storefront.

</p>
<div class="param">is_customized (optional)</div>

Boolean: Returns `true` if the URL has been changed from its default state (the auto-assigned URL that BigCommerce provides).

</div>

</div>


### <a name="DetailedErrors">DetailedErrors </a>

</div>


### <a name="ErrorResponse">ErrorResponse </a>

<div class="field-items">

</p>
<div class="param">status (optional)</div>

Integer: The HTTP status code

</p>
<div class="param">title (optional)</div>

String: The error title describing the particular error.

</p>
<div class="param">type (optional)</div>

String:

</p>
<div class="param">instance (optional)</div>

String:

</p>
<div class="param">errors (optional)</div>

(DetailedErrors)

</div>

</div>


### <a name="ImageResponse">ImageResponse </a>

<div class="model-description">Response payload for the Bigcommerce API.</div>

<div class="field-items">

</p>
<div class="param">data (optional)</div>

(ResourceImage)

</p>
<div class="param">meta (optional)</div>

(Meta)

</div>

</div>


### <a name="Meta">Meta </a>

<div class="model-description">Empty meta object; might be used later.</div>

</div>


### <a name="MetaFieldCollectionResponse">MetaFieldCollectionResponse </a>

<div class="model-description">Response payload for the Bigcommerce API.</div>

<div class="field-items">

</p>
<div class="param">data (optional)</div>

(array[Metafield])

</p>
<div class="param">meta (optional)</div>

(CollectionMeta)

</div>

</div>


### <a name="Metafield">Metafield </a>

<div class="model-description">Allows app partners to write custom data to various resources in the API.</div>

<div class="field-items">

</p>
<div class="param">id (optional)</div>

Integer: The unique identifier for the metafields.

</p>
<div class="param">description (optional)</div>

String: Description for the metafields.

</p>
<div class="param">permission_set (optional)</div>

String: Determines whether the field is completely private to the app that owns the field (`app_only`), or visible to other API consumers (`read`), or completely open for reading and writing to other apps (`write`).

</p>
<div class="param-enum-header">Enum:</div>

<div class="param-enum">app_only</div>

<div class="param-enum">read</div>

<div class="param-enum">write</div>

</p>
<div class="param">namespace (optional)</div>

String: Namespace for the metafield, for organizational purposes.

</p>
<div class="param">resource_type (optional)</div>

String: The type of resource with which the metafield is associated.

</p>
<div class="param-enum-header">Enum:</div>

<div class="param-enum">category</div>

<div class="param-enum">brand</div>

<div class="param-enum">product</div>

<div class="param-enum">variant</div>

</p>
<div class="param">resource_id (optional)</div>

Integer: The unique identifier for the resource with which the metafield is associated.

</p>
<div class="param">key (optional)</div>

String: The name of the field, for example: `location_id`, `color`.

</p>
<div class="param">value (optional)</div>

String: The value of the field, for example: `1`, `blue`

</p>
<div class="param">created_at (optional)</div>

Date(DateTime): Date and time of the metafield's creation. format: date-time

</p>
<div class="param">updated_at (optional)</div>

Date(DateTime): Date and time when the metafield was last updated. format: date-time

</div>

</div>


### <a name="MetafieldResponse">MetafieldResponse </a>

<div class="model-description">Response payload for the Bigcommerce API.</div>

<div class="field-items">

</p>
<div class="param">data (optional)</div>

(Metafield)

</p>
<div class="param">meta (optional)</div>

(Meta)

</div>

</div>


### <a name="Modifier">Modifier </a>

<div class="field-items">

</p>
<div class="param">id (optional)</div>

Integer: The unique numeric ID of the modifier; increments sequentially.

</p>
<div class="param">product_id (optional)</div>

Integer: The unique numeric ID of the product to which the option belongs.

</p>
<div class="param">name (optional)</div>

String: The unique option name. Auto-generated from the display name, a timestamp, and the product ID.

</p>
<div class="param">display_name (optional)</div>

String: The name of the option shown on the storefront.

</p>
<div class="param">type (optional)</div>

String: The type of modifier, which determines how it will display on the storefront. For reference, the former v2 API values are: D = date, C = checkbox, F = file, T = text, MT = multi_line_text, N = numbers_only_text, RB = radio_buttons, RT = rectangles, S = dropdown, P = product_list, PI = product_list_with_images, CS = swatch.

</p>
<div class="param-enum-header">Enum:</div>

<div class="param-enum">date</div>

<div class="param-enum">checkbox</div>

<div class="param-enum">file</div>

<div class="param-enum">text</div>

<div class="param-enum">multi_line_text</div>

<div class="param-enum">numbers_only_text</div>

<div class="param-enum">radio_buttons</div>

<div class="param-enum">rectangles</div>

<div class="param-enum">dropdown</div>

<div class="param-enum">product_list</div>

<div class="param-enum">product_list_with_images</div>

<div class="param-enum">swatch</div>

</p>
<div class="param">required (optional)</div>

Boolean: Whether or not this modifer is required at checkout.

</p>
<div class="param">config (optional)</div>

(OptionConfig)

</p>
<div class="param">option_values (optional)</div>

(array[ModifierValue])

</div>

</div>


### <a name="ModifierCollectionResponse">ModifierCollectionResponse </a>

<div class="model-description">Response payload for the Bigcommerce API.</div>

<div class="field-items">

</p>
<div class="param">data (optional)</div>

(array[Modifier])

</p>
<div class="param">meta (optional)</div>

(CollectionMeta)

</div>

</div>


### <a name="ModifierResponse">ModifierResponse </a>

<div class="model-description">Response payload for the Bigcommerce API.</div>

<div class="field-items">

</p>
<div class="param">data (optional)</div>

(Modifier)

</p>
<div class="param">meta (optional)</div>

(CollectionMeta)

</div>

</div>


### <a name="ModifierValue">ModifierValue </a>

<div class="field-items">

</p>
<div class="param">id (optional)</div>

Integer: The unique numeric ID of the value; increments sequentially.

</p>
<div class="param">is_default (optional)</div>

Boolean: The flag for preselecting a value as the default on the storefront. This field is not supported for swatch options/modifiers.

</p>
<div class="param">label (optional)</div>

String: The text display identifying the value on the storefront.

</p>
<div class="param">sort_order (optional)</div>

Integer: The order in which the value will be displayed on the product page.

</p>
<div class="param">value_data (optional)</div>

Object: Extra data describing the value, based on the type of option or modifier with which the value is associated. `swatch` requires an array of colors, with up to three hexidecimal color keys; `product list` requires a `product_id`; and `checkbox` requires a boolean flag, called `checked_value`, to determine which value is considered to be the checked state.

</p>
<div class="param">adjusters (optional)</div>

(ModifierValue_adjusters)

</div>

</div>


### <a name="ModifierValue_adjusters">ModifierValue_adjusters </a>

<div class="field-items">

</p>
<div class="param">price (optional)</div>

(Adjuster)

</p>
<div class="param">weight (optional)</div>

(Adjuster)

</p>
<div class="param">image_url (optional)</div>

String: The URL for an image displayed on the storefront when the modifier value is selected.

</p>
<div class="param">purchasing_disabled (optional)</div>

(ModifierValue_adjusters_purchasing_disabled)

</div>

</div>


### <a name="ModifierValue_adjusters_purchasing_disabled">ModifierValue_adjusters_purchasing_disabled </a>

<div class="field-items">

</p>
<div class="param">status (optional)</div>

Boolean: Flag for whether the modifier value disables purchasing when selected on the storefront. This can be used for temporarily disabling a particular modifier value.

</p>
<div class="param">message (optional)</div>

String: The message displayed on the storefront when the purchasing disabled status is `true`.

</div>

</div>


### <a name="NotFound">NotFound </a>

<div class="model-description">Error payload for the BigCommerce API.</div>

<div class="field-items">

</p>
<div class="param">status (optional)</div>

Integer: 404 HTTP status code.

</p>
<div class="param">title (optional)</div>

String: The error title describing the particular error.

</p>
<div class="param">type (optional)</div>

String:

</p>
<div class="param">instance (optional)</div>

String:

</div>

</div>


### <a name="Option">Option </a>

<div class="field-items">

</p>
<div class="param">id (optional)</div>

Integer: The unique numeric ID of the option; increments sequentially.

</p>
<div class="param">product_id (optional)</div>

Integer: The unique numeric ID of the product to which the option belongs.

</p>
<div class="param">name (optional)</div>

String: The unique option name, auto-generated from the display name, a timestamp, and the product ID.

</p>
<div class="param">display_name (optional)</div>

String: The name of the option shown on the storefront.

</p>
<div class="param">type (optional)</div>

String: The type of option, which determines how it will display on the storefront. For reference, the former v2 API values are: RB = radio_buttons, RT = rectangles, S = dropdown, P = product_list, PI = product_list_with_images, CS = swatch.

</p>
<div class="param-enum-header">Enum:</div>

<div class="param-enum">radio_buttons</div>

<div class="param-enum">rectangles</div>

<div class="param-enum">dropdown</div>

<div class="param-enum">product_list</div>

<div class="param-enum">product_list_with_images</div>

<div class="param-enum">swatch</div>

</p>
<div class="param">config (optional)</div>

(OptionConfig)

</p>
<div class="param">option_values (optional)</div>

(array[OptionValue])

</div>

</div>


### <a name="OptionCollectionResponse">OptionCollectionResponse </a>

<div class="model-description">Response payload for the Bigcommerce API.</div>

<div class="field-items">

</p>
<div class="param">data (optional)</div>

(array[Option])

</p>
<div class="param">meta (optional)</div>

(CollectionMeta)

</div>

</div>


### <a name="OptionConfig">OptionConfig </a>

<div class="field-items">

</p>
<div class="param">default_value (optional)</div>

String: (date, text, multi_line_text, numbers_only_text) The default value. Shown on a date option as an ISO-8601–formatted string, or on a text option as a string.

</p>
<div class="param">checked_by_default (optional)</div>

Boolean: (checkbox) Flag for setting the checkbox to be checked by default.

</p>
<div class="param">checkbox_label (optional)</div>

String: (checkbox) Label displayed for the checkbox option.

</p>
<div class="param">date_limited (optional)</div>

Boolean: (date) Flag to limit the dates allowed to be entered on a date option.

</p>
<div class="param">date_limit_mode (optional)</div>

String: (date) The type of limit that is allowed to be entered on a date option.

</p>
<div class="param-enum-header">Enum:</div>

<div class="param-enum">earliest</div>

<div class="param-enum">range</div>

<div class="param-enum">latest</div>

</p>
<div class="param">date_earliest_value (optional)</div>

Date (date) The earliest date allowed to be entered on the date option, as an ISO-8601 formatted string. format: date

</p>
<div class="param">date_latest_value (optional)</div>

Date (date) The latest date allowed to be entered on the date option, as an ISO-8601 formatted string. format: date

</p>
<div class="param">file_types_mode (optional)</div>

String: (file) The kind of restriction on the file types that can be uploaded with a file upload option. Values: `specific` - restricts uploads to particular file types; `all` - allows all file types.

</p>
<div class="param-enum-header">Enum:</div>

<div class="param-enum">specific</div>

<div class="param-enum">all</div>

</p>
<div class="param">file_types_supported (optional)</div>

array[String]: (file) The type of files allowed to be uploaded if the `file_type_option` is set to `specific`. Values: `images` - Allows upload of image MIME types (`bmp`,`gif`,`jpg`,`jpeg`,`jpe`,`jif`,`jfif`,`jfi`,`png`,`wbmp`,`xbm`,`tiff`). `documents` - Allows upload of document MIME types (`txt`,`pdf`,`rtf`,`doc`,`docx`,`xls`,`xlsx`,`accdb`,`mdb`,`one`,`pps`,`ppsx`,`ppt`,`pptx`,`pub`,`odt`,`ods`,`odp`,`odg`,`odf`). `other` - Allows file types defined in the `file_types_other` array.

</p>
<div class="param">file_types_other (optional)</div>

array[String]: (file) A list of other file types allowed with the file upload option.

</p>
<div class="param">file_max_size (optional)</div>

Integer: (file) The maximum size for a file that can be used with the file upload option.

</p>
<div class="param">text_characters_limited (optional)</div>

Boolean: (text, multi_line_text) Flag to validate the length of a text or multi-line text input.

</p>
<div class="param">text_min_length (optional)</div>

Integer: (text, multi_line_text) The minimum length allowed for a text or multi-line text option.

</p>
<div class="param">text_max_length (optional)</div>

Integer: (text, multi_line_text) The maximum length allowed for a text or multi line text option.

</p>
<div class="param">text_lines_limited (optional)</div>

Boolean: (multi_line_text) Flag to validate the maximum number of lines allowed on a multi-line text input.

</p>
<div class="param">text_max_lines (optional)</div>

Integer: (multi_line_text) The maximum number of lines allowed on a multi-line text input.

</p>
<div class="param">number_limited (optional)</div>

Boolean: (numbers_only_text) Flag to limit the value of a number option.

</p>
<div class="param">number_limit_mode (optional)</div>

String: (numbers_only_text) The type of limit on values entered for a number option.

</p>
<div class="param-enum-header">Enum:</div>

<div class="param-enum">lowest</div>

<div class="param-enum">highest</div>

<div class="param-enum">range</div>

</p>
<div class="param">number_lowest_value (optional)</div>

(BigDecimal:) (numbers_only_text) The lowest allowed value for a number option if `number_limited` is true.

</p>
<div class="param">number_highest_value (optional)</div>

(BigDecimal:) (numbers_only_text) The highest allowed value for a number option if `number_limited` is true.

</p>
<div class="param">number_integers_only (optional)</div>

Boolean: (numbers_only_text) Flag to limit the input on a number option to whole numbers only.

</p>
<div class="param">product_list_adjusts_inventory (optional)</div>

Boolean: (product_list, product_list_with_images) Flag for automatically adjusting inventory on a product included in the list.

</p>
<div class="param">product_list_adjusts_pricing (optional)</div>

Boolean: (product_list, product_list_with_images) Flag to add the optional product's price to the main product's price.

</p>
<div class="param">product_list_shipping_calc (optional)</div>

String: (product_list, product_list_with_images) How to factor the optional product's weight and package dimensions into the shipping quote. Values: `none` - don't adjust; `weight` - use shipping weight only; `package` - use weight and dimensions.

</p>
<div class="param-enum-header">Enum:</div>

<div class="param-enum">none</div>

<div class="param-enum">weight</div>

<div class="param-enum">package</div>

</div>

</div>


### <a name="OptionResponse">OptionResponse </a>

<div class="model-description">Response payload for the Bigcommerce API.</div>

<div class="field-items">

</p>
<div class="param">data (optional)</div>

[Option)

</p>
<div class="param">meta (optional)</div>

(CollectionMeta)

</div>

</div>


### <a name="OptionValue">OptionValue </a>

<div class="field-items">

</p>
<div class="param">id (optional)</div>

Integer: The unique numeric ID of the value; increments sequentially.

</p>
<div class="param">is_default (optional)</div>

Boolean: The flag for preselecting a value as the default on the storefront. This field is not supported for swatch options/modifiers.

</p>
<div class="param">label (optional)</div>

String: The text display identifying the value on the storefront.

</p>
<div class="param">sort_order (optional)</div>

Integer: The order in which the value will be displayed on the product page.

</p>
<div class="param">value_data (optional)</div>

Object: Extra data describing the value, based on the type of option or modifier with which the value is associated. `swatch` requires an array of colors, with up to three hexidecimal color keys; `product list` requires a `product_id`; and `checkbox` requires a boolean flag, called `checked_value`, to determine which value is considered to be the checked state.

</div>

</div>


### <a name="OptionValueShort">OptionValueShort </a>

<div class="field-items">

</p>
<div class="param">id (optional)</div>

Integer:

</p>
<div class="param">option_id (optional)</div>

Integer:

</p>
<div class="param">option_display_name (optional)</div>

String: The name of the option.

</p>
<div class="param">label (optional)</div>

String: The label of the option value.

</div>

</div>


### <a name="OptionValueShortPost">OptionValueShortPost </a>

<div class="field-items">

</p>
<div class="param">option_display_name (optional)</div>

String: The name of the option to be created on POST.

</p>
<div class="param">label (optional)</div>

String: The label of the option value to be created on POST.

</div>

</div>


### <a name="Product">Product </a>

<div class="model-description">A BigCommerce Product object describes a single purchasable unit or a collection of purchasable units.</div>

<div class="field-items">

</p>
<div class="param">id (optional)</div>

Integer: The unique numeric ID of the product; increments sequentially.

</p>
<div class="param">name (optional)</div>

String: The product name.

</p>
<div class="param">type (optional)</div>

String: The product type: physical - a physical stock unit; digital - a digital download.

</p>
<div class="param-enum-header">Enum:</div>

<div class="param-enum">physical</div>

<div class="param-enum">digital</div>

</p>
<div class="param">sku (optional)</div>

String: User-defined product code/stock keeping unit (SKU).

</p>
<div class="param">description (optional)</div>

String: The product description, which can include HTML formatting.

</p>
<div class="param">weight (optional)</div>

Double: Weight of the product, which can be used when calculating shipping costs. format: double

</p>
<div class="param">width (optional)</div>

Double: Width of the product, which can be used when calculating shipping costs. format: double

</p>
<div class="param">depth (optional)</div>

Double: Depth of the product, which can be used when calculating shipping costs. format: double

</p>
<div class="param">height (optional)</div>

Double: Height of the product, which can be used when calculating shipping costs. format: double

</p>
<div class="param">price (optional)</div>

Double: The price of the product. The price should include or exclude tax, based on the store settings. format: double

</p>
<div class="param">cost_price (optional)</div>

Double: The cost price of the product. Stored for reference only; it is not used or displayed anywhere on the store. format: double

</p>
<div class="param">retail_price (optional)</div>

Double: The retail cost of the product. If entered, the retail cost price will be shown on the product page. format: double

</p>
<div class="param">sale_price (optional)</div>

Double: If entered, the sale price will be used instead of value in the price field when calculating the product's cost. format: double

</p>
<div class="param">tax_class_id (optional)</div>

Integer: The ID of the tax class applied to the product. (NOTE: Value ignored if automatic tax is enabled.)

</p>
<div class="param">product_tax_code (optional)</div>

String: Accepts AvaTax System Tax Codes, which identify products and services that fall into special sales-tax categories. By using these codes, merchants who subscribe to Avalara Premium can calculate sales taxes more accurately. Stores without Avalara Premium will ignore the code when calculating sales tax. Do not pass more than one code. The codes are case-sensitive. For details, please see Avalara's documentation.

</p>
<div class="param">calculated_price (optional)</div>

Double: The price of the product, unless a `sale_price` is set. format: double

</p>
<div class="param">categories (optional)</div>

(array[Integer]) An array of IDs for the categories to which this product belongs. When updating a product, if an array of categories is supplied, all product categories will be overwritten. Does not accept more than 1,000 ID values.

</p>
<div class="param">brand_id (optional)</div>

Integer: The ID associated with the product's brand.

</p>
<div class="param">inventory_level (optional)</div>

Integer: Current inventory level of the product. Simple inventory tracking must be enabled (See the inventory_tracking field) for this to take any effect.

</p>
<div class="param">inventory_warning_level (optional)</div>

Integer: Inventory Warning level for the product. When the product's inventory level drops below the warning level, the store owner will be informed. Simple inventory tracking must be enabled (see the `inventory_tracking` field) for this to take any effect.

</p>
<div class="param">inventory_tracking (optional)</div>

String: The type of inventory tracking for the product. Values are: none - inventory levels will not be tracked; product - inventory levels will be tracked using the `inventory_level` and `inventory_warning_level` fields; variant - inventory levels will be tracked based on variants, which maintain their own warning levels and inventory levels.

</p>
<div class="param-enum-header">Enum:</div>

<div class="param-enum">none</div>

<div class="param-enum">product</div>

<div class="param-enum">variant</div>

</p>
<div class="param">fixed_cost_shipping_price (optional)</div>

Integer: A fixed shipping cost for the product. If defined, this value will be used during checkout instead of normal shipping-cost calculation.

</p>
<div class="param">is_free_shipping (optional)</div>

Boolean: Flag used to indicate whether the product has free shipping. If `true`, the shipping cost for the product will be zero.

</p>
<div class="param">is_visible (optional)</div>

Boolean: Flag to determine whether the product should be displayed to customers browsing the store. If `true`, the product will be displayed. If `false`, the product will be hidden from view.

</p>
<div class="param">is_featured (optional)</div>

Boolean: Flag to determine whether the product should be included in the `featured products` panel when viewing the store.

</p>
<div class="param">warranty (optional)</div>

String: Warranty information displayed on the product page. Can include HTML formatting.

</p>
<div class="param">bin_picking_number (optional)</div>

String: The BIN picking number for the product.

</p>
<div class="param">layout_file (optional)</div>

String: The layout template file used to render this product.

</p>
<div class="param">upc (optional)</div>

String: The product UPC code, which is used in feeds for shopping comparison sites and external channel integrations.

</p>
<div class="param">search_keywords (optional)</div>

String: A comma-separated list of keywords that can be used to locate the product when searching the store.

</p>
<div class="param">availability (optional)</div>

String: Availability of the product. Availability options are: available - the product can be purchased in the storefront; disabled - the product is listed in the storefront, but cannot be purchased; preorder - the product is listed for pre-orders.

</p>
<div class="param-enum-header">Enum:</div>

<div class="param-enum">available</div>

<div class="param-enum">disabled</div>

<div class="param-enum">preorder</div>

</p>
<div class="param">availability_description (optional)</div>

String: Availability text displayed on the checkout page, under the product title. Tells the customer how long it will normally take to ship this product, such as 'Usually ships in 24 hours.'

</p>
<div class="param">gift_wrapping_options_type (optional)</div>

String: Type of gift-wrapping options. Values: `any` - allow any gift-wrapping options in the store; `none` - disallow gift wrapping on the product; `list` – provide a list of IDs in the `gift_wrapping_options_list` field.

</p>
<div class="param-enum-header">Enum:</div>

<div class="param-enum">any</div>

<div class="param-enum">none</div>

<div class="param-enum">list</div>

</p>
<div class="param">gift_wrapping_options_list (optional)</div>

array[Integer]: A list of gift-wrapping option IDs.

</p>
<div class="param">sort_order (optional)</div>

Integer: Priority to give this product when included in product lists on category pages and in search results. Lower integers will place the product closer to the top of the results.

</p>
<div class="param">condition (optional)</div>

String: The product condition. Will be shown on the product page if the `is_condition_shown` field's value is `true`. Possible values: `New`, `Used`, `Refurbished`.

</p>
<div class="param">is_condition_shown (optional)</div>

Boolean: Flag used to determine whether the product condition is shown to the customer on the product page.

</p>
<div class="param">order_quantity_minimum (optional)</div>

Integer: The minimum quantity an order must contain, to be eligible to purchase this product.

</p>
<div class="param">order_quantity_maximum (optional)</div>

Integer: The maximum quantity an order can contain when purchasing the product.

</p>
<div class="param">page_title (optional)</div>

String: Custom title for the product page. If not defined, the product name will be used as the meta title.

</p>
<div class="param">meta_keywords (optional)</div>

array[String]: Custom meta keywords for the product page. If not defined, the store's default keywords will be used.

</p>
<div class="param">meta_description (optional)</div>

String: Custom meta description for the product page. If not defined, the store's default meta description will be used.

</p>
<div class="param">date_created (optional)</div>

String: The date on which the product was created. format: data-time

</p>
<div class="param">date_modified (optional)</div>

String: The date on which the product was modified. format: data-time

</p>
<div class="param">view_count (optional)</div>

Integer: The number of times the product has been viewed.

</p>
<div class="param">preorder_release_date (optional)</div>

String: Pre-order release date. See the `availability` field for details on setting a product's availability to accept pre-orders. format: data-time

</p>
<div class="param">preorder_message (optional)</div>

String: Custom expected-date message to display on the product page. If undefined, the message defaults to the storewide setting. Can contain the `%%DATE%%` placeholder, which will be substituted for the release date.

</p>
<div class="param">is_preorder_only (optional)</div>

Boolean: If set to `false`, the product will not change its availability from `preorder` to `available` on the release date. Otherwise, on the release date the product's availability/status will change to `available`.

</p>
<div class="param">is_price_hidden (optional)</div>

Boolean: False by default, indicating that this product's price should be shown on the product page. If set to `true`, the price is hidden. (NOTE: To successfully set `is_price_hidden` to `true`, the `availability` value must be `disabled`.)

</p>
<div class="param">price_hidden_label (optional)</div>

String: By default, an empty string. If `is_price_hidden` is `true`, the value of `price_hidden_label` is displayed instead of the price. (NOTE: To successfully set a non-empty string value with `is_price_hidden` set to `true`, the `availability` value must be `disabled`.)

</p>
<div class="param">images (optional)</div>

(array[ProductImage])

</p>
<div class="param">videos (optional)</div>

(array[ProductVideo])

</p>
<div class="param">custom_fields (optional)</div>

(array[CustomField])

</p>
<div class="param">custom_url (optional)</div>

(CustomUrl)

</p>
<div class="param">bulk_pricing_rules (optional)</div>

(array[BulkPricingRule])

</p>
<div class="param">variants (optional)</div>

(array[Variant])

</div>

</div>


### <a name="ProductCollectionResponse">ProductCollectionResponse </a>

<div class="model-description">Response payload for the Bigcommerce API.</div>

<div class="field-items">

</p>
<div class="param">data (optional)</div>

(array[Product])

</p>
<div class="param">meta (optional)</div>

(CollectionMeta)

</div>

</div>


### <a name="ProductImage">ProductImage </a>

<div class="model-description">The full ProductImage model.</div>

<div class="field-items">

</p>
<div class="param">is_thumbnail (optional)</div>

Boolean: Flag for identifying whether the image is used as the product's thumbnail.

</p>
<div class="param">sort_order (optional)</div>

Integer: The order in which the image will be displayed on the product page. Higher integers give the image a lower priority. When updating, if the image is given a lower priority, all images with a `sort_order` the same as or greater than the image's new `sort_order` value will have their `sort_order`s reordered.

</p>
<div class="param">description (optional)</div>

String: The description for the image.

</p>
<div class="param">id (optional)</div>

Integer: The unique numeric ID of the image; increments sequentially.

</p>
<div class="param">product_id (optional)</div>

Integer: The unique numeric identifier for the product with which the image is associated.

</p>
<div class="param">image_file (optional)</div>

String: The local path to the original image file uploaded to BigCommerce.

</p>
<div class="param">url_zoom (optional)</div>

String: The zoom URL for this image. By default, this is used as the zoom image on product pages when zoom images are enabled.

</p>
<div class="param">url_standard (optional)</div>

String: The standard URL for this image. By default, this is used for product-page images.

</p>
<div class="param">url_thumbnail (optional)</div>

String: The thumbnail URL for this image. By default, this is the image size used on the category page and in side panels.

</p>
<div class="param">url_tiny (optional)</div>

String: The tiny URL for this image. By default, this is the image size used for thumbnails beneath the product image on a product page.

</div>

</div>


### <a name="ProductImageBase">ProductImageBase </a>

<div class="model-description">Common ProductImage properties.</div>

<div class="field-items">

</p>
<div class="param">is_thumbnail (optional)</div>

Boolean: Flag for identifying whether the image is used as the product's thumbnail.

</p>
<div class="param">sort_order (optional)</div>

Integer: The order in which the image will be displayed on the product page. Higher integers give the image a lower priority. When updating, if the image is given a lower priority, all images with a `sort_order` the same as or greater than the image's new `sort_order` value will have their `sort_order`s reordered.

</p>
<div class="param">description (optional)</div>

String: The description for the image.

</div>

</div>


### <a name="ProductImageCollectionResponse">ProductImageCollectionResponse </a>

<div class="model-description">Response payload for the Bigcommerce API.</div>

<div class="field-items">

</p>
<div class="param">data (optional)</div>

(array[ProductImage])

</p>
<div class="param">meta (optional)</div>

(CollectionMeta)

</div>

</div>


### <a name="ProductImagePost">ProductImagePost </a>

<div class="model-description">The model for a POST to create an image on a product.</div>

<div class="field-items">

</p>
<div class="param">is_thumbnail (optional)</div>

Boolean: Flag for identifying whether the image is used as the product's thumbnail.

</p>
<div class="param">sort_order (optional)</div>

Integer: The order in which the image will be displayed on the product page. Higher integers give the image a lower priority. When updating, if the image is given a lower priority, all images with a `sort_order` the same as or greater than the image's new `sort_order` value will have their `sort_order`s reordered.

</p>
<div class="param">description (optional)</div>

String: The description for the image.

</p>
<div class="param">image_url (optional)</div>

String: Must be a fully qualified URL path, including protocol.

</p>
<div class="param">image_file (optional)</div>

String: Must be sent as a multipart/form-data field in the request body.

</div>

</div>


### <a name="ProductImagePut">ProductImagePut </a>

<div class="model-description">The model for a PUT to update applicable ProductImage fields.</div>

<div class="field-items">

</p>
<div class="param">is_thumbnail (optional)</div>

Boolean: Flag for identifying whether the image is used as the product's thumbnail.

</p>
<div class="param">sort_order (optional)</div>

Integer: The order in which the image will be displayed on the product page. Higher integers give the image a lower priority. When updating, if the image is given a lower priority, all images with a `sort_order` the same as or greater than the image's new `sort_order` value will have their `sort_order`s reordered.

</p>
<div class="param">description (optional)</div>

String: The description for the image.

</div>

</div>


### <a name="ProductImageResponse">ProductImageResponse </a>

<div class="model-description">Response payload for the Bigcommerce API.</div>

<div class="field-items">

</p>
<div class="param">data (optional)</div>

(ProductImage)

</p>
<div class="param">meta (optional)</div>

(Meta)

</div>

</div>


### <a name="ProductPost">ProductPost </a>

<div class="field-items">

</p>
<div class="param">id (optional)</div>

Integer: The unique numeric ID of the product; increments sequentially.

</p>
<div class="param">name (optional)</div>

String: The product name.

</p>
<div class="param">type (optional)</div>

String: The product type: physical - a physical stock unit; digital - a digital download.

</p>
<div class="param-enum-header">Enum:</div>

<div class="param-enum">physical</div>

<div class="param-enum">digital</div>

</p>
<div class="param">sku (optional)</div>

String: User-defined product code/stock keeping unit (SKU).

</p>
<div class="param">description (optional)</div>

String: The product description, which can include HTML formatting.

</p>
<div class="param">weight (optional)</div>

Double: Weight of the product, which can be used when calculating shipping costs. format: double

</p>
<div class="param">width (optional)</div>

Double: Width of the product, which can be used when calculating shipping costs. format: double

</p>
<div class="param">depth (optional)</div>

Double: Depth of the product, which can be used when calculating shipping costs. format: double

</p>
<div class="param">height (optional)</div>

Double: Height of the product, which can be used when calculating shipping costs. format: double

</p>
<div class="param">price (optional)</div>

Double: The price of the product. The price should include or exclude tax, based on the store settings. format: double

</p>
<div class="param">cost_price (optional)</div>

Double: The cost price of the product. Stored for reference only; it is not used or displayed anywhere on the store. format: double

</p>
<div class="param">retail_price (optional)</div>

Double: The retail cost of the product. If entered, the retail cost price will be shown on the product page. format: double

</p>
<div class="param">sale_price (optional)</div>

Double: If entered, the sale price will be used instead of value in the price field when calculating the product's cost. format: double

</p>
<div class="param">tax_class_id (optional)</div>

Integer: The ID of the tax class applied to the product. (NOTE: Value ignored if automatic tax is enabled.)

</p>
<div class="param">product_tax_code (optional)</div>

String: Accepts AvaTax System Tax Codes, which identify products and services that fall into special sales-tax categories. By using these codes, merchants who subscribe to Avalara Premium can calculate sales taxes more accurately. Stores without Avalara Premium will ignore the code when calculating sales tax. Do not pass more than one code. The codes are case-sensitive. For details, please see Avalara's documentation.

</p>
<div class="param">calculated_price (optional)</div>

Double: The price of the product, unless a `sale_price` is set. format: double

</p>
<div class="param">categories (optional)</div>

array[Integer]: An array of IDs for the categories to which this product belongs. When updating a product, if an array of categories is supplied, all product categories will be overwritten. Does not accept more than 1,000 ID values.

</p>
<div class="param">brand_id (optional)</div>

Integer: The ID associated with the product's brand.

</p>
<div class="param">inventory_level (optional)</div>

Integer: Current inventory level of the product. Simple inventory tracking must be enabled (See the inventory_tracking field) for this to take any effect.

</p>
<div class="param">inventory_warning_level (optional)</div>

Integer: Inventory Warning level for the product. When the product's inventory level drops below the warning level, the store owner will be informed. Simple inventory tracking must be enabled (see the `inventory_tracking` field) for this to take any effect.

</p>
<div class="param">inventory_tracking (optional)</div>

String: The type of inventory tracking for the product. Values are: none - inventory levels will not be tracked; product - inventory levels will be tracked using the `inventory_level` and `inventory_warning_level` fields; variant - inventory levels will be tracked based on variants, which maintain their own warning levels and inventory levels.

</p>
<div class="param-enum-header">Enum:</div>

<div class="param-enum">none</div>

<div class="param-enum">product</div>

<div class="param-enum">variant</div>

</p>
<div class="param">fixed_cost_shipping_price (optional)</div>

Integer: A fixed shipping cost for the product. If defined, this value will be used during checkout instead of normal shipping-cost calculation.

</p>
<div class="param">is_free_shipping (optional)</div>

Boolean: Flag used to indicate whether the product has free shipping. If `true`, the shipping cost for the product will be zero.

</p>
<div class="param">is_visible (optional)</div>

Boolean: Flag to determine whether the product should be displayed to customers browsing the store. If `true`, the product will be displayed. If `false`, the product will be hidden from view.

</p>
<div class="param">is_featured (optional)</div>

Boolean: Flag to determine whether the product should be included in the `featured products` panel when viewing the store.

</p>
<div class="param">warranty (optional)</div>

String: Warranty information displayed on the product page. Can include HTML formatting.

</p>
<div class="param">bin_picking_number (optional)</div>

String: The BIN picking number for the product.

</p>
<div class="param">layout_file (optional)</div>

String: The layout template file used to render this product.

</p>
<div class="param">upc (optional)</div>

String: The product UPC code, which is used in feeds for shopping comparison sites and external channel integrations.

</p>
<div class="param">search_keywords (optional)</div>

String: A comma-separated list of keywords that can be used to locate the product when searching the store.

</p>
<div class="param">availability (optional)</div>

String: Availability of the product. Availability options are: available - the product can be purchased in the storefront; disabled - the product is listed in the storefront, but cannot be purchased; preorder - the product is listed for pre-orders.

</p>
<div class="param-enum-header">Enum:</div>

<div class="param-enum">available</div>

<div class="param-enum">disabled</div>

<div class="param-enum">preorder</div>

</p>
<div class="param">availability_description (optional)</div>

String: Availability text displayed on the checkout page, under the product title. Tells the customer how long it will normally take to ship this product, such as 'Usually ships in 24 hours.'

</p>
<div class="param">gift_wrapping_options_type (optional)</div>

String: Type of gift-wrapping options. Values: `any` - allow any gift-wrapping options in the store; `none` - disallow gift wrapping on the product; `list` – provide a list of IDs in the `gift_wrapping_options_list` field.

</p>
<div class="param-enum-header">Enum:</div>

<div class="param-enum">any</div>

<div class="param-enum">none</div>

<div class="param-enum">list</div>

</p>
<div class="param">gift_wrapping_options_list (optional)</div>

array[Integer]: A list of gift-wrapping option IDs.

</p>
<div class="param">sort_order (optional)</div>

Integer: Priority to give this product when included in product lists on category pages and in search results. Lower integers will place the product closer to the top of the results.

</p>
<div class="param">condition (optional)</div>

String: The product condition. Will be shown on the product page if the `is_condition_shown` field's value is `true`. Possible values: `New`, `Used`, `Refurbished`.

</p>
<div class="param">is_condition_shown (optional)</div>

Boolean: Flag used to determine whether the product condition is shown to the customer on the product page.

</p>
<div class="param">order_quantity_minimum (optional)</div>

Integer: The minimum quantity an order must contain, to be eligible to purchase this product.

</p>
<div class="param">order_quantity_maximum (optional)</div>

Integer: The maximum quantity an order can contain when purchasing the product.

</p>
<div class="param">page_title (optional)</div>

String: Custom title for the product page. If not defined, the product name will be used as the meta title.

</p>
<div class="param">meta_keywords (optional)</div>

array[String]: Custom meta keywords for the product page. If not defined, the store's default keywords will be used.

</p>
<div class="param">meta_description (optional)</div>

String: Custom meta description for the product page. If not defined, the store's default meta description will be used.

</p>
<div class="param">date_created (optional)</div>

String: The date on which the product was created. format: data-time

</p>
<div class="param">date_modified (optional)</div>

String: The date on which the product was modified. format: data-time

</p>
<div class="param">view_count (optional)</div>

Integer: The number of times the product has been viewed.

</p>
<div class="param">preorder_release_date (optional)</div>

String: Pre-order release date. See the `availability` field for details on setting a product's availability to accept pre-orders. format: data-time

</p>
<div class="param">preorder_message (optional)</div>

String: Custom expected-date message to display on the product page. If undefined, the message defaults to the storewide setting. Can contain the `%%DATE%%` placeholder, which will be substituted for the release date.

</p>
<div class="param">is_preorder_only (optional)</div>

Boolean: If set to `false`, the product will not change its availability from `preorder` to `available` on the release date. Otherwise, on the release date the product's availability/status will change to `available`.

</p>
<div class="param">is_price_hidden (optional)</div>

Boolean: False by default, indicating that this product's price should be shown on the product page. If set to `true`, the price is hidden. (NOTE: To successfully set `is_price_hidden` to `true`, the `availability` value must be `disabled`.)

</p>
<div class="param">price_hidden_label (optional)</div>

String: By default, an empty string. If `is_price_hidden` is `true`, the value of `price_hidden_label` is displayed instead of the price. (NOTE: To successfully set a non-empty string value with `is_price_hidden` set to `true`, the `availability` value must be `disabled`.)

</p>
<div class="param">images (optional)</div>

(array[ProductImage])

</p>
<div class="param">videos (optional)</div>

(array[ProductVideo])

</p>
<div class="param">custom_fields (optional)</div>

(array[CustomField])

</p>
<div class="param">custom_url (optional)</div>

(CustomUrl)

</p>
<div class="param">bulk_pricing_rules (optional)</div>

(array[BulkPricingRule])

</p>
<div class="param">variants (optional)</div>

(array[VariantPost])

</div>

</div>


### <a name="ProductPut">ProductPut </a>

<div class="field-items">

</p>
<div class="param">id (optional)</div>

Integer: The unique numeric ID of the product; increments sequentially.

</p>
<div class="param">name (optional)</div>

String: The product name.

</p>
<div class="param">type (optional)</div>

String: The product type: physical - a physical stock unit; digital - a digital download.

</p>
<div class="param-enum-header">Enum:</div>

<div class="param-enum">physical</div>

<div class="param-enum">digital</div>

</p>
<div class="param">sku (optional)</div>

String: User-defined product code/stock keeping unit (SKU).

</p>
<div class="param">description (optional)</div>

String: The product description, which can include HTML formatting.

</p>
<div class="param">weight (optional)</div>

Double: Weight of the product, which can be used when calculating shipping costs. format: double

</p>
<div class="param">width (optional)</div>

Double: Width of the product, which can be used when calculating shipping costs. format: double

</p>
<div class="param">depth (optional)</div>

Double: Depth of the product, which can be used when calculating shipping costs. format: double

</p>
<div class="param">height (optional)</div>

Double: Height of the product, which can be used when calculating shipping costs. format: double

</p>
<div class="param">price (optional)</div>

Double: The price of the product. The price should include or exclude tax, based on the store settings. format: double

</p>
<div class="param">cost_price (optional)</div>

Double: The cost price of the product. Stored for reference only; it is not used or displayed anywhere on the store. format: double

</p>
<div class="param">retail_price (optional)</div>

Double: The retail cost of the product. If entered, the retail cost price will be shown on the product page. format: double

</p>
<div class="param">sale_price (optional)</div>

Double: If entered, the sale price will be used instead of value in the price field when calculating the product's cost. format: double

</p>
<div class="param">tax_class_id (optional)</div>

Integer: The ID of the tax class applied to the product. (NOTE: Value ignored if automatic tax is enabled.)

</p>
<div class="param">product_tax_code (optional)</div>

String: Accepts AvaTax System Tax Codes, which identify products and services that fall into special sales-tax categories. By using these codes, merchants who subscribe to Avalara Premium can calculate sales taxes more accurately. Stores without Avalara Premium will ignore the code when calculating sales tax. Do not pass more than one code. The codes are case-sensitive. For details, please see Avalara's documentation.

</p>
<div class="param">calculated_price (optional)</div>

Double: The price of the product, unless a `sale_price` is set. format: double

</p>
<div class="param">categories (optional)</div>

array[Integer]: An array of IDs for the categories to which this product belongs. When updating a product, if an array of categories is supplied, all product categories will be overwritten. Does not accept more than 1,000 ID values.

</p>
<div class="param">brand_id (optional)</div>

Integer: The ID associated with the product's brand.

</p>
<div class="param">inventory_level (optional)</div>

Integer: Current inventory level of the product. Simple inventory tracking must be enabled (See the inventory_tracking field) for this to take any effect.

</p>
<div class="param">inventory_warning_level (optional)</div>

Integer: Inventory Warning level for the product. When the product's inventory level drops below the warning level, the store owner will be informed. Simple inventory tracking must be enabled (see the `inventory_tracking` field) for this to take any effect.

</p>
<div class="param">inventory_tracking (optional)</div>

String: The type of inventory tracking for the product. Values are: none - inventory levels will not be tracked; product - inventory levels will be tracked using the `inventory_level` and `inventory_warning_level` fields; variant - inventory levels will be tracked based on variants, which maintain their own warning levels and inventory levels.

</p>
<div class="param-enum-header">Enum:</div>

<div class="param-enum">none</div>

<div class="param-enum">product</div>

<div class="param-enum">variant</div>

</p>
<div class="param">fixed_cost_shipping_price (optional)</div>

Integer: A fixed shipping cost for the product. If defined, this value will be used during checkout instead of normal shipping-cost calculation.

</p>
<div class="param">is_free_shipping (optional)</div>

Boolean: Flag used to indicate whether the product has free shipping. If `true`, the shipping cost for the product will be zero.

</p>
<div class="param">is_visible (optional)</div>

Boolean: Flag to determine whether the product should be displayed to customers browsing the store. If `true`, the product will be displayed. If `false`, the product will be hidden from view.

</p>
<div class="param">is_featured (optional)</div>

Boolean: Flag to determine whether the product should be included in the `featured products` panel when viewing the store.

</p>
<div class="param">warranty (optional)</div>

String: Warranty information displayed on the product page. Can include HTML formatting.

</p>
<div class="param">bin_picking_number (optional)</div>

String: The BIN picking number for the product.

</p>
<div class="param">layout_file (optional)</div>

String: The layout template file used to render this product.

</p>
<div class="param">upc (optional)</div>

String: The product UPC code, which is used in feeds for shopping comparison sites and external channel integrations.

</p>
<div class="param">search_keywords (optional)</div>

String: A comma-separated list of keywords that can be used to locate the product when searching the store.

</p>
<div class="param">availability (optional)</div>

String: Availability of the product. Availability options are: available - the product can be purchased in the storefront; disabled - the product is listed in the storefront, but cannot be purchased; preorder - the product is listed for pre-orders.

</p>
<div class="param-enum-header">Enum:</div>

<div class="param-enum">available</div>

<div class="param-enum">disabled</div>

<div class="param-enum">preorder</div>

</p>
<div class="param">availability_description (optional)</div>

String: Availability text displayed on the checkout page, under the product title. Tells the customer how long it will normally take to ship this product, such as 'Usually ships in 24 hours.'

</p>
<div class="param">gift_wrapping_options_type (optional)</div>

String: Type of gift-wrapping options. Values: `any` - allow any gift-wrapping options in the store; `none` - disallow gift wrapping on the product; `list` – provide a list of IDs in the `gift_wrapping_options_list` field.

</p>
<div class="param-enum-header">Enum:</div>

<div class="param-enum">any</div>

<div class="param-enum">none</div>

<div class="param-enum">list</div>

</p>
<div class="param">gift_wrapping_options_list (optional)</div>

array[Integer]: A list of gift-wrapping option IDs.

</p>
<div class="param">sort_order (optional)</div>

Integer: Priority to give this product when included in product lists on category pages and in search results. Lower integers will place the product closer to the top of the results.

</p>
<div class="param">condition (optional)</div>

String: The product condition. Will be shown on the product page if the `is_condition_shown` field's value is `true`. Possible values: `New`, `Used`, `Refurbished`.

</p>
<div class="param">is_condition_shown (optional)</div>

Boolean: Flag used to determine whether the product condition is shown to the customer on the product page.

</p>
<div class="param">order_quantity_minimum (optional)</div>

Integer: The minimum quantity an order must contain, to be eligible to purchase this product.

</p>
<div class="param">order_quantity_maximum (optional)</div>

Integer: The maximum quantity an order can contain when purchasing the product.

</p>
<div class="param">page_title (optional)</div>

String: Custom title for the product page. If not defined, the product name will be used as the meta title.

</p>
<div class="param">meta_keywords (optional)</div>

array[String]: Custom meta keywords for the product page. If not defined, the store's default keywords will be used.

</p>
<div class="param">meta_description (optional)</div>

String: Custom meta description for the product page. If not defined, the store's default meta description will be used.

</p>
<div class="param">date_created (optional)</div>

String: The date on which the product was created. format: data-time

</p>
<div class="param">date_modified (optional)</div>

String: The date on which the product was modified. format: data-time

</p>
<div class="param">view_count (optional)</div>

Integer: The number of times the product has been viewed.

</p>
<div class="param">preorder_release_date (optional)</div>

String: Pre-order release date. See the `availability` field for details on setting a product's availability to accept pre-orders. format: data-time

</p>
<div class="param">preorder_message (optional)</div>

String: Custom expected-date message to display on the product page. If undefined, the message defaults to the storewide setting. Can contain the `%%DATE%%` placeholder, which will be substituted for the release date.

</p>
<div class="param">is_preorder_only (optional)</div>

Boolean: If set to `false`, the product will not change its availability from `preorder` to `available` on the release date. Otherwise, on the release date the product's availability/status will change to `available`.

</p>
<div class="param">is_price_hidden (optional)</div>

Boolean: False by default, indicating that this product's price should be shown on the product page. If set to `true`, the price is hidden. (NOTE: To successfully set `is_price_hidden` to `true`, the `availability` value must be `disabled`.)

</p>
<div class="param">price_hidden_label (optional)</div>

String: By default, an empty string. If `is_price_hidden` is `true`, the value of `price_hidden_label` is displayed instead of the price. (NOTE: To successfully set a non-empty string value with `is_price_hidden` set to `true`, the `availability` value must be `disabled`.)

</p>
<div class="param">images (optional)</div>

(array[ProductImage])

</p>
<div class="param">videos (optional)</div>

(array[ProductVideo])

</p>
<div class="param">custom_fields (optional)</div>

(array[CustomField])

</p>
<div class="param">custom_url (optional)</div>

(CustomUrl))

</p>
<div class="param">bulk_pricing_rules (optional)</div>

(array[BulkPricingRule])

</p>
<div class="param">variants (optional)</div>

(array[Variant])

</div>

</div>(Meta)


### <a name="ProductResponse">ProductResponse </a>

<div class="model-description">Response payload for the Bigcommerce API.</div>

<div class="field-items">

</p>
<div class="param">data (optional)</div>

(Product)

</p>
<div class="param">meta (optional)</div>

(Meta)

</div>

</div>


### <a name="ProductVideo">ProductVideo </a>

<div class="model-description">A product video model.</div>

<div class="field-items">

</p>
<div class="param">id (optional)</div>

Integer: The ID of a YouTube video.

</p>
<div class="param">product_id (optional)</div>

Integer: The unique numeric identifier for the product with which the image is associated.

</p>
<div class="param">sort_order (optional)</div>

Integer: The order in which the video will be displayed on the product page. Higher integers give the video a lower priority. When updating, if the video is given a lower priority, all videos with a `sort_order` the same as or greater than the video's new `sort_order` value will have their `sort_order`s reordered.

</p>
<div class="param">description (optional)</div>

String: The description for the video. If left blank, this will be filled in according to data on YouTube.

</p>
<div class="param">title (optional)</div>

String: The title for the video. If left blank, this will be filled in according to data on YouTube.

</p>
<div class="param">length (optional)</div>

String: Length of the video. This will be filled in according to data on YouTube.

</div>

</div>


### <a name="ProductVideoCollectionResponse">ProductVideoCollectionResponse </a>

<div class="model-description">Response payload for the Bigcommerce API.</div>

<div class="field-items">

</p>
<div class="param">data (optional)</div>

(array[ProductVideo])

</p>
<div class="param">meta (optional)</div>

(CollectionMeta)

</div>

</div>


### <a name="ProductVideoResponse">ProductVideoResponse </a>

<div class="model-description">Response payload for the Bigcommerce API.</div>

<div class="field-items">

</p>
<div class="param">data (optional)</div>

(ProductVideo)

</p>
<div class="param">meta (optional)</div>

(CollectionMeta)

</div>

</div>


### <a name="ResourceImage">ResourceImage </a>

<div class="model-description">An object containing a publicly accessible image URL, or a form post that contains an image file.</div>

<div class="field-items">

</p>
<div class="param">image_url (optional)</div>

String: A public URL for a GIF, JPEG, or PNG image.

</div>

</div>


### <a name="Subscriber">Subscriber </a>

<div class="field-items">

</p>
<div class="param">id (optional)</div>

Integer: The unique numeric ID of the subscriber; increments sequentially.

</p>
<div class="param">email (optional)</div>

String: The email of the subscriber. Must be unique.

</p>
<div class="param">first_name (optional)</div>

String: The first name of the subscriber.

</p>
<div class="param">last_name (optional)</div>

String: The last name of the subscriber.

</p>
<div class="param">source (optional)</div>

String: The source of the subscriber. Values are: `storefront`, `order`, or `custom`.

</p>
<div class="param">order_id (optional)</div>

Integer: The ID of the source order, if source was an order.

</p>
<div class="param">date_modified (optional)</div>

String: The date on which the subscriber was modified. format: data-time

</p>
<div class="param">date_created (optional)</div>

String: The date on which the subscriber was created. format: data-time

</div>

</div>


### <a name="SubscriberCollectionResponse">SubscriberCollectionResponse </a>

<div class="model-description">Response payload for the Bigcommerce API.</div>

<div class="field-items">

</p>
<div class="param">data (optional)</div>

(array[Subscriber])

</p>
<div class="param">meta (optional)</div>

(CollectionMeta)

</div>

</div>


### <a name="SubscriberResponse">SubscriberResponse </a>

<div class="model-description">Response payload for the Bigcommerce API.</div>

<div class="field-items">

</p>
<div class="param">data (optional)</div>

(Subscriber)

</p>
<div class="param">meta (optional)</div>

(Meta)

</div>

</div>


### <a name="Variant">Variant </a>

<div class="field-items">

</p>
<div class="param">id (optional)</div>

Integer:

</p>
<div class="param">product_id (optional)</div>

Integer:

</p>
<div class="param">sku (optional)</div>

String:

</p>
<div class="param">sku_id (optional)</div>

Integer: Read-only reference to v2 API's SKU ID. Null if it is a base variant.

</p>
<div class="param">cost_price (optional)</div>

Double: The cost price of the variant. format: double

</p>
<div class="param">price (optional)</div>

Double: This variant's base price on the storefront. If this value is null, the product's default price (set in the Product resource's `price` field) will be used as the base price. format: double

</p>
<div class="param">weight (optional)</div>

Double: This variant's base weight on the storefront. If this value is null, the product's default weight (set in the Product resource's `weight` field) will be used as the base weight. format: double

</p>
<div class="param">purchasing_disabled (optional)</div>

Boolean: If `true`, this variant will not be purchasable on the storefront.

</p>
<div class="param">purchasing_disabled_message (optional)</div>

String: If `purchasing_disabled` is `true`, this message should show on the storefront when the variant is selected.

</p>
<div class="param">image_url (optional)</div>

String: The image that will be displayed when this variant is selected on the storefront. When updating a SKU image, send the publicly accessible URL. Supported image formats are JPEG, PNG, and GIF. Generic product images (not specific to the variant) should be stored on the product.

</p>
<div class="param">upc (optional)</div>

String: The UPC code used in feeds for shopping comparison sites and external channel integrations.

</p>
<div class="param">inventory_level (optional)</div>

Integer: Inventory level for the variant, which is used when the product's `inventory_tracking` is set to `variant`.

</p>
<div class="param">inventory_warning_level (optional)</div>

Integer: When the variant hits this inventory level, it is considered low stock.

</p>
<div class="param">bin_picking_number (optional)</div>

String: Identifies where in a warehouse the variant is located.

</p>
<div class="param">option_values (optional)</div>

array[OptionValueShort]: Array of option and option values IDs that make up this variant. Will be empty if the variant is the product's base variant.

</div>

</div>


### <a name="VariantCollectionResponse">VariantCollectionResponse </a>

<div class="model-description">Response payload for the Bigcommerce API.</div>

<div class="field-items">

</p>
<div class="param">data (optional)</div>

(array[Variant])

</p>
<div class="param">meta (optional)</div>

(CollectionMeta)

</div>

</div>


### <a name="VariantPost">VariantPost </a>

<div class="field-items">

</p>
<div class="param">id (optional)</div>

Integer:

</p>
<div class="param">product_id (optional)</div>

Integer:

</p>
<div class="param">sku (optional)</div>

String:

</p>
<div class="param">sku_id (optional)</div>

Integer: Read-only reference to v2 API's SKU ID. Null if it is a base variant.

</p>
<div class="param">price (optional)</div>

String: This variant's base price on the storefront. If this value is null, the product's default price (set in the Product resource's `price` field) will be used as the base price.

</p>
<div class="param">weight (optional)</div>

String: This variant's base weight on the storefront. If this value is null, the product's default weight (set in the Product resource's `weight` field) will be used as the base weight.

</p>
<div class="param">purchasing_disabled (optional)</div>

Boolean: If `true`, this variant will not be purchasable on the storefront.

</p>
<div class="param">purchasing_disabled_message (optional)</div>

String: If `purchasing_disabled` is `true`, this message should show on the storefront when the variant is selected.

</p>
<div class="param">image_url (optional)</div>

String: The image that will be displayed when this variant is selected on the storefront. When updating a SKU image, send the publicly accessible URL. Supported image formats are JPEG, PNG, and GIF. Generic product images (not specific to the variant) should be stored on the product.

</p>
<div class="param">cost_price (optional)</div>

String: The variant's cost price.

</p>
<div class="param">upc (optional)</div>

String: The UPC code used in feeds for shopping comparison sites and external channel integrations.

</p>
<div class="param">inventory_level (optional)</div>

Integer: Inventory level for the variant, which is used when the product's `inventory_tracking` is set to `variant`.

</p>
<div class="param">inventory_warning_level (optional)</div>

Integer: When the variant hits this inventory level, it is considered low stock.

</p>
<div class="param">bin_picking_number (optional)</div>

String: Identifies where in a warehouse the variant is located.

</p>
<div class="param">option_values (optional)</div>

(array[OptionValueShortPost])

</div>

</div>


### <a name="VariantResponse">VariantResponse </a>

<div class="model-description">Successful response</div>

<div class="field-items">

</p>
<div class="param">data (optional)</div>

(Variant)

</p>
<div class="param">meta (optional)</div>

(Meta)

</div>

</div>