# v3 API Documentation

_Welcome! Please note that this API is in our partner-release stage, which means that in the short term, we'll be iterating on feedback from partners. Our goal is to make sure that all concerns are addressed, and that we reach our goal of creating the most powerful, easiest-to-use catalog API in ecommerce. Because of this iterative approach, please expect small changes and many additions to occur._

**Have suggestions, feedback or questions? Submit them as an issue here:**  
https://github.com/bigcommerce/api/issues

**Want to see what we have in development, and help direct our roadmap? View our public API roadmap here:**  
https://trello.com/b/1Od4oCsl/bigcommerce-api-roadmap

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

# <a name="api_ref"></a> v3 API Reference

Please view the documentation generated from the Swagger file in YAML format [here](http://editor.swagger.io/#/?import=https://raw.githubusercontent.com/bigcommerce/api/master/swagger/v3-api.yaml).

<html>
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
    Catalog
   </a>
  </h4>
  <ul>
   <li>
    <a href="#catalogSummaryGet">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/summary
     </code>
    </a>
   </li>
   <li>
    <a href="#createBrand">
     <code>
      <span class="http-method">
       post
      </span>
      /catalog/brands
     </code>
    </a>
   </li>
   <li>
    <a href="#createBrandImage">
     <code>
      <span class="http-method">
       post
      </span>
      /catalog/brands/{brand_id}/image
     </code>
    </a>
   </li>
   <li>
    <a href="#createBrandMetafield">
     <code>
      <span class="http-method">
       post
      </span>
      /catalog/brands/{brand_id}/metafields
     </code>
    </a>
   </li>
   <li>
    <a href="#createCategory">
     <code>
      <span class="http-method">
       post
      </span>
      /catalog/categories
     </code>
    </a>
   </li>
   <li>
    <a href="#createCategoryImage">
     <code>
      <span class="http-method">
       post
      </span>
      /catalog/categories/{category_id}/image
     </code>
    </a>
   </li>
   <li>
    <a href="#createCategoryMetafield">
     <code>
      <span class="http-method">
       post
      </span>
      /catalog/categories/{category_id}/metafields
     </code>
    </a>
   </li>
   <li>
    <a href="#createComplexRule">
     <code>
      <span class="http-method">
       post
      </span>
      /catalog/products/{product_id}/complex-rules
     </code>
    </a>
   </li>
   <li>
    <a href="#createCustomField">
     <code>
      <span class="http-method">
       post
      </span>
      /catalog/products/{product_id}/custom-fields
     </code>
    </a>
   </li>
   <li>
    <a href="#createModifier">
     <code>
      <span class="http-method">
       post
      </span>
      /catalog/products/{product_id}/modifiers
     </code>
    </a>
   </li>
   <li>
    <a href="#createModifierImage">
     <code>
      <span class="http-method">
       post
      </span>
      /catalog/products/{product_id}/modifiers/{modifier_id}/values/{value_id}/image
     </code>
    </a>
   </li>
   <li>
    <a href="#createModifierValue">
     <code>
      <span class="http-method">
       post
      </span>
      /catalog/products/{product_id}/modifiers/{modifier_id}/values
     </code>
    </a>
   </li>
   <li>
    <a href="#createOption">
     <code>
      <span class="http-method">
       post
      </span>
      /catalog/products/{product_id}/options
     </code>
    </a>
   </li>
   <li>
    <a href="#createOptionValue">
     <code>
      <span class="http-method">
       post
      </span>
      /catalog/products/{product_id}/options/{option_id}/values
     </code>
    </a>
   </li>
   <li>
    <a href="#createProduct">
     <code>
      <span class="http-method">
       post
      </span>
      /catalog/products
     </code>
    </a>
   </li>
   <li>
    <a href="#createProductImage">
     <code>
      <span class="http-method">
       post
      </span>
      /catalog/products/{product_id}/images
     </code>
    </a>
   </li>
   <li>
    <a href="#createProductMetafield">
     <code>
      <span class="http-method">
       post
      </span>
      /catalog/products/{product_id}/metafields
     </code>
    </a>
   </li>
   <li>
    <a href="#createProductVideo">
     <code>
      <span class="http-method">
       post
      </span>
      /catalog/products/{product_id}/videos
     </code>
    </a>
   </li>
   <li>
    <a href="#createVariant">
     <code>
      <span class="http-method">
       post
      </span>
      /catalog/products/{product_id}/variants
     </code>
    </a>
   </li>
   <li>
    <a href="#createVariantImage">
     <code>
      <span class="http-method">
       post
      </span>
      /catalog/products/{product_id}/variants/{variant_id}/image
     </code>
    </a>
   </li>
   <li>
    <a href="#createVariantMetafield">
     <code>
      <span class="http-method">
       post
      </span>
      /catalog/products/{product_id}/variants/{variant_id}/metafields
     </code>
    </a>
   </li>
   <li>
    <a href="#deleteBrandById">
     <code>
      <span class="http-method">
       delete
      </span>
      /catalog/brands/{brand_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#deleteBrandImage">
     <code>
      <span class="http-method">
       delete
      </span>
      /catalog/brands/{brand_id}/image
     </code>
    </a>
   </li>
   <li>
    <a href="#deleteBrandMetafieldById">
     <code>
      <span class="http-method">
       delete
      </span>
      /catalog/brands/{brand_id}/metafields/{metafield_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#deleteBrands">
     <code>
      <span class="http-method">
       delete
      </span>
      /catalog/brands
     </code>
    </a>
   </li>
   <li>
    <a href="#deleteCategories">
     <code>
      <span class="http-method">
       delete
      </span>
      /catalog/categories
     </code>
    </a>
   </li>
   <li>
    <a href="#deleteCategoryById">
     <code>
      <span class="http-method">
       delete
      </span>
      /catalog/categories/{category_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#deleteCategoryImage">
     <code>
      <span class="http-method">
       delete
      </span>
      /catalog/categories/{category_id}/image
     </code>
    </a>
   </li>
   <li>
    <a href="#deleteCategoryMetafieldById">
     <code>
      <span class="http-method">
       delete
      </span>
      /catalog/categories/{category_id}/metafields/{metafield_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#deleteComplexRuleById">
     <code>
      <span class="http-method">
       delete
      </span>
      /catalog/products/{product_id}/complex-rules/{complex_rule_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#deleteCustomFieldById">
     <code>
      <span class="http-method">
       delete
      </span>
      /catalog/products/{product_id}/custom-fields/{custom_field_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#deleteModifierById">
     <code>
      <span class="http-method">
       delete
      </span>
      /catalog/products/{product_id}/modifiers/{modifier_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#deleteModifierImage">
     <code>
      <span class="http-method">
       delete
      </span>
      /catalog/products/{product_id}/modifiers/{modifier_id}/values/{value_id}/image
     </code>
    </a>
   </li>
   <li>
    <a href="#deleteModifierValueById">
     <code>
      <span class="http-method">
       delete
      </span>
      /catalog/products/{product_id}/modifiers/{modifier_id}/values/{value_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#deleteOptionById">
     <code>
      <span class="http-method">
       delete
      </span>
      /catalog/products/{product_id}/options/{option_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#deleteOptionValueById">
     <code>
      <span class="http-method">
       delete
      </span>
      /catalog/products/{product_id}/options/{option_id}/values/{value_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#deleteProductById">
     <code>
      <span class="http-method">
       delete
      </span>
      /catalog/products/{product_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#deleteProductImage">
     <code>
      <span class="http-method">
       delete
      </span>
      /catalog/products/{product_id}/images/{image_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#deleteProductMetafieldById">
     <code>
      <span class="http-method">
       delete
      </span>
      /catalog/products/{product_id}/metafields/{metafield_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#deleteProductVideo">
     <code>
      <span class="http-method">
       delete
      </span>
      /catalog/products/{product_id}/videos/{video_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#deleteProducts">
     <code>
      <span class="http-method">
       delete
      </span>
      /catalog/products
     </code>
    </a>
   </li>
   <li>
    <a href="#deleteVariantById">
     <code>
      <span class="http-method">
       delete
      </span>
      /catalog/products/{product_id}/variants/{variant_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#deleteVariantMetafieldById">
     <code>
      <span class="http-method">
       delete
      </span>
      /catalog/products/{product_id}/variants/{variant_id}/metafields/{metafield_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#getBrandById">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/brands/{brand_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#getBrandMetafieldByBrandId">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/brands/{brand_id}/metafields/{metafield_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#getBrandMetafieldsByBrandId">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/brands/{brand_id}/metafields
     </code>
    </a>
   </li>
   <li>
    <a href="#getBrands">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/brands
     </code>
    </a>
   </li>
   <li>
    <a href="#getCategories">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/categories
     </code>
    </a>
   </li>
   <li>
    <a href="#getCategoryById">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/categories/{category_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#getCategoryMetafieldByCategoryId">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/categories/{category_id}/metafields/{metafield_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#getCategoryMetafieldsByCategoryId">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/categories/{category_id}/metafields
     </code>
    </a>
   </li>
   <li>
    <a href="#getCategoryTree">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/categories/tree
     </code>
    </a>
   </li>
   <li>
    <a href="#getComplexRuleById">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/products/{product_id}/complex-rules/{complex_rule_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#getComplexRules">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/products/{product_id}/complex-rules
     </code>
    </a>
   </li>
   <li>
    <a href="#getCustomFieldById">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/products/{product_id}/custom-fields/{custom_field_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#getCustomFields">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/products/{product_id}/custom-fields
     </code>
    </a>
   </li>
   <li>
    <a href="#getModifierById">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/products/{product_id}/modifiers/{modifier_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#getModifierValueById">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/products/{product_id}/modifiers/{modifier_id}/values/{value_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#getModifierValues">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/products/{product_id}/modifiers/{modifier_id}/values
     </code>
    </a>
   </li>
   <li>
    <a href="#getModifiers">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/products/{product_id}/modifiers
     </code>
    </a>
   </li>
   <li>
    <a href="#getOptionById">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/products/{product_id}/options/{option_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#getOptionValueById">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/products/{product_id}/options/{option_id}/values/{value_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#getOptionValues">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/products/{product_id}/options/{option_id}/values
     </code>
    </a>
   </li>
   <li>
    <a href="#getOptions">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/products/{product_id}/options
     </code>
    </a>
   </li>
   <li>
    <a href="#getProductById">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/products/{product_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#getProductImageById">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/products/{product_id}/images/{image_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#getProductImages">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/products/{product_id}/images
     </code>
    </a>
   </li>
   <li>
    <a href="#getProductMetafieldByProductId">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/products/{product_id}/metafields/{metafield_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#getProductMetafieldsByProductId">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/products/{product_id}/metafields
     </code>
    </a>
   </li>
   <li>
    <a href="#getProductVideoById">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/products/{product_id}/videos/{video_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#getProductVideos">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/products/{product_id}/videos
     </code>
    </a>
   </li>
   <li>
    <a href="#getProducts">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/products
     </code>
    </a>
   </li>
   <li>
    <a href="#getVariantById">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/products/{product_id}/variants/{variant_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#getVariantMetafieldByProductIdAndVariantId">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/products/{product_id}/variants/{variant_id}/metafields/{metafield_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#getVariantMetafieldsByProductIdAndVariantId">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/products/{product_id}/variants/{variant_id}/metafields
     </code>
    </a>
   </li>
   <li>
    <a href="#getVariants">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/variants
     </code>
    </a>
   </li>
   <li>
    <a href="#getVariantsByProductId">
     <code>
      <span class="http-method">
       get
      </span>
      /catalog/products/{product_id}/variants
     </code>
    </a>
   </li>
   <li>
    <a href="#updateBrand">
     <code>
      <span class="http-method">
       put
      </span>
      /catalog/brands/{brand_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#updateBrandMetafield">
     <code>
      <span class="http-method">
       put
      </span>
      /catalog/brands/{brand_id}/metafields/{metafield_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#updateCategory">
     <code>
      <span class="http-method">
       put
      </span>
      /catalog/categories/{category_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#updateCategoryMetafield">
     <code>
      <span class="http-method">
       put
      </span>
      /catalog/categories/{category_id}/metafields/{metafield_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#updateComplexRule">
     <code>
      <span class="http-method">
       put
      </span>
      /catalog/products/{product_id}/complex-rules/{complex_rule_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#updateCustomField">
     <code>
      <span class="http-method">
       put
      </span>
      /catalog/products/{product_id}/custom-fields/{custom_field_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#updateModifier">
     <code>
      <span class="http-method">
       put
      </span>
      /catalog/products/{product_id}/modifiers/{modifier_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#updateModifierValue">
     <code>
      <span class="http-method">
       put
      </span>
      /catalog/products/{product_id}/modifiers/{modifier_id}/values/{value_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#updateOption">
     <code>
      <span class="http-method">
       put
      </span>
      /catalog/products/{product_id}/options/{option_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#updateOptionValue">
     <code>
      <span class="http-method">
       put
      </span>
      /catalog/products/{product_id}/options/{option_id}/values/{value_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#updateProduct">
     <code>
      <span class="http-method">
       put
      </span>
      /catalog/products/{product_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#updateProductImage">
     <code>
      <span class="http-method">
       put
      </span>
      /catalog/products/{product_id}/images/{image_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#updateProductMetafield">
     <code>
      <span class="http-method">
       put
      </span>
      /catalog/products/{product_id}/metafields/{metafield_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#updateProductVideo">
     <code>
      <span class="http-method">
       put
      </span>
      /catalog/products/{product_id}/videos/{video_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#updateVariant">
     <code>
      <span class="http-method">
       put
      </span>
      /catalog/products/{product_id}/variants/{variant_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#updateVariantMetafield">
     <code>
      <span class="http-method">
       put
      </span>
      /catalog/products/{product_id}/variants/{variant_id}/metafields/{metafield_id}
     </code>
    </a>
   </li>
  </ul>
  <h4>
   <a href="#Customers">
    Customers
   </a>
  </h4>
  <ul>
   <li>
    <a href="#createSubscriber">
     <code>
      <span class="http-method">
       post
      </span>
      /customers/subscribers
     </code>
    </a>
   </li>
   <li>
    <a href="#deleteSubscriberById">
     <code>
      <span class="http-method">
       delete
      </span>
      /customers/subscribers/{subscriber_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#deleteSubscribers">
     <code>
      <span class="http-method">
       delete
      </span>
      /customers/subscribers
     </code>
    </a>
   </li>
   <li>
    <a href="#getSubscriberById">
     <code>
      <span class="http-method">
       get
      </span>
      /customers/subscribers/{subscriber_id}
     </code>
    </a>
   </li>
   <li>
    <a href="#getSubscribers">
     <code>
      <span class="http-method">
       get
      </span>
      /customers/subscribers
     </code>
    </a>
   </li>
   <li>
    <a href="#updateSubscriber">
     <code>
      <span class="http-method">
       put
      </span>
      /customers/subscribers/{subscriber_id}
     </code>
    </a>
   </li>
  </ul>
  <h1>
   <a name="Catalog">
    Catalog
   </a>
  </h1>
  <div class="method">
   <a name="catalogSummaryGet">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/summary
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     catalogSummaryGet
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Returns a lightweight inventory summary from the BigCommerce Catalog.
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#CatalogSummaryResponse">
     CatalogSummaryResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : {
    "inventory_value" : 1.3579000000000001069366817318950779736042022705078125,
    "inventory_count" : 123,
    "primary_category_id" : 123,
    "primary_category_name" : "aeiou"
  },
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   An array of catalog summary and metadata.
   <a href="#CatalogSummaryResponse">
    CatalogSummaryResponse
   </a>
  </div>
  <div class="method">
   <a name="createBrand">
   </a>
   <div class="method-path">
    <h2 class="post">
     <span class="http-method">
      POST
     </span>
     /catalog/brands
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     createBrand
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Creates a `Brand` object.
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     Brand
     <a href="#BrandPost">
      BrandPost
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — A `Brand` object.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#BrandResponse">
     BrandResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A `Brand` object.
   <a href="#BrandResponse">
    BrandResponse
   </a>
   <h4 class="field-label">
    409
   </h4>
   Brand was in conflict with another brand. This is the result of duplicate unique fields such as name.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
   <h4 class="field-label">
    422
   </h4>
   Brand was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
  </div>
  <div class="method">
   <a name="createBrandImage">
   </a>
   <div class="method-path">
    <h2 class="post">
     <span class="http-method">
      POST
     </span>
     /catalog/brands/{brand_id}/image
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     createBrandImage
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Creates an image on a `Brand`. Publicly accessible URLs and files (form post) are valid parameters.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     brand_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Brand` to which the resource belongs.
 format: int
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      multipart/form-data
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Form parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     image_file (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Form Parameter
     </span>
     — An image file. Supported MIME types include GIF, JPEG, and PNG.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#ImageResponse">
     ImageResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : {
    "image_url" : "aeiou"
  },
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A ResourceImage and metadata.
   <a href="#ImageResponse">
    ImageResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
   <h4 class="field-label">
    422
   </h4>
   Image was not valid. This is the result of a missing `image_file` field or an incorrect file type. See the response for more details.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
  </div>
  <div class="method">
   <a name="createBrandMetafield">
   </a>
   <div class="method-path">
    <h2 class="post">
     <span class="http-method">
      POST
     </span>
     /catalog/brands/{brand_id}/metafields
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     createBrandMetafield
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Creates a product `Metafield`.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     brand_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Brand` to which the resource belongs.
 format: int
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     Metafield
     <a href="#MetafieldPost">
      MetafieldPost
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — A `Metafield` object.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#MetafieldResponse">
     MetafieldResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A `Metafield` object.
   <a href="#MetafieldResponse">
    MetafieldResponse
   </a>
   <h4 class="field-label">
    409
   </h4>
   The `Metafield` was in conflict with another `Metafield`. This can be the result of duplicate unique key combination of the app's client id, namespace, key, resource_type, and resource_id.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
   <h4 class="field-label">
    422
   </h4>
   The `Metafield` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
  </div>
  <div class="method">
   <a name="createCategory">
   </a>
   <div class="method-path">
    <h2 class="post">
     <span class="http-method">
      POST
     </span>
     /catalog/categories
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     createCategory
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Creates a `Category` in the BigCommerce Catalog.
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     category
     <a href="#CategoryPost">
      CategoryPost
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — A BigCommerce `Category` object.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#CategoryResponse">
     CategoryResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A category object.
   <a href="#CategoryResponse">
    CategoryResponse
   </a>
   <h4 class="field-label">
    409
   </h4>
   The `Category` was in conflict with another category. This is the result of duplicate unique values, such as `name` or `custom_url`.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
   <h4 class="field-label">
    422
   </h4>
   The `Category` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
  </div>
  <div class="method">
   <a name="createCategoryImage">
   </a>
   <div class="method-path">
    <h2 class="post">
     <span class="http-method">
      POST
     </span>
     /catalog/categories/{category_id}/image
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     createCategoryImage
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Creates an image on a category. Publicly accessible URLs and files (form post) are valid parameters.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     category_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Category` to which the resource belongs.
 format: int
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      multipart/form-data
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Form parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     image_file (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Form Parameter
     </span>
     — An image file. Supported MIME types include GIF, JPEG, and PNG.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#ImageResponse">
     ImageResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : {
    "image_url" : "aeiou"
  },
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A ResourceImage and metadata.
   <a href="#ImageResponse">
    ImageResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
   <h4 class="field-label">
    422
   </h4>
   Image was not valid. This is the result of a missing `image_file` field or an incorrect file type. See the response for more details.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
  </div>
  <div class="method">
   <a name="createCategoryMetafield">
   </a>
   <div class="method-path">
    <h2 class="post">
     <span class="http-method">
      POST
     </span>
     /catalog/categories/{category_id}/metafields
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     createCategoryMetafield
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Creates a product `Metafield`.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     category_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Category` to which the resource belongs.
 format: int
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     Metafield
     <a href="#MetafieldPost">
      MetafieldPost
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — A `Metafield` object.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#MetafieldResponse">
     MetafieldResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A `Metafield` object.
   <a href="#MetafieldResponse">
    MetafieldResponse
   </a>
   <h4 class="field-label">
    409
   </h4>
   The `Metafield` was in conflict with another `Metafield`. This can be the result of duplicate unique key combinations of the app's client id, namespace, key, resource_type, and resource_id.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
   <h4 class="field-label">
    422
   </h4>
   The `Metafield` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
  </div>
  <div class="method">
   <a name="createComplexRule">
   </a>
   <div class="method-path">
    <h2 class="post">
     <span class="http-method">
      POST
     </span>
     /catalog/products/{product_id}/complex-rules
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     createComplexRule
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Creates a `ComplexRule`.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     ComplexRule
     <a href="#ComplexRulePost">
      ComplexRulePost
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — `ComplexRule` object.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#ComplexRuleResponse">
     ComplexRuleResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A `ComplexRule` object.
   <a href="#ComplexRuleResponse">
    ComplexRuleResponse
   </a>
   <h4 class="field-label">
    409
   </h4>
   The `ComplexRule` was in conflict with another `ComplexRule`. This is the result of duplicate conditions.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
   <h4 class="field-label">
    422
   </h4>
   The `ComplexRule` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
  </div>
  <div class="method">
   <a name="createCustomField">
   </a>
   <div class="method-path">
    <h2 class="post">
     <span class="http-method">
      POST
     </span>
     /catalog/products/{product_id}/custom-fields
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     createCustomField
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Creates a `CustomField`.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     CustomField
     <a href="#CustomFieldPost">
      CustomFieldPost
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — `CustomField` object.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#CustomFieldResponse">
     CustomFieldResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A `CustomField` object.
   <a href="#CustomFieldResponse">
    CustomFieldResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The parent resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
   <h4 class="field-label">
    422
   </h4>
   The `CustomField` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
  </div>
  <div class="method">
   <a name="createModifier">
   </a>
   <div class="method-path">
    <h2 class="post">
     <span class="http-method">
      POST
     </span>
     /catalog/products/{product_id}/modifiers
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     createModifier
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Creates a `Modifier`.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     Modifier
     <a href="#ModifierPost">
      ModifierPost
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — A `Modifier` object.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#ModifierResponse">
     ModifierResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A `Modifier` object.
   <a href="#ModifierResponse">
    ModifierResponse
   </a>
   <h4 class="field-label">
    409
   </h4>
   The `Modifier` was in conflict with another option. This is the result of duplicate unique fields, such as `name`.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
   <h4 class="field-label">
    422
   </h4>
   The `Modifier` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
  </div>
  <div class="method">
   <a name="createModifierImage">
   </a>
   <div class="method-path">
    <h2 class="post">
     <span class="http-method">
      POST
     </span>
     /catalog/products/{product_id}/modifiers/{modifier_id}/values/{value_id}/image
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     createModifierImage
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Adds an image to a modifier value; the image will show on the storefront when the value is selected.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     modifier_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Modifier`.
    </div>
    </p>
<div class="param">
     value_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Modifier`.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      multipart/form-data
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Form parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     image_file (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Form Parameter
     </span>
     — An image file. Supported MIME types include GIF, JPEG, and PNG.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#ImageResponse">
     ImageResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : {
    "image_url" : "aeiou"
  },
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A ResourceImage and metadata.
   <a href="#ImageResponse">
    ImageResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
   <h4 class="field-label">
    422
   </h4>
   Modifier image was not valid. This is the result of missing `image_file` fields, orof a non-URL value for the `image_file` field. See the response for more details.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
  </div>
  <div class="method">
   <a name="createModifierValue">
   </a>
   <div class="method-path">
    <h2 class="post">
     <span class="http-method">
      POST
     </span>
     /catalog/products/{product_id}/modifiers/{modifier_id}/values
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     createModifierValue
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Creates a `ModifierValue`.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     modifier_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Modifier`.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     ModifierValue
     <a href="#ModifierValuePost">
      ModifierValuePost
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — A `ModifierValue` object.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#ModifierValueResponse">
     ModifierValueResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A `ModifierValue` object.
   <a href="#ModifierValueResponse">
    ModifierValueResponse
   </a>
   <h4 class="field-label">
    422
   </h4>
   The `ModifierValue` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
  </div>
  <div class="method">
   <a name="createOption">
   </a>
   <div class="method-path">
    <h2 class="post">
     <span class="http-method">
      POST
     </span>
     /catalog/products/{product_id}/options
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     createOption
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Creates an `Option`.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     Option
     <a href="#OptionPost">
      OptionPost
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — An `Option` object.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#OptionResponse">
     OptionResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   An `Option` object.
   <a href="#OptionResponse">
    OptionResponse
   </a>
   <h4 class="field-label">
    409
   </h4>
   Option was in conflict with another option. This is the result of duplicate unique fields, such as `name`.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
   <h4 class="field-label">
    422
   </h4>
   Option was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
  </div>
  <div class="method">
   <a name="createOptionValue">
   </a>
   <div class="method-path">
    <h2 class="post">
     <span class="http-method">
      POST
     </span>
     /catalog/products/{product_id}/options/{option_id}/values
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     createOptionValue
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Creates a `OptionValue`.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     option_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Option`.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     OptionValue
     <a href="#OptionValuePost">
      OptionValuePost
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — A `OptionValue` object.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#OptionValueResponse">
     OptionValueResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A `OptionValue` object.
   <a href="#OptionValueResponse">
    OptionValueResponse
   </a>
   <h4 class="field-label">
    422
   </h4>
   The `OptionValue` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
  </div>
  <div class="method">
   <a name="createProduct">
   </a>
   <div class="method-path">
    <h2 class="post">
     <span class="http-method">
      POST
     </span>
     /catalog/products
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     createProduct
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Creates a `Product` in the BigCommerce Catalog.
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product
     <a href="#ProductPost">
      ProductPost
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — A BigCommerce `Product` object.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#ProductResponse">
     ProductResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A product.
   <a href="#ProductResponse">
    ProductResponse
   </a>
   <h4 class="field-label">
    409
   </h4>
   `Product` was in conflict with another product. This is the result of duplicate unique values, such as name or SKU; a missing or invalid category id, brand id, or tax_class id; or a conflicting `bulk_pricing_rule`.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
   <h4 class="field-label">
    422
   </h4>
   `Product` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
  </div>
  <div class="method">
   <a name="createProductImage">
   </a>
   <div class="method-path">
    <h2 class="post">
     <span class="http-method">
      POST
     </span>
     /catalog/products/{product_id}/images
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     createProductImage
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Creates an image on a product. Publically accessible URLs and files (form post) are valid parameters.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     productImage
     <a href="#ProductImagePost">
      ProductImagePost
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — A BigCommerce `ProductImage` object.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#ProductImageResponse">
     ProductImageResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A product image.
   <a href="#ProductImageResponse">
    ProductImageResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The product ID does not exist.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="createProductMetafield">
   </a>
   <div class="method-path">
    <h2 class="post">
     <span class="http-method">
      POST
     </span>
     /catalog/products/{product_id}/metafields
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     createProductMetafield
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Creates a product `Metafield`.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     Metafield
     <a href="#MetafieldPost">
      MetafieldPost
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — A `Metafield` object.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#MetafieldResponse">
     MetafieldResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A `Metafield` object.
   <a href="#MetafieldResponse">
    MetafieldResponse
   </a>
   <h4 class="field-label">
    409
   </h4>
   The `Metafield` was in conflict with another `Metafield`. This can be the result of duplicate unique key combinations of the app's client id, namespace, key, resource_type, and resource_id.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
   <h4 class="field-label">
    422
   </h4>
   The `Metafield` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
  </div>
  <div class="method">
   <a name="createProductVideo">
   </a>
   <div class="method-path">
    <h2 class="post">
     <span class="http-method">
      POST
     </span>
     /catalog/products/{product_id}/videos
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     createProductVideo
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Creates a video on a product, using a video ID from YouTube.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     productVideo
     <a href="#ProductVideoPost">
      ProductVideoPost
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — A BigCommerce `ProductVideo` object.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#ProductVideoResponse">
     ProductVideoResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A product video.
   <a href="#ProductVideoResponse">
    ProductVideoResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="createVariant">
   </a>
   <div class="method-path">
    <h2 class="post">
     <span class="http-method">
      POST
     </span>
     /catalog/products/{product_id}/variants
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     createVariant
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Creates a `Variant` object.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     Variant
     <a href="#VariantPost">
      VariantPost
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — `Variant` object.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#VariantResponse">
     VariantResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A variant and metadata.
   <a href="#VariantResponse">
    VariantResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="createVariantImage">
   </a>
   <div class="method-path">
    <h2 class="post">
     <span class="http-method">
      POST
     </span>
     /catalog/products/{product_id}/variants/{variant_id}/image
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     createVariantImage
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     variant_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Variant` to which the resource belongs.
 format: int
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      multipart/form-data
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Form parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     image_file (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Form Parameter
     </span>
     — An image file. Supported MIME types include GIF, JPEG, and PNG.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#ImageResponse">
     ImageResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : {
    "image_url" : "aeiou"
  },
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A ResourceImage and metadata.
   <a href="#ImageResponse">
    ImageResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
   <h4 class="field-label">
    422
   </h4>
   Image was not valid. This is the result of a missing image_file field or an incorrect file type. See the response for more details.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
  </div>
  <div class="method">
   <a name="createVariantMetafield">
   </a>
   <div class="method-path">
    <h2 class="post">
     <span class="http-method">
      POST
     </span>
     /catalog/products/{product_id}/variants/{variant_id}/metafields
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     createVariantMetafield
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Creates a variant `Metafield`.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     variant_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Variant` to which the resource belongs.
 format: int
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     Metafield
     <a href="#MetafieldPost">
      MetafieldPost
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — A `Metafield` object.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#MetafieldResponse">
     MetafieldResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A `Metafield` object.
   <a href="#MetafieldResponse">
    MetafieldResponse
   </a>
   <h4 class="field-label">
    409
   </h4>
   The `Metafield` was in conflict with another `Metafield`. This can be the result of duplicate unique-key combinations of the app's client id, namespace, key, resource_type, and resource_id.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
   <h4 class="field-label">
    422
   </h4>
   The `Metafield` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
  </div>
  <div class="method">
   <a name="deleteBrandById">
   </a>
   <div class="method-path">
    <h2 class="delete">
     <span class="http-method">
      DELETE
     </span>
     /catalog/brands/{brand_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     deleteBrandById
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Deletes a `Brand` from the BigCommerce Catalog.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     brand_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Brand` to which the resource belongs.
 format: int
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    204
   </h4>
   An empty response.
   <a href="#">
   </a>
  </div>
  <div class="method">
   <a name="deleteBrandImage">
   </a>
   <div class="method-path">
    <h2 class="delete">
     <span class="http-method">
      DELETE
     </span>
     /catalog/brands/{brand_id}/image
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     deleteBrandImage
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Deletes a `Brand` image from the BigCommerce Catalog.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     brand_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Brand` to which the resource belongs.
 format: int
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    204
   </h4>
   Image cleared from the brand.
   <a href="#">
   </a>
  </div>
  <div class="method">
   <a name="deleteBrandMetafieldById">
   </a>
   <div class="method-path">
    <h2 class="delete">
     <span class="http-method">
      DELETE
     </span>
     /catalog/brands/{brand_id}/metafields/{metafield_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     deleteBrandMetafieldById
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Deletes a `Metafield`.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     metafield_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Metafield`.
 format: int
    </div>
    </p>
<div class="param">
     brand_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Brand` to which the resource belongs.
 format: int
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    204
   </h4>
   An empty response.
   <a href="#">
   </a>
  </div>
  <div class="method">
   <a name="deleteBrands">
   </a>
   <div class="method-path">
    <h2 class="delete">
     <span class="http-method">
      DELETE
     </span>
     /catalog/brands
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     deleteBrands
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Deletes one or more `Brand` objects from the BigCommerce Catalog.
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Query parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by name.
    </div>
    </p>
<div class="param">
     page_title (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by page_title.
    </div>
   </div>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    204
   </h4>
   An empty response.
   <a href="#">
   </a>
  </div>
  <div class="method">
   <a name="deleteCategories">
   </a>
   <div class="method-path">
    <h2 class="delete">
     <span class="http-method">
      DELETE
     </span>
     /catalog/categories
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     deleteCategories
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Deletes a product or products from the BigCommerce Catalog.
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Query parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by name.
    </div>
    </p>
<div class="param">
     parent_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by parent_id.
    </div>
    </p>
<div class="param">
     page_title (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by page_title.
    </div>
    </p>
<div class="param">
     keyword (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by keywords.
    </div>
    </p>
<div class="param">
     is_visible (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by is_visible.
    </div>
   </div>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    204
   </h4>
   An empty response.
   <a href="#">
   </a>
  </div>
  <div class="method">
   <a name="deleteCategoryById">
   </a>
   <div class="method-path">
    <h2 class="delete">
     <span class="http-method">
      DELETE
     </span>
     /catalog/categories/{category_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     deleteCategoryById
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Deletes one or more `Category` objects from the BigCommerce catalog.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     category_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Category` to which the resource belongs.
 format: int
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    204
   </h4>
   An empty response.
   <a href="#">
   </a>
  </div>
  <div class="method">
   <a name="deleteCategoryImage">
   </a>
   <div class="method-path">
    <h2 class="delete">
     <span class="http-method">
      DELETE
     </span>
     /catalog/categories/{category_id}/image
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     deleteCategoryImage
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Deletes a `Category` image from the BigCommerce Catalog.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     category_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Category` to which the resource belongs.
 format: int
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    204
   </h4>
   An empty response.
   <a href="#">
   </a>
  </div>
  <div class="method">
   <a name="deleteCategoryMetafieldById">
   </a>
   <div class="method-path">
    <h2 class="delete">
     <span class="http-method">
      DELETE
     </span>
     /catalog/categories/{category_id}/metafields/{metafield_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     deleteCategoryMetafieldById
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Deletes a `Metafield`.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     metafield_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Metafield`.
 format: int
    </div>
    </p>
<div class="param">
     category_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Category` to which the resource belongs.
 format: int
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    204
   </h4>
   An empty response.
   <a href="#">
   </a>
  </div>
  <div class="method">
   <a name="deleteComplexRuleById">
   </a>
   <div class="method-path">
    <h2 class="delete">
     <span class="http-method">
      DELETE
     </span>
     /catalog/products/{product_id}/complex-rules/{complex_rule_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     deleteComplexRuleById
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Deletes a Product's `ComplexRule`, based on the `product_id` and `complex_rule_id`.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     complex_rule_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `ComplexRule`.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    204
   </h4>
   An empty response.
   <a href="#">
   </a>
  </div>
  <div class="method">
   <a name="deleteCustomFieldById">
   </a>
   <div class="method-path">
    <h2 class="delete">
     <span class="http-method">
      DELETE
     </span>
     /catalog/products/{product_id}/custom-fields/{custom_field_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     deleteCustomFieldById
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Deletes a Product's `CustomField`, based on the `product_id` and `custom_field_id`.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     custom_field_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `CustomField`.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    204
   </h4>
   An empty response.
   <a href="#">
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="deleteModifierById">
   </a>
   <div class="method-path">
    <h2 class="delete">
     <span class="http-method">
      DELETE
     </span>
     /catalog/products/{product_id}/modifiers/{modifier_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     deleteModifierById
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Deletes a Product's `Modifier` based on the product_id and modifier_id.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     modifier_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Modifier`.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    204
   </h4>
   An empty response.
   <a href="#">
   </a>
  </div>
  <div class="method">
   <a name="deleteModifierImage">
   </a>
   <div class="method-path">
    <h2 class="delete">
     <span class="http-method">
      DELETE
     </span>
     /catalog/products/{product_id}/modifiers/{modifier_id}/values/{value_id}/image
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     deleteModifierImage
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Deletes the image applied to show when the modifier value is selected
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     modifier_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Modifier`.
    </div>
    </p>
<div class="param">
     value_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Modifier`.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    204
   </h4>
   Image cleared for the modifier value.
   <a href="#">
   </a>
  </div>
  <div class="method">
   <a name="deleteModifierValueById">
   </a>
   <div class="method-path">
    <h2 class="delete">
     <span class="http-method">
      DELETE
     </span>
     /catalog/products/{product_id}/modifiers/{modifier_id}/values/{value_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     deleteModifierValueById
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Deletes a Product's `ModifierValue` based on the product_id, modifier_id and value_id.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     modifier_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Modifier`.
    </div>
    </p>
<div class="param">
     value_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Modifier/Option Value`.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    204
   </h4>
   An empty response.
   <a href="#">
   </a>
  </div>
  <div class="method">
   <a name="deleteOptionById">
   </a>
   <div class="method-path">
    <h2 class="delete">
     <span class="http-method">
      DELETE
     </span>
     /catalog/products/{product_id}/options/{option_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     deleteOptionById
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Deletes a Product's `Option`, based on the product_id and option_id.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     option_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Option`.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    204
   </h4>
   An empty response.
   <a href="#">
   </a>
  </div>
  <div class="method">
   <a name="deleteOptionValueById">
   </a>
   <div class="method-path">
    <h2 class="delete">
     <span class="http-method">
      DELETE
     </span>
     /catalog/products/{product_id}/options/{option_id}/values/{value_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     deleteOptionValueById
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Deletes a Product's `OptionValue` based on the product_id, option_id and value_id.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     option_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Option`.
    </div>
    </p>
<div class="param">
     value_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Modifier/Option Value`.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    204
   </h4>
   An empty response.
   <a href="#">
   </a>
  </div>
  <div class="method">
   <a name="deleteProductById">
   </a>
   <div class="method-path">
    <h2 class="delete">
     <span class="http-method">
      DELETE
     </span>
     /catalog/products/{product_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     deleteProductById
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Deletes a `Product` object from the BigCommerce Catalog.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    204
   </h4>
   An empty response.
   <a href="#">
   </a>
  </div>
  <div class="method">
   <a name="deleteProductImage">
   </a>
   <div class="method-path">
    <h2 class="delete">
     <span class="http-method">
      DELETE
     </span>
     /catalog/products/{product_id}/images/{image_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     deleteProductImage
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Deletes a `ProductImage` in the BigCommerce Catalog.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     image_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Image` that is being operated on.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    204
   </h4>
   An empty response.
   <a href="#">
   </a>
  </div>
  <div class="method">
   <a name="deleteProductMetafieldById">
   </a>
   <div class="method-path">
    <h2 class="delete">
     <span class="http-method">
      DELETE
     </span>
     /catalog/products/{product_id}/metafields/{metafield_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     deleteProductMetafieldById
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Deletes a `Metafield`.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     metafield_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Metafield`.
 format: int
    </div>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    204
   </h4>
   An empty response.
   <a href="#">
   </a>
  </div>
  <div class="method">
   <a name="deleteProductVideo">
   </a>
   <div class="method-path">
    <h2 class="delete">
     <span class="http-method">
      DELETE
     </span>
     /catalog/products/{product_id}/videos/{video_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     deleteProductVideo
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Deletes a `ProductVideo` in the BigCommerce Catalog.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     video_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Video` being operated on.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    204
   </h4>
   An empty response.
   <a href="#">
   </a>
  </div>
  <div class="method">
   <a name="deleteProducts">
   </a>
   <div class="method-path">
    <h2 class="delete">
     <span class="http-method">
      DELETE
     </span>
     /catalog/products
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     deleteProducts
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Deletes one or more `Product` objects from the BigCommerce Catalog.
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Query parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by name.
    </div>
    </p>
<div class="param">
     sku (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by sku.
    </div>
    </p>
<div class="param">
     price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by price.
    </div>
    </p>
<div class="param">
     weight (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by weight.
    </div>
    </p>
<div class="param">
     condition (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by condition.
    </div>
    </p>
<div class="param">
     brand_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by brand_id.
    </div>
    </p>
<div class="param">
     date_modified (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by date_modified.
 format: date-time
    </div>
    </p>
<div class="param">
     date_last_imported (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by date_last_imported.
 format: date-time
    </div>
    </p>
<div class="param">
     is_visible (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by is_visible.
    </div>
    </p>
<div class="param">
     is_featured (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by is_featured.
    </div>
    </p>
<div class="param">
     inventory_level (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by inventory_level.
    </div>
    </p>
<div class="param">
     total_sold (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by total_sold.
    </div>
    </p>
<div class="param">
     type (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by type: `physical` or `digital`.
    </div>
    </p>
<div class="param">
     categories (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by categories.
    </div>
    </p>
<div class="param">
     keyword (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by keywords found in the name, description, sku, keywords, or brand name.
    </div>
   </div>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    204
   </h4>
   An empty response.
   <a href="#">
   </a>
  </div>
  <div class="method">
   <a name="deleteVariantById">
   </a>
   <div class="method-path">
    <h2 class="delete">
     <span class="http-method">
      DELETE
     </span>
     /catalog/products/{product_id}/variants/{variant_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     deleteVariantById
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Deletes a `Variant`.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     variant_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Variant` to which the resource belongs.
 format: int
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    204
   </h4>
   An empty response.
   <a href="#">
   </a>
  </div>
  <div class="method">
   <a name="deleteVariantMetafieldById">
   </a>
   <div class="method-path">
    <h2 class="delete">
     <span class="http-method">
      DELETE
     </span>
     /catalog/products/{product_id}/variants/{variant_id}/metafields/{metafield_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     deleteVariantMetafieldById
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Deletes a `Metafield`.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     metafield_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Metafield`.
 format: int
    </div>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     variant_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Variant` to which the resource belongs.
 format: int
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    204
   </h4>
   An empty response.
   <a href="#">
   </a>
  </div>
  <div class="method">
   <a name="getBrandById">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/brands/{brand_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getBrandById
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Gets a `Brand` object.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     brand_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Brand` to which the resource belongs.
 format: int
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#BrandResponse">
     BrandResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A `Brand` object.
   <a href="#BrandResponse">
    BrandResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="getBrandMetafieldByBrandId">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/brands/{brand_id}/metafields/{metafield_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getBrandMetafieldByBrandId
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Gets a `Metafield`, by `category_id`.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     metafield_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Metafield`.
 format: int
    </div>
    </p>
<div class="param">
     brand_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Brand` to which the resource belongs.
 format: int
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#MetafieldResponse">
     MetafieldResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A `Metafield` object.
   <a href="#MetafieldResponse">
    MetafieldResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="getBrandMetafieldsByBrandId">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/brands/{brand_id}/metafields
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getBrandMetafieldsByBrandId
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Gets a `Metafield` object list, by `category_id`.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     brand_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Brand` to which the resource belongs.
 format: int
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Query parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     page (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Control the page in a limited list of products.
    </div>
    </p>
<div class="param">
     limit (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Control the items per page.
    </div>
    </p>
<div class="param">
     key (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter based on a metafield's key.
    </div>
    </p>
<div class="param">
     namespace (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter based on a metafield's key.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#MetaFieldCollectionResponse">
     MetaFieldCollectionResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
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
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   An array of metafields and metadata.
   <a href="#MetaFieldCollectionResponse">
    MetaFieldCollectionResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="getBrands">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/brands
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getBrands
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Gets `Brand` objects.
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Query parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by name.
    </div>
    </p>
<div class="param">
     page_title (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by page_title.
    </div>
    </p>
<div class="param">
     page (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Control the page in a limited list of products.
    </div>
    </p>
<div class="param">
     limit (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Control the items per page.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#BrandCollectionResponse">
     BrandCollectionResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
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
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   An array of brand objects and metadata.
   <a href="#BrandCollectionResponse">
    BrandCollectionResponse
   </a>
  </div>
  <div class="method">
   <a name="getCategories">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/categories
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getCategories
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Returns a paginated categories collection from the BigCommerce Catalog.
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Query parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by name.
    </div>
    </p>
<div class="param">
     parent_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by parent_id.
    </div>
    </p>
<div class="param">
     page_title (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by page_title.
    </div>
    </p>
<div class="param">
     keyword (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by keywords.
    </div>
    </p>
<div class="param">
     is_visible (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by is_visible.
    </div>
    </p>
<div class="param">
     page (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Control the page in a limited list of products.
    </div>
    </p>
<div class="param">
     limit (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Control the items per page.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#CategoryCollectionResponse">
     CategoryCollectionResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
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
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   An array of category objects and metadata.
   <a href="#CategoryCollectionResponse">
    CategoryCollectionResponse
   </a>
  </div>
  <div class="method">
   <a name="getCategoryById">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/categories/{category_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getCategoryById
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Returns a `Category` from the BigCommerce Catalog.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     category_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Category` to which the resource belongs.
 format: int
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#CategoryResponse">
     CategoryResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A category object.
   <a href="#CategoryResponse">
    CategoryResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="getCategoryMetafieldByCategoryId">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/categories/{category_id}/metafields/{metafield_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getCategoryMetafieldByCategoryId
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Gets a `Metafield` by category_id.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     metafield_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Metafield`.
 format: int
    </div>
    </p>
<div class="param">
     category_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Category` to which the resource belongs.
 format: int
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#MetafieldResponse">
     MetafieldResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A `Metafield` object.
   <a href="#MetafieldResponse">
    MetafieldResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="getCategoryMetafieldsByCategoryId">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/categories/{category_id}/metafields
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getCategoryMetafieldsByCategoryId
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Gets a `Metafield` object list, by category_id.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     category_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Category` to which the resource belongs.
 format: int
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Query parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     page (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Control the page in a limited list of products.
    </div>
    </p>
<div class="param">
     limit (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Control the items per page.
    </div>
    </p>
<div class="param">
     key (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter based on a metafield's key.
    </div>
    </p>
<div class="param">
     namespace (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter based on a metafield's key.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#MetaFieldCollectionResponse">
     MetaFieldCollectionResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
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
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   An array of metafields and metadata.
   <a href="#MetaFieldCollectionResponse">
    MetaFieldCollectionResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="getCategoryTree">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/categories/tree
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getCategoryTree
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Returns the categories tree, a nested lineage of the categories with parent-&gt;child relationship. The `Category` objects returned are simplified versions of the category objects returned in the rest of this API.
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#CategoryTreeCollectionResponse">
     CategoryTreeCollectionResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
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
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A array of nested category tree objects and metadata.
   <a href="#CategoryTreeCollectionResponse">
    CategoryTreeCollectionResponse
   </a>
  </div>
  <div class="method">
   <a name="getComplexRuleById">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/products/{product_id}/complex-rules/{complex_rule_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getComplexRuleById
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Gets a `ComplexRule` by product_id.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     complex_rule_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `ComplexRule`.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#ComplexRuleResponse">
     ComplexRuleResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A `Modifier` object.
   <a href="#ComplexRuleResponse">
    ComplexRuleResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="getComplexRules">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/products/{product_id}/complex-rules
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getComplexRules
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Gets an array of `ComplexRule` objects.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#ComplexRuleCollectionResponse">
     ComplexRuleCollectionResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
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
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   An array of `ComplexRule` objects and metadata.
   <a href="#ComplexRuleCollectionResponse">
    ComplexRuleCollectionResponse
   </a>
  </div>
  <div class="method">
   <a name="getCustomFieldById">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/products/{product_id}/custom-fields/{custom_field_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getCustomFieldById
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Gets a `CustomField` by `product_id` and `custom_field_id`.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     custom_field_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `CustomField`.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#CustomFieldResponse">
     CustomFieldResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A `CustomField` object.
   <a href="#CustomFieldResponse">
    CustomFieldResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="getCustomFields">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/products/{product_id}/custom-fields
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getCustomFields
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Gets an array of `CustomField` objects.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#CustomFieldCollectionResponse">
     CustomFieldCollectionResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
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
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   An array of `CustomField` objects and metadata.
   <a href="#CustomFieldCollectionResponse">
    CustomFieldCollectionResponse
   </a>
  </div>
  <div class="method">
   <a name="getModifierById">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/products/{product_id}/modifiers/{modifier_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getModifierById
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Gets a `Modifier` by product_id and modifier_id.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     modifier_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Modifier`.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#ModifierResponse">
     ModifierResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A `Modifier` object.
   <a href="#ModifierResponse">
    ModifierResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="getModifierValueById">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/products/{product_id}/modifiers/{modifier_id}/values/{value_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getModifierValueById
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Gets a `ModifierValue` by product_id, modifier_id and value_id.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     modifier_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Modifier`.
    </div>
    </p>
<div class="param">
     value_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Modifier/Option Value`.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#ModifierValueResponse">
     ModifierValueResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A `ModifierValue` object.
   <a href="#ModifierValueResponse">
    ModifierValueResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="getModifierValues">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/products/{product_id}/modifiers/{modifier_id}/values
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getModifierValues
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Gets an array of `ModifierValue` objects.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     modifier_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Modifier`.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#ModifierValueCollectionResponse">
     ModifierValueCollectionResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
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
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   An array of modifier values and metadata.
   <a href="#ModifierValueCollectionResponse">
    ModifierValueCollectionResponse
   </a>
  </div>
  <div class="method">
   <a name="getModifiers">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/products/{product_id}/modifiers
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getModifiers
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Gets an array of `Modifier` objects.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#ModifierCollectionResponse">
     ModifierCollectionResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
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
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   An array of modifiers and metadata.
   <a href="#ModifierCollectionResponse">
    ModifierCollectionResponse
   </a>
  </div>
  <div class="method">
   <a name="getOptionById">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/products/{product_id}/options/{option_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getOptionById
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Gets `Option` object by product ID and option id.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     option_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Option`.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#OptionResponse">
     OptionResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   An `Option` object.
   <a href="#OptionResponse">
    OptionResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="getOptionValueById">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/products/{product_id}/options/{option_id}/values/{value_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getOptionValueById
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Gets a `OptionValue` by product_id, option_id and value_id.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     option_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Option`.
    </div>
    </p>
<div class="param">
     value_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Modifier/Option Value`.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#OptionValueResponse">
     OptionValueResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A `OptionValue` object.
   <a href="#OptionValueResponse">
    OptionValueResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="getOptionValues">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/products/{product_id}/options/{option_id}/values
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getOptionValues
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Gets an array of `OptionValue` objects.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     option_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Option`.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#OptionValueCollectionResponse">
     OptionValueCollectionResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
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
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   An array of option values and metadata.
   <a href="#OptionValueCollectionResponse">
    OptionValueCollectionResponse
   </a>
  </div>
  <div class="method">
   <a name="getOptions">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/products/{product_id}/options
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getOptions
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Gets an array of `Option` objects.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#OptionCollectionResponse">
     OptionCollectionResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
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
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   An array of options and metadata.
   <a href="#OptionCollectionResponse">
    OptionCollectionResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="getProductById">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/products/{product_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getProductById
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Returns a `Product` from the BigCommerce Catalog.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Query parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     include (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Include sub-resources on a product, with a comma-separated list. Valid expansions currently include `variants`, `images`, `custom_fields`, and `bulk_pricing_rules`.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#ProductResponse">
     ProductResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A product.
   <a href="#ProductResponse">
    ProductResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="getProductImageById">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/products/{product_id}/images/{image_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getProductImageById
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Gets image on a product.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     image_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Image` that is being operated on.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#ProductImageResponse">
     ProductImageResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   An array of product images and metadata.
   <a href="#ProductImageResponse">
    ProductImageResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="getProductImages">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/products/{product_id}/images
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getProductImages
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Gets all images on a product.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#ProductImageCollectionResponse">
     ProductImageCollectionResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
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
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   List of product images and metadata.
   <a href="#ProductImageCollectionResponse">
    ProductImageCollectionResponse
   </a>
   <h4 class="field-label">
    204
   </h4>
   There are not any images on this product.
   <a href="#">
   </a>
   <h4 class="field-label">
    404
   </h4>
   The product ID does not exist.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="getProductMetafieldByProductId">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/products/{product_id}/metafields/{metafield_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getProductMetafieldByProductId
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Gets a `Metafield`, by `product_id`.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     metafield_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Metafield`.
 format: int
    </div>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#MetafieldResponse">
     MetafieldResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A `Metafield` object.
   <a href="#MetafieldResponse">
    MetafieldResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="getProductMetafieldsByProductId">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/products/{product_id}/metafields
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getProductMetafieldsByProductId
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Gets a `Metafield` object list, by `product_id`.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Query parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     page (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Control the page in a limited list of products.
    </div>
    </p>
<div class="param">
     limit (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Control the items per page.
    </div>
    </p>
<div class="param">
     key (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter based on a metafield's key.
    </div>
    </p>
<div class="param">
     namespace (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter based on a metafield's key.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#MetaFieldCollectionResponse">
     MetaFieldCollectionResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
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
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   An array of metafields and metadata.
   <a href="#MetaFieldCollectionResponse">
    MetaFieldCollectionResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="getProductVideoById">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/products/{product_id}/videos/{video_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getProductVideoById
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Gets video on a product.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     video_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Video` being operated on.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#ProductVideoResponse">
     ProductVideoResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   An array of product videos and metadata.
   <a href="#ProductVideoResponse">
    ProductVideoResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="getProductVideos">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/products/{product_id}/videos
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getProductVideos
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Gets all videos on a product.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#ProductVideoCollectionResponse">
     ProductVideoCollectionResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
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
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   List of product videos and metadata.
   <a href="#ProductVideoCollectionResponse">
    ProductVideoCollectionResponse
   </a>
  </div>
  <div class="method">
   <a name="getProducts">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/products
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getProducts
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Returns a paginated collection of `Products` objects from the BigCommerce Catalog.
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Query parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by id.
    </div>
    </p>
<div class="param">
     name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by name.
    </div>
    </p>
<div class="param">
     sku (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by sku.
    </div>
    </p>
<div class="param">
     upc (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by upc.
    </div>
    </p>
<div class="param">
     price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by price.
    </div>
    </p>
<div class="param">
     weight (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by weight.
    </div>
    </p>
<div class="param">
     condition (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by condition.
    </div>
    </p>
<div class="param">
     brand_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by brand_id.
    </div>
    </p>
<div class="param">
     date_modified (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by date_modified.
 format: date-time
    </div>
    </p>
<div class="param">
     date_last_imported (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by date_last_imported.
 format: date-time
    </div>
    </p>
<div class="param">
     is_visible (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by is_visible.
    </div>
    </p>
<div class="param">
     is_featured (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by is_featured.
    </div>
    </p>
<div class="param">
     is_free_shipping (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by is_free_shipping.
    </div>
    </p>
<div class="param">
     inventory_level (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by inventory_level.
    </div>
    </p>
<div class="param">
     inventory_low (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by inventory_low; values: 1, 0.
    </div>
    </p>
<div class="param">
     out_of_stock (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by out_of_stock. To enable the filter, pass `out_of_stock`=`1`.
    </div>
    </p>
<div class="param">
     total_sold (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by total_sold.
    </div>
    </p>
<div class="param">
     type (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by type: `physical` or `digital`.
    </div>
    </p>
<div class="param">
     categories (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by categories.
    </div>
    </p>
<div class="param">
     keyword (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by keywords found in the name, description, sku, keywords, or brand name.
    </div>
    </p>
<div class="param">
     keyword_context (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Set context for a product search.
    </div>
    </p>
<div class="param">
     channel_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by channel.
    </div>
    </p>
<div class="param">
     status (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by status.
    </div>
    </p>
<div class="param">
     include (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Include sub-resources on a product, with a comma-separated list. Valid expansions currently include `variants`, `images`, `custom_fields`, and `bulk_pricing_rules`.
    </div>
    </p>
<div class="param">
     availability (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by availability. Values are: available, disabled, preorder.
    </div>
    </p>
<div class="param">
     page (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Control the page in a limited list of products.
    </div>
    </p>
<div class="param">
     limit (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Control the items per page.
    </div>
    </p>
<div class="param">
     direction (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Sort direction. Values are: asc, desc.
    </div>
    </p>
<div class="param">
     sort (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Field name to sort by.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#ProductCollectionResponse">
     ProductCollectionResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
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
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   An array of products and metadata.
   <a href="#ProductCollectionResponse">
    ProductCollectionResponse
   </a>
  </div>
  <div class="method">
   <a name="getVariantById">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/products/{product_id}/variants/{variant_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getVariantById
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Gets a `Variant` object.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     variant_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Variant` to which the resource belongs.
 format: int
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#VariantResponse">
     VariantResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A variant and metadata.
   <a href="#VariantResponse">
    VariantResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="getVariantMetafieldByProductIdAndVariantId">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/products/{product_id}/variants/{variant_id}/metafields/{metafield_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getVariantMetafieldByProductIdAndVariantId
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Gets a `Metafield`, by product_id and variant_id.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     metafield_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Metafield`.
 format: int
    </div>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     variant_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Variant` to which the resource belongs.
 format: int
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#MetafieldResponse">
     MetafieldResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A `Metafield` object.
   <a href="#MetafieldResponse">
    MetafieldResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="getVariantMetafieldsByProductIdAndVariantId">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/products/{product_id}/variants/{variant_id}/metafields
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getVariantMetafieldsByProductIdAndVariantId
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Gets a `Metafield` object list, by product_id and variant_id.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     variant_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Variant` to which the resource belongs.
 format: int
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Query parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     page (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Control the page in a limited list of products.
    </div>
    </p>
<div class="param">
     limit (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Control the items per page.
    </div>
    </p>
<div class="param">
     key (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter based on a metafield's key.
    </div>
    </p>
<div class="param">
     namespace (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter based on a metafield's key.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#MetaFieldCollectionResponse">
     MetaFieldCollectionResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
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
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   An array of metafields and metadata.
   <a href="#MetaFieldCollectionResponse">
    MetaFieldCollectionResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="getVariants">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/variants
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getVariants
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Returns a `Variant` object list from the BigCommerce Catalog.
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Query parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by id.
    </div>
    </p>
<div class="param">
     sku (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by sku.
    </div>
    </p>
<div class="param">
     page (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Control the page in a limited list of products.
    </div>
    </p>
<div class="param">
     limit (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Control the items per page.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#VariantCollectionResponse">
     VariantCollectionResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
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
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   An array of variants and metadata.
   <a href="#VariantCollectionResponse">
    VariantCollectionResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="getVariantsByProductId">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /catalog/products/{product_id}/variants
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getVariantsByProductId
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Returns a `Variant` object list from the BigCommerce Catalog.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Query parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     page (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Control the page in a limited list of products.
    </div>
    </p>
<div class="param">
     limit (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Control the items per page.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#VariantCollectionResponse">
     VariantCollectionResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
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
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   An array of variants and metadata.
   <a href="#VariantCollectionResponse">
    VariantCollectionResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="updateBrand">
   </a>
   <div class="method-path">
    <h2 class="put">
     <span class="http-method">
      PUT
     </span>
     /catalog/brands/{brand_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     updateBrand
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Updates a `Brand` in the BigCommerce Catalog.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     brand_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Brand` to which the resource belongs.
 format: int
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     brand
     <a href="#BrandPut">
      BrandPut
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — Returns a `Brand` from the BigCommerce Catalog.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#BrandResponse">
     BrandResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A `Brand` object.
   <a href="#BrandResponse">
    BrandResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
   <h4 class="field-label">
    409
   </h4>
   The `Brand` was in conflict with another product. This is the result of duplicate unique values, such as `name`.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
   <h4 class="field-label">
    422
   </h4>
   The `Brand` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
  </div>
  <div class="method">
   <a name="updateBrandMetafield">
   </a>
   <div class="method-path">
    <h2 class="put">
     <span class="http-method">
      PUT
     </span>
     /catalog/brands/{brand_id}/metafields/{metafield_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     updateBrandMetafield
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Updates a `Metafield` object.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     metafield_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Metafield`.
 format: int
    </div>
    </p>
<div class="param">
     brand_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Brand` to which the resource belongs.
 format: int
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     Metafield
     <a href="#MetafieldPut">
      MetafieldPut
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — A `Metafield` object.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#MetafieldResponse">
     MetafieldResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A metafield and metadata.
   <a href="#MetafieldResponse">
    MetafieldResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="updateCategory">
   </a>
   <div class="method-path">
    <h2 class="put">
     <span class="http-method">
      PUT
     </span>
     /catalog/categories/{category_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     updateCategory
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Updates a `Category` in the BigCommerce Catalog.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     category_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Category` to which the resource belongs.
 format: int
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     category
     <a href="#CategoryPut">
      CategoryPut
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — A BigCommerce `Category` object.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#CategoryResponse">
     CategoryResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A category object.
   <a href="#CategoryResponse">
    CategoryResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
   <h4 class="field-label">
    409
   </h4>
   The `Category` was in conflict with another category. This is the result of duplicate unique values, such as `name` or `custom_url`.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
   <h4 class="field-label">
    422
   </h4>
   The `Category` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
  </div>
  <div class="method">
   <a name="updateCategoryMetafield">
   </a>
   <div class="method-path">
    <h2 class="put">
     <span class="http-method">
      PUT
     </span>
     /catalog/categories/{category_id}/metafields/{metafield_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     updateCategoryMetafield
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Updates a `Metafield` object.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     metafield_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Metafield`.
 format: int
    </div>
    </p>
<div class="param">
     category_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Category` to which the resource belongs.
 format: int
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     Metafield
     <a href="#MetafieldPut">
      MetafieldPut
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — A `Metafield` object.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#MetafieldResponse">
     MetafieldResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A metafield and metadata.
   <a href="#MetafieldResponse">
    MetafieldResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="updateComplexRule">
   </a>
   <div class="method-path">
    <h2 class="put">
     <span class="http-method">
      PUT
     </span>
     /catalog/products/{product_id}/complex-rules/{complex_rule_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     updateComplexRule
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Updates an Product's `ComplexRule`, based on the `product_id` and `complex_rule_id`.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     complex_rule_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `ComplexRule`.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     ComplexRule
     <a href="#ComplexRulePut">
      ComplexRulePut
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — `ComplexRule` object.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#ComplexRuleResponse">
     ComplexRuleResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A `ComplexRule` object.
   <a href="#ComplexRuleResponse">
    ComplexRuleResponse
   </a>
   <h4 class="field-label">
    409
   </h4>
   The `ComplexRule` was in conflict with another `ComplexRule`. This is the result of duplicate conditions.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
   <h4 class="field-label">
    422
   </h4>
   The `ComplexRule` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
  </div>
  <div class="method">
   <a name="updateCustomField">
   </a>
   <div class="method-path">
    <h2 class="put">
     <span class="http-method">
      PUT
     </span>
     /catalog/products/{product_id}/custom-fields/{custom_field_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     updateCustomField
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Updates an Product's `CustomField`, based on the `product_id` and `custom_field_id`.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     custom_field_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `CustomField`.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     CustomField
     <a href="#CustomFieldPut">
      CustomFieldPut
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — `CustomField` object.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#CustomFieldResponse">
     CustomFieldResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A `CustomField` object.
   <a href="#CustomFieldResponse">
    CustomFieldResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
   <h4 class="field-label">
    422
   </h4>
   The `CustomField` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
  </div>
  <div class="method">
   <a name="updateModifier">
   </a>
   <div class="method-path">
    <h2 class="put">
     <span class="http-method">
      PUT
     </span>
     /catalog/products/{product_id}/modifiers/{modifier_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     updateModifier
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Updates an Product's `Modifier` based on the product_id and modifier_id.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     modifier_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Modifier`.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     modifier
     <a href="#ModifierPut">
      ModifierPut
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — A BigCommerce `Modifier` object.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#ModifierResponse">
     ModifierResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A `Modifier` object.
   <a href="#ModifierResponse">
    ModifierResponse
   </a>
   <h4 class="field-label">
    409
   </h4>
   The `Modifier` was in conflict with another modifier or option. This is the result of duplicate unique fields, such as `name`.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
   <h4 class="field-label">
    422
   </h4>
   The `Modifier` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
  </div>
  <div class="method">
   <a name="updateModifierValue">
   </a>
   <div class="method-path">
    <h2 class="put">
     <span class="http-method">
      PUT
     </span>
     /catalog/products/{product_id}/modifiers/{modifier_id}/values/{value_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     updateModifierValue
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Updates an Product's `ModifierValue` based on the product_id, modifier_id and value_id.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     modifier_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Modifier`.
    </div>
    </p>
<div class="param">
     value_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Modifier/Option Value`.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     ModifierValue
     <a href="#ModifierValuePut">
      ModifierValuePut
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — A BigCommerce `ModifierValue` object.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#ModifierValueResponse">
     ModifierValueResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A `ModifierValue` object.
   <a href="#ModifierValueResponse">
    ModifierValueResponse
   </a>
   <h4 class="field-label">
    422
   </h4>
   The `ModifierValue` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
  </div>
  <div class="method">
   <a name="updateOption">
   </a>
   <div class="method-path">
    <h2 class="put">
     <span class="http-method">
      PUT
     </span>
     /catalog/products/{product_id}/options/{option_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     updateOption
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Updates a Product's `Option`, based on the product_id and option_id.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     option_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Option`.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     option
     <a href="#OptionPut">
      OptionPut
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — A BigCommerce `Option` object.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#OptionResponse">
     OptionResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   An `Option` object.
   <a href="#OptionResponse">
    OptionResponse
   </a>
   <h4 class="field-label">
    409
   </h4>
   The `Option` was in conflict with another option. This is the result of duplicate unique fields, such as `name`.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
   <h4 class="field-label">
    422
   </h4>
   The `Option` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
  </div>
  <div class="method">
   <a name="updateOptionValue">
   </a>
   <div class="method-path">
    <h2 class="put">
     <span class="http-method">
      PUT
     </span>
     /catalog/products/{product_id}/options/{option_id}/values/{value_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     updateOptionValue
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Updates an Product's `OptionValue` based on the product_id, option_id and value_id.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     option_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Option`.
    </div>
    </p>
<div class="param">
     value_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Modifier/Option Value`.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     OptionValue
     <a href="#OptionValuePut">
      OptionValuePut
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — A BigCommerce `OptionValue` object.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#OptionValueResponse">
     OptionValueResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A `OptionValue` object.
   <a href="#OptionValueResponse">
    OptionValueResponse
   </a>
   <h4 class="field-label">
    422
   </h4>
   The `OptionValue` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
  </div>
  <div class="method">
   <a name="updateProduct">
   </a>
   <div class="method-path">
    <h2 class="put">
     <span class="http-method">
      PUT
     </span>
     /catalog/products/{product_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     updateProduct
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Updates a `Product` in the BigCommerce Catalog.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product
     <a href="#ProductPut">
      ProductPut
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — A BigCommerce `Product` object.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#ProductResponse">
     ProductResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A product.
   <a href="#ProductResponse">
    ProductResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
   <h4 class="field-label">
    409
   </h4>
   `Product` was in conflict with another product. This is the result of duplicate unique values such as name or SKU, a missing category, brand, or tax_class that the product is being associate to, or a conflicting bulk pricing rule.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
   <h4 class="field-label">
    422
   </h4>
   `Product` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
  </div>
  <div class="method">
   <a name="updateProductImage">
   </a>
   <div class="method-path">
    <h2 class="put">
     <span class="http-method">
      PUT
     </span>
     /catalog/products/{product_id}/images/{image_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     updateProductImage
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Updates an image on a product. Publicly accessible URLs and files (form post) are valid parameters.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     image_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Image` that is being operated on.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     productImage
     <a href="#ProductImagePut">
      ProductImagePut
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — A BigCommerce `ProductImage` object.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#ProductImageResponse">
     ProductImageResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A product image.
   <a href="#ProductImageResponse">
    ProductImageResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="updateProductMetafield">
   </a>
   <div class="method-path">
    <h2 class="put">
     <span class="http-method">
      PUT
     </span>
     /catalog/products/{product_id}/metafields/{metafield_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     updateProductMetafield
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Updates a `Metafield` object.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     metafield_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Metafield`.
 format: int
    </div>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     Metafield
     <a href="#MetafieldPut">
      MetafieldPut
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — A `Metafield` object.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#MetafieldResponse">
     MetafieldResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A metafield and metadata.
   <a href="#MetafieldResponse">
    MetafieldResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="updateProductVideo">
   </a>
   <div class="method-path">
    <h2 class="put">
     <span class="http-method">
      PUT
     </span>
     /catalog/products/{product_id}/videos/{video_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     updateProductVideo
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Updates a video on a product.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     video_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Video` being operated on.
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     productVideo
     <a href="#ProductVideoPut">
      ProductVideoPut
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — A BigCommerce `ProductVideo` object.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#ProductVideoResponse">
     ProductVideoResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A product video.
   <a href="#ProductVideoResponse">
    ProductVideoResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="updateVariant">
   </a>
   <div class="method-path">
    <h2 class="put">
     <span class="http-method">
      PUT
     </span>
     /catalog/products/{product_id}/variants/{variant_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     updateVariant
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Updates a `Variant` object.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     variant_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Variant` to which the resource belongs.
 format: int
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     Variant
     <a href="#VariantPut">
      VariantPut
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — A `Variant` object.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#VariantResponse">
     VariantResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A variant and metadata.
   <a href="#VariantResponse">
    VariantResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="updateVariantMetafield">
   </a>
   <div class="method-path">
    <h2 class="put">
     <span class="http-method">
      PUT
     </span>
     /catalog/products/{product_id}/variants/{variant_id}/metafields/{metafield_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     updateVariantMetafield
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Updates a `Metafield` object.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     metafield_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Metafield`.
 format: int
    </div>
    </p>
<div class="param">
     product_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Product` the resource belongs to.
    </div>
    </p>
<div class="param">
     variant_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Variant` to which the resource belongs.
 format: int
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     Metafield
     <a href="#MetafieldPut">
      MetafieldPut
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — A `Metafield` object.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#MetafieldResponse">
     MetafieldResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A metafield and metadata.
   <a href="#MetafieldResponse">
    MetafieldResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <hr/>
  <h1>
   <a name="Customers">
    Customers
   </a>
  </h1>
  <div class="method">
   <a name="createSubscriber">
   </a>
   <div class="method-path">
    <h2 class="post">
     <span class="http-method">
      POST
     </span>
     /customers/subscribers
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     createSubscriber
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Creates a `Subscriber` object.
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     subscriber
     <a href="#SubscriberPost">
      SubscriberPost
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — `Subscriber` object.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#SubscriberResponse">
     SubscriberResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A `Subscriber` object.
   <a href="#SubscriberResponse">
    SubscriberResponse
   </a>
   <h4 class="field-label">
    409
   </h4>
   The `Subscriber` was in conflict with another subscriber. This is the result of duplicate unique values such as email
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
   <h4 class="field-label">
    422
   </h4>
   The `Subscriber` was not valid. This is the result of missing required fields or invalid data. See the response for more details.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
  </div>
  <div class="method">
   <a name="deleteSubscriberById">
   </a>
   <div class="method-path">
    <h2 class="delete">
     <span class="http-method">
      DELETE
     </span>
     /customers/subscribers/{subscriber_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     deleteSubscriberById
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Deletes a `Subscriber` object.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     subscriber_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Subscriber` requested.
 format: int
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    204
   </h4>
   An empty response.
   <a href="#">
   </a>
  </div>
  <div class="method">
   <a name="deleteSubscribers">
   </a>
   <div class="method-path">
    <h2 class="delete">
     <span class="http-method">
      DELETE
     </span>
     /customers/subscribers
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     deleteSubscribers
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Deletes a Subscriber or Subscribers from BigCommerce Customers.
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Query parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     email (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by email.
    </div>
    </p>
<div class="param">
     first_name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by first_name.
    </div>
    </p>
<div class="param">
     last_name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by last_name.
    </div>
    </p>
<div class="param">
     source (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by source.
    </div>
    </p>
<div class="param">
     order_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by order_id.
    </div>
    </p>
<div class="param">
     date_created (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by date_created.
 format: date-time
    </div>
    </p>
<div class="param">
     date_modified (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by date_modified.
 format: date-time
    </div>
   </div>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    204
   </h4>
   An empty response.
   <a href="#">
   </a>
  </div>
  <div class="method">
   <a name="getSubscriberById">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /customers/subscribers/{subscriber_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getSubscriberById
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Gets `Subscriber` object.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     subscriber_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Subscriber` requested.
 format: int
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#SubscriberResponse">
     SubscriberResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A `Subscriber` object.
   <a href="#SubscriberResponse">
    SubscriberResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
  </div>
  <div class="method">
   <a name="getSubscribers">
   </a>
   <div class="method-path">
    <h2 class="get">
     <span class="http-method">
      GET
     </span>
     /customers/subscribers
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     getSubscribers
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Returns a paginated Subscribers collection.
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Query parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     email (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by email.
    </div>
    </p>
<div class="param">
     first_name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by first_name.
    </div>
    </p>
<div class="param">
     last_name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by last_name.
    </div>
    </p>
<div class="param">
     source (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by source.
    </div>
    </p>
<div class="param">
     order_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by order_id.
    </div>
    </p>
<div class="param">
     date_created (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by date_created.
 format: date-time
    </div>
    </p>
<div class="param">
     date_modified (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Filter items by date_modified.
 format: date-time
    </div>
    </p>
<div class="param">
     page (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Control the page in a limited list of products.
    </div>
    </p>
<div class="param">
     limit (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Query Parameter
     </span>
     — Control the items per page.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#SubscriberCollectionResponse">
     SubscriberCollectionResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
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
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   An array of subscriber objects and metadata.
   <a href="#SubscriberCollectionResponse">
    SubscriberCollectionResponse
   </a>
  </div>
  <div class="method">
   <a name="updateSubscriber">
   </a>
   <div class="method-path">
    <h2 class="put">
     <span class="http-method">
      PUT
     </span>
     /customers/subscribers/{subscriber_id}
    </h2>
   </div>
   <div class="method-summary">
    (
    <span class="nickname">
     updateSubscriber
    </span>
    )
   </div>
   <br/>
   <div class="method-notes">
    Updates a `Subscriber` object.
   </div>
   <h3 class="field-label">
    Path parameters
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     subscriber_id (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Path Parameter
     </span>
     — The ID of the `Subscriber` requested.
 format: int
    </div>
   </div>
   <h3 class="field-label">
    Consumes
   </h3>
   This API call consumes the following media types via the
   <span class="heaader">
    Content-Type
   </span>
   request header:
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Request body
   </h3>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     subscriber
     <a href="#SubscriberPut">
      SubscriberPut
     </a>
     (required)
    </div>
    <div class="param-desc">
     <span class="param-type">
      Body Parameter
     </span>
     — Returns a `Subscriber` object.
    </div>
   </div>
   <h3 class="field-label">
    Return type
   </h3>
   <div class="return-type">
    <a href="#SubscriberResponse">
     SubscriberResponse
    </a>
   </div>
   <h3 class="field-label">
    Example data
   </h3>
   <div class="example-data-content-type">
    Content-Type: application/json
   </div>
   <pre class="example"><code>{
  "data" : "",
  "meta" : { }
}</code></pre>
   <h3 class="field-label">
    Produces
   </h3>
   This API call produces the following media types according to the
   <span class="header">
    Accept
   </span>
   request header;
    the media type will be conveyed by the
   <span class="heaader">
    Content-Type
   </span>
   response header.
   <ul>
    <li>
     <code>
      application/json
     </code>
    </li>
   </ul>
   <h3 class="field-label">
    Responses
   </h3>
   <h4 class="field-label">
    200
   </h4>
   A `Subscriber` object.
   <a href="#SubscriberResponse">
    SubscriberResponse
   </a>
   <h4 class="field-label">
    404
   </h4>
   The resource was not found.
   <a href="#NotFound">
    NotFound
   </a>
   <h4 class="field-label">
    409
   </h4>
   The `Subscriber` was in conflict with another subscriber. This is the result of duplicate unique values, such as `email`.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
   <h4 class="field-label">
    422
   </h4>
   The `Subscriber` was not valid. This is the result of missing required fields, or of invalid data. See the response for more details.
   <a href="#ErrorResponse">
    ErrorResponse
   </a>
  </div>
  <hr/>
  <div class="up">
   <a href="#__Models">
    Up
   </a>
  </div>
  <h2>
   <a name="__Models">
    Models
   </a>
  </h2>
  [ Jump to
  <a href="#__Methods">
   Methods
  </a>
  ]
  <h3>
   Table of Contents
  </h3>
  <ol>
   <li>
    <a href="#Adjuster">
     <code>
      Adjuster
     </code>
    </a>
   </li>
   <li>
    <a href="#BaseError">
     <code>
      BaseError
     </code>
    </a>
   </li>
   <li>
    <a href="#Brand">
     <code>
      Brand
     </code>
    </a>
   </li>
   <li>
    <a href="#BrandBase">
     <code>
      BrandBase
     </code>
    </a>
   </li>
   <li>
    <a href="#BrandCollectionResponse">
     <code>
      BrandCollectionResponse
     </code>
    </a>
   </li>
   <li>
    <a href="#BrandPost">
     <code>
      BrandPost
     </code>
    </a>
   </li>
   <li>
    <a href="#BrandPut">
     <code>
      BrandPut
     </code>
    </a>
   </li>
   <li>
    <a href="#BrandResponse">
     <code>
      BrandResponse
     </code>
    </a>
   </li>
   <li>
    <a href="#BulkPricingRule">
     <code>
      BulkPricingRule
     </code>
    </a>
   </li>
   <li>
    <a href="#CatalogSummary">
     <code>
      CatalogSummary
     </code>
    </a>
   </li>
   <li>
    <a href="#CatalogSummaryResponse">
     <code>
      CatalogSummaryResponse
     </code>
    </a>
   </li>
   <li>
    <a href="#Category">
     <code>
      Category
     </code>
    </a>
   </li>
   <li>
    <a href="#CategoryBase">
     <code>
      CategoryBase
     </code>
    </a>
   </li>
   <li>
    <a href="#CategoryCollectionResponse">
     <code>
      CategoryCollectionResponse
     </code>
    </a>
   </li>
   <li>
    <a href="#CategoryNode">
     <code>
      CategoryNode
     </code>
    </a>
   </li>
   <li>
    <a href="#CategoryPost">
     <code>
      CategoryPost
     </code>
    </a>
   </li>
   <li>
    <a href="#CategoryPut">
     <code>
      CategoryPut
     </code>
    </a>
   </li>
   <li>
    <a href="#CategoryResponse">
     <code>
      CategoryResponse
     </code>
    </a>
   </li>
   <li>
    <a href="#CategoryTreeCollectionResponse">
     <code>
      CategoryTreeCollectionResponse
     </code>
    </a>
   </li>
   <li>
    <a href="#CollectionMeta">
     <code>
      CollectionMeta
     </code>
    </a>
   </li>
   <li>
    <a href="#ComplexRule">
     <code>
      ComplexRule
     </code>
    </a>
   </li>
   <li>
    <a href="#ComplexRuleBase">
     <code>
      ComplexRuleBase
     </code>
    </a>
   </li>
   <li>
    <a href="#ComplexRuleCollectionResponse">
     <code>
      ComplexRuleCollectionResponse
     </code>
    </a>
   </li>
   <li>
    <a href="#ComplexRuleCondition">
     <code>
      ComplexRuleCondition
     </code>
    </a>
   </li>
   <li>
    <a href="#ComplexRuleConditionBase">
     <code>
      ComplexRuleConditionBase
     </code>
    </a>
   </li>
   <li>
    <a href="#ComplexRuleConditionPost">
     <code>
      ComplexRuleConditionPost
     </code>
    </a>
   </li>
   <li>
    <a href="#ComplexRuleConditionPut">
     <code>
      ComplexRuleConditionPut
     </code>
    </a>
   </li>
   <li>
    <a href="#ComplexRulePost">
     <code>
      ComplexRulePost
     </code>
    </a>
   </li>
   <li>
    <a href="#ComplexRulePut">
     <code>
      ComplexRulePut
     </code>
    </a>
   </li>
   <li>
    <a href="#ComplexRuleResponse">
     <code>
      ComplexRuleResponse
     </code>
    </a>
   </li>
   <li>
    <a href="#CustomField">
     <code>
      CustomField
     </code>
    </a>
   </li>
   <li>
    <a href="#CustomFieldBase">
     <code>
      CustomFieldBase
     </code>
    </a>
   </li>
   <li>
    <a href="#CustomFieldCollectionResponse">
     <code>
      CustomFieldCollectionResponse
     </code>
    </a>
   </li>
   <li>
    <a href="#CustomFieldPost">
     <code>
      CustomFieldPost
     </code>
    </a>
   </li>
   <li>
    <a href="#CustomFieldPut">
     <code>
      CustomFieldPut
     </code>
    </a>
   </li>
   <li>
    <a href="#CustomFieldResponse">
     <code>
      CustomFieldResponse
     </code>
    </a>
   </li>
   <li>
    <a href="#CustomUrlCategory">
     <code>
      CustomUrlCategory
     </code>
    </a>
   </li>
   <li>
    <a href="#CustomUrlProduct">
     <code>
      CustomUrlProduct
     </code>
    </a>
   </li>
   <li>
    <a href="#DetailedErrors">
     <code>
      DetailedErrors
     </code>
    </a>
   </li>
   <li>
    <a href="#ErrorResponse">
     <code>
      ErrorResponse
     </code>
    </a>
   </li>
   <li>
    <a href="#ImageResponse">
     <code>
      ImageResponse
     </code>
    </a>
   </li>
   <li>
    <a href="#Meta">
     <code>
      Meta
     </code>
    </a>
   </li>
   <li>
    <a href="#MetaFieldCollectionResponse">
     <code>
      MetaFieldCollectionResponse
     </code>
    </a>
   </li>
   <li>
    <a href="#Metafield">
     <code>
      Metafield
     </code>
    </a>
   </li>
   <li>
    <a href="#MetafieldBase">
     <code>
      MetafieldBase
     </code>
    </a>
   </li>
   <li>
    <a href="#MetafieldPost">
     <code>
      MetafieldPost
     </code>
    </a>
   </li>
   <li>
    <a href="#MetafieldPut">
     <code>
      MetafieldPut
     </code>
    </a>
   </li>
   <li>
    <a href="#MetafieldResponse">
     <code>
      MetafieldResponse
     </code>
    </a>
   </li>
   <li>
    <a href="#Modifier">
     <code>
      Modifier
     </code>
    </a>
   </li>
   <li>
    <a href="#ModifierBase">
     <code>
      ModifierBase
     </code>
    </a>
   </li>
   <li>
    <a href="#ModifierCollectionResponse">
     <code>
      ModifierCollectionResponse
     </code>
    </a>
   </li>
   <li>
    <a href="#ModifierPost">
     <code>
      ModifierPost
     </code>
    </a>
   </li>
   <li>
    <a href="#ModifierPut">
     <code>
      ModifierPut
     </code>
    </a>
   </li>
   <li>
    <a href="#ModifierResponse">
     <code>
      ModifierResponse
     </code>
    </a>
   </li>
   <li>
    <a href="#ModifierValue">
     <code>
      ModifierValue
     </code>
    </a>
   </li>
   <li>
    <a href="#ModifierValueBase">
     <code>
      ModifierValueBase
     </code>
    </a>
   </li>
   <li>
    <a href="#ModifierValueBase_adjusters">
     <code>
      ModifierValueBase_adjusters
     </code>
    </a>
   </li>
   <li>
    <a href="#ModifierValueBase_adjusters_purchasing_disabled">
     <code>
      ModifierValueBase_adjusters_purchasing_disabled
     </code>
    </a>
   </li>
   <li>
    <a href="#ModifierValueCollectionResponse">
     <code>
      ModifierValueCollectionResponse
     </code>
    </a>
   </li>
   <li>
    <a href="#ModifierValuePost">
     <code>
      ModifierValuePost
     </code>
    </a>
   </li>
   <li>
    <a href="#ModifierValuePut">
     <code>
      ModifierValuePut
     </code>
    </a>
   </li>
   <li>
    <a href="#ModifierValueResponse">
     <code>
      ModifierValueResponse
     </code>
    </a>
   </li>
   <li>
    <a href="#NotFound">
     <code>
      NotFound
     </code>
    </a>
   </li>
   <li>
    <a href="#Option">
     <code>
      Option
     </code>
    </a>
   </li>
   <li>
    <a href="#OptionBase">
     <code>
      OptionBase
     </code>
    </a>
   </li>
   <li>
    <a href="#OptionCollectionResponse">
     <code>
      OptionCollectionResponse
     </code>
    </a>
   </li>
   <li>
    <a href="#OptionConfig">
     <code>
      OptionConfig
     </code>
    </a>
   </li>
   <li>
    <a href="#OptionPost">
     <code>
      OptionPost
     </code>
    </a>
   </li>
   <li>
    <a href="#OptionPut">
     <code>
      OptionPut
     </code>
    </a>
   </li>
   <li>
    <a href="#OptionResponse">
     <code>
      OptionResponse
     </code>
    </a>
   </li>
   <li>
    <a href="#OptionValue">
     <code>
      OptionValue
     </code>
    </a>
   </li>
   <li>
    <a href="#OptionValueBase">
     <code>
      OptionValueBase
     </code>
    </a>
   </li>
   <li>
    <a href="#OptionValueCollectionResponse">
     <code>
      OptionValueCollectionResponse
     </code>
    </a>
   </li>
   <li>
    <a href="#OptionValuePost">
     <code>
      OptionValuePost
     </code>
    </a>
   </li>
   <li>
    <a href="#OptionValueProductBase">
     <code>
      OptionValueProductBase
     </code>
    </a>
   </li>
   <li>
    <a href="#OptionValueProductPost">
     <code>
      OptionValueProductPost
     </code>
    </a>
   </li>
   <li>
    <a href="#OptionValuePut">
     <code>
      OptionValuePut
     </code>
    </a>
   </li>
   <li>
    <a href="#OptionValueResponse">
     <code>
      OptionValueResponse
     </code>
    </a>
   </li>
   <li>
    <a href="#OptionValueVariant">
     <code>
      OptionValueVariant
     </code>
    </a>
   </li>
   <li>
    <a href="#OptionValueVariantPost">
     <code>
      OptionValueVariantPost
     </code>
    </a>
   </li>
   <li>
    <a href="#Pagination">
     <code>
      Pagination
     </code>
    </a>
   </li>
   <li>
    <a href="#Pagination_links">
     <code>
      Pagination_links
     </code>
    </a>
   </li>
   <li>
    <a href="#Product">
     <code>
      Product
     </code>
    </a>
   </li>
   <li>
    <a href="#ProductBase">
     <code>
      ProductBase
     </code>
    </a>
   </li>
   <li>
    <a href="#ProductCollectionResponse">
     <code>
      ProductCollectionResponse
     </code>
    </a>
   </li>
   <li>
    <a href="#ProductImage">
     <code>
      ProductImage
     </code>
    </a>
   </li>
   <li>
    <a href="#ProductImageBase">
     <code>
      ProductImageBase
     </code>
    </a>
   </li>
   <li>
    <a href="#ProductImageCollectionResponse">
     <code>
      ProductImageCollectionResponse
     </code>
    </a>
   </li>
   <li>
    <a href="#ProductImagePost">
     <code>
      ProductImagePost
     </code>
    </a>
   </li>
   <li>
    <a href="#ProductImagePut">
     <code>
      ProductImagePut
     </code>
    </a>
   </li>
   <li>
    <a href="#ProductImageResponse">
     <code>
      ProductImageResponse
     </code>
    </a>
   </li>
   <li>
    <a href="#ProductPost">
     <code>
      ProductPost
     </code>
    </a>
   </li>
   <li>
    <a href="#ProductPut">
     <code>
      ProductPut
     </code>
    </a>
   </li>
   <li>
    <a href="#ProductResponse">
     <code>
      ProductResponse
     </code>
    </a>
   </li>
   <li>
    <a href="#ProductVideo">
     <code>
      ProductVideo
     </code>
    </a>
   </li>
   <li>
    <a href="#ProductVideoBase">
     <code>
      ProductVideoBase
     </code>
    </a>
   </li>
   <li>
    <a href="#ProductVideoCollectionResponse">
     <code>
      ProductVideoCollectionResponse
     </code>
    </a>
   </li>
   <li>
    <a href="#ProductVideoPost">
     <code>
      ProductVideoPost
     </code>
    </a>
   </li>
   <li>
    <a href="#ProductVideoPut">
     <code>
      ProductVideoPut
     </code>
    </a>
   </li>
   <li>
    <a href="#ProductVideoResponse">
     <code>
      ProductVideoResponse
     </code>
    </a>
   </li>
   <li>
    <a href="#ResourceImage">
     <code>
      ResourceImage
     </code>
    </a>
   </li>
   <li>
    <a href="#Subscriber">
     <code>
      Subscriber
     </code>
    </a>
   </li>
   <li>
    <a href="#SubscriberBase">
     <code>
      SubscriberBase
     </code>
    </a>
   </li>
   <li>
    <a href="#SubscriberCollectionResponse">
     <code>
      SubscriberCollectionResponse
     </code>
    </a>
   </li>
   <li>
    <a href="#SubscriberPost">
     <code>
      SubscriberPost
     </code>
    </a>
   </li>
   <li>
    <a href="#SubscriberPut">
     <code>
      SubscriberPut
     </code>
    </a>
   </li>
   <li>
    <a href="#SubscriberResponse">
     <code>
      SubscriberResponse
     </code>
    </a>
   </li>
   <li>
    <a href="#Variant">
     <code>
      Variant
     </code>
    </a>
   </li>
   <li>
    <a href="#VariantBase">
     <code>
      VariantBase
     </code>
    </a>
   </li>
   <li>
    <a href="#VariantCollectionResponse">
     <code>
      VariantCollectionResponse
     </code>
    </a>
   </li>
   <li>
    <a href="#VariantPost">
     <code>
      VariantPost
     </code>
    </a>
   </li>
   <li>
    <a href="#VariantProductPost">
     <code>
      VariantProductPost
     </code>
    </a>
   </li>
   <li>
    <a href="#VariantProductPut">
     <code>
      VariantProductPut
     </code>
    </a>
   </li>
   <li>
    <a href="#VariantPut">
     <code>
      VariantPut
     </code>
    </a>
   </li>
   <li>
    <a href="#VariantResponse">
     <code>
      VariantResponse
     </code>
    </a>
   </li>
  </ol>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="Adjuster">
     Adjuster
    </a>
   </h3>
   <div class="model-description">
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     adjuster (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The type of adjuster for either the price or the weight of the variant, when the modifier value is selected on the storefront.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     relative
    </div>
    <div class="param-enum">
     percentage
    </div>
    </p>
<div class="param">
     adjuster_value (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#number">
       BigDecimal
      </a>
     </span>
     The numeric amount by which the adjuster will change either the price or the weight of the variant, when the modifier value is selected on the storefront.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="BaseError">
     BaseError
    </a>
   </h3>
   <div class="model-description">
    Error payload for the BigCommerce API.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     status (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The HTTP status code.
    </div>
    </p>
<div class="param">
     title (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The error title describing the particular error.
    </div>
    </p>
<div class="param">
     type (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
    </div>
    </p>
<div class="param">
     instance (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="Brand">
     Brand
    </a>
   </h3>
   <div class="model-description">
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The name of the brand. Must be unique.
    </div>
    </p>
<div class="param">
     page_title (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The title shown in the browser while viewing the brand.
    </div>
    </p>
<div class="param">
     meta_keywords (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       array[String]
      </a>
     </span>
     Comma separated list of meta keywords to include in the HTML.
    </div>
    </p>
<div class="param">
     meta_description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     A meta description to include.
    </div>
    </p>
<div class="param">
     search_keywords (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     A comma separated list of keywords that can be used to locate this brand.
    </div>
    </p>
<div class="param">
     image_url (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Image URL used for this category on the storefront. Images can be uploaded via form file post to `/brands/{brandId}/image`, or by providing a publicly accessible URL in this field.
    </div>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the brand; increments sequentially.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="BrandBase">
     BrandBase
    </a>
   </h3>
   <div class="model-description">
    Common Brand properties.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The name of the brand. Must be unique.
    </div>
    </p>
<div class="param">
     page_title (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The title shown in the browser while viewing the brand.
    </div>
    </p>
<div class="param">
     meta_keywords (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       array[String]
      </a>
     </span>
     Comma separated list of meta keywords to include in the HTML.
    </div>
    </p>
<div class="param">
     meta_description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     A meta description to include.
    </div>
    </p>
<div class="param">
     search_keywords (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     A comma separated list of keywords that can be used to locate this brand.
    </div>
    </p>
<div class="param">
     image_url (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Image URL used for this category on the storefront. Images can be uploaded via form file post to `/brands/{brandId}/image`, or by providing a publicly accessible URL in this field.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="BrandCollectionResponse">
     BrandCollectionResponse
    </a>
   </h3>
   <div class="model-description">
    Response payload for the Bigcommerce API.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Brand">
       array[Brand]
      </a>
     </span>
    </div>
    </p>
<div class="param">
     meta (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#CollectionMeta">
       CollectionMeta
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="BrandPost">
     BrandPost
    </a>
   </h3>
   <div class="model-description">
    The model for a POST to create brand.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The name of the brand. Must be unique.
    </div>
    </p>
<div class="param">
     page_title (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The title shown in the browser while viewing the brand.
    </div>
    </p>
<div class="param">
     meta_keywords (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       array[String]
      </a>
     </span>
     Comma separated list of meta keywords to include in the HTML.
    </div>
    </p>
<div class="param">
     meta_description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     A meta description to include.
    </div>
    </p>
<div class="param">
     search_keywords (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     A comma separated list of keywords that can be used to locate this brand.
    </div>
    </p>
<div class="param">
     image_url (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Image URL used for this category on the storefront. Images can be uploaded via form file post to `/brands/{brandId}/image`, or by providing a publicly accessible URL in this field.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="BrandPut">
     BrandPut
    </a>
   </h3>
   <div class="model-description">
    The model for a PUT to update brand.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The name of the brand. Must be unique.
    </div>
    </p>
<div class="param">
     page_title (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The title shown in the browser while viewing the brand.
    </div>
    </p>
<div class="param">
     meta_keywords (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       array[String]
      </a>
     </span>
     Comma separated list of meta keywords to include in the HTML.
    </div>
    </p>
<div class="param">
     meta_description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     A meta description to include.
    </div>
    </p>
<div class="param">
     search_keywords (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     A comma separated list of keywords that can be used to locate this brand.
    </div>
    </p>
<div class="param">
     image_url (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Image URL used for this category on the storefront. Images can be uploaded via form file post to `/brands/{brandId}/image`, or by providing a publicly accessible URL in this field.
    </div>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the brand; increments sequentially.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="BrandResponse">
     BrandResponse
    </a>
   </h3>
   <div class="model-description">
    Response payload for the Bigcommerce API.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Brand">
       Brand
      </a>
     </span>
    </div>
    </p>
<div class="param">
     meta (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Meta">
       Meta
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="BulkPricingRule">
     BulkPricingRule
    </a>
   </h3>
   <div class="model-description">
    Rules that offer price discounts based on quantity breaks.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The ID of the bulk pricing rule.
    </div>
    </p>
<div class="param">
     quantity_min (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The minimum inclusive quantity of a product to satisfy this rule. Must be greater than or equal to zero.
    </div>
    </p>
<div class="param">
     quantity_max (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The maximum inclusive quantity of a product to satisfy this rule. Must be greater than the `quantity_min` value – unless this field has a value of 0 (zero), in which case there will be no maximum bound for this rule.
    </div>
    </p>
<div class="param">
     type (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The type of adjustment that is made. Values: `price` - the adjustment amount per product; `percent` - the adjustment as a percentage of the original price; `fixed` - the adjusted absolute price of the product.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     price
    </div>
    <div class="param-enum">
     percent
    </div>
    <div class="param-enum">
     fixed
    </div>
    </p>
<div class="param">
     amount (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     The value of the adjustment by the bulk pricing rule.
 format: double
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="CatalogSummary">
     CatalogSummary
    </a>
   </h3>
   <div class="model-description">
    A BigCommerce Catalog Summary object describes a lightweight summary of the catalog.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     inventory_count (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     A count of all inventory items in the catalog.
    </div>
    </p>
<div class="param">
     inventory_value (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     Total value of store's inventory.
 format: double
    </div>
    </p>
<div class="param">
     primary_category_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     ID of the category containing the most products.
    </div>
    </p>
<div class="param">
     primary_category_name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Name of the category containing the most products.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="CatalogSummaryResponse">
     CatalogSummaryResponse
    </a>
   </h3>
   <div class="model-description">
    Response payload for the Bigcommerce API.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#CatalogSummary">
       CatalogSummary
      </a>
     </span>
    </div>
    </p>
<div class="param">
     meta (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Meta">
       Meta
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="Category">
     Category
    </a>
   </h3>
   <div class="model-description">
    A BigCommerce category object.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     parent_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the category's parent. This field controls where the category sits in the tree of categories that organize the catalog.
    </div>
    </p>
<div class="param">
     name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The name displayed for the category. Name is unique with respect to the category's siblings.
    </div>
    </p>
<div class="param">
     description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The product description, which can include HTML formatting.
    </div>
    </p>
<div class="param">
     views (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     Number of views the category has on the storefront.
    </div>
    </p>
<div class="param">
     sort_order (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     Priority this category will be given when included in the menu and category pages. The lower the number, the closer to the top of the results the category will be.
    </div>
    </p>
<div class="param">
     page_title (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Custom title for the category page, if not defined the category name will be used as the meta title.
    </div>
    </p>
<div class="param">
     search_keywords (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     A comma separated list of keywords that can be used to locate the category when searching the store.
    </div>
    </p>
<div class="param">
     meta_keywords (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       array[String]
      </a>
     </span>
     Custom meta keywords for the category page. If not defined, the store's default keywords will be used. Must post as an array like: ["awesome","sauce"].
    </div>
    </p>
<div class="param">
     meta_description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Custom meta description for the category page. If not defined, the store's default meta description will be used.
    </div>
    </p>
<div class="param">
     layout_file (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The layout template file used to render this category.
    </div>
    </p>
<div class="param">
     is_visible (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag to determine whether the product should be displayed to customers browsing the store. If `true`, the category will be displayed. If `false`, the category will be hidden from view.
    </div>
    </p>
<div class="param">
     default_product_sort (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Determines how the products are sorted on category page load.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     use_store_settings
    </div>
    <div class="param-enum">
     featured
    </div>
    <div class="param-enum">
     newest
    </div>
    <div class="param-enum">
     best_selling
    </div>
    <div class="param-enum">
     alpha_asc
    </div>
    <div class="param-enum">
     alpha_desc
    </div>
    <div class="param-enum">
     avg_customer_review
    </div>
    <div class="param-enum">
     price_asc
    </div>
    <div class="param-enum">
     price_desc
    </div>
    </p>
<div class="param">
     image_url (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Image URL used for this category on the storefront. Images can be uploaded via form file post to `/categories/{categoryId}/image`, or by providing a publicly accessible URL in this field.
    </div>
    </p>
<div class="param">
     custom_url (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#CustomUrlCategory">
       CustomUrlCategory
      </a>
     </span>
    </div>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the category; increments sequentially.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="CategoryBase">
     CategoryBase
    </a>
   </h3>
   <div class="model-description">
    Common Category properties.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     parent_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the category's parent. This field controls where the category sits in the tree of categories that organize the catalog.
    </div>
    </p>
<div class="param">
     name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The name displayed for the category. Name is unique with respect to the category's siblings.
    </div>
    </p>
<div class="param">
     description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The product description, which can include HTML formatting.
    </div>
    </p>
<div class="param">
     views (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     Number of views the category has on the storefront.
    </div>
    </p>
<div class="param">
     sort_order (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     Priority this category will be given when included in the menu and category pages. The lower the number, the closer to the top of the results the category will be.
    </div>
    </p>
<div class="param">
     page_title (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Custom title for the category page, if not defined the category name will be used as the meta title.
    </div>
    </p>
<div class="param">
     search_keywords (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     A comma separated list of keywords that can be used to locate the category when searching the store.
    </div>
    </p>
<div class="param">
     meta_keywords (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       array[String]
      </a>
     </span>
     Custom meta keywords for the category page. If not defined, the store's default keywords will be used. Must post as an array like: ["awesome","sauce"].
    </div>
    </p>
<div class="param">
     meta_description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Custom meta description for the category page. If not defined, the store's default meta description will be used.
    </div>
    </p>
<div class="param">
     layout_file (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The layout template file used to render this category.
    </div>
    </p>
<div class="param">
     is_visible (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag to determine whether the product should be displayed to customers browsing the store. If `true`, the category will be displayed. If `false`, the category will be hidden from view.
    </div>
    </p>
<div class="param">
     default_product_sort (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Determines how the products are sorted on category page load.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     use_store_settings
    </div>
    <div class="param-enum">
     featured
    </div>
    <div class="param-enum">
     newest
    </div>
    <div class="param-enum">
     best_selling
    </div>
    <div class="param-enum">
     alpha_asc
    </div>
    <div class="param-enum">
     alpha_desc
    </div>
    <div class="param-enum">
     avg_customer_review
    </div>
    <div class="param-enum">
     price_asc
    </div>
    <div class="param-enum">
     price_desc
    </div>
    </p>
<div class="param">
     image_url (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Image URL used for this category on the storefront. Images can be uploaded via form file post to `/categories/{categoryId}/image`, or by providing a publicly accessible URL in this field.
    </div>
    </p>
<div class="param">
     custom_url (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#CustomUrlCategory">
       CustomUrlCategory
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="CategoryCollectionResponse">
     CategoryCollectionResponse
    </a>
   </h3>
   <div class="model-description">
    Response payload for the Bigcommerce API.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Category">
       array[Category]
      </a>
     </span>
    </div>
    </p>
<div class="param">
     meta (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#CollectionMeta">
       CollectionMeta
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="CategoryNode">
     CategoryNode
    </a>
   </h3>
   <div class="model-description">
    A BigCommerce category node object. Used to reflect parent &lt;&gt; child category relationships.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the category; increments sequentially.
    </div>
    </p>
<div class="param">
     parent_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the category's parent. This field controls where the category sits in the tree of categories that organize the catalog.
    </div>
    </p>
<div class="param">
     name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The name displayed for the category. Name is unique with respect to the category's siblings.
    </div>
    </p>
<div class="param">
     is_visible (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag to determine whether the product should be displayed to customers browsing the store. If `true`, the category will be displayed. If `false`, the category will be hidden from view.
    </div>
    </p>
<div class="param">
     url (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The custom URL for the category on the storefront.
    </div>
    </p>
<div class="param">
     children (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#CategoryNode">
       array[CategoryNode]
      </a>
     </span>
     The list of children of the category.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="CategoryPost">
     CategoryPost
    </a>
   </h3>
   <div class="model-description">
    The model for a POST to create category.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     parent_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the category's parent. This field controls where the category sits in the tree of categories that organize the catalog.
    </div>
    </p>
<div class="param">
     name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The name displayed for the category. Name is unique with respect to the category's siblings.
    </div>
    </p>
<div class="param">
     description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The product description, which can include HTML formatting.
    </div>
    </p>
<div class="param">
     views (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     Number of views the category has on the storefront.
    </div>
    </p>
<div class="param">
     sort_order (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     Priority this category will be given when included in the menu and category pages. The lower the number, the closer to the top of the results the category will be.
    </div>
    </p>
<div class="param">
     page_title (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Custom title for the category page, if not defined the category name will be used as the meta title.
    </div>
    </p>
<div class="param">
     search_keywords (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     A comma separated list of keywords that can be used to locate the category when searching the store.
    </div>
    </p>
<div class="param">
     meta_keywords (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       array[String]
      </a>
     </span>
     Custom meta keywords for the category page. If not defined, the store's default keywords will be used. Must post as an array like: ["awesome","sauce"].
    </div>
    </p>
<div class="param">
     meta_description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Custom meta description for the category page. If not defined, the store's default meta description will be used.
    </div>
    </p>
<div class="param">
     layout_file (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The layout template file used to render this category.
    </div>
    </p>
<div class="param">
     is_visible (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag to determine whether the product should be displayed to customers browsing the store. If `true`, the category will be displayed. If `false`, the category will be hidden from view.
    </div>
    </p>
<div class="param">
     default_product_sort (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Determines how the products are sorted on category page load.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     use_store_settings
    </div>
    <div class="param-enum">
     featured
    </div>
    <div class="param-enum">
     newest
    </div>
    <div class="param-enum">
     best_selling
    </div>
    <div class="param-enum">
     alpha_asc
    </div>
    <div class="param-enum">
     alpha_desc
    </div>
    <div class="param-enum">
     avg_customer_review
    </div>
    <div class="param-enum">
     price_asc
    </div>
    <div class="param-enum">
     price_desc
    </div>
    </p>
<div class="param">
     image_url (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Image URL used for this category on the storefront. Images can be uploaded via form file post to `/categories/{categoryId}/image`, or by providing a publicly accessible URL in this field.
    </div>
    </p>
<div class="param">
     custom_url (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#CustomUrlCategory">
       CustomUrlCategory
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="CategoryPut">
     CategoryPut
    </a>
   </h3>
   <div class="model-description">
    The model for a PUT to update category.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     parent_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the category's parent. This field controls where the category sits in the tree of categories that organize the catalog.
    </div>
    </p>
<div class="param">
     name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The name displayed for the category. Name is unique with respect to the category's siblings.
    </div>
    </p>
<div class="param">
     description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The product description, which can include HTML formatting.
    </div>
    </p>
<div class="param">
     views (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     Number of views the category has on the storefront.
    </div>
    </p>
<div class="param">
     sort_order (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     Priority this category will be given when included in the menu and category pages. The lower the number, the closer to the top of the results the category will be.
    </div>
    </p>
<div class="param">
     page_title (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Custom title for the category page, if not defined the category name will be used as the meta title.
    </div>
    </p>
<div class="param">
     search_keywords (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     A comma separated list of keywords that can be used to locate the category when searching the store.
    </div>
    </p>
<div class="param">
     meta_keywords (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       array[String]
      </a>
     </span>
     Custom meta keywords for the category page. If not defined, the store's default keywords will be used. Must post as an array like: ["awesome","sauce"].
    </div>
    </p>
<div class="param">
     meta_description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Custom meta description for the category page. If not defined, the store's default meta description will be used.
    </div>
    </p>
<div class="param">
     layout_file (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The layout template file used to render this category.
    </div>
    </p>
<div class="param">
     is_visible (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag to determine whether the product should be displayed to customers browsing the store. If `true`, the category will be displayed. If `false`, the category will be hidden from view.
    </div>
    </p>
<div class="param">
     default_product_sort (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Determines how the products are sorted on category page load.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     use_store_settings
    </div>
    <div class="param-enum">
     featured
    </div>
    <div class="param-enum">
     newest
    </div>
    <div class="param-enum">
     best_selling
    </div>
    <div class="param-enum">
     alpha_asc
    </div>
    <div class="param-enum">
     alpha_desc
    </div>
    <div class="param-enum">
     avg_customer_review
    </div>
    <div class="param-enum">
     price_asc
    </div>
    <div class="param-enum">
     price_desc
    </div>
    </p>
<div class="param">
     image_url (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Image URL used for this category on the storefront. Images can be uploaded via form file post to `/categories/{categoryId}/image`, or by providing a publicly accessible URL in this field.
    </div>
    </p>
<div class="param">
     custom_url (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#CustomUrlCategory">
       CustomUrlCategory
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="CategoryResponse">
     CategoryResponse
    </a>
   </h3>
   <div class="model-description">
    Response payload for the Bigcommerce API.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Category">
       Category
      </a>
     </span>
    </div>
    </p>
<div class="param">
     meta (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Meta">
       Meta
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="CategoryTreeCollectionResponse">
     CategoryTreeCollectionResponse
    </a>
   </h3>
   <div class="model-description">
    Response payload for the Bigcommerce API.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#CategoryNode">
       array[CategoryNode]
      </a>
     </span>
    </div>
    </p>
<div class="param">
     meta (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#CollectionMeta">
       CollectionMeta
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="CollectionMeta">
     CollectionMeta
    </a>
   </h3>
   <div class="model-description">
    Data about the response, including pagination and collection totals.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     pagination (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Pagination">
       Pagination
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ComplexRule">
     ComplexRule
    </a>
   </h3>
   <div class="model-description">
    Apply price, weight, image, or availabilty adjustments to product, based on a set of conditions. A complex rule's condition must either contain more than one modifier value, or else contain a modifier value and a variant id.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the product with which the rule is associated; increments sequentially.
    </div>
    </p>
<div class="param">
     sort_order (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     Priority this rule will be given, when making adjustments to the product properties.
    </div>
    </p>
<div class="param">
     enabled (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag for determining whether the rule is to be used when adjusting a product's price, weight, image, or availabilty.
    </div>
    </p>
<div class="param">
     stop (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag for determining whether other rules should not be applied after this rule has been applied.
    </div>
    </p>
<div class="param">
     purchasing_disabled (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag for determining whether the rule should disable purchasing of a product when the conditions are applied.
    </div>
    </p>
<div class="param">
     purchasing_disabled_message (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Message displayed on the storefront when a rule disables the purchasing of a product.
    </div>
    </p>
<div class="param">
     purchasing_hidden (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag for determining whether the rule should hide purchasing of a product when the conditions are applied.
    </div>
    </p>
<div class="param">
     price_adjuster (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Adjuster">
       Adjuster
      </a>
     </span>
    </div>
    </p>
<div class="param">
     weight_adjuster (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Adjuster">
       Adjuster
      </a>
     </span>
    </div>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the rule; increments sequentially.
    </div>
    </p>
<div class="param">
     image_url (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The URL for an image displayed on the storefront when the conditions are applied.
    </div>
    </p>
<div class="param">
     conditions (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#ComplexRuleCondition">
       array[ComplexRuleCondition]
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ComplexRuleBase">
     ComplexRuleBase
    </a>
   </h3>
   <div class="model-description">
    Common ComplexRule properties.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the product with which the rule is associated; increments sequentially.
    </div>
    </p>
<div class="param">
     sort_order (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     Priority this rule will be given, when making adjustments to the product properties.
    </div>
    </p>
<div class="param">
     enabled (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag for determining whether the rule is to be used when adjusting a product's price, weight, image, or availabilty.
    </div>
    </p>
<div class="param">
     stop (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag for determining whether other rules should not be applied after this rule has been applied.
    </div>
    </p>
<div class="param">
     purchasing_disabled (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag for determining whether the rule should disable purchasing of a product when the conditions are applied.
    </div>
    </p>
<div class="param">
     purchasing_disabled_message (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Message displayed on the storefront when a rule disables the purchasing of a product.
    </div>
    </p>
<div class="param">
     purchasing_hidden (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag for determining whether the rule should hide purchasing of a product when the conditions are applied.
    </div>
    </p>
<div class="param">
     price_adjuster (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Adjuster">
       Adjuster
      </a>
     </span>
    </div>
    </p>
<div class="param">
     weight_adjuster (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Adjuster">
       Adjuster
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ComplexRuleCollectionResponse">
     ComplexRuleCollectionResponse
    </a>
   </h3>
   <div class="model-description">
    Response payload for the Bigcommerce API.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#ComplexRule">
       array[ComplexRule]
      </a>
     </span>
    </div>
    </p>
<div class="param">
     meta (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#CollectionMeta">
       CollectionMeta
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ComplexRuleCondition">
     ComplexRuleCondition
    </a>
   </h3>
   <div class="model-description">
    Complex rules may return with conditions that apply to one or more variants, or with a single modifier value (if the rules were created using the v2 API or the control panel). Complex rules created or updated in the v3 API must have conditions that either reference multiple `modifier_value_id`'s, or else reference a `modifier_value_id` and a `variant_id`.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the rule condition; increments sequentially.
    </div>
    </p>
<div class="param">
     rule_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the rule with which the condition is associated.
    </div>
    </p>
<div class="param">
     modifier_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the modifier with which the rule condition is associated.
    </div>
    </p>
<div class="param">
     modifier_value_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the modifier value with which the rule condition is associated.
    </div>
    </p>
<div class="param">
     variant_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the variant the rule condition is associated with.
    </div>
    </p>
<div class="param">
     combination_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     (READ-ONLY:) The unique numeric ID of the SKU (v2 API), or Combination, with which the rule condition is associated. This is to maintain cross-compatibility between v2 and v3.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ComplexRuleConditionBase">
     ComplexRuleConditionBase
    </a>
   </h3>
   <div class="model-description">
    Common ComplexRuleCondition properties.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the rule condition; increments sequentially.
    </div>
    </p>
<div class="param">
     rule_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the rule with which the condition is associated.
    </div>
    </p>
<div class="param">
     modifier_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the modifier with which the rule condition is associated.
    </div>
    </p>
<div class="param">
     modifier_value_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the modifier value with which the rule condition is associated.
    </div>
    </p>
<div class="param">
     variant_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the variant the rule condition is associated with.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ComplexRuleConditionPost">
     ComplexRuleConditionPost
    </a>
   </h3>
   <div class="model-description">
    The model for a POST to create conditions on a complex rule.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the rule condition; increments sequentially.
    </div>
    </p>
<div class="param">
     rule_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the rule with which the condition is associated.
    </div>
    </p>
<div class="param">
     modifier_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the modifier with which the rule condition is associated.
    </div>
    </p>
<div class="param">
     modifier_value_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the modifier value with which the rule condition is associated.
    </div>
    </p>
<div class="param">
     variant_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the variant the rule condition is associated with.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ComplexRuleConditionPut">
     ComplexRuleConditionPut
    </a>
   </h3>
   <div class="model-description">
    The model for a PUT to update conditions on a complex rule.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the rule condition; increments sequentially.
    </div>
    </p>
<div class="param">
     rule_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the rule with which the condition is associated.
    </div>
    </p>
<div class="param">
     modifier_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the modifier with which the rule condition is associated.
    </div>
    </p>
<div class="param">
     modifier_value_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the modifier value with which the rule condition is associated.
    </div>
    </p>
<div class="param">
     variant_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the variant the rule condition is associated with.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ComplexRulePost">
     ComplexRulePost
    </a>
   </h3>
   <div class="model-description">
    The model for a POST to create complex rule on a product.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the product with which the rule is associated; increments sequentially.
    </div>
    </p>
<div class="param">
     sort_order (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     Priority this rule will be given, when making adjustments to the product properties.
    </div>
    </p>
<div class="param">
     enabled (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag for determining whether the rule is to be used when adjusting a product's price, weight, image, or availabilty.
    </div>
    </p>
<div class="param">
     stop (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag for determining whether other rules should not be applied after this rule has been applied.
    </div>
    </p>
<div class="param">
     purchasing_disabled (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag for determining whether the rule should disable purchasing of a product when the conditions are applied.
    </div>
    </p>
<div class="param">
     purchasing_disabled_message (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Message displayed on the storefront when a rule disables the purchasing of a product.
    </div>
    </p>
<div class="param">
     purchasing_hidden (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag for determining whether the rule should hide purchasing of a product when the conditions are applied.
    </div>
    </p>
<div class="param">
     price_adjuster (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Adjuster">
       Adjuster
      </a>
     </span>
    </div>
    </p>
<div class="param">
     weight_adjuster (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Adjuster">
       Adjuster
      </a>
     </span>
    </div>
    </p>
<div class="param">
     conditions (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#ComplexRuleConditionPost">
       array[ComplexRuleConditionPost]
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ComplexRulePut">
     ComplexRulePut
    </a>
   </h3>
   <div class="model-description">
    The model for a PUT to update complex rule on a product.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     product_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the product with which the rule is associated; increments sequentially.
    </div>
    </p>
<div class="param">
     sort_order (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     Priority this rule will be given, when making adjustments to the product properties.
    </div>
    </p>
<div class="param">
     enabled (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag for determining whether the rule is to be used when adjusting a product's price, weight, image, or availabilty.
    </div>
    </p>
<div class="param">
     stop (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag for determining whether other rules should not be applied after this rule has been applied.
    </div>
    </p>
<div class="param">
     purchasing_disabled (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag for determining whether the rule should disable purchasing of a product when the conditions are applied.
    </div>
    </p>
<div class="param">
     purchasing_disabled_message (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Message displayed on the storefront when a rule disables the purchasing of a product.
    </div>
    </p>
<div class="param">
     purchasing_hidden (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag for determining whether the rule should hide purchasing of a product when the conditions are applied.
    </div>
    </p>
<div class="param">
     price_adjuster (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Adjuster">
       Adjuster
      </a>
     </span>
    </div>
    </p>
<div class="param">
     weight_adjuster (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Adjuster">
       Adjuster
      </a>
     </span>
    </div>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the rule; increments sequentially.
    </div>
    </p>
<div class="param">
     conditions (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#ComplexRuleConditionPut">
       array[ComplexRuleConditionPut]
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ComplexRuleResponse">
     ComplexRuleResponse
    </a>
   </h3>
   <div class="model-description">
    Response payload for the Bigcommerce API.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#ComplexRule">
       ComplexRule
      </a>
     </span>
    </div>
    </p>
<div class="param">
     meta (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Meta">
       Meta
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="CustomField">
     CustomField
    </a>
   </h3>
   <div class="model-description">
    Gets custom fields associated with a product. These allow you to specify additional information that will appear on the product's page, such as a book's ISBN or a DVD's release date.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The name of the field, shown on the storefront, orders, etc.
    </div>
    </p>
<div class="param">
     value (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The name of the field, shown on the storefront, orders, etc.
    </div>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the custom field; increments sequentially.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="CustomFieldBase">
     CustomFieldBase
    </a>
   </h3>
   <div class="model-description">
    Common CustomField properties.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The name of the field, shown on the storefront, orders, etc.
    </div>
    </p>
<div class="param">
     value (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The name of the field, shown on the storefront, orders, etc.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="CustomFieldCollectionResponse">
     CustomFieldCollectionResponse
    </a>
   </h3>
   <div class="model-description">
    Response payload for the Bigcommerce API.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#CustomField">
       array[CustomField]
      </a>
     </span>
    </div>
    </p>
<div class="param">
     meta (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#CollectionMeta">
       CollectionMeta
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="CustomFieldPost">
     CustomFieldPost
    </a>
   </h3>
   <div class="model-description">
    The model for a POST to create custom field on a product.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The name of the field, shown on the storefront, orders, etc.
    </div>
    </p>
<div class="param">
     value (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The name of the field, shown on the storefront, orders, etc.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="CustomFieldPut">
     CustomFieldPut
    </a>
   </h3>
   <div class="model-description">
    The model for a PUT to update custom field on a product.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The name of the field, shown on the storefront, orders, etc.
    </div>
    </p>
<div class="param">
     value (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The name of the field, shown on the storefront, orders, etc.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="CustomFieldResponse">
     CustomFieldResponse
    </a>
   </h3>
   <div class="model-description">
    Response payload for the Bigcommerce API.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#CustomField">
       CustomField
      </a>
     </span>
    </div>
    </p>
<div class="param">
     meta (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Meta">
       Meta
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="CustomUrlCategory">
     CustomUrlCategory
    </a>
   </h3>
   <div class="model-description">
    The custom URL for the category on the storefront.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     url (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Category URL on the storefront.
    </div>
    </p>
<div class="param">
     is_customized (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Returns `true` if the URL has been changed from its default state (the auto-assigned URL that BigCommerce provides).
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="CustomUrlProduct">
     CustomUrlProduct
    </a>
   </h3>
   <div class="model-description">
    The custom URL for the product on the storefront.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     url (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Product URL on the storefront.
    </div>
    </p>
<div class="param">
     is_customized (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Returns `true` if the URL has been changed from its default state (the auto-assigned URL that BigCommerce provides).
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="DetailedErrors">
     DetailedErrors
    </a>
   </h3>
   <div class="model-description">
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ErrorResponse">
     ErrorResponse
    </a>
   </h3>
   <div class="model-description">
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     status (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The HTTP status code.
    </div>
    </p>
<div class="param">
     title (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The error title describing the particular error.
    </div>
    </p>
<div class="param">
     type (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
    </div>
    </p>
<div class="param">
     instance (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
    </div>
    </p>
<div class="param">
     errors (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#DetailedErrors">
       DetailedErrors
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ImageResponse">
     ImageResponse
    </a>
   </h3>
   <div class="model-description">
    Response payload for the Bigcommerce API.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#ResourceImage">
       ResourceImage
      </a>
     </span>
    </div>
    </p>
<div class="param">
     meta (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Meta">
       Meta
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="Meta">
     Meta
    </a>
   </h3>
   <div class="model-description">
    Empty meta object, may be used later
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="MetaFieldCollectionResponse">
     MetaFieldCollectionResponse
    </a>
   </h3>
   <div class="model-description">
    Response payload for the Bigcommerce API.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Metafield">
       array[Metafield]
      </a>
     </span>
    </div>
    </p>
<div class="param">
     meta (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#CollectionMeta">
       CollectionMeta
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="Metafield">
     Metafield
    </a>
   </h3>
   <div class="model-description">
    Allows app partners to write custom data to various resources in the API.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     permission_set (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Determines whether the field is completely private to the app that owns the field (`app_only`), or visible to other API consumers (`read`), or completely open for reading and writing to other apps (`write`).
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     app_only
    </div>
    <div class="param-enum">
     read
    </div>
    <div class="param-enum">
     write
    </div>
    </p>
<div class="param">
     namespace (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Namespace for the metafield, for organizational purposes.
    </div>
    </p>
<div class="param">
     key (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The name of the field, for example: `location_id`, `color`.
    </div>
    </p>
<div class="param">
     value (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The value of the field, for example: `1`, `blue`.
    </div>
    </p>
<div class="param">
     description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Description for the metafields.
    </div>
    </p>
<div class="param">
     resource_type (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The type of resource that the metafield is associated with.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     category
    </div>
    <div class="param-enum">
     brand
    </div>
    <div class="param-enum">
     product
    </div>
    <div class="param-enum">
     variant
    </div>
    </p>
<div class="param">
     resource_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique identifier for the resource with which the metafield is associated.
    </div>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique identifier for the metafields.
    </div>
    </p>
<div class="param">
     created_at (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#DateTime">
       Date
      </a>
     </span>
     Date and time of the metafield's creation.
 format: date-time
    </div>
    </p>
<div class="param">
     updated_at (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#DateTime">
       Date
      </a>
     </span>
     Date and time when the metafield was last updated.
 format: date-time
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="MetafieldBase">
     MetafieldBase
    </a>
   </h3>
   <div class="model-description">
    Common Metafield properties.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     permission_set (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Determines whether the field is completely private to the app that owns the field (`app_only`), or visible to other API consumers (`read`), or completely open for reading and writing to other apps (`write`).
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     app_only
    </div>
    <div class="param-enum">
     read
    </div>
    <div class="param-enum">
     write
    </div>
    </p>
<div class="param">
     namespace (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Namespace for the metafield, for organizational purposes.
    </div>
    </p>
<div class="param">
     key (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The name of the field, for example: `location_id`, `color`.
    </div>
    </p>
<div class="param">
     value (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The value of the field, for example: `1`, `blue`.
    </div>
    </p>
<div class="param">
     description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Description for the metafields.
    </div>
    </p>
<div class="param">
     resource_type (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The type of resource that the metafield is associated with.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     category
    </div>
    <div class="param-enum">
     brand
    </div>
    <div class="param-enum">
     product
    </div>
    <div class="param-enum">
     variant
    </div>
    </p>
<div class="param">
     resource_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique identifier for the resource with which the metafield is associated.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="MetafieldPost">
     MetafieldPost
    </a>
   </h3>
   <div class="model-description">
    The model for a POST to create metafield.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     permission_set (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Determines whether the field is completely private to the app that owns the field (`app_only`), or visible to other API consumers (`read`), or completely open for reading and writing to other apps (`write`).
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     app_only
    </div>
    <div class="param-enum">
     read
    </div>
    <div class="param-enum">
     write
    </div>
    </p>
<div class="param">
     namespace (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Namespace for the metafield, for organizational purposes.
    </div>
    </p>
<div class="param">
     key (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The name of the field, for example: `location_id`, `color`.
    </div>
    </p>
<div class="param">
     value (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The value of the field, for example: `1`, `blue`.
    </div>
    </p>
<div class="param">
     description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Description for the metafields.
    </div>
    </p>
<div class="param">
     resource_type (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The type of resource that the metafield is associated with.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     category
    </div>
    <div class="param-enum">
     brand
    </div>
    <div class="param-enum">
     product
    </div>
    <div class="param-enum">
     variant
    </div>
    </p>
<div class="param">
     resource_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique identifier for the resource with which the metafield is associated.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="MetafieldPut">
     MetafieldPut
    </a>
   </h3>
   <div class="model-description">
    The model for a PUT to update metafield.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     permission_set (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Determines whether the field is completely private to the app that owns the field (`app_only`), or visible to other API consumers (`read`), or completely open for reading and writing to other apps (`write`).
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     app_only
    </div>
    <div class="param-enum">
     read
    </div>
    <div class="param-enum">
     write
    </div>
    </p>
<div class="param">
     namespace (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Namespace for the metafield, for organizational purposes.
    </div>
    </p>
<div class="param">
     key (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The name of the field, for example: `location_id`, `color`.
    </div>
    </p>
<div class="param">
     value (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The value of the field, for example: `1`, `blue`.
    </div>
    </p>
<div class="param">
     description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Description for the metafields.
    </div>
    </p>
<div class="param">
     resource_type (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The type of resource that the metafield is associated with.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     category
    </div>
    <div class="param-enum">
     brand
    </div>
    <div class="param-enum">
     product
    </div>
    <div class="param-enum">
     variant
    </div>
    </p>
<div class="param">
     resource_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique identifier for the resource with which the metafield is associated.
    </div>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique identifier for the metafields.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="MetafieldResponse">
     MetafieldResponse
    </a>
   </h3>
   <div class="model-description">
    Response payload for the Bigcommerce API.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Metafield">
       Metafield
      </a>
     </span>
    </div>
    </p>
<div class="param">
     meta (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Meta">
       Meta
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="Modifier">
     Modifier
    </a>
   </h3>
   <div class="model-description">
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     type (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The type of modifier, which determines how it will display on the storefront. For reference, former v2 API values:
D = date, C = checkbox, F = file, T = text, MT = multi_line_text, N = numbers_only_text, RB = radio_buttons,
RT = rectangles, S = dropdown, P = product_list, PI = product_list_with_images, CS = swatch.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     date
    </div>
    <div class="param-enum">
     checkbox
    </div>
    <div class="param-enum">
     file
    </div>
    <div class="param-enum">
     text
    </div>
    <div class="param-enum">
     multi_line_text
    </div>
    <div class="param-enum">
     numbers_only_text
    </div>
    <div class="param-enum">
     radio_buttons
    </div>
    <div class="param-enum">
     rectangles
    </div>
    <div class="param-enum">
     dropdown
    </div>
    <div class="param-enum">
     product_list
    </div>
    <div class="param-enum">
     product_list_with_images
    </div>
    <div class="param-enum">
     swatch
    </div>
    </p>
<div class="param">
     required (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Whether this modifer is required or not at checkout
    </div>
    </p>
<div class="param">
     config (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#OptionConfig">
       OptionConfig
      </a>
     </span>
    </div>
    </p>
<div class="param">
     option_values (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#ModifierValue">
       array[ModifierValue]
      </a>
     </span>
    </div>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the modifier; increments sequentially.
    </div>
    </p>
<div class="param">
     product_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the product to which the option belongs.
    </div>
    </p>
<div class="param">
     name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The unique option name. Auto-generated from the display name, a timestamp, and the product ID.
    </div>
    </p>
<div class="param">
     display_name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The name of the option shown on the storefront.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ModifierBase">
     ModifierBase
    </a>
   </h3>
   <div class="model-description">
    Common Modifier properties.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     type (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The type of modifier, which determines how it will display on the storefront. For reference, former v2 API values:
D = date, C = checkbox, F = file, T = text, MT = multi_line_text, N = numbers_only_text, RB = radio_buttons,
RT = rectangles, S = dropdown, P = product_list, PI = product_list_with_images, CS = swatch.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     date
    </div>
    <div class="param-enum">
     checkbox
    </div>
    <div class="param-enum">
     file
    </div>
    <div class="param-enum">
     text
    </div>
    <div class="param-enum">
     multi_line_text
    </div>
    <div class="param-enum">
     numbers_only_text
    </div>
    <div class="param-enum">
     radio_buttons
    </div>
    <div class="param-enum">
     rectangles
    </div>
    <div class="param-enum">
     dropdown
    </div>
    <div class="param-enum">
     product_list
    </div>
    <div class="param-enum">
     product_list_with_images
    </div>
    <div class="param-enum">
     swatch
    </div>
    </p>
<div class="param">
     required (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Whether this modifer is required or not at checkout
    </div>
    </p>
<div class="param">
     config (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#OptionConfig">
       OptionConfig
      </a>
     </span>
    </div>
    </p>
<div class="param">
     option_values (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#ModifierValue">
       array[ModifierValue]
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ModifierCollectionResponse">
     ModifierCollectionResponse
    </a>
   </h3>
   <div class="model-description">
    Response payload for the Bigcommerce API.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Modifier">
       array[Modifier]
      </a>
     </span>
    </div>
    </p>
<div class="param">
     meta (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#CollectionMeta">
       CollectionMeta
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ModifierPost">
     ModifierPost
    </a>
   </h3>
   <div class="model-description">
    The model for a POST to create modifier on a product.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     type (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The type of modifier, which determines how it will display on the storefront. For reference, former v2 API values:
D = date, C = checkbox, F = file, T = text, MT = multi_line_text, N = numbers_only_text, RB = radio_buttons,
RT = rectangles, S = dropdown, P = product_list, PI = product_list_with_images, CS = swatch.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     date
    </div>
    <div class="param-enum">
     checkbox
    </div>
    <div class="param-enum">
     file
    </div>
    <div class="param-enum">
     text
    </div>
    <div class="param-enum">
     multi_line_text
    </div>
    <div class="param-enum">
     numbers_only_text
    </div>
    <div class="param-enum">
     radio_buttons
    </div>
    <div class="param-enum">
     rectangles
    </div>
    <div class="param-enum">
     dropdown
    </div>
    <div class="param-enum">
     product_list
    </div>
    <div class="param-enum">
     product_list_with_images
    </div>
    <div class="param-enum">
     swatch
    </div>
    </p>
<div class="param">
     required (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Whether this modifer is required or not at checkout
    </div>
    </p>
<div class="param">
     config (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#OptionConfig">
       OptionConfig
      </a>
     </span>
    </div>
    </p>
<div class="param">
     option_values (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#ModifierValue">
       array[ModifierValue]
      </a>
     </span>
    </div>
    </p>
<div class="param">
     display_name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The name of the option shown on the storefront.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ModifierPut">
     ModifierPut
    </a>
   </h3>
   <div class="model-description">
    The model for a PUT to update modifier on a product.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     type (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The type of modifier, which determines how it will display on the storefront. For reference, former v2 API values:
D = date, C = checkbox, F = file, T = text, MT = multi_line_text, N = numbers_only_text, RB = radio_buttons,
RT = rectangles, S = dropdown, P = product_list, PI = product_list_with_images, CS = swatch.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     date
    </div>
    <div class="param-enum">
     checkbox
    </div>
    <div class="param-enum">
     file
    </div>
    <div class="param-enum">
     text
    </div>
    <div class="param-enum">
     multi_line_text
    </div>
    <div class="param-enum">
     numbers_only_text
    </div>
    <div class="param-enum">
     radio_buttons
    </div>
    <div class="param-enum">
     rectangles
    </div>
    <div class="param-enum">
     dropdown
    </div>
    <div class="param-enum">
     product_list
    </div>
    <div class="param-enum">
     product_list_with_images
    </div>
    <div class="param-enum">
     swatch
    </div>
    </p>
<div class="param">
     required (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Whether this modifer is required or not at checkout
    </div>
    </p>
<div class="param">
     config (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#OptionConfig">
       OptionConfig
      </a>
     </span>
    </div>
    </p>
<div class="param">
     option_values (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#ModifierValue">
       array[ModifierValue]
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ModifierResponse">
     ModifierResponse
    </a>
   </h3>
   <div class="model-description">
    Response payload for the Bigcommerce API.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Modifier">
       Modifier
      </a>
     </span>
    </div>
    </p>
<div class="param">
     meta (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Meta">
       Meta
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ModifierValue">
     ModifierValue
    </a>
   </h3>
   <div class="model-description">
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     is_default (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     The flag for preselecting a value as the default on the storefront. This field is not supported for swatch options/modifiers.
    </div>
    </p>
<div class="param">
     label (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The text display identifying the value on the storefront.
    </div>
    </p>
<div class="param">
     sort_order (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The order in which the value will be displayed on the product page.
    </div>
    </p>
<div class="param">
     value_data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#object">
       Object
      </a>
     </span>
     Extra data describing the value, based on the type of option or modifier with which the value is associated. `swatch` requires an array of colors, with up to three hexidecimal color keys; `product list` requires a `product_id`; and `checkbox` requires a boolean flag, called `checked_value`, to determine which value is considered to be the checked state.
    </div>
    </p>
<div class="param">
     adjusters (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#ModifierValueBase_adjusters">
       ModifierValueBase_adjusters
      </a>
     </span>
    </div>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the value; increments sequentially.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ModifierValueBase">
     ModifierValueBase
    </a>
   </h3>
   <div class="model-description">
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     is_default (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     The flag for preselecting a value as the default on the storefront. This field is not supported for swatch options/modifiers.
    </div>
    </p>
<div class="param">
     label (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The text display identifying the value on the storefront.
    </div>
    </p>
<div class="param">
     sort_order (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The order in which the value will be displayed on the product page.
    </div>
    </p>
<div class="param">
     value_data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#object">
       Object
      </a>
     </span>
     Extra data describing the value, based on the type of option or modifier with which the value is associated. `swatch` requires an array of colors, with up to three hexidecimal color keys; `product list` requires a `product_id`; and `checkbox` requires a boolean flag, called `checked_value`, to determine which value is considered to be the checked state.
    </div>
    </p>
<div class="param">
     adjusters (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#ModifierValueBase_adjusters">
       ModifierValueBase_adjusters
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ModifierValueBase_adjusters">
     ModifierValueBase_adjusters
    </a>
   </h3>
   <div class="model-description">
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Adjuster">
       Adjuster
      </a>
     </span>
    </div>
    </p>
<div class="param">
     weight (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Adjuster">
       Adjuster
      </a>
     </span>
    </div>
    </p>
<div class="param">
     image_url (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The URL for an image displayed on the storefront when the modifier value is selected.
    </div>
    </p>
<div class="param">
     purchasing_disabled (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#ModifierValueBase_adjusters_purchasing_disabled">
       ModifierValueBase_adjusters_purchasing_disabled
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ModifierValueBase_adjusters_purchasing_disabled">
     ModifierValueBase_adjusters_purchasing_disabled
    </a>
   </h3>
   <div class="model-description">
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     status (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag for whether the modifier value disables purchasing when selected on the storefront. This can be used for temporarily disabling a particular modifier value.
    </div>
    </p>
<div class="param">
     message (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The message displayed on the storefront when the purchasing disabled status is `true`.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ModifierValueCollectionResponse">
     ModifierValueCollectionResponse
    </a>
   </h3>
   <div class="model-description">
    Response payload for the Bigcommerce API.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#ModifierValue">
       array[ModifierValue]
      </a>
     </span>
    </div>
    </p>
<div class="param">
     meta (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#CollectionMeta">
       CollectionMeta
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ModifierValuePost">
     ModifierValuePost
    </a>
   </h3>
   <div class="model-description">
    The model for a POST to create modifier value on a product.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     is_default (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     The flag for preselecting a value as the default on the storefront. This field is not supported for swatch options/modifiers.
    </div>
    </p>
<div class="param">
     label (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The text display identifying the value on the storefront.
    </div>
    </p>
<div class="param">
     sort_order (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The order in which the value will be displayed on the product page.
    </div>
    </p>
<div class="param">
     value_data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#object">
       Object
      </a>
     </span>
     Extra data describing the value, based on the type of option or modifier with which the value is associated. `swatch` requires an array of colors, with up to three hexidecimal color keys; `product list` requires a `product_id`; and `checkbox` requires a boolean flag, called `checked_value`, to determine which value is considered to be the checked state.
    </div>
    </p>
<div class="param">
     adjusters (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#ModifierValueBase_adjusters">
       ModifierValueBase_adjusters
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ModifierValuePut">
     ModifierValuePut
    </a>
   </h3>
   <div class="model-description">
    The model for a PUT to update modifier value on a product.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     is_default (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     The flag for preselecting a value as the default on the storefront. This field is not supported for swatch options/modifiers.
    </div>
    </p>
<div class="param">
     label (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The text display identifying the value on the storefront.
    </div>
    </p>
<div class="param">
     sort_order (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The order in which the value will be displayed on the product page.
    </div>
    </p>
<div class="param">
     value_data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#object">
       Object
      </a>
     </span>
     Extra data describing the value, based on the type of option or modifier with which the value is associated. `swatch` requires an array of colors, with up to three hexidecimal color keys; `product list` requires a `product_id`; and `checkbox` requires a boolean flag, called `checked_value`, to determine which value is considered to be the checked state.
    </div>
    </p>
<div class="param">
     adjusters (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#ModifierValueBase_adjusters">
       ModifierValueBase_adjusters
      </a>
     </span>
    </div>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the value; increments sequentially.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ModifierValueResponse">
     ModifierValueResponse
    </a>
   </h3>
   <div class="model-description">
    Response payload for the Bigcommerce API.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#ModifierValue">
       ModifierValue
      </a>
     </span>
    </div>
    </p>
<div class="param">
     meta (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Meta">
       Meta
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="NotFound">
     NotFound
    </a>
   </h3>
   <div class="model-description">
    Error payload for the BigCommerce API.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     status (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     404 HTTP status code.
    </div>
    </p>
<div class="param">
     title (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The error title describing the particular error.
    </div>
    </p>
<div class="param">
     type (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
    </div>
    </p>
<div class="param">
     instance (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="Option">
     Option
    </a>
   </h3>
   <div class="model-description">
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numerical ID of the option, increments sequentially.
    </div>
    </p>
<div class="param">
     product_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numerical ID of the product that the option belongs to.
    </div>
    </p>
<div class="param">
     display_name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The name of the option shown on the storefront.
    </div>
    </p>
<div class="param">
     type (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The type of option, which determines how it will display on the storefront. For reference, the former v2 API values are:
RB = radio_buttons, RT = rectangles, S = dropdown, P = product_list, PI = product_list_with_images, CS = swatch.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     radio_buttons
    </div>
    <div class="param-enum">
     rectangles
    </div>
    <div class="param-enum">
     dropdown
    </div>
    <div class="param-enum">
     product_list
    </div>
    <div class="param-enum">
     product_list_with_images
    </div>
    <div class="param-enum">
     swatch
    </div>
    </p>
<div class="param">
     config (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#OptionConfig">
       OptionConfig
      </a>
     </span>
    </div>
    </p>
<div class="param">
     option_values (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#OptionValue">
       array[OptionValue]
      </a>
     </span>
    </div>
    </p>
<div class="param">
     name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The name unique option name auto-generated from the display name, a timestamp, and the product id.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="OptionBase">
     OptionBase
    </a>
   </h3>
   <div class="model-description">
    Common Option properties.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numerical ID of the option, increments sequentially.
    </div>
    </p>
<div class="param">
     product_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numerical ID of the product that the option belongs to.
    </div>
    </p>
<div class="param">
     display_name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The name of the option shown on the storefront.
    </div>
    </p>
<div class="param">
     type (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The type of option, which determines how it will display on the storefront. For reference, the former v2 API values are:
RB = radio_buttons, RT = rectangles, S = dropdown, P = product_list, PI = product_list_with_images, CS = swatch.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     radio_buttons
    </div>
    <div class="param-enum">
     rectangles
    </div>
    <div class="param-enum">
     dropdown
    </div>
    <div class="param-enum">
     product_list
    </div>
    <div class="param-enum">
     product_list_with_images
    </div>
    <div class="param-enum">
     swatch
    </div>
    </p>
<div class="param">
     config (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#OptionConfig">
       OptionConfig
      </a>
     </span>
    </div>
    </p>
<div class="param">
     option_values (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#OptionValue">
       array[OptionValue]
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="OptionCollectionResponse">
     OptionCollectionResponse
    </a>
   </h3>
   <div class="model-description">
    Response payload for the Bigcommerce API.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Option">
       array[Option]
      </a>
     </span>
    </div>
    </p>
<div class="param">
     meta (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#CollectionMeta">
       CollectionMeta
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="OptionConfig">
     OptionConfig
    </a>
   </h3>
   <div class="model-description">
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     default_value (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     (date, text, multi_line_text, numbers_only_text) The default value. Shown on a date option as an ISO-8601–formatted string, or on a text option as a string.
    </div>
    </p>
<div class="param">
     checked_by_default (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     (checkbox) Flag for setting the checkbox to be checked by default.
    </div>
    </p>
<div class="param">
     checkbox_label (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     (checkbox) Label displayed for the checkbox option.
    </div>
    </p>
<div class="param">
     date_limited (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     (date) Flag to limit the dates allowed to be entered on a date option.
    </div>
    </p>
<div class="param">
     date_limit_mode (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     (date) The type of limit that is allowed to be entered on a date option.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     earliest
    </div>
    <div class="param-enum">
     range
    </div>
    <div class="param-enum">
     latest
    </div>
    </p>
<div class="param">
     date_earliest_value (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#date">
       date
      </a>
     </span>
     (date) The earliest date allowed to be entered on the date option, as an ISO-8601 formatted string.
 format: date
    </div>
    </p>
<div class="param">
     date_latest_value (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#date">
       date
      </a>
     </span>
     (date) The latest date allowed to be entered on the date option, as an ISO-8601 formatted string.
 format: date
    </div>
    </p>
<div class="param">
     file_types_mode (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     (file) The kind of restriction on the file types that can be uploaded with a file upload option. Values: `specific` - restricts uploads to particular file types; `all` - allows all file types.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     specific
    </div>
    <div class="param-enum">
     all
    </div>
    </p>
<div class="param">
     file_types_supported (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       array[String]
      </a>
     </span>
     (file) The type of files allowed to be uploaded if the `file_type_option` is set to `specific`. Values:
  `images` - Allows upload of image MIME types (`bmp`,`gif`,`jpg`,`jpeg`,`jpe`,`jif`,`jfif`,`jfi`,`png`,`wbmp`,`xbm`,`tiff`).
  `documents` - Allows upload of document MIME types (`txt`,`pdf`,`rtf`,`doc`,`docx`,`xls`,`xlsx`,`accdb`,`mdb`,`one`,`pps`,`ppsx`,`ppt`,`pptx`,`pub`,`odt`,`ods`,`odp`,`odg`,`odf`).
  `other` - Allows file types defined in the `file_types_other` array.
    </div>
    </p>
<div class="param">
     file_types_other (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       array[String]
      </a>
     </span>
     (file) A list of other file types allowed with the file upload option.
    </div>
    </p>
<div class="param">
     file_max_size (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     (file) The maximum size for a file that can be used with the file upload option.
    </div>
    </p>
<div class="param">
     text_characters_limited (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     (text, multi_line_text) Flag to validate the length of a text or multi-line text input.
    </div>
    </p>
<div class="param">
     text_min_length (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     (text, multi_line_text) The minimum length allowed for a text or multi-line text option.
    </div>
    </p>
<div class="param">
     text_max_length (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     (text, multi_line_text) The maximum length allowed for a text or multi line text option.
    </div>
    </p>
<div class="param">
     text_lines_limited (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     (multi_line_text) Flag to validate the maximum number of lines allowed on a multi-line text input.
    </div>
    </p>
<div class="param">
     text_max_lines (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     (multi_line_text) The maximum number of lines allowed on a multi-line text input.
    </div>
    </p>
<div class="param">
     number_limited (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     (numbers_only_text) Flag to limit the value of a number option.
    </div>
    </p>
<div class="param">
     number_limit_mode (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     (numbers_only_text) The type of limit on values entered for a number option.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     lowest
    </div>
    <div class="param-enum">
     highest
    </div>
    <div class="param-enum">
     range
    </div>
    </p>
<div class="param">
     number_lowest_value (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#number">
       BigDecimal
      </a>
     </span>
     (numbers_only_text) The lowest allowed value for a number option if `number_limited` is true.
    </div>
    </p>
<div class="param">
     number_highest_value (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#number">
       BigDecimal
      </a>
     </span>
     (numbers_only_text) The highest allowed value for a number option if `number_limited` is true.
    </div>
    </p>
<div class="param">
     number_integers_only (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     (numbers_only_text) Flag to limit the input on a number option to whole numbers only.
    </div>
    </p>
<div class="param">
     product_list_adjusts_inventory (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     (product_list, product_list_with_images) Flag for automatically adjusting inventory on a product included in the list.
    </div>
    </p>
<div class="param">
     product_list_adjusts_pricing (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     (product_list, product_list_with_images) Flag to add the optional product's price to the main product's price.
    </div>
    </p>
<div class="param">
     product_list_shipping_calc (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     (product_list, product_list_with_images) How to factor the optional product's weight and package dimensions into the shipping quote. Values: `none` - don't adjust; `weight` - use shipping weight only; `package` - use weight and dimensions.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     none
    </div>
    <div class="param-enum">
     weight
    </div>
    <div class="param-enum">
     package
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="OptionPost">
     OptionPost
    </a>
   </h3>
   <div class="model-description">
    The model for a POST to create option on a product.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numerical ID of the option, increments sequentially.
    </div>
    </p>
<div class="param">
     product_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numerical ID of the product that the option belongs to.
    </div>
    </p>
<div class="param">
     display_name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The name of the option shown on the storefront.
    </div>
    </p>
<div class="param">
     type (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The type of option, which determines how it will display on the storefront. For reference, the former v2 API values are:
RB = radio_buttons, RT = rectangles, S = dropdown, P = product_list, PI = product_list_with_images, CS = swatch.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     radio_buttons
    </div>
    <div class="param-enum">
     rectangles
    </div>
    <div class="param-enum">
     dropdown
    </div>
    <div class="param-enum">
     product_list
    </div>
    <div class="param-enum">
     product_list_with_images
    </div>
    <div class="param-enum">
     swatch
    </div>
    </p>
<div class="param">
     config (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#OptionConfig">
       OptionConfig
      </a>
     </span>
    </div>
    </p>
<div class="param">
     option_values (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#OptionValue">
       array[OptionValue]
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="OptionPut">
     OptionPut
    </a>
   </h3>
   <div class="model-description">
    The model for a PUT to update option on a product.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numerical ID of the option, increments sequentially.
    </div>
    </p>
<div class="param">
     product_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numerical ID of the product that the option belongs to.
    </div>
    </p>
<div class="param">
     display_name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The name of the option shown on the storefront.
    </div>
    </p>
<div class="param">
     type (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The type of option, which determines how it will display on the storefront. For reference, the former v2 API values are:
RB = radio_buttons, RT = rectangles, S = dropdown, P = product_list, PI = product_list_with_images, CS = swatch.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     radio_buttons
    </div>
    <div class="param-enum">
     rectangles
    </div>
    <div class="param-enum">
     dropdown
    </div>
    <div class="param-enum">
     product_list
    </div>
    <div class="param-enum">
     product_list_with_images
    </div>
    <div class="param-enum">
     swatch
    </div>
    </p>
<div class="param">
     config (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#OptionConfig">
       OptionConfig
      </a>
     </span>
    </div>
    </p>
<div class="param">
     option_values (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#OptionValue">
       array[OptionValue]
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="OptionResponse">
     OptionResponse
    </a>
   </h3>
   <div class="model-description">
    Response payload for the Bigcommerce API.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Option">
       Option
      </a>
     </span>
    </div>
    </p>
<div class="param">
     meta (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Meta">
       Meta
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="OptionValue">
     OptionValue
    </a>
   </h3>
   <div class="model-description">
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     is_default (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     The flag for preselecting a value as the default on the storefront. This field is not supported for swatch options/modifiers.
    </div>
    </p>
<div class="param">
     label (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The text display identifying the value on the storefront.
    </div>
    </p>
<div class="param">
     sort_order (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The order in which the value will be displayed on the product page.
    </div>
    </p>
<div class="param">
     value_data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#object">
       Object
      </a>
     </span>
     Extra data describing the value, based on the type of option or modifier with which the value is associated. `swatch` requires an array of colors, with up to three hexidecimal color keys; `product list` requires a `product_id`; and `checkbox` requires a boolean flag, called `checked_value`, to determine which value is considered to be the checked state.
    </div>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the value; increments sequentially.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="OptionValueBase">
     OptionValueBase
    </a>
   </h3>
   <div class="model-description">
    Common OptionValue properties.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     is_default (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     The flag for preselecting a value as the default on the storefront. This field is not supported for swatch options/modifiers.
    </div>
    </p>
<div class="param">
     label (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The text display identifying the value on the storefront.
    </div>
    </p>
<div class="param">
     sort_order (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The order in which the value will be displayed on the product page.
    </div>
    </p>
<div class="param">
     value_data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#object">
       Object
      </a>
     </span>
     Extra data describing the value, based on the type of option or modifier with which the value is associated. `swatch` requires an array of colors, with up to three hexidecimal color keys; `product list` requires a `product_id`; and `checkbox` requires a boolean flag, called `checked_value`, to determine which value is considered to be the checked state.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="OptionValueCollectionResponse">
     OptionValueCollectionResponse
    </a>
   </h3>
   <div class="model-description">
    Response payload for the Bigcommerce API.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#OptionValue">
       array[OptionValue]
      </a>
     </span>
    </div>
    </p>
<div class="param">
     meta (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#CollectionMeta">
       CollectionMeta
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="OptionValuePost">
     OptionValuePost
    </a>
   </h3>
   <div class="model-description">
    The model for a POST to create option value on a product.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     is_default (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     The flag for preselecting a value as the default on the storefront. This field is not supported for swatch options/modifiers.
    </div>
    </p>
<div class="param">
     label (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The text display identifying the value on the storefront.
    </div>
    </p>
<div class="param">
     sort_order (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The order in which the value will be displayed on the product page.
    </div>
    </p>
<div class="param">
     value_data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#object">
       Object
      </a>
     </span>
     Extra data describing the value, based on the type of option or modifier with which the value is associated. `swatch` requires an array of colors, with up to three hexidecimal color keys; `product list` requires a `product_id`; and `checkbox` requires a boolean flag, called `checked_value`, to determine which value is considered to be the checked state.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="OptionValueProductBase">
     OptionValueProductBase
    </a>
   </h3>
   <div class="model-description">
    Common OptionValueProduct properties.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     option_display_name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The name of the option.
    </div>
    </p>
<div class="param">
     label (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The label of the option value.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="OptionValueProductPost">
     OptionValueProductPost
    </a>
   </h3>
   <div class="model-description">
    The model for a POST to create option values with a product.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     option_display_name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The name of the option.
    </div>
    </p>
<div class="param">
     label (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The label of the option value.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="OptionValuePut">
     OptionValuePut
    </a>
   </h3>
   <div class="model-description">
    The model for a PUT to update option value on a product.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     is_default (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     The flag for preselecting a value as the default on the storefront. This field is not supported for swatch options/modifiers.
    </div>
    </p>
<div class="param">
     label (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The text display identifying the value on the storefront.
    </div>
    </p>
<div class="param">
     sort_order (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The order in which the value will be displayed on the product page.
    </div>
    </p>
<div class="param">
     value_data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#object">
       Object
      </a>
     </span>
     Extra data describing the value, based on the type of option or modifier with which the value is associated. `swatch` requires an array of colors, with up to three hexidecimal color keys; `product list` requires a `product_id`; and `checkbox` requires a boolean flag, called `checked_value`, to determine which value is considered to be the checked state.
    </div>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the value; increments sequentially.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="OptionValueResponse">
     OptionValueResponse
    </a>
   </h3>
   <div class="model-description">
    Response payload for the Bigcommerce API.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#OptionValue">
       OptionValue
      </a>
     </span>
    </div>
    </p>
<div class="param">
     meta (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Meta">
       Meta
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="OptionValueVariant">
     OptionValueVariant
    </a>
   </h3>
   <div class="model-description">
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     option_display_name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The name of the option.
    </div>
    </p>
<div class="param">
     label (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The label of the option value.
    </div>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
    </div>
    </p>
<div class="param">
     option_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="OptionValueVariantPost">
     OptionValueVariantPost
    </a>
   </h3>
   <div class="model-description">
    The model for a POST to create option values with a variant.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
    </div>
    </p>
<div class="param">
     option_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="Pagination">
     Pagination
    </a>
   </h3>
   <div class="model-description">
    Data about the response, including pagination and collection totals.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     total (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     Total number of items in the result set.
    </div>
    </p>
<div class="param">
     count (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     Total number of items in the collection response.
    </div>
    </p>
<div class="param">
     per_page (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The amount of items returned in the collection per page, controlled by the limit parameter.
    </div>
    </p>
<div class="param">
     current_page (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The page you are currently on within the collection.
    </div>
    </p>
<div class="param">
     total_pages (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The total number of pages in the collection.
    </div>
    </p>
<div class="param">
     links (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Pagination_links">
       Pagination_links
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="Pagination_links">
     Pagination_links
    </a>
   </h3>
   <div class="model-description">
    Pagination links for the previous and next parts of the whole collection.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     previous (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Link to the previous page returned in the response.
    </div>
    </p>
<div class="param">
     current (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Link to the current page returned in the response.
    </div>
    </p>
<div class="param">
     next (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Link to the next page returned in the response.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="Product">
     Product
    </a>
   </h3>
   <div class="model-description">
    A BigCommerce Product object describes a single purchasable unit or a collection of purchasable units.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The product name.
    </div>
    </p>
<div class="param">
     type (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The product type: physical - a physical stock unit, digital - a digital download
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     physical
    </div>
    <div class="param-enum">
     digital
    </div>
    </p>
<div class="param">
     sku (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     User defined product code/stock keeping unit (SKU).
    </div>
    </p>
<div class="param">
     description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The product description, which can include HTML formatting.
    </div>
    </p>
<div class="param">
     weight (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     Weight of the product, which can be used when calculating shipping costs.
 format: double
    </div>
    </p>
<div class="param">
     width (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     Width of the product, which can be used when calculating shipping costs.
 format: double
    </div>
    </p>
<div class="param">
     depth (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     Depth of the product, which can be used when calculating shipping costs.
 format: double
    </div>
    </p>
<div class="param">
     height (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     Height of the product, which can be used when calculating shipping costs.
 format: double
    </div>
    </p>
<div class="param">
     price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     The price of the product. The price should include or exclude tax, based on the store settings.
 format: double
    </div>
    </p>
<div class="param">
     cost_price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     The cost price of the product. Stored for reference only; it is not used or displayed anywhere on the store.
 format: double
    </div>
    </p>
<div class="param">
     retail_price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     The retail cost of the product. If entered, the retail cost price will be shown on the product page.
 format: double
    </div>
    </p>
<div class="param">
     sale_price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     If entered, the sale price will be used instead of value in the price field when calculating the product's cost.
 format: double
    </div>
    </p>
<div class="param">
     tax_class_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The ID of the tax class applied to the product. (NOTE: Value ignored if automatic tax is enabled.)
    </div>
    </p>
<div class="param">
     product_tax_code (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Accepts AvaTax System Tax Codes, which identify products and services that fall into special sales-tax categories. By using these codes, merchants who subscribe to Avalara Premium can calculate sales taxes more accurately. Stores without Avalara Premium will ignore the code when calculating sales tax. Do not pass more than one code. The codes are case-sensitive. For details, please see Avalara's documentation.
    </div>
    </p>
<div class="param">
     categories (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       array[Integer]
      </a>
     </span>
     An array of IDs for the categories to which this product belongs. When updating a product, if an array of categories is supplied, all product categories will be overwritten. Does not accept more than 1,000 ID values.
    </div>
    </p>
<div class="param">
     brand_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The ID associated with the product's brand.
    </div>
    </p>
<div class="param">
     inventory_level (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     Current inventory level of the product. Simple inventory tracking must be enabled (See the inventory_tracking field) for this to take any effect.
    </div>
    </p>
<div class="param">
     inventory_warning_level (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     Inventory Warning level for the product. When the product's inventory level drops below the warning level, the store owner will be informed. Simple inventory tracking must be enabled (see the `inventory_tracking` field) for this to take any effect.
    </div>
    </p>
<div class="param">
     inventory_tracking (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The type of inventory tracking for the product. Values are: none - inventory levels will not be tracked; product - inventory levels will be tracked using the `inventory_level` and `inventory_warning_level` fields; variant - inventory levels will be tracked based on variants, which maintain their own warning levels and inventory levels.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     none
    </div>
    <div class="param-enum">
     product
    </div>
    <div class="param-enum">
     variant
    </div>
    </p>
<div class="param">
     fixed_cost_shipping_price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     A fixed shipping cost for the product. If defined, this value will be used during checkout instead of normal shipping-cost calculation.
 format: double
    </div>
    </p>
<div class="param">
     is_free_shipping (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag used to indicate whether the product has free shipping. If `true`, the shipping cost for the product will be zero.
    </div>
    </p>
<div class="param">
     is_visible (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag to determine whether the product should be displayed to customers browsing the store. If `true`, the product will be displayed. If `false`, the product will be hidden from view.
    </div>
    </p>
<div class="param">
     is_featured (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag to determine whether the product should be included in the `featured products` panel when viewing the store.
    </div>
    </p>
<div class="param">
     related_products (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       array[Integer]
      </a>
     </span>
     An array of IDs for the related products.
    </div>
    </p>
<div class="param">
     warranty (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Warranty information displayed on the product page. Can include HTML formatting.
    </div>
    </p>
<div class="param">
     bin_picking_number (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The BIN picking number for the product.
    </div>
    </p>
<div class="param">
     layout_file (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The layout template file used to render this product.
    </div>
    </p>
<div class="param">
     upc (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The product UPC code, which is used in feeds for shopping comparison sites and external channel integrations.
    </div>
    </p>
<div class="param">
     search_keywords (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     A comma-separated list of keywords that can be used to locate the product when searching the store.
    </div>
    </p>
<div class="param">
     availability (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Availability of the product. Availability options are: available - the product can be purchased in the storefront; disabled - the product is listed in the storefront, but cannot be purchased; preorder - the product is listed for pre-orders.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     available
    </div>
    <div class="param-enum">
     disabled
    </div>
    <div class="param-enum">
     preorder
    </div>
    </p>
<div class="param">
     availability_description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Availability text displayed on the checkout page, under the product title. Tells the customer how long it will normally take to ship this product, such as 'Usually ships in 24 hours.'
    </div>
    </p>
<div class="param">
     gift_wrapping_options_type (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Type of gift-wrapping options. Values: `any` - allow any gift-wrapping options in the store; `none` - disallow gift-wrapping on the product; `list` – provide a list of IDs in `gift_wrapping_options_list` field.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     any
    </div>
    <div class="param-enum">
     none
    </div>
    <div class="param-enum">
     list
    </div>
    </p>
<div class="param">
     gift_wrapping_options_list (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       array[Integer]
      </a>
     </span>
     A list of gift-wrapping option IDs.
    </div>
    </p>
<div class="param">
     sort_order (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     Priority to give this product when included in product lists on category pages and in search results. Lower integers will place the product closer to the top of the results.
    </div>
    </p>
<div class="param">
     condition (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The product condition. Will be shown on the product page if the `is_condition_shown` field's value is `true`. Possible values: `New`, `Used`, `Refurbished`.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     New
    </div>
    <div class="param-enum">
     Used
    </div>
    <div class="param-enum">
     Refurbished
    </div>
    </p>
<div class="param">
     is_condition_shown (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag used to determine whether the product condition is shown to the customer on the product page.
    </div>
    </p>
<div class="param">
     order_quantity_minimum (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The minimum quantity an order must contain, to be eligible to purchase this product.
    </div>
    </p>
<div class="param">
     order_quantity_maximum (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The maximum quantity an order can contain when purchasing the product.
    </div>
    </p>
<div class="param">
     page_title (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Custom title for the product page. If not defined, the product name will be used as the meta title.
    </div>
    </p>
<div class="param">
     meta_keywords (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       array[String]
      </a>
     </span>
     Custom meta keywords for the product page. If not defined, the store's default keywords will be used.
    </div>
    </p>
<div class="param">
     meta_description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Custom meta description for the product page. If not defined, the store's default meta description will be used.
    </div>
    </p>
<div class="param">
     view_count (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The number of times the product has been viewed.
    </div>
    </p>
<div class="param">
     preorder_release_date (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#DateTime">
       Date
      </a>
     </span>
     Pre-order release date. See the `availability` field for details on setting a product's availability to accept pre-orders.
 format: date-time
    </div>
    </p>
<div class="param">
     preorder_message (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Custom expected-date message to display on the product page. If undefined, the message defaults to the storewide setting. Can contain the `%%DATE%%` placeholder, which will be substituted for the release date.
    </div>
    </p>
<div class="param">
     is_preorder_only (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     If set to `false`, the product will not change its availability from 	`preorder` to `available` on the release date. Otherwise, on the release date the product's availability/status will change to `available`.
    </div>
    </p>
<div class="param">
     is_price_hidden (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     False by default, indicating that this product's price should be shown on the product page. If set to `true`, the price is hidden. (NOTE: To successfully set `is_price_hidden` to `true`, the `availability` value must be `disabled`.)
    </div>
    </p>
<div class="param">
     price_hidden_label (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     By default, an empty string. If `is_price_hidden` is `true`, the value of `price_hidden_label` is displayed instead of the price. (NOTE: To successfully set a non-empty string value with `is_price_hidden` set to `true`, the `availability` value must be `disabled`.)
    </div>
    </p>
<div class="param">
     custom_url (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#CustomUrlProduct">
       CustomUrlProduct
      </a>
     </span>
    </div>
    </p>
<div class="param">
     bulk_pricing_rules (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#BulkPricingRule">
       array[BulkPricingRule]
      </a>
     </span>
    </div>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the product; increments sequentially.
    </div>
    </p>
<div class="param">
     calculated_price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     The price of the product, unless a `sale_price` is set.
 format: double
    </div>
    </p>
<div class="param">
     custom_fields (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#CustomField">
       array[CustomField]
      </a>
     </span>
    </div>
    </p>
<div class="param">
     date_created (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#DateTime">
       Date
      </a>
     </span>
     The date on which the product was created.
 format: date-time
    </div>
    </p>
<div class="param">
     date_modified (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#DateTime">
       Date
      </a>
     </span>
     The date on which the product was modified.
 format: date-time
    </div>
    </p>
<div class="param">
     images (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#ProductImage">
       array[ProductImage]
      </a>
     </span>
    </div>
    </p>
<div class="param">
     videos (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#ProductVideo">
       array[ProductVideo]
      </a>
     </span>
    </div>
    </p>
<div class="param">
     variants (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Variant">
       array[Variant]
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ProductBase">
     ProductBase
    </a>
   </h3>
   <div class="model-description">
    Common Product properties.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The product name.
    </div>
    </p>
<div class="param">
     type (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The product type: physical - a physical stock unit, digital - a digital download
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     physical
    </div>
    <div class="param-enum">
     digital
    </div>
    </p>
<div class="param">
     sku (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     User defined product code/stock keeping unit (SKU).
    </div>
    </p>
<div class="param">
     description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The product description, which can include HTML formatting.
    </div>
    </p>
<div class="param">
     weight (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     Weight of the product, which can be used when calculating shipping costs.
 format: double
    </div>
    </p>
<div class="param">
     width (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     Width of the product, which can be used when calculating shipping costs.
 format: double
    </div>
    </p>
<div class="param">
     depth (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     Depth of the product, which can be used when calculating shipping costs.
 format: double
    </div>
    </p>
<div class="param">
     height (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     Height of the product, which can be used when calculating shipping costs.
 format: double
    </div>
    </p>
<div class="param">
     price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     The price of the product. The price should include or exclude tax, based on the store settings.
 format: double
    </div>
    </p>
<div class="param">
     cost_price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     The cost price of the product. Stored for reference only; it is not used or displayed anywhere on the store.
 format: double
    </div>
    </p>
<div class="param">
     retail_price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     The retail cost of the product. If entered, the retail cost price will be shown on the product page.
 format: double
    </div>
    </p>
<div class="param">
     sale_price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     If entered, the sale price will be used instead of value in the price field when calculating the product's cost.
 format: double
    </div>
    </p>
<div class="param">
     tax_class_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The ID of the tax class applied to the product. (NOTE: Value ignored if automatic tax is enabled.)
    </div>
    </p>
<div class="param">
     product_tax_code (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Accepts AvaTax System Tax Codes, which identify products and services that fall into special sales-tax categories. By using these codes, merchants who subscribe to Avalara Premium can calculate sales taxes more accurately. Stores without Avalara Premium will ignore the code when calculating sales tax. Do not pass more than one code. The codes are case-sensitive. For details, please see Avalara's documentation.
    </div>
    </p>
<div class="param">
     categories (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       array[Integer]
      </a>
     </span>
     An array of IDs for the categories to which this product belongs. When updating a product, if an array of categories is supplied, all product categories will be overwritten. Does not accept more than 1,000 ID values.
    </div>
    </p>
<div class="param">
     brand_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The ID associated with the product's brand.
    </div>
    </p>
<div class="param">
     inventory_level (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     Current inventory level of the product. Simple inventory tracking must be enabled (See the inventory_tracking field) for this to take any effect.
    </div>
    </p>
<div class="param">
     inventory_warning_level (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     Inventory Warning level for the product. When the product's inventory level drops below the warning level, the store owner will be informed. Simple inventory tracking must be enabled (see the `inventory_tracking` field) for this to take any effect.
    </div>
    </p>
<div class="param">
     inventory_tracking (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The type of inventory tracking for the product. Values are: none - inventory levels will not be tracked; product - inventory levels will be tracked using the `inventory_level` and `inventory_warning_level` fields; variant - inventory levels will be tracked based on variants, which maintain their own warning levels and inventory levels.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     none
    </div>
    <div class="param-enum">
     product
    </div>
    <div class="param-enum">
     variant
    </div>
    </p>
<div class="param">
     fixed_cost_shipping_price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     A fixed shipping cost for the product. If defined, this value will be used during checkout instead of normal shipping-cost calculation.
 format: double
    </div>
    </p>
<div class="param">
     is_free_shipping (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag used to indicate whether the product has free shipping. If `true`, the shipping cost for the product will be zero.
    </div>
    </p>
<div class="param">
     is_visible (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag to determine whether the product should be displayed to customers browsing the store. If `true`, the product will be displayed. If `false`, the product will be hidden from view.
    </div>
    </p>
<div class="param">
     is_featured (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag to determine whether the product should be included in the `featured products` panel when viewing the store.
    </div>
    </p>
<div class="param">
     related_products (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       array[Integer]
      </a>
     </span>
     An array of IDs for the related products.
    </div>
    </p>
<div class="param">
     warranty (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Warranty information displayed on the product page. Can include HTML formatting.
    </div>
    </p>
<div class="param">
     bin_picking_number (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The BIN picking number for the product.
    </div>
    </p>
<div class="param">
     layout_file (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The layout template file used to render this product.
    </div>
    </p>
<div class="param">
     upc (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The product UPC code, which is used in feeds for shopping comparison sites and external channel integrations.
    </div>
    </p>
<div class="param">
     search_keywords (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     A comma-separated list of keywords that can be used to locate the product when searching the store.
    </div>
    </p>
<div class="param">
     availability (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Availability of the product. Availability options are: available - the product can be purchased in the storefront; disabled - the product is listed in the storefront, but cannot be purchased; preorder - the product is listed for pre-orders.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     available
    </div>
    <div class="param-enum">
     disabled
    </div>
    <div class="param-enum">
     preorder
    </div>
    </p>
<div class="param">
     availability_description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Availability text displayed on the checkout page, under the product title. Tells the customer how long it will normally take to ship this product, such as 'Usually ships in 24 hours.'
    </div>
    </p>
<div class="param">
     gift_wrapping_options_type (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Type of gift-wrapping options. Values: `any` - allow any gift-wrapping options in the store; `none` - disallow gift-wrapping on the product; `list` – provide a list of IDs in `gift_wrapping_options_list` field.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     any
    </div>
    <div class="param-enum">
     none
    </div>
    <div class="param-enum">
     list
    </div>
    </p>
<div class="param">
     gift_wrapping_options_list (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       array[Integer]
      </a>
     </span>
     A list of gift-wrapping option IDs.
    </div>
    </p>
<div class="param">
     sort_order (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     Priority to give this product when included in product lists on category pages and in search results. Lower integers will place the product closer to the top of the results.
    </div>
    </p>
<div class="param">
     condition (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The product condition. Will be shown on the product page if the `is_condition_shown` field's value is `true`. Possible values: `New`, `Used`, `Refurbished`.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     New
    </div>
    <div class="param-enum">
     Used
    </div>
    <div class="param-enum">
     Refurbished
    </div>
    </p>
<div class="param">
     is_condition_shown (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag used to determine whether the product condition is shown to the customer on the product page.
    </div>
    </p>
<div class="param">
     order_quantity_minimum (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The minimum quantity an order must contain, to be eligible to purchase this product.
    </div>
    </p>
<div class="param">
     order_quantity_maximum (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The maximum quantity an order can contain when purchasing the product.
    </div>
    </p>
<div class="param">
     page_title (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Custom title for the product page. If not defined, the product name will be used as the meta title.
    </div>
    </p>
<div class="param">
     meta_keywords (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       array[String]
      </a>
     </span>
     Custom meta keywords for the product page. If not defined, the store's default keywords will be used.
    </div>
    </p>
<div class="param">
     meta_description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Custom meta description for the product page. If not defined, the store's default meta description will be used.
    </div>
    </p>
<div class="param">
     view_count (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The number of times the product has been viewed.
    </div>
    </p>
<div class="param">
     preorder_release_date (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#DateTime">
       Date
      </a>
     </span>
     Pre-order release date. See the `availability` field for details on setting a product's availability to accept pre-orders.
 format: date-time
    </div>
    </p>
<div class="param">
     preorder_message (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Custom expected-date message to display on the product page. If undefined, the message defaults to the storewide setting. Can contain the `%%DATE%%` placeholder, which will be substituted for the release date.
    </div>
    </p>
<div class="param">
     is_preorder_only (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     If set to `false`, the product will not change its availability from 	`preorder` to `available` on the release date. Otherwise, on the release date the product's availability/status will change to `available`.
    </div>
    </p>
<div class="param">
     is_price_hidden (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     False by default, indicating that this product's price should be shown on the product page. If set to `true`, the price is hidden. (NOTE: To successfully set `is_price_hidden` to `true`, the `availability` value must be `disabled`.)
    </div>
    </p>
<div class="param">
     price_hidden_label (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     By default, an empty string. If `is_price_hidden` is `true`, the value of `price_hidden_label` is displayed instead of the price. (NOTE: To successfully set a non-empty string value with `is_price_hidden` set to `true`, the `availability` value must be `disabled`.)
    </div>
    </p>
<div class="param">
     custom_url (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#CustomUrlProduct">
       CustomUrlProduct
      </a>
     </span>
    </div>
    </p>
<div class="param">
     bulk_pricing_rules (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#BulkPricingRule">
       array[BulkPricingRule]
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ProductCollectionResponse">
     ProductCollectionResponse
    </a>
   </h3>
   <div class="model-description">
    Response payload for the Bigcommerce API.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Product">
       array[Product]
      </a>
     </span>
    </div>
    </p>
<div class="param">
     meta (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#CollectionMeta">
       CollectionMeta
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ProductImage">
     ProductImage
    </a>
   </h3>
   <div class="model-description">
    The full ProductImage model.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     is_thumbnail (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag for identifying whether the image is used as the product's thumbnail.
    </div>
    </p>
<div class="param">
     sort_order (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The order in which the image will be displayed on the product page. Higher integers give the image a lower priority. When updating, if the image is given a lower priority, all images with a `sort_order` the same as or greater than the image's new `sort_order` value will have their `sort_order`s reordered.
    </div>
    </p>
<div class="param">
     description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The description for the image.
    </div>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the image; increments sequentially.
    </div>
    </p>
<div class="param">
     product_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric identifier for the product with which the image is associated.
    </div>
    </p>
<div class="param">
     image_file (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The local path to the original image file uploaded to BigCommerce.
    </div>
    </p>
<div class="param">
     url_zoom (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The zoom URL for this image. By default, this is used as the zoom image on product pages when zoom images are enabled.
    </div>
    </p>
<div class="param">
     url_standard (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The standard URL for this image. By default, this is used for product-page images.
    </div>
    </p>
<div class="param">
     url_thumbnail (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The thumbnail URL for this image. By default, this is the image size used on the category page and in side panels.
    </div>
    </p>
<div class="param">
     url_tiny (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The tiny URL for this image. By default, this is the image size used for thumbnails beneath the product image on a product page.
    </div>
    </p>
<div class="param">
     date_modified (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#DateTime">
       Date
      </a>
     </span>
     The date on which the product image was modified.
 format: date-time
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ProductImageBase">
     ProductImageBase
    </a>
   </h3>
   <div class="model-description">
    Common ProductImage properties.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     is_thumbnail (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag for identifying whether the image is used as the product's thumbnail.
    </div>
    </p>
<div class="param">
     sort_order (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The order in which the image will be displayed on the product page. Higher integers give the image a lower priority. When updating, if the image is given a lower priority, all images with a `sort_order` the same as or greater than the image's new `sort_order` value will have their `sort_order`s reordered.
    </div>
    </p>
<div class="param">
     description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The description for the image.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ProductImageCollectionResponse">
     ProductImageCollectionResponse
    </a>
   </h3>
   <div class="model-description">
    Response payload for the Bigcommerce API.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#ProductImage">
       array[ProductImage]
      </a>
     </span>
    </div>
    </p>
<div class="param">
     meta (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#CollectionMeta">
       CollectionMeta
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ProductImagePost">
     ProductImagePost
    </a>
   </h3>
   <div class="model-description">
    The model for a POST to create an image on a product.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     is_thumbnail (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag for identifying whether the image is used as the product's thumbnail.
    </div>
    </p>
<div class="param">
     sort_order (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The order in which the image will be displayed on the product page. Higher integers give the image a lower priority. When updating, if the image is given a lower priority, all images with a `sort_order` the same as or greater than the image's new `sort_order` value will have their `sort_order`s reordered.
    </div>
    </p>
<div class="param">
     description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The description for the image.
    </div>
    </p>
<div class="param">
     image_url (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Must be a fully qualified URL path, including protocol.
    </div>
    </p>
<div class="param">
     image_file (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Must be sent as a multipart/form-data field in the request body.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ProductImagePut">
     ProductImagePut
    </a>
   </h3>
   <div class="model-description">
    The model for a PUT to update applicable ProductImage fields.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     is_thumbnail (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag for identifying whether the image is used as the product's thumbnail.
    </div>
    </p>
<div class="param">
     sort_order (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The order in which the image will be displayed on the product page. Higher integers give the image a lower priority. When updating, if the image is given a lower priority, all images with a `sort_order` the same as or greater than the image's new `sort_order` value will have their `sort_order`s reordered.
    </div>
    </p>
<div class="param">
     description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The description for the image.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ProductImageResponse">
     ProductImageResponse
    </a>
   </h3>
   <div class="model-description">
    Response payload for the Bigcommerce API.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#ProductImage">
       ProductImage
      </a>
     </span>
    </div>
    </p>
<div class="param">
     meta (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Meta">
       Meta
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ProductPost">
     ProductPost
    </a>
   </h3>
   <div class="model-description">
    The model for a POST to create product.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The product name.
    </div>
    </p>
<div class="param">
     type (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The product type: physical - a physical stock unit, digital - a digital download
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     physical
    </div>
    <div class="param-enum">
     digital
    </div>
    </p>
<div class="param">
     sku (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     User defined product code/stock keeping unit (SKU).
    </div>
    </p>
<div class="param">
     description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The product description, which can include HTML formatting.
    </div>
    </p>
<div class="param">
     weight (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     Weight of the product, which can be used when calculating shipping costs.
 format: double
    </div>
    </p>
<div class="param">
     width (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     Width of the product, which can be used when calculating shipping costs.
 format: double
    </div>
    </p>
<div class="param">
     depth (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     Depth of the product, which can be used when calculating shipping costs.
 format: double
    </div>
    </p>
<div class="param">
     height (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     Height of the product, which can be used when calculating shipping costs.
 format: double
    </div>
    </p>
<div class="param">
     price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     The price of the product. The price should include or exclude tax, based on the store settings.
 format: double
    </div>
    </p>
<div class="param">
     cost_price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     The cost price of the product. Stored for reference only; it is not used or displayed anywhere on the store.
 format: double
    </div>
    </p>
<div class="param">
     retail_price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     The retail cost of the product. If entered, the retail cost price will be shown on the product page.
 format: double
    </div>
    </p>
<div class="param">
     sale_price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     If entered, the sale price will be used instead of value in the price field when calculating the product's cost.
 format: double
    </div>
    </p>
<div class="param">
     tax_class_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The ID of the tax class applied to the product. (NOTE: Value ignored if automatic tax is enabled.)
    </div>
    </p>
<div class="param">
     product_tax_code (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Accepts AvaTax System Tax Codes, which identify products and services that fall into special sales-tax categories. By using these codes, merchants who subscribe to Avalara Premium can calculate sales taxes more accurately. Stores without Avalara Premium will ignore the code when calculating sales tax. Do not pass more than one code. The codes are case-sensitive. For details, please see Avalara's documentation.
    </div>
    </p>
<div class="param">
     categories (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       array[Integer]
      </a>
     </span>
     An array of IDs for the categories to which this product belongs. When updating a product, if an array of categories is supplied, all product categories will be overwritten. Does not accept more than 1,000 ID values.
    </div>
    </p>
<div class="param">
     brand_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The ID associated with the product's brand.
    </div>
    </p>
<div class="param">
     inventory_level (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     Current inventory level of the product. Simple inventory tracking must be enabled (See the inventory_tracking field) for this to take any effect.
    </div>
    </p>
<div class="param">
     inventory_warning_level (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     Inventory Warning level for the product. When the product's inventory level drops below the warning level, the store owner will be informed. Simple inventory tracking must be enabled (see the `inventory_tracking` field) for this to take any effect.
    </div>
    </p>
<div class="param">
     inventory_tracking (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The type of inventory tracking for the product. Values are: none - inventory levels will not be tracked; product - inventory levels will be tracked using the `inventory_level` and `inventory_warning_level` fields; variant - inventory levels will be tracked based on variants, which maintain their own warning levels and inventory levels.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     none
    </div>
    <div class="param-enum">
     product
    </div>
    <div class="param-enum">
     variant
    </div>
    </p>
<div class="param">
     fixed_cost_shipping_price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     A fixed shipping cost for the product. If defined, this value will be used during checkout instead of normal shipping-cost calculation.
 format: double
    </div>
    </p>
<div class="param">
     is_free_shipping (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag used to indicate whether the product has free shipping. If `true`, the shipping cost for the product will be zero.
    </div>
    </p>
<div class="param">
     is_visible (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag to determine whether the product should be displayed to customers browsing the store. If `true`, the product will be displayed. If `false`, the product will be hidden from view.
    </div>
    </p>
<div class="param">
     is_featured (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag to determine whether the product should be included in the `featured products` panel when viewing the store.
    </div>
    </p>
<div class="param">
     related_products (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       array[Integer]
      </a>
     </span>
     An array of IDs for the related products.
    </div>
    </p>
<div class="param">
     warranty (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Warranty information displayed on the product page. Can include HTML formatting.
    </div>
    </p>
<div class="param">
     bin_picking_number (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The BIN picking number for the product.
    </div>
    </p>
<div class="param">
     layout_file (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The layout template file used to render this product.
    </div>
    </p>
<div class="param">
     upc (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The product UPC code, which is used in feeds for shopping comparison sites and external channel integrations.
    </div>
    </p>
<div class="param">
     search_keywords (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     A comma-separated list of keywords that can be used to locate the product when searching the store.
    </div>
    </p>
<div class="param">
     availability (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Availability of the product. Availability options are: available - the product can be purchased in the storefront; disabled - the product is listed in the storefront, but cannot be purchased; preorder - the product is listed for pre-orders.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     available
    </div>
    <div class="param-enum">
     disabled
    </div>
    <div class="param-enum">
     preorder
    </div>
    </p>
<div class="param">
     availability_description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Availability text displayed on the checkout page, under the product title. Tells the customer how long it will normally take to ship this product, such as 'Usually ships in 24 hours.'
    </div>
    </p>
<div class="param">
     gift_wrapping_options_type (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Type of gift-wrapping options. Values: `any` - allow any gift-wrapping options in the store; `none` - disallow gift-wrapping on the product; `list` – provide a list of IDs in `gift_wrapping_options_list` field.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     any
    </div>
    <div class="param-enum">
     none
    </div>
    <div class="param-enum">
     list
    </div>
    </p>
<div class="param">
     gift_wrapping_options_list (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       array[Integer]
      </a>
     </span>
     A list of gift-wrapping option IDs.
    </div>
    </p>
<div class="param">
     sort_order (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     Priority to give this product when included in product lists on category pages and in search results. Lower integers will place the product closer to the top of the results.
    </div>
    </p>
<div class="param">
     condition (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The product condition. Will be shown on the product page if the `is_condition_shown` field's value is `true`. Possible values: `New`, `Used`, `Refurbished`.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     New
    </div>
    <div class="param-enum">
     Used
    </div>
    <div class="param-enum">
     Refurbished
    </div>
    </p>
<div class="param">
     is_condition_shown (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag used to determine whether the product condition is shown to the customer on the product page.
    </div>
    </p>
<div class="param">
     order_quantity_minimum (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The minimum quantity an order must contain, to be eligible to purchase this product.
    </div>
    </p>
<div class="param">
     order_quantity_maximum (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The maximum quantity an order can contain when purchasing the product.
    </div>
    </p>
<div class="param">
     page_title (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Custom title for the product page. If not defined, the product name will be used as the meta title.
    </div>
    </p>
<div class="param">
     meta_keywords (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       array[String]
      </a>
     </span>
     Custom meta keywords for the product page. If not defined, the store's default keywords will be used.
    </div>
    </p>
<div class="param">
     meta_description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Custom meta description for the product page. If not defined, the store's default meta description will be used.
    </div>
    </p>
<div class="param">
     view_count (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The number of times the product has been viewed.
    </div>
    </p>
<div class="param">
     preorder_release_date (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#DateTime">
       Date
      </a>
     </span>
     Pre-order release date. See the `availability` field for details on setting a product's availability to accept pre-orders.
 format: date-time
    </div>
    </p>
<div class="param">
     preorder_message (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Custom expected-date message to display on the product page. If undefined, the message defaults to the storewide setting. Can contain the `%%DATE%%` placeholder, which will be substituted for the release date.
    </div>
    </p>
<div class="param">
     is_preorder_only (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     If set to `false`, the product will not change its availability from 	`preorder` to `available` on the release date. Otherwise, on the release date the product's availability/status will change to `available`.
    </div>
    </p>
<div class="param">
     is_price_hidden (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     False by default, indicating that this product's price should be shown on the product page. If set to `true`, the price is hidden. (NOTE: To successfully set `is_price_hidden` to `true`, the `availability` value must be `disabled`.)
    </div>
    </p>
<div class="param">
     price_hidden_label (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     By default, an empty string. If `is_price_hidden` is `true`, the value of `price_hidden_label` is displayed instead of the price. (NOTE: To successfully set a non-empty string value with `is_price_hidden` set to `true`, the `availability` value must be `disabled`.)
    </div>
    </p>
<div class="param">
     custom_url (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#CustomUrlProduct">
       CustomUrlProduct
      </a>
     </span>
    </div>
    </p>
<div class="param">
     bulk_pricing_rules (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#BulkPricingRule">
       array[BulkPricingRule]
      </a>
     </span>
    </div>
    </p>
<div class="param">
     custom_fields (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#CustomField">
       array[CustomField]
      </a>
     </span>
    </div>
    </p>
<div class="param">
     variants (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#VariantProductPost">
       array[VariantProductPost]
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ProductPut">
     ProductPut
    </a>
   </h3>
   <div class="model-description">
    The model for a PUT to update product.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The product name.
    </div>
    </p>
<div class="param">
     type (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The product type: physical - a physical stock unit, digital - a digital download
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     physical
    </div>
    <div class="param-enum">
     digital
    </div>
    </p>
<div class="param">
     sku (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     User defined product code/stock keeping unit (SKU).
    </div>
    </p>
<div class="param">
     description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The product description, which can include HTML formatting.
    </div>
    </p>
<div class="param">
     weight (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     Weight of the product, which can be used when calculating shipping costs.
 format: double
    </div>
    </p>
<div class="param">
     width (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     Width of the product, which can be used when calculating shipping costs.
 format: double
    </div>
    </p>
<div class="param">
     depth (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     Depth of the product, which can be used when calculating shipping costs.
 format: double
    </div>
    </p>
<div class="param">
     height (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     Height of the product, which can be used when calculating shipping costs.
 format: double
    </div>
    </p>
<div class="param">
     price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     The price of the product. The price should include or exclude tax, based on the store settings.
 format: double
    </div>
    </p>
<div class="param">
     cost_price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     The cost price of the product. Stored for reference only; it is not used or displayed anywhere on the store.
 format: double
    </div>
    </p>
<div class="param">
     retail_price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     The retail cost of the product. If entered, the retail cost price will be shown on the product page.
 format: double
    </div>
    </p>
<div class="param">
     sale_price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     If entered, the sale price will be used instead of value in the price field when calculating the product's cost.
 format: double
    </div>
    </p>
<div class="param">
     tax_class_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The ID of the tax class applied to the product. (NOTE: Value ignored if automatic tax is enabled.)
    </div>
    </p>
<div class="param">
     product_tax_code (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Accepts AvaTax System Tax Codes, which identify products and services that fall into special sales-tax categories. By using these codes, merchants who subscribe to Avalara Premium can calculate sales taxes more accurately. Stores without Avalara Premium will ignore the code when calculating sales tax. Do not pass more than one code. The codes are case-sensitive. For details, please see Avalara's documentation.
    </div>
    </p>
<div class="param">
     categories (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       array[Integer]
      </a>
     </span>
     An array of IDs for the categories to which this product belongs. When updating a product, if an array of categories is supplied, all product categories will be overwritten. Does not accept more than 1,000 ID values.
    </div>
    </p>
<div class="param">
     brand_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The ID associated with the product's brand.
    </div>
    </p>
<div class="param">
     inventory_level (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     Current inventory level of the product. Simple inventory tracking must be enabled (See the inventory_tracking field) for this to take any effect.
    </div>
    </p>
<div class="param">
     inventory_warning_level (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     Inventory Warning level for the product. When the product's inventory level drops below the warning level, the store owner will be informed. Simple inventory tracking must be enabled (see the `inventory_tracking` field) for this to take any effect.
    </div>
    </p>
<div class="param">
     inventory_tracking (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The type of inventory tracking for the product. Values are: none - inventory levels will not be tracked; product - inventory levels will be tracked using the `inventory_level` and `inventory_warning_level` fields; variant - inventory levels will be tracked based on variants, which maintain their own warning levels and inventory levels.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     none
    </div>
    <div class="param-enum">
     product
    </div>
    <div class="param-enum">
     variant
    </div>
    </p>
<div class="param">
     fixed_cost_shipping_price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     A fixed shipping cost for the product. If defined, this value will be used during checkout instead of normal shipping-cost calculation.
 format: double
    </div>
    </p>
<div class="param">
     is_free_shipping (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag used to indicate whether the product has free shipping. If `true`, the shipping cost for the product will be zero.
    </div>
    </p>
<div class="param">
     is_visible (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag to determine whether the product should be displayed to customers browsing the store. If `true`, the product will be displayed. If `false`, the product will be hidden from view.
    </div>
    </p>
<div class="param">
     is_featured (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag to determine whether the product should be included in the `featured products` panel when viewing the store.
    </div>
    </p>
<div class="param">
     related_products (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       array[Integer]
      </a>
     </span>
     An array of IDs for the related products.
    </div>
    </p>
<div class="param">
     warranty (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Warranty information displayed on the product page. Can include HTML formatting.
    </div>
    </p>
<div class="param">
     bin_picking_number (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The BIN picking number for the product.
    </div>
    </p>
<div class="param">
     layout_file (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The layout template file used to render this product.
    </div>
    </p>
<div class="param">
     upc (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The product UPC code, which is used in feeds for shopping comparison sites and external channel integrations.
    </div>
    </p>
<div class="param">
     search_keywords (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     A comma-separated list of keywords that can be used to locate the product when searching the store.
    </div>
    </p>
<div class="param">
     availability (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Availability of the product. Availability options are: available - the product can be purchased in the storefront; disabled - the product is listed in the storefront, but cannot be purchased; preorder - the product is listed for pre-orders.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     available
    </div>
    <div class="param-enum">
     disabled
    </div>
    <div class="param-enum">
     preorder
    </div>
    </p>
<div class="param">
     availability_description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Availability text displayed on the checkout page, under the product title. Tells the customer how long it will normally take to ship this product, such as 'Usually ships in 24 hours.'
    </div>
    </p>
<div class="param">
     gift_wrapping_options_type (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Type of gift-wrapping options. Values: `any` - allow any gift-wrapping options in the store; `none` - disallow gift-wrapping on the product; `list` – provide a list of IDs in `gift_wrapping_options_list` field.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     any
    </div>
    <div class="param-enum">
     none
    </div>
    <div class="param-enum">
     list
    </div>
    </p>
<div class="param">
     gift_wrapping_options_list (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       array[Integer]
      </a>
     </span>
     A list of gift-wrapping option IDs.
    </div>
    </p>
<div class="param">
     sort_order (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     Priority to give this product when included in product lists on category pages and in search results. Lower integers will place the product closer to the top of the results.
    </div>
    </p>
<div class="param">
     condition (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The product condition. Will be shown on the product page if the `is_condition_shown` field's value is `true`. Possible values: `New`, `Used`, `Refurbished`.
    </div>
    <div class="param-enum-header">
     Enum:
    </div>
    <div class="param-enum">
     New
    </div>
    <div class="param-enum">
     Used
    </div>
    <div class="param-enum">
     Refurbished
    </div>
    </p>
<div class="param">
     is_condition_shown (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     Flag used to determine whether the product condition is shown to the customer on the product page.
    </div>
    </p>
<div class="param">
     order_quantity_minimum (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The minimum quantity an order must contain, to be eligible to purchase this product.
    </div>
    </p>
<div class="param">
     order_quantity_maximum (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The maximum quantity an order can contain when purchasing the product.
    </div>
    </p>
<div class="param">
     page_title (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Custom title for the product page. If not defined, the product name will be used as the meta title.
    </div>
    </p>
<div class="param">
     meta_keywords (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       array[String]
      </a>
     </span>
     Custom meta keywords for the product page. If not defined, the store's default keywords will be used.
    </div>
    </p>
<div class="param">
     meta_description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Custom meta description for the product page. If not defined, the store's default meta description will be used.
    </div>
    </p>
<div class="param">
     view_count (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The number of times the product has been viewed.
    </div>
    </p>
<div class="param">
     preorder_release_date (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#DateTime">
       Date
      </a>
     </span>
     Pre-order release date. See the `availability` field for details on setting a product's availability to accept pre-orders.
 format: date-time
    </div>
    </p>
<div class="param">
     preorder_message (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Custom expected-date message to display on the product page. If undefined, the message defaults to the storewide setting. Can contain the `%%DATE%%` placeholder, which will be substituted for the release date.
    </div>
    </p>
<div class="param">
     is_preorder_only (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     If set to `false`, the product will not change its availability from 	`preorder` to `available` on the release date. Otherwise, on the release date the product's availability/status will change to `available`.
    </div>
    </p>
<div class="param">
     is_price_hidden (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     False by default, indicating that this product's price should be shown on the product page. If set to `true`, the price is hidden. (NOTE: To successfully set `is_price_hidden` to `true`, the `availability` value must be `disabled`.)
    </div>
    </p>
<div class="param">
     price_hidden_label (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     By default, an empty string. If `is_price_hidden` is `true`, the value of `price_hidden_label` is displayed instead of the price. (NOTE: To successfully set a non-empty string value with `is_price_hidden` set to `true`, the `availability` value must be `disabled`.)
    </div>
    </p>
<div class="param">
     custom_url (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#CustomUrlProduct">
       CustomUrlProduct
      </a>
     </span>
    </div>
    </p>
<div class="param">
     bulk_pricing_rules (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#BulkPricingRule">
       array[BulkPricingRule]
      </a>
     </span>
    </div>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numerical ID of the product, increments sequentially.
    </div>
    </p>
<div class="param">
     custom_fields (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#CustomField">
       array[CustomField]
      </a>
     </span>
    </div>
    </p>
<div class="param">
     variants (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#VariantProductPut">
       array[VariantProductPut]
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ProductResponse">
     ProductResponse
    </a>
   </h3>
   <div class="model-description">
    Response payload for the Bigcommerce API.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Product">
       Product
      </a>
     </span>
    </div>
    </p>
<div class="param">
     meta (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Meta">
       Meta
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ProductVideo">
     ProductVideo
    </a>
   </h3>
   <div class="model-description">
    A product video model.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     title (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The title for the video. If left blank, this will be filled in according to data on YouTube.
    </div>
    </p>
<div class="param">
     description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The description for the video. If left blank, this will be filled in according to data on YouTube.
    </div>
    </p>
<div class="param">
     sort_order (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The order in which the video will be displayed on the product page. Higher integers give the video a lower priority. When updating, if the video is given a lower priority, all videos with a `sort_order` the same as or greater than the video's new `sort_order` value will have their `sort_order`s reordered.
    </div>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The ID of a YouTube video.
    </div>
    </p>
<div class="param">
     product_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric identifier for the product with which the image is associated.
    </div>
    </p>
<div class="param">
     length (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Length of the video. This will be filled in according to data on YouTube.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ProductVideoBase">
     ProductVideoBase
    </a>
   </h3>
   <div class="model-description">
    Common ProductVideo properties.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     title (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The title for the video. If left blank, this will be filled in according to data on YouTube.
    </div>
    </p>
<div class="param">
     description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The description for the video. If left blank, this will be filled in according to data on YouTube.
    </div>
    </p>
<div class="param">
     sort_order (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The order in which the video will be displayed on the product page. Higher integers give the video a lower priority. When updating, if the video is given a lower priority, all videos with a `sort_order` the same as or greater than the video's new `sort_order` value will have their `sort_order`s reordered.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ProductVideoCollectionResponse">
     ProductVideoCollectionResponse
    </a>
   </h3>
   <div class="model-description">
    Response payload for the Bigcommerce API.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#ProductVideo">
       array[ProductVideo]
      </a>
     </span>
    </div>
    </p>
<div class="param">
     meta (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#CollectionMeta">
       CollectionMeta
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ProductVideoPost">
     ProductVideoPost
    </a>
   </h3>
   <div class="model-description">
    The model for a POST to create video on a product.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     title (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The title for the video. If left blank, this will be filled in according to data on YouTube.
    </div>
    </p>
<div class="param">
     description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The description for the video. If left blank, this will be filled in according to data on YouTube.
    </div>
    </p>
<div class="param">
     sort_order (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The order in which the video will be displayed on the product page. Higher integers give the video a lower priority. When updating, if the video is given a lower priority, all videos with a `sort_order` the same as or greater than the video's new `sort_order` value will have their `sort_order`s reordered.
    </div>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The ID of a YouTube video.
    </div>
    </p>
<div class="param">
     product_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric identifier for the product with which the image is associated.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ProductVideoPut">
     ProductVideoPut
    </a>
   </h3>
   <div class="model-description">
    The model for a PUT to update video on a product.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     title (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The title for the video. If left blank, this will be filled in according to data on YouTube.
    </div>
    </p>
<div class="param">
     description (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The description for the video. If left blank, this will be filled in according to data on YouTube.
    </div>
    </p>
<div class="param">
     sort_order (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The order in which the video will be displayed on the product page. Higher integers give the video a lower priority. When updating, if the video is given a lower priority, all videos with a `sort_order` the same as or greater than the video's new `sort_order` value will have their `sort_order`s reordered.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ProductVideoResponse">
     ProductVideoResponse
    </a>
   </h3>
   <div class="model-description">
    Response payload for the Bigcommerce API.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#ProductVideo">
       ProductVideo
      </a>
     </span>
    </div>
    </p>
<div class="param">
     meta (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Meta">
       Meta
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="ResourceImage">
     ResourceImage
    </a>
   </h3>
   <div class="model-description">
    An object containing a publicly accessible image URL, or a form post that contains an image file.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     image_url (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     A public URL for a GIF, JPEG, or PNG image.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="Subscriber">
     Subscriber
    </a>
   </h3>
   <div class="model-description">
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the subscriber; increments sequentially.
    </div>
    </p>
<div class="param">
     email (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The email of the subscriber. Must be unique.
    </div>
    </p>
<div class="param">
     first_name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The first name of the subscriber.
    </div>
    </p>
<div class="param">
     last_name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The last name of the subscriber.
    </div>
    </p>
<div class="param">
     source (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The source of the subscriber. Values are: `storefront`, `order`, or `custom`.
    </div>
    </p>
<div class="param">
     order_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The ID of the source order, if source was an order.
    </div>
    </p>
<div class="param">
     date_modified (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#DateTime">
       Date
      </a>
     </span>
     The date on which the subscriber was modified.
 format: date-time
    </div>
    </p>
<div class="param">
     date_created (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#DateTime">
       Date
      </a>
     </span>
     The date of which the subscriber was created.
 format: date-time
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="SubscriberBase">
     SubscriberBase
    </a>
   </h3>
   <div class="model-description">
    Common Subscriber properties.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the subscriber; increments sequentially.
    </div>
    </p>
<div class="param">
     email (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The email of the subscriber. Must be unique.
    </div>
    </p>
<div class="param">
     first_name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The first name of the subscriber.
    </div>
    </p>
<div class="param">
     last_name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The last name of the subscriber.
    </div>
    </p>
<div class="param">
     source (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The source of the subscriber. Values are: `storefront`, `order`, or `custom`.
    </div>
    </p>
<div class="param">
     order_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The ID of the source order, if source was an order.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="SubscriberCollectionResponse">
     SubscriberCollectionResponse
    </a>
   </h3>
   <div class="model-description">
    Response payload for the Bigcommerce API.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Subscriber">
       array[Subscriber]
      </a>
     </span>
    </div>
    </p>
<div class="param">
     meta (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#CollectionMeta">
       CollectionMeta
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="SubscriberPost">
     SubscriberPost
    </a>
   </h3>
   <div class="model-description">
    The model for a POST to create subscriber.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the subscriber; increments sequentially.
    </div>
    </p>
<div class="param">
     email (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The email of the subscriber. Must be unique.
    </div>
    </p>
<div class="param">
     first_name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The first name of the subscriber.
    </div>
    </p>
<div class="param">
     last_name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The last name of the subscriber.
    </div>
    </p>
<div class="param">
     source (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The source of the subscriber. Values are: `storefront`, `order`, or `custom`.
    </div>
    </p>
<div class="param">
     order_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The ID of the source order, if source was an order.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="SubscriberPut">
     SubscriberPut
    </a>
   </h3>
   <div class="model-description">
    The model for a PUT to update subscriber.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The unique numeric ID of the subscriber; increments sequentially.
    </div>
    </p>
<div class="param">
     email (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The email of the subscriber. Must be unique.
    </div>
    </p>
<div class="param">
     first_name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The first name of the subscriber.
    </div>
    </p>
<div class="param">
     last_name (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The last name of the subscriber.
    </div>
    </p>
<div class="param">
     source (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The source of the subscriber. Values are: `storefront`, `order`, or `custom`.
    </div>
    </p>
<div class="param">
     order_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     The ID of the source order, if source was an order.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="SubscriberResponse">
     SubscriberResponse
    </a>
   </h3>
   <div class="model-description">
    Response payload for the Bigcommerce API.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Subscriber">
       Subscriber
      </a>
     </span>
    </div>
    </p>
<div class="param">
     meta (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Meta">
       Meta
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="Variant">
     Variant
    </a>
   </h3>
   <div class="model-description">
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     cost_price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     The cost price of the variant. format: double
    </div>
    </p>
<div class="param">
     price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     This variant's base price on the storefront. If this value is null, the product's default price (set in the Product resource's `price` field) will be used as the base price. format: double
    </div>
    </p>
<div class="param">
     weight (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     This variant's base weight on the storefront. If this value is null, the product's default weight (set in the Product resource's weight field) will be used as the base weight. format: double
    </div>
    </p>
<div class="param">
     purchasing_disabled (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     If `true`, this variant will not be purchasable on the storefront.
    </div>
    </p>
<div class="param">
     purchasing_disabled_message (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     If `purchasing_disabled` is `true`, this message should show on the storefront when the variant is selected.
    </div>
    </p>
<div class="param">
     image_url (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The image that will be displayed when this variant is selected on the storefront. When updating a SKU image, send the publicly accessible URL. Supported image formats are JPEG, PNG, and GIF. Generic product images not specific to the variant should be stored on the product.
    </div>
    </p>
<div class="param">
     upc (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The UPC code used in feeds for shopping comparison sites and external channel integrations.
    </div>
    </p>
<div class="param">
     inventory_level (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     Inventory level for the variant, which is used when the product's inventory_tracking is set to variant
    </div>
    </p>
<div class="param">
     inventory_warning_level (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     When the variant hits this inventory level, it is considered low stock.
    </div>
    </p>
<div class="param">
     bin_picking_number (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Identifies where in a warehouse the variant is located.
    </div>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
    </div>
    </p>
<div class="param">
     product_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
    </div>
    </p>
<div class="param">
     sku (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
    </div>
    </p>
<div class="param">
     sku_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     Read-only reference to v2 API's SKU ID. Null if it is a base variant.
    </div>
    </p>
<div class="param">
     option_values (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#OptionValueVariant">
       array[OptionValueVariant]
      </a>
     </span>
     Array of option and option values IDs that make up this variant. Will be empty if the variant is the product's base variant.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="VariantBase">
     VariantBase
    </a>
   </h3>
   <div class="model-description">
    Common Variant properties.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     cost_price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     The cost price of the variant. format: double
    </div>
    </p>
<div class="param">
     price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     This variant's base price on the storefront. If this value is null, the product's default price (set in the Product resource's `price` field) will be used as the base price. format: double
    </div>
    </p>
<div class="param">
     weight (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     This variant's base weight on the storefront. If this value is null, the product's default weight (set in the Product resource's weight field) will be used as the base weight. format: double
    </div>
    </p>
<div class="param">
     purchasing_disabled (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     If `true`, this variant will not be purchasable on the storefront.
    </div>
    </p>
<div class="param">
     purchasing_disabled_message (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     If `purchasing_disabled` is `true`, this message should show on the storefront when the variant is selected.
    </div>
    </p>
<div class="param">
     image_url (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The image that will be displayed when this variant is selected on the storefront. When updating a SKU image, send the publicly accessible URL. Supported image formats are JPEG, PNG, and GIF. Generic product images not specific to the variant should be stored on the product.
    </div>
    </p>
<div class="param">
     upc (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The UPC code used in feeds for shopping comparison sites and external channel integrations.
    </div>
    </p>
<div class="param">
     inventory_level (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     Inventory level for the variant, which is used when the product's inventory_tracking is set to variant
    </div>
    </p>
<div class="param">
     inventory_warning_level (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     When the variant hits this inventory level, it is considered low stock.
    </div>
    </p>
<div class="param">
     bin_picking_number (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Identifies where in a warehouse the variant is located.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="VariantCollectionResponse">
     VariantCollectionResponse
    </a>
   </h3>
   <div class="model-description">
    Response payload for the Bigcommerce API.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Variant">
       array[Variant]
      </a>
     </span>
    </div>
    </p>
<div class="param">
     meta (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#CollectionMeta">
       CollectionMeta
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="VariantPost">
     VariantPost
    </a>
   </h3>
   <div class="model-description">
    The model for a POST to create variant on a product.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     cost_price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     The cost price of the variant. format: double
    </div>
    </p>
<div class="param">
     price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     This variant's base price on the storefront. If this value is null, the product's default price (set in the Product resource's `price` field) will be used as the base price. format: double
    </div>
    </p>
<div class="param">
     weight (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     This variant's base weight on the storefront. If this value is null, the product's default weight (set in the Product resource's weight field) will be used as the base weight. format: double
    </div>
    </p>
<div class="param">
     purchasing_disabled (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     If `true`, this variant will not be purchasable on the storefront.
    </div>
    </p>
<div class="param">
     purchasing_disabled_message (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     If `purchasing_disabled` is `true`, this message should show on the storefront when the variant is selected.
    </div>
    </p>
<div class="param">
     image_url (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The image that will be displayed when this variant is selected on the storefront. When updating a SKU image, send the publicly accessible URL. Supported image formats are JPEG, PNG, and GIF. Generic product images not specific to the variant should be stored on the product.
    </div>
    </p>
<div class="param">
     upc (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The UPC code used in feeds for shopping comparison sites and external channel integrations.
    </div>
    </p>
<div class="param">
     inventory_level (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     Inventory level for the variant, which is used when the product's inventory_tracking is set to variant
    </div>
    </p>
<div class="param">
     inventory_warning_level (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     When the variant hits this inventory level, it is considered low stock.
    </div>
    </p>
<div class="param">
     bin_picking_number (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Identifies where in a warehouse the variant is located.
    </div>
    </p>
<div class="param">
     product_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
    </div>
    </p>
<div class="param">
     sku (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
    </div>
    </p>
<div class="param">
     option_values (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#OptionValueVariantPost">
       array[OptionValueVariantPost]
      </a>
     </span>
     Array of option and option values IDs that make up this variant. Will be empty if the variant is the product's base variant.
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="VariantProductPost">
     VariantProductPost
    </a>
   </h3>
   <div class="model-description">
    The model for a POST to create variant with a product.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     cost_price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     The cost price of the variant. format: double
    </div>
    </p>
<div class="param">
     price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     This variant's base price on the storefront. If this value is null, the product's default price (set in the Product resource's `price` field) will be used as the base price. format: double
    </div>
    </p>
<div class="param">
     weight (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     This variant's base weight on the storefront. If this value is null, the product's default weight (set in the Product resource's weight field) will be used as the base weight. format: double
    </div>
    </p>
<div class="param">
     purchasing_disabled (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     If `true`, this variant will not be purchasable on the storefront.
    </div>
    </p>
<div class="param">
     purchasing_disabled_message (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     If `purchasing_disabled` is `true`, this message should show on the storefront when the variant is selected.
    </div>
    </p>
<div class="param">
     image_url (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The image that will be displayed when this variant is selected on the storefront. When updating a SKU image, send the publicly accessible URL. Supported image formats are JPEG, PNG, and GIF. Generic product images not specific to the variant should be stored on the product.
    </div>
    </p>
<div class="param">
     upc (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The UPC code used in feeds for shopping comparison sites and external channel integrations.
    </div>
    </p>
<div class="param">
     inventory_level (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     Inventory level for the variant, which is used when the product's inventory_tracking is set to variant
    </div>
    </p>
<div class="param">
     inventory_warning_level (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     When the variant hits this inventory level, it is considered low stock.
    </div>
    </p>
<div class="param">
     bin_picking_number (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Identifies where in a warehouse the variant is located.
    </div>
    </p>
<div class="param">
     product_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
    </div>
    </p>
<div class="param">
     sku (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
    </div>
    </p>
<div class="param">
     option_values (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#OptionValueProductPost">
       array[OptionValueProductPost]
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="VariantProductPut">
     VariantProductPut
    </a>
   </h3>
   <div class="model-description">
    The model for a PUT to update variant with a product.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     cost_price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     The cost price of the variant. format: double
    </div>
    </p>
<div class="param">
     price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     This variant's base price on the storefront. If this value is null, the product's default price (set in the Product resource's `price` field) will be used as the base price. format: double
    </div>
    </p>
<div class="param">
     weight (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     This variant's base weight on the storefront. If this value is null, the product's default weight (set in the Product resource's weight field) will be used as the base weight. format: double
    </div>
    </p>
<div class="param">
     purchasing_disabled (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     If `true`, this variant will not be purchasable on the storefront.
    </div>
    </p>
<div class="param">
     purchasing_disabled_message (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     If `purchasing_disabled` is `true`, this message should show on the storefront when the variant is selected.
    </div>
    </p>
<div class="param">
     image_url (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The image that will be displayed when this variant is selected on the storefront. When updating a SKU image, send the publicly accessible URL. Supported image formats are JPEG, PNG, and GIF. Generic product images not specific to the variant should be stored on the product.
    </div>
    </p>
<div class="param">
     upc (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The UPC code used in feeds for shopping comparison sites and external channel integrations.
    </div>
    </p>
<div class="param">
     inventory_level (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     Inventory level for the variant, which is used when the product's inventory_tracking is set to variant
    </div>
    </p>
<div class="param">
     inventory_warning_level (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     When the variant hits this inventory level, it is considered low stock.
    </div>
    </p>
<div class="param">
     bin_picking_number (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Identifies where in a warehouse the variant is located.
    </div>
    </p>
<div class="param">
     product_id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
    </div>
    </p>
<div class="param">
     sku (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="VariantPut">
     VariantPut
    </a>
   </h3>
   <div class="model-description">
    The model for a PUT to update variant on a product.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     cost_price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     The cost price of the variant. format: double
    </div>
    </p>
<div class="param">
     price (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     This variant's base price on the storefront. If this value is null, the product's default price (set in the Product resource's `price` field) will be used as the base price. format: double
    </div>
    </p>
<div class="param">
     weight (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#double">
       Double
      </a>
     </span>
     This variant's base weight on the storefront. If this value is null, the product's default weight (set in the Product resource's weight field) will be used as the base weight. format: double
    </div>
    </p>
<div class="param">
     purchasing_disabled (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#boolean">
       Boolean
      </a>
     </span>
     If `true`, this variant will not be purchasable on the storefront.
    </div>
    </p>
<div class="param">
     purchasing_disabled_message (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     If `purchasing_disabled` is `true`, this message should show on the storefront when the variant is selected.
    </div>
    </p>
<div class="param">
     image_url (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The image that will be displayed when this variant is selected on the storefront. When updating a SKU image, send the publicly accessible URL. Supported image formats are JPEG, PNG, and GIF. Generic product images not specific to the variant should be stored on the product.
    </div>
    </p>
<div class="param">
     upc (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     The UPC code used in feeds for shopping comparison sites and external channel integrations.
    </div>
    </p>
<div class="param">
     inventory_level (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     Inventory level for the variant, which is used when the product's inventory_tracking is set to variant
    </div>
    </p>
<div class="param">
     inventory_warning_level (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
     When the variant hits this inventory level, it is considered low stock.
    </div>
    </p>
<div class="param">
     bin_picking_number (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#string">
       String
      </a>
     </span>
     Identifies where in a warehouse the variant is located.
    </div>
    </p>
<div class="param">
     id (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#integer">
       Integer
      </a>
     </span>
    </div>
   </div>
  </div>
  <div class="model">
   </p>
   <h3 class="field-label">
    <a name="VariantResponse">
     VariantResponse
    </a>
   </h3>
   <div class="model-description">
    Successful response.
   </div>
   <br/>
   <div class="field-items"> <b>Properties:</b>
    </p>
<div class="param">
     data (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Variant">
       Variant
      </a>
     </span>
    </div>
    </p>
<div class="param">
     meta (optional)
    </div>
    <div class="param-desc">
     <span class="param-type">
      <a href="#Meta">
       Meta
      </a>
     </span>
    </div>
   </div>
  </div>
 </body>
</html>
