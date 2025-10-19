invoke
======

https://www.pyinvoke.org/  
https://github.com/pyinvoke/invoke (нужен VPN)  
```shell
pip install invoke  
```

# 0. It relies on `tasks.py`


So it has to be gitignored `tasks.py` at the very root of a git repo.  
And a not-gitignored `tasks.py.tmpl`.  
  
Celerys' conf file have to be `app.py` or something.  
Or it'll confuse some IDEs when one `import tasks`.

# 1. Minimal example:

```
@task
def hi(c):
    c.run(f'echo Hi!, pty=True)
```

# 2. Optional params:

```
@task
def up(c, service=''):
    c.run(f'docker compose up {service}, pty=True)
```

All valid:

```shell
invoke up
invoke up --service container_name
invoke up --service=container_name
invoke up --service='container_name'
```

# 3. Params separated by whitespaces must be in string literals:

```shell
invoke up --param='multiple pieces'
```

# 4. Symlinking:

```
@task
def a(c):
    b(c)
```

`a = b` will not work.

# 5. `pty=True` is required

For a pure restream — colored and interactive.
 