# PhpNuget V. 3.0.0.0

## Purpose

This is born to have my personal repository for nuget, on my cheapo PHP hostin

Verified on:

* PHP 5.3.17-IIS 7
* PHP 5.3.21-IIS 7
* PHP 5.5.8-Apache 2.0 (Windows 8 Pro)
* PHP 5.6-IIS 8
* PHP 5.4.2-Apache 2.0 (OpenSuse 13.1)

## Installation

### Prerequisities For IIS

These steps are NOT needed if your hosting already configured PHP

* Create the website with a standard web.config
* Install PHP for IIS (see at the end of this document 'Installation of PHP for IIS')
* Mark the location of *php-cgi.exe*

### Common

* Download from [Kendar.org](http://www.kendar.org/?p=/dotnet/phpnuget)
* Extract somewhere
* Copy the content of the directory "src" in the location you choose for the server. *DO NOT COPY THE WEB.CONFIG IF IN IIS*
* Enable write/read/delete access on
	* db: Directory where all databases and packages are stored
	* settings.php
	* Web.Config
	* .htaccess
* Verify to have write permissions on the "db" directory.
* Open the setup page at http://myhost/mynuget/setup.php and follow the wizard
* If under IIS set the path of 'php-cgi.exe' (leave blank if your hosting already configured PHP)
* Change the password, email and login of the administration without worries.
* Rename the setup.php to setup.bak
* Remove write access on 
	* settings.php
	* Web.Config
	* .htaccess
* Now open http://myhost/mynuget and see the gallery
* Happy Nugetting!

## Daily usage

* Configure your Visual Studio or Chocolatey to use http://myhost/mynuget/api/v2/ as repository
* To upload packages through command line:

<pre>
	nuget push mypackage.nupkg myApiKey http://myhost/mynuget/upload
</pre>

* To search items the syntax follow the OData specification (more or less). Some example
	* Id eq \'NUnit\': Search for all packages with id equals to NUnit
	* substringof(\'Microsoft\',Author): Search for all packages with Microsoft between the authors
	* Version gt \'1.0.1.0.beta\': Search for all packages with version greater than 1.0.1.0.beta
* All search blocks can be grouped with parenthesis and with 'and' and 'or' keywords.
* The keywords are
	* gt/gte: Greater/Greater equal
	* lt/lte: Lower/Lower equal
	* eq/neq: Equal/Not Equals
	* substringof: Check if the first parameter is contained in the second parameter

## Administration

### Simple user

* Must be registered to upload/modify packages
* Can only edit the packages that he uploaded
* When an user upload for the first time a package with a certain id, then all versions will be owned by the same user. No other users but the administrators could modify or upload packages with the same id.
* The token available to access the API for the upload can be regenerated by any user. 
* The API token can be used with our without the curly braces.
* The users will see in their list of packages only the packages that they uploaded.

### For the administrators

* They can add/remove/enable users
* They can add/modify any package. When a package uploaded initially by another user is uploaded by the administrator, the ownership of the package will be kept the same.
* It is possible to download packages from external sources. It will be sufficent to insert the address from which the packages would be taken, specifying where should be put the identifier and the version of the package to download locally (through the @ID and @VERSION tags)
* When the database is corrupted it can be regenerated through "Refresh packages db from packages directory." mind that all the packages will become owned by the administrators.
* It would be possible to upload the packages directly on the 'db/packages' directory and then run the "Refresh packages db from packages directory.". All packages not yet present on the database will be loaded.

## The Api

The nuget API is based more or less on the OData protocol.
All of the api lsited that returns a collection support the usage of parameters 

* $skip: Optional. The number of items to skip. Default to 0.
* $top: Optional. The number of items returned. Default to 10.

### Api V1

* /api/v1: Retrieves the root for the entities that will be used by the API. No parameters.
* /api/v1/package/\[package-id\]/\[package-version\]: Download the specified package. No parameters.
* /api/v1/$metadata: Retrieves the OData metadata, the actions allowed and the entities specifications. No parameters.
* /api/v1/FindPackagesById(): Search for packages by id, returns all packages with a certain id ordered by version descending
	* Id: Mandatory parameter, specify the identifier of the package (e.g. Angular-UI-Router)
* /api/v1/FindPackagesById()/$count: Count all packages with a certain id ordered by version descending. Same parameters as 'FindPackagesById()'
* /api/v1/Search(): Search for packages satisfyng the query with a certain id ordered by version descending
	* $filter: Optional. The query that will be used for the search, see 'Daily usage' for the syntax
	* $orderby: Optional. List of fields for the order by. e.g. 'Id desc, Version, Author asc' will order by Id descending, then by Version ascending (the default) then by Author ascending.
	* searchTerm: Optional. Equivalent to write 'substringof(\'searchTerm\',Id) or substringof(\'searchTerm\',Name)'
	* targetFramework: Optional. Target framework required, the result will contain the matching packages and the ones without framework specified.
	* includePrerelease: Optional. If set to 'true' include event the prereleases (the ones with the flag set by hand on package editing or the ones with versions that contains alphabetic characters, e.g. 1.0.0.1 is not prerelease but 1.0.beta is)
* /api/v1/Search()/$count: Count all packages satisfyng the query. Same parameters as 'FindPackagesById()'
* /api/v1/Packages(): Same as Search()
* /api/v1/Packages()/$count: Same as Search()/$count
* /api/v1/Packages: Same as Search()
* /api/v1/Packages/$count: Same as Search()/$count

### Api V2

All v1 APIs are present, remind to replace the v1 in the previous section with v2!

* /api/v2/FindPackageById(): Same as FindPackagesById()
* /api/v2/GetUpdates(): Search for packages satisfyng the query with a certain id ordered by version descending
	* packageIds: Optional. Pipe (|) separated list of package ids
	* versions: Optional. Pipe (|) separated list of package versions, their position is matching with the ones in packageIds
	* targetFrameworks: Optional. Pipe (|) separated list of package target frameworks, their position is matching with the ones in packageIds. If the framework is not specified in the package then it will be returned anyway
	* includePrerelease: Optional. If set to 'true' include event the prereleases (the ones with the flag set by hand on package editing or the ones with versions that contains alphabetic characters, e.g. 1.0.0.1 is not prerelease but 1.0.beta is)
	* includeAllVersions: Not supported yet
	* versionConstraints: Not supported yet
* /api/v2/GetUpdates()/$count: Count all packages satisfyng the query. Same parameters as 'FindPackagesById()'

### Api V3

As soon as the guys from nuget defines it.

### Other entry points

* /packages?q=term1 term2: Given the terms passed in 'q' a search is made checking that the Id or Name of the package corespond to at least one of the term passed as parameter
* /upload: The access point to upload the packages.

## Installation of PHP for IIS

### Installation for IIS 8

Tested on Windows 8 pro X64 bit and Windows 8.1 pro X64

This guid is adapted from [iis.net](http://www.iis.net/learn/application-frameworks/install-and-configure-php-on-iis/using-php-manager-for-iis-to-setup-and-configure-php)

1. download [php for windows](http://windows.php.net/)
2. extract downloaded zip ( C:php )
3. download [php manager[(http://phpmanager.codeplex.com/releases) for IIS ( its an extension  for managing PHP from IIS control panel )
4. open php manager and click on register new php installation
5. choose php-cgi.exe and click ok (usually is inside 'C:\Program Files (x86)\PHP\v5.3\php-cgi.exe')
6. now check phpinfo and choose error reporting
7. open your favorite code editor and type

<?php echo "Hello World !" ; ?>

and save this as hello.php ( or anything ) on c:inetpubwwwroot
and open http://localhost/hello.php

now IIS is serving PHP now in next tutorial i will show you how to set up MySQL in windows

### Features of PHP manger

* Register PHP with IIS
* Validate and properly configure existing PHP installations
* Run multiple PHP versions side by side on the same server and even within the same web site
* Check PHP runtime configuration and environment (output of phpinfo() function)
* Configure various PHP settings
* Enable or disable PHP extensions
* Remotely manage PHP configuration in php.ini file.
* Easily install, configure, manage and troubleshoot one or many PHP versions on the same IIS server.


## TODO

### Must

* Add captcha and max retries of password for users
* Verify that 'GetUpdates' work as nuget expects.
	* Support includeAllVersions
	* Support versionConstraints	
* Support Packages(Id='AngularJS.Core',Version='1.2.14') on v1 and v2 API
* Support /packages/id to select all packages with the given id on the gallery
* Support /packages/id/version to select a package on the gallery
* Support on v1 the alternate address https://www.nuget.org/v1/FeedService.svc/ for https://www.nuget.org/api/v1/

### Should

* Add robots.txt
* Port to MySQL
* Import v3.0.0.0 txt database on MySQL

### Would

* Support viewing the profile in gallery, "nuget like" https://www.nuget.org/profiles/kendar.org
* Add Email token to reset users passwords
* Upload multiple nupkg
* Align the Settings::ResultsPerPage in all the views

## Updates

* 3.0.0.0
	* UI improved with AngularJS
	* Improved the search engine with better OData query support
	* Changed the database fields (manual import only)
	* Downloading of packages from other repositories
	* Better support for the standard nuget server behaviours

* 2.0.0.0
	* Added support for GetPackageById
	* General improvements and bugfixing
	* Support for installation on IIS

* 1.0.0.0
	* Added interface to edit packages
	* Added mod_rewrite support via .htaccess
	* Simplified the whole structure of the program
