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
```yml
core_version_requirement: ^9 || ^10 || ^11 to core_version_requirement: ^8
```

Clear the cache. 

Result: Drupal will mark the module as incompatible and may prevent it from being installed or enabled.

**Add the module workflows as a dependency to your dependencies, what happens ? :** 

In the .info.yml we add Workflows as dependency : 
```yml
dependencies:
  - workflows:workflows
```

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

```php
[error]  ServiceCircularReferenceException:
  Circular reference detected for service 'module_handler', path: 'module_handler -> http_kernel -> http_middleware.ajax_page_state -> http_middleware.negotiation -> http_middleware.reverse_proxy -> shield.middleware -> ban.middleware -> http_middleware.page_cache -> page_cache_request_policy -> persistent_login.page_cache_request_policy.pending_persistent_login -> persistent_login.cookie_helper -> config.factory -> config.typed'.
```

**Why ?**

1. Missing Dependencies: Some modules may be core to the functionality of other modules. If a module is  deleted and was a dependency for others, those other modules will fail to load properly, which might involve services that depend on each other in a circular way.
2. Service Destruction: The deletion of a module can remove its service from the container. If other services (like module_handler, http_kernel, etc.) rely on that missing service, it creates unresolved dependencies. If these dependencies form a circular chain, it leads to the circular reference error.
3. Improper Cleanup: Sometimes, when you delete a module, it doesn't fully clean up all its associated configurations, services, or links. This can leave services or configurations that are still referenced by other parts of the system, leading to circular dependencies.

**DAY 2 :**
--------------


**1. How do I control or sort the menus (weight)?**
   Drupal allows you to control the order of menu items using the weight property in .links.menu.yml. Lower values appear first.

```yml
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
```

Lower weight = Higher priority in the menu order.

**2. How do I set up child menus?**
   To create a submenu, define a parent in .links.menu.yml.
```yml
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
```

"Forecast" appears as a submenu under "Weather".

**3. How do I retrieve a query string in a Controller?**
```php

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
```

**4. What is Guzzle and Logger?**

Guzzle (HTTP Client)
A library for making HTTP requests (fetching external APIs).
Used via \Drupal::httpClient().
```php

$client = \Drupal::httpClient();
$response = $client->get('https://api.example.com/data');
$data = json_decode($response->getBody(), true);

Logger (Logging System)
Logs system messages (info, warning, error).

Accessed via \Drupal::logger() or Dependency Injection.
\Drupal::logger('my_module')->info('This is an info log.');
```

**5. Use Logger to log a message in a Controller. Where do these messages appear?**
```php

use Psr\Log\LoggerInterface;
public function logExample(LoggerInterface $logger) {
$logger->info('Weather module loaded successfully.');
$logger->warning('Something might be wrong.');
$logger->error('An error occurred!');
}
```

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

```php
use Drupal\Core\Logger\LoggerChannelFactoryInterface;

class MyService {
protected $logger;

    public function __construct(LoggerChannelFactoryInterface $logger) {
        $this->logger = $logger;
    }
} 
```

Using create() Method (Manual Injection) : 
Useful when the class implements ContainerInjectionInterface.

```php
use Symfony\Component\DependencyInjection\ContainerInterface;

class MyController {
protected $logger;

    public static function create(ContainerInterface $container) {
        return new static($container->get('logger.factory'));
    }
}
```

Using Trait (For Common Services)

```php

use Drupal\Core\DependencyInjection\ContainerInjectionInterface;
use Drupal\Core\Logger\LoggerChannelTrait;

class MyService implements ContainerInjectionInterface {
use LoggerChannelTrait;
}
```
**8. How do you return a Twig template in a Controller?**

```php
Using #theme and #variables.
public function weatherPage() {
return [
'#theme' => 'weather_template',
'#data' => ['city' => 'London'],
];
} 
```

Create a Twig template: templates/weather-template.html.twig : 

```php
<p>Weather in {{ data.city }}</p>

**10. Given  $this->t('Hello, @name!', ['@name' => $username]), what is the search
    keyword to find it at  /admin/config/regional/translate?**

    The search keyword is  "Hello, " because @name is changeable and not fixed.
```

**11. How do you make a string translatable in Drupal JavaScript files?**

```php
    alert(Drupal.t('Hello World!'));
```

**12. Use Drush PHP to get the service path.alias_manager.**

drush php
Then, inside Drush PHP shell:
```php
$aliasManager = \Drupal::service('path_alias.manager');
```

**13. Get the alias of a node, given node/1.**

```php
$aliasManager = \Drupal::service('path.alias_manager');
$alias = $aliasManager->getAliasByPath('/node/1');
echo $alias;
```

**14. Use Link and Url to get the full URL of one of your routes.**
```php
use Drupal\Core\Link;
use Drupal\Core\Url;

$url = Url::fromRoute('my_module.weather_page');
$link = Link::fromTextAndUrl('Weather Page', $url)->toString();
Returns:
<a href="/weather">Weather Page</a>
```

**15. How do you send a JSON response in a Controller instead of a display?**

```php
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
```

DAY 4__QUESTIONS : Drupal Form Handling Guide
---------------------------------------------


**1. Where can you validate your form data?**

In Drupal, you can validate form data using the validateForm method inside a Form class or by implementing a hook.

Method 1: Using validateForm in a Custom Form Class

```php
use Drupal\Core\Form\FormBase;
use Drupal\Core\Form\FormStateInterface;

class MyCustomForm extends FormBase {
  public function getFormId() {
    return 'my_custom_form';
  }

  public function buildForm(array $form, FormStateInterface $form_state) {
    $form['email'] = [
      '#type' => 'textfield',
      '#title' => 'Email',
      '#required' => TRUE,
    ];

    $form['submit'] = [
      '#type' => 'submit',
      '#value' => 'Submit',
    ];

    return $form;
  }

  public function validateForm(array &$form, FormStateInterface $form_state) {
    $email = $form_state->getValue('email');
    if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
      $form_state->setErrorByName('email', $this->t('Invalid email address.'));
    }
  }
}
```

Method 2: Using hook_form_alter() to Add Validation : 
```php
function mymodule_form_alter(&$form, FormStateInterface $form_state, $form_id) {
  if ($form_id == 'some_existing_form') {
    $form['#validate'][] = 'mymodule_custom_validation';
  }
}

function mymodule_custom_validation(&$form, FormStateInterface $form_state) {
  $value = $form_state->getValue('some_field');
  if (empty($value)) {
    $form_state->setErrorByName('some_field', t('This field cannot be empty.'));
  }
}
```

**2. How to Render a Form Inside a Block?**

Step 1: Create a Block Plugin

Create a new file: my_module/src/Plugin/Block/MyCustomFormBlock.php

```php
namespace Drupal\my_module\Plugin\Block;

use Drupal\Core\Block\BlockBase;
use Drupal\Core\Form\FormBuilderInterface;
use Symfony\Component\DependencyInjection\ContainerInterface;
use Drupal\Core\Plugin\ContainerFactoryPluginInterface;

/**
 * Provides a block with a custom form.
 *
 * @Block(
 *   id = "my_custom_form_block",
 *   admin_label = @Translation("My Custom Form Block")
 * )
 */
class MyCustomFormBlock extends BlockBase implements ContainerFactoryPluginInterface {
  protected $formBuilder;

  public function __construct(array $configuration, $plugin_id, $plugin_definition, FormBuilderInterface $form_builder) {
    parent::__construct($configuration, $plugin_id, $plugin_definition);
    $this->formBuilder = $form_builder;
  }

  public static function create(ContainerInterface $container, array $configuration, $plugin_id, $plugin_definition) {
    return new static(
      $configuration,
      $plugin_id,
      $plugin_definition,
      $container->get('form_builder')
    );
  }

  public function build() {
    return $this->formBuilder->getForm('Drupal\my_module\Form\MyCustomForm');
  }
}
```

**Step 2: Enable the Block**

Go to Structure → Block layout.

Click Place Block in a region.

Search for "My Custom Form Block" and place it.

**3. How to Redirect a User After Submitting a Form?**

Inside the submitForm() method:

```php
public function submitForm(array &$form, FormStateInterface $form_state) {
  $form_state->setRedirect('user.page'); // Redirect to user profile page
}
```

To redirect to a custom route:

```php
$form_state->setRedirect('my_module.custom_page');
```
Ensure that my_module.custom_page is defined in mymodule.routing.yml.

**4. How to Display a Success Message (Green) After Submitting a Form
**

Use the Messenger service inside submitForm():

```php
\Drupal::messenger()->addMessage($this->t('Form submitted successfully!'), 'status');
```

Other message types:

'status' (green) → Success

'warning' (yellow) → Warning

'error' (red) → Error

Example:

```php
\Drupal::messenger()->addMessage($this->t('Something went wrong!'), 'error');
```

**5. How to Hide a Field for Anonymous Users Using #access?**

Check the user role in buildForm() and use #access:

```php
use Drupal\Core\Session\AccountInterface;
use Drupal::currentUser;

public function buildForm(array $form, FormStateInterface $form_state) {
  $user = \Drupal::currentUser();
  
  $form['admin_only_field'] = [
    '#type' => 'textfield',
    '#title' => 'Admin Only Field',
    '#access' => $user->hasPermission('administer site configuration'),
  ];

  return $form;
}
```

For anonymous users:
```php
$form['secret_field'] = [
  '#type' => 'textfield',
  '#title' => 'Secret Field',
  '#access' => \Drupal::currentUser()->isAuthenticated(), // Hides from anonymous users
];
```

**6. How to Group Fields Together in a Form?**
```php
Use #type => 'details' to create collapsible fieldsets:

$form['group'] = [
  '#type' => 'details',
  '#title' => $this->t('Grouped Fields'),
  '#open' => TRUE,
];

$form['group']['field_one'] = [
  '#type' => 'textfield',
  '#title' => $this->t('Field One'),
];

$form['group']['field_two'] = [
  '#type' => 'textfield',
  '#title' => $this->t('Field Two'),
];
```

Other options:
```php
#type => 'fieldset' (non-collapsible)

#type => 'container' (group fields without UI change)
```
