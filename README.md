# clean-drupal
Clean drupal install for infrastructure testing.

## Configure the database connection
The database connection details should be set in the untracked file `web/sites/default/settings/settings.local.php`. An example of how this should look is in the file `web/sites/default/settings/settings.local.php.dist`.

## First install
```shell script
# Clone the repo
git clone https://github.com/dreamproduction/clean-drupal.git clean-drupal
# Install dependecies, optimized for production
php -d memory_limit=-1 /path/to/composer install --no-interaction --no-dev -o
# Install a clean drupal.
drush site-install --account-name=$SITE_USERNAME --account-pass=$SITE_PASSWORD --yes
# Delete shortcut entities so we can import the site config.
drush entity:delete shortcut
# Set the site uuid the same as the site uuid in the exported config,
# Drupal only allows importing config for the same site.
drush cset system.site uuid 08efa902-1be5-4e01-90f6-577b4c3de64f -y
# Run import of config
drush cim -y sync
# Clear the cache.
drush cr
```

## Update
```shell script
# Make sure the config folder is writeable in case there are changes in it
# that need to be merged.
chmod +w -R web/sites/default
# Get the latest changes from git. Alternatively if the strategy is to do a
# clean install every time, use git clone and skip the first 
git pull
# Install dependecies, optimized for production
php -d memory_limit=-1 /path/to/composer install --no-interaction --no-dev -o
# Run the db updates.
drush updb -y
# Run import of config twice, sometimes the config dependencies are not
# resolved correctly by drupal and running this twice is needed.
drush cim -y sync
drush cim -y sync
# Clear the cache.
drush cr
```

## Detecting if it's fist install or update
```shell script
isBootstraped=$(drush st bootstrap | grep Successful > /dev/null && echo yes || echo no)
if [ $isBootstraped == no ] ; then
   # install Drupal
else
   # update Drupal
fi
```
