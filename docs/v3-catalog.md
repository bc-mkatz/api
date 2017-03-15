# v3 API Documentation

_Welcome! Please note that this API is in our partner-release stage, which means that in the short term, we'll be iterating on feedback from partners. Our goal is to make sure that all concerns are addressed, and that we reach our goal of creating the most powerful, easiest-to-use catalog API in ecommerce. Because of this iterative approach, please expect small changes and many additions to occur._

**Have suggestions, feedback or questions? Submit them as an issue here:** 
https://github.com/bigcommerce/api/issues

**Want to see what we have in development, and help direct our roadmap? View our public API roadmap here:** https://trello.com/b/1Od4oCsl/bigcommerce-api-roadmap

**Jump directly to: [API Reference](#api_ref)**

## Access and Authentication

All BigCommerce stores have access to the v3 Catalog API.

The base URI is: https://api.bigcommerce.com/stores/{store_hash}/v3/

To authenticate, you'll need to use an OAuth client ID and token, sent along with the following headers:
  
- Accept: application/json  
- X-Auth-Client: {client_id}  
- X-Auth-Token: {oauth_token}

The flow to obtain a client ID and token is the same as with the v2 API. You can now obtain OAuth credentials directly from the BigCommerce control panel, as outlined [here](https://developer.bigcommerce.com/api/#authenticating-with-oauth).

_Note that in the future, we'll be deprecating legacy keys from v2, and removing the ability to create v2 keys within the CP. So grokking our OAuth flow now is not a wasted effort!_

Existing v2 client IDs and tokens will also work with the v3 API. So, if you've already integrated with v2 using our OAuth flow, you should be ready to work with this v3 API!

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

# <span id="api_ref"> v3 API Reference </span>

Please view the documentation generated from the Swagger file in YAML format [here](http://editor.swagger.io/#/?import=https://raw.githubusercontent.com/bigcommerce/api/master/swagger/v3-catalog.yaml).

<html>
 <head>
  <title>
   BigCommerce v3 API
  </title>
 </head>
 <body>
  <div class="app-desc">
   A Swagger Document for the BigCommmerce API V3.
  </div>
  <h2>
   <a name="__Methods">
    Methods/Endpoints
   </a>
  </h2>
  [ Jump to:
  <a href="#__Models">
   Models
  </a>
  ]
  <h3>
   Table of Contents
  </h3>
  <div class="method-summary">
  </div>
  <h4>
   <a href="#Catalog">
    Catalog API
   </a>
  </h4>
  <ul>
  <li><a href="#catalogSummaryGet"><code><span class="http-method">get</span> /catalog/summary</code></a></li>
  <li><a href="#createBrand"><code><span class="http-method">post</span> /catalog/brands</code></a></li>
  <li><a href="#createBrandImage"><code><span class="http-method">post</span> /catalog/brands/{brand_id}/image</code></a></li>
  <li><a href="#createBrandMetafield"><code><span class="http-method">post</span> /catalog/brands/{brand_id}/metafields</code></a></li>
  <li><a href="#createCategory"><code><span class="http-method">post</span> /catalog/categories</code></a></li>
  <li><a href="#createCategoryImage"><code><span class="http-method">post</span> /catalog/categories/{category_id}/image</code></a></li>
  <li><a href="#createCategoryMetafield"><code><span class="http-method">post</span> /catalog/categories/{category_id}/metafields</code></a></li>
  <li><a href="#createComplexRule"><code><span class="http-method">post</span> /catalog/products/{product_id}/complex-rules</code></a></li>
  <li><a href="#createCustomField"><code><span class="http-method">post</span> /catalog/products/{product_id}/custom-fields</code></a></li>
  <li><a href="#createModifier"><code><span class="http-method">post</span> /catalog/products/{product_id}/modifiers</code></a></li>
  <li><a href="#createModifierImage"><code><span class="http-method">post</span> /catalog/products/{product_id}/modifiers/{modifier_id}/values/{value_id}/image</code></a></li>
  <li><a href="#createModifierValue"><code><span class="http-method">post</span> /catalog/products/{product_id}/modifiers/{modifier_id}/values</code></a></li>
  <li><a href="#createOption"><code><span class="http-method">post</span> /catalog/products/{product_id}/options</code></a></li>
  <li><a href="#createOptionValue"><code><span class="http-method">post</span> /catalog/products/{product_id}/options/{option_id}/values</code></a></li>
  <li><a href="#createProduct"><code><span class="http-method">post</span> /catalog/products</code></a></li>
  <li><a href="#createProductImage"><code><span class="http-method">post</span> /catalog/products/{product_id}/images</code></a></li>
  <li><a href="#createProductMetafield"><code><span class="http-method">post</span> /catalog/products/{product_id}/metafields</code></a></li>
  <li><a href="#createProductVideo"><code><span class="http-method">post</span> /catalog/products/{product_id}/videos</code></a></li>
  <li><a href="#createVariant"><code><span class="http-method">post</span> /catalog/products/{product_id}/variants</code></a></li>
  <li><a href="#createVariantImage"><code><span class="http-method">post</span> /catalog/products/{product_id}/variants/{variant_id}/image</code></a></li>
  <li><a href="#createVariantMetafield"><code><span class="http-method">post</span> /catalog/products/{product_id}/variants/{variant_id}/metafields</code></a></li>
  <li><a href="#deleteBrandById"><code><span class="http-method">delete</span> /catalog/brands/{brand_id}</code></a></li>
  <li><a href="#deleteBrandImage"><code><span class="http-method">delete</span> /catalog/brands/{brand_id}/image</code></a></li>
  <li><a href="#deleteBrandMetafieldById"><code><span class="http-method">delete</span> /catalog/brands/{brand_id}/metafields/{metafield_id}</code></a></li>
  <li><a href="#deleteBrands"><code><span class="http-method">delete</span> /catalog/brands</code></a></li>
  <li><a href="#deleteCategories"><code><span class="http-method">delete</span> /catalog/categories</code></a></li>
  <li><a href="#deleteCategoryById"><code><span class="http-method">delete</span> /catalog/categories/{category_id}</code></a></li>
  <li><a href="#deleteCategoryImage"><code><span class="http-method">delete</span> /catalog/categories/{category_id}/image</code></a></li>
  <li><a href="#deleteCategoryMetafieldById"><code><span class="http-method">delete</span> /catalog/categories/{category_id}/metafields/{metafield_id}</code></a></li>
  <li><a href="#deleteComplexRuleById"><code><span class="http-method">delete</span> /catalog/products/{product_id}/complex-rules/{complex_rule_id}</code></a></li>
  <li><a href="#deleteCustomFieldById"><code><span class="http-method">delete</span> /catalog/products/{product_id}/custom-fields/{custom_field_id}</code></a></li>
  <li><a href="#deleteModifierById"><code><span class="http-method">delete</span> /catalog/products/{product_id}/modifiers/{modifier_id}</code></a></li>
  <li><a href="#deleteModifierImage"><code><span class="http-method">delete</span> /catalog/products/{product_id}/modifiers/{modifier_id}/values/{value_id}/image</code></a></li>
  <li><a href="#deleteModifierValueById"><code><span class="http-method">delete</span> /catalog/products/{product_id}/modifiers/{modifier_id}/values/{value_id}</code></a></li>
  <li><a href="#deleteOptionById"><code><span class="http-method">delete</span> /catalog/products/{product_id}/options/{option_id}</code></a></li>
  <li><a href="#deleteOptionValueById"><code><span class="http-method">delete</span> /catalog/products/{product_id}/options/{option_id}/values/{value_id}</code></a></li>
  <li><a href="#deleteProductById"><code><span class="http-method">delete</span> /catalog/products/{product_id}</code></a></li>
  <li><a href="#deleteProductImage"><code><span class="http-method">delete</span> /catalog/products/{product_id}/images/{image_id}</code></a></li>
  <li><a href="#deleteProductMetafieldById"><code><span class="http-method">delete</span> /catalog/products/{product_id}/metafields/{metafield_id}</code></a></li>
  <li><a href="#deleteProductVideo"><code><span class="http-method">delete</span> /catalog/products/{product_id}/videos/{video_id}</code></a></li>
  <li><a href="#deleteProducts"><code><span class="http-method">delete</span> /catalog/products</code></a></li>
  <li><a href="#deleteVariantById"><code><span class="http-method">delete</span> /catalog/products/{product_id}/variants/{variant_id}</code></a></li>
  <li><a href="#deleteVariantMetafieldById"><code><span class="http-method">delete</span> /catalog/products/{product_id}/variants/{variant_id}/metafields/{metafield_id}</code></a></li>
  <li><a href="#getBrandById"><code><span class="http-method">get</span> /catalog/brands/{brand_id}</code></a></li>
  <li><a href="#getBrandMetafieldByBrandId"><code><span class="http-method">get</span> /catalog/brands/{brand_id}/metafields/{metafield_id}</code></a></li>
  <li><a href="#getBrandMetafieldsByBrandId"><code><span class="http-method">get</span> /catalog/brands/{brand_id}/metafields</code></a></li>
  <li><a href="#getBrands"><code><span class="http-method">get</span> /catalog/brands</code></a></li>
  <li><a href="#getCategories"><code><span class="http-method">get</span> /catalog/categories</code></a></li>
  <li><a href="#getCategoryById"><code><span class="http-method">get</span> /catalog/categories/{category_id}</code></a></li>
  <li><a href="#getCategoryMetafieldByCategoryId"><code><span class="http-method">get</span> /catalog/categories/{category_id}/metafields/{metafield_id}</code></a></li>
  <li><a href="#getCategoryMetafieldsByCategoryId"><code><span class="http-method">get</span> /catalog/categories/{category_id}/metafields</code></a></li>
  <li><a href="#getCategoryTree"><code><span class="http-method">get</span> /catalog/categories/tree</code></a></li>
  <li><a href="#getComplexRuleById"><code><span class="http-method">get</span> /catalog/products/{product_id}/complex-rules/{complex_rule_id}</code></a></li>
  <li><a href="#getComplexRules"><code><span class="http-method">get</span> /catalog/products/{product_id}/complex-rules</code></a></li>
  <li><a href="#getCustomFieldById"><code><span class="http-method">get</span> /catalog/products/{product_id}/custom-fields/{custom_field_id}</code></a></li>
  <li><a href="#getCustomFields"><code><span class="http-method">get</span> /catalog/products/{product_id}/custom-fields</code></a></li>
  <li><a href="#getModifierById"><code><span class="http-method">get</span> /catalog/products/{product_id}/modifiers/{modifier_id}</code></a></li>
  <li><a href="#getModifierValueById"><code><span class="http-method">get</span> /catalog/products/{product_id}/modifiers/{modifier_id}/values/{value_id}</code></a></li>
  <li><a href="#getModifierValues"><code><span class="http-method">get</span> /catalog/products/{product_id}/modifiers/{modifier_id}/values</code></a></li>
  <li><a href="#getModifiers"><code><span class="http-method">get</span> /catalog/products/{product_id}/modifiers</code></a></li>
  <li><a href="#getOptionById"><code><span class="http-method">get</span> /catalog/products/{product_id}/options/{option_id}</code></a></li>
  <li><a href="#getOptionValueById"><code><span class="http-method">get</span> /catalog/products/{product_id}/options/{option_id}/values/{value_id}</code></a></li>
  <li><a href="#getOptionValues"><code><span class="http-method">get</span> /catalog/products/{product_id}/options/{option_id}/values</code></a></li>
  <li><a href="#getOptions"><code><span class="http-method">get</span> /catalog/products/{product_id}/options</code></a></li>
  <li><a href="#getProductById"><code><span class="http-method">get</span> /catalog/products/{product_id}</code></a></li>
  <li><a href="#getProductImageById"><code><span class="http-method">get</span> /catalog/products/{product_id}/images/{image_id}</code></a></li>
  <li><a href="#getProductImages"><code><span class="http-method">get</span> /catalog/products/{product_id}/images</code></a></li>
  <li><a href="#getProductMetafieldByProductId"><code><span class="http-method">get</span> /catalog/products/{product_id}/metafields/{metafield_id}</code></a></li>
  <li><a href="#getProductMetafieldsByProductId"><code><span class="http-method">get</span> /catalog/products/{product_id}/metafields</code></a></li>
  <li><a href="#getProductVideoById"><code><span class="http-method">get</span> /catalog/products/{product_id}/videos/{video_id}</code></a></li>
  <li><a href="#getProductVideos"><code><span class="http-method">get</span> /catalog/products/{product_id}/videos</code></a></li>
  <li><a href="#getProducts"><code><span class="http-method">get</span> /catalog/products</code></a></li>
  <li><a href="#getVariantById"><code><span class="http-method">get</span> /catalog/products/{product_id}/variants/{variant_id}</code></a></li>
  <li><a href="#getVariantMetafieldByProductIdAndVariantId"><code><span class="http-method">get</span> /catalog/products/{product_id}/variants/{variant_id}/metafields/{metafield_id}</code></a></li>
  <li><a href="#getVariantMetafieldsByProductIdAndVariantId"><code><span class="http-method">get</span> /catalog/products/{product_id}/variants/{variant_id}/metafields</code></a></li>
  <li><a href="#getVariants"><code><span class="http-method">get</span> /catalog/variants</code></a></li>
  <li><a href="#getVariantsByProductId"><code><span class="http-method">get</span> /catalog/products/{product_id}/variants</code></a></li>
  <li><a href="#updateBrand"><code><span class="http-method">put</span> /catalog/brands/{brand_id}</code></a></li>
  <li><a href="#updateBrandMetafield"><code><span class="http-method">put</span> /catalog/brands/{brand_id}/metafields/{metafield_id}</code></a></li>
  <li><a href="#updateCategory"><code><span class="http-method">put</span> /catalog/categories/{category_id}</code></a></li>
  <li><a href="#updateCategoryMetafield"><code><span class="http-method">put</span> /catalog/categories/{category_id}/metafields/{metafield_id}</code></a></li>
  <li><a href="#updateComplexRule"><code><span class="http-method">put</span> /catalog/products/{product_id}/complex-rules/{complex_rule_id}</code></a></li>
  <li><a href="#updateCustomField"><code><span class="http-method">put</span> /catalog/products/{product_id}/custom-fields/{custom_field_id}</code></a></li>
  <li><a href="#updateModifier"><code><span class="http-method">put</span> /catalog/products/{product_id}/modifiers/{modifier_id}</code></a></li>
  <li><a href="#updateModifierValue"><code><span class="http-method">put</span> /catalog/products/{product_id}/modifiers/{modifier_id}/values/{value_id}</code></a></li>
  <li><a href="#updateOption"><code><span class="http-method">put</span> /catalog/products/{product_id}/options/{option_id}</code></a></li>
  <li><a href="#updateOptionValue"><code><span class="http-method">put</span> /catalog/products/{product_id}/options/{option_id}/values/{value_id}</code></a></li>
  <li><a href="#updateProduct"><code><span class="http-method">put</span> /catalog/products/{product_id}</code></a></li>
  <li><a href="#updateProductImage"><code><span class="http-method">put</span> /catalog/products/{product_id}/images/{image_id}</code></a></li>
  <li><a href="#updateProductMetafield"><code><span class="http-method">put</span> /catalog/products/{product_id}/metafields/{metafield_id}</code></a></li>
  <li><a href="#updateProductVideo"><code><span class="http-method">put</span> /catalog/products/{product_id}/videos/{video_id}</code></a></li>
  <li><a href="#updateVariant"><code><span class="http-method">put</span> /catalog/products/{product_id}/variants/{variant_id}</code></a></li>
  <li><a href="#updateVariantMetafield"><code><span class="http-method">put</span> /catalog/products/{product_id}/variants/{variant_id}/metafields/{metafield_id}</code></a></li>
  </ul>
  <h4>
   <a href="#Customers">
    Customers API
   </a>
  </h4>
  <ul>
  <li><a href="#createSubscriber"><code><span class="http-method">post</span> /customers/subscribers</code></a></li>
  <li><a href="#deleteSubscriberById"><code><span class="http-method">delete</span> /customers/subscribers/{subscriber_id}</code></a></li>
  <li><a href="#deleteSubscribers"><code><span class="http-method">delete</span> /customers/subscribers</code></a></li>
  <li><a href="#getSubscriberById"><code><span class="http-method">get</span> /customers/subscribers/{subscriber_id}</code></a></li>
  <li><a href="#getSubscribers"><code><span class="http-method">get</span> /customers/subscribers</code></a></li>
  <li><a href="#updateSubscriber"><code><span class="http-method">put</span> /customers/subscribers/{subscriber_id}</code></a></li>
  </ul>
  <h1>
   <a name="Catalog">
    Catalog API
   </a>
  </h1>
<div class="method"><a name="catalogSummaryGet"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/summary</code></pre></div>
    <div class="method-summary"> (<span class="nickname">catalogSummaryGet</span>)</div>
    <div class="method-notes">Returns a lightweight inventory summary from the BigCommerce Catalog.
</div>


    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>





    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#CatalogSummaryResponse">CatalogSummaryResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : {
    "inventory_value" : 1.3579000000000001069366817318950779736042022705078125,
    "inventory_count" : 123,
    "primary_category_id" : 123,
    "primary_category_name" : "aeiou"
  },
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    An array of catalog summary and metadata.

        <a href="#CatalogSummaryResponse">CatalogSummaryResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="createBrand"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="post"><code class="huge"><span class="http-method">post</span> /catalog/brands</code></pre></div>
    <div class="method-summary"> (<span class="nickname">createBrand</span>)</div>
    <div class="method-notes">Creates a &#x60;Brand&#x60; object.</div>


    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">Brand <a href="#BrandPost">BrandPost</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; A &#x60;Brand&#x60; object.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#BrandResponse">BrandResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A &#x60;Brand&#x60; object.

        <a href="#BrandResponse">BrandResponse</a>
    <h4 class="field-label">409</h4>
    Brand was in conflict with another brand. This is the result of duplicate unique fields such as name.

        <a href="#ErrorResponse">ErrorResponse</a>
    <h4 class="field-label">422</h4>
    Brand was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.

        <a href="#ErrorResponse">ErrorResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="createBrandImage"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="post"><code class="huge"><span class="http-method">post</span> /catalog/brands/{brand_id}/image</code></pre></div>
    <div class="method-summary"> (<span class="nickname">createBrandImage</span>)</div>
    <div class="method-notes">Creates an image on a &#x60;Brand&#x60;. Publicly accessible URLs and files (form post) are valid parameters.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">brand_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Brand&#x60; to which the resource belongs.
 format: int</div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>multipart/form-data</code></li>
    </ul>




    <h3 class="field-label">Form parameters</h3>
    <div class="field-items">
      <div class="param">image_file (required)</div>

      <div class="param-desc"><span class="param-type">Form Parameter</span> &mdash; An image file. Supported MIME types include GIF, JPEG, and PNG.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#ImageResponse">ImageResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : {
    "image_url" : "aeiou"
  },
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A ResourceImage and metadata.

        <a href="#ImageResponse">ImageResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
    <h4 class="field-label">422</h4>
    Image was not valid. This is the result of a missing &#x60;image_file&#x60; field or an incorrect file type. See the response for more details.

        <a href="#ErrorResponse">ErrorResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="createBrandMetafield"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="post"><code class="huge"><span class="http-method">post</span> /catalog/brands/{brand_id}/metafields</code></pre></div>
    <div class="method-summary"> (<span class="nickname">createBrandMetafield</span>)</div>
    <div class="method-notes">Creates a product &#x60;Metafield&#x60;.</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">brand_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Brand&#x60; to which the resource belongs.
 format: int</div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">Metafield <a href="#MetafieldPost">MetafieldPost</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; A &#x60;Metafield&#x60; object.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#MetafieldResponse">MetafieldResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A &#x60;Metafield&#x60; object.

        <a href="#MetafieldResponse">MetafieldResponse</a>
    <h4 class="field-label">409</h4>
    The &#x60;Metafield&#x60; was in conflict with another &#x60;Metafield&#x60;. This can be the result of duplicate unique key combination of the app&#39;s client id, namespace, key, resource_type, and resource_id.

        <a href="#ErrorResponse">ErrorResponse</a>
    <h4 class="field-label">422</h4>
    The &#x60;Metafield&#x60; was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.

        <a href="#ErrorResponse">ErrorResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="createCategory"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="post"><code class="huge"><span class="http-method">post</span> /catalog/categories</code></pre></div>
    <div class="method-summary"> (<span class="nickname">createCategory</span>)</div>
    <div class="method-notes">Creates a &#x60;Category&#x60; in the BigCommerce Catalog.
</div>


    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">category <a href="#CategoryPost">CategoryPost</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; A BigCommerce &#x60;Category&#x60; object.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#CategoryResponse">CategoryResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A category object.

        <a href="#CategoryResponse">CategoryResponse</a>
    <h4 class="field-label">409</h4>
    The &#x60;Category&#x60; was in conflict with another category. This is the result of duplicate unique values, such as &#x60;name&#x60; or &#x60;custom_url&#x60;.

        <a href="#ErrorResponse">ErrorResponse</a>
    <h4 class="field-label">422</h4>
    The &#x60;Category&#x60; was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.

        <a href="#ErrorResponse">ErrorResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="createCategoryImage"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="post"><code class="huge"><span class="http-method">post</span> /catalog/categories/{category_id}/image</code></pre></div>
    <div class="method-summary"> (<span class="nickname">createCategoryImage</span>)</div>
    <div class="method-notes">Creates an image on a category. Publicly accessible URLs and files (form post) are valid parameters.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">category_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Category&#x60; to which the resource belongs.
 format: int</div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>multipart/form-data</code></li>
    </ul>




    <h3 class="field-label">Form parameters</h3>
    <div class="field-items">
      <div class="param">image_file (required)</div>

      <div class="param-desc"><span class="param-type">Form Parameter</span> &mdash; An image file. Supported MIME types include GIF, JPEG, and PNG.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#ImageResponse">ImageResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : {
    "image_url" : "aeiou"
  },
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A ResourceImage and metadata.

        <a href="#ImageResponse">ImageResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
    <h4 class="field-label">422</h4>
    Image was not valid. This is the result of a missing &#x60;image_file&#x60; field or an incorrect file type. See the response for more details.

        <a href="#ErrorResponse">ErrorResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="createCategoryMetafield"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="post"><code class="huge"><span class="http-method">post</span> /catalog/categories/{category_id}/metafields</code></pre></div>
    <div class="method-summary"> (<span class="nickname">createCategoryMetafield</span>)</div>
    <div class="method-notes">Creates a product &#x60;Metafield&#x60;.</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">category_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Category&#x60; to which the resource belongs.
 format: int</div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">Metafield <a href="#MetafieldPost">MetafieldPost</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; A &#x60;Metafield&#x60; object.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#MetafieldResponse">MetafieldResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A &#x60;Metafield&#x60; object.

        <a href="#MetafieldResponse">MetafieldResponse</a>
    <h4 class="field-label">409</h4>
    The &#x60;Metafield&#x60; was in conflict with another &#x60;Metafield&#x60;. This can be the result of duplicate unique key combinations of the app&#39;s client id, namespace, key, resource_type, and resource_id.

        <a href="#ErrorResponse">ErrorResponse</a>
    <h4 class="field-label">422</h4>
    The &#x60;Metafield&#x60; was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.

        <a href="#ErrorResponse">ErrorResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="createComplexRule"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="post"><code class="huge"><span class="http-method">post</span> /catalog/products/{product_id}/complex-rules</code></pre></div>
    <div class="method-summary"> (<span class="nickname">createComplexRule</span>)</div>
    <div class="method-notes">Creates a &#x60;ComplexRule&#x60;.</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">ComplexRule <a href="#ComplexRulePost">ComplexRulePost</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; &#x60;ComplexRule&#x60; object.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#ComplexRuleResponse">ComplexRuleResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A &#x60;ComplexRule&#x60; object.

        <a href="#ComplexRuleResponse">ComplexRuleResponse</a>
    <h4 class="field-label">409</h4>
    The &#x60;ComplexRule&#x60; was in conflict with another &#x60;ComplexRule&#x60;. This is the result of duplicate conditions.

        <a href="#ErrorResponse">ErrorResponse</a>
    <h4 class="field-label">422</h4>
    The &#x60;ComplexRule&#x60; was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.

        <a href="#ErrorResponse">ErrorResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="createCustomField"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="post"><code class="huge"><span class="http-method">post</span> /catalog/products/{product_id}/custom-fields</code></pre></div>
    <div class="method-summary"> (<span class="nickname">createCustomField</span>)</div>
    <div class="method-notes">Creates a &#x60;CustomField&#x60;.</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">CustomField <a href="#CustomFieldPost">CustomFieldPost</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; &#x60;CustomField&#x60; object.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#CustomFieldResponse">CustomFieldResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A &#x60;CustomField&#x60; object.

        <a href="#CustomFieldResponse">CustomFieldResponse</a>
    <h4 class="field-label">404</h4>
    The parent resource was not found.

        <a href="#NotFound">NotFound</a>
    <h4 class="field-label">422</h4>
    The &#x60;CustomField&#x60; was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.

        <a href="#ErrorResponse">ErrorResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="createModifier"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="post"><code class="huge"><span class="http-method">post</span> /catalog/products/{product_id}/modifiers</code></pre></div>
    <div class="method-summary"> (<span class="nickname">createModifier</span>)</div>
    <div class="method-notes">Creates a &#x60;Modifier&#x60;.</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">Modifier <a href="#ModifierPost">ModifierPost</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; A &#x60;Modifier&#x60; object.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#ModifierResponse">ModifierResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A &#x60;Modifier&#x60; object.

        <a href="#ModifierResponse">ModifierResponse</a>
    <h4 class="field-label">409</h4>
    The &#x60;Modifier&#x60; was in conflict with another option. This is the result of duplicate unique fields, such as &#x60;name&#x60;.

        <a href="#ErrorResponse">ErrorResponse</a>
    <h4 class="field-label">422</h4>
    The &#x60;Modifier&#x60; was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.

        <a href="#ErrorResponse">ErrorResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="createModifierImage"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="post"><code class="huge"><span class="http-method">post</span> /catalog/products/{product_id}/modifiers/{modifier_id}/values/{value_id}/image</code></pre></div>
    <div class="method-summary"> (<span class="nickname">createModifierImage</span>)</div>
    <div class="method-notes">Adds an image to a modifier value; the image will show on the storefront when the value is selected.</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">modifier_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Modifier&#x60;.
 </div><div class="param">value_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Modifier&#x60;.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>multipart/form-data</code></li>
    </ul>




    <h3 class="field-label">Form parameters</h3>
    <div class="field-items">
      <div class="param">image_file (required)</div>

      <div class="param-desc"><span class="param-type">Form Parameter</span> &mdash; An image file. Supported MIME types include GIF, JPEG, and PNG.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#ImageResponse">ImageResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : {
    "image_url" : "aeiou"
  },
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A ResourceImage and metadata.

        <a href="#ImageResponse">ImageResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
    <h4 class="field-label">422</h4>
    Modifier image was not valid. This is the result of missing &#x60;image_file&#x60; fields, orof a non-URL value for the &#x60;image_file&#x60; field. See the response for more details.

        <a href="#ErrorResponse">ErrorResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="createModifierValue"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="post"><code class="huge"><span class="http-method">post</span> /catalog/products/{product_id}/modifiers/{modifier_id}/values</code></pre></div>
    <div class="method-summary"> (<span class="nickname">createModifierValue</span>)</div>
    <div class="method-notes">Creates a &#x60;ModifierValue&#x60;.</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">modifier_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Modifier&#x60;.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">ModifierValue <a href="#ModifierValuePost">ModifierValuePost</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; A &#x60;ModifierValue&#x60; object.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#ModifierValueResponse">ModifierValueResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A &#x60;ModifierValue&#x60; object.

        <a href="#ModifierValueResponse">ModifierValueResponse</a>
    <h4 class="field-label">422</h4>
    The &#x60;ModifierValue&#x60; was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.

        <a href="#ErrorResponse">ErrorResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="createOption"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="post"><code class="huge"><span class="http-method">post</span> /catalog/products/{product_id}/options</code></pre></div>
    <div class="method-summary"> (<span class="nickname">createOption</span>)</div>
    <div class="method-notes">Creates an &#x60;Option&#x60;.</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">Option <a href="#OptionPost">OptionPost</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; An &#x60;Option&#x60; object.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#OptionResponse">OptionResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    An &#x60;Option&#x60; object.

        <a href="#OptionResponse">OptionResponse</a>
    <h4 class="field-label">409</h4>
    Option was in conflict with another option. This is the result of duplicate unique fields, such as &#x60;name&#x60;.

        <a href="#ErrorResponse">ErrorResponse</a>
    <h4 class="field-label">422</h4>
    Option was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.

        <a href="#ErrorResponse">ErrorResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="createOptionValue"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="post"><code class="huge"><span class="http-method">post</span> /catalog/products/{product_id}/options/{option_id}/values</code></pre></div>
    <div class="method-summary"> (<span class="nickname">createOptionValue</span>)</div>
    <div class="method-notes">Creates a &#x60;OptionValue&#x60;.</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">option_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Option&#x60;.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">OptionValue <a href="#OptionValuePost">OptionValuePost</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; A &#x60;OptionValue&#x60; object.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#OptionValueResponse">OptionValueResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A &#x60;OptionValue&#x60; object.

        <a href="#OptionValueResponse">OptionValueResponse</a>
    <h4 class="field-label">422</h4>
    The &#x60;OptionValue&#x60; was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.

        <a href="#ErrorResponse">ErrorResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="createProduct"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="post"><code class="huge"><span class="http-method">post</span> /catalog/products</code></pre></div>
    <div class="method-summary"> (<span class="nickname">createProduct</span>)</div>
    <div class="method-notes">Creates a &#x60;Product&#x60; in the BigCommerce Catalog.
</div>


    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">product <a href="#ProductPost">ProductPost</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; A BigCommerce &#x60;Product&#x60; object.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#ProductResponse">ProductResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A product.

        <a href="#ProductResponse">ProductResponse</a>
    <h4 class="field-label">409</h4>
    &#x60;Product&#x60; was in conflict with another product. This is the result of duplicate unique values, such as name or SKU; a missing or invalid category id, brand id, or tax_class id; or a conflicting &#x60;bulk_pricing_rule&#x60;.

        <a href="#ErrorResponse">ErrorResponse</a>
    <h4 class="field-label">422</h4>
    &#x60;Product&#x60; was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.

        <a href="#ErrorResponse">ErrorResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="createProductImage"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="post"><code class="huge"><span class="http-method">post</span> /catalog/products/{product_id}/images</code></pre></div>
    <div class="method-summary"> (<span class="nickname">createProductImage</span>)</div>
    <div class="method-notes">Creates an image on a product. Publically accessible URLs and files (form post) are valid parameters.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">productImage <a href="#ProductImagePost">ProductImagePost</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; A BigCommerce &#x60;ProductImage&#x60; object.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#ProductImageResponse">ProductImageResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A product image.

        <a href="#ProductImageResponse">ProductImageResponse</a>
    <h4 class="field-label">404</h4>
    The product ID does not exist.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="createProductMetafield"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="post"><code class="huge"><span class="http-method">post</span> /catalog/products/{product_id}/metafields</code></pre></div>
    <div class="method-summary"> (<span class="nickname">createProductMetafield</span>)</div>
    <div class="method-notes">Creates a product &#x60;Metafield&#x60;.</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">Metafield <a href="#MetafieldPost">MetafieldPost</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; A &#x60;Metafield&#x60; object.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#MetafieldResponse">MetafieldResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A &#x60;Metafield&#x60; object.

        <a href="#MetafieldResponse">MetafieldResponse</a>
    <h4 class="field-label">409</h4>
    The &#x60;Metafield&#x60; was in conflict with another &#x60;Metafield&#x60;. This can be the result of duplicate unique key combinations of the app&#39;s client id, namespace, key, resource_type, and resource_id.

        <a href="#ErrorResponse">ErrorResponse</a>
    <h4 class="field-label">422</h4>
    The &#x60;Metafield&#x60; was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.

        <a href="#ErrorResponse">ErrorResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="createProductVideo"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="post"><code class="huge"><span class="http-method">post</span> /catalog/products/{product_id}/videos</code></pre></div>
    <div class="method-summary"> (<span class="nickname">createProductVideo</span>)</div>
    <div class="method-notes">Creates a video on a product, using a video ID from YouTube.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">productVideo <a href="#ProductVideoPost">ProductVideoPost</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; A BigCommerce &#x60;ProductVideo&#x60; object.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#ProductVideoResponse">ProductVideoResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A product video.

        <a href="#ProductVideoResponse">ProductVideoResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="createVariant"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="post"><code class="huge"><span class="http-method">post</span> /catalog/products/{product_id}/variants</code></pre></div>
    <div class="method-summary"> (<span class="nickname">createVariant</span>)</div>
    <div class="method-notes">Creates a &#x60;Variant&#x60; object.</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">Variant <a href="#VariantPost">VariantPost</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; &#x60;Variant&#x60; object.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#VariantResponse">VariantResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A variant and metadata.

        <a href="#VariantResponse">VariantResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="createVariantImage"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="post"><code class="huge"><span class="http-method">post</span> /catalog/products/{product_id}/variants/{variant_id}/image</code></pre></div>
    <div class="method-summary"> (<span class="nickname">createVariantImage</span>)</div>
    <div class="method-notes"></div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">variant_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Variant&#x60; to which the resource belongs.
 format: int</div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>multipart/form-data</code></li>
    </ul>




    <h3 class="field-label">Form parameters</h3>
    <div class="field-items">
      <div class="param">image_file (required)</div>

      <div class="param-desc"><span class="param-type">Form Parameter</span> &mdash; An image file. Supported MIME types include GIF, JPEG, and PNG.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#ImageResponse">ImageResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : {
    "image_url" : "aeiou"
  },
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A ResourceImage and metadata.

        <a href="#ImageResponse">ImageResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
    <h4 class="field-label">422</h4>
    Image was not valid. This is the result of a missing image_file field or an incorrect file type. See the response for more details.

        <a href="#ErrorResponse">ErrorResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="createVariantMetafield"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="post"><code class="huge"><span class="http-method">post</span> /catalog/products/{product_id}/variants/{variant_id}/metafields</code></pre></div>
    <div class="method-summary"> (<span class="nickname">createVariantMetafield</span>)</div>
    <div class="method-notes">Creates a variant &#x60;Metafield&#x60;.</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">variant_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Variant&#x60; to which the resource belongs.
 format: int</div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">Metafield <a href="#MetafieldPost">MetafieldPost</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; A &#x60;Metafield&#x60; object.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#MetafieldResponse">MetafieldResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A &#x60;Metafield&#x60; object.

        <a href="#MetafieldResponse">MetafieldResponse</a>
    <h4 class="field-label">409</h4>
    The &#x60;Metafield&#x60; was in conflict with another &#x60;Metafield&#x60;. This can be the result of duplicate unique-key combinations of the app&#39;s client id, namespace, key, resource_type, and resource_id.

        <a href="#ErrorResponse">ErrorResponse</a>
    <h4 class="field-label">422</h4>
    The &#x60;Metafield&#x60; was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.

        <a href="#ErrorResponse">ErrorResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="deleteBrandById"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="delete"><code class="huge"><span class="http-method">delete</span> /catalog/brands/{brand_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">deleteBrandById</span>)</div>
    <div class="method-notes">Deletes a &#x60;Brand&#x60; from the BigCommerce Catalog.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">brand_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Brand&#x60; to which the resource belongs.
 format: int</div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>






    <!--Todo: process Response Object and its headers, schema, examples -->


    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">204</h4>
    An empty response.

        <a href="#"></a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="deleteBrandImage"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="delete"><code class="huge"><span class="http-method">delete</span> /catalog/brands/{brand_id}/image</code></pre></div>
    <div class="method-summary"> (<span class="nickname">deleteBrandImage</span>)</div>
    <div class="method-notes">Deletes a &#x60;Brand&#x60; image from the BigCommerce Catalog.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">brand_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Brand&#x60; to which the resource belongs.
 format: int</div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>






    <!--Todo: process Response Object and its headers, schema, examples -->


    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">204</h4>
    Image cleared from the brand.
        <a href="#"></a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="deleteBrandMetafieldById"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="delete"><code class="huge"><span class="http-method">delete</span> /catalog/brands/{brand_id}/metafields/{metafield_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">deleteBrandMetafieldById</span>)</div>
    <div class="method-notes">Deletes a &#x60;Metafield&#x60;.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">metafield_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Metafield&#x60;.
 format: int</div><div class="param">brand_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Brand&#x60; to which the resource belongs.
 format: int</div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>






    <!--Todo: process Response Object and its headers, schema, examples -->


    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">204</h4>
    An empty response.

        <a href="#"></a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="deleteBrands"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="delete"><code class="huge"><span class="http-method">delete</span> /catalog/brands</code></pre></div>
    <div class="method-summary"> (<span class="nickname">deleteBrands</span>)</div>
    <div class="method-notes">Deletes one or more &#x60;Brand&#x60; objects from the BigCommerce Catalog.
</div>


    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>



    <h3 class="field-label">Query parameters</h3>
    <div class="field-items">
      <div class="param">name (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by name.
 </div><div class="param">page_title (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by page_title.
 </div>
    </div>  <!-- field-items -->



    <!--Todo: process Response Object and its headers, schema, examples -->


    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">204</h4>
    An empty response.

        <a href="#"></a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="deleteCategories"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="delete"><code class="huge"><span class="http-method">delete</span> /catalog/categories</code></pre></div>
    <div class="method-summary"> (<span class="nickname">deleteCategories</span>)</div>
    <div class="method-notes">Deletes a product or products from the BigCommerce Catalog.
</div>


    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>



    <h3 class="field-label">Query parameters</h3>
    <div class="field-items">
      <div class="param">name (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by name.
 </div><div class="param">parent_id (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by parent_id.
 </div><div class="param">page_title (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by page_title.
 </div><div class="param">keyword (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by keywords.
 </div><div class="param">is_visible (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by is_visible.
 </div>
    </div>  <!-- field-items -->



    <!--Todo: process Response Object and its headers, schema, examples -->


    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">204</h4>
    An empty response.

        <a href="#"></a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="deleteCategoryById"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="delete"><code class="huge"><span class="http-method">delete</span> /catalog/categories/{category_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">deleteCategoryById</span>)</div>
    <div class="method-notes">Deletes one or more &#x60;Category&#x60; objects from the BigCommerce catalog.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">category_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Category&#x60; to which the resource belongs.
 format: int</div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>






    <!--Todo: process Response Object and its headers, schema, examples -->


    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">204</h4>
    An empty response.

        <a href="#"></a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="deleteCategoryImage"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="delete"><code class="huge"><span class="http-method">delete</span> /catalog/categories/{category_id}/image</code></pre></div>
    <div class="method-summary"> (<span class="nickname">deleteCategoryImage</span>)</div>
    <div class="method-notes">Deletes a &#x60;Category&#x60; image from the BigCommerce Catalog.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">category_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Category&#x60; to which the resource belongs.
 format: int</div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>






    <!--Todo: process Response Object and its headers, schema, examples -->


    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">204</h4>
    An empty response.

        <a href="#"></a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="deleteCategoryMetafieldById"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="delete"><code class="huge"><span class="http-method">delete</span> /catalog/categories/{category_id}/metafields/{metafield_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">deleteCategoryMetafieldById</span>)</div>
    <div class="method-notes">Deletes a &#x60;Metafield&#x60;.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">metafield_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Metafield&#x60;.
 format: int</div><div class="param">category_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Category&#x60; to which the resource belongs.
 format: int</div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>






    <!--Todo: process Response Object and its headers, schema, examples -->


    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">204</h4>
    An empty response.

        <a href="#"></a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="deleteComplexRuleById"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="delete"><code class="huge"><span class="http-method">delete</span> /catalog/products/{product_id}/complex-rules/{complex_rule_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">deleteComplexRuleById</span>)</div>
    <div class="method-notes">Deletes a Product&#39;s &#x60;ComplexRule&#x60;, based on the &#x60;product_id&#x60; and &#x60;complex_rule_id&#x60;.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">complex_rule_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;ComplexRule&#x60;.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>






    <!--Todo: process Response Object and its headers, schema, examples -->


    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">204</h4>
    An empty response.

        <a href="#"></a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="deleteCustomFieldById"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="delete"><code class="huge"><span class="http-method">delete</span> /catalog/products/{product_id}/custom-fields/{custom_field_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">deleteCustomFieldById</span>)</div>
    <div class="method-notes">Deletes a Product&#39;s &#x60;CustomField&#x60;, based on the &#x60;product_id&#x60; and &#x60;custom_field_id&#x60;.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">custom_field_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;CustomField&#x60;.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>






    <!--Todo: process Response Object and its headers, schema, examples -->


    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">204</h4>
    An empty response.

        <a href="#"></a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="deleteModifierById"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="delete"><code class="huge"><span class="http-method">delete</span> /catalog/products/{product_id}/modifiers/{modifier_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">deleteModifierById</span>)</div>
    <div class="method-notes">Deletes a Product&#39;s &#x60;Modifier&#x60; based on the product_id and modifier_id.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">modifier_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Modifier&#x60;.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>






    <!--Todo: process Response Object and its headers, schema, examples -->


    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">204</h4>
    An empty response.

        <a href="#"></a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="deleteModifierImage"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="delete"><code class="huge"><span class="http-method">delete</span> /catalog/products/{product_id}/modifiers/{modifier_id}/values/{value_id}/image</code></pre></div>
    <div class="method-summary"> (<span class="nickname">deleteModifierImage</span>)</div>
    <div class="method-notes">Deletes the image applied to show when the modifier value is selected</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">modifier_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Modifier&#x60;.
 </div><div class="param">value_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Modifier&#x60;.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>






    <!--Todo: process Response Object and its headers, schema, examples -->


    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">204</h4>
    Image cleared for the modifier value.
        <a href="#"></a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="deleteModifierValueById"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="delete"><code class="huge"><span class="http-method">delete</span> /catalog/products/{product_id}/modifiers/{modifier_id}/values/{value_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">deleteModifierValueById</span>)</div>
    <div class="method-notes">Deletes a Product&#39;s &#x60;ModifierValue&#x60; based on the product_id, modifier_id and value_id.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">modifier_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Modifier&#x60;.
 </div><div class="param">value_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Modifier/Option Value&#x60;.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>






    <!--Todo: process Response Object and its headers, schema, examples -->


    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">204</h4>
    An empty response.

        <a href="#"></a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="deleteOptionById"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="delete"><code class="huge"><span class="http-method">delete</span> /catalog/products/{product_id}/options/{option_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">deleteOptionById</span>)</div>
    <div class="method-notes">Deletes a Product&#39;s &#x60;Option&#x60;, based on the product_id and option_id.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">option_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Option&#x60;.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>






    <!--Todo: process Response Object and its headers, schema, examples -->


    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">204</h4>
    An empty response.

        <a href="#"></a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="deleteOptionValueById"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="delete"><code class="huge"><span class="http-method">delete</span> /catalog/products/{product_id}/options/{option_id}/values/{value_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">deleteOptionValueById</span>)</div>
    <div class="method-notes">Deletes a Product&#39;s &#x60;OptionValue&#x60; based on the product_id, option_id and value_id.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">option_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Option&#x60;.
 </div><div class="param">value_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Modifier/Option Value&#x60;.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>






    <!--Todo: process Response Object and its headers, schema, examples -->


    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">204</h4>
    An empty response.

        <a href="#"></a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="deleteProductById"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="delete"><code class="huge"><span class="http-method">delete</span> /catalog/products/{product_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">deleteProductById</span>)</div>
    <div class="method-notes">Deletes a &#x60;Product&#x60; object from the BigCommerce Catalog.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>






    <!--Todo: process Response Object and its headers, schema, examples -->


    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">204</h4>
    An empty response.

        <a href="#"></a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="deleteProductImage"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="delete"><code class="huge"><span class="http-method">delete</span> /catalog/products/{product_id}/images/{image_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">deleteProductImage</span>)</div>
    <div class="method-notes">Deletes a &#x60;ProductImage&#x60; in the BigCommerce Catalog.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">image_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Image&#x60; that is being operated on.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>






    <!--Todo: process Response Object and its headers, schema, examples -->


    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">204</h4>
    An empty response.

        <a href="#"></a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="deleteProductMetafieldById"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="delete"><code class="huge"><span class="http-method">delete</span> /catalog/products/{product_id}/metafields/{metafield_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">deleteProductMetafieldById</span>)</div>
    <div class="method-notes">Deletes a &#x60;Metafield&#x60;.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">metafield_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Metafield&#x60;.
 format: int</div><div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>






    <!--Todo: process Response Object and its headers, schema, examples -->


    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">204</h4>
    An empty response.

        <a href="#"></a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="deleteProductVideo"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="delete"><code class="huge"><span class="http-method">delete</span> /catalog/products/{product_id}/videos/{video_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">deleteProductVideo</span>)</div>
    <div class="method-notes">Deletes a &#x60;ProductVideo&#x60; in the BigCommerce Catalog.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">video_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Video&#x60; being operated on.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>






    <!--Todo: process Response Object and its headers, schema, examples -->


    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">204</h4>
    An empty response.

        <a href="#"></a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="deleteProducts"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="delete"><code class="huge"><span class="http-method">delete</span> /catalog/products</code></pre></div>
    <div class="method-summary"> (<span class="nickname">deleteProducts</span>)</div>
    <div class="method-notes">Deletes one or more &#x60;Product&#x60; objects from the BigCommerce Catalog.
</div>


    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>



    <h3 class="field-label">Query parameters</h3>
    <div class="field-items">
      <div class="param">name (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by name.
 </div><div class="param">sku (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by sku.
 </div><div class="param">price (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by price.
 </div><div class="param">weight (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by weight.
 </div><div class="param">condition (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by condition.
 </div><div class="param">brand_id (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by brand_id.
 </div><div class="param">date_modified (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by date_modified.
 format: date-time</div><div class="param">date_last_imported (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by date_last_imported.
 format: date-time</div><div class="param">is_visible (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by is_visible.
 </div><div class="param">is_featured (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by is_featured.
 </div><div class="param">inventory_level (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by inventory_level.
 </div><div class="param">total_sold (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by total_sold.
 </div><div class="param">type (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by type: &#x60;physical&#x60; or &#x60;digital&#x60;.
 </div><div class="param">categories (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by categories.
 </div><div class="param">keyword (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by keywords found in the name, description, sku, keywords, or brand name.
 </div>
    </div>  <!-- field-items -->



    <!--Todo: process Response Object and its headers, schema, examples -->


    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">204</h4>
    An empty response.

        <a href="#"></a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="deleteVariantById"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="delete"><code class="huge"><span class="http-method">delete</span> /catalog/products/{product_id}/variants/{variant_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">deleteVariantById</span>)</div>
    <div class="method-notes">Deletes a &#x60;Variant&#x60;.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">variant_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Variant&#x60; to which the resource belongs.
 format: int</div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>






    <!--Todo: process Response Object and its headers, schema, examples -->


    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">204</h4>
    An empty response.

        <a href="#"></a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="deleteVariantMetafieldById"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="delete"><code class="huge"><span class="http-method">delete</span> /catalog/products/{product_id}/variants/{variant_id}/metafields/{metafield_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">deleteVariantMetafieldById</span>)</div>
    <div class="method-notes">Deletes a &#x60;Metafield&#x60;.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">metafield_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Metafield&#x60;.
 format: int</div><div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">variant_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Variant&#x60; to which the resource belongs.
 format: int</div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>






    <!--Todo: process Response Object and its headers, schema, examples -->


    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">204</h4>
    An empty response.

        <a href="#"></a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getBrandById"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/brands/{brand_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getBrandById</span>)</div>
    <div class="method-notes">Gets a &#x60;Brand&#x60; object.</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">brand_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Brand&#x60; to which the resource belongs.
 format: int</div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>





    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#BrandResponse">BrandResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A &#x60;Brand&#x60; object.

        <a href="#BrandResponse">BrandResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getBrandMetafieldByBrandId"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/brands/{brand_id}/metafields/{metafield_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getBrandMetafieldByBrandId</span>)</div>
    <div class="method-notes">Gets a &#x60;Metafield&#x60;, by &#x60;category_id&#x60;.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">metafield_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Metafield&#x60;.
 format: int</div><div class="param">brand_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Brand&#x60; to which the resource belongs.
 format: int</div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>





    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#MetafieldResponse">MetafieldResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A &#x60;Metafield&#x60; object.

        <a href="#MetafieldResponse">MetafieldResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getBrandMetafieldsByBrandId"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/brands/{brand_id}/metafields</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getBrandMetafieldsByBrandId</span>)</div>
    <div class="method-notes">Gets a &#x60;Metafield&#x60; object list, by &#x60;category_id&#x60;.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">brand_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Brand&#x60; to which the resource belongs.
 format: int</div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>



    <h3 class="field-label">Query parameters</h3>
    <div class="field-items">
      <div class="param">page (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Control the page in a limited list of products.
 </div><div class="param">limit (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Control the items per page.
 </div><div class="param">key (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter based on a metafield&#39;s key.
 </div><div class="param">namespace (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter based on a metafield&#39;s key.
 </div>
    </div>  <!-- field-items -->


    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#MetaFieldCollectionResponse">MetaFieldCollectionResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : [ "" ],
  "meta" : {
    "pagination" : {
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
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    An array of metafields and metadata.

        <a href="#MetaFieldCollectionResponse">MetaFieldCollectionResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getBrands"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/brands</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getBrands</span>)</div>
    <div class="method-notes">Gets &#x60;Brand&#x60; objects.
</div>


    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>



    <h3 class="field-label">Query parameters</h3>
    <div class="field-items">
      <div class="param">name (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by name.
 </div><div class="param">page_title (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by page_title.
 </div><div class="param">page (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Control the page in a limited list of products.
 </div><div class="param">limit (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Control the items per page.
 </div>
    </div>  <!-- field-items -->


    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#BrandCollectionResponse">BrandCollectionResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : [ "" ],
  "meta" : {
    "pagination" : {
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
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    An array of brand objects and metadata.

        <a href="#BrandCollectionResponse">BrandCollectionResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getCategories"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/categories</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getCategories</span>)</div>
    <div class="method-notes">Returns a paginated categories collection from the BigCommerce Catalog.
</div>


    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>



    <h3 class="field-label">Query parameters</h3>
    <div class="field-items">
      <div class="param">name (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by name.
 </div><div class="param">parent_id (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by parent_id.
 </div><div class="param">page_title (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by page_title.
 </div><div class="param">keyword (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by keywords.
 </div><div class="param">is_visible (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by is_visible.
 </div><div class="param">page (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Control the page in a limited list of products.
 </div><div class="param">limit (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Control the items per page.
 </div>
    </div>  <!-- field-items -->


    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#CategoryCollectionResponse">CategoryCollectionResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : [ "" ],
  "meta" : {
    "pagination" : {
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
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    An array of category objects and metadata.

        <a href="#CategoryCollectionResponse">CategoryCollectionResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getCategoryById"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/categories/{category_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getCategoryById</span>)</div>
    <div class="method-notes">Returns a &#x60;Category&#x60; from the BigCommerce Catalog.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">category_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Category&#x60; to which the resource belongs.
 format: int</div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>





    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#CategoryResponse">CategoryResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A category object.

        <a href="#CategoryResponse">CategoryResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getCategoryMetafieldByCategoryId"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/categories/{category_id}/metafields/{metafield_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getCategoryMetafieldByCategoryId</span>)</div>
    <div class="method-notes">Gets a &#x60;Metafield&#x60; by category_id.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">metafield_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Metafield&#x60;.
 format: int</div><div class="param">category_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Category&#x60; to which the resource belongs.
 format: int</div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>





    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#MetafieldResponse">MetafieldResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A &#x60;Metafield&#x60; object.

        <a href="#MetafieldResponse">MetafieldResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getCategoryMetafieldsByCategoryId"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/categories/{category_id}/metafields</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getCategoryMetafieldsByCategoryId</span>)</div>
    <div class="method-notes">Gets a &#x60;Metafield&#x60; object list, by category_id.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">category_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Category&#x60; to which the resource belongs.
 format: int</div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>



    <h3 class="field-label">Query parameters</h3>
    <div class="field-items">
      <div class="param">page (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Control the page in a limited list of products.
 </div><div class="param">limit (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Control the items per page.
 </div><div class="param">key (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter based on a metafield&#39;s key.
 </div><div class="param">namespace (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter based on a metafield&#39;s key.
 </div>
    </div>  <!-- field-items -->


    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#MetaFieldCollectionResponse">MetaFieldCollectionResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : [ "" ],
  "meta" : {
    "pagination" : {
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
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    An array of metafields and metadata.

        <a href="#MetaFieldCollectionResponse">MetaFieldCollectionResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getCategoryTree"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/categories/tree</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getCategoryTree</span>)</div>
    <div class="method-notes">Returns the categories tree, a nested lineage of the categories with parent-&gt;child relationship. The &#x60;Category&#x60; objects returned are simplified versions of the category objects returned in the rest of this API.
</div>


    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>





    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#CategoryTreeCollectionResponse">CategoryTreeCollectionResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : [ {
    "is_visible" : true,
    "children" : [ "" ],
    "parent_id" : 123,
    "name" : "aeiou",
    "id" : 123,
    "url" : "aeiou"
  } ],
  "meta" : {
    "pagination" : {
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
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A array of nested category tree objects and metadata.

        <a href="#CategoryTreeCollectionResponse">CategoryTreeCollectionResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getComplexRuleById"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/products/{product_id}/complex-rules/{complex_rule_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getComplexRuleById</span>)</div>
    <div class="method-notes">Gets a &#x60;ComplexRule&#x60; by product_id.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">complex_rule_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;ComplexRule&#x60;.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>





    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#ComplexRuleResponse">ComplexRuleResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A &#x60;Modifier&#x60; object.

        <a href="#ComplexRuleResponse">ComplexRuleResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getComplexRules"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/products/{product_id}/complex-rules</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getComplexRules</span>)</div>
    <div class="method-notes">Gets an array of &#x60;ComplexRule&#x60; objects.</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>





    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#ComplexRuleCollectionResponse">ComplexRuleCollectionResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : [ "" ],
  "meta" : {
    "pagination" : {
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
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    An array of &#x60;ComplexRule&#x60; objects and metadata.

        <a href="#ComplexRuleCollectionResponse">ComplexRuleCollectionResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getCustomFieldById"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/products/{product_id}/custom-fields/{custom_field_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getCustomFieldById</span>)</div>
    <div class="method-notes">Gets a &#x60;CustomField&#x60; by &#x60;product_id&#x60; and &#x60;custom_field_id&#x60;.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">custom_field_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;CustomField&#x60;.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>





    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#CustomFieldResponse">CustomFieldResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A &#x60;CustomField&#x60; object.

        <a href="#CustomFieldResponse">CustomFieldResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getCustomFields"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/products/{product_id}/custom-fields</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getCustomFields</span>)</div>
    <div class="method-notes">Gets an array of &#x60;CustomField&#x60; objects.</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>





    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#CustomFieldCollectionResponse">CustomFieldCollectionResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : [ "" ],
  "meta" : {
    "pagination" : {
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
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    An array of &#x60;CustomField&#x60; objects and metadata.

        <a href="#CustomFieldCollectionResponse">CustomFieldCollectionResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getModifierById"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/products/{product_id}/modifiers/{modifier_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getModifierById</span>)</div>
    <div class="method-notes">Gets a &#x60;Modifier&#x60; by product_id and modifier_id.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">modifier_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Modifier&#x60;.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>





    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#ModifierResponse">ModifierResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A &#x60;Modifier&#x60; object.

        <a href="#ModifierResponse">ModifierResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getModifierValueById"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/products/{product_id}/modifiers/{modifier_id}/values/{value_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getModifierValueById</span>)</div>
    <div class="method-notes">Gets a &#x60;ModifierValue&#x60; by product_id, modifier_id and value_id.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">modifier_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Modifier&#x60;.
 </div><div class="param">value_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Modifier/Option Value&#x60;.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>





    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#ModifierValueResponse">ModifierValueResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A &#x60;ModifierValue&#x60; object.

        <a href="#ModifierValueResponse">ModifierValueResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getModifierValues"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/products/{product_id}/modifiers/{modifier_id}/values</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getModifierValues</span>)</div>
    <div class="method-notes">Gets an array of &#x60;ModifierValue&#x60; objects.</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">modifier_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Modifier&#x60;.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>





    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#ModifierValueCollectionResponse">ModifierValueCollectionResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : [ "" ],
  "meta" : {
    "pagination" : {
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
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    An array of modifier values and metadata.

        <a href="#ModifierValueCollectionResponse">ModifierValueCollectionResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getModifiers"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/products/{product_id}/modifiers</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getModifiers</span>)</div>
    <div class="method-notes">Gets an array of &#x60;Modifier&#x60; objects.</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>





    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#ModifierCollectionResponse">ModifierCollectionResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : [ "" ],
  "meta" : {
    "pagination" : {
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
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    An array of modifiers and metadata.

        <a href="#ModifierCollectionResponse">ModifierCollectionResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getOptionById"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/products/{product_id}/options/{option_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getOptionById</span>)</div>
    <div class="method-notes">Gets &#x60;Option&#x60; object by product ID and option id.</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">option_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Option&#x60;.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>





    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#OptionResponse">OptionResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    An &#x60;Option&#x60; object.

        <a href="#OptionResponse">OptionResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getOptionValueById"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/products/{product_id}/options/{option_id}/values/{value_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getOptionValueById</span>)</div>
    <div class="method-notes">Gets a &#x60;OptionValue&#x60; by product_id, option_id and value_id.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">option_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Option&#x60;.
 </div><div class="param">value_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Modifier/Option Value&#x60;.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>





    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#OptionValueResponse">OptionValueResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A &#x60;OptionValue&#x60; object.

        <a href="#OptionValueResponse">OptionValueResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getOptionValues"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/products/{product_id}/options/{option_id}/values</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getOptionValues</span>)</div>
    <div class="method-notes">Gets an array of &#x60;OptionValue&#x60; objects.</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">option_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Option&#x60;.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>





    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#OptionValueCollectionResponse">OptionValueCollectionResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : [ "" ],
  "meta" : {
    "pagination" : {
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
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    An array of option values and metadata.

        <a href="#OptionValueCollectionResponse">OptionValueCollectionResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getOptions"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/products/{product_id}/options</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getOptions</span>)</div>
    <div class="method-notes">Gets an array of &#x60;Option&#x60; objects.</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>





    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#OptionCollectionResponse">OptionCollectionResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : [ "" ],
  "meta" : {
    "pagination" : {
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
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    An array of options and metadata.

        <a href="#OptionCollectionResponse">OptionCollectionResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getProductById"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/products/{product_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getProductById</span>)</div>
    <div class="method-notes">Returns a &#x60;Product&#x60; from the BigCommerce Catalog.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>



    <h3 class="field-label">Query parameters</h3>
    <div class="field-items">
      <div class="param">include (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Include sub-resources on a product, with a comma-separated list. Valid expansions currently include &#x60;variants&#x60;, &#x60;images&#x60;, &#x60;custom_fields&#x60;, and &#x60;bulk_pricing_rules&#x60;.
 </div>
    </div>  <!-- field-items -->


    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#ProductResponse">ProductResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A product.

        <a href="#ProductResponse">ProductResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getProductImageById"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/products/{product_id}/images/{image_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getProductImageById</span>)</div>
    <div class="method-notes">Gets image on a product.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">image_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Image&#x60; that is being operated on.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>





    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#ProductImageResponse">ProductImageResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    An array of product images and metadata.

        <a href="#ProductImageResponse">ProductImageResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getProductImages"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/products/{product_id}/images</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getProductImages</span>)</div>
    <div class="method-notes">Gets all images on a product.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>





    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#ProductImageCollectionResponse">ProductImageCollectionResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : [ "" ],
  "meta" : {
    "pagination" : {
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
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    List of product images and metadata.

        <a href="#ProductImageCollectionResponse">ProductImageCollectionResponse</a>
    <h4 class="field-label">204</h4>
    There are not any images on this product.

        <a href="#"></a>
    <h4 class="field-label">404</h4>
    The product ID does not exist.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getProductMetafieldByProductId"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/products/{product_id}/metafields/{metafield_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getProductMetafieldByProductId</span>)</div>
    <div class="method-notes">Gets a &#x60;Metafield&#x60;, by &#x60;product_id&#x60;.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">metafield_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Metafield&#x60;.
 format: int</div><div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>





    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#MetafieldResponse">MetafieldResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A &#x60;Metafield&#x60; object.

        <a href="#MetafieldResponse">MetafieldResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getProductMetafieldsByProductId"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/products/{product_id}/metafields</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getProductMetafieldsByProductId</span>)</div>
    <div class="method-notes">Gets a &#x60;Metafield&#x60; object list, by &#x60;product_id&#x60;.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>



    <h3 class="field-label">Query parameters</h3>
    <div class="field-items">
      <div class="param">page (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Control the page in a limited list of products.
 </div><div class="param">limit (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Control the items per page.
 </div><div class="param">key (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter based on a metafield&#39;s key.
 </div><div class="param">namespace (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter based on a metafield&#39;s key.
 </div>
    </div>  <!-- field-items -->


    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#MetaFieldCollectionResponse">MetaFieldCollectionResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : [ "" ],
  "meta" : {
    "pagination" : {
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
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    An array of metafields and metadata.

        <a href="#MetaFieldCollectionResponse">MetaFieldCollectionResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getProductVideoById"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/products/{product_id}/videos/{video_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getProductVideoById</span>)</div>
    <div class="method-notes">Gets video on a product.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">video_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Video&#x60; being operated on.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>





    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#ProductVideoResponse">ProductVideoResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    An array of product videos and metadata.

        <a href="#ProductVideoResponse">ProductVideoResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getProductVideos"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/products/{product_id}/videos</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getProductVideos</span>)</div>
    <div class="method-notes">Gets all videos on a product.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>





    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#ProductVideoCollectionResponse">ProductVideoCollectionResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : [ "" ],
  "meta" : {
    "pagination" : {
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
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    List of product videos and metadata.

        <a href="#ProductVideoCollectionResponse">ProductVideoCollectionResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getProducts"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/products</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getProducts</span>)</div>
    <div class="method-notes">Returns a paginated collection of &#x60;Products&#x60; objects from the BigCommerce Catalog.
</div>


    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>



    <h3 class="field-label">Query parameters</h3>
    <div class="field-items">
      <div class="param">id (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by id.
 </div><div class="param">name (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by name.
 </div><div class="param">sku (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by sku.
 </div><div class="param">upc (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by upc.
 </div><div class="param">price (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by price.
 </div><div class="param">weight (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by weight.
 </div><div class="param">condition (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by condition.
 </div><div class="param">brand_id (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by brand_id.
 </div><div class="param">date_modified (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by date_modified.
 format: date-time</div><div class="param">date_last_imported (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by date_last_imported.
 format: date-time</div><div class="param">is_visible (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by is_visible.
 </div><div class="param">is_featured (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by is_featured.
 </div><div class="param">is_free_shipping (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by is_free_shipping.
 </div><div class="param">inventory_level (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by inventory_level.
 </div><div class="param">inventory_low (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by inventory_low; values: 1, 0.
 </div><div class="param">out_of_stock (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by out_of_stock. To enable the filter, pass &#x60;out_of_stock&#x60;&#x3D;&#x60;1&#x60;.
 </div><div class="param">total_sold (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by total_sold.
 </div><div class="param">type (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by type: &#x60;physical&#x60; or &#x60;digital&#x60;.
 </div><div class="param">categories (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by categories.
 </div><div class="param">keyword (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by keywords found in the name, description, sku, keywords, or brand name.
 </div><div class="param">keyword_context (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Set context for a product search.
 </div><div class="param">channel_id (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by channel.
 </div><div class="param">status (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by status.
 </div><div class="param">include (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Include sub-resources on a product, with a comma-separated list. Valid expansions currently include &#x60;variants&#x60;, &#x60;images&#x60;, &#x60;custom_fields&#x60;, and &#x60;bulk_pricing_rules&#x60;.
 </div><div class="param">availability (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by availability. Values are: available, disabled, preorder.
 </div><div class="param">page (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Control the page in a limited list of products.
 </div><div class="param">limit (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Control the items per page.
 </div><div class="param">direction (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Sort direction. Values are: asc, desc.
 </div><div class="param">sort (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Field name to sort by.
 </div>
    </div>  <!-- field-items -->


    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#ProductCollectionResponse">ProductCollectionResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : [ "" ],
  "meta" : {
    "pagination" : {
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
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    An array of products and metadata.

        <a href="#ProductCollectionResponse">ProductCollectionResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getVariantById"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/products/{product_id}/variants/{variant_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getVariantById</span>)</div>
    <div class="method-notes">Gets a &#x60;Variant&#x60; object.</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">variant_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Variant&#x60; to which the resource belongs.
 format: int</div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>





    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#VariantResponse">VariantResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A variant and metadata.

        <a href="#VariantResponse">VariantResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getVariantMetafieldByProductIdAndVariantId"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/products/{product_id}/variants/{variant_id}/metafields/{metafield_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getVariantMetafieldByProductIdAndVariantId</span>)</div>
    <div class="method-notes">Gets a &#x60;Metafield&#x60;, by product_id and variant_id.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">metafield_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Metafield&#x60;.
 format: int</div><div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">variant_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Variant&#x60; to which the resource belongs.
 format: int</div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>





    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#MetafieldResponse">MetafieldResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A &#x60;Metafield&#x60; object.

        <a href="#MetafieldResponse">MetafieldResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getVariantMetafieldsByProductIdAndVariantId"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/products/{product_id}/variants/{variant_id}/metafields</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getVariantMetafieldsByProductIdAndVariantId</span>)</div>
    <div class="method-notes">Gets a &#x60;Metafield&#x60; object list, by product_id and variant_id.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">variant_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Variant&#x60; to which the resource belongs.
 format: int</div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>



    <h3 class="field-label">Query parameters</h3>
    <div class="field-items">
      <div class="param">page (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Control the page in a limited list of products.
 </div><div class="param">limit (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Control the items per page.
 </div><div class="param">key (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter based on a metafield&#39;s key.
 </div><div class="param">namespace (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter based on a metafield&#39;s key.
 </div>
    </div>  <!-- field-items -->


    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#MetaFieldCollectionResponse">MetaFieldCollectionResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : [ "" ],
  "meta" : {
    "pagination" : {
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
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    An array of metafields and metadata.

        <a href="#MetaFieldCollectionResponse">MetaFieldCollectionResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getVariants"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/variants</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getVariants</span>)</div>
    <div class="method-notes">Returns a &#x60;Variant&#x60; object list from the BigCommerce Catalog.
</div>


    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>



    <h3 class="field-label">Query parameters</h3>
    <div class="field-items">
      <div class="param">id (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by id.
 </div><div class="param">sku (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by sku.
 </div><div class="param">page (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Control the page in a limited list of products.
 </div><div class="param">limit (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Control the items per page.
 </div>
    </div>  <!-- field-items -->


    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#VariantCollectionResponse">VariantCollectionResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : [ "" ],
  "meta" : {
    "pagination" : {
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
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    An array of variants and metadata.

        <a href="#VariantCollectionResponse">VariantCollectionResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getVariantsByProductId"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /catalog/products/{product_id}/variants</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getVariantsByProductId</span>)</div>
    <div class="method-notes">Returns a &#x60;Variant&#x60; object list from the BigCommerce Catalog.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>



    <h3 class="field-label">Query parameters</h3>
    <div class="field-items">
      <div class="param">page (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Control the page in a limited list of products.
 </div><div class="param">limit (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Control the items per page.
 </div>
    </div>  <!-- field-items -->


    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#VariantCollectionResponse">VariantCollectionResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : [ "" ],
  "meta" : {
    "pagination" : {
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
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    An array of variants and metadata.

        <a href="#VariantCollectionResponse">VariantCollectionResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="updateBrand"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="put"><code class="huge"><span class="http-method">put</span> /catalog/brands/{brand_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">updateBrand</span>)</div>
    <div class="method-notes">Updates a &#x60;Brand&#x60; in the BigCommerce Catalog.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">brand_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Brand&#x60; to which the resource belongs.
 format: int</div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">brand <a href="#BrandPut">BrandPut</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; Returns a &#x60;Brand&#x60; from the BigCommerce Catalog.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#BrandResponse">BrandResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A &#x60;Brand&#x60; object.

        <a href="#BrandResponse">BrandResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
    <h4 class="field-label">409</h4>
    The &#x60;Brand&#x60; was in conflict with another product. This is the result of duplicate unique values, such as &#x60;name&#x60;.

        <a href="#ErrorResponse">ErrorResponse</a>
    <h4 class="field-label">422</h4>
    The &#x60;Brand&#x60; was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.

        <a href="#ErrorResponse">ErrorResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="updateBrandMetafield"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="put"><code class="huge"><span class="http-method">put</span> /catalog/brands/{brand_id}/metafields/{metafield_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">updateBrandMetafield</span>)</div>
    <div class="method-notes">Updates a &#x60;Metafield&#x60; object.</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">metafield_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Metafield&#x60;.
 format: int</div><div class="param">brand_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Brand&#x60; to which the resource belongs.
 format: int</div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">Metafield <a href="#MetafieldPut">MetafieldPut</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; A &#x60;Metafield&#x60; object.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#MetafieldResponse">MetafieldResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A metafield and metadata.

        <a href="#MetafieldResponse">MetafieldResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="updateCategory"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="put"><code class="huge"><span class="http-method">put</span> /catalog/categories/{category_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">updateCategory</span>)</div>
    <div class="method-notes">Updates a &#x60;Category&#x60; in the BigCommerce Catalog.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">category_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Category&#x60; to which the resource belongs.
 format: int</div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">category <a href="#CategoryPut">CategoryPut</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; A BigCommerce &#x60;Category&#x60; object.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#CategoryResponse">CategoryResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A category object.

        <a href="#CategoryResponse">CategoryResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
    <h4 class="field-label">409</h4>
    The &#x60;Category&#x60; was in conflict with another category. This is the result of duplicate unique values, such as &#x60;name&#x60; or &#x60;custom_url&#x60;.

        <a href="#ErrorResponse">ErrorResponse</a>
    <h4 class="field-label">422</h4>
    The &#x60;Category&#x60; was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.

        <a href="#ErrorResponse">ErrorResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="updateCategoryMetafield"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="put"><code class="huge"><span class="http-method">put</span> /catalog/categories/{category_id}/metafields/{metafield_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">updateCategoryMetafield</span>)</div>
    <div class="method-notes">Updates a &#x60;Metafield&#x60; object.</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">metafield_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Metafield&#x60;.
 format: int</div><div class="param">category_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Category&#x60; to which the resource belongs.
 format: int</div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">Metafield <a href="#MetafieldPut">MetafieldPut</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; A &#x60;Metafield&#x60; object.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#MetafieldResponse">MetafieldResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A metafield and metadata.

        <a href="#MetafieldResponse">MetafieldResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="updateComplexRule"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="put"><code class="huge"><span class="http-method">put</span> /catalog/products/{product_id}/complex-rules/{complex_rule_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">updateComplexRule</span>)</div>
    <div class="method-notes">Updates an Product&#39;s &#x60;ComplexRule&#x60;, based on the &#x60;product_id&#x60; and &#x60;complex_rule_id&#x60;.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">complex_rule_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;ComplexRule&#x60;.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">ComplexRule <a href="#ComplexRulePut">ComplexRulePut</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; &#x60;ComplexRule&#x60; object.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#ComplexRuleResponse">ComplexRuleResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A &#x60;ComplexRule&#x60; object.

        <a href="#ComplexRuleResponse">ComplexRuleResponse</a>
    <h4 class="field-label">409</h4>
    The &#x60;ComplexRule&#x60; was in conflict with another &#x60;ComplexRule&#x60;. This is the result of duplicate conditions.

        <a href="#ErrorResponse">ErrorResponse</a>
    <h4 class="field-label">422</h4>
    The &#x60;ComplexRule&#x60; was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.

        <a href="#ErrorResponse">ErrorResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="updateCustomField"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="put"><code class="huge"><span class="http-method">put</span> /catalog/products/{product_id}/custom-fields/{custom_field_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">updateCustomField</span>)</div>
    <div class="method-notes">Updates an Product&#39;s &#x60;CustomField&#x60;, based on the &#x60;product_id&#x60; and &#x60;custom_field_id&#x60;.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">custom_field_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;CustomField&#x60;.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">CustomField <a href="#CustomFieldPut">CustomFieldPut</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; &#x60;CustomField&#x60; object.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#CustomFieldResponse">CustomFieldResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A &#x60;CustomField&#x60; object.

        <a href="#CustomFieldResponse">CustomFieldResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
    <h4 class="field-label">422</h4>
    The &#x60;CustomField&#x60; was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.

        <a href="#ErrorResponse">ErrorResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="updateModifier"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="put"><code class="huge"><span class="http-method">put</span> /catalog/products/{product_id}/modifiers/{modifier_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">updateModifier</span>)</div>
    <div class="method-notes">Updates an Product&#39;s &#x60;Modifier&#x60; based on the product_id and modifier_id.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">modifier_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Modifier&#x60;.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">modifier <a href="#ModifierPut">ModifierPut</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; A BigCommerce &#x60;Modifier&#x60; object.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#ModifierResponse">ModifierResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A &#x60;Modifier&#x60; object.

        <a href="#ModifierResponse">ModifierResponse</a>
    <h4 class="field-label">409</h4>
    The &#x60;Modifier&#x60; was in conflict with another modifier or option. This is the result of duplicate unique fields, such as &#x60;name&#x60;.

        <a href="#ErrorResponse">ErrorResponse</a>
    <h4 class="field-label">422</h4>
    The &#x60;Modifier&#x60; was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.

        <a href="#ErrorResponse">ErrorResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="updateModifierValue"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="put"><code class="huge"><span class="http-method">put</span> /catalog/products/{product_id}/modifiers/{modifier_id}/values/{value_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">updateModifierValue</span>)</div>
    <div class="method-notes">Updates an Product&#39;s &#x60;ModifierValue&#x60; based on the product_id, modifier_id and value_id.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">modifier_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Modifier&#x60;.
 </div><div class="param">value_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Modifier/Option Value&#x60;.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">ModifierValue <a href="#ModifierValuePut">ModifierValuePut</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; A BigCommerce &#x60;ModifierValue&#x60; object.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#ModifierValueResponse">ModifierValueResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A &#x60;ModifierValue&#x60; object.

        <a href="#ModifierValueResponse">ModifierValueResponse</a>
    <h4 class="field-label">422</h4>
    The &#x60;ModifierValue&#x60; was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.

        <a href="#ErrorResponse">ErrorResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="updateOption"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="put"><code class="huge"><span class="http-method">put</span> /catalog/products/{product_id}/options/{option_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">updateOption</span>)</div>
    <div class="method-notes">Updates a Product&#39;s &#x60;Option&#x60;, based on the product_id and option_id.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">option_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Option&#x60;.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">option <a href="#OptionPut">OptionPut</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; A BigCommerce &#x60;Option&#x60; object.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#OptionResponse">OptionResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    An &#x60;Option&#x60; object.

        <a href="#OptionResponse">OptionResponse</a>
    <h4 class="field-label">409</h4>
    The &#x60;Option&#x60; was in conflict with another option. This is the result of duplicate unique fields, such as &#x60;name&#x60;.

        <a href="#ErrorResponse">ErrorResponse</a>
    <h4 class="field-label">422</h4>
    The &#x60;Option&#x60; was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.

        <a href="#ErrorResponse">ErrorResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="updateOptionValue"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="put"><code class="huge"><span class="http-method">put</span> /catalog/products/{product_id}/options/{option_id}/values/{value_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">updateOptionValue</span>)</div>
    <div class="method-notes">Updates an Product&#39;s &#x60;OptionValue&#x60; based on the product_id, option_id and value_id.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">option_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Option&#x60;.
 </div><div class="param">value_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Modifier/Option Value&#x60;.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">OptionValue <a href="#OptionValuePut">OptionValuePut</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; A BigCommerce &#x60;OptionValue&#x60; object.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#OptionValueResponse">OptionValueResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A &#x60;OptionValue&#x60; object.

        <a href="#OptionValueResponse">OptionValueResponse</a>
    <h4 class="field-label">422</h4>
    The &#x60;OptionValue&#x60; was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.

        <a href="#ErrorResponse">ErrorResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="updateProduct"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="put"><code class="huge"><span class="http-method">put</span> /catalog/products/{product_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">updateProduct</span>)</div>
    <div class="method-notes">Updates a &#x60;Product&#x60; in the BigCommerce Catalog.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">product <a href="#ProductPut">ProductPut</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; A BigCommerce &#x60;Product&#x60; object.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#ProductResponse">ProductResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A product.

        <a href="#ProductResponse">ProductResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
    <h4 class="field-label">409</h4>
    &#x60;Product&#x60; was in conflict with another product. This is the result of duplicate unique values such as name or SKU, a missing category, brand, or tax_class that the product is being associate to, or a conflicting bulk pricing rule.

        <a href="#ErrorResponse">ErrorResponse</a>
    <h4 class="field-label">422</h4>
    &#x60;Product&#x60; was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.

        <a href="#ErrorResponse">ErrorResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="updateProductImage"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="put"><code class="huge"><span class="http-method">put</span> /catalog/products/{product_id}/images/{image_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">updateProductImage</span>)</div>
    <div class="method-notes">Updates an image on a product. Publicly accessible URLs and files (form post) are valid parameters.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">image_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Image&#x60; that is being operated on.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">productImage <a href="#ProductImagePut">ProductImagePut</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; A BigCommerce &#x60;ProductImage&#x60; object.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#ProductImageResponse">ProductImageResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A product image.

        <a href="#ProductImageResponse">ProductImageResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="updateProductMetafield"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="put"><code class="huge"><span class="http-method">put</span> /catalog/products/{product_id}/metafields/{metafield_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">updateProductMetafield</span>)</div>
    <div class="method-notes">Updates a &#x60;Metafield&#x60; object.</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">metafield_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Metafield&#x60;.
 format: int</div><div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">Metafield <a href="#MetafieldPut">MetafieldPut</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; A &#x60;Metafield&#x60; object.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#MetafieldResponse">MetafieldResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A metafield and metadata.

        <a href="#MetafieldResponse">MetafieldResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="updateProductVideo"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="put"><code class="huge"><span class="http-method">put</span> /catalog/products/{product_id}/videos/{video_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">updateProductVideo</span>)</div>
    <div class="method-notes">Updates a video on a product.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">video_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Video&#x60; being operated on.
 </div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">productVideo <a href="#ProductVideoPut">ProductVideoPut</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; A BigCommerce &#x60;ProductVideo&#x60; object.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#ProductVideoResponse">ProductVideoResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A product video.

        <a href="#ProductVideoResponse">ProductVideoResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="updateVariant"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="put"><code class="huge"><span class="http-method">put</span> /catalog/products/{product_id}/variants/{variant_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">updateVariant</span>)</div>
    <div class="method-notes">Updates a &#x60;Variant&#x60; object.</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">variant_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Variant&#x60; to which the resource belongs.
 format: int</div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">Variant <a href="#VariantPut">VariantPut</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; A &#x60;Variant&#x60; object.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#VariantResponse">VariantResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A variant and metadata.

        <a href="#VariantResponse">VariantResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="updateVariantMetafield"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="put"><code class="huge"><span class="http-method">put</span> /catalog/products/{product_id}/variants/{variant_id}/metafields/{metafield_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">updateVariantMetafield</span>)</div>
    <div class="method-notes">Updates a &#x60;Metafield&#x60; object.</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">metafield_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Metafield&#x60;.
 format: int</div><div class="param">product_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Product&#x60; the resource belongs to.
 </div><div class="param">variant_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Variant&#x60; to which the resource belongs.
 format: int</div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">Metafield <a href="#MetafieldPut">MetafieldPut</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; A &#x60;Metafield&#x60; object.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#MetafieldResponse">MetafieldResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A metafield and metadata.

        <a href="#MetafieldResponse">MetafieldResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <h1><a name="Customers">Customers API</a></h1>
  <div class="method"><a name="createSubscriber"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="post"><code class="huge"><span class="http-method">post</span> /customers/subscribers</code></pre></div>
    <div class="method-summary"> (<span class="nickname">createSubscriber</span>)</div>
    <div class="method-notes">Creates a &#x60;Subscriber&#x60; object.
</div>


    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">subscriber <a href="#SubscriberPost">SubscriberPost</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; &#x60;Subscriber&#x60; object.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#SubscriberResponse">SubscriberResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A &#x60;Subscriber&#x60; object.

        <a href="#SubscriberResponse">SubscriberResponse</a>
    <h4 class="field-label">409</h4>
    The &#x60;Subscriber&#x60; was in conflict with another subscriber. This is the result of duplicate unique values such as email

        <a href="#ErrorResponse">ErrorResponse</a>
    <h4 class="field-label">422</h4>
    The &#x60;Subscriber&#x60; was not valid. This is the result of missing required fields or invalid data. See the response for more details.

        <a href="#ErrorResponse">ErrorResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="deleteSubscriberById"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="delete"><code class="huge"><span class="http-method">delete</span> /customers/subscribers/{subscriber_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">deleteSubscriberById</span>)</div>
    <div class="method-notes">Deletes a &#x60;Subscriber&#x60; object.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">subscriber_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Subscriber&#x60; requested.
 format: int</div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>






    <!--Todo: process Response Object and its headers, schema, examples -->


    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">204</h4>
    An empty response.

        <a href="#"></a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="deleteSubscribers"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="delete"><code class="huge"><span class="http-method">delete</span> /customers/subscribers</code></pre></div>
    <div class="method-summary"> (<span class="nickname">deleteSubscribers</span>)</div>
    <div class="method-notes">Deletes a Subscriber or Subscribers from BigCommerce Customers.
</div>


    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>



    <h3 class="field-label">Query parameters</h3>
    <div class="field-items">
      <div class="param">email (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by email.
 </div><div class="param">first_name (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by first_name.
 </div><div class="param">last_name (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by last_name.
 </div><div class="param">source (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by source.
 </div><div class="param">order_id (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by order_id.
 </div><div class="param">date_created (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by date_created.
 format: date-time</div><div class="param">date_modified (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by date_modified.
 format: date-time</div>
    </div>  <!-- field-items -->



    <!--Todo: process Response Object and its headers, schema, examples -->


    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">204</h4>
    An empty response.

        <a href="#"></a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getSubscriberById"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /customers/subscribers/{subscriber_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getSubscriberById</span>)</div>
    <div class="method-notes">Gets &#x60;Subscriber&#x60; object.</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">subscriber_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Subscriber&#x60; requested.
 format: int</div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>





    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#SubscriberResponse">SubscriberResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A &#x60;Subscriber&#x60; object.

        <a href="#SubscriberResponse">SubscriberResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="getSubscribers"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="get"><code class="huge"><span class="http-method">get</span> /customers/subscribers</code></pre></div>
    <div class="method-summary"> (<span class="nickname">getSubscribers</span>)</div>
    <div class="method-notes">Returns a paginated Subscribers collection.
</div>


    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>



    <h3 class="field-label">Query parameters</h3>
    <div class="field-items">
      <div class="param">email (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by email.
 </div><div class="param">first_name (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by first_name.
 </div><div class="param">last_name (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by last_name.
 </div><div class="param">source (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by source.
 </div><div class="param">order_id (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by order_id.
 </div><div class="param">date_created (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by date_created.
 format: date-time</div><div class="param">date_modified (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Filter items by date_modified.
 format: date-time</div><div class="param">page (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Control the page in a limited list of products.
 </div><div class="param">limit (optional)</div>

      <div class="param-desc"><span class="param-type">Query Parameter</span> &mdash; Control the items per page.
 </div>
    </div>  <!-- field-items -->


    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#SubscriberCollectionResponse">SubscriberCollectionResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : [ "" ],
  "meta" : {
    "pagination" : {
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
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    An array of subscriber objects and metadata.

        <a href="#SubscriberCollectionResponse">SubscriberCollectionResponse</a>
  </div> <!-- method -->
  <hr/>
  <div class="method"><a name="updateSubscriber"/>
    <div class="method-path">
    <a class="up" href="#__Methods">Up</a>
    <pre class="put"><code class="huge"><span class="http-method">put</span> /customers/subscribers/{subscriber_id}</code></pre></div>
    <div class="method-summary"> (<span class="nickname">updateSubscriber</span>)</div>
    <div class="method-notes">Updates a &#x60;Subscriber&#x60; object.
</div>

    <h3 class="field-label">Path parameters</h3>
    <div class="field-items">
      <div class="param">subscriber_id (required)</div>

      <div class="param-desc"><span class="param-type">Path Parameter</span> &mdash; The ID of the &#x60;Subscriber&#x60; requested.
 format: int</div>
    </div>  <!-- field-items -->

    <h3 class="field-label">Consumes</h3>
    This API call consumes the following media types via the <span class="heaader">Content-Type</span> request header:
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Request body</h3>
    <div class="field-items">
      <div class="param">subscriber <a href="#SubscriberPut">SubscriberPut</a> (required)</div>

      <div class="param-desc"><span class="param-type">Body Parameter</span> &mdash; Returns a &#x60;Subscriber&#x60; object.
 </div>

    </div>  <!-- field-items -->




    <h3 class="field-label">Return type</h3>
    <div class="return-type">
      <a href="#SubscriberResponse">SubscriberResponse</a>
      
    </div>

    <!--Todo: process Response Object and its headers, schema, examples -->

    <h3 class="field-label">Example data</h3>
    <div class="example-data-content-type">Content-Type: application/json</div>
    <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>

    <h3 class="field-label">Produces</h3>
    This API call produces the following media types according to the <span class="header">Accept</span> request header;
    the media type will be conveyed by the <span class="heaader">Content-Type</span> response header.
    <ul>
      <li><code>application/json</code></li>
    </ul>

    <h3 class="field-label">Responses</h3>
    <h4 class="field-label">200</h4>
    A &#x60;Subscriber&#x60; object.

        <a href="#SubscriberResponse">SubscriberResponse</a>
    <h4 class="field-label">404</h4>
    The resource was not found.

        <a href="#NotFound">NotFound</a>
    <h4 class="field-label">409</h4>
    The &#x60;Subscriber&#x60; was in conflict with another subscriber. This is the result of duplicate unique values, such as &#x60;email&#x60;.

        <a href="#ErrorResponse">ErrorResponse</a>
    <h4 class="field-label">422</h4>
    The &#x60;Subscriber&#x60; was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.

        <a href="#ErrorResponse">ErrorResponse</a>
  </div> <!-- method -->
  <hr/>

  <div class="up"><a href="#__Models">Up</a></div>
  <h2><a name="__Models">Models</a></h2>
  [ Jump to <a href="#__Methods">Methods</a> ]

  <h3>Table of Contents</h3>
  <ol>
    <li><a href="#Adjuster"><code>Adjuster</code></a></li>
    <li><a href="#BaseError"><code>BaseError</code></a></li>
    <li><a href="#Brand"><code>Brand</code></a></li>
    <li><a href="#BrandBase"><code>BrandBase</code></a></li>
    <li><a href="#BrandCollectionResponse"><code>BrandCollectionResponse</code></a></li>
    <li><a href="#BrandPost"><code>BrandPost</code></a></li>
    <li><a href="#BrandPut"><code>BrandPut</code></a></li>
    <li><a href="#BrandResponse"><code>BrandResponse</code></a></li>
    <li><a href="#BulkPricingRule"><code>BulkPricingRule</code></a></li>
    <li><a href="#CatalogSummary"><code>CatalogSummary</code></a></li>
    <li><a href="#CatalogSummaryResponse"><code>CatalogSummaryResponse</code></a></li>
    <li><a href="#Category"><code>Category</code></a></li>
    <li><a href="#CategoryBase"><code>CategoryBase</code></a></li>
    <li><a href="#CategoryCollectionResponse"><code>CategoryCollectionResponse</code></a></li>
    <li><a href="#CategoryNode"><code>CategoryNode</code></a></li>
    <li><a href="#CategoryPost"><code>CategoryPost</code></a></li>
    <li><a href="#CategoryPut"><code>CategoryPut</code></a></li>
    <li><a href="#CategoryResponse"><code>CategoryResponse</code></a></li>
    <li><a href="#CategoryTreeCollectionResponse"><code>CategoryTreeCollectionResponse</code></a></li>
    <li><a href="#CollectionMeta"><code>CollectionMeta</code></a></li>
    <li><a href="#ComplexRule"><code>ComplexRule</code></a></li>
    <li><a href="#ComplexRuleBase"><code>ComplexRuleBase</code></a></li>
    <li><a href="#ComplexRuleCollectionResponse"><code>ComplexRuleCollectionResponse</code></a></li>
    <li><a href="#ComplexRuleCondition"><code>ComplexRuleCondition</code></a></li>
    <li><a href="#ComplexRuleConditionBase"><code>ComplexRuleConditionBase</code></a></li>
    <li><a href="#ComplexRuleConditionPost"><code>ComplexRuleConditionPost</code></a></li>
    <li><a href="#ComplexRuleConditionPut"><code>ComplexRuleConditionPut</code></a></li>
    <li><a href="#ComplexRulePost"><code>ComplexRulePost</code></a></li>
    <li><a href="#ComplexRulePut"><code>ComplexRulePut</code></a></li>
    <li><a href="#ComplexRuleResponse"><code>ComplexRuleResponse</code></a></li>
    <li><a href="#CustomField"><code>CustomField</code></a></li>
    <li><a href="#CustomFieldBase"><code>CustomFieldBase</code></a></li>
    <li><a href="#CustomFieldCollectionResponse"><code>CustomFieldCollectionResponse</code></a></li>
    <li><a href="#CustomFieldPost"><code>CustomFieldPost</code></a></li>
    <li><a href="#CustomFieldPut"><code>CustomFieldPut</code></a></li>
    <li><a href="#CustomFieldResponse"><code>CustomFieldResponse</code></a></li>
    <li><a href="#CustomUrlCategory"><code>CustomUrlCategory</code></a></li>
    <li><a href="#CustomUrlProduct"><code>CustomUrlProduct</code></a></li>
    <li><a href="#DetailedErrors"><code>DetailedErrors</code></a></li>
    <li><a href="#ErrorResponse"><code>ErrorResponse</code></a></li>
    <li><a href="#ImageResponse"><code>ImageResponse</code></a></li>
    <li><a href="#Meta"><code>Meta</code></a></li>
    <li><a href="#MetaFieldCollectionResponse"><code>MetaFieldCollectionResponse</code></a></li>
    <li><a href="#Metafield"><code>Metafield</code></a></li>
    <li><a href="#MetafieldBase"><code>MetafieldBase</code></a></li>
    <li><a href="#MetafieldPost"><code>MetafieldPost</code></a></li>
    <li><a href="#MetafieldPut"><code>MetafieldPut</code></a></li>
    <li><a href="#MetafieldResponse"><code>MetafieldResponse</code></a></li>
    <li><a href="#Modifier"><code>Modifier</code></a></li>
    <li><a href="#ModifierBase"><code>ModifierBase</code></a></li>
    <li><a href="#ModifierCollectionResponse"><code>ModifierCollectionResponse</code></a></li>
    <li><a href="#ModifierPost"><code>ModifierPost</code></a></li>
    <li><a href="#ModifierPut"><code>ModifierPut</code></a></li>
    <li><a href="#ModifierResponse"><code>ModifierResponse</code></a></li>
    <li><a href="#ModifierValue"><code>ModifierValue</code></a></li>
    <li><a href="#ModifierValueBase"><code>ModifierValueBase</code></a></li>
    <li><a href="#ModifierValueBase_adjusters"><code>ModifierValueBase_adjusters</code></a></li>
    <li><a href="#ModifierValueBase_adjusters_purchasing_disabled"><code>ModifierValueBase_adjusters_purchasing_disabled</code></a></li>
    <li><a href="#ModifierValueCollectionResponse"><code>ModifierValueCollectionResponse</code></a></li>
    <li><a href="#ModifierValuePost"><code>ModifierValuePost</code></a></li>
    <li><a href="#ModifierValuePut"><code>ModifierValuePut</code></a></li>
    <li><a href="#ModifierValueResponse"><code>ModifierValueResponse</code></a></li>
    <li><a href="#NotFound"><code>NotFound</code></a></li>
    <li><a href="#Option"><code>Option</code></a></li>
    <li><a href="#OptionBase"><code>OptionBase</code></a></li>
    <li><a href="#OptionCollectionResponse"><code>OptionCollectionResponse</code></a></li>
    <li><a href="#OptionConfig"><code>OptionConfig</code></a></li>
    <li><a href="#OptionPost"><code>OptionPost</code></a></li>
    <li><a href="#OptionPut"><code>OptionPut</code></a></li>
    <li><a href="#OptionResponse"><code>OptionResponse</code></a></li>
    <li><a href="#OptionValue"><code>OptionValue</code></a></li>
    <li><a href="#OptionValueBase"><code>OptionValueBase</code></a></li>
    <li><a href="#OptionValueCollectionResponse"><code>OptionValueCollectionResponse</code></a></li>
    <li><a href="#OptionValuePost"><code>OptionValuePost</code></a></li>
    <li><a href="#OptionValueProductBase"><code>OptionValueProductBase</code></a></li>
    <li><a href="#OptionValueProductPost"><code>OptionValueProductPost</code></a></li>
    <li><a href="#OptionValuePut"><code>OptionValuePut</code></a></li>
    <li><a href="#OptionValueResponse"><code>OptionValueResponse</code></a></li>
    <li><a href="#OptionValueVariant"><code>OptionValueVariant</code></a></li>
    <li><a href="#OptionValueVariantPost"><code>OptionValueVariantPost</code></a></li>
    <li><a href="#Pagination"><code>Pagination</code></a></li>
    <li><a href="#Pagination_links"><code>Pagination_links</code></a></li>
    <li><a href="#Product"><code>Product</code></a></li>
    <li><a href="#ProductBase"><code>ProductBase</code></a></li>
    <li><a href="#ProductCollectionResponse"><code>ProductCollectionResponse</code></a></li>
    <li><a href="#ProductImage"><code>ProductImage</code></a></li>
    <li><a href="#ProductImageBase"><code>ProductImageBase</code></a></li>
    <li><a href="#ProductImageCollectionResponse"><code>ProductImageCollectionResponse</code></a></li>
    <li><a href="#ProductImagePost"><code>ProductImagePost</code></a></li>
    <li><a href="#ProductImagePut"><code>ProductImagePut</code></a></li>
    <li><a href="#ProductImageResponse"><code>ProductImageResponse</code></a></li>
    <li><a href="#ProductPost"><code>ProductPost</code></a></li>
    <li><a href="#ProductPut"><code>ProductPut</code></a></li>
    <li><a href="#ProductResponse"><code>ProductResponse</code></a></li>
    <li><a href="#ProductVideo"><code>ProductVideo</code></a></li>
    <li><a href="#ProductVideoBase"><code>ProductVideoBase</code></a></li>
    <li><a href="#ProductVideoCollectionResponse"><code>ProductVideoCollectionResponse</code></a></li>
    <li><a href="#ProductVideoPost"><code>ProductVideoPost</code></a></li>
    <li><a href="#ProductVideoPut"><code>ProductVideoPut</code></a></li>
    <li><a href="#ProductVideoResponse"><code>ProductVideoResponse</code></a></li>
    <li><a href="#ResourceImage"><code>ResourceImage</code></a></li>
    <li><a href="#Subscriber"><code>Subscriber</code></a></li>
    <li><a href="#SubscriberBase"><code>SubscriberBase</code></a></li>
    <li><a href="#SubscriberCollectionResponse"><code>SubscriberCollectionResponse</code></a></li>
    <li><a href="#SubscriberPost"><code>SubscriberPost</code></a></li>
    <li><a href="#SubscriberPut"><code>SubscriberPut</code></a></li>
    <li><a href="#SubscriberResponse"><code>SubscriberResponse</code></a></li>
    <li><a href="#Variant"><code>Variant</code></a></li>
    <li><a href="#VariantBase"><code>VariantBase</code></a></li>
    <li><a href="#VariantCollectionResponse"><code>VariantCollectionResponse</code></a></li>
    <li><a href="#VariantPost"><code>VariantPost</code></a></li>
    <li><a href="#VariantProductPost"><code>VariantProductPost</code></a></li>
    <li><a href="#VariantProductPut"><code>VariantProductPut</code></a></li>
    <li><a href="#VariantPut"><code>VariantPut</code></a></li>
    <li><a href="#VariantResponse"><code>VariantResponse</code></a></li>
  </ol>

  <div class="model">
    <h3 class="field-label"><a name="Adjuster">Adjuster - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'></div>
    <div class="field-items">
      <div class="param">adjuster (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The type of adjuster for either the price or the weight of the variant, when the modifier value is selected on the storefront.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">relative</div><div class="param-enum">percentage</div>
<div class="param">adjuster_value (optional)</div><div class="param-desc"><span class="param-type"><a href="#number">BigDecimal</a></span> The numeric amount by which the adjuster will change either the price or the weight of the variant, when the modifier value is selected on the storefront.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="BaseError">BaseError - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Error payload for the BigCommerce API.
</div>
    <div class="field-items">
      <div class="param">status (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The HTTP status code.
 </div>
<div class="param">title (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The error title describing the particular error.
 </div>
<div class="param">type (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span>  </div>
<div class="param">instance (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="Brand">Brand - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'></div>
    <div class="field-items">
      <div class="param">name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The name of the brand. Must be unique.
 </div>
<div class="param">page_title (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The title shown in the browser while viewing the brand.
 </div>
<div class="param">meta_keywords (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">array[String]</a></span> Comma separated list of meta keywords to include in the HTML.
 </div>
<div class="param">meta_description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> A meta description to include.
 </div>
<div class="param">search_keywords (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> A comma separated list of keywords that can be used to locate this brand.
 </div>
<div class="param">image_url (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Image URL used for this category on the storefront. Images can be uploaded via form file post to &#x60;/brands/{brandId}/image&#x60;, or by providing a publicly accessible URL in this field.
 </div>
<div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the brand; increments sequentially.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="BrandBase">BrandBase - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Common Brand properties.
</div>
    <div class="field-items">
      <div class="param">name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The name of the brand. Must be unique.
 </div>
<div class="param">page_title (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The title shown in the browser while viewing the brand.
 </div>
<div class="param">meta_keywords (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">array[String]</a></span> Comma separated list of meta keywords to include in the HTML.
 </div>
<div class="param">meta_description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> A meta description to include.
 </div>
<div class="param">search_keywords (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> A comma separated list of keywords that can be used to locate this brand.
 </div>
<div class="param">image_url (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Image URL used for this category on the storefront. Images can be uploaded via form file post to &#x60;/brands/{brandId}/image&#x60;, or by providing a publicly accessible URL in this field.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="BrandCollectionResponse">BrandCollectionResponse - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Response payload for the Bigcommerce API.
</div>
    <div class="field-items">
      <div class="param">data (optional)</div><div class="param-desc"><span class="param-type"><a href="#Brand">array[Brand]</a></span>  </div>
<div class="param">meta (optional)</div><div class="param-desc"><span class="param-type"><a href="#CollectionMeta">CollectionMeta</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="BrandPost">BrandPost - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a POST to create brand.
</div>
    <div class="field-items">
      <div class="param">name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The name of the brand. Must be unique.
 </div>
<div class="param">page_title (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The title shown in the browser while viewing the brand.
 </div>
<div class="param">meta_keywords (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">array[String]</a></span> Comma separated list of meta keywords to include in the HTML.
 </div>
<div class="param">meta_description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> A meta description to include.
 </div>
<div class="param">search_keywords (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> A comma separated list of keywords that can be used to locate this brand.
 </div>
<div class="param">image_url (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Image URL used for this category on the storefront. Images can be uploaded via form file post to &#x60;/brands/{brandId}/image&#x60;, or by providing a publicly accessible URL in this field.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="BrandPut">BrandPut - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a PUT to update brand.
</div>
    <div class="field-items">
      <div class="param">name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The name of the brand. Must be unique.
 </div>
<div class="param">page_title (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The title shown in the browser while viewing the brand.
 </div>
<div class="param">meta_keywords (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">array[String]</a></span> Comma separated list of meta keywords to include in the HTML.
 </div>
<div class="param">meta_description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> A meta description to include.
 </div>
<div class="param">search_keywords (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> A comma separated list of keywords that can be used to locate this brand.
 </div>
<div class="param">image_url (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Image URL used for this category on the storefront. Images can be uploaded via form file post to &#x60;/brands/{brandId}/image&#x60;, or by providing a publicly accessible URL in this field.
 </div>
<div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the brand; increments sequentially.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="BrandResponse">BrandResponse - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Response payload for the Bigcommerce API.
</div>
    <div class="field-items">
      <div class="param">data (optional)</div><div class="param-desc"><span class="param-type"><a href="#Brand">Brand</a></span>  </div>
<div class="param">meta (optional)</div><div class="param-desc"><span class="param-type"><a href="#Meta">Meta</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="BulkPricingRule">BulkPricingRule - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Rules that offer price discounts based on quantity breaks.
</div>
    <div class="field-items">
      <div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The ID of the bulk pricing rule.
 </div>
<div class="param">quantity_min (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The minimum inclusive quantity of a product to satisfy this rule. Must be greater than or equal to zero.
 </div>
<div class="param">quantity_max (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The maximum inclusive quantity of a product to satisfy this rule. Must be greater than the &#x60;quantity_min&#x60; value – unless this field has a value of 0 (zero), in which case there will be no maximum bound for this rule.
 </div>
<div class="param">type (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The type of adjustment that is made. Values: &#x60;price&#x60; - the adjustment amount per product; &#x60;percent&#x60; - the adjustment as a percentage of the original price; &#x60;fixed&#x60; - the adjusted absolute price of the product.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">price</div><div class="param-enum">percent</div><div class="param-enum">fixed</div>
<div class="param">amount (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> The value of the adjustment by the bulk pricing rule.
 format: double</div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="CatalogSummary">CatalogSummary - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>A BigCommerce Catalog Summary object describes a lightweight summary of the catalog.
</div>
    <div class="field-items">
      <div class="param">inventory_count (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> A count of all inventory items in the catalog.
 </div>
<div class="param">inventory_value (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> Total value of store&#39;s inventory.
 format: double</div>
<div class="param">primary_category_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> ID of the category containing the most products.
 </div>
<div class="param">primary_category_name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Name of the category containing the most products.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="CatalogSummaryResponse">CatalogSummaryResponse - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Response payload for the Bigcommerce API.
</div>
    <div class="field-items">
      <div class="param">data (optional)</div><div class="param-desc"><span class="param-type"><a href="#CatalogSummary">CatalogSummary</a></span>  </div>
<div class="param">meta (optional)</div><div class="param-desc"><span class="param-type"><a href="#Meta">Meta</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="Category">Category - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>A BigCommerce category object.
</div>
    <div class="field-items">
      <div class="param">parent_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the category&#39;s parent. This field controls where the category sits in the tree of categories that organize the catalog.
 </div>
<div class="param">name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The name displayed for the category. Name is unique with respect to the category&#39;s siblings.
 </div>
<div class="param">description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The product description, which can include HTML formatting.
 </div>
<div class="param">views (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> Number of views the category has on the storefront.
 </div>
<div class="param">sort_order (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> Priority this category will be given when included in the menu and category pages. The lower the number, the closer to the top of the results the category will be.
 </div>
<div class="param">page_title (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Custom title for the category page, if not defined the category name will be used as the meta title.
 </div>
<div class="param">search_keywords (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> A comma separated list of keywords that can be used to locate the category when searching the store.
 </div>
<div class="param">meta_keywords (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">array[String]</a></span> Custom meta keywords for the category page. If not defined, the store&#39;s default keywords will be used. Must post as an array like: [&quot;awesome&quot;,&quot;sauce&quot;].
 </div>
<div class="param">meta_description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Custom meta description for the category page. If not defined, the store&#39;s default meta description will be used.
 </div>
<div class="param">layout_file (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The layout template file used to render this category.
 </div>
<div class="param">is_visible (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag to determine whether the product should be displayed to customers browsing the store. If &#x60;true&#x60;, the category will be displayed. If &#x60;false&#x60;, the category will be hidden from view.
 </div>
<div class="param">default_product_sort (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Determines how the products are sorted on category page load.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">use_store_settings</div><div class="param-enum">featured</div><div class="param-enum">newest</div><div class="param-enum">best_selling</div><div class="param-enum">alpha_asc</div><div class="param-enum">alpha_desc</div><div class="param-enum">avg_customer_review</div><div class="param-enum">price_asc</div><div class="param-enum">price_desc</div>
<div class="param">image_url (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Image URL used for this category on the storefront. Images can be uploaded via form file post to &#x60;/categories/{categoryId}/image&#x60;, or by providing a publicly accessible URL in this field.
 </div>
<div class="param">custom_url (optional)</div><div class="param-desc"><span class="param-type"><a href="#CustomUrlCategory">CustomUrlCategory</a></span>  </div>
<div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the category; increments sequentially.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="CategoryBase">CategoryBase - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Common Category properties.
</div>
    <div class="field-items">
      <div class="param">parent_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the category&#39;s parent. This field controls where the category sits in the tree of categories that organize the catalog.
 </div>
<div class="param">name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The name displayed for the category. Name is unique with respect to the category&#39;s siblings.
 </div>
<div class="param">description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The product description, which can include HTML formatting.
 </div>
<div class="param">views (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> Number of views the category has on the storefront.
 </div>
<div class="param">sort_order (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> Priority this category will be given when included in the menu and category pages. The lower the number, the closer to the top of the results the category will be.
 </div>
<div class="param">page_title (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Custom title for the category page, if not defined the category name will be used as the meta title.
 </div>
<div class="param">search_keywords (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> A comma separated list of keywords that can be used to locate the category when searching the store.
 </div>
<div class="param">meta_keywords (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">array[String]</a></span> Custom meta keywords for the category page. If not defined, the store&#39;s default keywords will be used. Must post as an array like: [&quot;awesome&quot;,&quot;sauce&quot;].
 </div>
<div class="param">meta_description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Custom meta description for the category page. If not defined, the store&#39;s default meta description will be used.
 </div>
<div class="param">layout_file (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The layout template file used to render this category.
 </div>
<div class="param">is_visible (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag to determine whether the product should be displayed to customers browsing the store. If &#x60;true&#x60;, the category will be displayed. If &#x60;false&#x60;, the category will be hidden from view.
 </div>
<div class="param">default_product_sort (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Determines how the products are sorted on category page load.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">use_store_settings</div><div class="param-enum">featured</div><div class="param-enum">newest</div><div class="param-enum">best_selling</div><div class="param-enum">alpha_asc</div><div class="param-enum">alpha_desc</div><div class="param-enum">avg_customer_review</div><div class="param-enum">price_asc</div><div class="param-enum">price_desc</div>
<div class="param">image_url (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Image URL used for this category on the storefront. Images can be uploaded via form file post to &#x60;/categories/{categoryId}/image&#x60;, or by providing a publicly accessible URL in this field.
 </div>
<div class="param">custom_url (optional)</div><div class="param-desc"><span class="param-type"><a href="#CustomUrlCategory">CustomUrlCategory</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="CategoryCollectionResponse">CategoryCollectionResponse - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Response payload for the Bigcommerce API.
</div>
    <div class="field-items">
      <div class="param">data (optional)</div><div class="param-desc"><span class="param-type"><a href="#Category">array[Category]</a></span>  </div>
<div class="param">meta (optional)</div><div class="param-desc"><span class="param-type"><a href="#CollectionMeta">CollectionMeta</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="CategoryNode">CategoryNode - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>A BigCommerce category node object. Used to reflect parent &lt;&gt; child category relationships.
</div>
    <div class="field-items">
      <div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the category; increments sequentially.
 </div>
<div class="param">parent_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the category&#39;s parent. This field controls where the category sits in the tree of categories that organize the catalog.
 </div>
<div class="param">name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The name displayed for the category. Name is unique with respect to the category&#39;s siblings.
 </div>
<div class="param">is_visible (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag to determine whether the product should be displayed to customers browsing the store. If &#x60;true&#x60;, the category will be displayed. If &#x60;false&#x60;, the category will be hidden from view.
 </div>
<div class="param">url (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The custom URL for the category on the storefront.
 </div>
<div class="param">children (optional)</div><div class="param-desc"><span class="param-type"><a href="#CategoryNode">array[CategoryNode]</a></span> The list of children of the category.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="CategoryPost">CategoryPost - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a POST to create category.
</div>
    <div class="field-items">
      <div class="param">parent_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the category&#39;s parent. This field controls where the category sits in the tree of categories that organize the catalog.
 </div>
<div class="param">name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The name displayed for the category. Name is unique with respect to the category&#39;s siblings.
 </div>
<div class="param">description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The product description, which can include HTML formatting.
 </div>
<div class="param">views (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> Number of views the category has on the storefront.
 </div>
<div class="param">sort_order (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> Priority this category will be given when included in the menu and category pages. The lower the number, the closer to the top of the results the category will be.
 </div>
<div class="param">page_title (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Custom title for the category page, if not defined the category name will be used as the meta title.
 </div>
<div class="param">search_keywords (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> A comma separated list of keywords that can be used to locate the category when searching the store.
 </div>
<div class="param">meta_keywords (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">array[String]</a></span> Custom meta keywords for the category page. If not defined, the store&#39;s default keywords will be used. Must post as an array like: [&quot;awesome&quot;,&quot;sauce&quot;].
 </div>
<div class="param">meta_description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Custom meta description for the category page. If not defined, the store&#39;s default meta description will be used.
 </div>
<div class="param">layout_file (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The layout template file used to render this category.
 </div>
<div class="param">is_visible (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag to determine whether the product should be displayed to customers browsing the store. If &#x60;true&#x60;, the category will be displayed. If &#x60;false&#x60;, the category will be hidden from view.
 </div>
<div class="param">default_product_sort (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Determines how the products are sorted on category page load.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">use_store_settings</div><div class="param-enum">featured</div><div class="param-enum">newest</div><div class="param-enum">best_selling</div><div class="param-enum">alpha_asc</div><div class="param-enum">alpha_desc</div><div class="param-enum">avg_customer_review</div><div class="param-enum">price_asc</div><div class="param-enum">price_desc</div>
<div class="param">image_url (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Image URL used for this category on the storefront. Images can be uploaded via form file post to &#x60;/categories/{categoryId}/image&#x60;, or by providing a publicly accessible URL in this field.
 </div>
<div class="param">custom_url (optional)</div><div class="param-desc"><span class="param-type"><a href="#CustomUrlCategory">CustomUrlCategory</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="CategoryPut">CategoryPut - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a PUT to update category.
</div>
    <div class="field-items">
      <div class="param">parent_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the category&#39;s parent. This field controls where the category sits in the tree of categories that organize the catalog.
 </div>
<div class="param">name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The name displayed for the category. Name is unique with respect to the category&#39;s siblings.
 </div>
<div class="param">description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The product description, which can include HTML formatting.
 </div>
<div class="param">views (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> Number of views the category has on the storefront.
 </div>
<div class="param">sort_order (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> Priority this category will be given when included in the menu and category pages. The lower the number, the closer to the top of the results the category will be.
 </div>
<div class="param">page_title (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Custom title for the category page, if not defined the category name will be used as the meta title.
 </div>
<div class="param">search_keywords (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> A comma separated list of keywords that can be used to locate the category when searching the store.
 </div>
<div class="param">meta_keywords (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">array[String]</a></span> Custom meta keywords for the category page. If not defined, the store&#39;s default keywords will be used. Must post as an array like: [&quot;awesome&quot;,&quot;sauce&quot;].
 </div>
<div class="param">meta_description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Custom meta description for the category page. If not defined, the store&#39;s default meta description will be used.
 </div>
<div class="param">layout_file (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The layout template file used to render this category.
 </div>
<div class="param">is_visible (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag to determine whether the product should be displayed to customers browsing the store. If &#x60;true&#x60;, the category will be displayed. If &#x60;false&#x60;, the category will be hidden from view.
 </div>
<div class="param">default_product_sort (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Determines how the products are sorted on category page load.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">use_store_settings</div><div class="param-enum">featured</div><div class="param-enum">newest</div><div class="param-enum">best_selling</div><div class="param-enum">alpha_asc</div><div class="param-enum">alpha_desc</div><div class="param-enum">avg_customer_review</div><div class="param-enum">price_asc</div><div class="param-enum">price_desc</div>
<div class="param">image_url (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Image URL used for this category on the storefront. Images can be uploaded via form file post to &#x60;/categories/{categoryId}/image&#x60;, or by providing a publicly accessible URL in this field.
 </div>
<div class="param">custom_url (optional)</div><div class="param-desc"><span class="param-type"><a href="#CustomUrlCategory">CustomUrlCategory</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="CategoryResponse">CategoryResponse - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Response payload for the Bigcommerce API.
</div>
    <div class="field-items">
      <div class="param">data (optional)</div><div class="param-desc"><span class="param-type"><a href="#Category">Category</a></span>  </div>
<div class="param">meta (optional)</div><div class="param-desc"><span class="param-type"><a href="#Meta">Meta</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="CategoryTreeCollectionResponse">CategoryTreeCollectionResponse - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Response payload for the Bigcommerce API.
</div>
    <div class="field-items">
      <div class="param">data (optional)</div><div class="param-desc"><span class="param-type"><a href="#CategoryNode">array[CategoryNode]</a></span>  </div>
<div class="param">meta (optional)</div><div class="param-desc"><span class="param-type"><a href="#CollectionMeta">CollectionMeta</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="CollectionMeta">CollectionMeta - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Data about the response, including pagination and collection totals.
</div>
    <div class="field-items">
      <div class="param">pagination (optional)</div><div class="param-desc"><span class="param-type"><a href="#Pagination">Pagination</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ComplexRule">ComplexRule - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Apply price, weight, image, or availabilty adjustments to product, based on a set of conditions. A complex rule&#39;s condition must either contain more than one modifier value, or else contain a modifier value and a variant id.
</div>
    <div class="field-items">
      <div class="param">product_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the product with which the rule is associated; increments sequentially.
 </div>
<div class="param">sort_order (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> Priority this rule will be given, when making adjustments to the product properties.
 </div>
<div class="param">enabled (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag for determining whether the rule is to be used when adjusting a product&#39;s price, weight, image, or availabilty.
 </div>
<div class="param">stop (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag for determining whether other rules should not be applied after this rule has been applied.
 </div>
<div class="param">purchasing_disabled (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag for determining whether the rule should disable purchasing of a product when the conditions are applied.
 </div>
<div class="param">purchasing_disabled_message (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Message displayed on the storefront when a rule disables the purchasing of a product.
 </div>
<div class="param">purchasing_hidden (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag for determining whether the rule should hide purchasing of a product when the conditions are applied.
 </div>
<div class="param">price_adjuster (optional)</div><div class="param-desc"><span class="param-type"><a href="#Adjuster">Adjuster</a></span>  </div>
<div class="param">weight_adjuster (optional)</div><div class="param-desc"><span class="param-type"><a href="#Adjuster">Adjuster</a></span>  </div>
<div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the rule; increments sequentially.
 </div>
<div class="param">image_url (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The URL for an image displayed on the storefront when the conditions are applied.
 </div>
<div class="param">conditions (optional)</div><div class="param-desc"><span class="param-type"><a href="#ComplexRuleCondition">array[ComplexRuleCondition]</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ComplexRuleBase">ComplexRuleBase - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Common ComplexRule properties.
</div>
    <div class="field-items">
      <div class="param">product_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the product with which the rule is associated; increments sequentially.
 </div>
<div class="param">sort_order (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> Priority this rule will be given, when making adjustments to the product properties.
 </div>
<div class="param">enabled (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag for determining whether the rule is to be used when adjusting a product&#39;s price, weight, image, or availabilty.
 </div>
<div class="param">stop (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag for determining whether other rules should not be applied after this rule has been applied.
 </div>
<div class="param">purchasing_disabled (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag for determining whether the rule should disable purchasing of a product when the conditions are applied.
 </div>
<div class="param">purchasing_disabled_message (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Message displayed on the storefront when a rule disables the purchasing of a product.
 </div>
<div class="param">purchasing_hidden (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag for determining whether the rule should hide purchasing of a product when the conditions are applied.
 </div>
<div class="param">price_adjuster (optional)</div><div class="param-desc"><span class="param-type"><a href="#Adjuster">Adjuster</a></span>  </div>
<div class="param">weight_adjuster (optional)</div><div class="param-desc"><span class="param-type"><a href="#Adjuster">Adjuster</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ComplexRuleCollectionResponse">ComplexRuleCollectionResponse - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Response payload for the Bigcommerce API.
</div>
    <div class="field-items">
      <div class="param">data (optional)</div><div class="param-desc"><span class="param-type"><a href="#ComplexRule">array[ComplexRule]</a></span>  </div>
<div class="param">meta (optional)</div><div class="param-desc"><span class="param-type"><a href="#CollectionMeta">CollectionMeta</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ComplexRuleCondition">ComplexRuleCondition - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Complex rules may return with conditions that apply to one or more variants, or with a single modifier value (if the rules were created using the v2 API or the control panel). Complex rules created or updated in the v3 API must have conditions that either reference multiple &#x60;modifier_value_id&#x60;&#39;s, or else reference a &#x60;modifier_value_id&#x60; and a &#x60;variant_id&#x60;.
</div>
    <div class="field-items">
      <div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the rule condition; increments sequentially.
 </div>
<div class="param">rule_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the rule with which the condition is associated.
 </div>
<div class="param">modifier_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the modifier with which the rule condition is associated.
 </div>
<div class="param">modifier_value_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the modifier value with which the rule condition is associated.
 </div>
<div class="param">variant_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the variant the rule condition is associated with.
 </div>
<div class="param">combination_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> (READ-ONLY:) The unique numeric ID of the SKU (v2 API), or Combination, with which the rule condition is associated. This is to maintain cross-compatibility between v2 and v3.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ComplexRuleConditionBase">ComplexRuleConditionBase - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Common ComplexRuleCondition properties.
</div>
    <div class="field-items">
      <div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the rule condition; increments sequentially.
 </div>
<div class="param">rule_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the rule with which the condition is associated.
 </div>
<div class="param">modifier_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the modifier with which the rule condition is associated.
 </div>
<div class="param">modifier_value_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the modifier value with which the rule condition is associated.
 </div>
<div class="param">variant_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the variant the rule condition is associated with.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ComplexRuleConditionPost">ComplexRuleConditionPost - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a POST to create conditions on a complex rule.
</div>
    <div class="field-items">
      <div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the rule condition; increments sequentially.
 </div>
<div class="param">rule_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the rule with which the condition is associated.
 </div>
<div class="param">modifier_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the modifier with which the rule condition is associated.
 </div>
<div class="param">modifier_value_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the modifier value with which the rule condition is associated.
 </div>
<div class="param">variant_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the variant the rule condition is associated with.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ComplexRuleConditionPut">ComplexRuleConditionPut - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a PUT to update conditions on a complex rule.
</div>
    <div class="field-items">
      <div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the rule condition; increments sequentially.
 </div>
<div class="param">rule_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the rule with which the condition is associated.
 </div>
<div class="param">modifier_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the modifier with which the rule condition is associated.
 </div>
<div class="param">modifier_value_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the modifier value with which the rule condition is associated.
 </div>
<div class="param">variant_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the variant the rule condition is associated with.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ComplexRulePost">ComplexRulePost - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a POST to create complex rule on a product.
</div>
    <div class="field-items">
      <div class="param">product_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the product with which the rule is associated; increments sequentially.
 </div>
<div class="param">sort_order (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> Priority this rule will be given, when making adjustments to the product properties.
 </div>
<div class="param">enabled (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag for determining whether the rule is to be used when adjusting a product&#39;s price, weight, image, or availabilty.
 </div>
<div class="param">stop (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag for determining whether other rules should not be applied after this rule has been applied.
 </div>
<div class="param">purchasing_disabled (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag for determining whether the rule should disable purchasing of a product when the conditions are applied.
 </div>
<div class="param">purchasing_disabled_message (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Message displayed on the storefront when a rule disables the purchasing of a product.
 </div>
<div class="param">purchasing_hidden (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag for determining whether the rule should hide purchasing of a product when the conditions are applied.
 </div>
<div class="param">price_adjuster (optional)</div><div class="param-desc"><span class="param-type"><a href="#Adjuster">Adjuster</a></span>  </div>
<div class="param">weight_adjuster (optional)</div><div class="param-desc"><span class="param-type"><a href="#Adjuster">Adjuster</a></span>  </div>
<div class="param">conditions (optional)</div><div class="param-desc"><span class="param-type"><a href="#ComplexRuleConditionPost">array[ComplexRuleConditionPost]</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ComplexRulePut">ComplexRulePut - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a PUT to update complex rule on a product.
</div>
    <div class="field-items">
      <div class="param">product_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the product with which the rule is associated; increments sequentially.
 </div>
<div class="param">sort_order (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> Priority this rule will be given, when making adjustments to the product properties.
 </div>
<div class="param">enabled (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag for determining whether the rule is to be used when adjusting a product&#39;s price, weight, image, or availabilty.
 </div>
<div class="param">stop (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag for determining whether other rules should not be applied after this rule has been applied.
 </div>
<div class="param">purchasing_disabled (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag for determining whether the rule should disable purchasing of a product when the conditions are applied.
 </div>
<div class="param">purchasing_disabled_message (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Message displayed on the storefront when a rule disables the purchasing of a product.
 </div>
<div class="param">purchasing_hidden (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag for determining whether the rule should hide purchasing of a product when the conditions are applied.
 </div>
<div class="param">price_adjuster (optional)</div><div class="param-desc"><span class="param-type"><a href="#Adjuster">Adjuster</a></span>  </div>
<div class="param">weight_adjuster (optional)</div><div class="param-desc"><span class="param-type"><a href="#Adjuster">Adjuster</a></span>  </div>
<div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the rule; increments sequentially.
 </div>
<div class="param">conditions (optional)</div><div class="param-desc"><span class="param-type"><a href="#ComplexRuleConditionPut">array[ComplexRuleConditionPut]</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ComplexRuleResponse">ComplexRuleResponse - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Response payload for the Bigcommerce API.
</div>
    <div class="field-items">
      <div class="param">data (optional)</div><div class="param-desc"><span class="param-type"><a href="#ComplexRule">ComplexRule</a></span>  </div>
<div class="param">meta (optional)</div><div class="param-desc"><span class="param-type"><a href="#Meta">Meta</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="CustomField">CustomField - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Gets custom fields associated with a product. These allow you to specify additional information that will appear on the product&#39;s page, such as a book&#39;s ISBN or a DVD&#39;s release date.
</div>
    <div class="field-items">
      <div class="param">name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The name of the field, shown on the storefront, orders, etc.
 </div>
<div class="param">value (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The name of the field, shown on the storefront, orders, etc.
 </div>
<div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the custom field; increments sequentially.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="CustomFieldBase">CustomFieldBase - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Common CustomField properties.
</div>
    <div class="field-items">
      <div class="param">name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The name of the field, shown on the storefront, orders, etc.
 </div>
<div class="param">value (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The name of the field, shown on the storefront, orders, etc.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="CustomFieldCollectionResponse">CustomFieldCollectionResponse - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Response payload for the Bigcommerce API.
</div>
    <div class="field-items">
      <div class="param">data (optional)</div><div class="param-desc"><span class="param-type"><a href="#CustomField">array[CustomField]</a></span>  </div>
<div class="param">meta (optional)</div><div class="param-desc"><span class="param-type"><a href="#CollectionMeta">CollectionMeta</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="CustomFieldPost">CustomFieldPost - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a POST to create custom field on a product.
</div>
    <div class="field-items">
      <div class="param">name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The name of the field, shown on the storefront, orders, etc.
 </div>
<div class="param">value (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The name of the field, shown on the storefront, orders, etc.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="CustomFieldPut">CustomFieldPut - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a PUT to update custom field on a product.
</div>
    <div class="field-items">
      <div class="param">name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The name of the field, shown on the storefront, orders, etc.
 </div>
<div class="param">value (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The name of the field, shown on the storefront, orders, etc.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="CustomFieldResponse">CustomFieldResponse - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Response payload for the Bigcommerce API.
</div>
    <div class="field-items">
      <div class="param">data (optional)</div><div class="param-desc"><span class="param-type"><a href="#CustomField">CustomField</a></span>  </div>
<div class="param">meta (optional)</div><div class="param-desc"><span class="param-type"><a href="#Meta">Meta</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="CustomUrlCategory">CustomUrlCategory - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The custom URL for the category on the storefront.
</div>
    <div class="field-items">
      <div class="param">url (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Category URL on the storefront.
 </div>
<div class="param">is_customized (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Returns &#x60;true&#x60; if the URL has been changed from its default state (the auto-assigned URL that BigCommerce provides).
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="CustomUrlProduct">CustomUrlProduct - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The custom URL for the product on the storefront.
</div>
    <div class="field-items">
      <div class="param">url (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Product URL on the storefront.
 </div>
<div class="param">is_customized (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Returns &#x60;true&#x60; if the URL has been changed from its default state (the auto-assigned URL that BigCommerce provides).
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="DetailedErrors">DetailedErrors - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'></div>
    <div class="field-items">
          </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ErrorResponse">ErrorResponse - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'></div>
    <div class="field-items">
      <div class="param">status (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The HTTP status code.
 </div>
<div class="param">title (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The error title describing the particular error.
 </div>
<div class="param">type (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span>  </div>
<div class="param">instance (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span>  </div>
<div class="param">errors (optional)</div><div class="param-desc"><span class="param-type"><a href="#DetailedErrors">DetailedErrors</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ImageResponse">ImageResponse - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Response payload for the Bigcommerce API.
</div>
    <div class="field-items">
      <div class="param">data (optional)</div><div class="param-desc"><span class="param-type"><a href="#ResourceImage">ResourceImage</a></span>  </div>
<div class="param">meta (optional)</div><div class="param-desc"><span class="param-type"><a href="#Meta">Meta</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="Meta">Meta - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Empty meta object, may be used later</div>
    <div class="field-items">
          </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="MetaFieldCollectionResponse">MetaFieldCollectionResponse - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Response payload for the Bigcommerce API.
</div>
    <div class="field-items">
      <div class="param">data (optional)</div><div class="param-desc"><span class="param-type"><a href="#Metafield">array[Metafield]</a></span>  </div>
<div class="param">meta (optional)</div><div class="param-desc"><span class="param-type"><a href="#CollectionMeta">CollectionMeta</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="Metafield">Metafield - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Allows app partners to write custom data to various resources in the API.
</div>
    <div class="field-items">
      <div class="param">permission_set (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Determines whether the field is completely private to the app that owns the field (&#x60;app_only&#x60;), or visible to other API consumers (&#x60;read&#x60;), or completely open for reading and writing to other apps (&#x60;write&#x60;).
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">app_only</div><div class="param-enum">read</div><div class="param-enum">write</div>
<div class="param">namespace (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Namespace for the metafield, for organizational purposes.
 </div>
<div class="param">key (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The name of the field, for example: &#x60;location_id&#x60;, &#x60;color&#x60;.
 </div>
<div class="param">value (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The value of the field, for example: &#x60;1&#x60;, &#x60;blue&#x60;.
 </div>
<div class="param">description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Description for the metafields.
 </div>
<div class="param">resource_type (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The type of resource that the metafield is associated with.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">category</div><div class="param-enum">brand</div><div class="param-enum">product</div><div class="param-enum">variant</div>
<div class="param">resource_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique identifier for the resource with which the metafield is associated.
 </div>
<div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique identifier for the metafields.
 </div>
<div class="param">created_at (optional)</div><div class="param-desc"><span class="param-type"><a href="#DateTime">Date</a></span> Date and time of the metafield&#39;s creation.
 format: date-time</div>
<div class="param">updated_at (optional)</div><div class="param-desc"><span class="param-type"><a href="#DateTime">Date</a></span> Date and time when the metafield was last updated.
 format: date-time</div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="MetafieldBase">MetafieldBase - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Common Metafield properties.
</div>
    <div class="field-items">
      <div class="param">permission_set (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Determines whether the field is completely private to the app that owns the field (&#x60;app_only&#x60;), or visible to other API consumers (&#x60;read&#x60;), or completely open for reading and writing to other apps (&#x60;write&#x60;).
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">app_only</div><div class="param-enum">read</div><div class="param-enum">write</div>
<div class="param">namespace (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Namespace for the metafield, for organizational purposes.
 </div>
<div class="param">key (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The name of the field, for example: &#x60;location_id&#x60;, &#x60;color&#x60;.
 </div>
<div class="param">value (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The value of the field, for example: &#x60;1&#x60;, &#x60;blue&#x60;.
 </div>
<div class="param">description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Description for the metafields.
 </div>
<div class="param">resource_type (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The type of resource that the metafield is associated with.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">category</div><div class="param-enum">brand</div><div class="param-enum">product</div><div class="param-enum">variant</div>
<div class="param">resource_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique identifier for the resource with which the metafield is associated.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="MetafieldPost">MetafieldPost - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a POST to create metafield.
</div>
    <div class="field-items">
      <div class="param">permission_set (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Determines whether the field is completely private to the app that owns the field (&#x60;app_only&#x60;), or visible to other API consumers (&#x60;read&#x60;), or completely open for reading and writing to other apps (&#x60;write&#x60;).
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">app_only</div><div class="param-enum">read</div><div class="param-enum">write</div>
<div class="param">namespace (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Namespace for the metafield, for organizational purposes.
 </div>
<div class="param">key (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The name of the field, for example: &#x60;location_id&#x60;, &#x60;color&#x60;.
 </div>
<div class="param">value (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The value of the field, for example: &#x60;1&#x60;, &#x60;blue&#x60;.
 </div>
<div class="param">description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Description for the metafields.
 </div>
<div class="param">resource_type (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The type of resource that the metafield is associated with.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">category</div><div class="param-enum">brand</div><div class="param-enum">product</div><div class="param-enum">variant</div>
<div class="param">resource_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique identifier for the resource with which the metafield is associated.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="MetafieldPut">MetafieldPut - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a PUT to update metafield.
</div>
    <div class="field-items">
      <div class="param">permission_set (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Determines whether the field is completely private to the app that owns the field (&#x60;app_only&#x60;), or visible to other API consumers (&#x60;read&#x60;), or completely open for reading and writing to other apps (&#x60;write&#x60;).
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">app_only</div><div class="param-enum">read</div><div class="param-enum">write</div>
<div class="param">namespace (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Namespace for the metafield, for organizational purposes.
 </div>
<div class="param">key (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The name of the field, for example: &#x60;location_id&#x60;, &#x60;color&#x60;.
 </div>
<div class="param">value (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The value of the field, for example: &#x60;1&#x60;, &#x60;blue&#x60;.
 </div>
<div class="param">description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Description for the metafields.
 </div>
<div class="param">resource_type (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The type of resource that the metafield is associated with.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">category</div><div class="param-enum">brand</div><div class="param-enum">product</div><div class="param-enum">variant</div>
<div class="param">resource_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique identifier for the resource with which the metafield is associated.
 </div>
<div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique identifier for the metafields.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="MetafieldResponse">MetafieldResponse - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Response payload for the Bigcommerce API.
</div>
    <div class="field-items">
      <div class="param">data (optional)</div><div class="param-desc"><span class="param-type"><a href="#Metafield">Metafield</a></span>  </div>
<div class="param">meta (optional)</div><div class="param-desc"><span class="param-type"><a href="#Meta">Meta</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="Modifier">Modifier - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'></div>
    <div class="field-items">
      <div class="param">type (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The type of modifier, which determines how it will display on the storefront. For reference, former v2 API values:
D &#x3D; date, C &#x3D; checkbox, F &#x3D; file, T &#x3D; text, MT &#x3D; multi_line_text, N &#x3D; numbers_only_text, RB &#x3D; radio_buttons,
RT &#x3D; rectangles, S &#x3D; dropdown, P &#x3D; product_list, PI &#x3D; product_list_with_images, CS &#x3D; swatch.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">date</div><div class="param-enum">checkbox</div><div class="param-enum">file</div><div class="param-enum">text</div><div class="param-enum">multi_line_text</div><div class="param-enum">numbers_only_text</div><div class="param-enum">radio_buttons</div><div class="param-enum">rectangles</div><div class="param-enum">dropdown</div><div class="param-enum">product_list</div><div class="param-enum">product_list_with_images</div><div class="param-enum">swatch</div>
<div class="param">required (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Whether this modifer is required or not at checkout
 </div>
<div class="param">config (optional)</div><div class="param-desc"><span class="param-type"><a href="#OptionConfig">OptionConfig</a></span>  </div>
<div class="param">option_values (optional)</div><div class="param-desc"><span class="param-type"><a href="#ModifierValue">array[ModifierValue]</a></span>  </div>
<div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the modifier; increments sequentially.
 </div>
<div class="param">product_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the product to which the option belongs.
 </div>
<div class="param">name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The unique option name. Auto-generated from the display name, a timestamp, and the product ID.
 </div>
<div class="param">display_name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The name of the option shown on the storefront.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ModifierBase">ModifierBase - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Common Modifier properties.
</div>
    <div class="field-items">
      <div class="param">type (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The type of modifier, which determines how it will display on the storefront. For reference, former v2 API values:
D &#x3D; date, C &#x3D; checkbox, F &#x3D; file, T &#x3D; text, MT &#x3D; multi_line_text, N &#x3D; numbers_only_text, RB &#x3D; radio_buttons,
RT &#x3D; rectangles, S &#x3D; dropdown, P &#x3D; product_list, PI &#x3D; product_list_with_images, CS &#x3D; swatch.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">date</div><div class="param-enum">checkbox</div><div class="param-enum">file</div><div class="param-enum">text</div><div class="param-enum">multi_line_text</div><div class="param-enum">numbers_only_text</div><div class="param-enum">radio_buttons</div><div class="param-enum">rectangles</div><div class="param-enum">dropdown</div><div class="param-enum">product_list</div><div class="param-enum">product_list_with_images</div><div class="param-enum">swatch</div>
<div class="param">required (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Whether this modifer is required or not at checkout
 </div>
<div class="param">config (optional)</div><div class="param-desc"><span class="param-type"><a href="#OptionConfig">OptionConfig</a></span>  </div>
<div class="param">option_values (optional)</div><div class="param-desc"><span class="param-type"><a href="#ModifierValue">array[ModifierValue]</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ModifierCollectionResponse">ModifierCollectionResponse - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Response payload for the Bigcommerce API.
</div>
    <div class="field-items">
      <div class="param">data (optional)</div><div class="param-desc"><span class="param-type"><a href="#Modifier">array[Modifier]</a></span>  </div>
<div class="param">meta (optional)</div><div class="param-desc"><span class="param-type"><a href="#CollectionMeta">CollectionMeta</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ModifierPost">ModifierPost - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a POST to create modifier on a product.
</div>
    <div class="field-items">
      <div class="param">type (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The type of modifier, which determines how it will display on the storefront. For reference, former v2 API values:
D &#x3D; date, C &#x3D; checkbox, F &#x3D; file, T &#x3D; text, MT &#x3D; multi_line_text, N &#x3D; numbers_only_text, RB &#x3D; radio_buttons,
RT &#x3D; rectangles, S &#x3D; dropdown, P &#x3D; product_list, PI &#x3D; product_list_with_images, CS &#x3D; swatch.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">date</div><div class="param-enum">checkbox</div><div class="param-enum">file</div><div class="param-enum">text</div><div class="param-enum">multi_line_text</div><div class="param-enum">numbers_only_text</div><div class="param-enum">radio_buttons</div><div class="param-enum">rectangles</div><div class="param-enum">dropdown</div><div class="param-enum">product_list</div><div class="param-enum">product_list_with_images</div><div class="param-enum">swatch</div>
<div class="param">required (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Whether this modifer is required or not at checkout
 </div>
<div class="param">config (optional)</div><div class="param-desc"><span class="param-type"><a href="#OptionConfig">OptionConfig</a></span>  </div>
<div class="param">option_values (optional)</div><div class="param-desc"><span class="param-type"><a href="#ModifierValue">array[ModifierValue]</a></span>  </div>
<div class="param">display_name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The name of the option shown on the storefront.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ModifierPut">ModifierPut - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a PUT to update modifier on a product.
</div>
    <div class="field-items">
      <div class="param">type (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The type of modifier, which determines how it will display on the storefront. For reference, former v2 API values:
D &#x3D; date, C &#x3D; checkbox, F &#x3D; file, T &#x3D; text, MT &#x3D; multi_line_text, N &#x3D; numbers_only_text, RB &#x3D; radio_buttons,
RT &#x3D; rectangles, S &#x3D; dropdown, P &#x3D; product_list, PI &#x3D; product_list_with_images, CS &#x3D; swatch.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">date</div><div class="param-enum">checkbox</div><div class="param-enum">file</div><div class="param-enum">text</div><div class="param-enum">multi_line_text</div><div class="param-enum">numbers_only_text</div><div class="param-enum">radio_buttons</div><div class="param-enum">rectangles</div><div class="param-enum">dropdown</div><div class="param-enum">product_list</div><div class="param-enum">product_list_with_images</div><div class="param-enum">swatch</div>
<div class="param">required (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Whether this modifer is required or not at checkout
 </div>
<div class="param">config (optional)</div><div class="param-desc"><span class="param-type"><a href="#OptionConfig">OptionConfig</a></span>  </div>
<div class="param">option_values (optional)</div><div class="param-desc"><span class="param-type"><a href="#ModifierValue">array[ModifierValue]</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ModifierResponse">ModifierResponse - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Response payload for the Bigcommerce API.
</div>
    <div class="field-items">
      <div class="param">data (optional)</div><div class="param-desc"><span class="param-type"><a href="#Modifier">Modifier</a></span>  </div>
<div class="param">meta (optional)</div><div class="param-desc"><span class="param-type"><a href="#Meta">Meta</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ModifierValue">ModifierValue - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'></div>
    <div class="field-items">
      <div class="param">is_default (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> The flag for preselecting a value as the default on the storefront. This field is not supported for swatch options/modifiers.
 </div>
<div class="param">label (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The text display identifying the value on the storefront.
 </div>
<div class="param">sort_order (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The order in which the value will be displayed on the product page.
 </div>
<div class="param">value_data (optional)</div><div class="param-desc"><span class="param-type"><a href="#object">Object</a></span> Extra data describing the value, based on the type of option or modifier with which the value is associated. &#x60;swatch&#x60; requires an array of colors, with up to three hexidecimal color keys; &#x60;product list&#x60; requires a &#x60;product_id&#x60;; and &#x60;checkbox&#x60; requires a boolean flag, called &#x60;checked_value&#x60;, to determine which value is considered to be the checked state.
 </div>
<div class="param">adjusters (optional)</div><div class="param-desc"><span class="param-type"><a href="#ModifierValueBase_adjusters">ModifierValueBase_adjusters</a></span>  </div>
<div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the value; increments sequentially.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ModifierValueBase">ModifierValueBase - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'></div>
    <div class="field-items">
      <div class="param">is_default (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> The flag for preselecting a value as the default on the storefront. This field is not supported for swatch options/modifiers.
 </div>
<div class="param">label (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The text display identifying the value on the storefront.
 </div>
<div class="param">sort_order (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The order in which the value will be displayed on the product page.
 </div>
<div class="param">value_data (optional)</div><div class="param-desc"><span class="param-type"><a href="#object">Object</a></span> Extra data describing the value, based on the type of option or modifier with which the value is associated. &#x60;swatch&#x60; requires an array of colors, with up to three hexidecimal color keys; &#x60;product list&#x60; requires a &#x60;product_id&#x60;; and &#x60;checkbox&#x60; requires a boolean flag, called &#x60;checked_value&#x60;, to determine which value is considered to be the checked state.
 </div>
<div class="param">adjusters (optional)</div><div class="param-desc"><span class="param-type"><a href="#ModifierValueBase_adjusters">ModifierValueBase_adjusters</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ModifierValueBase_adjusters">ModifierValueBase_adjusters - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'></div>
    <div class="field-items">
      <div class="param">price (optional)</div><div class="param-desc"><span class="param-type"><a href="#Adjuster">Adjuster</a></span>  </div>
<div class="param">weight (optional)</div><div class="param-desc"><span class="param-type"><a href="#Adjuster">Adjuster</a></span>  </div>
<div class="param">image_url (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The URL for an image displayed on the storefront when the modifier value is selected.
 </div>
<div class="param">purchasing_disabled (optional)</div><div class="param-desc"><span class="param-type"><a href="#ModifierValueBase_adjusters_purchasing_disabled">ModifierValueBase_adjusters_purchasing_disabled</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ModifierValueBase_adjusters_purchasing_disabled">ModifierValueBase_adjusters_purchasing_disabled - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'></div>
    <div class="field-items">
      <div class="param">status (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag for whether the modifier value disables purchasing when selected on the storefront. This can be used for temporarily disabling a particular modifier value.
 </div>
<div class="param">message (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The message displayed on the storefront when the purchasing disabled status is &#x60;true&#x60;.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ModifierValueCollectionResponse">ModifierValueCollectionResponse - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Response payload for the Bigcommerce API.
</div>
    <div class="field-items">
      <div class="param">data (optional)</div><div class="param-desc"><span class="param-type"><a href="#ModifierValue">array[ModifierValue]</a></span>  </div>
<div class="param">meta (optional)</div><div class="param-desc"><span class="param-type"><a href="#CollectionMeta">CollectionMeta</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ModifierValuePost">ModifierValuePost - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a POST to create modifier value on a product.
</div>
    <div class="field-items">
      <div class="param">is_default (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> The flag for preselecting a value as the default on the storefront. This field is not supported for swatch options/modifiers.
 </div>
<div class="param">label (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The text display identifying the value on the storefront.
 </div>
<div class="param">sort_order (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The order in which the value will be displayed on the product page.
 </div>
<div class="param">value_data (optional)</div><div class="param-desc"><span class="param-type"><a href="#object">Object</a></span> Extra data describing the value, based on the type of option or modifier with which the value is associated. &#x60;swatch&#x60; requires an array of colors, with up to three hexidecimal color keys; &#x60;product list&#x60; requires a &#x60;product_id&#x60;; and &#x60;checkbox&#x60; requires a boolean flag, called &#x60;checked_value&#x60;, to determine which value is considered to be the checked state.
 </div>
<div class="param">adjusters (optional)</div><div class="param-desc"><span class="param-type"><a href="#ModifierValueBase_adjusters">ModifierValueBase_adjusters</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ModifierValuePut">ModifierValuePut - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a PUT to update modifier value on a product.
</div>
    <div class="field-items">
      <div class="param">is_default (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> The flag for preselecting a value as the default on the storefront. This field is not supported for swatch options/modifiers.
 </div>
<div class="param">label (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The text display identifying the value on the storefront.
 </div>
<div class="param">sort_order (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The order in which the value will be displayed on the product page.
 </div>
<div class="param">value_data (optional)</div><div class="param-desc"><span class="param-type"><a href="#object">Object</a></span> Extra data describing the value, based on the type of option or modifier with which the value is associated. &#x60;swatch&#x60; requires an array of colors, with up to three hexidecimal color keys; &#x60;product list&#x60; requires a &#x60;product_id&#x60;; and &#x60;checkbox&#x60; requires a boolean flag, called &#x60;checked_value&#x60;, to determine which value is considered to be the checked state.
 </div>
<div class="param">adjusters (optional)</div><div class="param-desc"><span class="param-type"><a href="#ModifierValueBase_adjusters">ModifierValueBase_adjusters</a></span>  </div>
<div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the value; increments sequentially.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ModifierValueResponse">ModifierValueResponse - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Response payload for the Bigcommerce API.
</div>
    <div class="field-items">
      <div class="param">data (optional)</div><div class="param-desc"><span class="param-type"><a href="#ModifierValue">ModifierValue</a></span>  </div>
<div class="param">meta (optional)</div><div class="param-desc"><span class="param-type"><a href="#Meta">Meta</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="NotFound">NotFound - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Error payload for the BigCommerce API.
</div>
    <div class="field-items">
      <div class="param">status (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> 404 HTTP status code.
 </div>
<div class="param">title (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The error title describing the particular error. </div>
<div class="param">type (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span>  </div>
<div class="param">instance (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="Option">Option - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'></div>
    <div class="field-items">
      <div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numerical ID of the option, increments sequentially.
 </div>
<div class="param">product_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numerical ID of the product that the option belongs to.
 </div>
<div class="param">display_name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The name of the option shown on the storefront.
 </div>
<div class="param">type (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The type of option, which determines how it will display on the storefront. For reference, the former v2 API values are:
RB &#x3D; radio_buttons, RT &#x3D; rectangles, S &#x3D; dropdown, P &#x3D; product_list, PI &#x3D; product_list_with_images, CS &#x3D; swatch.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">radio_buttons</div><div class="param-enum">rectangles</div><div class="param-enum">dropdown</div><div class="param-enum">product_list</div><div class="param-enum">product_list_with_images</div><div class="param-enum">swatch</div>
<div class="param">config (optional)</div><div class="param-desc"><span class="param-type"><a href="#OptionConfig">OptionConfig</a></span>  </div>
<div class="param">option_values (optional)</div><div class="param-desc"><span class="param-type"><a href="#OptionValue">array[OptionValue]</a></span>  </div>
<div class="param">name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The name unique option name auto-generated from the display name, a timestamp, and the product id.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="OptionBase">OptionBase - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Common Option properties.</div>
    <div class="field-items">
      <div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numerical ID of the option, increments sequentially.
 </div>
<div class="param">product_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numerical ID of the product that the option belongs to.
 </div>
<div class="param">display_name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The name of the option shown on the storefront.
 </div>
<div class="param">type (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The type of option, which determines how it will display on the storefront. For reference, the former v2 API values are:
RB &#x3D; radio_buttons, RT &#x3D; rectangles, S &#x3D; dropdown, P &#x3D; product_list, PI &#x3D; product_list_with_images, CS &#x3D; swatch.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">radio_buttons</div><div class="param-enum">rectangles</div><div class="param-enum">dropdown</div><div class="param-enum">product_list</div><div class="param-enum">product_list_with_images</div><div class="param-enum">swatch</div>
<div class="param">config (optional)</div><div class="param-desc"><span class="param-type"><a href="#OptionConfig">OptionConfig</a></span>  </div>
<div class="param">option_values (optional)</div><div class="param-desc"><span class="param-type"><a href="#OptionValue">array[OptionValue]</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="OptionCollectionResponse">OptionCollectionResponse - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Response payload for the Bigcommerce API.
</div>
    <div class="field-items">
      <div class="param">data (optional)</div><div class="param-desc"><span class="param-type"><a href="#Option">array[Option]</a></span>  </div>
<div class="param">meta (optional)</div><div class="param-desc"><span class="param-type"><a href="#CollectionMeta">CollectionMeta</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="OptionConfig">OptionConfig - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'></div>
    <div class="field-items">
      <div class="param">default_value (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> (date, text, multi_line_text, numbers_only_text) The default value. Shown on a date option as an ISO-8601–formatted string, or on a text option as a string.
 </div>
<div class="param">checked_by_default (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> (checkbox) Flag for setting the checkbox to be checked by default.
 </div>
<div class="param">checkbox_label (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> (checkbox) Label displayed for the checkbox option.
 </div>
<div class="param">date_limited (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> (date) Flag to limit the dates allowed to be entered on a date option.
 </div>
<div class="param">date_limit_mode (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> (date) The type of limit that is allowed to be entered on a date option.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">earliest</div><div class="param-enum">range</div><div class="param-enum">latest</div>
<div class="param">date_earliest_value (optional)</div><div class="param-desc"><span class="param-type"><a href="#date">date</a></span> (date) The earliest date allowed to be entered on the date option, as an ISO-8601 formatted string.
 format: date</div>
<div class="param">date_latest_value (optional)</div><div class="param-desc"><span class="param-type"><a href="#date">date</a></span> (date) The latest date allowed to be entered on the date option, as an ISO-8601 formatted string.
 format: date</div>
<div class="param">file_types_mode (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> (file) The kind of restriction on the file types that can be uploaded with a file upload option. Values: &#x60;specific&#x60; - restricts uploads to particular file types; &#x60;all&#x60; - allows all file types.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">specific</div><div class="param-enum">all</div>
<div class="param">file_types_supported (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">array[String]</a></span> (file) The type of files allowed to be uploaded if the &#x60;file_type_option&#x60; is set to &#x60;specific&#x60;. Values:
  &#x60;images&#x60; - Allows upload of image MIME types (&#x60;bmp&#x60;,&#x60;gif&#x60;,&#x60;jpg&#x60;,&#x60;jpeg&#x60;,&#x60;jpe&#x60;,&#x60;jif&#x60;,&#x60;jfif&#x60;,&#x60;jfi&#x60;,&#x60;png&#x60;,&#x60;wbmp&#x60;,&#x60;xbm&#x60;,&#x60;tiff&#x60;).
  &#x60;documents&#x60; - Allows upload of document MIME types (&#x60;txt&#x60;,&#x60;pdf&#x60;,&#x60;rtf&#x60;,&#x60;doc&#x60;,&#x60;docx&#x60;,&#x60;xls&#x60;,&#x60;xlsx&#x60;,&#x60;accdb&#x60;,&#x60;mdb&#x60;,&#x60;one&#x60;,&#x60;pps&#x60;,&#x60;ppsx&#x60;,&#x60;ppt&#x60;,&#x60;pptx&#x60;,&#x60;pub&#x60;,&#x60;odt&#x60;,&#x60;ods&#x60;,&#x60;odp&#x60;,&#x60;odg&#x60;,&#x60;odf&#x60;).
  &#x60;other&#x60; - Allows file types defined in the &#x60;file_types_other&#x60; array.
 </div>
<div class="param">file_types_other (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">array[String]</a></span> (file) A list of other file types allowed with the file upload option.
 </div>
<div class="param">file_max_size (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> (file) The maximum size for a file that can be used with the file upload option.
 </div>
<div class="param">text_characters_limited (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> (text, multi_line_text) Flag to validate the length of a text or multi-line text input.
 </div>
<div class="param">text_min_length (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> (text, multi_line_text) The minimum length allowed for a text or multi-line text option.
 </div>
<div class="param">text_max_length (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> (text, multi_line_text) The maximum length allowed for a text or multi line text option.
 </div>
<div class="param">text_lines_limited (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> (multi_line_text) Flag to validate the maximum number of lines allowed on a multi-line text input.
 </div>
<div class="param">text_max_lines (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> (multi_line_text) The maximum number of lines allowed on a multi-line text input.
 </div>
<div class="param">number_limited (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> (numbers_only_text) Flag to limit the value of a number option.
 </div>
<div class="param">number_limit_mode (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> (numbers_only_text) The type of limit on values entered for a number option.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">lowest</div><div class="param-enum">highest</div><div class="param-enum">range</div>
<div class="param">number_lowest_value (optional)</div><div class="param-desc"><span class="param-type"><a href="#number">BigDecimal</a></span> (numbers_only_text) The lowest allowed value for a number option if &#x60;number_limited&#x60; is true.
 </div>
<div class="param">number_highest_value (optional)</div><div class="param-desc"><span class="param-type"><a href="#number">BigDecimal</a></span> (numbers_only_text) The highest allowed value for a number option if &#x60;number_limited&#x60; is true.
 </div>
<div class="param">number_integers_only (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> (numbers_only_text) Flag to limit the input on a number option to whole numbers only.
 </div>
<div class="param">product_list_adjusts_inventory (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> (product_list, product_list_with_images) Flag for automatically adjusting inventory on a product included in the list.
 </div>
<div class="param">product_list_adjusts_pricing (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> (product_list, product_list_with_images) Flag to add the optional product&#39;s price to the main product&#39;s price.
 </div>
<div class="param">product_list_shipping_calc (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> (product_list, product_list_with_images) How to factor the optional product&#39;s weight and package dimensions into the shipping quote. Values: &#x60;none&#x60; - don&#39;t adjust; &#x60;weight&#x60; - use shipping weight only; &#x60;package&#x60; - use weight and dimensions.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">none</div><div class="param-enum">weight</div><div class="param-enum">package</div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="OptionPost">OptionPost - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a POST to create option on a product.
</div>
    <div class="field-items">
      <div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numerical ID of the option, increments sequentially.
 </div>
<div class="param">product_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numerical ID of the product that the option belongs to.
 </div>
<div class="param">display_name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The name of the option shown on the storefront.
 </div>
<div class="param">type (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The type of option, which determines how it will display on the storefront. For reference, the former v2 API values are:
RB &#x3D; radio_buttons, RT &#x3D; rectangles, S &#x3D; dropdown, P &#x3D; product_list, PI &#x3D; product_list_with_images, CS &#x3D; swatch.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">radio_buttons</div><div class="param-enum">rectangles</div><div class="param-enum">dropdown</div><div class="param-enum">product_list</div><div class="param-enum">product_list_with_images</div><div class="param-enum">swatch</div>
<div class="param">config (optional)</div><div class="param-desc"><span class="param-type"><a href="#OptionConfig">OptionConfig</a></span>  </div>
<div class="param">option_values (optional)</div><div class="param-desc"><span class="param-type"><a href="#OptionValue">array[OptionValue]</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="OptionPut">OptionPut - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a PUT to update option on a product.
</div>
    <div class="field-items">
      <div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numerical ID of the option, increments sequentially.
 </div>
<div class="param">product_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numerical ID of the product that the option belongs to.
 </div>
<div class="param">display_name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The name of the option shown on the storefront.
 </div>
<div class="param">type (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The type of option, which determines how it will display on the storefront. For reference, the former v2 API values are:
RB &#x3D; radio_buttons, RT &#x3D; rectangles, S &#x3D; dropdown, P &#x3D; product_list, PI &#x3D; product_list_with_images, CS &#x3D; swatch.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">radio_buttons</div><div class="param-enum">rectangles</div><div class="param-enum">dropdown</div><div class="param-enum">product_list</div><div class="param-enum">product_list_with_images</div><div class="param-enum">swatch</div>
<div class="param">config (optional)</div><div class="param-desc"><span class="param-type"><a href="#OptionConfig">OptionConfig</a></span>  </div>
<div class="param">option_values (optional)</div><div class="param-desc"><span class="param-type"><a href="#OptionValue">array[OptionValue]</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="OptionResponse">OptionResponse - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Response payload for the Bigcommerce API.
</div>
    <div class="field-items">
      <div class="param">data (optional)</div><div class="param-desc"><span class="param-type"><a href="#Option">Option</a></span>  </div>
<div class="param">meta (optional)</div><div class="param-desc"><span class="param-type"><a href="#Meta">Meta</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="OptionValue">OptionValue - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'></div>
    <div class="field-items">
      <div class="param">is_default (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> The flag for preselecting a value as the default on the storefront. This field is not supported for swatch options/modifiers.
 </div>
<div class="param">label (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The text display identifying the value on the storefront.
 </div>
<div class="param">sort_order (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The order in which the value will be displayed on the product page.
 </div>
<div class="param">value_data (optional)</div><div class="param-desc"><span class="param-type"><a href="#object">Object</a></span> Extra data describing the value, based on the type of option or modifier with which the value is associated. &#x60;swatch&#x60; requires an array of colors, with up to three hexidecimal color keys; &#x60;product list&#x60; requires a &#x60;product_id&#x60;; and &#x60;checkbox&#x60; requires a boolean flag, called &#x60;checked_value&#x60;, to determine which value is considered to be the checked state.
 </div>
<div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the value; increments sequentially.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="OptionValueBase">OptionValueBase - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Common OptionValue properties.</div>
    <div class="field-items">
      <div class="param">is_default (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> The flag for preselecting a value as the default on the storefront. This field is not supported for swatch options/modifiers.
 </div>
<div class="param">label (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The text display identifying the value on the storefront.
 </div>
<div class="param">sort_order (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The order in which the value will be displayed on the product page.
 </div>
<div class="param">value_data (optional)</div><div class="param-desc"><span class="param-type"><a href="#object">Object</a></span> Extra data describing the value, based on the type of option or modifier with which the value is associated. &#x60;swatch&#x60; requires an array of colors, with up to three hexidecimal color keys; &#x60;product list&#x60; requires a &#x60;product_id&#x60;; and &#x60;checkbox&#x60; requires a boolean flag, called &#x60;checked_value&#x60;, to determine which value is considered to be the checked state.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="OptionValueCollectionResponse">OptionValueCollectionResponse - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Response payload for the Bigcommerce API.
</div>
    <div class="field-items">
      <div class="param">data (optional)</div><div class="param-desc"><span class="param-type"><a href="#OptionValue">array[OptionValue]</a></span>  </div>
<div class="param">meta (optional)</div><div class="param-desc"><span class="param-type"><a href="#CollectionMeta">CollectionMeta</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="OptionValuePost">OptionValuePost - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a POST to create option value on a product.
</div>
    <div class="field-items">
      <div class="param">is_default (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> The flag for preselecting a value as the default on the storefront. This field is not supported for swatch options/modifiers.
 </div>
<div class="param">label (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The text display identifying the value on the storefront.
 </div>
<div class="param">sort_order (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The order in which the value will be displayed on the product page.
 </div>
<div class="param">value_data (optional)</div><div class="param-desc"><span class="param-type"><a href="#object">Object</a></span> Extra data describing the value, based on the type of option or modifier with which the value is associated. &#x60;swatch&#x60; requires an array of colors, with up to three hexidecimal color keys; &#x60;product list&#x60; requires a &#x60;product_id&#x60;; and &#x60;checkbox&#x60; requires a boolean flag, called &#x60;checked_value&#x60;, to determine which value is considered to be the checked state.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="OptionValueProductBase">OptionValueProductBase - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Common OptionValueProduct properties.</div>
    <div class="field-items">
      <div class="param">option_display_name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The name of the option.
 </div>
<div class="param">label (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The label of the option value.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="OptionValueProductPost">OptionValueProductPost - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a POST to create option values with a product.
</div>
    <div class="field-items">
      <div class="param">option_display_name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The name of the option.
 </div>
<div class="param">label (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The label of the option value.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="OptionValuePut">OptionValuePut - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a PUT to update option value on a product.
</div>
    <div class="field-items">
      <div class="param">is_default (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> The flag for preselecting a value as the default on the storefront. This field is not supported for swatch options/modifiers.
 </div>
<div class="param">label (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The text display identifying the value on the storefront.
 </div>
<div class="param">sort_order (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The order in which the value will be displayed on the product page.
 </div>
<div class="param">value_data (optional)</div><div class="param-desc"><span class="param-type"><a href="#object">Object</a></span> Extra data describing the value, based on the type of option or modifier with which the value is associated. &#x60;swatch&#x60; requires an array of colors, with up to three hexidecimal color keys; &#x60;product list&#x60; requires a &#x60;product_id&#x60;; and &#x60;checkbox&#x60; requires a boolean flag, called &#x60;checked_value&#x60;, to determine which value is considered to be the checked state.
 </div>
<div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the value; increments sequentially.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="OptionValueResponse">OptionValueResponse - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Response payload for the Bigcommerce API.
</div>
    <div class="field-items">
      <div class="param">data (optional)</div><div class="param-desc"><span class="param-type"><a href="#OptionValue">OptionValue</a></span>  </div>
<div class="param">meta (optional)</div><div class="param-desc"><span class="param-type"><a href="#Meta">Meta</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="OptionValueVariant">OptionValueVariant - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'></div>
    <div class="field-items">
      <div class="param">option_display_name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The name of the option.
 </div>
<div class="param">label (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The label of the option value.
 </div>
<div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span>  </div>
<div class="param">option_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="OptionValueVariantPost">OptionValueVariantPost - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a POST to create option values with a variant.
</div>
    <div class="field-items">
      <div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span>  </div>
<div class="param">option_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="Pagination">Pagination - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Data about the response, including pagination and collection totals.
</div>
    <div class="field-items">
      <div class="param">total (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> Total number of items in the result set.
 </div>
<div class="param">count (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> Total number of items in the collection response.
 </div>
<div class="param">per_page (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The amount of items returned in the collection per page, controlled by the limit parameter.
 </div>
<div class="param">current_page (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The page you are currently on within the collection.
 </div>
<div class="param">total_pages (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The total number of pages in the collection.
 </div>
<div class="param">links (optional)</div><div class="param-desc"><span class="param-type"><a href="#Pagination_links">Pagination_links</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="Pagination_links">Pagination_links - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Pagination links for the previous and next parts of the whole collection.
</div>
    <div class="field-items">
      <div class="param">previous (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Link to the previous page returned in the response.
 </div>
<div class="param">current (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Link to the current page returned in the response.
 </div>
<div class="param">next (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Link to the next page returned in the response.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="Product">Product - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>A BigCommerce Product object describes a single purchasable unit or a collection of purchasable units.
</div>
    <div class="field-items">
      <div class="param">name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The product name.
 </div>
<div class="param">type (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The product type: physical - a physical stock unit, digital - a digital download
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">physical</div><div class="param-enum">digital</div>
<div class="param">sku (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> User defined product code/stock keeping unit (SKU).
 </div>
<div class="param">description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The product description, which can include HTML formatting.
 </div>
<div class="param">weight (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> Weight of the product, which can be used when calculating shipping costs.
 format: double</div>
<div class="param">width (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> Width of the product, which can be used when calculating shipping costs.
 format: double</div>
<div class="param">depth (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> Depth of the product, which can be used when calculating shipping costs.
 format: double</div>
<div class="param">height (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> Height of the product, which can be used when calculating shipping costs.
 format: double</div>
<div class="param">price (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> The price of the product. The price should include or exclude tax, based on the store settings.
 format: double</div>
<div class="param">cost_price (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> The cost price of the product. Stored for reference only; it is not used or displayed anywhere on the store.
 format: double</div>
<div class="param">retail_price (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> The retail cost of the product. If entered, the retail cost price will be shown on the product page.
 format: double</div>
<div class="param">sale_price (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> If entered, the sale price will be used instead of value in the price field when calculating the product&#39;s cost.
 format: double</div>
<div class="param">tax_class_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The ID of the tax class applied to the product. (NOTE: Value ignored if automatic tax is enabled.)
 </div>
<div class="param">product_tax_code (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Accepts AvaTax System Tax Codes, which identify products and services that fall into special sales-tax categories. By using these codes, merchants who subscribe to Avalara Premium can calculate sales taxes more accurately. Stores without Avalara Premium will ignore the code when calculating sales tax. Do not pass more than one code. The codes are case-sensitive. For details, please see Avalara&#39;s documentation.
 </div>
<div class="param">categories (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">array[Integer]</a></span> An array of IDs for the categories to which this product belongs. When updating a product, if an array of categories is supplied, all product categories will be overwritten. Does not accept more than 1,000 ID values.
 </div>
<div class="param">brand_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The ID associated with the product&#39;s brand.
 </div>
<div class="param">inventory_level (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> Current inventory level of the product. Simple inventory tracking must be enabled (See the inventory_tracking field) for this to take any effect.
 </div>
<div class="param">inventory_warning_level (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> Inventory Warning level for the product. When the product&#39;s inventory level drops below the warning level, the store owner will be informed. Simple inventory tracking must be enabled (see the &#x60;inventory_tracking&#x60; field) for this to take any effect.
 </div>
<div class="param">inventory_tracking (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The type of inventory tracking for the product. Values are: none - inventory levels will not be tracked; product - inventory levels will be tracked using the &#x60;inventory_level&#x60; and &#x60;inventory_warning_level&#x60; fields; variant - inventory levels will be tracked based on variants, which maintain their own warning levels and inventory levels.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">none</div><div class="param-enum">product</div><div class="param-enum">variant</div>
<div class="param">fixed_cost_shipping_price (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> A fixed shipping cost for the product. If defined, this value will be used during checkout instead of normal shipping-cost calculation.
 format: double</div>
<div class="param">is_free_shipping (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag used to indicate whether the product has free shipping. If &#x60;true&#x60;, the shipping cost for the product will be zero.
 </div>
<div class="param">is_visible (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag to determine whether the product should be displayed to customers browsing the store. If &#x60;true&#x60;, the product will be displayed. If &#x60;false&#x60;, the product will be hidden from view.
 </div>
<div class="param">is_featured (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag to determine whether the product should be included in the &#x60;featured products&#x60; panel when viewing the store.
 </div>
<div class="param">related_products (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">array[Integer]</a></span> An array of IDs for the related products.
 </div>
<div class="param">warranty (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Warranty information displayed on the product page. Can include HTML formatting.
 </div>
<div class="param">bin_picking_number (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The BIN picking number for the product.
 </div>
<div class="param">layout_file (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The layout template file used to render this product.
 </div>
<div class="param">upc (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The product UPC code, which is used in feeds for shopping comparison sites and external channel integrations.
 </div>
<div class="param">search_keywords (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> A comma-separated list of keywords that can be used to locate the product when searching the store.
 </div>
<div class="param">availability (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Availability of the product. Availability options are: available - the product can be purchased in the storefront; disabled - the product is listed in the storefront, but cannot be purchased; preorder - the product is listed for pre-orders.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">available</div><div class="param-enum">disabled</div><div class="param-enum">preorder</div>
<div class="param">availability_description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Availability text displayed on the checkout page, under the product title. Tells the customer how long it will normally take to ship this product, such as &#39;Usually ships in 24 hours.&#39;
 </div>
<div class="param">gift_wrapping_options_type (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Type of gift-wrapping options. Values: &#x60;any&#x60; - allow any gift-wrapping options in the store; &#x60;none&#x60; - disallow gift-wrapping on the product; &#x60;list&#x60; – provide a list of IDs in &#x60;gift_wrapping_options_list&#x60; field.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">any</div><div class="param-enum">none</div><div class="param-enum">list</div>
<div class="param">gift_wrapping_options_list (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">array[Integer]</a></span> A list of gift-wrapping option IDs.
 </div>
<div class="param">sort_order (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> Priority to give this product when included in product lists on category pages and in search results. Lower integers will place the product closer to the top of the results.
 </div>
<div class="param">condition (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The product condition. Will be shown on the product page if the &#x60;is_condition_shown&#x60; field&#39;s value is &#x60;true&#x60;. Possible values: &#x60;New&#x60;, &#x60;Used&#x60;, &#x60;Refurbished&#x60;.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">New</div><div class="param-enum">Used</div><div class="param-enum">Refurbished</div>
<div class="param">is_condition_shown (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag used to determine whether the product condition is shown to the customer on the product page.
 </div>
<div class="param">order_quantity_minimum (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The minimum quantity an order must contain, to be eligible to purchase this product.
 </div>
<div class="param">order_quantity_maximum (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The maximum quantity an order can contain when purchasing the product.
 </div>
<div class="param">page_title (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Custom title for the product page. If not defined, the product name will be used as the meta title.
 </div>
<div class="param">meta_keywords (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">array[String]</a></span> Custom meta keywords for the product page. If not defined, the store&#39;s default keywords will be used.
 </div>
<div class="param">meta_description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Custom meta description for the product page. If not defined, the store&#39;s default meta description will be used.
 </div>
<div class="param">view_count (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The number of times the product has been viewed.
 </div>
<div class="param">preorder_release_date (optional)</div><div class="param-desc"><span class="param-type"><a href="#DateTime">Date</a></span> Pre-order release date. See the &#x60;availability&#x60; field for details on setting a product&#39;s availability to accept pre-orders.
 format: date-time</div>
<div class="param">preorder_message (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Custom expected-date message to display on the product page. If undefined, the message defaults to the storewide setting. Can contain the &#x60;%%DATE%%&#x60; placeholder, which will be substituted for the release date.
 </div>
<div class="param">is_preorder_only (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> If set to &#x60;false&#x60;, the product will not change its availability from 	&#x60;preorder&#x60; to &#x60;available&#x60; on the release date. Otherwise, on the release date the product&#39;s availability/status will change to &#x60;available&#x60;.
 </div>
<div class="param">is_price_hidden (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> False by default, indicating that this product&#39;s price should be shown on the product page. If set to &#x60;true&#x60;, the price is hidden. (NOTE: To successfully set &#x60;is_price_hidden&#x60; to &#x60;true&#x60;, the &#x60;availability&#x60; value must be &#x60;disabled&#x60;.)
 </div>
<div class="param">price_hidden_label (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> By default, an empty string. If &#x60;is_price_hidden&#x60; is &#x60;true&#x60;, the value of &#x60;price_hidden_label&#x60; is displayed instead of the price. (NOTE: To successfully set a non-empty string value with &#x60;is_price_hidden&#x60; set to &#x60;true&#x60;, the &#x60;availability&#x60; value must be &#x60;disabled&#x60;.)
 </div>
<div class="param">custom_url (optional)</div><div class="param-desc"><span class="param-type"><a href="#CustomUrlProduct">CustomUrlProduct</a></span>  </div>
<div class="param">bulk_pricing_rules (optional)</div><div class="param-desc"><span class="param-type"><a href="#BulkPricingRule">array[BulkPricingRule]</a></span>  </div>
<div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the product; increments sequentially.
 </div>
<div class="param">calculated_price (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> The price of the product, unless a &#x60;sale_price&#x60; is set.
 format: double</div>
<div class="param">custom_fields (optional)</div><div class="param-desc"><span class="param-type"><a href="#CustomField">array[CustomField]</a></span>  </div>
<div class="param">date_created (optional)</div><div class="param-desc"><span class="param-type"><a href="#DateTime">Date</a></span> The date on which the product was created.
 format: date-time</div>
<div class="param">date_modified (optional)</div><div class="param-desc"><span class="param-type"><a href="#DateTime">Date</a></span> The date on which the product was modified.
 format: date-time</div>
<div class="param">images (optional)</div><div class="param-desc"><span class="param-type"><a href="#ProductImage">array[ProductImage]</a></span>  </div>
<div class="param">videos (optional)</div><div class="param-desc"><span class="param-type"><a href="#ProductVideo">array[ProductVideo]</a></span>  </div>
<div class="param">variants (optional)</div><div class="param-desc"><span class="param-type"><a href="#Variant">array[Variant]</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ProductBase">ProductBase - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Common Product properties.
</div>
    <div class="field-items">
      <div class="param">name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The product name.
 </div>
<div class="param">type (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The product type: physical - a physical stock unit, digital - a digital download
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">physical</div><div class="param-enum">digital</div>
<div class="param">sku (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> User defined product code/stock keeping unit (SKU).
 </div>
<div class="param">description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The product description, which can include HTML formatting.
 </div>
<div class="param">weight (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> Weight of the product, which can be used when calculating shipping costs.
 format: double</div>
<div class="param">width (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> Width of the product, which can be used when calculating shipping costs.
 format: double</div>
<div class="param">depth (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> Depth of the product, which can be used when calculating shipping costs.
 format: double</div>
<div class="param">height (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> Height of the product, which can be used when calculating shipping costs.
 format: double</div>
<div class="param">price (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> The price of the product. The price should include or exclude tax, based on the store settings.
 format: double</div>
<div class="param">cost_price (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> The cost price of the product. Stored for reference only; it is not used or displayed anywhere on the store.
 format: double</div>
<div class="param">retail_price (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> The retail cost of the product. If entered, the retail cost price will be shown on the product page.
 format: double</div>
<div class="param">sale_price (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> If entered, the sale price will be used instead of value in the price field when calculating the product&#39;s cost.
 format: double</div>
<div class="param">tax_class_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The ID of the tax class applied to the product. (NOTE: Value ignored if automatic tax is enabled.)
 </div>
<div class="param">product_tax_code (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Accepts AvaTax System Tax Codes, which identify products and services that fall into special sales-tax categories. By using these codes, merchants who subscribe to Avalara Premium can calculate sales taxes more accurately. Stores without Avalara Premium will ignore the code when calculating sales tax. Do not pass more than one code. The codes are case-sensitive. For details, please see Avalara&#39;s documentation.
 </div>
<div class="param">categories (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">array[Integer]</a></span> An array of IDs for the categories to which this product belongs. When updating a product, if an array of categories is supplied, all product categories will be overwritten. Does not accept more than 1,000 ID values.
 </div>
<div class="param">brand_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The ID associated with the product&#39;s brand.
 </div>
<div class="param">inventory_level (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> Current inventory level of the product. Simple inventory tracking must be enabled (See the inventory_tracking field) for this to take any effect.
 </div>
<div class="param">inventory_warning_level (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> Inventory Warning level for the product. When the product&#39;s inventory level drops below the warning level, the store owner will be informed. Simple inventory tracking must be enabled (see the &#x60;inventory_tracking&#x60; field) for this to take any effect.
 </div>
<div class="param">inventory_tracking (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The type of inventory tracking for the product. Values are: none - inventory levels will not be tracked; product - inventory levels will be tracked using the &#x60;inventory_level&#x60; and &#x60;inventory_warning_level&#x60; fields; variant - inventory levels will be tracked based on variants, which maintain their own warning levels and inventory levels.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">none</div><div class="param-enum">product</div><div class="param-enum">variant</div>
<div class="param">fixed_cost_shipping_price (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> A fixed shipping cost for the product. If defined, this value will be used during checkout instead of normal shipping-cost calculation.
 format: double</div>
<div class="param">is_free_shipping (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag used to indicate whether the product has free shipping. If &#x60;true&#x60;, the shipping cost for the product will be zero.
 </div>
<div class="param">is_visible (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag to determine whether the product should be displayed to customers browsing the store. If &#x60;true&#x60;, the product will be displayed. If &#x60;false&#x60;, the product will be hidden from view.
 </div>
<div class="param">is_featured (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag to determine whether the product should be included in the &#x60;featured products&#x60; panel when viewing the store.
 </div>
<div class="param">related_products (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">array[Integer]</a></span> An array of IDs for the related products.
 </div>
<div class="param">warranty (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Warranty information displayed on the product page. Can include HTML formatting.
 </div>
<div class="param">bin_picking_number (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The BIN picking number for the product.
 </div>
<div class="param">layout_file (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The layout template file used to render this product.
 </div>
<div class="param">upc (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The product UPC code, which is used in feeds for shopping comparison sites and external channel integrations.
 </div>
<div class="param">search_keywords (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> A comma-separated list of keywords that can be used to locate the product when searching the store.
 </div>
<div class="param">availability (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Availability of the product. Availability options are: available - the product can be purchased in the storefront; disabled - the product is listed in the storefront, but cannot be purchased; preorder - the product is listed for pre-orders.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">available</div><div class="param-enum">disabled</div><div class="param-enum">preorder</div>
<div class="param">availability_description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Availability text displayed on the checkout page, under the product title. Tells the customer how long it will normally take to ship this product, such as &#39;Usually ships in 24 hours.&#39;
 </div>
<div class="param">gift_wrapping_options_type (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Type of gift-wrapping options. Values: &#x60;any&#x60; - allow any gift-wrapping options in the store; &#x60;none&#x60; - disallow gift-wrapping on the product; &#x60;list&#x60; – provide a list of IDs in &#x60;gift_wrapping_options_list&#x60; field.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">any</div><div class="param-enum">none</div><div class="param-enum">list</div>
<div class="param">gift_wrapping_options_list (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">array[Integer]</a></span> A list of gift-wrapping option IDs.
 </div>
<div class="param">sort_order (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> Priority to give this product when included in product lists on category pages and in search results. Lower integers will place the product closer to the top of the results.
 </div>
<div class="param">condition (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The product condition. Will be shown on the product page if the &#x60;is_condition_shown&#x60; field&#39;s value is &#x60;true&#x60;. Possible values: &#x60;New&#x60;, &#x60;Used&#x60;, &#x60;Refurbished&#x60;.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">New</div><div class="param-enum">Used</div><div class="param-enum">Refurbished</div>
<div class="param">is_condition_shown (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag used to determine whether the product condition is shown to the customer on the product page.
 </div>
<div class="param">order_quantity_minimum (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The minimum quantity an order must contain, to be eligible to purchase this product.
 </div>
<div class="param">order_quantity_maximum (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The maximum quantity an order can contain when purchasing the product.
 </div>
<div class="param">page_title (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Custom title for the product page. If not defined, the product name will be used as the meta title.
 </div>
<div class="param">meta_keywords (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">array[String]</a></span> Custom meta keywords for the product page. If not defined, the store&#39;s default keywords will be used.
 </div>
<div class="param">meta_description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Custom meta description for the product page. If not defined, the store&#39;s default meta description will be used.
 </div>
<div class="param">view_count (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The number of times the product has been viewed.
 </div>
<div class="param">preorder_release_date (optional)</div><div class="param-desc"><span class="param-type"><a href="#DateTime">Date</a></span> Pre-order release date. See the &#x60;availability&#x60; field for details on setting a product&#39;s availability to accept pre-orders.
 format: date-time</div>
<div class="param">preorder_message (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Custom expected-date message to display on the product page. If undefined, the message defaults to the storewide setting. Can contain the &#x60;%%DATE%%&#x60; placeholder, which will be substituted for the release date.
 </div>
<div class="param">is_preorder_only (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> If set to &#x60;false&#x60;, the product will not change its availability from 	&#x60;preorder&#x60; to &#x60;available&#x60; on the release date. Otherwise, on the release date the product&#39;s availability/status will change to &#x60;available&#x60;.
 </div>
<div class="param">is_price_hidden (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> False by default, indicating that this product&#39;s price should be shown on the product page. If set to &#x60;true&#x60;, the price is hidden. (NOTE: To successfully set &#x60;is_price_hidden&#x60; to &#x60;true&#x60;, the &#x60;availability&#x60; value must be &#x60;disabled&#x60;.)
 </div>
<div class="param">price_hidden_label (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> By default, an empty string. If &#x60;is_price_hidden&#x60; is &#x60;true&#x60;, the value of &#x60;price_hidden_label&#x60; is displayed instead of the price. (NOTE: To successfully set a non-empty string value with &#x60;is_price_hidden&#x60; set to &#x60;true&#x60;, the &#x60;availability&#x60; value must be &#x60;disabled&#x60;.)
 </div>
<div class="param">custom_url (optional)</div><div class="param-desc"><span class="param-type"><a href="#CustomUrlProduct">CustomUrlProduct</a></span>  </div>
<div class="param">bulk_pricing_rules (optional)</div><div class="param-desc"><span class="param-type"><a href="#BulkPricingRule">array[BulkPricingRule]</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ProductCollectionResponse">ProductCollectionResponse - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Response payload for the Bigcommerce API.
</div>
    <div class="field-items">
      <div class="param">data (optional)</div><div class="param-desc"><span class="param-type"><a href="#Product">array[Product]</a></span>  </div>
<div class="param">meta (optional)</div><div class="param-desc"><span class="param-type"><a href="#CollectionMeta">CollectionMeta</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ProductImage">ProductImage - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The full ProductImage model.
</div>
    <div class="field-items">
      <div class="param">is_thumbnail (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag for identifying whether the image is used as the product&#39;s thumbnail.
 </div>
<div class="param">sort_order (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The order in which the image will be displayed on the product page. Higher integers give the image a lower priority. When updating, if the image is given a lower priority, all images with a &#x60;sort_order&#x60; the same as or greater than the image&#39;s new &#x60;sort_order&#x60; value will have their &#x60;sort_order&#x60;s reordered.
 </div>
<div class="param">description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The description for the image.
 </div>
<div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the image; increments sequentially.
 </div>
<div class="param">product_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric identifier for the product with which the image is associated.
 </div>
<div class="param">image_file (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The local path to the original image file uploaded to BigCommerce.
 </div>
<div class="param">url_zoom (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The zoom URL for this image. By default, this is used as the zoom image on product pages when zoom images are enabled.
 </div>
<div class="param">url_standard (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The standard URL for this image. By default, this is used for product-page images.
 </div>
<div class="param">url_thumbnail (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The thumbnail URL for this image. By default, this is the image size used on the category page and in side panels.
 </div>
<div class="param">url_tiny (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The tiny URL for this image. By default, this is the image size used for thumbnails beneath the product image on a product page.
 </div>
<div class="param">date_modified (optional)</div><div class="param-desc"><span class="param-type"><a href="#DateTime">Date</a></span> The date on which the product image was modified.
 format: date-time</div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ProductImageBase">ProductImageBase - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Common ProductImage properties.
</div>
    <div class="field-items">
      <div class="param">is_thumbnail (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag for identifying whether the image is used as the product&#39;s thumbnail.
 </div>
<div class="param">sort_order (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The order in which the image will be displayed on the product page. Higher integers give the image a lower priority. When updating, if the image is given a lower priority, all images with a &#x60;sort_order&#x60; the same as or greater than the image&#39;s new &#x60;sort_order&#x60; value will have their &#x60;sort_order&#x60;s reordered.
 </div>
<div class="param">description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The description for the image.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ProductImageCollectionResponse">ProductImageCollectionResponse - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Response payload for the Bigcommerce API.
</div>
    <div class="field-items">
      <div class="param">data (optional)</div><div class="param-desc"><span class="param-type"><a href="#ProductImage">array[ProductImage]</a></span>  </div>
<div class="param">meta (optional)</div><div class="param-desc"><span class="param-type"><a href="#CollectionMeta">CollectionMeta</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ProductImagePost">ProductImagePost - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a POST to create an image on a product.
</div>
    <div class="field-items">
      <div class="param">is_thumbnail (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag for identifying whether the image is used as the product&#39;s thumbnail.
 </div>
<div class="param">sort_order (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The order in which the image will be displayed on the product page. Higher integers give the image a lower priority. When updating, if the image is given a lower priority, all images with a &#x60;sort_order&#x60; the same as or greater than the image&#39;s new &#x60;sort_order&#x60; value will have their &#x60;sort_order&#x60;s reordered.
 </div>
<div class="param">description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The description for the image.
 </div>
<div class="param">image_url (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Must be a fully qualified URL path, including protocol.
 </div>
<div class="param">image_file (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Must be sent as a multipart/form-data field in the request body.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ProductImagePut">ProductImagePut - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a PUT to update applicable ProductImage fields.
</div>
    <div class="field-items">
      <div class="param">is_thumbnail (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag for identifying whether the image is used as the product&#39;s thumbnail.
 </div>
<div class="param">sort_order (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The order in which the image will be displayed on the product page. Higher integers give the image a lower priority. When updating, if the image is given a lower priority, all images with a &#x60;sort_order&#x60; the same as or greater than the image&#39;s new &#x60;sort_order&#x60; value will have their &#x60;sort_order&#x60;s reordered.
 </div>
<div class="param">description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The description for the image.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ProductImageResponse">ProductImageResponse - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Response payload for the Bigcommerce API.
</div>
    <div class="field-items">
      <div class="param">data (optional)</div><div class="param-desc"><span class="param-type"><a href="#ProductImage">ProductImage</a></span>  </div>
<div class="param">meta (optional)</div><div class="param-desc"><span class="param-type"><a href="#Meta">Meta</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ProductPost">ProductPost - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a POST to create product.
</div>
    <div class="field-items">
      <div class="param">name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The product name.
 </div>
<div class="param">type (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The product type: physical - a physical stock unit, digital - a digital download
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">physical</div><div class="param-enum">digital</div>
<div class="param">sku (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> User defined product code/stock keeping unit (SKU).
 </div>
<div class="param">description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The product description, which can include HTML formatting.
 </div>
<div class="param">weight (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> Weight of the product, which can be used when calculating shipping costs.
 format: double</div>
<div class="param">width (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> Width of the product, which can be used when calculating shipping costs.
 format: double</div>
<div class="param">depth (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> Depth of the product, which can be used when calculating shipping costs.
 format: double</div>
<div class="param">height (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> Height of the product, which can be used when calculating shipping costs.
 format: double</div>
<div class="param">price (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> The price of the product. The price should include or exclude tax, based on the store settings.
 format: double</div>
<div class="param">cost_price (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> The cost price of the product. Stored for reference only; it is not used or displayed anywhere on the store.
 format: double</div>
<div class="param">retail_price (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> The retail cost of the product. If entered, the retail cost price will be shown on the product page.
 format: double</div>
<div class="param">sale_price (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> If entered, the sale price will be used instead of value in the price field when calculating the product&#39;s cost.
 format: double</div>
<div class="param">tax_class_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The ID of the tax class applied to the product. (NOTE: Value ignored if automatic tax is enabled.)
 </div>
<div class="param">product_tax_code (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Accepts AvaTax System Tax Codes, which identify products and services that fall into special sales-tax categories. By using these codes, merchants who subscribe to Avalara Premium can calculate sales taxes more accurately. Stores without Avalara Premium will ignore the code when calculating sales tax. Do not pass more than one code. The codes are case-sensitive. For details, please see Avalara&#39;s documentation.
 </div>
<div class="param">categories (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">array[Integer]</a></span> An array of IDs for the categories to which this product belongs. When updating a product, if an array of categories is supplied, all product categories will be overwritten. Does not accept more than 1,000 ID values.
 </div>
<div class="param">brand_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The ID associated with the product&#39;s brand.
 </div>
<div class="param">inventory_level (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> Current inventory level of the product. Simple inventory tracking must be enabled (See the inventory_tracking field) for this to take any effect.
 </div>
<div class="param">inventory_warning_level (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> Inventory Warning level for the product. When the product&#39;s inventory level drops below the warning level, the store owner will be informed. Simple inventory tracking must be enabled (see the &#x60;inventory_tracking&#x60; field) for this to take any effect.
 </div>
<div class="param">inventory_tracking (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The type of inventory tracking for the product. Values are: none - inventory levels will not be tracked; product - inventory levels will be tracked using the &#x60;inventory_level&#x60; and &#x60;inventory_warning_level&#x60; fields; variant - inventory levels will be tracked based on variants, which maintain their own warning levels and inventory levels.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">none</div><div class="param-enum">product</div><div class="param-enum">variant</div>
<div class="param">fixed_cost_shipping_price (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> A fixed shipping cost for the product. If defined, this value will be used during checkout instead of normal shipping-cost calculation.
 format: double</div>
<div class="param">is_free_shipping (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag used to indicate whether the product has free shipping. If &#x60;true&#x60;, the shipping cost for the product will be zero.
 </div>
<div class="param">is_visible (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag to determine whether the product should be displayed to customers browsing the store. If &#x60;true&#x60;, the product will be displayed. If &#x60;false&#x60;, the product will be hidden from view.
 </div>
<div class="param">is_featured (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag to determine whether the product should be included in the &#x60;featured products&#x60; panel when viewing the store.
 </div>
<div class="param">related_products (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">array[Integer]</a></span> An array of IDs for the related products.
 </div>
<div class="param">warranty (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Warranty information displayed on the product page. Can include HTML formatting.
 </div>
<div class="param">bin_picking_number (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The BIN picking number for the product.
 </div>
<div class="param">layout_file (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The layout template file used to render this product.
 </div>
<div class="param">upc (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The product UPC code, which is used in feeds for shopping comparison sites and external channel integrations.
 </div>
<div class="param">search_keywords (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> A comma-separated list of keywords that can be used to locate the product when searching the store.
 </div>
<div class="param">availability (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Availability of the product. Availability options are: available - the product can be purchased in the storefront; disabled - the product is listed in the storefront, but cannot be purchased; preorder - the product is listed for pre-orders.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">available</div><div class="param-enum">disabled</div><div class="param-enum">preorder</div>
<div class="param">availability_description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Availability text displayed on the checkout page, under the product title. Tells the customer how long it will normally take to ship this product, such as &#39;Usually ships in 24 hours.&#39;
 </div>
<div class="param">gift_wrapping_options_type (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Type of gift-wrapping options. Values: &#x60;any&#x60; - allow any gift-wrapping options in the store; &#x60;none&#x60; - disallow gift-wrapping on the product; &#x60;list&#x60; – provide a list of IDs in &#x60;gift_wrapping_options_list&#x60; field.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">any</div><div class="param-enum">none</div><div class="param-enum">list</div>
<div class="param">gift_wrapping_options_list (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">array[Integer]</a></span> A list of gift-wrapping option IDs.
 </div>
<div class="param">sort_order (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> Priority to give this product when included in product lists on category pages and in search results. Lower integers will place the product closer to the top of the results.
 </div>
<div class="param">condition (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The product condition. Will be shown on the product page if the &#x60;is_condition_shown&#x60; field&#39;s value is &#x60;true&#x60;. Possible values: &#x60;New&#x60;, &#x60;Used&#x60;, &#x60;Refurbished&#x60;.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">New</div><div class="param-enum">Used</div><div class="param-enum">Refurbished</div>
<div class="param">is_condition_shown (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag used to determine whether the product condition is shown to the customer on the product page.
 </div>
<div class="param">order_quantity_minimum (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The minimum quantity an order must contain, to be eligible to purchase this product.
 </div>
<div class="param">order_quantity_maximum (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The maximum quantity an order can contain when purchasing the product.
 </div>
<div class="param">page_title (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Custom title for the product page. If not defined, the product name will be used as the meta title.
 </div>
<div class="param">meta_keywords (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">array[String]</a></span> Custom meta keywords for the product page. If not defined, the store&#39;s default keywords will be used.
 </div>
<div class="param">meta_description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Custom meta description for the product page. If not defined, the store&#39;s default meta description will be used.
 </div>
<div class="param">view_count (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The number of times the product has been viewed.
 </div>
<div class="param">preorder_release_date (optional)</div><div class="param-desc"><span class="param-type"><a href="#DateTime">Date</a></span> Pre-order release date. See the &#x60;availability&#x60; field for details on setting a product&#39;s availability to accept pre-orders.
 format: date-time</div>
<div class="param">preorder_message (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Custom expected-date message to display on the product page. If undefined, the message defaults to the storewide setting. Can contain the &#x60;%%DATE%%&#x60; placeholder, which will be substituted for the release date.
 </div>
<div class="param">is_preorder_only (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> If set to &#x60;false&#x60;, the product will not change its availability from 	&#x60;preorder&#x60; to &#x60;available&#x60; on the release date. Otherwise, on the release date the product&#39;s availability/status will change to &#x60;available&#x60;.
 </div>
<div class="param">is_price_hidden (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> False by default, indicating that this product&#39;s price should be shown on the product page. If set to &#x60;true&#x60;, the price is hidden. (NOTE: To successfully set &#x60;is_price_hidden&#x60; to &#x60;true&#x60;, the &#x60;availability&#x60; value must be &#x60;disabled&#x60;.)
 </div>
<div class="param">price_hidden_label (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> By default, an empty string. If &#x60;is_price_hidden&#x60; is &#x60;true&#x60;, the value of &#x60;price_hidden_label&#x60; is displayed instead of the price. (NOTE: To successfully set a non-empty string value with &#x60;is_price_hidden&#x60; set to &#x60;true&#x60;, the &#x60;availability&#x60; value must be &#x60;disabled&#x60;.)
 </div>
<div class="param">custom_url (optional)</div><div class="param-desc"><span class="param-type"><a href="#CustomUrlProduct">CustomUrlProduct</a></span>  </div>
<div class="param">bulk_pricing_rules (optional)</div><div class="param-desc"><span class="param-type"><a href="#BulkPricingRule">array[BulkPricingRule]</a></span>  </div>
<div class="param">custom_fields (optional)</div><div class="param-desc"><span class="param-type"><a href="#CustomField">array[CustomField]</a></span>  </div>
<div class="param">variants (optional)</div><div class="param-desc"><span class="param-type"><a href="#VariantProductPost">array[VariantProductPost]</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ProductPut">ProductPut - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a PUT to update product.
</div>
    <div class="field-items">
      <div class="param">name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The product name.
 </div>
<div class="param">type (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The product type: physical - a physical stock unit, digital - a digital download
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">physical</div><div class="param-enum">digital</div>
<div class="param">sku (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> User defined product code/stock keeping unit (SKU).
 </div>
<div class="param">description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The product description, which can include HTML formatting.
 </div>
<div class="param">weight (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> Weight of the product, which can be used when calculating shipping costs.
 format: double</div>
<div class="param">width (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> Width of the product, which can be used when calculating shipping costs.
 format: double</div>
<div class="param">depth (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> Depth of the product, which can be used when calculating shipping costs.
 format: double</div>
<div class="param">height (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> Height of the product, which can be used when calculating shipping costs.
 format: double</div>
<div class="param">price (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> The price of the product. The price should include or exclude tax, based on the store settings.
 format: double</div>
<div class="param">cost_price (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> The cost price of the product. Stored for reference only; it is not used or displayed anywhere on the store.
 format: double</div>
<div class="param">retail_price (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> The retail cost of the product. If entered, the retail cost price will be shown on the product page.
 format: double</div>
<div class="param">sale_price (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> If entered, the sale price will be used instead of value in the price field when calculating the product&#39;s cost.
 format: double</div>
<div class="param">tax_class_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The ID of the tax class applied to the product. (NOTE: Value ignored if automatic tax is enabled.)
 </div>
<div class="param">product_tax_code (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Accepts AvaTax System Tax Codes, which identify products and services that fall into special sales-tax categories. By using these codes, merchants who subscribe to Avalara Premium can calculate sales taxes more accurately. Stores without Avalara Premium will ignore the code when calculating sales tax. Do not pass more than one code. The codes are case-sensitive. For details, please see Avalara&#39;s documentation.
 </div>
<div class="param">categories (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">array[Integer]</a></span> An array of IDs for the categories to which this product belongs. When updating a product, if an array of categories is supplied, all product categories will be overwritten. Does not accept more than 1,000 ID values.
 </div>
<div class="param">brand_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The ID associated with the product&#39;s brand.
 </div>
<div class="param">inventory_level (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> Current inventory level of the product. Simple inventory tracking must be enabled (See the inventory_tracking field) for this to take any effect.
 </div>
<div class="param">inventory_warning_level (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> Inventory Warning level for the product. When the product&#39;s inventory level drops below the warning level, the store owner will be informed. Simple inventory tracking must be enabled (see the &#x60;inventory_tracking&#x60; field) for this to take any effect.
 </div>
<div class="param">inventory_tracking (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The type of inventory tracking for the product. Values are: none - inventory levels will not be tracked; product - inventory levels will be tracked using the &#x60;inventory_level&#x60; and &#x60;inventory_warning_level&#x60; fields; variant - inventory levels will be tracked based on variants, which maintain their own warning levels and inventory levels.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">none</div><div class="param-enum">product</div><div class="param-enum">variant</div>
<div class="param">fixed_cost_shipping_price (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> A fixed shipping cost for the product. If defined, this value will be used during checkout instead of normal shipping-cost calculation.
 format: double</div>
<div class="param">is_free_shipping (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag used to indicate whether the product has free shipping. If &#x60;true&#x60;, the shipping cost for the product will be zero.
 </div>
<div class="param">is_visible (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag to determine whether the product should be displayed to customers browsing the store. If &#x60;true&#x60;, the product will be displayed. If &#x60;false&#x60;, the product will be hidden from view.
 </div>
<div class="param">is_featured (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag to determine whether the product should be included in the &#x60;featured products&#x60; panel when viewing the store.
 </div>
<div class="param">related_products (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">array[Integer]</a></span> An array of IDs for the related products.
 </div>
<div class="param">warranty (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Warranty information displayed on the product page. Can include HTML formatting.
 </div>
<div class="param">bin_picking_number (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The BIN picking number for the product.
 </div>
<div class="param">layout_file (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The layout template file used to render this product.
 </div>
<div class="param">upc (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The product UPC code, which is used in feeds for shopping comparison sites and external channel integrations.
 </div>
<div class="param">search_keywords (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> A comma-separated list of keywords that can be used to locate the product when searching the store.
 </div>
<div class="param">availability (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Availability of the product. Availability options are: available - the product can be purchased in the storefront; disabled - the product is listed in the storefront, but cannot be purchased; preorder - the product is listed for pre-orders.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">available</div><div class="param-enum">disabled</div><div class="param-enum">preorder</div>
<div class="param">availability_description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Availability text displayed on the checkout page, under the product title. Tells the customer how long it will normally take to ship this product, such as &#39;Usually ships in 24 hours.&#39;
 </div>
<div class="param">gift_wrapping_options_type (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Type of gift-wrapping options. Values: &#x60;any&#x60; - allow any gift-wrapping options in the store; &#x60;none&#x60; - disallow gift-wrapping on the product; &#x60;list&#x60; – provide a list of IDs in &#x60;gift_wrapping_options_list&#x60; field.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">any</div><div class="param-enum">none</div><div class="param-enum">list</div>
<div class="param">gift_wrapping_options_list (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">array[Integer]</a></span> A list of gift-wrapping option IDs.
 </div>
<div class="param">sort_order (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> Priority to give this product when included in product lists on category pages and in search results. Lower integers will place the product closer to the top of the results.
 </div>
<div class="param">condition (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The product condition. Will be shown on the product page if the &#x60;is_condition_shown&#x60; field&#39;s value is &#x60;true&#x60;. Possible values: &#x60;New&#x60;, &#x60;Used&#x60;, &#x60;Refurbished&#x60;.
 </div>
        <div class="param-enum-header">Enum:</div>
        <div class="param-enum">New</div><div class="param-enum">Used</div><div class="param-enum">Refurbished</div>
<div class="param">is_condition_shown (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> Flag used to determine whether the product condition is shown to the customer on the product page.
 </div>
<div class="param">order_quantity_minimum (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The minimum quantity an order must contain, to be eligible to purchase this product.
 </div>
<div class="param">order_quantity_maximum (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The maximum quantity an order can contain when purchasing the product.
 </div>
<div class="param">page_title (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Custom title for the product page. If not defined, the product name will be used as the meta title.
 </div>
<div class="param">meta_keywords (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">array[String]</a></span> Custom meta keywords for the product page. If not defined, the store&#39;s default keywords will be used.
 </div>
<div class="param">meta_description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Custom meta description for the product page. If not defined, the store&#39;s default meta description will be used.
 </div>
<div class="param">view_count (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The number of times the product has been viewed.
 </div>
<div class="param">preorder_release_date (optional)</div><div class="param-desc"><span class="param-type"><a href="#DateTime">Date</a></span> Pre-order release date. See the &#x60;availability&#x60; field for details on setting a product&#39;s availability to accept pre-orders.
 format: date-time</div>
<div class="param">preorder_message (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Custom expected-date message to display on the product page. If undefined, the message defaults to the storewide setting. Can contain the &#x60;%%DATE%%&#x60; placeholder, which will be substituted for the release date.
 </div>
<div class="param">is_preorder_only (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> If set to &#x60;false&#x60;, the product will not change its availability from 	&#x60;preorder&#x60; to &#x60;available&#x60; on the release date. Otherwise, on the release date the product&#39;s availability/status will change to &#x60;available&#x60;.
 </div>
<div class="param">is_price_hidden (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> False by default, indicating that this product&#39;s price should be shown on the product page. If set to &#x60;true&#x60;, the price is hidden. (NOTE: To successfully set &#x60;is_price_hidden&#x60; to &#x60;true&#x60;, the &#x60;availability&#x60; value must be &#x60;disabled&#x60;.)
 </div>
<div class="param">price_hidden_label (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> By default, an empty string. If &#x60;is_price_hidden&#x60; is &#x60;true&#x60;, the value of &#x60;price_hidden_label&#x60; is displayed instead of the price. (NOTE: To successfully set a non-empty string value with &#x60;is_price_hidden&#x60; set to &#x60;true&#x60;, the &#x60;availability&#x60; value must be &#x60;disabled&#x60;.)
 </div>
<div class="param">custom_url (optional)</div><div class="param-desc"><span class="param-type"><a href="#CustomUrlProduct">CustomUrlProduct</a></span>  </div>
<div class="param">bulk_pricing_rules (optional)</div><div class="param-desc"><span class="param-type"><a href="#BulkPricingRule">array[BulkPricingRule]</a></span>  </div>
<div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numerical ID of the product, increments sequentially.
 </div>
<div class="param">custom_fields (optional)</div><div class="param-desc"><span class="param-type"><a href="#CustomField">array[CustomField]</a></span>  </div>
<div class="param">variants (optional)</div><div class="param-desc"><span class="param-type"><a href="#VariantProductPut">array[VariantProductPut]</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ProductResponse">ProductResponse - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Response payload for the Bigcommerce API.
</div>
    <div class="field-items">
      <div class="param">data (optional)</div><div class="param-desc"><span class="param-type"><a href="#Product">Product</a></span>  </div>
<div class="param">meta (optional)</div><div class="param-desc"><span class="param-type"><a href="#Meta">Meta</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ProductVideo">ProductVideo - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>A product video model.
</div>
    <div class="field-items">
      <div class="param">title (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The title for the video. If left blank, this will be filled in according to data on YouTube.
 </div>
<div class="param">description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The description for the video. If left blank, this will be filled in according to data on YouTube.
 </div>
<div class="param">sort_order (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The order in which the video will be displayed on the product page. Higher integers give the video a lower priority. When updating, if the video is given a lower priority, all videos with a &#x60;sort_order&#x60; the same as or greater than the video&#39;s new &#x60;sort_order&#x60; value will have their &#x60;sort_order&#x60;s reordered.
 </div>
<div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The ID of a YouTube video.
 </div>
<div class="param">product_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric identifier for the product with which the image is associated.
 </div>
<div class="param">length (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Length of the video. This will be filled in according to data on YouTube.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ProductVideoBase">ProductVideoBase - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Common ProductVideo properties.</div>
    <div class="field-items">
      <div class="param">title (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The title for the video. If left blank, this will be filled in according to data on YouTube.
 </div>
<div class="param">description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The description for the video. If left blank, this will be filled in according to data on YouTube.
 </div>
<div class="param">sort_order (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The order in which the video will be displayed on the product page. Higher integers give the video a lower priority. When updating, if the video is given a lower priority, all videos with a &#x60;sort_order&#x60; the same as or greater than the video&#39;s new &#x60;sort_order&#x60; value will have their &#x60;sort_order&#x60;s reordered.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ProductVideoCollectionResponse">ProductVideoCollectionResponse - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Response payload for the Bigcommerce API.
</div>
    <div class="field-items">
      <div class="param">data (optional)</div><div class="param-desc"><span class="param-type"><a href="#ProductVideo">array[ProductVideo]</a></span>  </div>
<div class="param">meta (optional)</div><div class="param-desc"><span class="param-type"><a href="#CollectionMeta">CollectionMeta</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ProductVideoPost">ProductVideoPost - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a POST to create video on a product.
</div>
    <div class="field-items">
      <div class="param">title (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The title for the video. If left blank, this will be filled in according to data on YouTube.
 </div>
<div class="param">description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The description for the video. If left blank, this will be filled in according to data on YouTube.
 </div>
<div class="param">sort_order (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The order in which the video will be displayed on the product page. Higher integers give the video a lower priority. When updating, if the video is given a lower priority, all videos with a &#x60;sort_order&#x60; the same as or greater than the video&#39;s new &#x60;sort_order&#x60; value will have their &#x60;sort_order&#x60;s reordered.
 </div>
<div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The ID of a YouTube video.
 </div>
<div class="param">product_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric identifier for the product with which the image is associated.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ProductVideoPut">ProductVideoPut - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a PUT to update video on a product.
</div>
    <div class="field-items">
      <div class="param">title (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The title for the video. If left blank, this will be filled in according to data on YouTube.
 </div>
<div class="param">description (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The description for the video. If left blank, this will be filled in according to data on YouTube.
 </div>
<div class="param">sort_order (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The order in which the video will be displayed on the product page. Higher integers give the video a lower priority. When updating, if the video is given a lower priority, all videos with a &#x60;sort_order&#x60; the same as or greater than the video&#39;s new &#x60;sort_order&#x60; value will have their &#x60;sort_order&#x60;s reordered.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ProductVideoResponse">ProductVideoResponse - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Response payload for the Bigcommerce API.
</div>
    <div class="field-items">
      <div class="param">data (optional)</div><div class="param-desc"><span class="param-type"><a href="#ProductVideo">ProductVideo</a></span>  </div>
<div class="param">meta (optional)</div><div class="param-desc"><span class="param-type"><a href="#Meta">Meta</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="ResourceImage">ResourceImage - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>An object containing a publicly accessible image URL, or a form post that contains an image file.
</div>
    <div class="field-items">
      <div class="param">image_url (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> A public URL for a GIF, JPEG, or PNG image.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="Subscriber">Subscriber - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'></div>
    <div class="field-items">
      <div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the subscriber; increments sequentially.
 </div>
<div class="param">email (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The email of the subscriber. Must be unique.
 </div>
<div class="param">first_name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The first name of the subscriber.
 </div>
<div class="param">last_name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The last name of the subscriber.
 </div>
<div class="param">source (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The source of the subscriber. Values are: &#x60;storefront&#x60;, &#x60;order&#x60;, or &#x60;custom&#x60;.
 </div>
<div class="param">order_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The ID of the source order, if source was an order.
 </div>
<div class="param">date_modified (optional)</div><div class="param-desc"><span class="param-type"><a href="#DateTime">Date</a></span> The date on which the subscriber was modified.
 format: date-time</div>
<div class="param">date_created (optional)</div><div class="param-desc"><span class="param-type"><a href="#DateTime">Date</a></span> The date of which the subscriber was created.
 format: date-time</div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="SubscriberBase">SubscriberBase - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Common Subscriber properties.</div>
    <div class="field-items">
      <div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the subscriber; increments sequentially.
 </div>
<div class="param">email (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The email of the subscriber. Must be unique.
 </div>
<div class="param">first_name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The first name of the subscriber.
 </div>
<div class="param">last_name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The last name of the subscriber.
 </div>
<div class="param">source (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The source of the subscriber. Values are: &#x60;storefront&#x60;, &#x60;order&#x60;, or &#x60;custom&#x60;.
 </div>
<div class="param">order_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The ID of the source order, if source was an order.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="SubscriberCollectionResponse">SubscriberCollectionResponse - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Response payload for the Bigcommerce API.
</div>
    <div class="field-items">
      <div class="param">data (optional)</div><div class="param-desc"><span class="param-type"><a href="#Subscriber">array[Subscriber]</a></span>  </div>
<div class="param">meta (optional)</div><div class="param-desc"><span class="param-type"><a href="#CollectionMeta">CollectionMeta</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="SubscriberPost">SubscriberPost - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a POST to create subscriber.
</div>
    <div class="field-items">
      <div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the subscriber; increments sequentially.
 </div>
<div class="param">email (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The email of the subscriber. Must be unique.
 </div>
<div class="param">first_name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The first name of the subscriber.
 </div>
<div class="param">last_name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The last name of the subscriber.
 </div>
<div class="param">source (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The source of the subscriber. Values are: &#x60;storefront&#x60;, &#x60;order&#x60;, or &#x60;custom&#x60;.
 </div>
<div class="param">order_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The ID of the source order, if source was an order.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="SubscriberPut">SubscriberPut - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a PUT to update subscriber.
</div>
    <div class="field-items">
      <div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The unique numeric ID of the subscriber; increments sequentially.
 </div>
<div class="param">email (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The email of the subscriber. Must be unique.
 </div>
<div class="param">first_name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The first name of the subscriber.
 </div>
<div class="param">last_name (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The last name of the subscriber.
 </div>
<div class="param">source (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The source of the subscriber. Values are: &#x60;storefront&#x60;, &#x60;order&#x60;, or &#x60;custom&#x60;.
 </div>
<div class="param">order_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> The ID of the source order, if source was an order.
 </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="SubscriberResponse">SubscriberResponse - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Response payload for the Bigcommerce API.
</div>
    <div class="field-items">
      <div class="param">data (optional)</div><div class="param-desc"><span class="param-type"><a href="#Subscriber">Subscriber</a></span>  </div>
<div class="param">meta (optional)</div><div class="param-desc"><span class="param-type"><a href="#Meta">Meta</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="Variant">Variant - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'></div>
    <div class="field-items">
      <div class="param">cost_price (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> The cost price of the variant. format: double</div>
<div class="param">price (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> This variant&#39;s base price on the storefront. If this value is null, the product&#39;s default price (set in the Product resource&#39;s &#x60;price&#x60; field) will be used as the base price. format: double</div>
<div class="param">weight (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> This variant&#39;s base weight on the storefront. If this value is null, the product&#39;s default weight (set in the Product resource&#39;s weight field) will be used as the base weight. format: double</div>
<div class="param">purchasing_disabled (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> If &#x60;true&#x60;, this variant will not be purchasable on the storefront. </div>
<div class="param">purchasing_disabled_message (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> If &#x60;purchasing_disabled&#x60; is &#x60;true&#x60;, this message should show on the storefront when the variant is selected. </div>
<div class="param">image_url (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The image that will be displayed when this variant is selected on the storefront. When updating a SKU image, send the publicly accessible URL. Supported image formats are JPEG, PNG, and GIF. Generic product images not specific to the variant should be stored on the product. </div>
<div class="param">upc (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The UPC code used in feeds for shopping comparison sites and external channel integrations. </div>
<div class="param">inventory_level (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> Inventory level for the variant, which is used when the product&#39;s inventory_tracking is set to variant </div>
<div class="param">inventory_warning_level (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> When the variant hits this inventory level, it is considered low stock. </div>
<div class="param">bin_picking_number (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Identifies where in a warehouse the variant is located. </div>
<div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span>  </div>
<div class="param">product_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span>  </div>
<div class="param">sku (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span>  </div>
<div class="param">sku_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> Read-only reference to v2 API&#39;s SKU ID. Null if it is a base variant. </div>
<div class="param">option_values (optional)</div><div class="param-desc"><span class="param-type"><a href="#OptionValueVariant">array[OptionValueVariant]</a></span> Array of option and option values IDs that make up this variant. Will be empty if the variant is the product&#39;s base variant. </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="VariantBase">VariantBase - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Common Variant properties.</div>
    <div class="field-items">
      <div class="param">cost_price (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> The cost price of the variant. format: double</div>
<div class="param">price (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> This variant&#39;s base price on the storefront. If this value is null, the product&#39;s default price (set in the Product resource&#39;s &#x60;price&#x60; field) will be used as the base price. format: double</div>
<div class="param">weight (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> This variant&#39;s base weight on the storefront. If this value is null, the product&#39;s default weight (set in the Product resource&#39;s weight field) will be used as the base weight. format: double</div>
<div class="param">purchasing_disabled (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> If &#x60;true&#x60;, this variant will not be purchasable on the storefront. </div>
<div class="param">purchasing_disabled_message (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> If &#x60;purchasing_disabled&#x60; is &#x60;true&#x60;, this message should show on the storefront when the variant is selected. </div>
<div class="param">image_url (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The image that will be displayed when this variant is selected on the storefront. When updating a SKU image, send the publicly accessible URL. Supported image formats are JPEG, PNG, and GIF. Generic product images not specific to the variant should be stored on the product. </div>
<div class="param">upc (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The UPC code used in feeds for shopping comparison sites and external channel integrations. </div>
<div class="param">inventory_level (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> Inventory level for the variant, which is used when the product&#39;s inventory_tracking is set to variant </div>
<div class="param">inventory_warning_level (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> When the variant hits this inventory level, it is considered low stock. </div>
<div class="param">bin_picking_number (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Identifies where in a warehouse the variant is located. </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="VariantCollectionResponse">VariantCollectionResponse - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Response payload for the Bigcommerce API.
</div>
    <div class="field-items">
      <div class="param">data (optional)</div><div class="param-desc"><span class="param-type"><a href="#Variant">array[Variant]</a></span>  </div>
<div class="param">meta (optional)</div><div class="param-desc"><span class="param-type"><a href="#CollectionMeta">CollectionMeta</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="VariantPost">VariantPost - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a POST to create variant on a product.
</div>
    <div class="field-items">
      <div class="param">cost_price (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> The cost price of the variant. format: double</div>
<div class="param">price (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> This variant&#39;s base price on the storefront. If this value is null, the product&#39;s default price (set in the Product resource&#39;s &#x60;price&#x60; field) will be used as the base price. format: double</div>
<div class="param">weight (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> This variant&#39;s base weight on the storefront. If this value is null, the product&#39;s default weight (set in the Product resource&#39;s weight field) will be used as the base weight. format: double</div>
<div class="param">purchasing_disabled (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> If &#x60;true&#x60;, this variant will not be purchasable on the storefront. </div>
<div class="param">purchasing_disabled_message (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> If &#x60;purchasing_disabled&#x60; is &#x60;true&#x60;, this message should show on the storefront when the variant is selected. </div>
<div class="param">image_url (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The image that will be displayed when this variant is selected on the storefront. When updating a SKU image, send the publicly accessible URL. Supported image formats are JPEG, PNG, and GIF. Generic product images not specific to the variant should be stored on the product. </div>
<div class="param">upc (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The UPC code used in feeds for shopping comparison sites and external channel integrations. </div>
<div class="param">inventory_level (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> Inventory level for the variant, which is used when the product&#39;s inventory_tracking is set to variant </div>
<div class="param">inventory_warning_level (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> When the variant hits this inventory level, it is considered low stock. </div>
<div class="param">bin_picking_number (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Identifies where in a warehouse the variant is located. </div>
<div class="param">product_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span>  </div>
<div class="param">sku (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span>  </div>
<div class="param">option_values (optional)</div><div class="param-desc"><span class="param-type"><a href="#OptionValueVariantPost">array[OptionValueVariantPost]</a></span> Array of option and option values IDs that make up this variant. Will be empty if the variant is the product&#39;s base variant. </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="VariantProductPost">VariantProductPost - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a POST to create variant with a product.
</div>
    <div class="field-items">
      <div class="param">cost_price (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> The cost price of the variant. format: double</div>
<div class="param">price (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> This variant&#39;s base price on the storefront. If this value is null, the product&#39;s default price (set in the Product resource&#39;s &#x60;price&#x60; field) will be used as the base price. format: double</div>
<div class="param">weight (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> This variant&#39;s base weight on the storefront. If this value is null, the product&#39;s default weight (set in the Product resource&#39;s weight field) will be used as the base weight. format: double</div>
<div class="param">purchasing_disabled (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> If &#x60;true&#x60;, this variant will not be purchasable on the storefront. </div>
<div class="param">purchasing_disabled_message (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> If &#x60;purchasing_disabled&#x60; is &#x60;true&#x60;, this message should show on the storefront when the variant is selected. </div>
<div class="param">image_url (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The image that will be displayed when this variant is selected on the storefront. When updating a SKU image, send the publicly accessible URL. Supported image formats are JPEG, PNG, and GIF. Generic product images not specific to the variant should be stored on the product. </div>
<div class="param">upc (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The UPC code used in feeds for shopping comparison sites and external channel integrations. </div>
<div class="param">inventory_level (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> Inventory level for the variant, which is used when the product&#39;s inventory_tracking is set to variant </div>
<div class="param">inventory_warning_level (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> When the variant hits this inventory level, it is considered low stock. </div>
<div class="param">bin_picking_number (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Identifies where in a warehouse the variant is located. </div>
<div class="param">product_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span>  </div>
<div class="param">sku (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span>  </div>
<div class="param">option_values (optional)</div><div class="param-desc"><span class="param-type"><a href="#OptionValueProductPost">array[OptionValueProductPost]</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="VariantProductPut">VariantProductPut - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a PUT to update variant with a product.
</div>
    <div class="field-items">
      <div class="param">cost_price (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> The cost price of the variant. format: double</div>
<div class="param">price (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> This variant&#39;s base price on the storefront. If this value is null, the product&#39;s default price (set in the Product resource&#39;s &#x60;price&#x60; field) will be used as the base price. format: double</div>
<div class="param">weight (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> This variant&#39;s base weight on the storefront. If this value is null, the product&#39;s default weight (set in the Product resource&#39;s weight field) will be used as the base weight. format: double</div>
<div class="param">purchasing_disabled (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> If &#x60;true&#x60;, this variant will not be purchasable on the storefront. </div>
<div class="param">purchasing_disabled_message (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> If &#x60;purchasing_disabled&#x60; is &#x60;true&#x60;, this message should show on the storefront when the variant is selected. </div>
<div class="param">image_url (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The image that will be displayed when this variant is selected on the storefront. When updating a SKU image, send the publicly accessible URL. Supported image formats are JPEG, PNG, and GIF. Generic product images not specific to the variant should be stored on the product. </div>
<div class="param">upc (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The UPC code used in feeds for shopping comparison sites and external channel integrations. </div>
<div class="param">inventory_level (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> Inventory level for the variant, which is used when the product&#39;s inventory_tracking is set to variant </div>
<div class="param">inventory_warning_level (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> When the variant hits this inventory level, it is considered low stock. </div>
<div class="param">bin_picking_number (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Identifies where in a warehouse the variant is located. </div>
<div class="param">product_id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span>  </div>
<div class="param">sku (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="VariantPut">VariantPut - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>The model for a PUT to update variant on a product.
</div>
    <div class="field-items">
      <div class="param">cost_price (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> The cost price of the variant. format: double</div>
<div class="param">price (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> This variant&#39;s base price on the storefront. If this value is null, the product&#39;s default price (set in the Product resource&#39;s &#x60;price&#x60; field) will be used as the base price. format: double</div>
<div class="param">weight (optional)</div><div class="param-desc"><span class="param-type"><a href="#double">Double</a></span> This variant&#39;s base weight on the storefront. If this value is null, the product&#39;s default weight (set in the Product resource&#39;s weight field) will be used as the base weight. format: double</div>
<div class="param">purchasing_disabled (optional)</div><div class="param-desc"><span class="param-type"><a href="#boolean">Boolean</a></span> If &#x60;true&#x60;, this variant will not be purchasable on the storefront. </div>
<div class="param">purchasing_disabled_message (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> If &#x60;purchasing_disabled&#x60; is &#x60;true&#x60;, this message should show on the storefront when the variant is selected. </div>
<div class="param">image_url (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The image that will be displayed when this variant is selected on the storefront. When updating a SKU image, send the publicly accessible URL. Supported image formats are JPEG, PNG, and GIF. Generic product images not specific to the variant should be stored on the product. </div>
<div class="param">upc (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> The UPC code used in feeds for shopping comparison sites and external channel integrations. </div>
<div class="param">inventory_level (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> Inventory level for the variant, which is used when the product&#39;s inventory_tracking is set to variant </div>
<div class="param">inventory_warning_level (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span> When the variant hits this inventory level, it is considered low stock. </div>
<div class="param">bin_picking_number (optional)</div><div class="param-desc"><span class="param-type"><a href="#string">String</a></span> Identifies where in a warehouse the variant is located. </div>
<div class="param">id (optional)</div><div class="param-desc"><span class="param-type"><a href="#integer">Integer</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  <div class="model">
    <h3 class="field-label"><a name="VariantResponse">VariantResponse - </a> <a class="up" href="#__Models">Up</a></h3>
    <div class='model-description'>Successful response.
</div>
    <div class="field-items">
      <div class="param">data (optional)</div><div class="param-desc"><span class="param-type"><a href="#Variant">Variant</a></span>  </div>
<div class="param">meta (optional)</div><div class="param-desc"><span class="param-type"><a href="#Meta">Meta</a></span>  </div>
    </div>  <!-- field-items -->
  </div>
  </body>
</html>