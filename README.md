Example chef-repo directory structure https://github.com/stephenlauck/pipeline-example-chef-repo

# Common release starting points

1. No cookbook version pinning and nothing has reached production yet (greenfield)
2. No cookbook version pinning and first release has delivered to production
3. pinning for the right reasons in the wrong places and in varying states of release (big mess)

Whichever state we are in, it's a good idea to pin the cookbooks running in anything considered 'production' which usually means anything that isn't dev or integration including production, pre-production, QA (if persistent) so new changes can be introduced without unplanned consumption.

# Release promotion setup

If no cookbooks are pinned in environment files:

 1. list all cookbooks and their versions on the Chef server

 `knife cookbook list`

 ```
apt               2.4.0
build-essential   2.0.4
chef_handler      1.1.6
dmg               2.2.0
emacs             0.9.0
git               4.0.2
runit             1.5.10
sudo              2.6.0
users             1.7.0
windows           1.33.1
yum               3.2.2
```

 2. Convert versions to json and store them in all environment files that are not dev or integration.

```
"cookbook_versions": {
  "apt": "= 2.4.0",
  "build-essential": "= 2.0.4",
  "chef_handler": "= 1.1.6",
  "dmg": "= 2.2.0",
  "emacs": "= 0.9.0",
  "git": "= 4.0.2",
  "runit": "= 1.5.10",
  "sudo": "= 2.6.0",
  "users": "= 1.7.0",
  "windows": "= 1.33.1",
  "yum": "= 3.2.2"
  },
```
3. Freeze all the cookbooks on the Chef server. Lots of ways to do this. Here is one example:

`cd /tmp`

`knife download cookbooks --chef-repo-path .`

`knife cookbook list | awk -F " " '{print $1}' | xargs -n1 knife cookbook upload --freeze -o cookbooks/`

# Release promotion

There should be at least one environment where no cookbook versions are pinned. This environment should get the latest cookbooks that have been delivered to the Chef server. This integration environment is meant to be broken while new code is being shipped and integrated. When this integration environment is not broken, it is a good time to cut a release and move the versions listed in step 1 above to a pre_production environment where the release can be tested again. If the pre_production environment passes all tests, the cookbook versions are then migrated into the production environment file and new cookbook versions are consumed in production when this file is delivered to the Chef server (if you have chef-client running on production nodes via cron/daemon).

Let's consider this release "v1.0.0"

#### environments/integration.json
```
{
  "name": "integration",
  "json_class": "Chef::Environment",
  "description": "Integration environment from which releases are cut",
  "chef_type": "environment"
}
```

#### environments/pre_production.json
```
{
  "name": "pre_production",
  "default_attributes": {

  },
  "json_class": "Chef::Environment",
  "description": "One step before production to test releases",
  "cookbook_versions": {
    "apt": "= 2.4.0",
    "build-essential": "= 2.0.4",
    "chef_handler": "= 1.1.6",
    "dmg": "= 2.2.0",
    "emacs": "= 0.9.0",
    "git": "= 4.0.2",
    "runit": "= 1.5.10",
    "sudo": "= 2.6.0",
    "users": "= 1.7.0",
    "windows": "= 1.33.1",
    "yum": "= 3.2.2"
  },
  "chef_type": "environment"
}
```

# Create new release
Ship new cookbook versions to the Chef server, make sure integration environment is not broken (all tests passing) and move a new set of versions from unpinned integration to pre_production. Notice only 3 cookbooks have been updated in this release. Submit new pull request on new branch with this change and when pull request ismerged onto master, create new tag with new release version ie "v1.0.1"

#### environments/pre_production.json
```
{
  "name": "pre_production",
  "default_attributes": {

  },
  "json_class": "Chef::Environment",
  "description": "One step before production to test releases",
  "cookbook_versions": {
    "apt": "= 2.4.0",
     "build-essential": "= 2.0.4",
     "chef_handler": "= 1.1.6",
     "dmg": "= 2.2.0",
-    "emacs": "= 0.9.0",
-    "git": "= 4.0.2",
+    "emacs": "= 0.9.2",
+    "git": "= 4.0.5",
     "runit": "= 1.5.10",
     "sudo": "= 2.6.0",
     "users": "= 1.7.0",
     "windows": "= 1.33.1",
-    "yum": "= 3.2.2"
+    "yum": "= 3.4.2"
   },
  "chef_type": "environment"
}
```

# Promote release to production
Move versions from pre_production environment to production environment and create new pull request for production change. When merged, this pull request is effectively a production deploy of the new version "v1.0.1".

#### environments/production.json
```
{
  "name": "production",
  "default_attributes": {

  },
  "json_class": "Chef::Environment",
  "description": "The production environment",
  "cookbook_versions": {
    "apt": "= 2.4.0",
     "build-essential": "= 2.0.4",
     "chef_handler": "= 1.1.6",
     "dmg": "= 2.2.0",
    -    "emacs": "= 0.9.0",
    -    "git": "= 4.0.2",
    +    "emacs": "= 0.9.2",
    +    "git": "= 4.0.5",
     "runit": "= 1.5.10",
     "sudo": "= 2.6.0",
     "users": "= 1.7.0",
     "windows": "= 1.33.1",
    -    "yum": "= 3.2.2"
    +    "yum": "= 3.4.2"
  },
  "chef_type": "environment"
}
```
