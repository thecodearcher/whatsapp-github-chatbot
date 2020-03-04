## Building a WhatsApp chatbot with Twilio Whatsapp API

Often at times when chatting with a friend or family onWhatsApp, you might have found yourself in a situation to fact check a statement or quickly look up some information and this will often lead you to switch to your web browser or even pull out your PC and this can sometimes be quite inconveniencing. However, with services like the [Twilio WhatsApp API](https://www.twilio.com/whatsapp), you can build custom solutions to power-up your chatting experience. 

In this tutorial, we will build a simple WhatsApp Chatbot that allows you to get information about a developer's GitHub profile using just their username.

After successfully following this tutorial, you would have learned how to respond to WhatsApp Messages sent to your Twilio WhatsApp Number and also how to send out free-form messages using the Twilio WhatsApp API. 

## Prerequisite

To  follow through with this tutorial, you will need the following:

- Basic knowledge of Laravel
- [Laravel](https://laravel.com/docs/master) Installed on your local machine
- [Composer](https://getcomposer.org/) globally installed
- [Twilio Account](https://www.twilio.com/referral/B2YAW1)
- [Whatsapp Enabled](https://www.twilio.com/console/sms/whatsapp/learn) Twilio Number

## Getting Started

This tutorial will make use of Laravel. Start off by first creating a new Laravel project using the [Laravel installer](https://laravel.com/docs/6.x#installing-laravel). Open up a terminal and run the following command to generate a new Laravel application:

    $ laravel new whatsapp-chatbot

Next, you will need the [Twilio SDK](https://www.twilio.com/docs/libraries/php) and [Guzzle HTTP client](https://github.com/guzzle/guzzle) for interacting with the Twilio Whatsapp API and making HTTP requests respectively. Both packages can be installed using [Composer](https://getcomposer.org/), so open up a terminal that points to the `whatsapp-chatbot` directory and run the following commands to have them installed:

    $ composer require twilio/sdk
    $ composer require guzzlehttp/guzzle

Next, head over to your [Twilio dashboard](https://www.twilio.com/console) and copy out your *ACCOUNT SID* and *AUTH TOKEN* which will be used for authenticating your request with the Twilio SDK:

![https://res.cloudinary.com/brianiyoha/image/upload/v1582534932/Articles%20sample/s_14AED1E729777868A76C728380D4E7434CFBFCFA0C71AD83ED009C3DCFE403E8_1574552733012_Group_8.png](https://res.cloudinary.com/brianiyoha/image/upload/v1582534932/Articles%20sample/s_14AED1E729777868A76C728380D4E7434CFBFCFA0C71AD83ED009C3DCFE403E8_1574552733012_Group_8.png)

Proceed to update your environmental variables with this credentials. Open up your `.env` file and add the following variables:

    TWILIO_SID="ACCOUNT_SID"
    TWILIO_AUTH_TOKEN="YOUR_AUTH_TOKEN"

## Setting up WhatsApp Sandbox

As you might have already figured, to allow your chatbot to respond to messages it receives, you must have a way of sending messages via Whatsapp. Fortunately, Twilio provides a very robust [Whatsapp API](https://www.twilio.com/whatsapp) that allows you to send and receive Whatsapp messages right from your application.

Before you can start sending and receiving messages using the Twilio Whatsapp API in production, you must first get a [Whatsapp Approved Twilio Number](https://www.twilio.com/whatsapp/request-access) which will act as your Whatsapp number for sending and receiving messages. Since getting a Twilio number approved can take days, Twilio also provides a safe [Sandbox](https://www.twilio.com/console/sms/whatsapp/learn) which can be used for development and testing purposes.

To get started using the Twilio Whatsapp Sandbox, head over to the [Whatsapp section](https://www.twilio.com/console/sms/whatsapp/learn) on your Twilio dashboard and send a message to the sandbox number provided; usually, *+14155238886* with the provided *code* which is in the format `join-{unique word}`: 

![https://res.cloudinary.com/brianiyoha/image/upload/v1582537775/Articles%20sample/Group_15.png](https://res.cloudinary.com/brianiyoha/image/upload/v1582537775/Articles%20sample/Group_15.png)

After successfully sending your *code* to the sandbox number you should receive a response like this:

![https://res.cloudinary.com/brianiyoha/image/upload/v1581952691/Articles%20sample/-4SgUEt2_1.png](https://res.cloudinary.com/brianiyoha/image/upload/v1581952691/Articles%20sample/-4SgUEt2_1.png)

Next, update your `.env` file to include your whatsapp number; in this case the sandbox number:

    TWILIO_WHATSAPP_NUMBER="+14155238886"

## Building the Chatbot

At this point, you should have your Twilio sandbox setup and also installed the needed packages to build the chatbot. Now create a [Controller](https://laravel.com/docs/6.x/controllers) which will house the logic of the chatbot, open up a terminal in the project directory and run the following command to generate a new Controller class:

     $ php artisan make:controller ChatBotController

Open up the just generated `app/Http/Controllers/ChatBotController.php` file and make the following changes:

    <?php
    
    namespace App\Http\Controllers;
    
    use GuzzleHttp\Exception\RequestException;
    use Illuminate\Http\Request;
    use Twilio\Rest\Client;
    
    class ChatBotController extends Controller
    {
        public function listenToReplies(Request $request)
        {
            $from = $request->input('From');
            $body = $request->input('Body');
    
            $client = new \GuzzleHttp\Client();
            try {
                $response = $client->request('GET', "https://api.github.com/users/$body");
                $githubResponse = json_decode($response->getBody());
                if ($response->getStatusCode() == 200) {
                    $message = "*Name:* $githubResponse->name\n";
                    $message .= "*Bio:* $githubResponse->bio\n";
                    $message .= "*Lives in:* $githubResponse->location\n";
                    $message .= "*Number of Repos:* $githubResponse->public_repos\n";
                    $message .= "*Followers:* $githubResponse->followers devs\n";
                    $message .= "*Following:* $githubResponse->following devs\n";
                    $message .= "*URL:* $githubResponse->html_url\n";
                    $this->sendWhatsappMessage($message, $from);
                } else {
                    $this->sendWhatsappMessage($githubResponse->message, $from);
                }
            } catch (RequestException $th) {
                $response = json_decode($th->getResponse()->getBody());
                $this->sendWhatsappMessage($response->message, $from);
            }
            return;
        }
    
        /**
         * Sends a WhatsApp message  to user using
         * @param string $message Body of sms
         * @param string $recipient Number of recipient
         */
        public function sendWhatsappMessage(string $message, string $recipient)
        {
            $twilio_whatsapp_number = getenv('TWILIO_WHATSAPP_NUMBER');
            $account_sid = getenv("TWILIO_SID");
            $auth_token = getenv("TWILIO_AUTH_TOKEN");
    
            $client = new Client($account_sid, $auth_token);
            return $client->messages->create($recipient, array('from' => "whatsapp:$twilio_whatsapp_number", 'body' => $message));
        }
    }

Two new methods have been added to the class; `listenToReplies()` and `sendWhatsappMessage()`, let's break down each function. The `listenToReplies()` method is where messages sent to your Whatsapp number will be processed and response will be sent depending on the `body` of the message received. Whenever a new message comes in, Twilio will call the endpoint linked to this method while passing information about the message as a body of the [request](https://laravel.com/docs/6.x/requests). From the *request* body, you can get details about the message a user sent to your Whatsapp number and among which are the `From` and `Body` which holds the sends phone number and message respectively:

    $from = $request->input('From');
    $body = $request->input('Body');

Next, an HTTP request is made to the [Github Developer API](https://developer.github.com/v3/) using the Guzzle HTTP library (installed in the earlier part of this tutorial) to get the users' details using the username gotten from the Whatsapp message body.  Depending on the response gotten from the HTTP request, a Whatsapp message will be sent back to the user with either a summary of the user's profile or an error message. The helper `sendWhatsappMessage()` function is what is used for sending out Whatsapp messages. It takes in two arguments; `$message` and `$receipent`. 

The  `sendWhatsappMessage()` method makes use of [Twilio SDK](https://www.twilio.com/docs/libraries/php) for sending out Whatsapp messages:

    public function sendWhatsappMessage(string $message, string $recipient)
        {
            $twilio_whatsapp_number = getenv('TWILIO_WHATSAPP_NUMBER');
            $account_sid = getenv("TWILIO_SID");
            $auth_token = getenv("TWILIO_AUTH_TOKEN");
    
            $client = new Client($account_sid, $auth_token);
            return $client->messages->create($recipient, array('from' => "whatsapp:$twilio_whatsapp_number", 'body' => $message));
        }

Firstly, you have to retrieve your Twilio credentials stored in your `.env` file earlier before proceeding to create a new instance of the Twilio `Client` using your *account sid* and *auth token.* Next, you have to pass in the `recipient` and an array of options to the `messages->create()` method from the *client* instance to actually send out a request via the Twilio API. The `messages->create()` method takes in two arguments, the recipient which is the Whatsapp enabled phone number you want to send a message to - in this case, the *sender* of the initial text - and an associative array with the keys: `from` and `body`. The `from` property should be your Twilio WhatsApp phone number or sandbox number (for testing only) with the *text* `whatsapp:` prepended to it while the `body` property holds the `$message` to be sent to the `recipient`.

**NOTE:** *The recipient number should also have the text `whatsapp:` prepended to it but in this case the number gotten from the request body already has it as part of the string returned for the `From` property hence why we don't manually input it.* 

## Setup Webhook

To enable you to receive messages sent to your WhatsApp number, you must first add a webhook URL to your Twilio dashboard. Before you can do this, you must have set up your application route which will serve as your webhook URL. Open up `routes/api.php` and make the following changes to add a new route (`/chat-bot`) to your application:

    <?php
    
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Route;
    
    /*
    |--------------------------------------------------------------------------
    | API Routes
    |--------------------------------------------------------------------------
    |
    | Here is where you can register API routes for your application. These
    | routes are loaded by the RouteServiceProvider within a group which
    | is assigned the "api" middleware group. Enjoy building your API!
    |
     */
    
    Route::middleware('auth:api')->get('/user', function (Request $request) {
        return $request->user();
    });
    
    Route::post('/chat-bot', 'ChatBotController@listenToReplies');

**NOTE:** 

- *Routes defined in `routes/api.php` makes use of the `api` [middleware group](https://laravel.com/docs/6.x/middleware#middleware-groups) .*
- *Routes defined in `routes/api.php` will be prefixed with `/api`.*

### Exposing application to the internet

To allow access to your Laravel application through a webhook, your application has to be accessible via the internet. Since you are still building your application, you can make use of [ngrok](https://ngrok.com/) to make the application accessible from the internet.

If you don’t have [ngrok](https://ngrok.com/) set up on your computer, you can see how to do so by following the instructions on their [official download page](https://ngrok.com/download). Next, open up your terminal and run the following commands to start your Laravel application:

    $ php artisan serve

This will get your Laravel application running on your local machine on a specific port which will be printed out to the terminal after successfully running the command. Take note of the port as it will be used shortly. Now, while still running the `artisan serve` command, open up another instance of your terminal and run this command to make your application publicly accessible:

    $ ngrok http 8000

**NOTE:** *Replace `8000` with the which your application is running on.*

After successful execution of the above command, you should see a screen like this:

![https://res.cloudinary.com/brianiyoha/image/upload/v1583284629/Articles%20sample/ngrok-screenshot.png](https://res.cloudinary.com/brianiyoha/image/upload/v1583284629/Articles%20sample/ngrok-screenshot.png)

Take note of the `forwarding url` as we will be making use of it next.

### Updating sandbox webhook

Having exposed your application to the internet, you can now make use of your application route as the webhook URL for your WhatsApp sandbox. Head to the [WhatsApp sandbox settings](https://www.twilio.com/console/sms/whatsapp/sandbox) section in your Twilio dashboard and update the input field labeled *"WHEN A MESSAGE COMES IN"* with the complete URL to your chatbot path: **

![https://res.cloudinary.com/brianiyoha/image/upload/v1583278704/Articles%20sample/Group_16.png](https://res.cloudinary.com/brianiyoha/image/upload/v1583278704/Articles%20sample/Group_16.png)

**NOTE:** *Your webhook URL should be in the format `https://{ngrok-forward-url}/api/chat-bot`*

## Testing

Awesome! You have successfully built your chatbot application. Now proceed to test it by sending a WhatsApp message to your sandbox number with a Github user' *username* and if everything works as expected you should get a response with a summary of the person's profile and a link to view their profile directly:

![https://res.cloudinary.com/brianiyoha/image/upload/v1583281559/Articles%20sample/Screenshot_20200304-012415_1.png](https://res.cloudinary.com/brianiyoha/image/upload/v1583281559/Articles%20sample/Screenshot_20200304-012415_1.png)

## Conclusion

Now that you have successfully finished this tutorial, you should have a simple WhatsApp chatbot for looking up a user's profile. With this, you also learned how to send out free-form messages using the Twilio WhatsApp API and also how to respond to messages sent to your Twilio WhatsApp number. If you will like to take a look at the complete source code for this tutorial, you can find it on [Github](mailto:git@github.com).

I’d love to answer any question(s) you might have concerning this tutorial. You can reach me via:

- Email: [brian.iyoha@gmail.com](mailto:brian.iyoha@gmail.com)
- Twitter: [thecodearcher](https://twitter.com/thecodearcher)
- GitHub: [thecodearcher](https://github.com/thecodearcher)
