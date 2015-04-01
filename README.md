# Mojolicious_Lite_on_OpenShift
How to make Mojolicious::Lite work in a Perl 5.10 cartridge on OpenShift

##Introduction

Anyone that has tried to make Mojolicious::Lite work on OpenShift has probably ended up being very frustrated by a combination of outdated or just no information at all. Hopefully my pain will help you.

This tutorial assumes:

* you have created an account on [OpenShift](https://openshift.redhat.com)
* if you plan to use the command line interface, have the [Client Tools](https://developers.openshift.com/en/managing-client-tools.html) installed
* you have the [git Source Code Management](http://git-scm.com/downloads) tools installed & are familiar with its use

Some of the commands will be specific to your environment:

* \_\_DOMAIN\_\_ refers to your OpenShift domain name you have defined in your [settings](https://openshift.redhat.com/app/console/settings)
* \_\_APPNAME\_\_ refers to the name you choose to create your application with
* \_\_APP_ID\_\_ refers to your OpenShift application identifier

##Steps

Create a new OpenShift application with the [Perl 5.10 Cartridge](https://developers.openshift.com/en/perl-overview.html), either in the OpenShift online or using the client tools. Replace \_\_APPNAME\_\_ with your application name:
```
rhc app create __APPNAME__ perl-5.10
```

Change to the application directory you just created: 
```
cd __APPNAME__
```

Determine your application id. This can be done using the client tools or online:
```
rhc ssh __APPNAME__
```

Copy the 24 character hexadecimal string, e.g. **551c3c605973ca4a4e00018f**. This is your application id.

The application id can also be determined in OpenShift online by looking at the git clone URL on the top right, e.g. ssh://**551c3c605973ca4a4e00018f**@\_\_APPNAME\_\_-\_\_DOMAIN\_\_.rhcloud.com/~/git/\_\_APPNAME\_\_.git/

Edit the **.htaccess** file, replacing **\_\_APP_ID\_\_** with your application id:
```
SetHandler perl-script
PerlResponseHandler Plack::Handler::Apache2
PerlSetVar psgi_app /var/lib/openshift/__APP_ID__/app-root/runtime/repo/index.pl
RewriteEngine on
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-l
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{DOCUMENT_ROOT}/%{REQUEST_URI} !-f
RewriteRule (.*) /index.pl/$1 [QSA,L]
```

Edit the **.openshift/cpan.txt** file. All perl dependencies go in here during your development:
```
Time::HiRes
Mojolicious::Lite
Plack::Handler::Apache2
```

The file **index.pl** is where your Mojolicious code goes. For a quick test, replace the contents with this:
```
    use Mojolicious::Lite;

    get '/' => sub {
       my $self = shift;
       $self->render(text => "Hello World!");
    };

    app->start;
```

Check in code:
```
git commit -am "Initial code check-in"
git push
```

Access your site at:
```
http://__APPNAME__-__DOMAIN__.rhcloud.com
```
