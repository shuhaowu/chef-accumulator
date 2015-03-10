Description
===========

Accumulator is a Chef primitaive that allows you to collect (accumulate) 
`Chef::Resource` from the current `run_context`, optionally transform this
collection and then provide as input for a target `Template` or `File` 
resource.

Cookbooks using Accumulator
===========================

- [sysctl](https://github.com/spheromak/sysctl-cookbook)

Howto
=====

Basics
------

Chef runs happen in two phases: the compilation phase and the execution phase.
In the compliation phase, all resources gets collected by Chef and stored.
In the execution phase, accumulator goes through and filters out the desired
resources and then pass them as a list variable to target. It then calls
the `create` action on the target.


Example usage
-------------

For the example, we'll look into writing into the fstab file. 

First, we can setup a cookbook under the directory `fstab`. It will have a
structure like the following:

```
fstab/
  recipes/
    default.rb
  providers/
    default.rb
  resources/
    default.rb
  templates/
    default/
      fstab.erb
  metadata.rb
```

First, we want to define a LWRP, which means a resource and a provider. 
The resource will describe all the properties of fstab lines.

```ruby
# fstab/resources/default.rb

action :create
default_action :create

attribute :fs_spec, :kind_of => String, :name_attribute => true
attribute :fs_file, :kind_of => String, :required => true

# .... fill out this file from the fields in fstab's man page

```

The provider is basically a noop. This is because we are not actually writing
anything with this resource. The accumulator will do the job.

```ruby
# fstab/providers/default.rb
def whyrun_supported?
  true
end

use_inline_resources

action :create do
  # implemented using accumulator
end
```

With the LWRP defined, we can define a recipe that collects all the resource
defined in various cookbooks and write it out to a file.

```ruby
# fstab/recipes/default.rb
template "/etc/fstab" do
  source "fstab.erb"
  owner "root"
  group "root"
  mode 0644
  action :nothing # We don't want it to write out because the accumulator will actually write the file
end

accumulator "/etc/fstab" do
  target :template => "/etc/fstab" # same as the name of the template
  # filters for if the resource is an fstab. This can be any function.
  # in some context this is known as the `map` function
  filter { |res| res.is_a? Chef::Resource::Fstab } 

  # transforms the resources sorting them alphabetically.
  # again, this can be any function you want
  transform { |resources| resources.sort_by { |r| r.fs_spec } }

  variable_name :fstabs
end
```


License and Authors
-------------------

Author:: Mathieu Sauve-Frankel (<msf@kisoku.net>)

Copyright:: Copyright (c) 2012 Mathieu Sauve-Frankel

License:: Apache License, Version 2.0

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
