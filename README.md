![Build and populate cache](https://github.com/imincik/geonix/workflows/Build%20and%20populate%20cache/badge.svg)
[![Cachix Cache](https://img.shields.io/badge/cachix-geonix-blue.svg)](https://geonix.cachix.org)

# Geonix - the geospatial environment built on Nix

**WARNING: this project is safely usable right now, but things are in active
development. Incompatible changes without any notice can occur at any time !**

Geospatial software environment is a complex system of software packages, tools
and libraries built to work together. For example [GDAL](https://gdal.org/),
[PROJ](https://proj.org/) or [GEOS](https://libgeos.org/) libraries are core
components of most of the free (and many non-free) geospatial tools, desktop
applications or database systems. Change of such core library has immediate
impact on all depending software. On traditional systems, it is very difficult
to update them without breaking whole environment.

Various components of this environment are usually coming from different sources
and in different versions depending on platform (Linux, Mac or Windows),
platform version (or Linux distribution and its version) and other additional
sources such as different Linux users repositories (PPAs, AURs, ...), Mac
Homebrew, OSGeo4W, Python PyPi, and many others.

And to make it even worse, different projects usually have different, very often
conflicting software requirements.

The primary goal of Geonix project is to bring cross-platform, consistent,
reproducible and up-to-date geospatial environment for geospatial software
developers, data analysts, scientists and end users which allows to build and
run software in isolated environments as needed by particular use case without
breaking other things.


## About Nix

[Nix](https://nixos.org/) is the most advanced and unique package manager.
[Nix packages (nixpkgs)](https://github.com/NixOS/nixpkgs) is the largest collection
of up-to-date free software in the world
(see [repology comparison](https://repology.org/repositories/graphs)).

Nix runs on Linux, Mac and inside of Windows WSL with very minimal interference
with host system software and configuration.

Beside the installation of Nix package manager itself, installation of any Nix
package or project environment doesn't have any impact on the host system or
other environments. Everything what Nix installs is stored in the `/nix/store`
directory. All `/nix/store` content is stored under uniquely bit-to-bit
reproducible cryptographic hash and is shared across whole Nix system for all
components requiring that exact content.

TODO: add information how Nix software is provided to the user

Nix ecosystem offers many very unique ideas and tools which are not found
anywhere else in IT industry.


## About Geonix

Geonix is built on top of Nix, Nix packages (nixpkgs) and unique tooling provided
by this ecosystem.

Geonix is providing latest versions of the core geospatial libraries such as
GDAL, GEOS, PDAL or PROJ, growing selection of Python libraries, desktop
software like QGIS and PostgreSQL/PostGIS database system, everything always
consistently built together.

Using the new Nix feature called Flakes, Geonix provides very easy way to build
isolated geospatial software environments (think about it as a Python
virtualenv, but containing all software above operating system kernel which is
required to build, test, run and package the project) which are bit-to-bit (or
bug-to-bug) reproducible on other machines at any time in the future.

Whole Nix Flake environment (all software coming from nixpkgs, Geonix or other
Flakes) is pinned to the exact Git commit revision using lock file and can be
updated only when required.

New Flake projects can be started from templates and applications can be
packaged as OCI compatible container images or can be launched directly from
source code repository (for example from GitHub).

Nix provides strong dependencies management. In case of any package changes in
the environment, Nix guarantees that all package reverse dependencies will be
automatically rebuilt. For example in case of GDAL version update or even after
any build configuration change, all reverse dependencies (like QGIS or Fiona)
are automatically rebuilt.

Such environments makes software development, distribution, deployment and bug
fixing much more reliable and easier and can significantly lower the time needed
for a new developer to pick up a new project or lower the time needed to get on
speed when returning to the project after longer time period.


## Packages

For a list of Geonix maintained packages see [pkgs directory](pkgs/).


## Usage

### Install and configure Nix

* Install Nix on non-NixOS Linux (Ubuntu, Fedora, ...)
```
sh <(curl -L https://nixos.org/nix/install) --daemon
```

* Enable new Nix command interface and Nix Flakes
```
echo "experimental-features = nix-command flakes" | sudo tee -a /etc/nix/nix.conf
```

* Add current user to Nix trusted users group
```
echo "trusted-users = $USER" | sudo tee -a /etc/nix/nix.conf
```

* Restart Nix daemon
```
sudo systemctl restart nix-daemon.service
```

For Nix installation on Mac or Windows (WSL2) see
[Install Nix documentation](https://nix.dev/tutorials/install-nix#install-nix) .


### Explore Geonix Flake content

* Explore Flake content
```
nix flake show github:imincik/geonix
```

### Try applications without installation

QGIS is available only for Linux, other platforms will be available soon.

* Launch latest stable QGIS version
```
nix run github:imincik/geonix#qgis
```

* Launch QGIS LTR version
```
nix run github:imincik/geonix#qgis-ltr
```

### Install applications permanently

* Install latest stable QGIS version
```
nix profile install github:imincik/geonix#qgis
```

* Install QGIS LTR version
```
nix profile install github:imincik/geonix#qgis-ltr
```

### Try CLI shell

* Enter shell containing Python interpreter and basic geospatial CLI tools
```
nix develop github:imincik/geonix#cli
```

* Launch Python interpreter
```
[geonix] > python -c "import fiona; print(fiona.supported_drivers)"

{'DXF': 'rw', 'CSV': 'raw', 'OpenFileGDB': 'r', 'ESRIJSON': 'r', ... }
```

* Launch gdalinfo
```
[geonix] > gdalinfo --version

GDAL 3.6.1, released 2022/12/14
```

### Try PostgreSQL/PostGIS shell

* Enter shell containing running PostGIS database
```
nix develop github:imincik/geonix#postgres
```

* Connect to PostGIS database and run query
```
[geonix] > psql -c "CREATE EXTENSION postgis;"

CREATE EXTENSION

[geonix] > psql -c "SELECT ST_AsText(ST_Buffer(ST_GeomFromText('POINT(1 1)'), 1));"

POLYGON((2 1,1.98078528040323 0.804909677983872,1.923879532511287
0.61731656763491, ...
```

### Start new project from templates

Geonix provides multiple ready to use Flake templates for starting different types of geospatial
projects on top of Geonix environment.

Current list of templates:

* [postgis-db](templates/postgis-db): run PostgreSQL/PostGIS database with PgAdmin for queries
* [python-app](templates/python-app): Python development environment with Poetry managed dependencies
* [python-containers](templates/python-containers): build and run Python and Jupyter container images

Flake templates must be initialized in Git-initialized directory.

* Create new project directory and initialize it with Git
```
mkdir my-project
cd my-project
git init
```

* Initialize new Flake project from template
```
nix flake init --accept-flake-config --template github:imincik/geonix#<TEMPLATE>
```

* Follow instructions printed during Flake initialization


## Geonix development

* Build single package
```
nix build .#<PACKAGE>
```

* Build all packages
```
nix build .#all-packages
```

* Run package passthru tests
```
nix-build -A packages.x86_64-linux.<PACKAGE>.passthru.tests
```

_To re-build already built package or to re-run already succeeded tests use
`--check` switch._

* Explore package store path content
```
nix path-info -rsSh .#<PACKAGE> | sort -nk3
```

* Explain package dependencies
```
nix why-depends .#<PACKAGE> .#<DEPENDENCY>
```


_Credits belongs to all current and past [Nix](https://github.com/NixOS/nix/graphs/contributors),
[Nixpkgs and NixOS](https://github.com/NixOS/nixpkgs/graphs/contributors) developers and
maintainers._

