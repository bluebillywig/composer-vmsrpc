# VMS RPC

https://support.bluebillywig.com/vms-rpc-php

The vms rpc client makes it possible to connect to the vms more easily. This is achieved by taking away some of the complexities like connecting to the vms. This document provides a technical summary of the vms rpc client for PHP.

## Usage

### Getting Started

Create a new instance of BlueBillywig\RPC. This can be done by providing the username and password or the shared secret. The shared secret will be supplied by `support@bluebillywig.com` when asking to use the VMSRPC.

#### Creating instance with username and password

```php
use BlueBillywig\VMSRPC;

$host = 'https://YourPublication.bbvms.com';
$user = 'UserName';
$password = 'Password';

$vms = new RPC($host, $user, $password);
```

#### Creating instance with shared secret

```php
use BlueBillywig\VMSRPC;

$host = 'https://YourPublication.bbvms.com';
$sharedSecret = 123-1234567890abcdefghijklmnopqrstuv;

$vms = new RPC($host, null, null, $sharedSecret);
```

#### Quick start

The following example demonstrates how to search for a couple of mediaclips and immediately print the embed code for each mediaclip.

```php
$arProps = array(
    'query' => 'type:mediaclip AND status:published',
    'limit' => '10',
    'sort' =>'createddate desc'
);

// Search action
$search_result = json_decode($vms->json('search', null, $arProps));

if($search_result->count > 0) {
    foreach($search_result->items as $oMc) {            
        echo '<script type="text/javascript" src="https://YourPublication.bbvms.com/p/default/c/'.$oMc->id.'.js"></script>';
    }
}
```

### Function list

Here you can find the available functions within the RPC, which make it possible to execute API requests. More information concerning API requests can be found here: https://support.bluebillywig.com/vms-api-guide

#### doAction

Execute API request/action and return the result as an array.

`array doAction(string $entity, string $action, array $arProps)`

$entity: The relevant api entity that you want to query to obtain the desired result. For example mediaclip, mediacliplist, publication or playout.

$action: The action you want to execute. The actions differ per entity, see the API documentation for details. For example for the mediaclip entity possible actions are get, put or getUsageStats.

$arProps: This array will contain required or optional properties needed for the specific request. For example for the action put of the entity mediaclip you'll need to provide at least the property xml, see the example below.

Here an example to update a mediaclip:

```php
use BlueBillywig\VMSRPC;

$host = 'http://YourPublication.bbvms.com';
$user = 'UserName';
$password = 'Password'; 

$vms = new RPC($host, $user, $password);

//Set the status to draft
$arProps = array(
    'xml' => '<media-clip id="1234567" status="draft"></media-clip>'
);
$r = $vms->doAction('mediaclip', 'put', $arProps);
```

And an example to add a new mediaclip including a video-file:

```php
use BlueBillywig\VMSRPC;

$host = 'http://YourPublication.bbvms.com';
$user = 'UserName';
$password = 'Password'; 

$vms = new RPC($host, $user, $password);

//Add a new mediaclip, add a file, set the title and set the status.
$file = getcwd() . "/someVideo.mp4";
$arProps = array(
    'xml' => '<media-clip title="New mediaclip"  status="draft"></media-clip>', 
     'Filedata' => '@'.$file
);

$r = $vms->doAction('mediaclip', 'put', $arProps);
```

#### JSON

Execute API request/action and return the result in JSON format. At this moment it's **only** possible to get results in JSON format for **'get'** and **'search'** actions on **mediaclips**, **timelines**, **widgets** and **zones**.

`string json(string $entity, int $objectId = null, array $arProps = null)`

$entity: The relevant api entity that you want to query to obtain the desired result. For example mediaclip, mediacliplist, publication or playout.

$objectId: (optional) The id of a certain entity, for example the mediaclipid. If $objectId is set the action will become 'get'.

$arProps: (optional) This array contains required or optional properties needed for the specific request.

Below an example to search for a couple of mediaclips.

```php
use BlueBillywig\VMSRPC;

$host = 'http://YourPublication.bbvms.com';
$user = 'UserName';
$password = 'Password'; 

$vms = new RPC($host, $user, $password);

$arProps = array(
    'query' => 'type:mediaclip','limit' => '10',
    'sort' =>'createddate desc'
);
$r = json_decode($vms->json('search', null, $arProps));
```

#### XML

Execute API request/action and return the result in XML format.

`string xml(string $entity, int $objectId = null, array $arProps = null)`

$entity: The relevant api entity that you want to query to obtain the desired result. For example mediaclip, mediacliplist, publication or playout.

$objectId: (optional) The id of a certain entity, for example the mediaclipid. If $objectId is set the action will become 'get'.

$arProps: (optional) This array contains required or optional properties needed for the specific request. If not set the default action will become 'get'.

Below an example to get global configurations of a publication.

```php
use BlueBillywig\VMSRPC;

$host = 'http://YourPublication.bbvms.com';
$user = 'UserName';
$password = 'Password'; 

$vms = new RPC($host, $user, $password);

$r = $vms->xml('publication');
```

#### URI

Execute an API request/action and send all properties for the specific request on the query string. This may be useful whether multiple properties with the same key need to be provided, which could be the case for Solr requests.

`string uri(string $apiEntityUrl, string $qs = null)`

$apiEntityUrl: The relevant api entity that you want to query to obtain the desired result. For example json/mediaclip or json/qstats.

$qs: (optional) The Query String to be send with the request'.

Below an example to get the mainconfig.xml, which contains configuration of your publication such as asset-paths.

```php
use BlueBillywig\VMSRPC;

$host = 'http://YourPublication.bbvms.com';
$user = 'UserName';
$password = 'Password'; 

$vms = new RPC($host, $user, $password);

$apiEntityUrl = "";
$qs = 'mainconfig.xml';

$r = $vms->uri($apiEntityUrl, $qs);
```

## Unit Testing

To run the PHP unit tests, copy `tests/config.example.yaml` to `tests/config.yaml` and update `tests/config.yml` with the settings that apply to your publication.

Run the tests with the following command.

```php
./vendor/bin/phpunit --bootstrap ./vendor/autoload.php tests/RPCTest.php
```