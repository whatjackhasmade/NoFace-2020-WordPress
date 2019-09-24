
# wp-graphql-gutenberg

## What’s Included?

* Supports post_types that return `true` for `use_block_editor_for_post_type` and are allowed through wp-graphql
* Supports blocks `deprecation`.
* Supports reusable blocks (`wp_block` post_type)
* Supports server side rendered blocks registered through `register_block_type`
* Supports posts edited prior to this plugin activation through admin interface

### 👉  `How does it work`
If you are familiar with gutenberg, you wonder how this thing works since the core of the gutenberg is written in javascript. Since gutenberg uses WP REST API, this plugin intercepts every post save (PUT) request and adds metadata to request's payload.

The metadata contains block type definitions from  `get_block_types()`, parsed reusable blocks and posts's blocks. These are than processed on the server side and saved to post's meta key `wp_graphql_gutenberg` and `wp_graphql_gutenberg_block_types` option respectively. Upon saving, the data is further processed to be able to know all `__typename` when resolving.

Since this plugin generates its content upon saving a post, there is a option in admin, to resave all posts on the background so you can get definitions for posts which were edited and saved before activation of this plugin. This is done through hidden `<iframe>` blackmagic.

### 👉  `Why php parser is not used`
There is also a wordpress's php `parse_blocks()` function. However the attributes which are parsed from html nodes are not parsed and therefore not available. Also there is no block_type registry on the server, so you have no way to know how the blocks are defined.

### 👉  `Block type deprecation and types`
Since block attributes can be deprecated in block type definition, this plugin tries to reuse graphql type upon breaking changes. Good example of this situation is `core/paragraph` block where fontSize attribute changes from `number` to `string`. Since there is no way to make union of scalar types in graphql a new version of type for attributes is created. That's why u see `CoreParagraphBlockAttributesV2` or `CoreParagraphBlockAttributesV3`. When the block is saved, its attributes `__typename` is also resolved and saved, so you can be sure that you get correct version when quering.
### 👉  `How to query?`

```graphql
query posts {
  postBy(slug: "hello-world") {
    id
    blocks {
      __typename
      ... on CoreHeadingBlock {
        name
        attributes {
          __typename
          ... on CoreHeadingBlockAttributes {
            content
            level
          }
        }
      }
      ... on CoreParagraphBlock {
        name
        attributes {
          __typename
          ... on CoreParagraphBlockAttributes {
            backgroundColor
            content
          }
          ... on CoreParagraphBlockAttributesV3 {
            fontSize
            content
          }
        }
      }
    }
  }
}
```

```json
{
  "data": {
    "postBy": {
      "id": "cG9zdDox",
      "blocks": [
        {
          "__typename": "CoreHeadingBlock",
          "name": "core/heading",
          "attributes": {
            "__typename": "CoreHeadingBlockAttributes",
            "content": "Welcome to WordPress.",
            "level": 2
          }
        },
        {
          "__typename": "CoreParagraphBlock",
          "name": "core/paragraph",
          "attributes": {
            "__typename": "CoreParagraphBlockAttributesV3",
            "fontSize": null,
            "content": "Welcome to WordPress. This is your first post. Edit or delete it, then start writing!"
          }
        }
      ]
    }
  }
}
```



### 👉  `WP Filters`

#### PHP

```php
graphql_gutenberg_block_attributes_fields
  
/* Filters the fields for block attributes type.
  *
  * @param object    $fields           Fields config.
  * @param string    $type_name        GraphQL type name.
  * @param array     $attributes 	  Block type attributes definition.
  * @param array     $block_type 	  Block type definition.
  */

graphql_gutenberg_block_type_fields
/* Filters the fields for block type.
*
* @param object    $fields           Fields config.
* @param array     $block_type 	  Block type definition.
*/

graphql_gutenberg_prepare_block
/* Filters block data before saving to post meta.
*
* @param object    $data             		Data.
* @param array     $block_types_per_name 	GraphQL types named array for blocks.
*/

graphql_gutenberg_register_block_type
/* Filters block type graphql config.
*
* @param object    $block_type             GraphQL type for block type.
*/

```

#### JS

``` js
wpGraphqlGutenberg.postContentBlocks
/* Filters post_content_blocks field in PUT request.
*
* @param array    postContentBlocks
*/
wpGraphqlGutenberg.reusableBlock
/* Filters reusable_block field in PUT request.
*
* @param array    reusableBlock
*/
wpGraphqlGutenberg.reusableBlocks
/* Filters reusable_blocks field in PUT request.
*
* @param array    reusableBlocks
*/
```


### 👉  `Need support for ACF blocks (PRO version)?`

Check out this [plugin](https://github.com/pristas-peter/wp-graphql-gutenberg-acf), which is build upon this plugin.