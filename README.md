**SPRINT 5 :** 
---------------

**Uninstall the module, and install them again, what happens ?**

The module is disabled and removed from the system.
Any configuration and data related to the module will be deleted.
The database entries related to this module will be removed.

And when we Reinstall the Module : 

The module is added back into Drupal’s system.
A new entry is created in the database.


**Make change to core_version_requirement to a lower version, what happens ?**

In the **.info.yml,** We modify the file : 
core_version_requirement: ^9 || ^10 || ^11 to core_version_requirement: ^8

Clear the cache. 

Result: Drupal will mark the module as incompatible and may prevent it from being installed or enabled.


**Add the module workflows as a dependency to your dependencies, what happens ? :** 

In the .info.yml we add Workflows as dependency : 

dependencies:
  - workflows:workflows

The workflows:workflows part tells Drupal that the Workflows module is required for the module to work.
When we try to install it from the UI , it gonna tell us that the workflows is required for the module.



**Add a composer.json file to your modules :** 

By adding composer.json, we ensure that your module's dependencies (if any) are properly managed and can be automatically installed using Composer.
It also allows us to integrate our module into the Drupal ecosystem, ensuring proper versioning and dependency handling.


In the /modules/custom/hello_world : we create a new composer.json file.

To install the dependencies with composer : we run composer install in the same path of the composer.json and enbale it after with drush en hello_word . 


**Delete your modules or move them to another directory, what happens ? what message appear in drupal ? what the log says drush ws ?**

The error : (Circular Reference)

The website encountered an unexpected error. Try again later.

[error]  ServiceCircularReferenceException: Circular reference detected for service 'module_handler', path: 'module_handler -> http_kernel -> http_middleware.ajax_page_state -> http_middleware.negotiation -> http_middleware.reverse_proxy -> shield.middleware -> ban.middleware -> http_middleware.page_cache -> page_cache_request_policy -> persistent_login.page_cache_request_policy.pending_persistent_login -> persistent_login.cookie_helper -> config.factory -> config.typed'.

**Why ?**

1. Missing Dependencies: Some modules may be core to the functionality of other modules. If a module is  deleted and was a dependency for others, those other modules will fail to load properly, which might involve services that depend on each other in a circular way.
2. Service Destruction: The deletion of a module can remove its service from the container. If other services (like module_handler, http_kernel, etc.) rely on that missing service, it creates unresolved dependencies. If these dependencies form a circular chain, it leads to the circular reference error.
3. Improper Cleanup: Sometimes, when you delete a module, it doesn't fully clean up all its associated configurations, services, or links. This can leave services or configurations that are still referenced by other parts of the system, leading to circular dependencies.

**DAY 2 :**
--------------


**1. How do I control or sort the menus (weight)?**
   Drupal allows you to control the order of menu items using the weight property in .links.menu.yml. Lower values appear first.

my_module.weather_menu:
title: 'Weather'
route_name: my_module.weather_page
menu_name: main
weight: 1  # Appears earlier

my_module.forecast_menu:
title: 'Forecast'
route_name: my_module.forecast_page
menu_name: main
weight: 5  # Appears later
Lower weight = Higher priority in the menu order.

**2. How do I set up child menus?**
   To create a submenu, define a parent in .links.menu.yml.

my_module.weather_menu:
title: 'Weather'
route_name: my_module.weather_page
menu_name: main
weight: 1

my_module.forecast_menu:
title: 'Forecast'
route_name: my_module.forecast_page
parent: my_module.weather_menu  # Makes it a child menu
menu_name: main
weight: 2
"Forecast" appears as a submenu under "Weather".

**3. How do I retrieve a query string in a Controller?**
   Use Symfony's Request object to get query strings.

    /weather?city=London&unit=metric

Controller:

use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\JsonResponse;

public function weatherPage(Request $request) {
$city = $request->query->get('city'); // Retrieves 'London'
$unit = $request->query->get('unit', 'imperial'); // Default: 'imperial' if missing

return new JsonResponse([
'city' => $city,
'unit' => $unit,
]);
}
$request->query->get('key') fetches query string values.

**4. What is Guzzle and Logger?**

   Guzzle (HTTP Client)
   A library for making HTTP requests (fetching external APIs).
   Used via \Drupal::httpClient().

$client = \Drupal::httpClient();
$response = $client->get('https://api.example.com/data');
$data = json_decode($response->getBody(), true);

Logger (Logging System)
Logs system messages (info, warning, error).

Accessed via \Drupal::logger() or Dependency Injection.
\Drupal::logger('my_module')->info('This is an info log.');

**5. Use Logger to log a message in a Controller. Where do these messages appear?**

use Psr\Log\LoggerInterface;
public function logExample(LoggerInterface $logger) {
$logger->info('Weather module loaded successfully.');
$logger->warning('Something might be wrong.');
$logger->error('An error occurred!');
}
Where to Find Logs?

Drupal Admin UI: admin/reports/dblog (Enable Database Logging module).
Drush: Run  drush ws (watchdog show 

**6. Where can I find the services defined by Drupal core?**
    File Location:
    core/core.services.yml

    For custom modules:
    modules/custom/my_module/my_module.services.yml

**7. How do you inject other needed services into your service?**

To inject services into your service in Drupal, you have three main methods:

Using Autowiring (Recommended) : 

Define the service in your_module.services.yml.
Use dependency injection in the constructor.

use Drupal\Core\Logger\LoggerChannelFactoryInterface;

class MyService {
protected $logger;

    public function __construct(LoggerChannelFactoryInterface $logger) {
        $this->logger = $logger;
    }
} 

Using create() Method (Manual Injection) : 

Useful when the class implements ContainerInjectionInterface.

use Symfony\Component\DependencyInjection\ContainerInterface;

class MyController {
protected $logger;

    public static function create(ContainerInterface $container) {
        return new static($container->get('logger.factory'));
    }
}
Using Trait (For Common Services)

use Drupal\Core\DependencyInjection\ContainerInjectionInterface;
use Drupal\Core\Logger\LoggerChannelTrait;

class MyService implements ContainerInjectionInterface {
use LoggerChannelTrait;
}

**8. How do you return a Twig template in a Controller?**

Using #theme and #variables.
public function weatherPage() {
return [
'#theme' => 'weather_template',
'#data' => ['city' => 'London'],
];
} 

Create a Twig template: templates/weather-template.html.twig : 

<p>Weather in {{ data.city }}</p>

**10. Given  $this->t('Hello, @name!', ['@name' => $username]), what is the search
    keyword to find it at  /admin/config/regional/translate?**

    The search keyword is  "Hello, " because @name is changeable and not fixed.

**11. How do you make a string translatable in Drupal JavaScript files?**

    alert(Drupal.t('Hello World!'));

**12. Use Drush PHP to get the service path.alias_manager.**

drush php
Then, inside Drush PHP shell:
$aliasManager = \Drupal::service('path_alias.manager');

**13. Get the alias of a node, given node/1.**

$aliasManager = \Drupal::service('path.alias_manager');
$alias = $aliasManager->getAliasByPath('/node/1');
echo $alias;

**14. Use Link and Url to get the full URL of one of your routes.**

use Drupal\Core\Link;
use Drupal\Core\Url;

$url = Url::fromRoute('my_module.weather_page');
$link = Link::fromTextAndUrl('Weather Page', $url)->toString();
Returns:
<a href="/weather">Weather Page</a>


**15. How do you send a JSON response in a Controller instead of a display?**

Using JsonResponse with a proper Content-Type.

use Symfony\Component\HttpFoundation\JsonResponse;

public function weatherApi() {
$data = [
'city' => 'London',
'temperature' => 20,
];
return new JsonResponse($data, 200, ['Content-Type' => 'application/json']);
}
Returns JSON with Content-Type: application/json.

