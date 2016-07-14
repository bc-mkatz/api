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

## v2 Catalog API and Control Panel Interoperability

The v3 Catalog API is essentially our catalog's future state. This means that many concepts don't map visibly to their v2 and control panel relatives.

The good news here is we've built this API with v2 interoperability in mind. So you should be able to use both APIs simultaneously as you (ideally) fully transition all catalog management to v3. The key areas to be aware of are:    

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

## v3 Catalog API Reference

Please view the documentation generated from the Swagger file [here](http://editor.swagger.io/#/?import=https://raw.githubusercontent.com/bigcommerce/api/master/swagger/v3-catalog.yaml).

### Endpoints

HTTP request | Description
------------- | -------------
**GET** /catalog/products | Returns a paginated collection of Products objects from the BigCommerce Catalog.
**POST** /catalog/products | Creates a Product object in the BigCommerce Catalog.
**DELETE** /catalog/products | Deletes one or more Product objects from the BigCommerce Catalog, by ID or filter.
**GET** /catalog/products/{product_id} | Returns a Product object from the BigCommerce Catalog.
**PUT** /catalog/products/{product_id} | Updates a Product object in the BigCommerce Catalog.
**DELETE** /catalog/products/{product_id} | Deletes a Product object from the BigCommerce Catalog.
**GET** /catalog/variants | Returns a paginated collection of Variant objects from the BigCommerce Catalog.
**GET** /catalog/variants/{variant_id} | Returns a Variant from the BigCommerce Catalog.
**PUT** /catalog/variants/{variant_id} | Updates a Variant in the BigCommerce Catalog.
**DELETE** /catalog/variants/{variant_id} | Deletes a Variant object from the BigCommerce Catalog.
**GET** /catalog/products/{product_id}/images | Returns a list of product images.
**POST** /catalog/products/{product_id}/images | Creates an image on a product. Files (`form post`) are valid parameters.
**GET** /catalog/products/{product_id}/images/{image_id} | Returns a specific product image.
**PUT** /catalog/products/{product_id}/images/{image_id} | Updates a specific product image.
**DELETE** /catalog/products/{product_id}/images/{image_id} | Removes a specific product image.
**GET** /catalog/products/{product_id}/variants | Returns a Variant object list from the BigCommerce Catalog.
**POST** /catalog/products/{product_id}/variants | Creates a Variant object.
**GET** /catalog/products/{product_id}/variants/{variant_id} | Gets a Variant object.
**PUT** /catalog/products/{product_id}/variants/{variant_id} | Updates a Variant object.
**DELETE** /catalog/products/{product_id}/variants/{variant_id} | Deletes a Variant object.
**POST** /catalog/products/{product_id}/variants/{variant_id}/image | Adds an image to a variant; the image will show on the storefront when the variant is selected.
**GET** /catalog/products/{product_id}/options | Gets an array of Option objects.
**POST** /catalog/products/{product_id}/options | Creates an Option object.
**PUT** /catalog/products/{product_id}/options/{option_id} | Updates a Product's Option, based on the `product_id` and `option_id`.
**DELETE** /catalog/products/{product_id}/options/{option_id} | Deletes a Product's Option, based on the `product_id` and `option_id`.
**GET** /catalog/products/{product_id}/modifiers | Gets an array of Modifier objects.
**POST** /catalog/products/{product_id}/modifiers | Creates a modifier.
**GET** /catalog/products/{product_id}/modifiers/{modifier_id} | Gets a Product Modifier, by `modifier_id`.
**PUT** /catalog/products/{product_id}/modifiers/{modifier_id} | Updates an Product's Modifier, based on the `product_id` and `modifier_id`.
**DELETE** /catalog/products/{product_id}/modifiers/{modifier_id} | Delete a Product's Modifier based on the product_id and modifier_id.
**POST** /catalog/products/{product_id}/modifiers/{modifier_id}/values/{value_id}image | Adds an image to a modifier value; the image will show on the storefront when the modifier is selected.
**DELETE** /catalog/products/{product_id}/modifiers/{modifier_id}/values/{value_id}/image | Deletes the image applied to show when the modifier value is selected.
**GET** /catalog/categories | Returns a paginated categories collection from the BigCommerce Catalog.
**POST** /catalog/categories | Creates a Category object in the BigCommerce Catalog.
**DELETE** /catalog/categories | Delete a Category object from the BigCommerce Catalog.
**GET** /catalog/categories/{category_id} | Returns a Category object from the BigCommerce Catalog.
**PUT** /catalog/categories/{category_id} | Updates a Category object in the BigCommerce Catalog.
**DELETE** /catalog/categories/{category_id} | Deletes one or more Catggory objects from the BigCommerce catalog.
**POST** /catalog/categories/{category_id}/image | Creates an image on a category; publicly accessible URLs and files (`form post`) are valid parameters.
**DELETE** /catalog/categories/{category_id}/image | Deletes a Category image from the BigCommerce Catalog.
**GET** /catalog/categories/tree | Returns the categories tree, a nested lineage of the categories with parent->child relationship. The Category objects returned reflect a simplified version of the Category objects returned in the rest of this API.
**GET** /catalog/brands | Returns a paginated collection of Brand objects from the BigCommerce Catalog.
**POST** /catalog/brands | Create a Brand object in the BigCommerce Catalog.
**DELETE** /catalog/brands | Delete one or more Brand objects from the BigCommerce Catalog.
**GET** /catalog/brands/{brand_id} | Gets Brand objects.
**PUT** /catalog/brands/{brand_id} | Updates a Brand in the BigCommerce Catalog.
**DELETE** /catalog/brands/{brand_id} | Deletes a Brand from the BigCommerce Catalog.
**POST** /catalog/brands/{brand_id}/image | Creates an image on a Brand; publicly accessible URLs and files (`form post`) are valid parameters.
**DELETE** /catalog/brands/{brand_id}/image | Delete a Brand image the BigCommerce Catalog.

### Models

##### Brand
------

Name | Type | Description | Notes
------------ | ------------- | ------------- | -------------
**id** | **int** | The unique numeric ID of the brand; increments sequentially. | [optional] 
**name** | **string** | The name of the brand; must be unique. | [optional] 
**page_title** | **string** | The title shown in the browser while viewing the brand. | [optional] 
**meta_keywords** | **string[]** | Comma-separated list of meta keywords to include in the HTML. | [optional] 
**meta_description** | **string** | A meta description to include. | [optional] 
**search_keywords** | **string** | A comma-separated list of keywords that can be used to locate this brand. | [optional] 
**image_url** | **string** | A valid image. | [optional] 


##### BulkPricingRule
------

Name | Type | Description | Notes
------------ | ------------- | ------------- | -------------
**id** | **int** | The ID of the bulk pricing rule. | [optional] 
**quantity_min** | **int** | The minimum (inclusive) quantity of a product to satisfy this rule. Must be greater than or equal to 0 (zero). | [optional] 
**quantity_max** | **int** | The maximum (inclusive) quantity of a product to satisfy this rule. Must be greater than the `quantity_min` value – unless this field has a value of zero, in which case there will be no maximum bound for this rule. | [optional] 
**type** | **string** | The type of adjustment that is made. One of: `price` - adjustment amount per product; `percent` - adjustment as a percent of the original price; `fixed` - adjusted absolute price of the product. | [optional] 
**amount** | **int** | The value of the ?? adjusted by the bulk pricing rule. | [optional] 


##### Category
------

Name | Type | Description | Notes
------------ | ------------- | ------------- | -------------
**id** | **int** | The unique numeric ID of the category; increments sequentially. | [optional] 
**parent_id** | **int** | The unique numeric ID of the category&#39;s parent. This field controls where the category sits in the tree of categories that organize the catalog. | [optional] 
**name** | **string** | The name displayed for the category. Name is unique to the category&#39;s siblings. | [optional] 
**description** | **string** | The product description, which can include HTML formatting. | [optional] 
**views** | **int** | Number of views that the category has on the storefront. | [optional] 
**sort** | **int** | Priority that this category will be given when included in the menu and category pages. The lower the number, the closer the category will be to the top of the results. | [optional] 
**page_title** | **string** | Custom title for the category page. If not defined, the category name will be used as the meta title. | [optional] 
**meta_keywords** | **string[]** | Custom meta keywords for the category page. If not defined, the store default keywords will be used. Must post as an array like: `[\&quot;awesome\&quot;,\&quot;sauce\&quot;]`. | [optional] 
**meta_description** | **string** | Custom meta description for the category page. If not defined, the store default meta description will be used. | [optional] 
**layout_file** | **string** | The layout template file used to render this category. | [optional] 
**image_file** | **string** | URL of the image file shown for this category on the storefront. Images can be uploaded to `/categories/{categoryId}/images`. | [optional] 
**is_visible** | **bool** | Flag to determine whether or not the product should be displayed to customers browsing the store. If `true`, the category will be displayed. If `false`, the category will be hidden from view. | [optional] 
**search_keywords** | **string** | A comma-separated list of keywords that can be used to locate the category when searching the store. | [optional] 
**default_product_sort** | **string** | Determines how the products are sorted upon category page load. | [optional] 
**custom_url** | **string** | The custom URL for the category on the storefront. | [optional] 


##### CategoryNode
------

Name | Type | Description | Notes
------------ | ------------- | ------------- | -------------
**id** | **int** | The unique numeric ID of the category; increments sequentially. | [optional] 
**parent_id** | **int** | The unique numeric ID of the category&#39;s parent. This field controls where the category sits in the tree of categories that organize the catalog. | [optional] 
**name** | **string** | The name displayed for the category. Name is unique to the category&#39;s siblings. | [optional] 
**is_visible** | **bool** | Flag to determine whether or not the product should be displayed to customers browsing the store. If `true`, the category will be displayed. If `false`, the category will be hidden from view. | [optional] 
**url** | **string** | The custom URL for the category on the storefront. | [optional] 
**children** | **array** | The list of children of the category. | [optional] 


##### CollectionMeta
------

Name | Type | Description | Notes
------------ | ------------- | ------------- | -------------
**total** | **int** | Total number of items returned in the result set. | [optional]
**count** | **int** | Total number of items in the collection. | [optional] 
**per_page** | **int** | The number of items returned in the collection per page, controlled by the `limit` parameter. | [optional] 
**current_page** | **int** | The page you are currently on within the collection. | [optional] 
**total_pages** | **int** | The total number of pages in the collection. | [optional] 
**links** | **object** | ?? Collection of page links. ?? | [optional] 
**links.previous** | **string** | Link to the previous page returned in the response. | [optional] 
**links.current** | **string** | Link to the current page returned in the response. | [optional] 
**links.next** | **string** | Link to the next page returned in the response. | [optional] 

##### Modifier
------

Name | Type | Description | Notes
------------ | ------------- | ------------- | -------------
**id** | **int** | The unique numeric ID of the modifier; increments sequentially. | [optional] 
**product_id** | **int** | The unique numeric ID of the product to which the option belongs. | [optional] 
**name** | **string** | The unique option name; auto-generated from the display name, a timestamp, and the product ID. | [optional] 
**display_name** | **string** | The name of the option shown on the storefront. | [optional] 
**type** | **string** | Possible values: `date`, `checkbox`, `file`, `text`, `multi_line_text`, `numbers_only_text`, `radio_buttons`, `rectangles`, `dropdown`, `product_list`, `product_list_with_images`, `swatch`. | [optional] 
**config** | **object** | (Please see [OptionConfig](#optionconfig).) | [optional] 
**values** | **object** | ?? Collection of values. ?? | [optional] 
**values.id** | **int** | The unique numeric ID of the value; increments sequentially. | [optional] 
**values.is_default** | **bool** | The flag for preselecting a value as the default on the storefront. This field is not supported for `swatch` options/modifiers. | [optional] 
**values.label** | **string** | The text displayed to identify the value on the storefront. | [optional] 
**values.sort_order** | **int** | The order in which the value will be displayed on the product page. | [optional] 
**values.value_data** | **object** | Extra data describing the value, based on the value's associated type of option or modifier. A `swatch` type requires an array of colors, with up to three hexidecimal color keys. A `product_list` type requires a `product_id`. A `checkbox` type requires a boolean flag called `checked_value` to determine which value is considered to be the checked state. | [optional] 
**values.adjusters** | **object** | ?? Collection of adjusters. ?? | [optional] ??
**values.adjusters.adjuster** | **string** | The type of adjuster – either `relative` or `percentage` – for the variant's price or weight, when the modifier value is selected on the storefront. | [optional] 
**values.adjusters.adjuster_value** | **float** | The numeric amount by which the adjuster will change the variant's price or the weight, when the modifier value is selected on the storefront. | [optional] 


##### Option
------

Name | Type | Description | Notes
------------ | ------------- | ------------- | -------------
**id** | **int** | The unique numeric ID of the option; increments sequentially. | [optional] 
**product_id** | **int** | The unique numeric ID of the product to which the option belongs. | [optional] 
**name** | **string** | The unique option name, auto-generated from the display name, a timestamp, and the product ID. | [optional] 
**display_name** | **string** | The text displayed to identify the option on the storefront. | [optional] 
**type** | **string** | Possible values: `radio_buttons`, `rectangles`, `dropdown`, `product_list`, `product_list_with_images`, `swatch` | [optional] 
**config** | **object** | (Please see [OptionConfig](#optionconfig).) | [optional] 
**values** | **object** | ?? Collection of values ?? | [optional] 
**values.id** | **int** | The unique numeric ID of the value; increments sequentially. | [optional] 
**values.is_default** | **bool** | The flag for preselecting a value as the default on the storefront. This field is not supported for `swatch` options/modifiers. | [optional] 
**values.label** | **string** | The text displayed to identify the value on the storefront. | [optional] 
**values.sort_order** | **int** | The order in which the value will be displayed on the product page. | [optional] 
**values.value_data** | **object** | Extra data describing the value, based on the value's associated type of option or modifier. A `swatch` type requires an array of colors, with up to three hexidecimal color keys. A `product_list` type requires a `product_id`. A `checkbox` type requires a boolean flag called `checked_value` to determine which value is considered to be the checked state. | [optional]

##### <a name="optionconfig"></a> OptionConfig
------

Name | Type | Description | Notes
------------ | ------------- | ------------- | -------------
**default_value** | **string** | The default value shown for an option. One of: an ISO-8601–formatted string (`date`), a string (`text` or `multi_line_text`), or a numeric string (`numbers_only_text`). | [optional] 
**checked_by_default** | **bool** | (`checkbox`) Flag that sets the checkbox to be checked by default. | [optional] 
**checkbox_label** | **string** | (`checkbox`) Label displayed for the checkbox option. | [optional] 
**date_limited** | **bool** | (`date`) Flag that limits the date allowed to be entered on a date option. | [optional] 
**date_limit_mode** | **string** | (`date`) The type of limit allowed to be entered on a date option (`earliest`, `range`, or `latest`). | [optional] 
**date_earliest_value** | **int** | (`date`) The earliest date allowed to be entered on a date option, as an ISO-8601–formatted string. | [optional] 
**date_latest_value** | **int** | (`date`) The latest date allowed to be entered on a date option, as an ISO-8601–formatted string. | [optional] 
**file_types_mode** | **string** | (`file`) Optional restriction on file types for a file-upload option. One of: `specific` - restricts upload to particular file types; or `all` - allows all files types. | [optional] 
**file_types_supported** | **string[]** | (`file`) The types of files allowed for uploading if the `file_type_option` is set to `specific`. One of: `images` - allows upload of image MIME types (`bmp`,`gif`,`jpg`,`jpeg`,`jpe`,`jif`,`jfif`,`jfi`,`png`,`wbmp`,`xbm`,`tiff`); `documents` - allows upload of document MIME types (`txt`,`pdf`,`rtf`,`doc`,`docx`,`xls`,`xlsx`,`accdb`,`mdb`,`one`,`pps`,`ppsx`,`ppt`,`pptx`,`pub`,`odt`,`ods`,`odp`,`odg`,`odf`); or `other` - allows other file types defined in the `file_types_other` array. | [optional] 
**file_types_other** | **string[]** | (`file`) A list of custom file types allowed for the file upload option. | [optional] 
**file_max_size** | **int** | (`file`) The maximum size of files that can be used in the file upload option. | [optional] 
**text_characters_limited** | **bool** | (`text`, `multi_line_text`) Flag to validate the length of the text of a text or multi-line text input. | [optional] 
**text_min_length** | **int** | (`text`, `multi_line_text`) The minimum length allowed for a text or multi-line text option. | [optional] 
**text_max_length** | **int** | (`text`, `multi_line_text`) The maximum length allowed for a text or multi line text option. | [optional] 
**text_lines_limited** | **bool** | (`multi_line_text`) Flag to validate the maximum number of lines allowed on a multi-line text input. | [optional] 
**text_max_lines** | **int** | (`multi_line_text`) The maximum number of lines allowed on a multi-line text input. | [optional] 
**number_limited** | **bool** | (`numbers_only_text`) Flag to limit the value of a number option. | [optional] 
**number_limit_mode** | **string** | (`numbers_only_text`) The type of limit allowed to be entered on a number option. One of: `lowest`, `highest`, or `range`. | [optional] 
**number_lowest_value** | **float** | (`numbers_only_text`) The lowest allowed value for a number option if `number_limited` is true. | [optional] 
**number_highest_value** | **float** | (`numbers_only_text`) The highest allowed value for a number option if `number_limited` is true. | [optional] 
**number_integers_only** | **bool** | (`numbers_only_text`) Flag to restrict a number option's input to whole numbers only. | [optional] 
**product_list_adjusts_inventory** | **bool** | (`product_list`, `product_list_with_images`) Flag for automatically adjusting inventory on a product included in the list. | [optional] 
**product_list_adjusts_pricing** | **bool** | (`product_list`, `product_list_with_images`) Flag to add the optional product's price to the main product's price. | [optional] 
**product_list_shipping_calc** | **string** | (`product_list`, `product_list_with_images`)  How to factor the optional product&#39;s weight and dimensions into the shipping quote. One of: `none` - don&#39;t adjust; `weight` - use shipping weight only; or `package` - use shipping weight and package dimensions. | [optional] 

##### OptionValue
------

Name | Type | Description | Notes
------------ | ------------- | ------------- | -------------
**id** | **int** | The unique numeric ID of the value; increments sequentially. | [optional] 
**is_default** | **bool** | The flag for pre-selecting a value as the default on the storefront. This field is not supported for `swatch` options/modifiers. | [optional] 
**label** | **string** | The text display identifying the value on the storefront. | [optional] 
**sort_order** | **int** | The order in which the value will be displayed on the product page. | [optional] 
**value_data** | **object** | Extra data describing the value, based on the value's associated type of option or modifier. A `swatch` type requires an array of colors, with up to three hexidecimal color keys. A `product_list` type requires a `product_id`. A `checkbox` type requires a boolean flag called `checked_value` to determine which value is considered to be the checked state. | [optional]

##### Product
------

Name | Type | Description | Notes
------------ | ------------- | ------------- | -------------
**id** | **int** | The unique numeric ID of the product; increments sequentially. | [optional] 
**name** | **string** | The product name. | [optional] 
**type** | **int** | The product type: `physical` - a physical stock unit, or `digital` - a digital download. | [optional] 
**sku** | **string** | User-defined stock keeping unit (SKU)/product code. | [optional] 
**description** | **string** | The product description. Can include HTML formatting. | [optional] 
**weight** | **double** | Weight of the product. Can be used when calculating shipping costs. | [optional] 
**width** | **double** | Width of the product. Can be used when calculating shipping costs. | [optional] 
**depth** | **double** | Depth of the product. Can be used when calculating shipping costs. | [optional] 
**height** | **double** | Height of the product. Can be used when calculating shipping costs. | [optional] 
**price** | **double** | Price of the product. Should either include or exclude tax, based on the store settings. | [optional] 
**cost_price** | **double** | Cost price of the product. Stored for reference only; it is not used or displayed anywhere on the store. | [optional] 
**retail_price** | **double** | Retail cost of the product. If entered, this retail cost price will be shown on the product page. | [optional] 
**sale_price** | **double** | Sale price. If entered, will be used instead of value in the `price` field when calculating the product's cost. | [optional] 
**tax_class_id** | **int** | The ID of the tax class applied to the product. This value will be ignored if automatic tax is enabled. | [optional] 
**product_tax_code** | **string** | Accepts an AvaTax system code. (These codes identify products and services that fall into special sales-tax categories. Where merchants subscribe to Avalara Premium, these codes increase the accuracy of sales-tax calculations. Stores not subscribed to Avalara Premium will ignore this code when calculating sales tax.) Do not pass more than one code. The codes are case-sensitive. For further information and the full list of codes, please refer to: https://help.avalara.com/000_AvaTax_Calc/000AvaTaxCalc_User_Guide/040_Managing_Tax_Profiles/050_Tax_Codes/001_What_is_a_Tax_Code -> "AvaTax System tax codes" section. | [optional] 
**calculated_price** | **double** | The price of the product, unless a `sale_price` is set. | [optional] 
**categories** | **int[]** | An array of IDs for the categories to which this product belongs. If an array of categories is supplied, then when updating a product, all product categories will be overwritten. Does not accept more than 1,000 ID values. | [optional] 
**brand_id** | **int** | The ID associated with the product&#39;s brand. | [optional] 
**inventory_level** | **int** | Current inventory level of the product. Simple inventory tracking must be enabled (see the `inventory_tracking` field) for this to take effect. | [optional] 
**inventory_warning_level** | **int** | Inventory Warning level for the product. When the product's inventory level drops below this warning level, the store owner will be informed. Simple inventory tracking must be enabled (see the `inventory_tracking` field) for this to take effect. | [optional] 
**inventory_tracking** | **string** | The type of inventory tracking for the product: One of: `none` - inventory levels will not be tracked; `product` - inventory levels will be tracked using the `inventory_level` and `inventory_warning_level` fields; or `variant` - inventory levels will be tracked based on variants, which maintain their own warning levels and inventory levels. | [optional] 
**fixed_cost_shipping_price** | **int** | A fixed shipping cost for the product. If defined, this value will be used during checkout, instead of the normal shipping-cost calculation. | [optional] 
**is_free_shipping** | **bool** | Flag used to indicate whether or not the product has free shipping. If `true`, the shipping cost for the product will be zero. | [optional] 
**is_visible** | **bool** | Flag to determine whether or not the product should be displayed to customers browsing the store. If `true`, the product will be displayed; if `false`, the product will be hidden from view. | [optional] 
**is_featured** | **bool** | Flag to determine whether or not the product should be included in the "Featured Products" panel when viewing the store. | [optional] 
**warranty** | **string** | Warranty information displayed on the product page. Can include HTML formatting. | [optional] 
**bin_picking_number** | **string** | The BIN picking number for the product. | [optional] 
**layout_file** | **string** | The layout template file used to render this product. | [optional] 
**upc** | **string** | The product's UPC code. Used in feeds for shopping comparison sites and external channel integrations. | [optional] 
**search_keywords** | **string** | A comma-separated list of keywords that can be used to locate the product when searching the store. | [optional] 
**availability** | **string** | Availability of the product. One of: `available` - the product can be purchased on the storefront; `disabled` - the product is listed on the storefront, but cannot be purchased; `preorder` - the product is listed for pre-orders. | [optional] 
**availability_description** | **string** | Availability text displayed on the checkout page (under the product's title), telling the customer how long it will normally take to ship this product. Example value: "Usually ships in 24 hours". | [optional] 
**gift_wrapping_options** | **int[]** | A list of IDs for gift-wrapping options. One of: `0` – allow any gift-wrapping options in the store; or `-1` – disallow gift-wrapping on the product. | [optional] 
**sort_order** | **int** | Priority to give this product when included in product lists on category pages and search results. The lower the number, the closer the product will be to the top of the displayed results. | [optional] 
**condition** | **string** | The product's condition. Will be shown on the product page if the `is_condition_shown` field's value is `true`. Possible values for this field: `New`, `Used`, `Refurbished`. | [optional] 
**is_condition_shown** | **bool** | Flag used to determine whether or not the product condition is shown to the customer on the product page. | [optional] 
**order_quantity_minimum** | **int** | The minimum quantity an order must contain in order to purchase this product. | [optional] 
**order_quantity_maximum** | **int** | The maximum quantity an order can contain when purchasing the product. | [optional] 
**page_title** | **string** | Custom title for the Products page. If this value is not defined, the product name will be used as the meta title. | [optional] 
**meta_keywords** | **string[]** | Custom meta keywords for the product page. If not defined, the store default keywords will be used. | [optional] 
**meta_description** | **string** | Custom meta description for the product page. If not defined, the store default meta description will be used. | [optional] 
**date_created** | **string** | The date when the product was created. | [optional] 
**date_modified** | **string** | The date when the product was modified. | [optional] 
**view_count** | **int** | The number of times the product has been viewed. | [optional] 
**preorder_release_date** | **string** | Pre-order release date. See the `availability` field for details on setting a product's availability to accept pre-orders. | [optional] 
**preorder_message** | **string** | Custom "expected date" message to display on the product page. If undefined, the message defaults to the storewide setting. Can contain the `%%DATE%%` placeholder, which will be substituted for the release date. | [optional] 
**is_preorder_only** | **bool** | If set to `false`, the product will _not_ change its availability from `pre-order` to `available` on the release date. Otherwise, the product's availability/status will change to `available` on the release date. | [optional] 
**is_price_hidden** | **bool** | The default `false` value indicates that this product's price will be shown on the product page. If set to `true`, the price will be hidden. (Note: To successfully set `is_price_hidden` to `true`, the `availability` value must be `disabled`.) | [optional] 
**price_hidden_label** | **string** | By default, an empty string. If `is_price_hidden` is `true`, the `price_hidden_label` value will be displayed instead of the price. (Note: To successfully set a non-empty string value for `price_hidden_label`, the `availability` value must be `disabled`.) | [optional] 
**images** | **object** | ?? [Description to be added] ?? | [optional] 
**images.id** | **int** | The unique numeric ID of the image; increments sequentially. | [optional] 
**images.product_id** | **int** | The unique numeric identifier for the product associated with the image. | [optional] 
**images.is_thumbnail** | **bool** | Flag for identifying whether or ot the image is used as the product&#39;s thumbnail. | [optional] 
**images.sort_order** | **int** | The order in which the image will be displayed on the product page. Lower values have higher priority. When an image is updated to a lower priority (i.e., a greater `images.sort_order` value), all images with values matching or exceeding the image's new `images.sort_order` value will have their `images.sort_order` values rewritten. | [optional] 
**images.description** | **string** | Description for the image. | [optional]
**images.image_file** | **string** | The local path to the original image file that was uploaded to BigCommerce. | [optional] 
**images.url_zoom** | **string** | The zoom URL for this image. | [optional] 
**images.url_standard** | **string** | The standard URL for this image. | [optional] 
**images.url_thumbnail** | **string** | The thumbnail URL for this image. | [optional] 
**images.url_tiny** | **string** | The tiny URL for this image. | [optional] 
**custom_fields** | **object** | ?? Collection of custom fields. ?? | [optional] 
**custom_fields.id** | **int** | The unique numeric ID of the custom field; increments sequentially. | [optional] 
**custom_fields.name** | **string** | The name of the field, as shown on the storefront: "orders", etc. | [optional] 
**custom_fields.value** | **string** | The values or text of the field, as shown on the storefront" "orders", etc. | [optional] 
**custom_url** | **object** | ?? [Description to be added] ?? | [optional] 
**custom_url.url** | **string** | Product URL on the storefront. | [optional] 
**custom_url.is_customized** | **bool** | Returns `true` if the URL has been changed from its default state (i.e., the auto-assigned URL BigCommerce provides). | [optional] 
**bulk_pricing_rules** | **object** | ?? Collection of bulk pricing rules. ?? | [optional] 
**bulk_pricing_rules.id** | **int** | The ID of the bulk pricing rule. | [optional] 
**bulk_pricing_rules.quantity_min** | **int** | The minimum (inclusive) quantity of a product to satisfy this rule.  Must be greater than or equal to 0 (zero). | [optional] 
**bulk_pricing_rules.quantity_max** | **int** | The maximum (inclusive) quantity of a product to satisfy this rule. Must be greater than the `bulk_pricing_rules.quantity_min` value – unless this field has a value of zero, in which case there will be no maximum bound for this rule. | [optional] 
**bulk_pricing_rules.type** | **string** | The type of bulk pricing adjustment that is made. One of: `price` - adjustment amount per product; `percent` - adjustment as a percentage of the original price; or `fixed` - adjusted absolute price of the product. | [optional] 
**bulk_pricing_rules.amount** | **int** | The value of the ?? adjusted by the bulk pricing rule. | [optional] 
**variants** | **object**  | (Please see [Variant](#variant).) | [optional] 

##### <a name="variant"></a> Variant
------

Name | Type | Description | Notes
------------ | ------------- | ------------- | -------------
**id** | **int** | ?? [Description to be added] ?? | [optional] 
**product_id** | **int** | ?? [Description to be added] ?? | [optional] 
**sku** | **string** | ?? [Description to be added] ?? | [optional] 
**sku_id** | **int** | Read-only reference to a v2 SKU ID. Will be `null` on a base variant. | [optional] 
**cost_price** | **string** | The cost price of the variant. | [optional] 
**price** | **string** | This variant&#39;s base price on the storefront. If this value is `null`, the product&#39;s default price (set in the Product resource&#39;s `price` field) will be used as the base price. | [optional] 
**weight** | **string** | This variant&#39;s base weight on the storefront. If this value is `null`, the product&#39;s default weight (set in the Product resource&#39;s `weight` field) will be used as the base weight. | [optional] 
**purchasing_disabled** | **bool** | If `true`, this variant will not be purchasable on the storefront. | [optional] 
**purchasing_disabled_message** | **string** | If `purchasing_disabled` is `true`, this message will show on the storefront when the non-purchasable variant is selected. | [optional] 
**image_file** | **string** | The image that will be displayed when this variant is selected on the storefront. When updating a SKU image, send the publicly accessible URL. Supported image formats are JPEG, PNG, and GIF. Generic product images that are _not_ specific to the variant should be stored on the product. | [optional] 
**upc** | **string** | The variant's UPC code. Used in feeds for shopping comparison sites and external channel integrations. | [optional] 
**inventory_level** | **int** | Inventory level for the variant. Used when the product&#39;s `inventory_tracking` value is set to `variant`. | [optional] 
**inventory_warning_level** | **int** | When the variant hits this inventory level, it is considered low-stock. | [optional] 
**bin_picking_number** | **string** | Identifies where in a warehouse the variant is located. | [optional] 
**option_values** | **object** | Array of option, and option values, IDs that make up this variant. Will be empty if the variant is the product&#39;s base variant. | [optional] 
**option_values.id** | **int** | ?? [Description to be added] ?? | [optional] 
**option_values.option_id** | **int** | ?? [Description to be added] ?? | [optional] 


##### VariantPost
------

Name | Type | Description | Notes
------------ | ------------- | ------------- | -------------
**id** | **int** | ?? [Description to be added] ?? | [optional] 
**product_id** | **int** | ?? [Description to be added] ?? | [optional] 
**sku** | **string** | ?? [Description to be added] ?? | [optional] 
**sku_id** | **int** | Read-only reference to a v2 SKU ID. Will be `null` on a base variant. | [optional] 
**price** | **string** | This variant&#39;s base price on the storefront. If this value is `null`, the product&#39;s default price (set in the Product resource&#39;s `price` field) will be used as the base price. | [optional] 
**weight** | **string** | This variant&#39;s base weight on the storefront. If this value is `null`, the product&#39;s default weight (set in the Product resource&#39;s `weight` field) will be used as the base weight. | [optional] 
**purchasing_disabled** | **bool** | If set to `true`, this variant will not be purchasable on the storefront. | [optional] 
**purchasing_disabled_message** | **string** | If `purchasing_disabled` is true, this message will show on the storefront when the non-purchasable variant is selected. | [optional] 
**image_file** | **string** | The image that will be displayed when this variant is selected on the storefront. When updating a SKU image, send the publicly accessible URL. Supported image formats are JPEG, PNG, and GIF. Generic product images that are not specific to the variant should be stored on the product. | [optional] 
**cost_price** | **string** | The variant&#39;s cost price. | [optional] 
**upc** | **string** | The variant's UPC code. Used in feeds for shopping comparison sites and external channel integrations. | [optional] 
**inventory_level** | **int** | Inventory level for the variant. Used when the product&#39;s inventory_tracking is set to `variant`. | [optional] 
**inventory_warning_level** | **int** | When the variant hits this inventory level, it is considered low-stock. | [optional] 
**bin_picking_number** | **string** | Identifies where in a warehouse the variant is located. | [optional] 
**option_values** | **object** | ?? [Description to be added] ?? | [optional] 
**option_values.option_display_name** | **string** | The name of the option to be created on POST. | [optional] 
**option_values.label** | **string** | The label of the option value to be created on POST. | [optional] 



