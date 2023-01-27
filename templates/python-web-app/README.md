# Example Python web application

This is a example web application demonstrating development of Python
application using dependencies provided by multiple sources.

Application data can be generated by local Python server or by
PostgreSQL/PostGIS database backend.


## Dependencies

### Provided by Geonix

* shapely
* PostgreSQL/PostGIS DB container image

### Provided by Nixpkgs

* matplotlib

### Provided by Poetry

* black
* flask
* isort
* pytest


## Template initialization

```
mkdir my-project
cd my-project

git init

nix flake init --accept-flake-config --template github:imincik/geonix#python-web-app

nix flake lock
git add *
```


## Usage

### Development

* Enter development shell

```
nix develop
```

* Install Python virtual environment using Poetry

```
poetry env use $(which python)
poetry install
```

* Launch Python application development server (with local data backend)

```
poetry run flask --app src/python_app run --reload
```

* Launch Python application development server (with database data backend)

```
geonix build geonix-postgresql-image
docker load < ./result
docker-compose up -d

BACKEND=db poetry run flask --app src/python_app run --reload
```

* Exit development shell

```
exit
```

### PostgreSQL

* Connect to PostgreSQL DB container (run in development shell)

```
psql
```

### Geonix CLI

* Search for additional packages (run in development shell)

```
geonix search <PACKAGE>
```


## Deployment

* Build and load base container image

```
nix develop --command geonix build geonix-base-image
docker load < ./result
```

* Test container entrypoint script (optional)

```
nix build .\#deployment
./result/bin/entrypoint
```

* Build a container image

```
docker build -t python-app:latest .
```

* Run container

```
docker run --rm -p 8000:8000 python-app:latest
```


## Customized packages

See
[CUSTOMIZED-PACKAGES.md](https://github.com/imincik/geonix/blob/master/CUSTOMIZED-PACKAGES.md)
for instructions how to customize packages.


## More info

* [Zero to Nix](https://zero-to-nix.com/)