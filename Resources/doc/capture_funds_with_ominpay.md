# Capture funds with omnipay gateways.

Omnipay lib provide [support of 25+ gateways](https://github.com/adrianmacneil/omnipay#payment-gateways). 

## Step 1. Download Omnipay Bridge.

Add the following lines in your `composer.json` file:

```json
{
    "require": {
        "payum/omnipay-bridge": "dev-master"
    }
}
```

_**Note:** You may want to adapt this line to use a specific version._

Now, run composer.phar to download the bundle:

```bash
$ php composer.phar install
```

_**Note:** You can immediately start using it. The autoloading files have been generated by composer and already included to the app autoload file._

## Step 2: Basic configuration

### 1. Configure payment context

```yaml
#app/config/config.yml

payum:
    contexts:
        your_payment_name:
            ominpay:
                type: Stripe
                options:
                    apiKey: abc123
                    testMode: true
```

_**Attention**: You have to changed this name `your_payment_name` to something related to your domain, for example `post_a_job_with_stripe`._
 
### 2. Configure filesystem storage

Create a PaymentDetails class with added `id` property:

```php
<?php
//src/Acme/DemoBundle/Model

namespace AcmeDemoBundle\Model;

class OmnipayPaymentDetails extends \ArrayObject
{
    protected $id;
    
    public function getId()
    {
        return $this->id;
    }
}
```

and configure storage to use this model:

```yaml
#app/config/config.yml

payum:
    contexts:
        your_payment_name:
            storages:
                Acme\DemoBundle\Model\OmnipayPaymentDetails
                    filesystem:
                        storage_dir: %kernel.root_dir%/Resources/payments
                        id_property: id
                        payment_extension: true
```

## Step 3. Capture payment:

_**Note** : We assume you [configured capture controller](basic_setup.md#step-3-configure-capture-controller-optional)_

_**Note** : We assume you choose a storage._

```php
<?php
//src/Acme/DemoBundle/Controller
namespace AcmeDemoBundle\Controller;

use Symfony\Component\HttpFoundation\Request;

class PaymentController extends Controller 
{
    public function prepareStripePaymentAction(Request $request)
    {        
        $paymentName = 'your_payment_name';
                
        $storage = $this->get('payum')->getStorageForClass(
            'Acme\DemoBundle\Model\OmnipayPaymentDetails',
            $paymentName
        );
    
        /** @var OmnipayPaymentDetails */
        $paymentDetails = $storage->createModel();
        $paymentDetails['amount'] = 10;
        $paymentDetails['card'] = array(
            'number' => '5555556778250000',
            'cvv' => 123,
            'expiryMonth' => 6,
            'expiryYear' => 16,
            'firstName' => 'foo',
            'lastName' => 'bar',
        );
        
        $storage->updateModel($paymentDetails);

        $captureToken = $this->get('payum.security.token_factory')->createCaptureToken(
            $paymentName,
            $paymentDetails,
            'acme_payment_done' // the route to redirect after capture;
        );

        return $this->forward('PayumBundle:Capture:do', array(
            'payum_token' => $captureToken,
        ));
    }
}
```

## Next Step

* [Simple purchase done action](simple_purchase_done_action.md).
* [Configuration reference](configuration_reference.md).
* [Sandbox](sandbox.md).
* [Debugging](debugging.md).
* [Back to index](index.md).
