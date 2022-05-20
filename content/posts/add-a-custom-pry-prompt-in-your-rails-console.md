---
title: "Add a Custom Pry Prompt in your Rails Console"
date: 2022-01-20T02:26:25+06:00
draft: false
---

[Pry](https://github.com/pry/pry) is an awesome debugging tool for Ruby. The gem [pry-rails](https://github.com/pry/pry-rails) seamlessly integrates Pry with rails.

With `pry-rails` included in the project Gemfile, it opens the pry prompt whenever we run `rails console`.

```
[1] pry(main)>
```

We can customize this default prompt to show useful information like the current project name & environment name. [pry-rails](https://github.com/pry/pry-rails) gem provides a custom prompt for just that. Add the following line in your `~/.pryrc` file to enable the rails prompt provided by `pry-rails` gem.

```ruby
Pry.config.prompt = Pry::Prompt[:rails]
```

This makes the prompt looks like below:

```
[1] [my_project][development] pry(main)>
```
To see other available prompts, run this command from pry repl: `change-prompt --help`

## Add our own pry prompt

Like the rails prompt, we can add our own prompt to customize the prompt string to our liking.

I wanted to add the current tenant name in the prompt when working in a multi-tenant application.

To do that, I added the following snippet in `~/.pryrc`

```ruby
# Custom prompt for Apartment MultiTenant App
if Pry::Prompt[:rails] && Object.const_defined?('Apartment')
  begin
    current_tenant_name = proc { Pry::Helpers::Text.magenta(Apartment::Tenant.current) }
    name = 'multi_tenant'
    desc = 'Pry prompt with tenant name from Apartment gem'
    separators = %w[> *]

    Pry::Prompt.add(name, desc, separators) do |target_self, nest_level, pry, sep|
      "[#{pry.input_ring.size}] " \
      "[#{PryRails::Prompt.project_name}][#{PryRails::Prompt.formatted_env}]" \
      "[#{current_tenant_name.call}] " \
      "#{pry.config.prompt_name}(#{Pry.view_clip(target_self)})" \
      "#{":#{nest_level}" unless nest_level.zero?}#{sep} "
    end

    Pry.config.prompt = Pry::Prompt[:multi_tenant]
  rescue StandardError => e
    puts e.inspect
    Pry.config.prompt = Pry::Prompt[:rails]
  end
end
```
Here, we are basically customizing the rails prompt code found [here](https://github.com/pry/pry-rails/blob/12a517e974a290485b25a54413e314e3a5e2d59d/lib/pry-rails/prompt.rb?_pjax=%23js-repo-pjax-container%2C%20div%5Bitemtype%3D%22http%3A%2F%2Fschema.org%2FSoftwareSourceCode%22%5D%20main%2C%20%5Bdata-pjax-container%5D#L28) to include the current tenant name.

Now our prompt looks like this:

```
[1] [my_project][development][public] pry(main)>
[2] [my_project][development][public] pry(main)> Apartment::Tenant.switch! 'admin'
[3] [my_project][development][admin] pry(main)>
```
Look how switching the current tenant name is reflected in the prompt. This is quite useful when debugging and helps not to loose context.

We can also run specific queries for a project before opening the console.
For example, I wanted to switch to admin tenant every time I open the console in my `test_project`.

So, I added the following snippet in my `~/.pryrc` along with the two helper methods that I can call from the console.

```ruby
if PryRails::Prompt.project_name == 'test_project'
  def switch_admin
    Apartment::Tenant.switch! 'admin'
  end

  def switch_public
    Apartment::Tenant.switch! 'public'
  end

  switch_admin if Tenant.where(identifier: 'admin').present?
end
```

Result..

```
$ rails console
[1] [my_project][development][admin] pry(main)> switch_public
[2] [my_project][development][public] pry(main)>
```

Thank you for reading. Let me know in the comments if you find this useful.
