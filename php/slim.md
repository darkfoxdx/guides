Install Slim framework
https://www.slimframework.com/docs/v4/start/installation.html

1. composer require slim/slim:"4.*"
2. composer require slim/psr7

To run on Apache
https://github.com/selective-php/basepath

1. composer require selective/basepath
2. $app->add(new BasePathMiddleware($app));
