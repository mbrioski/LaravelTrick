#How to make your Laravel package config file overwritten after its publication

I spent a lot of time to try to figure out of this stupid problem that anyway is not well documented in Laravel 4.2
After have developed your package or a workbech and had published your personal configuration:
```
php artisan config:publish vendor/namespace
```
The library still load the origin package configuration.

Why?

How to overwrite the package config file is not well documented in the [workbench Laravel documentation 4.2](http://laravel.com/docs/4.2/packages).

The solution is very simple: **tell your service provider to overwrite origin config file if an application config file exists**

So in your service provider definition add:
```
    public function boot() {
        $this->package('vendor/name', 'namespace');
    }
    
    public function register() {
        $loader = $this->app['config']->getLoader();

        // Get environment name
        $env = $this->app['config']->getEnvironment();

        // Add package namespace with path set, override package if app config exists in the main app directory
        if (file_exists(app_path() . '/config/packages/vendor/namespace')) {
            $loader->addNamespace('namespace', app_path() . '/config/packages/vendor/namespace');
        } else {
            $loader->addNamespace('namespace', __DIR__ . '/../../config');
        }

        $config = $loader->load($env, 'config', 'namespace');

        $this->app['config']->set('namespace::config', $config);
      
      ...  
    
    ```
    
    Now everybody use your library can overwrite the original configuration.
