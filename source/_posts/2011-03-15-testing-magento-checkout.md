---
layout: post
title: "Testing Magento Checkout"
author: Alistair Stead
date: 2011-03-15 18:40
comments: true
categories:
- PHP
- Magento
- MageTest
---

While developing new functionality in Magento one of the key elements that you must ensure continues to work as expected is the checkout. After all an commerce website that can not trade is rather worthless.

Using <a title="Mage-Test" href="http://github.com/alistairstead/Mage-Test">Mage-Test</a> you can easily create integration tests for the checkout process. This will allow you to ensure that your store still functions correctly after you have made any modifications.

## Automated Integration Test ##

The example test method below is run within a Mage-Test controller test case.

``` php
<?php
/**
 * @author Alistair Stead
 * @test
 */
public function magentoCheckoutIntegrationTest()
{
    // Add the product to the basket
    $this->request->setMethod('POST')
        ->setPost(
            array(
                'product' => '16',
                'related_product' => '',
                'qty' => '1'
            )
        );
    // Set an initial cookie so that the Magento cookie validation passes
    $this->request->setCookie('mage-test', true);
    $this->dispatch('/checkout/cart/add/');
    $this->assertResponseCode('302', '/checkout/cart/add/ has not redirected to the cart');
    $this->assertRedirectRegex("/^.*checkout.*$/", 'We are not directed to the checkout as expected');
    $this->reset();

    // Set the checkout as guest
    $this->request->setMethod('POST')
        ->setPost(
            array(
                'method' => 'guest'
            )
        );
    $this->dispatch('/checkout/onepage/saveMethod/');
    $this->assertResponseCode('200', '/checkout/onepage/saveMethod/ has not responded as expected');
    $this->reset();

    // Save billing address
    $this->request->setMethod('POST')
        ->setPost(
            array(
                'billing' => array(
                    'address_id' => '',
                    'firstname' => 'Authorize',
                    'lastname' => 'Capture',
                    'company' => '',
                    'email' => 'user@mage-test.co.uk',
                    'street' => array('Flat 14b', 'Baker Street'),
                    'city' => 'London',
                    'region_id' => '',
                    'postcode' => 'W1B OW3',
                    'country_id' => 'GB',
                    'telephone' => '+44 1234 123456',
                    'fax' => '+44 1234 123456',
                    'customer_password' => '',
                    'confirm_password' => '',
                    'save_in_address_book' => '1',
                    'use_for_shipping' => '1'
                )
            )
        );
    $this->dispatch('/checkout/onepage/saveBilling/');
    $this->assertResponseCode('200', '/checkout/onepage/saveBilling/ has not responded as expected');
    $this->reset();

    // Set shipping method
    $this->request->setMethod('POST')
        ->setPost(
            array(
                'shipping_method' => 'flatrate_flatrate'
            )
        );
    $this->dispatch('/checkout/onepage/saveShippingMethod/');
    $this->assertResponseCode('200', '/checkout/onepage/saveShippingMethod/ has not responded as expected');
    $this->reset();

    // Save payment
    $this->request->setMethod('POST')
        ->setPost(
            array(
                'payment' => array(
                    'method' => 'paypal_direct',
                    'cc_type' => 'SM',
                    'cc_number' => 'XXXXXXXXXXXXXXX',
                    'cc_exp_month' => '1',
                    'cc_exp_year' => (date('Y') + 6),
                    'cc_cid' => '444',
                    'cc_ss_issue' => '1',
                    'cc_ss_start_month' => '1',
                    'cc_ss_start_year' => (date('Y') - 2)
                )
            )
        );
    $this->dispatch('/checkout/onepage/savePayment/');
    $this->assertResponseCode('200', '/checkout/onepage/savePayment/ has not responded as expected');
    $this->reset();

    // Save order
    $this->request->setMethod('POST')
        ->setPost(
            array(
                'payment' => array(
                    'method' => 'paypal_direct',
                    'cc_type' => 'SM',
                    'cc_number' => 'XXXXXXXXXXXXXXX',
                    'cc_exp_month' => '1',
                    'cc_exp_year' => (date('Y') + 6),
                    'cc_cid' => '444',
                    'cc_ss_issue' => '1',
                    'cc_ss_start_month' => '1',
                    'cc_ss_start_year' => (date('Y') - 2)
                )
            )
        );
    $this->dispatch('/checkout/onepage/saveOrder/');
    $this->assertResponseCode('200', '/checkout/onepage/saveOrder/ has not responded as expected');
    // Decode the JSON response so that we can evaluate the success
    $data = json_decode($this->response->getBody());
    $this->assertTrue(
        $data->success,
        "Unexpected status expected result :: {$data->error_messages}"
    );
}
?>
```

You can use a <a title="PHPUnit" href="https://github.com/sebastianbergmann/phpunit/">PHPUnit</a> data provider to supply a set of data fixtures to be used for credit card numbers, address information and other values. Using a data provider you can test for not only the successful checkout but also the error handling within the checkout process.