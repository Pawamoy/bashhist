# Changelog
All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](http://keepachangelog.com/en/1.0.0/)
and this project adheres to [Semantic Versioning](http://semver.org/spec/v2.0.0.html).

## [v0.2.0](https://gitlab.com/pawamoy/shell-history/tags/v0.2.0) ([compare](https://gitlab.com/pawamoy/shell-history/compare/v0.1.0...v0.2.0)) - 2018-12-23

- Package the application ([edaf15b](https://gitlab.com/pawamoy/shell-history/commit/edaf15b7424d40ef13442be03ae04828eb80571d)).
  
  The application is now packaged as a Python package. It is easier to install and setup.
  - To install it: `pip install shellhistory`.
  - To enable it: `. $(shellhistory-location); shellhistory enable` at shell startup.
  
  **BREAKING CHANGES:**
  - The default directory in which `shellhistory` reads the history file and write the SQlite3 database file
  is now `~/.shellhistory` instead of `~/.shell_history`.
  - The SQlite3 database file is now named `db.sqlite3` instead of `db`.
  
    The migration is very easy:
    - rename the directory: `mv ~/.shell_history ~/.shellhistory`,
    - and the database file: `mv ~/.shellhistory/db ~/.shellhistory/db.sqlite3`.
    
    You can even skip renaming the database file:
    delete it and reimport your data with `shellhistory-cli --import`
    
  Please remember this application is alpha software, and is subject to change without guarantee of backward compatibility.
   
## [v0.1.0](https://gitlab.com/pawamoy/shell-history/tags/v0.1.0) ([compare](https://gitlab.com/pawamoy/shell-history/compare/4a9781fb20047c4c5f9d7bd04f60db5e35295070...v0.1.0)) - 2018-12-23

- Initial version without packaging.
