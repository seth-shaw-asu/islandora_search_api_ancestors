Additional Membership Drupal Search
===================================

# Overview

We currently can't have a collection search contain the results of _both_ subcollections _and_ additional membership relationships. This is described with a bit more detail in a [problem statement document on Sharepoint](https://arizonastateu.sharepoint.com/:w:/r/sites/O365LIBInfoArch/Shared%20Documents/Our%20Products/Repository%20(%20KEEP%20and%20PRISM%20)/Search%20within%20Collection.docx?d=w77f4e094f6864361ae03240729d6b065&csf=1&web=1&e=IBRKKK). We need a solution that creates a union of these two sets.

## Discarded Idea: Multiple Contextual Filters

An early idea was to include multiple contextual filters, for both the member of and additional member fields. However, this would not include sub-collection descendant items where a sub-collection uses the additional member field.

# DDev Setup

This directory contains a database dump and composer files for setting up a DDev starting point for doing so.

Copy this `index-hierarchy` directory to where you want to use it. Run the following commands in that directory:

```
ddev config --project-type=drupal10 --php-version=8.3 --docroot=web
ddev start
ddev composer install
ddev import-db --file=../index-hierarchy.sql.gz
ddev launch
```

The username and password are both 'admin'.

# Testing

As currently set up, which mimics our current scenario, the two following SQL queries represent our current 'member of' options:

- `select * from  search_api_db_default_index_field_member_of where value = 1;` represents the `member_of` field with 'Index Heirarchy' configured.
- `select * from  search_api_db_default_index_field_combined_member_of where value = 1;` represents the aggregated field combining `field_member_of` and `field_additional_member_of`.

The ideal solution would equate a union of these two sets.

# Notes

The current solution of the aggregated field doesn't work because it combines the values _before_ the 'Index Heirarchy' processor runs.

Ideally the 'Index Heirarchy' processor would allow 'checkboxes' instead of 'radios' for the fields it will walk; that way we could just add the additional memberships field to the member of field which would remove the need for the aggregated one. It isn't a matter of just changing the configuration form, because it won't match the config schema. We _might_ be able to _extend_ the plugin to make it work and not rewrite _everything_.

# Solution

I created a new [Preprocessor Plugin](web/modules/islandora_search_api_ancestors/src/Plugin/search_api/processor/AddAncestors.php) that takes heavily from the `search_api` Index Heirarchy plugin but instead allows for multiple fields to follow. It uses IslandoraUtils to get a list of ancestors using all of the configured fields. It also starts with the item itself instead of with the existing field values to ensure all the relevant ancestor fields get crawled.

To use it, enable the 'Islandora Ancestors' preprocessor plugin and configure it in the section below and then run a re-index.
