See https://github.com/symfony/recipes/issues/829

```bash
composer install
```

Note that `access_decision_manager.strategy` is set to `unanimous` in `config/packages/security.yaml`.

```bash
./bin/console debug:config security access_decision_manager

# Current configuration for "security.access_decision_manager"
# ============================================================
#
# strategy: affirmative
# allow_if_all_abstain: false
# allow_if_equal_granted_denied: true
```

```bash
APP_ENV=prod ./bin/console cache:clear
APP_ENV=prod ./bin/console debug:config security access_decision_manager

# Current configuration for "security.access_decision_manager"
# ============================================================
#
# strategy: unanimous
# allow_if_all_abstain: false
# allow_if_equal_granted_denied: true
```


Now swap the file loading order in `src/Kernel.php`:

```diff
     protected function configureContainer(ContainerConfigurator $container): void
     {
         // Default order
-        $container->import('../config/{packages}/*.yaml');
-        $container->import('../config/{packages}/'.$this->environment.'/*.yaml');
+        // $container->import('../config/{packages}/*.yaml');
+        // $container->import('../config/{packages}/'.$this->environment.'/*.yaml');

         // If loaded in this order an exception is (rightfully) thrown
         // "You are not allowed to define new elements for path
         // "security.firewalls". Please define all elements for this path in
         // one config file."
-        // $container->import('../config/{packages}/'.$this->environment.'/*.yaml');
-        // $container->import('../config/{packages}/*.yaml');
+        $container->import('../config/{packages}/'.$this->environment.'/*.yaml');
+        $container->import('../config/{packages}/*.yaml');

         if (is_file(\dirname(__DIR__).'/config/services.yaml')) {
             $container->import('../config/services.yaml');
```

Dump the security config for `APP_ENV=dev` again.
We get an error this time, which is expected.

```bash
./bin/console debug:config security access_decision_manager

# In PrototypedArrayNode.php line 303:
#
#   You are not allowed to define new elements for path "security.firewalls". Please define all elements for this path in one config file.
```
