# PDK Templates
<!-- TOC -->

- [PDK Templates](#pdk-templates)
    - [Basic Usage](#basic-usage)
    - [Config_default Values <a name="values"></a>](#config_default-values-a-namevaluesa)
        - [.gitattributes](#gitattributes)
        - [.gitignore](#gitignore)
        - [.pdkignore](#pdkignore)
        - [.travis.yml](#travisyml)
        - [appveyor.yml](#appveyoryml)
        - [Rakefile](#rakefile)
        - [.rubocop.yml](#rubocopyml)
        - [Gemfile](#gemfile)
    - [Further Notes <a name="notes"></a>](#further-notes-a-namenotesa)
    - [Portal Operations adaptations](#portal-operations-adaptations)
        - [Usage](#usage)
        - [Example to create a new module](#example-to-create-a-new-module)
        - [Gain advantage](#gain-advantage)

<!-- /TOC -->

The PDK Templates is the default templates repository for use with the [Puppet Development Kit](https://github.com/puppetlabs/pdk), within which we have defined all the templates for the creation and configuration of a module. Look into these directories to find the templates:
* `moduleroot` templates get deployed on `new module`, `convert` and `update`; use them to enforce a common boilerplate for central files.
* `moduleroot_init` templates get only deployed when the target module does not yet exist; use them to provide skeletons for files the developer needs to modify heavily.
* `object_templates` templates are used by the various `new ...` commands for classes, defined types, etc.

The PDK also absorbs the `config_defaults.yml` file to apply a set of default configurations to the module. Each top-level key in the file corresponds to a target file, and will be merged with the `:global` section at the top. Within the template evaluation the values are available under `@config`. In the module itself, you can override/amend the values by putting new values into `.sync.yml` in the module's root. You can remove default values by using the [knockout prefix](#removing-default-configuration-values). The data for a target file also use `delete: true` and `unmanaged: true` to remove, or ignore the particular file.

* [Basic usage](#basic-usage)
* [Values of config\_defaults](#values)
* [Making local changes to the Template](#making-local-changes-to-the-template)
* [Further Notes](#notes)

## Basic Usage

Templates like this one can be used in conjunction with the PDK. As default the PDK itself uses the templates within this repository to render files for use within a module. Templates can be passed to the PDK as a flag for several of the commands.

> pdk convert --template-url https://github.com/puppetlabs/pdk-templates

Please note that the template only needs to be passed in once if you wish to change it, every command run on the PDK will use the last specified template.
For more on basic usage and more detailed description of the PDK in action please refer to the [PDK documentation](https://github.com/puppetlabs/pdk/blob/master/README.md).

##  Values of config\_defaults <a name="values"></a>

The following is a description and explaination of each of the keys within config\_defaults. This will help clarify the default settings we choose to apply to pdk modules.

### .gitattributes

>A .gitattributes file in your repo allows you to ensure consistent git settings.

| Key               | Description   |
| :-----------------|:--------------|
| include           | Defines which extensions are handled by git automatic conversions (see the [gitattributes](https://git-scm.com/docs/gitattributes) documentation). The default configuration helps to keep line endings consistent between windows and linux users.|

### .gitignore

>A .gitignore file in your repo allows you to specify intentionally untracked files to ignore.

| Key               | Description   |
| :-----------------|:--------------|
| paths             | Defines which files or paths for git to ignore or untrack. (see the [gitignore](https://git-scm.com/docs/gitignore) documentation). The default configuration helps to set up commonly ignored or untracked files in a module project.

### .pdkignore

>A .pdkignore file in your repo allows you to specify files to ignore when building a module package with `pdk build`.

| Key               | Description   |
| :-----------------|:--------------|
| paths             | Defines which files or paths for PDK to ignore when building a module package. The default configuration helps to set up commonly ignored files in a module project when building a package.

### .travis.yml

>[Travis CI](https://travis-ci.org/) is a hosted continuous integration platform that is free for all open source projects hosted on Github.
We can trigger automated builds with every change to our code base in the master branch, other branches or even a pull request.
Travis uses a .travis.yml file in the root of your repository to learn about your project and how you want your builds to be executed.

| Key            | Description   |
| :------------- |:--------------|
| ruby versions  |Define the ruby versions on which you want your builds to be executed.|
| bundler\_args   |Define any arguments you want to pass through to bundler. The default is `--without system_tests` which avoids installing unnessesary gems.|
| env            |Allows you to add new travis job matrix entries based on the included environmnet variables, one per `env` entry; for example, for adding jobs with specific `PUPPET_GEM_VERSION` and/or `CHECK` values.  See the [Travis Environment Variables](https://docs.travis-ci.com/user/environment-variables) documentation for details.|
| global_env     |Allows you to set global environment variables which will be defined for all travis jobs; for example, `PARALLEL_TEST_PROCESSORS` or `TIMEOUT`.  See the [Travis Global Environment Variables](https://docs.travis-ci.com/user/environment-variables/#Global-Variables) documentation for details.|
|docker_sets     |Allows you to configure sets of docker to run your tests on. For example, if I wanted to run on a docker instance of Ubuntu I would add  `set: docker/ubuntu-14.04` to my docker\_sets attribute.|
|docker_defaults |Defines what values are used as default when using the `docker_sets` definition. Includes ruby version, sudo being enabled, the distro, the services, the env variables and the script to execute.|
|includes        |Ensures that the .travis file includes the following checks by default: Rubocop, Puppet Lint, Metadata Lint.|


### appveyor.yml

>[AppVeyor](https://www.appveyor.com/) is a hosted, distributed continuous integration service used to build and test projects hosted on GitHub by spinning up a Microsoft Windows virtual machine. AppVeyor is configured by adding a file named appveyor.yml, which is a YAML format text file, to the root directory of the code repository.

| Key            | Description   |
| :------------- |:--------------|
|appveyor\_bundle\_install|Defines the bundle install command for the appveyor execution run. In our case we use bundle install ```--without system_tests``` as default, therefore avoiding redundant gem installation.|
|environment|Defines any environment variables wanted for the job run. In our case we default to the latest Puppet 4 gem version.|
|matrix|This defines the matrix of jobs to be executed at runtime. Each defines environment variables for that specific job run. In our defaults we have a Ruby version specfied, followed by the check that will be run for that job.|
|test\_script|This defines the test script that will be executed. For our purposes the default is set to ```bundle exec rake %CHECK%```. As appveyor iterates through the test matrix as we defined above, it resolves the variable CHECK and runs the resulting command. For example, our last test script would be executed as ```bundle exec rake spec```, which would run the spec tests of the module.|

### Rakefile

>[Rake](https://github.com/ruby/rake) is a Make-like program implemented in Ruby. Tasks and dependencies are specified in standard Ruby syntax within the Rakefile, present in the root directory of the code repository. Within modules context Rake tasks are used quite frequently, from ensuring the integrity of a module, running validation and tests, to tasks for releasing modules.

| Key            | Description   |
| :------------- |:--------------|
|default\_disabled\_lint\_checks| Defines any checks that are to be disabled by default when running lint checks. As default we disable the `--relative` lint check, which compares the module layout relative to the module root. |
|extra\_disabled\_lint\_checks| Defines any checks that are to be disabled as extras when running lint checks. No defaults are defined for this configuration. |
|extras|An array of extra lines to add into your Rakefile. As an alternative you can add a directory named `rakelib` to your module and files in that directory that end in `.rake` would be loaded by the Rakefile.|

### .rubocop.yml

>[RuboCop](https://github.com/bbatsov/rubocop) is a Ruby static code analyzer. We use Rubocop to enforce a level of quility and consistancy within Ruby code. Rubocop can be configured within .rubocop.yml which is located in the root directory of the code repository. Rubocop works by defining a sanitized list of cops that'll cleanup a code base without much effort, all of which support autocorrect and that are fairly uncontroversial across wide segments of the Community.

| Key            | Description   |
| :------------- |:--------------|
|selected\_profile|Allows you to define which profile is used by default, which is set to ```strict```, which is fully defined within the ```profiles``` section.|
|default\_configs |Allows you to define the default configuration of which cops will run. Includes the full name of the cop followed by a description of it and an enforced style. Can also make use of the key ```excludes``` to exclude any files from that specific cop.|
|cleanup\_cops    |Defines a set of cleanup cops to then be included within a rubocop profile. Cops are defined by their full name, and further configuration can be done by specifying secondary keys. By default we specify a large amount of cleanup cops using their default configuration.|
|profiles         |Defines the profiles that can be enabled and used within rubocop through the ```selected_profile``` option. By default we have set up three profiles: cleanups\_only, strict, hardcore and off.|


### Gemfile

>A Gemfile is a file we create which is used for describing gem dependencies for Ruby programs. All modules should have an associated Gemfile for installing the relevant gems. As development and testing is somewhat consistant between modules we have used the template to define a set of gems relevant to these processes.

| Key            | Description   |
| :------------- |:--------------|
|required|Allows you to specify gems that are required within the Gemfile. Gems can be defined here within groups, for example we use the :development gem group to add in several gems that are relevant to the development of any module.|
|optional|Allows you to specify additional gems that are required within the Gemfile. This key can be used to further configure the Gemfile through assignment of a value in the .sync.yml file.|

### spec/default_facts.yml
> The spec/default_facts.yml file contains a list of facts to be used by default when running rspec tests

| Key            | Description   |
| :------------- |:--------------|
|concat_basedir|Overrides the concat_basedir fact's value in the base template. Defaults to "/tmp".|
|ipaddress|Overrides the ipaddress fact's value in the base template. Defaults to "172.16.254.254".|
|is_pe|Overrides the is_pe fact's value in the base template. Defaults to false.
|macaddress|Overrides the macaddress fact's value in the base template. Defaults to "AA:AA:AA:AA:AA:AA".
|extra_facts|List of extra facts to be added to the default_facts.yml file. They are in the form: "`name of fact`: `value of fact`"|

### spec/spec_helper.rb
> The spec/spec_helper.rb file contains setup for rspec tests

| Key            | Description   |
| :------------- |:--------------|
|hiera_config|Sets the [`hiera_config`](http://rspec-puppet.com/documentation/configuration/#hiera_config) rspec-puppet parameter.|
|mock_with|Defaults to `':mocha'`. Recommended to be set to `':rspec'`, which uses RSpec's built-in mocking library, instead of a third-party one.|
|spec_overrides|An array of extra lines to add into your `spec_helper.rb`. Can be used as an alternative to `spec_helper_local`|
|strict_level| Defines the [Puppet Strict configuration parameter](https://puppet.com/docs/puppet/5.4/configuration.html#strict). Defaults to `:warning`. Other values are: `:error` and `:off`. `:error` provides strictest level checking and is encouraged.|
|coverage_report|Enable [rspec-puppet coverage reports](https://rspec-puppet.com/documentation/coverage/). Defaults to `false`|
|minimum_code_coverage_percentage|The desired code coverage percentage required for tests to pass. Defaults to `0`|

## Removing default configuration values

Values can be removed from the data passed to the templates using the [knockout prefix](https://www.rubydoc.info/gems/puppet/DeepMerge) `---` in `.sync.yml`.

To remove a value from an array, prefix the value `---`. For example, to remove
`2.5.1` from the `ruby_versions` array in `.travis.yml`:

```yaml
.travis.yml:
  ruby_versions:
    - '---2.5.1'
```

To remove a key from a hash, set the value to `---`. For example, to remove the
`ipaddress` fact from `spec/default_facts.yml`:

```yaml
spec/default_facts.yml:
  ipaddress: '---'
```

## Further Notes <a name="notes"></a>

Please note that the early version of this template contained only a 'moduleroot' directory, and did not have a 'moduleroot\_init'. The PDK 'pdk new module' command will still work with templates that only have 'moduleroot', however the 'pdk convert' command will fail if the template does not have a 'moduleroot_init' directory present. To remedy this please use the up to date version of the template.

## Portal Operations adaptations

Similar to [po-puppet-module-skeleton](https://git.mamdev.server.lan/pocu/po-puppet-module-skeleton), this is an adaptation of upstream module template [puppetlabs/pdk-templates](https://github.com/puppetlabs/pdk-templates), to be used with [PDK](https://rubygems.org/gems/pdk), the Puppet Development Kit version 1.3 and later.

It is recommended to use the latest supported gem version of PDK along with this templates repository. Refer to [po-pdk-templates/tags](https://git.mamdev.server.lan/pocu/po-pdk-templates/tags) for supported PDK versions.

### Usage

1. Install PDK: as Gem or Package. See [PDK Documentation](https://puppet.com/docs/pdk/1.3/index.html), check out [PDK source](https://github.com/puppetlabs/pdk) or preferably, use the [PDK Gem](https://rubygems.org/gems/pdk).
1. Set template repository to this one. First time, by adding ``--template-url`` to the pdk command line; or edit the answer file at ``~/.pdk/cache/answers.json``.

### Example to create a new module
```
$ pdk new module --template-url https://git.mamdev.server.lan/pocu/po-pdk-templates.git po_demo
pdk (INFO): Creating new module: po_demo

We need to create the metadata.json file for this module, so we're going to ask you 4 questions.
If the question is not applicable to this module, accept the default option shown after each question. You can modify any answers at any time by manually updating the metadata.json file.

[Q 1/4] If you have a Puppet Forge username, add it here.
We can use this to upload your module to the Forge when it's complete.
--> trepasi

[Q 2/4] Who wrote this module?
This is used to credit the module's author.
--> trepasi

[Q 3/4] What license does this module code fall under?
This should be an identifier from https://spdx.org/licenses/. Common values are "Apache-2.0", "MIT", or "proprietary".
--> Apache-2.0

[Q 4/4] What operating systems does this module support?
Use the up and down keys to move between the choices, space to select and enter to continue.
--> Debian based Linux

Metadata will be generated based on this information, continue? Yes
pdk (INFO): Module 'po_demo' generated at path '/Users/trepasi/git/PuppetModules/po_demo', from template 'https://git.mamdev.server.lan/pocu/po-pdk-templates.git'.
pdk (INFO): In your module directory, add classes with the 'pdk new class' command.
```

### Gain advantage

Push the resulting module wireframe to a PO-GitLab project. This will instantly let some magic happen, by starting some pre-defined GitLab CI pipelines:
* ``rubocop`` - runs [RuboCop](https://rubocop.readthedocs.io/en/latest/) to style check all ruby code found in the module
* ``spec`` - runs the ``release_checks`` task from [puppetlabs_spec_helper](https://github.com/puppetlabs/puppetlabs_spec_helper)
* ``beaker:*`` - runs acceptance tests with [Beaker](https://github.com/puppetlabs/beaker) against multiple OS releases<sup>[1](#beaker-footnote)</sup>
* ``pages`` - render module documentation using [puppet-strings](https://puppet.com/blog/using-puppet-strings-generate-great-documentation-puppet-modules)
* ``pages::deploy`` - deploy the documentation to [GitLab-Pages](https://mam-confluence.1and1.com/x/qZaVAw)

<a name="beaker-footnote">1</a>: Note, that Beaker tests are not currently  running out of the box, however, pipelines are defined some manual setup is still needed. Don't hesitate to ask us for details, if you need to get that running.
