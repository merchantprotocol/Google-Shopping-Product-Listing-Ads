# GoogleShoppingAPI v2
#### Product Listing Ads
#### PLAs

Find out more at Merchant Protocol, You're [Magento Support](https://merchantprotocol.com/) Partner.

# Update to 0.2.4

IMPORTANT!

If you are updating to 0.2.4 from an earlier version, please be aware that the
category ids have changed. 
There is a [shell script](shell/googleshopping_taxonomy_mapping/Readme.md) 
which might help you to map the old category ids to the new ones.

Fortunately Google has added static category ids which makes it easier for 
future updates.

## Magento Module GoogleShoppingAPI

This module is based on the official Magento GoogleShopping module and enhances
the original module features with APIv2 support (APIv1 support removed),
OAuth2 support and several additional features from the original 
EnhancedGoogleShopping module.

If the original Magento GoogleShopping module is installed, data will be migrated.

~~The observer (auto sync after saving product changes) is disabled in the current
version, but will be re-enabled soon.~~

The observer was re-enabled with version 0.1.0 . To prevent problems when users
are editing products which have no access to GoogleShopping through OAuth2, products
are only updated on GoogleShopping if a valid access token for the store exists.

To authenticate and get an access token go to Magento Admin -> Catalog -> Google 
Content APIv2 and select the store view in which you want to authenticate. 
After selecting a store view without valid access token you will be automatically
redirected to OAuth2 authentication.

## Features


* update item expiration date on sync
* option to renew not listed items on sync
* option to remove disabled items on sync
* convert html entities in description to UTF-8 chars
* strip tags from description
* make sales price available in countries outside the US
* possibility to define a separate google shopping image with base image fallback
* option to add Google Analytics source to product link (utm_source=GoogleShopping)
* option to add custom parameters to product link
* adds Austria as target country
* ability to set Google product category in Magento product details
* ability to use OAuth2 service account
* Automated daily sync to Google Content
  * Option to enable by StoreView
  * At the moment the products are synced every day at 02:30 (cron)
  * Needs improvement in future versions (e.g. batch processing, option to set sync period)

## Installation

### Install using composer

As the Google ApiClient must be installed in addition, it is recommended to 
install using composer.

Create or adapt the composer.json file in your Magento root directory with the 
following content:

```json
{
	"require": {
		"bluevisiontec/googleshoppingapi": "*",
		"magento-hackathon/magento-composer-installer": "*",
		"google/apiclient": "*"
	},
	"repositories": [
		{
			"type": "composer",
			"url": "http://packages.firegento.com"
		},
		{
				"type": "vcs",
				"url": "https://github.com/bluevisiontec/GoogleShoppingApi"
		}
	],
	"extra": {
		"magento-root-dir": "./",
		"magento-deploystrategy": "copy"
	}
}
```

#### Install composer
```bash
mkdir bin
curl -s https://getcomposer.org/installer | php -- --install-dir=bin
php bin/composer.phar install
```

### Install manually

* Copy app, js, var to your magento root directory
* Download Google Content API Client for PHP Release 1.1.2: https://github.com/google/google-api-php-client/archive/1.1.2.tar.gz
	* Thanks to @damek132 for notice
	* Please not that version 1.1.4 has an error which will be fixed in 1.1.5
	* see https://github.com/google/google-api-php-client/commit/818b20c291b074a609da633d243bf61bcf7dfaac
* Install Google Content API to [MAGENTO_ROOT]/vendor/google/apiclient/
* You should have at least the autoload.php file and the src folder in [MAGENTO_ROOT]/vendor/google/apiclient/

### After installation

* Clean the Magento Cache and log out of the backend

## Configuration

As the module has to use Google OAuth2, credentials for Google
Content API are required. Those can be generated in the 
http://console.developers.google.com/

You can choose between using a Client ID for web application or using a service account.
If the Client ID for web application is used a manual user interaction is needed to provide
access to Google Content API. In this case automated processes like cron jobs are not available.

Using a service account is recommended and needed if you want to use automated processes.

### Using Service account

#### Create a project in Google developers console

* Login to Google developers console or create an account
* Create a Project
  * Name: Magento-GoogleShoppingApi
  * Project-ID: use the generated id or something like magento-gshopping-841
* After the project is created go to "APIs & auth" -> "APIs"
* Search for "Content API for Shopping" and enable it
* Next go to "APIs & auth" -> "Credentials" and click "Create new Client ID"
* Select "Service account"
  * Select "P12 Key" as key type
  * Click on "Create Client ID"
  * Save the P12 file and write down the private key's password
  
#### Allow the service account to access your merchant center account
  
* Login to Google Merchant center
* Go to Settings -> Users
* Click on "+User"
  * Enter the email address of your service account as "User email address" (*****@developer.gserviceaccount.com)
  * Select "Standard access" as access level

### Using Client ID for web application

#### Create a project in Google developers console

* Login to Google developers console or create an account
* Create a Project
  * Name: Magento-GoogleShoppingApi
  * Project-ID: use the generated id or something like magento-gshopping-841
* After the project is created go to "APIs & auth" -> "APIs"
* Search for "Content API for Shopping" and enable it
* Next go to "APIs & auth" -> "Credentials" and click "Create new Client ID"
* Select "Web application"
  * Fill out the fields "Email address" and "Product name"
  * save
* In the next step the shop backend data has to be enterend
  * "Authorized JavaScript origins": https://www.yourmagentobackend.com/
  * "Authorized redirect uris":
  * https://www.yourmagentobackend.com/index.php/admin/googleShoppingApi_oauth/auth/
* After finishing the process you can see your API credentials
  * Client ID and Client Secret must be entered in the Magento Module Configuration

### Magento Module Configuration

* Basic Module configuration: Magento Admin -> System -> Configuration -> 
BlueVisionTec Modules -> GoogleShoppingApi

  * Account-ID: Your GoogleShopping Merchant ID
  * Use service account: Use Client ID for web application or Service account (as mentioned above)
  * Google Developer Project Client ID: The Client ID generated above
  * Google Developer Project Client Secret: The Client Secret generated above (Client ID for web application only)
  * Google Developer Project E-Mail: The E-Mail address from your credentials
  * Google Developer Project Private Key file: upload the P12 file here (Service account only)
  * Google Developer Project Private Key password: The private key's password (Service account only)
  * Target Country: The country for which you want to upload your products
  * Update Google Shopping Item when Product is Updated
	* ~~Not implemented (observer disabled in current version, will be readded)~~
  * Renew not listed items
  * When syncing a product which is not listed on GoogleShopping, it will be added
  * Remove disabled items
  * Removes items which are disabled or out of stock from GoogleShopping

* Product configuration
  * In Product edit view you will find a new tab "GoogleShopping". 
    Here you can set the GoogleShopping Category. 
    The language of the category is taken from the configured store language.
    The taxonomy files for de_DE and en_US are shipped with the module package.
    Further taxonomy files should be added to /var/bluevisiontec/googleshoppingapi/data .
  * Links to taxonomy files:
    * http://www.google.com/basepages/producttype/taxonomy.en-US.txt
    * http://www.google.com/basepages/producttype/taxonomy.de-DE.txt
    
* Attributes configuration and item management can be found in Magento Admin ->
  Catalog -> Google Content APIv2

* Before uploading an item you will have to set the attribute mapping
	* Magento Admin -> Catalog -> Google Content API V2 -> Manage attributes
	* See https://support.google.com/merchants/answer/1344057 for requirements
	* Example for default attribute set
		* SKU => Manufacturer's Part Number (MPN)
		* Condition => Condition
			* You might have to add the attribute condition as DropDown with the Options new, refurbished, used
		* Name => Title
		* Description => Description
		* Price => Price 
			* Sales price is taken if set
		* EAN13 => GTIN
			* You need 2 out of 3 (MPN, GTIN, Brand), so you might add a similar attribute
		* Manufacturer => Brand
		
![GoogleShoppingAPI attribute mapping](docs/images/attribute-mapping.png)

## Shell script

**NOT YET READY FOR PRODUCTION**

### Examples

Sync all items in storeid 1

```
php -f shell/googleshopping.php -- --action syncitems --storeid 1
```
