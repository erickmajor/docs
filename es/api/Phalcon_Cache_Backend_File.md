# Clase **Phalcon\\Cache\\Backend\\File**

*extends* abstract class [Phalcon\Cache\Backend](/[[language]]/[[version]]/api/Phalcon_Cache_Backend)

*implements* [Phalcon\Cache\BackendInterface](/[[language]]/[[version]]/api/Phalcon_Cache_BackendInterface)

<a href="https://github.com/phalcon/cphalcon/blob/master/phalcon/cache/backend/file.zep" class="btn btn-default btn-sm">Codigo fuente en GitHub</a>

Permite almacenar en caché los fragmentos de salida usando un backend de archivo

```php
<?php

use Phalcon\Cache\Backend\File;
use Phalcon\Cache\Frontend\Output as FrontOutput;

// Almacena el archivo por dos días
$frontendOptions = [
    "lifetime" => 172800,
];

// Crea un cache de salida
$frontCache = FrontOutput($frontOptions);

// Establece el directorio de cacheo
$backendOptions = [
    "cacheDir" => "../app/cache/",
];

// Crea el archivo en el backend
$cache = new File($frontCache, $backendOptions);

$content = $cache->start("my-cache");

if ($content === null) {
    echo "<h1>", time(), "</h1>";

    $cache->save();
} else {
    echo $content;
}

```

## Métodos

public **__construct** ([Phalcon\Cache\FrontendInterface](/[[language]]/[[version]]/api/Phalcon_Cache_FrontendInterface) $frontend, *array* $options)

Constructor de Phalcon\\Cache\\Backend\\File

public **get** (*mixed* $keyName, [*mixed* $lifetime])

Devuelve un contenido cacheado

public **save** ([*int* | *string* $keyName], [*string* $content], [*int* $lifetime], [*boolean* $stopBuffer])

Almacena el contenido en caché en el backend del archivo y detiene el Frontend

public **delete** (*int* | *string* $keyName)

Borra un valor de la memoria caché por su clave

public **queryKeys** ([*mixed* $prefix])

Consulta las claves existentes almacenadas en caché.

```php
<?php

$cache->save("users-ids", [1, 2, 3]);
$cache->save("projects-ids", [4, 5, 6]);

var_dump($cache->queryKeys("users")); // ["users-ids"]

```

public **exists** ([*string* | *int* $keyName], [*int* $lifetime])

Verifica si existe el caché y que no está expirado

public **increment** ([*string* | *int* $keyName], [*mixed* $value])

Incremento de una clave dada, por número $value

public **decrement** ([*string* | *int* $keyName], [*mixed* $value])

Decremento de una clave dada, por número $value

public **flush** ()

Invalida inmediatamente todos los elementos existentes.

public **getKey** (*mixed* $key)

Devolver un identificador de sistema de archivos seguros para una clave determinada

public **useSafeKey** (*mixed* $useSafeKey)

Establece si se usa safekey o no

public **getFrontend** () inherited from [Phalcon\Cache\Backend](/[[language]]/[[version]]/api/Phalcon_Cache_Backend)

...

public **setFrontend** (*mixed* $frontend) inherited from [Phalcon\Cache\Backend](/[[language]]/[[version]]/api/Phalcon_Cache_Backend)

...

public **getOptions** () inherited from [Phalcon\Cache\Backend](/[[language]]/[[version]]/api/Phalcon_Cache_Backend)

...

public **setOptions** (*mixed* $options) inherited from [Phalcon\Cache\Backend](/[[language]]/[[version]]/api/Phalcon_Cache_Backend)

...

public **getLastKey** () inherited from [Phalcon\Cache\Backend](/[[language]]/[[version]]/api/Phalcon_Cache_Backend)

...

public **setLastKey** (*mixed* $lastKey) inherited from [Phalcon\Cache\Backend](/[[language]]/[[version]]/api/Phalcon_Cache_Backend)

...

public *mixed* **start** (*int* | *string* $keyName, [*int* $lifetime]) inherited from [Phalcon\Cache\Backend](/[[language]]/[[version]]/api/Phalcon_Cache_Backend)

Starts a cache. The keyname allows to identify the created fragment

public **stop** ([*mixed* $stopBuffer]) inherited from [Phalcon\Cache\Backend](/[[language]]/[[version]]/api/Phalcon_Cache_Backend)

Detiene el frontend sin grabar ningún contenido en caché

public **isFresh** () inherited from [Phalcon\Cache\Backend](/[[language]]/[[version]]/api/Phalcon_Cache_Backend)

Comprueba si el último caché está actualizado o almacenado en cache

public **isStarted** () inherited from [Phalcon\Cache\Backend](/[[language]]/[[version]]/api/Phalcon_Cache_Backend)

Verifica si el caché tiene un buffering inicial o no

public *int* **getLifetime** () inherited from [Phalcon\Cache\Backend](/[[language]]/[[version]]/api/Phalcon_Cache_Backend)

Obtiene el último lifetime establecido