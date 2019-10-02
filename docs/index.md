# Welcome to the Autolab Docs

Autolab is a course management platform that enables instructors to offer autograded programming assignments to their students. The two key ideas in Autolab are _autograding_ that is, programs evaluating other programs, and _scoreboards_ that display the latest autograded scores for each student. Autolab also provides gradebooks, rosters, handins/handouts, lab writeups, code annotation, manual grading, late penalties, grace days, cheat checking, meetings, partners, and bulk emails.

For information on how to use Autolab for your course see the [Guide for Instructors](/instructors). To learn how to write an autograded lab see the [Guide for Lab Authors](/lab).

## Getting Started

Autolab consists of two services: (1) the Ruby on Rails frontend, and (2) [Tango](/tango), the RESTful Python autograding server. Either service can run independently without the other. But in order to use all features of Autolab, we highly recommend installing both services.

Currently, we have support for installing Autolab on [AWS](#aws), [Ubuntu 14.04+](#ubuntu-1404), and [Mac OSX](#mac-osx-1011).

### AWS

If you want to try Autolab or Tango without any installation, we have a pre-built AMI you can deploy on AWS' free tier. To deploy it, follow the official [EC2 Launch Instructions](https://aws.amazon.com/premiumsupport/knowledge-center/launch-instance-custom-ami/). Make sure that you

1. Select 'Autolab vx.x.x' as the ami. You can find this in the community amis tab.

2. Make sure you're using atleast a 30GB volume on the Add Storage tab.

3. On the Configure Security Group tab, add rules to allow ports 80 and 8600.

Once your EC2 Instance is up, you can ssh into it using `ssh -i <aws_key>.pem ubuntu@<Public DNS>` and follow the instructions in readme.txt

### Mac OSX 10.11+

Follow the step-by-step instructions below:

1.  Install [rbenv](https://github.com/sstephenson/rbenv) (use the Basic GitHub Checkout method)

2.  Install [ruby-build](https://github.com/sstephenson/ruby-build) as an rbenv plugin:

        :::bash
        git clone https://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build

    Restart your shell at this point in order to start using your newly installed rbenv

3.  Clone the Autolab repo into home directory and enter it:

        :::bash
        cd ~/
        git clone https://github.com/autolab/Autolab.git && cd Autolab

4.  Install the correct version of ruby:

        :::bash
        rbenv install $(cat .ruby-version)

    At this point, confirm that `rbenv` is working (you might need to restart your shell):

        :::bash
        $ which ruby
        ~/.rbenv/shims/ruby

        $ which rake
        ~/.rbenv/shims/rake

5.  Install `bundler`:

        :::bash
        gem install bundler
        rbenv rehash

6.  Install the required gems (run the following commands in the cloned Autolab repo):

        :::bash
        cd bin
        bundle install

    Refer to the [FAQ](#faq) for issues installing gems

7.  Install one of two database options

    -   [SQLite](https://www.tutorialspoint.com/sqlite/sqlite_installation.htm) should **only** be used in development
    -   [MySQL](https://dev.mysql.com/doc/refman/5.7/en/osx-installation-pkg.html) can be used in development or production

8.  Configure your database:

        :::bash
        cp config/database.yml.template config/database.yml

    Edit `database.yml` with the correct credentials for your chosen database. Refer to the [FAQ](#faq) for any issues.

9.  Configure school/organization specific information (new feature):

        :::bash
        cp config/school.yml.template config/school.yml

    Edit `school.yml` with your school/organization specific names and emails

10. Configure the Devise Auth System with a unique key (run these commands exactly - leave `<YOUR-SECRET-KEY>` as it is):

        :::bash
        cp config/initializers/devise.rb.template config/initializers/devise.rb
        sed -i "s/<YOUR-SECRET-KEY>/`bundle exec rake secret`/g" config/initializers/devise.rb

    Fill in `<YOUR_WEBSITE>` in the `config/initializers/devise.rb` file. To skip this step for now, fill with `foo.bar`.

11. Create and initialize the database tables:

        :::bash
        bundle exec rake db:create
        bundle exec rake db:migrate

    Do not forget to use `bundle exec` in front of every rake/rails command.

12. Populate dummy data (development only):

        :::bash
        bundle exec rake autolab:populate

13. Start the rails server:

        :::bash
        bundle exec rails s -p 3000

14. Go to localhost:3000 and login with `Developer Login`:

        :::bash
        Email: "admin@foo.bar".

15. Install [Tango](/tango), the backend autograding service.

16. Now you are all set to start using Autolab! Visit the [Guide for Instructors](/instructors) and [Guide for Lab Authors](/lab) pages for more info.

### Ubuntu 14.04+

There are two ways to install Autolab on Ubuntu.

**Option 1**

The recommended way is to use the [OneClick option](/one-click). This option uses Docker to provide a complete installation of the Autolab frontend and Tango, for either development or production. Because it provides things like integration with SSL certificates and mail services, this option is specially useful for installing on external services like Heroku, EC2, DigitalOcean, or other Ubuntu VPS providers.

**Option 2**

Another option is to install the frontend and Tango manually, without using Docker. This gives you more control over the installation, but is only appropriate for advanced users with knowledge of the Unix command line, Rails, and Ruby Gems. To install the Autolab frontend in developer mode, run the following script:

```bash
AUTOLAB_SCRIPT=`mktemp` && \curl -sSL https://raw.githubusercontent.com/autolab/Autolab/master/bin/setup.sh > $AUTOLAB_SCRIPT && \bash $AUTOLAB_SCRIPT
```

When the script runs, you will be prompted for the `sudo` password and other confirmations. You can see the details of the script [here](https://github.com/autolab/Autolab/blob/master/bin/setup.sh). Once finished, [install Tango](/tango).

### Ubuntu 18.04+ (rails-5-upgrade)

This set of instruction is meant to install the currently in-development version of AutoLab located at [rails-5-upgrade](https://github.com/autolab/Autolab/tree/rails-5-upgrade) on 18.04 LTS.

1. Upgrade system packages and installing prerequisites
        
        :::bash
        sudo apt-get update
        sudo apt-get upgrade
        sudo apt-get install build-essential git libffi-dev zlib1g-dev autoconf bison build-essential libssl1.0-dev libyaml-dev libreadline6-dev libncurses5-dev libgdbm5 libgdbm-dev libmysqlclient-dev libjansson-dev ctags

2. Cloning Autolab repo from Github to ~/Autolab

        :::bash
        cd ~/
        git clone https://github.com/autolab/Autolab.git
        cd Autolab
        git checkout rails-5-upgrade
        git pull

3. Setting up rbenv and ruby-build plugin
        
        :::bash
        cd ~/
        git clone https://github.com/rbenv/rbenv.git ~/.rbenv
        echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
        echo 'eval "$(rbenv init -)"' >> ~/.bashrc
        source ~/.bashrc

        ~/.rbenv/bin/rbenv init
        git clone https://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build

4. Installing Ruby (Based on ruby version)

        :::bash
        cd Autolab
        rbenv install  `cat .ruby-version`

5. Installing SQLite
        
        :::bash
        sudo apt-get install sqlite3 libsqlite3-dev

6. Installing MySQL. (If you would like to only use SQLite, which is for testing & development only, you can skip this step.)
Following instructions from [How to Install MySQL on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-18-04). 


        :::bash 
        sudo apt install mysql-server
        sudo mysql_secure_installation

        > There will be a few questions asked during the MySQL setup.

        * Validate Password Plugin? N
        * Remove Annonymous Users? Y
        * Disallow Root Login Remotely? Y
        * Remove Test Database and Access to it? Y
        * Reload Privilege Tables Now? Y

7. (If you are using MySQL) Because a password rather than auth_socket is needed, we need to change the plugin that is used to access the root

        :::bash
        sudo mysql
        mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '<password>';
        mysql> FLUSH PRIVILEGES;
        mysql> exit;

8. Installing Rails

        :::bash
        cd Autolab
        gem install bundler
        rbenv rehash
        bundle install

9. Initializing Autolab Configs

        :::bash
        cd Autolab
        cp config/database.yml.template config/database.yml
        cp config/school.yml.template config/school.yml
        cp config/initializers/devise.rb.template config/initializers/devise.rb
        sed -i "s/<YOUR-SECRET-KEY>/`bundle exec rake secret`/g" config/initializers/devise.rb
        cp config/autogradeConfig.rb.template config/autogradeConfig.rb

10. (Using MySQL) Editing Database YML. 
Change the <username> and <password> fields in config/database.yml to the username and password that has been set up for the mysql. For example if your username is user1, and your password is 123456, then your yml would be

        :::yml
        development:
        adapter: mysql2
        database: user1_autolab_development
        pool: 5
        username: user1
        password: '123456'
        socket: /var/run/mysqld/mysqld.sock
        host: localhost

        test:
        adapter: mysql2
        database: user1_autolab_test
        pool: 5
        username: user1
        password: '123456'
        socket: /var/run/mysqld/mysqld.sock
        host: localhost

11. (Using SQLite) Editing Database YML.
Comment out the configurations meant for MySQL in config/database.yml, and insert the following

        :::yml
        development:
        adapter: sqlite3
        database: db/autolab_development
        pool: 5
        timeout: 5000

        test:
        adapter: sqlite3
        database: db/autolab_test
        pool: 5
        timeout: 5000

12. Granting permissions on the databases. Setting global sql mode is important to relax the rules of mysql when it comes to group by mode

        :::bash
        mysql --user=<username> -p
        (enter your password)
        mysql> grant all privileges on <username>_autolab_development.* to '<username>'@localhost;
        mysql> grant all privileges on <username>_autolab_test.* to '<username>'@localhost;
        mysql> SET GLOBAL sql_mode=(SELECT REPLACE(@@sql_mode,'ONLY_FULL_GROUP_BY','')); 
        mysql> exit

12. Initializing Autolab Database

        :::bash
        cd Autolab
        bundle exec rake db:create
        bundle exec rake db:reset
        bundle exec rake db:migrate

13. Populating sample course & students

        :::bash
        cd Autolab
        bundle exec rake autolab:populate

14. Run Autolab!

        :::bash
        cd Autolab
        bundle exec rails s -p 3000 --binding=0.0.0.0

15. Visit localhost:3000 on your browser to view your local deployment of Autolab, and login with `Developer Login`

        Email: "admin@foo.bar"


## FAQ

This is a general list of questions that we get often. If you find a solution to an issue not mentioned here,
please contact us at <autolab-dev@andrew.cmu.edu>

#### Ubuntu Script Bugs

If you get the following error

```bash
Failed to fetch http://dl.google.com/linux/chrome/deb/dists/stable/Release
Unable to find expected entry 'main/binary-i386/Packages' in Release file (Wrong sources.list entry or malformed file)
```

then follow the solution in [this post](http://askubuntu.com/questions/743814/unable-to-find-expected-entry-main-binary-i386-packages-chrome).

#### Where do I find the MySQL username and password?
If this is your first time logging into MySQL, your username is 'root'. You may also need to set the root password:

Start the server:

```bash
sudo /usr/local/mysql/support-files/mysql.server start
```

Set the password:

```bash
mysqladmin -u root password "[New_Password]"
```

If you lost your root password, refer to the [MySQL wiki](http://dev.mysql.com/doc/refman/5.7/en/resetting-permissions.html)

#### Bundle Install Errors
This happens as gems get updated. These fixes are gem-specific, but two common ones are

`eventmachine`

```bash
bundle config build.eventmachine --with-cppflags=-I/usr/local/opt/openssl/include
```

`libv8`

```bash
bundle config build.libv8 --with-system-v8
```

Run `bundle install` again

If this still does not work, try exploring [this StackOverflow link](http://stackoverflow.com/questions/23536893/therubyracer-gemextbuilderror-error-failed-to-build-gem-native-extension)

#### Can't connect to local MySQL server through socket
Make sure you've started the MySQL server and double-check the socket in `config/database.yml`

The default socket location is `/tmp/mysql.sock`.

#### I forgot my MySQL root password

You can reset it following the instructions on [this Stack Overflow post](http://stackoverflow.com/questions/6474775/setting-the-mysql-root-user-password-on-os-x)

If `mysql` complains that the password is expired, follow the instructions on the second answer on [this post](http://stackoverflow.com/questions/33326065/unable-to-access-mysql-after-it-automatically-generated-a-temporary-password)
