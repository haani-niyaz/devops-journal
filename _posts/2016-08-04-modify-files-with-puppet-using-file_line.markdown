---
layout: post
title:  "Modify Files with Puppet Using file_line"
author: "Haani Niyaz"
tags:  puppet file_line create_resource
---


## Requirements

Recently we needed to manage a set of credentials sprawled across multple files. Puppet was already managing our rpm deployments. However the configuration was slated to be managed via a configuration server(not Puppet). In the interim we promptly needed to ensure credentials were updated in place within multiple configuration files when the rpm was deployed.


## Design

The files contained key value pairs and only the credentials within the file had to be managed. All other configuration parameters needed to be left intact. 


### Where to put credentials?

This was the easy part. Considering the credentials were site specific it belonged in hiera.

> *Site specific data belongs in heira* 

Knowing that we would want to write DRY code to update the credentials in all the files, 
it made sense to put them to a hash we can iterate over. I called it `credentials`. 


#### Secure your Data

If you are storing credentials or sensitive data in hiera, makes sure they are encrypted using [hiera-eyaml](https://github.com/TomPoulton/hiera-eyaml) since they are more often than not stored in source control.


**Example hiera yaml file:**

{% highlight yaml %}
credentials:
  payment:
   keystore:  
    key: keystore.password 
    value: CFNEepR8WMZJLdWjwrjAqoDr8password
    file_path: /var/conf/connector.properties
  tenant1:
    youtube:
      t1-secret:
        key: youtube.secretKey
        value:  oPO2y95ZDmbTsh14pTba8r9T6RQnv6password
        file_path: /var/conf/common/tenants/tenant1/tenant1.properties
      t1-api:
        key: youtube.apiKey
        value: A5yhq89F7R86JgdOZ2wILxqoShUb/7fpassword
        file_path: /var/conf/common/tenants/tenant1/tenant1.properties
    db1:
      t1-ident:
        key: tenant.db.db1.password
        value: password
        file_path: /var/conf/common/tenants/tenant1/tenant1.properties
    db2:
      t1-pref:
        key: tenant.db.dbStore.password
        value: password
        file_path: /var/conf/common/tenants/tenant1/tenant1.properties
    account:
      t1-account-password:
        key: comEngine.password
        value: AK5imwsopFsqT+nWe5PWKykHv9p2xQbpassword
        file_path: /var/conf/common/tenants/tenant1/tenant1.properties
    multimedia:
      t1-multimedia-password:
        key: voucher.password
        value: password
        file_path: /var/conf/common/tenants/tenant1/tenant1.properties
    mail:
      t1-mail-password:
        key: tenant.smtp.password
        value: password
        file_path: /var/conf/common/tenants/tenant1/tenant1.properties
    billing:
      t1-password:
        key: tenant.billing.password
        value: password
        file_path: /var/conf/common/tenants/tenant1/tenant1.properties
      t1-consumerKey:
        key: tenant.billing.consumerKey
        value: BgD2FvIKo9OkNJIRU9yc28QYW6x2ah3password
        file_path: /var/conf/common/tenants/tenant1/tenant1.properties
      t1-consumerSecret:
        key: tenant.billing.consumerSecret
        value:  j08ZEo9TaGc/befTK4/6N6Jpassword
        file_path: /var/conf/common/tenants/tenant1/tenant1.properties
      t1-secretToken:
        key: tenant.billing.secretToken
        value: DR0P/FQMWGrB3sVKz4A/GFFRpassword
        file_path: /var/conf/common/tenants/tenant1/tenant1.properties
  tenant2:
    youtube:
      t2-secret:
        key: youtube.secretKey
        value:  S/qADgSRKYtoyLaZ8i4t5w+4UoOkTqpassword
        file_path: /var/conf/common/tenants/tenant2/tenant2.properties
      t2-api:
        key: youtube.apiKey
        value: BHLZjqZqqFuZkwdbKuYMioOdx0UnKfwpassword
        file_path: /var/conf/common/tenants/tenant2/tenant2.properties
    db1:
      t2-ident:
        key: tenant.db.db1.password
        value: password
        file_path: /var/conf/common/tenants/tenant2/tenant2.properties
    db2:
      t2-pref:
        key: tenant.db.dbStore.password
        value: password
        file_path: /var/conf/common/tenants/tenant2/tenant2.properties
    account:
      t2-account-password:
        key: comEngine.password
        value: Ar0pfLDAdcvq4yAfZ3Tzj+v1AReq7Xmpassword
        file_path: /var/conf/common/tenants/tenant2/tenant2.properties
    mail:
      t2-mail-password:
        key: tenant.smtp.password
        value: password
        file_path: /var/conf/common/tenants/tenant2/tenant2.properties
    billing:
      t2-password:
        key: tenant.billing.password
        value: password
        file_path: /var/conf/common/tenants/tenant2/tenant2.properties
      t2-consumerKey:
        key: tenant.billing.consumerKey
        value: BgrCbS4et4Dd8dxM6X4er8tZSYYWvGnpassword
        file_path: /var/conf/common/tenants/tenant2/tenant2.properties
      t2-consumerSecret:
        key: tenant.billing.consumerSecret
        value: DaR7mYBL6WIi2+Gfb0P0RHZtpassword
        file_path: /var/conf/common/tenants/tenant2/tenant2.properties
      t2-secretToken:
        key: tenant.billing.secretToken
        value: AFFqzrxtxQSudrFYFPZRdPGJpassword
        file_path: /var/conf/common/tenants/tenant2/tenant2.properties
  {% endhighlight %}



It worth observing a few things in the snippet above. The `credentials` hash is structured with a parent element i.e: `payment`, `tenant1`, `tenant2`. 

Each parent element either has 1 top level child element or nested child elements. If there are nested child elements you will notice an extended hierachy as illustrated below:


#### Single child

Parent element `payment` has child `keystore`. Easy.

{% highlight ruby %}
 payment:
   keystore:  
    key: keystore.password 
    value: CFNEepR8WMZJLdWjwrjAqoDr8password
    file_path: /var/conf/connector.properties
{% endhighlight %}

#### Nested children

Here the hierachy is extended with `youtube` followed by child elements `t1-secret` and `t1-api.`

{% highlight ruby %}
tenant1:
    youtube:
      t1-secret:
        key: youtube.secretKey
        value:  oPO2y95ZDmbTsh14pTba8r9T6RQnv6password
        file_path: /var/conf/common/tenants/tenant1/tenant1.properties
      t1-api:
        key: youtube.apiKey
        value: A5yhq89F7R86JgdOZ2wILxqoShUb/7fpassword
        file_path: /var/conf/common/tenants/tenant1/tenant1.properties
{% endhighlight %}


the lowest hanging child element has 3 distinct properties called `key`, `value` and `file_path`. 


### How to update the files?

Now that we have our data structured, let's take a look at accessing them and manging credentials programatically.

I created a defnition which accepts the `key`, `value` and `file_path` as parameters.


{% highlight  ruby%}
# manage_credentials.pp
define manage_credentials(
	$key, 
	$value, 
	$file_path) {

    # Do something
}
{% endhighlight %}


So there's a few things to recognize here. Puppet's golden rule is *resources must be unique*. To faciliate this you will notice how all elements in the hash with properties have unique names. This becomes the `title` of the defined type. the properties `key`, `value` and `file_path` become the parameters to the defined type.

Now we just plug this into the [create_resouces](https://docs.puppet.com/puppet/latest/reference/function.html#createresources) function like so:

{% highlight  ruby%}
create_resources(manage_credentials, $credentials['tenant1']['youtube'])
create_resources(manage_credentials, $credentials['tenant1']['db1'])
...
{% endhighlight %}


Inside the `mananged_credentials` defined type we use the `file_line` type from the [stdlib](https://forge.puppet.com/puppetlabs/stdlib) module to replace the the `match` with the `line` provided. For example:



{% highlight  ruby%}
# file_line pseudo code

file_line { 'change password for youtube.secretKey in /var/conf/test.properties':
	path  => '/var/conf/test.properties',
	line  => 'youtube.secretKey=password',
	match => 'youtube.secretKey'
}
{% endhighlight%}


Our final piece of code:

{% highlight  ruby%}
# manage_credentials.pp
define ovs_mule::app::manage_credentials(
	$key, 
	$value, 
	$file_path) {
    	
	file_line { "change password for $key in $file_path":
		  path   => $file_path,
		  line   => "$key=$value",
		  match  => "$key",
	}

}
{% endhighlight%}


Once again, keep in mind that the `file_line` resource `title` needs to be unique. To satisfy this, I am using a combination of the `key` and `file_path` property in the `title`.








