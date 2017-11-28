PHP Wallet Sample
==================================
This is a simple php app that uses podland wallet APIs. It only uses curl function in php to send HTTP requests.

How to use this project
-----------------------
Clone the project and edit the config.php file and fill it with real values. and serve it using a php enabled webserver like apache2 or nginx.

```php
    $config = [
        ""service"=>"[platform-address]",
    "sso"=>"[sso-address]/oauth2/",
    "sso_service"=>"[sso-address]",
    "client_id"=>"[client_id]",
    "client_secret"=>"[client_secret]",
    "home"=>"[your-app-home]",
    "api_token"=>"[api_token]",
    "private_call_address"=>"[private-calls-address]"
    ];
```

Wallet Usage Flow
-----------------
In order to use Basic Platform service, your users must sign up and log in using SSO. After allowed by user to see business their info, then an access_token is given to business that it should use it in every header of request that it sends to the platform.
 The access_token will be expired after a while so it must be refreshed using refresh_token. For using business APIs api_token must be used in header.
 Also in every response from platform a one time token(ott) will be sent which is valid only for one use and it always changes. In important financial services the last received ott must be used.
 
 
 ### Following Business
 In order to use wallet operation, like any other platform api, the customer(user) must be follower of your business. so first after returning from sso registration or login you must add customer to your follower list. this is done with following code in this sample project:
  
  ```php
  function followBusiness($access_token)
  {
      global $config;
      $businessId = getBusinessId();
      $curl = curl_init();
      curl_setopt_array($curl, [
          CURLOPT_URL => $config['service'] . "nzh/follow/?businessId={$businessId}&follow=true",
          CURLOPT_RETURNTRANSFER => true,
          CURLOPT_CUSTOMREQUEST => 'GET',
          CURLOPT_HTTPHEADER => [
              "_token_: {$access_token}",
              "_token_issuer_: 1"
          ],
      ]);
      $response = curl_exec($curl);
      return json_decode($response);
  }
  ```
  The above function can be found in return.php, The code for handling errors and verifying ssl is removed from above.
  
  ### Retrieving OTT
  for getting the OTT the following code is used:
  ```php
function getOtt(){
    $curl = curl_init();
    global $config;
    curl_setopt_array($curl, [
        CURLOPT_URL => $config['service'] . "nzh/ott/",
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_CUSTOMREQUEST => 'GET',
        CURLOPT_HTTPHEADER => [
            '_token_: ' . $config['api_token'],
            '_token_issuer_: 1',
            'X-Requested-With: XMLHttpRequest'
        ],
    ]);
    $responseOtt = curl_exec($curl);
    $err = curl_error($curl);
    return json_decode($responseOtt);
}
```

### Issuing Invoice
In order to sell first you have to issue invoice, after paying the invoice by the customer the credit will be added to your wallet
following code is used to issue invoice:
```php
 curl_setopt_array($curl, [
        CURLOPT_URL => $config['service'] . "nzh/biz/issueInvoice/?bizId=" . getBusinessId() . "&userId=" . getUserPlatform()->userId . "&description=buy&pay=false&postalCode=000000000&phoneNumber={$_SESSION['phone_number']}&city=tehran&redirectUrl={$config['home']}&productId[]=0&price[]=" . ($_POST['price']) . "&quantity[]=1&productDescription[]=<Description>&state=<State>&address=<Address>&deadline=<JalaliDate>&guildCode=<GuildCode>",
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_CUSTOMREQUEST => 'GET',
        CURLOPT_HTTPHEADER => [
            '_token_: ' . $config['api_token'],
            '_ott_: ' . getOtt(),
            '_token_issuer_: 1',
            'X-Requested-With: XMLHttpRequest'
        ],
    ]);
    $responseInvoice = curl_exec($curl);
```
above code can be found in order.php

### Paying Invoice
In order to make customer pay the issue, They must be redirected to platform pbc that is done with following code:

```php
    header("Location: http://<private-call-address>/v1/pbc/payinvoice/?invoiceId={$invoiceId}&redirectUri={$config['home']}pay_return_redirect.php&callUri={$config['home']}pay_return_call.php");
```
Above code can be found in order.php and the invoice id is gotten from issueInvoice response in previous call
