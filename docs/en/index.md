# MetisFW/PayPal

## Setup

1) Register extension
```
extensions:
  payPal: MetisFW\PayPal\DI\PayPalExtension
```

2) Set up extension parameters

```neon
payPal:
  clientId: AUqne4ywvozUaSQ1THTZYKFr88bhtA0SS_fXBoJTfeSTIasDBWuXLiLcFlfmSXRfL-kZ3Z5shvNrT6rP
  secret: EDGPDc3a65JBBY7-IKkNak7aGTVTvY-NhJgfhptegSML58fWjfp89U7UKNgGk9UI-UEZ-btfaE2sGST1
  currency: EUR
  # optional Payment Experience Profile ID
  # https://developer.paypal.com/docs/api/payment-experience/
  # experienceProfileId: XP-AAAA-BBBB-CCCC-DDDD

  # optional payment intent
  # default value is "sale"
  # https://developer.paypal.com/docs/classic/paypal-payments-standard/integration-guide/authcapture/
  # Valid Values: ["sale", "authorize", "order"]
  # intent: authorize
  sdkConfig:
    mode: sandbox
    log.Enabled: true
    log.FileName: '%tempDir%/PayPal.log'
    log.LogLevel: DEBUG
    validation.level: log
    cache.enabled: true
    # 'http.CURLOPT_CONNECTTIMEOUT' => 30
    # 'http.headers.PayPal-Partner-Attribution-Id' => '123123123'/
```

sdkConfig is config to [paypal/PayPal-PHP-SDK](https://github.com/paypal/PayPal-PHP-SDK)
see [sdk-config-sample](https://github.com/paypal/PayPal-PHP-SDK/blob/master/sample/sdk_config.ini)

##Usage

##### Sample usage of `PaymentControl`

###### In Presenter

```php
use \MetisFW\Paypal\Payment\SimplePaymentOperation;
use \MetisFW\PayPal\UI\PaymentControl;
use PayPal\Api\Payment;
use Nette\Application\UI\Presenter;

class MyPresenter extends Presenter {

  public function createComponentPayPalPaymentButton(SimplePaymentOperationFactory $factory) {
    $operation = $factory->create('Coffee', 5);
    $control = new PaymentControl($operation);
  
    //set different template if u want to use own
    $control->setTemplateFilePath(__DIR__ . './myPayPalButton.latte');
  
    //called before redirect to paypal after first api call, which create payment
    $control->onCheckout[] = function(PaymentControl $control, Payment $created) {
      //something
    };
  
    //called after successfully completed payment proccess
    $control->onSuccess[] = function(PaymentControl $control, Payment $paid) {
      //something
    };
  
    //called when user cancelled payment process
    $control->onCancel[] = function(PaymentControl $control) {
      //something
    };
  
    return $control;
  }
}
```

###### In latte

```latte
#just
{control payPalPaymentButton}

#or

#cannot use attributes directly in control
# see http://doc.nette.org/en/2.3/default-macros#toc-component-rendering
{var attributes = array('class' => 'paypal-payment-button')} 
{control payPalPaymentButton $attributes, 'Pay me now!'}
```

##### Sample usage of `SimplePaymentOperation`

```php
  public function createComponentPayPalpaymentButton(SimplePaymentOperationFactory $factory) {
    $operation = $factory->create('Coffee', 10);
    $control = new PaymentControl($operation);
    return $control;
  }
```

##### Sample usage of `PlainPaymentOperation`

```php
use PayPal\Api\Transaction;

  public function createComponentPayPalpaymentButton(PlainPaymentOperationFactory $factory) {
    $transactions = array();
    $transaction = new Transaction();
    //setup transaction - see paypal-php-sdk
    
  
    $operation = $factory->create($transactions);
    $control = new PaymentControl($operation);
    return $control;
  }
```

##### Sample usage of own descendant `\MetisFW\PayPal\Payment\BasePaymentOperation`

```php
<?php

namespace MetisApp\Components\Payment;

use MetisFW\PayPal\Payment\BasePaymentOperation;
use MetisFW\PayPal\PayPalContext;
use PayPal\Api\Amount;
use PayPal\Api\Details;
use PayPal\Api\Item;
use PayPal\Api\ItemList;
use PayPal\Api\Transaction;

class OrderPayPalOperation extends BasePaymentOperation {

  /** @var mixed */
  private $order;

  /**
   * @param PayPalContext $context
   * @param mixed $order some data - object/array/...
   */
  public function __construct(PayPalContext $context, $order) {
    parent::__construct($context);
    $this->order = $order;
  }

  /**
   * @return array array of PayPal\Api\Transaction
   */
  protected function getTransactions() {
    $transactions = array();
    $transaction = new Transaction();
    
    //setup transaction via data passed in constructor

    $transactions[] = $transaction;
    return $transactions;
  }

}

```

###### Events in Operation
```php
  public function createComponentPayPalPaymentButton(FactorType $factory) {
    $operation = $factory->create();
    $operation->onReturn[] = function($operation, Payment $paid) {
      //something
    }
    $operation->onCancel[] = function($operation) {
      //something
    }
    
    ...
  }
```
