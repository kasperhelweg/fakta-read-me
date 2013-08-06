fakta-wordpress-plugin
======================

Wordpress plugin for managing API resources. Please read at least the TESTING section of this draft documentation before interacting with the plugin.

####Overall architecture
The plugin follows a simple MVC pattern. All classes are abstract and all methods are static, 
implying that class hierarchies are used merely as an aid in organization of concerns and namespacing. Controllers are meant to
be "skinny" and most processing takes place in the models and in the Core classes. A typical controller procedure might look like this:

    protected static function edit( )
    {
      global $meal_plan;
      global $recipes;
      
      $meal_plan   = MealPlan::edit( $_POST['resource_id'] );
        
      $all_recipes = Recipe::all( );
      $recipes     = Recipe::recipes_to_autocomplete( $all_recipes );
    }

In this case the corresponding model function is comparatively short:

    public static function edit( $id ) 
    { 
      self::$meal_plan = self::find( $id );
      set_transient('meal_plan', self::$meal_plan, 15 * MINUTE_IN_SECONDS );
    
      return self::$meal_plan;
    }
 
The programming style is classic; imperative and stateful. No particular attention has been given to performance
besides the caching techniques described in a later section. Performance is considered adequate and overhead generally lies in calling the API, allthough the code
leaves plenty of room for optimization.  

######Current state
The plugin is considered to be stable at the moment, but have known quirks and bugs. Furthermore, code refactoring is ongoing until considered adequate.
Current state 
  
######Language
PHP, Javascript, css and Wordpress action/filter hooks.

######Dependencies
The plugin is dependent on a HTML5 capable browser with Javascript enabled.
PHP >= 5.3.3 with curllib and Wp = 5.3.2. It is not known if these are "hard" dependencies. Curllib is and i would recommend freezing WP at 5.3.2 unless another version is really needed.
     
######Datastructures
"Datastructures" for displaying data are internal to WP. An example is the Wp-list-table. Other datastructures used are
PHP hash-tables, mapping to JSON for interacting with the API. As an aside, the datatables code is in need of heavy clean-up. It's a hacky mess. Also it should be considered whether 
extending the Wp-list-table is a good idea.  

######Staging environment
http://fakta.molamil.com/dash/wp-login.php

#####Folder structure
    /
    /documentation 
    -- Documentation goes here. 
    /css           
    -- Top-level css goes here.
    /js            
    -- Top-level Javascript goes here.
    /tests         
    -- Test's and spec's goes here.
    /includes
      /core
      -- Core classes goes here. These include CoreController, CoreModel and CoreHttp, among others.
      /static
      -- "Static" files goes here. These include plugin settings and global constants.
      /utilities
      -- Auxiliary functions.
      /:resource
  /:resource.constants.php
        -- Constants for the :resource
        /:resource.php
        -- Resource controller
        /:resource.class.php
        -- Resource model
        /views
          -- Views for the :resource goes here
        /css
          -- :resource specific css
        /js
          -- :resource specific Javascript

#####Javascript, css and Ajax:
- Besides a few libraries, only native Wordpress javascript and css is used. 
  Javascript and css is registered and enqueued using the appropriate Wordpress handlers.
- Ajax is handled using the native Wordpress Ajax handler. Ajax is NOT implemented.

#### Core classes and Interfaces
All controllers should extend the CoreController and implement the iController interface. The classes are easy to understand.
All models should extend the CoreModel class.
#### Caching
The caching system has two variants; consistent and non-consistent caching. All caching uses the database or memcached if available. 
The wordpress transient API is responsible for deciding in that matter. 

Since there are no API endpoints for the "utility" resources, these are not yet cacheable. "Utility" resources include "Authors", "Occasions", etc.

######Consistent caching
All GET request for resources are implemented as conditional's. This is classic, consistent caching.
######Non-consistent caching
Whenever a resource is changed by the CURRENT user, that resource is marked as changed.
The next time it is requested, a full GET is sent to the API. The advantage of consistent caching is 
user percieved performance; a resource is only requested if it was changed during the current users "session". 
The downside is that, if the plugin is used in a timeshared fashion, the cache might be inconsistent in the sense that it does not reflect the
actual, current state of the remote DB. 

###### Notes
If a single user system can be guaranteed i recommend turning OFF consistent caching and turning ON eager loading of resources 
(if this seems resonable form a performance perspective). If at any point a timeshared system is deployed, please turn ON consistent caching.
Also note that eager loading of resources has not been implemented yet, and may never be. This is an optimization issue that will be adressed
whenever the plugin is considered stable and ready for production.

####Manipulating resources, routing and the Http layer
Manipulation of resources follows a classic CRUD pattern, and actions are mapped as directly as possible to the API. 

####Settings and The Menu
- The settings can be accessed from the Wordpress settings menu.
- The Menu is a regular hack. Seperate documentation will be written on this glorious hack :)

####Testing
**!Important** : Please ensure that Use staging environment is checked in the settings section when testing.

When doing manual testing of core functionality please use a browser from the Chromium family with HTML5 capabilities and Javascript enabled.
For crossbrowser issues; use any browser( noting that HTML5 and Javascript enabled are denpendencies ).

There are no automated test's yet, and no plans to create such. 

####Security and data validation
No particular attention has been given to security. Should be considered a concern at some point, by someone. The plugig mirrors the API in terms of validation. If you would like data to be validated, please do so in the API first.

####Documentation
A lot is left wanted in terms of documentation. This is ongoing and will, at some point, be adequate enough for 
developers to use and understand. Documentation is understood as both top-level abstract documentation,
and class / method documentation. 

