# ShipIt

Librería que permite la integración con el API de ShipIt (https://developers.shipit.cl/docs) para 
enviar solicitudes de despacho, consultar su estados y otras acciones.

## Obtener las Credenciales de Acceso

Para usar el API de ShipIt debes tener una cuenta y acceder al menú "API" para
copiar tu email y token de acceso.

https://clientes.shipit.cl/settings/api

## Instalación

Para instalar la librería ejecuta el siguiente comando en la consola:

```bash
composer require kattatzu/ship-it
```

**[Instalación y Uso en Laravel](#instalación-y-uso-en-laravel)**

## Uso de forma Standalone

Si tu sistema no trabaja con Laravel puedes usarlo de forma directa:

```php
use Kattatzu/ShipIt/ShipIt;

$shipIt = new ShipIt('EMAIL', 'TOKEN', 'development');
// o
$shipIt = new ShipIt;
$shipIt->email('EMAIL');
$shipIt->token('TOKEN');
$shipIt->environment(ShipIt::ENV_PRODUCTION);

var_export($shipIt->getCommunes());
```
```php
array:425 [▼
  0 => Commune {#208 ▼
    -data: array:10 [▼
      "id" => 1
      "region_id" => 1
      "name" => "ARICA"
      "code" => "15101"
      "is_starken" => null
      "is_chilexpress" => null
      "is_generic" => true
      "is_reachable" => true
      "couriers_availables" => {#211}
      "is_available" => false
    ]
  },
  ...
]
```

## Acciones Disponibles

### Obtener las Regiones y Comunas

Puedes listar las regiones y comunas que tiene registradas ShipIt para sincronizar 
tu sistema.

```php
$shipIt->getRegions();
$shipIt->getCommunes()
```
#### Ejemplo

```php
$regions = $shipIt->getRegions();
echo $regions[0]->name;
// "Arica y Parinacota"
```

### Obtener una Cotización

Puedes enviar los datos de tu despacho y obtener una cotización con las opciones
de cariers que dispone ShipIt.

Para esto es necesario que crees una instancia **QuotationRequest** para ser enviada al método **getQuotation**.

#### Ejemplo

```php
$request = new QuotationRequest([
    'commune_id' => 317,    // id de la Comuna en ShipIt
    'height' => 10,         // altura en centimetros
    'length' => 10,         // largo en centimetros
    'width' => 10,          // ancho en centimetros
    'weight' => 1          // peso en kilogramos
]);
$quotationItems = $shipIt->getQuotation($request)->getItems();

foreach($quotationItems as $item){
    echo $item->courier . "<br>";
}
```

Si prefieres trabajar los items como array lo puedes hacer usando el método **toArray()**

```php
$request = new QuotationRequest(...);

$quotationItems = $shipIt->getQuotation($request)->toArray();

foreach($quotationItems as $item){
    echo $item['total'] . "<br>";
}
```


### Obtener la Cotización más Económica

Puedes enviar los datos de tu despacho y obtener la cotización más económica.


#### Ejemplo

```php
$request = new QuotationRequest([
    'commune_id' => 317,    // id de la Comuna en ShipIt
    'height' => 10,         // altura en centimetros
    'length' => 10,         // largo en centimetros
    'width' => 10,          // ancho en centimetros
    'weight' => 1          // peso en kilogramos
]);

$quotationItem = $shipIt->getEconomicQuotation($request);
echo $quotationItem->total;

// o como array

$quotationItem = $shipIt->getEconomicQuotation($request)->toArray();
echo $quotationItem['total'];

```

### Obtener la Cotización más Conveniente

Puedes obtener la cotización más conveniente tanto en tiempo de respuesta (SLA) como en precio.


#### Ejemplo

```php
$request = new QuotationRequest([
    'commune_id' => 317,    // id de la Comuna en ShipIt
    'height' => 10,         // altura en centimetros
    'length' => 10,         // largo en centimetros
    'width' => 10,          // ancho en centimetros
    'weight' => 1          // peso en kilogramos
]);

$quotationItem = $shipIt->getBestQuotation($request);
echo $quotationItem->total;

// o como array

$quotationItem = $shipIt->getBestQuotation($request)->toArray();
echo $quotationItem['total'];
```

### Enviar una solicitud de Despacho

Para enviar una solicitud de despacho debes crear una instancia **ShippingRequest** para ser enviada al método **requestShipping**:

```php
$request = new ShippingRequest([
    'reference' => 'S000001',
    'full_name' => 'José Eduardo Rios',
    'email' => 'cliente@gmail.com',
    'items_count' => 1,
    'cellphone' => '912341234',
    'is_payable' => false,
    'packing' => ShippingRequest::PACKING_NONE,
    'shipping_type' => ShippingRequest::DELIVERY_NORMAL,
    'destiny' => ShippingRequest::DESTINATION_HOME,
    'courier_for_client' => ShippingRequest::COURIER_CHILEXPRESS,
    'approx_size' => ShippingRequest::SIZE_SMALL,
    'address_commune_id' => 317,
    'address_street' => 'San Carlos',
    'address_number' => 123,
    'address_complement' => null,
]);

$response =  $shipIt->requestShipping($request);
echo $response->id; // id de la solicitud en ShipIt
```

Puedes trabajarlo como array usando el método **toArray()**:

```php
$request = new ShippingRequest(...);

$response =  $shipIt->requestShipping($request)->toArray();
echo $response['id'];
```

### Listar Solicitudes de Despacho

Puedes consultar el historial de solicitudes realizadas por día usando el método **getAllShippings**:

```php
$history = $shipIt->getAllShippings('2017-04-06');

// ó

$history = $shipIt->getAllShippings(Carbon::yesterday());

// ó

$history = $shipIt->getAllShippings(); // Por defecto será la fecha actual

foreach($history->getShippings() as $shipping){
    echo $shipping->id . "<br>";
}
```

Puedes trabajar el resultado como array usando el método **toArray()**:

```php
$history = $shipIt->getAllShippings();

foreach($history->toArray() as $shipping){
    echo $shipping['id'] . "<br>";
}
```


### Obtener el detalle de un Despacho

Puedes consultar los datos de un despacho historico enviando el id entregado por ShipIt 
usando el método **getShipping**:

```php
$shipping = $shipIt->getShipping(136107);
echo $shipping->reference;

// o como array

$shipping = $shipIt->getShipping(136107)->toArray();
echo $shipping['reference'];
```

### Utilidades
 
#### Obtener la url de seguimiento

Puede generar la url de seguimiento de un despacho fácilmente:

```php
$url = $shipIt->getTrackingUrl('chilexpress', 72626262);

// o

$url = ShipIt::getShipping(136097)->getTrackingUrl()
```

#### Tamaño aproximado del envio

Puedes obtener el tamaño aproximado en el formato que utiliza ShipIt de un paquete.

```php
$size = $shipIt->getPackageSize($width = 14, $height = 23, $length = 45);

Resultado: Grande (50x50x50cm)
```

## Instalación y Uso en Laravel

Después de hacer la instalación con Composer debes registrar el ServiceProvider y el alias en tu archivo config/app.php:

```php
'providers' => [
    ...
    Kattatzu\ShipIt\Providers\ShipItServiceProvider::class,
],
'aliases' => [
    ...
    'ShipIt' => Kattatzu\ShipIt\Facades\ShipItFacade::class,
]
```

Publica el archivo de configuración ejecutando en Artisan:

```bash
php artisan vendor:publish --provider="Kattatzu\ShipIt\Providers\ShipItServiceProvider"
```

Ya puedes ingresar las credenciales en el archivo config/shipit.php o en en archivo .env con las keys:

```bash
SHIPIT_ENV=development o production
SHIPIT_EMAIL=xxxxxxxx
SHIPIT_TOKEN=xxxxxxxx
SHIPIT_CALLBACK_URL=callback/shipit
```

### Facades

Ya puedes usar el Facade para acceder de forma rápida a las funciones:

```php
$shipping = ShipIt::getShipping(136107);
$regions = ShipIt::getRegions();
$communes = ShipIt::getCommunes()
$url = ShipIt::getTrackingUrl('chilexpress', 72626262);
$size = ShipIt::getPackageSize(14, 23, 45);
```

### Helpers

También puedes utilizar los helpers:

```php
$regions = shipit_regions();
$communes = shipit_communes();
$shippings = shipit_shippings('2017-06-01');
$shipping = shipit_shipping(12322);
$quotationItems = shipit_quotation($request);
$quotationItem = shipit_best_quotation($request);
$quotationItem = shipit_economic_quotation($request);
$response = shipit_send_shipping($request);
$url = shipit_tracking_url('starken', 23312332);
$size = shipit_package_size(14, 23, 45);
```
### Callback

ShipIt nos envia dos callback, una para notificarnos que una solicitud ha sido registrada y otra que indica que 
se le asigno un código de seguimiento o cambio en sus estados.

La librería permite escuchar dos eventos para recibir estos callbacks y realizar las operaciones que correspondan.

Los eventos son:

- **Kattatzu\ShipIt\Events\ShipItCallbackPostEvent**: nos indica que una solicitud fue registrada
- **Kattatzu\ShipIt\Events\ShipItCallbackPutEvent**: nos indica que una solicitud fue actualizada

#### Instalación

Lo primero que debemos hacer es crear los EventListeners que escucharán a estos eventos y registrarlos en el 
**EventServiceProvider** (app/Providers/EventServiceProvider.php).

```php
protected $listen = [
    'Kattatzu\ShipIt\Events\ShipItCallbackPostEvent' => [
        'App\Listeners\ShipItCallbackPostListener',
    ],
    'Kattatzu\ShipIt\Events\ShipItCallbackPutEvent' => [
        'App\Listeners\ShipItCallbackPutListener',
    ],
];
```

Una vez definidos los EventListeners podrás apuntar el Webhook de ShipIt (http://clientes.shipit.cl/settings/api) a la
url:

**https://tudominio.cl/callback/shipit**

Puedes personalizar este endpoint en el archivo de configuración **shipit.php** o en tu archivo .env con la key
**SHIPIT_CALLBACK_URL**.

No dudes en enviarme tus feedbacks o pull-request para mejorar esta librería.

## Licencia

MIT License

Copyright (c) 2017 José Eduardo Ríos

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.










