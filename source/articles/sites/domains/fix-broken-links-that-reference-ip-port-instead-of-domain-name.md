---
title: Fix broken links that reference IP:PORT instead of domain name
description: Learn how to update broken links so that the URL references the correct file path and domain name.
category:
  - debugging
  - supporting

---

## Scenario:

When editing content, links are inserted that don't reflect the site's domain name. For example, an image URL appears as http://192.237.142.203:5555/files/cernettes.gif instead of the proper http://www.example.com/files/cernettes.gif

The link may work at first, but will eventually break when your application container’s IP address changes due to the nature of Pantheon’s cloud-based infrastructure.

## Solution:

**Drupal:** Set the $base\_url per environment in settings.php and clear caches.

**WordPress:** Currently no known solution.

## Background

Sometimes, the $base\_path, if not explicitly set, is the URL from which the asset was loaded. If a module cache is populated on a drush cron run from the CLI, this can be the app-container IP which is dynamic and will ultimately become a broken link. The current resolution is to set $base\_url per environment in settings.php and clear caches. There are some notes about this in the Pathologic documentation:

https://www.drupal.org/node/257026

If you look through the Media module issue queue, you’ll see that other users have experienced the same or very similar issues with WYSIWYG-inserted images having their base URLs cached. The IP address is coming from a cached reference to the image’s location as generated by the media filter. Dave Reid (the Media module maintainer) has explained it fairly well in this comment:

https://drupal.org/node/1660936#comment-6270618

The recommended solution is setting the $base\_url in your settings.php. This should prevent the IP:PORT address from being stored in URLs. Check that you're using the latest version of the module(s).

Here is an example of a code snippet you could use to set the $base\_path per environment:

    if (isset($_SERVER['PANTHEON_ENVIRONMENT'])) {
      switch ($_SERVER['PANTHEON_ENVIRONMENT']) {
        case 'dev':
          $base_url = 'http://dev-example.gotpantheon.com'; // NO trailing slash!
          break;
        case 'test':
          $base_url = 'http://test-example.gotpantheon.com'; // NO trailing slash!
          break;
        case 'live':
          $base_url = 'http://www.example.tld'; // NO trailing slash!
          break;
        }
    }

Try customizing that snippet for your purposes and putting in your settings.php, and then clearing caches. All cached IP:PORT references will be wiped out, and in the future, re-populated with the correct base url.