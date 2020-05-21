---
title: Self-Hosted-Callback Payment Page Setup
keywords: Self Hosted Callback Payment Page Setup Javascript Add Open Banking Payment Page
summary: "Adding Open Banking (in Self-Hosted-Callback mode) lets you fully control your PSU's payment journey and allows you to bypass the Nuapay TPP."
sidebar: ob_sidebar
permalink: ob_selfcallbacksetupoverview.html
folder: prodOpenBanking
toc: false
---

## Overview

{% include warning.html content="SELF_HOSTED_CALLBACK mode is currently only available in the Nuapay Test environment." %}


{% include note.html content="This section is for **Partner** users who want to use the **SELF_HOSTED_CALLBACK** mode - see [Implementation Options](ob_pispimplementation.html) for more on this." %}

## Prerequisites

In this mode, once a <a href="#" data-toggle="tooltip" data-original-title="{{site.data.glossary.psu}}">PSU</a> has been redirected to a selected <a href="#" data-toggle="tooltip" data-original-title="{{site.data.glossary.aspsp}}">ASPSP</a> and has completed his/her interactions with that bank, the payer is redirected to the `merchantPostAuthUrl`, as provided in the [Create Payment](ob_createpayment.html) request. 

In order for the payer to be successfully redirected, an <a href="#" data-toggle="tooltip" data-original-title="{{site.data.glossary.aspsp}}">ASPSP</a> must do two things:

1. Determine what the callback URL is for the specific <a href="#" data-toggle="tooltip" data-original-title="{{site.data.glossary.psu}}">PSU</a>.
1. Confirm that this URL has been properly configured and registered (with that <a href="#" data-toggle="tooltip" data-original-title="{{site.data.glossary.aspsp}}">ASPSP</a>).

{% include important.html content="The Nuapay API Support team manage the TPP-to-ASPSP registration process (including callback URL registration) on behalf of our partners and merchants. Please contact your Account Manager if you need to register a new callback URL." %}

## Sequence Steps

{% include note.html content="If you want to use this mode you must have first registered your OAuth 2.0 Callback Handler with the platform. Your Account Manager can provide you with more information on this." %}

To use Self-Hosted-Callback mode see the flow described in the image below (click to enlarge) and refer to the steps 1-6 below for more detail:

{% include tip.html content="Click Extend from the top menu to enlarge or click the image itself to open it in a new browser tab/window" %}
{% include image.html file="ob_selfhostedcallback_flow-partner.png" url="images/ob_selfhostedcallback_flow-partner.png" target = "_new" alt="Self-Hosted-Callback Partner Flow" caption="SELF_HOSTED_CALLBACK Partner Flow" %}

1. Using your partner-level API key retrieve the [list of your merchants](ob_partnerintegration.html#api-details---get-organisations) and [retrieve a token](ob_partnerintegration.html#api-details---post-tokens) representing the required merchant.
1. You will design your own Bank Selection screen; use [Retrieve Banks](ob_getbank.html) to populate your interface. 
1. Once the payer has selected a bank, call the `/payments` endpoint using the OAUth token, representing the required merchant, (see [Create Payment](ob_createpayment.html)).
Set the `integrationType` to `SELF_HOSTED_CALLBACK`, specify the `bankId` provided by the payer and set the `merchantPostAuthUrl`.
1. Redirect the <a href="#" data-toggle="tooltip" data-original-title="{{site.data.glossary.psu}}">PSU</a> to the <a href="#" data-toggle="tooltip" data-original-title="{{site.data.glossary.aspsp}}">ASPSP</a> to authorise the payment - note that in this mode the `callbackUrl` = the `merchantPostAuthUrl`.
   * Unlike in the SELF_HOSTED mode, the OAuth callback from the bank is not passed to the <a href="#" data-toggle="tooltip" data-original-title="{{site.data.glossary.nupay_tpp}}">Nuapay TPP</a> directly, instead it is directed to the merchantPostAuthUrl. 
   * The partner retrieves An OAuth token (providing the Partner API key and setting scope = `openbanking_callback`).
   * The payment callback is passed to the <a href="#" data-toggle="tooltip" data-original-title="{{site.data.glossary.nupay_tpp}}">Nuapay TPP</a>.
1. The <a href="#" data-toggle="tooltip" data-original-title="{{site.data.glossary.nupay_tpp}}">Nuapay TPP</a> posts the payment ID to the partner/merchant URL (`merchantPostAuthUrl`).
1. Use [Retrieve Payment](ob_retrievepayment.html) to determine the final payment status, if required (an optional step). 

## Processing the Callback

Note that:

1. Since response parameters are returned in the Redirection URI fragment value, the Client needs to have the User Agent parse the fragment encoded values and pass them on to the Client's processing logic for consumption. (Basically parse the details in JavaScript to post back to your server)
1. Accept the data on your systems.
1. Post the information to the `/payments/callback` [endpoint](ob_paymentcallback.html).

Please see a sample piece of JavaScript code below, which parses the data after the anchor:

	#check for the anchor
	var data = window.location.href.split(#);
	if(data.length < 2) { //just in case not using anchors
	    data = window.location.href.split(?);
	}

	var params = data[1]; 

	var xhr = new XMLHttpRequest();
	xhr.open("POST", YOUR_URL, true);
	xhr.setRequestHeader('Content-Type', 'application/json');
	xhr.send(JSON.stringify({
    		value: params
	}));


This assumes you can accept a `POST` of json to YOUR_URL.

In your logic on the server side, you can parse all the params and submit to the [endpoint](ob_paymentcallback.html).

{% include note.html content="You should ensure that the JavaScript you define works in all browsers for both desktop and mobile and we recommend that you fully test your implementation on all the browsers that you intend to support." %}


{% include links.html %}





