SPRINT 5 - DAY 1 : 
------------------
------------------

Uninstall the module, and install them again, what happens ?
------------------------------------------------------------- 

The module is disabled and removed from the system.
Any configuration and data related to the module will be deleted.
The database entries related to this module will be removed.

And when we Reinstall the Module : 

The module is added back into Drupal’s system.
A new entry is created in the database.


Make change to core_version_requirement to a lower version, what happens ?
---------------------------------------------------------------------------

In the .info.yml, We modify the file : 
core_version_requirement: ^9 || ^10 || ^11 to core_version_requirement: ^8


Clear the cache. 


Result: Drupal will mark the module as incompatible and may prevent it from being installed or enabled.


Add the module workflows as a dependency to your dependencies, what happens ? : 
---------------------------------------------------------------------------------- 

In the .info.yml we add Workflows as dependency : 

dependencies:
  - workflows:workflows

The workflows:workflows part tells Drupal that the Workflows module is required for the module to work.
When we try to install it from the UI , it gonna tell us that the workflows is required for the module.



Add a composer.json file to your modules : 
---------------------------------------------

By adding composer.json, we ensure that your module's dependencies (if any) are properly managed and can be automatically installed using Composer.
It also allows us to integrate our module into the Drupal ecosystem, ensuring proper versioning and dependency handling.


In the /modules/custom/hello_world : we create a new composer.json file.

To install the dependencies with composer : we run composer install in the same path of the composer.json and enbale it after with drush en hello_word . 


Delete your modules or move them to another directory, what happens ? what message appear in drupal ? what the log says drush ws ?
------------------------------------------------------------------------------------------------------

The error : (Circular Reference)

The website encountered an unexpected error. Try again later.

[error]  ServiceCircularReferenceException: Circular reference detected for service 'module_handler', path: 'module_handler -> http_kernel -> http_middleware.ajax_page_state -> http_middleware.negotiation -> http_middleware.reverse_proxy -> shield.middleware -> ban.middleware -> http_middleware.page_cache -> page_cache_request_policy -> persistent_login.page_cache_request_policy.pending_persistent_login -> persistent_login.cookie_helper -> config.factory -> config.typed'.

WHY : ?? 
--------- 
1. Missing Dependencies: Some modules may be core to the functionality of other modules. If a module is  deleted and was a dependency for others, those other modules will fail to load properly, which might involve services that depend on each other in a circular way.
2. Service Destruction: The deletion of a module can remove its service from the container. If other services (like module_handler, http_kernel, etc.) rely on that missing service, it creates unresolved dependencies. If these dependencies form a circular chain, it leads to the circular reference error.
3. Improper Cleanup: Sometimes, when you delete a module, it doesn't fully clean up all its associated configurations, services, or links. This can leave services or configurations that are still referenced by other parts of the system, leading to circular dependencies.

