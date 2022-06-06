# WooGraphQL / WPGatsby Featured Image Bug

**Error demo**

This repo only exists as a demo for the issue reported here:

https://github.com/wp-graphql/wp-graphql-woocommerce/issues/620


## Description

Gatsby doesn't create nodes for featured images when WordPress plugin "WPGraphQL WooCommerce (WooGraphQL)" is active.

 - GraphiQL IDE `mediaItems` returns all media nodes correctly.
 - Altair GraphQL client `mediaItems` returns all media nodes correctly.
 - Gatsby (`/___graphql`) `allWpMediaItem` returns only content-inline media nodes, NOT `featuredImage` nodes.
 - Gatsby (`/___graphql`) `allWpPage` and `allWpPost` return ID values for `featuredImageId`, even though the `featuredImage` nodes are `null`.

## WP Debug Logs

This error is produced on `gatsby build`:
```
[06-Apr-2022 03:39:47 UTC] PHP Warning:  Trying to access array offset on value of type bool in /var/www/woo-gatsby-demo.bigupweb.uk/wp-content/plugins/wp-gatsby/src/Schema/WPGatsbyWPGraphQLSchemaChanges.php on line 54
```

## The Reproduction Environment

**Gatsby**
[This repo](https://github.com/BigupWeb/gatsby-wordpress-woo-featured-image-bug) is a clone of the "Gatsby WordPress Blog Starter". The only edits, are the setting of the WordPress source url in gatsby-config.js, and the README.

**WordPress Version: 5.9.3**
[This self hosted demo](https://woo-gatsby-demo.bigupweb.uk) is a fresh install with only the basic setup completed and 6 images uploaded via the media library.
[WordPress GraphQL Endpoint](https://woo-gatsby-demo.bigupweb.uk/graphql)

**WordPress Plugins**
 - WooCommerce 6.3.1
 - WPGraphQL WooCommerce (WooGraphQL) 1.7.2
 - WP Gatsby 2.3.2
 - WP GraphQL 1.7.2

**Test Images**
The 6 images are titled and alt-texted as below, which reflects their location in published posts to aid debugging in GraphQL output:

```
page-featured
page-inline
post-featured
post-inline
product-gallery-image
product-main-image
```

## Reproduce the Bug

1. Clone [this repo](https://github.com/BigupWeb/gatsby-wordpress-woo-featured-image-bug) and npm install.

2. Run `gatsby clean && gatsby develop` (the WP source is hard-coded in config, so no setup required).

3. Browse `http://localhost:8000/___graphql` and copy in these test queries:

```
query MediaQuery {
  allWpMediaItem {
    nodes {
    	id
    	altText
    }
  }
}

query PostQuery {
  allWpPost {
    nodes {
      title
      featuredImageId
      featuredImage {
        node {
          id
          altText
        }
      }
    }
  }
}

query PageQuery {
  allWpPage {
    nodes {
      title
      featuredImageId
      featuredImage {
        node {
          id
          altText
        }
      }
    }
  }
}
```

4. Run each query and observe the following:

 - `allWpMediaItem` only returns the 2 `inline` images, not the featured.
 - `allWpPost` returns the post "Hello world!" with a `featuredImageId`, but a `null` `featuredImage` node.
 - `allWpPage` returns 5 pages, "Sample Page" has a featured image set, but again the `featuredImage` node is `null` even though the `featuredImageId` is populated.
 - `allWpProduct` returns no image nodes at all.

 ## Expected Results

 As soon as the WooGraphQL plugin is disabled, the image nodes appear after the next `gatsby build`. See the produced results below, noteably, the featured images now appear in all cases. Only one page has a featured image, so the other four nulls can be ignored.

 In this reproduction instance, I will leave the plugin ENABLED so the API can be interrogated with the bug present.

 ```
##
# Output With WooGraphQL Disabled
##

# Media Items
"allWpMediaItem": {
	"nodes": [
		{
			"id": "cG9zdDoyMg==",
			"altText": "post-inline"
		},
		{
			"id": "cG9zdDoyNg==",
			"altText": "page-inline"
		},
		{
			"id": "cG9zdDoyNA==",
			"altText": "post-featured"
		},
		{
			"id": "cG9zdDoyOA==",
			"altText": "page-featured"
		}
	]
}

# Posts
"allWpPost": {
	"nodes": [
		{
			"title": "Hello world!",
			"featuredImageId": "cG9zdDoyNA==",
			"featuredImage": {
			"node": {
				"id": "cG9zdDoyNA==",
				"altText": "post-featured"
			}
			}
		}
	]
}

# Pages
"allWpPage": {
	"nodes": [
		{
			"title": "My account",
			"featuredImageId": null,
			"featuredImage": null
		},
		{
			"title": "Checkout",
			"featuredImageId": null,
			"featuredImage": null
		},
		{
			"title": "Cart",
			"featuredImageId": null,
			"featuredImage": null
		},
		{
			"title": "Shop",
			"featuredImageId": null,
			"featuredImage": null
		},
		{
			"title": "Sample Page",
			"featuredImageId": "cG9zdDoyOA==",
			"featuredImage": {
			"node": {
				"id": "cG9zdDoyOA==",
				"altText": "page-featured"
			}
			}
		}
	]
}

# Products
# Woo plugin disabled, so cannot test. 

```
