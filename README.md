# real_API
Real.de -> Online API




real.de Onlineshop API

    Introduction
    Getting Started
    Overview
    Rest API
    Managing Product Data
    Managing Inventory
    Managing Orders
    Managing Order Invoices
    Managing Tickets
    Managing Claims
    Managing Returns
    Managing Reports
    Push Notifications
    Product Data CSV Files
    Inventory CSV Files
    Code Examples
        Getting an Item (GET)
        Adding a Unit (POST)
        Deleting a Unit (DELETE)
        Opening a Claim (POST)
        Closing a Claim (PATCH)
    Endpoints Documentation
    PHP SDK

Code Examples

This page includes a number of examples of full PHP scripts which interact with the Hitmeister API.

Note: you will need the API keys from your API settings page to set the $clientKey and $secretKey in the below examples.
Getting an Item (GET)

This code listing is a standalone PHP script that will query the API for a product with a known ID and output all of its properties. This is an example of a GET request.



			<?php

			function signRequest($method, $uri, $body, $timestamp, $secretKey)
			{
				$string = implode("\n", [
					$method,
					$uri,
					$body,
					$timestamp,
				]);

				return hash_hmac('sha256', $string, $secretKey);
			}

			$baseUrl = 'https://www.real.de/api/v1';

			// Credentials for the API
			$clientKey = '35f5dba43b471cd2ad28fd68e4825e2b';
			$secretKey = 'b3575f222f7b768c25160b879699118b331c6883dd6010864b7ead130be77cd5';

			// We are going to get information about the product with ID 20574181
			// (at the URL https://www.real.de/product/20574181/)
			$uri = $baseUrl . '/items/20574181/';

			// Also include the item's category and units in the response
			$params = [
				'embedded' => 'category,units',
			];

			//The query parameters must be URL-encoded, but the "=" and "&" signs should not be encoded
			$uri .= '?' . http_build_query($params);

			// Current Unix timestamp in seconds
			$timestamp = time();

			// Define all the mandatory headers
			$headers = [
				'Accept: application/json',
				'Hm-Client: ' . $clientKey,
				'Hm-Timestamp: ' . $timestamp,
				'Hm-Signature: ' . signRequest('GET', $uri, '', $timestamp, $secretKey),
			];

			$ch = curl_init();
			curl_setopt($ch, CURLOPT_URL, $uri);
			curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
			curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);

			$item = json_decode(curl_exec($ch), true);
			var_export($item);
			curl_close($ch);

			When executed, the above code will print the following result:

			{
				"id_item": 20574181,
				"title": "PHP and MySQL for Dummies",
				"eans": {
					0: "9780470527580"
				},
				"id_category": 53021,
				"main_picture": "https://media.real.de/images/items/original/567ff112bb416618b057802817dfe1d2.jpg",
				"manufacturer": "For Dummies",
				"url": "https://www.real.de/product/20574181/",
				"category": {
					"id_category": 53021,
					"name": "naturwissenschaften-medizin-informatik-und-technik-buecher",
					"id_parent_category": 6261,
					"title_singular": "Naturwissenschaften, Medizin, Informatik & Technik",
					"title_plural": "Naturwissenschaften, Medizin, Informatik & Technik",
					"level": 3,
					"url": "https://www.real.de/naturwissenschaften-medizin-informatik-und-technik-buecher/",
					"shipping_category": "A",
					"variable_fee": "12.50",
					"fixed_fee": 0,
					"vat": "7.00"
				},
				"units": {
					0: {
						"id_unit": 99792311008,
						"id_item": 20574181,
						"condition": "new",
						"location": "DE",
						"warehouse": "Theresa GmbH - Lager: 1024",
						"amount": 1,
						"price": 2639,
						"delivery_time_min": 3,
						"delivery_time_max": 8,
						"shipping_group": null,
						"note": "Versand durch DHL",
						"seller": {
							"pseudonym": "Tolle Buecher"
						},
						"shipping_rate": 0
					},
					1: {
						"id_unit": 105194581008,
						"id_item": 20574181,
						"condition": "new",
						"location": "DE",
						"warehouse": "Kevin GmbH - Lager: 1262"
						"amount": 1,
						"price": 2890,
						"delivery_time_min": 2,
						"delivery_time_max": 6,
						"shipping_group" "paket",
						"note": "Bei Fragen helfen wir gerne.",
						"seller": {
							"pseudonym": "Buecherland"
						},
						"shipping_rate": 430
					}
			}

Adding a Unit (POST)

This code example is a PHP script that will add a unit (i.e. offer) to the item we got in the previous example. This is an example of a POST request. This is the same script as the one on the Getting Started page. Please notice that you have to specify a listing price greater than zero.

<?php

function signRequest($method, $uri, $body, $timestamp, $secretKey)
{
	$string = implode("\n", [
		$method,
		$uri,
		$body,
		$timestamp,
	]);
	
	return hash_hmac('sha256', $string, $secretKey);
}

$baseUrl = 'https://www.real.de/api/v1';

// Credentials for the API
$clientKey = '35f5dba43b471cd2ad28fd68e4825e2b';
$secretKey = 'b3575f222f7b768c25160b879699118b331c6883dd6010864b7ead130be77cd5';

// Define the POST data
$data = [
	"ean"           => "9780470527580",
	"condition"     => "new",
	"listing_price" => 4550,
	"minimum_price" => 3990,
	"amount"        => 9,
    "delivery_time_min" => 2,
	"delivery_time_max" => 5,
	"location"      => "DE"
];

$jsonData = json_encode($data);

// We're adding a new unit
$uri = $baseUrl . '/units/';

// Current Unix timestamp in seconds
$timestamp = time();

// Define all the mandatory headers
$headers = [
    'Accept: application/json',
    'Hm-Client: ' . $clientKey,
    'Hm-Timestamp: ' . $timestamp,
    'Hm-Signature: ' . signRequest('POST', $uri, $jsonData, $timestamp, $secretKey),
];

$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, $uri);
curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_POST, 1);
curl_setopt($ch, CURLOPT_POSTFIELDS, $jsonData);
curl_setopt($ch, CURLOPT_HEADER, 1);
curl_setopt($ch, CURLINFO_HEADER_OUT, 1);

$response = curl_exec($ch);
list($header, $body) = explode("\r\n\r\n", $response, 2);
print("$header\n\n");

$responseCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
print("HTTP Response Code: $responseCode\n");
curl_close($ch);

If you run this script, it will output the response headers and the success response code 204:

HTTP Response Code: 204

One of the response headers is the location of the API endpoint for the newly-created unit, which we will use in the next code example:

Location: /units/151177892008/

Deleting a Unit (DELETE)

This code example is a PHP script that will delete the unit that was added in the last example. This is an example of a DELETE request.

<?php
function signRequest($method, $uri, $body, $timestamp, $secretKey)
{
	$string = implode("\n", [
		$method,
		$uri,
		$body,
		$timestamp,
	]);
	
	return hash_hmac('sha256', $string, $secretKey);
}

$baseUrl = 'https://www.real.de/api/v1';

// Credentials for the API
$clientKey = '35f5dba43b471cd2ad28fd68e4825e2b';
$secretKey = 'b3575f222f7b768c25160b879699118b331c6883dd6010864b7ead130be77cd5';

// We're deleting the unit with id_unit=99792311008
// We got this URL from the headers returned when we created the unit
$uri = $baseUrl . '/units/151177892008/';

// Current Unix timestamp in seconds
$timestamp = time();

// Define all the mandatory headers
$headers = [
    'Accept: application/json',
    'Hm-Client: ' . $clientKey,
    'Hm-Timestamp: ' . $timestamp,
    'Hm-Signature: ' . signRequest('DELETE', $uri, '', $timestamp, $secretKey),
];

$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, $uri);
curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
curl_setopt($ch, CURLOPT_CUSTOMREQUEST, 'DELETE');
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HEADER, 1);
curl_setopt($ch, CURLINFO_HEADER_OUT, 1);

$response = curl_exec($ch);
list($header, $body) = explode("\r\n\r\n", $response, 2);
print("$header\n\n");

$responseCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
print("HTTP Response Code: $responseCode\n");
curl_close($ch);

If you run this script, it will output the response headers and the success response code 204:

HTTP Response Code: 204

Opening a Claim (POST)

This code example is a PHP script that will open a new claim on an order unit. This is an example of a POST request.

<?php

function signRequest($method, $uri, $body, $timestamp, $secretKey)
{
	$string = implode("\n", [
		$method,
		$uri,
		$body,
		$timestamp,
	]);
	
	return hash_hmac('sha256', $string, $secretKey);
}

$baseUrl = 'https://www.real.de/api/v1';

// Credentials for the API
$clientKey = '35f5dba43b471cd2ad28fd68e4825e2b';
$secretKey = 'b3575f222f7b768c25160b879699118b331c6883dd6010864b7ead130be77cd5';

// Define the POST data
$data = [
	"id_order_unit" => 314567837137231,
	"text" => "The customer never paid.",
];

$jsonData = json_encode($data);

// We're adding a new claim
$uri = $baseUrl . '/claims/';

// Current Unix timestamp in seconds
$timestamp = time();

// Define all the mandatory headers
$headers = [
    'Accept: application/json',
    'Hm-Client: ' . $clientKey,
    'Hm-Timestamp: ' . $timestamp,
    'Hm-Signature: ' . signRequest('POST', $uri, $jsonData, $timestamp, $secretKey),
];

$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, $uri);
curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_POST, 1);
curl_setopt($ch, CURLOPT_POSTFIELDS, $jsonData);
curl_setopt($ch, CURLOPT_HEADER, 1);
curl_setopt($ch, CURLINFO_HEADER_OUT, 1);

$response = curl_exec($ch);
list($header, $body) = explode("\r\n\r\n", $response, 2);
print("$header\n\n");

$responseCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
print("HTTP Response Code: $responseCode\n");
curl_close($ch);

If you run this example, a new claim will be opened on order unit 314567837137231 and the text will be sent to the customer. The script will print out all of the headers returned by the request, including this line:

Location: /claims/2498572/

This is the URL of the new claim's GET endpoint and 2498572 is the new claim's id_claim.
Closing a Claim (PATCH)

This code example is a PHP script that will close the claim opened in the previous example. This is an example of a PATCH request.

<?php

function signRequest($method, $uri, $body, $timestamp, $secretKey)
{
	$string = implode("\n", [
		$method,
		$uri,
		$body,
		$timestamp,
	]);
	
	return hash_hmac('sha256', $string, $secretKey);
}

$baseUrl = 'https://www.real.de/api/v1';

// Credentials for the API
$clientKey = '35f5dba43b471cd2ad28fd68e4825e2b';
$secretKey = 'b3575f222f7b768c25160b879699118b331c6883dd6010864b7ead130be77cd5';

// We're closing the claim with id_claim=2498572
$uri = $baseUrl . '/claims/2498572/close/';

// Current Unix timestamp in seconds
$timestamp = time();

// Define all the mandatory headers
$headers = [
    'Accept: application/json',
    'Hm-Client: ' . $clientKey,
    'Hm-Timestamp: ' . $timestamp,
    'Hm-Signature: ' . signRequest('PATCH', $uri, null, $timestamp, $secretKey),
];

$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, $uri);
curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
curl_setopt($ch, CURLOPT_CUSTOMREQUEST, 'PATCH');
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HEADER, 1);
curl_setopt($ch, CURLINFO_HEADER_OUT, 1);

$response = curl_exec($ch);
list($header, $body) = explode("\r\n\r\n", $response, 2);
print("$header\n\n");

$responseCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
print("HTTP Response Code: $responseCode\n");
curl_close($ch);

If you run this script, it will output the response headers and the success response code 204:

HTTP Response Code: 204

