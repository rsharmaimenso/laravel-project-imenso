# Implementing the Open-Closed Principle in Laravel

#### Open Closed principle is the 2nd principle of the SOLID principle. In SOLID O is stand for Open Closed principle. Open Closed principle means A class/method should be open for extension but closed for modification, which means you can change behavior without modifying source code. The main idea of this principle is to keep existing code from breaking when you implement new features, which means extending functionality by adding new code instead of changing existing code.

Let’s create a new Laravel Project

To create a new project, run the following command wherever your projects are stored:

```
$ composer create-project --prefer-dist laravel/laravel open-closed-principle

```

Change directories into `open-closed-principle` and install the `Twilio SDK for PHP` by running the command:

```
$ composer require twilio/sdk

```

Open the `.env` file in your preferred IDE and add the following credentials:

```
TWILIO_SID="INSERT YOUR TWILIO SID HERE"
TWILIO_TOKEN="INSERT YOUR TWILIO TOKEN HERE"
TWILIO_NUMBER="INSERT YOUR TWILIO NUMBER IN E.164 FORMAT"

```

Let’s start by creating a directory in the `app` folder called `Sms`. In it, we will add an `SmsInterface.php` file:

```
<?php

namespace App\Sms;

interface SmsInterface

{
 public function sendSms($to, $from, $message);
}

```

In the same directory let's create our first SMS provider file called `TwilioSms.php` and add the following code to it:

```js

<?php

namespace App\Sms;
use Twilio\Rest\Client;

class TwilioSms implements SmsInterface
{
   protected $client;

   public function __construct($sid, $authToken)
   {
     $this->client = new Client($sid, $authToken);
   }

   public function sendSms($to, $from , $message)
   {
       $message = $this->client->messages->create($to, [
           'from' => $from,
           'body' => $message,
       ]);
       return $message->sid;
   }
}

```

Configure and Register the Twilio service
In `config/services.php`, let’s configure the credentials by appending the following piece of code in the array:

`'twilio' => [ 'account_sid' => env('TWILIO_SID'), 'auth_token' => env('TWILIO_TOKEN'), ],`


In `app/Providers/AppServiceProvider.php`, <u>register the Twilio service by copying this code in the register method</u>:

```js

<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use App\Sms\SmsInterface;
use App\Sms\TwilioSms;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        $this->app->bind(TwilioSms::class, function () {

           $config = config('services.twilio');

           return new TwilioSms($config['account_sid'], $config['auth_token']);

       });

       $this->app->bind(SmsInterface::class, TwilioSms::class);

    }
}

```

## Create an SMS Controller

In the directory `app/Http/Controllers` create a new file called `SmsController.php` and paste the following code in it.

```js

<?php

namespace App\Http\Controllers;
use Illuminate\Http\Request;
use App\Sms\SmsInterface;

class SmsController extends Controller
{
 /**
 * Create a new instance
 *
 * @return void
 */
 public function __construct(SmsInterface $smsProvider)
 {
     $this->smsProvider = $smsProvider;
 }

 public function sendSms(Request $request)
 {
   $messageId = $this->smsProvider->sendSms(
       $request->input('to'),
       $request->input('from'),
       $request->input('text_message')
   );

   return response()->json(['message_id' => $messageId]);

 }
}

```

In order to access this controller via HTTP, let’s add a route in the `routes/web.php` directory:

```
 Route::post('send-sms', 'SmsController@sendSms');

```

In the `app/Middleware/VerifyCsrfToken.php` directory, exclude this URI <u>http://localhost:8000/send-sms</u>. Your file should look like this:

```js

<?php

namespace App\Http\Middleware;

use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as Middleware;

class VerifyCsrfToken extends Middleware
{
   /**
    * Indicates whether the XSRF-TOKEN cookie should be set on the response.
    *
    * @var bool
    */
   protected $addHttpCookie = true;

   /**
    * The URIs that should be excluded from CSRF verification.
    *
    * @var array
    */
   protected $except = [
       '/send-sms'
   ];
}

```

Use Postman to make an API call to the same URL `http://localhost:8000/send-sms. `


By adding and registering  new SMS provider in  register() function we can change SMS provider. 