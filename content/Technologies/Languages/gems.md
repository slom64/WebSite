RVM allows you to install and switch between different Ruby versions. Each Ruby version managed by RVM can have its own isolated set of gems, called a "gemset." This prevents conflicts between different projects that might require different gem versions. Installing a Ruby version.

```ruby
rvm install ruby-2.7.5
```
Using a specific Ruby version.
```
rvm use ruby-2.7.5
```
Creating a gemset.
```
rvm gemset create my_project_gemset
```
Using a specific gemset.
```
rvm use ruby-2.7.5@my_project_gemset
```

### Installing Gems.

Once you have selected a Ruby version and optionally a gemset with RVM, you then use the standard `gem install` command to install gems within that specific environment. RVM ensures that these gems are installed into the correct, isolated location associated with your chosen Ruby version and gemset.
```
gem install rails
```

### Listing Gems.

Similarly, when you use `gem list`, it will show you the gems installed in the currently active RVM Ruby version and gemset.
```
gem list
```

```
╭─slom at slom-Legion-5-Pro-16IAH7H in ~
╰─○ rvm use 3.2.2@wpscan --create

RVM is not a function, selecting rubies with 'rvm use ...' will not work.

You need to change your terminal emulator preferences to allow login shell.
Sometimes it is required to use `/bin/bash --login` as the command.
Please visit https://rvm.io/integration/gnome-terminal/ for an example.

╭─slom at slom-Legion-5-Pro-16IAH7H in ~
╰─○ /bin/bash --login; source ~/.rvm/scripts/rvm; rvm use 3.2.2@wpscan --create
```