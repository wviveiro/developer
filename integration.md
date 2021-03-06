# Integrating With Your Site

If you'd like to be able to programmatically track a user through the checkout process, this page is for you.


## Passing a Custom ID

### Linked Checkout

In order to track a known user from your site through a MoonClerk checkout you can pass a `cid` parameter (short for custom_id) into the checkout. If you are linking to your checkout you can simply tack it onto the URL. If your user ID is 234, you can add the following parameter to your checkout URL:

```
https://app.moonclerk.com/pay/8h7frjfytj
```

becomes

```
https://app.moonclerk.com/pay/8h7frjfytj?cid=234
```

**A note about security. All traffic to and from MoonClerk is encrypted with SSL which includes the URL. If you are concerned about sending real IDs over the URL you can create a MD5 or SHA hash based on the id and timestamp, store it with the user in your database, and pass that parameter instead of the acutal ID.**


### Embedded Checkout

If you are using an embed code for your form, you'll need to dynamically add the `cid` to the `opts` object. Looking inside the embed code, you'll find an `opts` object similar to this:

```
opts={"checkoutToken":"8h7frjfytj","width":"100%"};
```

You'll need to add the cid as follows:

```
opts={"checkoutToken":"8h7frjfytj","width":"100%","cid":"234"};
```

Here you can see the entire embed snippet with the updated `opts` object:

```
<div id="mc8h7frjfytj"><a href="http://moonclerk.dev/pay/8h7frjfytj">Easy Time</a></div><script type="text/javascript">var mc8h7frjfytj;(function(d,t) {var s=d.createElement(t),opts={"checkoutToken":"8h7frjfytj","width":"100%","cid":"234"};s.src='http://moonclerk.dev/assets/embed.js';s.onload=s.onreadystatechange = function() {var rs=this.readyState;if(rs) if(rs!='complete') if(rs!='loaded') return;try {mc8h7frjfytj=new MoonclerkEmbed(opts);mc8h7frjfytj.display();} catch(e){}};var scr=d.getElementsByTagName(t)[0];scr.parentNode.insertBefore(s,scr);})(document,'script');</script>
```


## Retrieving the Custom ID and Checkout Info

### Passing Parameters via the Redirect URL (Simpler)

This is the simplest way to retrieve successful checkout information. When creating/editing your payment form in the MoonClerk dashboard, choose "Redirect to another web page" in the Confirmation section. Add your redirect URL. For this example we will use `http://example.com/success.html`.

We allow for 3 populated parameters to be appended to your redirect URL:

* `custom_id`
* `payment_id` OR `customer_id`

If a custom ID is passed into the checkout, it will be available for use in the redirect URL. Depending on the checkout type either the customer_id (for recurring) or the payment_id (for one time) will be available. Let's say you have a $10/month recurring checkout and you are sending the `cid` into the checkout, you could create a redirect URL as follows:

```
http://example.com/success.html?custom_id={{custom_id}}&customer_id={{customer_id}}
```

After a successful checkout, those curly brackets will be replaced with the actual values. You can set the parameter names to whatever you like. With a cid of 123, it my look like:

```
http://example.com/success.html?custom_id=123&customer_id=23452
```

You can then use the [MoonClerk API](https://github.com/moonclerk/developer/blob/master/api/README.md) to look up the customer (plan) using the `customer_id`. [See note on plan vs. customer nomenclature](https://github.com/moonclerk/developer/blob/master/api/v1/customers.md).


### Using Stripe Webhooks (More Complex)

If you are displaying a message at checkout you will not have a redirect url to use on your site. Instead, we recommend subscribing to Stripe's webhooks for your account. We add metadata to MoonClerk checkouts so you should have access to the form ID and either the MoonClerk payment ID or the customer ID.

When you pass a `cid` parameter to the checkout, we add it to the customer (when recurring) or the payment (when one time) as metadata. Once you receive the Stripe webhook, you can parse out the `custom_id` and will also have much of the other data provided by Stripe.

Checkout Stripe's [webhook walkthrough](https://stripe.com/docs/webhooks) and their [full API docs](https://stripe.com/docs/api).

If you find that you need additional MoonClerk data that is not included in the Stripe payload, you can use the various IDs in metadata to access MoonClerk's API and retrieve any other data (such as custom information).

Though this is a much more involved integration, it gives you the ultimate flexibility as well.

